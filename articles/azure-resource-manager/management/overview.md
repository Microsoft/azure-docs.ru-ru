---
title: Общие сведения о диспетчере ресурсов Azure
description: Описывает, как использовать диспетчер ресурсов Azure для развертывания, контроля ресурсов в Azure и управления доступом к ним.
ms.topic: overview
ms.date: 03/25/2021
ms.custom: contperf-fy21q1,contperf-fy21q3-portal
ms.openlocfilehash: 6cd9aa82ad2f8a821ae82a361b3f11b72ca25f7a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105608550"
---
# <a name="what-is-azure-resource-manager"></a>Azure Resource Manager

Azure Resource Manager — это служба развертывания и управления для Azure. Она обеспечивает уровень управления для создания, обновления и удаления ресурсов в учетной записи Azure. Вы можете использовать ее функции управления, такие как управление доступом, блокировка и добавление тегов, чтобы защитить и упорядочить ресурсы после развертывания.

Дополнительные сведения о шаблонах Azure Resource Manager см. в статье с [общими сведениями о шаблонах Azure Resource Manager](../templates/overview.md).

## <a name="consistent-management-layer"></a>Уровень согласованного управления

Когда пользователь отправляет запрос из любого из средств Azure, API или пакетов SDK, он направляет к Resource Manager. Resource Manager выполняет аутентификацию и авторизацию запроса. Resource Manager отправляет запрос в службу Azure, которая принимает запрошенное действие. Так как все запросы обрабатываются через один API, результаты и возможности будут согласованы в различных средствах.

На следующем рисунке показана роль Azure Resource Manager при обработке запросов Azure.

![Модель запросов Resource Manager](./media/overview/consistent-management-layer.png)

Все доступные на портале возможности также доступны в PowerShell, Azure CLI, REST API и клиентских пакетах SDK. Функции, предоставленные через API, будут представлены на портале в течение 180 дней после выпуска.

## <a name="terminology"></a>Терминология

Если у вас еще нет опыта работы с Azure Resource Manager, возможно, некоторые термины окажутся незнакомыми.

