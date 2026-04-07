# 01 / Conceptos Previos
Antes de comenzar con la configuración del entorno, es importante que todos los miembros del equipo tengan claridad sobre los conceptos fundamentales que se utilizan a lo largo de esta actividad. Esta sección no reemplaza la documentación oficial de AWS, pero sí contiene el contexto minimo que necesitan para desarrollar la actividad. 🤠

---

## 1. ¿Qué es AWS?
AWS (Amazon Web Services) es la plataforma de servicios en la nube de Amazon. Ofrece infraestructura tecnológica bajo demanda: servidores, bases de datos, almacenamiento, redes, seguridad y mucho más, todo accesible desde internet sin necesidad de hardware físico propio.<br><br>
En lugar de comprar y mantener servidores, con AWS se alquila capacidad computacional y se paga únicamente por lo que se usa.<br><br>
En esta actividad usamos dos servicios principales de AWS:

| Servicio | Para qué lo usamos |
|:---------|:------------------|
| **IAM**  | Crear y gestionar los usuarios del equipo con sus permisos |
| **EC2**  | Crear las máquinas virtuales (Master y Workers) |

---

## 2. ¿Qué es IAM?
**IAM** (Identity and Access Management) es el servicio de AWS que permite gestionar quién tiene acceso a los recursos de la cuenta y qué puede hacer con ellos.<br><br>
Dentro de IAM existen cuatro elementos clave:<br><br>
**`Usuarios`**: representa a una persona real dentro de la cuenta de AWS. En esta actividad, el Líder creó un usuario por cada integrante del equipo.<br><br>
**`Grupos`**: conjunto de usuarios que comparten los mismos permisos. En lugar de asignar permisos uno por uno, se asignan al grupo y todos los usuarios dentro de él los heredan automáticamente.<br><br>
En esta actividad se crearon dos grupos:

- `Ingenieros de Datos`: Permisos completos en EC2 + Lectura/Escritura en S3 (sin borrado).
- `Analistas de Datos`: Permisos completos en EC2 + Acceso total a Amazon Athena.
- `Ambos`: Permiso para cambiar su propia contraseña de IAM.

**Política** (Policy): documento que define exactamente qué acciones están permitidas o denegadas. Se asigna a grupos o usuarios. <br>

**Rol** (Role): conjunto de permisos que se le asigna a un recurso de AWS (como una instancia EC2) en lugar de a una persona. Permite que esa instancia interactúe con otros servicios de AWS de forma controlada.

---

## 3. ¿Qué es EC2?
EC2 (Elastic Compute Cloud) es el servicio de AWS que permite crear y administrar máquinas virtuales en la nube, llamadas `instancias`.<br><br>
Una instancia EC2 funciona como un servidor: tiene sistema operativo, CPU, memoria RAM y almacenamiento. En esta actividad todas las instancias corren Ubuntu Server como sistema operativo.<br><br>
Cada integrante del equipo creó su propia instancia según su rol:

| Instancia  | Creada por           | Propósito        |
|:-----------|:---------------------|:-----------------|
| `master`   | Líder                | Nodo principal, se conecta a todos los Workers |
| `worker-1` | Ingeniero de Datos 2 | Nodo secundario  |
| `worker-2` | Analista de Datos 1  | Nodo secundario  |
| `worker-3` | Analista de Datos 2  | Nodo secundario  |

---

## 4. ¿Qué es una VPC?

**VPC** (Virtual Private Cloud) es una red virtual privada dentro de AWS. Funciona como si fuera la red interna de una oficina, pero completamente en la nube. Dentro de una VPC se definen rangos de direcciones IP, subnets, reglas de tráfico y conexiones hacia internet o hacia otros servicios de AWS.

Cuando se crea una cuenta de AWS, esta genera automáticamente una VPC por defecto en cada región. En esta actividad **no usamos esa VPC por defecto**, sino que creamos una propia para tener control total sobre la arquitectura de red.

La VPC que configuramos tiene el siguiente rango de direcciones IP:
```text
10.0.0.0/16
```

Esto significa que la red puede contener hasta 65.536 direcciones IP internas, más que suficiente para el ejercicio. El rango `/16` es el más amplio disponible en AWS para una VPC y es el estándar recomendado para redes que necesitan subdividirse en múltiples subnets.

---

## 5. ¿Qué es una Subnet?

Una **Subnet** (subred) es una subdivisión de la VPC. Permite separar los recursos en segmentos de red con comportamientos distintos, por ejemplo: algunos con acceso a internet y otros sin él.

En esta actividad se crearon dos subnets dentro de la VPC:

| Subnet | Tipo | Usada por | Acceso a internet |
|:---|:---|:---|:---|
| `10.0.1.0/24` | Pública | Instancia Master | ✅ Sí |
| `10.0.2.0/24` | Privada | Instancias Worker | ❌ No |

**Subnet pública**: está asociada a una tabla de ruteo que dirige el tráfico hacia un Internet Gateway, lo que permite que la instancia Master tenga una IP pública y sea accesible desde internet.

**Subnet privada**: no tiene ruta hacia internet. Las instancias Worker viven aquí, lo que significa que no son accesibles directamente desde fuera de la VPC. Solo el Master puede comunicarse con ellas, desde adentro de la red.

---

## 6. ¿Qué es un Internet Gateway?

Un **Internet Gateway** (IGW) es el componente que conecta una VPC con internet. Sin él, ningún recurso dentro de la VPC puede enviar o recibir tráfico desde el exterior, sin importar si tiene IP pública o no.

