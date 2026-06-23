# Practica en consola

```bash
Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (main)
$ docker compose up -d
unable to get image 'minio/minio': error during connect: Get "http://%2F%2F.%2Fpipe%2FdockerDesktopLinuxEngine/v1.51/images/minio/minio/json": open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified.

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-06-ValentínKebat)
$ docker compose up -d
[+] Running 0/5
 - Container cloud-foundations-redis       Starting                                                                                                   0.0s 
[+] Running 0/5oud-foundations-minio       Starting                                                                                                   0.0s 
 - Container cloud-foundations-redis       Starting                                                                                                   0.1s 
[+] Running 0/5oud-foundations-minio       Starting                                                                                                   0.1s 
 - Container cloud-foundations-redis       Starting                                                                                                   0.2s 
[+] Running 0/5oud-foundations-minio       Starting                                                                                                   0.2s 
 - Container cloud-foundations-redis       Starting                                                                                                   0.3s 
[+] Running 0/5oud-foundations-minio       Starting                                                                                                   0.3s 
 - Container cloud-foundations-redis       Starting                                                                                                   0.4s 
[+] Running 0/5oud-foundations-minio       Starting                                                                                                   0.4s 
 - Container cloud-foundations-redis       Starting                                                                                                   0.5s 
[+] Running 0/5oud-foundations-minio       Starting                                                                                                   0.5s 
 - Container cloud-foundations-redis       Starting                                                                                                   0.6s 
[+] Running 1/5oud-foundations-minio       Starting                                                                                                   0.6s 
 ✔ Container cloud-foundations-redis       Started                                                                                                    0.7s 
[+] Running 1/5oud-foundations-minio       Starting                                                                                                   0.7s 
 ✔ Container cloud-foundations-redis       Started                                                                                                    0.7s 
[+] Running 1/5oud-foundations-minio       Starting                                                                                                   0.8s 
 ✔ Container cloud-foundations-redis       Started                                                                                                    0.7s 
[+] Running 1/5oud-foundations-minio       Starting                                                                                                   0.9s 
 ✔ Container cloud-foundations-redis       Started                                                                                                    0.7s 
[+] Running 1/5oud-foundations-minio       Starting                                                                                                   1.0s 
 ✔ Container cloud-foundations-redis       Started                                                                                                    0.7s 
[+] Running 3/5oud-foundations-minio       Starting                                                                                                   1.1s 
 ✔ Container cloud-foundations-redis       Started                                                                                                    0.7s 
[+] Running 3/5oud-foundations-minio       Started                                                                                                    1.1s 
 ✔ Container cloud-foundations-redis       Started                                                                                                    0.7s 
[+] Running 5/5oud-foundations-minio       Started                                                                                                    1.1s 
 ✔ Container cloud-foundations-redis       Started                                                                                                    0.7s 
 ✔ Container cloud-foundations-minio       Started                                                                                                    1.1s 
 ✔ Container cloud-foundations-localstack  Started                                                                                                    1.4s 
 ✔ Container cloud-foundations-redpanda    Started                                                                                                    1.4s 
 ✔ Container cloud-foundations-postgres    Started                                                                                                    1.1s 

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-06-ValentínKebat)
$ awslocal iam get-role --role-name app-role --query "Role.Arn"
bash: awslocal: command not found

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-06-ValentínKebat)
$ python scripts/iam_demo.py
=== Lab 04 — IAM demo ===

AVISO: LocalStack Community no enforcea policies (Deny no bloquea).
       Practicamos la mecánica: crear, adjuntar, asumir.

1. Bucket S3
  bucket 'course-data-raw' creado con objeto de ejemplo

2. Grupo + policy administrada
  grupo 'bigdata-read' creado
  policy 'S3ReadOnlyLab' creada: arn:aws:iam::000000000000:policy/S3ReadOnlyLab
  policy adjuntada al grupo 'bigdata-read'

3. Usuario → grupo
  usuario 'lab-user' creado
  usuario 'lab-user' agregado al grupo 'bigdata-read'
  access key creada: LKIAQAAAAAAAAKOUYNYF (larga duración — evitar en prod)

4. Rol con trust policy (EC2) + inline policy mínima
  rol 'app-role' creado
  inline policy 'InlineS3Read' adjuntada al rol 'app-role'

5. AssumeRole vía STS → credenciales temporales

  asumiendo rol: arn:aws:iam::000000000000:role/app-role
  AccessKeyId:  LSIAQAAAAAAANI4MJIFC
  Expiration:   2026-06-23 23:49:00.308000+00:00  ← credencial temporal
  objetos en 'course-data-raw' con credenciales temporales:
    - sample/hello.txt (17 bytes)

=== Resumen de recursos creados ===
  Bucket:  course-data-raw
  Grupo:   bigdata-read
  Policy:  arn:aws:iam::000000000000:policy/S3ReadOnlyLab
  Usuario: lab-user
  Rol:     arn:aws:iam::000000000000:role/app-role

Listo. Revisá los JSON en iam/ para entender cada documento.

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-06-ValentínKebat)
$ python scripts/ec2_demo.py
=== Lab 05 — EC2 demo: IAM → EC2 → S3 ===

AVISO: LocalStack Community mock-ea EC2.
       El flujo CLI/API es real; la VM no arranca y el user-data no se ejecuta.

1. Key pair
  key pair 'lab05-key' creada (fingerprint: dc:0b:52:f8:76:14:a6...)

2. Security group + reglas de ingress
  security group 'web-sg' creado: sg-4a3b75d75fadc53fd
  ingress permitido: tcp/80
  ingress permitido: tcp/22

3. Instance profile (wrapper del rol app-role del lab-04)
  instance profile 'app-instance-profile' ya existe
  rol 'app-role' adjuntado al profile
   profile ARN: arn:aws:iam::000000000000:instance-profile/app-instance-profile

4. run-instances con user-data + profile
  instancia lanzada: i-fc06b0ad2c4af6982 (t3.micro, AMI ami-0c02fb55956c7d316)

5. describe-instances — ver lo que quedó aprovisionado
  estado: running
  AMI:    ami-0c02fb55956c7d316
  type:   t3.micro
  SG:     ['web-sg']
  profile: arn:aws:iam::000000000000:instance-profile/app-instance-profile

6. describe-instance-attribute — user-data almacenado
  user-data cargado (1411 chars). Primera línea: '#!/bin/bash'

=== Resumen ===
  Key pair:         lab05-key
  Security group:   web-sg (sg-4a3b75d75fadc53fd)
  Instance profile: app-instance-profile
  Instancia:        i-fc06b0ad2c4af6982

Para terminar la instancia (no olvidar — cattle, not pets):
  awslocal ec2 terminate-instances --instance-ids i-fc06b0ad2c4af6982

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-06-ValentínKebat)
$ scripts/s3_demo.py
scripts/s3_demo.py: line 14: 
Lab 06 — S3 demo: data lake del módulo + cierre IAM → EC2 → S3.

Crea el bucket course-data-lake como fuente de verdad durable del curso:
  - Block Public Access ON, encryption SSE-S3, versioning desde día 1
  - Carga Olist + GitHub Archive + processed
  - Demuestra versioning sobrescribiendo un archivo
  - Aplica bucket policy que autoriza solo al rol app-role (lab 04)
  - Asume el rol y descarga un objeto — sin access keys
  - Genera una presigned URL como demo de acceso temporario

Uso:
    python scripts/s3_demo.py
: No such file or directory
scripts/s3_demo.py: line 16: import: command not found
scripts/s3_demo.py: line 17: import: command not found
scripts/s3_demo.py: line 18: from: command not found
scripts/s3_demo.py: line 19: from: command not found
scripts/s3_demo.py: line 21: ENDPOINT: command not found
scripts/s3_demo.py: line 22: REGION: command not found
scripts/s3_demo.py: line 23: syntax error near unexpected token `('
scripts/s3_demo.py: line 23: `ROOT = Path(__file__).parent.parent'

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-06-ValentínKebat)
$ python scripts/s3_demo.py
=== Lab 06 — S3 data lake + cierre IAM → EC2 → S3 ===

1. Bucket
  bucket 'course-data-lake' creado

2. Hardening por defecto (BPA + encryption)
  Block Public Access: ON (4 flags)
  Encryption: SSE-S3 (AES256) por defecto

3. Versioning
  Versioning: Enabled

4. Upload del dataset (Olist + GitHub Archive + processed)
  10 objetos nuevos (2.7 MB)
    - raw/olist/category_translations.csv (2,613 bytes)
    - raw/olist/customers.csv (262,528 bytes)
    - raw/olist/order_items.csv (456,173 bytes)
    ... y 7 más

5. Demo versioning (sobrescribir orders.csv)
  sobrescrito: raw/olist/orders.csv (+1 fila ficticia)
  versiones de 'raw/olist/orders.csv': 2
    - VersionId=tHGjcJc6s2wfj9sz... Size=528,930 ← actual
    - VersionId=I1AddVF5Vmx.leDv... Size=528,854

6. Bucket policy: solo app-role puede leer
  bucket policy aplicada: GetObject + ListBucket para app-role sobre raw/* y processed/*

7. AssumeRole + GetObject — cierre del círculo
  asumiendo rol app-role...
  creds temporales obtenidas (expiran: 2026-06-23 23:52:17.527000+00:00)
  GetObject como app-role: 'raw/olist/customers.csv' OK (262,528 bytes)

8. Presigned URL — acceso temporario sin asumir rol
  presigned URL para 'processed/push_events.json' (válida 5 min):
    http://localhost:4566/course-data-lake/processed/push_events.json?AWSAccessKeyId=test&Signature=k832...

=== Resumen final ===
  objetos:   10
  versiones: 11 (incluye sobreescritas)
  tamaño:    2.7 MB

Bucket en S3: s3://course-data-lake/
Listar todo: awslocal s3 ls s3://course-data-lake --recursive

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-06-ValentínKebat)
```