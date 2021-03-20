---
title: Файлы SharePoint — QnA Maker
description: Добавьте защищенные источники данных SharePoint в базу знаний, чтобы расширить базу знаний с вопросами и ответами, которые могут быть защищены с помощью Active Directory.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: 0832b54e02cabecb0b1f0e7af600b8adc621a8b0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99584776"
---
# <a name="add-a-secured-sharepoint-data-source-to-your-knowledge-base"></a>Добавление защищенного источника данных SharePoint в базу знаний

Добавьте защищенные облачные источники данных SharePoint в базу знаний, чтобы расширить базу знаний с вопросами и ответами, которые могут быть защищены с помощью Active Directory.

При добавлении защищенного документа SharePoint в базу знаний в качестве диспетчера QnA Maker необходимо запросить разрешение Active Directory для QnA Maker. После того как это разрешение предоставляется диспетчеру Active Directory для QnA Maker доступа к SharePoint, его не нужно предоставлять повторно. Каждое последующее добавление документа к базе знаний не требует авторизации, если оно находится в том же ресурсе SharePoint.

Если QnA Maker диспетчер базы знаний не является диспетчером Active Directory, необходимо установить связь с диспетчером Active Directory, чтобы завершить этот процесс.

## <a name="prerequisites"></a>Предварительные условия

* В облачном SharePoint-QnA Maker для разрешений используется Microsoft Graph. Если SharePoint находится в локальной среде, вы не сможете извлечь из SharePoint, так как Microsoft Graph не сможет определить разрешения.
* Формат URL-QnA Maker поддерживает только URL-адреса SharePoint, которые создаются для общего доступа и имеют формат `https://\*.sharepoint.com`

## <a name="add-supported-file-types-to-knowledge-base"></a>Добавить Поддерживаемые типы файлов в базу знаний

