---
title: Учебник. Доступ из веб-приложения к хранилищу с помощью управляемых удостоверений | Azure
description: Из этого учебника вы узнаете, как получить доступ к службе хранилища Azure для приложения с помощью управляемых удостоверений.
services: storage, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 11/30/2020
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1, devx-track-azurecli
ms.openlocfilehash: 7d84b3f8e654940a8f2c36075b92d630505e88b9
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107482344"
---
# <a name="tutorial-access-azure-storage-from-a-web-app"></a>Руководство по доступу к службе хранилища Azure из веб-приложения

Узнайте, как с помощью управляемых удостоверений получить доступ к службе хранилища Azure для веб-приложения (не от имени вошедшего в систему пользователя), выполняемого в Службе приложений Azure.

:::image type="content" alt-text="Схема, на которой показано, как получить доступ к хранилищу." source="./media/scenario-secure-app-access-storage/web-app-access-storage.svg" border="false":::

Предположим, что вам нужен доступ из веб-приложения к плоскости данных Azure (к службе хранилища Azure, Базе данных SQL Azure, Azure Key Vault или другим службам). Вы могли бы использовать общий ключ, но это создаст дополнительные проблемы с оперативной безопасностью, то есть с распределением прав на создание, развертывание секрета и управление им. Также этот ключ может случайно оказаться отправленным в GitHub, где злоумышленники смогут его найти. Есть более безопасный способ предоставить веб-приложению доступ к данным — применить [управляемое удостоверение](../active-directory/managed-identities-azure-resources/overview.md).

Управляемое удостоверение из Azure Active Directory (Azure AD) позволяет Службам приложений получать доступ к ресурсам через управление доступом на основе ролей (RBAC) без учетных данных приложения. Когда вы назначаете веб-приложению управляемое удостоверение, Azure создает и распространяет сертификат. Вам не придется беспокоиться об управлении секретами или учетными данными приложения.

В этом руководстве вы узнаете, как:

> [!div class="checklist"]
>
> * создавать для веб-приложения управляемое удостоверение, назначаемое системой;
> * создавать учетную запись хранения и контейнер хранилища BLOB-объектов Azure;
> * обращаться к хранилищу из веб-приложения с помощью управляемых удостоверений.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Обязательные условия

* Веб-приложение, выполняющееся в Службе приложений Azure, с [включенным модулем проверки подлинности или авторизации Службы приложений](scenario-secure-app-authentication-app-service.md).

## <a name="enable-managed-identity-on-an-app"></a>Включение управляемого удостоверения в приложении

Если вы создавали и публиковали веб-приложение с помощью Visual Studio, управляемое удостоверение уже включено для этого приложения. В Службе приложений в области слева выберите **Удостоверение**, а затем — **Назначаемое системой**. Убедитесь, что для параметра **Состояние** задано значение **Вкл.** В противном случае выберите **Сохранить**, а затем — **Да**, чтобы включить управляемое удостоверение, назначаемое системой. Если управляемое удостоверение включено, для состояния устанавливается значение **Вкл.** и будет доступен идентификатор объекта.

:::image type="content" alt-text="Снимок экрана с параметром системного удостоверения." source="./media/scenario-secure-app-access-storage/create-system-assigned-identity.png":::

Это действие создает новый идентификатор объекта, который отличается от идентификатора приложения, созданного на панели **Проверка подлинности и авторизация**. Скопируйте идентификатор объекта для управляемого удостоверения, назначаемого системой. Он понадобится вам позднее.

## <a name="create-a-storage-account-and-blob-storage-container"></a>Создание учетной записи хранения и контейнера хранилища BLOB-объектов

Теперь вы готовы создать учетную запись хранения и контейнер хранилища BLOB-объектов.

Каждая учетная запись хранения должна принадлежать группе ресурсов Azure. Группа ресурсов — это логический контейнер для группирования служб Azure. При создании учетной записи хранения у вас есть возможность создать новую или использовать имеющуюся группу ресурсов. В этой статье показано, как создать группу ресурсов.

