# Ejercicio AWS EC2 Arquitectura Master / Workers

### Riwi · Cohorte 6 · Analitica de Datos

---
 
> **¿Para quién es este repo?**  
> Para todos los que no entendieron muy bien qué hicimos, se perdieron alguna clase o simplemente quieren tener todo documentado en un solo lugar (Yo).
 
---

## Sección 0 / Introduccción

1. [`Contexto de la actividad`](docs/00-introducción.md#contexto-de-la-actividad)    
2. [`Diagrama de los equipos con sus roles`](docs/00-introducción.md#diagrama-de-los-equipos-con-sus-roles)    

---

## Sección 1 / Conceptos claves

1. [`¿Qué es AWS?`](docs/01-conceptos-previos.md#1-qué-es-aws)
2. [`¿Qué es IAM?`](docs/01-conceptos-previos.md#2-qué-es-iam)
3. [`¿Qué es EC2?`](docs/01-conceptos-previos.md#3-qué-es-ec2)
4. [`¿Qué es una VPC?`](docs/01-conceptos-previos.md#4-qué-es-una-vpc)
5. [`¿Qué es una Subnet?`](docs/01-conceptos-previos.md#5-qué-es-una-subnet)
6. [`¿Qué es un Internet Gateway?`](docs/01-conceptos-previos.md#6-qué-es-un-internet-gateway)
7. [`¿Qué es una Tabla de Ruteo?`](docs/01-conceptos-previos.md#7-qué-es-una-tabla-de-ruteo)
8. [`¿Qué es un Security Group?`](docs/01-conceptos-previos.md#8-qué-es-un-security-group)
9. [`¿Qué es una Key Pair?`](docs/01-conceptos-previos.md#9-qué-es-una-key-pair)
10. [`¿Qué es SSH y cómo funciona el sistema de llaves?`](docs/01-conceptos-previos.md#10-qué-es-ssh-y-cómo-funciona-el-sistema-de-llaves)

---

## Sección 2 / Políticas personalizadas en IAM

1. [`¿Qué es una política en IAM?`](docs/02-políticas-personalizadas.md#qué-es-una-política-en-iam)
2. [`Política personalizada — Data Engineers`](docs/02-políticas-personalizadas.md#política-personalizada--data-engineers)
    - [`Bloque S3ReadWriteNoDelete`](docs/02-políticas-personalizadas.md#bloque-s3readwritenodelete)
    - [`Bloque EC2ReadWriteNoDelete`](docs/02-políticas-personalizadas.md#bloque-ec2readwritenodelete)
3. [`Cómo crear la política en la consola de AWS`](docs/02-políticas-personalizadas.md#cómo-crear-esta-política-en-la-consola-de-aws)

---

## Sección 3 / Lo que hace el Líder: crear los usuarios IAM

1. [`¿Por qué no usar la cuenta root?`](docs/03-creacion-usuarios-iam.md#por-qué-no-trabajar-directamente-con-la-cuenta-root)
2. [`Ingresar a IAM desde la consola de AWS`](docs/03-creacion-usuarios-iam.md#paso-1--ingresar-a-iam-desde-la-consola-de-aws)
3. [`Crear los grupos con sus políticas`](docs/03-creacion-usuarios-iam.md#paso-2--crear-los-grupos-con-sus-políticas)
    - [`Crear el grupo Data Engineers`](docs/03-creacion-usuarios-iam.md#crear-el-grupo-ingenieros-de-datos)
   - [`Crear el grupo Data Analysts`](docs/03-creacion-usuarios-iam.md#crear-el-grupo-analistas-de-datos-mismo-proceso-que-el-de-los-ingenieros-solo-que-cambian-las-politicas-que-se-le-asignan)
5. [`Crear los 4 usuarios IAM`](docs/03-creacion-usuarios-iam.md#paso-3--crear-los-usuarios-iam)
6. [`Obtener las credenciales de cada usuario`](docs/03-creacion-usuarios-iam.md#paso-4--obtener-las-credenciales-de-cada-usuario)
7. [`Enviar las credenciales al equipo`](docs/03-creacion-usuarios-iam.md#paso-5--enviar-las-credenciales-a-cada-integrante)

---

## Sección 4 / Ingreso a la consola de AWS con usuario IAM

1. [`Acceder a la URL de inicio de sesión`](docs/04-ingreso-ala-consola-AWS.md#paso-1--acceder-a-la-url-de-inicio-de-sesión)
2. [`Iniciar sesión`](docs/04-ingreso-ala-consola-AWS.md#paso-2--iniciar-sesión)
3. [`Cambiar la contraseña en el primer ingreso`](docs/04-ingreso-ala-consola-AWS.md#paso-3--cambiar-la-contraseña-en-el-primer-ingreso)
4. [`Reconocer los permisos del rol asignado`](docs/04-ingreso-ala-consola-AWS.md#paso-4--reconocer-los-permisos-del-rol-asignado)
5. [`Verificar la región de trabajo`](docs/04-ingreso-ala-consola-AWS.md#paso-5--verificar-la-región-de-trabajo)

---

## Sección 5 / Configuración de la VPC

1. [`¿Por qué crear una VPC propia?`](docs/05-configuracion-vpc.md#paso-1--por-qué-crear-una-vpc-propia)
2. [`Crear la VPC`](docs/05-configuracion-vpc.md#paso-2--crear-la-vpc)
3. [`Crear la Subnet Pública`](docs/05-configuracion-vpc.md#paso-3--crear-la-subnet-pública-master)
4. [`Crear la Subnet Privada`](docs/05-configuracion-vpc.md#paso-4--crear-la-subnet-privada-workers)
5. [`Crear el Internet Gateway`](docs/05-configuracion-vpc.md#paso-5--crear-el-internet-gateway)
6. [`Asociar el Internet Gateway a la VPC`](docs/05-configuracion-vpc.md#paso-6--asociar-el-internet-gateway-a-la-vpc)
7. [`Configurar la Tabla de Ruteo pública`](docs/05-configuracion-vpc.md#paso-7--configurar-la-tabla-de-ruteo-pública)
8. [`Crear los Security Groups`](docs/05-configuracion-vpc.md#paso-8--crear-los-security-groups)
9. [`VPC Endpoint — intento fallido`](docs/05-configuracion-vpc.md#paso-9--vpc-endpoint--intento-fallido)

---

## Sección 6 / Creación de las instancias EC2

1. **6.1 El Líder crea el Master**
   - [`Ingresar a EC2 y lanzar una instancia`](docs/06-crear-instancias.md#paso-1--ingresar-a-ec2-y-lanzar-una-instancia)
   - [`Nombre de la instancia`](docs/06-crear-instancias.md#paso-2--nombre-de-la-instancia)
   - [`Elegir la AMI`](docs/06-crear-instancias.md#paso-3--elegir-la-ami)
   - [`Tipo de instancia`](docs/06-crear-instancias.md#paso-4--tipo-de-instancia)
   - [`Crear el Key Pair`](docs/06-crear-instancias.md#paso-5--crear-el-key-pair)
   - [`Configurar Network settings`](docs/06-crear-instancias.md#paso-6--configurar-network-settings)   
   - [`Almacenamiento`](docs/06-crear-instancias.md#paso-7--almacenamiento)
   - [`Lanzar la instancia`](docs/06-crear-instancias.md#paso-8--lanzar-la-instancia)
   - [`Obtener el DNS público y compartirlo`](docs/06-crear-instancias.md#paso-9--obtener-el-dns-público-y-compartirlo-con-el-equipo)

2. **6.2 Cada Worker crea su instancia**
   - [`Diferencias en Network settings`](docs/06-crear-instancias.md#paso-6--configurar-network-settings-workers)
   - [`Compartir la IP privada con el Líder`](docs/06-crear-instancias.md#al-finalizar--compartir-la-ip-privada-con-el-líder)

---

## Sección 7 / Conexión Master → Workers por SSH

1. [`¿Por qué el Master necesita la llave de los Workers?`](docs/07-conexion-master-workers.md#71--por-qué-el-master-necesita-la-llave-de-los-workers)
2. [`Crear el Host del Master en Termius y conectarse`](docs/07-conexion-master-workers.md#72--crear-el-host-del-master-en-termius-y-conectarse)
   - [`Crear el Keychain en Termius`](docs/07-conexion-master-workers.md#crear-el-keychain-en-termius)
   - [`Crear el Host del Master en Termius`](docs/07-conexion-master-workers.md#crear-el-host-del-master-en-termius)
   - [`Conectarse al Master`](docs/07-conexion-master-workers.md#verificar-conexión-con-el-master)
3. [`Copiar los .pem de los Workers al Master`](docs/07-conexion-master-workers.md#73--copiar-los-pem-de-los-workers-al-master)
   - [`Opción A — Transferir con el panel SFTP de Termius`](docs/07-conexion-master-workers.md#opción-a--transferir-con-el-panel-sftp-de-termius)
   - [`Opción B — Transferir con scp desde Windows`](docs/07-conexion-master-workers.md#opción-b--transferir-con-scp-desde-windows)
   - [`Opción C — Transferir con scp desde Linux o macOS`](docs/07-conexion-master-workers.md#opción-c--transferir-con-scp-desde-linux-o-macos)
   - [`Verificar que los archivos llegaron correctamente`](docs/07-conexion-master-workers.md#verificar-que-los-archivos-llegaron-correctamente)
4. [`Editar "~/.ssh/config" con nano`](docs/07-conexion-master-workers.md#74--editar-sshconfig-con-nano)
   - [`Contenido del archivo`](docs/07-conexion-master-workers.md#contenido-del-archivo)
   - [`Explicación línea por línea`](docs/07-conexion-master-workers.md#explicación-línea-por-línea)
5. [`Ajustar permisos del "config" y los ".pem"`](docs/07-conexion-master-workers.md#75--ajustar-permisos-del-config-y-los-pem)
   - [`¿Qué significan estos permisos?`](docs/07-conexion-master-workers.md#qué-significan-estos-permisos)
   - [`Verificar que los permisos quedaron correctos`](docs/07-conexion-master-workers.md#verificar-que-los-permisos-quedaron-correctos)
6. [`Probar la conexión desde el Master`](docs/07-conexion-master-workers.md#76--probar-la-conexión-desde-el-master-hacia-cada-worker)
   - [`Conectarse a cada Worker`](docs/07-conexion-master-workers.md#conectarse-a-worker-1)
   - [`Tabla resumen de conexiones`](docs/07-conexion-master-workers.md#tabla-resumen-de-conexiones)

---

## Sección 8 / Verificación final

1. [`Checklist del sistema completo`](docs/08-verificacion-final.md#1--checklist-del-sistema-completo)
2. [`Verificar la VPC y la red`](docs/08-verificacion-final.md#2--verificar-la-vpc-y-la-red)
3. [`Verificar el acceso al Master`](docs/08-verificacion-final.md#3--verificar-el-acceso-al-master)
4. [`Verificar el acceso a los Workers desde el Master`](docs/08-verificacion-final.md#4--verificar-el-acceso-a-los-workers-desde-el-master)
5. [`Verificar que los Workers no tienen salida a internet`](docs/08-verificacion-final.md#5--verificar-que-los-workers-no-tienen-salida-a-internet)