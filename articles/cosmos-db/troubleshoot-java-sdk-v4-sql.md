---
title: Диагностика и устранение неполадок с пакетом SDK Java версии 4 для Azure Cosmos DB
description: Воспользуйтесь такими функциями, как ведение журнала на стороне клиента, и другими сторонними инструментами для выявления, диагностики и устранения проблем, связанных с Azure Cosmos DB в пакете SDK Java версии 4.
author: anfeldma-ms
ms.service: cosmos-db
ms.date: 06/11/2020
ms.author: anfeldma
ms.devlang: java
ms.subservice: cosmosdb-sql
ms.topic: troubleshooting
ms.custom: devx-track-java
ms.openlocfilehash: cba8b97adb40ca2c277268188ff6ad541c7e9676
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100596469"
---
# <a name="troubleshoot-issues-when-you-use-azure-cosmos-db-java-sdk-v4-with-sql-api-accounts"></a>Устранение неполадок при использовании SDK Java версии 4 для Azure Cosmos DB с учетными записями API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Пакет SDK для Java версии 4](troubleshoot-java-sdk-v4-sql.md)
> * [Пакет SDK для Async Java версии 2](troubleshoot-java-async-sdk.md)
> * [.NET](troubleshoot-dot-net-sdk.md)
> 

> [!IMPORTANT]
> В этой статье описывается устранение неполадок только для пакета SDK Java версии 4 для Azure Cosmos DB. Дополнительные сведения см. в [заметках о выпуске](sql-api-sdk-java-v4.md), [репозитории Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos) и [советах по повышению производительности](performance-tips-java-sdk-v4-sql.md) для пакета SDK Java версии 4 для Azure Cosmos DB. Если сейчас вы используете более раннюю версию, чем версия 4, руководство [Перевод приложения на использование пакета средств разработки Java для Azure Cosmos DB версии 4](migrate-java-v4-sdk.md) поможет вам обновить его до версии 4.
>

В этой статье рассматриваются распространенные проблемы при использовании пакета SDK Java версии 4 для Azure Cosmos DB с учетными записями API SQL для Azure Cosmos DB, а также связанные с этим обходные пути, средства и действия для диагностики.
Пакет SDK Java версии 4 для Azure Cosmos DB обеспечивает клиентское логическое представление для доступа к API SQL для Azure Cosmos DB. В этой статье описываются средства и подходы, которые помогут вам, если вы столкнетесь с проблемами.

Начните с этого списка:

