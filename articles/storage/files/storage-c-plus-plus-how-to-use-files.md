---
title: Разработка приложений службы файлов Azure с помощью C++ | Документация Майкрософт
description: Сведения о разработке приложений и служб C++, хранящих файлы данных в службе файлов Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 09/19/2017
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: bb74ab16e51fbb3a157757353d5743e889f993dd
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94629705"
---
# <a name="develop-for-azure-files-with-c"></a>Разработка приложений службы файлов Azure с помощью C++

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

[!INCLUDE [storage-try-azure-tools-files](../../../includes/storage-try-azure-tools-files.md)]

## <a name="about-this-tutorial"></a>О данном учебнике

В рамках этого руководства вы узнаете, как выполнять основные операции в службе файлов Azure. С помощью примеров на языке C++ вы узнаете, как создавать общие папки и каталоги, отправлять и удалять файлы, а также получать список файлов. Если вы не знакомы со службой файлов Azure, мы предлагаем вам изучить понятия, описанные в последующих разделах. Это будет полезно для понимания примеров.

* Создание и удаление общих папок Azure.
* Создание и удаление каталогов.
* Перечисление файлов и каталогов в общей папке Azure
* Передача, загрузка и удаление файлов.
* Установка квоты (максимального размера) для общей папки Azure.
* Создайте подпись общего доступа (ключ SAS) для файла, который использует политику общего доступа, определенную в общей папке.

> [!Note]  
> Так как к службе файлов Azure можно обращаться через SMB, вы можете создавать простые приложения, которые получают доступ к общей папке Azure с использованием стандартных классов и функций ввода-вывода в C++. Из этой статьи вы узнаете, как создавать приложения, использующие пакет SDK C++ для службы хранилища Azure, который использует [REST API службы файлов](/rest/api/storageservices/file-service-rest-api) для взаимодействия со службой файлов Azure.

## <a name="create-a-c-application"></a>Создание приложения на C++

Для создания примеров необходимо установить клиентскую библиотеку хранилища Azure версии 2.4.0 для C++. Вам также необходимо создать учетную запись хранилища Azure.

Чтобы установить клиентскую библиотеку хранилища Azure версии 2.4.0 для C++, можно использовать следующие методы:

* **Linux:** следуйте инструкциям, указанным в файле README [клиентской библиотеки хранилища Azure для C++](https://github.com/Azure/azure-storage-cpp/blob/master/README.md) .
* **Windows.** В Visual Studio щелкните **Инструменты &gt; Диспетчер пакетов NuGet** Консоль диспетчера пакетов&gt;. Введите следующую команду в [консоли диспетчера пакетов NuGet](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) и нажмите клавишу **ВВОД**.
  

```powershell
Install-Package wastorage
```

## <a name="set-up-your-application-to-use-azure-files"></a>Настройка приложения для работы со службой файлов Azure

Добавьте следующие инструкции include в начало исходного файла C++, в котором вам нужен доступ к службе файлов Azure:

```cpp
#include <was/storage_account.h>
#include <was/file.h>
```

## <a name="set-up-an-azure-storage-connection-string"></a>Настройка строки подключения к хранилищу Azure

Чтобы начать работу с хранилищем файлов, необходимо подключиться к вашей учетной записи хранилища Azure. Для начала потребуется настроить строку подключения, которая будет использоваться для подключения к учетной записи хранения. Для этого определим статическую переменную.

```cpp
// Define the connection-string with your values.
const utility::string_t
storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key"));
```

## <a name="connecting-to-an-azure-storage-account"></a>Подключение к учетной записи хранилища Azure

Для представления сведений об учетной записи хранения можно использовать класс **cloud_storage_account** . Чтобы получить данные учетной записи хранения из строки подключения хранилища, можно использовать метод **синтаксического анализа** .

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account =
  azure::storage::cloud_storage_account::parse(storage_connection_string);
```

## <a name="create-an-azure-file-share"></a>Создание файлового ресурса Azure

Все файлы и каталоги в службе файлов Azure находятся в контейнере, который называется **общей папкой**. Учетная запись хранения может иметь столько общих папок, насколько позволяет емкость вашей учетной записи. Чтобы получить доступ к общей папке и ее содержимому, необходимо использовать клиент файлов Azure.

```cpp
// Create the Azure Files client.
azure::storage::cloud_file_client file_client =
  storage_account.create_cloud_file_client();
```

С его помощью вы получите ссылку на общую папку.

```cpp
// Get a reference to the file share
azure::storage::cloud_file_share share =
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));
```

Чтобы создать общий ресурс, используйте метод **create_if_not_exists** объекта **cloud_file_share**.

```cpp
if (share.create_if_not_exists()) {
    std::wcout << U("New share created") << std::endl;
}
```

На этом этапе объект **share** содержит ссылку на общую папку с именем **my-sample-share**.

## <a name="delete-an-azure-file-share"></a>Удаление общей папки Azure

Удаление общей папки осуществляется путем вызова метода **delete_if_exists** для объекта cloud_file_share. Ниже приведен пример кода, который выполняет это действие.

```cpp
// Get a reference to the share.
azure::storage::cloud_file_share share =
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));

