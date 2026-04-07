# 02 — Creación de políticas personalizadas en IAM
Cuando los permisos que necesita un grupo no coinciden exactamente con las políticas predefinidas que ofrece AWS, se debe crear una política personalizada (custom policy). Este fue el caso del grupo `Data Ingenners`, donde se requería acceso a S3 con lectura y escritura pero sin permiso de borrado.

---

## ¿Qué es una política en IAM?
Una política es un documento en formato JSON que le dice a AWS exactamente qué acciones están permitidas, sobre qué recursos y bajo qué condiciones. Toda política tiene la misma estructura base:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "NombreDescriptivo",
            "Effect": "Allow" o "Deny",
            "Action": [ "servicio:acción" ],
            "Resource": [ "arn del recurso" ]
        }
    ]
}
```

| Campo       | Descripción                                                      |
|:------------|:-----------------------------------------------------------------|
| `Version`   | Versión del lenguaje de políticas. Siempre `"2012-10-17"`        |
| `Statement` | Lista de reglas de permiso o denegación                          |
| `Sid`       | Identificador descriptivo del bloque (opcional pero recomendado) |
| `Effect`    | `Allow` para permitir, `Deny` para denegar                       |
| `Action`    | Lista de acciones específicas del servicio de AWS                |
| `Resource`  | ARN del recurso sobre el que aplica. `*` significa todos         |

---
## Política personalizada / Data Engineers
Esta fue la política creada para el grupo Data Ingineers. Contiene dos bloques: uno para S3 y otro para EC2.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "S3ReadWriteNoDelete",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:ListBucket",
                "s3:GetBucketLocation"
            ],
            "Resource": [
                "arn:aws:s3:::*",
                "arn:aws:s3:::*/*"
            ]
        },
        {
            "Sid": "EC2ReadWriteNoDelete",
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances",
                "ec2:Describe*",
                "ec2:RunInstances"
            ],
            "Resource": "*"
        }
    ]
}
```

### ¿Qué permite exactamente cada bloque?
#### **Bloque** `S3ReadWriteNoDelete`:

| Acción                 | Qué permite                                       |
|:-----------------------|:--------------------------------------------------|
| `s3:GetObject`         | Leer y descargar archivos de un bucket            |
| `s3:PutObject`         | Subir y escribir archivos en un bucket            |
| `s3:ListBucket`        | Ver el contenido (lista de archivos) de un bucket |
| `s3:GetBucketLocation` | Consultar la región donde está alojado el bucket  |


Nótese que `s3:DeleteObject` no está incluido, lo que impide que un Ingeniero de Datos pueda eliminar archivos de S3.<br>

#### **Bloque** `EC2ReadWriteNoDelete`:

| `ec2:RunInstances` | Lanzar nuevas instancias |

| Acción               | Qué permite                                                 |
|:---------------------|:------------------------------------------------------------|
| `ec2:StartInstances` | Encender instancias                                         |
| `ec2:StopInstances`  | Apagar instancias                                           |
| `ec2:Describe*`      | Ver información de instancias, redes, security groups, etc. |
| `ec2:RunInstances`   | Lanzar nuevas instancias                                    |


De la misma forma, `ec2:TerminateInstances` **no está incluido**, por lo que ningún Ingeniero puede eliminar una instancia permanentemente.

---

## Cómo crear esta política en la consola de AWS

1. Ingresar a **IAM** → en el panel izquierdo seleccionar Policies → `Create policy`.
![Select Create Policy](../assets/screenshots/IAM/02.1-selecciona-createPolicy.png)
2. Seleccionar la pestaña **JSON**.
![Select JSON](../assets/screenshots/IAM/02.2-selecciona-JSON.png)
3. Borrar el contenido que aparece por defecto y pegar la política completa.
![Copy Code](../assets/screenshots/IAM/02.3-copiar-codigo.png)
4. Hacer clic en `Next`.
5. Asignar un nombre descriptivo a la política, por ejemplo: `DataEngineer-S3-EC2-ReadWrite-NoDelete`.
![Put name](../assets/screenshots/IAM/02.4-poner-nombre.png)
6. Opcionalmente agregar una descripción: "Política personalizada para Ingenieros de Datos: EC2 sin terminación, S3 sin borrado."
7. Hacer clic en `Create policy`.

Una vez creada, esta política aparece en el listado de políticas disponibles y puede asignarse al grupo `Data Engineers` durante su creación o posteriormente desde la configuración del grupo.