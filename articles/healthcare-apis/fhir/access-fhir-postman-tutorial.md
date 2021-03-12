---
title: Сервер Postman FHIR в Azure — Azure API для FHIR
description: В этом учебнике описаны шаги, необходимые для использования Postman для доступа к серверу FHIR. Платформа Postman полезна для отладки приложений, имеющих доступ к API.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 02/01/2021
ms.openlocfilehash: aa0b18b701c573d4b2542359cb45b2d7694e78bd
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103019532"
---
# <a name="access-azure-api-for-fhir-with-postman"></a>Получение доступа к Azure API для FHIR с помощью Postman

Клиентское приложение будет получать доступ к API FHIR через [REST API](https://www.hl7.org/fhir/http.html). Возможно вам нужно будет взаимодействовать с сервером FHIR при создании приложений напрямую, например в целях отладки. В этом учебнике описаны шаги, необходимые для использования [Postman](https://www.getpostman.com/) для получения доступа к серверу FHIR. Postman — это средство, часто используемое для отладки при создании приложений, имеющих доступ к API.

## <a name="prerequisites"></a>Предварительные требования

- Конечная точка FHIR в Azure. Ее можно настроить с помощью управляемой службы Azure API для FHIR или сервера FHIR с открытым исходным кодом для Azure. Настройте управляемую службу Azure API для FHIR с помощью [портала Azure](fhir-paas-portal-quickstart.md), [PowerShell](fhir-paas-powershell-quickstart.md) или [Azure CLI](fhir-paas-cli-quickstart.md).
- [Клиентское приложение](register-confidential-azure-ad-client-app.md), которое вы будете использовать для доступа к службе FHIR.
- Вы должны предоставить клиентскому приложению разрешения (например, участника для данных FHIR) для доступа к службе FHIR. Дополнительные сведения см. в статье [Настройка Azure RBAC для FHIR](configure-azure-rbac.md).
- Установленное приложение Postman. Его можно получить на сайте [https://www.getpostman.com](https://www.getpostman.com).

## <a name="fhir-server-and-authentication-details"></a>Сведения о сервере FHIR и проверке подлинности

Чтобы использовать Postman, необходимы следующие сведения:

- URL-адрес сервера FHIR, например `https://MYACCOUNT.azurehealthcareapis.com`.
- Поставщик удостоверений `Authority` для сервера FHIR, например `https://login.microsoftonline.com/{TENANT-ID}`.
- Необходимо настроить `audience`. Обычно это URL-адрес сервера FHIR, например `https://<FHIR-SERVER-NAME>.azurehealthcareapis.com` или просто `https://azurehealthcareapis.com`.
- `client_id` (или идентификатор приложения) [клиентского приложения](register-confidential-azure-ad-client-app.md), которое будет использоваться для доступа к службе FHIR.
- `client_secret` (или секрет приложения) клиентского приложения.

Наконец, следует убедиться, что `https://www.getpostman.com/oauth2/callback` является зарегистрированным URL-адресом ответа для клиентского приложения.

## <a name="connect-to-fhir-server"></a>Подключение к серверу FHIR

Используя Postman, выполните запрос `GET` для `https://fhir-server-url/metadata`:

![Заявление о возможностях метаданных Postman](media/tutorial-postman/postman-metadata.png)

URL-адрес метаданных для Azure API для FHIR — `https://MYACCOUNT.azurehealthcareapis.com/metadata`. В этом примере URL-адрес сервера FHIR — `https://fhirdocsmsft.azurewebsites.net`. Заявление о возможностях сервера доступно по адресу `https://fhirdocsmsft.azurewebsites.net/metadata`. Эта конечная точка должна быть доступна без проверки подлинности.

При попытке доступа к ограниченным ресурсам вы должны получить ответ "Не удалось выполнить проверку подлинности".

![Не удалось выполнить проверку подлинности](media/tutorial-postman/postman-authentication-failed.png)

## <a name="obtaining-an-access-token"></a>Получение маркера доступа

Чтобы получить допустимый маркер доступа, щелкните "Авторизация" и выберите тип OAuth 2.0:

![Настройка OAuth 2.0](media/tutorial-postman/postman-select-oauth2.png)

Нажмите кнопку Get New Access Token (Получить новый маркер доступа), после чего откроется диалоговое окно:

![Запрос нового маркера доступа](media/tutorial-postman/postman-request-token.png)

Вам понадобятся некоторые сведения:

| Поле                 | Пример значения                                                                                                   | Комментарий                    |
|-----------------------|-----------------------------------------------------------------------------------------------------------------|----------------------------|
| Имя токена            | MYTOKEN                                                                                                         | Выбранное имя          |
| Тип предоставления разрешения            | Код авторизации                                                                                              |                            |
| URL-адрес обратного вызова          | `https://www.getpostman.com/oauth2/callback`                                                                      |                            |
| URL-адрес аутентификации              | `https://login.microsoftonline.com/{TENANT-ID}/oauth2/authorize?resource=<audience>` | `audience` — это `https://MYACCOUNT.azurehealthcareapis.com` в Azure API для FHIR. |
| Access Token URL (URL-адрес маркера доступа)      | `https://login.microsoftonline.com/{TENANT ID}/oauth2/token`                                                      |                            |
| Идентификатор клиента             | `XXXXXXXX-XXX-XXXX-XXXX-XXXXXXXXXXXX`                                                                            | Идентификатор приложения             |
| Секрет клиента         | `XXXXXXXX`                                                                                                        | Секретный ключ клиента          |
| Область | `<Leave Blank>` |
| Состояние                 | `1234`                                                                                                            |                            |
| Аутентификация клиента | Отправьте учетные данные клиента в тексте запроса                                                                                 |                 

Нажмите кнопку Request Token (Запросить маркер) и вы пройдете через поток проверки подлинности Azure Active Directory, а маркер будет возвращен Postman. При возникновении проблем откройте консоль Postman (в пункте меню View->Show Postman Console (Представление > Показать консоль Postman)).

Прокрутите возвращенный экран маркера вниз и нажмите кнопку Use Token (Использовать маркер):

![Использование маркера](media/tutorial-postman/postman-use-token.png)

Теперь этот маркер будет помещен в поле "Маркер доступа", а маркеры можно выбрать в разделе Available Tokens (Доступные маркеры). Если вы снова отправите повторный запрос поиска ресурса `Patient`, то получите состояние `200 OK`:

![200 ОК](media/tutorial-postman/postman-200-OK.png)

Это означает, что в базе данных нет зарегистрированных пациентов и результат поиска пуст.

Если вы просматриваете маркер доступа с помощью такого средства как [https://jwt.ms](https://jwt.ms), должно отображаться примерно следующее:

```jsonc
{
  "aud": "https://MYACCOUNT.azurehealthcareapis.com",
  "iss": "https://sts.windows.net/{TENANT-ID}/",
  "iat": 1545343803,
  "nbf": 1545343803,
  "exp": 1545347703,
  "acr": "1",
  "aio": "AUQAu/8JXXXXXXXXXdQxcxn1eis459j70Kf9DwcUjlKY3I2G/9aOnSbw==",
  "amr": [
    "pwd"
  ],
  "appid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "oid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "appidacr": "1",

  ...// Truncated
}
```

В ситуациях, связанных с устранением неполадок, необходимо выполнить проверку правильности аудитории (утверждение `aud`). Если ваш маркер относится к надлежащему поставщику (утверждение `iss`) и имеет правильную аудиторию (утверждение `aud`), но вы по-прежнему не можете получить доступ к API FHIR, вероятно, что у пользователя или субъекта-службы (утверждение `oid`) нет доступа к плоскости данных FHIR. Для назначения ролей плоскости данных пользователям рекомендуется [использовать управление доступом на основе ролей Azure (Azure RBAC)](configure-azure-rbac.md). Если вы используете внешний, вторичный клиент Azure Active Directory для своей плоскости данных, необходимо [настроить локальные назначения RBAC](configure-local-rbac.md).

Кроме того, можно [получить маркер для Azure API для FHIR с помощью Azure CLI](get-healthcare-apis-access-token-cli.md). Если вы используете маркер, полученный с помощью Azure CLI, следует использовать тип авторизации "Маркер носителя" и вставить маркер непосредственно.

## <a name="inserting-a-patient"></a>Вставка записи пациента

Теперь у вас есть допустимый маркер доступа. Вы можете вставить запись нового пациента. Переключитесь на метод POST и добавьте следующий документ JSON в текст запроса:

[!code-json[](../samples/sample-patient.json)]

Нажмите кнопку "Отправить" и вы увидите, что запись пациента успешно создана:

![Снимок экрана: запись пациента успешно создана.](media/tutorial-postman/postman-patient-created.png)

Эта запись пациента отобразится при повторении поиска пациента:

![Созданная запись пациента](media/tutorial-postman/postman-patient-found.png)

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого учебника вы получили доступ к API FHIR с помощью postman. Ознакомьтесь с поддерживаемыми функциями API на следующей странице:
 
>[!div class="nextstepaction"]
>[Поддерживаемые функции](fhir-features-supported.md)