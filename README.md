# 📝 Resumen del Laboratorio Ansible + Docker (Nodos Ubuntu y Rocky)

## 1️⃣ Objetivo

Este laboratorio permite probar **Ansible** sobre múltiples nodos Docker con Ubuntu y Rocky Linux, incluyendo:

* Creación de nodos Docker: Ubuntu 24.04, Ubuntu 22.04 y Rocky 9
* Configuración de SSH con llaves
* Instalación de utilidades básicas (`vim`, `htop`, `net-tools`, `iproute`, `procps`)
* Personalización del login con `neofetch`
* Gestión de todo con Ansible usando un **rol base**

---

## 2️⃣ Preparación del entorno

### 2.1 Instalación de Docker y Docker Compose en el host físico

* Solo se instala en el **nodo físico/host**, que orquesta los contenedores (los nodos virtuales).

```bash
# Actualizar repositorios
sudo apt update && sudo apt upgrade -y        # Ubuntu/Debian
sudo dnf update -y                            # Rocky

# Instalar Docker
sudo apt install -y docker.io                 # Ubuntu
sudo dnf install -y docker                    # Rocky

# Habilitar y arrancar Docker
sudo systemctl enable docker --now
sudo systemctl status docker

# Instalar Docker Compose (si no viene incluido)
sudo apt install -y docker-compose            # Ubuntu
sudo dnf install -y docker-compose            # Rocky

# Agregar usuario actual a grupo docker (opcional)
sudo usermod -aG docker $USER
newgrp docker
```

* Verificar instalación:

```bash
docker --version
docker compose version
```

> Nota: Docker Compose no se instala dentro de los nodos Docker; solo en el host físico.

---

### 2.2 SSH y llaves

* Crear par de llaves SSH para Ansible:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_ansible -N ""
```

* Limpiar `known_hosts` para evitar errores:

```bash
ssh-keygen -f ~/.ssh/known_hosts -R '[127.0.0.1]:2221'
ssh-keygen -f ~/.ssh/known_hosts -R '[127.0.0.1]:2222'
ssh-keygen -f ~/.ssh/known_hosts -R '[127.0.0.1]:2223'
```

* Copiar la llave a los nodos Docker:

```bash
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 2221 root@localhost
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 2222 root@localhost
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 2223 root@localhost
```

* Verificar conectividad Ansible:

```bash
ansible all -i inventory.ini -m ping
```

---

## 3️⃣ Estructura de archivos

```
ansible-lab/
├── Dockerfile.ubuntu.24      # Imagen base Ubuntu 24.04 para nodo Docker
├── Dockerfile.ubuntu.22      # Imagen base Ubuntu 22.04 para nodo Docker
├── Dockerfile.rocky.9        # Imagen base Rocky Linux 9 para nodo Docker
├── docker-compose.yml        # Levanta los contenedores (nodos) y la red virtual en host físico
├── inventory.ini             # Inventario Ansible usando llaves SSH
├── playbook.yml              # Playbook principal
├── ansible.cfg               # Configuración básica de Ansible
├── roles/base/tasks/main.yml # Tareas principales
├── roles/base/handlers/main.yml # Handlers opcionales
├── roles/base/templates/motd.j2 # Mensaje de login
└── README.md
```

> Nota: Los Dockerfiles definen los **nodos virtuales**; Docker Compose se ejecuta solo en el host físico para levantar y conectar estos nodos.

---

## 4️⃣ Preparación de los Dockerfiles (nodos Docker)

* Instalar `openssh-server`, `sudo`, `python3`
* Crear `/var/run/sshd` para iniciar SSH
* Configurar root con contraseña temporal (`root:root`)
* Generar claves host con `ssh-keygen -A`
* Instalar utilidades para login: `neofetch` (fallback fastfetch)
* Exponer puerto 22 y mapear a puertos distintos en el host

**Errores detectados y soluciones:**

* `fastfetch` no encontrado → usar `neofetch`
* `ssh-keygen: command not found` en Rocky → instalar `openssh-clients`
* Conflicto `curl` en Rocky (`curl-minimal` vs `curl`) → no instalar curl en rol base

---

## 5️⃣ Docker Compose (en host físico)

* `docker-compose.yml` define los 3 nodos, cada uno con su Dockerfile, hostname y container_name, todos en la red `ansible-net`.
* Puertos mapeados:

| Nodo  | Puerto host |
| ----- | ----------- |
| node1 | 2221        |
| node2 | 2222        |
| node3 | 2223        |

**Comandos principales:**

```bash
docker compose build --no-cache
docker compose up -d
docker ps -a
docker logs <container>
```

---

## 6️⃣ Rol Base de Ansible

El rol base realiza:

* Actualización de repositorios según el sistema del nodo Docker:

  * Ubuntu/Debian → `apt update`
  * Rocky → `dnf update`
* Instalación de utilidades esenciales:

  * Ubuntu/Debian: `sudo`, `vim`, `htop`, `net-tools`, `iproute`, `procps`
  * Rocky: `sudo`, `vim`, `htop`, `net-tools`, `iproute`, `procps-ng`
* Personalización del `.bashrc` con `neofetch`
* Handlers opcionales: reinicio de SSH (omitido)

---

## 7️⃣ Resultado final

* Contenedores levantados y accesibles vía SSH con llaves
* Ansible puede ejecutar tareas básicas sin problemas
* Ubuntu y Rocky muestran el OS en login (`neofetch`)
* Rol base reproducible y seguro, sin conflictos de paquetes

---

## 8️⃣ Limpieza del laboratorio

```bash
docker compose down
docker system prune -f
```

---

## 9️⃣ Conceptos adicionales

* **Arquitectura clara:** Host físico con Docker Compose orquesta contenedores que son nodos virtuales.
* **Dockerfiles de nodos:** Cada contenedor es autónomo, con SSH y utilidades, gestionable por Ansible.
* **SSH con Ansible:** Llaves facilitan CI/CD y automatización.
* **Reutilización y mantenimiento:** Mantener Dockerfiles separados de roles permite escalabilidad y pipelines de prueba.
* **Aplicación práctica:** Base para despliegue de stacks de aplicaciones, pipelines CI/CD y experimentos con AWS, ECS o EKS.