---
title: Руководство по Использование службы "Сетка событий Azure" для автоматического изменения размера переданных изображений
description: Руководство по Служба "Сетка событий Azure" может быть активирована при передаче больших двоичных объектов в службу хранилища Azure. Это можно использовать для отправки файлов изображений, которые передаются в службу хранилища Azure, в другие службы, например службу "Функции Azure", для изменения размера и других улучшений.
ms.topic: tutorial
ms.date: 07/07/2020
ms.openlocfilehash: ca231fc65162fe38f4dcb8b8d5677ef42c7807bb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99550524"
---
# <a name="tutorial-automate-resizing-uploaded-images-using-event-grid"></a>Руководство по Автоматическое изменение размера переданных изображений с помощью службы "Сетка событий"

[Сетка событий Azure](overview.md) — это служба обработки событий для облака. Служба "Сетка событий" дает возможность создавать подписки на события, порождаемые службами Azure или сторонними ресурсами.  

Это руководство — вторая часть из цикла руководств по службе хранилища. Оно дополняет [предыдущее руководство по службе хранилища][previous-tutorial] и содержит сведения о том, как добавить автоматическое бессерверное создание эскизов с помощью служб "Функции Azure" и "Сетка событий Azure". Служба "Сетка событий" позволяет службе [Функции Azure](../azure-functions/functions-overview.md) реагировать на события [хранилища BLOB-объектов Azure](../storage/blobs/storage-blobs-introduction.md) и создавать эскизы передаваемых изображений. Подписка на событие создается для события создания хранилища BLOB-объектов. При добавлении большого двоичного объекта в конкретный контейнер хранилища BLOB-объектов вызывается конечная точка функции. Данные, передаваемые посредством привязки функции из службы "Сетка событий", используются для доступа к большому двоичному объекту и создания эскизов.

Для добавления возможностей изменения размера в существующее приложение для передачи изображений можно использовать Azure CLI и портал Azure.

# <a name="net-v12-sdk"></a>[\.NET (пакет SDK версии 12)](#tab/dotnet)

![Снимок экрана: опубликованное веб-приложение в браузере для пакета SDK \.NET 12.](./media/resize-images-on-storage-blob-upload-event/tutorial-completed.png)

# <a name="nodejs-v10-sdk"></a>[Пакет SDK для Node.js версии 10](#tab/nodejsv10)

![Снимок экрана: опубликованное веб-приложение в браузере для пакета SDK \.NET 10.](./media/resize-images-on-storage-blob-upload-event/upload-app-nodejs-thumb.png)

---

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание учетной записи хранения Azure
> * развертывание бессерверного кода с помощью службы "Функции Azure";
> * создание подписки на событие хранилища BLOB-объектов в службе "Сетка событий".

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Для работы с этим руководством сделайте следующее:

Необходимо изучить руководство по использованию хранилища BLOB-объектов [Передача данных изображений в облако с помощью службы хранилища Azure][previous-tutorial].

Вам понадобится [подписка Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing). Для работы с этим учебником не подойдет подписка уровня **Бесплатный**. 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этим руководством вам понадобится Azure CLI 2.0.14 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

Если вы не используете Cloud Shell, сначала выполните вход с помощью `az login`.

Зарегистрируйте поставщик ресурсов службы "Сетка событий" в подписке, если вы еще не сделали это.

```bash
az provider register --namespace Microsoft.EventGrid
```

```powershell
az provider register --namespace Microsoft.EventGrid
```

## <a name="create-an-azure-storage-account"></a>Создание учетной записи хранения Azure

Для службы "Функции Azure" требуется общая учетная запись хранения. Кроме учетной записи хранения BLOB-объектов, созданной в рамках предыдущего руководства, создайте отдельно общую учетную запись хранения в группе ресурсов с помощью команды [az storage account create](/cli/azure/storage/account). Имя учетной записи хранения должно содержать от 3 до 24 символов и состоять только из цифр и строчных букв.

1. Задайте переменную для хранения имени группы ресурсов, созданную при работе с предыдущим руководством.

    ```bash
    resourceGroupName="myResourceGroup"
    ```

    ```powershell
    $resourceGroupName="myResourceGroup"
    ```

1. Задайте переменную, в которой будет храниться расположение создаваемых ресурсов. 

    ```bash
    location="eastus"
    ```

    ```powershell
    $location="eastus"
    ```

1. Задайте переменную имени новой учетной записи хранения, которая требуется для приложения-функции Azure.

    ```bash
    functionstorage="<name of the storage account to be used by the function>"
    ```

    ```powershell
    $functionstorage="<name of the storage account to be used by the function>"
    ```

