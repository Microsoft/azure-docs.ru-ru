---
title: Краткое руководство. Клиентская библиотека Хранилища BLOB-объектов Azure — Ruby
description: Создайте учетную запись хранения и контейнер в Хранилище BLOB-объектов Azure. Используйте клиентскую библиотеку хранилища для Ruby, чтобы создавать и загружать большие двоичные объекты и выводить список больших двоичных объектов в контейнере.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 12/04/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: ec3fc490466f5fce36b67b2f3744e4ee5cc0ae79
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96781102"
---
# <a name="quickstart-azure-blob-storage-client-library-for-ruby"></a>Краткое руководство. Клиентская библиотека Хранилища BLOB-объектов Azure для Ruby

Узнайте, как использовать Ruby для создания, загрузки и получения списка больших двоичных объектов в контейнере в Хранилище BLOB-объектов Microsoft Azure.

## <a name="prerequisites"></a>предварительные требования

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

Прежде чем приступить к работе, убедитесь, что у вас установлены следующие дополнительные компоненты:

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [Библиотека Службы хранилища Azure для Ruby](https://github.com/azure/azure-storage-ruby), которая устанавливается с помощью [пакета rubygem](https://rubygems.org/gems/azure-storage-blob).

    ```console
    gem install azure-storage-blob
    ```

## <a name="download-the-sample-application"></a>Загрузка примера приложения

[Пример приложения](https://github.com/Azure-Samples/storage-blobs-ruby-quickstart.git), используемый в этом кратком руководстве, представляет собой простое приложение Ruby.

Используйте сайт [Git](https://git-scm.com/), чтобы загрузить копию приложения в среду разработки. Эта команда клонирует репозиторий на локальный компьютер.

```console
git clone https://github.com/Azure-Samples/storage-blobs-ruby-quickstart.git
```

Перейдите в папку *storage-blobs-ruby-quickstart* и откройте файл *example.rb* в редакторе кода.

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Настройка строки подключения хранилища

Укажите имя и ключ учетной записи хранения, чтобы создать экземпляр [BlobService](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/BlobService) для приложения.

Следующий код в файле *example.rb* создает новый объект [BlobService](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/BlobService). Замените строки *accountname* и *accountkey* фактическими значениями имени и ключа учетной записи.

```ruby
# Create a BlobService object
account_name = "accountname"
account_key = "accountkey"

blob_client = Azure::Storage::Blob::BlobService.create(
    storage_account_name: account_name,
    storage_access_key: account_key
)
```

## <a name="run-the-sample"></a>Запуск примера

Этот пример создает контейнер в Хранилище BLOB-объектов, создает новый большой двоичный объект в контейнере, перечисляет большие двоичные объекты в контейнере и загружает большой двоичный объект в локальный файл.

Запустите образец. Ниже приведен пример результата запуска приложения.

```console
C:\azure-samples\storage-blobs-ruby-quickstart> ruby example.rb

Creating a container: quickstartblobs18cd9ec0-f4ac-4688-a979-75c31a70503e

Creating blob: QuickStart_6f8f29a8-879a-41fb-9db2-0b8595180728.txt

List blobs in the container following continuation token
        Blob name: QuickStart_6f8f29a8-879a-41fb-9db2-0b8595180728.txt

Downloading blob to C:/Users/azureuser/Documents/QuickStart_6f8f29a8-879a-41fb-9db2-0b8595180728.txt

Paused, press the Enter key to delete resources created by the sample and exit the application
```

Теперь нажмите клавишу ВВОД, чтобы пример программы удалил контейнер хранилища и локальный файл. Прежде чем продолжить, проверьте наличие загруженного файла в папке *Документы*.

Для просмотра файлов в учетной записи хранения можно также использовать [Обозреватель службы хранилища Azure](https://storageexplorer.com). Обозреватель службы хранилища Azure — это бесплатное кроссплатформенное средство для доступа к данным учетной записи хранения.

После проверки файлов нажмите клавишу ВВОД, чтобы завершить работу с демонстрационной версией и удалить тестовые файлы. Откройте файл *example.rb*, чтобы просмотреть код.

## <a name="understand-the-sample-code"></a>Разбор примера кода

Разберем пример кода, чтобы понять, как он работает.

### <a name="get-references-to-the-storage-objects"></a>Получение ссылок на объекты хранилища

Сначала необходимо создать экземпляры объектов, используемых для доступа к Хранилищу BLOB-объектов и управлению им. Эти объекты компилируются друг с другом. Каждый используется следующим в списке.

- Создайте экземпляр объекта [BlobService](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/BlobService) службы хранилища Azure, чтобы настроить учетные данные для подключения.
- Создайте объект [Container](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Container/Container), представляющий контейнер, к которому осуществляется доступ. Контейнеры используются для организации BLOB-объектов аналогично папкам для упорядочения файлов на компьютере.

Получив объект контейнера, можно создать BLOB-объект [Block](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Block), указывающий на конкретный BLOB-объект, который вас интересует. Используйте объект [Block](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Block) для создания, загрузки и копирования больших двоичных объектов.

> [!IMPORTANT]
> Имена контейнеров должны состоять из знаков нижнего регистра. Дополнительные сведения об именовании контейнеров и больших двоичных объектов см. в статье [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Именование контейнеров, больших двоичных объектов и метаданных и ссылка на них).

Следующий пример кода

- Создает новый контейнер
- Устанавливает разрешения на контейнер, чтобы большие двоичные объекты были общедоступными. Контейнер называется *quickstartblobs* и имеет уникальный идентификатор.

```ruby
# Create a container
container_name = "quickstartblobs" + SecureRandom.uuid
puts "\nCreating a container: " + container_name
container = blob_client.create_container(container_name)

# Set the permission so the blobs are public
blob_client.set_container_acl(container_name, "container")
```

### <a name="create-a-blob-in-the-container"></a>Создание большого двоичного объекта в контейнере

Хранилище BLOB-объектов поддерживает блочные, добавочные и страничные BLOB-объекты. Чтобы создать большой двоичный объект, вызовите метод [create_block_blob](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob#create_block_blob-instance_method), передав данные для большого двоичного объекта.

В следующем примере создается большой двоичный объект с именем *QuickStart_* , уникальным идентификатором и расширением файла *TXT* в контейнере, созданном ранее.

```ruby
# Create a new block blob containing 'Hello, World!'
blob_name = "QuickStart_" + SecureRandom.uuid + ".txt"
blob_data = "Hello, World!"
puts "\nCreating blob: " + blob_name
blob_client.create_block_blob(container.name, blob_name, blob_data)
```

Блочные BLOB-объекты могут иметь размер 4,7 ТБ и представлять собой любые объекты, начиная от электронных таблиц до больших видеофайлов. Страничные BLOB-объекты в основном используются для файлов VHD, применяемых для поддержки виртуальных машин IaaS. Добавочные BLOB-объекты широко используются для ведения журнала, например если требуется выполнить запись в файл и затем добавлять дополнительные сведения.

### <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Список файлов в контейнере можно получить с помощью метода [list_blobs](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Container#list_blobs-instance_method). Следующий код извлекает список больших двоичных объектов, а затем отображает их имена.

```ruby
# List the blobs in the container
puts "\nList blobs in the container following continuation token"
nextMarker = nil
loop do
    blobs = blob_client.list_blobs(container_name, { marker: nextMarker })
    blobs.each do |blob|
        puts "\tBlob name: #{blob.name}"
    end
    nextMarker = blobs.continuation_token
    break unless nextMarker && !nextMarker.empty?
end
```

### <a name="download-a-blob"></a>Загрузка больших двоичных объектов

Загрузите большой двоичный объект на локальный диск с помощью метода [get_blob](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob#get_blob-instance_method). Следующий код загружает BLOB-объект, который создан в предыдущем разделе.

```ruby
# Download the blob

# Set the path to the local folder for downloading
if(is_windows)
    local_path = File.expand_path("~/Documents")
else 
    local_path = File.expand_path("~/")
end

# Create the full path to the downloaded file
full_path_to_file = File.join(local_path, blob_name)

puts "\nDownloading blob to " + full_path_to_file
blob, content = blob_client.get_blob(container_name, blob_name)
File.open(full_path_to_file,"wb") {|f| f.write(content)}
```

### <a name="clean-up-resources"></a>Очистка ресурсов

Если большой двоичный объект больше не нужен, используйте метод [delete_blob](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob#delete_blob-instance_method), чтобы удалить его. Удалите весь контейнер с помощью метода [delete_container](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Container#delete_container-instance_method). При удалении контейнера также удаляются все большие двоичные объекты, хранящиеся в этом контейнере.

```ruby
# Clean up resources, including the container and the downloaded file
blob_client.delete_container(container_name)
File.delete(full_path_to_file)
```

## <a name="resources-for-developing-ruby-applications-with-blobs"></a>Ресурсы для разработки приложений Ruby с большими двоичными объектами

См. следующие дополнительные ресурсы по разработке с использованием Ruby:

- Просматривайте и скачивайте [исходный код клиентской библиотеки Ruby](https://github.com/Azure/azure-storage-ruby) для службы хранилища Azure на GitHub.
- Изучите [примеры Azure](/samples/browse/?products=azure&languages=ruby), написанные с использованием клиентской библиотеки Ruby.
- [Пример: Начало работы со службой хранилища Azure в Ruby](https://github.com/Azure-Samples/storage-blob-ruby-getting-started)

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как передавать файлы между хранилищем BLOB-объектов Azure и локальным диском с помощью Ruby. Дополнительные сведения о работе с хранилищем BLOB-объектов см. в обзоре учетной записи хранения.

> [!div class="nextstepaction"]
> [Общие сведения об учетной записи хранения](../common/storage-account-overview.md)

Дополнительные сведения об обозревателе объектов и BLOB-объектах см. в статье [Управление ресурсами хранилища BLOB-объектов Azure с помощью Обозревателя службы хранилищ](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
