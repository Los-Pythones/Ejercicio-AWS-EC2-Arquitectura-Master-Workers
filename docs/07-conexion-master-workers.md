# 07 / Conexión Master → Workers por SSH

Con las instancias ya corriendo, el siguiente paso es habilitar la comunicación desde el Master hacia cada Worker a través de SSH. Para esto se usa **Termius**, el cliente SSH que el equipo ya tiene instalado.

| Paso | Responsable | Acción                                                         |
|:-----|:------------|:---------------------------------------------------------------|
| 7.1  | Todos       | Entender por qué el Master necesita las llaves de los Workers  |
| 7.2  | Líder       | Crear el Host del Master en Termius y conectarse               |
| 7.3  | Líder       | Copiar los `.pem` de los Workers al Master vía SFTP en Termius |
| 7.4  | Líder       | Editar `~/.ssh/config` en el Master con nano                   |
| 7.5  | Líder       | Ajustar permisos del `config` y los `.pem`                     |
| 7.6  | Líder       | Probar la conexión desde el Master hacia cada Worker           |

---

## 7.1 / ¿Por qué el Master necesita la llave de los Workers?

Cada Worker fue creado con su propio Key Pair (archivo `.pem`). Cuando el Líder intenta conectarse desde el Master hacia un Worker por SSH, el sistema exige autenticarse con esa llave privada.

El problema es que la llave `.pem` de cada Worker fue descargada en el computador local del Líder, no en el Master. Para que el Master pueda iniciar conexiones SSH hacia los Workers, el archivo `.pem` correspondiente debe estar físicamente dentro del Master.

El flujo completo es:

```
Computador local del Líder
        │
        │  (SSH con SRV-MASTER-KEY.pem)
        ▼
  SRV-MASTER-DATA  ──────────────────────────────────────────────┐
        │  (SSH con SRV-WORKER-X-KEY.pem)                        │
        ▼                                                         │
  SRV-WORKER-1                                                   │
  SRV-WORKER-2    ◄──── Solo accesibles desde dentro de la VPC ─┘
  SRV-WORKER-3
```

> 💡 Los Workers no tienen IP pública, por lo que **no es posible conectarse a ellos directamente desde internet**. El Master actúa como punto de entrada (*jump host*) hacia la red privada.

---

## 7.2 / Crear el Host del Master en Termius y conectarse

Antes de poder transferir archivos al Master, el Líder debe tener configurada la sesión del Master en Termius. Este es el primer paso que permite operar todo lo demás desde la interfaz gráfica.

### Requisito previo

El Líder debe tener en su computador local:

- El archivo `SRV-MASTER-KEY.pem` descargado al crear la instancia Master.
- El **DNS público** del Master copiado desde la consola de AWS (campo **Public IPv4 DNS** en la pestaña Details de la instancia `SRV-MASTER-DATA`).

> ⚠️ El DNS público tiene el formato `ec2-XX-XX-XX-XX.compute-1.amazonaws.com`. Cambia cada vez que la instancia se apaga y se vuelve a encender.

### Crear el Keychain en Termius

El Keychain es donde Termius almacena las llaves privadas de forma segura. Antes de crear el Host, se debe registrar la llave del Master.

1. Abrir **Termius**.
2. En la barra lateral izquierda hacer clic en **Keychain**.
3. Hacer clic en el botón `+` → seleccionar `New Key`.
4. Completar los campos:

| Campo           | Valor                                                                                            |
|:----------------|:-------------------------------------------------------------------------------------------------|
| **Label**       | `SRV-MASTER-KEY`                                                                                 |
| **Private Key** | Hacer clic en `Import from file` y seleccionar el archivo `SRV-MASTER-KEY.pem` descargado de AWS |
| **Passphrase**  | Dejar vacío (las llaves de AWS no tienen passphrase por defecto)                                 |
![Keychan Config](../assets/screenshots/Termius/00-keychan-config.png)
5. Los cambios se guardan automáticamente, por lo que no tienes que pulsar ningún botón; puedes continuar con el siguiente paso.

> 💡 Una vez guardada la llave en el Keychain, Termius la recordará para futuras sesiones. No es necesario volver a importarla.

### Crear el Host del Master en Termius

1. En la barra lateral izquierda hacer clic en **Hosts**.
2. Hacer clic en el botón `+` → seleccionar `New Host`.
3. Completar los campos:

| Campo                   | Valor                                                              |
|:------------------------|:-------------------------------------------------------------------|
| **Address**             | DNS público del Master (`ec2-XX-XX-XX-XX.compute-1.amazonaws.com`) |
| **Label**               | `SRV-MASTER-DATA`                                                  |
| **Port**                | `22`                                                               |
| **Username**            | `ubuntu`                                                           |
| **Password**            | Dejar vacío                                                        |
| **Keys & Certificates** | Seleccionar la llave `SRV-MASTER-KEY` creada en el Keychain        |
![Host Config](../assets/screenshots/Termius/01-host-config.png)
4. Puedes darle al botón azul `Connect` para entrar a la instancia.

