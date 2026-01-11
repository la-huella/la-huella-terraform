### Crear repositorio git en la carpeta "la-huella-terraform"

# git init

[javi@localhost la-huella-terraform]$ git init
Inicializado repositorio Git vacío en /home/javi/DEVOPS/7_desplieguesindrama/ext/la-huella-terraform/.git/

# git status

[javi@localhost la-huella-terraform]$ git status
En la rama main

No hay commits todavía

Archivos sin seguimiento:
  (usa "git add <archivo>..." para incluirlo a lo que será confirmado)
	Readme.md
	backend.tf
	providers.tf

no hay nada agregado al commit pero hay archivos sin seguimiento presentes (usa "git add" para hacerles seguimiento)

# Crear .gitignore

[javi@localhost la-huella-terraform]$ cat <<EOF > .gitignore
.terraform/
.terraform.lock.hcl
*.tfstate
*.tfstate.*
crash.log
EOF

# Primer commit

[javi@localhost la-huella-terraform]$ git add .
git commit -m "Initial commit: Terraform remote state with LocalStack"
[main (commit-raíz) 1878c43] Initial commit: Terraform remote state with LocalStack
 Committer: javierfg1 <javi@localhost.localdomain>
Tu nombre y correo fueron configurados automáticamente basados
en tu usuario y nombre de host. Por favor verifica que sean correctos.
Tú puedes suprimir este mensaje configurándolos de forma explícita:

    git config --global user.name "Tu nombre"
    git config --global user.email you@example.com

Tras hacer esto, puedes arreglar tu identidad para este commit con:

    git commit --amend --reset-author

 4 files changed, 80 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 Readme.md
 create mode 100644 backend.tf
 create mode 100644 providers.tf

 # Creamos el repositorio en github y push inicial

 https://github.com/javierfg1/la-huella-terraform.git

 git remote add origin https://github.com/javierfg1/la-huella-terraform.git

[javi@localhost la-huella-terraform]$ git remote set-url origin git@github.com:javierfg1/la-huella-terraform.git
[javi@localhost la-huella-terraform]$ git push origin main
Enumerando objetos: 12, listo.
Contando objetos: 100% (12/12), listo.
Compresión delta usando hasta 12 hilos
Comprimiendo objetos: 100% (12/12), listo.
Escribiendo objetos: 100% (12/12), 2.17 KiB | 445.00 KiB/s, listo.
Total 12 (delta 4), reusados 0 (delta 0), pack-reusados 0
remote: Resolving deltas: 100% (4/4), done.
To github.com:javierfg1/la-huella-terraform.git
 * [new branch]      main -> main

 
 # Push

git add .
     o
git add .github/workflows/la-huella-wf.yaml 
     o
git add Readme.md
git commit -m "Terraform remote state with LocalStack"
git push origin main

# Crear manualmente el bucket S3 en LocalStack

export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1

aws s3api create-bucket \
  --bucket la-huella-remote-state \
  --region us-east-1 \
  --endpoint-url=http://midominio.local


[javi@localhost la-huella-terraform]$ aws s3api create-bucket   --bucket la-huella-remote-state   --region us-east-1   --endpoint-url=http://midominio.local
{
    "Location": "/la-huella-remote-state"
}


aws s3 ls --endpoint-url=http://midominio.local

[javi@localhost la-huella-terraform]$ aws s3 ls --endpoint-url=http://midominio.local
2026-01-06 09:07:33 la-huella-remote-state


# (Opcional pero recomendable) Crear la tabla DynamoDB para locking

aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --endpoint-url=http://midominio.local



