# Configuración del Servidor DNS y Servidor Web

## Requisitos Previos
- *Sistema Operativo:* Debian 12 (sin interfaz gráfica).
- *VirtualBox:* Instalado y configurado.
- *Acceso a Internet:* Para instalar los paquetes necesarios.
- *Permisos de superusuario:* Para realizar configuraciones del sistema.

## Pasos para la Configuración del Servidor DNS

### Configuración de Red
1. **Configurar las interfaces de red en Debian (`/etc/network/interfaces`):**
    ```plaintext
    auto eth0
    iface eth0 inet static
        address 192.168.1.20  
        netmask 255.255.255.0
        gateway 192.168.1.1
        dns-nameservers 192.168.1.20
    ```

2. **Configurar las interfaces de red en Ubuntu Server (`/etc/netplan/01-netcfg.yaml`):**
    ```yaml
    network:
      version: 2
      ethernets:
        eth0:
          addresses:
            - 192.168.1.X/24  # Reemplaza X con la IP correspondiente
          gateway4: 192.168.1.1
          nameservers:
            addresses:
              - 192.168.1.20
    ```

3. **Aplicar la configuración de red:**
    ```bash
    sudo netplan apply
    ```

### Configuración del Servidor DNS (serverMadrid)
1. **Actualizar el sistema e instalar BIND9:**
    ```bash
    sudo apt update
    sudo apt install bind9
    ```

2. **Editar el archivo de configuración de BIND:**
    ```bash
    sudo nano /etc/bind/named.conf.local
    ```
    Añadir la configuración para la zona `unisimon.com`:
    ```plaintext
    zone "unisimon.com" {
        type master;
        file "/etc/bind/zones/db.unisimon.com";
    };
    ```

3. **Crear y configurar el archivo de zona directa:**
    ```bash
    sudo mkdir -p /etc/bind/zones
    sudo nano /etc/bind/zones/db.unisimon.com
    ```
    Añadir el contenido:
    ```plaintext
    $TTL    604800
    @       IN      SOA     ns.unisimon.com. admin.unisimon.com. (
                            2024010101         ; Serial
                            604800             ; Refresh
                            86400              ; Retry
                            2419200            ; Expire
                            604800 )           ; Negative Cache TTL
    ;
    @       IN      NS      ns.unisimon.com.
    ns      IN      A       192.168.1.20
    @       IN      A       192.168.1.20
    ```

4. **Añadir la configuración para la zona inversa en el archivo de configuración de BIND:**
    ```bash
    sudo nano /etc/bind/named.conf.local
    ```
    Añadir la configuración:
    ```plaintext
    zone "1.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/zones/db.192.168.1";
    };
    ```

5. **Crear y configurar el archivo de zona inversa:**
    ```bash
    sudo nano /etc/bind/zones/db.192.168.1
    ```
    Añadir el contenido:
    ```plaintext
    $TTL    604800
    @       IN      SOA     ns.unisimon.com. admin.unisimon.com. (
                            2024010101         ; Serial
                            604800             ; Refresh
                            86400              ; Retry
                            2419200            ; Expire
                            604800 )           ; Negative Cache TTL
    ;
    @       IN      NS      ns.unisimon.com.
    20      IN      PTR     unisimon.com.
    ```

6. **Configurar opciones adicionales de seguridad en BIND:**
    ```bash
    sudo nano /etc/bind/named.conf.options
    ```
    Añadir o modificar las opciones:
    ```plaintext
    options {
        directory "/var/cache/bind";
        recursion no;  # Desactiva la recursión si no es necesaria
        allow-query { any; };
        allow-transfer { none; };  # Desactiva las transferencias de zona
        dnssec-validation auto;
        auth-nxdomain no;  # Conformidad con RFC 1035
        listen-on { any; };
    };
    ```

7. **Verificar la configuración de BIND:**
    ```bash
    sudo named-checkconf
    sudo named-checkzone unisimon.com /etc/bind/zones/db.unisimon.com
    sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.192.168.1
    ```

