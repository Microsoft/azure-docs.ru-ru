---
title: Установка и разработка локально с помощью эмулятора Azure Cosmos DB
description: Узнайте, как установить и использовать эмулятор Azure Cosmos DB в средах Windows, Linux, macOS и Windows DOCKER. С помощью эмулятора можно локально разрабатывать и тестировать приложение, не создавая подписку Azure.
ms.service: cosmos-db
ms.topic: how-to
author: markjbrown
ms.author: mjbrown
ms.date: 09/22/2020
ms.custom: devx-track-csharp, contperf-fy21q1
ms.openlocfilehash: 89e82e210dbf7c404ebd5978d87bf391d8565ee2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97033975"
---
# <a name="install-and-use-the-azure-cosmos-db-emulator-for-local-development-and-testing"></a>Установка и использование эмулятора Azure Cosmos DB для локальной разработки и тестирования
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Эмулятор Azure Cosmos DB предоставляет локальную среду для разработки, которая эмулирует службу Azure Cosmos DB. С помощью эмулятора Azure Cosmos DB вы можете локально разрабатывать и тестировать приложения, не создавая подписку Azure и не тратя средства. Если вы удовлетворены работой приложения в эмуляторе Azure Cosmos DB, вы можете переключиться на использование учетной записи Azure Cosmos в облаке. В этой статье описывается, как установить и использовать эмулятор в средах Windows, Linux, macOS и Windows DOCKER.

## <a name="download-the-emulator"></a>Скачать эмулятор

Чтобы приступить к работе, скачайте и установите последнюю версию эмулятора Azure Cosmos DB на локальном компьютере. В статье [заметки о выпуске эмулятора](local-emulator-release-notes.md) перечислены все доступные версии и обновления компонентов, которые были сделаны в каждом выпуске.