// delete the share if exists
share.delete_share_if_exists();
```

## <a name="create-a-directory"></a>Создание каталога

Вы можете упорядочить файлы в хранилище, разместив их в подкаталогах, чтобы не захламлять корневой каталог. Служба файлов Azure позволяет создать такое количество каталогов, которое допускается в вашей учетной записи. В следующем примере кода создается каталог с именем **my-sample-directory** в корневом каталоге, а также подкаталог **my-sample-subdirectory**.

```cpp
// Retrieve a reference to a directory
azure::storage::cloud_file_directory directory = share.get_directory_reference(_XPLATSTR("my-sample-directory"));

// Return value is true if the share did not exist and was successfully created.
directory.create_if_not_exists();

// Create a subdirectory.
azure::storage::cloud_file_directory subdirectory =
  directory.get_subdirectory_reference(_XPLATSTR("my-sample-subdirectory"));
subdirectory.create_if_not_exists();
```

## <a name="delete-a-directory"></a>Удаление каталога

Удаление каталога является простой задачей, однако следует отметить, что нельзя удалить каталог, по-прежнему содержащий файлы или другие каталоги.

```cpp
// Get a reference to the share.
azure::storage::cloud_file_share share =
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));

// Get a reference to the directory.
azure::storage::cloud_file_directory directory =
  share.get_directory_reference(_XPLATSTR("my-sample-directory"));

// Get a reference to the subdirectory you want to delete.
azure::storage::cloud_file_directory sub_directory =
  directory.get_subdirectory_reference(_XPLATSTR("my-sample-subdirectory"));

// Delete the subdirectory and the sample directory.
sub_directory.delete_directory_if_exists();

directory.delete_directory_if_exists();
```

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>Перечисление файлов и каталогов в общей папке Azure

Получить список файлов и каталогов в общей папке довольно просто, вызвав метод **list_files_and_directorie** по ссылке **cloud_file_directory**. Для доступа к широкому набору свойств и методов возвращаемого объекта **list_file_and_directory_item** необходимо вызвать метод **list_file_and_directory_item.as_file**, чтобы получить объект **cloud_file**, или метод **list_file_and_directory_item.as_directory**, чтобы получить объект **cloud_file_directory**.

Следующий код демонстрирует, как получить и вывести URI каждого элемента в корневом каталоге общей папки.

```cpp
//Get a reference to the root directory for the share.
azure::storage::cloud_file_directory root_dir =
  share.get_root_directory_reference();

// Output URI of each item.
azure::storage::list_file_and_directory_result_iterator end_of_results;