Para que funcione correctamente se deben cumplir dos condiciones:

1. El Internet Gateway debe estar **creado y asociado** a la VPC.
2. La **tabla de ruteo** de la subnet pública debe tener una regla que dirija el tráfico de salida (`0.0.0.0/0`) hacia ese Internet Gateway.

En esta actividad el Internet Gateway se creó y asoció a la VPC del equipo, y se configuró en la tabla de ruteo de la subnet pública para que el Master pudiera tener acceso a internet. Las instancias Worker, al estar en la subnet privada, no pasan por el Internet Gateway y por eso no tienen salida a internet.

---

## 7. ¿Qué es una Tabla de Ruteo?

Una **Tabla de Ruteo** (Route Table) es un conjunto de reglas que le indican a la VPC hacia dónde debe dirigir el tráfico de red según su destino.

Cada subnet está asociada a una tabla de ruteo. En esta actividad se configuraron dos:

**Tabla de ruteo pública** — asociada a la subnet del Master:

| Destino | Target | Significado |
|:---|:---|:---|
| `10.0.0.0/16` | local | El tráfico interno de la VPC se queda dentro |
| `0.0.0.0/0` | Internet Gateway | Todo el tráfico externo sale por el IGW hacia internet |

**Tabla de ruteo privada** — asociada a la subnet de los Workers:

| Destino | Target | Significado |
|:---|:---|:---|
| `10.0.0.0/16` | local | Solo tráfico interno de la VPC. Sin salida a internet |

La regla `local` existe por defecto en toda VPC y garantiza que las instancias dentro de la misma red puedan comunicarse entre sí. Es gracias a esta regla que el Master puede llegar a los Workers por SSH usando su **IP privada**, sin necesidad de pasar por internet.

---

## 8. ¿Qué es un Security Group?
Un **Security Group** es el firewall virtual de una instancia EC2. Define qué tráfico de red tiene permitido entrar (inbound) y salir (outbound) de la instancia.

| Campo      | Descripción                                       |
|:-----------|:--------------------------------------------------|
| **Tipo**   | El protocolo (SSH, HTTP, HTTPS, etc.)             |
| **Puerto** | El número de puerto (SSH usa el 22)               |
| **Origen** | Desde qué IP o rango se permite el acceso         |

En esta actividad, el Security Group de cada instancia fue configurado para **permitir conexiones SSH (puerto 22)**. Sin esta regla, ninguna conexión por terminal sería posible.

> 💡 Para pruebas en entornos académicos se suele permitir el origen `0.0.0.0/0` (cualquier IP). En entornos productivos esto es una mala práctica; lo correcto es restringirlo a IPs específicas.

---

## 9. ¿Qué es una Key Pair?
Una **Key Pair** (par de llaves) es el mecanismo de autenticación que utiliza AWS para permitir el acceso a una instancia EC2 mediante SSH.<br><br>
Está compuesta por dos partes:<br><br>
**Llave pública**: AWS la instala automáticamente dentro de la instancia al momento de crearla. No es necesario manipularla directamente.<br><br>
**Llave privada (.pem)**: es el archivo que se descarga al crear el Key Pair desde la consola de AWS. Es el único medio para autenticarse ante la instancia. Si se pierde, no es posible recuperarlo.

> ⚠️ El archivo `.pem` se descarga una sola vez. Una vez cerrado el asistente de creación, AWS no vuelve a mostrarlo. Guárdalo en un lugar seguro desde el primer momento.

En esta actividad, cada integrante generó su propio Key Pair al crear su instancia.

---

## 10. ¿Qué es SSH y cómo funciona el sistema de llaves?
**SSH** (Secure Shell) es un protocolo de red que permite conectarse a una máquina remota de forma segura a través de la terminal. Es el método que usamos para acceder a las instancias EC2.<br><br>
El sistema de autenticación funciona así:

```bash
Al crear la instancia:
  AWS instala la llave pública dentro del servidor (instancia EC2)

Al conectarse:
  Tu PC presenta la llave privada (.pem)
  La instancia verifica que coincida con la llave pública que tiene guardada
  Si coinciden → acceso concedido
  Si no coinciden → acceso denegado
```

En esta actividad usamos **`Termius`** como cliente SSH, que ofrece una interfaz gráfica para gestionar las conexiones sin necesidad de usar comandos en la terminal.<br><br>
La configuración de cada conexión en Termius se hizo así:

1. Obtener el DNS público de la instancia
   Desde la consola de AWS → EC2 → seleccionar la instancia → copiar el valor del campo **Public IPv4 DNS**. Se ve así:

```bash
ec2-XX-XX-XX-XX.compute-1.amazonaws.com
```

2. Crear el host en Termius

- Abrir Termius → New Host
- En el campo **Address** pegar el DNS público copiado
- En el campo **Username** escribir ubuntu (usuario por defecto en instancias Ubuntu de AWS)

3. Importar la llave privada

- Dentro de la configuración del host, ir a la sección **Keys**
- Seleccionar Import from file y cargar el archivo .pem descargado al crear el Key Pair
- Termius usa esa llave para autenticarse automáticamente cada vez que se conecta

4. Conectarse

- Hacer doble clic sobre el host creado
- Termius establece la conexión SSH y abre la terminal de la instancia

> ⚠️ El DNS público cambia cada vez que se apaga y vuelve a encender la instancia. Si la conexión falla, verificar en la consola de AWS que el DNS en Termius siga siendo el correcto y actualizarlo si es necesario.