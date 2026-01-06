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



