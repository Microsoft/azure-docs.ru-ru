---
title: Интеграция со службой "Аналитика временных рядов Azure"
titleSuffix: Azure Digital Twins
description: См. статью Настройка маршрутов событий из Azure Digital двойников в службу "аналитика временных рядов Azure".
author: alexkarcher-msft
ms.author: alkarche
ms.date: 1/19/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 608f883304dbc8e1ea8b0127668125ae50ca0b11
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105564948"
---
# <a name="integrate-azure-digital-twins-with-azure-time-series-insights"></a>Интеграция Azure Digital двойников со службой "аналитика временных рядов Azure"

В этой статье вы узнаете, как интегрировать Azure Digital двойников со службой ["аналитика временных рядов Azure" (TSI)](../time-series-insights/overview-what-is-tsi.md).

Решение, описанное в этой статье, позволит собирать и анализировать исторические данные о решении IoT. Azure Digital Twins отлично подходит для передачи данных в Аналитику временных рядов, так как позволяет сопоставлять несколько потоков данных и стандартизировать информацию перед отправкой в Аналитику временных рядов. 

## <a name="prerequisites"></a>Предварительные условия

Прежде чем можно будет настроить связь со службой "аналитика временных рядов", необходимо иметь **экземпляр Digital двойников для Azure**. Этот экземпляр следует настроить с возможностью обновлять сведения о цифровом двойника на основе данных, так как вам потребуется обновить информацию двойника несколько раз, чтобы увидеть, что данные отписываются в службе "аналитика временных рядов". 

Если вы еще не настроили этот параметр, вы можете создать его, следуя руководству по цифровому двойников Azure [*: подключение комплексного решения*](./tutorial-end-to-end.md). В этом руководстве описано, как настроить экземпляр Digital двойников для Azure, который работает с виртуальным устройством IoT, чтобы активировать цифровые двойника обновления.

## <a name="solution-architecture"></a>Архитектура решения

Вы будете присоединить службу "аналитика временных рядов" к Azure Digital двойников по указанному ниже пути.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-integrate-time-series-insights/diagram-simple.png" alt-text="Представление служб Azure в комплексном сценарии, выделяющий аналитику временных рядов" lightbox="media/how-to-integrate-time-series-insights/diagram-simple.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="create-a-route-and-filter-to-twin-update-notifications"></a>Создание маршрута и фильтра для уведомлений об обновлении цифрового двойника

Экземпляры Azure Digital двойников могут создавать [события обновления двойника](how-to-interpret-event-data.md) при каждом обновлении состояния двойника. В этом разделе вы создадите [**маршрут событий**](concepts-route-events.md) Azure Digital двойников, который будет направлять эти события обновления в [концентраторы событий](../event-hubs/event-hubs-about.md) Azure для дальнейшей обработки.

Руководство по работе с цифровыми Двойниковми Azure [*: подключение к комплексному решению*](./tutorial-end-to-end.md) с использованием термометра для обновления атрибута температуры цифрового двойника, представляющего комнату. Этот шаблон основан на обновлениях двойника, а не на пересылке телеметрических данных с устройства IoT, что дает вам возможность изменить базовый источник, не требуя обновления логики "аналитика временных рядов".

