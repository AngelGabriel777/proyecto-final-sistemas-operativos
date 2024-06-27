# Configuración del Servicio SFTP

## 1. Introducción

Este documento describe el proceso de instalación y configuración del servicio SFTP en un servidor Debian, y cómo establecer una conexión segura desde un cliente Windows utilizando FileZilla. El SFTP (SSH File Transfer Protocol) proporciona un método seguro para la transferencia de archivos entre sistemas. También se detalla la configuración de la autenticación mediante claves SSH para mejorar la seguridad.

## 2. Preparativos y Requisitos

### Requisitos del Sistema

**Servidor:**
- Debian (o cualquier distribución Linux compatible con OpenSSH).
- Acceso sudo o root.

**Cliente:**
- Windows con FileZilla.
- Claves SSH generadas para autenticación.

### Paquetes Necesarios

En el servidor, asegúrate de tener instalados los siguientes paquetes:
- openssh-server
- filezilla (opcional, solo si quieres probar la conexión desde el mismo servidor)

## 3. Configuración del Servidor SFTP

### Instalación del Servidor SSH

1. Actualiza los paquetes e instala el servidor SSH:
    ```sh
    sudo apt-get update
    sudo apt-get install openssh-server
    ```

### Configuración de Usuarios SFTP

1. Crea un usuario `sftpuser` con un directorio específico para SFTP:
    ```sh
    sudo adduser sftpuser
    sudo mkdir -p /home/sftpuser/uploads
    ```

2. Asegúrate de que el directorio de SFTP tenga los permisos correctos:
    ```sh
    sudo chown root:root /home/sftpuser
    sudo chmod 755 /home/sftpuser
    sudo chown sftpuser:sftpuser /home/sftpuser/uploads
    ```

### Configuración del Servidor SSH para SFTP

1. Abre el archivo de configuración de SSH:
    ```sh
    sudo nano /etc/ssh/sshd_config
    ```

2. Añade las siguientes líneas al final del archivo para configurar el entorno chroot para `sftpuser`:
    ```
    Match User sftpuser
    ChrootDirectory /home/sftpuser
    ForceCommand internal-sftp
    AllowTcpForwarding no
    ```

3. Aplica los cambios reiniciando el servicio SSH:
    ```sh
    sudo systemctl restart ssh
    ```

### Configuración de Autenticación con Claves SSH

#### Generar Claves SSH en el Cliente

1. En el cliente Windows, abre PuTTYgen.
2. Selecciona el tipo de clave a generar (por ejemplo, RSA).
3. Haz clic en `Generate` y sigue las instrucciones para mover el ratón sobre el área en blanco.
4. Guarda la clave pública en un archivo `.pub`.
5. Guarda la clave privada en un archivo `.ppk`. Opcionalmente, configura una frase de contraseña.
6. En PuTTYgen, copia la clave pública desde el campo `Public key for pasting into OpenSSH authorized_keys file`.

#### Configurar Claves SSH en el Servidor

1. En el servidor, crea el directorio `.ssh` en el home de `sftpuser`:
    ```sh
    sudo mkdir /home/sftpuser/.ssh
    sudo touch /home/sftpuser/.ssh/authorized_keys
    sudo chown sftpuser:sftpuser /home/sftpuser/.ssh
    sudo chmod 700 /home/sftpuser/.ssh
    sudo chown sftpuser:sftpuser /home/sftpuser/.ssh/authorized_keys
    sudo chmod 600 /home/sftpuser/.ssh/authorized_keys
    ```

2. Abre el archivo `authorized_keys` para `sftpuser` y pega la clave pública copiada desde PuTTYgen:
    ```sh
    sudo nano /home/sftpuser/.ssh/authorized_keys
    ```

3. Aplica los cambios reiniciando el servicio SSH:
    ```sh
    sudo systemctl restart ssh
    ```

## 4. Configuración del Cliente FileZilla

### Configuración Básica en FileZilla

1. Abre FileZilla y accede al Gestor de sitios (`Archivo -> Gestor de sitios`).
2. Haz clic en `Nuevo sitio` y configura el sitio con los siguientes parámetros:
    - Protocolo: `SFTP - SSH File Transfer Protocol`.
    - Servidor: `[Dirección IP del servidor]` (e.g., 192.168.1.30).
    - Puerto: `22`.
    - Modo de acceso: `Archivo de claves`.
    - Usuario: `sftpuser`.
    - Archivo de claves: `[Ruta al archivo .ppk generado]`.

### Probar Conexión

1. Haz clic en `Conectar` para probar la conexión.
2. Una vez conectado, navega al directorio `/uploads`.

## 5. Verificación del Servicio SFTP

1. En FileZilla, verifica que puedes acceder y listar el contenido de `/uploads`.
2. Intenta subir un archivo a `/uploads` y luego descargarlo para asegurarte de que la transferencia funciona correctamente.
3. Asegúrate de que los archivos subidos tengan permisos adecuados para `sftpuser`.

## 6. Conclusión

Este informe documenta la configuración del servicio SFTP en un servidor Debian, incluyendo la autenticación mediante claves SSH, y la conexión desde un cliente Windows utilizando FileZilla. Se han cubierto los aspectos esenciales de la instalación, configuración, verificación del servicio y resolución de problemas para asegurar una operación exitosa y segura.
