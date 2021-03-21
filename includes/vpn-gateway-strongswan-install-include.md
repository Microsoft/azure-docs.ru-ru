---
title: включить файл
description: включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 08/14/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 089ad704a466590a9649107fef245a88d6a4d6e3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102244948"
---
Для приведенных ниже действий использовалась следующая конфигурация.

- Компьютер: Ubuntu Server 18,04
- Зависимости: strongSwan


Чтобы установить необходимую конфигурацию strongSwan, используйте следующие команды:

```
sudo apt install strongswan
```

```
sudo apt install strongswan-pki
```

```
sudo apt install libstrongswan-extra-plugins
```

Чтобы установить интерфейс командной строки Azure, используйте следующую команду:

```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

[Дополнительные инструкции по установке Azure CLI](/cli/azure/install-azure-cli-apt)