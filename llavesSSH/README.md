# Configuración de Llaves Públicas y Privadas

Este documento proporciona instrucciones sobre cómo configurar y crear llaves en Puttygen, así como la configuración necesaria en el servidor.

## Proceso

### Paso 1: Crear el Directorio `.ssh` y el Archivo `authorized_keys`

1. Ejecuta los siguientes comandos en el servidor para crear el directorio `.ssh` y el archivo `authorized_keys`:

    ```bash
    mkdir -p ~/.ssh
    touch ~/.ssh/authorized_keys
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    ```

### Paso 2: Actualizar la Configuración de SSH

1. Edita el archivo de configuración de SSH (usualmente ubicado en `/etc/ssh/sshd_config`) y asegúrate de que las siguientes líneas estén presentes y sin comentarios:

    ```plaintext
    PubkeyAuthentication yes
    AuthorizedKeysFile .ssh/authorized_keys
    ```

### Paso 3: Reiniciar el Servicio SSH

1. Para aplicar los cambios, reinicia el servicio SSH con el siguiente comando:

    ```bash
    sudo systemctl restart ssh
    ```

### Paso 4: Crear Llaves en Puttygen

1. Abre Puttygen.
2. Selecciona el tipo de clave que deseas generar (por ejemplo, RSA).
3. Haz clic en el botón "Generate" y mueve el mouse para generar la entropía necesaria.
4. Una vez generada la clave, guarda la clave privada en un lugar seguro.
5. Copia la clave pública desde la ventana de Puttygen y agrégala al archivo `authorized_keys` en el servidor.

    ```plaintext
    # Ejemplo de clave pública (No utilices esta clave en producción)
    ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAr8nFkjw+9...

    # Guarda la clave privada con una extensión .ppk
    ```

---

Ahora tu servidor debería estar configurado para autenticarse usando llaves públicas y privadas.
