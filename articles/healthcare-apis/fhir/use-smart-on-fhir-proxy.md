---
title: Прокси-сервер SMART on FHIR в Azure Active Directory
description: В этом учебнике содержатся сведения об использовании прокси-сервера для поддержки приложений SMART on FHIR с помощью Azure API для FHIR.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 04/02/2019
ms.openlocfilehash: 2e13a9fc32964781dda07e5534e5cab79868ddf0
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020734"
---
# <a name="tutorial-azure-active-directory-smart-on-fhir-proxy"></a>Руководство по настройке прокси-сервера SMART on FHIR в Azure Active Directory

[SMART on FHIR](https://docs.smarthealthit.org/) — это набор открытых спецификаций для интеграции приложений-партнеров с серверами FHIR и системами электронных медицинских карт с интерфейсами FHIR. Одной из основных целей этих спецификаций является описание обнаружения приложением конечных точек проверки подлинности для сервера FHIR и запуска последовательности проверки подлинности. 

Проверка подлинности основана на OAuth2. Но поскольку SMART on FHIR использует соглашение об именовании параметров, которое не совместимо непосредственно с Azure Active Directory (Azure AD), служба Azure API для FHIR содержит встроенный прокси-сервер SMART on FHIR в Azure AD, который включает подмножество последовательностей запуска SMART on FHIR. В частности, прокси-сервер включает [последовательность запуска EHR](https://hl7.org/fhir/smart-app-launch/#ehr-launch-sequence).

В этом учебнике описано использование прокси-сервера для включения приложений SMART on FHIR с помощью Azure API для FHIR.

## <a name="prerequisites"></a>Предварительные требования

- Экземпляр Azure API для FHIR
- [.NET Core 2.2](https://dotnet.microsoft.com/download/dotnet-core/2.2)

## <a name="configure-azure-ad-registrations"></a>Настройка регистраций Azure AD

При использовании SMART on FHIR для `Audience` требуется идентификатор URI, совпадающий с URI службы FHIR. В стандартной конфигурации Azure API для FHIR для `Audience` задано значение `https://azurehealthcareapis.com`. Однако вы можете изменить это значение на соответствующее конкретному URL-адресу вашей службы FHIR (например, `https://MYFHIRAPI.azurehealthcareapis.com`). Это необходимо при работе с прокси-сервером SMART on FHIR.

Вам также потребуется регистрация клиентского приложения. Большинство приложений SMART on FHIR представляют собой одностраничные приложения JavaScript. Поэтому вам необходимо выполнить инструкции по настройке [общедоступного клиентского приложения Azure AD](register-public-azure-ad-client-app.md).

После выполнения этих шагов у вас должен иметься:

- Сервер FHIR с аудиторией rge со значением `https://MYFHIRAPI.azurehealthcareapis.com`, где `MYFHIRAPI` — это имя экземпляра Azure API для FHIR.
- Регистрация общедоступного клиентского приложения. Запишите идентификатор клиентского приложения.

## <a name="enable-the-smart-on-fhir-proxy"></a>Включение прокси-сервера SMART on FHIR

Включите прокси-сервер SMART on FHIR в параметрах **Проверка подлинности** для своего экземпляра Azure API для FHIR, установив флажок **SMART on FHIR proxy** (Прокси-сервер SMART on FHIR):

![Выделенные параметры для включения прокси-сервера SMART on FHIR](media/tutorial-smart-on-fhir/enable-smart-on-fhir-proxy.png)

## <a name="enable-cors"></a>Включение CORS

Поскольку большинство приложений SMART on FHIR представляют собой одностраничные приложения JavaScript, необходимо [включить общий доступ к ресурсам независимо от источника (CORS)](configure-cross-origin-resource-sharing.md) для Azure API для FHIR:

![Выделенные параметры для включения CORS](media/tutorial-smart-on-fhir/enable-cors.png)

## <a name="configure-the-reply-url"></a>Настройка URL-адреса ответа

Прокси-сервер SMART on FHIR выступает в качестве посредника между приложением SMART on FHIR и Azure AD. Ответ проверки подлинности (код проверки подлинности) должен поступать на прокси-сервер SMART on FHIR, а не в само приложение. Затем прокси-сервер переадресует ответ приложению. 

Из-за такой двухэтапной ретрансляции кода проверки подлинности в качестве URL-адреса ответа (обратный вызов) для клиентского приложения Azure AD необходимо задать URL-адрес, который представляет собой комбинацию URL-адреса ответа для прокси-сервера SMART on FHIR и URL-адреса ответа для приложения SMART on FHIR. Комбинированный URL-адрес выглядит следующим образом:

```http
https://MYFHIRAPI.azurehealthcareapis.com/AadSmartOnFhirProxy/callback/aHR0cHM6Ly9sb2NhbGhvc3Q6NTAwMS9zYW1wbGVhcHAvaW5kZXguaHRtbA
```

В этом ответе `aHR0cHM6Ly9sb2NhbGhvc3Q6NTAwMS9zYW1wbGVhcHAvaW5kZXguaHRtbA` является безопасной версией URL-адреса ответа в кодировке base64 для приложения SMART on FHIR. Для средства запуска приложения SMART on FHIR, если приложение работает локально, используется следующий URL-адрес ответа: `https://localhost:5001/sampleapp/index.html`. 

Вы можете сгенерировать комбинированный URL-адрес ответа, используя такой сценарий:

```PowerShell
$replyUrl = "https://localhost:5001/sampleapp/index.html"
$fhirServerUrl = "https://MYFHIRAPI.azurewebsites.net"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($ReplyUrl)
$encodedText = [Convert]::ToBase64String($bytes)
$encodedText = $encodedText.TrimEnd('=');
$encodedText = $encodedText.Replace('/','_');
$encodedText = $encodedText.Replace('+','-');

$newReplyUrl = $FhirServerUrl.TrimEnd('/') + "/AadSmartOnFhirProxy/callback/" + $encodedText
```

Добавьте URL-адрес ответа в общедоступное клиентское приложение, созданное ранее для Azure AD:

![URL-адрес ответа, настроенный для общедоступного клиента](media/tutorial-smart-on-fhir/configure-reply-url.png)

## <a name="get-a-test-patient"></a>Тестирование с использованием данных пациента

Чтобы проверить функционирование Azure API для FHIR и прокси-сервера SMART on FHIR, в базе данных должны иметься данные хотя бы одного пациента. Если вы еще не использовали API и ваша база данных пуста, выполните [эти инструкции](access-fhir-postman-tutorial.md) по загрузке данных пациента. Запишите идентификатор конкретного пациента.

## <a name="download-the-smart-on-fhir-app-launcher"></a>Скачивание средства запуска приложений SMART on FHIR

[Сервер FHIR с открытым исходным кодом для хранилища Azure ](https://github.com/Microsoft/fhir-server) содержит простое средство запуска приложений SMART on FHIR и пример приложения SMART on FHIR. В рамках данного учебника это средство запуска SMART on FHIR используется для проверки настроек локально.

Вы можете клонировать репозиторий GitHub и перейти к приложению с помощью этих команд:

```PowerShell
git clone https://github.com/Microsoft/fhir-server
cd fhir-server/samples/apps/SmartLauncher
```

Приложению требуется несколько параметров конфигурации, которые можно установить в `appsettings.json`:

```json
{
    "FhirServerUrl": "https://MYFHIRAPI.azurehealthcareapis.com",
    "ClientId": "APP-ID",
    "DefaultSmartAppUrl": "/sampleapp/launch.html"
}
```

Мы рекомендуем воспользоваться функцией `dotnet user-secrets`:

```PowerShell
dotnet user-secrets set FhirServerUrl https://MYFHIRAPI.azurehealthcareapis.com
dotnet user-secrets set ClientId <APP-ID>
```

Используйте эту команду для запуска приложения:

```PowerShell
dotnet run
```

## <a name="test-the-smart-on-fhir-proxy"></a>Проверка прокси-сервера SMART on FHIR

После запуска средства запуска приложений SMART on FHIR в браузере можно указать `https://localhost:5001`. Откроется следующая страница:

![Средство запуска приложений SMART on FHIR](media/tutorial-smart-on-fhir/smart-on-fhir-app-launcher.png)

После заполнения полей **Patient** (Пациент), **Encounter** (Прием) или **Practitioner** (Врач-практик) содержимое раздела **Launch context** (Контекст запуска) будет обновлено. При использовании Azure API для FHIR контекст запуска представляет собой простой документ JSON, содержащий информацию о пациенте, практикующем враче и т. д. Этот контекст запуска кодируется с использованием base64 и передается в приложение SMART on FHIR в качестве параметра запроса `launch`. В соответствии со спецификацией SMART on FHIR эта переменная непрозрачна для приложения SMART on FHIR и передается поставщику удостоверений. 

Прокси-сервер SMART on FHIR использует эту информацию для заполнения полей в ответе маркера. Приложение SMART on FHIR *может* использовать эти поля, чтобы определять, для какого пациента запрашиваются данные и как приложение отображается на экране. На прокси-сервере SMART on FHIR поддерживаются следующие поля:

* `patient`
* `encounter`
* `practitioner`
* `need_patient_banner`
* `smart_style_url`

Эти поля предназначены для предоставления инструкций приложению. Они не передают никаких сведений о безопасности. Приложение SMART on FHIR может их игнорировать.

Обратите внимание, что средство запуска приложений SMART on FHIR обновляет данные поля **URL-адрес запуска** внизу страницы. Выберите **Запустить**, чтобы запустить пример приложения. После этого должно отобразиться следующее:

![Приложение SMART on FHIR](media/tutorial-smart-on-fhir/smart-on-fhir-app.png)

Проверьте ответ маркера, чтобы увидеть, как поля контекста запуска передаются приложению.

## <a name="next-steps"></a>Дальнейшие действия

Изучив этот учебник, вы настроили прокси-сервер SMART on FHIR в Azure Active Directory. Чтобы изучить использование приложений SMART on FHIR с Azure API для FHIR и FHIR Server для Azure с открытым исходным кодом, перейдите в репозиторий примеров серверов FHIR на сайте GitHub:

>[!div class="nextstepaction"]
>[Примеры серверов FHIR](https://github.com/Microsoft/fhir-server-samples)