1. Создайте учетную запись хранения для функции Azure.

    ```bash
    az storage account create --name $functionstorage --location $location \
    --resource-group $resourceGroupName --sku Standard_LRS --kind StorageV2
    ```

    ```powershell
    az storage account create --name $functionstorage --location $location `
    --resource-group $resourceGroupName --sku Standard_LRS --kind StorageV2
    ```

## <a name="create-a-function-app"></a>Создание приложения-функции  

Для выполнения функций вам понадобится приложение-функция, предоставляющее среду для выполнения кода функции без сервера. Создайте приложение-функцию с помощью команды [az functionapp create](/cli/azure/functionapp).

В следующей команде укажите уникальное название приложения-функции. Имя приложения-функции используется по умолчанию в качестве его домена DNS. Поэтому оно должно быть уникальным среди всех приложений в Azure.

1. Укажите имя для создаваемого приложения-функции.

    ```bash
    functionapp="<name of the function app>"
    ```

    ```powershell
    $functionapp="<name of the function app>"
    ```

1. Создайте функцию Azure.

    ```bash
    az functionapp create --name $functionapp --storage-account $functionstorage \
      --resource-group $resourceGroupName --consumption-plan-location $location \
      --functions-version 2
    ```

    ```powershell
    az functionapp create --name $functionapp --storage-account $functionstorage `
      --resource-group $resourceGroupName --consumption-plan-location $location `
      --functions-version 2
    ```

Теперь настройте приложение-функцию для подключения к учетной записи хранения, созданной в ходе выполнения инструкций из предыдущего [руководства][previous-tutorial].

## <a name="configure-the-function-app"></a>Настройка приложения-функции

Чтобы обеспечить работу приложения-функции, введите данные входа в учетную запись хранения BLOB-объектов, добавив эти данные в настройки приложения-функции с помощью команды [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings).

# <a name="net-v12-sdk"></a>[\.NET (пакет SDK версии 12)](#tab/dotnet)

```bash
storageConnectionString=$(az storage account show-connection-string --resource-group $resourceGroupName \
  --name $blobStorageAccount --query connectionString --output tsv)

az functionapp config appsettings set --name $functionapp --resource-group $resourceGroupName \
  --settings AzureWebJobsStorage=$storageConnectionString THUMBNAIL_CONTAINER_NAME=thumbnails \
  THUMBNAIL_WIDTH=100 FUNCTIONS_EXTENSION_VERSION=~2
```

```powershell
$storageConnectionString=$(az storage account show-connection-string --resource-group $resourceGroupName `
  --name $blobStorageAccount --query connectionString --output tsv)

az functionapp config appsettings set --name $functionapp --resource-group $resourceGroupName `
  --settings AzureWebJobsStorage=$storageConnectionString THUMBNAIL_CONTAINER_NAME=thumbnails `
  THUMBNAIL_WIDTH=100 FUNCTIONS_EXTENSION_VERSION=~2
```

# <a name="nodejs-v10-sdk"></a>[Пакет SDK для Node.js версии 10](#tab/nodejsv10)

```bash
blobStorageAccountKey=$(az storage account keys list -g $resourceGroupName \
  -n $blobStorageAccount --query [0].value --output tsv)

storageConnectionString=$(az storage account show-connection-string --resource-group $resourceGroupName \
  --name $blobStorageAccount --query connectionString --output tsv)

az functionapp config appsettings set --name $functionapp --resource-group $resourceGroupName \
  --settings FUNCTIONS_EXTENSION_VERSION=~2 BLOB_CONTAINER_NAME=thumbnails \
  AZURE_STORAGE_ACCOUNT_NAME=$blobStorageAccount \
  AZURE_STORAGE_ACCOUNT_ACCESS_KEY=$blobStorageAccountKey \
  AZURE_STORAGE_CONNECTION_STRING=$storageConnectionString
```

```powershell
$blobStorageAccountKey=$(az storage account keys list -g $resourceGroupName `
  -n $blobStorageAccount --query [0].value --output tsv)