* Ознакомьте с разделом [Распространенные проблемы и их обходные решения] в этой статье.
* Просмотрите пакет SDK для Java в центральном репозитории Azure Cosmos DB, который доступен [с открытым исходным кодом на GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos). Он имеет [раздел проблем](https://github.com/Azure/azure-sdk-for-java/issues), который активно отслеживается. Вы можете там найти уже готовое обходное решение похожей проблемы. Одним из полезных советов является фильтрация проблем по тегу *cosmos:v4-item*.
* Просмотрите [советы по повышению производительности](performance-tips-java-sdk-v4-sql.md) для пакета SDK Java версии 4 для Azure Cosmos DB и следуйте предложенным рекомендациям.
* Следуйте указаниям в оставшейся части этой статьи, если решение не нашлось. Затем [сообщите о проблеме в GitHub](https://github.com/Azure/azure-sdk-for-java/issues). Если есть возможность добавить теги к проблеме на GitHub, добавьте тег *cosmos:v4-item*.

### <a name="retry-logic"></a>Логика повторных попыток <a id="retry-logics"></a>
При любой ошибке ввода-вывода пакет SDK Cosmos DB попытается повторить неудачную операцию, если в нем поддерживаются повторные попытки. Наличие повторных попыток для любой ошибки является хорошей практикой, но, в частности, требуется обработка или повторная попытка записи. Рекомендуется использовать последний пакет SDK, так как логика повторных попыток постоянно улучшается.

1. Ошибки ввода-вывода при чтении и запросе будут повторяться пакетом SDK, не отображая их конечному пользователю.
2. Операции записи (Create, Upsert, Replace, DELETE) не являются идемпотентными, и поэтому пакет SDK не всегда может без каких-либо ошибок повторить операции записи. Необходимо, чтобы логика приложения пользователя обрабатывала ошибку и повторить попытку.
3. Устранение [проблем с доступностью пакета SDK](troubleshoot-sdk-availability.md) объясняет повторные попытки для учетных записей Cosmos DB в нескольких регионах.

## <a name="common-issues-and-workarounds"></a><a name="common-issues-workarounds"></a>Распространенные проблемы и их обходные решения

### <a name="network-issues-netty-read-timeout-failure-low-throughput-high-latency"></a>Проблемы с сетью, ошибка из-за истечения времени ожидания чтения Netty, низкая пропускная способность, высокая задержка

#### <a name="general-suggestions"></a>Общие рекомендации
Обеспечить наилучшую производительность можно так.
* Убедитесь, что приложение выполняется в том же регионе, который указан для вашей учетной записи Azure Cosmos DB. 
* Проверьте использование ЦП на узле, где выполняется приложение. Если используются 50 или более процентов ЦП, запустите приложение на узле с лучшей конфигурацией. Можно также распределить нагрузку на большее число компьютеров.
    * Если вы запускаете приложение в службе Azure Kubernetes, вы можете [использовать Azure Monitor для мониторинга загрузки ЦП](../azure-monitor/containers/container-insights-analyze.md).

#### <a name="connection-throttling"></a>Регулирование подключения
Регулирование подключения может произойти из-за [Ограничение числа подключений на узле] или [Нехватка портов SNAT (PAT) Azure].

##### <a name="connection-limit-on-a-host-machine"></a><a name="connection-limit-on-host"></a>Ограничение числа подключений на узле
Некоторые системы Linux (например, Red Hat) имеют ограничение на общее число открытых файлов. Сокеты в Linux реализованы как файлы, поэтому это число также существенно ограничивает общее число подключений.
Выполните следующую команду.

```bash
ulimit -a
```
Максимальное разрешенное количество открытых файлов, которые определены как "nofile", должно быть по крайней мере вдвое больше, чем размер пула подключений. Дополнительные сведения см. в [советах по повышению производительности](performance-tips-java-sdk-v4-sql.md) для пакета SDK Java версии 4 для Azure Cosmos DB.

##### <a name="azure-snat-pat-port-exhaustion"></a><a name="snat"></a>Нехватка портов SNAT (PAT) Azure

Если ваше приложение развернуто в службе "Виртуальные машины Azure" без общедоступного IP-адреса, по умолчанию [порты Azure SNAT](../load-balancer/load-balancer-outbound-connections.md#preallocatedports) используются для установления подключений к любой конечной точке вне вашей виртуальной машины. Количество разрешенных подключений от виртуальной машины к конечной точке Azure Cosmos DB ограничивается [конфигурацией Azure SNAT](../load-balancer/load-balancer-outbound-connections.md#preallocatedports).

 Порты Azure SNAT используются, только если у виртуальной машины есть частный IP-адрес и процесс на виртуальной машине пытается установить подключение к общедоступному IP-адресу. Избежать ограничений Azure SNAT можно двумя способами:

* Добавьте конечную точку службы Azure Cosmos DB к подсети виртуальной сети для службы "Виртуальные машины Azure". Дополнительные сведения см. в статье [Конечные точки служб для виртуальной сети Azure](../virtual-network/virtual-network-service-endpoints-overview.md). 

    При включении конечной точки службы запросы больше не отправляются с общедоступного IP-адреса в Azure Cosmos DB. Вместо этого отправляются идентификаторы виртуальной сети и подсети. Это изменение может привести к сбою брандмауэра, если разрешены только общедоступные IP-адреса. Если вы используете брандмауэр, при включении конечной точки службы добавьте подсеть в брандмауэре с помощью [списков управления доступом для виртуальных сетей](/previous-versions/azure/virtual-network/virtual-networks-acl).
* Назначьте общедоступный IP-адрес виртуальной машине Azure.

##### <a name="cant-reach-the-service---firewall"></a><a name="cant-connect"></a>Не удается связаться со службой — брандмауэр
Сообщение ``ConnectTimeoutException`` указывает, что пакет SDK не может связаться со службой.
При использовании режима прямого подключения может возникнуть ошибка, аналогичная приведенной ниже.
```
GoneException{error=null, resourceAddress='https://cdb-ms-prod-westus-fd4.documents.azure.com:14940/apps/e41242a5-2d71-5acb-2e00-5e5f744b12de/services/d8aa21a5-340b-21d4-b1a2-4a5333e7ed8a/partitions/ed028254-b613-4c2a-bf3c-14bd5eb64500/replicas/131298754052060051p//', statusCode=410, message=Message: The requested resource is no longer available at the server., getCauseInfo=[class: class io.netty.channel.ConnectTimeoutException, message: connection timed out: cdb-ms-prod-westus-fd4.documents.azure.com/101.13.12.5:14940]
```

Если на компьютере с приложением работает брандмауэр, откройте диапазон портов с 10 000 по 20 000, которые используются в режиме прямого подключения.
Также ознакомьтесь с разделом [Ограничение числа подключений на узле](#connection-limit-on-host).

#### <a name="http-proxy"></a>Прокси-сервер HTTP

При использовании прокси-сервера HTTP убедитесь, что он может поддерживать число подключений, указанное в свойстве `ConnectionPolicy` пакета SDK.
В противном случае возникнут проблемы с подключением.

#### <a name="invalid-coding-pattern-blocking-netty-io-thread"></a>Недопустимый шаблон кодирования: блокировка потока Netty для ввода-вывода

Пакет SDK использует библиотеку [Netty](https://netty.io/) для ввода-вывода, чтобы связываться с Azure Cosmos DB. Пакет SDK имеет асинхронный API и использует неблокирующие API Netty для ввода-вывода. Операции ввода-вывода, присущие пакету SDK, выполняются в потоках Netty для ввода-вывода. Число потоков Netty для ввода-вывода настроено так, чтобы совпадать с числом ядер ЦП компьютера, где выполняется приложение. 

Потоки Netty для ввода-вывода предназначены только для операций Netty для ввода-вывода без блокировки. Пакет SDK возвращает результат вызова API, касающийся одного из потоков Netty для ввода-вывода, в код приложения. Если приложение выполняет операцию длительное время после получения результатов касательно потока Netty, пакет SDK может не располагать достаточным количеством потоков ввода-вывода для выполнения внутренних операций ввода-вывода. Кодирование такого приложения может привести к низкой пропускной способности, длительной задержке и сбоям `io.netty.handler.timeout.ReadTimeoutException`. Обходное решение заключается в том, чтобы переключить поток, когда вы знаете, что операция займет некоторое время.

Рассмотрим, например, следующий фрагмент кода, который добавляет элементы в контейнер (рекомендации по настройке базы данных и контейнера см. [здесь](create-sql-api-java.md)). Можно выполнить длительную работу, которая занимает более нескольких миллисекунд, в потоке Netty. В таком случае рано или поздно возникнет состояние, при котором не будет потока Netty для ввода-вывода, способного обрабатывать операции ввода-вывода. В результате произойдет сбой ReadTimeoutException.

### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-readtimeout"></a>Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=TroubleshootNeedsSchedulerAsync)]

Обходное решение заключается в изменении потока, в котором вы выполняете длительные операции. Определите отдельный экземпляр планировщика для приложения.

### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-scheduler"></a>Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=TroubleshootCustomSchedulerAsync)]

Может потребоваться выполнить длительную работу (например, такую, которая требует значительных вычислений или блокировки ввода-вывода). В этом случае необходимо переключение потока на рабочую роль, предоставляемую параметром `customScheduler` с помощью API `.publishOn(customScheduler)`.

### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-apply-custom-scheduler"></a>Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=TroubleshootPublishOnSchedulerAsync)]

Используя `publishOn(customScheduler)`, вы освобождаете поток Netty для ввода-вывода и переключаетесь на собственный поток, предоставляемый настраиваемым планировщиком. Это изменение позволяет решить проблему. Сбой `io.netty.handler.timeout.ReadTimeoutException` больше возникать не будет.

### <a name="request-rate-too-large"></a>Высокая частота запросов
Этот сбой происходит на сервере. Он означает, что вы использовали подготовленную пропускную способность. Повторите попытку позже. Если эта ошибка возникает часто, попробуйте увеличить пропускную способность коллекции.

* **Применение интервала задержки getRetryAfterInMilliseconds.**

    Во время проверки производительности следует увеличивать нагрузку до тех пор, пока регулирование не будет выполняться при небольшой частоте запросов. При регулировании клиентское приложение должно реализовать интервал частоты повтора для частоты повтора сервера. Это гарантирует, что время ожидания между повторными попытками будет минимальным.

### <a name="failure-connecting-to-azure-cosmos-db-emulator"></a>Сбой при подключении к эмулятору Azure Cosmos DB

Сертификат HTTPS эмулятора Azure Cosmos DB является самозаверяющим. Чтобы SDK работал с эмулятором, необходимо импортировать сертификат эмулятора в Java TrustStore. Дополнительные сведения см. в статье [экспорт Azure Cosmos DB сертификатов эмулятора](local-emulator-export-ssl-certificates.md).

### <a name="dependency-conflict-issues"></a>Проблемы с конфликтом зависимостей

Пакет SDK Java для Azure Cosmos DB требует несколько зависимостей. Вообще говоря, если дерево зависимостей вашего проекта включает более раннюю версию артефакта, от которой зависит пакет SDK Java для Azure Cosmos DB, это может привести к непредвиденным ошибкам при запуске приложения. При проведении отладки приложения для поиска причины возникновения исключения рекомендуется дважды проверить, что дерево зависимостей случайно не извлекает более раннюю версию одной или нескольких зависимостей пакета SDK Java для Azure Cosmos DB.

Обходной путь для такой проблемы состоит в том, чтобы определить, какие из зависимостей вашего проекта требуют старую версию, исключить транзитивную зависимость от этой более старой версии и позволить пакету SDK Java для Azure Cosmos DB установить более новую версию.

Чтобы узнать, какие зависимости проекта используют более старую версию, что необходима пакету SDK Java для Azure Cosmos DB, выполните следующую команду для файла pom.xml вашего проекта.
```bash
mvn dependency:tree
```
Дополнительные сведения см. в [руководстве Maven по дереву зависимостей](https://maven.apache.org/plugins/maven-dependency-plugin/examples/resolving-conflicts-using-the-dependency-tree.html).

После того как вы узнаете, какая зависимость проекта зависит от более старой версии, вы можете изменить зависимость от этой библиотеки в файле POM и исключить транзитивную зависимость, следуя приведенному ниже примеру (предполагается, что устаревшей зависимостью является *reactor-core*):

```xml
<dependency>
  <groupId>${groupid-of-lib-which-brings-in-reactor}</groupId>
  <artifactId>${artifactId-of-lib-which-brings-in-reactor}</artifactId>
  <version>${version-of-lib-which-brings-in-reactor}</version>
  <exclusions>
    <exclusion>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-core</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

Дополнительные сведения см. в [руководстве по исключению транзитивной зависимости](https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html).


## <a name="enable-client-sdk-logging"></a><a name="enable-client-sice-logging"></a>Включение ведения журнала для пакета SDK клиента

Пакет SDK Java версии 4 для Azure Cosmos DB использует SLF4j как интерфейс ведения журнала, который поддерживает запись на популярные платформы ведения журналов, такие как log4j и logback.

Например, если вы хотите использовать log4j как платформу ведения журналов, добавьте в путь к классу Java следующие библиотеки:

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>${slf4j.version}</version>
</dependency>
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>${log4j.version}</version>
</dependency>
```

Добавьте также конфигурацию log4j.
```
# this is a sample log4j configuration

# Set root logger level to INFO and its only appender to A1.
log4j.rootLogger=INFO, A1

log4j.category.com.azure.cosmos=INFO
#log4j.category.io.netty=OFF
#log4j.category.io.projectreactor=OFF
# A1 is set to be a ConsoleAppender.
log4j.appender.A1=org.apache.log4j.ConsoleAppender

# A1 uses PatternLayout.
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%d %5X{pid} [%t] %-5p %c - %m%n
```

Дополнительные сведения см. в [руководстве по ведению журналов с использованием sfl4j](https://www.slf4j.org/manual.html).

## <a name="os-network-statistics"></a><a name="netstats"></a>Сетевая статистика ОС
Запустите команду netstat, чтобы понять, сколько подключений находится в состоянии `ESTABLISHED`, `CLOSE_WAIT` и т. д.

В Linux вы можете запустить следующую команду:
```bash
netstat -nap
```

В Windows можно выполнить ту же команду, но с другими аргументами:
```bash
netstat -abn
```

Отфильтруйте результат для отображения только подключений к конечной точке Azure Cosmos DB.

Количество подключений к конечной точке Azure Cosmos DB в состоянии `ESTABLISHED` не должно превышать настроенного размера пула подключений.

Много подключений к конечной точке Azure Cosmos DB могут быть в состоянии `CLOSE_WAIT`. Возможно, более 1000. Такое большое количество указывает, что подключения быстро устанавливаются и быстро разрываются. Подобная ситуация может привести к проблемам. Дополнительные сведения см. в разделе [Распространенные проблемы и их обходные решения].

 <!--Anchors-->
[Распространенные проблемы и их обходные решения]: #common-issues-workarounds
[Enable client SDK logging]: #enable-client-sice-logging
[Ограничение числа подключений на узле]: #connection-limit-on-host
[Нехватка портов SNAT (PAT) Azure]: #snat
