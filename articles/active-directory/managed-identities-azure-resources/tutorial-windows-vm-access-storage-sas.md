---
title: Руководство`:` использование управляемого удостоверения для доступа к службе хранилища Azure с учетными данными SAS — Azure AD
description: В этом руководстве показано, как использовать управляемое удостоверение, назначаемое системой, виртуальной машины Linux для доступа к службе хранилища Azure при помощи учетных данных SAS (подписанного URL-адреса) вместо ключа доступа к учетной записи хранения.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4f8b23d8f717e430e865e391a40692773f0beace
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776399"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-storage-via-a-sas-credential"></a>Руководство по Использование управляемого удостоверения, назначаемого системой, на виртуальной машине Windows для доступа к службе хранилища Azure с помощью учетных данных SAS

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

В этом руководстве вы узнаете, как использовать удостоверение, назначаемое системой, для виртуальной машины Windows, чтобы получить учетные данные SAS (подписанный URL-адрес) для хранилища. Для этого используются [учетные данные SAS службы](../../storage/common/storage-sas-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-shared-access-signatures). 

SAS службы позволяет предоставить определенной службе (в нашем случае службе BLOB-объектов) ограниченный доступ к объектам в учетной записи хранения на ограниченный срок, не передавая ей ключ доступа к учетной записи. Подписанный URL-адрес можно использовать стандартным образом для операций с хранилищем, например при использовании пакета SDK для службы хранилища Azure. В этом руководстве демонстрируются отправка и скачивание большого двоичного объекта с помощью PowerShell службы хранилища Azure. Вы научитесь:

> [!div class="checklist"]
> * Создание учетной записи хранения
> * предоставлять виртуальной машине доступ к подписанным URL-адресам учетной записи хранения в Resource Manager; 
> * получать маркер доступа с помощью удостоверения виртуальной машины и использовать его для извлечения SAS из Resource Manager. 

## <a name="prerequisites"></a>Предварительные требования

