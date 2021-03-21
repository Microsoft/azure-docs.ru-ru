---
title: Проверка подлинности между службами в Python — Data Lake Storage 1-го поколения
description: Узнайте, как реализовать аутентификацию между службами в Azure Data Lake Storage 1-го поколения с помощью Azure Active Directory и Python.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: has-adal-ref, devx-track-python
ms.openlocfilehash: fe987070438c689e899dee18edaf4f708555ab11
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92106762"
---
# <a name="service-to-service-authentication-with-azure-data-lake-storage-gen1-using-python"></a>Аутентификация между службами в Azure Data Lake Storage 1-го поколения с использованием Python
> [!div class="op_single_selector"]
> * [Использование Java](data-lake-store-service-to-service-authenticate-java.md)
> * [Использование пакета SDK для .NET](data-lake-store-service-to-service-authenticate-net-sdk.md)
> * [Использование Python](data-lake-store-service-to-service-authenticate-python.md)
> * [Использование REST API](data-lake-store-service-to-service-authenticate-rest-api.md)
>
>

В этой статье описывается, как использовать пакет SDK для Python для аутентификации между службами в Azure Data Lake Storage 1-го поколения. Дополнительные сведения об аутентификации пользователей в Azure Data Lake Storage 1-го поколения с помощью Python см. статье [Аутентификация пользователей в Data Lake Storage 1-го поколения с использованием Python](data-lake-store-end-user-authenticate-python.md).


## <a name="prerequisites"></a>Предварительные условия

* **Python**. Скачать Python можно [здесь](https://www.python.org/downloads/). В этой статье используется версия Python 3.6.2.

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Создайте веб-приложение Azure Active Directory**. Вам нужно выполнить инструкции по [аутентификации между службами в Data Lake Storage 1-го поколения с помощью Azure Active Directory](data-lake-store-service-to-service-authenticate-using-active-directory.md).

## <a name="install-the-modules"></a>Установка модулей

Для работы с Data Lake Storage 1-го поколения с использованием Python необходимо установить три модуля.

* Модуль `azure-mgmt-resource`, который включает в себя модули Azure для Active Directory и т. д.
* Модуль `azure-mgmt-datalake-store`, который включает в себя операции по управлению учетной записью Data Lake Storage 1-го поколения. Дополнительные сведения см. в [справочнике по модулю управления Azure Data Lake Storage 1-го поколения](/python/api/azure-mgmt-datalake-store/).
* Модуль `azure-datalake-store`, который включает в себя операции с файловой системой Data Lake Storage 1-го поколения. Дополнительные сведения об этом модуле см. в статье [Справочник по модулю файловой системы Azure-Lake-Store](/python/api/azure-datalake-store/azure.datalake.store.core/).

Чтобы установить модули, используйте следующие команды.

```
pip install azure-mgmt-resource
pip install azure-mgmt-datalake-store
pip install azure-datalake-store
```

## <a name="create-a-new-python-application"></a>Создание приложения Python

1. Для создания приложения Python используйте IDE по своему усмотрению, например **mysample.py**.

2. Чтобы импортировать необходимые модули, добавьте следующий фрагмент кода.

    ```
    ## Use this for Azure AD authentication
    from msrestazure.azure_active_directory import AADTokenCredentials

    ## Required for Data Lake Storage Gen1 account management
    from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
    from azure.mgmt.datalake.store.models import DataLakeStoreAccount

    ## Required for Data Lake Storage Gen1 filesystem management
    from azure.datalake.store import core, lib, multithread

    # Common Azure imports
    import adal
    from azure.mgmt.resource.resources import ResourceManagementClient
    from azure.mgmt.resource.resources.models import ResourceGroup

    ## Use these as needed for your application
    import logging, getpass, pprint, uuid, time
    ```

3. Сохраните изменения в mysample.py.

## <a name="service-to-service-authentication-with-client-secret-for-account-management"></a>Проверки подлинности через секрет клиента со взаимодействием между службами для управления учетными записями

Используйте этот фрагмент кода для проверки подлинности в Azure AD для операций управления учетными записями Data Lake Storage 1-го поколения таких как создание учетной записи Data Lake Storage 1-го поколения, удаление учетной записи Data Lake Storage 1-го поколения и т. д. Следующий фрагмент кода можно использовать для проверки подлинности приложения в неинтерактивном режиме, используя секрет клиента для субъекта приложения или службы существующего приложения Azure AD "веб-приложение".

```python
authority_host_uri = 'https://login.microsoftonline.com'
tenant = '<TENANT>'
authority_uri = authority_host_uri + '/' + tenant
RESOURCE = 'https://management.core.windows.net/'
client_id = '<CLIENT_ID>'
client_secret = '<CLIENT_SECRET>'

context = adal.AuthenticationContext(authority_uri, api_version=None)
mgmt_token = context.acquire_token_with_client_credentials(RESOURCE, client_id, client_secret)
armCreds = AADTokenCredentials(mgmt_token, client_id, resource=RESOURCE)
```

## <a name="service-to-service-authentication-with-client-secret-for-filesystem-operations"></a>Проверка подлинности между службами через секрет клиента для операций файловой системы

Используйте следующий фрагмент кода для проверки подлинности в Azure AD для операций файловой системы на Data Lake Storage 1-го поколения таких как создание папки, передача файла и т. д. Следующий фрагмент кода можно использовать для проверки подлинности приложения в неинтерактивном режиме с помощью секрета клиента для приложения или субъекта-службы. Примените этот фрагмент кода к существующему веб-приложению Azure AD.

```python
tenant = '<TENANT>'
RESOURCE = 'https://datalake.azure.net/'
client_id = '<CLIENT_ID>'
client_secret = '<CLIENT_SECRET>'

adlCreds = lib.auth(tenant_id = tenant,
                client_secret = client_secret,
                client_id = client_id,
                resource = RESOURCE)
```

<!-- ## Service-to-service authentication with certificate for account management

Use this snippet to authenticate with Azure AD for account management operations on Data Lake Storage Gen1 such as create a Data Lake Storage Gen1 account, delete a Data Lake Storage Gen1 account, etc. The following snippet can be used to authenticate your application non-interactively, using the certificate of an existing Azure AD "Web App" application. For instructions on how to create an Azure AD application, see [Create service principal with certificates](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-self-signed-certificate).

    authority_host_uri = 'https://login.microsoftonline.com'
    tenant = '<TENANT>'
    authority_uri = authority_host_uri + '/' + tenant
    resource_uri = 'https://management.core.windows.net/'
    client_id = '<CLIENT_ID>'
    client_cert = '<CLIENT_CERT>'
    client_cert_thumbprint = '<CLIENT_CERT_THUMBPRINT>'

    context = adal.AuthenticationContext(authority_uri, api_version=None)
    mgmt_token = context.acquire_token_with_client_certificate(resource_uri, client_id, client_cert, client_cert_thumbprint)
    credentials = AADTokenCredentials(mgmt_token, client_id) -->

## <a name="next-steps"></a>Дальнейшие действия
В этой статье вы узнали, как использовать аутентификацию между службами, чтобы реализовать аутентификацию в Data Lake Storage 1-го поколения с использованием Python. Дополнительные сведения об использовании Python для работы с Data Lake Storage 1-го поколения см. в следующих статьях.

* [Операции управления учетными записями в Data Lake Storage 1-го поколения c использованием Python](data-lake-store-get-started-python.md)
* [Операции с данными в Data Lake Storage 1-го поколения c использованием Python](data-lake-store-data-operations-python.md)