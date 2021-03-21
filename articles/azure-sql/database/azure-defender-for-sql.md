---
title: Azure Defender для SQL
description: Узнайте о функциях для управления уязвимостями базы данных и обнаружения аномальных действий, которые могут указывать на угрозу для базы данных в базе данных SQL Azure, Управляемый экземпляр Azure SQL или Azure синапсе.
ms.service: sql-db-mi
ms.subservice: security
ms.devlang: ''
ms.custom: sqldbrb=2
ms.topic: conceptual
ms.author: memildin
manager: rkarlin
author: memildin
ms.date: 03/08/2021
ms.openlocfilehash: 27f17b3d1060e8b693c2a34cdb4643840f1bfd13
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102452323"
---
# <a name="azure-defender-for-sql"></a>Azure Defender для SQL

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Azure Defender для SQL — это унифицированный пакет расширенных функций защиты SQL, Служба "защитник Azure" доступна для базы данных SQL Azure, Azure SQL Управляемый экземпляр и Azure синапсе Analytics. Он включает в себя функции для обнаружения и классификации конфиденциальных данных, обнаружения и устранения потенциальных уязвимостей базы данных и обнаружения аномальных действий, которые могут указывать на угрозу для вашей базы данных. Эта служба предоставляет единый центр для включения этих возможностей и управления ими.

## <a name="what-are-the-benefits-of-azure-defender-for-sql"></a>Каковы преимущества Azure Defender для SQL?

Защитник Azure предоставляет набор расширенных возможностей безопасности SQL, включая оценку уязвимостей SQL и расширенную защиту от угроз.
- [Оценка уязвимостей](sql-vulnerability-assessment.md) — это простая в настройке служба, которая может обнаруживать, записывать и помогать устранять потенциальные уязвимости базы данных. Он предоставляет сведения о состоянии безопасности и содержит действия по устранению проблем безопасности и улучшению фортификатионс базы данных.
- [Расширенная защита от угроз](threat-detection-overview.md) позволяет выявить подозрительную активность, указывающую на необычные и потенциально опасные попытки получить доступ к базам данных или воспользоваться ими. Она постоянно отслеживает подозрительные действия в базе данных и обеспечивает немедленное оповещение системы безопасности о потенциальных уязвимостях, атаках путем внедрения кода SQL Azure и аномальных шаблонах доступа к базам данных. Оповещения Расширенной защиты от угроз содержат сведения о подозрительных операциях и рекомендации о том, как исследовать причину угрозы и устранить ее.

