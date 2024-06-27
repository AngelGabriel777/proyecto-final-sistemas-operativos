# Configuración del Servidor DNS y Servidor Web

## Requisitos Previos
- **Sistema Operativo:** Debian 12 (sin interfaz gráfica).
- **VirtualBox:** Instalado y configurado.
- **Acceso a Internet:** Para instalar los paquetes necesarios.
- **Permisos de superusuario:** Para realizar configuraciones del sistema.

## Pasos para la Configuración del Servidor DNS

1. **Instalación del Software Necesario**

    Primero, asegúrate de que el sistema esté actualizado e instala el servidor DNS `bind9`:
    ```bash
    sudo apt update
    sudo apt install bind9
    ```

2. **Configuración de las Zonas de DNS**

    Edita el archivo de configuración principal de BIND `/etc/bind/named.conf.local`:
    ```bash
    sudo nano /etc/bind/named.conf.local
    ```
    Añade la configuración para la zona `unisimon.com`:
    ```plaintext
    zone "unisimon.com" {
        type master;
        file "/etc/bind/zones/db.unisimon.com";
    };
    ```

3. **Configuración del Archivo de Zona Directa**

    Crea el directorio para las zonas si no existe y configura el archivo de zona directa:
    ```bash
    sudo mkdir -p /etc/bind/zones
    sudo nano /etc/bind/zones/db.unisimon.com
    ```
    El contenido del archivo de zona directa debe ser:
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

4. **Verificación de la Configuración de BIND**

    Verifica que la configuración de BIND no tenga errores:
    ```bash
    sudo named-checkconf
    sudo named-checkzone unisimon.com /etc/bind/zones/db.unisimon.com
    ```

5. **Reinicio y Habilitación del Servicio BIND**

    Reinicia el servicio BIND para aplicar los cambios:
    ```bash
    sudo systemctl restart bind9
    sudo systemctl enable bind9
    ```

6. **Configuración del Firewall**

    Si tienes un firewall configurado, permite el tráfico DNS. Asumiendo que estás utilizando `ufw`:
    ```bash
    sudo ufw allow Bind9
    ```

7. **Verificación del Funcionamiento del Servidor DNS**

    Para verificar que el servidor DNS está funcionando correctamente, utiliza el comando `dig`:
    ```bash
    dig @192.168.1.20 unisimon.com
    ```
    Verifica que la respuesta contenga la dirección IP correcta (`192.168.1.20`).

## Configuración del Servidor Web

1. **Instalación del Servidor Web**

    Instala el servidor web NGINX:
    ```bash
    sudo apt install nginx
    ```

2. **Configuración del Sitio Web**

    Crea el archivo de configuración del sitio para `unisimon.com`:
    ```bash
    sudo nano /etc/nginx/sites-available/unisimon.com
    ```
    Añade la configuración del sitio:
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
    Haz un enlace simbólico en el directorio de sitios habilitados:
    ```bash
    sudo ln -s /etc/nginx/sites-available/unisimon.com /etc/nginx/sites-enabled/
    ```

3. **Creación del Archivo `index.html`**

    Crea el directorio para el sitio web y el archivo `index.html`:
    ```bash
    sudo mkdir -p /var/www/unisimon.com
    sudo nano /var/www/unisimon.com/index.html
    ```
    Añade el contenido del archivo `index.html`:
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

4. **Ajuste de Permisos**

    Asegura los permisos adecuados para el directorio del sitio web:
    ```bash
    sudo chown -R www-data:www-data /var/www/unisimon.com
    sudo chmod -R 755 /var/www/unisimon.com
    ```

5. **Reinicio del Servidor Web**

    Reinicia NGINX para aplicar los cambios:
    ```bash
    sudo systemctl restart nginx
    ```

6. **Verificación del Funcionamiento del Servidor Web**

    Accede a `http://unisimon.com` desde un navegador web para verificar que la página se carga correctamente.
