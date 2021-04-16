---
title: Краткое руководство. Библиотека Хранилища BLOB-объектов Azure версии 12 для Xamarin
description: Из этого краткого руководства вы узнаете, как использовать клиентскую библиотеку службы "Хранилище BLOB-объектов Azure" версии 12 для Xamarin для создания контейнера и большого двоичного объекта в хранилище BLOB-объектов. Далее вы узнаете, как скачать большой двоичный объект на мобильное устройство и как получить список всех больших двоичных объектов в контейнере.
author: codemillmatt
ms.author: masoucou
ms.date: 10/09/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: c6b502bbc56d6c9521ec736150fbc0046ff78125
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105642479"
---
# <a name="quickstart-azure-blob-storage-client-library-v12-with-xamarin"></a>Краткое руководство. Использование клиентской библиотеки Хранилища BLOB-объектов Azure версии 12 для Xamarin

Приступите к работе с клиентской библиотекой Хранилища BLOB-объектов Azure версии 12 для Xamarin. Хранилище BLOB-объектов Azure — это решение корпорации Майкрософт для хранения объектов в облаке. Чтобы установить пакет и испробовать пример кода для выполнения базовых задач, выполните приведенные здесь действия. Хранилище BLOB-объектов оптимизировано для хранения больших объемов неструктурированных данных.

С помощью клиентской библиотеки Хранилища BLOB-объектов Azure версии 12 для Xamarin вы можете выполнять следующие действия:

* Создание контейнера
* передача большого двоичного объекта в хранилище Azure;
* перечисление всех больших двоичных объектов в контейнере;
* скачивание большого двоичного объекта на устройство;
* Удаление контейнера

Ссылки для справки:

