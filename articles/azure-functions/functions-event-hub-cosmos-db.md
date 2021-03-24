---
title: Руководство по использованию функций Java с Azure Cosmos DB и Центрами событий
description: В этом руководстве показано, как получать события из Центров событий для внесения обновлений в Azure Cosmos DB с помощью функции на языке Java.
author: KarlErickson
ms.topic: tutorial
ms.date: 11/04/2019
ms.author: karler
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 0d5d285812aa61cf9fe684b268dc07718f6d1a1d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101708753"
---
# <a name="tutorial-create-a-function-in-java-with-an-event-hub-trigger-and-an-azure-cosmos-db-output-binding"></a>Руководство по созданию функции на языке Java с триггером Центра событий и выходной привязкой Azure Cosmos DB

В этом руководстве показано, как с помощью Функций Azure создать функцию на языке Java, которая анализирует непрерывный поток данных о температуре и давлении. Функция активируется по событиям концентратора событий, которые передают показания датчика. Функция обрабатывает данные события и добавляет записи о состоянии в Azure Cosmos DB.

В этом руководстве вы выполните следующие действия.

> [!div class="checklist"]
> * Создание и настройка ресурсов Azure с помощью Azure CLI.
> * Создание и тестирование функций Java, которые взаимодействуют с этими ресурсами.
> * Развертывание функций в Azure и их отслеживание с помощью Application Insights.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для выполнения действий в этом руководстве необходимо установить следующие средства:

