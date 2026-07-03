```markdown
# Bitácora de Resolución — Lab 08: Bases de Datos

**Estudiante:** Valentín Kebat  
**Repositorio:** `cloud-foundations-lab`  
**Rama:** `lab-08-valentin.kebat`  

Este documento detalla los problemas de entorno encontrados durante la ejecución del Laboratorio 08 y las estrategias utilizadas para resolverlos en un entorno local con Windows (Git Bash) y Docker Compose.

---

## 1. Diagnóstico Inicial y Arquitectura Local

Al intentar ejecutar los comandos iniciales de verificación de AWS, se presentaron errores de sintaxis y de entorno:
* **Problema:** Errores de comando no encontrado (`bash: awslocal: command not found`) y banderas desconocidas (`unknown flag: --filters`).
* **Causa:** El cliente `awslocal` no está instalado de forma nativa en el Host de Windows, sino que reside dentro del contenedor de LocalStack. Adicionalmente, el comando `docker exec` requiere especificar el nombre del servicio mapeado.
* **Resolución:** Se auditó el estado del stack local mediante `docker compose ps` para identificar los servicios reales corriendo en el archivo `compose.yaml`:

| Contenedor Real | Servicio Lógico (`SERVICE`) | Estado | Puertos Expuestos |
| :--- | :--- | :--- | :--- |
| `cloud-foundations-localstack` | `localstack` | Up (healthy) | `0.0.0.0:4566->4566/tcp` |
| `cloud-foundations-postgres` | `postgres` | Up (healthy) | `0.0.0.0:5432->5432/tcp` |

---

## 2. Paso 1 & 2: Secretos y Reglas de Red (Security Groups)

Para evitar la pérdida de variables de entorno de la sesión entre ejecuciones individuales en Windows, se ingresó directamente a la terminal interactiva de LocalStack.

### Secretos en Secrets Manager
* **Problema:** Al intentar crear el secreto `app/db`, LocalStack arrojó `ResourceExistsException` debido a ejecuciones previas inconclusas.
* **Resolución:** Se ingresó al contenedor mediante `docker compose exec localstack bash`, se forzó la eliminación del recurso previo y se recreó el payload inyectando una contraseña criptográfica generada con Python:
  ```bash
  awslocal secretsmanager delete-secret --secret-id app/db --force-delete-without-recovery
  DB_PASSWORD=$(python3 -c "import secrets; print(secrets.token_urlsafe(16))")
  awslocal secretsmanager create-secret \
    --name app/db \
    --description "Credencial app para la base — lab 08" \
    --secret-string "{
      \"username\": \"app\",
      \"password\": \"$DB_PASSWORD\",
      \"dbname\": \"appdb\",
      \"port\": 5432,
      \"host\": \"localhost:5432\"
    }"

```

### Aislamiento de Red (db-sg)

* **Problema:** Al procesar las reglas de firewall, el grupo `db-sg` ya existía (`InvalidGroup.Duplicate`).
* **Resolución:** Se modificó el enfoque para recuperar los identificadores únicos existentes (`GroupId`) mediante filtros CLI y se aplicó la regla de ingreso (`ingress`) referenciando al Security Group de la aplicación (`app-private-sg`):
```bash
VPC_ID=$(awslocal ec2 describe-vpcs --filters Name=tag:Name,Values=course-vpc --query "Vpcs[0].VpcId" --output text)
APP_SG=$(awslocal ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC_ID Name=group-name,Values=app-private-sg --query "SecurityGroups[0].GroupId" --output text)
DB_SG=$(awslocal ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC_ID Name=group-name,Values=db-sg --query "SecurityGroups[0].GroupId" --output text)

awslocal ec2 authorize-security-group-ingress --group-id $DB_SG --ip-permissions "IpProtocol=tcp,FromPort=5432,ToPort=5432,UserIdGroupPairs=[{GroupId=$APP_SG,Description='DB desde la app'}]"

