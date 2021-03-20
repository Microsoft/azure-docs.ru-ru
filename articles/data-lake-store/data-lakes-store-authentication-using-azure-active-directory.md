---
title: Проверка подлинности — Data Lake Storage 1-го поколения с Azure AD
description: Узнайте, как выполнять аутентификацию с помощью Azure Data Lake Storage 1-го поколения Azure Active Directory.
author: twooley
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 6fc09f9145b7a1652b621ed38a8bf9af7c4c82a8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92106575"
---
# <a name="authentication-with-azure-data-lake-storage-gen1-using-azure-active-directory"></a>Аутентификация в ADLS 1-го поколения с помощью Azure Active Directory

Azure Data Lake Storage 1-го поколения (ADLS 1-го поколения) использует Azure Active Directory для аутентификации. Прежде чем создать приложение, которое работает с ADLS 1-го поколения, необходимо решить, как для него выполнять аутентификацию в Azure Active Directory (Azure AD).

## <a name="authentication-options"></a>Параметры проверки подлинности

* **Аутентификация пользователей**. Для аутентификации в ADLS 1-го поколения используются учетные данные пользователя Azure. Приложение, создаваемое для работы с ADLS 1-го поколения, запрашивает эти учетные данные пользователя. В результате такой механизм аутентификации является *интерактивным*, и приложение запускается в контексте пользователя, вошедшего в систему. Дополнительные сведения и инструкции приведены в статье [Аутентификация пользователей в Data Lake Store с помощью Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md).

* **Аутентификация между службами**. Используйте этот вариант, если требуется, чтобы приложение проходило аутентификацию в ADLS 1-го поколения. В таких случаях можно создать приложение Azure Active Directory (AD) и использовать ключ из приложения Azure AD для аутентификации в ADLS 1-го поколения. Такой механизм аутентификации будет *неинтерактивным*. Дополнительные сведения и инструкции приведены в статье [Аутентификация между службами в Data Lake Store с помощью Azure Active Directory](data-lake-store-service-to-service-authenticate-using-active-directory.md).

В следующей таблице показано, как механизмы аутентификации пользователей и служб поддерживаются для ADLS 1-го поколения. Ниже приведены инструкции к этой таблице:

* символ ✔* означает, что этот вариант аутентификации поддерживается, и можно перейти по ссылке на статью, в которой показано, как его использовать; 
* символ ✔ означает, что этот вариант аутентификации поддерживается; 
* пустые ячейки означают, что этот вариант аутентификации не поддерживается.


|Использование варианта аутентификации для…                   |.NET         |Java     |PowerShell |Azure CLI | Python   |REST     |
|:---------------------------------------------|:------------|:--------|:----------|:-------------|:---------|:--------|
|Аутентификация пользователей (без MFA**)                        |   ✔ |    ✔    |    ✔      |       ✔      |    **[✔ *](data-lake-store-end-user-authenticate-python.md#end-user-authentication-without-multi-factor-authentication)**(не рекомендуется)     |    **[✔ *](data-lake-store-end-user-authenticate-rest-api.md)**    |
|Аутентификация пользователей (с MFA)                           |    **[✔ *](data-lake-store-end-user-authenticate-net-sdk.md)**        |    **[✔ *](data-lake-store-end-user-authenticate-java-sdk.md)**     |    ✔      |       **[✔ *](data-lake-store-get-started-cli-2.0.md)**      |    **[✔ *](data-lake-store-end-user-authenticate-python.md#end-user-authentication-with-multi-factor-authentication)**     |    ✔    |
|Аутентификация между службами (с помощью ключа клиента)         |    **[✔ *](data-lake-store-service-to-service-authenticate-net-sdk.md#service-to-service-authentication-with-client-secret)** |    **[✔ *](data-lake-store-service-to-service-authenticate-java.md)**    |    ✔      |       ✔      |    **[✔ *](data-lake-store-service-to-service-authenticate-python.md#service-to-service-authentication-with-client-secret-for-account-management)**     |    **[✔ *](data-lake-store-service-to-service-authenticate-rest-api.md)**    |
|Аутентификация между службами (с помощью сертификата клиента) |    **[✔ *](data-lake-store-service-to-service-authenticate-net-sdk.md#service-to-service-authentication-with-certificate)**        |    ✔    |    ✔      |       ✔      |    ✔     |    ✔    |

<i>* Щелкните символ <b>✔ \* </b> . Это ссылка.</i><br>
<i>** MFA — многофакторная идентификация (Multi-Factor Authentication).</i>

Дополнительные сведения об использовании Azure Active Directory для аутентификации см. в статье [Сценарии аутентификации в Azure Active Directory](../active-directory/develop/authentication-vs-authorization.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Проверка подлинности конечных пользователей](data-lake-store-end-user-authenticate-using-active-directory.md)
* [Аутентификация между службами](data-lake-store-service-to-service-authenticate-using-active-directory.md)