---
title: Azure Defender для Resource Manager — преимущества и возможности
description: Сведения о преимуществах и функциях Azure Defender для Azure Resource Manager
author: memildin
ms.author: memildin
ms.date: 12/07/2020
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 903853f9f37921a68c918d09a94087548b5c562c
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102100667"
---
# <a name="introduction-to-azure-defender-for-resource-manager"></a>Общие сведения об Azure Defender для Resource Manager

[Azure Resource Manager](../azure-resource-manager/management/overview.md) — это служба развертывания и управления для Azure. Она обеспечивает уровень управления для создания, обновления и удаления ресурсов в учетной записи Azure. Вы можете использовать ее функции управления, такие как управление доступом, блокировка и добавление тегов, чтобы защитить и упорядочить ресурсы после развертывания.

Уровень управления облаком — это важная служба, подключенная ко всем облачным ресурсам. По этой причине он также является потенциальной целью для злоумышленников. Следовательно, мы рекомендуем группам по обеспечению безопасности внимательно отслеживать уровень управления ресурсами. 

Azure Defender для Resource Manager автоматически отслеживает операции управления ресурсами в вашей организации, независимо от того, выполняются ли они с помощью портала Azure, Azure REST API, Azure CLI или других программных клиентов Azure. Azure Defender запускает расширенную аналитику безопасности для обнаружения угроз и предупреждает о подозрительных действиях.

## <a name="availability"></a>Доступность

|Аспект|Сведения|
|----|:----|
|Состояние выпуска:|Предварительный просмотр<br>[!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)] |
|Цены:|Плата за использование **Azure Defender для Resource Manager** начисляется по тарифам, приведенным на странице [Цены на Центр безопасности](https://azure.microsoft.com/pricing/details/security-center/).|
|Облако.|![Да](./media/icons/yes-icon.png) Коммерческие облака<br>![Нет](./media/icons/no-icon.png) Национальные и независимые (US Gov, China Gov, другие правительственные облака)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-resource-manager"></a>Каковы преимущества Azure Defender для Resource Manager?

Azure Defender для Resource Manager защищает от проблем, в том числе нижеперечисленных.

- **Подозрительные операции управления ресурсами**, такие как операции с подозрительными IP-адресами, отключение защиты от вредоносных программ и подозрительные сценарии, выполняемые в расширениях ВМ.
- **Использование наборов средств для эксплуатации**, таких как Microburst или PowerZure
- **Перемещение** с уровня управления Azure в плоскость данных ресурсов Azure

:::image type="content" source="media/defender-for-resource-manager-introduction/consistent-management-layer-with-defender.png" alt-text="Структурная схема Azure Resource Manager":::

Полный список оповещений, предоставляемых Azure Defender для Resource Manager, находится на [странице справки по оповещениям](alerts-reference.md#alerts-resourcemanager).


 ## <a name="how-to-investigate-alerts-from-azure-defender-for-resource-manager"></a>Как изучать оповещения Azure Defender для Resource Manager

Оповещения системы безопасности Azure Defender для Resource Manager основаны на угрозах, обнаруженных при мониторинге операций Azure Resource Manager. В Azure Defender используются внутренние источники журналов Azure Resource Manager, а также журнал действий Azure, журнал платформы в Azure, который предоставляет сведения о событиях на уровне подписки.

Дополнительные сведения о [журналах действий Azure](../azure-monitor/essentials/activity-log.md).

Порядок изучения оповещений системы безопасности Azure Defender для Resource Manager.

1. Откройте журнал действий Azure

    :::image type="content" source="media/defender-for-resource-manager-introduction/opening-azure-activity-log.png" alt-text="Как открыть журнал действий Azure":::

1. Отфильтруйте события по доступным параметрам.
    - Подписке, упомянутой в предупреждении
    - Интервалу времени обнаружения действия
    - Связанной учетной записи пользователя (если есть)

1. Найдите подозрительные действия.

> [!TIP]
> Чтобы получить более широкие возможности исследования оповещений, передайте журналы действий Azure в Azure Sentinel, как описано в разделе [Подключение данных из журнала действий Azure](../sentinel/connect-azure-activity.md).



## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали об Azure Defender для Resource Manager. Связанные материалы см. в следующих статьях. 

- Оповещения системы безопасности могут создаваться самим центром безопасности или приниматься центром безопасности от различных продуктов по обеспечению безопасности. Чтобы экспортировать все эти оповещения в Azure Sentinel, любое стороннее средство SIEM или другие внешние инструменты, следуйте инструкциям в статье [Экспорт оповещений о безопасности и рекомендаций](continuous-export.md).

- > [!div class="nextstepaction"]
    > [Включение Azure Defender](enable-azure-defender.md)