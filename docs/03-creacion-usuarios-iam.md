# 03 / Lo que hace el Líder: crear los usuarios IAM
Esta sección está dirigida principalmente al Líder del equipo, ya que es quien ejecuta todos los pasos aquí descritos. El resto del equipo solo necesita leerla para entender qué sucedió antes de que recibieran sus credenciales.

---

# ¿Por qué no trabajar directamente con la cuenta root?
La cuenta root es la que se creó al registrarse en AWS con el correo del equipo. Tiene acceso total e irrestricto a todos los recursos y configuraciones de la cuenta, sin ninguna limitación.
Trabajar directamente con la cuenta root para las tareas del día a día representa un riesgo innecesario: cualquier error cometido desde esa cuenta puede tener consecuencias irreversibles. Por esa razón, la práctica estándar en AWS es usar la cuenta root únicamente para la configuración inicial y luego crear usuarios IAM individuales para cada persona, con únicamente los permisos que necesitan para su rol.
En esta actividad, el Líder usó la cuenta root exclusivamente para crear los grupos, las políticas y los usuarios IAM. A partir de ese punto, cada integrante trabajó desde su propio usuario.

--- 

## Paso 1 / Ingresar a IAM desde la consola de AWS

1. Iniciar sesión en [Amazon](https://aws.amazon.com) con el correo y contraseña del equipo (cuenta root).
2. En la barra de búsqueda superior escribir IAM y seleccionar el servicio.
![Search IAM](../assets/screenshots/IAM/01-buscar-IAM.png)

---

## Paso 2 / Crear los grupos con sus políticas
Los grupos permiten asignar permisos a varios usuarios a la vez. En esta actividad se crearon dos grupos, cada uno con un conjunto de permisos diferente.<br><br>
#### Crear el grupo Ingenieros de Datos

1. En el panel de IAM, ir a User groups → `Create group`.
![Select Create Group](../assets/screenshots/IAM/02-seleccionar-createGroup.png)
2. Asignar el nombre `Data Ingenners`.
3. En la sección Attach permissions policies, buscar y seleccionar las siguientes políticas:

| Política                 | Permiso que otorga                                        |
|:-------------------------|:----------------------------------------------------------|
| `AmazonEC2FullAccess`    | Control completo sobre instancias EC2                     |
| `AmazonS3ReadOnlyAccess` | Lectura de objetos en S3                                  |
| `AmazonS3FullAccess`     | (Ajustar para excluir `s3:DeleteObject` — ver nota abajo) |
| `IAMUserChangePassword`  | Permite cambiar su propia contraseña                      |

> ⚠️ Nota sobre S3: AWS no tiene una política predefinida que otorgue lectura/escritura en S3 sin permiso de borrado. Para aplicar esto correctamente se debe crear una **política personalizada** con las acciones `s3:GetObject`, `s3:PutObject` y `s3:ListBucket` explícitamente, sin incluir `s3:DeleteObject`. Si en la actividad se usó `AmazonS3FullAccess` por simplicidad, es válido para el ejercicio académico.

PD para los quieran crear la politica personalida: [`Custom Policy`](02-políticas-personalizadas.md)

4. Hacer clic en `Create group`.

#### Crear el grupo Analistas-de-Datos (Mismo proceso que el de los ingenieros solo que cambian las politicas que se le asignan)

1. Ir a User groups → `Create group`.
2. Asignar el nombre `Data analysts` .
3. Seleccionar las siguientes políticas:


| Política                 | Permiso que otorga                                        |
|:-------------------------|:----------------------------------------------------------|
| `AmazonEC2FullAccess`    | Control completo sobre instancias EC2                     |
| `AmazonAthenaFullAccess` | Acceso total al servicio Amazon Athena                            |

4. Hacer clic en `Create group`.

---

## Paso 3 / Crear los usuarios IAM

Se creó un usuario por cada integrante del equipo. El proceso es el mismo para los cuatro.

1. En el panel de IAM ir a Users → `Create user`.
2. Asignar un nombre de usuario representativo. Por ejemplo:

| Usuario | Grupo asignado |
| :--- | :--- |
| `lider-ingeniero` | `Data Ingenners` |
| `ingeniero-2` | `Data Ingenners` |
| `analista-1` | `Data analysts` |
| `analista-2` | `Data analysts` |


3. Activar la opción "Provide user access to the AWS Management Console".
![Put name](../assets/screenshots/IAM/03-poner-nombre.png)
4. Seleccionar el grupo al que pertenezca el usuario que estes creando.
![Select Group](../assets/screenshots/IAM/04-escoger-grupo.png)
5. En la sección de detalles del perfil podremos ver todo lo que le hemos asignado a ese usuario hasta ahora.
6. Finalizar con Create user.

---

## Paso 4 / Obtener las credenciales de cada usuario

Una vez creado cada usuario, AWS muestra un resumen con:

1. URL de inicio de sesión de la cuenta (única por cuenta de AWS, no por usuario)
2. Nombre de usuario
3. Contraseña temporal

> ⚠️ Esta información solo se muestra una vez. Es el momento de copiarla o descargar el archivo `.csv` que ofrece AWS antes de cerrar esa pantalla.

---

## Paso 5 / Enviar las credenciales a cada integrante

El Líder envió las credenciales de cada usuario por el canal de comunicación del equipo (grupo de WhatsApp o Discord), incluyendo:

```text
URL de acceso:  https://XXXXXXXXXXXX.signin.aws.amazon.com/console
Usuario:        analista-1
Contraseña:     ******* (temporal, deberás cambiarla al ingresar)
```

>💡 **Nota de seguridad**: compartir credenciales por mensajería instantánea no es una práctica recomendada en entornos productivos. Lo correcto sería usar un gestor de secretos o un canal cifrado. Sin embargo, para el contexto de esta actividad académica es suficiente.