$storageConnectionString=$(az storage account show-connection-string --resource-group $resourceGroupName `
  --name $blobStorageAccount --query connectionString --output tsv)

az functionapp config appsettings set --name $functionapp --resource-group $resourceGroupName `
  --settings FUNCTIONS_EXTENSION_VERSION=~2 BLOB_CONTAINER_NAME=thumbnails `
  AZURE_STORAGE_ACCOUNT_NAME=$blobStorageAccount `
  AZURE_STORAGE_ACCOUNT_ACCESS_KEY=$blobStorageAccountKey `
  AZURE_STORAGE_CONNECTION_STRING=$storageConnectionString
```

---

Параметр `FUNCTIONS_EXTENSION_VERSION=~2` позволяет запустить приложение-функцию в версии 2 среды выполнения решения "Функции Azure".

Теперь можно развернуть проект кода функции в этом приложении-функции.

## <a name="deploy-the-function-code"></a>Развертывание кода функции 

# <a name="net-v12-sdk"></a>[\.NET (пакет SDK версии 12)](#tab/dotnet)

Пример функции изменения размера на C# см. на [GitHub](https://github.com/Azure-Samples/function-image-upload-resize). Разверните этот код в приложение-функцию с помощью команды [az functionapp deployment source config](/cli/azure/functionapp/deployment/source).

```bash
az functionapp deployment source config --name $functionapp --resource-group $resourceGroupName \
  --branch master --manual-integration \
  --repo-url https://github.com/Azure-Samples/function-image-upload-resize
```

```powershell
az functionapp deployment source config --name $functionapp --resource-group $resourceGroupName `
  --branch master --manual-integration `
  --repo-url https://github.com/Azure-Samples/function-image-upload-resize
```

# <a name="nodejs-v10-sdk"></a>[Пакет SDK для Node.js версии 10](#tab/nodejsv10)

Пример функции изменения размера на Node.js можно найти в [GitHub](https://github.com/Azure-Samples/storage-blob-resize-function-node-v10). Разверните этот проект кода функции в приложение-функцию с помощью команды [az functionapp deployment source config](/cli/azure/functionapp/deployment/source).

```bash
az functionapp deployment source config --name $functionapp \
  --resource-group $resourceGroupName --branch master --manual-integration \
  --repo-url https://github.com/Azure-Samples/storage-blob-resize-function-node-v10
```

```powershell
az functionapp deployment source config --name $functionapp `
  --resource-group $resourceGroupName --branch master --manual-integration `
  --repo-url https://github.com/Azure-Samples/storage-blob-resize-function-node-v10
```

---

Функция изменения размера образа активируется HTTP-запросом, отправленным из службы "Сетка событий". Вы указываете этой службе, что хотите получать эти уведомления по URL-адресу функции, создав подписку на события. В этом руководстве вы подписываетесь на события, созданные большим двоичным объектом.

Данные, передаваемые в функцию из уведомления службы "Сетка событий", содержат URL-адрес большого двоичного объекта. Этот URL-адрес передается входной привязке, что позволяет получить отправленный образ из хранилища BLOB-объектов. Функция создает эскиз и записывает результирующий поток в отдельный контейнер в хранилище BLOB-объектов.

Этот проект использует `EventGridTrigger` как тип триггера. Рекомендуется использовать триггер Сетки событий, а не универсальные триггеры HTTP. Служба "Сетка событий" автоматически проверяет триггеры функций сетки событий. При использовании универсальных триггеров HTTP вам нужно реализовать [ответ проверки](security-authentication.md).

# <a name="net-v12-sdk"></a>[\.NET (пакет SDK версии 12)](#tab/dotnet)

Дополнительные сведения об этой функции см. в [файлах function.json и run.csx](https://github.com/Azure-Samples/function-image-upload-resize/tree/master/ImageFunctions).

# <a name="nodejs-v10-sdk"></a>[Пакет SDK для Node.js версии 10](#tab/nodejsv10)

См. дополнительные сведения об этой функции в [файлах function.json и index.js](https://github.com/Azure-Samples/storage-blob-resize-function-node-v10/tree/master/Thumbnail).

---

Код проекта функция развертывается непосредственно из общедоступного примера репозитория. Чтобы узнать больше о параметрах развертывания службы "Функции Azure", ознакомьтесь с разделом [Непрерывное развертывание для Функций Azure](../azure-functions/functions-continuous-deployment.md).

## <a name="create-an-event-subscription"></a>Создание подписки на событие

Подписка на событие определяет, какие создаваемые поставщиком события необходимо отправлять в конкретную конечную точку. В данном случае конечная точка предоставляется функцией. Чтобы создать подписку на событие, которая отправляет уведомления в функцию, выполните следующие действия на портале Azure:

1. На [портале Azure](https://portal.azure.com)в верхней части страницы выполните поиск `Function App` и щелкните требуемый результат. Затем выберите только что созданное приложение-функцию. Щелкните элемент **Функции** и выберите функцию **Эскиз**.

    :::image type="content" source="media/resize-images-on-storage-blob-upload-event/choose-thumbnail-function.png" alt-text="Выбор функции Эскиз на портале":::

1.  Щелкните элемент **Интеграция**. Затем последовательно выберите элементы **Триггер сетки событий** и **Создание подписки для Сетки событий**.

    :::image type="content" source="./media/resize-images-on-storage-blob-upload-event/add-event-subscription.png" alt-text="Переход к параметру добавления подписки Сетки событий на портале Azure" :::

1. Задайте значения параметров подписки на событие, указанные в таблице.
    
    ![Создание подписки на событие из функции на портале Azure](./media/resize-images-on-storage-blob-upload-event/event-subscription-create.png)

    | Параметр      | Рекомендуемое значение  | Описание                                        |
    | ------------ | ---------------- | -------------------------------------------------- |
    | **имя**; | imageresizersub | Имя, которое идентифицирует новую подписку на событие. |
    | **Тип раздела** | Учетные записи хранения | Выберите поставщик событий учетной записи хранения. |
    | **Подписка** | Ваша подписка Azure. | По умолчанию должна быть выбрана ваша текущая подписка Azure. |
    | **Группа ресурсов** | myResourceGroup | Щелкните **Использовать существующую** и выберите группу ресурсов, используемую в этом руководстве. |
    | **Ресурс** | Учетная запись хранения больших двоичных объектов | Выберите учетную запись хранения BLOB-объектов, созданную вами. |
    | **Имя системного раздела** | imagestoragesystopic | Введите имя системного раздела. См. [общие сведения о системных разделах](system-topics.md). |    
    | **Типы событий** | Blob created | Снимите флажки всех типов, кроме **Blob created** (Большой двоичный объект создан). Только события типа `Microsoft.Storage.BlobCreated` будут передаваться в функцию. |
    | **Тип конечной точки** | autogenerated | Предварительно определен как **Функции Azure**. |
    | **Конечная точка** | autogenerated | Имя функции. В нашем примере это функция **Эскиз**. |

1. Перейдите на вкладку **Фильтры** и выполните следующие действия:
    1. Выберите параметр **Включить фильтрацию тем**.
    1. Для **Тема начинается с** укажите следующее значение: **/blobServices/default/containers/images/** .

        ![Выбор фильтра для подписки на событие](./media/resize-images-on-storage-blob-upload-event/event-subscription-filter.png)

1. Нажмите кнопку **Создать**, чтобы добавить подписку на событие. Будет создана подписка на событие, которая вызывает функцию `Thumbnail` при добавлении большого двоичного объекта в контейнер `images`. Эта функция изменяет размер изображений и добавляет их в контейнер `thumbnails`.

Теперь, когда внутренние службы настроены, следует проверить функциональные возможности изменения размера изображений в примере веб-приложения.

## <a name="test-the-sample-app"></a>Поверка примера приложения

Чтобы проверить изменение размера изображений в веб-приложении, перейдите на URL-адрес опубликованного приложения. URL-адрес приложения по умолчанию: `https://<web_app>.azurewebsites.net`.

# <a name="net-v12-sdk"></a>[\.NET (пакет SDK версии 12)](#tab/dotnet)

Щелкните область **Upload photos** (Передача фотографий), чтобы выбрать и передать файл. Можно также перетащить фотографию в эту область.

Обратите внимание, что после того как переданное изображение исчезнет, его копия появится в карусели **Generated Thumbnails** (Созданные эскизы). С помощью функции размер этого образа был изменен, после чего он был добавлен в контейнер *эскизов* и скачан веб-клиентом.

![Снимок экрана: опубликованное веб-приложение с названием ImageResizer в браузере для пакета SDK \.NET 12.](./media/resize-images-on-storage-blob-upload-event/tutorial-completed.png)

# <a name="nodejs-v10-sdk"></a>[Пакет SDK для Node.js версии 10](#tab/nodejsv10)

Чтобы выбрать файл, щелкните **Выбрать файл**, а затем — **Загрузить изображение**. Если изображение загрузилось, браузер направит вас к странице успешной загрузки. Щелкните ссылку, чтобы вернуться к начальной странице. Копия отправленного изображения отображается в области **Generated Thumbnails** (Созданные эскизы). (Если изображение не появилось сразу, попробуйте перезагрузить страницу.) С помощью функции размер этого образа был изменен, после чего он был добавлен в контейнер *эскизов* и скачан веб-клиентом.

![Опубликованное веб-приложение в браузере](./media/resize-images-on-storage-blob-upload-event/upload-app-nodejs-thumb.png)

---

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * создание общей учетной записи хранения Azure;
> * развертывание бессерверного кода с помощью службы "Функции Azure";
> * создание подписки на событие хранилища BLOB-объектов в службе "Сетка событий".

Перейдите к третьей части цикла руководств по службе хранилища, чтобы узнать, как защитить доступ к учетной записи хранения.

> [!div class="nextstepaction"]
> [Безопасный доступ к данным приложения в облаке](../storage/blobs/storage-secure-access-application.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

+ Чтобы узнать больше о службе "Сетка событий", ознакомьтесь с разделом [Общие сведения о службе "Сетка событий Azure"](overview.md).
+ Чтобы ознакомиться с еще одним руководством по возможностям службы "Функции Azure", прочитайте в статью [Создание функции, интегрируемой с Azure Logic Apps](../azure-functions/functions-twitter-email.md).

[previous-tutorial]: ../storage/blobs/storage-upload-process-images.md
