---
title: Управление учетной записью Azure Data Lake Storage 1-го поколения с помощью службы "ОСТАВШАЯся"
description: Используйте REST APIи HDFS для выполнения операций управления учетными записями в учетной записи Azure Data Lake Storage 1-го поколения.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 13467a51b2a06dbc0ca0ec5eadd139fde8b82ad0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92103498"
---
# <a name="account-management-operations-on-azure-data-lake-storage-gen1-using-rest-api"></a>Операции управления учетными записями в Azure Data Lake Storage 1-го поколения c использованием REST API
> [!div class="op_single_selector"]
> * [Пакет SDK для .NET](data-lake-store-get-started-net-sdk.md)
> * [REST API](data-lake-store-get-started-rest-api.md)
> * [Python](data-lake-store-get-started-python.md)
>
>

В этой статье содержатся сведения о выполнении операций управления учетными записями в Azure Data Lake Storage 1-го поколения с использованием REST API. Операции управления учетными записями включают создание учетной записи Data Lake Storage 1-го поколения, удаление учетной записи Data Lake Storage 1-го поколения и т. д. Инструкции по выполнению операций файловой системы в Data Lake Storage 1-го поколения с помощью REST API см. [в разделе операции файловой системы на Data Lake Storage 1-го поколения с помощью REST API](data-lake-store-data-operations-rest-api.md).

## <a name="prerequisites"></a>Предварительные условия
* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

* **[перелистывание](https://curl.haxx.se/)**. В этой статье для демонстрации вызовов REST API к учетной записи Data Lake Storage 1-го поколения используется cURL.

## <a name="how-do-i-authenticate-using-azure-active-directory"></a>Как выполнить аутентификацию с помощью Azure Active Directory?
Существует два способа проверки подлинности с помощью Azure Active Directory.

* Дополнительные сведения о проверке подлинности пользователей в приложении (интерактивная проверка подлинности) см. в статье [Аутентификация пользователей в Data Lake Store с помощью REST API](data-lake-store-end-user-authenticate-rest-api.md).
* Дополнительные сведения о проверке подлинности между службами в приложении (неинтерактивная проверка подлинности) см. в статье [Аутентификация между службами в Data Lake Store с помощью REST API](data-lake-store-service-to-service-authenticate-rest-api.md).


## <a name="create-a-data-lake-storage-gen1-account"></a>Создание учетной записи Data Lake Storage 1-го поколения
Эта операция основана на вызове REST API, определенном [здесь](/rest/api/datalakestore/accounts/create).

Используйте следующую команду cURL: Замените на **\<yourstoragegen1name>** имя Data Lake Storage 1-го поколения.

```console
curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -H "Content-Type: application/json" https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.DataLakeStore/accounts/<yourstoragegen1name>?api-version=2015-10-01-preview -d@"C:\temp\input.json"
```

В приведенной выше команде замените \<`REDACTED`\> полученным ранее маркером авторизации. Полезные данные запроса для этой команды находятся в файле **input.json**, предоставленном для параметра `-d` выше. Содержимое файла input.json выглядит как следующий фрагмент кода:

```json
{
"location": "eastus2",
"tags": {
    "department": "finance"
    },
"properties": {}
}
```

## <a name="delete-a-data-lake-storage-gen1-account"></a>Удаление учетной записи Data Lake Storage 1-го поколения
Эта операция основана на вызове REST API, определенном [здесь](/rest/api/datalakestore/accounts/delete).

Чтобы удалить учетную запись Data Lake Storage 1-го поколения, используйте следующую команду cURL. Замените на **\<yourstoragegen1name>** имя учетной записи Data Lake Storage 1-го поколения.

```console
curl -i -X DELETE -H "Authorization: Bearer <REDACTED>" https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.DataLakeStore/accounts/<yourstoragegen1name>?api-version=2015-10-01-preview
```

Вы должны увидеть выходные данные, подобные следующему фрагменту кода:

```output
HTTP/1.1 200 OK
...
...
```

## <a name="next-steps"></a>Дальнейшие действия
* [Операции файловой системы в Data Lake Storage 1-го поколения c использованием REST API](data-lake-store-data-operations-rest-api.md).

## <a name="see-also"></a>См. также раздел
* [Справочник по REST API для Azure Data Lake Storage 1-го поколения](/rest/api/datalakestore/)
* [Приложения больших данных с открытым исходным кодом, которые работают с Azure Data Lake Storage Gen1](data-lake-store-compatible-oss-other-applications.md)