# Uso de Ansible para configurar EC2 con Secret Manager

El objetivo de este repositorio es compartir como automatizar la configuración mediante **Ansible** una  instancia EC2 que se encuantra instalada y corriendo en AWS.

Utilizaremos la instancia Creada con **Terraform** en el repositorio https://github.com/ezequiellladoce/Despliegue_EC2_en_Infraestructura_Core_con_Secrets_Manager.

Para poder ejecutar el playbook de **Ansible** es necesario la ip publica de la instancia y su clave privada.

La clave privada la obtendremos en el AWS Secrets Manager y la ip publica en el terraform backend de S3.

## Pre-requisitos 📋

- TERRAFORM .12 o superior
- AWS CLI
- CUENTA FREE TIER AWS 
- Ansible

## Comenzando 🚀

### Preparamos el ambiente:

1) Instalamos Terrafom https://learn.hashicorp.com/tutorials/terraform/install-cli
2) Creamos cuenta free tier en AWS  https://aws.amazon.com/
3) Instalamos AWS CLI https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html
4) Creamos usuario AWS en la sección IAM con acceso Programático y permisos de administrador https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html   
5) Configuramos el AWS CLI https://docs.aws.amazon.com/polly/latest/dg/setup-aws-cli.html
6) Instalamos Ansible https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu
7) Si no lo hemos realizado aun realizamos el despliegue despripto en el repositorio  https://github.com/ezequiellladoce/Despliegue_EC2_en_Infraestructura_Core_con_Secrets_Manager.

## Recursos a utilizar:

### Paybook

Para configurar la instancia utilizaremos el siguinte  Playbook que instala un Servidor Apache en la instalcia EC2 con AWS Linux2.

```
- hosts: webservers
  remote_user: ec2-user
  become: yes
  gather_facts: no
  pre_tasks:
   - name: 'install python'
     raw: 'sudo yum install python3 -y'
  tasks:
   - name: Install Apache
     yum:
       name: httpd 
       state: present
   - service: 
       name: httpd 
       state: started
       enabled: yes
```

### AWS CLI

Con el AWS CLI obtendremos la clave privada del AWS **Secrets Manager** y la gaurdaremos en un archivo .pem:

```
 aws secretsmanager get-secret-value --secret-id "EC2-key-4" --region "us-east-2" --query 'SecretString' --output text > key.pem

```

### Terraform CLI

Con Terraform Output obtenemos la ip pública de la instancia se la asignamos al archivo hosts 

```
terraform output public_ip >> /etc/ansible/hosts
```

###  Ansible

Con la ip cargada en el archivo hosts y la clave privada podremos configurar la instancia. Primero verificaremos que Ansible conecte con la EC2.

```
ansible all -m ping -u ec2-user --key-file key.pem
```

Luego Aplicaremos la configuración:

```
ansible-playbook Playbook/playbook.yml -u ec2-user --key-file key.pem
```

## Despliegue 📦

### Ejecutamos los comandos

#### Obtenemos la ip Publica de la instancia y la insertamos en el archivo hosts

1) Una vez ejecutado el procedimineto del repositorio https://github.com/ezequiellladoce/Despliegue_EC2_en_Infraestructura_Core_con_Secrets_Manager.
2) En la carpeta Public_ip_from_backend ejecutamos los comandos:

```
echo "[webservers]" > /etc/ansible/hosts
terraform output pub_ip >> /etc/ansible/hosts
 ```  

Los que incertan la ip en el archivo hosts.  

#### Obtenemos la clave desde el AWS Secrets Manager

1) Clonamos el Repositorio https://github.com/ezequiellladoce/Ansible_EC2_con_Secret_Manager.git
2) Dentro de la carpeta Ansible_EC2_con_Secret_Manager ejecutamo los comandos:
 
```
aws secretsmanager get-secret-value --secret-id "EC2-key-4" --region "us-east-2" --query 'SecretString' --output text > key.pem
chmod 400 key.pem 
cat key.pem

```
Que bajaran la clave desde el secret manager y la guardará en el archivo key.pem

#### Ejecutamos la configuracion en la instancia

1) Corremos el comando ansible ping para verificar la comunicación con la instancia:

```
ansible all -m ping -u ec2-user --key-file key.pem

```
2) Ejecutamos el comando ansible-playbook para que instale lo indicado en el playbook. 

```
ansible-playbook Playbook/playbook.yml -u ec2-user --key-file key.pem

```


