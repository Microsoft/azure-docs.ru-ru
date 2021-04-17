---
title: Краткое руководство. Создание учетной записи Purview с помощью Python
description: Создание экземпляра учетной записи Azure Purview
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.devlang: python
ms.topic: quickstart
ms.date: 04/02/2021
ms.openlocfilehash: f8ac611d25507913d6d5f2e2dd289ea52ce4ae9f
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/05/2021
ms.locfileid: "106387214"
---
# <a name="quickstart-create-a-purview-account-using-python"></a>Краткое руководство. Создание учетной записи Purview с помощью Python

Краткое руководство. Создание учетной записи Purview с помощью Python 

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

* Собственный [клиент Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

* Ваша учетная запись должна иметь разрешение на создание ресурсов в подписке.

* Если используется **политика Azure**, которая запрещает всем приложениям создавать **учетные записи хранения** и **пространства имен Центра событий**, следует создать исключение из этой политики с помощью тега, который можно указать при создании учетной записи Azure Purview. Основная причина заключается в том, что для каждой созданной учетной записи Azure Purview необходимо создать управляемую группу ресурсов, а в этой группе ресурсов создать учетную запись хранения и пространство имен EventHub. Дополнительные сведения см. в статье [Создание портала каталога](create-catalog-portal.md) .


## <a name="install-the-python-package"></a>Установка пакета Python

1. Откройте терминал или командную строку с правами администратора. 
2. Сначала установите пакет Python для ресурсов управления Azure:

    ```python
    pip install azure-mgmt-resource
    ```
3. Чтобы установить пакет Python для Purview, выполните следующую команду:

    ```python
    pip install azure-mgmt-purview
    ```

    [Пакет SDK Python для Purview](https://github.com/Azure/azure-sdk-for-python) поддерживает Python 2.7, 3.3, 3.4, 3.5, 3.6 и 3.7.

4. Чтобы установить пакет Python для проверки подлинности по удостоверению Azure, выполните следующую команду:

    ```python
    pip install azure-identity
    ```
    > [!NOTE] 
    > Пакет azure-identity может конфликтовать с azure-cli в отношении некоторых общих зависимостей. Если вы столкнетесь с проблемами проверки подлинности, удалите пакет azure-cli и его зависимости или используйте компьютер, на котором не установлен этот пакет, для обеспечения надлежащей работы.
    
## <a name="create-a-purview-client"></a>Создание клиента Purview

1. Создайте файл с именем **purview.py**. Добавьте следующие инструкции, чтобы добавить ссылки на пространства имен.

    ```python
    from azure.identity import ClientSecretCredential 
    from azure.mgmt.resource import ResourceManagementClient
    from azure.mgmt.purview import PurviewManagementClient
    from azure.mgmt.purview.models import *
    from datetime import datetime, timedelta
    import time
    ```

2. Добавьте в метод **Main** приведенный ниже код, создающий экземпляр класса DataFactoryManagementClient. Этот объект используется для создания учетной записи Purview, удаления учетной записи Purview, проверки доступности имени и других операций поставщика ресурсов.
 
 ```python
    def main():
    
    # Azure subscription ID
    subscription_id = '<subscription ID>'
    
    # This program creates this resource group. If it's an existing resource group, comment out the code that creates the resource group
    rg_name = '<resource group>'

    # The purview name. It must be globally unique.
    purview_name = '<purview account name>'

    # Location name, where Purview account must be created.
    location = '<location name>'    

    # Specify your Active Directory client ID, client secret, and tenant ID
    credentials = ClientSecretCredential(client_id='<service principal ID>', client_secret='<service principal key>', tenant_id='<tenant ID>') 
    # resource_client = ResourceManagementClient(credentials, subscription_id)
    purview_client = PurviewManagementClient(credentials, subscription_id)
    ```

## <a name="create-a-purview-account"></a>Создание учетной записи Purview

Добавьте следующий код, создающий **учетную запись Purview**, в метод **Main**. Если группа ресурсов уже существует, закомментируйте первую инструкцию `create_or_update`.

```python
    # create the resource group
    # comment out if the resource group already exits
    resource_client.resource_groups.create_or_update(rg_name, rg_params)

    #Create a purview
    identity = Identity(type= "SystemAssigned")
    sku = AccountSku(name= 'Standard', capacity= 4)
    purview_resource = Account(identity=identity,sku=sku,location =location )
       
    try:
        pa = (purview_client.accounts.begin_create_or_update(rg_name, purview_name, purview_resource)).result()
        print("location:", pa.location, " Purview Account Name: ", pa.name, " Id: " , pa.id ," tags: " , pa.tags)  
    except:
        print("Error")
        print_item(pa)
 
    while (getattr(pa,'provisioning_state')) != "Succeeded" :
        pa = (purview_client.accounts.get(rg_name, purview_name))  
        print(getattr(pa,'provisioning_state'))
        if getattr(pa,'provisioning_state') != "Failed" :
            print("Error in creating Purview account")
            break
        time.sleep(30)      
        
```

Теперь добавьте следующую инструкцию, чтобы метод **main** вызывался при запуске программы:

```python
# Start the main method
main()
```

## <a name="full-script"></a>Полный сценарий

Ниже приведен полный код Python:

```python
    
    from azure.identity import ClientSecretCredential 
    from azure.mgmt.resource import ResourceManagementClient
    from azure.mgmt.purview import PurviewManagementClient
    from azure.mgmt.purview.models import *
    from datetime import datetime, timedelta
    import time
    
     # Azure subscription ID
    subscription_id = '<subscription ID>'
    
    # This program creates this resource group. If it's an existing resource group, comment out the code that creates the resource group
    rg_name = '<resource group>'

    # The purview name. It must be globally unique.
    purview_name = '<purview account name>'

    # Specify your Active Directory client ID, client secret, and tenant ID
    credentials = ClientSecretCredential(client_id='<service principal ID>', client_secret='<service principal key>', tenant_id='<tenant ID>') 
    # resource_client = ResourceManagementClient(credentials, subscription_id)
    purview_client = PurviewManagementClient(credentials, subscription_id)
    
    # create the resource group
    # comment out if the resource group already exits
    resource_client.resource_groups.create_or_update(rg_name, rg_params)

    #Create a purview
    identity = Identity(type= "SystemAssigned")
    sku = AccountSku(name= 'Standard', capacity= 4)
    purview_resource = Account(identity=identity,sku=sku,location ="southcentralus" )
       
    try:
        pa = (purview_client.accounts.begin_create_or_update(rg_name, purview_name, purview_resource)).result()
        print("location:", pa.location, " Purview Account Name: ", purview_name, " Id: " , pa.id ," tags: " , pa.tags) 
    except:
        print("Error in submitting job to create account")
        print_item(pa)
 
    while (getattr(pa,'provisioning_state')) != "Succeeded" :
        pa = (purview_client.accounts.get(rg_name, purview_name))  
        print(getattr(pa,'provisioning_state'))
        if getattr(pa,'provisioning_state') != "Failed" :
            print("Error in creating Purview account")
            break
        time.sleep(30)    

# Start the main method
main()
```

## <a name="run-the-code"></a>Выполнение кода

Создайте и запустите приложение, а затем проверьте выполнение конвейера.

Консоль выведет ход выполнения создания фабрики данных, связанной службы, наборов данных, конвейера и выполнения конвейера. Дождитесь появления сведений о действии копирования с размером записанных и прочитанных данных. Затем воспользуйтесь такими средствами, как [обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/), чтобы проверить, скопирован ли большой двоичный объект в outputBlobPath из inputBlobPath, как указано в переменных.

Пример выходных данных:

```console
location: southcentralus  Purview Account Name:  purviewpython7  Id:  /subscriptions/8c2c7b23-848d-40fe-b817-690d79ad9dfd/resourceGroups/Demo_Catalog/providers/Microsoft.Purview/accounts/purviewpython7  tags:  None
Creating
Creating
Succeeded
```

## <a name="verify-the-output"></a>Проверка выходных данных

Перейдите на страницу **учетных записей Purview** в портале Azure и проверьте учетную запись, созданную с помощью приведенного выше кода. 

## <a name="delete-purview-account"></a>Удаление учетной записи Purview

Чтобы удалить учетную запись Purview, добавьте в программу следующий код:

```python
pa = purview_client.accounts.begin_delete(rg_name, purview_name).result()
```

## <a name="next-steps"></a>Дальнейшие действия

Код в этом руководстве создает учетную запись Purview и удаляет учетную запись Purview. Теперь вы можете скачать пакет SDK для Python и узнать о других действиях поставщика ресурсов, которые можно выполнить для учетной записи Purview.

Перейдите к следующей статье, чтобы узнать, как разрешить пользователям доступ к учетной записи Azure Purview. 

> [!div class="nextstepaction"]
> [Добавление пользователей в учетную запись Azure Purview](catalog-permissions.md)
