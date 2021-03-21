---
title: Справочник по командной строке и PowerShell для эмулятора Azure Cosmos DB
description: Сведения о параметрах командной строки для эмулятора Azure Cosmos DB, управлении эмулятором с помощью PowerShell и о том, как изменить число контейнеров, которые можно создать в эмуляторе.
ms.service: cosmos-db
ms.topic: how-to
author: markjbrown
ms.author: mjbrown
ms.date: 09/17/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: e0c795484bf860402d05c1dc5779633962a44ec2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99979510"
---
# <a name="command-line-and-powershell-reference-for-azure-cosmos-db-emulator"></a>Справочник по командной строке и PowerShell для эмулятора Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Эмулятор Azure Cosmos DB предоставляет локальную среду, которая эмулирует службу Azure Cosmos DB для локальных целей разработки. После [установки эмулятора](local-emulator.md)можно управлять эмулятором с помощью командной строки и команд PowerShell. В этой статье описывается, как использовать командную строку и команды PowerShell для запуска и завершения работы эмулятора, настройки параметров и выполнения других операций. Необходимо выполнить команды из расположения установки.

##  <a name="manage-the-emulator-with-command-line-syntax"></a><a id="command-line"></a>Управление эмулятором с помощью синтаксиса командной строки

```cmd
Microsoft.Azure.Cosmos.Emulator.exe [/Shutdown] [/DataPath] [/Port] [/MongoPort] [/DirectPorts] [/Key] [/EnableRateLimiting] [/DisableRateLimiting] [/NoUI] [/NoExplorer] [/EnableMongoDbEndpoint] [/?]
```

Чтобы просмотреть список параметров, в командной строке введите `Microsoft.Azure.Cosmos.Emulator.exe /?` .

