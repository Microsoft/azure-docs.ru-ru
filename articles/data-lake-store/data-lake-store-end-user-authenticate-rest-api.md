---
title: Проверка подлинности конечным пользователем — остальное с Data Lake Storage 1-го поколения — Azure
description: Узнайте, как реализовать проверку подлинности пользователей в Azure Data Lake Storage 1-го поколения с помощью Azure Active Directory и REST API.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 462cd06c9da3b1f0a57c293d52c59181372b709b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92103753"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-rest-api"></a>Проверка подлинности пользователей в Azure Data Lake Storage 1-го поколения с помощью REST API
> [!div class="op_single_selector"]
> * [Использование Java](data-lake-store-end-user-authenticate-java-sdk.md)
> * [Использование пакета SDK для .NET](data-lake-store-end-user-authenticate-net-sdk.md)
> * [Использование Python](data-lake-store-end-user-authenticate-python.md)
> * [Использование REST API](data-lake-store-end-user-authenticate-rest-api.md)
> 
>  

В этой статье описывается, как использовать REST API для проверки подлинности пользователей с помощью Azure Data Lake Storage 1-го поколения. См. дополнительные сведения о [проверке подлинности между службами в Data Lake Storage 1-го поколения с помощью REST API](data-lake-store-service-to-service-authenticate-rest-api.md).

## <a name="prerequisites"></a>Предварительные условия

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Создание собственного приложения Active Directory** Вам нужно выполнить инструкции по [аутентификации пользователей в Data Lake Storage 1-го поколения с помощью Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md).

* **[перелистывание](https://curl.haxx.se/)**. В этой статье для демонстрации вызовов REST API к учетной записи Data Lake Storage 1-го поколения используется cURL.

## <a name="end-user-authentication"></a>Проверка подлинности конечных пользователей
Мы рекомендуем использовать аутентификацию пользователей, если вам нужно, чтобы пользователи входили в приложение с помощью Azure AD. Приложение будет иметь тот же уровень доступа к ресурсам Azure, что и вошедший пользователь. Пользователю нужно периодически вводить свои учетные данные, чтобы обеспечить для приложения возможность доступа.

В результате входа пользователя приложение получает маркер доступа и маркер обновления. Маркер доступа вкладывается в каждый запрос к Data Lake Storage 1-го поколения и Data Lake Analytics и по умолчанию действителен в течение одного часа. Маркер обновления может использоваться для получения нового маркера доступа. По умолчанию он действителен до двух недель, при условии, что используется регулярно. Для входа пользователей можно использовать два разных подхода.

В этом сценарии приложение предлагает пользователю войти в систему, и все операции выполняются в контексте пользователя. Выполните следующие действия:

1. В приложении перенаправьте пользователя на следующий URL-адрес.

    `https://login.microsoftonline.com/<TENANT-ID>/oauth2/authorize?client_id=<APPLICATION-ID>&response_type=code&redirect_uri=<REDIRECT-URI>`

   > [!NOTE]
   > \<REDIRECT-URI> необходимо закодировать для использования в URL-адресе. В качестве https://localhost используйте `https%3A%2F%2Flocalhost`.

    В целях обучения можно заменить значения заполнителей в URL-адресе выше и вставить его в адресную строку веб-браузера. Вы перейдете на страницу аутентификации с помощью учетной записи Azure. Войдя в систему, вы увидите ответ в адресной строке браузера. Ответ имеет следующий формат.

    `http://localhost/?code=<AUTHORIZATION-CODE>&session_state=<GUID>`

2. Запишите код авторизации из ответа. В этом руководстве вы можете скопировать код авторизации из адресной строки веб-браузера и передать его в запрос POST к конечной точке маркера, как показано в следующем фрагменте кода:

    ```console
    curl -X POST https://login.microsoftonline.com/<TENANT-ID>/oauth2/token \
    -F redirect_uri=<REDIRECT-URI> \
    -F grant_type=authorization_code \
    -F resource=https://management.core.windows.net/ \
    -F client_id=<APPLICATION-ID> \
    -F code=<AUTHORIZATION-CODE>
    ```

   > [!NOTE]
   > В этом случае \<REDIRECT-URI> не требуется кодировать.
   > 
   > 

3. Ответ является объектом JSON, который содержит маркер доступа (например, `"access_token": "<ACCESS_TOKEN>"`) и маркер обновления (например, `"refresh_token": "<REFRESH_TOKEN>"`). Приложение использует маркер доступа для обращения к Azure Data Lake Storage 1-го поколения, а маркер обновления — для получения другого маркера доступа, когда срок действия текущего маркера доступа истечет.

    ```json
    {"token_type":"Bearer","scope":"user_impersonation","expires_in":"3599","expires_on":"1461865782","not_before":    "1461861882","resource":"https://management.core.windows.net/","access_token":"<REDACTED>","refresh_token":"<REDACTED>","id_token":"<REDACTED>"}
    ```

4. Когда срок действия маркера доступа истечет, вы можете запросить новый маркер доступа, используя маркер обновления, как показано в следующем фрагменте кода:

    ```console
    curl -X POST https://login.microsoftonline.com/<TENANT-ID>/oauth2/token  \
         -F grant_type=refresh_token \
         -F resource=https://management.core.windows.net/ \
         -F client_id=<APPLICATION-ID> \
         -F refresh_token=<REFRESH-TOKEN>
    ```

Дополнительные сведения об интерактивной проверке подлинности пользователей см. в статье [Авторизация доступа к веб-приложениям с помощью OAuth 2.0 и Azure Active Directory](/previous-versions/azure/dn645542(v=azure.100)).

## <a name="next-steps"></a>Дальнейшие действия
В этой статье описывается, как использовать проверку подлинности между службами, чтобы реализовать проверку подлинности в Azure Data Lake Storage 1-го поколения с помощью REST API. Дополнительные сведения об использовании REST API для работы с Azure Data Lake Storage 1-го поколения см. в следующих статьях.

* [Операции управления учетными записями для Data Lake Storage 1-го поколения с помощью REST API](data-lake-store-get-started-rest-api.md)
* [Операции с данными в Data Lake Storage 1-го поколения c использованием REST API](data-lake-store-data-operations-rest-api.md)