- [Java Developer Kit (JDK)](/azure/developer/java/fundamentals/java-jdk-long-term-support) версии 8.
- [Apache Maven](https://maven.apache.org) 3.0 или более поздней версии.
- [Azure Functions Core Tools](https://www.npmjs.com/package/azure-functions-core-tools) 2.6.666 или более поздней версии [!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)].

> [!IMPORTANT]
> В переменной среды `JAVA_HOME` следует сохранить расположение установки JDK, чтобы завершить работу с этим руководством.

Если вы предпочитаете сразу получить код для этого руководства, см. репозиторий примера [java-functions-eventhub-cosmosdb](https://github.com/Azure-Samples/java-functions-eventhub-cosmosdb).

## <a name="create-azure-resources"></a>Создание ресурсов Azure

Для этого руководства вам понадобятся следующие ресурсы:

* группа ресурсов, которая будет содержать остальные ресурсы;
* пространство имен Центров событий Azure, концентратор событий и правило авторизации;
* учетная запись, база данных и коллекция Cosmos DB;
* приложение-функция и учетная запись хранения для его размещения.

В следующих разделах показано, как создать эти ресурсы с помощью Azure CLI.

### <a name="set-environment-variables"></a>Настройка переменных среды

Теперь создайте несколько переменных среды для имен и расположения ресурсов, которые вы создадите. Добавьте следующие команды, заменив заполнители `<value>` значениями на свой выбор. Эти значения должны соответствовать [правилам и ограничениям именования для ресурсов Azure](/azure/architecture/best-practices/resource-naming). Для переменной `LOCATION` укажите одно из значений, созданных командой `az functionapp list-consumption-locations`.

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
RESOURCE_GROUP=<value>
EVENT_HUB_NAMESPACE=<value>
EVENT_HUB_NAME=<value>
EVENT_HUB_AUTHORIZATION_RULE=<value>
COSMOS_DB_ACCOUNT=<value>
STORAGE_ACCOUNT=<value>
FUNCTION_APP=<value>
LOCATION=<value>
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
set RESOURCE_GROUP=<value>
set EVENT_HUB_NAMESPACE=<value>
set EVENT_HUB_NAME=<value>
set EVENT_HUB_AUTHORIZATION_RULE=<value>
set COSMOS_DB_ACCOUNT=<value>
set STORAGE_ACCOUNT=<value>
set FUNCTION_APP=<value>
set LOCATION=<value>
```

---

Эти переменные используются далее в этом руководстве. Имейте в виду, что эти переменные сохраняются только в течение текущего сеанса Azure CLI или Cloud Shell. Вам придется выполнить эти команды снова, если вы перейдете в другое окно терминала или завершится время действия сеанса Cloud Shell.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Azure использует группы ресурсов для объединения всех связанных ресурсов учетной записи. Это позволяет рассматривать их как единое целое и удалять одной командой, когда они больше не требуются.

Выполните следующую команду, чтобы создать группу ресурсов.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az group create \
    --name $RESOURCE_GROUP \
    --location $LOCATION
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az group create ^
    --name %RESOURCE_GROUP% ^
    --location %LOCATION%
```

---

### <a name="create-an-event-hub"></a>Создание концентратора событий

Теперь создайте пространство имен Центров событий Azure, концентратор событий и правило авторизации, выполнив следующие команды:

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az eventhubs namespace create \
    --resource-group $RESOURCE_GROUP \
    --name $EVENT_HUB_NAMESPACE
az eventhubs eventhub create \
    --resource-group $RESOURCE_GROUP \
    --name $EVENT_HUB_NAME \
    --namespace-name $EVENT_HUB_NAMESPACE \
    --message-retention 1
az eventhubs eventhub authorization-rule create \
    --resource-group $RESOURCE_GROUP \
    --name $EVENT_HUB_AUTHORIZATION_RULE \
    --eventhub-name $EVENT_HUB_NAME \
    --namespace-name $EVENT_HUB_NAMESPACE \
    --rights Listen Send
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az eventhubs namespace create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %EVENT_HUB_NAMESPACE%
az eventhubs eventhub create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %EVENT_HUB_NAME% ^
    --namespace-name %EVENT_HUB_NAMESPACE% ^
    --message-retention 1
az eventhubs eventhub authorization-rule create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %EVENT_HUB_AUTHORIZATION_RULE% ^
    --eventhub-name %EVENT_HUB_NAME% ^
    --namespace-name %EVENT_HUB_NAMESPACE% ^
    --rights Listen Send
```

---

Пространство имен Центров событий содержит сам концентратор событий и правило авторизации для него. Правило авторизации позволяет функциям отправлять сообщения в концентратор и прослушивать нужные события. Одна функция отправляет сообщения, которые представляют данные телеметрии. Другая функция прослушивает события, анализирует данные этих событий и сохраняет результаты в Azure Cosmos DB.

### <a name="create-an-azure-cosmos-db"></a>Создание Azure Cosmos DB

Теперь создайте учетную запись Azure Cosmos DB, базу данных и коллекцию с помощью следующих команд:

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az cosmosdb create \
    --resource-group $RESOURCE_GROUP \
    --name $COSMOS_DB_ACCOUNT
az cosmosdb sql database create \
    --resource-group $RESOURCE_GROUP \
    --account-name $COSMOS_DB_ACCOUNT \
    --name TelemetryDb
az cosmosdb sql container create \
    --resource-group $RESOURCE_GROUP \
    --account-name $COSMOS_DB_ACCOUNT \
    --database-name TelemetryDb \
    --name TelemetryInfo \
    --partition-key-path '/temperatureStatus'
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az cosmosdb create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %COSMOS_DB_ACCOUNT%
az cosmosdb sql database create ^
    --resource-group %RESOURCE_GROUP% ^
    --account-name %COSMOS_DB_ACCOUNT% ^
    --name TelemetryDb
az cosmosdb sql container create ^
    --resource-group %RESOURCE_GROUP% ^
    --account-name %COSMOS_DB_ACCOUNT% ^
    --database-name TelemetryDb ^
    --name TelemetryInfo ^
    --partition-key-path "/temperatureStatus"
```

---

Значение `partition-key-path` разделяет данные по значению `temperatureStatus` каждого элемента. Ключ секции позволяет Cosmos DB повысить производительность, разделив данные на несколько подмножеств с независимым доступом.

### <a name="create-a-storage-account-and-function-app"></a>Создание учетной записи хранения для приложения-функции

Теперь создайте учетную запись хранения Azure, которая нужна для Функций Azure, а затем — приложение-функцию. Используйте следующие команды:

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az storage account create \
    --resource-group $RESOURCE_GROUP \
    --name $STORAGE_ACCOUNT \
    --sku Standard_LRS
az functionapp create \
    --resource-group $RESOURCE_GROUP \
    --name $FUNCTION_APP \
    --storage-account $STORAGE_ACCOUNT \
    --consumption-plan-location $LOCATION \
    --runtime java \
    --functions-version 2
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az storage account create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %STORAGE_ACCOUNT% ^
    --sku Standard_LRS
az functionapp create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %FUNCTION_APP% ^
    --storage-account %STORAGE_ACCOUNT% ^
    --consumption-plan-location %LOCATION% ^
    --runtime java ^
    --functions-version 2
```

---

Помимо приложения-функции, команда `az functionapp create` создает ресурс Application Insights с тем же именем. Для приложения-функции автоматически настраивается параметр `APPINSIGHTS_INSTRUMENTATIONKEY`, который подключает его к Application Insights. После развертывания функций в Azure вы сможете просмотреть данные телеметрии приложения, как описано далее в этом руководстве.

## <a name="configure-your-function-app"></a>Настройка приложения-функции

Для правильной работы приложению-функции потребуется доступ к другим ресурсам. В следующих разделах показано, как настроить приложение-функцию для работы на локальном компьютере.

### <a name="retrieve-resource-connection-strings"></a>Получение строк подключения к ресурсам

Используйте следующие команды, чтобы получить строки подключения к хранилищу, концентратору событий и Cosmos DB и сохранить их в переменных среды:

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
AZURE_WEB_JOBS_STORAGE=$( \
    az storage account show-connection-string \
        --name $STORAGE_ACCOUNT \
        --query connectionString \
        --output tsv)
echo $AZURE_WEB_JOBS_STORAGE
EVENT_HUB_CONNECTION_STRING=$( \
    az eventhubs eventhub authorization-rule keys list \
        --resource-group $RESOURCE_GROUP \
        --name $EVENT_HUB_AUTHORIZATION_RULE \
        --eventhub-name $EVENT_HUB_NAME \
        --namespace-name $EVENT_HUB_NAMESPACE \
        --query primaryConnectionString \
        --output tsv)
echo $EVENT_HUB_CONNECTION_STRING
COSMOS_DB_CONNECTION_STRING=$( \
    az cosmosdb keys list \
        --resource-group $RESOURCE_GROUP \
        --name $COSMOS_DB_ACCOUNT \
        --type connection-strings \
        --query 'connectionStrings[0].connectionString' \
        --output tsv)
echo $COSMOS_DB_CONNECTION_STRING
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
FOR /F "delims=" %X IN (' ^
    az storage account show-connection-string ^
        --name %STORAGE_ACCOUNT% ^
        --query connectionString ^
        --output tsv') DO SET AZURE_WEB_JOBS_STORAGE=%X
FOR /F "delims=" %X IN (' ^
    az eventhubs eventhub authorization-rule keys list ^
        --resource-group %RESOURCE_GROUP% ^
        --name %EVENT_HUB_AUTHORIZATION_RULE% ^
        --eventhub-name %EVENT_HUB_NAME% ^
        --namespace-name %EVENT_HUB_NAMESPACE% ^
        --query primaryConnectionString ^
        --output tsv') DO SET EVENT_HUB_CONNECTION_STRING=%X
FOR /F "delims=" %X IN (' ^
    az cosmosdb keys list ^
        --resource-group %RESOURCE_GROUP% ^
        --name %COSMOS_DB_ACCOUNT% ^
        --type connection-strings ^
        --query connectionStrings[0].connectionString ^
        --output tsv') DO SET COSMOS_DB_CONNECTION_STRING=%X
```

---

Для этих переменных задаются значения, полученные из команд Azure CLI. Каждая из этих команд выполняет запрос JMESPath, извлекая строку подключения из возвращаемых полезных данных JSON. Строки подключения также можно отобразить с помощью `echo` и убедиться, что они успешно извлечены.

### <a name="update-your-function-app-settings"></a>Обновление параметров приложения-функции

Затем используйте следующую команду для перемещения значений строки подключения в параметры приложения в учетной записи Функций Azure:

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az functionapp config appsettings set \
    --resource-group $RESOURCE_GROUP \
    --name $FUNCTION_APP \
    --settings \
        AzureWebJobsStorage=$AZURE_WEB_JOBS_STORAGE \
        EventHubConnectionString=$EVENT_HUB_CONNECTION_STRING \
        CosmosDBConnectionString=$COSMOS_DB_CONNECTION_STRING
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az functionapp config appsettings set ^
    --resource-group %RESOURCE_GROUP% ^
    --name %FUNCTION_APP% ^
    --settings ^
        AzureWebJobsStorage=%AZURE_WEB_JOBS_STORAGE% ^
        EventHubConnectionString=%EVENT_HUB_CONNECTION_STRING% ^
        CosmosDBConnectionString=%COSMOS_DB_CONNECTION_STRING%
```

---

Теперь ресурсы Azure успешно созданы и правильно настроены для совместной работы.

## <a name="create-and-test-your-functions"></a>Создание и тестирование функций

Далее вам предстоит создать проект на локальном компьютере, добавить в него код Java и протестировать его. Вы будете использовать команды, которые работают с подключаемым модулем Maven для Функций Azure и Azure Functions Core Tools. Ваши функции выполняются локально, но используют только что созданные облачные ресурсы. Когда функции будут правильно работать локально, с помощью Maven вы сможете развернуть их в облаке и наблюдать, как накапливаются статистические и аналитические данные.

Если вы использовали для создания ресурсов Cloud Shell, у вас пока нет подключения к Azure из локальной среды. В этом случае выполните команду `az login`, чтобы запустить процесс входа в браузере. Если потребуется, задайте подписку по умолчанию, указав `az account set --subscription`, а затем идентификатор подписки. Наконец, выполните следующие команды, чтобы восстановить переменные среды на локальном компьютере. Замените заполнители `<value>` теми же значениями, которые вы использовали ранее.

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
RESOURCE_GROUP=<value>
FUNCTION_APP=<value>
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
set RESOURCE_GROUP=<value>
set FUNCTION_APP=<value>
```

---

### <a name="create-a-local-functions-project"></a>Создание локального проекта службы "Функции"

Используйте следующую команду Maven, чтобы создать проект функций и добавить необходимые зависимости.

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
mvn archetype:generate --batch-mode \
    -DarchetypeGroupId=com.microsoft.azure \
    -DarchetypeArtifactId=azure-functions-archetype \
    -DappName=$FUNCTION_APP \
    -DresourceGroup=$RESOURCE_GROUP \
    -DappRegion=$LOCATION \
    -DgroupId=com.example \
    -DartifactId=telemetry-functions
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
mvn archetype:generate --batch-mode ^
    -DarchetypeGroupId=com.microsoft.azure ^
    -DarchetypeArtifactId=azure-functions-archetype ^
    -DappName=%FUNCTION_APP% ^
    -DresourceGroup=%RESOURCE_GROUP% ^
    -DappRegion=%LOCATION% ^
    -DgroupId=com.example ^
    -DartifactId=telemetry-functions
```

---

Эта команда создает несколько файлов в папке `telemetry-functions`:

* файл `pom.xml` для использования с Maven;
* файл `local.settings.json` для хранения параметров приложения для локального тестирования;
* файл `host.json`, который включает пакет расширений Функций Azure, необходимый для выходной привязки к Cosmos DB в функции анализа данных;
* файл `Function.java`, который содержит стандартную реализацию функции;
* несколько тестовых файлов, которые не требуются для этого руководства.

Чтобы избежать ошибок компиляции, удалите эти тестовые файлы. Выполните следующие команды, чтобы перейти к новой папке проекта и удалить тестовую папку:

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
cd telemetry-functions
rm -r src/test
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
cd telemetry-functions
rmdir /s /q src\test
```

---

### <a name="retrieve-your-function-app-settings-for-local-use"></a>Получение параметров приложения-функции для локального использования

Для локального тестирования вашему проекту функции нужны строки подключения, которые вы добавили в приложение-функцию Azure ранее в этом руководстве. Используйте следующую команду Azure Functions Core Tools, которая извлекает все сохраненные в облаке параметры приложения-функции и добавляет их в файл `local.settings.json`:

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
func azure functionapp fetch-app-settings $FUNCTION_APP
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
func azure functionapp fetch-app-settings %FUNCTION_APP%
```

---

### <a name="add-java-code"></a>Добавление кода Java

Теперь откройте файл `Function.java` и замените его содержимое следующим кодом.

```java
package com.example;

import com.example.TelemetryItem.status;
import com.microsoft.azure.functions.annotation.Cardinality;
import com.microsoft.azure.functions.annotation.CosmosDBOutput;
import com.microsoft.azure.functions.annotation.EventHubOutput;
import com.microsoft.azure.functions.annotation.EventHubTrigger;
import com.microsoft.azure.functions.annotation.FunctionName;
import com.microsoft.azure.functions.annotation.TimerTrigger;
import com.microsoft.azure.functions.ExecutionContext;
import com.microsoft.azure.functions.OutputBinding;

public class Function {

    @FunctionName("generateSensorData")
    @EventHubOutput(
        name = "event",
        eventHubName = "", // blank because the value is included in the connection string
        connection = "EventHubConnectionString")
    public TelemetryItem generateSensorData(
        @TimerTrigger(
            name = "timerInfo",
            schedule = "*/10 * * * * *") // every 10 seconds
            String timerInfo,
        final ExecutionContext context) {

        context.getLogger().info("Java Timer trigger function executed at: "
            + java.time.LocalDateTime.now());
        double temperature = Math.random() * 100;
        double pressure = Math.random() * 50;
        return new TelemetryItem(temperature, pressure);
    }

    @FunctionName("processSensorData")
    public void processSensorData(
        @EventHubTrigger(
            name = "msg",
            eventHubName = "", // blank because the value is included in the connection string
            cardinality = Cardinality.ONE,
            connection = "EventHubConnectionString")
            TelemetryItem item,
        @CosmosDBOutput(
            name = "databaseOutput",
            databaseName = "TelemetryDb",
            collectionName = "TelemetryInfo",
            connectionStringSetting = "CosmosDBConnectionString")
            OutputBinding<TelemetryItem> document,
        final ExecutionContext context) {

        context.getLogger().info("Event hub message received: " + item.toString());

        if (item.getPressure() > 30) {
            item.setNormalPressure(false);
        } else {
            item.setNormalPressure(true);
        }

        if (item.getTemperature() < 40) {
            item.setTemperatureStatus(status.COOL);
        } else if (item.getTemperature() > 90) {
            item.setTemperatureStatus(status.HOT);
        } else {
            item.setTemperatureStatus(status.WARM);
        }

        document.setValue(item);
    }
}
```

Как вы видите, этот файл содержит две функции: `generateSensorData` и `processSensorData`. Функция `generateSensorData` имитирует датчик, который отправляет показания температуры и давления в концентратор событий. Триггер таймера запускает эту функцию каждые 10 секунд, а выходная привязка концентратора событий отправляет возвращаемое значение в концентратор событий.

Когда концентратор событий получает сообщение, он создает событие. При получении события выполняется функция `processSensorData`. Она обрабатывает данные события и использует выходную привязку Azure Cosmos DB для отправки результатов в Azure Cosmos DB.

Данные, которые используются этими функциями, хранятся в классе с именем `TelemetryItem`, который вам нужно реализовать. Создайте файл с именем `TelemetryItem.java` в том же расположении, что и `Function.java`, и добавьте в него следующий код:

```java
package com.example;

public class TelemetryItem {

    private String id;
    private double temperature;
    private double pressure;
    private boolean isNormalPressure;
    private status temperatureStatus;
    static enum status {
        COOL,
        WARM,
        HOT
    }

    public TelemetryItem(double temperature, double pressure) {
        this.temperature = temperature;
        this.pressure = pressure;
    }

    public String getId() {
        return id;
    }

    public double getTemperature() {
        return temperature;
    }

    public double getPressure() {
        return pressure;
    }

    @Override
    public String toString() {
        return "TelemetryItem={id=" + id + ",temperature="
            + temperature + ",pressure=" + pressure + "}";
    }

    public boolean isNormalPressure() {
        return isNormalPressure;
    }

    public void setNormalPressure(boolean isNormal) {
        this.isNormalPressure = isNormal;
    }

    public status getTemperatureStatus() {
        return temperatureStatus;
    }

    public void setTemperatureStatus(status temperatureStatus) {
        this.temperatureStatus = temperatureStatus;
    }
}
```

### <a name="run-locally"></a>Локальный запуск

Теперь вы можете создавать и запускать локально функции, данные из которых будут отображаться в Azure Cosmos DB.

Выполните следующие команды Maven для сборки и запуска функций:

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
mvn clean package
mvn azure-functions:run
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
mvn clean package
mvn azure-functions:run
```

---

После нескольких сообщений о сборке и запуске вы увидите для каждого запуска функций выходные данные, аналогичные приведенным ниже:

```output
[10/22/19 4:01:30 AM] Executing 'Functions.generateSensorData' (Reason='Timer fired at 2019-10-21T21:01:30.0016769-07:00', Id=c1927c7f-4f70-4a78-83eb-bc077d838410)
[10/22/19 4:01:30 AM] Java Timer trigger function executed at: 2019-10-21T21:01:30.015
[10/22/19 4:01:30 AM] Function "generateSensorData" (Id: c1927c7f-4f70-4a78-83eb-bc077d838410) invoked by Java Worker
[10/22/19 4:01:30 AM] Executed 'Functions.generateSensorData' (Succeeded, Id=c1927c7f-4f70-4a78-83eb-bc077d838410)
[10/22/19 4:01:30 AM] Executing 'Functions.processSensorData' (Reason='', Id=f4c3b4d7-9576-45d0-9c6e-85646bb52122)
[10/22/19 4:01:30 AM] Event hub message received: TelemetryItem={id=null,temperature=32.728691307527015,pressure=10.122563042388165}
[10/22/19 4:01:30 AM] Function "processSensorData" (Id: f4c3b4d7-9576-45d0-9c6e-85646bb52122) invoked by Java Worker
[10/22/19 4:01:38 AM] Executed 'Functions.processSensorData' (Succeeded, Id=1cf0382b-0c98-4cc8-9240-ee2a2f71800d)
```

Теперь вы можете перейти на [портал Azure](https://portal.azure.com) и в учетную запись Azure Cosmos DB. Выберите **Data Explorer**, разверните узел **TelemetryInfo** и щелкните **Элементы**, чтобы просмотреть поступающие данные.

![Обозреватель данных Cosmos DB](media/functions-event-hub-cosmos-db/data-explorer.png)

## <a name="deploy-to-azure-and-view-app-telemetry"></a>Развертывание в Azure и просмотр телеметрии приложений

И наконец, вы можете развернуть приложение в Azure и убедиться, что оно будет работать так же, как в локальной среде.

Разверните проект в Azure, используя приведенную ниже команду.

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
mvn azure-functions:deploy
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
mvn azure-functions:deploy
```

---

Теперь ваши функции выполняются в Azure и продолжают накапливать данные в Azure Cosmos DB. Развернутое приложение-функцию можно просмотреть на портале Azure, а данные телеметрии приложений — через подключенный ресурс Application Insights, как показано на следующих снимках экрана.

**Live Metrics Stream:**

![Live Metrics Stream для Application Insights](media/functions-event-hub-cosmos-db/application-insights-live-metrics-stream.png)

**Производительность.**

![Колонка производительности в Application Insights](media/functions-event-hub-cosmos-db/application-insights-performance.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

По завершении работы с ресурсами Azure, созданными в этом руководстве, их можно удалить с помощью следующей команды:

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az group delete --name $RESOURCE_GROUP
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az group delete --name %RESOURCE_GROUP%
```

---

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как создать функцию Azure, которая обрабатывает события Центра событий и обновляет Cosmos DB. Подробные сведения см. в [руководстве разработчика Java по Функциям Azure](./functions-reference-java.md). Сведения об используемых аннотациях см. в справочнике по [com.microsoft.azure.functions.annotation](/java/api/com.microsoft.azure.functions.annotation).

В этом руководстве переменные среды и параметры приложения используются для хранения секретов, таких как строки подключения. Сведения о хранении этих секретов в Azure Key Vault см. в статье об [использовании ссылок на Key Vault для Службы приложений и Функций Azure](../app-service/app-service-key-vault-references.md).

Далее вы узнаете, как использовать конвейеры CI/CD в Azure Pipelines для автоматического развертывания:

> [!div class="nextstepaction"]
> [Создание и развертывание приложений Java в Функциях Azure](/azure/devops/pipelines/ecosystems/java-function)
