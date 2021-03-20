---
title: Использование буферов протоколов для имитации устройства в Azure | Документация Майкрософт
description: В этом руководстве вы узнаете, как использовать буферы протоколов для сериализации телеметрии, отправляемой из акселератора решений для имитации устройств.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.custom: mvc, amqp, devx-track-csharp
ms.date: 11/06/2018
ms.author: dobett
ms.openlocfilehash: 77a795b8f704084b612a2dcbd364b8d5be0a3333
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96852156"
---
# <a name="serialize-telemetry-using-protocol-buffers"></a>Сериализация данных телеметрии с помощью буферов протокола

Буферы протокола (Protobuf) — это формат двоичной сериализации для структурированных данных. Protobuf призван подчеркнуть простоту и производительность с целью быть меньше и быстрее, чем XML.

Симулятор устройств поддерживает версию **proto3** протокола буфера языка.

Так как в Protobuf для сериализации данных требуется скомпилированный код, необходимо создать пользовательскую версию для имитации устройств.

В этом практическом руководстве описаны следующие процедуры:

1. Подготовка среды разработки.
1. указать использование формата Protobuf в модели устройства;
1. Определение своего формата Protobuf
1. формировать классы Protobuf.
1. Локальное тестирование

## <a name="prerequisites"></a>Предварительные условия

Выполните следующие шаги для изучения данного руководства:

