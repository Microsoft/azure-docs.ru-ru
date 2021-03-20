---
title: REST API. Операции файловой системы в Azure Data Lake Storage 1-го поколения | Документация Майкрософт
description: Использование интерфейсов REST API WebHDFS для выполнения операций файловой системы в Azure Data Lake Storage 1-го поколения
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 51e0fb2ffa7b573ecfeda163d9ad99597ff735a2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92109210"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-rest-api"></a>Операции файловой системы в Azure Data Lake Storage 1-го поколения с использованием REST API
> [!div class="op_single_selector"]
> * [Пакет SDK для .NET](data-lake-store-data-operations-net-sdk.md)
> * [пакет SDK для Java](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

В этой статье содержатся сведения об использовании интерфейсов REST API WebHDFS и REST API Data Lake Storage 1-го поколения для выполнения операций файловой системы в Azure Data Lake Storage 1-го поколения. Дополнительные сведения о том, как выполнять операции управления учетными записями в Data Lake Storage 1-го поколения с помощью REST API, см. в [этой статье](data-lake-store-get-started-rest-api.md).

## <a name="prerequisites"></a>Предварительные условия
* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Учетная запись Azure Data Lake Storage 1-го поколения**. Следуйте инструкциям из статьи [Начало работы с Azure Data Lake Storage Gen1 с помощью портала Azure](data-lake-store-get-started-portal.md).

* **[перелистывание](https://curl.haxx.se/)**. В этой статье для демонстрации вызовов REST API к учетной записи Data Lake Storage 1-го поколения используется cURL.

## <a name="how-do-i-authenticate-using-azure-active-directory"></a>Как выполнить аутентификацию с помощью Azure Active Directory?
Существует два способа проверки подлинности с помощью Azure Active Directory.

* Дополнительные сведения о проверке подлинности пользователей в приложении (интерактивная проверка подлинности) см. в статье [Аутентификация пользователей в Data Lake Store с помощью REST API](data-lake-store-end-user-authenticate-rest-api.md).
* Дополнительные сведения о проверке подлинности между службами в приложении (неинтерактивная проверка подлинности) см. в статье [Аутентификация между службами в Data Lake Store с помощью REST API](data-lake-store-service-to-service-authenticate-rest-api.md).


## <a name="create-folders"></a>Создайте папки:
Эта операция основана на вызове REST API WebHDFS, определенном [здесь](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Make_a_Directory).

Используйте следующую команду cURL: Замените на **\<yourstorename>** имя учетной записи Data Lake Storage 1-го поколения.

```console
curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -d "" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/?op=MKDIRS'
```

В приведенной выше команде замените \<`REDACTED`\> полученным ранее маркером авторизации. Эта команда создает каталог с именем **mytempdir** в корневой папке учетной записи Data Lake Storage 1-го поколения.

Если работа приложения завершится успешно, отобразится результат, подобный приведенному ниже фрагменту кода:

```output
{"boolean":true}
```

## <a name="list-folders"></a>Вывод списка папок
Эта операция основана на вызове REST API WebHDFS, определенном [здесь](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#List_a_Directory).

Используйте следующую команду cURL: Замените на **\<yourstorename>** имя учетной записи Data Lake Storage 1-го поколения.

```console
curl -i -X GET -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS'
```

В приведенной выше команде замените \<`REDACTED`\> полученным ранее маркером авторизации.

Если работа приложения завершится успешно, отобразится результат, подобный приведенному ниже фрагменту кода:

```output
{
"FileStatuses": {
    "FileStatus": [{
        "length": 0,
        "pathSuffix": "mytempdir",
        "type": "DIRECTORY",
        "blockSize": 268435456,
        "accessTime": 1458324719512,
        "modificationTime": 1458324719512,
        "replication": 0,
        "permission": "777",
        "owner": "<GUID>",
        "group": "<GUID>"
    }]
}
}
```

## <a name="upload-data"></a>Отправка данных
Эта операция основана на вызове REST API WebHDFS, определенном [здесь](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Create_and_Write_to_a_File).

Используйте следующую команду cURL: Замените на **\<yourstorename>** имя учетной записи Data Lake Storage 1-го поколения.

```console
curl -i -X PUT -L -T 'C:\temp\list.txt' -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/list.txt?op=CREATE'
```

В предыдущем синтаксисе параметр **-T** представляет расположение отправляемого файла.

Вывод аналогичен приведенному ниже фрагменту кода:
   
```output
HTTP/1.1 307 Temporary Redirect
...
Location: https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/list.txt?op=CREATE&write=true
...
Content-Length: 0

HTTP/1.1 100 Continue

HTTP/1.1 201 Created
...
```

## <a name="read-data"></a>Чтение данных
Эта операция основана на вызове REST API WebHDFS, определенном [здесь](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Open_and_Read_a_File).

Процесс чтения данных из учетной записи Data Lake Storage 1-го поколения состоит из двух этапов.

* Сначала следует отправить запрос GET к конечной точке `https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN`. Будет возвращено расположение для отправки следующего запроса GET.
* Затем нужно отправить запрос GET к конечной точке `https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN&read=true`. Будет отображено содержимое файла.

Однако поскольку на первом и втором этапе применяются одинаковые входные параметры, для отправки первого запроса можно использовать параметр `-L`. `-L` фактически объединяет два запроса в один, а также позволяет cURL повторно отправить запрос к новому расположению. И наконец, отображаются выходные данные всех вызовов запросов, как показано в следующем фрагменте кода. Замените на **\<yourstorename>** имя учетной записи Data Lake Storage 1-го поколения.

```console
curl -i -L GET -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN'
```

Должен отобразиться результат, аналогичный приведенному ниже фрагменту кода.

```output
HTTP/1.1 307 Temporary Redirect
...
Location: https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/somerandomfile.txt?op=OPEN&read=true
...

HTTP/1.1 200 OK
...

Hello, Data Lake Store user!
```

## <a name="rename-a-file"></a>Переименуйте файл
Эта операция основана на вызове REST API WebHDFS, определенном [здесь](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Rename_a_FileDirectory).

Чтобы переименовать файл, используйте следующую команду cURL: Замените на **\<yourstorename>** имя учетной записи Data Lake Storage 1-го поколения.

```console
curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -d "" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=RENAME&destination=/mytempdir/myinputfile1.txt'
```

Должен отобразиться результат, аналогичный приведенному ниже фрагменту кода.

```output
HTTP/1.1 200 OK
...

{"boolean":true}
```

## <a name="delete-a-file"></a>Удаление файла
Эта операция основана на вызове REST API WebHDFS, определенном [здесь](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Delete_a_FileDirectory).

Чтобы удалить файл, используйте следующую команду cURL: Замените на **\<yourstorename>** имя учетной записи Data Lake Storage 1-го поколения.

```console
curl -i -X DELETE -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile1.txt?op=DELETE'
```

Вы должны увидеть подобные выходные данные:

```output
HTTP/1.1 200 OK
...

{"boolean":true}
```

## <a name="next-steps"></a>Дальнейшие действия
* [Операции управления учетными записями в Azure Data Lake Store с использованием REST API](data-lake-store-get-started-rest-api.md)

## <a name="see-also"></a>См. также раздел
* [Справочник по REST API для Azure Data Lake Storage 1-го поколения](/rest/api/datalakestore/)
* [Приложения больших данных с открытым исходным кодом, которые работают с Azure Data Lake Storage Gen1](data-lake-store-compatible-oss-other-applications.md)