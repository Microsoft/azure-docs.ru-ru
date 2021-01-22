---
title: Управление конечными точками и маршрутами (API и интерфейс командной строки)
titleSuffix: Azure Digital Twins
description: Узнайте, как настроить конечные точки и маршруты событий для данных Azure Digital двойников и управлять ими.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 11/18/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: fa699163fdf445624c918e714fda890a41a67f07
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/22/2021
ms.locfileid: "98682653"
---
# <a name="manage-endpoints-and-routes-in-azure-digital-twins-apis-and-cli"></a>Управление конечными точками и маршрутами в Azure Digital двойников (API и CLI)

[!INCLUDE [digital-twins-route-selector.md](../../includes/digital-twins-route-selector.md)]

В Azure Digital двойников можно маршрутизировать [уведомления о событиях](how-to-interpret-event-data.md) в подчиненные службы или подключенные ресурсы вычислений. Для этого сначала необходимо настроить **конечные точки**, которые могут получать события. Затем можно создать  [**маршруты событий**](concepts-route-events.md) , указывающие, какие события, создаваемые Azure Digital двойников, доставляются в конечные точки.

В этой статье описывается процесс создания конечных точек и маршрутов с помощью [API маршрутов событий](/rest/api/digital-twins/dataplane/eventroutes), [пакета SDK для .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true)и [интерфейса командной строки Azure Digital двойников](how-to-use-cli.md).

