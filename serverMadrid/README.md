# Configuración del Servidor DNS y Servidor Web

## Introducción
Este informe detalla el proceso de desarrollo y configuración de un servidor DNS en un sistema operativo Debian 12 sin interfaz gráfica, alojado en VirtualBox. El servidor DNS se utilizará para resolver nombres de dominio a direcciones IP, facilitando la navegación en la red mediante nombres de dominio amigables. Además, se configurará un servidor web que servirá una página HTML de bienvenida para `unisimon.com`.

## Requisitos Previos
- **Sistema Operativo:** Debian 12 (sin interfaz gráfica).
- **VirtualBox:** Instalado y configurado.
- **Acceso a Internet:** Para instalar los paquetes necesarios.
- **Permisos de superusuario:** Para realizar configuraciones del sistema.

## Pasos para la Configuración del Servidor DNS

### 1. Instalación del Software Necesario
Primero, asegúrate de que el sistema esté actualizado e instala el servidor DNS `bind9`:
```bash
sudo apt update
sudo apt install bind9