:::image type="icon" source="media/local-emulator/download-icon.png" border="false":::**[Скачивание эмулятора Azure Cosmos DB](https://aka.ms/cosmosdb-emulator)**

Вы можете разрабатывать приложения с помощью Azure Cosmos DB Emulator с учетными записями [SQL](local-emulator.md#sql-api), [Cassandra](local-emulator.md#cassandra-api), [MongoDB](local-emulator.md#azure-cosmos-dbs-api-for-mongodb), [Gremlin](local-emulator.md#gremlin-api)и API [таблиц](local-emulator.md#table-api) . В настоящее время обозреватель данных в эмуляторе полностью поддерживает просмотр только данных SQL; данные, созданные с помощью клиентских приложений MongoDB, Gremlin/Graph и Cassandra, в настоящее время недоступны для просмотра. Дополнительные сведения см. в разделе [Подключение к конечной точке эмулятора](#connect-with-emulator-apis) из разных интерфейсов API.

## <a name="how-does-the-emulator-work"></a>Как работает эмулятор?

Эмулятор Azure Cosmos DB обеспечивает высокоточную эмуляцию службы Azure Cosmos DB. Он поддерживает эквивалентные функциональные возможности, такие как Azure Cosmos DB, включая создание данных, запрос данных, подготовку и масштабирование контейнеров, а также выполнение хранимых процедур и триггеров. Вы можете разрабатывать и тестировать приложения с помощью эмулятора Azure Cosmos DB и развертывать их в Azure в глобальном масштабе, обновив конечную точку подключения Azure Cosmos DB.

Хотя эмуляция службы Azure Cosmos DB является точной, реализация эмулятора отличается от службы. Например, эмулятор использует стандартные компоненты операционной системы: локальную файловую систему для сохранения данных и стек протокола HTTPS для подключений. Функциональные возможности, основанные на инфраструктуре Azure, такие как Глобальная репликация, задержка в однозначной миллисекунде для операций чтения и записи, а также настраиваемые уровни согласованности, не применяются при использовании эмулятора.

Вы можете перенести данные между эмулятором Azure Cosmos DB и Azure Cosmos DBной службой с помощью [Azure Cosmos DB средства переноса данных](https://github.com/azure/azure-documentdb-datamigrationtool).

## <a name="differences-between-the-emulator-and-the-cloud-service"></a>Различия между эмулятором и облачной службой

Поскольку эмулятор Azure Cosmos DB предоставляет имитируемую среду, которая работает на локальной рабочей станции разработчика, существуют некоторые различия между эмулятором и учетной записью Azure Cosmos в облаке.

* В настоящее время область **Обозреватель данных** в эмуляторе полностью поддерживает только клиенты API SQL. **Обозреватель данных** представление и операции для интерфейсов API Azure Cosmos DB, таких как MongoDB, Table, Graph и Cassandra, не полностью поддерживаются.

* Эмулятор поддерживает только одну фиксированную учетную запись и известный первичный ключ. Нельзя повторно создать ключ при использовании эмулятора Azure Cosmos DB, однако ключ по умолчанию можно изменить с помощью параметра [командной строки](emulator-command-line-parameters.md) .

* С помощью эмулятора вы можете создать учетную запись Azure Cosmos в режиме [подготовленной пропускной способности](set-throughput.md) . Сейчас он не поддерживает [бессерверный](serverless.md) режим.

* Эмулятор не является масштабируемым сервисом и не поддерживает большое количество контейнеров. При использовании эмулятора Azure Cosmos DB по умолчанию можно создать до 25 контейнеров фиксированного размера в 400 единиц запросов в секунду (поддерживается только с помощью Azure Cosmos DB пакетов SDK) или 5 неограниченных контейнеров. Дополнительные сведения о том, как изменить это значение, см. в статье [Установка значения PartitionCount](emulator-command-line-parameters.md#set-partitioncount) .

* Эмулятор не предлагает различные [уровни согласованности Azure Cosmos DB](consistency-levels.md) , такие как облачная служба.

* Эмулятор не поддерживает [репликацию в нескольких регионах](distribute-data-globally.md).

* Поскольку копия эмулятора Azure Cosmos DB может не всегда быть актуальной с самыми последними изменениями в службе Azure Cosmos DB, следует всегда обращаться к [планировщику ресурсов Azure Cosmos DB](estimate-ru-with-capacity-planner.md) , чтобы точно оценить требования к пропускной способности приложения.

* Эмулятор поддерживает максимальный размер свойства идентификатора (254 символов).

## <a name="install-the-emulator"></a>Установка эмулятора

Перед установкой эмулятора убедитесь в наличии следующих требований к оборудованию и программному обеспечению.

* Требования к программному обеспечению:
  * В настоящее время поддерживаются ОС Windows Server 2016, 2019 или Windows 10. ОС узла с включенным Active Directory в настоящее время не поддерживается.
  * 64-разрядная операционная система.

* Минимальные требования к оборудованию:
  * 2 ГБ ОЗУ
  * 10 ГБ свободного дискового пространства

* Чтобы установить, настроить и запустить эмулятор Azure Cosmos DB, нужны права администратора на локальном компьютере. Эмулятор добавит сертификат, а также правила брандмауэра, чтобы запустить его службы. Поэтому для выполнения таких операций эмулятору необходимо иметь права администратора.

Чтобы приступить к работе, скачайте и установите последнюю версию [эмулятора Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) на локальном компьютере. Если при установке эмулятора возникнут проблемы, см. статью [Устранение неполадок эмулятора](troubleshoot-local-emulator.md) для отладки.

В зависимости от требований к системе можно запустить эмулятор в [Windows](#run-on-windows), [DOCKER для Windows](#run-on-windows-docker), [Linux или macOS](#run-on-linux-macos) , как описано в следующих разделах этой статьи.

## <a name="check-for-emulator-updates"></a>Проверить наличие обновлений для эмулятора

Каждая версия эмулятора поставляется с набором обновлений компонентов или исправлений ошибок. Сведения о доступных версиях см. в статье [заметки о выпуске эмулятора](local-emulator-release-notes.md) .

Если после установки использовались параметры по умолчанию, данные, соответствующие эмулятору, сохраняются в расположении%Локалаппдата%\космосдбемулатор. Можно настроить другое расположение, используя необязательные параметры пути к данным. Это является `/DataPath=PREFERRED_LOCATION` [параметром командной строки](emulator-command-line-parameters.md). Данные, созданные в одной версии эмулятора Azure Cosmos DB, не обязательно будут доступны при использовании другой версии. Если необходимо сохранить данные в долгосрочной перспективе, рекомендуется хранить их в учетной записи Azure Cosmos, а не в эмуляторе Azure Cosmos DB.

## <a name="use-the-emulator-on-windows"></a><a id="run-on-windows"></a>Использование эмулятора в Windows

Эмулятор Azure Cosmos DB устанавливается в `C:\Program Files\Azure Cosmos DB Emulator` расположение по умолчанию. Чтобы запустить эмулятор Azure Cosmos DB в Windows, нажмите кнопку **Пуск** или клавишу Windows. Начните вводить текст **Эмулятор Azure Cosmos DB** и выберите эмулятор в списке приложений.

:::image type="content" source="./media/local-emulator/database-local-emulator-start.png" alt-text="Нажмите кнопку Запуск или клавишу Windows, начните вводить Azure Cosmos DB Emulator и выберите эмулятор из списка приложений.":::

После запуска эмулятора в области уведомлений панели задач Windows появится значок. Он автоматически открывает обозреватель данных Azure Cosmos в браузере по этому URL- `https://localhost:8081/_explorer/index.html` адресу.

:::image type="content" source="./media/local-emulator/database-local-emulator-taskbar.png" alt-text="Уведомление локального эмулятора Azure Cosmos DB на панели задач":::

Можно также запускать и прекращать работу эмулятора из командной строки или команд PowerShell. Дополнительные сведения см. в [справочной статье программы командной строки](emulator-command-line-parameters.md) .

Эмулятор Azure Cosmos DB по умолчанию выполняется на локальном компьютере (localhost) и прослушивает порт 8081. В адресной строке отобразится адрес `https://localhost:8081/_explorer/index.html`. Чтобы позднее открыть страницу обозревателя данных, вы можете ввести в браузере этот URL-адрес или запустить обозреватель из эмулятора Azure Cosmos DB, используя значок в области уведомлений Windows, как показано ниже.

:::image type="content" source="./media/local-emulator/database-local-emulator-data-explorer-launcher.png" alt-text="Запуск обозревателя данных из локального эмулятора Azure Cosmos":::

## <a name="use-the-emulator-on-docker-for-windows"></a><a id="run-on-windows-docker"></a>Использование эмулятора на Docker для Windows

Вы можете запустить эмулятор Azure Cosmos DB в контейнере Windows DOCKER. Дополнительные сведения см. в [центре DOCKER](https://hub.docker.com/r/microsoft/azure-cosmosdb-emulator/) для команды DOCKER Pull и [GitHub](https://github.com/Azure/azure-cosmos-db-emulator-docker) `Dockerfile` . В настоящее время эмулятор не работает в DOCKER для Oracle Linux. Чтобы запустить эмулятор на Docker для Windows, выполните следующие действия:

1. После установки [DOCKER для Windows](https://www.docker.com/docker-windows) переключитесь на контейнеры Windows, щелкнув правой кнопкой мыши значок DOCKER на панели инструментов и выбрав пункт **Переключиться в контейнеры Windows**.

1. Затем извлеките образ эмулятора из Docker Hub, выполнив следующую команду в любой оболочке.

   ```bash
   docker pull mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator
   ```

1. Чтобы запустить образ, выполните следующие команды в зависимости от командной строки или среды PowerShell:

   # <a name="command-line"></a>[командная строка;](#tab/cli)

   ```bash

   md %LOCALAPPDATA%\CosmosDBEmulator\bind-mount

   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=%LOCALAPPDATA%\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator
   ```
   Образы DOCKER на основе Windows могут быть не совместимы с каждой ОС узла Windows. Например, образ эмулятора Azure Cosmos DB по умолчанию совместим только с Windows 10 и Windows Server 2016. Если требуется образ, совместимый с Windows Server 2019, выполните следующую команду:

   ```bash
   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=%hostDirectory%,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/winsrv2019/azure-cosmos-emulator:latest
   ```

   # <a name="powershell"></a>[PowerShell](#tab/powershell)

   ```powershell

   md $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount 2>null

   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=$env:LOCALAPPDATA\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator

   ```

   Ответ выглядит примерно так:

   ```bash
   Starting emulator
   Emulator Endpoint: https://172.20.229.193:8081/
   Primary Key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
   Exporting SSL Certificate
   You can import the SSL certificate from an administrator command prompt on the host by running:
   cd /d %LOCALAPPDATA%\CosmosDBEmulatorCert
   powershell .\importcert.ps1
   --------------------------------------------------------------------------------------------------
   Starting interactive shell
   ```
   ---

   > [!NOTE]
   > Если при выполнении `docker run` команды возникает ошибка конфликта портов (то есть если указанный порт уже используется), передайте пользовательский порт, изменив номера портов. Например, можно изменить параметр "-p 8081:8081" на "-p 443:8081"

1. Теперь используйте конечную точку эмулятора и первичный ключ из ответа и импортируйте сертификат TLS/SSL в узел. Чтобы импортировать сертификат TLS/SSL, выполните следующие действия в командной строке администратора:

   # <a name="command-line"></a>[командная строка;](#tab/cli)

   ```bash
   cd  %LOCALAPPDATA%\CosmosDBEmulator\bind-mount
   powershell .\importcert.ps1
   ```

   # <a name="powershell"></a>[PowerShell](#tab/powershell)

   ```powershell
   cd $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount
   .\importcert.ps1
   ```
   ---

1. Если вы закроете интерактивную оболочку после запуска эмулятора, он завершит работу контейнера эмулятора. Чтобы снова открыть обозреватель данных, перейдите по следующему URL-адресу в браузере. Конечная точка эмулятора отобразится в сообщении ответа, приведенном выше.

   `https://<emulator endpoint provided in response>/_explorer/index.html`

Если клиентское приложение .NET выполняется в контейнере DOCKER для Linux и если вы используете Azure Cosmos DB Emulator на размещающем компьютере, выполните инструкции из следующего раздела, чтобы импортировать сертификат в контейнере DOCKER в Linux.

### <a name="regenerate-the-emulator-certificates-when-running-on-a-docker-container"></a>Повторное создание сертификатов эмулятора при выполнении в контейнере DOCKER

При запуске эмулятора в контейнере DOCKER сертификаты, связанные с эмулятором, создаются повторно при каждом запуске и перезапуске соответствующего контейнера. Из-за этого необходимо повторно импортировать сертификаты после каждого запуска контейнера. Чтобы обойти это ограничение, можно использовать файл создания DOCKER, чтобы привязать контейнер DOCKER к определенному IP-адресу и образу контейнера.

Например, вы можете использовать следующую конфигурацию в файле создания DOCKER, не забудьте отформатировать его в соответствии с вашими требованиями: 

```yml
version: '2.4' # Do not upgrade to 3.x yet, unless you plan to use swarm/docker stack: https://github.com/docker/compose/issues/4513

networks:
  default:
    external: false
    ipam:
      driver: default
      config:
        - subnet: "172.16.238.0/24"

services:

  # First create a directory that will hold the emulator traces and certificate to be imported
  # set hostDirectory=C:\emulator\bind-mount
  # mkdir %hostDirectory%

  cosmosdb:
    container_name: "azurecosmosemulator"
    hostname: "azurecosmosemulator"
    image: 'mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator'
    platform: windows
    tty: true
    mem_limit: 3GB
    ports:
        - '8081:8081'
        - '8900:8900'
        - '8901:8901'
        - '8902:8902'
        - '10250:10250'
        - '10251:10251'
        - '10252:10252'
        - '10253:10253'
        - '10254:10254'
        - '10255:10255'
        - '10256:10256'
        - '10350:10350'
    networks:
      default:
        ipv4_address: 172.16.238.246
    volumes:
        - '${hostDirectory}:C:\CosmosDB.Emulator\bind-mount'
```

## <a name="use-the-emulator-on-linux-or-macos"></a><a id="run-on-linux-macos"></a>Использование эмулятора в Linux или macOS

В настоящее время эмулятор Azure Cosmos DB может быть запущен только в Windows. Если вы используете Linux или macOS, можно запустить эмулятор на виртуальной машине Windows, размещенной в гипервизоре, например Parallels или VirtualBox.

> [!NOTE]
> При каждом перезапуске виртуальной машины Windows, размещенной в гипервизоре, необходимо повторно импортировать сертификат, так как изменяется IP-адрес виртуальной машины. Импорт сертификата не требуется, если виртуальная машина настроена для сохранения IP-адреса.

Чтобы использовать эмулятор в средах Linux или macOS, выполните следующие действия:

1. Выполните следующую команду на виртуальной машине Windows и запишите IPv4-адрес:

   ```bash
   ipconfig.exe
   ```

1. В приложении измените URL-адрес конечной точки, чтобы он использовал адрес IPv4, возвращенный `ipconfig.exe` вместо `localhost` .

1. На виртуальной машине Windows запустите эмулятор Azure Cosmos DB из командной строки, используя следующие параметры. Дополнительные сведения о параметрах, поддерживаемых командной строкой, см. в [справочнике по программе командной строки эмулятора](emulator-command-line-parameters.md):

   ```bash
   Microsoft.Azure.Cosmos.Emulator.exe /AllowNetworkAccess /Key=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
   ```

1. Наконец, необходимо разрешить процесс доверия сертификатов между приложением, работающим в среде Linux или Mac, и эмулятором. Для разрешения сертификата можно использовать один из следующих двух вариантов.

   1. [Импортируйте сертификат эмулятора TLS/SSL в среду Linux или Mac](#import-certificate) или
   2. [Отключение проверки TLS/SSL в приложении](#disable-ssl-validation)

### <a name="option-1-import-the-emulator-tlsssl-certificate"></a><a id="import-certificate"></a>Вариант 1. Импорт сертификата TLS/SSL эмулятора

В следующих разделах показано, как импортировать сертификат TLS/SSL в среду Linux и macOS.

#### <a name="linux-environment"></a>Среда Linux

При работе в Linux для выполнения проверки .NET полагается на OpenSSL:

1. [Экспортируйте сертификат в формате PFX](local-emulator-export-ssl-certificates.md#export-emulator-certificate). Параметр PFX доступен при выборе экспорта закрытого ключа.

1. Скопируйте этот PFX-файл в среду Linux.

1. Преобразуйте PFX-файл в CRT-файл

   ```bash
   openssl pkcs12 -in YourPFX.pfx -clcerts -nokeys -out YourCTR.crt
   ```

1. Скопируйте CRT-файл в папку, содержащую пользовательские сертификаты в дистрибутиве Linux. Обычно в дистрибутивах Debian она находится в `/usr/local/share/ca-certificates/`.

   ```bash
   cp YourCTR.crt /usr/local/share/ca-certificates/
   ```

1. Обновите сертификаты TLS/SSL, которые будут обновлять `/etc/ssl/certs/` папку.

   ```bash
   update-ca-certificates
   ```

#### <a name="macos-environment"></a>среда macOS

При работе на компьютере Mac выполните следующие действия.

1. [Экспортируйте сертификат в формате PFX](local-emulator-export-ssl-certificates.md#export-emulator-certificate). Параметр PFX доступен при выборе экспорта закрытого ключа.

1. Скопируйте этот PFX-файл в среду Mac.

1. Откройте приложение *Keychain Access* (Доступ к цепочке ключей) и импортируйте PFX-файл.

1. Откройте список сертификатов и найдите сертификат с именем `localhost`.

1. Откройте контекстное меню для этого конкретного элемента, выберите *Получить элемент* и в разделе *Доверие*, в параметре  > *When using this certificate* (Во время использовании этого сертификата) выберите *Always Trust* (Всегда доверять). 

   :::image type="content" source="./media/local-emulator/mac-trust-certificate.png" alt-text="Откройте контекстное меню для этого конкретного элемента, выберите Get Item (Получить элемент) и в разделе Trust (Доверие) для параметра When using this certificate (Во время использовании этого сертификата) выберите Always Trust (Всегда доверять).":::
  
### <a name="option-2-disable-the-ssl-validation-in-the-application"></a><a id="disable-ssl-validation"></a>Вариант 2. Отключение проверки SSL в приложении

Отключение проверки SSL рекомендуется только для целей разработки и не должно выполняться при работе в рабочей среде. В следующих примерах показано, как отключить проверку SSL для приложений .NET и Node.js.

# <a name="net-standard-21"></a>[.NET Standard 2.1 и выше](#tab/ssl-netstd21)

Для любого приложения, работающего в платформе, совместимой с .NET Standard 2,1 или более поздней версии, мы можем использовать `CosmosClientOptions.HttpClientFactory` :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/HttpClientFactory/Program.cs?name=DisableSSLNETStandard21)]

# <a name="net-standard-20"></a>[.NET Standard 2.0](#tab/ssl-netstd20)

Для любого приложения, работающего в платформе, совместимой с .NET Standard 2,0, мы можем использовать `CosmosClientOptions.HttpClientFactory` :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/HttpClientFactory/Program.cs?name=DisableSSLNETStandard20)]

# <a name="nodejs"></a>[Node.js](#tab/ssl-nodejs)

Для Node.js приложений можно изменить `package.json` файл, чтобы задать `NODE_TLS_REJECT_UNAUTHORIZED` время при запуске приложения:

```json
"start": NODE_TLS_REJECT_UNAUTHORIZED=0 node app.js
```
--- 

## <a name="enable-access-to-emulator-on-a-local-network"></a>Разрешить доступ к эмулятору в локальной сети

При наличии нескольких компьютеров, использующих одну сеть, и при настройке эмулятора на одном компьютере и необходимости доступа к нему с другого компьютера. В таком случае необходимо включить доступ к эмулятору в локальной сети.

Вы можете запустить эмулятор в локальной сети. Чтобы разрешить доступ к сети, укажите параметр `/AllowNetworkAccess` в [командной строке](emulator-command-line-parameters.md), а также один из дополнительных параметров `/Key=key_string` или `/KeyFile=file_name`. Можно также применить параметр `/GenKeyFile=file_name`, чтобы заранее создать файл со случайным ключом. Его можно затем передать в `/KeyFile=file_name` или `/Key=contents_of_file`.

Чтобы включить сетевой доступ в первый раз, пользователь должен завершить работу эмулятора и удалить каталог данных эмулятора *%локалаппдата%\космосдбемулатор*.

## <a name="authenticate-connections-when-using-emulator"></a><a id="authenticate-requests"></a>Проверять подлинность подключений при использовании эмулятора

Как и в настоящей облачной службе Azure, в эмуляторе Azure Cosmos DB для каждого полученного запроса должна выполняться аутентификация. Эмулятор Azure Cosmos DB поддерживает только безопасное взаимодействие через TLS. Эмулятор Azure Cosmos DB поддерживает одну фиксированную учетную запись и известный ключ аутентификации для проверки подлинности с помощью первичного ключа. В качестве учетных данных для эмулятора Azure Cosmos DB можно использовать только эту учетную запись и только этот ключ. Их можно сформулировать следующим образом.

```bash
Account name: localhost:<port>
Account key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
```

> [!NOTE]
> Первичный ключ, поддерживаемый эмулятором Azure Cosmos DB, предназначен для использования только с эмулятором. Нельзя использовать рабочую учетную запись Azure Cosmos DB и ключ с эмулятором Azure Cosmos DB.

> [!NOTE]
> Если вы запустили эмулятор с параметром/key, используйте созданный ключ вместо ключа по умолчанию `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==` . Дополнительные сведения о параметре /Key см. в [справочнике по программе командной строки](emulator-command-line-parameters.md).

## <a name="connect-to-different-apis-with-the-emulator"></a><a id="connect-with-emulator-apis"></a>Подключение к различным API с помощью эмулятора

### <a name="sql-api"></a>API SQL

После запуска эмулятора Azure Cosmos DB на рабочем столе можно использовать любой поддерживаемый [пакет SDK Azure Cosmos DB](sql-api-sdk-dotnet-standard.md) или [Azure Cosmos DB REST API](/rest/api/cosmos-db/) для взаимодействия с эмулятором. Эмулятор Azure Cosmos DB также включает встроенный обозреватель данных, который позволяет создавать контейнеры для API SQL или Azure Cosmos DB для API Mongo DB. С помощью обозревателя данных можно просматривать и изменять элементы без написания кода.

```csharp
// Connect to the Azure Cosmos DB Emulator running locally
CosmosClient client = new CosmosClient(
   "https://localhost:8081", 
    "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==");

```

### <a name="azure-cosmos-dbs-api-for-mongodb"></a>API Azure Cosmos DB для MongoDB

После запуска эмулятора Azure Cosmos DB на рабочем столе можно использовать [API Azure Cosmos DB для MongoDB](mongodb-introduction.md) , чтобы взаимодействовать с эмулятором. Запустите эмулятор из [командной строки](emulator-command-line-parameters.md) от имени администратора с «/енаблемонгодбендпоинт». Затем подключитесь к учетной записи API MongoDB, используя следующую строку подключения:

```bash
mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true
```

### <a name="table-api"></a>API таблиц

После запуска эмулятора Azure Cosmos DB на рабочем столе можно использовать [Azure Cosmos DB API ТАБЛИЦ SDK](./tutorial-develop-table-dotnet.md) для взаимодействия с эмулятором. Запустите эмулятор из [командной строки](emulator-command-line-parameters.md) от имени администратора с «/енаблетаблиндпоинт». Затем выполните следующий код для подключения к учетной записи API таблиц:

```csharp
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Table;
using CloudTable = Microsoft.WindowsAzure.Storage.Table.CloudTable;
using CloudTableClient = Microsoft.WindowsAzure.Storage.Table.CloudTableClient;

string connectionString = "DefaultEndpointsProtocol=http;AccountName=localhost;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==;TableEndpoint=http://localhost:8902/;";

CloudStorageAccount account = CloudStorageAccount.Parse(connectionString);
CloudTableClient tableClient = account.CreateCloudTableClient();
CloudTable table = tableClient.GetTableReference("testtable");
table.CreateIfNotExists();
table.Execute(TableOperation.Insert(new DynamicTableEntity("partitionKey", "rowKey")));
```

### <a name="cassandra-api"></a>API Cassandra

Запустите эмулятор из [командной строки](emulator-command-line-parameters.md) администратора с помощью команды "/енаблекассандраендпоинт". В качестве альтернативы можно также задать переменную среды `AZURE_COSMOS_EMULATOR_CASSANDRA_ENDPOINT=true`.

1. [Установите Python 2.7](https://www.python.org/downloads/release/python-2716/).

1. [Установите Cassandra CLI/CQLSH](https://cassandra.apache.org/download/)

1. Выполните следующие команды в обычном окне командной строки:

   ```bash
   set Path=c:\Python27;%Path%
   cd /d C:\sdk\apache-cassandra-3.11.3\bin
   set SSL_VERSION=TLSv1_2
   set SSL_VALIDATE=false
   cqlsh localhost 10350 -u localhost -p C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw== --ssl
   ```

1. В оболочке CQLSH выполните следующие команды, чтобы подключиться к конечной точке Cassandra:

   ```bash
   CREATE KEYSPACE MyKeySpace WITH replication = {'class':'MyClass', 'replication_factor': 1};
   DESCRIBE keyspaces;
   USE mykeyspace;
   CREATE table table1(my_id int PRIMARY KEY, my_name text, my_desc text);
   INSERT into table1 (my_id, my_name, my_desc) values( 1, 'name1', 'description 1');
   SELECT * from table1;
   EXIT
   ```

### <a name="gremlin-api"></a>API Gremlin

Запустите эмулятор из [командной строки](emulator-command-line-parameters.md)администратора с помощью команды "/енаблегремлинендпоинт". В качестве альтернативы можно также задать переменную среды `AZURE_COSMOS_EMULATOR_GREMLIN_ENDPOINT=true`.

1. [Установите apache-tinkerpop-gremlin-console-3.3.4.](https://archive.apache.org/dist/tinkerpop/3.3.4)

1. В обозревателе данных эмулятора создайте базу данных «DB1» и коллекцию «coll1». для ключа секции выберите/Name.

1. Выполните следующие команды в обычном окне командной строки:

   ```bash
   cd /d C:\sdk\apache-tinkerpop-gremlin-console-3.3.4-bin\apache-tinkerpop-gremlin-console-3.3.4
  
   copy /y conf\remote.yaml conf\remote-localcompute.yaml
   notepad.exe conf\remote-localcompute.yaml
     hosts: [localhost]
     port: 8901
     username: /dbs/db1/colls/coll1
     password: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
     connectionPool: {
     enableSsl: false}
     serializer: { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV1d0,
     config: { serializeResultToString: true  }}

   bin\gremlin.bat
   ```

1. В оболочке Gremlin выполните следующие команды, чтобы подключиться к конечной точке Gremlin:

   ```bash
   :remote connect tinkerpop.server conf/remote-localcompute.yaml
   :remote console
   :> g.V()
   :> g.addV('person1').property(id, '1').property('name', 'somename1')
   :> g.addV('person2').property(id, '2').property('name', 'somename2')
   :> g.V()
   ```

## <a name="uninstall-the-local-emulator"></a><a id="uninstall"></a>Удаление локального эмулятора

Чтобы удалить эмулятор, выполните следующие действия.

1. Закройте все открытые экземпляры локального эмулятора, щелкнув правой кнопкой мыши значок **эмулятора Azure Cosmos DB** в области уведомлений и выбрав команду **выход**. Выход из всех экземпляров может занять около минуты.

1. В поле поиска Windows введите **приложения & компоненты** и выберите **приложения & компоненты (системные параметры)** результат.

1. В списке приложений перейдите к **эмулятору Azure Cosmos DB**, выберите его, щелкните **Удалить**, затем подтвердите и снова выберите **Удалить** .

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как использовать локальный эмулятор для бесплатной локальной разработки. Теперь можно перейти к следующим статьям:

* [Экспорт Azure Cosmos DB сертификатов эмулятора для использования с приложениями Java, Python и Node.js](local-emulator-export-ssl-certificates.md)
* [Использование параметров командной строки и команд PowerShell для управления эмулятором](emulator-command-line-parameters.md)
* [Отладка проблем с эмулятором](troubleshoot-local-emulator.md)
