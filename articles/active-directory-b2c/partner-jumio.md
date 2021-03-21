---
title: Руководство по настройке Azure Active Directory B2C с помощью Жумио
titleSuffix: Azure AD B2C
description: В этом руководстве вы настраиваете Azure Active Directory B2C с помощью Жумио для автоматической проверки ИДЕНТИФИКАТОРов, защищая данные клиентов.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/20/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 66ec0d4b09dc983eb898d63d45b3dd7cab291c4c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96928670"
---
# <a name="tutorial-for-configuring-jumio-with-azure-active-directory-b2c"></a>Руководство по настройке Жумио с помощью Azure Active Directory B2C

В этом примере учебника мы предоставляем рекомендации по интеграции Azure Active Directory B2C (Azure AD B2C) с [жумио](https://www.jumio.com/). Жумио — это служба проверки идентификатора, которая обеспечивает автоматическую проверку кода в режиме реального времени для защиты данных клиентов.

## <a name="prerequisites"></a>Предварительные условия

Чтобы приступить к работе, вам потребуется:

- Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).

- [Azure AD B2C клиент](./tutorial-create-tenant.md) , связанный с подпиской Azure.

## <a name="scenario-description"></a>Описание сценария

Интеграция Жумио включает следующие компоненты:

- Azure AD B2C: сервер авторизации, отвечающий за проверку учетных данных пользователя. Он также известен как поставщик удостоверений.

- Жумио: служба, которая принимает сведения об ИДЕНТИФИКАТОРе, предоставленные пользователем, и проверяет их.

- Промежуточный API-интерфейс RESTful. API, реализующий интеграцию между Azure AD B2C и службой Жумио.

- Хранилище BLOB-объектов Azure. служба, предоставляющая пользовательские файлы пользовательского интерфейса для политик Azure AD B2C.

Реализация показана на следующей схеме архитектуры.

![Схема архитектуры Azure AD B2C интеграции с Жумио.](./media/partner-jumio/jumio-architecture-diagram.png)

|Шаг | Описание |
|:-----| :-----------|
| 1. | Пользователь поступает на страницу для входа или регистрации, чтобы создать учетную запись. Azure AD B2C собирает атрибуты пользователя.
| 2. | Azure AD B2C вызывает API среднего уровня и передает атрибуты пользователя.
| 3. | API среднего уровня собирает атрибуты пользователя и преобразует их в формат, который может использовать API Жумио. Затем он отправляет атрибуты в Жумио.
| 4. | После того как Жумио потребляет информацию и обрабатывает ее, она возвращает результат в API среднего уровня.
| 5. | API среднего уровня обрабатывает сведения и отправляет в Azure AD B2C соответствующие сведения.
| 6. | Azure AD B2C получает сведения от API среднего уровня. Если отображается ответ на сбой, пользователю выдается сообщение об ошибке. Если отображается ответ об успешном выполнении, пользователь проходит проверку подлинности и записывается в каталог.

## <a name="sign-up-with-jumio"></a>Регистрация с помощью Жумио

