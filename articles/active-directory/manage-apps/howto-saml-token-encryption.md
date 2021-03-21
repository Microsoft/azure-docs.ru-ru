---
title: Шифрование токенов SAML в Azure Active Directory
description: Узнайте, как настроить шифрование токенов SAML в Azure Active Directory.
services: active-directory
documentationcenter: ''
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: kenwith
ms.reviewer: paulgarn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 32a6034f407bd81e5d2d2ab8227d08a67f034d4c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99258648"
---
# <a name="how-to-configure-azure-ad-saml-token-encryption"></a>Как настроить шифрование маркеров SAML в Azure AD

> [!NOTE]
> Шифрование токенов — это функция уровня "Премиум" в Azure Active Directory (Azure AD). Дополнительные сведения о выпусках, функциях и ценах на Azure AD см. на странице [цен на Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

Шифрование токенов SAML позволяет использовать зашифрованные утверждения SAML в приложении, которое его поддерживает. Если эта функция настроена в приложении, служба Azure AD шифрует утверждения SAML, которые она выдает для этого приложения, с помощью открытого ключа, полученного из сертификата, хранящегося в этой службе. Приложение должно использовать соответствующий закрытый ключ для расшифровки токена, прежде чем токен можно будет применять в качестве доказательства проверки подлинности для пользователя, выполнившего вход.

Шифрование утверждений SAML между Azure AD и приложением обеспечивает дополнительную уверенность в том, что содержимое токена не может быть перехвачено, а личные или корпоративные данные не будут скомпрометированы.

Даже без шифрования токены SAML Azure AD никогда не передаются в сеть в открытом виде. Azure AD требует обмена запросами и ответами на токены по зашифрованным каналам HTTPS/TLS, чтобы связь между поставщиком удостоверений, браузером и приложением осуществлялась по зашифрованным подключениям. Рассмотрите ценность шифрования токенов в вашем сценарии по сравнению с затратами на управление дополнительными сертификатами.   

Чтобы настроить шифрование токенов, необходимо отправить файл сертификата X.509, содержащий открытый ключ, в объект, представляющий приложение Azure AD. Вы можете загрузить сертификат X.509 из самого приложения или получить его у поставщика приложения в тех случаях, когда последний предоставляет ключи шифрования. Если приложение ожидает, что вы предоставите закрытый ключ, вы можете создать ключ с помощью средств шифрования, части закрытого ключа, переданной в хранилище ключей приложения, и соответствующего сертификата открытого ключа, отправленного в Azure AD.

Чтобы зашифровать данные утверждения SAML, в Azure AD используется AES-256.

## <a name="configure-saml-token-encryption"></a>Настройка шифрования токенов SAML

Чтобы настроить шифрование токенов SAML, выполните следующие действия.

1. Получите сертификат открытого ключа, соответствующий закрытому ключу, настроенному в приложении.

    Создайте пару асимметричных ключей для шифрования. Если приложение предоставляет открытый ключ для шифрования, следуйте инструкциям приложения, чтобы загрузить сертификат X.509.

    Открытый ключ должен храниться в формате CER сертификата X.509.

    Если приложение использует ключ, который вы создали для своего экземпляра, следуйте инструкциям в приложении, чтобы установить закрытый ключ, который будет применяться приложением для расшифровки токенов из вашего клиента Azure AD.

1. Добавьте сертификат в конфигурацию приложения в Azure AD.

### <a name="to-configure-token-encryption-in-the-azure-portal"></a>Настройка шифрования токенов на портале Azure

Вы можете добавить открытый сертификат в конфигурацию вашего приложения на портале Azure.

1. Перейдите на [портал Microsoft Azure](https://portal.azure.com).

1. Перейдите в колонку **Azure Active Directory > Корпоративные приложения** и выберите приложение, для которого необходимо настроить шифрование токенов.

1. На странице приложения выберите **Шифрование токенов**.

    ![Параметр шифрования токенов на портале Azure](./media/howto-saml-token-encryption/token-encryption-option-small.png)

    > [!NOTE]
    > Параметр **Шифрование токенов** доступен только для приложений SAML, настроенных в колонке **Корпоративные приложения** на портале Azure (из коллекции приложений или из приложения, не входящего в коллекцию). Для других приложений этот пункт меню отключен. Для приложений, зарегистрированных с помощью функции **Регистрация приложений** на портале Azure, можно настроить шифрование токенов SAML с использованием манифеста приложения, Microsoft Graph или PowerShell.

1. На странице **Шифрование токенов** выберите **импортировать сертификат** , чтобы импортировать CER-файл, содержащий общедоступный сертификат X. 509.

    ![Импорт CER-файла, содержащего сертификат X.509](./media/howto-saml-token-encryption/import-certificate-small.png)

1. Как только сертификат будет импортирован, а закрытый ключ настроен для использования на стороне приложения, активируйте шифрование, выбрав **...** рядом с состоянием отпечатка, а затем щелкните **Активировать сертификат шифрования токенов** в раскрывающемся меню.

1. Щелкните **Да**, чтобы подтвердить активацию сертификата шифрования токенов.

1. Убедитесь, что утверждения SAML, выдаваемые для приложения, зашифрованы.

### <a name="to-deactivate-token-encryption-in-the-azure-portal"></a>Отключение шифрования токенов на портале Azure

1. На портале Azure перейдите в колонку **Azure Active Directory > Корпоративные приложения** и выберите приложение, в котором включено шифрование токенов SAML.

1. На странице приложения щелкните **Шифрование токенов**, найдите сертификат, а затем щелкните **...**, чтобы открыть раскрывающееся меню.

1. Щелкните **Deactivate token encryption** (Деактивировать шифрование токенов).

## <a name="configure-saml-token-encryption-using-graph-api-powershell-or-app-manifest"></a>Настройка шифрования токенов SAML с помощью Graph API, PowerShell или манифеста приложения

Сертификаты шифрования хранятся в объекте приложения в Azure AD с тегом использования `encrypt`. Можно настроить несколько сертификатов шифрования, а тот, который активен для шифрования токенов, идентифицируется с помощью атрибута `tokenEncryptionKeyID`.

Для настройки шифрования токенов с помощью API Microsoft Graph или PowerShell потребуется идентификатор объекта приложения. Это значение можно найти программным способом или на странице **Свойства** приложения на портале Azure, указав значение **ИД объекта**.

При настройке keyCredential с помощью Graph, PowerShell или в манифесте приложения следует создать идентификатор GUID для использования в качестве keyId.

### <a name="to-configure-token-encryption-using-microsoft-graph"></a>Настройка шифрования токенов с помощью Microsoft Graph

1. Обновите свойство `keyCredentials` приложения, указав сертификат X.509 для шифрования. В приведенном ниже примере показано, как это сделать.

    ```
    Patch https://graph.microsoft.com/beta/applications/<application objectid>

    { 
       "keyCredentials":[ 
          { 
             "type":"AsymmetricX509Cert","usage":"Encrypt",
             "keyId":"fdf8c5d8-f727-43fd-beaf-0f1521cf3d35",    (Use a GUID generator to obtain a value for the keyId)
             "key": "MIICADCCAW2gAwIBAgIQ5j9/b+n2Q4pDvQUCcy3…"  (Base64Encoded .cer file)
          }
        ]
    }
    ```

1. Укажите сертификат шифрования, который будет активным для шифрования токенов. В приведенном ниже примере показано, как это сделать.

    ```
    Patch https://graph.microsoft.com/beta/applications/<application objectid> 

    { 
       "tokenEncryptionKeyId":"fdf8c5d8-f727-43fd-beaf-0f1521cf3d35" (The keyId of the keyCredentials entry to use)
    }
    ```

### <a name="to-configure-token-encryption-using-powershell"></a>Настройка шифрования токенов с помощью PowerShell

1. Используйте последнюю версию модуля Azure AD PowerShell для подключения к вашему клиенту.

1. Задайте параметры шифрования маркеров с помощью команды **[Set-азуреаппликатион](/powershell/module/azuread/set-azureadapplication?view=azureadps-2.0-preview)** .

    ```
    Set-AzureADApplication -ObjectId <ApplicationObjectId> -KeyCredentials "<KeyCredentialsObject>"  -TokenEncryptionKeyId <keyID>
    ```

1. Прочтите параметры шифрования маркеров с помощью следующих команд.

    ```powershell
    $app=Get-AzureADApplication -ObjectId <ApplicationObjectId>
    $app.KeyCredentials
    $app.TokenEncryptionKeyId
    ```

### <a name="to-configure-token-encryption-using-the-application-manifest"></a>Настройка шифрования токенов с использованием манифеста приложения

1. На портале Azure перейдите к **Azure Active Directory > Регистрация приложений**.

1. Щелкните **Все приложения** в раскрывающемся списке, чтобы отобразить все приложения, а затем выберите корпоративное приложение, которое требуется настроить.

1. На странице приложения выберите **Манифест**, чтобы изменить [манифест приложения](../develop/reference-app-manifest.md).

1. Задайте значение для атрибута `tokenEncryptionKeyId`.

    В следующем примере показан манифест приложения с двумя сертификатами шифрования, второй из которых выбран в качестве активного с помощью tokenEnryptionKeyId.

    ```json
    { 
      "id": "3cca40e2-367e-45a5-8440-ed94edd6cc35",
      "accessTokenAcceptedVersion": null,
      "allowPublicClient": false,
      "appId": "cb2df8fb-63c4-4c35-bba5-3d659dd81bf1",
      "appRoles": [],
      "oauth2AllowUrlPathMatching": false,
      "createdDateTime": "2017-12-15T02:10:56Z",
      "groupMembershipClaims": "SecurityGroup",
      "informationalUrls": { 
         "termsOfService": null, 
         "support": null, 
         "privacy": null, 
         "marketing": null 
      },
      "identifierUris": [ 
        "https://testapp"
      ],
      "keyCredentials": [ 
        { 
          "customKeyIdentifier": "Tog/O1Hv1LtdsbPU5nPphbMduD=", 
          "endDate": "2039-12-31T23:59:59Z", 
          "keyId": "8be4cb65-59d9-404a-a6f5-3d3fb4030351", 
          "startDate": "2018-10-25T21:42:18Z", 
          "type": "AsymmetricX509Cert", 
          "usage": "Encrypt", 
          "value": <Base64EncodedKeyFile> 
          "displayName": "CN=SAMLEncryptTest" 
        }, 
        {
          "customKeyIdentifier": "U5nPphbMduDmr3c9Q3p0msqp6eEI=",
          "endDate": "2039-12-31T23:59:59Z", 
          "keyId": "6b9c6e80-d251-43f3-9910-9f1f0be2e851",
          "startDate": "2018-10-25T21:42:18Z", 
          "type": "AsymmetricX509Cert", 
          "usage": "Encrypt", 
          "value": <Base64EncodedKeyFile> 
          "displayName": "CN=SAMLEncryptTest2" 
        } 
      ], 
      "knownClientApplications": [], 
      "logoUrl": null, 
      "logoutUrl": null, 
      "name": "Test SAML Application", 
      "oauth2AllowIdTokenImplicitFlow": true, 
      "oauth2AllowImplicitFlow": false, 
      "oauth2Permissions": [], 
      "oauth2RequirePostResponse": false, 
      "orgRestrictions": [], 
      "parentalControlSettings": { 
         "countriesBlockedForMinors": [], 
         "legalAgeGroupRule": "Allow" 
        }, 
      "passwordCredentials": [], 
      "preAuthorizedApplications": [], 
      "publisherDomain": null, 
      "replyUrlsWithType": [], 
      "requiredResourceAccess": [], 
      "samlMetadataUrl": null, 
      "signInUrl": "https://127.0.0.1:444/applications/default.aspx?metadata=customappsso|ISV9.1|primary|z" 
      "signInAudience": "AzureADMyOrg",
      "tags": [], 
      "tokenEncryptionKeyId": "6b9c6e80-d251-43f3-9910-9f1f0be2e851" 
    }  
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Как в Azure AD используется протокол SAML](../develop/active-directory-saml-protocol-reference.md)
* Узнайте о формате, характеристиках безопасности и содержимом в статье [Справочник по токенам SAML в Azure AD](../develop/reference-saml-tokens.md).