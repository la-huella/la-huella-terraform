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

 # Push

git add .
     o
git add .github/workflows/la-huella-wf.yaml 
     o
git add README.md
git commit -m "Terraform remote state with LocalStack"
git push origin main