### Verificar conexión con la Master

1. Termius abrirá una terminal conectada al Master.
2. El prompt debe mostrar algo como:

```
ubuntu@ip-10-0-1-XXX:~$
```

Esto confirma que la conexión al Master está funcionando correctamente.

> ⚠️ Si la conexión falla, verificar que el DNS público del Master no haya cambiado. Ir a la consola de AWS → instancia `SRV-MASTER-DATA` → pestaña **Details** → campo **Public IPv4 DNS**, copiarlo y actualizar el campo **Address** en el Host de Termius.

---

## 7.3 / Copiar los `.pem` de los Workers al Master

Con la sesión del Master activa en Termius, el Líder puede transferir los archivos `.pem` de los Workers desde su computador local hacia el Master usando el **panel SFTP integrado de Termius**, sin necesidad de comandos.

### Requisito previo

El Líder debe tener en su computador local los tres archivos `.pem` de los Workers. Si los Workers generaron sus propias llaves, deben habérselas enviado al Líder antes de este paso.

```
SRV-WORKER-1-KEY.pem
SRV-WORKER-2-KEY.pem
SRV-WORKER-3-KEY.pem
```

> ⚠️ Sin los archivos `.pem`, no es posible continuar con los pasos siguientes.

### Transferir los archivos con el panel SFTP de Termius

1. En la sesión abierta del Master en Termius, hacer clic en el ícono **SFTP** en la barra superior de la terminal (o ir a `View → SFTP Panel`).
2. Termius abrirá un panel dividido en dos columnas:
    - **Columna izquierda**: sistema de archivos del computador local del Líder.
    - **Columna derecha**: sistema de archivos del Master (instancia EC2).
![Tranfers Workerss Keys](../assets/screenshots/Termius/02-transferir-keys.png)
3. En la **columna derecha** (Master), navegar hasta el directorio home del usuario:

```
/home/ubuntu/
```

4. En la **columna izquierda** (local), navegar hasta la carpeta donde están los tres archivos `.pem` de los Workers.
5. Seleccionar los tres archivos `.pem` y arrastrarlos hacia la columna derecha, o hacer clic derecho sobre ellos → `Upload`.
6. Esperar a que la transferencia termine. Termius mostrará una barra de progreso por cada archivo.

### Verificar que los archivos llegaron correctamente

En la terminal del Master ejecutar:

```bash
ls -la /home/ubuntu/*.pem
```

La salida esperada debe mostrar los tres archivos:

```
-rw-rw-r-- 1 ubuntu ubuntu 1678 ... SRV-WORKER-1-KEY.pem
-rw-rw-r-- 1 ubuntu ubuntu 1678 ... SRV-WORKER-2-KEY.pem
-rw-rw-r-- 1 ubuntu ubuntu 1678 ... SRV-WORKER-3-KEY.pem
```

> ⚠️ Los permisos en este momento son incorrectos (`rw-rw-r--`). Esto se corregirá en el **paso 7.5**. Si se intenta usar SSH con estos permisos, el sistema lo rechazará con el error `WARNING: UNPROTECTED PRIVATE KEY FILE`.

---

## 7.4 / Editar `~/.ssh/config` con nano

El archivo `~/.ssh/config` permite definir alias y configuraciones para cada conexión SSH. En lugar de escribir cada vez el comando completo con IP, usuario y llave, basta con escribir `ssh worker-1` y SSH usará la configuración definida automáticamente.

### Abrir o crear el archivo

En la terminal del Master ejecutar:

```bash
nano ~/.ssh/config
```

Si el archivo no existe, `nano` lo creará automáticamente al guardar por primera vez.

### Contenido del archivo

Escribir el siguiente bloque para cada Worker, reemplazando las IPs privadas con las que cada Worker compartió con el Líder al finalizar el paso 6.2:

```text
Host worker-1
    HostName 10.0.2.XXX
    User ubuntu
    IdentityFile /home/ubuntu/SRV-WORKER-1-KEY.pem

Host worker-2
    HostName 10.0.2.XXX
    User ubuntu
    IdentityFile /home/ubuntu/SRV-WORKER-2-KEY.pem

Host worker-3
    HostName 10.0.2.XXX
    User ubuntu
    IdentityFile /home/ubuntu/SRV-WORKER-3-KEY.pem
```

### Guardar y salir de nano

| Teclas     | Acción                                     |
|:-----------|:-------------------------------------------|
| `Ctrl + O` | Guardar el archivo (confirmar con `Enter`) |
| `Ctrl + X` | Salir de nano                              |

### Explicación línea por línea

