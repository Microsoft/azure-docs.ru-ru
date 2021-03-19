---
title: Ресурсы шаблонов управления API Azure | Документация Майкрософт
description: Сведения о типах ресурсов, доступных для использования в шаблонах портала разработчика в службе управления API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 51a1b4c6-a9fd-4524-9e0e-03a9800c3e94
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: c3b6123c63bf530463379a175745ef86baf2c5a3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84689993"
---
# <a name="azure-api-management-template-resources"></a>Ресурсы шаблонов управления API Azure
Служба управления API Azure предоставляет следующие типы ресурсов для использования в шаблонах портала разработчика:  
  
-   [строковые ресурсы](#strings);  
  
-   [ресурсы глифов](#glyphs).  

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]
  
##  <a name="string-resources"></a><a name="strings"></a> Строковые ресурсы  
 Служба управления API предусматривает множество строковых ресурсов, которые можно использовать на портале разработчика. Эти ресурсы переведены на все языки, поддерживаемые службой управления API. В наборе шаблонов по умолчанию эти ресурсы используются для заголовков страниц, меток и любых константных строк, отображаемых на портале разработчика. Чтобы использовать строковый ресурс в шаблонах, укажите префикс строкового ресурса, а затем имя строки, как показано в следующем примере.  
  
```  
{% localized "Prefix|Name" %}  
  
```  
  
 Следующий пример взят из шаблона списка продуктов. Он отображает элемент **Продукты** в верхней части страницы.  
  
```  
<h2>{% localized "ProductsStrings|PageTitleProducts" %}</h2>  
  
```  
  
Поддерживаются следующие варианты локализации.

| Локаль    | Язык               |
|-----------|------------------------|
| en      | English              |
| сложных      | Čeština              |
| ru      | Deutsch              |
| требуем      | Espaсol              |
| французский      | Franзais             |
| HU      | Magyar               |
| им      | Italiano             |
| ja-JP   | 日本語                |
| ko      | 한국어                |
| NL      | Nederlands           |
| Папка      | Polski               |
| "pt-br"   | "Portuguкs (Brasil)"   |
| "PT-PT"   | "Portuguкs (Португалия)" |
| ru      | Русский              |
| календар      | Svenska              |
| ТС      | Tьrkзe               |
| "zh-Ханс" | "中文 (简体)"           |
| "zh-Hant" | "中文 (繁體)"           |

 Строковые ресурсы, доступные для использования в шаблонах портала разработчика, см. в таблицах ниже. Используйте имя таблицы в качестве префикса для строковых ресурсов в этой таблице.  
  
-   [ApisStrings](#ApisStrings)  
  
-   [ApplicationListStrings](#ApplicationListStrings)  
  
-   [AppDetailsStrings](#AppDetailsStrings)  
  
-   [AppStrings](#AppStrings)  
  
-   [CommonResources](#CommonResources)  
  
-   [CommonStrings](#CommonStrings)  
  
-   [Документация](#Documentation)  
  
-   [ErrorPageStrings](#ErrorPageStrings)  
  
-   [IssuesStrings](#IssuesStrings)  
  
-   [NotFoundStrings](#NotFoundStrings)  
  
-   [ProductDetailsStrings](#ProductDetailsStrings)  
  
-   [ProductsStrings](#ProductsStrings)  
  
-   [ProviderInfoStrings](#ProviderInfoStrings)  
  
-   [SigninResources](#SigninResources)  
  
-   [SigninStrings](#SigninStrings)  
  
-   [SignupStrings](#SignupStrings)  
  
-   [SubscriptionListStrings](#SubscriptionListStrings)  
  
-   [SubscriptionStrings](#SubscriptionStrings)  
  
-   [UpdateProfileStrings](#UpdateProfileStrings)  
  
-   [UserProfile](#UserProfile)  
  
###  <a name="apisstrings"></a><a name="ApisStrings"></a> аписстрингс  
  
|Имя|Текст|  
|----------|----------|  
|PageTitleApis|API-интерфейсы|  
  
###  <a name="appdetailsstrings"></a><a name="AppDetailsStrings"></a> аппдетаилсстрингс  
  
|Имя|Текст|  
|----------|----------|  
|WebApplicationsDetailsTitle|Предварительная версия приложения|  
|WebApplicationsRequirementsHeader|Требования|  
|WebApplicationsScreenshotAlt|Снимок экрана|  
|WebApplicationsScreenshotsHeader|Снимки экрана|  
  
###  <a name="applicationliststrings"></a><a name="ApplicationListStrings"></a> аппликатионлистстрингс  
  
|Имя|Текст|  
|----------|----------|  
|WebDevelopersAppDeleteConfirmation|Вы действительно хотите удалить приложение?|  
|WebDevelopersAppNotPublished|Не опубликовано|  
|WebDevelopersAppNotSubmitted|Не отправлено|  
|WebDevelopersAppTableCategoryHeader|Category|  
|WebDevelopersAppTableNameHeader|Имя|  
|WebDevelopersAppTableStateHeader|Состояние|  
|WebDevelopersEditLink|Изменить|  
|WebDevelopersRegisterAppLink|Регистрация приложения|  
|WebDevelopersRemoveLink|Удалить|  
|WebDevelopersSubmitLink|Отправить|  
|WebDevelopersYourApplicationsHeader|Ваши приложения|  
  
###  <a name="appstrings"></a><a name="AppStrings"></a> аппстрингс  
  
|Имя|Текст|  
|----------|----------|  
|WebApplicationsHeader|Приложения|  
  
###  <a name="commonresources"></a><a name="CommonResources"></a> коммонресаурцес  
  
|Имя|Текст|  
|----------|----------|  
|NoItemsToDisplay|Результаты отсутствуют.|  
|GeneralExceptionMessage|Возникла проблема. Возможно, это временный сбой или ошибка. Повторите попытку позже.|  
|GeneralJsonExceptionMessage|Возникла проблема. Возможно, это временный сбой или ошибка. Обновите страницу и повторите попытку.|  
|ConfirmationMessageUnsavedChanges|Есть несохраненные изменения. Вы действительно хотите отменить и отклонить изменения?|  
|AzureActiveDirectory|Azure Active Directory|  
|HttpLargeRequestMessage|Текст HTTP-запроса слишком длинный.|  
  
###  <a name="commonstrings"></a><a name="CommonStrings"></a> коммонстрингс  
  
|Имя|Текст|  
|----------|----------|  
|ButtonLabelCancel|Отменить|  
|ButtonLabelSave|Сохранить|  
|GeneralExceptionMessage|Возникла проблема. Возможно, это временный сбой или ошибка. Повторите попытку позже.|  
|NoItemsToDisplay|Нет элементов для отображения|  
|PagerButtonLabelFirst|First|  
|PagerButtonLabelLast|Последний|  
|PagerButtonLabelNext|Следующая|  
|PagerButtonLabelPrevious|Предыдущий|  
|PagerLabelPageNOfM|Страница {0} из {1}|  
|PasswordTooShort|Пароль слишком короткий|  
|EmailAsPassword|Не используйте адрес электронной почты в качестве пароля|  
|PasswordSameAsUserName|Пароль не может содержать имя пользователя|  
|PasswordTwoCharacterClasses|Используйте разные классы символов|  
|PasswordTooManyRepetitions|Слишком много повторений|  
|PasswordSequenceFound|Пароль содержит последовательности|  
|PagerLabelPageSize|Размер страницы|  
|CurtainLabelLoading|Загрузка...|  
|TablePlaceholderNothingToDisplay|Нет данных для выбранного периода и области|  
|ButtonLabelClose|Закрыть|  
  
###  <a name="documentation"></a><a name="Documentation"></a> По  
  
|Имя|Текст|  
|----------|----------|  
|WebDocumentationInvalidHeaderErrorMessage|Недопустимый заголовок {0}|  
|WebDocumentationInvalidRequestErrorMessage|Недопустимый URL-адрес запроса|  
|TextboxLabelAccessToken|Маркер доступа*|  
|DropdownOptionPrimaryKeyFormat|Основной — {0}|  
|DropdownOptionSecondaryKeyFormat|Дополнительный — {0}|  
|WebDocumentationSubscriptionKeyText|Ваш ключ подписки|  
|WebDocumentationTemplatesAddHeaders|Добавьте необходимые заголовки HTTP|  
|WebDocumentationTemplatesBasicAuthSample|Пример базовой авторизации|  
|WebDocumentationTemplatesCurlForBasicAuth|Для использования базовой авторизации: --user {имя_пользователя}:{пароль}|  
|WebDocumentationTemplatesCurlValuesForPath|Укажите значения для параметров пути (отображаются как {...}), ключ подписки и значения для параметров запроса|  
|WebDocumentationTemplatesDeveloperKey|Укажите свой ключ подписки|  
|WebDocumentationTemplatesJavaApache|В этом примере используется HTTP-клиент Apache от HTTP Components (http://hc.apache.org/httpcomponents-client-ga/)|  
|WebDocumentationTemplatesOptionalParams|Укажите значения для необязательных параметров при необходимости|  
|WebDocumentationTemplatesPhpPackage|В этом примере используется пакет HTTP_Request2 (см. дополнительные сведения: https://pear.php.net/package/HTTP_Request2)|  
|WebDocumentationTemplatesPythonValuesForPath|Укажите значения для параметров пути (отображаются как {...}) и текст запроса при необходимости|  
|WebDocumentationTemplatesRequestBody|Укажите текст запроса|  
|WebDocumentationTemplatesRequiredParams|Укажите значения для следующих обязательных параметров|  
|WebDocumentationTemplatesValuesForPath|Укажите значения для параметров пути (отображаются как {...})|  
|OAuth2AuthorizationEndpointDescription|Конечная точка авторизации используется, чтобы взаимодействовать с владельцем ресурса и предоставлять авторизацию|  
|OAuth2AuthorizationEndpointName|Конечная точка авторизации|  
|OAuth2TokenEndpointDescription|Конечная точка маркера используется клиентом, чтобы получить маркер доступа с помощью предоставления авторизации или маркер обновления|  
|OAuth2TokenEndpointName|Конечная точка маркера|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_Description|<p\> Клиент запускает поток, направляя агент пользователя владельца ресурса в конечную точку авторизации.  Клиент добавляет свой идентификатор клиента, запрошенную область, локальное состояние и универсальный код ресурса (URI) перенаправления, на который сервер авторизации будет направлять агент пользователя после предоставления доступа (или отказа в доступе).     </p\>     <p\> Сервер авторизации выполняет проверку подлинности владельца ресурса (с помощью агента пользователя) и определяет, выполняет ли владелец ресурса запрос на доступ клиента или отклоняет его.     </p\>     <p\> При условии, что владелец ресурса предоставляет доступ, сервер авторизации перенаправляет агент пользователя в клиент с помощью URI перенаправления, предоставленного ранее (в запросе или во время регистрации клиента).  URI перенаправления включает код авторизации и любое локальное состояние, указанное клиентом ранее.     </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_ErrorDescription|<p\> Если пользователь отклоняет запрос на доступ или если запрос является недопустимым, клиент получит оповещение с помощью указанных ниже параметров, добавленных в перенаправление. </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_Name|Запрос авторизации|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_RequestDescription|<p\> Клиентское приложение должно отправлять пользователя в конечную точку для запуска процесса OAuth.          В конечной точке авторизации пользователь выполняет проверку подлинности, а затем предоставляет или запрещает доступ к приложению.     </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_ResponseDescription|</p\> При условии, что владелец ресурса предоставляет доступ, сервер авторизации перенаправляет агент пользователя в клиент с помощью URI перенаправления, предоставленного ранее (в запросе или во время регистрации клиента).  URI перенаправления включает код авторизации и любое локальное состояние, указанное клиентом ранее. </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_Description|<p\> Клиент запрашивает маркер доступа из конечной точки маркера сервера авторизации путем включения кода авторизации, полученного на предыдущем шаге.  При выполнении запроса клиент выполняет проверку подлинности с помощью сервера авторизации.  Клиент включает URI перенаправления, используемый для получения кода авторизации для проверки. </p\> <p\> Сервер авторизации выполняет проверку подлинности клиента, проверяет код авторизации и обеспечивает соответствие полученного URI перенаправления URI, используемому для перенаправления клиента на шаге (C).  Если это возможно, сервер авторизации отправляет маркер доступа и при необходимости маркер обновления. </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_ErrorDescription|<p\> Если проверка подлинности клиента запроса завершилась сбоем или является недопустимой, сервер авторизации возвращает код состояния HTTP 400 (недопустимый запрос), если не указано иное, и включает следующие параметры в ответе. </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_RequestDescription|<p\> Клиент выполняет запрос в конечную точку маркера, отправляя следующие параметры в формате application/x-www-form-urlencoded с кодировкой символов UTF-8 в тексте сущности HTTP-запроса. </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_ResponseDescription|<p\> Сервер авторизации выдает маркер доступа и необязательный маркер обновления, а также создает ответ, добавив следующие параметры в текст сущности HTTP-ответа с кодом состояния 200 (ОК). </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_Description|<p\> Клиент выполняет проверку подлинности с помощью сервера авторизации и запрашивает маркер доступа из конечной точки маркера. </p\> <p\> Сервер авторизации выполняет проверку подлинности клиента и, если это допустимо, выдает маркер доступа. </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_ErrorDescription|<p\> Если проверка подлинности клиента запроса завершилась сбоем или является недопустимой, сервер авторизации возвращает код состояния HTTP 400 (недопустимый запрос), если не указано иное, и включает следующие параметры в ответе. </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_RequestDescription|<p\> Клиент выполняет запрос в конечную точку маркера, добавляя следующие параметры в формате application/x-www-form-urlencoded с кодировкой символов UTF-8 в тексте сущности HTTP-запроса. </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_ResponseDescription|<p\> Если запрос на маркер доступа является допустимым или проходит авторизацию, сервер авторизации выдает маркер доступа и необязательный маркер обновления, а также создает ответ, добавив следующие параметры в текст сущности HTTP-ответа с кодом состояния 200 (ОК). </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_Description|<p\> Клиент запускает поток, направляя агент пользователя владельца ресурса в конечную точку авторизации.  Клиент добавляет свой идентификатор клиента, запрошенную область, локальное состояние и URI перенаправления, на который сервер авторизации будет направлять агент пользователя после предоставления доступа (или отказа в доступе). </p\>     <p\> Сервер авторизации выполняет проверку подлинности владельца ресурса (с помощью агента пользователя) и определяет, выполняет ли владелец ресурса запрос на доступ клиента или отклоняет его. </p\>     <p\> При условии, что владелец ресурса предоставляет доступ, сервер авторизации перенаправляет агент пользователя в клиент с помощью URI перенаправления, предоставленного ранее.  URI перенаправления содержит маркер доступа во фрагменте URI. </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_ErrorDescription|<p\> Если владелец ресурса отклоняет запрос на доступ или возникает сбой запроса по причинам, не связанным с отсутствующим или недопустимым URI перенаправления, сервер авторизации сообщает об этом клиенту, добавив следующие параметры в компонент фрагмента URI перенаправления в формате application/x-www-form-urlencoded. </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_RequestDescription|<p\> Клиентское приложение должно отправлять пользователя к конечной точке для запуска процесса OAuth.      В конечной точке авторизации пользователь выполняет проверку подлинности, а затем предоставляет или запрещает доступ к приложению. </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_ResponseDescription|<p\> Если владелец ресурса выполняет запрос на доступ, сервер авторизации выдает маркер доступа и доставляет его клиенту путем добавления следующих параметров компонента фрагмента URI перенаправления в формате application/x-www-form-urlencoded. </p\>|  
|OAuth2Flow_ObtainAuthorization_AuthorizationCodeGrant_Description|Поток кода авторизации оптимизирован для клиентов, способных сохранять конфиденциальность своих учетных данных (например, приложения веб-сервера, реализованные с помощью PHP, Java, Python, Ruby, ASP.NET и т. д.).|  
|OAuth2Flow_ObtainAuthorization_AuthorizationCodeGrant_Name|Предоставление кода авторизации|  
|OAuth2Flow_ObtainAuthorization_ClientCredentialsGrant_Description|Поток учетных данных клиента подходит в случаях, когда клиент (приложение) запрашивает доступ к защищенным ресурсам под его контролем. Клиент считается владельцем ресурса, поэтому взаимодействие с конечным пользователем не требуется.|  
|OAuth2Flow_ObtainAuthorization_ClientCredentialsGrant_Name|Предоставление учетных данных клиента|  
|OAuth2Flow_ObtainAuthorization_ImplicitGrant_Description|Неявный поток оптимизирован для клиентов, неспособных сохранять конфиденциальность своих учетных данных и способных работать с определенным URI перенаправления. Эти клиенты, как правило, реализуются в браузере с помощью языка скриптов, например JavaScript.|  
|OAuth2Flow_ObtainAuthorization_ImplicitGrant_Name|Неявное разрешение|  
|OAuth2Flow_ObtainAuthorization_ResourceOwnerPasswordCredentialsGrant_Description|Поток учетных данных владельца ресурса подходит в случаях, когда у владельца ресурса доверительные отношения с клиентом (вашим приложением), например операционной системой устройства или приложением с высоким уровнем привилегий. Этот поток подходит для клиентов, способных получить учетные данные владельца ресурса (имя пользователя и пароль, как правило, с помощью интерактивной формы).|  
|OAuth2Flow_ObtainAuthorization_ResourceOwnerPasswordCredentialsGrant_Name|Предоставление учетных данных владельца ресурса|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_Description|<p\> Владелец ресурса предоставляет клиенту имя пользователя и пароль. </p\> <p\> Клиент запрашивает маркер доступа из конечной точки маркера сервера авторизации путем включения учетных данных, полученных от владельца ресурса.  При выполнении запроса клиент выполняет проверку подлинности с помощью сервера авторизации. </p\> <p\> Сервер авторизации выполняет проверку подлинности клиента и проверяет учетные данные владельца ресурса, а также, если это допустимо, выдает маркер доступа. </p\>|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_ErrorDescription|<p\> Если проверка подлинности клиента запроса завершилась сбоем или является недопустимой, сервер авторизации возвращает код состояния HTTP 400 (недопустимый запрос), если не указано иное, и включает следующие параметры в ответе. </p\>|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_RequestDescription|<p\> Клиент выполняет запрос в конечную точку маркера, добавляя следующие параметры в формате application/x-www-form-urlencoded с кодировкой символов UTF-8 в тексте сущности HTTP-запроса. </p\>|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_ResponseDescription|<p\> Если запрос на маркер доступа является допустимым или проходит авторизацию, сервер авторизации выдает маркер доступа и необязательный маркер обновления, а также создает ответ, добавив следующие параметры в текст сущности HTTP-ответа с кодом состояния 200 (ОК). </p\>|  
|OAuth2Step_AccessTokenRequest_Name|Запрос маркера доступа|  
|OAuth2Step_AuthorizationRequest_Name|Запрос авторизации|  
|OAuth2AccessToken_AuthorizationCodeGrant_TokenResponse|Обязательный параметр. Маркер доступа, выданный сервером авторизации.|  
|OAuth2AccessToken_ClientCredentialsGrant_TokenResponse|Обязательный параметр. Маркер доступа, выданный сервером авторизации.|  
|OAuth2AccessToken_ImplicitGrant_AuthorizationResponse|Обязательный параметр. Маркер доступа, выданный сервером авторизации.|  
|OAuth2AccessToken_ResourceOwnerPasswordCredentialsGrant_TokenResponse|Обязательный параметр. Маркер доступа, выданный сервером авторизации.|  
|OAuth2ClientId_AuthorizationCodeGrant_AuthorizationRequest|Обязательный параметр. Идентификатор клиента.|  
|OAuth2ClientId_AuthorizationCodeGrant_TokenRequest|Требуется, если клиент не проходит проверку подлинности с помощью сервера авторизации.|  
|OAuth2ClientId_ImplicitGrant_AuthorizationRequest|Обязательный параметр. Идентификатор клиента.|  
|OAuth2Code_AuthorizationCodeGrant_AuthorizationResponse|Обязательный параметр. Код авторизации, созданный сервером авторизации.|  
|OAuth2Code_AuthorizationCodeGrant_TokenRequest|Обязательный параметр. Код авторизации, полученный из сервера авторизации.|  
|OAuth2ErrorDescription_AuthorizationCodeGrant_AuthorizationErrorResponse|OPTIONAL. Понятный для пользователя текст ASCII, предоставляющий дополнительные сведения.|  
|OAuth2ErrorDescription_AuthorizationCodeGrant_TokenErrorResponse|OPTIONAL. Понятный для пользователя текст ASCII, предоставляющий дополнительные сведения.|  
|OAuth2ErrorDescription_ClientCredentialsGrant_TokenErrorResponse|OPTIONAL. Понятный для пользователя текст ASCII, предоставляющий дополнительные сведения.|  
|OAuth2ErrorDescription_ImplicitGrant_AuthorizationErrorResponse|OPTIONAL. Понятный для пользователя текст ASCII, предоставляющий дополнительные сведения.|  
|OAuth2ErrorDescription_ResourceOwnerPasswordCredentialsGrant_TokenErrorResponse|OPTIONAL. Понятный для пользователя текст ASCII, предоставляющий дополнительные сведения.|  
|OAuth2ErrorUri_AuthorizationCodeGrant_AuthorizationErrorResponse|OPTIONAL. URI, идентифицирующий понятную для пользователя веб-страницу с помощью сведений об ошибке.|  
|OAuth2ErrorUri_AuthorizationCodeGrant_TokenErrorResponse|OPTIONAL. URI, идентифицирующий понятную для пользователя веб-страницу с помощью сведений об ошибке.|  
|OAuth2ErrorUri_ClientCredentialsGrant_TokenErrorResponse|OPTIONAL. URI, идентифицирующий понятную для пользователя веб-страницу с помощью сведений об ошибке.|  
|OAuth2ErrorUri_ImplicitGrant_AuthorizationErrorResponse|OPTIONAL. URI, идентифицирующий понятную для пользователя веб-страницу с помощью сведений об ошибке.|  
|OAuth2ErrorUri_ResourceOwnerPasswordCredentialsGrant_TokenErrorResponse|OPTIONAL. URI, идентифицирующий понятную для пользователя веб-страницу с помощью сведений об ошибке.|  
|OAuth2Error_AuthorizationCodeGrant_AuthorizationErrorResponse|Обязательный параметр. Один код ошибки ASCII из следующих параметров: invalid_request, unauthorized_client, access_denied, unsupported_response_type, invalid_scope, server_error, temporarily_unavailable.|  
|OAuth2Error_AuthorizationCodeGrant_TokenErrorResponse|Обязательный параметр. Один код ошибки ASCII из следующих параметров: invalid_request, invalid_client, invalid_grant, unauthorized_client, unsupported_grant_type, invalid_scope.|  
|OAuth2Error_ClientCredentialsGrant_TokenErrorResponse|Обязательный параметр. Один код ошибки ASCII из следующих параметров: invalid_request, invalid_client, invalid_grant, unauthorized_client, unsupported_grant_type, invalid_scope.|  
|OAuth2Error_ImplicitGrant_AuthorizationErrorResponse|Обязательный параметр. Один код ошибки ASCII из следующих параметров: invalid_request, unauthorized_client, access_denied, unsupported_response_type, invalid_scope, server_error, temporarily_unavailable.|  
|OAuth2Error_ResourceOwnerPasswordCredentialsGrant_TokenErrorResponse|Обязательный параметр. Один код ошибки ASCII из следующих параметров: invalid_request, invalid_client, invalid_grant, unauthorized_client, unsupported_grant_type, invalid_scope.|  
|OAuth2ExpiresIn_AuthorizationCodeGrant_TokenResponse|Рекомендуемый параметр. Время существования маркера доступа в секундах.|  
|OAuth2ExpiresIn_ClientCredentialsGrant_TokenResponse|Рекомендуемый параметр. Время существования маркера доступа в секундах.|  
|OAuth2ExpiresIn_ImplicitGrant_AuthorizationResponse|Рекомендуемый параметр. Время существования маркера доступа в секундах.|  
|OAuth2ExpiresIn_ResourceOwnerPasswordCredentialsGrant_TokenResponse|Рекомендуемый параметр. Время существования маркера доступа в секундах.|  
|OAuth2GrantType_AuthorizationCodeGrant_TokenRequest|Обязательный параметр. Для него необходимо задать значение authorization_code.|  
|OAuth2GrantType_ClientCredentialsGrant_TokenRequest|Обязательный параметр. Для него необходимо задать значение client_credentials.|  
|OAuth2GrantType_ResourceOwnerPasswordCredentialsGrant_TokenRequest|Обязательный параметр. Для него необходимо задать значение password.|  
|OAuth2Password_ResourceOwnerPasswordCredentialsGrant_TokenRequest|Обязательный параметр. Пароль владельца ресурса.|  
|OAuth2RedirectUri_AuthorizationCodeGrant_AuthorizationRequest|OPTIONAL. URI перенаправления конечной точки должен быть абсолютным URI.|  
|OAuth2RedirectUri_AuthorizationCodeGrant_TokenRequest|Обязательный параметр, если параметр redirect_uri включен в запрос на авторизацию, и их значения должны быть одинаковыми.|  
|OAuth2RedirectUri_ImplicitGrant_AuthorizationRequest|OPTIONAL. URI перенаправления конечной точки должен быть абсолютным URI.|  
|OAuth2RefreshToken_AuthorizationCodeGrant_TokenResponse|OPTIONAL. Маркер обновления, который можно использовать для получения нового маркера доступа.|  
|OAuth2RefreshToken_ClientCredentialsGrant_TokenResponse|OPTIONAL. Маркер обновления, который можно использовать для получения нового маркера доступа.|  
|OAuth2RefreshToken_ResourceOwnerPasswordCredentialsGrant_TokenResponse|OPTIONAL. Маркер обновления, который можно использовать для получения нового маркера доступа.|  
|OAuth2ResponseType_AuthorizationCodeGrant_AuthorizationRequest|Обязательный параметр. Для него необходимо задать значение code.|  
|OAuth2ResponseType_ImplicitGrant_AuthorizationRequest|Обязательный параметр. Для него необходимо задать значение token.|  
|OAuth2Scope_AuthorizationCodeGrant_AuthorizationRequest|OPTIONAL. Область запроса доступа.|  
|OAuth2Scope_AuthorizationCodeGrant_TokenResponse|Необязательный параметр, если он идентичен области, запрошенной клиентом, в противном случае — обязательный.|  
|OAuth2Scope_ClientCredentialsGrant_TokenRequest|OPTIONAL. Область запроса доступа.|  
|OAuth2Scope_ClientCredentialsGrant_TokenResponse|Необязательный параметр, если он идентичен области, запрошенной клиентом, в противном случае — обязательный.|  
|OAuth2Scope_ImplicitGrant_AuthorizationRequest|OPTIONAL. Область запроса доступа.|  
|OAuth2Scope_ImplicitGrant_AuthorizationResponse|Необязательный параметр, если он идентичен области, запрошенной клиентом, в противном случае — обязательный.|  
|OAuth2Scope_ResourceOwnerPasswordCredentialsGrant_TokenRequest|OPTIONAL. Область запроса доступа.|  
|OAuth2Scope_ResourceOwnerPasswordCredentialsGrant_TokenResponse|Необязательный параметр, если он идентичен области, запрошенной клиентом, в противном случае — обязательный.|  
|OAuth2State_AuthorizationCodeGrant_AuthorizationErrorResponse|Обязательный параметр, если параметр state присутствовал в запросе на авторизацию клиента.  Точное значение предоставляет клиент.|  
|OAuth2State_AuthorizationCodeGrant_AuthorizationRequest|Рекомендуемый параметр. Непрозрачное значение, используемое клиентом для поддержания состояния между запросом и обратным вызовом.  Сервер авторизации включает это значение при перенаправлении агента пользователя в клиент.  Параметр следует использовать для предотвращения подделки межсайтовых запросов.|  
|OAuth2State_AuthorizationCodeGrant_AuthorizationResponse|Обязательный параметр, если параметр state присутствовал в запросе на авторизацию клиента.  Точное значение предоставляет клиент.|  
|OAuth2State_ImplicitGrant_AuthorizationErrorResponse|Обязательный параметр, если параметр state присутствовал в запросе на авторизацию клиента.  Точное значение предоставляет клиент.|  
|OAuth2State_ImplicitGrant_AuthorizationRequest|Рекомендуемый параметр. Непрозрачное значение, используемое клиентом для поддержания состояния между запросом и обратным вызовом.  Сервер авторизации включает это значение при перенаправлении агента пользователя в клиент.  Параметр следует использовать для предотвращения подделки межсайтовых запросов.|  
|OAuth2State_ImplicitGrant_AuthorizationResponse|Обязательный параметр, если параметр state присутствовал в запросе на авторизацию клиента.  Точное значение предоставляет клиент.|  
|OAuth2TokenType_AuthorizationCodeGrant_TokenResponse|Обязательный параметр. Тип выдаваемого маркера.|  
|OAuth2TokenType_ClientCredentialsGrant_TokenResponse|Обязательный параметр. Тип выдаваемого маркера.|  
|OAuth2TokenType_ImplicitGrant_AuthorizationResponse|Обязательный параметр. Тип выдаваемого маркера.|  
|OAuth2TokenType_ResourceOwnerPasswordCredentialsGrant_TokenResponse|Обязательный параметр. Тип выдаваемого маркера.|  
|OAuth2UserName_ResourceOwnerPasswordCredentialsGrant_TokenRequest|Обязательный параметр. Имя пользователя владельца ресурса.|  
|OAuth2UnsupportedTokenType|Тип маркера {0} не поддерживается.|  
|OAuth2InvalidState|Недопустимый ответ из сервера авторизации|  
|OAuth2GrantType_AuthorizationCode|Код авторизации|  
|OAuth2GrantType_Implicit|Неявная|  
|OAuth2GrantType_ClientCredentials|Учетные данные клиента|  
|OAuth2GrantType_ResourceOwnerPassword|Пароль владельца ресурса|  
|WebDocumentation302Code|302 — объект найден|  
|WebDocumentation400Code|400 (недопустимый запрос)|  
|OAuth2SendingMethod_AuthHeader|Заголовок авторизации|  
|OAuth2SendingMethod_QueryParam|Параметр запроса|  
|OAuth2AuthorizationServerGeneralException|Произошла ошибка при авторизации доступа с использованием {0}.|  
|OAuth2AuthorizationServerCommunicationException|Не удалось установить подключение HTTP к серверу авторизации или оно неожиданно закрыто.|  
|WebDocumentationOAuth2GeneralErrorMessage|Произошла непредвиденная ошибка.|  
|AuthorizationServerCommunicationException|Произошло исключение взаимодействия с сервером авторизации. Обратитесь к администратору.|  
|TextblockSubscriptionKeyHeaderDescription|Ключ подписки, предоставляющий доступ к API. Находится в <a href='/developer'\>Profile</a\>.|  
|TextblockOAuthHeaderDescription|Маркер доступа OAuth 2.0, полученный из <i\>{0}</i\>. Поддерживаемые типы предоставления: <i\>{1}</i\>.|  
|TextblockContentTypeHeaderDescription|Тип мультимедиа текста, отправляемого в API.|  
|ErrorMessageApiNotAccessible|API, который вы пытаетесь вызвать, сейчас недоступен. Обратитесь к издателю API <a href="/issues"\>здесь</a\>.|  
|ErrorMessageApiTimedout|Получение ответа от API, который вы пытаетесь вызвать, занимает больше времени, чем обычно. Обратитесь к издателю API <a href="/issues"\>здесь</a\>.|  
|BadRequestParameterExpected|"Ожидается параметр {0}".|  
|TooltipTextDoubleClickToSelectAll|Дважды щелкните, чтобы выделить все.|  
|TooltipTextHideRevealSecret|Показать или скрыть|  
|ButtonLinkOpenConsole|Попробовать|  
|SectionHeadingRequestBody|Текст запроса|  
|SectionHeadingRequestParameters|Параметры запроса|  
|SectionHeadingRequestUrl|URL-адрес запроса|  
|SectionHeadingResponse|Ответ|  
|SectionHeadingRequestHeaders|Заголовки запросов|  
|FormLabelSubtextOptional|необязательно|  
|SectionHeadingCodeSamples|Примеры кода|  
|TextblockOpenidConnectHeaderDescription|Маркер OpenID Connect Connect ID получен от <i \> {0}</i \> . Поддерживаемые типы предоставления: <i\>{1}</i\>.|  
  
###  <a name="errorpagestrings"></a><a name="ErrorPageStrings"></a> еррорпажестрингс  
  
|Имя|Текст|  
|----------|----------|  
|LinkLabelBack|Назад|  
|LinkLabelHomePage|домашняя страница|  
|LinkLabelSendUsEmail|Отправьте нам сообщение электронной почты.|  
|PageTitleError|К сожалению, возникла проблема обслуживания запрошенной страницы.|  
|TextblockPotentialCauseIntermittentIssue|Это может быть проблема с доступом ко временным данным, которые уже удалены.|  
|TextblockPotentialCauseOldLink|Возможно, ссылка, которую вы щелкнули, старая или больше не указывает на правильное расположение.|  
|TextblockPotentialCauseTechnicalProblem|Возможно, возникли технические проблемы на нашей стороне.|  
|TextblockPotentialSolutionRefresh|Попробуйте обновить страницу.|  
|TextblockPotentialSolutionStartOver|Начните отсюда: {0}.|  
|TextblockPotentialSolutionTryAgain|Перейдите к свойству {0} и повторите выполненное действие.|  
|TextReportProblem|{0} описывает проблему, которую мы рассмотрим в кратчайшие сроки.|  
|TitlePotentialCause|Возможная причина|  
|TitlePotentialSolution|Возможно, это временная проблема. Ниже приведено несколько способов ее исправить.|  
  
###  <a name="issuesstrings"></a><a name="IssuesStrings"></a> иссуесстрингс  
  
|Имя|Текст|  
|----------|----------|  
|WebIssuesIndexTitle|Проблемы|  
|WebIssuesNoActiveSubscriptions|У вас нет активных подписок. Чтобы сообщить о проблеме, необходимо подписаться на продукт.|  
|WebIssuesNotSignin|Вы не выполнили вход. Выполните {0}, чтобы сообщить о проблеме или опубликовать комментарий.|  
|WebIssuesReportIssueButton|Сообщить о проблеме|  
|WebIssuesSignIn|войти|  
|WebIssuesStatusReportedBy|Состояние: {0} &#124; сообщение от {1}.|  
  
###  <a name="notfoundstrings"></a><a name="NotFoundStrings"></a> нотфаундстрингс  
  
|Имя|Текст|  
|----------|----------|  
|LinkLabelHomePage|домашняя страница|  
|LinkLabelSendUsEmail|Отправьте нам сообщение электронной почты.|  
|PageTitleNotFound|К сожалению, запрашиваемая страница не найдена.|  
|TextblockPotentialCauseMisspelledUrl|Возможно, вы допустили опечатку при вводе URL-адреса.|  
|TextblockPotentialCauseOldLink|Возможно, ссылка, которую вы щелкнули, старая или больше не указывает на правильное расположение.|  
|TextblockPotentialSolutionRetype|Попробуйте повторно ввести URL-адрес.|  
|TextblockPotentialSolutionStartOver|Начните отсюда: {0}.|  
|TextReportProblem|{0} описывает проблему, которую мы рассмотрим в кратчайшие сроки.|  
|TitlePotentialCause|Возможная причина|  
|TitlePotentialSolution|Возможное решение|  
  
###  <a name="productdetailsstrings"></a><a name="ProductDetailsStrings"></a> продуктдетаилсстрингс  
  
|Имя|Текст|  
|----------|----------|  
|WebProductsAgreement|Подписавшись на продукт `<a data-toggle='modal' href='#legal-terms'\>Terms of Use</a\>`, я принимаю {0}.|  
|WebProductsLegalTermsLink|Условия использования|  
|WebProductsSubscribeButton|Подписка.|  
|WebProductsUsageLimitsHeader|Ограничения использования|  
|WebProductsYouAreNotSubscribed|Вы подписаны на продукт.|  
|WebProductsYouRequestedSubscription|Вы запросили подписку на этот продукт.|  
|ErrorYouNeedToAgreeWithLegalTerms|Прежде чем продолжить, необходимо принять условия использования.|  
|ButtonLabelAddSubscription|Добавление подписки|  
|LinkLabelChangeSubscriptionName|Изменить|  
|ButtonLabelConfirm|Подтвердить|  
|TextblockMultipleSubscriptionsCount|У вас есть такое количество подписок на продукт: {0}|  
|TextblockSingleSubscriptionsCount|У вас есть такое количество подписок на продукт: {0}|  
|TextblockSingleApisCount|Этот продукт содержит такое количество API: {0}.|  
|TextblockMultipleApisCount|Этот продукт содержит такое количество API: {0}.|  
|TextblockHeaderSubscribe|Подписаться на продукт|  
|TextblockSubscriptionDescription|Новая подписка будет создана следующим образом.|  
|TextblockSubscriptionLimitReached|Достигнуто максимальное ограничение количества подписок.|  
  
###  <a name="productsstrings"></a><a name="ProductsStrings"></a> продуктсстрингс  
  
|Имя|Текст|  
|----------|----------|  
|PageTitleProducts|Продукты|  
  
###  <a name="providerinfostrings"></a><a name="ProviderInfoStrings"></a> провидеринфострингс  
  
|Имя|Текст|  
|----------|----------|  
|TextboxExternalIdentitiesDisabled|В данный момент вход отключен администратором.|  
|TextboxExternalIdentitiesSigninInvitation|Вход можно выполнить с помощью|  
|TextboxExternalIdentitiesSigninInvitationPrimary|Вход с помощью:|  
  
###  <a name="signinresources"></a><a name="SigninResources"></a> сигнинресаурцес  
  
|Имя|Текст|  
|----------|----------|  
|PrincipalNotFound|Субъект не найден или недопустимая подпись.|  
|ErrorSsoAuthenticationFailed|Сбой проверки подлинности SSO.|  
|ErrorSsoAuthenticationFailedDetailed|Указан недопустимый маркер или не удается проверить подпись.|  
|ErrorSsoTokenInvalid|Маркер SSO недопустим.|  
|ValidationErrorSpecificEmailAlreadyExists|Адрес электронной почты {0} уже зарегистрирован.|  
|ValidationErrorSpecificEmailInvalid|Адрес электронной почты {0} недопустимый.|  
|ValidationErrorPasswordInvalid|Недопустимый пароль. Исправьте ошибки и повторите попытку.|  
|PropertyTooShort|{0} содержит слишком мало символов.|  
|WebAuthenticationAddresserEmailInvalidErrorMessage|Недопустимый адрес электронной почты.|  
|ValidationMessageNewPasswordConfirmationRequired|Подтвердите новый пароль.|  
|ValidationErrorPasswordConfirmationRequired|Поле подтверждения пароля пустое.|  
|WebAuthenticationEmailChangeNotice|Ожидание сообщения электронной почты с подтверждением изменения для {0}. Следуйте инструкциям в нем, чтобы подтвердить свой новый адрес электронной почты. Если сообщение электронной почты не поступило в папку "Входящие" в течение нескольких минут, проверьте папку нежелательной почты.|  
|WebAuthenticationEmailChangeNoticeHeader|Запрос на изменение электронной почты успешно обработан.|  
|WebAuthenticationEmailChangeNoticeTitle|Запрошенное изменение электронной почты.|  
|WebAuthenticationEmailHasBeenRevertedNotice|Ваш адрес электронной почты уже существует. Запрос отменен.|  
|ValidationErrorEmailAlreadyExists|Адрес электронной почты уже существует.|  
|ValidationErrorEmailInvalid|Недопустимый адрес электронной почты.|  
|TextboxLabelEmail|Email|  
|ValidationErrorEmailRequired|Требуется адрес электронной почты.|  
|WebAuthenticationErrorNoticeHeader|Ошибка|  
|WebAuthenticationFieldLengthErrorMessage|Максимальная длина {0} — {1}.|  
|TextboxLabelEmailFirstName|Имя|  
|ValidationErrorFirstNameRequired|Имя — обязательное поле.|  
|ValidationErrorFirstNameInvalid|Недопустимое имя.|  
|NoticeInvalidInvitationToken|Обратите внимание, что ссылки подтверждения являются допустимыми в течение всего 48 часов. Если это время еще не истекло, убедитесь, что ссылка правильная. Если срок действия ссылки истек, повторите действие, которое вы пытаетесь подтвердить.|  
|NoticeHeaderInvalidInvitationToken|Недопустимый маркер приглашения.|  
|NoticeTitleInvalidInvitationToken|Ошибка подтверждения.|  
|WebAuthenticationLastNameInvalidErrorMessage|Недопустимая фамилия.|  
|TextboxLabelEmailLastName|Фамилия|  
|ValidationErrorLastNameRequired|Требуется указать фамилию.|  
|WebAuthenticationLinkExpiredNotice|Истек срок действия отправленной вам ссылки подтверждения. `<a href={0}?token={1}>Resend confirmation email.</a\>`|  
|NoticePasswordResetLinkInvalidOrExpired|Ваша ссылка для сброса пароля недопустима или ее срок действия истек.|  
|WebAuthenticationLinkExpiredNoticeTitle|Ссылка отправлена.|  
|WebAuthenticationNewPasswordLabel|Новый пароль.|  
|ValidationMessageNewPasswordRequired|Требуется новый пароль.|  
|TextboxLabelNotificationsSenderEmail|Электронная почта отправителя уведомлений|  
|TextboxLabelOrganizationName|Название организации|  
|WebAuthenticationOrganizationRequiredErrorMessage|Поле названия организации пустое.|  
|WebAuthenticationPasswordChangedNotice|Ваш пароль успешно обновлен.|  
|WebAuthenticationPasswordChangedNoticeTitle|Пароль обновлен.|  
|WebAuthenticationPasswordCompareErrorMessage|Пароли не совпадают.|  
|WebAuthenticationPasswordConfirmLabel|Подтверждение пароля|  
|ValidationErrorPasswordInvalidDetailed|Пароль слишком ненадежный.|  
|WebAuthenticationPasswordLabel|Пароль|  
|ValidationErrorPasswordRequired|Требуется указать пароль.|  
|WebAuthenticationPasswordResetSendNotice|Ожидание сообщения электронной почты с подтверждением изменения пароля для {0}. Следуйте инструкциям в сообщении электронной почты, чтобы продолжить процесс изменения пароля.|  
|WebAuthenticationPasswordResetSendNoticeHeader|Запрос на сброс пароля успешно обработан.|  
|WebAuthenticationPasswordResetSendNoticeTitle|Сброс пароля запрошен.|  
|WebAuthenticationRequestNotFoundNotice|Запрос не найден.|  
|WebAuthenticationSenderEmailRequiredErrorMessage|Поле электронной почты отправителя уведомлений пустое.|  
|WebAuthenticationSigninPasswordLabel|Подтвердите изменение путем ввода пароля.|  
|WebAuthenticationSignupConfirmNotice|Ожидание сообщения электронной почты с подтверждением регистрации для {0}.<br /\> Следуйте инструкциям в этом сообщении, чтобы активировать учетную запись.<br> /\> Если сообщение электронной почты не поступит в папку "Входящие" в течение следующих нескольких минут, проверьте папку нежелательной почты.|  
|WebAuthenticationSignupConfirmNoticeHeader|Ваша учетная запись успешно создана.|  
|WebAuthenticationSignupConfirmNoticeRepeatHeader|Сообщение электронной почты с подтверждением регистрации повторно отправлено.|  
|WebAuthenticationSignupConfirmNoticeTitle|Созданная учетная запись|  
|WebAuthenticationTokenRequiredErrorMessage|Пустой маркер.|  
|WebAuthenticationUserAlreadyRegisteredNotice|Похоже, что пользователь с этим адресом электронной почты уже зарегистрирован в системе. Если вы забыли пароль, попробуйте восстановить его или обратитесь к специалисту службы поддержки.|  
|WebAuthenticationUserAlreadyRegisteredNoticeHeader|Пользователь уже зарегистрирован.|  
|WebAuthenticationUserAlreadyRegisteredNoticeTitle|Уже зарегистрирован.|  
|ButtonLabelChangePassword|Изменить пароль|  
|ButtonLabelChangeAccountInfo|Изменение сведений об учетной записи|  
|ButtonLabelCloseAccount|Закрыть учетную запись|  
|WebAuthenticationInvalidCaptchaErrorMessage|Введенный текст не соответствует тексту на изображении. Повторите попытку.|  
|ValidationErrorCredentialsInvalid|Недопустимый адрес электронной почты или пароль. Исправьте ошибки и повторите попытку.|  
|WebAuthenticationRequestIsNotValid|Недопустимый запрос.|  
|WebAuthenticationUserIsNotConfirm|Прежде чем войти, подтвердите регистрацию.|  
|WebAuthenticationInvalidEmailFormated|Адрес электронной почты {0} недопустимый.|  
|WebAuthenticationUserNotFound|Не удалось найти пользователя.|  
|WebAuthenticationTenantNotRegistered|Ваша учетная запись принадлежит клиенту Azure Active Directory, который не имеет прав доступа к этому порталу.|  
|WebAuthenticationAuthenticationFailed|Сбой проверки подлинности.|  
|WebAuthenticationGooglePlusNotEnabled|Сбой проверки подлинности. Если приложение авторизовано, обратитесь к администратору, чтобы убедиться, что проверка подлинности Google настроена надлежащим образом.|  
|ValidationErrorAllowedTenantIsRequired|Требуется разрешенный клиент.|  
|ValidationErrorTenantIsNotValid|Клиент Azure Active Directory {0} недопустимый.|  
|WebAuthenticationActiveDirectoryTitle|Azure Active Directory|  
|WebAuthenticationLoginUsingYourProvider|Войдите со своей учетной записью {0}.|  
|WebAuthenticationUserLimitNotice|В этой службе достигнуто максимальное число разрешенных пользователей. `<a href="mailto:{0}"\>contact the administrator</a\>`, чтобы обновить службу и повторно включить регистрацию пользователей.|  
|WebAuthenticationUserLimitNoticeHeader|Регистрация пользователей отключена.|  
|WebAuthenticationUserLimitNoticeTitle|Регистрация пользователей отключена.|  
|WebAuthenticationUserRegistrationDisabledNotice|Регистрация пользователей отключена администратором. Войдите в систему с помощью внешнего поставщика удостоверений.|  
|WebAuthenticationUserRegistrationDisabledNoticeHeader|Регистрация пользователей отключена.|  
|WebAuthenticationUserRegistrationDisabledNoticeTitle|Регистрация пользователей отключена.|  
|WebAuthenticationSignupPendingConfirmationNotice|Прежде чем мы сможем завершить создание вашей учетной записи, нам необходимо проверить ваш адрес электронной почты. Мы отправили сообщение электронной почты на этот адрес: {0}. Чтобы активировать учетную запись, следуйте инструкциям в этом сообщении. Если вы не получите сообщение электронной почты в течение нескольких минут, проверьте папку нежелательной почты.|  
|WebAuthenticationSignupPendingConfirmationAccountFoundNotice|Мы обнаружили неподтвержденную учетную запись, связанную с адресом электронной почты {0}. Чтобы завершить создание вашей учетной записи, нам необходимо проверить ваш адрес электронной почты. Мы отправили сообщение электронной почты на этот адрес: {0}. Чтобы активировать учетную запись, следуйте инструкциям в этом сообщении. Если вы не получили сообщение электронной почты в течение нескольких минут, проверьте папку нежелательной почты.|  
|WebAuthenticationSignupConfirmationAlmostDone|Все почти готово.|  
|WebAuthenticationSignupConfirmationEmailSent|Мы отправили сообщение электронной почты на этот адрес: {0}. Чтобы активировать учетную запись, следуйте инструкциям в этом сообщении. Если вы не получите сообщение электронной почты в течение нескольких минут, проверьте папку нежелательной почты.|  
|WebAuthenticationEmailSentNotificationMessage|Сообщение электронной почты успешно отправлено по адресу {0}.|  
|WebAuthenticationNoAadTenantConfigured|Для службы не настроен клиент Azure Active Directory.|  
|CheckboxLabelUserRegistrationTermsConsentRequired|Я принимаю `<a data-toggle="modal" href="#" data-target="#terms"\>Terms of Use</a\>`.|  
|TextblockUserRegistrationTermsProvided|Просмотрите `<a data-toggle="modal" href="#" data-target="#terms"\>Terms of Use.</a\>`.|  
|DialogHeadingTermsOfUse|Условия использования|  
|ValidationMessageConsentNotAccepted|Прежде чем продолжить, необходимо принять условия использования.|  
  
###  <a name="signinstrings"></a><a name="SigninStrings"></a> сигнинстрингс  
  
|Имя|Текст|  
|----------|----------|  
|WebAuthenticationForgotPassword|Забыли пароль?|  
|WebAuthenticationIfAdministrator|Если вы являетесь администратором, войдите `<a href="{0}"\>here</a\>`.|  
|WebAuthenticationNotAMember|Еще не являетесь участником? `<a href="/signup"\>Sign up now</a\>`|  
|WebAuthenticationRemember|Запомнить меня на этом компьютере|  
|WebAuthenticationSigininWithPassword|Войдите с использованием имени пользователя и пароля.|  
|WebAuthenticationSigninTitle|Вход|  
|WebAuthenticationSignUpNow|Зарегистрируйтесь сейчас|  
  
###  <a name="signupstrings"></a><a name="SignupStrings"></a> сигнупстрингс  
  
|Имя|Текст|  
|----------|----------|  
|PageTitleSignup|Регистрация|  
|WebAuthenticationAlreadyAMember|Уже являетесь участником?|  
|WebAuthenticationCreateNewAccount|Создайте учетную запись для управления API.|  
|WebAuthenticationSigninNow|Войдите сейчас.|  
|ButtonLabelSignup|Регистрация|  
  
###  <a name="subscriptionliststrings"></a><a name="SubscriptionListStrings"></a> субскриптионлистстрингс  
  
|Имя|Текст|  
|----------|----------|  
|SubscriptionCancelConfirmation|Вы действительно хотите отменить эту подписку?|  
|SubscriptionRenewConfirmation|Вы действительно хотите обновить эту подписку?|  
|WebDevelopersManageSubscriptions|Управление подписками|  
|WebDevelopersPrimaryKey|Первичный ключ|  
|WebDevelopersRegenerateLink|Повторно создать|  
|WebDevelopersSecondaryKey|Вторичный ключ|  
|ButtonLabelShowKey|Показать|  
|ButtonLabelRenewSubscription|Возобновление|  
|WebDevelopersSubscriptionRequested|Дата запроса: {0}.|  
|WebDevelopersSubscriptionRequestedState|Запрошено|  
|WebDevelopersSubscriptionTableNameHeader|Имя|  
|WebDevelopersSubscriptionTableStateHeader|Состояние|  
|WebDevelopersUsageStatisticsLink|Аналитические отчеты|  
|WebDevelopersYourSubscriptions|Ваши подписки|  
|SubscriptionPropertyLabelRequestedDate|Дата запроса|  
|SubscriptionPropertyLabelStartedDate|Дата запуска|  
|PageTitleRenameSubscription|Переименовать подписку|  
|SubscriptionPropertyLabelName|имя подписки;|  
  
###  <a name="subscriptionstrings"></a><a name="SubscriptionStrings"></a> субскриптионстрингс  
  
|Имя|Текст|  
|----------|----------|  
|SectionHeadingCloseAccount|Хотите закрыть учетную запись?|  
|PageTitleDeveloperProfile|Профиль|  
|ButtonLabelHideKey|Скрыть|  
|ButtonLabelRegenerateKey|Повторно создать|  
|InformationMessageKeyWasRegenerated|Вы действительно хотите повторно создать этот ключ?|  
|ButtonLabelShowKey|Показать|  
  
###  <a name="updateprofilestrings"></a><a name="UpdateProfileStrings"></a> упдатепрофилестрингс  
  
|Имя|Текст|  
|----------|----------|  
|ButtonLabelUpdateProfile|Обновить профиль|  
|PageTitleUpdateProfile|Обновить сведения об учетной записи|  
  
###  <a name="userprofile"></a><a name="UserProfile"></a> Филе  
  
|Имя|Текст|  
|----------|----------|  
|ButtonLabelChangeAccountInfo|Изменение сведений об учетной записи|  
|ButtonLabelChangePassword|Изменить пароль|  
|ButtonLabelCloseAccount|Закрыть учетную запись|  
|TextboxLabelEmail|Email|  
|TextboxLabelEmailFirstName|Имя|  
|TextboxLabelEmailLastName|Фамилия|  
|TextboxLabelNotificationsSenderEmail|Электронная почта отправителя уведомлений|  
|TextboxLabelOrganizationName|Название организации|  
|SubscriptionStateActive|Активен|  
|SubscriptionStateCancelled|Отменено|  
|SubscriptionStateExpired|Срок действия истек|  
|SubscriptionStateRejected|Отклонено|  
|SubscriptionStateRequested|Запрошено|  
|SubscriptionStateSuspended|Приостановлена|  
|DefaultSubscriptionNameTemplate|{0} (значение по умолчанию).|  
|SubscriptionNameTemplate|Доступ для разработчика #{0}.|  
|TextboxLabelSubscriptionName|имя подписки;|  
|ValidationMessageSubscriptionNameRequired|Имя подписки не может быть пустым.|  
|ApiManagementUserLimitReached|В этой службе достигнуто максимальное число разрешенных пользователей. Выполните обновление до более высокой ценовой категории.|  
  
##  <a name="glyph-resources"></a><a name="glyphs"></a> Ресурсы глифов  
 В шаблонах портала разработчика управления API можно использовать глифы со страницы с [Glyphicons на сайте Bootstrap](https://getbootstrap.com/components/#glyphicons). Этот набор глифов содержит более 250 глифов в формате шрифта из набора Halflings [Glyphicon](https://glyphicons.com/). Чтобы использовать глиф из этого набора, используйте следующий синтаксис.  
  
```html  
<span class="glyphicon glyphicon-user">  
```  
  
 Полный список глифов см. на странице с [Glyphicons на сайте Bootstrap](https://getbootstrap.com/components/#glyphicons).

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о работе с шаблонами см. в статье [Настройка портала разработчика в службе управления API Azure с помощью шаблонов](api-management-developer-portal-templates.md).