[javi@localhost la-huella-terraform]$ aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --endpoint-url=http://midominio.local
{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "LockID",
                "AttributeType": "S"
            }
        ],
        "TableName": "terraform-locks",
        "KeySchema": [
            {
                "AttributeName": "LockID",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": "2026-01-06T09:09:13.060000+01:00",
        "ProvisionedThroughput": {
            "LastIncreaseDateTime": "1970-01-01T01:00:00+01:00",
            "LastDecreaseDateTime": "1970-01-01T01:00:00+01:00",
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 0,
            "WriteCapacityUnits": 0
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:us-east-1:000000000000:table/terraform-locks",
        "TableId": "090bc1e7-fadb-4b9c-b57d-0a7f789b8dde",
        "BillingModeSummary": {
            "BillingMode": "PAY_PER_REQUEST",
            "LastUpdateToPayPerRequestDateTime": "2026-01-06T09:09:13.060000+01:00"
        },
        "DeletionProtectionEnabled": false
    }
}


aws dynamodb list-tables --endpoint-url=http://midominio.local

[javi@localhost la-huella-terraform]$ aws dynamodb list-tables --endpoint-url=http://midominio.local
{
    "TableNames": [
        "terraform-locks"
    ]
}


# Crear backend.tf / providers.tf

Claves importantes aquí:

     endpoint → LocalStack

     force_path_style = true → obligatorio

     skip_* → evita llamadas a AWS real

# Inicializar el backend remoto

terraform init


[javi@localhost la-huella-terraform]$ terraform fmt -write=false *.tf
backend.tf
providers.tf
[javi@localhost la-huella-terraform]$ terraform init

Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v6.27.0...
- Installed hashicorp/aws v6.27.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

# terraform apply

Con este comando se gurada el fichero de estado en el bucket

[javi@localhost la-huella-terraform]$ terraform apply -lock=false

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.


# Verfificación

aws s3 ls --endpoint-url=http://midominio.local --region us-east-1

[javi@localhost la-huella-terraform]$ aws s3 ls --endpoint-url=http://midominio.local --region us-east-1
2026-01-06 09:07:33 la-huella-remote-state

aws dynamodb scan --table-name terraform-locks --endpoint-url=http://midominio.local --region us-east-1

[javi@localhost la-huella-terraform]$ aws dynamodb scan --table-name terraform-locks --endpoint-url=http://midominio.local --region us-east-1
{
    "Items": [],
    "Count": 0,
    "ScannedCount": 0,
    "ConsumedCapacity": null
}

[javi@localhost la-huella-terraform]$ aws s3 ls s3://la-huella-remote-state/mission7/   --endpoint-url=http://midominio.local   --region us-east-1
2026-01-06 19:43:53        180 terraform.tfstate

javi@localhost la-huella-terraform]$ aws s3 cp \
  s3://la-huella-remote-state/mission7/terraform.tfstate \
  /tmp/terraform.tfstate \
  --endpoint-url=http://midominio.local \
  --region us-east-1
download: s3://la-huella-remote-state/mission7/terraform.tfstate to ../../../../../../tmp/terraform.tfstate

[javi@localhost la-huella-terraform]$ cd ../../../../../../tmp/

[javi@localhost tmp]$ ls -a | grep terraform
terraform.tfstate

[javi@localhost tmp]$ cat terraform.tfstate 
{
  "version": 4,
  "terraform_version": "1.6.2",
  "serial": 1,
  "lineage": "35a49033-3ba7-3b0e-bdd1-efe7b2a0dc55",
  "outputs": {},
  "resources": [],
  "check_results": null
}


# Entrar en el contenedor de localstack

[javi@localhost la-huella-terraform]$ kubectl get pods -A | grep localstack
mission7               localstack7-86c59f6c78-skpxs                            1/1     Running            0                  2d15h

javi@localhost la-huella-terraform]$ kubectl exec -it localstack7-86c59f6c78-skpxs -n mission7 -- sh


# Notas

ls -a (para ver ficheros ocultos)

Si se eliminar el pod locastack, se elimina también el bucket y la tabla dynamodb porque toso se almacena en el pod localstack.

En este caso, hay que recrear localstack y el ingress

helm uninstall localstack7 localstack/localstack -n mission7

helm install localstack7 localstack/localstack -f 0-localstack-values.yaml -n mission7

kubectl apply -f 0-ingress-localstack.yaml -n mission7

Además habría que cambiar /etc/hosts con la nueva IP del svc

Comprobar todo con:

kubectl get pods -n mission7
kubectl get svc -n mission7
kubectl get ingress -n mission7

# Dos ficheros terraform.tfstate

1️⃣ terraform.tfstate LOCAL

📍 Normalmente en:

./terraform.tfstate

(o dentro de .terraform/)

¿Qué es?

👉 Es el estado del bootstrap, el que Terraform usa para crear y gestionar el backend remoto (el bucket S3 y la tabla DynamoDB).

¿Por qué existe?

Porque hay un problema de huevo y gallina:

Terraform necesita un state para crear el bucket
pero el bucket aún no existe para guardar el state

➡️ Solución: state local temporal

¿Qué contiene?

Bucket la-huella-remote-state

Tabla DynamoDB terraform-locks

Recursos mínimos de infraestructura