Кроме того, можно также управлять конечными точками и маршрутами с помощью [портал Azure](https://portal.azure.com). Версию этой статьи, которая использует портал, см. [*в разделе руководство. Управление конечными точками и маршрутами (портал)*](how-to-manage-routes-portal.md).

## <a name="prerequisites"></a>Предварительные условия

- Вам потребуется **учетная запись Azure** (вы можете настроить ее [здесь](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)бесплатно).
- Вам потребуется **экземпляр Azure Digital двойников** в подписке Azure. Если у вас еще нет экземпляра, его можно создать, выполнив действия, описанные в разделе [*инструкции. Настройка экземпляра и проверки подлинности*](how-to-set-up-instance-cli.md). Используйте следующие значения из программы установки, которые можно использовать далее в этой статье:
    - Имя экземпляра
    - Группа ресурсов

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
    
## <a name="create-an-endpoint-for-azure-digital-twins"></a>Создание конечной точки для Azure Digital двойников

Ниже приведены поддерживаемые типы конечных точек, которые можно создать для экземпляра.
* [Сетка событий](../event-grid/overview.md)
* [Центры событий](../event-hubs/event-hubs-about.md)
* [Служебная шина](../service-bus-messaging/service-bus-messaging-overview.md)

Дополнительные сведения о различных типах конечных точек см. в статье [*Выбор между службами обмена сообщениями Azure*](../event-grid/compare-messaging-services.md).

Чтобы связать конечную точку с цифровым двойников Azure, необходимо уже существовать раздел "Сетка событий", концентратор событий или служебная шина, которые вы используете для этой конечной точки. 

### <a name="create-an-event-grid-endpoint"></a>Создание конечной точки сетки событий

В следующем примере показано, как создать конечную точку типа "Сетка событий" с помощью Azure CLI.

Сначала создайте раздел "Сетка событий". Вы можете использовать следующую команду или просмотреть действия более подробно, перейдя [к разделу *Создание пользовательского раздела*](../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic) краткого руководства по *событиям* сетки событий.

```azurecli-interactive
az eventgrid topic create -g <your-resource-group-name> --name <your-topic-name> -l <region>
```

> [!TIP]
> Чтобы получить список имен регионов Azure, которые можно передать в команды в Azure CLI, выполните следующую команду:
> ```azurecli-interactive
> az account list-locations -o table
> ```

После создания раздела можно связать его с Azure Digital двойников, используя следующую [команду Azure Digital ДВОЙНИКОВ CLI](how-to-use-cli.md):

```azurecli-interactive
az dt endpoint create eventgrid --endpoint-name <Event-Grid-endpoint-name> --eventgrid-resource-group <Event-Grid-resource-group-name> --eventgrid-topic <your-Event-Grid-topic-name> -n <your-Azure-Digital-Twins-instance-name>
```

Теперь раздел "Сетка событий" доступен в качестве конечной точки в Azure Digital двойников с именем, указанным в `--endpoint-name` аргументе. Обычно это имя используется в качестве цели для **маршрута события**, который будет создан [Далее в этой статье](#create-an-event-route) с помощью API службы Digital двойников.

### <a name="create-an-event-hubs-or-service-bus-endpoint"></a>Создание концентраторов событий или конечной точки служебной шины

Процесс создания концентраторов событий или конечных точек служебной шины аналогичен процессу сетки событий, показанному выше.

Сначала создайте ресурсы, которые будут использоваться в качестве конечной точки. Вот что требуется:
* Служебная шина: _пространство имен служебной шины_, _раздел служебной шины_, _правило авторизации_
* Концентраторы событий: _пространство имен концентраторов событий_, _концентратор событий_, _правило авторизации_

Затем используйте следующие команды для создания конечных точек в Azure Digital двойников: 

* Добавить конечную точку раздела служебной шины (требуется предварительно созданный ресурс служебной шины)
```azurecli-interactive 
az dt endpoint create servicebus --endpoint-name <Service-Bus-endpoint-name> --servicebus-resource-group <Service-Bus-resource-group-name> --servicebus-namespace <Service-Bus-namespace> --servicebus-topic <Service-Bus-topic-name> --servicebus-policy <Service-Bus-topic-policy> -n <your-Azure-Digital-Twins-instance-name>
```

* Добавить конечную точку концентраторов событий (требуется предварительно созданный ресурс концентраторов событий)
```azurecli-interactive
az dt endpoint create eventhub --endpoint-name <Event-Hub-endpoint-name> --eventhub-resource-group <Event-Hub-resource-group> --eventhub-namespace <Event-Hub-namespace> --eventhub <Event-Hub-name> --eventhub-policy <Event-Hub-policy> -n <your-Azure-Digital-Twins-instance-name>
```

### <a name="create-an-endpoint-with-dead-lettering"></a>Создание конечной точки с недоставленным письмом

Если конечная точка не может доставить событие в течение определенного периода времени или после попытки доставить событие определенное количество раз, оно может отправить недоставленное событие в учетную запись хранения. Этот процесс называется **недоставленным**.

Дополнительные сведения о недоставленных сообщениях см. в разделе [*Основные понятия: маршруты событий*](concepts-route-events.md#dead-letter-events). Инструкции по настройке конечной точки с недоставленными сообщениями см. в оставшейся части этого раздела.

#### <a name="set-up-storage-resources"></a>Настройка ресурсов хранилища

Перед настройкой расположения недоставленных сообщений необходимо иметь [учетную запись хранения](../storage/common/storage-account-create.md?tabs=azure-portal) с [контейнером](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) , настроенным в учетной записи Azure. 

URL-адрес для этого контейнера будет предоставлен при создании конечной точки позже. Расположение недоставленной буквы будет предоставлено конечной точке в качестве URL-адреса контейнера с [маркером SAS](../storage/common/storage-sas-overview.md). Этому маркеру требуется `write` разрешение для целевого контейнера в учетной записи хранения. Полностью сформированный URL-адрес будет иметь формат: `https://<storageAccountname>.blob.core.windows.net/<containerName>?<SASToken>` .

Выполните следующие действия, чтобы настроить эти ресурсы хранилища в учетной записи Azure, чтобы подготовиться к настройке подключения к конечной точке в следующем разделе.

1. Выполните действия, описанные в разделе [*Создание учетной записи хранения*](../storage/common/storage-account-create.md?tabs=azure-portal) , чтобы создать **учетную запись хранения** в подписке Azure. Запишите имя учетной записи хранения, чтобы использовать ее позже.
2. Выполните действия, описанные в разделе [*Создание контейнера*](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) , чтобы создать **контейнер** в новой учетной записи хранения. Запишите имя контейнера, чтобы использовать его позже.
3. Затем создайте **маркер SAS** для учетной записи хранения, которую конечная точка может использовать для доступа к ней. Начните с перехода к своей учетной записи хранения в [портал Azure](https://ms.portal.azure.com/#home) (ее можно найти по имени с помощью панели поиска портала).
4. На странице Учетная запись хранения щелкните ссылку _подписанный_ URL-адрес на левой панели навигации, чтобы начать настройку маркера SAS.

    :::image type="content" source="./media/how-to-manage-routes-apis-cli/generate-sas-token-1.png" alt-text="Страница учетной записи хранения в портал Azure" lightbox="./media/how-to-manage-routes-apis-cli/generate-sas-token-1.png":::

1. На *странице подписанный URL-* адрес в разделе *Разрешенные службы* и *Разрешенные типы ресурсов* выберите необходимые параметры. Необходимо выбрать по крайней мере одно поле в каждой категории. В разделе *Разрешенные разрешения* выберите **запись** (при необходимости можно также выбрать другие разрешения).
1. Задайте необходимые значения для остальных параметров.
1. По завершении нажмите кнопку _создать SAS и строку подключения_ , чтобы создать маркер SAS. 

    :::image type="content" source="./media/how-to-manage-routes-apis-cli/generate-sas-token-2.png" alt-text="На странице учетной записи хранения в портал Azure отображаются все параметры, выбранные для создания маркера SAS, и выделяется кнопка &quot;создать SAS и строку подключения&quot;." lightbox="./media/how-to-manage-routes-apis-cli/generate-sas-token-2.png"::: 

1. При этом в нижней части той же страницы будут созданы несколько значений SAS и строк подключения, расположенных под выбранным параметром. Прокрутите вниз, чтобы просмотреть значения, и используйте значок *Копировать в буфер обмена* , чтобы скопировать значение **маркера SAS** . Сохраните его для последующего использования.

    :::image type="content" source="./media/how-to-manage-routes-apis-cli/copy-sas-token.png" alt-text="Скопируйте маркер SAS для использования в секрете недоставленной буквы." lightbox="./media/how-to-manage-routes-apis-cli/copy-sas-token.png":::
    
#### <a name="configure-the-endpoint"></a>Настройка конечной точки

Чтобы создать конечную точку с включенной недоставленным письмом, необходимо создать конечную точку с помощью API-интерфейсов Azure Resource Manager. 

1. Во-первых, используйте [документацию по api Azure Resource Manager](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate) , чтобы настроить запрос на создание конечной точки, и заполните необходимые параметры запроса. 

1. Затем добавьте `deadLetterSecret` поле в объект Properties в **тексте** запроса. Задайте это значение в соответствии с шаблоном ниже, который ремесла URL-адрес из имени учетной записи хранения, имени контейнера и значения маркера SAS, собранного в [предыдущем разделе](#set-up-storage-resources).
      
  :::code language="json" source="~/digital-twins-docs-samples/api-requests/deadLetterEndpoint.json":::

1. Отправьте запрос на создание конечной точки.

Дополнительные сведения о структурировании этого запроса см. в документации по цифровым двойников REST API Azure: [Endpoints-Дигиталтвинсендпоинт CreateOrUpdate](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate).

### <a name="message-storage-schema"></a>Схема хранилища сообщений

После настройки конечной точки с недоставленными сообщениями они будут храниться в следующем формате в учетной записи хранения:

`{container}/{endpointName}/{year}/{month}/{day}/{hour}/{eventId}.json`

Недоставленные сообщения будут соответствовать схеме исходного события, предназначенного для доставки в исходную конечную точку.

Ниже приведен пример недоставленного сообщения для [уведомления о создании двойника](how-to-interpret-event-data.md#digital-twin-life-cycle-notifications).

```json
{
  "specversion": "1.0",
  "id": "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.DigitalTwins.Twin.Create",
  "source": "<yourInstance>.api.<yourregion>.da.azuredigitaltwins-test.net",
  "data": {
    "$dtId": "<yourInstance>xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "$etag": "W/\"xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
    "TwinData": "some sample",
    "$metadata": {
      "$model": "dtmi:test:deadlettermodel;1",
      "room": {
        "lastUpdateTime": "2020-10-14T01:11:49.3576659Z"
      }
    }
  },
  "subject": "<yourInstance>xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "time": "2020-10-14T01:11:49.3667224Z",
  "datacontenttype": "application/json",
  "traceparent": "00-889a9094ba22b9419dd9d8b3bfe1a301-f6564945cb20e94a-01"
}
```

## <a name="create-an-event-route"></a>Создание маршрута события

Чтобы фактически передавать данные из Azure Digital двойников в конечную точку, необходимо определить **маршрут события**. **API-интерфейсы** Azure Digital двойников евентраутес позволяют разработчикам связывать потоки событий во всей системе и в подчиненных службах. Дополнительные сведения о маршрутах событий см. в статье [*Маршрутизация событий цифровых двойников Azure*](concepts-route-events.md).

В примерах в этом разделе используется [пакет SDK для .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true).

**Предварительные требования**. прежде чем можно будет перейти к созданию маршрута, необходимо создать конечные точки, как описано ранее в этой статье. После завершения настройки конечных точек можно перейти к созданию маршрута события.

> [!NOTE]
> Если вы недавно развернули конечные точки, убедитесь, что они завершили развертывание, **прежде чем** пытаться использовать их для нового маршрута событий. Если развертывание маршрута завершается сбоем из-за неготовности конечных точек, подождите несколько минут и повторите попытку.
>
> Если вы создаете скрипт для этого потока, это можно сделать, создав период времени ожидания в 2-3 минут до завершения развертывания службы конечной точки перед переходом к настройке маршрута.

### <a name="creation-code-with-apis-and-the-c-sdk"></a>Создание кода с помощью API и пакета SDK для C#

Маршруты событий определяются с помощью [API-интерфейсов плоскости данных](how-to-use-apis-sdks.md#overview-data-plane-apis). 

Определение маршрута может содержать следующие элементы:
* Имя маршрута, которое вы хотите использовать
* Имя конечной точки, которую вы хотите использовать.
* Фильтр, который определяет, какие события отправляются в конечную точку 

Если имя маршрута отсутствует, сообщения не направляются за пределы Azure Digital двойников. Если имеется имя маршрута и фильтр имеет значение `true` , все сообщения направляются в конечную точку. Если имеется имя маршрута и добавлен другой фильтр, сообщения будут отфильтрованы на основе фильтра.

Один маршрут должен разрешать выбор нескольких уведомлений и типов событий. 

`CreateOrReplaceEventRouteAsync` — Это вызов пакета SDK, который используется для добавления маршрута события. Ниже приведен пример использования.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/eventRoute_operations.cs" id="CreateEventRoute":::
    
> [!TIP]
> Все функции пакета SDK имеют синхронные и асинхронные версии.

### <a name="event-route-sample-code"></a>Пример кода для маршрута события

В следующем примере метода показано, как создать, перечислить и удалить маршрут события.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/eventRoute_operations.cs" id="FullEventRouteSample":::

## <a name="filter-events"></a>События фильтра

Без фильтрации конечные точки получают различные события из Azure Digital двойников:
* Данные телеметрии, созданные [цифровым двойников](concepts-twins-graph.md) с помощью API службы Digital двойников
* Двойника уведомления об изменении свойств для любых двойника в экземпляре цифрового двойников Azure
* События жизненного цикла, которые срабатывают при создании или удалении двойников или связей

Вы можете ограничить отправляемые события, добавив **Фильтр** для конечной точки в маршруте события.

Чтобы добавить фильтр, можно использовать запрос на размещение *https://{The-Azure-Digital-двойников-имя_узла}/евентраутес/{Event-Route-Name}? API-Version = 2020-10-31* со следующим текстом:

:::code language="json" source="~/digital-twins-docs-samples/api-requests/filter.json":::

Ниже приведены поддерживаемые фильтры маршрутов. Используйте сведения в столбце *схема текста фильтра* , чтобы заменить `<filter-text>` заполнитель в тексте запроса выше.

[!INCLUDE [digital-twins-route-filters](../../includes/digital-twins-route-filters.md)]

## <a name="manage-endpoints-and-routes-with-cli"></a>Управление конечными точками и маршрутами с помощью интерфейса командной строки

Также можно управлять конечными точками и маршрутами с помощью интерфейса командной строки Azure Digital двойников. Дополнительные сведения об использовании интерфейса командной строки и доступных команд см. в разделе [*как использовать интерфейс командной строки Azure Digital двойников*](how-to-use-cli.md).

[!INCLUDE [digital-twins-route-metrics](../../includes/digital-twins-route-metrics.md)]

## <a name="next-steps"></a>Следующие шаги

Сведения о различных типах сообщений о событиях, которые можно получить:
* [*Пошаговое руководство. анализ данных события*](how-to-interpret-event-data.md)
