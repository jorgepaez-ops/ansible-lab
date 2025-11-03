# 🧪 Laboratorio Ansible + Docker

Este laboratorio permite probar Ansible sobre múltiples nodos Docker con Ubuntu y Rocky Linux. Incluye configuración SSH con llaves, instalación de utilidades básicas y personalización del login con neofetch.

---

## 🎯 Objetivo

- Crear varios nodos Docker (Ubuntu 24.04, Ubuntu 22.04 y Rocky 9).  
- Configurar SSH para acceso mediante llaves.  
- Instalar utilidades básicas (`vim`, `htop`, `net-tools`, `iproute`, `procps`).  
- Personalizar login con `neofetch`.  
- Gestionar todo con Ansible usando un **rol base**.

---

## 📂 Estructura de archivos

ansible-lab/
├── Dockerfile.ubuntu.24 # Imagen base Ubuntu 24.04
├── Dockerfile.ubuntu.22 # Imagen base Ubuntu 22.04
├── Dockerfile.rocky.9 # Imagen base Rocky Linux 9
├── docker-compose.yml # Levanta 3 nodos y la red virtual
├── inventory.ini # Inventario de Ansible usando llaves SSH
├── playbook.yml # Playbook principal
├── ansible.cfg # Configuración básica de Ansible
├── roles/
│ └── base/
│ ├── tasks/main.yml # Tareas principales
│ ├── handlers/main.yml # Handlers opcionales
│ └── templates/motd.j2 # Plantilla para mensaje de login
└── README.md


---

## 1️⃣ Preparación de los Dockerfiles

- Instalar `openssh-server`, `sudo`, `python3`.  
- Crear `/var/run/sshd` para iniciar SSH.  
- Configurar root con contraseña temporal (`root:root`).  
- Generar claves host con `ssh-keygen -A`.  
- Instalar utilidades opcionales para login: `neofetch` (fallback para fastfetch).  
- Exponer puerto 22 y mapear a puertos distintos en el host.

**Errores detectados y soluciones:**

| Error | Causa | Solución |
|-------|-------|---------|
| fastfetch no encontrado | No está en repos de Ubuntu/Rocky | Usar `neofetch` |
| ssh-keygen: command not found | Rocky no tenía cliente SSH completo | Instalar `openssh-clients` |
| Conflicto curl en Rocky | `curl-minimal` vs `curl` | No instalar curl en rol base |

---

## 2️⃣ Docker Compose

**Archivo:** `docker-compose.yml`  

- Define los 3 nodos con sus Dockerfiles respectivos.  
- Hostname y container_name definidos.  
- Todos en la red `ansible-net`.  

**Puertos mapeados:**

| Nodo  | Puerto |
|-------|--------|
| node1 | 2221   |
| node2 | 2222   |
| node3 | 2223   |

**Comandos usados:**

```bash
docker compose build --no-cache
docker compose up -d
docker ps -a
docker logs <container>

3️⃣ Configuración SSH y Ansible:

ssh-keygen -f ~/.ssh/known_hosts -R '[127.0.0.1]:2221'
ssh-keygen -f ~/.ssh/known_hosts -R '[127.0.0.1]:2222'
ssh-keygen -f ~/.ssh/known_hosts -R '[127.0.0.1]:2223'

Copiar clave SSH de Ansible a los nodos:

ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 2221 root@localhost
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 2222 root@localhost
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 2223 root@localhost

Verificar conectividad:

ansible all -i inventory.ini -m ping

4️⃣ Rol Base de Ansible

Función: instalar utilidades básicas y personalizar login.

Tareas principales:

Actualizar repositorios según sistema:

Ubuntu/Debian → apt update

Rocky → dnf update

Instalar utilidades:

Ubuntu/Debian: sudo, vim, htop, net-tools, iproute, procps

Rocky: sudo, vim, htop, net-tools, iproute, procps-ng

Personalizar .bashrc con neofetch.

Handler opcional: reiniciar SSH (no necesario en este laboratorio).

Notas de ajuste:

Evitar conflictos de paquetes (curl-minimal) en Rocky.

Evitar reinicios de SSH innecesarios.

5️⃣ Resultado final

Contenedores levantados y accesibles vía SSH con llaves.

Ansible puede ejecutar tareas básicas sin problemas.

Ubuntu y Rocky muestran OS en login (neofetch).

Rol base reproducible y seguro, sin conflictos de paquetes.

6️⃣ Comandos de limpieza (opcional)

docker compose down
docker system prune -f


