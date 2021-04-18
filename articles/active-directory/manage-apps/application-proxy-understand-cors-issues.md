---
title: Общие сведения проблемах с механизмом CORS в Azure AD Application Proxy и их решении
description: Ознакомьтесь с общей информацией о механизме CORS в Azure AD Application Proxy и о том, как выявлять и устранять проблемы с ним.
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
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99259174"
---
# <a name="understand-and-solve-azure-active-directory-application-proxy-cors-issues"></a>Общие сведения проблемах с механизмом CORS в Azure Active Directory Application Proxy и их решении

Иногда при использовании механизма [общего доступа к ресурсам независимо от источника (CORS)](https://www.w3.org/TR/cors/) могут возникнуть проблемы с приложениями и API, которые публикуются с помощью Azure Active Directory Application Proxy. В этой статье описываются такие проблемы с CORS в Azure AD Application Proxy и пути и решения.

Параметры безопасности веб-браузера обычно предотвращают отправку запросов AJAX с веб-страницы к другому домену. Такое ограничение называется *политикой одного источника*. Эта политика предотвращает чтение вредоносным сайтом конфиденциальных данных с другого сайта. Однако иногда может потребоваться разрешить другим сайтам вызывать ваш веб-API. CORS — это стандарт консорциума W3C, позволяющий серверу смягчить ограничения политики одного источника и выполнять некоторые запросы независимо от источника, а другие — отклонять.

## <a name="understand-and-identify-cors-issues"></a>Общие сведения проблемах с механизмом CORS и их решении

Два URL-адреса имеют одинаковый источник, если у них идентичные схемы, узлы и порты ([RFC 6454](https://tools.ietf.org/html/rfc6454)). Например:

-   http:\//contoso.com/foo.html
-   http:\//contoso.com/bar.html

Указанные ниже URL-адреса имеют источники, отличные от предыдущих двух.

-   http:\//contoso.net — используется другой домен.
-   http:\//contoso.com:9000/foo.html — используется другой порт.
-   https:\//contoso.com/foo.html — используется другая схема.
-   https:\//contoso.com/foo.html — используется другой поддомен.

Политика одного источника не позволяет приложениям получать доступ к ресурсам из других источников, если в них не используются правильные заголовки для управления доступом. Если заголовки CORS отсутствуют или неверны, запросы к разным источникам завершаются сбоем. 

Вы можете выявить проблемы с CORS с помощью описанных ниже инструментов отладки в браузере.

1. Запустите браузер и перейдите к веб-приложению.
1. Нажмите клавишу **F12**, чтобы открыть консоль отладки.
1. Попробуйте возобновить транзакцию и просмотрите сообщение консоли. Нарушение CORS приводит к ошибке консоли в отношении источника.

На снимке экрана ниже показано, что при нажатии кнопки **Try It** появляется сообщение об ошибке CORS: источник https:\//corswebclient-contoso.msappproxy.net не найден в заголовке Access-Control-Allow-Origin.

![Проблема с CORS](./media/application-proxy-understand-cors-issues/image3.png)

## <a name="cors-challenges-with-application-proxy"></a>Проблемы с CORS и Application Proxy

В примере ниже показан типичный сценарий CORS для Azure AD Application Proxy. На внутреннем сервере размещен контроллер веб-API **CORSWebService**, а также **CORSWebClient**, который вызывает **CORSWebService**. Запрос AJAX поступает от **CORSWebClient** к **CORSWebService**.

![Локальный запрос к одному источнику](./media/application-proxy-understand-cors-issues/image1.png)

Приложение CORSWebClient работает, когда оно размещено локально. Но при публикации с помощью Azure AD Application Proxy его не удастся загрузить или оно будет завершать работу с ошибками. Если вы отдельно опубликовали CORSWebClient и CORSWebService через Application Proxy, то эти два приложения будут размещены в разных доменах. Запрос AJAX от CORSWebClient к CORSWebService будет являться запросом между разными источниками и завершится сбоем.

![Запрос CORS для Application Proxy](./media/application-proxy-understand-cors-issues/image2.png)

## <a name="solutions-for-application-proxy-cors-issues"></a>Решения проблем с CORS и Application Proxy

Описанную выше проблему с CORS можно устранить несколькими способами.

### <a name="option-1-set-up-a-custom-domain"></a>Способ 1: настройка личного домена

Используйте [личный домен](./application-proxy-configure-custom-domain.md) Azure AD Application Proxy для публикации из одного источника. Это не потребует внесения изменений в источники приложений, код или заголовки. 

### <a name="option-2-publish-the-parent-directory"></a>Способ 2: публикация родительского каталога

Опубликуйте родительский каталог обоих приложений. Это решение будет удобным, если на веб-сервере находятся только два приложения. Вместо публикации каждого приложения по отдельности можно опубликовать общий родительский каталог. В таком случае они будут иметь один и тот же источник.

В примерах ниже показана страница портала Azure AD Application Proxy для приложения CORSWebClient.  Если для параметра **Внутренний URL-адрес** задать значение *contoso.com/CORSWebClient*, то приложение не сможет выполнять успешные запросы к каталогу *contoso.com/CORSWebService*, так как они находятся в разных источниках. 

![Публикация приложений по отдельности](./media/application-proxy-understand-cors-issues/image4.png)

Вместо этого задайте **Внутренний URL-адрес** для публикации родительского каталога, который включает в себя каталоги *CORSWebClient* и *CORSWebService*.

![Публикация родительского каталога](./media/application-proxy-understand-cors-issues/image5.png)

Итоговые URL-адреса приложений позволят устранить проблему с CORS:

- https:\//corswebclient-contoso.msappproxy.net/CORSWebService
- https:\//corswebclient-contoso.msappproxy.net/CORSWebClient

### <a name="option-3-update-http-headers"></a>Способ 3: обновление заголовков HTTP

Добавьте заголовок ответа HTTP в веб-службе, чтобы обеспечить соответствие для запроса к источнику. На веб-сайтах, работающих на базе служб IIS, используйте для изменения заголовка диспетчер IIS.

![Добавление заголовка ответа в диспетчере IIS](./media/application-proxy-understand-cors-issues/image6.png)

Это изменение не требует внесения изменений в код. Проверку можно выполнить с помощью трассировок Fiddler.

**Публикация добавления заголовка**\
HTTP/1.1 200 OK\
Cache-Control: no-cache\
Pragma: no-cache\
Content-Type: text/plain; charset=utf-8\
Expires: -1\
Vary: Accept-Encoding\
Server: Microsoft-IIS/8.5 Microsoft-HTTPAPI/2.0\
**Access-Control-Allow-Origin: https\://corswebclient-contoso.msappproxy.net**\
X-AspNet-Version: 4.0.30319\
X-Powered-By: ASP.NET\
Content-Length: 17

### <a name="option-4-modify-the-app"></a>Способ 4: изменение приложения

Вы можете изменить приложение для поддержки CORS, добавив заголовок Access-Control-Allow-Origin с соответствующими значениями. Способ добавления заголовка зависит от языка кода, с помощью которого создано приложение. Изменение кода требует больших усилий, поэтому мы рекомендуем использовать этот способ в последнюю очередь.

### <a name="option-5-extend-the-lifetime-of-the-access-token"></a>Способ 5: продление времени существования маркера доступа

Некоторые проблемы с CORS невозможно решить. Например, если приложение перенаправляется на сайт *login.microsoftonline.com* для проверки подлинности и срок действия маркера доступа истекает. В таком случае вызов CORS завершается ошибкой. Обходной путь для этого сценария — продлить время существования маркера доступа, чтобы предотвратить истечение срока его действия во время сеанса пользователя. Дополнительные сведения о том, как это сделать, см. в статье о [настраиваемых сроках существования маркеров в Azure AD](../develop/active-directory-configurable-token-lifetimes.md).

## <a name="see-also"></a>Дополнительные материалы
- [Руководство по добавлению локального приложения для удаленного доступа через Application Proxy в Azure Active Directory](application-proxy-add-on-premises-application.md) 
- [Планирование развертывания Azure AD Application Proxy](application-proxy-deployment-plan.md) 
- [Удаленный доступ к локальным приложениям с помощью Azure Active Directory Application Proxy](application-proxy.md)