Учетная запись хранения общего назначения версии 2 предоставляет доступ ко всем службам хранилища Azure (большим двоичным объектам, файлам, очередям, таблицам и дискам). С помощью описанной здесь процедуры создается учетная запись хранения общего назначения версии 2, но создание любого типа учетной записи хранения происходит таким же образом.

Большие двоичные объекты в службе хранилища Azure упорядочиваются в контейнеры. Прежде чем отправлять большой двоичный объект, необходимо создать для него контейнер.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы создать учетную запись хранения общего назначения версии 2 на портале Azure, сделайте следующее:

1. В меню портала Azure выберите **Все службы**. В списке ресурсов введите **Учетные записи хранения**. Как только вы начнете вводить символы, список отфильтруется соответствующим образом. Выберите **Учетные записи хранения**.

1. В появившемся окне **Учетные записи хранения** выберите **Добавить**.

1. Выберите подписку, в которой будет создана учетная запись хранения.

1. В поле **Группа ресурсов** выберите из раскрывающегося списка ту группу ресурсов, в которой размещено веб-приложение.

1. Далее введите имя своей учетной записи хранения. Выбранное вами имя должно быть уникальным в Azure. Также имя должно содержать от 3 до 24 символов и может состоять только из цифр и строчных букв.

1. Выберите расположение учетной записи хранения или используйте расположение по умолчанию.

1. Оставьте в следующих полях значения по умолчанию.

    |Поле|Значение|
    |--|--|
    |Модель развертывания|Resource Manager|
    |Производительность|Стандартный|
    |Тип учетной записи|StorageV2 (учетная запись общего назначения версии 2)|
    |Репликация|Геоизбыточное хранилище с доступом на чтение (RA-GRS)|
    |Уровень доступа|Горячий|

1. Выберите **Просмотр и создание**, чтобы просмотреть настройки учетной записи хранения и создать учетную запись.

1. Выберите **Создать**.

Чтобы создать контейнер хранилища BLOB-объектов в службе хранилища Azure, выполните следующие действия:

1. Войдите в новую учетную запись хранения на портале Azure.

1. В меню слева для учетной записи хранения перейдите к разделу **Служба BLOB-объектов**, а затем выберите **Контейнеры**.

1. Нажмите кнопку **+ Контейнер**.

1. Введите имя для нового контейнера. Все знаки в имени контейнера должны быть строчными. Оно должно начинаться с буквы или цифры и может содержать только буквы, цифры и дефисы (-).

1. Задайте для контейнера уровень общего доступа. Уровень по умолчанию — **Private (no anonymous access)** (Частный (не разрешать анонимный доступ)).

1. Нажмите кнопку **ОК**, чтобы создать контейнер.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы создать учетную запись хранения общего назначения версии 2 и контейнер хранилища BLOB-объектов, выполните следующий скрипт. Укажите имя группы ресурсов, которая содержит веб-приложение. Введите имя учетной записи хранения. Выбранное вами имя должно быть уникальным в Azure. Также имя должно содержать от 3 до 24 символов и может состоять только из цифр и строчных букв.

Укажите расположение для учетной записи хранения. Чтобы просмотреть список доступных расположений для вашей подписки, выполните ```Get-AzLocation | select Location```. Все знаки в имени контейнера должны быть строчными. Оно должно начинаться с буквы или цифры и может содержать только буквы, цифры и дефисы (-).

Не забудьте заменить значения заполнителей в угловых скобках фактическими значениями.