for (auto it = directory.list_files_and_directories(); it != end_of_results; ++it)
{
    if(it->is_directory())
    {
        ucout << "Directory: " << it->as_directory().uri().primary_uri().to_string() << std::endl;
    }
    else if (it->is_file())
    {
        ucout << "File: " << it->as_file().uri().primary_uri().to_string() << std::endl;
    }
}
```

## <a name="upload-a-file"></a>Отправка файла

Общая папка Azure содержит по меньшей мере один каталог для размещения файлов (корневой каталог). В этом разделе вы узнаете, как отправить файл из локального хранилища в корневой каталог общего ресурса.

Первым шагом при отправке файла является получение ссылки на каталог, где файл будет находиться. Это делается путем вызова метода **get_root_directory_reference** объекта общего доступа.

```cpp
//Get a reference to the root directory for the share.
azure::storage::cloud_file_directory root_dir = share.get_root_directory_reference();
```

Теперь, когда у вас имеется ссылка на корневой каталог общего ресурса, вы можете отправить туда файл. В этом примере выполняется отправка из файла, текста и потока.

```cpp
// Upload a file from a stream.
concurrency::streams::istream input_stream =
  concurrency::streams::file_stream<uint8_t>::open_istream(_XPLATSTR("DataFile.txt")).get();

azure::storage::cloud_file file1 =
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-1"));
file1.upload_from_stream(input_stream);

// Upload some files from text.
azure::storage::cloud_file file2 =
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-2"));
file2.upload_text(_XPLATSTR("more text"));

// Upload a file from a file.
azure::storage::cloud_file file4 =
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-3"));
file4.upload_from_file(_XPLATSTR("DataFile.txt"));
```

## <a name="download-a-file"></a>Загрузка файла

Чтобы скачать файлы, сначала извлеките ссылку на файл, а затем вызовите метод **download_to_stream** , чтобы передать содержимое файла в объект потока, который затем можно сохранить в локальный файл. Кроме того, можно использовать метод **download_to_file**, чтобы скачать содержимое файла в локальный файл. Вы можете также использовать метод **download_text**, чтобы скачать содержимое файла в виде текстовой строки.

В следующем примере используются методы **download_to_stream** и **download_text** для демонстрации загрузки файлов, которые были созданы в предыдущих разделах.

```cpp
// Download as text
azure::storage::cloud_file text_file =
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-2"));
utility::string_t text = text_file.download_text();
ucout << "File Text: " << text << std::endl;

// Download as a stream.
azure::storage::cloud_file stream_file =
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-3"));

concurrency::streams::container_buffer<std::vector<uint8_t>> buffer;
concurrency::streams::ostream output_stream(buffer);
stream_file.download_to_stream(output_stream);
std::ofstream outfile("DownloadFile.txt", std::ofstream::binary);
std::vector<unsigned char>& data = buffer.collection();
outfile.write((char *)&data[0], buffer.size());
outfile.close();
```

## <a name="delete-a-file"></a>Удаление файла

Другая распространенная операция в службе файлов Azure — это удаление файлов. Следующий код удаляет файл с именем my-sample-file-3, который хранится в корневом каталоге.

```cpp
// Get a reference to the root directory for the share.
azure::storage::cloud_file_share share =
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));

azure::storage::cloud_file_directory root_dir =
  share.get_root_directory_reference();

azure::storage::cloud_file file =
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-3"));

file.delete_file_if_exists();
```

## <a name="set-the-quota-maximum-size-for-an-azure-file-share"></a>Установка квоты (максимального размера) для общей папки Azure.

Для файлового ресурса можно установить квоту (или максимальный размер) в гигабайтах. Можно также проверить, какой объем данных хранится в настоящее время в общей папке.

Задав квоту для файлового ресурса, можно ограничить общий размер файлов, хранящихся в общей папке. Если общий размер файлов в файловом ресурсе превышает установленную квоту, клиенты не смогут увеличить размер существующих файлов или создать новые файлы, только если они не являются пустыми.

В приведенном ниже примере показано, как проверить текущее использование данных в файловом ресурсе, а также задать для него квоту.

```cpp
// Parse the connection string for the storage account.
azure::storage::cloud_storage_account storage_account =
  azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the file client.
