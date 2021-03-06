---
title: Защита серверной части SPA в службе управления API Azure с помощью Active Directory B2C
description: Защитите API с помощью OAuth 2,0, используя Azure Active Directory B2C, управление API Azure и простую проверку подлинности для вызова из JavaScript SPA с помощью потока проверки подлинности PKCE с включенным SPA.
services: api-management, azure-ad-b2c, app-service
documentationcenter: ''
author: WillEastbury
manager: alberts
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/18/2021
ms.author: wieastbu
ms.custom: fasttrack-new, fasttrack-update, devx-track-js
ms.openlocfilehash: baa6a0a6995e206924d14de25b98700e450f3a0c
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104954927"
---
# <a name="protect-spa-backend-with-oauth-20-azure-active-directory-b2c-and-azure-api-management"></a>Защита серверной части SPA с помощью OAuth 2,0, Azure Active Directory B2C и Azure API Management

В этом сценарии показано, как настроить экземпляр службы управления API Azure для защиты API.
Мы будем использовать поток Azure AD B2C SPA (auth code + PKCE) для получения маркера, а также управления API для защиты серверной части функций Azure с помощью Еасяус.

## <a name="aims"></a>Предназначен

Мы посмотрим, как служба управления API может использоваться в упрощенном сценарии с функциями и Azure AD B2C Azure. Вы создадите приложение JavaScript (JS), вызывающее API-интерфейс, который подписывает пользователей с Azure AD B2C. Затем вы будете использовать функции проверки-JWT, CORS и ограничения скорости службы управления API для защиты серверного API.

Для глубокой защиты мы используем Еасяус для повторной проверки маркера внутри API серверной части и убедитесь, что служба управления API является единственной службой, которая может вызывать серверную часть функций Azure.

## <a name="what-will-you-learn"></a>Новые знания

> [!div class="checklist"]
> * Установка одностраничного приложения и API серверной части в Azure Active Directory B2C
> * Создание API серверной части функций Azure
> * Импорт API функций Azure в службу управления API Azure
> * Защита API в службе управления API Azure
> * Вызов конечных точек авторизации Azure Active Directory B2C через библиотеки платформы удостоверений Майкрософт (MSAL.js)
> * Сохранение одностраничного приложения HTML/обычный JS и его обслуживание из конечной точки хранилища BLOB-объектов Azure

## <a name="prerequisites"></a>Предварительные условия

Чтобы выполнить шаги в этой статье, необходимо иметь следующее:

