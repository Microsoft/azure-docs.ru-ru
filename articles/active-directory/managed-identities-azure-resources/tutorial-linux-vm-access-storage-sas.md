---
title: Руководство. Доступ к службе хранилища Azure с помощью учетных данных SAS для Linux в Azure AD
description: В этом руководстве показано, как использовать назначаемое системой управляемое удостоверение виртуальной машины Linux для доступа к службе хранилища Azure при помощи учетных данных SAS (подписанного URL-адреса) вместо ключа доступа к учетной записи хранения.
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
ms.date: 11/03/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3edc63a1532bb6889fc490e400dbb57e7bce10d0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93360417"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-identity-to-access-azure-storage-via-a-sas-credential"></a>Руководство. Использование назначаемого системой управляемого удостоверения виртуальной машины Linux для доступа к службе хранилища Azure при помощи учетных данных SAS

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

В этом руководстве показано, как с помощью назначаемого системой управляемого удостоверения для виртуальной машины Linux получить учетные данные SAS (подписанный URL-адрес) для хранилища. Для этого используются [учетные данные SAS службы](../../storage/common/storage-sas-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-shared-access-signatures). 

> [!NOTE]
> Ключ SAS, создаваемый в этом руководстве, не ограничивается виртуальной машиной и не привязан к ней.  

SAS службы предоставляет определенной службе (в нашем случае службе BLOB-объектов) ограниченный доступ к объектам в учетной записи хранения на ограниченный срок, не передавая ей ключ доступа к учетной записи. Подписанный URL-адрес можно использовать стандартным образом для операций с хранилищем, например при использовании пакета SDK для службы хранилища Azure. В этом руководстве описано, как отправлять и скачивать большой двоичный объект с помощью интерфейса командной строки хранилища Azure. Вы научитесь:


> [!div class="checklist"]
> * Создание учетной записи хранения
> * Создание контейнера больших двоичных объектов в учетной записи хранения
> * предоставлять виртуальной машине доступ к подписанным URL-адресам учетной записи хранения в Resource Manager; 
> * получать маркер доступа с помощью удостоверения виртуальной машины и использовать его для извлечения SAS из Resource Manager. 

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

## <a name="create-a-storage-account"></a>Создание учетной записи хранения 

Если у вас еще нет учетной записи хранения, создайте ее.  Этот шаг можно пропустить и предоставить через назначаемое системой управляемое удостоверение виртуальной машины доступ к ключам существующей учетной записи хранения. 

1. Нажмите кнопку **+ Создание службы** в верхнем левом углу портала Azure.
2. Щелкните **Хранилище**, а затем **Учетная запись хранения**, после чего отобразится новая панель "Создание учетной записи хранения".
3. Введите **имя** учетной записи хранения, которая будет использоваться далее.  
4. Для параметра **Модель развертывания** необходимо выбрать "Resource Manager", а для параметра **Account kind** (Тип учетной записи) — "Общее назначение". 
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

В службе хранилища Azure не встроена поддержка проверки подлинности Azure AD.  Вместо этого вы можете использовать назначаемое системой управляемое удостоверение виртуальной машины для получения SAS хранилища из Resource Manager, а затем осуществлять доступ к хранилищу при помощи этого SAS.  На этом шаге мы предоставим управляемому удостоверению, назначаемому системой, виртуальной машины доступ к SAS учетной записи хранения.   

1. Вернитесь к только что созданной учетной записи хранения.
2. Щелкните ссылку **Управление доступом (IAM)** на панели слева.  
3. Щелкните **+ Добавить назначение ролей** в верхней части страницы, чтобы добавить новое назначение роли для виртуальной машины.
4. Для параметра **Роль** в правой части страницы выберите значение "Участник учетных записей хранения". 
5. Рядом с ним в раскрывающемся списке **Назначение доступа к** выберите ресурс "Виртуальная машина".  
6. Убедитесь, что нужная подписка присутствует в раскрывающемся списке **Подписки**, и установите для параметра **Группы ресурсов** значение "Все группы ресурсов".  
7. И, наконец, в разделе **Выбрать** выберите в раскрывающемся списке нужную виртуальную машину Linux и нажмите кнопку **Сохранить**.  

    ![Замещающий текст](./media/msi-tutorial-linux-vm-access-storage/msi-storage-role-sas.png)

## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-call-azure-resource-manager"></a>Получение маркера доступа с помощью удостоверения виртуальной машины и вызов Azure Resource Manager с его помощью

Далее в этом руководстве мы будем работать с виртуальной машиной, которую только что создали.

Для выполнения этих действий вам потребуется клиент SSH. Если вы используете Windows, можно использовать клиент SSH в [подсистеме Windows для Linux](/windows/wsl/install-win10). Если вам нужна помощь в настройке ключей SSH-клиента, ознакомьтесь с разделом [Использование ключей SSH с Windows в Azure](../../virtual-machines/linux/ssh-from-windows.md) или [Как создать и использовать пару из открытого и закрытого ключей SSH для виртуальных машин Linux в Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).

