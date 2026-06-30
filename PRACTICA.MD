# Consola

## Set Up consola

```bash
Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
$ docker compose up -d
[+] Running 5/5
 ✔ Container cloud-foundations-redpanda    Started                                                                                                                                                                0.6s 
 ✔ Container cloud-foundations-postgres    Started                                                                                                                                                                0.5s 
 ✔ Container cloud-foundations-redis       Started                                                                                                                                                                0.5s 
 ✔ Container cloud-foundations-localstack  Started                                                                                                                                                                0.7s 
 ✔ Container cloud-foundations-minio       Started                                                                                                                                                                0.7s 

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
$ docker ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED      STATUS                    PORTS                                                                                      NAMES
398acd987394   redis:7                        "docker-entrypoint.s…"   7 days ago   Up 17 seconds (healthy)   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp                                                cloud-foundations-redis
cb7fddc15d1c   redpandadata/redpanda:latest   "/entrypoint.sh redp…"   7 days ago   Up 17 seconds (healthy)   0.0.0.0:9092->9092/tcp, [::]:9092->9092/tcp, 0.0.0.0:9644->9644/tcp, [::]:9644->9644/tcp   cloud-foundations-redpanda
127fec314057   minio/minio                    "/usr/bin/docker-ent…"   7 days ago   Up 17 seconds (healthy)   0.0.0.0:9000-9001->9000-9001/tcp, [::]:9000-9001->9000-9001/tcp                            cloud-foundations-minio
94843365b9aa   postgres:16                    "docker-entrypoint.s…"   7 days ago   Up 17 seconds (healthy)   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp                                                cloud-foundations-postgres
f1f3830f17c1   localstack/localstack:3        "docker-entrypoint.sh"   8 days ago   Up 17 seconds (healthy)   0.0.0.0:4566->4566/tcp, [::]:4566->4566/tcp                                                cloud-foundations-localstack

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
$ docker f1f3830f17c1 exec awslocal s3 ls s3://course-data-lake | head -3
docker: unknown command: docker f1f3830f17c1

Run 'docker --help' for more information

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
$ docker exec f1f3830f17c1 awslocal s3 ls s3://course-data-lake | head -3

An error occurred (NoSuchBucket) when calling the ListObjectsV2 operation: The specified bucket does not exist

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
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
  access key creada: LKIAQAAAAAAAIQSCXUEO (larga duración — evitar en prod)

4. Rol con trust policy (EC2) + inline policy mínima
  rol 'app-role' creado
  inline policy 'InlineS3Read' adjuntada al rol 'app-role'

5. AssumeRole vía STS → credenciales temporales

  asumiendo rol: arn:aws:iam::000000000000:role/app-role
  AccessKeyId:  LSIAQAAAAAAACIUOYJLF
  Expiration:   2026-06-25 23:16:17.091000+00:00  ← credencial temporal
  objetos en 'course-data-raw' con credenciales temporales:
    - sample/hello.txt (17 bytes)

=== Resumen de recursos creados ===
  Bucket:  course-data-raw
  Grupo:   bigdata-read
  Policy:  arn:aws:iam::000000000000:policy/S3ReadOnlyLab
  Usuario: lab-user
  Rol:     arn:aws:iam::000000000000:role/app-role

Listo. Revisá los JSON en iam/ para entender cada documento.

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
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

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
$ docker exec f1f3830f17c1 awslocal s3 ls s3://course-data-lake | head -3

An error occurred (NoSuchBucket) when calling the ListObjectsV2 operation: The specified bucket does not exist

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
$ python scripts/iam_demo.py
=== Lab 04 — IAM demo ===

AVISO: LocalStack Community no enforcea policies (Deny no bloquea).
       Practicamos la mecánica: crear, adjuntar, asumir.

1. Bucket S3
  bucket 'course-data-raw' ya existe

2. Grupo + policy administrada
  grupo 'bigdata-read' ya existe
  policy 'S3ReadOnlyLab' ya existe: arn:aws:iam::000000000000:policy/S3ReadOnlyLab
  policy adjuntada al grupo 'bigdata-read'

3. Usuario → grupo
  usuario 'lab-user' ya existe
  usuario 'lab-user' agregado al grupo 'bigdata-read'
  access key creada: LKIAQAAAAAAAIAFABO42 (larga duración — evitar en prod)

4. Rol con trust policy (EC2) + inline policy mínima
  rol 'app-role' ya existe
  inline policy 'InlineS3Read' adjuntada al rol 'app-role'

5. AssumeRole vía STS → credenciales temporales

  asumiendo rol: arn:aws:iam::000000000000:role/app-role
  AccessKeyId:  LSIAQAAAAAAADW2G7YUW
  Expiration:   2026-06-25 23:18:39.371000+00:00  ← credencial temporal
  objetos en 'course-data-raw' con credenciales temporales:
    - sample/hello.txt (17 bytes)

=== Resumen de recursos creados ===
  Bucket:  course-data-raw
  Grupo:   bigdata-read
  Policy:  arn:aws:iam::000000000000:policy/S3ReadOnlyLab
  Usuario: lab-user
  Rol:     arn:aws:iam::000000000000:role/app-role

Listo. Revisá los JSON en iam/ para entender cada documento.

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
$ docker exec f1f3830f17c1 awslocal s3 ls s3://course-data-lake | head -3

An error occurred (NoSuchBucket) when calling the ListObjectsV2 operation: The specified bucket does not exist

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
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
    - VersionId=xeqyhqyWiXHD2g2E... Size=528,930 ← actual
    - VersionId=AzmjUvhcP_y7QYV8... Size=528,854

6. Bucket policy: solo app-role puede leer
  bucket policy aplicada: GetObject + ListBucket para app-role sobre raw/* y processed/*

7. AssumeRole + GetObject — cierre del círculo
  asumiendo rol app-role...
  creds temporales obtenidas (expiran: 2026-06-25 23:20:13.220000+00:00)
  GetObject como app-role: 'raw/olist/customers.csv' OK (262,528 bytes)

8. Presigned URL — acceso temporario sin asumir rol
  presigned URL para 'processed/push_events.json' (válida 5 min):
    http://localhost:4566/course-data-lake/processed/push_events.json?AWSAccessKeyId=test&Signature=pXBz...

=== Resumen final ===
  objetos:   10
  versiones: 11 (incluye sobreescritas)
  tamaño:    2.7 MB

Bucket en S3: s3://course-data-lake/
Listar todo: awslocal s3 ls s3://course-data-lake --recursive

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
$ docker exec f1f3830f17c1 awslocal s3 ls s3://course-data-lake | head -3
                           PRE processed/
                           PRE raw/

Valentin Kebat@DESKTOP-JG9POBC MINGW64 ~/Desktop/ITBA/cloud-foundations-lab (lab-07-valentin.kebat)
$ 
```


