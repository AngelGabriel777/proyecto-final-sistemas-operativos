# Configuración de Servidor DNS y Web en Debian 12

Este proyecto detalla cómo configurar un servidor DNS y un servidor web en Debian 12 sin interfaz gráfica.

## Requisitos Previos

- **Sistema Operativo:** Debian 12
- **VirtualBox:** Instalado
- **Acceso a Internet**
- **Permisos de superusuario**

## Configuración del Servidor DNS

### Instalación del Software Necesario

Actualice el sistema e instale `bind9`:

```sh
sudo apt update
sudo apt install bind9 bind9utils bind9-doc

## Configuracion de las Zonas de DNS
### Edite /etc/bind/named.conf.localy añada:

```sh
zone "unisimon.com" {
    type master;
    file "/etc/bind/zones/db.unisimon.com";
};

### Cree el archivo de zona:

```sh
sudo mkdir -p /etc/bind/zones
sudo nano /etc/bind/zones/db.unisimon.com

### Contenido de db.unisimon.com:

```sh
$TTL 86400
@   IN  SOA ns.unisimon.com. admin.unisimon.com. (
            2023061701; Serial
            3600; Refresh
            1800; Retry
            1209600; Expire
            86400 ); Minimum TTL
@       IN  NS      ns.unisimon.com.
ns      IN  A       192.168.1.20
www     IN  A       192.168.1.20

###Verificación y Reinicio de BIND
##Verifique y reinicie BIND:

```sh
sudo named-checkconf
sudo named-checkzone unisimon.com /etc/bind/zones/db.unisimon.com
sudo systemctl restart bind9
sudo systemctl enable bind9

###Configuración del cortafuegos
##Permita el tráfico DNS:
