---
title: Apache Kafkaное руководством для разработчиков концентраторов событий
description: В этой статье приведены ссылки на статьи, в которых описано, как интегрировать приложения Kafka с концентраторами событий Azure.
ms.date: 06/23/2020
ms.topic: article
ms.openlocfilehash: 757db4faf2a933e366bc818d5cf6aab04d0b08ba
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90061739"
---
# <a name="apache-kafka-developer-guide-for-azure-event-hubs"></a>Apache Kafka Guide для разработчиков концентраторов событий Azure
В этой статье содержатся ссылки на статьи, в которых описывается интеграция приложений Apache Kafka с концентраторами событий Azure. 

## <a name="overview"></a>Обзор
Центры событий предоставляют конечную точку Kafka, которую существующие приложения на основе Kafka могут использовать в качестве альтернативы запуска собственного кластера Kafka. Концентраторы событий работают с многими существующими Kafka приложениями. Дополнительные сведения см. в разделе [концентраторы событий для Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md)

## <a name="quickstarts"></a>Краткие руководства
Вы можете найти краткие руководства на сайте GitHub и в этом наборе содержимого, которые помогут быстро освоить концентраторы событий для Kafka.

### <a name="quickstarts-in-github"></a>Краткие руководства в GitHub
См. следующие краткие руководства в репозитории **Azure-Event-Hub-for-Kafka** . 

