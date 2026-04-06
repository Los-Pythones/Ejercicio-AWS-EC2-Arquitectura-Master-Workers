# Ejercicio AWS EC2 Arquitectura Master / Workers

### Riwi · Cohorte 6 · Analitica de Datos

---
 
> **¿Para quién es este repo?**  
> Para todos los que no entendieron muy bien qué hicimos, se perdieron alguna clase o simplemente quieren tener todo documentado en un solo lugar (Yo).
 
---

## Sección 0 / Introduccción
1. [`Contexto de la actividad`](docs/00-introducción.md)    
2. [`Diagrama de los equipos con sus roles`](docs/00-introducción.md)    

---

## Sección 1 / Conceptos claves

1. [`¿Qué es AWS?`](docs/1-conceptos-previos.md#1-qué-es-aws)
2. [`¿Qué es IAM?`](docs/1-conceptos-previos.md#2-qué-es-iam)
3. [`¿Qué es EC2?`](docs/1-conceptos-previos.md#3-qué-es-ec2)
4. [`¿Qué es una Key Pair?`](docs/1-conceptos-previos.md#4-qué-es-una-key-pair)
5. [`¿Qué es SSH y cómo funciona el sistema de llaves?`](docs/1-conceptos-previos.md#5-qué-es-ssh-y-cómo-funciona-el-sistema-de-llaves)
6. [`¿Qué es un Security Group?`](docs/1-conceptos-previos.md#6-qué-es-un-security-group)

---

## Sección 2 / Lo que hace el Líder

1. [`¿Por qué no usar la cuenta root para todo?`](docs/2-creacion-usuarios-iam.md#por-qué-no-trabajar-directamente-con-la-cuenta-root)
2. [`Ingresar a IAM desde la consola de AWS`](docs/2-creacion-usuarios-iam.md#paso-1--ingresar-a-iam-desde-la-consola-de-aws)
3. [`Crear los grupos con sus políticas`](docs/2-creacion-usuarios-iam.md#paso-2--crear-los-grupos-con-sus-políticas)
4. [`Crear los usuarios IAM`](docs/2-creacion-usuarios-iam.md#paso-3--crear-los-usuarios-iam)
5. [`Obtener las credenciales de cada usuario`](docs/2-creacion-usuarios-iam.md#paso-4--obtener-las-credenciales-de-cada-usuario)
6. [`Enviar las credenciales a cada integrante`](docs/2-creacion-usuarios-iam.md#paso-5--enviar-las-credenciales-a-cada-integrante)

---

## Sección 3 / Ingreso a la consola de AWS con usuario IAM

1. [`Acceder a la URL de inicio de sesión`](docs/3-ingreso-ala-consola-AWS.md#paso-1--acceder-a-la-url-de-inicio-de-sesión)
2. [`Iniciar sesión`](docs/3-ingreso-ala-consola-AWS.md#paso-2--iniciar-sesión)
3. [`Cambiar la contraseña en el primer ingreso`](docs/3-ingreso-ala-consola-AWS.md#03--ingreso-a-la-consola-de-aws-con-usuario-iam)
4. [`Reconocer los permisos del rol asignado`](docs/3-ingreso-ala-consola-AWS.md#paso-4--reconocer-los-permisos-del-rol-asignado)
5. [`Verificar la región de trabajo`](docs/3-ingreso-ala-consola-AWS.md#paso-5--verificar-la-región-de-trabajo)
