---
title: Получение доступа к службе хранилища Azure с помощью управляемого удостоверения, назначенного системой виртуальной машины Windows | Документация Майкрософт
description: Из этого руководства вы узнаете, как получить доступ к службе хранилища Azure с помощью управляемого удостоверения, назначаемого системой, на виртуальной машине Windows.
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
ms.date: 01/14/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: de1cc69b3cfdac307edf6dfe999a5d538c2cb811
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "89263184"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-storage"></a>Руководство. Использование управляемого удостоверения, назначаемого системой, на виртуальной машине Windows для доступа к службе хранилища Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

В этом руководстве описывается, как получить доступ к службе хранилища Azure, используя управляемое удостоверение, назначаемое системой, на виртуальной машине Windows. Вы научитесь:

> [!div class="checklist"]
> * создать контейнер больших двоичных объектов в учетной записи хранения;
> * предоставить виртуальной машине Windows доступ на основе управляемого удостоверения, назначаемого системой, к учетной записи хранения;
> * получить доступ и использовать его для вызова службы хранилища Azure.

> [!NOTE]
> Аутентификация Azure Active Directory для службы хранилища Azure находится на этапе общедоступной предварительной версии.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]



## <a name="enable"></a>Включить

[!INCLUDE [msi-tut-enable](../../../includes/active-directory-msi-tut-enable.md)]



## <a name="grant-access"></a>Предоставление доступа


### <a name="create-storage-account"></a>Создание учетной записи хранения

В этом разделе вы создадите учетную запись хранения.

