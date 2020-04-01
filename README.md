# Tarea de Ansible en GCP
En esta tarea se hace uso de tres instancias: una principal donde se instalará Ansible y dos secundarias que serán configuradas desde la instancia principal a través de Ansible. 

La configuración a realizar consiste en instalar Apache, PHP y MySQL en las instancias secundarias mediante un PlayBook en la instancia principal.

## Pasos
A continuación, se detallan los pasos que se han seguido para la realización de la tarea.

### Creación de red privada
- Trabajaremos sobre la siguiente red privada: `192.168.10/24`. 
- Dentro de esta red habilitamos el tráfico SSH a todas las instancias:
  - __Intervalos de IPs__: `0.0.0.0/0`.
  - __Protocolos y puertos__: `tcp:22`.
- Y permitimos cualquier el tráfico dentro de la red:
  - __Intervalos de IPs__: `192.168.10.0/24`.
  - __Protocolos y puertos__: `tcp:0-65535`, `udp:0-65535`, `icmp`.

### Creación de instancias secundarias
- Se crearán dos instancias identícas (exceptuando su dirección IP) con las siguientes características:
  - __Tipo de máquina__: `n1-standard-1 (1 vCPU, 3,75 GB de memoria)`.
  - __S. O.__: `Ubuntu 18.04`.
  - __Direcciones IP__: `192.168.10.20` y `192.168.10.30`.
- En cada instancia, se generan las claves SSH para poder acceder desde la instancia principal.

### Creación de instancia principal
- Esta instancia es donde se instalará y utilizará Ansible. Creamos una instancia con las siguientes características:
  - __Tipo de máquina__: `n1-standard-1 (1 vCPU, 3,75 GB de memoria)`.
  - __S. O.__: `Ubuntu 18.04`.
  - __Dirección IP__: `192.168.10.10`.

- Guardamos las claves SSH de las instancias secundarias para poder acceder sin contraseña.
  
- Instalación de Ansible:
  ```
  $ sudo apt update
  $ sudo apt install software-properties-common
  $ sudo apt-add-repository ppa:ansible/ansible
  $ sudo apt install ansible
  ```
- Creamos un inventario de las dos instancias secundarias:
  ```
  $ sudo nano /etc/ansible/hosts
    [servers]
    server1 ansible_host=192.168.10.20
    server2 ansible_host=192.168.10.30
  ```
- Creamos un Playbook para instalar Apache, PHP y MySQL en las instancias secundarias:
  ```
  $ mkdir playbooks
  $ cd playbooks
  $ nano apachephpmysql.yml 
    ---
    - hosts: servers
      tasks:
        - name: Actualiza el sistema
          become: true
          apt:
            upgrade: yes
            update_cache: yes
        - name: Instala Apache
          become: true
          apt: name=apache2
        - name: Instala MySQL
          become: true
          apt: name=mysql-server
        - name: Instala PHP
          become: true
          apt: name=php
        - name: Instala PHP-MySQL
          become: true
          apt: name=php-mysql
  ```
- Lanzamos el Playbook:
  ```
  $ ansible-playbook apachephpmysql.yml
  ```

