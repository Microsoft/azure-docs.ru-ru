---
title: Учебник по API отчетов Azure AD с сертификатами | Документация Майкрософт
description: В этом руководстве описывается использования API отчетов Azure AD с учетными данными сертификатов для получения данных из каталогов без вмешательства пользователя.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: c4f4ac6ab3825c82ac0cb8ef5c31f9396ef7b41f
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533732"
---
# <a name="tutorial-get-data-using-the-azure-active-directory-reporting-api-with-certificates"></a>Руководство по Получение данных с помощью API отчетов Azure Active Directory с сертификатами

[API-интерфейсы отчетов Azure Active Directory (Azure AD)](concept-reporting-api.md) предоставляют программный доступ к данным с помощью набора API-интерфейсов на базе REST. Эти интерфейсы API можно вызвать, используя различные языки и инструменты программирования. Чтобы получить доступ к API отчетов Azure AD без вмешательства пользователя, необходимо настроить доступ с помощью сертификатов.

Из этого руководства вы узнаете, как создать тестовый сертификат и использовать его для доступа к API Microsoft Graph для создания отчетов. Мы не рекомендуем использовать тестовые сертификаты в рабочей среде. 

## <a name="prerequisites"></a>Предварительные требования

1. Чтобы получить доступ к данным о входе в систему, вам потребуется клиент Azure Active Directory с лицензией Premium (P1 или P2). Чтобы обновить выпуск Azure Active Directory, ознакомьтесь со статьей [Регистрация для работы с выпусками Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md). Обратите внимание, если до обновления данные о действиях отсутствовали, данные отобразятся в отчетах через несколько дней после обновления до лицензии Premium. 

2. Создайте или перейдите в учетную запись с ролью **глобального администратора**, **администратора безопасности**, **читателя сведений о безопасности** или **читателя отчетов** для клиента. 

3. Выполните [предварительные требования для доступа к API отчетов Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md). 

4. Скачайте и установите [Azure AD PowerShell V2](https://github.com/Azure/azure-docs-powershell-azuread/blob/master/docs-conceptual/azureadps-2.0/install-adv2.md).

5. Установите [MSCloudIdUtils](https://www.powershellgallery.com/packages/MSCloudIdUtils/). Этот модуль предоставляет несколько служебных командлетов, позволяющих получить:
    - Библиотеки ADAL, необходимые для проверки подлинности
    - маркеры доступа пользователя, ключи приложений и сертификаты с использованием ADAL;
    - обработку результатов с разбивкой на страницы с помощью API Graph.

6. Если вы используете модуль впервые, выполните командлет **Install-MSCloudIdUtilsModule** или же импортируйте его с помощью команды PowerShell **Import-Module**. Сеанс должен выглядеть так, как показано ниже. ![Windows PowerShell](./media/tutorial-access-api-with-certificates/module-install.png)
  
7. Используйте командлет Powershell **New-SelfSignedCertificate** для создания тестового сертификата.

   ```
   $cert = New-SelfSignedCertificate -Subject "CN=MSGraph_ReportingAPI" -CertStoreLocation "Cert:\CurrentUser\My" -KeyExportPolicy Exportable -KeySpec Signature -KeyLength 2048 -KeyAlgorithm RSA -HashAlgorithm SHA256
   ```

8. Используйте командлет **Export-Certificate**, чтобы экспортировать его в файл сертификата.

   ```
   Export-Certificate -Cert $cert -FilePath "C:\Reporting\MSGraph_ReportingAPI.cer"

   ```

## <a name="get-data-using-the-azure-active-directory-reporting-api-with-certificates"></a>Получение данных с помощью API отчетов Azure Active Directory с сертификатами

1. Перейдите на [портал Azure](https://portal.azure.com), выберите **Azure Active Directory**, затем выберите **Регистрация приложений** и ваше приложение из списка. 

2. Выберите **Сертификаты и секреты** в разделе **Управление** колонки "Регистрация приложения" и выберите **Отправить сертификат**.

3. Выберите файл сертификата из предыдущего шага и нажмите **Добавить**. 

4. Запишите идентификатор приложения и отпечаток сертификата, который вы только что зарегистрировали для приложения. Чтобы найти отпечаток, на странице приложения на портале выберите **Сертификаты и секреты** в разделе **Управление**. Отпечаток будет указан в списке **Сертификаты**.

5. Откройте манифест приложения во встроенном редакторе манифеста и убедитесь, что в свойстве *keyCredentials* указаны данные нового сертификата, как показано ниже. 

   ```
   "keyCredentials": [
        {
            "customKeyIdentifier": "$base64Thumbprint", //base64 encoding of the certificate hash
            "keyId": "$keyid", //GUID to identify the key in the manifest
            "type": "AsymmetricX509Cert",
            "usage": "Verify",
            "value":  "$base64Value" //base64 encoding of the certificate raw data
        }
    ]
   ``` 
6. Теперь вы можете получить маркер доступа для API Microsoft Graph, используя этот сертификат. Используйте командлет **Get MSCloudIdMSGraphAccessTokenFromCert** из PowerShell-модуля MSCloudIdUtils, передав идентификатор приложения и отпечаток, полученный на предыдущем шаге. 

   ![Снимок экрана: окно PowerShell с командой, которая создает маркер доступа.](./media/tutorial-access-api-with-certificates/getaccesstoken.png)

7. Используйте маркер доступа в скрипте PowerShell для отправки запроса в API Graph. Используйте командлет **Invoke-MSCloudIdMSGraphQuery** из модуля MSCloudIDUtils для перечисления операций входа и запроса конечной точки diectoryAudits. Этот командлет обрабатывает результаты с разбивкой на несколько страниц и отправляет их в конвейер PowerShell.

8. Запросите конечную точку directoryAudits для получения журналов аудита. 

   ![Снимок экрана: окно PowerShell с командой для отправки запроса к конечной точке directoryAudits с использованием полученного ранее маркера доступа.](./media/tutorial-access-api-with-certificates/query-directoryAudits.png)

9. Запросите конечную точку signins для получения журналов входа в систему.

    ![Снимок экрана: окно PowerShell с командой для отправки запроса к конечной точке signins с использованием полученного ранее маркера доступа.](./media/tutorial-access-api-with-certificates/query-signins.png)

10. Теперь можно экспортировать эти данные в CSV-файл и сохранить его в системе SIEM. Также можно перенести скрипт в запланированную задачу, чтобы периодически получать данные Azure AD из клиента без необходимости сохранять ключи приложений в исходном коде. 

## <a name="next-steps"></a>Дальнейшие действия

* [Ознакомление с API отчетов](concept-reporting-api.md)
* [Справочник по API аудита](/graph/api/resources/directoryaudit) 
* [Справочник по API отчетов о действиях при входе](/graph/api/resources/signin)