* **Ресурс** — управляемый элемент, доступный в Azure. Виртуальные машины, учетные записи хранения, веб-приложения, базы данных и виртуальные сети являются примерами ресурсов. Примеры ресурсов: группы ресурсов, подписки, группы управления и теги.
* **Группа ресурсов** — контейнер, содержащий связанные ресурсы для решения Azure. Группа ресурсов содержит ресурсы, которыми вы хотите управлять как группой. Пользователи могут выбрать оптимальный для своей организации способ распределения ресурсов в группах ресурсов. См. раздел [Группы ресурсов](#resource-groups).
* **Поставщик ресурсов** — служба, которая предоставляет ресурсы Azure. Например, распространенный поставщик ресурсов — это `Microsoft.Compute`, который предоставляет ресурс виртуальной машины. `Microsoft.Storage` является еще одним распространенным поставщиком ресурсов. См. дополнительные сведения о [поставщиках и типах ресурсов](resource-providers-and-types.md).
* **Шаблон Resource Manager** — это файл в формате JSON (нотация объектов JavaScript), который определяет один или несколько ресурсов для развертывания в группе ресурсов, подписке, группе управления или арендаторе. Шаблон можно использовать для согласованного и многократного развертывания ресурсов. См. [общие сведения о развертывании шаблона](../templates/overview.md).
* **Декларативный синтаксис** — синтаксис, позволяющий указать объект, который вы собираетесь создать. При этом для создания объекта не нужно писать последовательность команд. Шаблон Resource Manager — пример декларативного синтаксиса. В файле можно задать свойства для инфраструктуры, развертываемой в Azure.  См. [общие сведения о развертывании шаблона](../templates/overview.md).

## <a name="the-benefits-of-using-resource-manager"></a>Преимущества использования диспетчера ресурсов

С помощью Resource Manager можно:

* Управлять своей инфраструктурой с помощью декларативных шаблонов, а не сценариев.

* Развертывать и отслеживать все ресурсы вашего решения, а также управлять ими как единой группой, а не работать с ними по отдельности.

* Повторно развертывать решение на протяжении всего цикла разработки и гарантировать, что ресурсы развертываются в согласованном состоянии.

* Определять зависимости между ресурсами, чтобы их развертывание выполнялось в правильном порядке.

* Применять управление доступом ко всем службам, так как функция управления доступом на основе ролей в Azure (Azure RBAC) изначально интегрирована в платформу управления.

* Применять теги в ресурсах для логического упорядочивания всех ресурсов в вашей подписке к ним.

* Узнать о выставлении счетов для вашей организации, просматривая затраты на группу ресурсов с одним тегом.

## <a name="understand-scope"></a>Общие сведения об области

В Azure предоставляются четыре уровня области: [группы управления](../../governance/management-groups/overview.md), подписки, [группы ресурсов](#resource-groups) и ресурсы. Пример этих уровней приведен на следующем изображении.

![Уровни управления](./media/overview/scope-levels.png)

Параметры управления можно применить на любом из этих уровней области. Выбранный уровень определяет, насколько широка область применения параметра. На более низких уровнях наследуются параметры более высоких уровней. Например, если [политика](../../governance/policy/overview.md) применяется к подписке, она применяется ко всем группам ресурсов и ресурсам в подписке. Если же политика применяется к группе ресурсов, то она применяется ко всем ресурсам в ней. Но в другой группе ресурсов этого назначения политики не будет.

Шаблоны можно развернуть в арендаторах, группах управления, подписках или группах ресурсов.

## <a name="resource-groups"></a>Группы ресурсов

Существуют некоторые важные факторы, которые необходимо учитывать при определении группы ресурсов:

* Все ресурсы в группе ресурсов должны совместно использовать один и тот же жизненный цикл. Развертывание, обновление и удаление производится сразу для всех ресурсов. Если один ресурс, например сервер, должен присутствовать в другом цикле развертывания, его следует поместить в другую группу ресурсов.

* Каждый ресурс может существовать только в одной группе ресурсов.

* Ресурс можно добавить в группу ресурсов или удалить из нее в любое время.

* Ресурс можно перемещать из одной группы ресурсов в другую. Дополнительные сведения см. в статье [Перемещение ресурсов в новую группу ресурсов или подписку](move-resource-group-and-subscription.md).

* Ресурсы в группе ресурсов могут находиться в регионах, отличных от региона группы ресурсов.

* При создании группы ресурсов необходимо указать ее расположение. У пользователя может возникнуть вопрос, зачем для группы ресурсов нужно расположение. И если ресурсы и группа ресурсов могут находиться в разных расположениях, то какой смысл указывать расположение группы ресурсов? В группе ресурсов хранятся метаданные о ресурсах. Указывая расположение группы ресурсов, вы определяете расположение метаданных. В целях обеспечения соответствия необходимо убедиться, что данные хранятся в определенном регионе.

   Если регион группы ресурсов временно недоступен, вы не сможете обновить ресурсы в группе ресурсов, так как недоступны их метаданные. Ресурсы в других регионах будут работать должным образом, но обновить их не получится. Дополнительные сведения о создании надежных приложений см. в разделе [Разработка надежных приложений Azure](/azure/architecture/checklist/resiliency-per-service).

* Группу ресурсов можно использовать, чтобы определить область действия управления доступом для административных действий. Для управления группой ресурсов можно назначать [Политики Azure](../../governance/policy/overview.md), [роли Azure](../../role-based-access-control/role-assignments-portal.md) или [блокировки ресурсов](lock-resources.md).

* К группе ресурсов можно [применять теги](tag-resources.md). Ресурсы в группе ресурсов не наследуют эти теги.

* Ресурс может подключаться к ресурсам в других группах ресурсов. Этот сценарий является типичным, если два ресурса связаны, но не имеют общего жизненного цикла. Например, веб-приложение может подключаться к базе данных в другой группе ресурсов.

* При удалении группы ресурсов также удаляются все ресурсы в ней. Сведения о том, как Azure Resource Manager управляет этими удалениями, см. в разделе [Группа ресурсов Azure Resource Manager и удаление ресурсов](delete-resource-group.md).

* В каждой группе ресурсов можно развернуть до 800 экземпляров типа ресурсов. Некоторых типов ресурсов [ограничение в 800 экземпляров не касается](resources-without-resource-group-limit.md). Дополнительные сведения см. в разделе [Ограничения группы ресурсов](azure-subscription-service-limits.md#resource-group-limits).

* Некоторые ресурсы могут существовать за пределами группы ресурсов. Эти ресурсы развертываются в [подписке](../templates/deploy-to-subscription.md), [группе управления](../templates/deploy-to-management-group.md) или [арендаторе](../templates/deploy-to-tenant.md). В этих областях поддерживаются только определенные типы ресурсов.

* Чтобы создать группу ресурсов, можно использовать [портал](manage-resource-groups-portal.md#create-resource-groups), [PowerShell](manage-resource-groups-powershell.md#create-resource-groups), [Azure CLI](manage-resource-groups-cli.md#create-resource-groups) или [шаблон Azure Resource Manager](../templates/deploy-to-subscription.md#resource-groups).

## <a name="resiliency-of-azure-resource-manager"></a>Устойчивость Azure Resource Manager

Служба Azure Resource Manager предназначена для обеспечения устойчивости и постоянной доступности. Операции Resource Manager и операции уровня управления (запросы, отправленные на `management.azure.com`) в REST API:

* Распределяются между регионами. Некоторые службы являются региональными.

* Распределяются между зонами доступности (как и регионами) в расположениях с несколькими зонами доступности.

* Не зависят от единственного логического центра обработки данных.

* Никогда не отключаются для операций обслуживания.

Эта устойчивость относится к службам, которые получают запросы через Resource Manager. Например, Key Vault получает преимущества от этой устойчивости.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения об ограничениях, применяемых в службах Azure, см. в статье [Подписка Azure, границы, квоты и ограничения службы](azure-subscription-service-limits.md).

* Чтобы узнать о том, как перемещать ресурсы, ознакомьтесь с разделом [Перемещение ресурсов в новую группу ресурсов или подписку](move-resource-group-and-subscription.md).

* Дополнительные сведения о маркировке ресурсов тегами см. в статье [Использование тегов для организации ресурсов в Azure](tag-resources.md).

* Дополнительные сведения о блокировке ресурсов см. в статье [Блокировка ресурсов для предотвращения непредвиденных изменений](lock-resources.md).