```powershell
Connect-AzAccount

$resourceGroup = "securewebappresourcegroup"
$location = "<location>"
$storageName="securewebappstorage"
$containerName = "securewebappblobcontainer"

$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -Location $location `
  -SkuName Standard_RAGRS `
  -Kind StorageV2

$ctx = $storageAccount.Context

New-AzStorageContainer -Name $containerName -Context $ctx -Permission blob
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы создать учетную запись хранения общего назначения версии 2 и контейнер хранилища BLOB-объектов, выполните следующий скрипт. Укажите имя группы ресурсов, которая содержит веб-приложение. Введите имя учетной записи хранения. Выбранное вами имя должно быть уникальным в Azure. Также имя должно содержать от 3 до 24 символов и может состоять только из цифр и строчных букв. 

Укажите расположение для учетной записи хранения. Все знаки в имени контейнера должны быть строчными. Оно должно начинаться с буквы или цифры и может содержать только буквы, цифры и дефисы (-).

В следующем примере учетная запись Azure AD используется для авторизации операции создания контейнера. Прежде чем создавать контейнер, назначьте себе роль Участник данных в хранилище BLOB-объектов. Даже если вы являетесь владельцем учетной записи, вам потребуются явные разрешения для выполнения операций с данными в учетной записи хранения.

Не забудьте заменить значения заполнителей в угловых скобках фактическими значениями.

```azurecli-interactive
az login

az storage account create \
    --name securewebappstorage \
    --resource-group securewebappresourcegroup \
    --location <location> \
    --sku Standard_ZRS \
    --encryption-services blob

storageId=$(az storage account show -n securewebappstorage -g securewebappresourcegroup --query id --out tsv)

az ad signed-in-user show --query objectId -o tsv | az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee @- \
    --scope $storageId

az storage container create \
    --account-name securewebappstorage \
    --name securewebappblobcontainer \
    --auth-mode login
```

---

## <a name="grant-access-to-the-storage-account"></a>Предоставление доступа к учетной записи хранения

Чтобы получить возможность создавать, считывать и удалять большие двоичные объекты, вам необходимо предоставить веб-приложению доступ к учетной записи хранения. На предыдущем шаге вы настроили управляемое удостоверение для веб-приложения, которое выполняется в Службе приложений. С помощью Azure RBAC вы можете предоставить этому управляемому удостоверению доступ к ресурсу, как и любому другому субъекту безопасности. Роль "Участник для данных BLOB-объектов хранилища" предоставляет веб-приложению (которое представлено этим управляемым удостоверением, назначаемым системой) доступ на чтение, запись и удаление самого контейнера больших двоичных объектов и данных в нем.

# <a name="portal"></a>[Портал](#tab/azure-portal)

На [портале Azure](https://portal.azure.com) откройте страницу учетной записи хранения, чтобы предоставить доступ веб-приложению. В области слева выберите **Управление доступом (IAM)** , а затем — **Назначение ролей**. Вы увидите список субъектов, которым разрешен доступ к учетной записи хранения. Теперь вам нужно добавить назначение роли для робота, то есть службы приложений, которая будет обращаться к этой учетной записи хранения. Выберите **Добавить** > **Добавить назначение ролей**.

В поле **Роль** выберите **Участник для данных BLOB-объектов хранилища**, чтобы веб-приложение получило доступ на чтение данных из больших двоичных объектов хранилища. В поле **Назначить доступ к** выберите **Служба приложений**. В поле **Подписка** выберите нужную подписку. Теперь выберите службу приложений, к которой вы намерены предоставить доступ. Щелкните **Сохранить**.

:::image type="content" alt-text="Снимок экрана: экран &quot;Добавление назначения ролей&quot;." source="./media/scenario-secure-app-access-storage/add-role-assignment.png":::

Теперь веб-приложение сможет обращаться к этой учетной записи хранения.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Выполните следующий скрипт, чтобы назначить веб-приложению, которое представлено этим управляемым удостоверением, назначаемым системой, роль "Участник для данных BLOB-объектов хранилища" для учетной записи хранения.

```powershell
$resourceGroup = "securewebappresourcegroup"
$webAppName="SecureWebApp20201102125811"
$storageName="securewebappstorage"

$spID = (Get-AzWebApp -ResourceGroupName $resourceGroup -Name $webAppName).identity.principalid
$storageId= (Get-AzStorageAccount -ResourceGroupName $resourceGroup -Name $storageName).Id
New-AzRoleAssignment -ObjectId $spID -RoleDefinitionName "Storage Blob Data Contributor" -Scope $storageId
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Выполните следующий скрипт, чтобы назначить веб-приложению, которое представлено этим управляемым удостоверением, назначаемым системой, роль "Участник для данных BLOB-объектов хранилища" для учетной записи хранения.

