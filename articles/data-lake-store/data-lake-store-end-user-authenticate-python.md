---
title: Проверка подлинности конечным пользователем с помощью Python с Data Lake Storage 1-го поколения — Azure
description: Узнайте, как реализовать проверку подлинности пользователей в Azure Data Lake Storage 1-го поколения с помощью Azure Active Directory и Python.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: has-adal-ref, devx-track-python
ms.openlocfilehash: c7ee58dec4e8bc04eb07695378e0d4dc973b6705
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92109091"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-python"></a>Проверка подлинности пользователей в Azure Data Lake Storage 1-го поколения с помощью Python
> [!div class="op_single_selector"]
> * [Использование Java](data-lake-store-end-user-authenticate-java-sdk.md)
> * [Использование пакета SDK для .NET](data-lake-store-end-user-authenticate-net-sdk.md)
> * [Использование Python](data-lake-store-end-user-authenticate-python.md)
> * [Использование REST API](data-lake-store-end-user-authenticate-rest-api.md)
>
>

В этой статье описывается, как использовать пакет SDK для Python для проверки подлинности пользователей с помощью Azure Data Lake Storage 1-го поколения. Аутентификация пользователей бывает двух типов:

* Аутентификация пользователей без многофакторной проверки подлинности.
* Аутентификация пользователей с многофакторной проверкой подлинности.

В этой статье рассматриваются оба типа. См. дополнительные сведения о [проверке подлинности между службами в Data Lake Storage 1-го поколения с помощью Python](data-lake-store-service-to-service-authenticate-python.md).

## <a name="prerequisites"></a>Предварительные условия

* **Python**. Скачать Python можно [здесь](https://www.python.org/downloads/). В этой статье используется версия Python 3.6.2.

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Создание собственного приложения Active Directory** Вам нужно выполнить инструкции по [аутентификации пользователей в Data Lake Storage 1-го поколения с помощью Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md).

## <a name="install-the-modules"></a>Установка модулей

Для работы с Data Lake Storage 1-го поколения с использованием Python необходимо установить три модуля.

* Модуль `azure-mgmt-resource`, который включает в себя модули Azure для Active Directory и т. д.
* Модуль `azure-mgmt-datalake-store`, который включает в себя операции по управлению учетной записью Azure Data Lake Storage 1-го поколения. Дополнительные сведения см. в [справочнике по модулю управления Azure Data Lake Storage 1-го поколения](/python/api/azure-mgmt-datalake-store/).
* Модуль `azure-datalake-store`, который включает в себя операции с файловой системой Azure Data Lake Storage 1-го поколения. Дополнительные сведения об этом модуле см. в статье [Справочник по модулю файловой системы Azure-Lake-Store](/python/api/azure-datalake-store/azure.datalake.store.core/).

Чтобы установить модули, используйте следующие команды.

```console
pip install azure-mgmt-resource
pip install azure-mgmt-datalake-store
pip install azure-datalake-store
```

## <a name="create-a-new-python-application"></a>Создание приложения Python

1. В интегрированной среде разработки создайте новое приложение Python, например **MySample.py**.

2. Чтобы импортировать необходимые модули, добавьте следующий фрагмент кода.

    ```
    ## Use this for Azure AD authentication
    from msrestazure.azure_active_directory import AADTokenCredentials

    ## Required for Azure Data Lake Storage Gen1 account management
    from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
    from azure.mgmt.datalake.store.models import DataLakeStoreAccount

    ## Required for Azure Data Lake Storage Gen1 filesystem management
    from azure.datalake.store import core, lib, multithread

    # Common Azure imports
    import adal
    from azure.mgmt.resource.resources import ResourceManagementClient
    from azure.mgmt.resource.resources.models import ResourceGroup

    ## Use these as needed for your application
    import logging, pprint, uuid, time
    ```

3. Сохраните изменения в mysample.py.

## <a name="end-user-authentication-with-multi-factor-authentication"></a>Аутентификация пользователей с многофакторной проверкой подлинности.

### <a name="for-account-management"></a>Управление учетными записями

Используйте следующий фрагмент кода для проверки подлинности в Azure AD и выполнения операций управления учетными записями в Data Lake Storage 1-го поколения. Следующий фрагмент кода можно использовать для проверки подлинности приложения с помощью многофакторной идентификации. Укажите следующие значения для существующего **собственного** приложения Azure AD.

```python
authority_host_url = "https://login.microsoftonline.com"
tenant = "FILL-IN-HERE"
authority_url = authority_host_url + '/' + tenant
client_id = 'FILL-IN-HERE'
redirect = 'urn:ietf:wg:oauth:2.0:oob'
RESOURCE = 'https://management.core.windows.net/'

context = adal.AuthenticationContext(authority_url)
code = context.acquire_user_code(RESOURCE, client_id)
print(code['message'])
mgmt_token = context.acquire_token_with_device_code(RESOURCE, code, client_id)
armCreds = AADTokenCredentials(mgmt_token, client_id, resource = RESOURCE)
```

### <a name="for-filesystem-operations"></a>Операции файловой системы

Используйте для проверки подлинности в Azure AD и выполнения операций файловой системы в учетной записи Data Lake Storage 1-го поколения. Следующий фрагмент кода можно использовать для проверки подлинности приложения с помощью многофакторной идентификации. Укажите следующие значения для существующего **собственного** приложения Azure AD.

```console
adlCreds = lib.auth(tenant_id='FILL-IN-HERE', resource = 'https://datalake.azure.net/')
```

## <a name="end-user-authentication-without-multi-factor-authentication"></a>Аутентификация пользователей без многофакторной проверки подлинности.

Мы не рекомендуем использовать этот метод. Дополнительные сведения см. в руководстве по [аутентификации в Azure с помощью пакета SDK для Python](/azure/python/python-sdk-azure-authenticate).

## <a name="next-steps"></a>Дальнейшие действия
В этой статье вы узнали, как использовать проверку подлинности пользователей, чтобы реализовать проверку подлинности в Data Lake Storage 1-го поколения с помощью Python. Дополнительные сведения об использовании Python для работы с Azure Data Lake Storage 1-го поколения см. в следующих статьях.

* [Операции управления учетными записями в Data Lake Storage 1-го поколения c использованием Python](data-lake-store-get-started-python.md)
* [Операции с данными в Data Lake Storage 1-го поколения c использованием Python](data-lake-store-data-operations-python.md)