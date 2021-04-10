---
title: Сервер Postman FHIR в Azure — Azure API для FHIR
description: В этом руководстве мы рассмотрим шаги, необходимые для использования POST для доступа к серверу FHIR. Платформа Postman полезна для отладки приложений, имеющих доступ к API.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 03/26/2021
ms.openlocfilehash: 59847f745037acec47415489cdf61d119a7807af
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105936280"
---
# <a name="access-azure-api-for-fhir-with-postman"></a>Получение доступа к Azure API для FHIR с помощью Postman

Клиентское приложение может получить доступ к API Azure для FHIR с помощью [REST API](https://www.hl7.org/fhir/http.html). Чтобы отправлять запросы, просматривать ответы и выполнять отладку приложения по мере его построения, используйте выбранный инструмент тестирования API. В этом руководстве мы рассмотрим шаги по доступу к серверу FHIR с помощью [процедуры POST](https://www.getpostman.com/). 

## <a name="prerequisites"></a>Предварительные требования

- Конечная точка FHIR в Azure. 

  Чтобы развернуть API Azure для FHIR (управляемую службу), можно использовать [портал Azure](fhir-paas-portal-quickstart.md), [PowerShell](fhir-paas-powershell-quickstart.md)или [Azure CLI](fhir-paas-cli-quickstart.md).

- Зарегистрированное [конфиденциальное клиентское приложение](register-confidential-azure-ad-client-app.md) для доступа к службе FHIR.
- Вы предоставили разрешения для конфиденциального клиентского приложения, например "участник данных FHIR", для доступа к службе FHIR. Дополнительные сведения см. в статье [Настройка Azure RBAC для FHIR](./configure-azure-rbac.md).
- Установленное приложение Postman. 
    
  Дополнительные сведения о POST см. в статье Начало [работы с POST](https://www.getpostman.com).

## <a name="fhir-server-and-authentication-details"></a>Сведения о сервере FHIR и проверке подлинности

Чтобы использовать POST, требуются следующие параметры проверки подлинности:

- URL-адрес сервера FHIR, например `https://MYACCOUNT.azurehealthcareapis.com`

- Поставщик удостоверений `Authority` для сервера FHIR, например `https://login.microsoftonline.com/{TENANT-ID}`.

- Настроенный `audience` , который обычно является URL-адресом сервера FHIR, например, `https://<FHIR-SERVER-NAME>.azurehealthcareapis.com` или `https://azurehealthcareapis.com` .

- `client_id`Идентификатор приложения или для [конфиденциального клиентского приложения](register-confidential-azure-ad-client-app.md) , используемого для доступа к службе FHIR.

- `client_secret`Секрет приложения или конфиденциального клиентского приложения.

Наконец, следует убедиться, что `https://www.getpostman.com/oauth2/callback` является зарегистрированным URL-адресом ответа для клиентского приложения.

## <a name="connect-to-fhir-server"></a>Подключение к серверу FHIR

Откройте POST, а затем выберите **получить** , чтобы выполнить запрос `https://fhir-server-url/metadata` .

![Заявление о возможностях метаданных Postman](media/tutorial-postman/postman-metadata.png)

URL-адрес метаданных для Azure API для FHIR — `https://MYACCOUNT.azurehealthcareapis.com/metadata`. 

В этом примере URL-адрес сервера FHIR имеет значение `https://fhirdocsmsft.azurewebsites.net` , а инструкция возможностей сервера доступна в `https://fhirdocsmsft.azurewebsites.net/metadata` . Эта конечная точка доступна без проверки подлинности.

При попытке доступа к ограниченным ресурсам происходит ответ "ошибка проверки подлинности".

![Не удалось выполнить проверку подлинности](media/tutorial-postman/postman-authentication-failed.png)

## <a name="obtaining-an-access-token"></a>Получение маркера доступа
Выберите элемент **Get New Access Token** (Получить новый маркер доступа).

Чтобы получить допустимый маркер доступа, выберите **авторизация** и в раскрывающемся меню **тип** выберите **OAuth 2,0** .

![Настройка OAuth 2.0](media/tutorial-postman/postman-select-oauth2.png)

Выберите элемент **Get New Access Token** (Получить новый маркер доступа).

![Запрос нового маркера доступа](media/tutorial-postman/postman-request-token.png)

В диалоговом окне **Получение нового маркера доступа** введите следующие сведения.

| Поле                 | Пример значения                                                                                                   | Комментарий                    |
|-----------------------|-----------------------------------------------------------------------------------------------------------------|----------------------------|
| Имя токена            | MYTOKEN                                                                                                         | Выбранное имя          |
| Тип предоставления разрешения            | Код авторизации                                                                                              |                            |
| URL-адрес обратного вызова          | `https://www.getpostman.com/oauth2/callback`                                                                    |                            |
| URL-адрес аутентификации              | `https://login.microsoftonline.com/{TENANT-ID}/oauth2/authorize?resource=<audience>` | `audience` — это `https://MYACCOUNT.azurehealthcareapis.com` в Azure API для FHIR. |
| Access Token URL (URL-адрес маркера доступа)      | `https://login.microsoftonline.com/{TENANT ID}/oauth2/token`                                                    |                            |
| Идентификатор клиента             | `XXXXXXXX-XXX-XXXX-XXXX-XXXXXXXXXXXX`                                                                           | Идентификатор приложения             |
| Секрет клиента         | `XXXXXXXX`                                                                                                      | Секретный ключ клиента          |
| Область | `<Leave Blank>` | Область не используется; Поэтому его можно оставить пустым.  
| Состояние                 | `1234`     | [Состояние](https://learning.postman.com/docs/sending-requests/authorization/) — это непрозрачное значение, предотвращающее подделку межсайтовых запросов. Он является необязательным и может принимать произвольное значение, например "1234".                           |
| Аутентификация клиента | Отправьте учетные данные клиента в тексте запроса                                                                                 |                 

Выберите **маркер запроса** для последующей проверки подлинности Azure Active Directory, и маркер будет возвращен в POST. Если происходит сбой проверки подлинности, дополнительные сведения см. в разделе POST Console. **Примечание**. на ленте выберите **вид**, а затем щелкните **Показать POST Console**. Сочетание клавиш для консоли Post — **ALT + CTRL + C**.

Прокрутите вниз, чтобы просмотреть возвращенный экран токена, а затем выберите **использовать токен**.

![Использование маркера](media/tutorial-postman/postman-use-token.png)

Чтобы просмотреть только что заполненный маркер, обратитесь к полю **маркера доступа** . Если нажать кнопку **Отправить** , чтобы повторить `Patient` Поиск ресурсов, возвращается **состояние** `200 OK` . Возвращенное состояние `200 OK` указывает на Успешный HTTP-запрос.

![200 ОК](media/tutorial-postman/postman-200-OK.png)

В примере *поиска пациента* в базе данных нет пациентов, что приведет к пустому результату поиска.

Маркер доступа можно проверить с помощью такого средства, как [JWT.MS](https://jwt.ms). Пример содержимого приведен ниже.

```json
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

В ситуациях, связанных с устранением неполадок, необходимо выполнить проверку правильности аудитории (утверждение `aud`). Если ваш маркер выдан правильным издателем ( `iss` Заявка) и имеет правильную аудиторию ( `aud` Заявка), но вы по-прежнему не можете получить доступ к API FHIR, вероятно, что у пользователя или субъекта-службы нет `oid` доступа к плоскости данных FHIR. Для назначения ролей плоскости данных пользователям рекомендуется использовать [Управление доступом на основе ролей Azure (Azure RBAC)](configure-azure-rbac.md) . Если вы используете внешний, вторичный клиент Azure Active Directory для своей плоскости данных, вам потребуется [настроить локальные RBAC для](configure-local-rbac.md) назначений FHIR.

Также можно получить маркер для [API Azure для FHIR с помощью Azure CLI](get-healthcare-apis-access-token-cli.md). Если вы используете маркер, полученный с Azure CLI, следует использовать *токен носителя* типа авторизации. Вставьте маркер непосредственно.

## <a name="inserting-a-patient"></a>Вставка записи пациента

С помощью допустимого маркера доступа теперь можно вставить новое пациент. В окне Post измените метод, выбрав **POST**, а затем добавьте следующий документ JSON в текст запроса.

[!code-json[](../samples/sample-patient.json)]

Выберите **Отправить** , чтобы определить, что пациент успешно создан.

![Снимок экрана: запись пациента успешно создана.](media/tutorial-postman/postman-patient-created.png)

При повторении поиска пациента вы увидите запись о пациентах.

![Созданная запись пациента](media/tutorial-postman/postman-patient-found.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы получили доступ к API Azure для FHIR, используя POST. Дополнительные сведения о функциях API Azure для FHIR см. в разделе.
 
>[!div class="nextstepaction"]
>[Поддерживаемые функции](fhir-features-supported.md)