Чтобы создать учетную запись Жумио, обратитесь в [жумио](https://www.jumio.com/contact/).

## <a name="configure-azure-ad-b2c-with-jumio"></a>Настройка Azure AD B2C с помощью Жумио

После создания учетной записи Жумио используйте учетную запись для настройки Azure AD B2C. В следующих разделах описывается процесс последовательно.

### <a name="deploy-the-api"></a>Развертывание API

Разверните предоставленный [код API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/API/Jumio.Api) в службе Azure. Вы можете опубликовать код из Visual Studio, следуя [этим инструкциям](/visualstudio/deployment/quickstart-deploy-to-azure).

>[!NOTE]
>Вам потребуется URL-адрес развернутой службы, чтобы настроить Azure AD с использованием требуемых параметров.

### <a name="deploy-the-client-certificate"></a>Развертывание сертификата клиента

1. Сертификат клиента помогает защитить вызов API Жумио. Создайте самозаверяющий сертификат, используя следующий пример кода PowerShell:

   ``` PowerShell
   $cert = New-SelfSignedCertificate -Type Custom -Subject "CN=Demo-SigningCertificate" -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3") -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -NotAfter (Get-Date).AddYears(2) -CertStoreLocation "Cert:\CurrentUser\My"
   $cert.Thumbprint
   $pwdText = "Your password"
   $pwd = ConvertTo-SecureString -String $pwdText -Force -AsPlainText
   Export-PfxCertificate -Cert $Cert -FilePath "{your-local-path}\Demo-SigningCertificate.pfx" -Password $pwd.

   ```

   Затем сертификат экспортируется в расположение, указанное в параметре ``{your-local-path}`` .

3. Импортируйте сертификат в службу приложений Azure, выполнив инструкции, приведенные в [этой статье](../app-service/configure-ssl-certificate.md#upload-a-private-certificate).

### <a name="create-a-signingencryption-key"></a>Создание ключа подписи или шифрования

Создайте случайную строку длиной более 64 символов, которая содержит только буквы и цифры.

Пример: ``C9CB44D98642A7062A0D39B94B6CDC1E54276F2E7CFFBF44288CEE73C08A8A65``

Для создания строки используйте следующий скрипт PowerShell:

```PowerShell
-join ((0x30..0x39) + ( 0x41..0x5A) + ( 0x61..0x7A) + ( 65..90 ) | Get-Random -Count 64  | % {[char]$_})

```

### <a name="configure-the-api"></a>Настройка API

[Параметры приложения можно настроить в службе приложений Azure](../app-service/configure-common.md#configure-app-settings). С помощью этого метода можно безопасно настроить параметры, не отвлекая их в репозиторий. Необходимо предоставить следующие параметры для API-интерфейса RESTful:

| Параметры приложений | Источник | Примечания |
| :-------- | :------------| :-----------|
|Жумиосеттингс: Аусусернаме | Конфигурация учетной записи жумио |     |
|Жумиосеттингс: Ауспассворд | Конфигурация учетной записи жумио |     |
|AppSettings: Сигнингцертсумбпринт|Отпечаток созданного самозаверяющего сертификата|  |
|AppSettings: Идтокенсигнингкэй| Создание ключа подписывания с помощью PowerShell | |
| AppSettings: Идтокененкриптионкэй |Ключ шифрования, созданный с помощью PowerShell
| AppSettings: Идтокениссуер | Издатель, используемый для маркера JWT (предпочтительно значение идентификатора GUID) |
| AppSettings: Идтокенаудиенце  | Аудитория, используемая для маркера JWT (предпочтительно значение идентификатора GUID) |
|AppSettings: Басередиректурл | Базовый URL-адрес политики Azure AD B2C | HTTPS://{имя вашего клиента}. b2clogin. com/{ваш-Application-ID}|
| WEBSITE_LOAD_CERTIFICATES| Отпечаток созданного самозаверяющего сертификата |

### <a name="deploy-the-ui"></a>Развертывание пользовательского интерфейса

1. Настройте [контейнер хранилища BLOB-объектов в учетной записи хранения](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container).

2. Храните файлы пользовательского интерфейса из [папки пользовательского интерфейса](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/UI) в контейнере больших двоичных объектов.

#### <a name="update-ui-files"></a>Обновление файлов пользовательского интерфейса

1. В файлах пользовательского интерфейса перейдите в папку [ocean_blue](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/UI/ocean_blue).

2. Откройте каждый HTML-файл.

3. Найдите и замените `{your-ui-blob-container-url}` URL-адресом контейнера больших двоичных объектов.

4. Найдите и замените `{your-intermediate-api-url}` URL-адресом промежуточной службы приложений API.

>[!NOTE]
> Рекомендуется добавить уведомление о согласии на странице Коллекция атрибутов. Уведомлять пользователей о том, что данные будут отправляться сторонним службам для проверки личности.

### <a name="configure-the-azure-ad-b2c-policy"></a>Настройка политики Azure AD B2C

1. Перейдите к [политике Azure AD B2C](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/Policies) в папке политики.

2. Чтобы скачать [LocalAccounts Starter Pack](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts), выполните инструкции из [этой статьи](./custom-policy-get-started.md?tabs=applications#custom-policy-starter-pack) .

3. Настройте политику для клиента Azure AD B2C.

>[!NOTE]
>Обновите указанные политики, чтобы они были связаны с конкретным клиентом.

## <a name="test-the-user-flow"></a>Тестирование потока пользователя

1. Откройте клиент Azure AD B2C. В разделе **Политики** выберите **Identity Experience Framework**.

2. Выберите ранее созданный **сигнупсигнин**.

3. Выберите **выполнить поток пользователя** , а затем:

   а. Для **приложения** выберите зарегистрированное приложение (пример — JWT).

   b. В поле **URL-адрес ответа** выберите **URL-адрес перенаправления**.

   c. Выберите **Выполнить поток пользователя**.

4. Пройдите процесс регистрации и создайте учетную запись.

5. Служба Жумио будет вызываться во время последовательности после создания пользовательского атрибута. Если поток неполон, убедитесь, что пользователь не сохранен в каталоге.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- [Пользовательские политики в Azure AD B2C](./custom-policy-overview.md)

- [Приступая к работе с пользовательскими политиками в Azure AD B2C](./custom-policy-get-started.md?tabs=applications)
