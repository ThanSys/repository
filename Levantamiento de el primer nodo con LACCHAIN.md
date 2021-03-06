# Levantamiento de el primer nodo con LACCHAIN Besu

Tres formas:

• Ansible ←
• Docker
• Genérica


## Instalar Ansible en la máquina local

```shell
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```

## Clonar el repositorio en la máquina local

La máquina tiene una instalación limpia de Ubuntu 20.04, entonces se necesita primero instalar git:

```shell
$ sudo apt update
$ sudo apt install git
$ git --version
```


## Después se hace el clonado del repositorio:

```shell
$ git clone https://github.com/lacchain/besu-network
$ cd besu-network/
```

## Descargar e instalar Oracle Java 11

https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html

Para poder descargar JDK 11 necesitas iniciar sesión.
El archivo a descargar es: jdk-11.0.15_linux-x64_bin.tar.gz

Enviar el archivo descargado al servidor (máquina remota):
```shell
$ cd Downloads
$ ls
$ scp jdk-11.0.15_linux-x64_bin.tar.gz chainlac@148.202.23.22:
$ cd -
```

Conectarse al servidor y verificar que el archivo esté ahí:
```shell
$ ssh chainlac@148.202.23.22
$ ls
```

Hacer una carpeta JDK en la máquina remota y mover el archivo ahí:
```shell
$ sudo mkdir -p /var/cache/oracle-jdk11-installer-local
$ sudo cp jdk-11.0.15_linux-x64_bin.tar.gz /var/cache/oracle-jdk11-installer-local/
$ sudo apt update
```

## Preparación para la instalación del nuevo nodo

En tu máquina local, copiar inventory.example como inventory y editarlo con los datos correspondientes:
```shell
$ cd besu-network
$ cp inventory.example inventory
$ vi inventory
```

Se va a agregar un nodo de tipo writer, así que se va a escribir esto:
```shell
[writer]
148.202.23.22 node_ip=148.202.23.22 node_name=cucea-lacchain-mx
```

Teclas para editar en vi
i - insertar donde está el cursor
ESC - dejar de insertar
:q - salir
:wq - guardar y salir

Más comandos de vi: https://www.guru99.com/the-vi-editor.html
Se puede usar cualquier otro editor, como gedit, si este no gusta.

En la máquina local, desde la carpeta besu-network:
```shell
$ cd roles
$ cd lacchain-writer-node
$ cd vars
$ vi main.yml
```
Aquí se va a cambiar el net_id = 648529 por la opción Academy, siguiendo el tutorial:

• Central: 648529
• David19: 648530
• Academy: 648539 ←


## Desplegar el nodo

En la máquina local, desde la carpeta besu-network:
```shell
$ ansible-playbook -i inventory --private-key=~/.ssh/id_rsa -u chainlac site-lacchain-writer.yml
```
Error
"msg": "Missing sudo password"

Para solucionar este error, se agregó esto al final del comando:
```shell
$ ansible-playbook -i inventory --private-key=~/.ssh/id_rsa -u chainlac site-lacchain-writer.yml --extra-vars 
"ansible_sudo_pass=Jhde5-Pudm-f7cf1_01"
```
Fuente: https://stackoverflow.com/questions/25582740/missing-sudo-password-in-ansible


Error
No se instaló Java.

Según el error, se debe instalar el JDK 11.0.4 o el 11.0.13 (que es el que concuerda con el sha256sum).
https://www.oracle.com/mx/java/technologies/javase/jdk11-archive-downloads.html

Checksum for Java SE 11.0.4 binaries: https://www.oracle.com/webfolder/s/digest/11-0-4-checksum.html

Después de descargarlo, empezar de nuevo desde el paso "Descargar e instalar Oracle Java 11" pero con la versión correcta.
↑ Tampoco funcionó.

https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-20-04


