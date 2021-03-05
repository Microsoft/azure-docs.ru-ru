---
title: Интеграция с Azure Maps
titleSuffix: Azure Digital Twins
description: Узнайте, как использовать функции Azure для создания функции, которая может использовать двойника Graph и уведомления Azure Digital двойников для обновления карт Azure Maps.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 1/19/2021
ms.topic: how-to
ms.service: digital-twins
ms.reviewer: baanders
ms.openlocfilehash: 990a0ee73bd91ccb748c948b5fcf0e6124d84a03
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102201436"
---
# <a name="use-azure-digital-twins-to-update-an-azure-maps-indoor-map"></a>Использование Azure Digital двойников для обновления карт Azure Maps

В этой статье рассматриваются шаги, необходимые для использования данных Azure Digital двойников для обновления сведений, отображаемых на *карте* с помощью [Azure Maps](../azure-maps/about-azure-maps.md). Azure Digital двойников хранит Граф отношений устройств IoT и направляет данные телеметрии в разные конечные точки, делая ее идеальной службой для обновления информационных наложений на картах.

В этом практические руководства будут рассмотрены следующие вопросы.

1. Настройка экземпляра Azure Digital двойников для отправки событий обновления двойника в функцию в функциях [Azure](../azure-functions/functions-overview.md).
2. Создание функции для обновления набором состояний функций для Azure Maps карт.
3. Сведения о хранении идентификатора сопоставления и идентификатора состояния компонентов в графе Azure Digital двойников.

### <a name="prerequisites"></a>Предварительные требования

* Следуйте указаниям в руководстве по цифровому двойников Azure [*: подключение комплексного решения*](./tutorial-end-to-end.md).
    * Вы будете расширять эту двойника с помощью дополнительной конечной точки и маршрута. Вы также добавите в приложение функции другую функцию из этого руководства. 