1. Нажмите кнопку **Создать ресурс** в верхнем левом углу окна портала Azure.
2. Щелкните **Хранилище**, а затем — **Учетная запись хранения — BLOB-объект, файл, таблица, очередь**.
3. В поле **Имя** введите имя учетной записи хранения.
4. Для параметра **Модель развертывания** выберите **Resource Manager**, а для поля **Тип учетной записи** — **Хранилище (версия 1, общего назначения)** .
5. Убедитесь, что значения **подписки** и **группы ресурсов** соответствуют указанным при создании виртуальной машины на предыдущем шаге.
6. Нажмите кнопку **Создать**.

    ![Создание учетной записи хранения](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

### <a name="create-a-blob-container-and-upload-a-file-to-the-storage-account"></a>Создание контейнера больших двоичных объектов и передача файла в учетную запись хранения

Так как файлам необходимо хранилище BLOB-объектов, нужно создать контейнер больших двоичных объектов, в котором будет храниться файл. Затем файл отправляется в контейнер больших двоичных объектов в новой учетной записи хранения.

1. Вернитесь к только что созданной учетной записи хранения.
2. В колонке **Служба BLOB-объектов** щелкните **Контейнеры**.
3. В верхней области страницы щелкните **+ Container** (+ Контейнер).
4. В разделе **Создание контейнера** введите имя контейнера, а в разделе **Общедоступный уровень доступа** оставьте значение по умолчанию.

    ![Создание контейнера хранилища](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

5. С помощью редактора по своему усмотрению создайте на локальном компьютере файл с именем *hello world.txt*. Откройте его и добавьте текст (без кавычек) "Hello world! :)", а затем сохраните его.
6. Передайте файл в только что созданный контейнер. Для этого щелкните имя контейнера, а затем нажмите кнопку **Отправить**.
7. В области **Отправить BLOB-объект** под полем **Файлы** щелкните значок папки и перейдите к файлу **hello_world.txt** на локальном компьютере, выберите этот файл и нажмите кнопку **Отправить**.
    ![Отправка текстового файла](./media/msi-tutorial-linux-vm-access-storage/upload-text-file.png)

### <a name="grant-access"></a>Предоставление доступа

В этом разделе показано, как предоставить виртуальной машине доступ к контейнеру службы хранилища Azure Вы можете использовать управляемое удостоверение, назначаемое системой, на виртуальной машине, чтобы извлечь данные в большом двоичном объекте службы хранилища Azure.

1. Вернитесь к только что созданной учетной записи хранения.
2. Щелкните ссылку **Управление доступом (IAM)** на панели слева.
3. Щелкните **+ Добавить назначение ролей** в верхней части страницы, чтобы добавить новое назначение роли для виртуальной машины.
4. В поле **Роль** из раскрывающегося списка выберите **Модуль чтения данных BLOB-объектов хранилища**.
5. В следующем раскрывающемся списке в поле **Назначение доступа к** выберите **Виртуальная машина**.
6. Убедитесь, что нужная подписка присутствует в раскрывающемся списке **Подписка**, и установите для параметра **Группа ресурсов** значение **Все группы ресурсов**.
7. В разделе **Выбранные элементы** выберите свою виртуальную машину и нажмите кнопку **Сохранить**.

    ![Назначение разрешений](./media/tutorial-linux-vm-access-storage/access-storage-perms.png)

## <a name="access-data"></a>Доступ к данным 

В службе хранилища Azure изначально реализована поддержка аутентификации Azure AD, поэтому она может напрямую принимать маркеры доступа, полученные с помощью управляемого удостоверения. Эта реализация является частью интеграции службы хранилища Azure с Azure AD и отличается от указания учетных данных в строке подключения.

Вот пример кода .NET, который позволяет открыть подключение к службе хранилища Azure с помощью маркера доступа, а затем считать содержимое файла, созданного ранее. Этот код должен выполняться на виртуальной машине, чтобы иметь доступ к конечной точке управляемого удостоверения виртуальной машины. Для использования метода с маркером доступа необходима платформа .NET Framework 4.6 или более поздней версии. Замените значение `<URI to blob file>` на соответствующее. Вы можете получить это значение, если перейдете в созданный и отправленный в хранилище BLOB-объектов файл и скопируете **URL-адрес** в разделе **свойств** страницы **обзора**.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.IO;
using System.Net;
using System.Web.Script.Serialization;
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage.Blob;

namespace StorageOAuthToken
{
    class Program
    {
        static void Main(string[] args)
        {
            //get token
            string accessToken = GetMSIToken("https://storage.azure.com/");

            //create token credential
            TokenCredential tokenCredential = new TokenCredential(accessToken);

            //create storage credentials
            StorageCredentials storageCredentials = new StorageCredentials(tokenCredential);

            Uri blobAddress = new Uri("<URI to blob file>");

            //create block blob using storage credentials
            CloudBlockBlob blob = new CloudBlockBlob(blobAddress, storageCredentials);

            //retrieve blob contents
            Console.WriteLine(blob.DownloadText());
            Console.ReadLine();
        }

        static string GetMSIToken(string resourceID)
        {
            string accessToken = string.Empty;
            // Build request to acquire MSI token
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=" + resourceID);
            request.Headers["Metadata"] = "true";
            request.Method = "GET";

            try
            {
                // Call /token endpoint
                HttpWebResponse response = (HttpWebResponse)request.GetResponse();

                // Pipe response Stream to a StreamReader, and extract access token
                StreamReader streamResponse = new StreamReader(response.GetResponseStream());
                string stringResponse = streamResponse.ReadToEnd();
                JavaScriptSerializer j = new JavaScriptSerializer();
                Dictionary<string, string> list = (Dictionary<string, string>)j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
                accessToken = list["access_token"];
                return accessToken;
            }
            catch (Exception e)
            {
                string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
                return accessToken;
            }
        }
    }
}
```

В ответе содержится содержимое файла:

`Hello world! :)`


## <a name="disable"></a>Отключить

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]



## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как включить удостоверение, назначаемое системой, на виртуальной машине Windows, чтобы получить доступ к службе хранилища Azure.  Дополнительные сведения о службе хранилища Azure см. в статье ниже.

> [!div class="nextstepaction"]
> [Хранилище Azure](../../storage/common/storage-introduction.md)