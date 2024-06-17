# Configuración del Servidor DHCP

## Introducción
Configuración de un servidor DHCP (ISC DHCP) en `serverOslo` (IP: 192.168.1.10) y su integración con un servidor DNS (BIND) en `serverMadrid` (IP: 192.168.1.20) para el dominio `example.com`.

## Instalación del Servidor DHCP
1. **Actualizar el sistema**:
    ```sh
    sudo apt-get update && sudo apt-get upgrade
    ```
2. **Instalar el servidor DHCP ISC**:
    ```sh
    sudo apt-get install isc-dhcp-server
    ```

## Configuración del Servidor DHCP
1. **Editar el archivo de configuración principal**:
    ```sh
    sudo nano /etc/dhcp/dhcpd.conf
    ```
    **Ejemplo**:
    ```conf
    default-lease-time 600;
    max-lease-time 7200;
    subnet 192.168.1.0 netmask 255.255.255.0 {
      range 192.168.1.50 192.168.1.100;
      option routers 192.168.1.1;
      option domain-name-servers 192.168.1.20;
      option domain-name "example.com";
    }
    ```
2. **Especificar la interfaz de red**:
    ```sh
    sudo nano /etc/default/isc-dhcp-server
    ```
    **Ejemplo**:
    ```sh
    INTERFACESv4="enp0s3"
    ```
3. **Configurar la IP estática de la interfaz**:
    ```sh
    sudo nano /etc/network/interfaces
    ```
    **Ejemplo**:
    ```conf
    auto enp0s3
    iface enp0s3 inet static
      address 192.168.1.10
      netmask 255.255.255.0
      gateway 192.168.1.1
    ```

## Reinicio y Verificación del Servicio DHCP
1. **Reiniciar el servicio DHCP**:
    ```sh
    sudo systemctl restart isc-dhcp-server
    ```
2. **Verificar el estado del servicio DHCP**:
    ```sh
    sudo systemctl status isc-dhcp-server
    ```

## Pruebas de Servicio DHCP
- **Eliminar dirección IP estática**:
    ```sh
    sudo ip addr del 192.168.1.20/24 dev enp0s3
    ```
- **Renovar dirección IP con DHCP**:
    ```sh
    sudo dhclient -r enp0s3 && sudo dhclient enp0s3
    ```

## Conclusión
La configuración del servidor DHCP en `serverOslo` se completó con éxito, permitiendo a los clientes de la red obtener direcciones IP y configuraciones DNS adecuadas para el dominio `example.com`.
