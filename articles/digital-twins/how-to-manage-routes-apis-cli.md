---
title: Управление конечными точками и маршрутами (API и интерфейс командной строки)
titleSuffix: Azure Digital Twins
description: Узнайте, как настроить конечные точки и маршруты событий для данных Azure Digital двойников и управлять ими.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 11/18/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 40cf83b0ca9e4c794979f2b20ddb73360758abc5
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102198546"
---
# <a name="manage-endpoints-and-routes-in-azure-digital-twins-apis-and-cli"></a>Управление конечными точками и маршрутами в Azure Digital двойников (API и CLI)

[!INCLUDE [digital-twins-route-selector.md](../../includes/digital-twins-route-selector.md)]

В Azure Digital двойников можно маршрутизировать [уведомления о событиях](how-to-interpret-event-data.md) в подчиненные службы или подключенные ресурсы вычислений. Для этого сначала необходимо настроить **конечные точки**, которые могут получать события. Затем можно создать  [**маршруты событий**](concepts-route-events.md) , указывающие, какие события, создаваемые Azure Digital двойников, доставляются в конечные точки.

В этой статье описывается процесс создания конечных точек и маршрутов с помощью [интерфейсов API](/rest/api/azure-digitaltwins/), [пакета SDK для .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true)и [интерфейса командной строки Azure Digital двойников](how-to-use-cli.md).