8. **Reiniciar y habilitar el servicio BIND:**
    ```bash
    sudo systemctl restart bind9
    sudo systemctl enable bind9
    ```

9. **Permitir tráfico DNS en el firewall:**
    ```bash
    sudo ufw allow Bind9
    ```

10. **Verificar el funcionamiento del servidor DNS:**
    ```bash
    dig @192.168.1.20 unisimon.com
    dig @192.168.1.20 -x 192.168.1.20
    ```

### Configuración del Servidor Web (serverMadrid)
1. **Instalar NGINX:**
    ```bash
    sudo apt install nginx
    ```

2. **Crear y configurar el archivo del sitio web:**
    ```bash
    sudo nano /etc/nginx/sites-available/unisimon.com
    ```
    Añadir la configuración:
    ```plaintext
    server {
        listen 80;
        server_name unisimon.com www.unisimon.com;
        root /var/www/unisimon.com;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```

3. **Hacer un enlace simbólico en el directorio de sitios habilitados:**
    ```bash
    sudo ln -s /etc/nginx/sites-available/unisimon.com /etc/nginx/sites-enabled/
    ```

4. **Crear el directorio y archivo `index.html`:**
    ```bash
    sudo mkdir -p /var/www/unisimon.com
    sudo nano /var/www/unisimon.com/index.html
    ```
    Añadir el contenido:
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Bienvenido a Unisimon</title>
    </head>
    <body>
        <h1>¡Bienvenido a Unisimon!</h1>
        <p>Esta es la página de bienvenida para unisimon.com</p>
    </body>
    </html>
    ```

5. **Ajustar permisos:**
    ```bash
    sudo chown -R www-data:www-data /var/www/unisimon.com
    sudo chmod -R 755 /var/www/unisimon.com
    ```

6. **Configurar opciones adicionales de seguridad en NGINX:**
    ```bash
    sudo nano /etc/nginx/nginx.conf
    ```
    Añadir las directivas dentro del bloque `http`:
    ```plaintext
    http {
        ...
        server_tokens off;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Frame-Options "SAMEORIGIN";
        add_header Content-Security-Policy "default-src 'self'";
    }
    ```

7. **Verificar la configuración de NGINX:**
    ```bash
    sudo nginx -t
    ```

8. **Reiniciar NGINX:**
    ```bash
    sudo systemctl restart nginx
    ```

9. **Permitir tráfico HTTP en el firewall:**
    ```bash
    sudo ufw allow 'Nginx HTTP'
    ```

10. **Verificar el funcionamiento del servidor web:**
    Accede a `http://unisimon.com` desde un navegador web.

### Pruebas de Funcionamiento

#### Pruebas para el Servidor DNS
1. **Consulta de dominio:**
    ```bash
    dig @192.168.1.20 unisimon.com
    ```
    Verificar que la respuesta incluya la dirección IP `192.168.1.20`.

2. **Consulta inversa:**
    ```bash
    dig @192.168.1.20 -x 192.168.1.20
    ```
    Verificar que la respuesta incluya el dominio `unisimon.com`.

3. **Prueba desde otro servidor:**
    ```bash
    dig @192.168.1.20 unisimon.com
    dig @192.168.1.20 -x 192.168.1.20
    ```
    Verificar que las respuestas sean correctas.

#### Pruebas para el Servidor Web
1. **Acceso desde un navegador web:**
    - Abrir un navegador web.
    - Acceder a `http://unisimon.com`.
    - Verificar que se muestra la página de bienvenida con el mensaje "¡Bienvenido a Unisimon!".

2. **Consulta desde la línea de comandos:**
    ```bash
    curl -I http://unisimon.com
    ```
    Verificar que la respuesta incluya un código de estado `200 OK`.

3. **Prueba desde otro servidor:**
    ```bash
    curl -I http://unisimon.com
    ```
    Verificar que la respuesta incluya un código de estado `200 OK`.