```azurecli-interactive
spID=$(az resource list -n SecureWebApp20201102125811 --query [*].identity.principalId --out tsv)

storageId=$(az storage account show -n securewebappstorage -g securewebappresourcegroup --query id --out tsv)

az role assignment create --assignee $spID --role 'Storage Blob Data Contributor' --scope $storageId
```

---

## <a name="access-blob-storage-net"></a>Доступ к хранилищу BLOB-объектов (.NET)

Класс [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) используется для получения учетных данных токена для кода, чтобы авторизовать запросы в службу хранилища Azure. Создайте экземпляр класса [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential), который использует управляемое удостоверение для получения токенов и их подключения к клиенту службы. Приведенный ниже пример кода получает учетные данные токена с пройденной проверкой подлинности и использует их для создания объекта клиента службы, который отправляет новый большой двоичный объект.

Просмотреть этот код как часть примера приложения можно на [сайте GitHub](https://github.com/Azure-Samples/ms-identity-easyauth-dotnet-storage-graphapi/tree/main/1-WebApp-storage-managed-identity).

### <a name="install-client-library-packages"></a>Установка пакетов клиентских библиотек

Установите [пакет NuGet для хранилища BLOB-объектов](https://www.nuget.org/packages/Azure.Storage.Blobs/), чтобы организовать работу с хранилищем BLOB-объектов, и [пакет NuGet клиентской библиотеки службы удостоверений Azure для .NET](https://www.nuget.org/packages/Azure.Identity/), чтобы выполнять проверку подлинности с учетными данными Azure AD. Для установки клиентских библиотек можно использовать интерфейс командной строки .NET Core или консоль диспетчера пакетов в Visual Studio.

# <a name="command-line"></a>[Командная строка](#tab/command-line)

Откройте командную строку и перейдите в каталог с файлом проекта.

Выполните команды установки.

```dotnetcli
dotnet add package Azure.Storage.Blobs

dotnet add package Azure.Identity
```

# <a name="package-manager"></a>[Диспетчер пакетов](#tab/package-manager)

Откройте проект или решение в Visual Studio, а затем — консоль с помощью команды **Средства** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.

Выполните команды установки.
```powershell
Install-Package Azure.Storage.Blobs

Install-Package Azure.Identity
```

---

### <a name="example"></a>Пример

```csharp
using System;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Text;
using System.IO;
using Azure.Identity;

// Some code omitted for brevity.

static public async Task UploadBlob(string accountName, string containerName, string blobName, string blobContents)
{
    // Construct the blob container endpoint from the arguments.
    string containerEndpoint = string.Format("https://{0}.blob.core.windows.net/{1}",
                                                accountName,
                                                containerName);

    // Get a credential and create a client object for the blob container.
    BlobContainerClient containerClient = new BlobContainerClient(new Uri(containerEndpoint),
                                                                    new DefaultAzureCredential());

    try
    {
        // Create the container if it does not exist.
        await containerClient.CreateIfNotExistsAsync();

        // Upload text to a new block blob.
        byte[] byteArray = Encoding.ASCII.GetBytes(blobContents);

        using (MemoryStream stream = new MemoryStream(byteArray))
        {
            await containerClient.UploadBlobAsync(blobName, stream);
        }
    }
    catch (Exception e)
    {
        throw e;
    }
}
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы завершили работу с этим учебником и вам больше не требуется веб-приложение или связанные с ним ресурсы, необходимо [очистить созданные ресурсы](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как:

> [!div class="checklist"]
>
> * создавать управляемое удостоверение, назначаемое системой;
> * создавать учетную запись хранения и контейнера хранилища BLOB-объектов;
> * обращаться к хранилищу из веб-приложения с помощью управляемых удостоверений.

> [!div class="nextstepaction"]
> [Доступ из Службы приложений к Microsoft Graph от имени пользователя](scenario-secure-app-access-microsoft-graph-as-user.md)