* Учетная запись хранения Azure (StorageV2) общего назначения v2 для размещения одностраничного приложения с одним внешним интерфейсом.
* Экземпляр службы управления API Azure (любой уровень будет работать, в том числе "потребление", но некоторые функции, применимые к полному сценарию, недоступны на этом уровне ("частота" и "выделенный виртуальный IP-адрес"), эти ограничения вызываются ниже в соответствующей статье.
* Пустое приложение-функция Azure (запуск среды выполнения версии 3.1 .NET Core в плане потребления) для размещения вызванного API
* Клиент Azure AD B2C, связанный с подпиской.

Хотя на практике вы бы использовали ресурсы в одном регионе в производственных рабочих нагрузках, в этом практическом руководстве Развертывание области развертывания не имеет значения.

## <a name="overview"></a>Обзор

Ниже приведена иллюстрация используемых компонентов и последовательность между ними после завершения процесса.
![Используемые компоненты и последовательность](../api-management/media/howto-protect-backend-frontend-azure-ad-b2c/image-arch.png "Используемые компоненты и последовательность")

Ниже приведен краткий обзор действий.

1. Создание Azure AD B2C вызова (интерфейсного интерфейса, управления API) и приложений API с областями и предоставление доступа через API
1. Создайте политики регистрации и входа, чтобы разрешить пользователям выполнять вход с помощью Azure AD B2C 
1. Настройка управления API с помощью новых идентификаторов и ключей клиента Azure AD B2C для включения авторизации пользователей OAuth2 в консоли разработчика
1. Создание API функции
1. Настройте API функции, чтобы включить Еасяус с новым ИДЕНТИФИКАТОРом и ключами клиента Azure AD B2C и заблокировать для APIM VIP.
1. Создание определения API в управлении API
1. Настройка Oauth2 для конфигурации API управления API
1. Настройте политику **CORS** и добавьте политику **Validate-JWT** , чтобы проверить маркер OAuth для каждого входящего запроса.
1. Создание вызывающего приложения для использования API
1. Отправка примера с помощью JS SPA
1. Настройка клиентского приложения Sample JS с помощью нового идентификатора клиента Azure AD B2C и ключей 
1. Тестирование клиентского приложения

   > [!TIP]
   > Мы собираемся захватывать несколько фрагментов информации и ключей и т. д. в этом документе может оказаться удобным открыть текстовый редактор для временного хранения следующих элементов конфигурации.  
   >
   > ИДЕНТИФИКАТОР ВНУТРЕННЕГО КЛИЕНТА B2C:  
   > СЕКРЕТНЫЙ КЛЮЧ B2C ВНУТРЕННЕГО КЛИЕНТА:  
   > URI ОБЛАСТИ API СЕРВЕРНОЙ ЧАСТИ B2C:  
   > ИДЕНТИФИКАТОР B2C ВНЕШНЕГО КЛИЕНТА:  
   > URI КОНЕЧНОЙ ТОЧКИ ПОТОКА ПОЛЬЗОВАТЕЛЕЙ B2C:  
   > B2C ХОРОШО ИЗВЕСТНАЯ КОНЕЧНАЯ ТОЧКА OPENID CONNECT:   
   > Имя политики B2C: Frontendapp_signupandsignin URL-адрес функции:  
   > БАЗОВЫЙ URL-АДРЕС API APIM: URL-АДРЕС ОСНОВНОЙ КОНЕЧНОЙ ТОЧКИ ХРАНИЛИЩА:  

## <a name="configure-the-backend-application"></a>Настройка серверного приложения

Откройте колонку Azure AD B2C на портале и выполните следующие действия.

1. Выберите вкладку **Регистрация приложений** .
1. Нажмите кнопку "создать регистрацию".
1. Выберите "веб-узел" в поле "Выбор URI перенаправления".
1. Теперь задайте отображаемое имя, выберите уникальное значение и соответствующее создаваемой службе. В этом примере мы будем использовать имя "серверное приложение".
1. Используйте заполнители для URL-адресов ответа, например " https://jwt.ms " (сайт декодирования токена Майкрософт), мы будем обновлять эти URL-адреса позже.
1. Убедитесь, что выбран параметр "учетные записи в любом поставщике удостоверений или в каталоге организации (для проверки подлинности пользователей с помощью потоков пользователей)".
1. В этом примере снимите флажок "предоставить согласие администратора", так как в настоящее время не требуются разрешения offline_access.
1. Щелкните "Зарегистрировать".
1. Запишите идентификатор клиента серверного приложения для последующего использования (отображается в разделе "Application (Client) ID").
1. Перейдите на вкладку *Сертификаты и секреты* (в разделе Управление) и нажмите кнопку "создать секрет клиента", чтобы создать ключ проверки подлинности (примите параметры по умолчанию и нажмите кнопку "Добавить").
1. После нажатия кнопки "Добавить" скопируйте ключ (в разделе "значение") в безопасном месте для дальнейшего использования в качестве секрета внутреннего клиента. Обратите внимание, что это диалоговое окно является единственным шансом скопировать этот ключ.
1. Теперь выберите вкладку *API* (в разделе Управление).
1. Вам будет предложено задать URI AppID, выбрать и записать значение по умолчанию.
1. Создайте и назовите область "Hello" для API функции. можно использовать фразу "Hello" для всех вводимых параметров, записав заполненный URI полной области, а затем нажмите кнопку "добавить область".
1. Вернитесь в корневую колонку Azure AD B2C, выбрав иерархическую панель "Azure AD B2C" в верхней левой части портала.

   > [!NOTE]
   > Области Azure AD B2C являются фактическими разрешениями в API, которые другие приложения могут запрашивать доступ к через колонку доступа API из своих приложений, фактически вы только что создали разрешения приложения для вызванного API.

## <a name="configure-the-frontend-application"></a>Настройка внешнего приложения

1. Выберите вкладку **Регистрация приложений** .
1. Нажмите кнопку "создать регистрацию".
1. Выберите "одностраничное приложение (SPA)" в поле "Выбор URI перенаправления".
1. Теперь задайте отображаемое имя и URI AppID, выберите уникальные значения и соответствующие интерфейсному приложению, которое будет использовать эту AAD B2C регистрации приложения. В этом примере можно использовать "интерфейсное приложение".
1. В соответствии с первой регистрацией приложения Оставьте выбранные типы учетных записей равными по умолчанию (проверка подлинности пользователей с помощью потоков пользователей).
1. Используйте заполнители для URL-адресов ответа, например " https://jwt.ms " (сайт декодирования токена Майкрософт), мы будем обновлять эти URL-адреса позже.
1. Оставьте поле согласия администратора с тактовым
1. Щелкните "Зарегистрировать".
1. Запишите идентификатор клиента внешнего приложения для последующего использования (отображается в разделе "Application (Client) ID").
1. Перейдите на вкладку *разрешения API* .
1. Предоставьте доступ к серверному приложению, щелкнув "добавить разрешение", затем "Мои API", выберите "серверное приложение", выберите "разрешения", выберите область, созданную в предыдущем разделе, и нажмите кнопку "добавить разрешения".
1. Щелкните "предоставить согласие администратора для {клиент}" и нажмите кнопку "Да" во всплывающем диалоговом окне. Это всплывающее окно позволяет отсылать "внешнее приложение" для использования разрешения "Hello", определенного в "серверном приложении", созданном ранее.
1. Теперь все разрешения должны отображаться для приложения в виде зеленого тика в столбце состояние.

## <a name="create-a-sign-up-and-sign-in-user-flow"></a>Создание потока пользователя "регистрация и вход"

1. Вернитесь в корневую колонку B2C, выбрав иерархию Azure AD B2C.
1. Перейдите на вкладку "Маршруты пользователей" (в разделе "политики").
1. Щелкните "новый поток пользователей".
1. Выберите тип потока пользователя "регистрация и вход" и выберите "рекомендуется", а затем "создать".
1. Присвойте политике имя и запишите ее для последующего использования. В этом примере можно использовать "Frontendapp_signupandsignin", обратите внимание, что в качестве префикса будет использоваться "B2C_1_", чтобы сделать "B2C_1_Frontendapp_signupandsignin"
1. В разделе "поставщики удостоверений" и "Локальные учетные записи" установите флажок "регистрация электронной почты" (или "Регистрация идентификатора пользователя" в зависимости от конфигурации клиента B2C) и нажмите кнопку "ОК". Эта конфигурация обусловлена тем, что мы будем регистрировать локальные учетные записи B2C, а не отменять доступ к другому поставщику удостоверений (например, поставщику социальных сетей) для использования существующей учетной записи социальных сетей пользователя.
1. Оставьте параметры MFA и условный доступ по умолчанию.
1. В разделе "атрибуты и утверждения пользователей" щелкните "Дополнительно...". затем выберите параметры утверждения, которые пользователи должны ввести и вернуть в маркер. Проверьте по меньшей мере "отображаемое имя" и "адрес электронной почты" для сбора с помощью команды "отображаемое имя" и "адреса электронной почты" (Обратите внимание на тот факт, что вы собираете значение EmailAddress, единственное, и запрашивается возврат адресов электронной почты, несколько) и нажмите кнопку "ОК", а затем — "создать".
1. Щелкните пользовательский поток, созданный в списке, а затем нажмите кнопку "запустить поток пользователя".
1. Это действие откроет колонку запуск потока пользователей, выберите внешнее приложение, скопируйте конечную точку потока пользователя и сохраните ее для последующего использования.
1. Скопируйте и сохраните ссылку вверху, записав в качестве стандартной конечной точки конфигурации OpenID Connect, для последующего использования.

   > [!NOTE]
   > Политики B2C позволяют предоставлять конечные точки входа в Azure AD B2C, чтобы иметь возможность записывать различные компоненты данных и выполнять вход пользователей различными способами.
   > 
   > В этом случае мы настроили последовательность регистрации или входа (политику). Это также предоставляло хорошо известную конечную точку конфигурации. в обоих случаях созданная политика была определена в URL-адресе параметром строки запроса "p =".  
   >
   > После этого вы получите функциональную платформу бизнес-клиентов, которая будет подписывать пользователей в нескольких приложениях.

## <a name="build-the-function-api"></a>Создание API функции

1. Вернитесь к стандартному клиенту Azure AD в портал Azure, чтобы мы могли еще раз настроить элементы в подписке.
1. Перейдите в колонку "приложения-функции" портал Azure откройте пустое приложение-функцию, а затем щелкните "функции" и нажмите кнопку "Добавить".
1. В открывшемся всплывающем окне выберите "Разработка на портале", в разделе "Выбор шаблона" выберите "триггер HTTP", в разделе "сведения о шаблоне" введите "Hello" с уровнем авторизации "функция", а затем выберите Добавить.
1. Перейдите в колонку код + тест и скопируйте и вставьте пример кода, приведенный ниже, *над имеющимся кодом* .
1. Щелкните "Сохранить".

   ```csharp
   
   using System.Net;
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.Extensions.Primitives;
   
   public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
   {
      log.LogInformation("C# HTTP trigger function processed a request.");
      
      return (ActionResult)new OkObjectResult($"Hello World, time and date are {DateTime.Now.ToString()}");
   }
   
   ```

   > [!TIP]
   > Только что вставленный код функции сценария c# просто записывает строку в журналы функций и возвращает текст "Hello World" с некоторыми динамическими данными (датой и временем).

1. Выберите "Интеграция" в левой колонке, а затем щелкните ссылку http (REQ) в поле "триггер".
1. В раскрывающемся списке "выбранные методы HTTP" снимите флажок HTTP POST, после чего выводится только флажок, а затем нажмите кнопку Сохранить.
1. Вернитесь на вкладку код + тест, щелкните "получить URL-адрес функции", затем скопируйте URL-адрес, который будет отображаться, и сохраните его для последующего использования.

   > [!NOTE]
   > Только что созданные привязки просто указывают функциям реагировать на анонимные HTTP-запросы GET на только что скопированный URL ( `https://yourfunctionappname.azurewebsites.net/api/hello?code=secretkey` ). Теперь у нас есть масштабируемый серверный API HTTPS, который способен возвращать очень простые полезные данные.
   >
   > Теперь вы можете проверить вызов этого API из веб-браузера, используя указанную вами версию URL-адреса, который вы только что скопировали и сохранили. Можно также удалить часть параметров строки запроса "? Code = секреткэй" в URL-адресе и повторить проверку, чтобы доказать, что функции Azure будут возвращать ошибку 401.

## <a name="configure-and-secure-the-function-api"></a>Настройка и защита API-интерфейса функции

1. Необходимо настроить две дополнительные области в приложении-функции (ограничения авторизации и сети).
1. Сначала давайте настроим проверку подлинности и авторизацию, чтобы вернуться к корневой колонке приложения-функции через навигатор.
1. Далее выберите Проверка подлинности или авторизация (в разделе "Параметры").
1. Включите функцию проверки подлинности службы приложений.
1. Задайте действие, которое будет выполняться, когда запрос не прошел проверку подлинности, чтобы "войти с помощью Azure Active Directory".
1. В разделе "поставщики проверки подлинности" выберите "Azure Active Directory".
1. Выберите "Дополнительно" в параметре режима управления.
1. Вставьте идентификатор клиента [Application] внутреннего приложения (из Azure AD B2C) в поле "идентификатор клиента".
1. Вставьте известную конечную точку конфигурации Open-ID из политики регистрации и входа в поле URL-адрес издателя (ранее мы записали эту конфигурацию).
1. Щелкните "отобразить секрет" и вставьте секрет клиента внутреннего приложения в соответствующее поле.
1. Нажмите кнопку ОК, чтобы вернуться к колонке или экрану выбора поставщика удостоверений.
1. В разделе Дополнительные параметры оставьте [хранилище токенов](../app-service/overview-authentication-authorization.md#token-store) включено (по умолчанию).
1. Нажмите кнопку Save (сохранить) (в верхней левой части колонки).

   > [!IMPORTANT]
   > Теперь API-интерфейс функции развернут и должен вызывать 401 ответов, если нужный JWT не указан в качестве заголовка Authorization: Bearer, и должен возвращать данные при появлении допустимого запроса.  
   > Вы добавили дополнительную защиту на уровне глубокой защиты в Еасяус, настроив параметр "Login with Azure AD" для обработки запросов без проверки подлинности. Имейте в виду, что это изменит поведение неавторизованного запроса между серверной частью приложение-функция и сервером SPA, так как Еасяус будет выдавать перенаправление 302 в AAD вместо ответа 401, не поддерживающего проверку подлинности, мы исправим это с помощью службы управления API позже.  
   >
   > Мы по-прежнему не установили безопасность IP, если у вас есть допустимый ключ и маркер OAuth2, любой пользователь может вызвать его из любого места. в идеале мы хотим принудительно выполнять все запросы через Управление API.  
   > 
   > Если вы используете уровень потребления APIM, то у вас [нет выделенного виртуального IP-адреса службы управления API Azure](./api-management-howto-ip-addresses.md#ip-addresses-of-consumption-tier-api-management-service) для списка разрешений с функциями ограничения доступа. В стандартном номере SKU службы управления API Azure и более поздних [виртуальных IP-адресов — это один клиент и время существования ресурса](./api-management-howto-ip-addresses.md#changes-to-the-ip-addresses). Для уровня потребления службы управления API Azure можно заблокировать вызовы API через ключ общей секретной функции в той части URI, которую вы скопировали ранее. Кроме того, для уровня потребления — шаги 12-17 ниже не применяются.

1. Закройте колонку "Проверка подлинности и авторизация" 
1. Откройте *колонку управления API на портале*, а затем откройте *свой экземпляр*.
1. Запишите частный виртуальный IP-адрес, показанный на вкладке Обзор.
1. Вернитесь в *колонку функций Azure на портале* и снова откройте *экземпляр* .
1. Выберите "сеть" и щелкните "настроить ограничения доступа".
1. Нажмите кнопку "добавить правило" и введите виртуальный IP-адрес, скопированный в шаге 3 выше, в формате XX. XX. XX. XX/32.
1. Если вы хотите продолжить взаимодействие с порталом функций и выполнить дополнительные действия, приведенные ниже, следует также добавить собственный общедоступный IP-адрес или диапазон CIDR.
1. Если в списке есть запись Allow, Azure добавит неявное запрещающее правило для блокировки всех остальных адресов.

Вам потребуется добавить отформатированные блоки CIDR на панель ограничения IP-адресов. Если вам нужно добавить один адрес, например, в качестве виртуального IP-адреса управления API, необходимо добавить его в формате XX. XX. XX. XX/32.

   > [!NOTE]
   > Теперь API-интерфейс функции не следует вызывать из любого места, отличного от управления через Управление API, или вашего адреса.

1. Откройте *колонку управления API*, а затем откройте *свой экземпляр*.
1. Выберите колонку API (в разделе API).
1. В области "добавить новый API" выберите "приложение-функция", а затем в верхней части всплывающего окна выберите "полная".
1. Нажмите кнопку Обзор, выберите приложение-функцию, в котором вы размещаете API, и нажмите кнопку Выбрать. Затем нажмите кнопку выбрать еще раз.
1. Присвойте API имя и описание для внутреннего использования управления API и добавьте его в продукт "неограниченный".
1. Скопируйте и запишите базовый URL-адрес API и нажмите кнопку "создать".
1. Перейдите на вкладку "Параметры", а затем в разделе "Подписка" — отключите флажок "требуется подписка", так как в этом случае будет использоваться токен OAuth JWT для ограничения скорости. Обратите внимание, что при использовании уровня потребления это по-прежнему потребуется в рабочей среде. 

   > [!TIP]
   > Если используется уровень потребления APIM, неограниченный продукт будет недоступен в качестве предустановленного. Вместо этого перейдите к разделу "продукты" в разделе "API" и нажмите кнопку "Добавить".  
   > Введите "unlimited" в качестве имени и описания продукта и выберите API, который вы только что добавили из выноски API "+" в нижней левой части экрана. Установите флажок "Опубликовано". Оставьте остальные значения по умолчанию. Наконец, нажмите кнопку "создать". При этом был создан неограниченный продукт и он был назначен вашему API. Вы можете настроить новый продукт позже.

## <a name="configure-and-capture-the-correct-storage-endpoint-settings"></a>Настройка и запись правильных параметров конечной точки хранилища

1. Откройте колонку учетные записи хранения в портал Azure 
1. Выберите созданную учетную запись и откройте колонку "статический веб-сайт" в разделе "Параметры" (если вы не видите параметр "статический веб-сайт", проверьте, создана ли учетная запись версии 2).
1. Задайте для компонента статического веб-размещения значение "включено" и задайте для индекса имя документа значение "index.html", а затем нажмите кнопку "Сохранить".
1. Запишите содержимое поля "Основная конечная точка" для последующего использования, так как в этом расположении будет размещаться интерфейсный сайт.

   > [!TIP]
   > Вы можете использовать хранилище BLOB-объектов Azure + перезапись CDN, или служба приложений Azure для размещения функции размещения статических веб-сайтов в режиме SPA, но в этом случае мы используем контейнер по умолчанию для обслуживания статического веб-содержимого/HTML/JS/CSS в службе хранилища Azure и выдаст для нас страницу по умолчанию для нулевой работы.

## <a name="set-up-the-cors-and-validate-jwt-policies"></a>Настройка политик **CORS** и **Validate-JWT**

> Следующие разделы должны следовать вне зависимости от используемого уровня APIM. URL-адрес учетной записи хранения доступен из учетной записи хранения, которую вы предоставили в разделе Предварительные требования в верхней части этой статьи.
1. Перейдите в колонку управления API портала и откройте свой экземпляр.
1. Выберите интерфейсы API, а затем выберите "все API".
1. В разделе "входящая обработка" нажмите кнопку представления кода "</>", чтобы открыть редактор политик.
1. Измените раздел Inbound и вставьте приведенный ниже XML-код, чтобы он выглядел следующим образом.
1. Замените следующие параметры в политике.
1. {Примаристоражеендпоинт} (Основная конечная точка хранилища, скопированная в предыдущем разделе), {b2cpolicy-known-OpenID Connect} ("хорошо известная конечная точка конфигурации OpenID Connect", скопированная ранее) и {серверный интерфейс API-приложения-клиент-идентификатор} (идентификатор приложения или клиента B2C для **серверного API**) с правильными значениями, сохраненными ранее.
1. Если вы используете уровень потребления для службы управления API, следует удалить обе политики с ограничением скорости по ключу, так как эта политика недоступна при использовании уровня потребления службы управления API Azure.

   ```xml
   <inbound>
      <cors allow-credentials="true">
            <allowed-origins>
                <origin>{PrimaryStorageEndpoint}</origin>
            </allowed-origins>
            <allowed-methods preflight-result-max-age="120">
                <method>GET</method>
            </allowed-methods>
            <allowed-headers>
                <header>*</header>
            </allowed-headers>
            <expose-headers>
                <header>*</header>
            </expose-headers>
        </cors>
      <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid." require-expiration-time="true" require-signed-tokens="true" clock-skew="300">
         <openid-config url="{b2cpolicy-well-known-openid}" />
         <required-claims>
            <claim name="aud">
               <value>{backend-api-application-client-id}</value>
            </claim>
         </required-claims>
      </validate-jwt>
      <rate-limit-by-key calls="300" renewal-period="120" counter-key="@(context.Request.IpAddress)" />
      <rate-limit-by-key calls="15" renewal-period="60" counter-key="@(context.Request.Headers.GetValueOrDefault("Authorization","").AsJwt()?.Subject)" />
   </inbound>
   ```

   > [!NOTE]
   > Теперь служба управления API Azure может отвечать на запросы между источниками из приложений JavaScript SPA, а также выполнять регулирование, ограничение скорости и предварительную проверку маркера проверки подлинности JWT перед перенаправлением запроса на API-интерфейс функции.
   > 
   > Поздравляем! теперь у вас есть Azure AD B2C, управление API и функции Azure, которые работают вместе для публикации, защиты и использования API!

   > [!TIP]
   > Если вы используете уровень потребления для управления API, а не ограничиваете его субъектом JWT или входящим IP-адресом (ограничение частоты вызовов по ключевым политикам не поддерживается в настоящее время для уровня потребления), вы можете ограничить квоты скорости вызовов, как показано [здесь](./api-management-access-restriction-policies.md#LimitCallRate).  
   > Так как этот пример является одностраничным приложением JavaScript, мы используем ключ управления API только для вызовов с ограничением частоты и выставления счетов. Фактическая авторизация и проверка подлинности обрабатываются Azure AD B2C и инкапсулированы в JWT, который проверяется дважды, один раз в службе управления API, а затем в серверной функции Azure.

## <a name="upload-the-javascript-spa-sample-to-static-storage"></a>Отправка образца JavaScript SPA в статическое хранилище

1. В колонке учетной записи хранения выберите колонку контейнеры в разделе Служба BLOB-объектов и щелкните контейнер $web, который отображается на правой панели.
1. Сохраните приведенный ниже код в файле локально на компьютере как index.html, а затем отправьте файл index.html в контейнер $web.

   ```html
    <!doctype html>
    <html lang="en">
    <head>
         <meta charset="utf-8">
         <meta http-equiv="X-UA-Compatible" content="IE=edge">
         <meta name="viewport" content="width=device-width, initial-scale=1">
         <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-BmbxuPwQa2lc/FVzBcNJ7UAyJxM6wuqIj61tLrc4wSX0szH/Ev+nYRRuWlolflfl" crossorigin="anonymous">
         <script type="text/javascript" src="https://alcdn.msauth.net/browser/2.11.1/js/msal-browser.min.js"></script>
    </head>
    <body>
         <div class="container-fluid">
             <div class="row">
                 <div class="col-md-12">
                    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
                        <div class="container-fluid">
                            <a class="navbar-brand" href="#">Azure Active Directory B2C with Azure API Management</a>
                            <div class="navbar-nav">
                                <button class="btn btn-success" id="signinbtn"  onClick="login()">Sign In</a>
                            </div>
                        </div>
                    </nav>
                 </div>
             </div>
             <div class="row">
                 <div class="col-md-12">
                     <div class="card" >
                        <div id="cardheader" class="card-header">
                            <div class="card-text"id="message">Please sign in to continue</div>
                        </div>
                        <div class="card-body">
                            <button class="btn btn-warning" id="callapibtn" onClick="getAPIData()">Call API</a>
                            <div id="progress" class="spinner-border" role="status">
                                <span class="visually-hidden">Loading...</span>
                            </div>
                        </div>
                     </div>
                 </div>
             </div>
         </div>
         <script lang="javascript">
                // Just change the values in this config object ONLY.
                var config = {
                    msal: {
                        auth: {
                            clientId: "{CLIENTID}", // This is the client ID of your FRONTEND application that you registered with the SPA type in AAD B2C
                            authority:  "{YOURAUTHORITYB2C}", // Formatted as https://{b2ctenantname}.b2clogin.com/tfp/{b2ctenantguid or full tenant name including onmicrosoft.com}/{signuporinpolicyname}
                            redirectUri: "{StoragePrimaryEndpoint}", // The storage hosting address of the SPA, a web-enabled v2 storage account - recorded earlier as the Primary Endpoint.
                            knownAuthorities: ["{B2CTENANTDOMAIN}"] // {b2ctenantname}.b2clogin.com
                        },
                        cache: {
                            cacheLocation: "sessionStorage",
                            storeAuthStateInCookie: false 
                        }
                    },
                    api: {
                        scopes: ["{BACKENDAPISCOPE}"], // The scope that we request for the API from B2C, this should be the backend API scope, with the full URI.
                        backend: "{APIBASEURL}/hello" // The location that we will call for the backend api, this should be hosted in API Management, suffixed with the name of the API operation (in the sample this is '/hello').
                    }
                }
                document.getElementById("callapibtn").hidden = true;
                document.getElementById("progress").hidden = true;
                const myMSALObj = new msal.PublicClientApplication(config.msal);
                myMSALObj.handleRedirectPromise().then((tokenResponse) => {
                    if(tokenResponse !== null){
                        console.log(tokenResponse.account);
                        document.getElementById("message").innerHTML = "Welcome, " + tokenResponse.account.name;
                        document.getElementById("signinbtn").hidden = true;
                        document.getElementById("callapibtn").hidden = false;
                    }}).catch((error) => {console.log("Error Signing in:" + error);
                });
                function login() {
                    try {
                        myMSALObj.loginRedirect({scopes: config.api.scopes});
                    } catch (err) {console.log(err);}
                }
                function getAPIData() {
                    document.getElementById("progress").hidden = false; 
                    document.getElementById("message").innerHTML = "Calling backend ... "
                    document.getElementById("cardheader").classList.remove('bg-success','bg-warning','bg-danger');
                    myMSALObj.acquireTokenSilent({scopes: config.api.scopes, account: getAccount()}).then(tokenResponse => {
                        const headers = new Headers();
                        headers.append("Authorization", `Bearer ${tokenResponse.accessToken}`);
                        fetch(config.api.backend, {method: "GET", headers: headers})
                            .then(async (response)  => {
                                if (!response.ok)
                                {
                                    document.getElementById("message").innerHTML = "Error: " + response.status + " " + JSON.parse(await response.text()).message;
                                    document.getElementById("cardheader").classList.add('bg-warning');
                                }
                                else
                                {
                                    document.getElementById("cardheader").classList.add('bg-success');
                                    document.getElementById("message").innerHTML = await response.text();
                                }
                                }).catch(async (error) => {
                                    document.getElementById("cardheader").classList.add('bg-danger');
                                    document.getElementById("message").innerHTML = "Error: " + error;
                                });
                    }).catch(error => {console.log("Error Acquiring Token Silently: " + error);
                        return myMSALObj.acquireTokenRedirect({scopes: config.api.scopes, forceRefresh: false})
                    });
                    document.getElementById("progress").hidden = true;
             }
            function getAccount() {
                var accounts = myMSALObj.getAllAccounts();
                if (!accounts || accounts.length === 0) {
                    return null;
                } else {
                    return accounts[0];
                }
            }
        </script>
     </body>
    </html>
   ```

1. Перейдите на основную конечную точку статического веб-сайта, сохраненную ранее в последнем разделе.

   > [!NOTE]
   > Поздравляем! вы только что развернули одностраничное приложение JavaScript в размещении статического содержимого в службе хранилища Azure.  
   > Так как приложение JS еще не настроено с помощью сведений о Azure AD B2C — страница не будет работать, если она открыта.

## <a name="configure-the-javascript-spa-for-azure-ad-b2c"></a>Настройка JavaScript SPA для Azure AD B2C

1. Теперь мы понимаем, что все: мы можем настроить SPA с помощью соответствующего адреса API управления API и правильных Azure AD B2C идентификаторов приложения или клиента.
1. Вернуться к колонке хранилища портал Azure 
1. Выберите "контейнеры" (в разделе "Параметры") 
1. Выберите контейнер "$web" из списка.
1. Выберите в списке index.htmбольшой двоичный объект. 
1. Нажмите кнопку "Изменить". 
1. Обновите значения проверки подлинности в разделе конфигурации msal, чтобы они соответствовали вашему *интерфейсному* приложению, зарегистрированному в B2C ранее. Используйте комментарии к коду для указания того, как должны выглядеть значения конфигурации.
Значение *Authority* должно быть в следующем формате:-https://{b2ctenantname}. b2clogin. com/TFP/{b2ctenantname}. onmicrosoft. com}/{сигнупандсигнинполицинаме}. Если вы использовали наши примеры имен, а клиент B2C называется Contoso, вы бы хотели, чтобы этот центр был " https://contoso.b2clogin.com/tfp/contoso.onmicrosoft.com}/Frontendapp_signupandsignin ".
1. Задайте значения API, соответствующие серверному адресу (базовому URL-адресу API, который вы записали ранее, и значения "b2cScopes" были записаны ранее для *серверного приложения*).
1. Щелкните Сохранить

## <a name="set-the-redirect-uris-for-the-azure-ad-b2c-frontend-app"></a>Задание URI перенаправления для многосерверного приложения Azure AD B2C

1. Откройте колонку Azure AD B2C и перейдите к регистрации приложения для внешнего приложения JavaScript.
1. Щелкните "перенаправить URI" и удалите заполнитель "", который https://jwt.ms мы введете ранее.
1. Добавьте новый универсальный код ресурса (URI) для основной конечной точки (хранилища) (за вычетом замыкающей косой черты).

   > [!NOTE]
   > Эта конфигурация приведет к тому, что клиент внешнего приложения получит маркер доступа с соответствующими утверждениями от Azure AD B2C.  
   > SPA сможет добавить это как токен носителя в заголовок HTTPS в вызове API серверной части.  
   > 
   > Служба управления API будет предварительно проверять маркер, вызовы лимита скорости в конечную точку, как субъект JWT, выданный ИДЕНТИФИКАТОРом Azure (пользователем), и IP-адрес вызывающего объекта (в зависимости от уровня службы управления API). перед передачей запроса на получение API функции Azure добавьте ключ безопасности функций.  
   > SPA будет отображать ответ в браузере.
   >
   > *Поздравляем! вы настроили Azure AD B2C, управление API Azure, функции Azure, авторизация службы приложений Azure для оптимального решения.*

Теперь у нас есть простое приложение с простым защищенным API, протестировать его.

## <a name="test-the-client-application"></a>Тестирование клиентского приложения

1. Откройте пример URL-адреса приложения, который вы заговорили в созданной ранее учетной записи хранения.
1. Нажмите кнопку "войти" в правом верхнем углу, чтобы открыть Azure AD B2C профиль регистрации или входа.
1. Приложение должно Вас приветствует Ваше имя профиля B2C.
1. Теперь нажмите кнопку "вызвать API", и страница должна быть обновлена значениями, отправленными из защищенного API.
1. Если вы *повторно* наберете кнопку Call API и работаете на уровне разработчика или более поздней версии службы управления API, следует отметить, что ваше решение начинает ОЦЕНИВАТЬ ограничение API, и эта функция должна быть указана в приложении с соответствующим сообщением.

## <a name="and-were-done"></a>И все готово

Описанные выше действия можно адаптировать и изменить, чтобы обеспечить множество различных применений Azure AD B2C с помощью управления API.

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте больше об [Azure Active Directory и OAuth 2.0](../active-directory/develop/authentication-vs-authorization.md).
* См. другие [видео](https://azure.microsoft.com/documentation/videos/index/?services=api-management) об управлении API.
* Другие способы защиты внутренней серверной службы см. в разделе [Взаимная проверка подлинности на основе сертификата](api-management-howto-mutual-certificates.md).
* [Создайте экземпляр службы управления API](get-started-create-service-instance.md).
* [Импорт и публикация первого API](import-and-publish.md)