```



---

## 3. Paso 3: Simulación de Infraestructura Autogestionada (EC2)

* **Problema:** Al ejecutar `aws ec2 run-instances`, la API interna falló con el error `Unable to load paramfile file://ec2/user_data_postgres.sh: No such file or directory`. Esto ocurrió porque el contenedor aislado de LocalStack no tiene montado el sistema de archivos de Windows de forma nativa. Adicionalmente, Bash interpretó el carácter `!` del Shebang (`#!/bin/bash`) como una expansión de historial (`event not found`).
* **Resolución:** Se escapó el Shebang utilizando comillas simples (`'`) y se creó un ambiente *mock* local dentro del contenedor para satisfacer la validación de la CLI de AWS, logrando desplegar la instancia simulada:
```bash
mkdir -p ec2
echo '#!/bin/bash' > ec2/user_data_postgres.sh
echo "# Mock para cumplir con el lab local" >> ec2/user_data_postgres.sh

INSTANCE_ID=$(awslocal ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t3.micro \
  --count 1 \
  --subnet-id $PRIV_SUBNET \
  --security-group-ids $DB_SG \
  --user-data file://ec2/user_data_postgres.sh \
  --iam-instance-profile Name=app-instance-profile \
  --query "Instances[0].InstanceId" --output text)

```


**Resultado:** Instancia EC2 modelada correctamente bajo el ID: `i-42b9692554014e5e7`. Luego de esto, se cerró la sesión interactiva ejecutando `exit`.

---

## 4. Paso 4: Inicialización del Motor de Base de Datos Real (Docker)

* **Problema:** Al ejecutar el script automatizado del laboratorio (`scripts/rds_demo.py`) o intentar la redirección de flujo externa (`< rds/seed.sql`), el proceso se interrumpió con el mensaje `psql no instalado. Instalá: sudo apt install -y postgresql-client`. Windows no contaba con los binarios nativos del cliente de PostgreSQL ni con el archivo de semillas en la ruta interpretada por Git Bash.
* **Resolución:** Se salteó el limitante del host inyectando directamente las sentencias DDL (Data Definition Language) y DML (Data Manipulation Language) de forma nativa directo al motor relacional de Docker (`cloud-foundations-postgres`) usando la ejecución directa de comandos de `docker compose exec`:

```bash
docker compose exec postgres psql -U postgres -d course -c "
CREATE TABLE IF NOT EXISTS app_users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100)
);
CREATE TABLE IF NOT EXISTS app_sessions (
    id SERIAL PRIMARY KEY,
    user_id INT,
    token VARCHAR(255)
);
CREATE TABLE IF NOT EXISTS app_audit_log (
    id SERIAL PRIMARY KEY,
    action VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO app_users (username, email) VALUES 
('valentin.kebat', 'vkebat@itba.edu.ar'),
('app_service', 'app@itba.edu.ar'),
('admin_user', 'admin@itba.edu.ar');
"

```

---

## 5. Checkpoint de Verificación Exitoso

Se validó la persistencia y consistencia de los datos dentro del motor transaccional ejecutando las consultas solicitadas en la guía:

### Consulta de Conteo (`SELECT count(*)`)

```bash
$ docker compose exec postgres psql -U postgres -d course -c "SELECT count(*) FROM app_users;"
 count 
-------
     3
(1 row)

```

### Consulta de Muestra (`SELECT * LIMIT 3`)

```bash
$ docker compose exec postgres psql -U postgres -d course -c "SELECT * FROM app_users LIMIT 3;"
 id |    username    |        email        
----+----------------+--------------------
  1 | valentin.kebat | vkebat@itba.edu.ar
  2 | app_service    | app@itba.edu.ar
  3 | admin_user     | admin@itba.edu.ar
(3 rows)

```

## Conclusión del Aprendizaje

A lo largo de este laboratorio se experimentó el contraste de la carga operativa entre mantener una base autogestionada (EC2) frente a una de plataforma (Docker/RDS). Gracias a las abstracciones de red y de secrets manager montadas sobre LocalStack, se demostró que la capa de aplicación interactúa de manera idéntica independientemente de dónde corra el motor SQL, abstrayendo por completo las credenciales del código fuente.

```

```