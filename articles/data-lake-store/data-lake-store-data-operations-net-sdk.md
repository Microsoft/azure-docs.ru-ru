---
title: Пакет SDK для .NET — операции файловой системы в Data Lake Storage 1-го поколения — Azure
description: Используйте пакет SDK для Azure Data Lake Storage 1-го поколения .NET для операций файловой системы на Data Lake Storage 1-го поколения таких как создание папок и т. д.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 01/03/2020
ms.author: twooley
ms.custom: devx-track-dotnet
ms.openlocfilehash: faa539119bbb8d73e66788b8ba472cf0db26a2e4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102431743"
---
# <a name="filesystem-operations-on-data-lake-storage-gen1-using-the-net-sdk"></a>Операции файловой системы на Data Lake Storage 1-го поколения с помощью пакета SDK для .NET

> [!div class="op_single_selector"]
> * [Пакет SDK для .NET](data-lake-store-data-operations-net-sdk.md)
> * [пакет SDK для Java](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
>

Из этой статьи вы узнаете, как выполнять операции файловой системы на Data Lake Storage 1-го поколения с помощью пакета SDK для .NET. К операциям файловой системы относятся операции создания папок в учетной записи Data Lake Storage 1-го поколения, передачи файлов, скачивания файлов и т. д.

Инструкции по выполнению операций управления учетными записями в Data Lake Storage 1-го поколения с помощью пакета SDK для .NET см. в разделе [операции управления учетными записями в Data Lake Storage 1-го поколения с помощью пакета SDK для .NET](data-lake-store-get-started-net-sdk.md).

## <a name="prerequisites"></a>Предварительные условия

* **Visual Studio 2013 или более поздней версии**. В инструкциях этой статьи используется Visual Studio 2019.

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Учетная запись Azure Data Lake Storage 1-го поколения**. Инструкции по созданию учетной записи см. в статье [Приступая к работе с Azure Data Lake Storage 1-го поколения](data-lake-store-get-started-portal.md).

## <a name="create-a-net-application"></a>Создание приложения .NET

На [сайте GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted) доступен пример кода, который используется для создания файлов в хранилище, объединения и скачивания файлов, а также для удаления файлов из хранилища. В этом разделе статьи рассматриваются основные части этого кода.

1. В Visual Studio выберите меню **файл** , **создать**, а затем **проект**.
1. Выберите **консольное приложение (платформа .NET Framework)**, а затем нажмите кнопку **Далее**.
1. В окне **Имя проекта** введите `CreateADLApplication`, а затем выберите **Создать**.
1. Добавьте пакеты NuGet в проект.

   1. В обозревателе решений щелкните правой кнопкой мыши имя проекта и выберите пункт **Управление пакетами NuGet**.
   1. На вкладке **Диспетчер пакетов NuGet** убедитесь, что для параметра **источник пакета** задано значение **NuGet.org**. Кроме того, убедитесь, что установлен флажок **включить предварительные выпуски** .
   1. Найдите и установите следующие пакеты NuGet:

      * `Microsoft.Azure.DataLake.Store` — В этой статье используется версия 1.0.0.
      * `Microsoft.Rest.ClientRuntime.Azure.Authentication` — В этой статье используется v 2.3.1.

      Закройте **Диспетчер пакетов NuGet**.

1. Откройте файл **Program.cs**, удалите существующий код и включите следующие инструкции, чтобы добавить ссылки на пространства имен.

    ```
    using System;
    using System.IO;using System.Threading;
    using System.Linq;
    using System.Text;
    using System.Collections.Generic;
    using System.Security.Cryptography.X509Certificates; // Required only if you're using an Azure AD application created with certificates
                
    using Microsoft.Rest;
    using Microsoft.Rest.Azure.Authentication;
    using Microsoft.Azure.DataLake.Store;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```

1. Объявите переменные, как показано ниже, и укажите значения вместо заполнителей. Кроме того, убедитесь, что локальный путь и имя файла, которые вы указываете, имеются на компьютере.

    ```
    namespace SdkSample
    {
        class Program
        {
            private static string _adlsg1AccountName = "<DATA-LAKE-STORAGE-GEN1-NAME>.azuredatalakestore.net";
        }
    }
    ```

В остальных разделах этой статьи можно увидеть, как использовать доступные методы .NET для выполнения таких операций, как проверка подлинности, отправка файлов и т. д.

## <a name="authentication"></a>Аутентификация

* Дополнительные сведения о проверке подлинности пользователей в приложении см. в статье [Аутентификация пользователей в Data Lake Store с помощью пакета SDK для .NET](data-lake-store-end-user-authenticate-net-sdk.md).
* Дополнительные сведения о проверке подлинности между службами в приложении см. в статье [Аутентификация между службами в Data Lake Store с помощью пакета SDK для .NET](data-lake-store-service-to-service-authenticate-net-sdk.md).

## <a name="create-client-object"></a>Создание клиентского объекта

Следующий фрагмент кода создает клиентский объект файловой системы Data Lake Storage 1-го поколения, который используется для передачи запросов к службе.

```
// Create client objects
AdlsClient client = AdlsClient.CreateClient(_adlsg1AccountName, adlCreds);
```

## <a name="create-a-file-and-directory"></a>Создание файла и каталога

Добавьте в приложение указанный ниже фрагмент кода. Этот фрагмент кода добавляет файл и любой родительский каталог, который не существует.

```
// Create a file - automatically creates any parent directories that don't exist
// The AdlsOutputStream preserves record boundaries - it does not break records while writing to the store

using (var stream = client.CreateFile(fileName, IfExists.Overwrite))
{
    byte[] textByteArray = Encoding.UTF8.GetBytes("This is test data to write.\r\n");
    stream.Write(textByteArray, 0, textByteArray.Length);

    textByteArray = Encoding.UTF8.GetBytes("This is the second line.\r\n");
    stream.Write(textByteArray, 0, textByteArray.Length);
}
```

## <a name="append-to-a-file"></a>Добавление данных в файл

Следующий фрагмент кода добавляет данные в имеющийся файл в учетной записи Data Lake Storage 1-го поколения.

```
// Append to existing file

using (var stream = client.GetAppendStream(fileName))
{
    byte[] textByteArray = Encoding.UTF8.GetBytes("This is the added line.\r\n");
    stream.Write(textByteArray, 0, textByteArray.Length);
}
```

## <a name="read-a-file"></a>Чтение файла

Следующий фрагмент кода считывает содержимое файла в Data Lake Storage 1-го поколения.

```
//Read file contents

using (var readStream = new StreamReader(client.GetReadStream(fileName)))
{
    string line;
    while ((line = readStream.ReadLine()) != null)
    {
        Console.WriteLine(line);
    }
}
```

## <a name="get-file-properties"></a>Получение свойств файла

Следующий фрагмент кода возвращает свойства, связанные с файлом или каталогом.

```
// Get file properties
var directoryEntry = client.GetDirectoryEntry(fileName);
PrintDirectoryEntry(directoryEntry);
```

Определение `PrintDirectoryEntry` метода доступно как часть примера [на GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted).

## <a name="rename-a-file"></a>Переименуйте файл

Следующий фрагмент кода переименовывает имеющийся файл в учетной записи Data Lake Storage 1-го поколения.

```
// Rename a file
string destFilePath = "/Test/testRenameDest3.txt";
client.Rename(fileName, destFilePath, true);
```

## <a name="enumerate-a-directory"></a>Перечисление каталогов

Следующий фрагмент кода Перечисляет каталоги в учетной записи Data Lake Storage 1-го поколения.

```
// Enumerate directory
foreach (var entry in client.EnumerateDirectory("/Test"))
{
    PrintDirectoryEntry(entry);
}
```

Определение `PrintDirectoryEntry` метода доступно как часть примера [на GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted).

## <a name="delete-directories-recursively"></a>Рекурсивное удаление каталогов

Следующий фрагмент кода рекурсивно удаляет каталог и все его подкаталоги.

```
// Delete a directory and all its subdirectories and files
client.DeleteRecursive("/Test");
```

## <a name="samples"></a>Примеры

Ниже приведено несколько примеров, демонстрирующих использование пакета SDK для файловой системы Data Lake Storage 1-го поколения.

* [Базовый пример на GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted)
* [Расширенный пример на GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-samples)

## <a name="see-also"></a>См. также раздел

* [Операции управления учетными записями в Data Lake Storage 1-го поколения c использованием пакета SDK для .NET](data-lake-store-get-started-net-sdk.md)
* [Библиотеки Azure Data Lake Store для .NET](/dotnet/api/overview/azure/data-lake-store)

## <a name="next-steps"></a>Дальнейшие действия

* [Защита данных в Data Lake Storage Gen1](data-lake-store-secure-data.md)