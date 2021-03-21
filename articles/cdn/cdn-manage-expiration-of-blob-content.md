---
title: Управление сроком действия хранилища BLOB-объектов Azure
titleSuffix: Azure Content Delivery Network
description: Сведения о возможностях контроля времени жизни BLOB-объектов в кэшировании Azure CDN.
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: ad4801e9-d09a-49bf-b35c-efdc4e6034e8
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: how-to
ms.date: 02/1/2018
ms.author: mazha
ms.openlocfilehash: 206ff6f888229356743bebb816cf03e4f7a7504b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92778708"
---
# <a name="manage-expiration-of-azure-blob-storage-in-azure-cdn"></a>Управление сроком действия хранилища BLOB-объектов Azure в Azure CDN
> [!div class="op_single_selector"]
> * [Веб-содержимое Azure](cdn-manage-expiration-of-cloud-service-content.md)
> * [Хранилище BLOB-объектов Azure](cdn-manage-expiration-of-blob-content.md)
> 
> 

[Служба хранилища BLOB-объектов](../storage/common/storage-introduction.md#blob-storage) в службе хранилища Azure — это один из нескольких источников в облаке Azure, интегрированных с сетью доставки содержимого (CDN) Azure. Любое общедоступное содержимое BLOB-объекта может кэшироваться в Azure CDN до истечения его срока жизни (TTL). Срок жизни определяется заголовком `Cache-Control`, указанным в HTTP-ответе исходного сервера. В этой статье описано несколько способов определения заголовка `Cache-Control` для большого двоичного объекта в службе хранилища Azure.

Вы также можете управлять параметрами кэша на портале Azure, определив правила кэширования CDN. Если создать правило кэширования и настроить его для **переопределения** или **отключения кэша**, предоставляемые системой параметры, которые описаны в этой статье, будут игнорироваться. См. дополнительные сведения о [функции кэширования](cdn-how-caching-works.md).

> [!TIP]
> Срок жизни для BLOB-объекта можно не указывать. В этом случае Azure CDN автоматически применяет стандартное значение TTL (семь дней), если только вы не настроили правила кэширования на портале Azure. Этот срок жизни по умолчанию применяется только к оптимизациям общей веб-доставки. Для оптимизаций больших файлов срок жизни по умолчанию составляет один день, а для оптимизаций потоковой передачи срок жизни по умолчанию составляет один год.
> 
> Дополнительные сведения о том, как Azure CDN ускоряет доступ к BLOB-объектам и другим файлам, см. в статье [Общие сведения о сети доставки содержимого (CDN) Azure](cdn-overview.md).
> 
> Дополнительные сведения о хранилище BLOB-объектов Azure см. в статье [Общие сведения о хранилище BLOB-объектов](../storage/blobs/storage-blobs-introduction.md).
 

## <a name="setting-cache-control-headers-by-using-cdn-caching-rules"></a>Определение заголовков Cache-Control с использованием правил кэширования CDN
Для установки заголовка `Cache-Control` BLOB-объекта рекомендуем использовать правила кэширования на портале Azure. Дополнительные сведения о правилах кэширования CDN см. в статье [Управление поведением кэширования сети доставки содержимого Azure с помощью правил кэширования](cdn-caching-rules.md).

> [!NOTE] 
> Правила кэширования доступны только для профилей **Azure CDN уровня "Стандартный" от Verizon** и **Azure CDN уровня "Стандартный" от Akamai**. Для профилей **Azure CDN уровня "Премиум" от Verizon** необходимо использовать [обработчик правил Azure CDN](./cdn-verizon-premium-rules-engine.md) на портале **Управление** с аналогичными функциональными возможностями.

**Чтобы перейти на страницу правил CDN для кэширования**:

1. На портале Azure выберите профиль CDN и конечную точку для BLOB-объекта.

2. В области слева в разделе "Параметры" выберите **Правила кэширования**.

   ![Кнопка правил кэширования CDN](./media/cdn-manage-expiration-of-blob-content/cdn-caching-rules-btn.png)

   Появится страница **Правила кэширования**.

   ![Страница кэширования CDN](./media/cdn-manage-expiration-of-blob-content/cdn-caching-page.png)


**Чтобы установить заголовки Cache-Control для службы хранения BLOB-объектов, используя глобальные правила кэширования, сделайте следующее:**

1. В разделе **Глобальные правила кэширования** задайте для параметра **Режим кэширования строк запросов** значение **Пропускать строки запросов**, а для параметра **Поведение кэширования** — значение **Переопределить**.
      
2. Для параметра **Срок действия кэша** введите 3600 в поле **Секунды** или 1 в поле **Часы**. 

   ![Пример глобальных правил кэширования CDN](./media/cdn-manage-expiration-of-blob-content/cdn-global-caching-rules-example.png)

   Это глобальное правило кэширования задает значение длительности кэширования в один час и влияет на все запросы к конечной точке. Оно переопределяет все заголовки HTTP `Cache-Control` или `Expires`, отправленные с сервера-источника, указанного конечной точкой.   

3. Щелкните **Сохранить**.
 
**Чтобы установить заголовки Cache-Control для файла большого двоичного объекта, используя настраиваемые правила кэширования, сделайте следующее:**

1. В разделе **Настраиваемые правила кэширования** создайте два условия соответствия.

     A. В первом условии соответствия задайте для параметра **Условие соответствия** значение **Путь** и введите значение `/blobcontainer1/*` для параметра **Значения соответствия**. Задайте для параметра **Поведение кэширования** значение **Переопределить** и введите 4 в поле **Часы**.

    Б. Во втором условии соответствия задайте параметру **Условие соответствия** значение **Путь** и введите значение `/blobcontainer1/blob1.txt` для параметра **Значения соответствия**. Задайте для параметра **Поведение кэширования** значение **Переопределить** и введите 2 в поле **Часы**.

    ![Пример настраиваемых правил кэширования CDN](./media/cdn-manage-expiration-of-blob-content/cdn-custom-caching-rules-example.png)

    Первое настраиваемое правило кэширования задает длительность кэширования четыре часа для всех файлов больших двоичных объектов в папке `/blobcontainer1` на сервере-источнике, указанном конечной точкой. Второе правило переопределяет первое правило только для файла большого двоичного объекта `blob1.txt` и задает для него длительность кэширования два часа.

2. Щелкните **Сохранить**.


## <a name="setting-cache-control-headers-by-using-azure-powershell"></a>Определение заголовков Cache-Control с помощью Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[Azure PowerShell](/powershell/azure/) — это одно из самых быстрых и мощных средств администрирования служб Azure. Используйте командлет `Get-AzStorageBlob`, чтобы получить ссылку на большой двоичный объект, а затем определите свойство `.ICloudBlob.Properties.CacheControl`. 

Пример:

```powershell
# Create a storage context
$context = New-AzStorageContext -StorageAccountName "<storage account name>" -StorageAccountKey "<storage account key>"

# Get a reference to the blob
$blob = Get-AzStorageBlob -Context $context -Container "<container name>" -Blob "<blob name>"

# Set the CacheControl property to expire in 1 hour (3600 seconds)
$blob.ICloudBlob.Properties.CacheControl = "max-age=3600"

# Send the update to the cloud
$blob.ICloudBlob.SetProperties()
```

> [!TIP]
> Можно также использовать PowerShell для [управления профилями и конечными точками CDN](cdn-manage-powershell.md).
> 
>

## <a name="setting-cache-control-headers-by-using-net"></a>Определение заголовков Cache-Control с помощью .NET
Чтобы определить заголовок `Cache-Control` для большого двоичного объекта с помощью кода .NET, задайте свойство [CloudBlob.Properties.CacheControl](/dotnet/api/microsoft.azure.storage.blob.blobproperties.cachecontrol) при помощи [клиентской библиотеки службы хранилища Azure для .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md).

Пример:

```csharp
class Program
{
    const string connectionString = "<storage connection string>";
    static void Main()
    {
        // Retrieve storage account information from connection string
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString);

        // Create a blob client for interacting with the blob service.
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        // Create a reference to the container
        CloudBlobContainer <container name> = blobClient.GetContainerReference("<container name>");

        // Create a reference to the blob
        CloudBlob <blob name> = container.GetBlobReference("<blob name>");

        // Set the CacheControl property to expire in 1 hour (3600 seconds)
        blob.Properties.CacheControl = "max-age=3600";

        // Update the blob's properties in the cloud
        blob.SetProperties();
    }
}
```

> [!TIP]
> Другие примеры кода .NET для хранилища BLOB-объектов Azure см. на [этой странице](https://azure.microsoft.com/documentation/samples/storage-blob-dotnet-getting-started/).
> 

## <a name="setting-cache-control-headers-by-using-other-methods"></a>Определение заголовков Cache-Control с помощью других методов

### <a name="azure-storage-explorer"></a>Обозреватель службы хранилища Azure
С помощью [обозревателя службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/) можно просматривать и изменять ресурсы хранилища BLOB-объектов, включая такие свойства, как *CacheControl*. 

Чтобы обновить свойство *CacheControl* большого двоичного объекта с помощью обозревателя хранилищ Azure, сделайте следующее.
   1. Выберите большой двоичный объект, а затем выберите **Свойства** в контекстном меню. 
   2. Прокрутите меню вниз до свойства *CacheControl*.
   3. Введите значение и выберите **Сохранить**.


![Свойства в обозревателе службы хранилища Azure](./media/cdn-manage-expiration-of-blob-content/cdn-storage-explorer-properties.png)

### <a name="azure-command-line-interface"></a>Интерфейс командной строки Azure
С помощью [интерфейса командной строки Azure](/cli/azure) (CLI) можно управлять ресурсами BLOB-объектов Azure из командной строки. Чтобы определить заголовок Cache-Control при передаче большого двоичного объекта с помощью Azure CLI, определите свойство *cacheControl* с помощью параметра `-p`. В следующем примере показано, как задать срок жизни, равный 1 часу (3600 секунд).
  
```azurecli
azure storage blob upload -c <connectionstring> -p cacheControl="max-age=3600" .\<blob name> <container name> <blob name>
```

### <a name="azure-storage-services-rest-api"></a>REST API служб хранилища Azure
Можно использовать [REST API служб хранилища Azure](/rest/api/storageservices/), чтобы явно задать свойство *x-ms-blob-cache-control* с помощью следующих операций в запросе:
  
   - [Put BLOB (Вставка BLOB-объекта)](/rest/api/storageservices/Put-Blob)
   - [Put Block List (Вставка списка блокировки)](/rest/api/storageservices/Put-Block-List)
   - [Set BLOB Properties (Задание свойств службы BLOB-объекта)](/rest/api/storageservices/Set-Blob-Properties)

## <a name="testing-the-cache-control-header"></a>Проверка заголовка Cache-Control
Вы легко можете проверить установленный для BLOB-объектов срок жизни. Используя встроенные в браузер [средства разработчика](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/), убедитесь, что ваш BLOB-объект содержит заголовок ответа `Cache-Control`. Для просмотра заголовков ответа также можно использовать такие средства, как [wget](https://www.gnu.org/software/wget/), [POST](https://www.getpostman.com/)или [Fiddler](https://www.telerik.com/fiddler) .

## <a name="next-steps"></a>Дальнейшие действия
* [Узнайте, как управлять сроком действия содержимого облачных служб в сети доставки содержимого (CDN) Azure](cdn-manage-expiration-of-cloud-service-content.md).
* [Дополнительные сведения о кэшировании](cdn-how-caching-works.md)