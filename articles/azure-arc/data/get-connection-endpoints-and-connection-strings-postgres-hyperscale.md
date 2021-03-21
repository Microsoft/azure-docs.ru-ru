---
title: Получение конечных точек подключения и формы строки подключения для PostgreSQL группы серверов с поддержкой Arc
titleSuffix: Azure Arc enabled data services
description: Получение конечных точек подключения и формы строки подключения для PostgreSQL группы серверов с поддержкой Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: de7d23689ae984ea0abece5edb03cf8a0c3a9be1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104670347"
---
# <a name="get-connection-endpoints-and-form-connection-strings-for-your-arc-enabled-postgresql-hyperscale-server-group"></a>Получение конечных точек подключения и формы строки подключения для PostgreSQL группы серверов с поддержкой Arc

В этой статье объясняется, как можно получить конечные точки подключения для группы серверов и как сформировать строки подключения, которые будут использоваться в приложениях и (или) средствах.


[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="get-connection-end-points"></a>Получить конечные точки подключения:

### <a name="from-cli-with-azdata"></a>Из CLI с аздата
#### <a name="1-connect-to-your-arc-data-controller"></a>1. Подключитесь к контроллеру данных ARC:
- Если на узле контроллера данных Arc уже открыт сеанс, выполните следующую команду:
```console
azdata login
```

- Если сеанс не открыт на узле контроллера данных Arc, выполните следующую команду: 
```console
azdata login --endpoint https://<external IP address of host/data controller>:30080
```

#### <a name="2-show-the-connection-endpoints"></a>2. Отображение конечных точек подключения
Выполните следующую команду.
```console
azdata arc postgres endpoint list -n <server group name>
```
Например:
```console
azdata arc postgres endpoint list -n postgres01
```

В нем отображается список конечных точек: конечная точка PostgreSQL, используемая для подключения приложения и использования конечных точек базы данных, Kibana и Grafana для log Analytics и мониторинга. Пример: 
```console
Arc
 ===================================================================================================================
 Postgres01 Instance
 -------------------------------------------------------------------------------------------------------------------
 Description           Endpoint

 PostgreSQL Instance   postgresql://postgres:<replace with password>@12.345.567.89:5432
 Log Search Dashboard  https://89.345.712.81:30777/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:postgres01'))
 Metrics Dashboard     https://89.345.712.81:30777/grafana/d/postgres-metrics?var-Namespace=arc&var-Name=postgres01

```
Используйте эти конечные точки для:
- Формирование строк подключения и подключение к клиентским средствам или приложениям
- Доступ к панелям мониторинга Grafana и Kibana из браузера

Например, конечную точку с именем _PostgreSQL instance_ можно использовать для подключения к группе серверов с помощью psql. Пример:
```console
psql postgresql://postgres:MyPassworkd@12.345.567.89:5432
psql (10.14 (Ubuntu 10.14-0ubuntu0.18.04.1), server 12.4 (Ubuntu 12.4-1.pgdg16.04+1))
WARNING: psql major version 10, server major version 12.
         Some psql features might not work.
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=#
```
> [!NOTE]
>
> - Пароль пользователя _postgres_ , указанный в конечной точке с именем "_PostgreSQL instance_", — это пароль, выбранный при развертывании группы серверов.
> - О аздата. Аренда, связанная с вашим подключением, длится примерно 10 часов. После этого необходимо выполнить повторное подключение. Если срок аренды истек, вы получите следующее сообщение об ошибке при попытке выполнить команду с аздата (кроме аздата login): _Ошибка: (401)_ 
>  _Причина: неавторизованные_ 
>  _заголовки ответа HTTP: хттфеадердикт ({' date ': ' Sun, 06 Sep 2020 16:58:38 GMT "," Content-Length ":" 0 "," www-Аутентификация ":"_ 
>  _Basic realm = "_ требуется учетные данные для входа", ошибка носителя = "invalid_token", error_description = "срок действия маркера истек"}) _ в этом случае необходимо повторно подключиться к аздата, как описано выше.

## <a name="from-cli-with-kubectl"></a>Из CLI с kubectl
- Если группа серверов имеет значение postgres версии 12 (по умолчанию), выполните следующую команду:
```console
kubectl get postgresql-12/<server group name> -n <namespace name>
```
- Если группа серверов имеет версию postgres 11, выполните следующую команду:
```console
kubectl get postgresql-11/<server group name> -n <namespace name>
```

Эти команды выдают выходные данные, как показано ниже. Эти сведения можно использовать для формирования строк подключения:
```console
NAME         STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgres01   Ready   3/3          123.456.789.4:31066      5d20h
``` 


## <a name="form-connection-strings"></a>Форма строк подключения:
Используйте приведенную ниже таблицу шаблонов строк подключения для группы серверов. Затем их можно скопировать и вставить и настроить по своему усмотрению:

### <a name="adonet"></a>ADO.NET

```ado.net
Server=192.168.1.121;Database=postgres;Port=24276;User Id=postgres;Password={your_password_here};Ssl Mode=Require;`
```

### <a name="c-libpq"></a>C++ (либпк)

```cpp
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
```

### <a name="jdbc"></a>JDBC

```jdbc
jdbc:postgresql://192.168.1.121:24276/postgres?user=postgres&password={your_password_here}&sslmode=require
```

### <a name="nodejs"></a>Node.js

```node.js
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
```

### <a name="php"></a>PHP

```php
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
```

### <a name="psql"></a>psql

```psql
psql "host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require"
```

### <a name="python"></a>Python

```python
dbname='postgres' user='postgres' host='192.168.1.121' password='{your_password_here}' port='24276' sslmode='true'
```

### <a name="ruby"></a>Ruby

```ruby
host=192.168.1.121; dbname=postgres user=postgres password={your_password_here} port=24276 sslmode=require
```

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о [масштабировании (Добавление рабочих узлов)](scale-out-postgresql-hyperscale-server-group.md) в группу серверов
- Дополнительные сведения о [масштабировании (увеличение или уменьшение объема памяти и виртуальных ядер)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) для группы серверов


