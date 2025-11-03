# 🧪 Laboratorio Ansible + Docker

Este laboratorio permite probar Ansible sobre múltiples nodos Docker con Ubuntu y Rocky Linux.  
Incluye configuración SSH con llaves, instalación de utilidades básicas y personalización del login con neofetch.

---
## 🎯 Objetivo

- Crear nodos Docker: Ubuntu 24.04, Ubuntu 22.04 y Rocky 9  
- Configurar SSH con llaves  
- Instalar utilidades básicas (`vim`, `htop`, `net-tools`, `iproute`, `procps`)  
- Personalizar login con `neofetch`  
- Gestionar todo con Ansible usando un **rol base**

---

## 📂 Estructura de archivos

ansible-lab/  
├── Dockerfile.ubuntu.24 → Imagen base Ubuntu 24.04  
├── Dockerfile.ubuntu.22 → Imagen base Ubuntu 22.04  
├── Dockerfile.rocky.9 → Imagen base Rocky Linux 9  
├── docker-compose.yml → Levanta 3 nodos y la red virtual  
├── inventory.ini → Inventario de Ansible usando llaves SSH  
├── playbook.yml → Playbook principal  
├── ansible.cfg → Configuración básica de Ansible  
├── roles/base/tasks/main.yml → Tareas principales  
├── roles/base/handlers/main.yml → Handlers opcionales  
├── roles/base/templates/motd.j2 → Plantilla para mensaje de login  
└── README.md  

---

## 🐳 Preparación de los Dockerfiles

- Instalar `openssh-server`, `sudo`, `python3`  
- Crear `/var/run/sshd` para iniciar SSH  
- Configurar root con contraseña temporal (`root:root`)  
- Generar claves host con `ssh-keygen -A`  
- Instalar utilidades para login: `neofetch` (fallback fastfetch)  
- Exponer puerto 22 y mapear a puertos distintos en el host

**Errores detectados y soluciones:**

- `fastfetch` no encontrado → usar `neofetch`  
- `ssh-keygen: command not found` en Rocky → instalar `openssh-clients`  
- Conflicto `curl` en Rocky (`curl-minimal` vs `curl`) → no instalar curl en rol base  

---

## 🔧 Docker Compose

Archivo `docker-compose.yml` define los 3 nodos, cada uno con su Dockerfile, hostname y container_name, todos en la red `ansible-net`.  

**Puertos mapeados:**  
- node1 → 2221  
- node2 → 2222  
- node3 → 2223  

**Comandos principales:**

```bash
docker compose build --no-cache
docker compose up -d
docker ps -a
docker logs <container>
````

---

## 🔑 Configuración SSH y Ansible

Limpiar `known_hosts` para evitar errores de “REMOTE HOST IDENTIFICATION HAS CHANGED”:

```bash
ssh-keygen -f ~/.ssh/known_hosts -R '[127.0.0.1]:2221'
ssh-keygen -f ~/.ssh/known_hosts -R '[127.0.0.1]:2222'
ssh-keygen -f ~/.ssh/known_hosts -R '[127.0.0.1]:2223'
```

Copiar la clave SSH de Ansible a los nodos para acceso sin contraseña:

```bash
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 2221 root@localhost
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 2222 root@localhost
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 2223 root@localhost
```

Verificar conectividad con Ansible:

```bash
ansible all -i inventory.ini -m ping
```

> Warnings sobre sftp/scp pueden ignorarse si los módulos funcionan correctamente.

Errores comunes en esta fase:

* `to use the 'ssh' connection type with passwords or pkcs11_provider, you must install sshpass` → si usamos llaves SSH, no es necesario `sshpass`.
* `Host key verification failed` → limpiar `known_hosts` o aceptar la clave al primer login (`yes`).

---

## ⚙️ Rol Base de Ansible

El rol base se encarga de:

* Actualizar repositorios según el sistema:
  * Ubuntu/Debian → `apt update`
  * Rocky → `dnf update`

* Instalar utilidades esenciales:
  * Ubuntu/Debian: `sudo`, `vim`, `htop`, `net-tools`, `iproute`, `procps`
  * Rocky: `sudo`, `vim`, `htop`, `net-tools`, `iproute`, `procps-ng`

* Personalizar `.bashrc` con `neofetch` para mostrar el OS al iniciar sesión
* Handler opcional: reiniciar SSH (omitido porque no es necesario para este laboratorio)

Notas de ajuste:

* Evitar reinicios innecesarios de SSH
* Evitar conflictos de paquetes (como `curl-minimal`)
* `become: yes` funciona, pero `true` también es aceptable

---

## ✅ Resultado final

* Contenedores levantados y accesibles vía SSH con llaves
* Ansible puede ejecutar tareas básicas sin problemas
* Ubuntu y Rocky muestran el OS en login (`neofetch`)
* Rol base reproducible y seguro, sin conflictos de paquetes

---

## 🧹 Limpieza del laboratorio

Para detener y eliminar los nodos y liberar recursos del host:

```bash
docker compose down
docker system prune -f
```