| Язык/платформа клиента | Описание | 
| ------------------------- | ----------- | 
| [.NET](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/dotnet) | <p>В этом кратком руководстве показано, как создать и подключиться к конечной точке Kafka концентраторов событий с помощью примера производителя и потребителя, написанного на C# с помощью .NET Core 2,0.</p><p>Этот пример основан на [Apache Kafka клиента .NET](https://github.com/confluentinc/confluent-kafka-dotnet), измененном для использования с концентраторами событий для Kafka.</p> | 
| [Java](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/java) | В этом кратком руководстве показано, как создать и подключиться к конечной точке Kafka концентраторов событий с помощью примера производителя и потребителя, написанного на языке Java. |
| [Node.js](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/node) | <p>В этом кратком руководстве показано, как создать и подключиться к конечной точке Kafka концентраторов событий с помощью примера производителя и потребителя, написанного на узле.</p><p>В этом примере используется библиотека [node-рдкафка](https://github.com/Blizzard/node-rdkafka) . </p>| 
| [Python](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/python) | <p>В этом кратком руководстве показано, как создать и подключиться к конечной точке Kafka концентраторов событий с помощью примера производителя и потребителя, написанного на Python.</p><p>Этот пример основан на [клиенте Apache Kafka Python](https://github.com/confluentinc/confluent-kafka-python), который был изменен для использования с концентраторами событий для Kafka.</p>|
| [GO](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/go) | <p>В этом кратком руководстве показано, как создать и подключиться к конечной точке Kafka концентраторов событий с помощью примера производителя и потребителя, написанного на go.</p><p>Этот пример основан на [Apache Kafka клиента Golang](https://github.com/confluentinc/confluent-kafka-go), измененном для использования с концентраторами событий для Kafka.</p>| 
| [Сарама Kafka](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/go-sarama-client) | В этом кратком руководстве показано, как создать и подключиться к конечной точке Kafka концентраторов событий с помощью примера производителя и потребителя, написанного на Go с помощью [клиентской библиотеки Сарама Kafka](https://github.com/Shopify/sarama) . |
| [Kafka](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafka-cli) | В этом кратком руководстве показано, как создать и подключиться к конечной точке Kafka концентраторов событий с помощью интерфейса командной строки, который входит в состав дистрибутива Apache Kafka.| 
| [кафкакат](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafkacat) | кафкакат — это не ВИРТУАЛЬНОЙ машины JAVAная в командной строке потребитель и производитель, основанный на либрдкафка, популярная из-за скорости и небольшого объема памяти. Это краткое руководство содержит образец конфигурации и несколько простых примеров кафкакат. | 
 
### <a name="quickstarts-in-docs"></a>Краткие руководства по ДОКУМЕНТам
Пошаговые инструкции по потоковой передаче в концентраторы событий см. в кратком руководстве: [потоковая передача данных с концентраторами событий с помощью протокола Kafka](event-hubs-quickstart-kafka-enabled-event-hubs.md) в этом наборе содержимого. Вы узнаете, как обеспечить взаимодействие отправителей и объектов-получателей с Центрами событий, изменив конфигурацию в приложениях. 


## <a name="tutorials"></a>Учебники 

### <a name="tutorials-in-github"></a>Учебники на GitHub
Ознакомьтесь со следующими учебниками на GitHub:

| Учебник | Описание | 
| ------------------------- | ----------- | 
| [Akka Streams](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/akka/java) | В этом руководстве показано, как подключить потоки Akka Streams к концентраторам событий с поддержкой Kafka, не изменяя Клиенты протокола или не запуская собственные кластеры. Существует два отдельных учебника, использующих языки программирования **Java** и **Scala** . | 
| [Подключить](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/connect) | В этом документе вы узнаете, как интегрировать Kafka Connect с концентраторами событий Azure и развертывать базовые соединители Филестреамсаурце и Филестреамсинк. Хотя эти соединители не предназначены для использования в рабочей среде, они демонстрируют комплексный сценарий Kafka Connect, в котором концентраторы событий Azure маскируются в качестве брокера Kafka.| 
| [филебеат](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/filebeat) | В этом документе приводится пошаговое руководство по интеграции Филебеат и концентраторов событий с помощью выходных данных Kafka Филебеат. | 
| [флинк](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/flink) | В этом руководстве показано, как подключить Apache Флинк к концентраторам событий с поддержкой Kafka, не изменяя Клиенты протокола или не запуская собственные кластеры. | 
| [FluentD](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/fluentd) | В этом документе приводится пошаговое руководство по интеграции концентраторов событий в свободном центре с использованием `out_kafka` выходного подключаемого модуля для свободно. |
| [Interop](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/interop) | В этом руководстве показано, как обмениваться событиями между потребителями и производителями с помощью разных протоколов. |
| [Logstash](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/logstash). | В этом руководстве описано, как интегрировать Logstash с концентраторами событий с поддержкой Kafka с помощью Logstash Kafka Input/Output plugins. | 
| [MirrorMaker](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/mirror-maker) | В этом руководстве показано, как концентратор событий и Kafka MirrorMaker могут интегрировать существующий конвейер Kafka в Azure, выполнив зеркальное отображение входного потока Kafka в службе концентраторов событий. |
| [NiFi](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/nifi) | В этом руководстве показано, как подключить Apache NiFi к пространству имен концентраторов событий. | 
| [OAuth](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth) | В кратком руководстве показано, как создать и подключиться к конечной точке Kafka концентраторов событий с помощью примера производителя и потребителя, написанного на языке программирования Go и Java. |
| [Реестр схемы, отличный от Fluent](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/schema-registry) | Из этого руководства вы узнаете, как интегрировать реестр схемы и концентраторы событий для Kafka. | 
| [Spark](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/spark) | В этом учебнике показано, как подключить приложение Spark к концентратору событий, не изменяя Клиенты протокола или не запуская собственные кластеры Kafka. | 

### <a name="tutorials-in-docs"></a>Учебники по ДОКУМЕНТам
Кроме того, см. Руководство по [обработке Apache Kafka для событий концентраторов событий с помощью Stream Analytics](event-hubs-kafka-stream-analytics.md) в этом наборе содержимого, в котором показано, как выполнять потоковую передачу данных в концентраторы событий и обрабатывать их с помощью Azure Stream Analytics.

## <a name="how-to-guides"></a>Практические руководства
Ознакомьтесь со следующими руководствами в нашей документации:

| Статья | Описание | 
| ------- | ----------- | 
| [Зеркальное отображение брокера Kafka в концентраторе событий](event-hubs-kafka-mirror-maker-tutorial.md) | Показывает, как зеркально отобразить брокер Kafka в концентраторе событий с помощью Kafka MirrorMaker. |
| [Подключение Apache Spark к концентратору событий](event-hubs-kafka-spark-tutorial.md) | Пошаговое руководство по подключению приложения Spark к концентраторам событий для потоковой передачи в режиме реального времени. |
| [Подключение Apache Flink к концентратору событий](event-hubs-kafka-flink-tutorial.md) | Показано, как подключить Apache Флинк к концентратору событий, не изменяя Клиенты протокола или не запуская собственные кластеры. |
| [Интеграция Apache Kafka подключение к концентратору событий (Предварительная версия)](event-hubs-kafka-connect-tutorial.md) | Руководство по интеграции Kafka Connect с концентратором событий и развертыванию базовых соединителей Филестреамсаурце и Филестреамсинк. |
| [Подключение Akka Streams к концентратору событий](event-hubs-kafka-akka-streams-tutorial.md) | Показано, как подключить потоки Akka Streams к концентратору событий, не изменяя Клиенты протокола или не запуская собственные кластеры. |
| [Использование начального Boot Starter для Apache Kafka с концентраторами событий Azure](/azure/developer/java/spring-framework/configure-spring-cloud-stream-binder-java-app-kafka-azure-event-hub) | Демонстрируется настройка связывателя облачного потока весны на основе Java, созданного с помощью инициализатора пружинной загрузки, для использования Apache Kafka с концентраторами событий Azure. |

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с примерами в разделе "Краткое руководство и учебники [Azure-Event-Hubs-for-Kafka](https://github.com/Azure/azure-event-hubs-for-kafka) " в репозитории GitHub.

См. также следующие статьи:

- [Руководство по устранению неполадок Apache Kafka для концентраторов событий](apache-kafka-troubleshooting-guide.md)
- [Часто задаваемые вопросы — концентраторы событий для Apache Kafka](apache-kafka-frequently-asked-questions.md)
- [Apache Kafka руководством по миграции для концентраторов событий](apache-kafka-migration-guide.md)



