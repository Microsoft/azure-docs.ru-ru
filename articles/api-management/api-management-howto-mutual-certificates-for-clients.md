---
title: Защита API с помощью проверки подлинности на основе сертификата клиента в управлении API
titleSuffix: Azure API Management
description: Узнайте, как защитить доступ к API с помощью сертификатов клиента. Для проверки входящих сертификатов можно использовать выражения политики.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/13/2020
ms.author: apimpm
ms.openlocfilehash: 553b4527796db3e5d0f430afd6c5e614626187e5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99988882"
---
# <a name="how-to-secure-apis-using-client-certificate-authentication-in-api-management"></a>Защита API-интерфейсов с помощью аутентификации на основе сертификата клиента в службе управления API Azure

Служба управления API помогает защитить доступ к API-интерфейсам (например, осуществляемый клиентом к службе управления API) с помощью сертификатов клиента. Вы можете проверить входящий сертификат и проверить свойства сертификата по нужным значениям с помощью выражений политики.

Сведения о защите доступа к внутренней службе API с помощью сертификатов клиента (т. е. управления API до серверной части) см. в статье [Защита серверных служб с помощью проверки подлинности на основе сертификата клиента](./api-management-howto-mutual-certificates.md) .

> [!IMPORTANT]
> Чтобы получить и проверить сертификаты клиента по протоколу HTTP/2 на уровнях Developer, Basic, Standard или Premium, необходимо включить параметр "согласовать сертификат клиента" в колонке "личные домены", как показано ниже.

![Согласование сертификата клиента](./media/api-management-howto-mutual-certificates-for-clients/negotiate-client-certificate.png)

> [!IMPORTANT]
> Чтобы получить и проверить сертификаты клиента на уровне потребления, необходимо включить параметр "запросить сертификат клиента" в колонке "личные домены", как показано ниже.

![Запрос сертификата клиента](./media/api-management-howto-mutual-certificates-for-clients/request-client-certificate.png)

## <a name="checking-the-issuer-and-subject"></a>Проверка издателя и субъекта

Следующие политики можно настроить для проверки издателя и субъекта сертификата клиента:

```xml
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify() || context.Request.Certificate.Issuer != "trusted-issuer" || context.Request.Certificate.SubjectName.Name != "expected-subject-name")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

> [!NOTE]
> Чтобы отключить проверку использования списка отзыва сертификатов `context.Request.Certificate.VerifyNoRevocation()` , используйте вместо `context.Request.Certificate.Verify()` .
> Если сертификат клиента является самозаверяющим, корневой (или промежуточный) сертификат ЦС необходимо [Отправить](api-management-howto-ca-certificates.md) в Управление API для `context.Request.Certificate.Verify()` и `context.Request.Certificate.VerifyNoRevocation()` для работы.

## <a name="checking-the-thumbprint"></a>Проверка отпечатка

Следующие политики можно настроить для проверки отпечатка сертификата клиента:

```xml
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify() || context.Request.Certificate.Thumbprint != "DESIRED-THUMBPRINT-IN-UPPER-CASE")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

> [!NOTE]
> Чтобы отключить проверку использования списка отзыва сертификатов `context.Request.Certificate.VerifyNoRevocation()` , используйте вместо `context.Request.Certificate.Verify()` .
> Если сертификат клиента является самозаверяющим, корневой (или промежуточный) сертификат ЦС необходимо [Отправить](api-management-howto-ca-certificates.md) в Управление API для `context.Request.Certificate.Verify()` и `context.Request.Certificate.VerifyNoRevocation()` для работы.

## <a name="checking-a-thumbprint-against-certificates-uploaded-to-api-management"></a>Проверка отпечатка на соответствие сертификатам, переданным в службу управления API

В примере ниже показано, как проверить отпечаток сертификата клиента на соответствие сертификатам, переданным в службу управления API.

```xml
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify()  || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>

```

> [!NOTE]
> Чтобы отключить проверку использования списка отзыва сертификатов `context.Request.Certificate.VerifyNoRevocation()` , используйте вместо `context.Request.Certificate.Verify()` .
> Если сертификат клиента является самозаверяющим, корневой (или промежуточный) сертификат ЦС необходимо [Отправить](api-management-howto-ca-certificates.md) в Управление API для `context.Request.Certificate.Verify()` и `context.Request.Certificate.VerifyNoRevocation()` для работы.

> [!TIP]
> Проблема взаимоблокировки сертификата клиента, описанная в этой [статье](https://techcommunity.microsoft.com/t5/Networking-Blog/HTTPS-Client-Certificate-Request-freezes-when-the-Server-is/ba-p/339672) , может быть манифестом несколькими способами, например запросы на замораживание, запросы, вызывающие `403 Forbidden` код состояния после истечения времени ожидания, `context.Request.Certificate` — `null` . Эта проблема обычно влияет `POST` на `PUT` запросы с длиной содержимого приблизительно 60KB или выше.
> Чтобы предотвратить возникновение этой проблемы, включите параметр "согласовать сертификат клиента" для нужных имен узлов в колонке "личные домены", как показано на первом рисунке этого документа. Эта функция недоступна в уровне потребления.

## <a name="certificate-validation-in-self-hosted-gateway"></a>Проверка сертификата в самостоятельно размещенном шлюзе

Образ [шлюза, размещенного](self-hosted-gateway-overview.md) в службе управления API по умолчанию, не поддерживает проверку сертификатов сервера и клиента с помощью [корневых сертификатов ЦС](api-management-howto-ca-certificates.md) , передаваемых экземпляру службы управления API. Клиенты, которые представляют пользовательский сертификат для самостоятельно размещенного шлюза, могут столкнуться с медленными ответами, так как проверка списка отзыва сертификатов (CRL) может занять много времени на шлюзе. 

В качестве обходного решения при запуске шлюза можно настроить IP-адрес PKI таким образом, чтобы он указывал на адрес localhost (127.0.0.1), а не на экземпляр службы управления API. Это приводит к тому, что проверка списка отзыва сертификатов быстро завершается сбоем, когда шлюз пытается проверить сертификат клиента. Чтобы настроить шлюз, добавьте запись DNS для экземпляра службы управления API, которая будет разрешаться в localhost в `/etc/hosts` файле в контейнере. Эту запись можно добавить во время развертывания шлюза:
 
* Для развертывания DOCKER — добавьте `--add-host <hostname>:127.0.0.1` параметр в `docker run` команду. Дополнительные сведения см. [в разделе Добавление записей в контейнер файл hosts](https://docs.docker.com/engine/reference/commandline/run/#add-entries-to-container-hosts-file---add-host) .
 
* Для развертывания Kubernetes `hostAliases` . Добавьте спецификацию в `myGateway.yaml` файл конфигурации. Дополнительные сведения см. [в разделе Добавление записей в Pod/etc/hosts с псевдонимами узлов](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/).




## <a name="next-steps"></a>Дальнейшие действия

-   [Как защитить серверные службы с помощью аутентификации на основе сертификата клиента](./api-management-howto-mutual-certificates.md)
-   [Как передавать сертификаты](./api-management-howto-mutual-certificates.md)
