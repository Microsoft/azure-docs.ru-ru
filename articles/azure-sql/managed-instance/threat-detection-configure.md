---
title: Настройка Расширенной защиты от угроз
titleSuffix: Azure SQL Managed Instance
description: Расширенная защита от угроз обнаруживает аномальные действия базы данных, указывающие потенциальные угрозы безопасности для базы данных в Управляемый экземпляр Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: rmatchoro
ms.author: ronmat
ms.reviewer: vanto
ms.date: 12/01/2020
ms.openlocfilehash: 69bebcf872f55055117acf5cef410d1f89eafe34
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96446899"
---
# <a name="configure-advanced-threat-protection-in-azure-sql-managed-instance"></a>Настройка расширенной защиты от угроз в Azure SQL Управляемый экземпляр
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

[Расширенная защита от угроз](../database/threat-detection-overview.md) для [управляемый экземпляр Azure SQL](sql-managed-instance-paas-overview.md) обнаруживает аномальные действия, указывающие на необычные и потенциально опасные попытки доступа к базам данных или их использования. Расширенная защита от угроз позволяет определить **потенциальное внедрение кода SQL**, **получить доступ из необычного расположения или центра обработки данных**, **получить доступ из незнакомого основного или потенциально опасного приложения** и **выполнить подбор учетных данных SQL** [. Дополнительные](../database/threat-detection-overview.md#alerts)сведения см.

Вы можете получать уведомления об обнаруженных угрозах с помощью [уведомлений по электронной почте](../database/threat-detection-overview.md#explore-detection-of-a-suspicious-event) или [портал Azure](../database/threat-detection-overview.md#explore-alerts-in-the-azure-portal)

[Расширенная защита от угроз](../database/threat-detection-overview.md) является частью предложения ["защитник Azure для SQL"](../database/azure-defender-for-sql.md)  , которое является единым пакетом для расширенных возможностей обеспечения безопасности SQL. Вы можете получить доступ к расширенной защите от угроз и управлять ими с помощью центрального портала Azure Defender для SQL.

##  <a name="azure-portal"></a>Портал Azure

1. Войдите в  [портал Azure](https://portal.azure.com). 
2. Перейдите на страницу конфигурации экземпляра SQL Управляемый экземпляр, который требуется защитить. В разделе **Безопасность** выберите **Центр безопасности**.
3. На странице конфигурации защитника Azure для SQL
   - Включите **защитник** Azure для SQL.
   - Настройте **отправку оповещений** по адресу электронной почты для получения оповещений системы безопасности при обнаружении аномальных действий базы данных.
   - Выберите **учетную запись хранения Azure** для сохранения записей аудита аномальных угроз.
   - Выберите **типы расширенной защиты от угроз** , которые вы хотите настроить. Дополнительные сведения об [оповещениях Advanced Threat protection](../database/threat-detection-overview.md).
4. Нажмите кнопку **сохранить** , чтобы сохранить новую или обновленную политику Azure Defender для SQL.

   :::image type="content" source="../database/media/azure-defender-for-sql/set-up-advanced-threat-protection-mi.png" alt-text="Настройка расширенной защиты от угроз":::

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [Advanced Threat protection](../database/threat-detection-overview.md).
- Дополнительные сведения об управляемых экземплярах см. [в статье что такое управляемый экземпляр Azure SQL](sql-managed-instance-paas-overview.md).
- Дополнительные сведения о [расширенной защите от угроз для базы данных SQL Azure](../database/threat-detection-configure.md).
- Дополнительные сведения об [аудите управляемый экземпляр SQL](./auditing-configure.md).
- Дополнительные сведения о [центре безопасности Azure](../../security-center/security-center-introduction.md).