1. На портале Azure перейдите к разделу **Виртуальные машины**, выберите свою виртуальную машину Linux и вверху в разделе **Обзор** щелкните **Подключить**. Скопируйте строку подключения к виртуальной машине. 
2. Подключитесь к виртуальной машине c помощью клиента SSH.  
3. После этого вам предложат ввести **пароль**, добавленный при создании **виртуальной машины Linux**. Вы должны без проблем войти в систему.  
4. Используйте cURL для получения маркера доступа для Azure Resource Manager.  

    Запрос CURL маркера доступа и соответствующий ответ приведены ниже:
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true    
    ```
    
    > [!NOTE]
    > В предыдущем запросе значение параметра resource должно точно совпадать со значением, которое ожидает Azure AD. Если используется идентификатор ресурса Resource Manager, добавьте косую черту после универсального кода ресурса (URI).
    > В следующем ответе элемент access_token сокращен для удобства восприятия.
    
    ```bash
    {"access_token":"eyJ0eXAiOiJ...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://management.azure.com",
    "token_type":"Bearer"} 
     ```

## <a name="get-a-sas-credential-from-azure-resource-manager-to-make-storage-calls"></a>Получение учетных данных SAS из Azure Resource Manager для обращения к хранилищу

Теперь мы с помощью CURL вызовем Resource Manager, используя полученный на предыдущем этапе маркер доступа, и создадим учетные данные SAS для хранилища. Получив учетные данные SAS, мы сможем выполнять операции отправки и (или) скачивания в хранилище.

Для этого мы будем использовать следующие параметры HTTP-запроса, чтобы создать учетные данные SAS:

```JSON
{
    "canonicalizedResource":"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>",
    "signedResource":"c",              // The kind of resource accessible with the SAS, in this case a container (c).
    "signedPermission":"rcw",          // Permissions for this SAS, in this case (r)ead, (c)reate, and (w)rite.  Order is important.
    "signedProtocol":"https",          // Require the SAS be used on https protocol.
    "signedExpiry":"<EXPIRATION TIME>" // UTC expiration time for SAS in ISO 8601 format, for example 2017-09-22T00:06:00Z.
}
```

Эти параметры включены в тело запроса POST на получение учетных данных SAS. Дополнительные сведения о параметрах для создания учетных данных SAS см. в [справочнике по выводу SAS службы с использованием REST](/rest/api/storagerp/storageaccounts/listservicesas).

Используйте следующий запрос CURL для получения учетных данных SAS. Не забудьте заменить значения параметров `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>`, `<STORAGE ACCOUNT NAME>`, `<CONTAINER NAME>` и `<EXPIRATION TIME>` своими значениями. Замените значение `<ACCESS TOKEN>` маркером доступа, который вы получили ранее:

```bash 
curl https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>/listServiceSas/?api-version=2017-06-01 -X POST -d "{\"canonicalizedResource\":\"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>\",\"signedResource\":\"c\",\"signedPermission\":\"rcw\",\"signedProtocol\":\"https\",\"signedExpiry\":\"<EXPIRATION TIME>\"}" -H "Authorization: Bearer <ACCESS TOKEN>"
```

> [!NOTE]
> Текст в приведенном выше URL-адресе чувствителен к регистру, поэтому используйте строчные и заглавные буквы, как указано в названиях групп ресурсов. Кроме того, учитывайте, что это запрос POST, а не GET.

Ответ от службы CURL возвращает учетные данные SAS:  

```bash 
{"serviceSasToken":"sv=2015-04-05&sr=c&spr=https&st=2017-09-22T00%3A10%3A00Z&se=2017-09-22T02%3A00%3A00Z&sp=rcw&sig=QcVwljccgWcNMbe9roAJbD8J5oEkYoq%2F0cUPlgriBn0%3D"} 
```

Создайте образец файла большого двоичного объекта, чтобы отправить его в контейнер хранилища BLOB-объектов. На виртуальной машине Linux это можно сделать с помощью следующей команды. 

```bash
echo "This is a test file." > test.txt
```

Теперь выполните аутентификацию с помощью команды CLI `az storage`, указав в ней учетные данные SAS, и отправьте файл в контейнер больших двоичных объектов. Для выполнения этого шага нужно [установить последнюю версию Azure CLI](/cli/azure/install-azure-cli) на виртуальной машине, если вы не сделали этого ранее.

```azurecli
 az storage blob upload --container-name 
                        --file 
                        --name
                        --account-name 
                        --sas-token
```

Ответ: 

```JSON
Finished[#############################################################]  100.0000%
{
  "etag": "\"0x8D4F9929765C139\"",
  "lastModified": "2017-09-21T03:58:56+00:00"
}
```

Кроме того, можно скачать файл с помощью Azure CLI и выполнить аутентификацию, используя учетные данные SAS. 

Запрос: 

```azurecli
az storage blob download --container-name
                         --file 
                         --name 
                         --account-name
                         --sas-token
```

Ответ: 

```JSON
{
  "content": null,
  "metadata": {},
  "name": "testblob",
  "properties": {
    "appendBlobCommittedBlockCount": null,
    "blobType": "BlockBlob",
    "contentLength": 16,
    "contentRange": "bytes 0-15/16",
    "contentSettings": {
      "cacheControl": null,
      "contentDisposition": null,
      "contentEncoding": null,
      "contentLanguage": null,
      "contentMd5": "Aryr///Rb+D8JQ8IytleDA==",
      "contentType": "text/plain"
    },
    "copy": {
      "completionTime": null,
      "id": null,
      "progress": null,
      "source": null,
      "status": null,
      "statusDescription": null
    },
    "etag": "\"0x8D4F9929765C139\"",
    "lastModified": "2017-09-21T03:58:56+00:00",
    "lease": {
      "duration": null,
      "state": "available",
      "status": "unlocked"
    },
    "pageBlobSequenceNumber": null,
    "serverEncrypted": false
  },
  "snapshot": null
}
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как использовать назначаемое системой управляемое удостоверение виртуальной машины Linux для получения доступа к службе хранилища Azure с помощью учетных данных SAS.  Дополнительные сведения о SAS службы хранилища Azure см. здесь:

> [!div class="nextstepaction"]
>[Использование подписанных URL-адресов (SAS)](../../storage/common/storage-sas-overview.md)