## CONSOLA DE DOCKER

```bash
# # 1. Crear VPC y configurar atributos
VPC_ID=$(awslocal ec2 create-vpc --cidr-block 10.0.0.0/16 --query "Vpc.VpcId" --output text)
awslocal ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=course-vpc Key=Lab,Value=07
awslocal ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames
awslocal ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-support
echo "VPC Creada: $VPC_ID"

# 2. Crear Subnets (Pública y Privada)
PUB_SUBNET=$(awslocal ec2 create-subnet --vpc-id $VPC_ID#  --cidr-block 10.0.1.0/24 --availability-zone us-east-1a --query "Subnet.SubnetId" --output text)
PRIV_SUBNET=$(awslocal ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.2.0/24 --availability-zone us-east-1b --query "Subnet.SubnetId" --output text)
awslocal ec2 create-tags --resources $PUB_SUBNET --tags Key=Name,Value=public-1a Key=Tier,Value=public
awslocal ec2 create-tags --resources $PRIV_SUBNET --tags Key=Name,Value=private-1b Key=Tier,Value=private
echo "Subnet Publica: $PUB_SUBNET"
echo "Subnet Privada: $PRIV_SUBNET"

# 3. Crear e interconectar el Internet Gateway
IGW_ID=$(awslocal ec2 create-internet-gateway --query "InternetGateway.InternetGatewayId" --output text)
awslocal ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
echo "IGW Creado y Adjuntado: $IGW_ID"

# 4. Tabla de ruteo privada y asociacion
RT_PRIV=$(awslocal ec2 create-route-table --vpc-id $VPC_ID --query "RouteTable.RouteTableId" --output text)
awslocal ec2 create-tags --resources $RT_PRIV --tags Key=Name,Value=rt-private
awslocal ec2 associate-route-table --route-table-id $RT_PRIV --subnet-id $PRIV_SUBNET
echo "Tabla de ruteo privada: $RT_PRIV"

# 5. Security Groups (Public y Private)
SG_PUB=$(awslocal ec2 create-security-group --vpc-id $VPC_ID --group-name web-public-sg --description "Capa publica - HTTP 80 desde Internet" --query "GroupId" --output text)
awslocal ec2 authorize-security-group-ingress --group-id $SG_PUB --protocol tcp --port 80 --cidr 0.0.0.0/0

SG_PRIV=$(awslocal ec2 create-security-group --vpc-id $VPC_ID --group-name app-private-sg --description "Capa privada - solo desde web-public-sg" --query "GroupId" --output text)
awslocal ec2 authorize-security-group-ingress --group-id $SG_PRIV --ip-permissions "IpProtocol=tcp,FromPort=8080,ToPort=8080,UserIdGroupPairs=[{GroupId=$SG_PUB,Description='Solo desde la capa publica'}]"
echo "SGs creados. Publico: $SG_PUB | Privado: $SG_PRIV"

# 6. VPC Endpoint para S3
ENDPOINT_ID=$(awslocal ec2 create-vpc-endpoint --vpc-id $VPC_ID --service-name com.amazonaws.us-east-1.s3 --vpc-endpoint-type Gateway --route-table-ids $RT_PRIV --query "VpcEndpoint.VpcEndpointId" --output text)
echo "VPC Endpoint S3 Creado: $ENDPOINT_ID"# # # # VPC Creada: vpc-cd189754
# # # # # # # Subnet Publica: subnet-10e64e7d
# Subnet Privada: subnet-a749963b
# # # # # IGW Creado y Adjuntado: igw-77ef7182
# # # # # {
    "AssociationId": "rtbassoc-787a716c"
}
# Tabla de ruteo privada: rtb-a88f7672
# # # # {
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-b8089cd0be6c266f4",
            "GroupId": "sg-79fb7c93ccf2cb5b5",
            "GroupOwnerId": "000000000000",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0",
            "Description": "",
            "Tags": []
        }
    ]
}
# # # {
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-f04dc0ce046460527",
            "GroupId": "sg-57652fee2b6e064c3",
            "GroupOwnerId": "000000000000",
            "IsEgress": true,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "ReferencedGroupInfo": {
                "GroupId": "sg-79fb7c93ccf2cb5b5",
                "UserId": "000000000000"
            },
            "Description": "Solo desde la capa publica"
        }
    ]
}
# SGs creados. Publico: sg-79fb7c93ccf2cb5b5 | Privado: sg-57652fee2b6e064c3
# # # # 
VPC Endpoint S3 Creado: vpce-b47fa481
# 
# awslocal ec2 describe-vpcs --filters Name=tag:Lab,Values=07
{
    "Vpcs": [
        {
            "OwnerId": "000000000000",
            "InstanceTenancy": "default",
            "Ipv6CidrBlockAssociationSet": [],
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-dab4a8e4",
                    "CidrBlock": "10.0.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": false,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "course-vpc"
                },
                {
                    "Key": "Lab",
                    "Value": "07"
                }
            ],
            "VpcId": "vpc-cd189754",
            "State": "available",
            "CidrBlock": "10.0.0.0/16",
            "DhcpOptionsId": "default"
        }
    ]
}
# awslocal ec2 describe-subnets --filters Name=vpc-id,Values=$VPC_ID
{
    "Subnets": [
        {
            "AvailabilityZoneId": "use1-az6",
            "OwnerId": "000000000000",
            "AssignIpv6AddressOnCreation": false,
            "Ipv6CidrBlockAssociationSet": [],
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "public-1a"
                },
                {
                    "Key": "Tier",
                    "Value": "public"
                }
            ],
            "SubnetArn": "arn:aws:ec2:us-east-1:000000000000:subnet/subnet-10e64e7d",
            "Ipv6Native": false,
            "PrivateDnsNameOptionsOnLaunch": {
                "HostnameType": "ip-name"
            },
            "SubnetId": "subnet-10e64e7d",
            "State": "available",
            "VpcId": "vpc-cd189754",
            "CidrBlock": "10.0.1.0/24",
            "AvailableIpAddressCount": 251,
            "AvailabilityZone": "us-east-1a",
            "DefaultForAz": false,
            "MapPublicIpOnLaunch": false
        },
        {
            "AvailabilityZoneId": "use1-az1",
            "OwnerId": "000000000000",
            "AssignIpv6AddressOnCreation": false,
            "Ipv6CidrBlockAssociationSet": [],
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "private-1b"
                },
                {
                    "Key": "Tier",
                    "Value": "private"
                }
            ],
            "SubnetArn": "arn:aws:ec2:us-east-1:000000000000:subnet/subnet-a749963b",
            "Ipv6Native": false,
            "PrivateDnsNameOptionsOnLaunch": {
                "HostnameType": "ip-name"
            },
            "SubnetId": "subnet-a749963b",
            "State": "available",
            "VpcId": "vpc-cd189754",
            "CidrBlock": "10.0.2.0/24",
            "AvailableIpAddressCount": 251,
            "AvailabilityZone": "us-east-1b",
            "DefaultForAz": false,
            "MapPublicIpOnLaunch": false
        }
    ]
}
# awslocal ec2 describe-route-tables --filters Name=vpc-id,Values=$VPC_ID
{
    "RouteTables": [
        {
            "Associations": [
                {
                    "Main": true,
                    "RouteTableAssociationId": "rtbassoc-38c4ecdc",
                    "RouteTableId": "rtb-3ef92cf1",
                    "AssociationState": {
                        "State": "associated"
                    }
                }
            ],
            "RouteTableId": "rtb-3ef92cf1",
            "Routes": [
                {
                    "DestinationCidrBlock": "10.0.0.0/16",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                }
            ],
            "Tags": [],
            "VpcId": "vpc-cd189754",
            "OwnerId": "000000000000"
        },
        {
            "Associations": [
                {
                    "Main": false,
                    "RouteTableAssociationId": "rtbassoc-787a716c",
                    "RouteTableId": "rtb-a88f7672",
                    "SubnetId": "subnet-a749963b",
                    "AssociationState": {
                        "State": "associated"
                    }
                }
            ],
            "RouteTableId": "rtb-a88f7672",
            "Routes": [
                {
                    "DestinationCidrBlock": "10.0.0.0/16",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                },
                {
                    "DestinationPrefixListId": "pl-a3d43a10",
                    "GatewayId": "vpce-b47fa481",
                    "Origin": "CreateRoute",
                    "State": "active"
                }
            ],
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "rt-private"
                }
            ],
            "VpcId": "vpc-cd189754",
            "OwnerId": "000000000000"
        }
    ]
}
# awslocal ec2 describe-internet-gateways --filters Name=attachment.vpc-id,Values=$VPC_ID
{
    "InternetGateways": [
        {
            "Attachments": [
                {
                    "State": "available",
                    "VpcId": "vpc-cd189754"
                }
            ],
            "InternetGatewayId": "igw-77ef7182",
            "OwnerId": "000000000000",
            "Tags": []
        }
    ]
}
# awslocal ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC_ID
{
    "SecurityGroups": [
        {
            "GroupId": "sg-60b21a6a8868b61a8",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "UserIdGroupPairs": [],
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": []
                }
            ],
            "Tags": [],
            "VpcId": "vpc-cd189754",
            "OwnerId": "000000000000",
            "GroupName": "default",
            "Description": "default VPC security group",
            "IpPermissions": []
        },
        {
            "GroupId": "sg-79fb7c93ccf2cb5b5",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "UserIdGroupPairs": [],
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": []
                }
            ],
            "Tags": [],
            "VpcId": "vpc-cd189754",
            "OwnerId": "000000000000",
            "GroupName": "web-public-sg",
            "Description": "Capa publica - HTTP 80 desde Internet",
            "IpPermissions": [
                {
                    "IpProtocol": "tcp",
                    "FromPort": 80,
                    "ToPort": 80,
                    "UserIdGroupPairs": [],
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": []
                }
            ]
        },
        {
            "GroupId": "sg-57652fee2b6e064c3",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "UserIdGroupPairs": [],
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": []
                }
            ],
            "Tags": [],
            "VpcId": "vpc-cd189754",
            "OwnerId": "000000000000",
            "GroupName": "app-private-sg",
            "Description": "Capa privada - solo desde web-public-sg",
            "IpPermissions": [
                {
                    "IpProtocol": "tcp",
                    "FromPort": 8080,
                    "ToPort": 8080,
                    "UserIdGroupPairs": [
                        {
                            "Description": "Solo desde la capa publica",
                            "UserId": "000000000000",
                            "GroupId": "sg-79fb7c93ccf2cb5b5"
                        }
                    ],
                    "IpRanges": [],
                    "Ipv6Ranges": [],
                    "PrefixListIds": []
                }
            ]
        }
    ]
}
# awslocal ec2 describe-vpc-endpoints --filters Name=vpc-id,Values=$VPC_ID
{
    "VpcEndpoints": [
        {
            "VpcEndpointId": "vpce-b47fa481",
            "VpcEndpointType": "Gateway",
            "VpcId": "vpc-cd189754",
            "ServiceName": "com.amazonaws.us-east-1.s3",
            "State": "available",
            "PolicyDocument": "{\"Statement\": [{\"Effect\": \"Allow\", \"Principal\": \"*\", \"Action\": \"*\", \"Resource\": \"*\"}]}",
            "RouteTableIds": [
                "rtb-a88f7672"
            ],
            "IpAddressType": "ipv4",
            "DnsOptions": {
                "DnsRecordIpType": "ipv4"
            },
            "PrivateDnsEnabled": true,
            "RequesterManaged": false,
            "DnsEntries": [],
            "CreationTimestamp": "2026-06-25T23:21:43Z",
            "Tags": [],
            "OwnerId": "000000000000"
        }
    ]
}
# 
```