# Laboratorio Ansible + Docker
Este repositorio contiene un laboratorio para probar Ansible sobre múltiples nodos Docker en Ubuntu 24.04.  
Se documenta todo el flujo desde la preparación de la máquina local hasta la ejecución de playbooks.

## Preparación del entorno local
1. Instalar Docker:
sudo apt update
sudo apt install docker.io -y
2. Instalar Docker Compose (plugin v2):
sudo apt install docker-compose-plugin -y
3. Instalar Ansible
   sudo apt install ansible -y
4. Generar llaves SSH para Ansible (recomendado):
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_ansible

## Contenido del repositorio
Dockerfile → Define la imagen base para los nodos.
docker-compose.yml → Levanta 3 contenedores como nodos (node1, node2, node3) y la red virtual.
inventory.ini → Inventario de Ansible usando llaves SSH.
playbook.yml → Ejemplo de playbook para instalar paquetes (htop, curl).

## Levantar nodos Docker
docker compose up -d --build

## Probar conectividad con Ansible
ansible all -i inventory.ini -m ping