1. Сначала создайте пространство имен концентратора событий, которое будет принимать события из вашего экземпляра Digital двойников для Azure. Вы можете использовать приведенные ниже инструкции Azure CLI или воспользоваться портал Azure: [*Краткое руководство. Создание концентратора событий с помощью портал Azure*](../event-hubs/event-hubs-create.md). Чтобы узнать, какие регионы поддерживают концентраторы событий, посетите страницу [*продукты Azure, доступные по регионам*](https://azure.microsoft.com/global-infrastructure/services/?products=event-hubs).

    ```azurecli-interactive
    az eventhubs namespace create --name <name for your Event Hubs namespace> --resource-group <resource group name> -l <region>
    ```

2. Создайте концентратор событий в пространстве имен для получения событий изменения двойника. Укажите имя концентратора событий.

    ```azurecli-interactive
    az eventhubs eventhub create --name <name for your Twins event hub> --resource-group <resource group name> --namespace-name <Event Hubs namespace from above>
    ```

3. Создайте [правило авторизации](/cli/azure/eventhubs/eventhub/authorization-rule#az-eventhubs-eventhub-authorization-rule-create) с разрешениями Send и Receive. Укажите имя правила.

    ```azurecli-interactive
        az eventhubs eventhub authorization-rule create --rights Listen Send --resource-group <resource group name> --namespace-name <Event Hubs namespace from above> --eventhub-name <Twins event hub name from above> --name <name for your Twins auth rule>
    ```

4. Создайте [конечную точку](concepts-route-events.md#create-an-endpoint) цифрового двойников Azure, которая связывает концентратор событий с экземпляром Azure Digital двойников.

    ```azurecli-interactive
    az dt endpoint create eventhub -n <your Azure Digital Twins instance name> --endpoint-name <name for your Event Hubs endpoint> --eventhub-resource-group <resource group name> --eventhub-namespace <Event Hubs namespace from above> --eventhub <Twins event hub name from above> --eventhub-policy <Twins auth rule from above>
    ```

5. Создайте [маршрут](concepts-route-events.md#create-an-event-route) в Azure Digital Twins для отправки событий обновления цифрового двойника в конечную точку. Фильтр в этом маршруте позволит передавать только сообщения обновления двойника в конечную точку.

    >[!NOTE]
    >В Cloud Shell присутствует **известная проблема**, которая затрагивает такие группы команд: `az dt route`, `az dt model`, `az dt twin`.
    >
    >Чтобы устранить эту проблему, выполните `az login` в Cloud Shell перед выполнением команды или используйте [локальный CLI](/cli/azure/install-azure-cli) вместо Cloud Shell. Дополнительные сведения см. в статье [*Устранение неполадок: известные проблемы в Azure Digital Twins*](troubleshoot-known-issues.md#400-client-error-bad-request-in-cloud-shell).

    ```azurecli-interactive
    az dt route create -n <your Azure Digital Twins instance name> --endpoint-name <Event Hub endpoint from above> --route-name <name for your route> --filter "type = 'Microsoft.DigitalTwins.Twin.Update'"
    ```

Прежде чем продолжить, запишите *пространство имен концентраторов событий* и *группу ресурсов*, так как они будут использоваться для создания другого концентратора событий далее в этой статье.

## <a name="create-a-function-in-azure"></a>создавать функции в Azure;

Далее вы будете использовать функции Azure для создания функции, **активируемой концентраторами событий** , внутри приложения-функции. Вы можете использовать приложение-функцию, созданное в комплексном учебнике ([*руководство по подключению комплексного решения*](./tutorial-end-to-end.md)) или к своему собственному интерфейсу. 

Эта функция преобразует события обновления двойника из их исходной формы в документы исправления JSON в объекты JSON, содержащие только обновленные и добавленные значения из двойников.

Дополнительные сведения об использовании концентраторов событий с функциями Azure см. в статье [*триггер концентраторов событий Azure для функций Azure*](../azure-functions/functions-bindings-event-hubs-trigger.md).

В опубликованном приложении-функции добавьте новую функцию с именем **процессдтупдатетотси** со следующим кодом.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/updateTSI.cs":::

>[!NOTE]
>Может потребоваться добавить пакеты в проект с помощью `dotnet add package` команды или диспетчера пакетов NuGet для Visual Studio.

Затем **опубликуйте** новую функцию Azure. Инструкции по выполнению этих действий см. в разделе [*как настроить функцию Azure для обработки данных*](how-to-create-azure-function.md#publish-the-function-app-to-azure).

Сейчас эта функция будет отсылать объекты JSON, которые он создает, во второй концентратор событий, который будет подключаться к службе "аналитика временных рядов". Этот концентратор событий будет создан в следующем разделе.

Позже вы также настроите некоторые переменные среды, которые эта функция будет использовать для подключения к вашим концентраторам событий.

## <a name="send-telemetry-to-an-event-hub"></a>Отправка телеметрии в Центр событий

Теперь вы создадите второй концентратор событий и настроите функцию для потоковой передачи выходных данных в этот концентратор событий. Затем этот Центр событий будет подключен к Аналитике временных рядов.

### <a name="create-an-event-hub"></a>Создание концентратора событий

Чтобы создать второй концентратор событий, можно либо использовать приведенные ниже инструкции Azure CLI, либо воспользоваться портал Azure: [*Краткое руководство. Создание концентратора событий с помощью портал Azure*](../event-hubs/event-hubs-create.md).

1. Подготовьте *пространство имен концентраторов событий* и имя *группы ресурсов* из ранее в этой статье.

2. Создайте Центр событий. Укажите имя концентратора событий.

    ```azurecli-interactive
    az eventhubs eventhub create --name <name for your TSI event hub> --resource-group <resource group name from earlier> --namespace-name <Event Hubs namespace from earlier>
    ```
3. Создайте [правило авторизации](/cli/azure/eventhubs/eventhub/authorization-rule#az-eventhubs-eventhub-authorization-rule-create) с разрешениями Send и Receive. Укажите имя правила.

    ```azurecli-interactive
    az eventhubs eventhub authorization-rule create --rights Listen Send --resource-group <resource group name> --namespace-name <Event Hubs namespace from earlier> --eventhub-name <TSI event hub name from above> --name <name for your TSI auth rule>
    ```

## <a name="configure-your-function"></a>Настройка функции

Далее необходимо задать в приложении функции переменные среды, которые содержат строки подключения для созданных концентраторов событий.

### <a name="set-the-twins-event-hub-connection-string"></a>Задание строки подключения Центра событий цифрового двойника

1. Получите [строку подключения концентратора событий](../event-hubs/event-hubs-get-connection-string.md)двойников, используя правила авторизации, созданные ранее для центра двойников.

    ```azurecli-interactive
    az eventhubs eventhub authorization-rule keys list --resource-group <resource group name> --namespace-name <Event Hubs namespace> --eventhub-name <Twins event hub name from earlier> --name <Twins auth rule from earlier>
    ```

2. Используйте значение *примариконнектионстринг* из результата, чтобы создать параметр приложения в приложении функции, который содержит строку подключения:

    ```azurecli-interactive
    az functionapp config appsettings set --settings "EventHubAppSetting-Twins=<Twins event hub connection string>" -g <resource group> -n <your App Service (function app) name>
    ```

### <a name="set-the-time-series-insights-event-hub-connection-string"></a>Задание строки подключения Центра событий Аналитики временных рядов

1. Получите [строку подключения концентратора событий](../event-hubs/event-hubs-get-connection-string.md)TSI, используя правила авторизации, созданные ранее для центра "аналитика временных рядов".

    ```azurecli-interactive
    az eventhubs eventhub authorization-rule keys list --resource-group <resource group name> --namespace-name <Event Hubs namespace> --eventhub-name <TSI event hub name> --name <TSI auth rule>
    ```

2. Используйте значение *примариконнектионстринг* из результата, чтобы создать параметр приложения в приложении функции, который содержит строку подключения:

    ```azurecli-interactive
    az functionapp config appsettings set --settings "EventHubAppSetting-TSI=<TSI event hub connection string>" -g <resource group> -n <your App Service (function app) name>
    ```

## <a name="create-and-connect-a-time-series-insights-instance"></a>Создание и подключение экземпляра Аналитики временных рядов

Далее вы настроите экземпляр Time Series Insights для получения данных из второго концентратора событий (TSI). Выполните приведенные ниже действия и дополнительные сведения об этом процессе см. в разделе [*учебник. Настройка среды Gen2 PAYG для службы "аналитика временных рядов Azure"*](../time-series-insights/tutorial-set-up-environment.md).

1. В портал Azure начните создавать среду "аналитика временных рядов". 
    1. Выберите ценовую категорию **Gen2 (L1)** .
    2. Для этого окружения необходимо выбрать **идентификатор временного ряда** . Идентификатор временных рядов может содержать до трех значений, которые будут использоваться для поиска данных в аналитике временных рядов. В этом руководстве можно использовать **$dtId**. Дополнительные сведения о выборе идентификатора [*временных рядов*](../time-series-insights/how-to-select-tsid.md)см. в этой статье.
    
        :::image type="content" source="media/how-to-integrate-time-series-insights/create-time-series-insights-environment-1.png" alt-text="Пользовательский интерфейс портала создания для среды &quot;аналитика временных рядов&quot;. Выберите подписку, группу ресурсов и расположение в соответствующих раскрывающихся списках и выберите имя для своей среды." lightbox="media/how-to-integrate-time-series-insights/create-time-series-insights-environment-1.png":::
        
        :::image type="content" source="media/how-to-integrate-time-series-insights/create-time-series-insights-environment-2.png" alt-text="Пользовательский интерфейс портала создания для среды &quot;аналитика временных рядов&quot;. Выбрана ценовая категория Gen2 (L1) и имя свойства идентификатора временных рядов $dtId" lightbox="media/how-to-integrate-time-series-insights/create-time-series-insights-environment-2.png":::

2. Выберите **Далее: источник события** и выберите сведения о концентраторе событий TSI ранее. Также потребуется создать группу потребителей концентраторов событий.
    
    :::image type="content" source="media/how-to-integrate-time-series-insights/event-source-twins.png" alt-text="Пользовательский интерфейс портала создания для источника событий среды &quot;аналитика временных рядов&quot;. Вы создаете источник событий со сведениями о концентраторе событий, указанными выше. Вы также создаете новую группу потребителей." lightbox="media/how-to-integrate-time-series-insights/event-source-twins.png":::

## <a name="begin-sending-iot-data-to-azure-digital-twins"></a>Начало отправки данных IoT в Azure Digital двойников

Чтобы начать отправку данных в службу "аналитика временных рядов", необходимо начать обновление свойств Digital двойника в Azure Digital двойников, изменив значения данных. Используйте команду [AZ DT двойника Update](/cli/azure/ext/azure-iot/dt/twin#ext-azure-iot-az-dt-twin-update) .

Если вы используете полное руководство ([*руководство по подключению комплексного решения*](tutorial-end-to-end.md)) для помощи в настройке среды, можно начать отправку смоделированных данных IOT, запустив проект *девицесимулатор* из примера. Инструкции см. в разделе [*Настройка и запуск модели моделирования*](tutorial-end-to-end.md#configure-and-run-the-simulation) руководства.

## <a name="visualize-your-data-in-time-series-insights"></a>Визуализация данных в Аналитике временных рядов

Теперь данные должны передаваться в экземпляр "аналитика временных рядов", готовый для анализа. Выполните следующие действия, чтобы просмотреть данные, поступающие в.

1. Откройте среду "аналитика временных рядов" в [портал Azure](https://portal.azure.com) (вы можете найти имя своей среды на панели поиска портала). Перейдите по *URL-адресу Обозревателя Аналитики временных рядов*, показанному на странице обзора экземпляра.
    
    :::image type="content" source="media/how-to-integrate-time-series-insights/view-environment.png" alt-text="Выберите URL-адрес обозревателя &quot;аналитика временных рядов&quot; на вкладке &quot;Обзор&quot; среды &quot;аналитика временных рядов&quot;.":::

2. В обозревателе вы увидите три двойников из Azure Digital двойников, показанные слева. Выберите _**thermostat67**_, выберите **температура** и нажмите кнопку **добавить**.

    :::image type="content" source="media/how-to-integrate-time-series-insights/add-data.png" alt-text="Выберите * * thermostat67 * *, выберите * * температура * * и нажмите * * Добавить * *.":::

3. Теперь вы увидите начальное считывание температуры из термостата, как показано ниже. То же самое считывание температуры обновляется для *room21* и *floor1*, и вы можете визуализировать эти потоки данных совместно.
    
    :::image type="content" source="media/how-to-integrate-time-series-insights/initial-data.png" alt-text="Начальные данные температуры отображаются в обозревателе TSI. Это строка случайных значений между 68 и 85":::

4. Если вы разрешаете выполнение симуляции намного дольше, Визуализация будет выглядеть примерно так:
    
    :::image type="content" source="media/how-to-integrate-time-series-insights/day-data.png" alt-text="Данные температуры для каждого двойника отображаются в виде трех параллельных линий различных цветов.":::

## <a name="next-steps"></a>Дальнейшие действия

Цифровые двойников по умолчанию хранятся в виде плоской иерархии в службе "аналитика временных рядов", но их можно расширить с помощью сведений о модели и многоуровневого дерева для Организации. Дополнительные сведения об этом процессе см. в следующих статье: 

* [*Учебник. Определение и применение модели*](../time-series-insights/tutorial-set-up-environment.md#define-and-apply-a-model) 

Вы можете написать настраиваемую логику для автоматического предоставления этих сведений с помощью модели и данных графа, уже сохраненных в Azure Digital двойников. Дополнительные сведения об управлении, обновлении и извлечении данных из графа двойников см. в следующих статьях:

* [*Практические руководства. Управление цифровым двойника*](./how-to-manage-twin.md)
* [*Пошаговое руководство. запрос графа двойника*](./how-to-query-graph.md)