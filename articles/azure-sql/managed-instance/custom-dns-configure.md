---
title: Пользовательский DNS
titleSuffix: Azure SQL Managed Instance
description: В этом разделе описываются параметры конфигурации для пользовательского DNS с Управляемый экземпляр Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 07/17/2019
ms.openlocfilehash: a54907dd3f7b3fbc06033624f14b12de14d9afb9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99831507"
---
# <a name="configure-a-custom-dns-for-azure-sql-managed-instance"></a>Настройка пользовательской службы DNS для Управляемого экземпляра SQL Azure
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Управляемый экземпляр Azure SQL необходимо развернуть в [виртуальной сети](../../virtual-network/virtual-networks-overview.md)Azure. Существует несколько сценариев (например, Database Mail, связанные серверы с другими экземплярами SQL Server в облачной или гибридной среде), для которых требуется, чтобы имена частных узлов разрешались из Управляемого экземпляра SQL. В таком случае в Azure необходимо настроить пользовательскую службу DNS. 

Так как SQL Управляемый экземпляр использует одну и ту же службу DNS для своей внутренней работы, настройте пользовательский DNS-сервер, чтобы он мог разрешать общедоступные доменные имена.

> [!IMPORTANT]
> Всегда используйте полное доменное имя (FQDN) для почтового сервера, для экземпляра SQL Server и для других служб, даже если они находятся в частной зоне DNS. Например, используйте `smtp.contoso.com` для почтового сервера, так как `smtp` не будет правильно разрешаться. Для создания связанного сервера или репликации, которая ссылается на SQL Server виртуальных машин в одной виртуальной сети, также требуется полное доменное имя и DNS-суффикс по умолчанию. Например, `SQLVM.internal.cloudapp.net`. Дополнительные сведения см. в разделе [разрешение имен, использующее собственный DNS-сервер](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

> [!IMPORTANT]
> Обновление DNS-серверов виртуальной сети не повлияет на Управляемый экземпляр SQL немедленно. Дополнительные сведения см. в статье [Синхронизация параметров DNS-серверов виртуальной сети в виртуальном кластере Управляемого экземпляра SQL](synchronize-vnet-dns-servers-setting-on-virtual-cluster.md).

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения см. в статье [что такое Azure SQL управляемый экземпляр?](sql-managed-instance-paas-overview.md).
- Руководство, в котором показано, как создать управляемый экземпляр, см. в разделе [Создание управляемого экземпляра](instance-create-quickstart.md).
- Сведения о настройке виртуальной сети для управляемого экземпляра см. в разделе [Конфигурация виртуальной сети для управляемых экземпляров](connectivity-architecture-overview.md).