| Directiva                  | Valor de ejemplo                    | Significado                                                                                                                                                                           |
|:---------------------------|:------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Host`                     | `worker-1`                          | Alias corto que se usará en el comando `ssh worker-1`. Puede ser cualquier nombre sin espacios.                                                                                       |
| `HostName`                 | `10.0.2.XXX`                        | IP privada del Worker dentro de la VPC. Se obtiene del campo **Private IPv4 address** en la consola de AWS.                                                                           |
| `User`                     | `ubuntu`                            | Usuario por defecto de las instancias Ubuntu en EC2. No cambia.                                                                                                                       |
| `IdentityFile`             | `/home/ubuntu/SRV-WORKER-1-KEY.pem` | Ruta absoluta al archivo `.pem` que corresponde a ese Worker. Cada Worker tiene su propio archivo.                                                                                    |


> 💡 Se usa la **IP privada** del Worker en lugar de un DNS porque la IP privada dentro de la VPC es estable: no cambia al apagar y encender la instancia. Esto hace la configuración más confiable a lo largo de la actividad.

---

## 7.5 / Ajustar permisos del `config` y los `.pem`

SSH es estricto con los permisos de los archivos de llave y configuración. Si los permisos son demasiado abiertos (es decir, otros usuarios del sistema también pueden leerlos), SSH rechaza la conexión por seguridad.

Ejecutar los siguientes comandos en la terminal del Master, **uno por uno**:

```bash
chmod 600 ~/.ssh/config
```

```bash
chmod 400 /home/ubuntu/SRV-WORKER-1-KEY.pem
chmod 400 /home/ubuntu/SRV-WORKER-2-KEY.pem
chmod 400 /home/ubuntu/SRV-WORKER-3-KEY.pem
```

### ¿Qué significan estos permisos?

| Comando     | Permiso resultante | Significado                                                                                                       |
|:------------|:-------------------|:------------------------------------------------------------------------------------------------------------------|
| `chmod 600` | `-rw-------`       | Solo el dueño puede leer y escribir. Nadie más puede acceder al archivo. Requerido para `~/.ssh/config`.          |
| `chmod 400` | `-r--------`       | Solo el dueño puede leer. Ni siquiera el dueño puede modificarlo accidentalmente. Requerido para archivos `.pem`. |

### Verificar que los permisos quedaron correctos

```bash
ls -la ~/.ssh/config /home/ubuntu/*.pem
```

La salida esperada:

```
-rw------- 1 ubuntu ubuntu  XXX ... /home/ubuntu/.ssh/config
-r-------- 1 ubuntu ubuntu 1678 ... /home/ubuntu/SRV-WORKER-1-KEY.pem
-r-------- 1 ubuntu ubuntu 1678 ... /home/ubuntu/SRV-WORKER-2-KEY.pem
-r-------- 1 ubuntu ubuntu 1678 ... /home/ubuntu/SRV-WORKER-3-KEY.pem
```

> ⚠️ Si los permisos son incorrectos al intentar conectarse, SSH mostrará el error `WARNING: UNPROTECTED PRIVATE KEY FILE` y rechazará la conexión. La solución siempre es volver a ejecutar el `chmod` correspondiente.

---

## 7.6 / Probar la conexión desde el Master hacia cada Worker

Con todo configurado, ya es posible conectarse a cada Worker directamente desde la terminal del Master usando los alias definidos en `~/.ssh/config`.

### Conectarse a Worker 1

Desde la terminal del Master en Termius ejecutar:

```bash
ssh worker-1
```

Si la conexión es exitosa, el prompt cambiará y mostrará algo como:

```
ubuntu@ip-10-0-2-XXX:~$
```

Esto indica que se está dentro del Worker 1. Para salir y volver al Master:

```bash
exit
```

### Repetir para los demás Workers

```bash
ssh worker-2
```

```bash
ssh worker-3
```

### Verificar el hostname dentro de cada Worker

Como confirmación adicional, ejecutar dentro de cada sesión de Worker:

```bash
hostname -I
```

La IP que aparece debe coincidir con la IP privada del Worker al que se conectó, tal como fue compartida en el paso 6.2.

### Tabla resumen de conexiones

| Desde            | Hacia    | Método                           | Llave usada                                   |
|:-----------------|:---------|:---------------------------------|:----------------------------------------------|
| Computador local | Master   | Termius → Host `SRV-MASTER-DATA` | `SRV-MASTER-KEY.pem` (en Keychain de Termius) |
| Master           | Worker 1 | `ssh worker-1` en terminal       | `SRV-WORKER-1-KEY.pem` (en `/home/ubuntu/`)   |
| Master           | Worker 2 | `ssh worker-2` en terminal       | `SRV-WORKER-2-KEY.pem` (en `/home/ubuntu/`)   |
| Master           | Worker 3 | `ssh worker-3` en terminal       | `SRV-WORKER-3-KEY.pem` (en `/home/ubuntu/`)   |

> 💡 Si en algún momento la conexión desde Termius al Master falla, recordar que el DNS público del Master **cambia cada vez que la instancia se reinicia**. Verificar el nuevo DNS en la consola de AWS y actualizar el campo **Address** en el Host de Termius.