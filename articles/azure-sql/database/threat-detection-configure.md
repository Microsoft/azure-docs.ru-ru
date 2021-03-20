---
title: Настройка Расширенной защиты от угроз
description: Расширенная защита от угроз обнаруживает аномальные действия базы данных, указывающие потенциальные угрозы безопасности для базы данных в базе данных SQL Azure.
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
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96453964"
---
# <a name="configure-advanced-threat-protection-for-azure-sql-database"></a>Настройка расширенной защиты от угроз для базы данных SQL Azure
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[Расширенная защита от угроз](threat-detection-overview.md) для базы данных SQL Azure обнаруживает аномальные действия, указывающие на необычные и потенциально опасные попытки доступа к базам данных или их использования. Расширенная защита от угроз позволяет определить **потенциальное внедрение кода SQL**, **получить доступ из необычного расположения или центра обработки данных**, **получить доступ из незнакомого основного или потенциально опасного приложения** и **выполнить подбор учетных данных SQL** [. Дополнительные](threat-detection-overview.md#alerts)сведения см.

Вы можете получать уведомления об обнаруженных угрозах с помощью [уведомлений по электронной почте](threat-detection-overview.md#explore-detection-of-a-suspicious-event) или [портал Azure](threat-detection-overview.md#explore-alerts-in-the-azure-portal)

[Расширенная защита от угроз](threat-detection-overview.md) является частью предложения ["защитник Azure для SQL"](azure-defender-for-sql.md) , которое является единым пакетом для расширенных возможностей обеспечения безопасности SQL. Вы можете получить доступ к расширенной защите от угроз и управлять ими с помощью центрального портала Azure Defender для SQL.

## <a name="set-up-advanced-threat-protection-in-the-azure-portal"></a>Настройка расширенной защиты от угроз в портал Azure

1. Войдите на [портал Azure](https://portal.azure.com).
2. Перейдите на страницу конфигурации сервера, который требуется защитить. В окне Параметры безопасности выберите **Центр безопасности**.
3. На странице конфигурации **защитника Azure для SQL** :

   - Включите **защитник Azure для SQL** на сервере.
   - В области **Дополнительные параметры защиты от угроз** укажите список сообщений электронной почты для получения оповещений системы безопасности при обнаружении аномальных действий базы данных в текстовом поле **Отправить оповещения в** .
   
   :::image type="content" source="media/azure-defender-for-sql/set-up-advanced-threat-protection.png" alt-text="Настройка расширенной защиты от угроз":::

## <a name="set-up-advanced-threat-protection-using-powershell"></a>Настройка Расширенной защиты от угроз с использованием PowerShell

Пример сценария см. в статье [Настройка аудита и расширенная защита от угроз с помощью PowerShell](scripts/auditing-threat-detection-powershell-configure.md).

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [Advanced Threat protection](threat-detection-overview.md).
- Дополнительные сведения о [расширенной защите от угроз в SQL управляемый экземпляр](../managed-instance/threat-detection-configure.md).  
- [Узнайте больше про Azure Defender для SQL](azure-defender-for-sql.md).
- Дополнительные сведения об [аудите](../../azure-sql/database/auditing-overview.md).
- Дополнительные сведения о [центре безопасности Azure](../../security-center/security-center-introduction.md)
- Дополнительные сведения о ценах см. на [странице цен на базу данных SQL](https://azure.microsoft.com/pricing/details/sql-database/) .