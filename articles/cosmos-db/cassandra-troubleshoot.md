---
title: Устранение распространенных ошибок в Azure Cosmos DB API Cassandra
description: В этом документе обсуждаются способы устранения распространенных проблем, возникающих в Azure Cosmos DB API Cassandra
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: troubleshooting
ms.date: 03/02/2021
ms.author: thvankra
ms.openlocfilehash: f9b6e586879b8697660ced7aa6f1e75083e3ee29
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101658577"
---
# <a name="troubleshoot-common-issues-in-azure-cosmos-db-cassandra-api"></a>Устранение распространенных неполадок в Azure Cosmos DB API Cassandra
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

API Cassandra в Azure Cosmos DB — это уровень совместимости, который обеспечивает [поддержку протокола проводного](cassandra-support.md) подключения для популярной базы данных Apache Cassandra с открытым кодом и работает с [Azure Cosmos DB](./introduction.md). В качестве полностью управляемой облачной службы Azure Cosmos DB предоставляет [гарантии доступности, пропускной способности и согласованности](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_3/) для API Cassandra. Эти гарантии невозможно реализовать в устаревших реализациях Apache Cassandra. API Cassandra также упрощает операции с платформой нулевого обслуживания и исправление без простоя. Таким образом, многие из внутренних серверных операций отличаются от Apache Cassandra, поэтому мы рекомендуем использовать определенные параметры и подходы, чтобы избежать распространенных ошибок. 

В этой статье описаны распространенные ошибки и решения для приложений, использующих Azure Cosmos DB API Cassandra. Если ошибка не указана ниже и возникла ошибка при выполнении [поддерживаемой операции в API Cassandra](cassandra-support.md), где ошибка отсутствует *при использовании собственного Apache Cassandra*, [Создайте запрос в службу поддержки Azure](../azure-portal/supportability/how-to-create-azure-support-request.md).

