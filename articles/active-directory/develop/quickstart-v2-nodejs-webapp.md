---
title: Краткое руководство. Добавление возможности входа пользователей в веб-приложение Node.js | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как реализовать проверку подлинности в веб-приложении Node.js с помощью OpenID Connect.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/28/2019
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:ASP.NET, devx-track-js
ms.openlocfilehash: f7f14b91dc69eeba4ac06f6608f6151634dc38d3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100103504"
---
# <a name="quickstart-add-sign-in-using-openid-connect-to-a-nodejs-web-app"></a>Краткое руководство. Добавление возможности входа в веб-приложение Node.js с помощью OpenID Connect

Из этого краткого руководства вы узнаете, как скачать и выполнить пример кода, в котором показана настройка проверки подлинности OpenID Connect в веб-приложении, созданном с помощью Node.js и Экспресс. Пример предназначен для работы на любой платформе.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Node.js](https://nodejs.org/en/download/).

## <a name="register-your-application"></a>Регистрация приложения

1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
1. Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
1. Введите **имя** приложения, например `MyWebApp`. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
1. В разделе **Поддерживаемые типы учетных записей** выберите **Учетные записи в любом каталоге организации и личные учетные записи Майкрософт (например, Skype, Xbox, Outlook.com)** .

    При наличии нескольких URI перенаправления добавьте их на вкладке **Аутентификация** после успешного создания приложения.

1. Выберите **Зарегистрировать**, чтобы создать приложение.
1. На странице приложения **Обзор** найдите **идентификатор приложения (клиента)** и запишите его, чтобы использовать позже. Это значение потребуется позже в этом проекте для настройки приложения.
1. В разделе **Управление** выберите **Проверка подлинности**.
1. Выберите **Добавить платформу** > **Веб**. 
1. В разделе **URI перенаправления** введите `http://localhost:3000/auth/openid/return`.
1. Задайте для параметра **URL-адрес выхода Front-channel** значение `https://localhost:3000`.
1. В разделе **Неявное предоставление разрешения и гибридные потоки** выберите **Маркеры идентификации**, так как при работе с этим примером необходимо включить [поток неявного предоставления разрешения](./v2-oauth2-implicit-grant-flow.md) для входа пользователя.
1. Нажмите кнопку **Настроить**.
1. В разделе **Управление** выберите **Сертификаты и секреты** > **Создать секрет клиента**.
1. Введите описание ключа (для экземпляра секрета приложения).
1. Выберите срок действия ключа **1 год, 2 года,** или **Срок действия неограничен**.
1. Выберите **Добавить**. Отобразится значение ключа. Скопируйте и сохраните значение ключа в безопасном месте, так как оно понадобится позже.


## <a name="download-the-sample-application-and-modules"></a>Загрузка примера приложения и модулей

Затем клонируйте пример репозитория и установите модули NPM.

Из оболочки или командной строки:

`$ git clone git@github.com:AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs.git`

или диспетчер конфигурации служб

`$ git clone https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs.git`

В корневом каталоге проекта выполните команду:

`$ npm install`

## <a name="configure-the-application"></a>Настройка приложения

Укажите параметры в файле `exports.creds` config.js, следуя инструкциям.

* Обновите `<tenant_name>` в `exports.identityMetadata` с именем клиента Azure AD в формате \*.onmicrosoft.com.
* Обновите `exports.clientID`, указав идентификатор приложения, указанный при регистрации приложения.
* Обновите `exports.clientSecret`, указав секрет приложения, указанный при регистрации приложения.
* Обновите `exports.redirectUrl`, указав URI-перенаправления, указанный при регистрации приложения.

**Дополнительные настройки для рабочих приложений:**

* Если вы хотите использовать другую `post_logout_redirect_uri`, обновите `exports.destroySessionUrl` в файле config.js.

* Задайте для параметра `exports.useMongoDBSessionStore` в файле config.js значение "true", если хотите использовать [mongoDB](https://www.mongodb.com) или другие [совместимые хранилища сеансов](https://github.com/expressjs/session#compatible-session-stores).
В этом примере хранилищем сеансов по умолчанию является `express-session`. Хранилище сеансов по умолчанию не подходит для рабочей среды.

* Обновите `exports.databaseUri`, если хотите использовать хранилище сеансов mongoDB и другой URI базы данных.

* Обновите `exports.mongoDBSessionMaxAge`. Здесь можно указать срок хранения сеанса в mongoDB. Единица измерения — секунды.

## <a name="build-and-run-the-application"></a>Создание и запуск приложения

Запустите службу mongoDB. Если в этом приложении вы используете хранилище сеансов mongoDB, вам необходимо сначала [установить mongoDB](http://www.mongodb.org/) и запустить службу. При использовании хранилища сеансов по умолчанию, этот шаг можно пропустить.

Запустите приложение, выполнив в командной строке следующую команду.

```
$ node app.js
```

**Сложно понять вывод сервера?:** Для ведения журнала в этом примере используется `bunyan`. Консоль не будет понятной, если не установить "bunyan" и запустить сервер, как описано выше, но передав его через двоичный файл "bunyan":

```
$ npm install -g bunyan

$ node app.js | bunyan
```

### <a name="youre-done"></a>Готово!

Сервер успешно запущен на `http://localhost:3000`.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о сценарии веб-приложений, поддерживаемом Платформой удостоверений Майкрософт:
> [!div class="nextstepaction"]
> [Scenario: Web app that signs in users](scenario-web-app-sign-user-overview.md) (Сценарий: веб-приложение, которое входит в систему пользователей)