---
title: Políticas y usuarios de IAM para Parameter Store
date: 2026-10-29
tags:
  - aws
  - iam
  - parameter-store
  - permisos
---

# Políticas y usuarios de IAM para Parameter Store

Esto configura un usuario para que un proyecto de software pueda usar Parameter Store. Ese usuario crea, lee y borra parámetros solo en las rutas que le permitas. No hace falta usar la cuenta root.

Una política dice qué acciones puede hacer y sobre qué parámetros. El usuario es la identidad del proyecto en tu máquina. La política se le asigna a ese usuario.

## Crear la política

1. En la consola de AWS, abre **IAM**.
2. En el menú izquierdo, entra a **Policies** y pulsa **Create policy**.
3. Abre la pestaña **JSON** y pega este permiso. Cambia la región, el ID de tu cuenta, la ruta del servicio y el ambiente:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:PutParameter",
        "ssm:GetParameter",
        "ssm:GetParameters",
        "ssm:GetParametersByPath",
        "ssm:DeleteParameter"
      ],
      "Resource": [
        "arn:aws:ssm:[REGION]:[ID-CUENTA-AWS]:parameter/[SERVICIO]/[AMBIENTE]",
        "arn:aws:ssm:[REGION]:[ID-CUENTA-AWS]:parameter/[SERVICIO]/[AMBIENTE]/*"
      ]
    }
  ]
}
```

El ID de la cuenta suele estar en la esquina superior izquierda de la consola.

La primera ruta es el ambiente. La segunda, con `/*`, cubre los parámetros que están dentro de ese ambiente.

Ejemplo, con otra aplicación:

```txt
arn:aws:ssm:us-east-2:123456789012:parameter/mi-app/dev
arn:aws:ssm:us-east-2:123456789012:parameter/mi-app/dev/*
```

4. Pulsa **Next**, ponle un nombre y termina con **Create policy**.

Qué permite cada acción:

- `ssm:PutParameter` crea o actualiza un parámetro.
- `ssm:GetParameter` lee un parámetro.
- `ssm:GetParameters` lee varios en una sola petición.
- `ssm:GetParametersByPath` lee los parámetros de una ruta.
- `ssm:DeleteParameter` elimina un parámetro.

`Resource` limita esas acciones a las rutas que escribiste. Otra ruta de la cuenta queda fuera.

## Crear el usuario

El usuario es quien usa esa política desde tu máquina, para que el proyecto pueda llamar a Parameter Store.

1. En **IAM**, entra a **Users** y pulsa **Create user**.
2. Ponle un nombre.
3. Elige **Attach policies directly** y marca la política que acabas de crear.
4. Crea el usuario.
5. Ábrelo, entra a **Security credentials** y en **Access keys** pulsa **Create access key**. Elige el uso para la CLI.
6. Guarda `AWS_ACCESS_KEY_ID` y `AWS_SECRET_ACCESS_KEY`. Solo se muestran una vez.