Включите Azure Defender для SQL один раз, чтобы активировать все эти встроенные функции. Одним щелчком можно включить защитник Azure для всех баз данных на [сервере](logical-servers.md) в Azure или в управляемый экземпляр SQL. Для включения или управления параметрами защитника Azure требуется принадлежность к роли [диспетчера безопасности SQL](../../role-based-access-control/built-in-roles.md#sql-security-manager) или одной из ролей администратора базы данных или сервера.

Дополнительные сведения о ценах на службу "защитник Azure для SQL" см. на [странице цен на центр безопасности Azure](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="enable-azure-defender"></a>Включение Azure Defender 
Существует несколько способов включения планов защитника Azure. Его можно включить на уровне подписки (**рекомендуется**):

- [Центр безопасности Azure](#enable-azure-defender-for-azure-sql-database-at-the-subscription-level-from-azure-security-center)
- [Программное REST API, Azure CLI, PowerShell или политика Azure](#enable-azure-defender-plans-programatically)

Кроме того, его можно включить на уровне ресурсов, как описано в разделе [Включение защитника Azure для базы данных SQL Azure на уровне ресурсов](#enable-azure-defender-for-azure-sql-database-at-the-resource-level) .

### <a name="enable-azure-defender-for-azure-sql-database-at-the-subscription-level-from-azure-security-center"></a>Включение защитника Azure для базы данных SQL Azure на уровне подписки из центра безопасности Azure
Чтобы включить защитник Azure для базы данных SQL Azure на уровне подписки в центре безопасности Azure, сделайте следующее:

1. На [портале Azure](https://portal.azure.com) откройте **Центр безопасности**.
1. В меню центра безопасности выберите пункт **цены и настройки**.
1. Выберите соответствующую подписку.
1. Измените значение параметра план на **вкл**.

    :::image type="content" source="media/azure-defender-for-sql/enable-azure-defender-sql-subscription-level.png" alt-text="Включение защитника Azure для базы данных SQL Azure на уровне подписки.":::

1. Щелкните **Сохранить**.


### <a name="enable-azure-defender-plans-programatically"></a>Программное включение планов защитника Azure 

Гибкость Azure позволяет использовать несколько программных методов для включения планов защитника Azure. 

Используйте любое из следующих средств, чтобы включить защитник Azure для вашей подписки: 

| Метод       | Инструкции                                                                                                                                       |
|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| REST API     | [API цен](/rest/api/securitycenter/pricings)                                                                                                  |
| Azure CLI    | [az security pricing](/cli/azure/security/pricing)                                                                                                 |
| PowerShell   | [Set-AzSecurityPricing](/powershell/module/az.security/set-azsecuritypricing)                                                                      |
| Политика Azure | [Цены на пакеты](https://github.com/Azure/Azure-Security-Center/blob/master/Pricing%20%26%20Settings/ARM%20Templates/Set-ASC-Bundle-Pricing.json) |
|              |                                                                                                                                                    |

### <a name="enable-azure-defender-for-azure-sql-database-at-the-resource-level"></a>Включение защитника Azure для базы данных SQL Azure на уровне ресурсов

Мы рекомендуем включить планы защитника Azure на уровне подписки, и это может помочь в создании незащищенных ресурсов. Однако если у вас есть причина включения защитника Azure на уровне сервера, выполните следующие действия.

1. В [портал Azure](https://portal.azure.com)откройте сервер или управляемый экземпляр.
1. Под заголовком **Безопасность** выберите **Центр безопасности**.
1. Выберите **включить защитник Azure для SQL**.

    :::image type="content" source="media/azure-defender-for-sql/enable-azure-defender.png" alt-text="Включение защитника Azure для SQL в базах данных SQL Azure.":::

> [!NOTE]
> Учетная запись хранения автоматически создается и настраивается для хранения результатов проверки **уязвимостей** . Если вы уже включили защитник Azure для другого сервера в той же группе ресурсов и регионе, используется существующая учетная запись хранения.
>
> Стоимость защитника Azure соответствует ценам на стандартный уровень центра безопасности Azure на каждый узел, где узел является полностью сервером или управляемым экземпляром. Поэтому вы платите только один раз для защиты всех баз данных на сервере или управляемом экземпляре с помощью защитника Azure. Вы можете опробовать Azure Defender изначально с помощью бесплатной пробной версии.


## <a name="manage-azure-defender-settings"></a>Управление параметрами защитника Azure

Для просмотра параметров защитника Azure и управления ими выполните следующие действия.

1. В области **безопасности** сервера или управляемого экземпляра выберите **Центр безопасности**.

    На этой странице вы увидите состояние защитника Azure для SQL:

    :::image type="content" source="media/azure-defender-for-sql/status-of-defender-for-sql.png" alt-text="Проверка состояния защитника Azure для SQL в базах данных SQL Azure.":::

1. Если включен Защитник Azure для SQL, вы увидите ссылку **настроить** , как показано на предыдущем рисунке. Чтобы изменить параметры для защитника Azure для SQL, выберите **настроить**.

    :::image type="content" source="media/azure-defender-for-sql/security-server-settings.png" alt-text="Параметры для защитника Azure для SQL.":::

1. Внесите необходимые изменения и нажмите кнопку **сохранить**.


## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об [оценке уязвимостей](sql-vulnerability-assessment.md)
- Дополнительные сведения о [расширенной защите от угроз](threat-detection-configure.md)
- Узнайте больше о [центре безопасности Azure](../../security-center/security-center-introduction.md).