* [Справочная документация по API](/dotnet/api/azure.storage.blobs)
* [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Blobs)
* [Пакет (NuGet)](https://www.nuget.org/packages/Azure.Storage.Blobs)
* [Образец](https://github.com/Azure-Samples/storage-blobs-xamarin-quickstart)

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
* Учетная запись хранения Azure — [создайте такую учетную запись](../common/storage-account-create.md).
* Visual Studio с установленной рабочей нагрузкой [Mobile Development for .NET](/xamarin/get-started/installation/?pivots=windows) (Разработка мобильных приложений для .NET) или [Visual Studio для Mac](/visualstudio/mac/installation?view=vsmac-2019&preserve-view=true).

## <a name="setting-up"></a>Настройка
    
В этом разделе рассматривается подготовка проекта для работы с клиентской библиотекой Хранилища BLOB-объектов Azure версии 12 для Xamarin.
    
### <a name="create-the-project"></a>Создание проекта

1. Откройте Visual Studio и создайте пустое приложение Forms.
1. Присвойте ему имя BlobQuickstartV12.

### <a name="install-the-package"></a>Установка пакета

1. В области обозревателя решений щелкните правой кнопкой мыши свое решение и выберите пункт **Управление пакетами NuGet для решения**.
1. Выполните поиск по строке **Azure.Storage.Blobs** и установите последнюю стабильную версию во все проекты в решении.

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

В каталоге **BlobQuickstartV12** сделайте следующее:

1. Открытие в любом редакторе файл *MainPage.xaml*.
1. Удалите все содержимое между элементами `<ContentPage></ContentPage>` и вставьте вместо него следующий код:

```xaml
<StackLayout HorizontalOptions="Center" VerticalOptions="Center">

    <Button x:Name="uploadButton" Text="Upload Blob" Clicked="Upload_Clicked"  IsEnabled="False"/>
    <Button x:Name="listButton" Text="List Blobs" Clicked="List_Clicked"  IsEnabled="False" />
    <Button x:Name="downloadButton" Text="Download Blob" Clicked="Download_Clicked"  IsEnabled="False" />
    <Button x:Name="deleteButton" Text="Delete Container" Clicked="Delete_Clicked" IsEnabled="False" />

    <Label Text="" x:Name="resultsLabel" HorizontalTextAlignment="Center" Margin="0,20,0,0" TextColor="Red" />
        
</StackLayout>
```

[!INCLUDE [storage-quickstart-credentials-xamarin-include](../../../includes/storage-quickstart-credentials-xamarin-include.md)]

## <a name="object-model"></a>Объектная модель

Хранилище BLOB-объектов Azure оптимизировано для хранения больших объемов неструктурированных данных. Неструктурированные данные — это данные, которые не соответствуют определенной модели данных или определению, например текстовых или двоичных данных. В хранилище BLOB-объектов предлагается три типа ресурсов:

* учетная запись хранения;
* контейнер в учетной записи хранения;
* большой двоичный объект в контейнере.

На следующей схеме показана связь между этими ресурсами.

![Схема архитектуры службы хранилища BLOB-объектов](./media/storage-blobs-introduction/blob1.png)

Используйте следующие классы .NET для взаимодействия с этими ресурсами:

* [BlobServiceClient](/dotnet/api/azure.storage.blobs.blobserviceclient). Класс `BlobServiceClient` позволяет управлять ресурсами службы хранилища Azure и контейнерами больших двоичных объектов.
* [BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient). Класс `BlobContainerClient` позволяет управлять контейнерами службы хранилища Azure и содержащимися в них большими двоичными объектами.
* [BlobClient](/dotnet/api/azure.storage.blobs.blobclient). Класс `BlobClient` позволяет управлять большими двоичными объектами службы хранилища Azure.
* [BlobDownloadInfo](/dotnet/api/azure.storage.blobs.models.blobdownloadinfo). Класс `BlobDownloadInfo` представляет свойства и содержимое, возвращаемое при скачивании большого двоичного объекта.

## <a name="code-examples"></a>Примеры кода

В этих фрагментах кода показано, как в приложении Xamarin.Forms выполнять следующие действия с помощью клиентской библиотеки Хранилища BLOB-объектов Azure для .NET:

* [создание переменных уровня класса](#create-class-level-variables);
* [Создание контейнера](#create-a-container)
* [отправка больших двоичных объектов в контейнер](#upload-blobs-to-a-container);
* [перечисление больших двоичных объектов в контейнере](#list-the-blobs-in-a-container);
* [скачивание больших двоичных объектов](#download-blobs);
* [Удаление контейнера](#delete-a-container)

### <a name="create-class-level-variables"></a>Создание переменных уровня класса

В следующем коде объявлено несколько переменных уровня класса. Они нужны для последующего взаимодействия с Хранилищем BLOB-объектов Azure в нашем примере.

Эти данные дополняют строку подключения к учетной записи хранения, которую мы настроили в разделе [Настройка строки подключения хранилища](#configure-your-storage-connection-string).

Добавьте этот код в качестве переменных уровня класса в файл *MainPage.xaml.cs*:

```csharp
string storageConnectionString = "{set in the Configure your storage connection string section}";
string fileName = $"{Guid.NewGuid()}-temp.txt";

BlobServiceClient client;
BlobContainerClient containerClient;
BlobClient blobClient;
```

### <a name="create-a-container"></a>Создание контейнера

Выберите имя нового контейнера. Приведенный ниже код добавляет к имени контейнера значение GUID, чтобы сделать это имя уникальным.

> [!IMPORTANT]
> Имена контейнеров должны состоять из знаков нижнего регистра. Дополнительные сведения об именовании контейнеров и больших двоичных объектов см. в статье [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Именование контейнеров, больших двоичных объектов и метаданных и ссылка на них).

Создайте экземпляр класса [BlobServiceClient](/dotnet/api/azure.storage.blobs.blobserviceclient). Затем вызовите метод [CreateBlobContainerAsync](/dotnet/api/azure.storage.blobs.blobserviceclient.createblobcontainerasync), чтобы создать контейнер в своей учетной записи хранения.

Добавьте следующий код в файл *MainPage.xaml.cs*.

```csharp
protected async override void OnAppearing()
{            
    string containerName = $"quickstartblobs{Guid.NewGuid()}";
    
    client = new BlobServiceClient(storageConnectionString);
    containerClient = await client.CreateBlobContainerAsync(containerName);

    resultsLabel.Text = "Container Created\n";

    blobClient = containerClient.GetBlobClient(fileName);

    uploadButton.IsEnabled = true;
}
```

### <a name="upload-blobs-to-a-container"></a>Отправка больших двоичных объектов в контейнер

Приведенный ниже фрагмент кода:

1. Создает `MemoryStream` с текстом.
1. Передает этот текст в большой двоичный объект, вызывая функцию [UploadAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.uploadblobasync#Azure_Storage_Blobs_BlobContainerClient_UploadBlobAsync_System_String_System_IO_Stream_System_Threading_CancellationToken_) из класса [BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient) и передавая ей имя файла, определенное в переменной уровня класса, и `MemoryStream` с текстом. С помощью этого метода создается большой двоичный объект, если он не был создан ранее, или же, если он имеется, происходит его замещение.

Добавьте следующий код в файл *MainPage.xaml.cs*.

```csharp
async void Upload_Clicked(object sender, EventArgs e)
{                                    
    using MemoryStream memoryStream = new MemoryStream(Encoding.UTF8.GetBytes("Hello World!"));

    await containerClient.UploadBlobAsync(fileName, memoryStream);

    resultsLabel.Text += "Blob Uploaded\n";

    uploadButton.IsEnabled = false;
    listButton.IsEnabled = true;
}
```

### <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Выведите список больших двоичных объектов в контейнере, вызвав метод [GetBlobsAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsasync). В этом случае в контейнер был добавлен лишь один большой двоичный объект, поэтому операция перечисления возвращает только его.

Добавьте следующий код в файл *MainPage.xaml.cs*.

```csharp
async void List_Clicked(object sender, EventArgs e)
{            
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        resultsLabel.Text += blobItem.Name + "\n";                
    }

    listButton.IsEnabled = false;
    downloadButton.IsEnabled = true;
}
```

### <a name="download-blobs"></a>Скачивание больших двоичных объектов

Скачайте созданный ранее BLOB-объект, вызвав метод [DownloadToAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.downloadtoasync). В этом примере кода сначала копируется представление `Stream` большого двоичного объекта в `MemoryStream`, а затем в `StreamReader` для отображения текста.

Добавьте следующий код в файл *MainPage.xaml.cs*.

```csharp
async void Download_Clicked(object sender, EventArgs e)
{
    BlobDownloadInfo downloadInfo = await blobClient.DownloadAsync();

    using MemoryStream memoryStream = new MemoryStream();
    
    await downloadInfo.Content.CopyToAsync(memoryStream);
    memoryStream.Position = 0;

    using StreamReader streamReader = new StreamReader(memoryStream);

    resultsLabel.Text += "Blob Contents: \n";
    resultsLabel.Text += await streamReader.ReadToEndAsync();
    resultsLabel.Text += "\n";

    downloadButton.IsEnabled = false;
    deleteButton.IsEnabled = true;
}
```

### <a name="delete-a-container"></a>Удаление контейнера

Следующий код очищает созданные приложением ресурсы, полностью удаляя контейнер с помощью метода [DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteasync).

Это приложение запрашивает подтверждение перед удалением BLOB-объекта и контейнера. Это хорошая возможность проверить правильность создания ресурсов перед их удалением.

Добавьте следующий код в файл *MainPage.xaml.cs*.

```csharp
async void Delete_Clicked(object sender, EventArgs e)
{            
    var deleteContainer = await Application.Current.MainPage.DisplayAlert("Delete Container",
        "You are about to delete the container proceeed?", "OK", "Cancel");

    if (deleteContainer == false)
        return;

    await containerClient.DeleteAsync();

    resultsLabel.Text += "Container Deleted";

    deleteButton.IsEnabled = false;
}
```

## <a name="run-the-code"></a>Выполнение кода

При запуске приложение сначала создаст контейнер и отобразит его. Затем вы с помощью соответствующих кнопок отправите, перечислите, скачаете и удалите BLOB-объекты.

Чтобы запустить приложение в среде Windows, нажмите клавишу F5. Чтобы запустить приложение на компьютере Mac, нажмите клавиши Cmd+ВВОД.

После каждой операции приложение выводит данные на экран. Выходные данные приложения выглядят примерно так:

```output
Container Created
Blob Uploaded
98d9a472-8e98-4978-ba4f-081d69d2e6f8-temp.txt
Blob Contents:
Hello World!
Container Deleted
```

Перед началом процесса очистки убедитесь, что выходные данные на экране с содержимым большого двоичного объекта соответствуют переданному значению.

После проверки значений подтвердите готовность удалить контейнер и завершить работу демонстрационного приложения.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как использовать Xamarin и клиентскую библиотеку Хранилища BLOB-объектов Azure версии 12 для передачи, скачивания и вывода BLOB-объектов.

Чтобы просмотреть примеры приложений для хранилища BLOB-объектов, перейдите к следующему разделу:

> [!div class="nextstepaction"]
> [Примеры для пакета SDK Хранилища BLOB-объектов Azure версии 12 для Xamarin](https://github.com/Azure-Samples/storage-blobs-xamarin-quickstart)

* Учебные материалы, примеры, краткие руководства и другую документацию можно найти на странице [Azure для разработчиков мобильных приложений](/azure/mobile-apps).
* Дополнительные сведения о Xamarin см. в статье [Начало работы с Xamarin](/xamarin/get-started/).