Кроме того, можно также управлять конечными точками и маршрутами с помощью [портал Azure](https://portal.azure.com). Версию этой статьи, которая использует портал, см. [*в разделе руководство. Управление конечными точками и маршрутами (портал)*](how-to-manage-routes-portal.md).

## <a name="prerequisites"></a>Предварительные требования

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

В этом разделе объясняется, как создать эти конечные точки с помощью Azure CLI. Вы также можете управлять конечными точками с помощью [API-интерфейсов плоскости управления дигиталтвинсендпоинт](/rest/api/digital-twins/controlplane/endpoints).

[!INCLUDE [digital-twins-endpoint-resources.md](../../includes/digital-twins-endpoint-resources.md)]

### <a name="create-the-endpoint"></a>Создание конечной точки

После создания ресурсов конечной точки их можно использовать для конечной точки Azure Digital двойников. В следующих примерах показано, как создать конечные точки с помощью команды [AZ DT Endpoint Create](/cli/azure/ext/azure-iot/dt/endpoint/create) для [интерфейса командной строки Azure Digital двойников](how-to-use-cli.md). Замените заполнители в командах на сведения о своих собственных ресурсах.

Чтобы создать конечную точку сетки событий, выполните следующие действия.

```azurecli-interactive
az dt endpoint create eventgrid --endpoint-name <Event-Grid-endpoint-name> --eventgrid-resource-group <Event-Grid-resource-group-name> --eventgrid-topic <your-Event-Grid-topic-name> -n <your-Azure-Digital-Twins-instance-name>
```

Чтобы создать конечную точку концентратора событий (проверка подлинности на основе ключей), выполните следующие действия.
```azurecli-interactive
az dt endpoint create eventhub --endpoint-name <Event-Hub-endpoint-name> --eventhub-resource-group <Event-Hub-resource-group> --eventhub-namespace <Event-Hub-namespace> --eventhub <Event-Hub-name> --eventhub-policy <Event-Hub-policy> -n <your-Azure-Digital-Twins-instance-name>
```

Чтобы создать конечную точку раздела служебной шины (проверка подлинности на основе ключей), выполните следующие действия.
```azurecli-interactive 
az dt endpoint create servicebus --endpoint-name <Service-Bus-endpoint-name> --servicebus-resource-group <Service-Bus-resource-group-name> --servicebus-namespace <Service-Bus-namespace> --servicebus-topic <Service-Bus-topic-name> --servicebus-policy <Service-Bus-topic-policy> -n <your-Azure-Digital-Twins-instance-name>
```

После успешного выполнения этих команд служба "Сетка событий", "концентратор событий" или "служебная шина" будет доступна в качестве конечной точки в Azure Digital двойников под именем, которое вы указали вместе с `--endpoint-name` аргументом. Обычно это имя используется в качестве цели для **маршрута события**, который будет создан [Далее в этой статье](#create-an-event-route).

#### <a name="create-an-endpoint-with-identity-based-authentication"></a>Создание конечной точки с проверкой подлинности на основе удостоверений

Можно также создать конечную точку с проверкой подлинности на основе удостоверений, чтобы использовать конечную точку с [управляемым удостоверением](concepts-security.md#managed-identity-for-accessing-other-resources-preview). Этот параметр доступен только для конечных точек концентратора событий и служебной шины (он не поддерживается для службы "Сетка событий").

Ниже приведена команда CLI для создания конечной точки этого типа. Для подключения к заполнителям в команде потребуются следующие значения:
* Идентификатор ресурса Azure для своего экземпляра Digital двойников Azure.
* имя конечной точки
* Тип конечной точки
* пространство имен ресурса конечной точки
* имя концентратора событий или раздела служебной шины
* расположение своего экземпляра Digital двойников для Azure

```azurecli-interactive
az resource create --id <Azure-Digital-Twins-instance-Azure-resource-ID>/endpoints/<endpoint-name> --properties '{\"properties\": { \"endpointType\": \"<endpoint-type>\", \"authenticationType\": \"IdentityBased\", \"endpointUri\": \"sb://<endpoint-namespace>.servicebus.windows.net\", \"entityPath\": \"<name-of-event-hub-or-Service-Bus-topic>\"}, \"location\":\"<instance-location>\" }' --is-full-object
```

### <a name="create-an-endpoint-with-dead-lettering"></a>Создание конечной точки с недоставленным письмом

Если конечная точка не может доставить событие в течение определенного периода времени или после попытки доставить событие определенное количество раз, оно может отправить недоставленное событие в учетную запись хранения. Этот процесс называется **недоставленным**.

Конечные точки с недоставленными сообщениями можно настроить с помощью [API-интерфейсов](how-to-use-apis-sdks.md#overview-control-plane-apis)Azure Digital двойников [CLI](how-to-use-cli.md) или плоскости управления.

Дополнительные сведения о недоставленных сообщениях см. в разделе [*Основные понятия: маршруты событий*](concepts-route-events.md#dead-letter-events). Инструкции по настройке конечной точки с недоставленными сообщениями см. в оставшейся части этого раздела.

#### <a name="set-up-storage-resources"></a>Настройка ресурсов хранилища

Перед настройкой расположения недоставленных сообщений необходимо иметь [учетную запись хранения](../storage/common/storage-account-create.md?tabs=azure-portal) с [контейнером](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) , настроенным в учетной записи Azure. 

Вы получите универсальный код ресурса (URI) для этого контейнера при создании конечной точки позже. Расположение недоставленной буквы будет предоставлено конечной точке в качестве URI контейнера с [маркером SAS](../storage/common/storage-sas-overview.md). Этому маркеру требуется `write` разрешение для целевого контейнера в учетной записи хранения. Полностью сформированный **URI SAS недоставленного письма** будет иметь формат: `https://<storage-account-name>.blob.core.windows.net/<container-name>?<SAS-token>` .

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
    
#### <a name="create-the-dead-letter-endpoint"></a>Создание конечной точки недоставленной буквы

Чтобы создать конечную точку с включенной недоставленным письмом, добавьте следующий параметр недоставленных сообщений в команду [AZ DT Endpoint Create](/cli/azure/ext/azure-iot/dt/endpoint/create) для [интерфейса командной строки двойников для Azure Digital](how-to-use-cli.md).

Значением параметра является **URI SAS недоставленной буквы** , который состоит из имени учетной записи хранения, имени контейнера и маркера SAS, собранного в [предыдущем разделе](#set-up-storage-resources). Этот параметр создает конечную точку с проверкой подлинности на основе ключей.

```azurecli
--deadletter-sas-uri https://<storage-account-name>.blob.core.windows.net/<container-name>?<SAS-token>
```

Добавьте этот параметр в конец команд создания конечной точки из раздела [*Создание конечной точки*](#create-the-endpoint) выше, чтобы создать конечную точку нужного типа, для которого включено неиспользуемое письмо.

Кроме того, можно создавать конечные точки недоставленных сообщений с помощью [API-интерфейсов плоскости управления цифровыми двойников Azure](how-to-use-apis-sdks.md#overview-control-plane-apis) вместо интерфейса командной строки. Для этого просмотрите [документацию по дигиталтвинсендпоинт](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate) , чтобы узнать, как структурировать запрос и добавить параметры недоставленных сообщений.

#### <a name="create-a-dead-letter-endpoint-with-identity-based-authentication"></a>Создание конечной точки недоставленной буквы с проверкой подлинности на основе удостоверений

Можно также создать конечную точку недоставленного письма с проверкой подлинности на основе удостоверений, чтобы использовать конечную точку с [управляемым удостоверением](concepts-security.md#managed-identity-for-accessing-other-resources-preview). Этот параметр доступен только для конечных точек концентратора событий и служебной шины (он не поддерживается для службы "Сетка событий").

Чтобы создать конечную точку этого типа, используйте ту же команду CLI, что и в предыдущих версиях, чтобы [создать конечную точку с проверкой подлинности на основе удостоверений](#create-an-endpoint-with-identity-based-authentication), а дополнительное поле в полезных данных JSON для `deadLetterUri` .

Ниже приведены значения, которые необходимо подключить к заполнителям в команде:
* Идентификатор ресурса Azure для своего экземпляра Digital двойников Azure.
* имя конечной точки
* Тип конечной точки
* пространство имен ресурса конечной точки
* имя концентратора событий или раздела служебной шины
* сведения о **URI SAS недоставленной буквы** : имя учетной записи хранения, имя контейнера
* расположение своего экземпляра Digital двойников для Azure

```azurecli-interactive
az resource create --id <Azure-Digital-Twins-instance-Azure-resource-ID>/endpoints/<endpoint-name> --properties '{\"properties\": { \"endpointType\": \"<endpoint-type>\", \"authenticationType\": \"IdentityBased\", \"endpointUri\": \"sb://<endpoint-namespace>.servicebus.windows.net\", \"entityPath\": \"<name-of-event-hub-or-Service-Bus-topic>\", \"deadLetterUri\": \"https://<storage-account-name>.blob.core.windows.net/<container-name>\"}, \"location\":\"<instance-location>\" }' --is-full-object
```

#### <a name="message-storage-schema"></a>Схема хранилища сообщений

После настройки конечной точки с недоставленными сообщениями они будут храниться в следующем формате в учетной записи хранения:

`{container}/{endpoint-name}/{year}/{month}/{day}/{hour}/{event-ID}.json`

Недоставленные сообщения будут соответствовать схеме исходного события, предназначенного для доставки в исходную конечную точку.

Ниже приведен пример недоставленного сообщения для [уведомления о создании двойника](how-to-interpret-event-data.md#digital-twin-life-cycle-notifications).

```json
{
  "specversion": "1.0",
  "id": "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.DigitalTwins.Twin.Create",
  "source": "<your-instance>.api.<your-region>.da.azuredigitaltwins-test.net",
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

**Предварительные требования**. прежде чем можно будет перейти к созданию маршрута, необходимо создать конечные точки, как описано ранее в этой статье. После завершения настройки конечных точек можно перейти к созданию маршрута события.

> [!NOTE]
> Если вы недавно развернули конечные точки, убедитесь, что они завершили развертывание, **прежде чем** пытаться использовать их для нового маршрута событий. Если развертывание маршрута завершается сбоем из-за неготовности конечных точек, подождите несколько минут и повторите попытку.
>
> Если вы создаете скрипт для этого потока, это можно сделать, создав период времени ожидания в 2-3 минут до завершения развертывания службы конечной точки перед переходом к настройке маршрута.

Чтобы фактически передавать данные из Azure Digital двойников в конечную точку, необходимо определить **маршрут события**. Маршруты событий используются для подключения потока событий во всей системе и к подчиненным службам.

Определение маршрута может содержать следующие элементы:
* Имя маршрута, которое вы хотите использовать
* Имя конечной точки, которую вы хотите использовать.
* Фильтр, который определяет, какие события отправляются в конечную точку 

Если имя маршрута отсутствует, сообщения не направляются за пределы Azure Digital двойников. Если имеется имя маршрута и фильтр имеет значение `true` , все сообщения направляются в конечную точку. Если имеется имя маршрута и добавлен другой фильтр, сообщения будут отфильтрованы на основе фильтра.

Один маршрут должен разрешать выбор нескольких уведомлений и типов событий. 

Маршруты событий можно создать с помощью [  интерфейсов API плоскости данных](/rest/api/digital-twins/dataplane/eventroutes) Azure Digital двойников евентраутес или [команды командной строки **AZ DT Route**](/cli/azure/ext/azure-iot/dt/route). В оставшейся части этого раздела рассматривается процесс создания.

### <a name="create-routes-with-the-apis-and-c-sdk"></a>Создание маршрутов с помощью API и пакета SDK для C#

Один из способов определения маршрутов событий — с помощью [API-интерфейсов плоскости данных](how-to-use-apis-sdks.md#overview-data-plane-apis). В примерах в этом разделе используется [пакет SDK для .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true).

`CreateOrReplaceEventRouteAsync` — Это вызов пакета SDK, который используется для добавления маршрута события. Ниже приведен пример использования.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/eventRoute_operations.cs" id="CreateEventRoute":::
    
> [!TIP]
> Все функции пакета SDK имеют синхронные и асинхронные версии.

#### <a name="event-route-sample-sdk-code"></a>Пример кода пакета SDK для маршрута события

В следующем примере метода показано, как создать, перечислить и удалить маршрут события с помощью пакета SDK для C#:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/eventRoute_operations.cs" id="FullEventRouteSample":::

### <a name="create-routes-with-the-cli"></a>Создание маршрутов с помощью интерфейса командной строки

Маршруты также можно управлять с помощью команд [AZ DT Route](/cli/azure/ext/azure-iot/dt/route) для интерфейса командной строки Azure Digital двойников. 

Дополнительные сведения об использовании интерфейса командной строки и доступных команд см. в разделе [*как использовать интерфейс командной строки Azure Digital двойников*](how-to-use-cli.md).

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

[!INCLUDE [digital-twins-route-metrics](../../includes/digital-twins-route-metrics.md)]

## <a name="next-steps"></a>Дальнейшие действия

Сведения о различных типах сообщений о событиях, которые можно получить:
* [*Пошаговое руководство. анализ данных события*](how-to-interpret-event-data.md)