azure::storage::cloud_file_client file_client =
  storage_account.create_cloud_file_client();

// Get a reference to the share.
azure::storage::cloud_file_share share =
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));
if (share.exists())
{
    std::cout << "Current share usage: " << share.download_share_usage() << "/" << share.properties().quota();

    //This line sets the quota to 2560GB
    share.resize(2560);

    std::cout << "Quota increased: " << share.download_share_usage() << "/" << share.properties().quota();

}
```

## <a name="generate-a-shared-access-signature-for-a-file-or-file-share"></a>Создание подписи общего доступа для файла или файлового ресурса

Для файлового ресурса или отдельного файла можно создать подписанный URL-адрес (SAS). Также можно создать политики общего доступа на файловом ресурсе, чтобы управлять подписями общего доступа. Создание политики общего доступа рекомендуется, так как она позволяет отменить SAS, если она скомпрометирована.

В приведенном ниже примере создаются политики общего доступа в общей папке, а затем используются для обеспечения ограничения SAS для файла в общей папке.

```cpp
// Parse the connection string for the storage account.
azure::storage::cloud_storage_account storage_account =
  azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the file client and get a reference to the share
azure::storage::cloud_file_client file_client =
  storage_account.create_cloud_file_client();

azure::storage::cloud_file_share share =
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));

if (share.exists())
{
    // Create and assign a policy
    utility::string_t policy_name = _XPLATSTR("sampleSharePolicy");

    azure::storage::file_shared_access_policy sharedPolicy =
      azure::storage::file_shared_access_policy();

    //set permissions to expire in 90 minutes
    sharedPolicy.set_expiry(utility::datetime::utc_now() +
       utility::datetime::from_minutes(90));

    //give read and write permissions
    sharedPolicy.set_permissions(azure::storage::file_shared_access_policy::permissions::write | azure::storage::file_shared_access_policy::permissions::read);

    //set permissions for the share
    azure::storage::file_share_permissions permissions;

    //retrieve the current list of shared access policies
    azure::storage::shared_access_policies<azure::storage::file_shared_access_policy> policies;

    //add the new shared policy
    policies.insert(std::make_pair(policy_name, sharedPolicy));

    //save the updated policy list
    permissions.set_policies(policies);
    share.upload_permissions(permissions);

    //Retrieve the root directory and file references
    azure::storage::cloud_file_directory root_dir =
        share.get_root_directory_reference();
    azure::storage::cloud_file file =
      root_dir.get_file_reference(_XPLATSTR("my-sample-file-1"));

    // Generate a SAS for a file in the share
    //  and associate this access policy with it.
    utility::string_t sas_token = file.get_shared_access_signature(sharedPolicy);

    // Create a new CloudFile object from the SAS, and write some text to the file.
    azure::storage::cloud_file file_with_sas(azure::storage::storage_credentials(sas_token).transform_uri(file.uri().primary_uri()));
    utility::string_t text = _XPLATSTR("My sample content");
    file_with_sas.upload_text(text);

    //Download and print URL with SAS.
    utility::string_t downloaded_text = file_with_sas.download_text();
    ucout << downloaded_text << std::endl;
    ucout << azure::storage::storage_credentials(sas_token).transform_uri(file.uri().primary_uri()).to_string() << std::endl;

}
```

## <a name="next-steps"></a>Дальнейшие действия

Для получения дополнительных сведений о службе хранилища Azure изучите следующие ресурсы:

* [Клиентская библиотека хранилища для C++](https://github.com/Azure/azure-storage-cpp)
* [Примеры для службы хранилища файлов Azure на языке C++](https://github.com/Azure-Samples/storage-file-cpp-getting-started)
* [Обозреватель службы хранилища Azure](https://go.microsoft.com/fwlink/?LinkID=822673&clcid=0x409)
* [Документация по хранилищу Azure](https://azure.microsoft.com/documentation/services/storage/)