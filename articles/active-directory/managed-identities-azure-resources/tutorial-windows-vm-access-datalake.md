---
title: Учебник. Получение доступа к Azure Data Lake Store для Windows в Azure AD с помощью управляемого удостоверения
description: В этом руководстве описано, как получить доступ к Azure Data Lake Storage с помощью назначаемого системой управляемого удостоверения на виртуальной машине Windows.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: cd28ee509dc76ea0b2aca9264c591a7176ae2661
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101093821"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-data-lake-store"></a>Руководство. Получение доступа к Azure Data Lake Storage с помощью назначаемого системой управляемого удостоверения на виртуальной машине Windows

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

В этом руководстве описано, как получить доступ к Azure Data Lake Storage с помощью назначаемого системой управляемого удостоверения на виртуальной машине Windows. Управляемыми удостоверениями службы автоматически управляет Azure. Они позволяют проходить проверку подлинности в службах, поддерживающих аутентификацию Azure AD, без указания учетных данных в коде. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Предоставление виртуальной машине доступа к Azure Data Lake Store.
> * Получение маркера доступа с использованием удостоверения виртуальной машины и получение доступа к Azure Data Lake Store с его помощью.

## <a name="prerequisites"></a>Предварительные требования

- Основные сведения об управляемых удостоверениях. См. дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](overview.md). 
- Учетная запись Azure. Зарегистрируйте [бесплатную учетную запись](https://azure.microsoft.com/free/).
- Разрешения роли "Владелец" в соответствующей области (подписка или группа ресурсов) для выполнения требуемых операций создания ресурсов и управления ролями учетной записи. Если нуждаетесь в помощи с назначением ролей, прочитайте статью [Назначение ролей Azure с помощью портала Azure](../../role-based-access-control/role-assignments-portal.md).
- Кроме того, вам потребуется виртуальная машина Windows с включенными управляемыми удостоверениями, назначенными системой.
  - Если вам нужно создать виртуальную машину для работы с этим руководством, см. раздел [Управляемое удостоверение, назначаемое системой](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity).




## <a name="enable"></a>Включить

[!INCLUDE [msi-tut-enable](../../../includes/active-directory-msi-tut-enable.md)]



## <a name="grant-access"></a>Предоставление доступа

Теперь можно предоставить виртуальной машине доступ к файлам и папкам в Azure Data Lake Store.  Для выполнения этого шага можно использовать имеющееся хранилище Azure Data Lake Store или создать новое.  Чтобы создать новое хранилище Data Lake Store с помощью портала Azure, следуйте этому [краткому руководству по Azure Data Lake Store](../../data-lake-store/data-lake-store-get-started-portal.md). В [документации по Azure Data Lake Store](../../data-lake-store/data-lake-store-overview.md) также есть краткие руководства, описывающие использование Azure CLI и Azure PowerShell.

Создайте папку в Data Lake Storage и предоставьте назначаемому системой удостоверению виртуальной машины разрешение на чтение, запись и выполнение файлов в этой папке:

1. На портале Azure щелкните **Data Lake Store** в области навигации слева.
2. Выберите хранилище Data Lake Store, которое хотите использовать для этого руководства.
3. Щелкните **Обозреватель данных** на панели команд.
4. Выбрана корневая папка Data Lake Store.  На панели команд щелкните **Доступ**.
5. Нажмите кнопку **Добавить**.  В поле **Выбор** введите имя виртуальной машины, например **DevTestVM**.  Выберите свою виртуальную машину в результатах поиска, а затем щелкните **Выбрать**.
6. Щелкните **Выбор разрешений**.  Выберите разрешения **Чтение** и **Выполнение** и добавьте их в раздел **Эта папка**, затем добавьте их в качестве **разрешений только для доступа**.  Нажмите кнопку **ОК**.  Разрешения должны быть успешно добавлены.
7. Закройте колонку **Доступ**.
8. В рамках этого руководства мы создадим папку.  Щелкните **Создать папку** в командной строке и введите имя новой папки, например **TestFolder**.  Нажмите кнопку **ОК**.
9. Щелкните созданную папку, а затем щелкните **Доступ** на панели команд.
10. Как и на шаге 5, нажмите кнопку **Добавить**, в поле **Выбор** введите имя виртуальной машины, выберите ее и нажмите кнопку **Выбрать**.
11. Как и на шаге 6, щелкните **Выбор разрешений**, выберите разрешения **Чтение**, **Запись** и **Выполнение**, добавьте их в раздел **Эта папка**, после чего добавьте их в качестве **записи разрешения доступа и записи разрешений по умолчанию**.  Нажмите кнопку **ОК**.  Разрешения должны быть успешно добавлены.

Теперь назначаемое системой управляемое удостоверение виртуальной машины сможет выполнять все операции с файлами в созданной папке.  Дополнительные сведения об управлении доступом к Data Lake Store см. в статье [Контроль доступа в Azure Data Lake Store](../../data-lake-store/data-lake-store-access-control.md).

## <a name="access-data"></a>Доступ к данным

В репозитории Azure Data Lake Storage изначально реализована поддержка аутентификации Azure AD, поэтому он может напрямую принимать маркеры доступа, полученные с помощью управляемых удостоверений для ресурсов Azure.  Для аутентификации в файловой системе Data Lake Store маркер доступа, выданный службой Azure AD конечной точке файловой системы Data Lake Store, отправляется в заголовке авторизации в формате "Bearer <ЗНАЧЕНИЕ_МАРКЕРА_ДОСТУПА>".  Дополнительные сведения о поддержке Data Lake Store для аутентификации Azure AD см. в разделе [Аутентификация в Data Lake Store с помощью Azure Active Directory](../../data-lake-store/data-lakes-store-authentication-using-azure-active-directory.md).

> [!NOTE]
> Клиентские пакеты SDK файловой системы Data Lake Storage пока не поддерживают управляемые удостоверения для ресурсов Azure.  Это руководство будет обновлено, как только поддержка будет добавлена в пакет SDK.

В этом руководстве выполняется аутентификация в файловой системе Data Lake Store посредством REST API с использованием PowerShell для выполнения запросов REST. Чтобы использовать назначаемое системой управляемое удостоверение виртуальной машины для проверки подлинности, нужно отправлять запросы с виртуальной машины.

1. На портале перейдите к разделу **Виртуальные машины**, выберите свою виртуальную машину Windows и в разделе **Обзор** щелкните **Подключить**.
2. Введите **имя пользователя** и **пароль**, добавленные при создании виртуальной машины Windows. 
3. Теперь, когда создано **подключение к удаленному рабочему столу** с виртуальной машиной, откройте **PowerShell** в удаленном сеансе. 
4. С помощью командлета PowerShell `Invoke-WebRequest` выполните запрос к локальным управляемым удостоверениям для конечной точки ресурсов Azure, чтобы получить маркер доступа к Azure Data Lake Storage.  Идентификатор ресурса для Data Lake Store: `https://datalake.azure.net/`.  Data Lake проводит точное сопоставление с идентификатором ресурса, и косая черта в конце важна.

   ```powershell
   $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatalake.azure.net%2F' -Method GET -Headers @{Metadata="true"}
   ```
    
   Преобразуйте ответ из объекта JSON в объект PowerShell. 
    
   ```powershell
   $content = $response.Content | ConvertFrom-Json
   ```

   Извлеките маркер доступа из ответа.
    
   ```powershell
   $AccessToken = $content.access_token
   ```

5. С помощью команды PowerShell Invoke-WebRequest выполните запрос к конечной точке REST Data Lake Store, чтобы вывести список папок в корневой папке.  Это простой способ проверить, все ли настроено правильно.  Очень важно, чтобы слово Bearer в заголовке авторизации начиналось с заглавной буквы B.  Имя хранилища Data Lake Store можно найти в разделе **Обзор** колонки "Data Lake Store" на портале Azure.

   ```powershell
   Invoke-WebRequest -Uri https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS -Headers @{Authorization="Bearer $AccessToken"}
   ```

   Успешный ответ выглядит следующим образом.

   ```powershell
   StatusCode        : 200
   StatusDescription : OK
   Content           : {"FileStatuses":{"FileStatus":[{"length":0,"pathSuffix":"TestFolder","type":"DIRECTORY", "blockSize":0,"accessTime":1507934941392, "modificationTime":1507944835699,"replication":0, "permission":"770","ow..."
   RawContent        : HTTP/1.1 200 OK
                       Pragma: no-cache
                       x-ms-request-id: b4b31e16-e968-46a1-879a-3474aa7d4528
                       x-ms-webhdfs-version: 17.04.22.00
                       Status: 0x0
                       X-Content-Type-Options: nosniff
                       Strict-Transport-Security: ma...
   Forms             : {}
   Headers           : {[Pragma, no-cache], [x-ms-request-id, b4b31e16-e968-46a1-879a-3474aa7d4528],
                       [x-ms-webhdfs-version, 17.04.22.00], [Status, 0x0]...}
   Images            : {}
   InputFields       : {}
   Links             : {}
   ParsedHtml        : System.__ComObject
   RawContentLength  : 556
   ```

6. Теперь вы можете попробовать передать файл в Data Lake Store.  Сначала создайте файл для передачи.

   ```powershell
   echo "Test file." > Test1.txt
   ```

7. С помощью команды PowerShell `Invoke-WebRequest` выполните запрос к конечной точке REST Data Lake Store, чтобы передать файл в папку, созданную ранее.  Этот запрос выполняется в два этапа.  На первом этапе вы создаете запрос и получаете адрес перенаправления для передачи файла.  На втором этапе вы фактически передаете файл.  Не забудьте задать имена папки и файла соответствующим образом, если вы использовали значения, отличные от приведенных в этом руководстве. 

   ```powershell
   $HdfsRedirectResponse = Invoke-WebRequest -Uri https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/TestFolder/Test1.txt?op=CREATE -Method PUT -Headers @{Authorization="Bearer $AccessToken"} -Infile Test1.txt -MaximumRedirection 0
   ```

   Если проверить значение ответа `$HdfsRedirectResponse`, оно должно иметь следующий вид.

   ```powershell
   PS C:\> $HdfsRedirectResponse

   StatusCode        : 307
   StatusDescription : Temporary Redirect
   Content           : {}
   RawContent        : HTTP/1.1 307 Temporary Redirect
                       Pragma: no-cache
                       x-ms-request-id: b7ab492f-b514-4483-aada-4aa0611d12b3
                       ContentLength: 0
                       x-ms-webhdfs-version: 17.04.22.00
                       Status: 0x0
                       X-Content-Type-Options: nosn...
   Headers           : {[Pragma, no-cache], [x-ms-request-id, b7ab492f-b514-4483-aada-4aa0611d12b3], 
                       [ContentLength, 0], [x-ms-webhdfs-version, 17.04.22.00]...}
   RawContentLength  : 0
   ```

   Выполните передачу, отправив запрос к конечной точке перенаправления.

   ```powershell
   Invoke-WebRequest -Uri $HdfsRedirectResponse.Headers.Location -Method PUT -Headers @{Authorization="Bearer $AccessToken"} -Infile Test1.txt -MaximumRedirection 0
   ```

   Успешный ответ выглядит следующим образом.

   ```powershell
   StatusCode        : 201
   StatusDescription : Created
   Content           : {}
   RawContent        : HTTP/1.1 201 Created
                       Pragma: no-cache
                       x-ms-request-id: 1e70f36f-ead1-4566-acfa-d0c3ec1e2307
                       ContentLength: 0
                       x-ms-webhdfs-version: 17.04.22.00
                       Status: 0x0
                       X-Content-Type-Options: nosniff
                       Strict...
   Headers           : {[Pragma, no-cache], [x-ms-request-id, 1e70f36f-ead1-4566-acfa-d0c3ec1e2307],
                       [ContentLength, 0], [x-ms-webhdfs-version, 17.04.22.00]...}
   RawContentLength  : 0
   ```

С помощью других интерфейсов API файловой системы Data Lake Store можно добавлять содержимое к файлам, скачивать файлы и многое другое.


## <a name="disable"></a>Отключить

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]


## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как получить доступ к Azure Data Lake Storage с помощью назначаемого системой управляемого удостоверения на виртуальной машине Windows. Дополнительные сведения об Azure Data Lake Store см. здесь:

> [!div class="nextstepaction"]
>[Azure Data Lake Storage](../../data-lake-store/data-lake-store-overview.md)