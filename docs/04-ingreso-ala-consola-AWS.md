# 04 / Ingreso a la consola de AWS con usuario IAM
Una vez que el Líder envió las credenciales por el canal de comunicación del equipo, cada integrante debió seguir los pasos de esta sección para ingresar a la consola de AWS con su propio usuario.

---

## Paso 1 / Acceder a la URL de inicio de sesión
La URL de inicio de sesión para usuarios IAM no es la misma que la de la cuenta root (`aws.amazon.com)`. Es una dirección única por cuenta y tiene este formato:

```text
https://XXXXXXXXXXXX.signin.aws.amazon.com/console
```

Donde `XXXXXXXXXXXX` es el ID de cuenta de AWS del equipo. Esta URL fue proporcionada por el Líder junto con las demás credenciales.

> ⚠️ Ingresar a través de esta URL es obligatorio. Si se intenta acceder desde `aws.amazon.com` con las credenciales IAM directamente, el sistema pedirá las credenciales de cuenta root, que son distintas.

---

## Paso 2 / Iniciar sesión

En la pantalla de inicio de sesión completar los siguientes campos:

| Campo                   | Valor                                                         |
|:------------------------|:--------------------------------------------------------------|
| **Account ID or alias** | Se completa automáticamente si se ingresó por la URL correcta |
| **IAM user name**       | El nombre de usuario enviado por el Líder                     |
| **Password**            | La contraseña temporal enviada por el Líder                   |

Hacer clic en `Sign in`.

---

## Paso 3 — Cambiar la contraseña en el primer ingreso
AWS solicitará de forma obligatoria el cambio de contraseña en el primer inicio de sesión, ya que el Líder activó la opción "Users must create a new password at next sign-in" al crear cada usuario.<br><br>
En la pantalla de cambio de contraseña:

1. En Current password ingresar la contraseña temporal recibida.
2. En New password escribir una contraseña nueva y personal.
3. En Confirm new password repetir la nueva contraseña.
4. Hacer clic en `Confirm password change`.

> 💡 La nueva contraseña queda bajo responsabilidad de cada usuario. El Líder no tiene forma de recuperarla desde IAM sin resetearla manualmente.

---

## Paso 4 / Reconocer los permisos del rol asignado
Una vez dentro de la consola, cada integrante verá la interfaz general de AWS. Sin embargo, los servicios y acciones disponibles varían según el grupo al que pertenece cada usuario.

#### Ingenieros de Datos
Tienen acceso a EC2 con capacidad de encender, apagar y crear instancias, además de acceso de lectura y escritura sobre S3 sin posibilidad de eliminar archivos.<br><br>
Al intentar acceder a un servicio para el cual no tienen permisos (por ejemplo, IAM), la consola mostrará un mensaje similar a:

```text
You don't have permissions to list IAM users.
```
Esto es el comportamiento esperado, no un error de configuración.

#### Analistas de Datos

Tienen acceso a EC2 con las mismas capacidades que los ingenieros, además de acceso completo a Amazon Athena. Los servicios fuera de estos dos también estarán restringidos.

---

## Paso 5 / Verificar la región de trabajo

Antes de crear cualquier recurso, cada integrante debe verificar que está trabajando en la misma región de AWS que el resto del equipo. La región activa se muestra en la esquina superior derecha de la consola.<br>

Si no coincide con la del equipo, los recursos creados no serán visibles entre sí.

> 💡 Para esta actividad todos debieron trabajar en la misma región. Confirmar con el Líder cuál fue la región seleccionada si hay dudas.