* Visual Studio Code. Вы можете загрузить [Visual Studio Code для Mac, Linux и Windows](https://code.visualstudio.com/download).
* .NET Core. Вы можете скачать [.NET Core для Mac, Linux и Windows](https://www.microsoft.com/net/download).
* Postman. Вы можете загрузить [POST для Mac, Windows или Linux](https://www.getpostman.com/apps).
* [Центр Интернета вещей для вашей подписки Azure](../iot-hub/iot-hub-create-through-portal.md). Для выполнения шагов этого руководства вам понадобится строка подключения Центра Интернета вещей. Вы можете получить строку подключения с портала Azure.
* [База данных Cosmos DB, развернутая в подписке Azure](../cosmos-db/create-sql-api-dotnet.md#create-account), которая использует SQL API и которая настроена на [строгую согласованность](../cosmos-db/how-to-manage-database-account.md). Для выполнения шагов этого руководства вам понадобится строка подключения базы данных Cosmos DB. Вы можете получить строку подключения с портала Azure.
* [Учетная запись хранения Azure, развернутая в подписке Azure.](../storage/common/storage-account-create.md) Для выполнения шагов этого руководства вам понадобится строка подключения учетной записи хранения. Вы можете получить строку подключения с портала Azure.

## <a name="prepare-your-development-environment"></a>Подготовка среды разработки

Для подготовки среды разработки выполните следующие действия:

* загрузите исходный код для микрослужбы моделирования устройств;
* загрузите исходный код для микрослужбы адаптера хранилища;
* запустите микрослужбу адаптера хранилища локально.

В инструкциях этой статьи предполагается, что вы используете Windows. Если вы используете другую операционную систему, может потребоваться изменить некоторые пути к файлам и команды в соответствии с требованиями вашей среды.

### <a name="download-the-microservices"></a>Загрузка микрослужб

Скачайте и распакуйте [микрослужбы удаленного мониторинга](https://github.com/Azure/remote-monitoring-services-dotnet/archive/master.zip) из GitHub в подходящее место на локальном компьютере. Этот репозиторий включает в себя микрослужбы адаптера хранилища, необходимые для этого практического руководства.

Скачайте и распакуйте [микрослужбу моделирования устройств](https://github.com/Azure/device-simulation-dotnet/archive/master.zip) из GitHub в подходящее место на вашем локальном компьютере.

### <a name="run-the-storage-adapter-microservice"></a>Запуск микрослужбы адаптера хранилища

Откройте папку **remote-monitoring-services-dotnet-master\storage-adapter** в Visual Studio Code. Чтобы исправить любые нерешенные зависимости, нажмите кнопку **Восстановить**.

Откройте файл **. vscode/launch.jsв** файле и назначьте строку подключения Cosmos DB для переменной среды **\_ сторажеадаптер \_ DOCUMENTDB \_ CONNSTRING** .

> [!NOTE]
> При запуске микрослужбы локально на компьютере ей по прежнему требуется экземпляр Cosmos DB в Azure для правильной работы.

Чтобы запустить микрослужбу адаптера хранилища локально, щелкните **Отладка \> Начать отладку**.

Окно **Терминал** в Visual Studio Code отображает выходные данные из работающей микрослужбы, включая URL-адрес проверки работоспособности веб-службы: <http://127.0.0.1:9022/v1/status>. При переходе на этот адрес, состояние должно быть "OK: живой и отлично".

Оставьте микрослужбу адаптера хранилища запущенной в данном экземпляре Visual Studio Code, пока вы выполняете следующие действия.

## <a name="define-your-device-model"></a>Определение модели устройства

Откройте папку **​​device-simulation-dotnet-master**, загруженную с GitHub, в новом экземпляре Visual Studio Code. Чтобы исправить любые нерешенные зависимости, щелкните любую кнопку **Восстановить**.

В этом практическом руководстве вы создадите модель устройства для отслеживания ресурсов:

1. Создайте новый файл модели устройства с именем **assettracker-01.json** в папке **Services\data\devicemodels**.

1. Определите функциональные возможности устройства в файле модели устройства **assettracker-01.json**. В разделе телеметрии модели устройства Protobuf сделайте следующее:

   * Включите имя класса Protobuf, создаваемое для устройства. В разделе ниже показано, как создать этот класс.
   * Укажите Protobuf в качестве формата сообщений.

     ```json
     {
     "SchemaVersion": "1.0.0",
     "Id": "assettracker-01",
     "Version": "0.0.1",
     "Name": "Asset Tracker",
     "Description": "An asset tracker with location, temperature, and humidity",
     "Protocol": "AMQP",
     "Simulation": {
       "InitialState": {
         "online": true,
         "latitude": 47.445301,
         "longitude": -122.296307,
         "temperature": 38.0,
         "humidity": 62.0
       },
       "Interval": "00:01:00",
       "Scripts": [
         {
           "Type": "javascript",
           "Path": "assettracker-01-state.js"
         }
       ]
     },
     "Properties": {
       "Type": "AssetTracker",
       "Location": "Field",
       "Latitude": 47.445301,
       "Longitude": -122.296307
     },
     "Telemetry": [
       {
         "Interval": "00:00:10",
         "MessageTemplate": "{\"latitude\":${latitude},\"longitude\":${longitude},\"temperature\":${temperature},\"humidity\":${humidity}}",
         "MessageSchema": {
           "Name": "assettracker-sensors;v1",
           "ClassName": "Microsoft.Azure.IoTSolutions.DeviceSimulation.Services.Models.Protobuf.AssetTracker",
           "Format": "Protobuf",
           "Fields": {
             "latitude": "double",
             "longitude": "double",
             "temperature": "double",
             "humidity": "double"
           }
         }
       }
     ]
     }
     ```

### <a name="create-device-behaviors-script"></a>Создание скрипта поведения устройства

Запишите скрипт поведения, который определяет поведение устройства. Дополнительные сведения см. в статье [Create an advanced device model](iot-accelerators-device-simulation-advanced-device.md) (Создание расширенной модели устройства).

## <a name="define-your-protobuf-format"></a>Определение своего формата Protobuf

Если имеется модель устройства и определен формат сообщений, можно создать файл **proto**. В файле **proto** можно добавить:

* Объект `csharp_namespace`, соответствующий свойству **ClassName** в модели устройства.
* Сообщение каждой структуры данных для сериализации.
* Имя и тип для каждого поля в сообщении.

1. Создайте файл с именем **assettracker.proto** в папке **Services\Models\Protobuf\proto**.

1. Определите синтаксис, пространство имен и схему сообщения в файле **proto** следующим образом:

    ```proto
    syntax = "proto3";

    option csharp_namespace = "Microsoft.Azure.IoTSolutions.DeviceSimulation.Services.Models.Protobuf";

    message AssetTracker {
      double latitude=1;
      double longitude=2;
      double temperature=5;
      double humidity=6;
    }
    ```

Маркеры `=1`, `=2` для каждого элемента указывают уникальный тег, который использует поле в двоичной кодировке. Числовым значениям 1–15 требуется меньше одного байта для кодировки более высоких числовых значений.

## <a name="generate-the-protobuf-class"></a>Создание классов Protobuf

При наличии файла **proto** следующим шагом является создание классов, необходимых для чтения и записи сообщений. Для выполнения этого шага необходим компилятор Protobuf **Protoc**.

1. [Компилятор Protobuf можно скачать с GitHub](https://github.com/protocolbuffers/protobuf/releases/download/v3.4.0/protoc-3.4.0-win32.zip)

1. Запустите компилятор, указав исходный каталог, целевой каталог и имя файла **proto**. Пример:

    ```cmd
    protoc -I c:\temp\device-simulation-dotnet-master\Services\Models\Protobuf\proto --csharp_out=C:\temp\device-simulation-dotnet-master\Services\Models\Protobuf assettracker.proto
    ```

    Эта команда создает файл **Assettracker.cs** в папке **Services\Models\Protobuf**.

## <a name="test-protobuf-locally"></a>Тестирование локально

В этом разделе вы тестируете устройство отслеживания ресурсов, созданное локально в предыдущих разделах.

### <a name="run-the-device-simulation-microservice"></a>Запуск микрослужбы моделирования устройств

Откройте файл **.vscode/launch.json** и назначьте ваш:

* Строка подключения Центра Интернета вещей для переменной среды **PCS\_IOTHUB\_CONNSTRING**.
* Строка подключения учетной записи хранения для переменной среды **PCS\_AZURE\_ХРАНИЛИЩЕ\_УЧЕТНАЯ ЗАПИСЬ**.
* Назначьте строку подключения Cosmos DB для переменной среды **PCS\_STORAGEADAPTER\_DOCUMENTDB\_CONNSTRING**.

Откройте файл **WebService/Properties/launchSettings.json** и назначьте свою:

* Строка подключения Центра Интернета вещей для переменной среды **PCS\_IOTHUB\_CONNSTRING**.
* Строка подключения учетной записи хранения для переменной среды **PCS\_AZURE\_ХРАНИЛИЩЕ\_УЧЕТНАЯ ЗАПИСЬ**.
* Назначьте строку подключения Cosmos DB для переменной среды **PCS\_STORAGEADAPTER\_DOCUMENTDB\_CONNSTRING**.

Откройте файл **WebService\appsettings.ini** и измените параметры следующим образом:

#### <a name="configure-the-solution-to-include-your-new-device-model-files"></a>Настройка решения для включения файлов новой модели устройства

По умолчанию новые файлы JSON и JS модели устройства не копируются в созданное решение. Необходимо явно включить их.

Добавьте запись в файл **services\services.csproj** для каждого файла, который требуется включить. Пример:

```xml
<None Update="data\devicemodels\assettracker-01.json">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
</None>
<None Update="data\devicemodels\scripts\assettracker-01-state.js">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
</None>
```

Чтобы запустить микрослужбу локально, выберите **Отладка \> Начать отладку**.

Окно **Терминал** в Visual Studio Code отображает выходные данные из работающей микрослужбы.

Оставьте микрослужбу моделирования устройств запущенной в данном экземпляре Visual Studio Code, пока вы выполняете следующие действия.

### <a name="set-up-a-monitor-for-device-events"></a>Настройка монитора для событий устройства

В этом разделе вы используете Azure CLI, чтобы настроить монитор событий для просмотра телеметрии, отправленной с устройств, подключенных к Центру Интернета вещей.

В следующем сценарии предполагается, что имя вашего Центра Интернета вещей — **device-simulation-test**.

```azurecli-interactive
# Install the IoT extension if it's not already installed
az extension add --name azure-iot

# Monitor telemetry sent to your hub
az iot hub monitor-events --hub-name device-simulation-test
```

Оставьте монитор событий включенным во время тестирования имитируемых устройств.

### <a name="create-a-simulation-with-the-asset-tracker-device-type"></a>Создание имитации с типом устройства для отслеживания ресурсов

В этом разделе вы используете средство Postman для запроса микрослужбы моделирования устройства, чтобы запустить имитацию с использованием типа устройства для отслеживания ресурсов. Postman — это средство, которое позволяет отправлять запросы REST на веб-службу.

Действия для настройки Postman.

1. Откройте Postman на локальном компьютере.

1. Выберите **Файл \> Импорт**. Затем нажмите **Выбор файлов**.

1. Выберите **решение "моделирование устройств Интернета вещей Azure". Опубликуйте \_ коллекцию** и **решение "моделирование устройств Azure IOT \_** ". После этого выполните команду " **Открыть**".

1. Разверните **Акселератор решений имитации устройств Azure loT** для просмотра запросов, которые вы можете отправить.

1. Щелкните **нет среды** и выберите средство **моделирования устройств Интернета вещей Azure**.

Теперь у вас есть коллекция и среда, загруженные в рабочее пространство Postman, которые вы можете использовать для взаимодействия с микрослужбой моделирования устройств.

Настройка и запуск имитации.

1. В коллекции Postman выберите **Create asset tracker simulation** (Создать имитацию отслеживания ресурс-контейнера) и нажмите **Отправить**. Этот запрос создает четыре экземпляра типов устройств для отслеживания ресурсов.

1. Выход монитора событий в окне Azure CLI показывает телеметрию с имитируемых устройств.

Чтобы остановить имитацию, выберите запрос в Postman **Остановить имитацию** и нажмите **Отправка**.

### <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете остановить две локально работающие микрослужбы в своих экземплярах Visual Studio Code (**Отладка \> Остановить отладку**).

Если вы больше не нуждаетесь в экземплярах Центра Интернета вещей и Cosmos DB, удалите их из подписки Azure, чтобы избежать ненужных расходов.

## <a name="iot-hub-support"></a>Поддержка Центра Интернета вещей

Многие функции Центра Интернета вещей изначально не поддерживают Protobuf или другие двоичные форматы. Например, нельзя настроить маршрутизацию в зависимости от полезных данных сообщения, так как Центр Интернета вещей не может обработать их. Однако данные можно маршрутизировать на основе заголовков сообщений.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как настроить эмуляцию устройств для использования protobuf для отправки данных телеметрии, следующий шаг — посетить репозиторий GitHub, чтобы узнать больше о [моделировании устройств](https://github.com/Azure/device-simulation-dotnet).
