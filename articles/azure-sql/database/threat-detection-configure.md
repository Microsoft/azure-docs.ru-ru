---
title: Настройка Расширенной защиты от угроз
description: Расширенная защита от угроз обнаруживает подозрительные действия в Базе данных Azure SQL, указывающие на наличие потенциальных угроз безопасности
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-dt-2019, sqldbrb=1
ms.topic: how-to
author: rmatchoro
ms.author: ronmat
ms.reviewer: vanto
ms.date: 12/01/2020
ms.openlocfilehash: 1425003c718ca52c0bea712e9d25cd3e4c035cf1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96453964"
---
# <a name="configure-advanced-threat-protection-for-azure-sql-database"></a>Настройка Расширенной защиты от угроз для Базы данных SQL Azure
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[Расширенная защита от угроз](threat-detection-overview.md) для Базы данных SQL Azure позволяет выявлять подозрительную активность, указывающую на необычные и потенциально опасные попытки получить доступ к базам данных или воспользоваться ими. Эта служба может определить **потенциальную атаку путем внедрения кода SQL**, **доступ из необычного центра обработки данных или расположения**, **доступ из незнакомого субъекта или потенциально опасного приложения**, а также **подбор учетных данных SQL** (подробнее см. в статье [Оповещениях Расширенной защиты от угроз](threat-detection-overview.md#alerts)).

Вы можете получать уведомления об обнаруженных угрозах с помощью [уведомлений по электронной почте](threat-detection-overview.md#explore-detection-of-a-suspicious-event) или [портала Azure](threat-detection-overview.md#explore-alerts-in-the-azure-portal)

[Расширенная защита от угроз](threat-detection-overview.md) входит в состав предложения [Azure Defender для SQL](azure-defender-for-sql.md), которое представляет собой единый пакет расширенных средств для обеспечения безопасности SQL. Доступ к Расширенной защите от угроз можно получить через центральный портал Azure Defender для SQL. Там же ею можно управлять.

## <a name="set-up-advanced-threat-protection-in-the-azure-portal"></a>Настройка Расширенной защиты от угроз на портале Azure

1. Войдите на [портал Azure](https://portal.azure.com).
2. Перейдите на страницу настройки сервера, который требуется защитить. В параметрах безопасности выберите **Центр безопасности**.
3. На странице конфигурации **Azure Defender для SQL**:

   - Включите **Azure Defender для SQL** на сервере.
   - В **параметрах Расширенной защиты от угроз** в текстовом поле **Кому отправлять оповещения** предоставьте список адресов электронной почты, на которые будут отправляться оповещения системы безопасности об обнаружении аномальных действий в базе данных.
   
   :::image type="content" source="media/azure-defender-for-sql/set-up-advanced-threat-protection.png" alt-text="Настройка Расширенной защиты от угроз":::

## <a name="set-up-advanced-threat-protection-using-powershell"></a>Настройка Расширенной защиты от угроз с использованием PowerShell

Пример сценария см. в статье [Настройка аудита и Расширенной защиты от угроз с помощью PowerShell](scripts/auditing-threat-detection-powershell-configure.md).

## <a name="next-steps"></a>Дальнейшие действия

- Подробнее о [Расширенной защите от угроз](threat-detection-overview.md).
- Подробнее о [Расширенной защиты от угроз в Управляемом экземпляре SQL](../managed-instance/threat-detection-configure.md).  
- [Узнайте больше про Azure Defender для SQL](azure-defender-for-sql.md).
- Дополнительные сведения об [аудите](../../azure-sql/database/auditing-overview.md).
- Дополнительные сведения о [Центре безопасности Azure](../../security-center/security-center-introduction.md).
- Дополнительные сведения о [ценах на Базу данных SQL](https://azure.microsoft.com/pricing/details/sql-database/).