В базу знаний можно добавить все [типы файлов](../index.yml) , поддерживаемые QnA Maker, с сайта SharePoint. Если файловый ресурс защищен, может потребоваться предоставить [разрешения](#permissions) .

1. В библиотеке с сайтом SharePoint выберите меню с многоточием для файла, `...` .
1. Скопируйте URL-адрес файла.

   ![Получите URL-адрес файла SharePoint, выбрав меню с многоточием файла, а затем скопировав URL-адрес.](../media/add-sharepoint-datasources/get-sharepoint-file-url.png)

1. На портале QnA Maker на странице **Параметры** добавьте URL-адрес базы знаний.

### <a name="images-with-sharepoint-files"></a>Изображения с файлами SharePoint

Если файлы содержат изображения, они не извлекаются. Вы можете добавить образ с QnA Maker портала после извлечения файла в пары QnA.

Добавьте образ со следующим синтаксисом Markdown:

```markdown
![Explanation or description of image](URL of public image)
```

Текст в квадратных скобках, `[]` посвященный изображению. URL-адрес в круглых скобках, `()` — это прямая ссылка на изображение.

При проверке пары QnA на интерактивной панели тестирования на QnA Maker портале отображается изображение вместо текста Markdown. Это позволит получить общий доступ к образу из клиентского приложения.

## <a name="permissions"></a>Разрешения

Предоставление разрешений происходит при добавлении защищенного файла с сервера SharePoint в базу знаний. В зависимости от настройки SharePoint и разрешений пользователя, который добавляет файл, это может потребовать:

* никаких дополнительных действий — пользователь, добавляющий файл, имеет все необходимые разрешения.
* как [Диспетчер базы знаний](#knowledge-base-manager-add-sharepoint-data-source-in-qna-maker-portal) , так и [Диспетчер Active Directory](#active-directory-manager-grant-file-read-access-to-qna-maker).

См. шаги, приведенные ниже.

### <a name="knowledge-base-manager-add-sharepoint-data-source-in-qna-maker-portal"></a>Диспетчер базы знаний: Добавление источника данных SharePoint на портале QnA Maker

Когда **диспетчер QnA Maker** добавляет защищенный документ SharePoint в базу знаний, диспетчер базы знаний инициирует запрос на разрешение, которое должен выполнить Диспетчер Active Directory.

Запрос начинается с всплывающего окна для проверки подлинности в учетной записи Active Directory.

![Проверка подлинности учетной записи пользователя](../media/add-sharepoint-datasources/authenticate-user-account.png)

После того как диспетчер QnA Maker выберет учетную запись, администратор Azure Active Directory получит уведомление о том, что ему нужно разрешить доступ к ресурсу SharePoint для приложения QnA Maker (а не QnA Maker Manager). Диспетчер Azure Active Directory должен сделать это для каждого ресурса SharePoint, но не для каждого документа в этом ресурсе.

### <a name="active-directory-manager-grant-file-read-access-to-qna-maker"></a>Диспетчер Active Directory: предоставление доступа к файлу для чтения QnA Maker

Диспетчеру Active Directory (не диспетчеру QnA Maker) необходимо предоставить доступ к ресурсу SharePoint QnA Maker, выбрав [эту ссылку](https://login.microsoftonline.com/common/oauth2/v2.0/authorize?response_type=id_token&scope=Files.Read%20Files.Read.All%20Sites.Read.All%20User.Read%20User.ReadBasic.All%20profile%20openid%20email&client_id=c2c11949-e9bb-4035-bda8-59542eb907a6&redirect_uri=https%3A%2F%2Fwww.qnamaker.ai%3A%2FCreate&state=68) , чтобы авторизовать приложение портала QnA Maker SharePoint Enterprise, чтобы иметь разрешения на чтение файлов.

![Диспетчер Azure Active Directory предоставляет разрешение в интерактивном режиме](../media/add-sharepoint-datasources/aad-manager-grants-permission-interactively.png)

<!--
The Active Directory manager must grant QnA Maker access either by application name, `QnAMakerPortalSharePoint`, or by application ID, `c2c11949-e9bb-4035-bda8-59542eb907a6`.
-->
<!--
### Grant access from the interactive pop-up window

The Active Directory manager will get a pop-up window requesting permissions to the `QnAMakerPortalSharePoint` app. The pop-up window includes the QnA Maker Manager email address that initiated the request, an `App Info` link to learn more about **QnAMakerPortalSharePoint**, and a list of permissions requested. Select **Accept** to provide those permissions.

![Azure Active Directory manager grants permission interactively](../media/add-sharepoint-datasources/aad-manager-grants-permission-interactively.png)
-->
<!--

### Grant access from the App Registrations list

1. The Active Directory manager signs in to the Azure portal and opens **[App registrations list](https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ApplicationsListBlade)**.

1. Search for and select the **QnAMakerPortalSharePoint** app. Change the second filter box from **My apps** to **All apps**. The app information will open on the right side.

    ![Select QnA Maker app in App registrations list](../media/add-sharepoint-datasources/select-qna-maker-app-in-app-registrations.png)

1. Select **Settings**.

    [![Select Settings in the right-side blade](../media/add-sharepoint-datasources/select-settings-for-qna-maker-app-registration.png)](../media/add-sharepoint-datasources/select-settings-for-qna-maker-app-registration.png#lightbox)

1. Under **API access**, select **Required permissions**.

    ![Select 'Settings', then under 'API access', select 'Required permission'](../media/add-sharepoint-datasources/select-required-permissions-in-settings-blade.png)

1. Do not change any settings in the **Enable Access** window. Select **Grant Permission**.

    [![Under 'Grant Permission', select 'Yes'](../media/add-sharepoint-datasources/grant-app-required-permissions.png)](../media/add-sharepoint-datasources/grant-app-required-permissions.png#lightbox)

1. Select **YES** in the pop-up confirmation windows.

    ![Grant required permissions](../media/add-sharepoint-datasources/grant-required-permissions.png)
-->
### <a name="grant-access-from-the-azure-active-directory-admin-center"></a>Предоставление доступа из центра администрирования Azure Active Directory

1. Active Directory Manager входит в портал Azure и открывает **[корпоративные приложения](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps)**.

1. Найдите `QnAMakerPortalSharePoint` и выберите приложение QnA Maker.

    [![Поиск Кнамакерпорталшарепоинт в списке корпоративных приложений](../media/add-sharepoint-datasources/search-enterprise-apps-for-qna-maker.png)](../media/add-sharepoint-datasources/search-enterprise-apps-for-qna-maker.png#lightbox)

1. В разделе **Безопасность** перейдите к разделу **разрешения**. Выберите **предоставить согласие администратора для Организации**.

    [![Выберите пользователя, прошедшего проверку подлинности, для Active Directory администратора](../media/add-sharepoint-datasources/grant-aad-permissions-to-enterprise-app.png)](../media/add-sharepoint-datasources/grant-aad-permissions-to-enterprise-app.png#lightbox)

1. Выберите учетную запись Sign-On с разрешениями на предоставление разрешений для Active Directory.




## <a name="add-sharepoint-data-source-with-apis"></a>Добавление источника данных SharePoint с помощью API

Ниже приведены действия по добавлению последнего содержимого SharePoint через API с помощью хранилища BLOB-объектов Azure. 
1.  Скачайте файлы SharePoint локально. Пользователь, обращающийся к API, должен иметь доступ к SharePoint. 
1.  Отправьте их в хранилище BLOB-объектов Azure. При этом будет создан безопасный общий доступ с [помощью маркера SAS.](../../../storage/common/storage-sas-overview.md#how-a-shared-access-signature-works) 
1. Передайте URL-адрес большого двоичного объекта, созданный с помощью маркера SAS, в API службы QnA Maker. Чтобы разрешить ответ на вопрос из файлов, необходимо добавить тип файла суффикса в конце URL-адреса в формате "&ext = PDF" или "&ext = doc", прежде чем передать его в API службы QnA Maker.


<!--
## Get SharePoint File URI

Use the following steps to transform the SharePoint URL into a sharing token.

1. Encode the URL using [base64](https://en.wikipedia.org/wiki/Base64).

1. Convert the base64-encoded result to an unpadded base64url format with the following character changes.

    * Remove the equal character, `=` from the end of the value.
    * Replace `/` with `_`.
    * Replace `+` with `-`.
    * Append `u!` to be beginning of the string.

1. Sign in to Graph explorer and run the following query, where `sharedURL` is ...:

    ```
    https://graph.microsoft.com/v1.0/shares/<sharedURL>/driveitem
    ```

    Get the **@microsoft.graph.downloadUrl** and use this as `fileuri` in the QnA Maker APIs.

### Add or update a SharePoint File URI to your knowledge base

Use the **@microsoft.graph.downloadUrl** from the previous section as the `fileuri` in the QnA Maker API for [adding a knowledge base](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase) or [updating a knowledge base](/rest/api/cognitiveservices/qnamaker/knowledgebase/update). The following fields are mandatory: name, fileuri, filename, source.

```
{
    "name": "Knowledge base name",
    "files": [
        {
            "fileUri": "<@microsoft.graph.downloadURL>",
            "fileName": "filename.xlsx",
            "source": "<SharePoint link>"
        }
    ],
    "urls": [],
    "users": [],
    "hostUrl": "",
    "qnaList": []
}
```



## Remove QnA Maker app from SharePoint authorization

1. Use the steps in the previous section to find the Qna Maker app in the Active Directory admin center.
1. When you select the **QnAMakerPortalSharePoint**, select **Overview**.
1. Select **Delete** to remove permissions.

-->

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Совместная работа с базой знаний](../index.yml)
