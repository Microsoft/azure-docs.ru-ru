---
title: Расширенная защита от угроз для Azure Cosmos DB
description: Узнайте, как Azure Cosmos DB обеспечивает шифрование неактивных данных и как его реализовать.
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/13/2019
ms.custom: seodec18
ms.author: memildin
author: memildin
manager: rkarlin
ms.openlocfilehash: b73e7f8c13f621bc359a2ae79a725829420a3ecc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102455620"
---
# <a name="advanced-threat-protection-for-azure-cosmos-db-preview"></a>Расширенная защита от угроз для Azure Cosmos DB (предварительная версия)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Расширенная защита от угроз для Azure Cosmos DB обеспечивает дополнительный уровень безопасности, обнаруживая необычные и потенциально опасные попытки доступа или использования ваших учетных записей Azure Cosmos DB. Этот уровень защиты позволяет устранять угрозы без необходимости в экспертных знаниях по безопасности или в интеграции с централизованными системами мониторинга безопасности.

Оповещения системы безопасности активируются при возникновении аномальных действий. Эти оповещения системы безопасности интегрируются с [Центром безопасности Azure](https://azure.microsoft.com/services/security-center/), а также отправляются по электронной почте администраторам подписки. Они содержат сведения о подозрительных операциях и рекомендации о том, как определить причину угроз и устранить их.

> [!NOTE]
>
> * Расширенная защита от угроз для Azure Cosmos DB в настоящее время доступна только для интерфейса API SQL.
> * Расширенная защита от угроз для Azure Cosmos DB в настоящее время недоступна в регионах Azure для национальных облаков и облаков для государственных организаций.

Для полного исследования оповещений системы безопасности рекомендуется включить [ведение журнала диагностики в Azure Cosmos DB](./monitor-cosmos-db.md), в котором регистрируются операции с самой базой данных, включая операции CRUD со всеми документами, контейнерами и базами данных.

## <a name="threat-types"></a>Типы угроз

Расширенная защита от угроз для Azure Cosmos DB позволяет выявлять подозрительную активность, указывающую на необычные и потенциально опасные попытки получить доступ к базам данных или воспользоваться ими. В настоящее время она может активировать перечисленные ниже оповещения.

- **Доступ из незнакомых расположений**. Это оповещение появляется при изменении способа доступа к учетной записи Azure Cosmos, если пользователь подключился к конечной точке Azure Cosmos DB из необычного географического расположения. Иногда оповещение определяет допустимые действия (новое приложение или операцию обслуживания разработчиком). В других случаях оповещение указывает на угрозу со стороны бывшего сотрудника, внешнего злоумышленника и т. д.

- **Необычное извлечение данных**. Это оповещение активируется, когда клиент извлекает необычный объем данных из учетной записи Azure Cosmos DB. Это может быть симптомом извлечения данных с целью передачи всех данных, хранящихся в учетной записи, во внешнее хранилище данных.



## <a name="configure-advanced-threat-protection"></a>Настройка Расширенной защиты от угроз

Расширенную защиту от угроз можно настроить любым из нескольких способов, описанных в следующих разделах.

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Запустите портал Azure: [https://portal.azure.com](https://portal.azure.com/).

2. В учетной записи Azure Cosmos DB в меню **Параметры** выберите пункт **Повышенная безопасность**.

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-atp.png" alt-text="Настройка Расширенной защиты от угроз":::

3. В колонке настройки **Повышенная безопасность** выполните указанные ниже действия.

    * Выберите пункт **Расширенная защита от угроз**, чтобы задать для него значение **Вкл**.
    * Щелкните **Сохранить**, чтобы сохранить новую или обновленную политику Расширенной защиты от угроз.   

### <a name="rest-api"></a>[REST API](#tab/rest-api)

Используйте команды интерфейса REST API для создания, изменения или получения параметра "Расширенная защита от угроз" для определенной учетной записи Azure Cosmos DB.

* [Расширенная защита от угроз — создание](/rest/api/securitycenter/advancedthreatprotection/create)
* [Расширенная защита от угроз — получение](/rest/api/securitycenter/advancedthreatprotection/get)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Используйте следующие командлеты PowerShell:

* [Включение Расширенной защиты от угроз](/powershell/module/az.security/enable-azsecurityadvancedthreatprotection)
* [Получение Расширенной защиты от угроз](/powershell/module/az.security/get-azsecurityadvancedthreatprotection)
* [Отключение Расширенной защиты от угроз](/powershell/module/az.security/disable-azsecurityadvancedthreatprotection)

### <a name="arm-template"></a>[Шаблон ARM](#tab/arm-template)

Используйте шаблон Azure Resource Manager (ARM), чтобы настроить Cosmos DB с включенной Расширенной защитой от угроз.
Дополнительные сведения см. в статье [Создание учетной записи CosmosDB с Расширенной защитой от угроз](https://azure.microsoft.com/resources/templates/201-cosmosdb-advanced-threat-protection-create-account/).

### <a name="azure-policy"></a>[Политика Azure](#tab/azure-policy)

Используйте Политику Azure, чтобы включить Расширенную защиту от угроз для Cosmos DB.

1. В Azure откройте страницу **Политика — определения** и выполните поиск политики по запросу **Развертывание Расширенной защиты от угроз для Cosmos DB**.

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db.png" alt-text="Поиск политики"::: 

1. Выберите политику **Развертывание Расширенной защиты от угроз для CosmosDB** и щелкните **Назначить**.

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-atp-policy.png" alt-text="Выбор подписки или группы":::


1. В поле **Область** щелкните многоточие, выберите подписку Azure или группу ресурсов, а затем щелкните **Выбрать**.

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-atp-details.png" alt-text="Страница определений политик":::


1. Введите остальные параметры и нажмите кнопку **Назначить**.




## <a name="manage-atp-security-alerts"></a>Управление оповещениями системы безопасности ATP

При возникновении аномальных операций с Azure Cosmos DB активируется оповещение системы безопасности с информацией о подозрительном событии безопасности. 

 В Центре безопасности Azure можно просматривать текущие [оповещения системы безопасности ](../security-center/security-center-alerts-overview.md) и управлять ими.  Щелкните одно из оповещений в [Центре безопасности](https://ms.portal.azure.com/#blade/Microsoft_Azure_Security/SecurityMenuBlade/0), чтобы просмотреть возможные причины и рекомендуемые действия по поиску и устранению потенциальной угрозы. На приведенном ниже рисунке показан пример сведений оповещения в Центре безопасности.

 :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-alert-details.png" alt-text="Сведения об угрозе":::

Кроме того, по электронной почте отправляется уведомление с подробными сведениями об оповещении и рекомендуемыми действиями. На рисунке ниже показан пример сообщения электронной почты с оповещением.

 :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-alert.png" alt-text="Сведения об оповещении":::

## <a name="cosmos-db-atp-alerts"></a>Оповещения ATP для Cosmos DB

 Список оповещений, создаваемых при наблюдении за учетными записями Azure Cosmos DB, см. в разделе [Оповещения Cosmos DB](../security-center/alerts-reference.md#alerts-azurecosmos) в документации по Центру безопасности Azure.

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте больше о [журнале ведения диагностики в Azure Cosmos DB](cosmosdb-monitor-resource-logs.md).
* Узнайте больше о [центре безопасности Azure](../security-center/security-center-introduction.md).