⚠️ Este state:

NO debe usar backend remoto

Debe quedarse local

No debe borrarse




2️⃣ terraform.tfstate REMOTO (S3 / LocalStack)

📍 Ruta lógica:

s3://la-huella-remote-state/mission7/terraform.tfstate

¿Qué es?

👉 Es el state real de tu infraestructura (clusters, servicios, redes, etc.).

¿Qué contiene?

Todos los recursos “reales” del proyecto

Se comparte entre equipos

Está bloqueado con DynamoDB

[ bootstrap terraform ]
        │
        ├── terraform.tfstate  (LOCAL)
        │       └─ crea S3 + DynamoDB
        │
        ▼
[ infraestructura real ]
        │
        ├── backend "s3"
        │
        └── terraform.tfstate (S3 / LocalStack)


# Añadimos proyecto terraform para creación objetos que están en main.tf

terraform init -upgrade


[javi@localhost la-huella-terraform]$ terraform init -upgrade

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.100.0...
- Installed hashicorp/aws v5.100.0 (signed by HashiCorp)

Terraform has made some changes to the provider dependency selections recorded
in the .terraform.lock.hcl file. Review those changes and commit them to your
version control system if they represent changes you intended to make.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.


# plan, apply


export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1

terraform init
terraform plan
terraform apply


# Notas

Ver si hay lock activo:

aws dynamodb scan \
  --table-name terraform-locks \
  --endpoint-url=http://midominio.local \
  --region us-east-1


Borrar lock activo:

aws dynamodb delete-item \
  --table-name terraform-locks \
  --key '{"LockID":{"S":"mission7/terraform.tfstate"}}' \    ##### Poner el LockID que toque.                                
  --endpoint-url=http://midominio.local \
  --region us-east-1

DEspués de borrar el lock activo:

terraform init -reconfigure
terraform plan
terraform apply


  Enter a value: yes

aws_sqs_queue.la_huella_processing_queue: Creating...
aws_s3_bucket.la_huella_sentiment_reports: Creating...
aws_s3_bucket.la_huella_uploads: Creating...
aws_s3_bucket.la_huella_sentiment_reports: Creation complete after 0s [id=la-huella-sentiment-reports]
aws_s3_bucket.la_huella_uploads: Creation complete after 0s [id=la-huella-uploads]
aws_sqs_queue.la_huella_processing_queue: Still creating... [10s elapsed]
aws_sqs_queue.la_huella_processing_queue: Still creating... [20s elapsed]
aws_sqs_queue.la_huella_processing_queue: Creation complete after 25s [id=http://sqs.us-east-1.localhost.localstack.cloud:4566/000000000000/la-huella-processing-queue]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

dynamodb_tables = [
  "la-huella-comments",
  "la-huella-products",
  "la-huella-analytics",
]
log_group = "/la-huella-aplication"
s3_buckets = [
  "la-huella-sentiment-reports",
  "la-huella-uploads",
]
sqs_queues = [
  "la-huella-processing-queue",
  "la-huella-notifications-queue",
]


# Misión completada

avi@localhost la-huella-terraform]$ missions submit f2acb699-7815-4050-8b32-b679f296e8e7

👀 Se van a ejecutar los siguientes comandos:
─────────────────────────────────────────
  ▶️  terraform state list | sort | tr '\n' ' ' | sed 's/ $//'

¿Quieres continuar? (si/no): si

📋 RESULTADOS DE LA MISIÓN
═════════════════════════

💻 Resultado de ejecución local:
─────────────────────────────
[aws_cloudwatch_log_group.la_huella_aplication aws_dynamodb_table.la_huella_analytics aws_dynamodb_table.la_huella_comments aws_dynamodb_table.la_huella_products aws_s3_bucket.la_huella_sentiment_reports aws_s3_bucket.la_huella_uploads aws_sqs_queue.la_huella_notifications_queue aws_sqs_queue.la_huella_processing_queue]

📊 Detalle de comandos:
─────────────────────
  ✅  terraform state list | sort | tr '\n' ' ' | sed 's/ $//'

🏁 Resultado final:
────────────────
  🎉 ETAPA COMPLETADA
  ➡️ Porcentaje de acierto: 100% (requerido: 100%)


### CREAR RUNNER EN LOCAL

1️⃣ Crear el runner en GitHub

Ve a tu repositorio en GitHub

Settings → Actions → Runners

Click en New self-hosted runner

Elige tu sistema operativo