Anotar el enode del nodo cuando aparezca mientras se realiza el proceso. Debe verse algo así:
```shell
TASK [lacchain-validator-node : print enode key] ***********************************************
ok: [x.x.x.x] => {
    "msg": "enode://cb24877f329e0e3fff6c7d7b88d601b698a9df6efbe1d91ce77130f065342b523418b38cb3c92ea3bcca15344e68c7d85a696eb9f8c0152c51b9b7b74729064e@a.b.c.d:60606"
}
```

***
```shell
ok: [148.202.23.22] => {
    "msg": "enode://a9865f676040a3b355b189449b49df4941c0769406edbbb04a46cc2ea51b696301a5869dff298b25e28790ef7317a5dc1a34cbeeb1886280cf7de483ed552326@148.202.23.22:60606"
}
```

TASK [lacchain-writer-node : print enode key] ************************************************************************************************
ok: [148.202.23.22] => {
    "msg": "enode://a9865f676040a3b355b189449b49df4941c0769406edbbb04a46cc2ea51b696301a5869dff298b25e28790ef7317a5dc1a34cbeeb1886280cf7de483ed552326@148.202.23.22:60606"
}

## Segundo intento

https://github.com/LACNetNetworks/besu-networks/blob/master/DEPLOY_NODE.md

```shell
Clonar el repositorio nuevo:
$ git clone https://github.com/LACNet-Networks/besu-networks
$ cd besu-networks/
$ cp inventory.example inventory
$ gedit inventory
[node]
148.202.23.22 node_ip=148.202.23.22 node_name=cucea-lacchain-mx
```

Desplegar el nodo:
```shell
$ cd besu-networks/
$ ansible-playbook -i inventory --private-key=~/.ssh/id_rsa -u chainlac
site-lacchain-node.yml --extra-vars "ansible_sudo_pass=Jhde5-Pudm-f7cf1_01"
```
^ Se tuvo que escribir a mano porque no se dejaba copiar y pegar.

Aparecerá esto:
```shell
[0]:validator
[1]:boot
[2]:writer
[3]:tessera
Please, choose which type of node are you deploying: 2


[0]:mainnet-omega
[1]:pro-testnet
[2]:testnet-david19
Please, choose in which network are you deploying: 2
```

Se eligió testnet-david19 para hacer la prueba

Funcionó con el JDK 11.0.13

enode key

```shell
TASK [lacchain-writer-node : print enode key] ************************************************************************************************ 
ok: [148.202.23.22] => { 
    "msg": "enode://a9865f676040a3b355b189449b49df4941c0769406edbbb04a46cc2ea51b696301a5869dff298b25e28790ef7317a5dc1a34cbeeb1886280cf7de483ed552326@148.202.23.22:60606" 
}
```

## Iniciar el nodo

En la máquina remota:
```shell
<remote_machine>$ sudo service pantheon start
```

Para reiniciar los servicios:
```shell
$ service pantheon restart
```

Obtener los permisos y después revisar si la conexión funcionó:
```shell
$ tail -100 /root/lacchain/logs/pantheon_info.log
```

## Después de obtener los permisos

En la máquina remota:
```shell
<remote_machine>$ sudo service pantheon start
<remote_machine>$ sudo service pantheon restart
```

Revisar la conexión:
```shell
$ sudo -i
$ curl -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}' localhost:4545
```

```shell
TASK [lacchain-writer-node : print enode key] **********************************
ok: [148.202.23.22] => {
    "msg": "enode://a9865f676040a3b355b189449b49df4941c0769406edbbb04a46cc2ea51b696301a5869dff298b25e28790ef7317a5dc1a34cbeeb1886280cf7de483ed552326@148.202.23.22:60606"
}
```

Nuevo enode:
```shell
TASK [lacchain-writer-node : print enode key] **********************************
ok: [148.202.23.22] => {
    "msg": "enode://ccbb8b05bf805e39c7ce4b86ae201a9e6428055a95b737f39e0b6d097fed70dd48e1623bb23a119e56b2db75c6f629b3d2898097a0c037dd39f99f5265a4ea10@148.202.23.22:60606"
}
```
