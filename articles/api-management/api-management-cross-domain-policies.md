---
title: Политики междоменного доступа в службе управления API Azure | Документация Майкрософт
description: Сведения о политиках междоменного доступа, доступных для использования в службе управления API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 7689d277-8abe-472a-a78c-e6d4bd43455d
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/01/2021
ms.author: apimpm
ms.openlocfilehash: 85abf30d792b24b92685e191f5b460a42dc29142
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101688422"
---
# <a name="api-management-cross-domain-policies"></a>API Management cross domain policies (Междоменные политики службы управления API).
В этой статье рассматриваются приведенные ниже политики управления API. Дополнительные сведения о добавлении и настройке политик см. в статье о [политиках в управлении API](./api-management-policies.md).

## <a name="cross-domain-policies"></a><a name="CrossDomainPolicies"></a> Междоменные политики

- [Разрешение кросс-доменных вызовов](api-management-cross-domain-policies.md#AllowCrossDomainCalls) – делает API доступным из клиентов на основе браузеров Adobe Flash и Microsoft Silverlight.
- [CORS](api-management-cross-domain-policies.md#CORS) – добавляет поддержку общего доступа к ресурсам независимо от источника (CORS) для операции или API, чтобы разрешить кросс-доменные вызовы от клиентов на основе браузера.
- [JSONP](api-management-cross-domain-policies.md#JSONP) – добавляет поддержку JSON с заполнением (JSONP) для операции или API, чтобы разрешить кросс-доменные вызовы из браузерных клиентов JavaScript.

## <a name="allow-cross-domain-calls"></a><a name="AllowCrossDomainCalls"></a> Разрешить междоменные вызовы
Используйте политику `cross-domain`, чтобы разрешить доступ к API из браузерных клиентов на базе Adobe Flash и Microsoft Silverlight.

### <a name="policy-statement"></a>Правило политики

```xml
<cross-domain>
    <!-Policy configuration is in the Adobe cross-domain policy file format,
        see https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html-->
</cross-domain>
```

### <a name="example"></a>Пример

```xml
<cross-domain>
        <allow-http-request-headers-from domain='*' headers='*' />
</cross-domain>
```

### <a name="elements"></a>Элементы

|Название|Описание|Обязательно|
|----------|-----------------|--------------|
|cross-domain|Корневой элемент. Дочерние элементы должны соответствовать [спецификации Adobe для файлов политики междоменного доступа](https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html).|Да|

### <a name="usage"></a>Использование
Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

- **Разделы политики:** inbound.
- **Области политики:** все области.

## <a name="cors"></a><a name="CORS"></a> CORS
Политика `cors` добавляет поддержку общего доступа к ресурсам независимо от источника (CORS) для операции или API, чтобы разрешить междоменные вызовы из браузерных клиентов. 

> [!NOTE]
> Если запрос соответствует операции с методом OPTIONS, определенным в API, то Предпечатная логика обработки запроса, связанная с политиками CORS, не будет выполнена. Поэтому такие операции можно использовать для реализации пользовательской логики предварительной обработки.

CORS позволяет браузеру и серверу взаимодействовать друг с другом и определять, разрешать конкретные запросы независимо от источника или нет (то есть, вызовы XMLHttpRequests, отправляемые из кода JavaScript на веб-странице в другие домены). Это обеспечивает большую гибкость, чем просто разрешение запросов из одного источника, и более высокую защищенность, чем разрешение всех запросов независимо от источника.

Чтобы включить интерактивную консоль на портале разработчика, необходимо применить политику CORS. Дополнительные сведения см. в [документации по порталу разработчика](./api-management-howto-developer-portal.md#cors) .

### <a name="policy-statement"></a>Правило политики

```xml
<cors allow-credentials="false|true" terminate-unmatched-request="true|false">
    <allowed-origins>
        <origin>origin uri</origin>
    </allowed-origins>
    <allowed-methods preflight-result-max-age="number of seconds">
        <method>http verb</method>
    </allowed-methods>
    <allowed-headers>
        <header>header name</header>
    </allowed-headers>
    <expose-headers>
        <header>header name</header>
    </expose-headers>
</cors>
```

### <a name="example"></a>Пример
В этом примере показано, как обеспечить поддержку предварительных запросов, например с пользовательскими заголовками или методами, отличными от GET и POST. Для поддержки пользовательских заголовков и дополнительных команд HTTP используйте разделы `allowed-methods` и `allowed-headers`, как показано в следующем примере.

```xml
<cors allow-credentials="true">
    <allowed-origins>
        <!-- Localhost useful for development -->
        <origin>http://localhost:8080/</origin>
        <origin>http://example.com/</origin>
    </allowed-origins>
    <allowed-methods preflight-result-max-age="300">
        <method>GET</method>
        <method>POST</method>
        <method>PATCH</method>
        <method>DELETE</method>
    </allowed-methods>
    <allowed-headers>
        <!-- Examples below show Azure Mobile Services headers -->
        <header>x-zumo-installation-id</header>
        <header>x-zumo-application</header>
        <header>x-zumo-version</header>
        <header>x-zumo-auth</header>
        <header>content-type</header>
        <header>accept</header>
    </allowed-headers>
    <expose-headers>
        <!-- Examples below show Azure Mobile Services headers -->
        <header>x-zumo-installation-id</header>
        <header>x-zumo-application</header>
    </expose-headers>
</cors>
```

### <a name="elements"></a>Элементы

|Название|Описание|Обязательно|По умолчанию|
|----------|-----------------|--------------|-------------|
|cors|Корневой элемент.|Да|Н/Д|
|allowed-origins|Содержит элементы `origin`, описывающие разрешенные источники для междоменных запросов. `allowed-origins` может содержать один элемент `origin` со значением `*`, если нужно разрешить любые источники, либо один или несколько элементов `origin`, содержащих URI источников.|Да|Н/Д|
|Исходное подключение|В качестве значений допускается `*`, которое разрешает любые источники, или URI конкретного источника. URI-адрес должен включать схему, узел и порт.|Да|Если в URI не указан порт, используется порт 80 для HTTP и порт 443 для HTTPS.|
|allowed-methods|Этот элемент является обязательным, если разрешены методы, отличные от GET или POST. Содержит элементы `method`, перечисляющие поддерживаемые HTTP-команды. Значение `*` указывает все методы.|Нет|Если этот раздел отсутствует, поддерживаются методы GET и POST.|
|method|Задает команду HTTP.|Если раздел `allowed-methods` присутствует, в нем обязательно должен быть по крайней мере один элемент `method`.|Н/Д|
|allowed-headers|Этот элемент содержит элементы `header`, указывающие имена заголовков, которые могут быть включены в запрос.|Нет|Н/Д|
|expose-headers|Этот элемент содержит элементы `header`, указывающие имена заголовков, которые будут доступны клиенту.|Нет|Н/Д|
|заголовок|Задает имя заголовка.|Если присутствует раздел `allowed-headers` или `expose-headers`, в нем обязательно должен быть по крайней мере один элемент `header`.|Н/Д|

### <a name="attributes"></a>Атрибуты

|Имя|Описание|Обязательно|По умолчанию|
|----------|-----------------|--------------|-------------|
|allow-credentials|`Access-Control-Allow-Credentials`Для заголовка в предварительном ответе будет задано значение этого атрибута, что повлияет на возможность клиента отправлять учетные данные в междоменных запросах.|Нет|false|
|завершение-несоответствие-запрос|Этот атрибут управляет обработкой запросов между источниками, которые не соответствуют параметрам политики CORS. Когда запрос OPTIONS обрабатывается как предварительный запрос и не соответствует параметрам политики CORS: Если атрибут имеет значение `true` , немедленно завершите запрос с пустым ответом 200 ОК. Если атрибут имеет значение `false` , проверьте входящие для других политик CORS в области, которые являются прямыми дочерними элементами входящего элемента и применяют их.  Если политики CORS не найдены, завершите запрос с пустым ответом 200 ОК. Когда запрос GET или HEAD включает заголовок источника (и, следовательно, обрабатывается как запрос между источниками) и не соответствует параметрам политики CORS: Если атрибут имеет значение `true` , немедленно завершите запрос с пустым ответом 200 ОК. Если атрибут имеет значение `false` , разрешить выполнение запроса в обычном режиме и не добавлять в ответ заголовки CORS.|Нет|true|
|preflight-result-max-age|`Access-Control-Max-Age`Для заголовка в предварительном ответе будет задано значение этого атрибута, что повлияет на способность агента пользователя кэшировать ответ предварительного полета.|Нет|0|

### <a name="usage"></a>Использование
Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

- **Разделы политики:** inbound.
- **Области политики:** все области.

## <a name="jsonp"></a><a name="JSONP"></a> JSONP
Политика `jsonp` добавляет поддержку JSON с заполнением (JSONP) для операции или API, чтобы разрешить междоменные вызовы из браузерных клиентов на основе JavaScript. JSONP — это метод, который используется в программах JavaScript для запроса данных из сервера в другом домене. JSONP обходит ограничение, принудительно устанавливаемое большинством веб-браузеров, когда доступ к веб-страницам должен выполняться в том же домене.

### <a name="policy-statement"></a>Правило политики

```xml
<jsonp callback-parameter-name="callback function name" />
```

### <a name="example"></a>Пример

```xml
<jsonp callback-parameter-name="cb" />
```

Если метод вызывается без параметра обратного вызова ?cb=XXX, будет возвращаться простой JSON (без оболочки для вызовов функции).

Если добавить к запросу параметр обратного вызова `?cb=XXX`, возвращается результат JSONP, заключающий в исходные результаты JSON функцию обратного вызова, например, `XYZ('<json result goes here>');`

### <a name="elements"></a>Элементы

|Название|Описание|Обязательно|
|----------|-----------------|--------------|
|jsonp|Корневой элемент.|Да|

### <a name="attributes"></a>Атрибуты

|Имя|Описание|Обязательно|По умолчанию|
|----------|-----------------|--------------|-------------|
|callback-parameter-name|Кросс-доменный вызов функции JavaScript, в качестве префикса в котором присутствует полное имя домена, в котором находится функция.|Да|Н/Д|

### <a name="usage"></a>Использование
Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

- **Разделы политики:** outbound.
- **Области политики:** все области.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о работе с политиками см. в следующих статьях:

+ [Политики в управлении API](api-management-howto-policies.md)
+ [Преобразование API-интерфейсов](transform-api.md).
+ Полный перечень операторов политик и их параметров см. в [справочнике по политикам](./api-management-policies.md).
+ [Примеры политик](./policy-reference.md).