GitHub te mostrará comandos personalizados (⚠️ usa los tuyos)

Download
# Create a folder
$ mkdir actions-runner && cd actions-runner
# Download the latest runner package
$ curl -o actions-runner-linux-x64-2.330.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.330.0/actions-runner-linux-x64-2.330.0.tar.gz
# Optional: Validate the hash
$ echo "af5c33fa94f3cc33b8e97937939136a6b04197e6dadfcfb3b6e33ae1bf41e79a  actions-runner-linux-x64-2.330.0.tar.gz" | shasum -a 256 -c
# Extract the installer
$ tar xzf ./actions-runner-linux-x64-2.330.0.tar.gz
Configure
# Create the runner and start the configuration experience
$ ./config.sh --url https://github.com/javierfg1/la-huella-terraform --token BDA6NA767VHYF5OLWYLU2A3JMNPXC
# Last step, run it!
$ ./run.sh
Using your self-hosted runner
# Use this YAML in your workflow file for each job
runs-on: self-hosted


[javi@localhost la-huella-terraform]$ mkdir actions-runner && cd actions-runner 
[javi@localhost actions-runner]$ ll
total 0
[javi@localhost actions-runner]$ curl -o actions-runner-linux-x64-2.330.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.330.0/actions-runner-linux-x64-2.330.0.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100  211M  100  211M    0     0  22.0M      0  0:00:09  0:00:09 --:--:-- 21.7M
[javi@localhost actions-runner]$ ls -la
total 216792
drwxr-xr-x. 2 javi javi        53 ene 11 08:31 .
drwxr-xr-x. 5 javi javi      4096 ene 11 08:30 ..
-rw-r--r--. 1 javi javi 221990519 ene 11 08:31 actions-runner-linux-x64-2.330.0.tar.gz
[javi@localhost actions-runner]$ echo "af5c33fa94f3cc33b8e97937939136a6b04197e6dadfcfb3b6e33ae1bf41e79a  actions-runner-linux-x64-2.330.0.tar.gz" | shasum -a 256 -c
actions-runner-linux-x64-2.330.0.tar.gz: OK
[javi@localhost actions-runner]$ tar xzf ./actions-runner-linux-x64-2.330.0.tar.gz
[javi@localhost actions-runner]$ ./config.sh --url https://github.com/javierfg1/la-huella-terraform --token BDA6NA767VHYF5OLWYLU2A3JMNPXC

--------------------------------------------------------------------------------
|        ____ _ _   _   _       _          _        _   _                      |
|       / ___(_) |_| | | |_   _| |__      / \   ___| |_(_) ___  _ __  ___      |
|      | |  _| | __| |_| | | | | '_ \    / _ \ / __| __| |/ _ \| '_ \/ __|     |
|      | |_| | | |_|  _  | |_| | |_) |  / ___ \ (__| |_| | (_) | | | \__ \     |
|       \____|_|\__|_| |_|\__,_|_.__/  /_/   \_\___|\__|_|\___/|_| |_|___/     |
|                                                                              |
|                       Self-hosted runner registration                        |
|                                                                              |
--------------------------------------------------------------------------------

# Authentication


√ Connected to GitHub

# Runner Registration

Enter the name of the runner group to add this runner to: [press Enter for Default] 

Enter the name of runner: [press Enter for localhost] 

This runner will have the following labels: 'self-hosted', 'Linux', 'X64' 
Enter any additional labels (ex. label-1,label-2): [press Enter to skip] 

√ Runner successfully added

# Runner settings

Enter name of work folder: [press Enter for _work] 

√ Settings Saved.

# Runner execution


[javi@localhost actions-runner]$ ./run.sh

√ Connected to GitHub

Current runner version: '2.330.0'
2026-01-11 07:38:53Z: Listening for Jobs

 # Cada vez que queramos ejecutarlo:


sudo rm -r actions-runner/_work/la-huella-wf/la-huella-wf/volume/
sudo rm -r /home/javi/DEVOPS/6-despliegaconestilo/ext/la-huella-wf/.git/index.lock


echo "# test" >> README.md
(Tener en cuenta --> No hay cambios → no hay push → no hay workflow)

git add .
git commit -m "mensaje"
git push origin main

O bien:

git add .github/workflows/la-huella-wf-def.yaml 
git add .github/workflows/la-huella-test.yaml 
     o
git add README.md
git commit -m "La Huella Pipeline Definitivo"
git push origin main
















