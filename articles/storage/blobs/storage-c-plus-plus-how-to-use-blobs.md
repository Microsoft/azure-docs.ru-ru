---
title: Как использовать хранилище BLOB-объектов из С++ в Azure | Документация Майкрософт
description: Узнайте, как хранить неструктурированные данные (большие двоичные объекты) в облаке с использованием хранилища больших двоичных объектов (объектов) Azure с помощью C++.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/16/2020
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.openlocfilehash: 64069292ea0059216d06bfc41316c2aed7484dd0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96011104"
---
# <a name="how-to-use-blob-storage-from-c"></a>Использование хранилища BLOB-объектов из C++

В этом руководстве показано, как реализовать типичные сценарии с использованием хранилища BLOB-объектов Azure. В примерах показано, как загружать, удалять BLOB-объекты и получать список этих объектов. Примеры написаны на C++ и используют [клиентскую библиотеку хранилища Azure для C++](https://github.com/Azure/azure-storage-cpp/blob/master/README.md).
Дополнительные сведения см. в статье [Общие сведения о хранилище BLOB-объектов Azure](storage-blobs-introduction.md).

> [!NOTE]
> Данное руководство предназначено для клиентской библиотеки хранилища Azure для С++ версии 1.0.0 и выше. Корпорация Майкрософт рекомендует использовать последнюю версию клиентской библиотеки хранилища Azure для С++, которая доступна в [NuGet](https://www.nuget.org/packages/wastorage) или [GitHub](https://github.com/Azure/azure-storage-cpp).

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-c-application"></a>Создание приложения на C++
В этом руководстве будут использоваться компоненты хранилища, которые могут выполняться в приложениях C++.

Для этого необходимо установить клиентскую библиотеку хранилища Azure для C++ и создать учетную запись хранения Azure в подписке Azure.

Чтобы установить клиентскую библиотеку хранилища для C++, можно использовать следующие методы.

- **Linux:** следуйте инструкциям, указанным в файле README [клиентской библиотеки хранилища Azure для C++ на странице для начала работы в Linux](https://github.com/Azure/azure-storage-cpp#getting-started-on-linux).
- **Windows:** В Windows в качестве диспетчера зависимостей используйте [vcpkg](https://github.com/microsoft/vcpkg). Выполните инструкции из [краткого руководства](https://github.com/microsoft/vcpkg#quick-start) , чтобы инициализировать vcpkg. Затем, чтобы установить библиотеку, используйте следующую команду:

```powershell
.\vcpkg.exe install azure-storage-cpp
```

Руководство по созданию исходного кода и экспорту в NuGet можно найти в файле [сведений](https://github.com/Azure/azure-storage-cpp#download--install) .

## <a name="configure-your-application-to-access-blob-storage"></a>Настройка приложения для доступа к хранилищу больших двоичных объектов
Добавьте следующие инструкции в начало файла C++, где требуется использовать API-интерфейсы Azure для доступа к BLOB-объектам.

```cpp
#include <was/storage_account.h>
#include <was/blob.h>
#include <cpprest/filestream.h>  
#include <cpprest/containerstream.h> 
```

## <a name="setup-an-azure-storage-connection-string"></a>Настройка строки подключения к службе хранилища Azure
Клиент хранилища Azure использует строку подключения с целью хранения конечных точек и учетных данных для доступа к службам управления данными. При запуске в клиентском приложении необходимо указать строку подключения к хранилищу в следующем формате, используя имя учетной записи хранения и ключ доступа к хранилищу для учетной записи хранения, указанной в [портал Azure](https://portal.azure.com) для значений *AccountName* и *AccountKey* . Сведения о учетных записях хранения и ключах доступа см. в статье [об учетных записях хранения Azure](../common/storage-account-create.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json). В этом примере показано, как объявить статическое поле для размещения строки подключения:

```cpp
// Define the connection-string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key"));
```

Чтобы протестировать приложение на локальном компьютере Windows, можно использовать [эмулятор хранения азурите](../common/storage-use-azurite.md). Азурите — это служебная программа, имитирующая службы BLOB-объектов и очередей, доступные в Azure на локальном компьютере разработки. В следующем примере показано, как объявить статическое поле для размещения строки подключения для эмулятора локального хранилища.

```cpp
// Define the connection-string with Azurite.
const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));
```

Сведения о запуске Азурите см. в статье [использование эмулятора азурите для локальной разработки службы хранилища Azure](../common/storage-use-azurite.md).

В приведенных ниже примерах предполагается, что вы использовали одно из этих двух определений для получения строки подключения к хранилищу.

## <a name="retrieve-your-storage-account"></a>Получение учетной записи хранения
Для представления сведений об учетной записи хранения можно использовать класс **cloud_storage_account** . Чтобы получить данные учетной записи хранения из строки подключения хранилища, можно использовать метод **синтаксического анализа** .

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);
```

Затем получите ссылку на класс **cloud_blob_client** для извлечения объектов, представляющих контейнеры и BLOB-объекты, которые хранятся в службе хранилища BLOB-объектов. Следующий код создает объект **cloud_blob_client** с помощью объекта учетной записи хранения, полученной выше:

```cpp
// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();  
```

## <a name="how-to-create-a-container"></a>Практическое руководство. Создание контейнера
[!INCLUDE [storage-container-naming-rules-include](../../../includes/storage-container-naming-rules-include.md)]

В этом примере показано, как создать контейнер:

```cpp
try
{
    // Retrieve storage account from connection string.
    azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

    // Create the blob client.
    azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

    // Retrieve a reference to a container.
    azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

    // Create the container if it doesn't already exist.
    container.create_if_not_exists();
}
catch (const std::exception& e)
{
    std::wcout << U("Error: ") << e.what() << std::endl;
}  
```

По умолчанию новый контейнер закрытый, и вам необходимо указать ключ доступа к хранилищу, чтобы загрузить BLOB-объекты из этого контейнера. Чтобы сделать файлы (BLOB) в этом контейнере доступными для всех пользователей, сделайте контейнер открытым, используя следующий код.

```cpp
// Make the blob container publicly accessible.
azure::storage::blob_container_permissions permissions;
permissions.set_public_access(azure::storage::blob_container_public_access_type::blob);
container.upload_permissions(permissions);  
```

Любой пользователь в Интернете может видеть большие двоичные объекты в открытом контейнере, но изменить или удалить их можно только при наличии ключа доступа.

## <a name="how-to-upload-a-blob-into-a-container"></a>Практическое руководство. Отправка BLOB-объекта в контейнер
Хранилище больших двоичных объектов Azure поддерживает блочные и страничные большие двоичные объекты. В большинстве случаев рекомендуется использовать блочные BLOB-объекты.

Для передачи файла в блочный BLOB-объект получите ссылку на контейнер и используйте ее для получения ссылки на блочный BLOB-объект. Получив ссылку на BLOB-объект, можно отправить любой поток данных в этот объект с помощью метода **upload_from_stream**. Эта операция создает большой двоичный объект, если он не существует, или заменяет его, если он существует. В следующем примере показано, как отправить BLOB-объект в контейнер. Предполагается, что контейнер уже был создан.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Retrieve reference to a blob named "my-blob-1".
azure::storage::cloud_block_blob blockBlob = container.get_block_blob_reference(U("my-blob-1"));

// Create or overwrite the "my-blob-1" blob with contents from a local file.
concurrency::streams::istream input_stream = concurrency::streams::file_stream<uint8_t>::open_istream(U("DataFile.txt")).get();
blockBlob.upload_from_stream(input_stream);
input_stream.close().wait();

// Create or overwrite the "my-blob-2" and "my-blob-3" blobs with contents from text.
// Retrieve a reference to a blob named "my-blob-2".
azure::storage::cloud_block_blob blob2 = container.get_block_blob_reference(U("my-blob-2"));
blob2.upload_text(U("more text"));

// Retrieve a reference to a blob named "my-blob-3".
azure::storage::cloud_block_blob blob3 = container.get_block_blob_reference(U("my-directory/my-sub-directory/my-blob-3"));
blob3.upload_text(U("other text"));  
```

Кроме того, можно использовать метод **upload_from_file** для отправки файла в блочный BLOB-объект.

## <a name="how-to-list-the-blobs-in-a-container"></a>Практическое руководство. Перечисление BLOB-объектов в контейнере
Для перечисления BLOB-объектов в контейнере сначала необходимо получить ссылку на контейнер. Затем можно использовать метод **list_blobs** контейнера для извлечения больших двоичных объектов и (или) каталогов внутри него. Для доступа к широкому набору свойств и методов возвращаемого объекта **list_blob_item** необходимо вызвать метод **list_blob_item.as_blob**, чтобы получить объект **cloud_blob**, или метод **list_blob.as_directory**, чтобы получить объект cloud_blob_directory. В следующем коде показано, как получить и вывести URI каждого элемента в контейнере **my-sample-container** .

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Output URI of each item.
azure::storage::list_blob_item_iterator end_of_results;
for (auto it = container.list_blobs(); it != end_of_results; ++it)
{
    if (it->is_blob())
    {
        std::wcout << U("Blob: ") << it->as_blob().uri().primary_uri().to_string() << std::endl;
    }
    else
    {
        std::wcout << U("Directory: ") << it->as_directory().uri().primary_uri().to_string() << std::endl;
    }
}
```

Дополнительные сведения об операциях перечисления см. в разделе [Перечисление ресурсов хранилища Azure в C++](../common/storage-c-plus-plus-enumeration.md).

## <a name="how-to-download-blobs"></a>Практическое руководство. Загрузка BLOB-объектов
Чтобы скачать большие двоичные объекты, сначала извлеките ссылку на большой двоичный объект, а затем вызовите метод **download_to_stream** . В следующем примере используется метод **download_to_stream** для передачи содержимого большого двоичного объекта в объект потока, который затем можно сохранить в локальном файле.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Retrieve reference to a blob named "my-blob-1".
azure::storage::cloud_block_blob blockBlob = container.get_block_blob_reference(U("my-blob-1"));

// Save blob contents to a file.
concurrency::streams::container_buffer<std::vector<uint8_t>> buffer;
concurrency::streams::ostream output_stream(buffer);
blockBlob.download_to_stream(output_stream);

std::ofstream outfile("DownloadBlobFile.txt", std::ofstream::binary);
std::vector<unsigned char>& data = buffer.collection();

outfile.write((char *)&data[0], buffer.size());
outfile.close();  
```

Кроме того, можно использовать метод **download_to_file**, чтобы загрузить содержимое BLOB-объекта в файл.
Также можно использовать метод **download_text**, чтобы загрузить содержимое BLOB-объекта в виде текстовой строки.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Retrieve reference to a blob named "my-blob-2".
azure::storage::cloud_block_blob text_blob = container.get_block_blob_reference(U("my-blob-2"));

// Download the contents of a blog as a text string.
utility::string_t text = text_blob.download_text();
```

## <a name="how-to-delete-blobs"></a>Практическое руководство. Удаление BLOB-объектов
Чтобы удалить BLOB-объект, сначала нужно получить ссылку на него, а затем вызвать для него метод **delete_blob**.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Retrieve reference to a blob named "my-blob-1".
azure::storage::cloud_block_blob blockBlob = container.get_block_blob_reference(U("my-blob-1"));

// Delete the blob.
blockBlob.delete_blob();
```

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда вы ознакомились с основными сведениями о хранилище BLOB-объектов, используйте следующие ссылки для получения дополнительных сведений о хранилище Azure.

- [Использование хранилища очередей из C++](../queues/storage-c-plus-plus-how-to-use-queues.md)
- [Использование табличного хранилища из C++](../../cosmos-db/table-storage-how-to-use-c-plus.md)
- [Список ресурсов службы хранилища Azure в C++](../common/storage-c-plus-plus-enumeration.md)
- [Справочник по клиентской библиотеке хранилища для C++](https://azure.github.io/azure-storage-cpp)
- [Документация по хранилищу Azure](https://azure.microsoft.com/documentation/services/storage/)
- [Перенос данных с помощью служебной программы командной строки AzCopy](../common/storage-use-azcopy-v10.md)