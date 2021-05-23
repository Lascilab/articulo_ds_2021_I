# LXC/LXD

Esta carpeta contiene el paso a paso para ejecutar un ambiente de puebas de estrés en la herramienta de virtualización de contenedores LXC/LXD

Ejecutar sobre una versión del kernel de linux 5.10.19

## Pruebas

En este ambiente se buscan ejecutar pruebas sobre diferentes ambitos mediante las herramientas:

- Stress-ng
- NPB
- HammerDB

## Configuración

Para llevar a cabo el despliegue del entorno virtual es necesario configurar:

- Instalar `sudo apt install lxc snapd make`
- Instalar lxd: `sudo snap install lxd`
- Inicializar lxd mediante: `cat config.yml | sudo lxd init --preseed`

### Vagrant

Se dispone un archivo **Vagrantfile** que corre una máquina virtual con la imagen `generic/ubuntu2010` donde se aproviciona la instalación de lxc/lxd y se copian los archivos *.sh necesarios para ejecutar las pruebas, una vez iniciada la maquina virtual es necesario ejecutar lo siguiente

- `vagrant ssh`
- `cd LXD`
- `cat config.yml | lxd init --preseed`

Para modificar las pruebas es necesario revisar el archivo test.sh y las carpetas para cada herramienta

## Ejecución

Después de configurar lxd se debe aprovicionar un contenedor de pruebas esto se puede hacer mediante el comando `make` tanto en una maquina baremetal que haya superado los pasos de la configuración como en una vm de vagrant.

El comando make se encarga de ejecutar el archivo `provision.sh` con la variable `CONTAINER=lxdtest` y el cual:

- Crea el contenedor de prueba con `lxc launch ubuntu:20.10 $CONTAINER`
- Espera la inicialización de este con `lxc exec $CONTAINER -- cloud-init status -w`
- Instala la herramienta stress-ng: `lxc exec $CONTAINER -- snap install stress-ng`
- Ejecuta el script `sh push_files.sh $CONTAINER` el cual carga los archivos de las carpetas MySQL, PostgreSQL, NPB y HammerDB al contenedor y les da capacidad de ejecutarse
- Ejecuta `lxc exec $CONTAINER -- bash -c "cd /root/NPB/ && sh script_instalacion.sh"` para instalar NPB
- Ejecuta `lxc exec $CONTAINER -- bash -c "cd /root/MySQL/ && sh inst-MySQL.sh"` para instalar MySQL
- Ejecuta `lxc exec $CONTAINER -- bash -c "cd /root/PostgreSQL/ && sh inst-PostgreSQL.sh"` para instalar PostgreSQL
- Ejecuta `lxc exec $CONTAINER -- bash -c "cd /root/HammerDB/ && sh inst-HammerDB.sh && mv HammerDB-4.1/* ."`  para instalar HammerDB

En caso de requerir aprovisionar desde cero el contenedor se recomienda ejecutar

- `make clean && make`

### Pruebas

Para ejecutar las pruebas en el contenedor se debe ejecutar `make run-tests` el cual:

- Ejecuta el archivo `run_tests.sh`
- Ejecuta las pruebas en el contenedor mediante el archivo `test.sh`

Además para obtener los resultados una vez realizadas las pruebas solo es necesario ejecutar `make pull-logs` el cual copia los archivos con los resultados a la carpeta `logs/`

## Makefile

```Makefile
all: provision
clean: stop delete

provision:
	sh provision.sh lxdtest

attach:
	lxc exec lxdtest -- bash

run-tests:
	sh run_tests.sh lxdtest

pull-logs:
	sh pull_logs.sh lxdtest

stop:
	lxc stop lxdtest

delete:
	lxc delete lxdtest
```