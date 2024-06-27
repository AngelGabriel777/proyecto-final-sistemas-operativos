# Configuración del Servidor de Archivos SMB en Ubuntu Server 24.04 LTS

Este informe detalla el proceso de configuración de un servidor de archivos utilizando Samba en un entorno Ubuntu Server 24.04 LTS, alojado en una máquina virtual de VirtualBox. La configuración permite compartir archivos entre el servidor y una máquina cliente Windows, facilitando así la colaboración y el acceso a datos compartidos.

## Pasos de Configuración

1. **Instalación de Samba**

    1.1. Actualizar el Sistema:
    ```bash
    sudo apt update
    sudo apt upgrade
    ```

    1.2. Instalar Samba:
    ```bash
    sudo apt install samba
    ```

2. **Configurar Samba**

    2.1. Editar el archivo de configuración de Samba:
    ```bash
    sudo nano /etc/samba/smb.conf
    ```

    2.2. Añadir la siguiente configuración al final del archivo:
    ```ini
    [share]
       path = /srv/samba/share
       available = yes
       valid users = usuario1
       read only = no
       browsable = yes
       public = yes
       writable = yes
    ```

    2.3. Guardar y cerrar el archivo.

3. **Reiniciar el servicio de Samba**
    ```bash
    sudo systemctl restart smbd
    ```

4. **Crear Usuarios (Opcional)**

    4.1. Crear un usuario del sistema (por ejemplo, usuario1):
    ```bash
    sudo adduser usuario1
    ```

    4.2. Añadir el usuario a Samba:
    ```bash
    sudo smbpasswd -a usuario1
    ```

    4.3. Reiniciar el servicio de Samba:
    ```bash
    sudo systemctl restart smbd
    ```

5. **Prueba de Conexión**

    5.1. Desde una máquina cliente (Windows/Linux), pruebe la conexión al servidor Samba:

    - En Windows:
        1. Abrir el Explorador de Archivos.
        2. En la barra de direcciones, escribir `\\192.168.1.5\share` y presionar Enter.
        3. Si se solicita, ingrese las credenciales del usuario Samba (si configuró usuarios).

    - En Linux:
    ```bash
    smbclient //192.168.1.5/share -U usuario1
    ```

6. **Navegar y Abrir el Archivo**

    6.1. Navegar a la Carpeta:
    Dentro de la carpeta compartida, busca y abre la carpeta llamada `prueba`.

    6.2. Abrir el Archivo de Texto:
    Dentro de la carpeta `prueba`, busca el archivo de texto que ha mencionado (por ejemplo, `prueba.txt`). Haga doble clic en el archivo de texto para abrirlo. Windows debería abrir el archivo con el editor de texto predeterminado, como el Bloc de notas.

7. **Verificación en el Servidor SMB**

    7.1. Conectarse al Servidor:
    Abre una terminal y conéctate a tu servidor Ubuntu si no estás ya conectado.

    7.2. Navegar a la Carpeta Compartida:
    ```bash
    cd /srv/samba/share
    ```

    7.3. Leer el contenido del archivo de texto en la terminal:
    ```bash
    cat /srv/samba/share/prueba/prueba.txt
    ```

    7.4. Eliminar un archivo:
    ```bash
    rm /srv/samba/share/prueba/prueba.txt
    ```
