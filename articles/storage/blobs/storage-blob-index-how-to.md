---
title: Использование тегов индекса больших двоичных объектов для управления и поиска данных в хранилище BLOB-объектов Azure
description: См. примеры использования тегов индекса больших двоичных объектов для категоризации, управления и запроса объектов BLOB.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 03/05/2021
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.reviewer: klaasl
ms.custom: devx-track-csharp
ms.openlocfilehash: a820f7efc39af8c6ab66c883d285b507c7bc7368
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102563274"
---
# <a name="use-blob-index-tags-preview-to-manage-and-find-data-on-azure-blob-storage"></a>Использование тегов индекса больших двоичных объектов (Предварительная версия) для управления и поиска данных в хранилище BLOB-объектов Azure

Теги индекса больших двоичных объектов классифицируют данные в учетной записи хранения с помощью атрибутов тегов "ключ — значение". Эти теги автоматически индексируются и представляются в виде многомерного индекса с возможностью поиска для простого поиска данных. В этой статье показано, как задавать, получать и находить данные с помощью тегов индекса больших двоичных объектов.

> [!IMPORTANT]
> Теги индекса BLOB-объектов сейчас доступны в **предварительной версии** и доступны во всех общедоступных регионах. Ознакомьтесь с [дополнительными условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не включены в общедоступную версию.

Дополнительные сведения об этой функции вместе с известными проблемами и ограничениями см. в статье [Управление данными большого двоичного объекта Azure с помощью тегов индекса BLOB-объектов (Предварительная версия)](storage-manage-find-blobs.md).

## <a name="prerequisites"></a>Предварительные требования

# <a name="portal"></a>[Портал](#tab/azure-portal)

- Подписка Azure, зарегистрированная и утвержденная для доступа к предварительной версии индекса больших двоичных объектов
- Доступ к [портал Azure](https://portal.azure.com/)

# <a name="net"></a>[.NET](#tab/net)

Так как индекс больших двоичных объектов находится в предварительной версии, пакет хранилища .NET выпускается в предварительной версии веб-канала NuGet. Эта библиотека может быть изменена в течение периода действия предварительной версии.

1. Настройте проект Visual Studio, чтобы начать работу с клиентской библиотекой версии 12 для хранилища BLOB-объектов Azure для .NET. Дополнительные сведения см. в разделе [Краткое руководство по .NET](storage-quickstart-blobs-dotnet.md) .

2. В диспетчере пакетов NuGet найдите пакет **Azure. Storage. BLOB** и установите в проекте версию **12.7.0-Preview. 1** или более позднюю. Вы также можете выполнить команду PowerShell: `Install-Package Azure.Storage.Blobs -Version 12.7.0-preview.1`

   Инструкции см. в разделе [Поиск и установка пакета](/nuget/consume-packages/install-use-packages-visual-studio#find-and-install-a-package).

3. Добавьте в начало файла кода приведенные ниже операторы using.

    ```csharp
    using Azure;
    using Azure.Storage.Blobs;
    using Azure.Storage.Blobs.Models;
    using Azure.Storage.Blobs.Specialized;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    ```

---

## <a name="upload-a-new-blob-with-index-tags"></a>Отправка нового большого двоичного объекта с тегами индекса

Эта задача может выполняться [владельцем данных большого двоичного объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) или субъектом безопасности, которому предоставлено разрешение на `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` [операцию поставщика ресурсов Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) с помощью настраиваемой роли Azure.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Выберите свою учетную запись хранения на [портале Azure](https://portal.azure.com/). 

2. Перейдите к параметру **контейнеры** в разделе **Служба BLOB-объектов**, выберите контейнер.

3. Нажмите кнопку **Отправить** и перейдите в локальную файловую систему, чтобы найти файл для передачи в виде блочного BLOB-объекта.

4. Разверните раскрывающийся список **Дополнительно** и перейдите к разделу **Теги индекса больших двоичных объектов**.

5. Введите теги индекса больших двоичных объектов, которые необходимо применить к данным, в виде пар "ключ-значение".

6. Нажмите кнопку **Отправить**, чтобы отправить большой двоичный объект.

:::image type="content" source="media/storage-blob-index-concepts/blob-index-upload-data-with-tags.png" alt-text="Снимок экрана портал Azure, показывающий, как отправить большой двоичный объект с тегами индекса.":::

# <a name="net"></a>[.NET](#tab/net)

В приведенном ниже примере показано, как создать добавочный большой двоичный объект с тегами, заданными во время создания.

```csharp
static async Task BlobIndexTagsOnCreate()
   {
      BlobServiceClient serviceClient = new BlobServiceClient(ConnectionString);
      BlobContainerClient container = serviceClient.GetBlobContainerClient("mycontainer");

      try
      {
          // Create a container
          await container.CreateIfNotExistsAsync();

          // Create an append blob
          AppendBlobClient appendBlobWithTags = container.GetAppendBlobClient("myAppendBlob0.logs");

          // Blob index tags to upload
          AppendBlobCreateOptions appendOptions = new AppendBlobCreateOptions();
          appendOptions.Tags = new Dictionary<string, string>
          {
              { "Sealed", "false" },
              { "Content", "logs" },
              { "Date", "2020-04-20" }
          };

          // Upload data with tags set on creation
          await appendBlobWithTags.CreateAsync(appendOptions);
      }
      finally
      {
      }
   }
```

---

## <a name="get-set-and-update-blob-index-tags"></a>Получение, задание и обновление тегов индекса больших двоичных объектов

Получение тегов индекса больших двоичных объектов может осуществляться [владельцем данных большого двоичного объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) или субъектом безопасности, которому предоставлено разрешение на `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read` [операцию поставщика ресурсов Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) с помощью настраиваемой роли Azure.

Установка и обновление тегов индекса BLOB-объектов может осуществляться [владельцем данных большого двоичного объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) или субъектом безопасности, которому предоставлено разрешение на `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` [операцию поставщика ресурсов Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) с помощью настраиваемой роли Azure.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Выберите свою учетную запись хранения на [портале Azure](https://portal.azure.com/). 

2. Перейдите к параметру **Контейнеры** в разделе **Служба BLOB-объектов** и выберите контейнер.

3. Выберите BLOB-объект из списка BLOB-объектов в выбранном контейнере.

4. На вкладке общих сведений о большом двоичном объекте отображаются его свойства, включая все **теги индекса больших двоичных объектов**.

5. Вы можете получить, задать, изменить или удалить любые теги индекса в виде пар "ключ-значение" для большого двоичного объекта.

6. Нажмите кнопку **Сохранить**, чтобы подтвердить изменения большого двоичного объекта.

:::image type="content" source="media/storage-blob-index-concepts/blob-index-get-set-tags.png" alt-text="Снимок экрана портал Azure, показывающий, как получить, задать, обновить и удалить теги индекса в больших двоичных объектах.":::

# <a name="net"></a>[.NET](#tab/net)

```csharp
static async Task BlobIndexTagsExample()
   {
      BlobServiceClient serviceClient = new BlobServiceClient(ConnectionString);
      BlobContainerClient container = serviceClient.GetBlobContainerClient("mycontainer");

      try
      {
          // Create a container
          await container.CreateIfNotExistsAsync();

          // Create a new append blob
          AppendBlobClient appendBlob = container.GetAppendBlobClient("myAppendBlob1.logs");
          await appendBlob.CreateAsync();

          // Set or update blob index tags on existing blob
          Dictionary<string, string> tags = new Dictionary<string, string>
          {
              { "Project", "Contoso" },
              { "Status", "Unprocessed" },
              { "Sealed", "true" }
          };
          await appendBlob.SetTagsAsync(tags);

          // Get blob index tags
          Response<IDictionary<string, string>> tagsResponse = await appendBlob.GetTagsAsync();
          Console.WriteLine(appendBlob.Name);
          foreach (KeyValuePair<string, string> tag in tagsResponse.Value)
          {
              Console.WriteLine($"{tag.Key}={tag.Value}");
          }

          // List blobs with all options returned including blob index tags
          await foreach (BlobItem blobItem in container.GetBlobsAsync(BlobTraits.All))
          {
              Console.WriteLine(Environment.NewLine + blobItem.Name);
              foreach (KeyValuePair<string, string> tag in blobItem.Tags)
              {
                  Console.WriteLine($"{tag.Key}={tag.Value}");
              }
          }

          // Delete existing blob index tags by replacing all tags
          Dictionary<string, string> noTags = new Dictionary<string, string>();
          await appendBlob.SetTagsAsync(noTags);

      }
      finally
      {
      }
   }
```

---

## <a name="filter-and-find-data-with-blob-index-tags"></a>Фильтрация и поиск данных с помощью тегов индекса больших двоичных объектов

Эта задача может выполняться [владельцем данных большого двоичного объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) или субъектом безопасности, которому предоставлено разрешение на `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/filter/action` [операцию поставщика ресурсов Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) с помощью настраиваемой роли Azure.

# <a name="portal"></a>[Портал](#tab/azure-portal)

В портал Azure фильтр тегов индекса больших двоичных объектов автоматически применяет `@container` параметр к области действия выбранного контейнера. Если вы хотите фильтровать и находить данные с тегами во всей учетной записи хранения, используйте наши REST API, пакеты SDK или средства.

1. Выберите свою учетную запись хранения на [портале Azure](https://portal.azure.com/). 

2. Перейдите к параметру **контейнеры** в разделе **Служба BLOB-объектов**, выберите контейнер.

3. Нажмите кнопку **Фильтр тегов индекса больших двоичных объектов** для фильтрации в выбранном контейнере.

4. Введите ключ тега индекса BLOB-объекта и значение тега

5. Нажмите кнопку **Фильтр тегов индекса больших двоичных объектов**, чтобы добавить дополнительные фильтры тегов (до 10).

:::image type="content" source="media/storage-blob-index-concepts/blob-index-tag-filter-within-container.png" alt-text="Снимок экрана портал Azure, показывающий, как фильтровать большие двоичные объекты с тегами и находить их с помощью тегов индекса":::

# <a name="net"></a>[.NET](#tab/net)

```csharp
static async Task FindBlobsByTagsExample()
   {
      BlobServiceClient serviceClient = new BlobServiceClient(ConnectionString);
      BlobContainerClient container1 = serviceClient.GetBlobContainerClient("mycontainer");
      BlobContainerClient container2 = serviceClient.GetBlobContainerClient("mycontainer2");

      // Blob index queries and selection
      String singleEqualityQuery = @"""Archive"" = 'false'";
      String andQuery = @"""Archive"" = 'false' AND ""Priority"" = '01'";
      String rangeQuery = @"""Date"" >= '2020-04-20' AND ""Date"" <= '2020-04-30'";
      String containerScopedQuery = @"@container = 'mycontainer' AND ""Archive"" = 'false'";

      String queryToUse = containerScopedQuery;

      try
      {
          // Create a container
          await container1.CreateIfNotExistsAsync();
          await container2.CreateIfNotExistsAsync();

          // Create append blobs
          AppendBlobClient appendBlobWithTags0 = container1.GetAppendBlobClient("myAppendBlob00.logs");
          AppendBlobClient appendBlobWithTags1 = container1.GetAppendBlobClient("myAppendBlob01.logs");
          AppendBlobClient appendBlobWithTags2 = container1.GetAppendBlobClient("myAppendBlob02.logs");
          AppendBlobClient appendBlobWithTags3 = container2.GetAppendBlobClient("myAppendBlob03.logs");
          AppendBlobClient appendBlobWithTags4 = container2.GetAppendBlobClient("myAppendBlob04.logs");
          AppendBlobClient appendBlobWithTags5 = container2.GetAppendBlobClient("myAppendBlob05.logs");
           
          // Blob index tags to upload
          CreateAppendBlobOptions appendOptions = new CreateAppendBlobOptions();
          appendOptions.Tags = new Dictionary<string, string>
          {
              { "Archive", "false" },
              { "Priority", "01" },
              { "Date", "2020-04-20" }
          };
          
          CreateAppendBlobOptions appendOptions2 = new CreateAppendBlobOptions();
          appendOptions2.Tags = new Dictionary<string, string>
          {
              { "Archive", "true" },
              { "Priority", "02" },
              { "Date", "2020-04-24" }
          };

          // Upload data with tags set on creation
          await appendBlobWithTags0.CreateAsync(appendOptions);
          await appendBlobWithTags1.CreateAsync(appendOptions);
          await appendBlobWithTags2.CreateAsync(appendOptions2);
          await appendBlobWithTags3.CreateAsync(appendOptions);
          await appendBlobWithTags4.CreateAsync(appendOptions2);
          await appendBlobWithTags5.CreateAsync(appendOptions2);

          // Find Blobs given a tags query
          Console.WriteLine("Find Blob by Tags query: " + queryToUse + Environment.NewLine);

          List<TaggedBlobItem> blobs = new List<TaggedBlobItem>();
          await foreach (TaggedBlobItem taggedBlobItem in serviceClient.FindBlobsByTagsAsync(queryToUse))
          {
              blobs.Add(taggedBlobItem);
          }

          foreach (var filteredBlob in blobs)
          {
              Console.WriteLine($"BlobIndex result: ContainerName= {filteredBlob.ContainerName}, " +
                  $"BlobName= {filteredBlob.Name}");
          }

      }
      finally
      {
      }
   }
```

---

## <a name="lifecycle-management-with-blob-index-tag-filters"></a>Управление жизненным циклом с помощью фильтров тегов индекса больших двоичных объектов

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Выберите свою учетную запись хранения на [портале Azure](https://portal.azure.com/). 

2. Перейдите к параметру **Управление жизненным циклом** в разделе **Служба BLOB-объектов**.

3. Выберите *Добавить правило*, а затем заполните поля формы набора действий.

4. Выберите **Фильтр** установить, чтобы добавить необязательный фильтр для сопоставления префиксов и индекса BLOB-объектов.

  :::image type="content" source="media/storage-blob-index-concepts/blob-index-match-lifecycle-filter-set.png" alt-text="Снимок экрана портал Azure, показывающий, как добавить теги индекса для управления жизненным циклом.":::

5. Выберите **Проверка и добавить** , чтобы проверить параметры правила.

  :::image type="content" source="media/storage-blob-index-concepts/blob-index-lifecycle-management-example.png" alt-text="Снимок экрана портал Azure показывающего правило управления жизненным циклом с использованием фильтра тегов индекса больших двоичных объектов":::

6. Выберите **Добавить**, чтобы применить новое правило к политике управления жизненным циклом.

# <a name="net"></a>[.NET](#tab/net)

Политики [управления жизненным циклом](storage-lifecycle-management-concepts.md) применяются к каждой учетной записи хранения на уровне управления. Для .NET установите [библиотеку хранилища Microsoft Azure Management](https://www.nuget.org/packages/Microsoft.Azure.Management.Storage/) 16.0.0 или более поздней версии.

---

## <a name="next-steps"></a>Дальнейшие действия

 - Дополнительные сведения о тегах индексов больших двоичных объектов см. в статье [Управление и поиск данных BLOB-объектов Azure с помощью тегов индекса BLOB (Предварительная версия)](storage-manage-find-blobs.md )
 - Дополнительные сведения об управлении жизненным циклом см. [в статье Управление жизненным циклом хранилища BLOB-объектов Azure](storage-lifecycle-management-concepts.md) .
