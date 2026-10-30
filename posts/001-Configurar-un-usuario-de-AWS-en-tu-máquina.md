---
title: Configurar un usuario de AWS en tu máquina
date: 2026-10-29
tags:
  - aws
  - aws-cli
  - credenciales
description: Configura las credenciales de un usuario de AWS en tu máquina con AWS CLI.
---

# Configurar un usuario de AWS en tu máquina

Para usar un usuario de AWS desde tu máquina necesitas el AWS CLI instalado. La instalación está en la documentación oficial: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Con el CLI instalado, asocias las claves de cualquier usuario de IAM:

```bash
aws configure
```

La terminal pide cuatro datos:

- **AWS Access Key ID:** la clave de acceso de ese usuario.
- **AWS Secret Access Key:** la clave secreta de ese usuario.
- **Default region name:** la región donde vas a trabajar, por ejemplo `us-east-2`.
- **Default output format:** `json`.

Esas claves salen de **IAM**, en el usuario, **Security credentials**, **Create access key**. Solo se muestran una vez.

Comprueba qué usuario quedó configurado:

```bash
aws sts get-caller-identity
```

La guía de `aws configure` está en https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html