* Следуйте указаниям в руководстве по Azure Maps [*. Создайте карты с помощью Azure Maps Creator*](../azure-maps/tutorial-creator-indoor-maps.md) , чтобы создать Azure Mapsную карту с набором *состояний компонентов*.
    * [Feature статесетс](../azure-maps/creator-indoor-maps.md#feature-statesets) — это коллекции динамических свойств (состояний), назначенных функциям набора данных, например комнатам или оборудованию. В приведенном выше руководстве по Azure Mapsю набор состояния компонента сохраняет состояние комнаты, которое будет отображаться на карте.
    * Вам понадобятся *идентификаторы* и *ключ подписки* Azure Maps.

### <a name="topology"></a>Топология

На рисунке ниже показано, где элементы интеграции карт в этом руководстве соответствуют более широкому комплексному сценарию Azure Digital двойников.

:::image type="content" source="media/how-to-integrate-maps/maps-integration-topology.png" alt-text="Представление служб Azure в комплексном сценарии, в котором выделяется элемент интеграции Maps" lightbox="media/how-to-integrate-maps/maps-integration-topology.png":::

## <a name="create-a-function-to-update-a-map-when-twins-update"></a>Создание функции для обновления схемы при двойников обновления

Сначала вы создадите маршрут в Azure Digital двойников, чтобы перенаправить все события обновления двойника в раздел сетки событий. Затем вы будете использовать функцию для считывания этих сообщений и обновления набор состояний компонентов в Azure Maps. 

## <a name="create-a-route-and-filter-to-twin-update-notifications"></a>Создание маршрута и фильтра для уведомлений об обновлении цифрового двойника

Экземпляры Azure Digital двойников могут создавать события обновления двойника при каждом обновлении состояния двойника. Руководство по цифровому двойников Azure. [*Подключение к комплексному решению*](./tutorial-end-to-end.md) , описанному выше, рассматривается в сценарии, в котором для обновления атрибута температуры, прикрепленного к двойника комнаты, используется термометр. Вы будете расширять это решение, подписавшись на уведомления об обновлениях для двойников и используя эти сведения для обновления карт.

Этот шаблон считывает данные из комнаты двойника напрямую, а не с устройства IoT, что дает возможность изменить базовый источник данных для температуры без необходимости обновления логики сопоставления. Например, можно добавить несколько термометров или задать для этой комнаты общий доступ к термометру в другой комнате, не требуя обновления логики карт.

1. Создайте раздел "Сетка событий", который будет принимать события из вашего экземпляра Digital двойников для Azure.
    ```azurecli-interactive
    az eventgrid topic create -g <your-resource-group-name> --name <your-topic-name> -l <region>
    ```

2. Создайте конечную точку, чтобы связать раздел "Сетка событий" с цифровым двойников Azure.
    ```azurecli-interactive
    az dt endpoint create eventgrid --endpoint-name <Event-Grid-endpoint-name> --eventgrid-resource-group <Event-Grid-resource-group-name> --eventgrid-topic <your-Event-Grid-topic-name> -n <your-Azure-Digital-Twins-instance-name>
    ```

3. Создайте маршрут в Azure Digital Twins для отправки событий обновления цифрового двойника в конечную точку.

    >[!NOTE]
    >В Cloud Shell присутствует **известная проблема**, которая затрагивает такие группы команд: `az dt route`, `az dt model`, `az dt twin`.
    >
    >Чтобы устранить эту проблему, выполните `az login` в Cloud Shell перед выполнением команды или используйте [локальный CLI](/cli/azure/install-azure-cli) вместо Cloud Shell. Дополнительные сведения см. в статье [*Устранение неполадок: известные проблемы в Azure Digital Twins*](troubleshoot-known-issues.md#400-client-error-bad-request-in-cloud-shell).

    ```azurecli-interactive
    az dt route create -n <your-Azure-Digital-Twins-instance-name> --endpoint-name <Event-Grid-endpoint-name> --route-name <my_route> --filter "type = 'Microsoft.DigitalTwins.Twin.Update'"
    ```

## <a name="create-a-function-to-update-maps"></a>Создание функции для обновления карт

Вы создадите **функцию, активируемую с** помощью функции "Сетка событий" в приложении-функции, из комплексного учебника ([*Учебник: подключение комплексного решения*](./tutorial-end-to-end.md)). Эта функция выполнит распаковку этих уведомлений и отправит обновления в Azure Maps наборе состояний компонентов, чтобы обновить температуру одной комнаты.

Справочные сведения см. в следующем документе: триггер службы " [*Сетка событий Azure" для функций Azure*](../azure-functions/functions-bindings-event-grid-trigger.md).

Замените код функции следующим кодом. Он фильтрует только обновления пространства двойников, считывает обновленную температуру и отправляет эти сведения в Azure Maps.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/updateMaps.cs":::

В приложении функции необходимо задать две переменные среды. Один из них — [ключ первичной подписки Azure Maps](../azure-maps/quick-demo-map-app.md#get-the-primary-key-for-your-account), а второй — [идентификатор своего Azure Maps](../azure-maps/tutorial-creator-indoor-maps.md#create-a-feature-stateset).

```azurecli-interactive
az functionapp config appsettings set --name <your-App-Service-(function-app)-name> --resource-group <your-resource-group> --settings "subscription-key=<your-Azure-Maps-primary-subscription-key>"
az functionapp config appsettings set --name <your-App-Service-(function-app)-name>  --resource-group <your-resource-group> --settings "statesetID=<your-Azure-Maps-stateset-ID>"
```

### <a name="view-live-updates-on-your-map"></a>Просмотр динамических обновлений на карте

Чтобы увидеть температуру в динамическом обновлении, выполните следующие действия:

1. Начните отправлять смоделированные данные IoT, запустив проект **девицесимулатор** из учебника по цифровым двойников Azure [*: подключение комплексного решения*](tutorial-end-to-end.md). Инструкции для этого см. в разделе [*Настройка и запуск моделирования*](././tutorial-end-to-end.md#configure-and-run-the-simulation) .
2. Используйте [модуль **Azure Maps**](../azure-maps/how-to-use-indoor-module.md) для отображения карт, созданных в Azure Maps Creator.
    1. Скопируйте HTML-код из [*примера: используйте модуль Maps*](../azure-maps/how-to-use-indoor-module.md#example-use-the-indoor-maps-module) в учебнике по картам для подразделов [*: используйте модуль map Maps (Azure Maps*](../azure-maps/how-to-use-indoor-module.md) ) к локальному файлу.
    1. Замените *ключ подписки*, *тилесетид* и *статесетид*  в локальном HTML-файле своими значениями.
    1. Откройте этот файл в браузере.

Оба образца отправляют температуру в совместимом диапазоне, поэтому на карте должен отображаться цвет для обновления комнаты 121 примерно каждые 30 секунд.

:::image type="content" source="media/how-to-integrate-maps/maps-temperature-update.png" alt-text="Схема Office, в которой отображается комната 121, окрашенный оранжевый":::

## <a name="store-your-maps-information-in-azure-digital-twins"></a>Хранение сведений о картах в цифровом двойников Azure

Теперь, когда у вас есть жестко закодированное решение для обновления сведений о картах, вы можете использовать граф Azure Digital двойников для хранения всей информации, необходимой для обновления карты. Это будет включать идентификатор набора состояний, идентификатор подписки и идентификатор функции для каждой карты и расположения соответственно. 

Решение для этого конкретного примера будет включать в себя обновление каждого пространства верхнего уровня, чтобы иметь идентификатор и атрибут идентификатора подписки, а для каждой комнаты — идентификатор компонента. Эти значения необходимо задать один раз при инициализации графа двойника, а затем запросить эти значения для каждого события обновления двойника.

В зависимости от конфигурации топологии вы сможете хранить эти три атрибута на разных уровнях, соотнесенных с степенью детализации вашей схемы.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управлении, обновлении и извлечении данных из графа двойников см. в следующих статьях:

* [*Практические руководства. Управление цифровыми двойников*](./how-to-manage-twin.md)
* [*Пошаговое руководство. запрос графа двойника*](./how-to-query-graph.md)