- Основные сведения об управляемых удостоверениях. См. дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](overview.md). 
- Учетная запись Azure. Зарегистрируйте [бесплатную учетную запись](https://azure.microsoft.com/free/).
- Разрешения роли "Владелец" в соответствующей области (подписка или группа ресурсов) для выполнения требуемых операций создания ресурсов и управления ролями учетной записи. Если нуждаетесь в помощи с назначением ролей, прочитайте статью [Назначение ролей Azure с помощью портала Azure](../../role-based-access-control/role-assignments-portal.md).
- Кроме того, вам потребуется виртуальная машина Windows с включенными управляемыми удостоверениями, назначенными системой.
  - Если вам нужно создать виртуальную машину для работы с этим руководством, см. раздел [Управляемое удостоверение, назначаемое системой](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="create-a-storage-account"></a>Создание учетной записи хранения 

Если у вас еще нет учетной записи хранения, создайте ее. Этот шаг можно пропустить и предоставить виртуальной машине доступ на основе управляемого удостоверения, назначаемого системой, к учетным данным SAS существующей учетной записи хранения. 

1. Нажмите кнопку **+ Создание службы** в верхнем левом углу портала Azure.
2. Щелкните **Хранилище**, а затем **Учетная запись хранения**, после чего отобразится новая панель "Создание учетной записи хранения".
3. Введите имя учетной записи хранения, которая будет использоваться далее.  
4. Для параметра **Модель развертывания** необходимо указать значение "Resource Manager", а для параметра **Тип учетной записи** — "Общее назначение". 
5. Убедитесь, что значения **подписки** и **группы ресурсов** соответствуют указанным при создании виртуальной машины на предыдущем шаге.
6. Нажмите кнопку **Создать**.

    ![Создание учетной записи хранения](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

## <a name="create-a-blob-container-in-the-storage-account"></a>Создание контейнера больших двоичных объектов в учетной записи хранения

Позже мы отправим и скачаем файл в новую учетную запись хранения. Так как файлам необходимо хранилище BLOB-объектов, нужно создать контейнер больших двоичных объектов, в котором будет храниться файл.

1. Вернитесь к только что созданной учетной записи хранения.
2. Щелкните ссылку **Контейнеры** в разделе "Служба BLOB-объектов" на панели слева.
3. Щелкните **+ Контейнер** в верхней части страницы, после чего появится панель "Новый контейнер".
4. Присвойте контейнеру имя, выберите уровень доступа, а затем нажмите кнопку **ОК**. Указанное имя будет использоваться далее в этом руководстве. 

    ![Создание контейнера хранилища](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

## <a name="grant-your-vms-system-assigned-managed-identity-access-to-use-a-storage-sas"></a>Предоставление управляемому удостоверению, назначаемому системой, виртуальной машины доступа к SAS хранилища 

В службе хранилища Azure не встроена поддержка проверки подлинности Azure AD.  Тем не менее управляемое удостоверение службы можно использовать для извлечения SAS хранилища из Resource Manager и последующего доступа к этому хранилищу.  На этом шаге мы предоставим управляемому удостоверению, назначаемому системой, виртуальной машины доступ к SAS учетной записи хранения.   

1. Вернитесь к только что созданной учетной записи хранения.   
2. Щелкните ссылку **Управление доступом (IAM)** на панели слева.  
3. Щелкните **+ Добавить назначение ролей** в верхней части страницы, чтобы добавить новое назначение роли для виртуальной машины.
4. Для параметра **Роль** в правой части страницы выберите значение "Участник учетных записей хранения".  
5. Рядом с ним в раскрывающемся списке **Назначение доступа к** выберите ресурс "Виртуальная машина".  
6. Убедитесь, что нужная подписка присутствует в раскрывающемся списке **Подписки**, и установите для параметра **Группы ресурсов** значение "Все группы ресурсов".  
7. И наконец, в разделе **Выбрать** выберите в раскрывающемся списке нужную виртуальную машину Windows и нажмите кнопку **Сохранить**. 

    ![Замещающий текст](./media/msi-tutorial-linux-vm-access-storage/msi-storage-role-sas.png)

## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-call-azure-resource-manager"></a>Получение маркера доступа с помощью удостоверения виртуальной машины и вызов Azure Resource Manager с его помощью 

Далее в этом руководстве мы будем работать с виртуальной машиной.

На этом этапе вы примените командлеты PowerShell для Azure Resource Manager.  Если у вас он не установлен, [скачайте последнюю версию](/powershell/azure/), прежде чем продолжить.

1. На портале Azure перейдите к разделу **Виртуальные машины**, выберите свою виртуальную машину Windows и вверху в разделе **Обзор** щелкните **Подключить**.
2. Введите **имя пользователя** и **пароль**, добавленные при создании виртуальной машины Windows. 
3. Теперь, когда создано **подключение к удаленному рабочему столу** с виртуальной машиной.
4. Откройте PowerShell в удаленном сеансе и с помощью команды Invoke-WebRequest получите маркер Azure Resource Manager на основе локального управляемого удостоверения для конечной точки ресурсов Azure.

    ```powershell
       $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Method GET -Headers @{Metadata="true"}
    ```
    
    > [!NOTE]
    > Значение параметра resource должно точно совпадать со значением, которое будет использоваться в Azure AD. Если используется идентификатор ресурса Resource Manager, добавьте косую черту после универсального кода ресурса (URI).
    
    Затем извлеките элемент content, который хранится в виде форматированной строки JSON в объекте $response. 
    
    ```powershell
    $content = $response.Content | ConvertFrom-Json
    ```
    Затем извлеките маркер доступа из ответа.
    
    ```powershell
    $ArmToken = $content.access_token
    ```

## <a name="get-a-sas-credential-from-azure-resource-manager-to-make-storage-calls"></a>Получение учетных данных SAS из Azure Resource Manager для обращения к хранилищу 

Теперь используйте PowerShell для выполнения вызова Resource Manager с помощью маркера доступа, полученного на предыдущем этапе, чтобы создать учетные данные SAS хранилища. Как только мы получим учетные данные SAS, можно вызывать операции хранилища.

Для этого мы будем использовать следующие параметры HTTP-запроса, чтобы создать учетные данные SAS:

```JSON
{
    "canonicalizedResource":"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>",
    "signedResource":"c",              // The kind of resource accessible with the SAS, in this case a container (c).
    "signedPermission":"rcw",          // Permissions for this SAS, in this case (r)ead, (c)reate, and (w)rite. Order is important.
    "signedProtocol":"https",          // Require the SAS be used on https protocol.
    "signedExpiry":"<EXPIRATION TIME>" // UTC expiration time for SAS in ISO 8601 format, for example 2017-09-22T00:06:00Z.
}
```

Эти параметры включены в тело запроса POST на получение учетных данных SAS. Дополнительные сведения о параметрах для создания учетных данных SAS см. в [справочнике по выводу SAS службы с использованием REST](/rest/api/storagerp/storageaccounts/listservicesas).

Сначала преобразуйте параметры в формат JSON, а затем выполните вызов конечной точки хранилища `listServiceSas` для создания учетных данных SAS:

```powershell
$params = @{canonicalizedResource="/blob/<STORAGE-ACCOUNT-NAME>/<CONTAINER-NAME>";signedResource="c";signedPermission="rcw";signedProtocol="https";signedExpiry="2017-09-23T00:00:00Z"}
$jsonParams = $params | ConvertTo-Json
```

```powershell
$sasResponse = Invoke-WebRequest -Uri https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE-ACCOUNT-NAME>/listServiceSas/?api-version=2017-06-01 -Method POST -Body $jsonParams -Headers @{Authorization="Bearer $ArmToken"}
```
> [!NOTE] 
> URL-адрес чувствителен к регистру, поэтому должен использоваться тот же регистр, который использовался, когда вы присваивали имя группе ресурсов. Проверьте, чтобы в resourceGroups обязательно использовался символ "G" (прописная буква). 

Теперь можно извлечь учетные данные SAS из ответа.

```powershell
$sasContent = $sasResponse.Content | ConvertFrom-Json
$sasCred = $sasContent.serviceSasToken
```

Учетные данные SAS будут выглядеть следующим образом:

```powershell
PS C:\> $sasCred
sv=2015-04-05&sr=c&spr=https&se=2017-09-23T00%3A00%3A00Z&sp=rcw&sig=JVhIWG48nmxqhTIuN0uiFBppdzhwHdehdYan1W%2F4O0E%3D
```

Затем нужно создать файл с именем "test.txt". Используйте учетные данные SAS для проверки подлинности с помощью командлета `New-AzStorageContent` и отправьте файл в контейнер больших двоичных объектов, а затем скачайте его.

```bash
echo "This is a test text file." > test.txt
```

Сначала установите командлеты службы хранилища Azure с помощью `Install-Module Azure.Storage`, а затем отправьте созданный большой двоичный объект с помощью командлета PowerShell `Set-AzStorageBlobContent`.

```powershell
$ctx = New-AzStorageContext -StorageAccountName <STORAGE-ACCOUNT-NAME> -SasToken $sasCred
Set-AzStorageBlobContent -File test.txt -Container <CONTAINER-NAME> -Blob testblob -Context $ctx
```

Ответ:

```powershell
ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob
BlobType          : BlockBlob
Length            : 56
ContentType       : application/octet-stream
LastModified      : 9/21/2017 6:14:25 PM +00:00
SnapshotTime      :
ContinuationToken :
Context           : Microsoft.WindowsAzure.Commands.Storage.AzureStorageContext
Name              : testblob
```

Скачать отправленный большой двоичный объект с помощью командлета PowerShell `Get-AzStorageBlobContent`.

```powershell
Get-AzStorageBlobContent -Blob testblob -Container <CONTAINER-NAME> -Destination test2.txt -Context $ctx
```

Ответ:

```powershell
ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob
BlobType          : BlockBlob
Length            : 56
ContentType       : application/octet-stream
LastModified      : 9/21/2017 6:14:25 PM +00:00
SnapshotTime      :
ContinuationToken :
Context           : Microsoft.WindowsAzure.Commands.Storage.AzureStorageContext
Name              : testblob
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как использовать управляемое удостоверение, назначаемое системой, виртуальной машины Windows для получения доступа к службе хранилища Azure с помощью учетных данных SAS.  Дополнительные сведения о SAS службы хранилища Azure см. здесь:

> [!div class="nextstepaction"]
>[Использование подписанных URL-адресов (SAS)](../../storage/common/storage-sas-overview.md)