|**Параметр** | **Описание** | **Command**| **Аргументы**|
|---|---|---|---|
|[Нет аргументов] | Запускает эмулятор Azure Cosmos DB с параметрами по умолчанию. |Microsoft.Azure.Cosmos.Emulator.exe| |
|[Help] |Отображает список поддерживаемых аргументов командной строки.|Microsoft.Azure.Cosmos.Emulator.exe /? | |
| GetStatus |Получает состояние эмулятора Azure Cosmos DB. Состояние обозначается кодом выхода: 1 = запуск, 2 = выполнение, 3 = остановка. Код выхода с отрицательным значением указывает, что произошла ошибка. Другие выходные данные не формируются. | Microsoft.Azure.Cosmos.Emulator.exe /GetStatus| |
| Shutdown| Завершает работу эмулятора Azure Cosmos DB.| Microsoft.Azure.Cosmos.Emulator.exe /Shutdown | |
|DataPath | Указывает путь для сохранения файлов данных. По умолчанию имеет значение %LocalAppdata%\CosmosDBEmulator. | Microsoft.Azure.Cosmos.Emulator.exe /DataPath=\<datapath\> | \<datapath\>: любой доступный путь. |
|Порт | Указывает номер порта, который должен использоваться эмулятором. Значение по умолчанию — 8081. |Microsoft.Azure.Cosmos.Emulator.exe /Port=\<port\> | \<port\>: один номер порта. |
| ComputePort | Указанный номер порта для службы Compute Interop Gateway. Порт пробы для конечной точки HTTP шлюза вычисляется как ComputePort+79. Это означает, что должны быть открыты и свободны оба порта: ComputePort и ComputePort+79. Значение по умолчанию ― 8900. | Microsoft.Azure.Cosmos.Emulator.exe /ComputePort=\<computeport\> | \<computeport\>: один номер порта. |
| EnableMongoDbEndpoint=3.2 | Включает API MongoDB 3.2 | Microsoft.Azure.Cosmos.Emulator.exe /EnableMongoDbEndpoint=3.2 | |
| EnableMongoDbEndpoint=3.6 | Включает API MongoDB 3.6 | Microsoft.Azure.Cosmos.Emulator.exe /EnableMongoDbEndpoint=3.6 | |
| MongoPort | Указывает номер порта для использования с интерфейсом совместимости с MongoDB. Значение по умолчанию — 10255. |Microsoft.Azure.Cosmos.Emulator.exe /MongoPort=\<mongoport\>|\<mongoport\>: один номер порта.|
| EnableCassandraEndpoint | Включает API Cassandra | Microsoft.Azure.Cosmos.Emulator.exe /EnableCassandraEndpoint | |
| CassandraPort | Указывает номер порта для конечной точки Cassandra. Значение по умолчанию — 10350. | Microsoft.Azure.Cosmos.Emulator.exe /CassandraPort=\<cassandraport\> | \<cassandraport\>: один номер порта. |
| EnableGremlinEndpoint | Включает API Gremlin | Microsoft.Azure.Cosmos.Emulator.exe /EnableGremlinEndpoint | |
| GremlinPort | Номер порта для конечной точки Gremlin. Значение по умолчанию — 8901. | Microsoft.Azure.Cosmos.Emulator.exe /GremlinPort=\<port\> | \<port\>: один номер порта. |
|EnableTableEndpoint | Включает API таблиц Azure | Microsoft.Azure.Cosmos.Emulator.exe /EnableTableEndpoint | |
|TablePort | Номер порта для конечной точки таблицы Azure. Значение по умолчанию — 8902. | Microsoft.Azure.Cosmos.Emulator.exe /TablePort=\<port\> | \<port\>: один номер порта.|
| KeyFile | Считывает ключ авторизации из указанного файла. Используйте параметр /GenKeyFile, чтобы сгенерировать файл ключа. | Microsoft.Azure.Cosmos.Emulator.exe /KeyFile=\<file_name\> | \<file_name\>: Путь к файлу. |
| ResetDataPath | Рекурсивно удаляет файлы по указанному пути. Если путь не указан, используется путь по умолчанию %LOCALAPPDATA%\CosmosDbEmulator | Microsoft.Azure.Cosmos.Emulator.exe /ResetDataPath=\<path> | \<path\>: Путь к файлу  |
| StartTraces  |  Начинает сбор журналов трассировки для отладки с помощью LOGMAN. | Microsoft.Azure.Cosmos.Emulator.exe /StartTraces | |
| StopTraces     | Завершает сбор журналов трассировки для отладки с помощью LOGMAN. | Microsoft.Azure.Cosmos.Emulator.exe /StopTraces  | |
| StartWprTraces  |  Начинает сбор журналов трассировки для отладки с помощью средства записи производительности Windows. | Microsoft.Azure.Cosmos.Emulator.exe /StartWprTraces | |
| StopWprTraces     | Останавливает сбор журналов трассировки для отладки с помощью средства записи производительности Windows. | Microsoft.Azure.Cosmos.Emulator.exe /StopWprTraces  | |
|FailOnSslCertificateNameMismatch | По умолчанию эмулятор повторно создает самозаверяющий сертификат TLS/SSL, если SAN сертификата не включает доменное имя узла эмулятора, локальный IPv4-адрес, localhost и 127.0.0.1. С таким параметром эмулятор завершится ошибкой при запуске. В этом случае следует применить параметр /GenCert, чтобы создать и установить новый самозаверяющий TLS/SSL-сертификат. | Microsoft.Azure.Cosmos.Emulator.exe /FailOnSslCertificateNameMismatch  | |
| GenCert | Создает и устанавливает новый самозаверяющий сертификат TLS/SSL-сертификат. При необходимости можно включить разделенный запятыми список дополнительных DNS-имен для доступа к эмулятору по сети. | Microsoft.Azure.Cosmos.Emulator.exe /GenCert=\<dns-names\> |\<dns-names\>: необязательный список дополнительны DNS-имен, разделенных запятыми  |
| DirectPorts |Указывает порты, которые нужно использовать для прямого подключения. По умолчанию это порты 10251, 10252, 10253, 10254. | Microsoft.Azure.Cosmos.Emulator.exe /DirectPorts:\<directports\> | \<directports\>: разделенный запятыми список из 4 портов. |
| Клавиши |Ключ проверки подлинности для эмулятора. Ключ должен иметь формат 64-разрядного вектора в кодировке Base-64. | Microsoft.Azure.Cosmos.Emulator.exe /Key:\<key\> | \<key\>: Ключ должен иметь формат 64-разрядного вектора в кодировке base-64|
| EnableRateLimiting | Указывает, что ограничение частоты запросов включено. |Microsoft.Azure.Cosmos.Emulator.exe /EnableRateLimiting | |
| DisableRateLimiting |Указывает, что ограничение частоты запросов отключено. |Microsoft.Azure.Cosmos.Emulator.exe /DisableRateLimiting | |
| NoUI | Скрывает пользовательский интерфейс эмулятора. | Microsoft.Azure.Cosmos.Emulator.exe /NoUI | |
| NoExplorer | Скрывает обозреватель данных при запуске. |Microsoft.Azure.Cosmos.Emulator.exe /NoExplorer | | 
| PartitionCount | Указывает максимальное количество секционированных контейнеров. Дополнительные сведения см. в разделе [Изменение количества контейнеров](#set-partitioncount). | Microsoft.Azure.Cosmos.Emulator.exe /PartitionCount=\<partitioncount\> | \<partitioncount\>: Максимально допустимое количество односекционных контейнеров. Значение по умолчанию равно 25. Максимально допустимое значение — 250.|
| DefaultPartitionCount| Указывает количество секций по умолчанию для секционированного контейнера. | Microsoft.Azure.Cosmos.Emulator.exe /DefaultPartitionCount=\<defaultpartitioncount\> | \<defaultpartitioncount\> Значение по умолчанию — 25.|
| AllowNetworkAccess | Разрешает доступ к эмулятору по сети. Для включения сетевого доступа нужно также передать один из параметров: /Key=\<key_string\> или /KeyFile=\<file_name\>. | Microsoft.Azure.Cosmos.Emulator.exe /AllowNetworkAccess /Key=\<key_string\> или Microsoft.Azure.Cosmos.Emulator.exe /AllowNetworkAccess /KeyFile=\<file_name\>| |
| NoFirewall | Не изменяйте правила брандмауэра при использовании /AllowNetworkAccess. |Microsoft.Azure.Cosmos.Emulator.exe /NoFirewall | |
| GenKeyFile | Создает новый ключ авторизации и сохраняет его в указанном файле. Созданный ключ можно использовать с параметрами/Key или/KeyFile. | Microsoft.Azure.Cosmos.Emulator.exe /GenKeyFile=\<path to key file\> | |
| Согласованность | Определяет уровень согласованности по умолчанию для учетной записи. | Microsoft.Azure.Cosmos.Emulator.exe /Consistency=\<consistency\> | \<consistency\>: значение должно быть одного из следующих [уровней согласованности](consistency-levels.md): Session, Strong, Eventual или BoundedStaleness. По умолчанию используется значение Session. |
| ? | Показывает справочные сообщения.| | |

## <a name="manage-the-emulator-with-powershell"></a>Управление эмулятором с помощью PowerShell

Эмулятор поставляется вместе с модулем PowerShell, который позволяет запускать, останавливать службу и получать сведения о ее состоянии. Выполните следующий командлет, чтобы использовать модуль PowerShell:

```powershell
Import-Module "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules\Microsoft.Azure.CosmosDB.Emulator"
```

Или поместите каталог `PSModules` в `PSModulePath` и импортируйте его с помощью следующей команды:

```powershell
$env:PSModulePath += "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules"
Import-Module Microsoft.Azure.CosmosDB.Emulator
```

Ниже приводится сводка команд для управления эмулятором из PowerShell:

### `Get-CosmosDbEmulatorStatus`

**Синтаксис**

`Get-CosmosDbEmulatorStatus`

**Замечания**

Возвращает одно из следующих значений ServiceControllerStatus: ServiceControllerStatus.StartPending, ServiceControllerStatus.Running или ServiceControllerStatus.Stopped.

### `Start-CosmosDbEmulator`

**Синтаксис**

`Start-CosmosDbEmulator [-DataPath <string>] [-DefaultPartitionCount <uint16>] [-DirectPort <uint16[]>] [-MongoPort <uint16>] [-NoUI] [-NoWait] [-PartitionCount <uint16>] [-Port <uint16>] [<CommonParameters>]`

**Замечания**

Запускает эмулятор. По умолчанию команда ожидает, когда эмулятор будет готов к приему запросов. Используйте параметр -NoWait, если командлет должен вернуть результат сразу после запуска эмулятора.

### `Stop-CosmosDbEmulator`

**Синтаксис**

 `Stop-CosmosDbEmulator [-NoWait]`

**Замечания**

Останавливает работу эмулятора. По умолчанию эта команда ожидает, когда эмулятор окончательно завершит работу. Используйте параметр -NoWait, если командлет должен вернуть результат сразу после перехода эмулятора в автономный режим.

### `Uninstall-CosmosDbEmulator`

**Синтаксис**

`Uninstall-CosmosDbEmulator [-RemoveData]`

**Замечания**

Удаляет эмулятор и при необходимости все содержимое $env:LOCALAPPDATA\CosmosDbEmulator.
Командлет гарантирует остановку работы эмулятора перед удалением.

## <a name="change-the-number-of-default-containers"></a><a id="set-partitioncount"></a>Изменение количества контейнеров по умолчанию

По умолчанию можно создать до 25 контейнеров фиксированного размера (поддерживается только с помощью Azure Cosmos DB пакетов SDK) или 5 неограниченных контейнеров с помощью эмулятора Azure Cosmos DB. Изменив значение **PartitionCount**, вы сможете создать до 250 контейнеров фиксированного размера или 50 контейнеров неограниченного размера либо любое их сочетание в пределах указанных ограничений (где один контейнер неограниченного размера приравнивается к 5 контейнерам фиксированного размера). Но мы не рекомендуем настраивать для эмулятора запуск более 200 контейнеров фиксированного размера. Дополнительное количество создаваемых ими операций ввода-вывода может привести к непредсказуемым превышениям времени ожидания при использовании API такой конечной точки.

При попытке создать контейнер, превышающий ограничение на количество секций, эмулятор генерирует исключение ServiceUnavailable со следующим сообщением:

> К сожалению, в настоящее время мы столкнулись с высокой потребностью в этом регионе и не можем выполнить ваш запрос. We work continuously to bring more and more capacity online, and encourage you to try again.
> ActivityId: 12345678-1234-1234-1234-123456789abc

Чтобы изменить число контейнеров, доступных в эмуляторе Azure Cosmos DB, выполните следующие действия.

1. Удалите все локальные данные эмулятора Azure Cosmos DB, щелкнув правой кнопкой мыши значок **эмулятора Azure Cosmos DB** в области уведомлений и выбрав **Сброс данных**.

1. Удалите все данные эмулятора в папке `%LOCALAPPDATA%\CosmosDBEmulator`.

1. Выйдите из всех открытых экземпляров, щелкнув правой кнопкой мыши значок **эмулятора Azure Cosmos DB** в области уведомлений и выбрав **Выход**. Выход из всех экземпляров может занять около минуты.

1. Установите последнюю версию [эмулятора Azure Cosmos DB](https://aka.ms/cosmosdb-emulator).

1. Запустите эмулятор с флагом PartitionCount, задав значение <= 250. Например: `C:\Program Files\Azure Cosmos DB Emulator> Microsoft.Azure.Cosmos.Emulator.exe /PartitionCount=100`.
 
## <a name="next-steps"></a>Дальнейшие действия

* [Экспорт Azure Cosmos DB сертификатов эмулятора для использования с приложениями Java, Python и Node.js](local-emulator-export-ssl-certificates.md)
* [Отладка проблем с эмулятором](troubleshoot-local-emulator.md)