## <a name="nonodeavailableexception"></a>нонодеаваилабликсцептион
Это исключение оболочки верхнего уровня с большим количеством возможных причин и внутренних исключений, многие из которых могут быть связаны с клиентом. 
### <a name="solution"></a>Решение
Ниже приведены некоторые популярные причины и решения. 
- Время ожидания простоя Azure LoadBalancers: это также может быть манифестом `ClosedConnectionException` . Чтобы устранить эту проблему, установите параметр проверки активности в разделе драйвер (см. [ниже](#enable-keep-alive-for-java-driver)) и увеличьте параметры поддержания активности в операционной системе или [Настройте время ожидания простоя в Azure Load Balancer](../load-balancer/load-balancer-tcp-idle-timeout.md?tabs=tcp-reset-idle-portal). 
- **Исчерпание ресурсов клиентских приложений.** убедитесь, что на клиентских компьютерах достаточно ресурсов для выполнения запроса. 

## <a name="cannot-connect-to-host"></a>Не удается подключиться к узлу
Может отобразиться следующее сообщение об ошибке: `Cannot connect to any host, scheduling retry in 600000 milliseconds` . 

### <a name="solution"></a>Решение
Это может быть нехватка SNAT на стороне клиента. Чтобы устранить эту ошибку, выполните действия, указанные в [SNAT для исходящих подключений](../load-balancer/load-balancer-outbound-connections.md) . Это также может быть проблемой времени ожидания простоя, когда в подсистеме балансировки нагрузки Azure по умолчанию используется 4 минуты ожидания простоя. См. документацию во [время ожидания простоя подсистемы балансировки нагрузки](../load-balancer/load-balancer-tcp-idle-timeout.md?tabs=tcp-reset-idle-portal). Включите функцию поддержания активности TCP в параметрах драйвера (см. [ниже](#enable-keep-alive-for-java-driver)) и задайте `keepAlive` для параметра Интервал в операционной системе значение менее 4 минут.

 

## <a name="overloadedexception-java"></a>Оверлоадедексцептион (Java)
Общее число потребляемых единиц запросов превышает количество единиц запросов, подготовленных для пространства ключей или таблицы. Поэтому запросы будут регулироваться.
### <a name="solution"></a>Решение
Рассмотрите возможность масштабирования пропускной способности, назначенной пространства ключей или таблице, из портал Azure (см. [здесь](manage-scale-cassandra.md) для операций масштабирования в API Cassandra) или можно реализовать политику повтора. Сведения об Java см. в статье примеры повторных попыток для драйвера [v3. x](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample) и [драйвера v4. x](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample-v4). См. также [расширения Azure Cosmos Cassandra для Java](https://github.com/Azure/azure-cosmos-cassandra-extensions).

### <a name="overloadedexception-even-with-sufficient-throughput"></a>Оверлоадедексцептион даже с достаточной пропускной способностью 
Система выглядит как запросы регулирования, несмотря на достаточную пропускную способность, необходимую для объема запросов и/или потребленных затрат на единицу запроса. Существует две возможные причины непредвиденного ограничения частоты.
- **Операции на уровне схемы:** API Cassandra реализует бюджет пропускной способности системы для операций на уровне схемы (CREATE TABLE, ALTER TABLE, DROP TABLE). Этот бюджет должен быть достаточно для операций с схемами в рабочей системе. Однако при наличии большого количества операций на уровне схемы возможно превышение этого предела. Так как этот бюджет не контролируется пользователем, необходимо рассмотреть возможность снижения числа выполняемых операций с схемой. Если это действие не устраняет проблему или не является целесообразным для вашей рабочей нагрузки, [Создайте запрос в службу поддержки Azure](../azure-portal/supportability/how-to-create-azure-support-request.md).
- **Неравномерное распределение данных.** когда пропускная способность подготавливается в API Cassandra, она делится между физическими секциями, а каждая физическая секция имеет верхний предел. При наличии большого объема данных, вставляемых или запрашиваемых из одного конкретного раздела, можно ограничить скорость, несмотря на подготовку большого количества общей пропускной способности (единиц запроса) для этой таблицы. Проверьте модель данных и убедитесь в отсутствии чрезмерных отклонений, которые могут вызвать горячую секцию. 

## <a name="intermittent-connectivity-errors-java"></a>Ошибки периодического подключения (Java) 
Неожиданное завершение соединения или время его ожидания.

### <a name="solution"></a>Решение 
Драйверы Apache Cassandra для Java предоставляют две встроенные политики переподключения: `ExponentialReconnectionPolicy` и `ConstantReconnectionPolicy` . Значение по умолчанию — `ExponentialReconnectionPolicy`. Однако для Azure Cosmos DB API Cassandra рекомендуется использовать `ConstantReconnectionPolicy` задержку в 2 секунды. См. [документацию](https://docs.datastax.com/en/developer/java-driver/4.9/manual/core/reconnection/)  по драйверу для Java v4. x [и руководство по Java](https://docs.datastax.com/en/developer/java-driver/3.7/manual/reconnection/) 3. x. Кроме того, см. инструкции по [настройке драйвера реконнектионполици для Java](#configuring-reconnectionpolicy-for-java-driver) ниже.

## <a name="error-with-load-balancing-policy"></a>Ошибка с политикой балансировки нагрузки

Если вы реализовали политику балансировки нагрузки в версии v3. x драйвера Datastax для Java, с кодом, аналогичным приведенному ниже:

```java
cluster = Cluster.builder()
        .addContactPoint(cassandraHost)
        .withPort(cassandraPort)
        .withCredentials(cassandraUsername, cassandraPassword)
        .withPoolingOptions(new PoolingOptions() .setConnectionsPerHost(HostDistance.LOCAL, 1, 2)
                .setMaxRequestsPerConnection(HostDistance.LOCAL, 32000).setMaxQueueSize(Integer.MAX_VALUE))
        .withSSL(sslOptions)
        .withLoadBalancingPolicy(DCAwareRoundRobinPolicy.builder().withLocalDc("West US").build())
        .withQueryOptions(new QueryOptions().setConsistencyLevel(ConsistencyLevel.LOCAL_QUORUM))
        .withSocketOptions(getSocketOptions())
        .build();
```

Если значение для `withLocalDc()` не соответствует центру обработки данных контактной точки, может возникнуть очень временная ошибка: `com.datastax.driver.core.exceptions.NoHostAvailableException: All host(s) tried for query failed (no host was tried)` . 

### <a name="solution"></a>Решение 
Реализуйте [космослоадбаланЦингполици](https://github.com/Azure/azure-cosmos-cassandra-extensions/blob/master/package/src/main/java/com/microsoft/azure/cosmos/cassandra/CosmosLoadBalancingPolicy.java) (возможно, потребуется обновить дополнительный номер версии datastax, чтобы сделать ее работоспособной):

```java
LoadBalancingPolicy loadBalancingPolicy = new CosmosLoadBalancingPolicy.Builder().withWriteDC("West US").withReadDC("West US").build();
```

## <a name="count-fails-on-large-table"></a>Сбой счетчика в большой таблице
При запуске `select count(*) from table` или аналогичном для большого количества строк время ожидания сервера истекает.

### <a name="solution"></a>Решение 
При использовании локального клиента CQLSH можно попытаться изменить `--connect-timeout` `--request-timeout` Параметры или (см. Дополнительные сведения [здесь](https://cassandra.apache.org/doc/latest/tools/cqlsh.html)). Если это не так и количество по-прежнему истекает, можно получить число записей из Azure Cosmos DB данных телеметрии серверной части, перейдя на вкладку метрики в портал Azure, выбрав метрику `document count` , а затем добавив фильтр для базы данных или коллекции (аналог таблицы в Azure Cosmos DB). Затем можно навести указатель мыши на полученный граф на момент времени, в котором нужно получить количество записей.

:::image type="content" source="./media/cassandra-troubleshoot/metrics.png" alt-text="Представление метрик":::


## <a name="configuring-reconnectionpolicy-for-java-driver"></a>Настройка драйвера Реконнектионполици для Java

### <a name="version-3x"></a>Версия 3.x

Для версии 3. x драйвера Java настройте политику повторного подключения при создании объекта кластера.

```java
import com.datastax.driver.core.policies.ConstantReconnectionPolicy;

Cluster.builder()
  .withReconnectionPolicy(new ConstantReconnectionPolicy(2000))
  .build();
```

### <a name="version-4x"></a>Версия 4. x

Для версии 4. x драйвера Java настройте политику повторного подключения, переопределив параметры в `reference.conf` файле:

```xml
datastax-java-driver {
  advanced {
    reconnection-policy{
      # The driver provides two implementations out of the box: ExponentialReconnectionPolicy and
      # ConstantReconnectionPolicy. We recommend ConstantReconnectionPolicy for Cassandra API, with 
      # base-delay of 2 seconds.
      class = ConstantReconnectionPolicy
      base-delay = 2 second
    }
}
```

## <a name="enable-keep-alive-for-java-driver"></a>Включение проверки активности для драйвера Java

### <a name="version-3x"></a>Версия 3.x

Для версии 3. x драйвера Java установите флажок "оставаться в активном состоянии" при создании объекта кластера и убедитесь, что [в операционной системе включен](https://knowledgebase.progress.com/articles/Article/configure-OS-TCP-KEEPALIVE-000080089)параметр проверки активности.

```java
import java.net.SocketOptions;
    
SocketOptions options = new SocketOptions();
options.setKeepAlive(true);
cluster = Cluster.builder().addContactPoints(contactPoints).withPort(port)
  .withCredentials(cassandraUsername, cassandraPassword)
  .withSocketOptions(options)
  .build();
```

### <a name="version-4x"></a>Версия 4. x

Для версии 4. x драйвера Java установите параметр "оставаться в активном состоянии", переопределив параметры в `reference.conf` и установив [флажок "Проверка активности" в операционной системе](https://knowledgebase.progress.com/articles/Article/configure-OS-TCP-KEEPALIVE-000080089).

```xml
datastax-java-driver {
  advanced {
    socket{
      keep-alive = true
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о [поддерживаемых функциях](cassandra-support.md) в Azure Cosmos DB API Cassandra.
- Узнайте, как [перейти с машинного кода Apache Cassandra на Azure Cosmos DB API Cassandra](cassandra-migrate-cosmos-db-databricks.md)
