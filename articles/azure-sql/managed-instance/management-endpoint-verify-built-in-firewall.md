---
title: Проверка безопасности портов во встроенном брандмауэре
description: Узнайте, как проверить встроенную защиту брандмауэра в Azure SQL Управляемый экземпляр.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein
ms.date: 12/04/2018
ms.openlocfilehash: d8d46518415dc56e2792ef090b0be654d538426f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96853277"
---
# <a name="verify-the-azure-sql-managed-instance-built-in-firewall"></a>Проверка встроенного брандмауэра Управляемого экземпляра SQL Azure
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Для использования [обязательных правил безопасности для входящего трафика](connectivity-architecture-overview.md#mandatory-inbound-security-rules-with-service-aided-subnet-configuration) Azure SQL управляемый экземпляр требуется, чтобы порты управления 9000, 9003, 1438, 1440 и 1452 были открыты из **любого источника** в группе безопасности сети (NSG), защищающей SQL управляемый экземпляр. Несмотря на то, что эти порты открыты на уровне группы безопасности сети, они защищены на уровне сети встроенным брандмауэром.

## <a name="verify-firewall"></a>Проверка брандмауэра

Чтобы проверить эти порты, используйте любое средство сканирования системы безопасности для проверки этих портов. На следующем снимке экрана показано, как использовать одно из этих средств.

![Проверка встроенного брандмауэра](./media/management-endpoint-verify-built-in-firewall/03_verify_firewall.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Управляемый экземпляр SQL и подключении см. в статье [Архитектура подключения управляемый экземпляр SQL Azure](connectivity-architecture-overview.md).
