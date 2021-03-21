---
title: Общие сведения и решение проблем AD Application Proxy CORS в Azure
description: Общие сведения об CORS в Azure AD Application Proxy, а также о том, как выявление и устранение проблем CORS.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 05/23/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: b57fc7e3af99819c9b27b6bc796e501d1db02818
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99259174"
---
# <a name="understand-and-solve-azure-active-directory-application-proxy-cors-issues"></a>Понимание и решение проблем с механизмом CORS Application Proxy для Azure Active Directory

[Общий доступ к ресурсам между источниками (CORS)](https://www.w3.org/TR/cors/) иногда может представлять проблемы для приложений и API, которые публикуются с помощью Azure Active Directory Application proxy. В этой статье обсуждаются проблемы и решения Azure AD Application Proxy CORS.

Безопасность в браузере обычно предотвращает выполнение запросов AJAX к другому домену на веб-странице. Это ограничение называется *политикой того же происхождения* и предотвращает чтение вредоносных данных с другого сайта злоумышленником. Однако иногда может потребоваться разрешить другим сайтам вызывать веб-API. CORS — это стандарт консорциума W3C, который позволяет серверу использовать политику одинакового происхождения и разрешать некоторые запросы между источниками, а также отклонять другие.

## <a name="understand-and-identify-cors-issues"></a>Понимание и определение проблем CORS

Два URL-адреса имеют одинаковый источник, если они имеют идентичные схемы, узлы и порты ([RFC 6454](https://tools.ietf.org/html/rfc6454)), например:

-   http: \/ /contoso.com/foo.html
-   http: \/ /contoso.com/bar.html

Следующие URL-адреса имеют разные источники, отличные от предыдущих двух:

-   http: \/ /contoso.NET-другой домен
-   http: \/ /contoso.com:9000/foo.html-другой порт
-   HTTPS: \/ /contoso.com/foo.html — другая схема
-   http: \/ /www.contoso.com/foo.html — другой поддомен

Политика с одним источником предотвращает доступ приложений к ресурсам из других источников, если они не используют правильные заголовки управления доступом. Если заголовки CORS отсутствуют или неверны, запросы между источниками завершаются сбоем. 

Проблемы CORS можно выявление с помощью средств отладки браузера:

1. Запустите браузер и перейдите к веб-приложению.
1. Нажмите клавишу **F12**, чтобы открыть консоль отладки.
1. Попробуйте возобновить транзакцию и просмотрите сообщение консоли. Нарушение CORS приводит к ошибке консоли в отношении источника.

На следующем снимке экрана при нажатии кнопки **попробовать** вызвать сообщение об ошибке CORS, что HTTPS: \/ /corswebclient-contoso.msappproxy.NET не найден в заголовке Access-Control-Allow-Origin.

![Проблемы CORS](./media/application-proxy-understand-cors-issues/image3.png)

## <a name="cors-challenges-with-application-proxy"></a>Проблемы CORS с прокси приложения

В следующем примере показан типичный сценарий Azure AD Application Proxy CORS. На внутреннем сервере размещен контроллер веб-API **корсвебсервице** , а также **корсвебклиент** , который вызывает **корсвебсервице**. Существует запрос AJAX от **корсвебклиент** к **корсвебсервице**.

![Локальный запрос на основе одного источника](./media/application-proxy-understand-cors-issues/image1.png)

Приложение Корсвебклиент работает, когда вы размещаете его локально, но не удается загрузить или выработать с ошибками при публикации с помощью Azure AD Application Proxy. Если вы опубликовали приложения Корсвебклиент и Корсвебсервице отдельно как различные приложения через прокси приложения, эти два приложения размещаются в разных доменах. Запрос AJAX от Корсвебклиент к Корсвебсервице является запросом между источниками и завершается с ошибкой.

![Запрос CORS прокси приложения](./media/application-proxy-understand-cors-issues/image2.png)

## <a name="solutions-for-application-proxy-cors-issues"></a>Решения для проблем прокси приложения CORS

Описанную выше проблему CORS можно устранить одним из нескольких способов.

### <a name="option-1-set-up-a-custom-domain"></a>Вариант 1. Настройка пользовательского домена

Используйте [пользовательский домен](./application-proxy-configure-custom-domain.md) AD application proxy Azure для публикации из того же источника, не внося никаких изменений в источники приложений, код или заголовки. 

### <a name="option-2-publish-the-parent-directory"></a>Вариант 2. Публикация родительского каталога

Опубликуйте родительский каталог обоих приложений. Это решение особенно удобно, если на веб-сервере имеется только два приложения. Вместо публикации каждого приложения по отдельности можно опубликовать общий родительский каталог, в котором будет находиться тот же источник.

В следующих примерах показана страница AD Application Proxy портала Azure для приложения Корсвебклиент.  Если для **внутреннего URL-адреса** задано значение *contoso.com/CORSWebClient*, приложение не сможет выполнить успешные запросы к каталогу *contoso.com/CORSWebService* , так как они находятся в разных источниках. 

![Публикация приложения по отдельности](./media/application-proxy-understand-cors-issues/image4.png)

Вместо этого задайте **внутренний URL-адрес** для публикации родительского каталога, который включает в себя каталоги *корсвебклиент* и *корсвебсервице* :

![Опубликовать родительский каталог](./media/application-proxy-understand-cors-issues/image5.png)

Итоговые URL-адреса приложений эффективно устраняют проблему CORS:

- HTTPS: \/ /corswebclient-contoso.msappproxy.NET/CORSWebService
- HTTPS: \/ /corswebclient-contoso.msappproxy.NET/CORSWebClient

### <a name="option-3-update-http-headers"></a>Вариант 3. Обновление заголовков HTTP

Добавьте пользовательский заголовок ответа HTTP для веб-службы в соответствии с исходным запросом. Для веб-сайтов, работающих в службы IIS (IIS), используйте диспетчер IIS для изменения заголовка:

![Добавление пользовательского заголовка ответа в диспетчере служб IIS](./media/application-proxy-understand-cors-issues/image6.png)

Это изменение не требует внесения изменений в код. Его можно проверить в трассировках Fiddler:

**Публикация добавления заголовка**\
HTTP/1.1 200 OK \
Cache-Control: без кэша \
Pragma: No-cache \
Тип содержимого: text/plain; charset = UTF-8 \
Срок действия истекает:-1 \
Разное: Accept-Encoding \
Сервер: Microsoft-IIS/8.5 Microsoft-HTTPAPI/2.0 \
**Доступ — управление-разрешение-источник: HTTPS \: //corswebclient-contoso.msappproxy.NET**\
X-AspNet-Version: 4.0.30319 \
X-питание от: ASP.NET \
Длина содержимого: 17

### <a name="option-4-modify-the-app"></a>Вариант 4. изменение приложения

Вы можете изменить приложение для поддержки CORS, добавив заголовок Access-Control-Allow-Origin с соответствующими значениями. Способ добавления заголовка зависит от языка кода приложения. Изменение кода является наименьшим рекомендуемым параметром, так как он требует наибольшей усилий.

### <a name="option-5-extend-the-lifetime-of-the-access-token"></a>Вариант 5. продление времени существования маркера доступа

Некоторые проблемы CORS невозможно разрешить, например, когда приложение перенаправляется на *Login.microsoftonline.com* для проверки подлинности и срок действия маркера доступа истечет. Затем вызов CORS завершается ошибкой. Обходной путь для этого сценария — продлить время существования маркера доступа, чтобы предотвратить истечение срока его действия во время сеанса пользователя. Дополнительные сведения о том, как это сделать, см. [в разделе настраиваемое время существования маркеров в Azure AD](../develop/active-directory-configurable-token-lifetimes.md).

## <a name="see-also"></a>См. также раздел
- [Руководство по Добавление локального приложения для удаленного доступа через Application Proxy в Azure Active Directory](application-proxy-add-on-premises-application.md) 
- [Планирование развертывания Azure AD Application Proxy](application-proxy-deployment-plan.md) 
- [Удаленный доступ к локальным приложениям через Azure Active Directory Application Proxy](application-proxy.md)