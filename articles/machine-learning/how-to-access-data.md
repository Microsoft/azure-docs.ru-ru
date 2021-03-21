---
title: Подключение к службам хранилища в Azure
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать хранилища данных для безопасного подключения к службам хранилища Azure во время обучения с помощью Службы машинного обучения Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
ms.reviewer: nibaccam
ms.date: 11/03/2020
ms.custom: how-to, contperf-fy21q1, devx-track-python, data4ml
ms.openlocfilehash: 78b7bab204a08b474ea3c5cf5c2f7735c019a9c3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102519934"
---
# <a name="connect-to-storage-services-on-azure"></a>Подключение к службам хранилища в Azure

В этой статье вы узнаете, как подключиться к службам хранилища данных в Azure с помощью Машинное обучение Azure хранилищ и [пакета SDK для машинное обучение Azure Python](/python/api/overview/azure/ml/intro).

Хранилища данных безопасно подключаются к службе хранилища в Azure, не заключая учетные данные проверки подлинности и целостность исходного источника. Они хранят сведения о подключении, такие как идентификатор подписки и авторизация маркера, в [Key Vault](https://azure.microsoft.com/services/key-vault/) , связанном с рабочей областью, чтобы обеспечить безопасный доступ к хранилищу без необходимости жесткого кодирования в скриптах. Вы можете создавать хранилища данных, которые подключаются к [этим решениям службы хранилища Azure](#matrix).

Сведения о работе с хранилищами данных в общем рабочем процессе доступа к данным в Машинном обучение Azure см. в статье о [безопасном доступе к данным](concept-data.md#data-workflow).

Сведения о низком уровне кода см. в статье Использование [машинное обучение Azure Studio для создания и регистрации хранилищ данных](how-to-connect-data-ui.md#create-datastores).

>[!TIP]
> В этой статье предполагается, что вы хотите подключиться к службе хранилища с использованием учетных данных проверки подлинности на основе учетных данных, например субъекта-службы или маркера подписанного URL-доступа (SAS). Помните, что если учетные данные зарегистрированы в хранилищах данных, все пользователи с ролью *читателя* рабочей области могут получить эти учетные данные. [Дополнительные сведения о роли *читателя* рабочей области.](how-to-assign-roles.md#default-roles) <br><br>Если это проблема, Узнайте, как [подключиться к службам хранилища с доступом на основе удостоверений](how-to-identity-based-data-access.md). <br><br>Эта возможность является [экспериментальной](/python/api/overview/azure/ml/#stable-vs-experimental) функцией предварительной версии и может быть изменена в любое время. 

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Попробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree).

- Учетная запись хранения Azure с [поддерживаемым типом хранилища](#matrix).

- [Пакет SDK для машинное обучение Azure для Python](/python/api/overview/azure/ml/intro).

- Рабочая область машинного обучения Azure.
  
  [Создайте рабочую область Машинного обучения Azure](how-to-manage-workspace.md) или используйте существующую с помощью пакета SDK для Python. 

    Импортируйте классы `Workspace` и `Datastore` и загрузите сведения о подписке из файла `config.json` с помощью функции `from_config()`. Эта функция выполняет поиск файла JSON по умолчанию в текущем каталоге, но можно также указать параметр path, указывающий на файл, с помощью `from_config(path="your/file/path")`.

   ```Python
   import azureml.core
   from azureml.core import Workspace, Datastore
        
   ws = Workspace.from_config()
   ```

    При создании рабочей области контейнер больших двоичных объектов Azure и файловый ресурс Azure автоматически регистрируются в качестве хранилищ данных в рабочей области. Они называются `workspaceblobstore` и `workspacefilestore`, соответственно. `workspaceblobstore`Используется для хранения артефактов рабочей области и журналов экспериментов машинного обучения. Он также задается в качестве **хранилища данных по умолчанию** и не может быть удален из рабочей области. `workspacefilestore`Используется для хранения записных книжек и скриптов R, авторизация которых осуществляется с помощью [вычислительного экземпляра](./concept-compute-instance.md#accessing-files).
    
    > [!NOTE]
    > Конструктор Машинное обучение Azure создаст хранилище данных с именем **azureml_globaldatasets** автоматически при открытии образца на домашней странице конструктора. В этом хранилище данных содержатся только примеры наборов данных. **Не** используйте это хранилище данных для доступа к конфиденциальным данным.

<a name="matrix"></a>

## <a name="supported-data-storage-service-types"></a>Поддерживаемые типы служб хранилища данных

В настоящее время хранилища данных поддерживают хранение сведений о подключении к службам хранилищ, указанных в следующей таблице. 

> [!TIP]
> **Для неподдерживаемых решений для хранения** данных и для сохранения затрат на исходящие данные во время экспериментов в машинном [коде перенесите данные](#move) в поддерживаемое решение службы хранилища Azure. 

| Тип&nbsp;хранилища | Тип&nbsp;проверки подлинности | [Студия&nbsp;машинного&nbsp;обучения Azure](https://ml.azure.com/) | [Пакет SDK для&nbsp;машинного&nbsp;обучения Azure для&nbsp;Python](/python/api/overview/azure/ml/intro) |  [CLI&nbsp;машинного&nbsp;обучения Azure](reference-azure-machine-learning-cli.md) | [Rest API&nbsp;машинного&nbsp;обучения&nbsp;Azure](/rest/api/azureml/) | VS Code
---|---|---|---|---|---|---
[Хранилище&nbsp;BLOB-объектов&nbsp;Azure](../storage/blobs/storage-blobs-overview.md)| Ключ учетной записи <br> Маркер SAS | ✓ | ✓ | ✓ |✓ |✓
[Общая&nbsp;папка&nbsp;Azure](../storage/files/storage-files-introduction.md)| Ключ учетной записи <br> Маркер SAS | ✓ | ✓ | ✓ |✓|✓
[Azure&nbsp;Data Lake&nbsp;Storage 1-го&nbsp;поколения](../data-lake-store/index.yml)| Субъект-служба| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;Data Lake&nbsp;Storage 2-го&nbsp;поколения](../storage/blobs/data-lake-storage-introduction.md)| Субъект-служба| ✓ | ✓ | ✓ |✓|
[База данных&nbsp;SQL&nbsp;Azure](../azure-sql/database/sql-database-paas-overview.md)| Проверка подлинности SQL <br>Субъект-служба| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;PostgreSQL](../postgresql/overview.md) | Проверка подлинности SQL| ✓ | ✓ | ✓ |✓|
[База данных&nbsp;Azure&nbsp;для&nbsp;MySQL](../mysql/overview.md) | Проверка подлинности SQL|  | ✓* | ✓* |✓*|
[Файловая&nbsp;система&nbsp;Databricks](/azure/databricks/data/databricks-file-system)| без аутентификации; | | ✓** | ✓ ** |✓** |

\* MySQL поддерживается только для конвейера [дататрансферстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep)<br />
\*\*Модуль обработки блоков обработки информации поддерживается только для [датабрикксстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricks_step.databricksstep) конвейера


### <a name="storage-guidance"></a>Рекомендации по выбору хранилища

Рекомендуется создать хранилище данных для [контейнера BLOB-объектов Azure](../storage/blobs/storage-blobs-introduction.md). Для BLOB-объектов доступны хранилища класса Standard и Premium. Хотя хранилище класса Premium является более дорогим, за счет его высокой пропускной способности можно повысить скорость выполнения обучения, особенно при обучении на основе большого набора данных. О стоимости учетных записей хранения можно узнать на странице [Калькулятор цен](https://azure.microsoft.com/pricing/calculator/?service=machine-learning-service).

[Azure Data Lake Storage 2-го поколения](../storage/blobs/data-lake-storage-introduction.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) создано на основе хранилища BLOB-объектов Azure и предназначено для анализа больших данных в организации. Основная часть Data Lake Storage 2-го поколения — это добавление [иерархического пространства имен](../storage/blobs/data-lake-storage-namespace.md) в хранилище BLOB-объектов. Иерархическое пространство имен позволяет упорядочивать объекты и файлы в иерархии каталогов для эффективного доступа к данным.

## <a name="storage-access-and-permissions"></a>Доступ к хранилищу и разрешения

Чтобы обеспечить безопасное подключение к службе хранилища Azure, Машинное обучение Azure требуется разрешение на доступ к соответствующему контейнеру хранилища данных. Этот доступ зависит от учетных данных проверки подлинности, используемых для регистрации хранилища данных. 

### <a name="virtual-network"></a>Виртуальная сеть 

По умолчанию Машинное обучение Azure не может взаимодействовать с учетной записью хранения, которая находится за брандмауэром или в виртуальной сети. Если учетная запись хранения данных находится в **виртуальной сети**, необходимо выполнить дополнительные действия по настройке, чтобы обеспечить машинное обучение Azure доступ к данным. 

> [!NOTE]
> Это руководство также применяется к [хранилищам данных, созданным с помощью доступа к данным на основе удостоверений (Предварительная версия)](how-to-identity-based-data-access.md). 

Для **пользователей пакета Python SDK**, чтобы получить доступ к данным с помощью сценария обучения на целевом объекте вычислений, целевой объект вычислений должен находиться в одной виртуальной сети и подсети хранилища.  

**Для пользователей машинное обучение Azure Studio** несколько функций полагаются на возможность чтения данных из набора данных. такие как предварительные версии наборов данных, профили и автоматизированное машинное обучение. Чтобы эти функции работали с хранилищем за пределами виртуальных сетей, используйте [управляемое удостоверение рабочей области в студии](how-to-enable-studio-virtual-network.md) , чтобы разрешить машинное обучение Azure доступ к учетной записи хранения извне виртуальной сети. 

Машинное обучение Azure может принимать запросы от клиентов за пределами виртуальной сети. Чтобы убедиться, что сущность, запрашивающая данные из службы, является надежной, [Настройте частную связь Azure для вашей рабочей области](how-to-configure-private-link.md).

### <a name="access-validation"></a>Проверка доступа

В ходе **процесса создания и регистрации хранилища данных** машинное обучение Azure автоматически проверяет, существует ли базовая служба хранилища, и предоставленный пользователем участник (имя пользователя, субъект-служба или маркер SAS) имеет доступ к указанному хранилищу.

**После создания хранилища данных** эта проверка выполняется только для методов, которым требуется доступ к базовому контейнеру хранилища, а **не** при извлечении объектов хранилища данных. Например, проверка выполняется, если требуется скачать файлы из хранилища данных. Но если вы просто хотите изменить хранилище данных по умолчанию, проверки не будет.

Для проверки подлинности доступа к базовой службе хранилища можно указать ключ учетной записи, маркеры общего доступа (SAS) или субъект-службу в соответствующем `register_azure_*()` методе создаваемого типа хранилища данных. В [матрице типа хранилища](#matrix) перечислены поддерживаемые типы проверки подлинности, соответствующие каждому типу хранилища данных.

Сведения о ключе учетной записи, маркере SAS и субъекте-службе можно найти на [портал Azure](https://portal.azure.com).

* Если для проверки подлинности вы планируете использовать ключ учетной записи или маркер SAS, выберите элемент **Учетные записи хранения** на панели слева и выберите учетную запись хранения, которую требуется зарегистрировать. 
  * На странице **Обзор** приводятся такие сведения, как имя учетной записи, контейнер и имя общей папки. 
      1. Чтобы использовать ключи учетной записи, перейдите к элементу **Ключи доступа** на панели **Параметры**. 
      1. Чтобы использовать маркеры SAS, перейдите к элементу **Подписанные URL-адреса** на панели **Параметры**.

* Если вы планируете использовать субъект-службу для проверки подлинности, перейдите к **Регистрация приложений** и выберите приложение, которое вы хотите использовать. 
    * Соответствующая страница **обзора** будет содержать требуемые сведения, такие как идентификатор клиента и идентификатор клиента.

> [!IMPORTANT]
> * Если вам нужно изменить ключи доступа для учетной записи хранения Azure (ключ учетной записи или маркер SAS), не забудьте синхронизировать новые учетные данные с рабочей областью и хранилищами данных, подключенными к ней. Узнайте, как [синхронизировать обновленные учетные данные](how-to-change-storage-access-key.md). 
### <a name="permissions"></a>Разрешения

Для контейнера больших двоичных объектов Azure и Azure Data Lake хранилища Gen 2 Убедитесь, что учетные данные для проверки подлинности имеют доступ для **чтения BLOB-объектов хранилища** . См. Дополнительные сведения о [модуле чтения BLOB-объектов хранилища](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). По умолчанию маркер SAS учетной записи не имеет разрешений. 
* Для **доступа на чтение** данных учетные данные проверки подлинности должны иметь минимум из списка и разрешений на чтение для контейнеров и объектов. 

* Для **доступа на запись** данных также требуются разрешения на запись и добавление.

<a name="python"></a>

## <a name="create-and-register-datastores"></a>Создание и регистрация хранилищ данных

При регистрации решения для хранилища Azure в качестве хранилища данных вы автоматически создаете и регистрируете хранилище данных в определенной рабочей области. Дополнительные сведения о сценариях виртуальной сети и о том, где найти необходимые учетные данные для проверки подлинности, см. в разделе [разрешения & доступа к хранилищу](#storage-access-and-permissions) 

В этом разделе приведены примеры создания и регистрации хранилища данных с помощью пакета SDK для Python для следующих типов хранилищ. Параметры, приведенные в этих примерах, являются **обязательными** для создания и регистрации хранилища данных.

* [Контейнер BLOB-объектов Azure](#azure-blob-container)
* [Общая папка Azure](#azure-file-share)
* [Azure Data Lake Storage 2-го поколения](#azure-data-lake-storage-generation-2)

 Сведения о создании хранилищ данных для других поддерживаемых служб хранилища см. в [справочной документации по соответствующим `register_azure_*` методам](/python/api/azureml-core/azureml.core.datastore.datastore#methods).

Если вы предпочитаете работу с кодом, см. статью [Подключение к данным с помощью машинное обучение Azure Studio](how-to-connect-data-ui.md).
>[!IMPORTANT]
> Если вы отмените регистрацию и повторно зарегистрируете хранилище данных с тем же именем и оно завершится сбоем, Azure Key Vault для вашей рабочей области не сможет включить обратимое удаление. По умолчанию обратимое удаление выполняется для экземпляра хранилища ключей, созданного рабочей областью, но он может быть не включен, если вы использовали существующее хранилище ключей или создали рабочую область, созданную до октября 2020. Сведения о том, как включить обратимое удаление, см. в разделе [Включение обратимого удаления для существующего хранилища ключей]( https://docs.microsoft.com/azure/key-vault/general/soft-delete-change#turn-on-soft-delete-for-an-existing-key-vault).

> [!NOTE]
> Имя хранилища данных должно состоять только из строчных букв, цифр и знаков подчеркивания. 

### <a name="azure-blob-container"></a>Контейнер BLOB-объектов Azure

Чтобы зарегистрировать контейнер BLOB-объектов в качестве хранилища данных, используйте [`register_azure_blob_container()`](/python/api/azureml-core/azureml.core.datastore%28class%29#register-azure-blob-container-workspace--datastore-name--container-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false--blob-cache-timeout-none--grant-workspace-access-false--subscription-id-none--resource-group-none-).

Следующий код создает и регистрирует хранилище данных `blob_datastore_name` в рабочей области `ws`. Это хранилище данных обращается к контейнеру BLOB-объектов `my-container-name` в учетной записи хранения `my-account-name` с помощью предоставленного ключа доступа учетной записи. Дополнительные сведения о сценариях виртуальной сети и о том, где найти необходимые учетные данные для проверки подлинности, см. в разделе [разрешения & доступа к хранилищу](#storage-access-and-permissions) 

```Python
blob_datastore_name='azblobsdk' # Name of the datastore to workspace
container_name=os.getenv("BLOB_CONTAINER", "<my-container-name>") # Name of Azure blob container
account_name=os.getenv("BLOB_ACCOUNTNAME", "<my-account-name>") # Storage account name
account_key=os.getenv("BLOB_ACCOUNT_KEY", "<my-account-key>") # Storage account access key

blob_datastore = Datastore.register_azure_blob_container(workspace=ws, 
                                                         datastore_name=blob_datastore_name, 
                                                         container_name=container_name, 
                                                         account_name=account_name,
                                                         account_key=account_key)
```

### <a name="azure-file-share"></a>Общая папка Azure

Чтобы зарегистрировать общую папку Azure в качестве хранилища данных, используйте [`register_azure_file_share()`](/python/api/azureml-core/azureml.core.datastore%28class%29#register-azure-file-share-workspace--datastore-name--file-share-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false-). 

Следующий код создает и регистрирует хранилище данных `file_datastore_name` в рабочей области `ws`. Это хранилище данных обращается к общей папке `my-fileshare-name` в учетной записи хранения `my-account-name` с помощью предоставленного ключа доступа учетной записи. Дополнительные сведения о сценариях виртуальной сети и о том, где найти необходимые учетные данные для проверки подлинности, см. в разделе [разрешения & доступа к хранилищу](#storage-access-and-permissions) 

```Python
file_datastore_name='azfilesharesdk' # Name of the datastore to workspace
file_share_name=os.getenv("FILE_SHARE_CONTAINER", "<my-fileshare-name>") # Name of Azure file share container
account_name=os.getenv("FILE_SHARE_ACCOUNTNAME", "<my-account-name>") # Storage account name
account_key=os.getenv("FILE_SHARE_ACCOUNT_KEY", "<my-account-key>") # Storage account access key

file_datastore = Datastore.register_azure_file_share(workspace=ws,
                                                     datastore_name=file_datastore_name, 
                                                     file_share_name=file_share_name, 
                                                     account_name=account_name,
                                                     account_key=account_key)
```

### <a name="azure-data-lake-storage-generation-2"></a>Azure Data Lake Storage 2-го поколения

Для хранилища данных Azure Data Lake Storage 2-го поколения (ADLS 2-го поколения) используйте [register_azure_data_lake_gen2()](/python/api/azureml-core/azureml.core.datastore.datastore#register-azure-data-lake-gen2-workspace--datastore-name--filesystem--account-name--tenant-id--client-id--client-secret--resource-url-none--authority-url-none--protocol-none--endpoint-none--overwrite-false-) для регистрации базы данных учетных данных, подключенной к хранилищу Azure DataLake 2-го поколения, с [разрешениями субъекта-службы](../active-directory/develop/howto-create-service-principal-portal.md).  

Для использования субъекта-службы необходимо [зарегистрировать приложение](../active-directory/develop/app-objects-and-service-principals.md) и предоставить доступ к данным субъекта-службы с помощью управления доступом на основе ролей (Azure RBAC) или списков управления доступом (ACL). Узнайте больше о [контроле доступа в ADLS 2-го поколения](../storage/blobs/data-lake-storage-access-control-model.md). 

Следующий код создает и регистрирует хранилище данных `adlsgen2_datastore_name` в рабочей области `ws`. Это хранилище данных обращается к файловой системе `test` в учетной записи хранения `account_name` с помощью предоставленных учетных данных субъекта-службы. Дополнительные сведения о сценариях виртуальной сети и о том, где найти необходимые учетные данные для проверки подлинности, см. в разделе [разрешения & доступа к хранилищу](#storage-access-and-permissions) 

```python 
adlsgen2_datastore_name = 'adlsgen2datastore'

subscription_id=os.getenv("ADL_SUBSCRIPTION", "<my_subscription_id>") # subscription id of ADLS account
resource_group=os.getenv("ADL_RESOURCE_GROUP", "<my_resource_group>") # resource group of ADLS account

account_name=os.getenv("ADLSGEN2_ACCOUNTNAME", "<my_account_name>") # ADLS Gen2 account name
tenant_id=os.getenv("ADLSGEN2_TENANT", "<my_tenant_id>") # tenant id of service principal
client_id=os.getenv("ADLSGEN2_CLIENTID", "<my_client_id>") # client id of service principal
client_secret=os.getenv("ADLSGEN2_CLIENT_SECRET", "<my_client_secret>") # the secret of service principal

adlsgen2_datastore = Datastore.register_azure_data_lake_gen2(workspace=ws,
                                                             datastore_name=adlsgen2_datastore_name,
                                                             account_name=account_name, # ADLS Gen2 account name
                                                             filesystem='test', # ADLS Gen2 filesystem
                                                             tenant_id=tenant_id, # tenant id of service principal
                                                             client_id=client_id, # client id of service principal
                                                             client_secret=client_secret) # the secret of service principal
```



## <a name="create-datastores-with-other-azure-tools"></a>Создание хранилищ данных с помощью других средств Azure
Помимо создания хранилищ данных с помощью пакета SDK для Python и студии, можно также использовать Azure Resource Manager шаблоны или расширение Машинное обучение Azure VS Code. 

<a name="arm"></a>
### <a name="azure-resource-manager"></a>Azure Resource Manager

Существует ряд шаблонов [https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-datastore-create-*](https://github.com/Azure/azure-quickstart-templates/tree/master/) , которые можно использовать для создания хранилищ данных.

Сведения об использовании этих шаблонов см. в разделе [Использование шаблона Azure Resource Manager для создания рабочей области для машинное обучение Azure](how-to-create-workspace-template.md).

### <a name="vs-code-extension"></a>Расширения VS Code

Если вы предпочитаете создавать хранилища данных и управлять ими с помощью расширения Машинное обучение Azure VS Code, ознакомьтесь с [руководством по VS Code Resource Management](how-to-manage-resources-vscode.md#datastores) .
<a name="train"></a>
## <a name="use-data-in-your-datastores"></a>Используйте данные в хранилищах данных

После создания хранилища данных [создайте машинное обучение Azure набор данных](how-to-create-register-datasets.md) для взаимодействия с данными. Наборы данных упаковывают ваши данные в отложенно вычисляемый объект для задач машинного обучения, например для обучения. 

С помощью наборов данных можно [скачать или подключить](how-to-train-with-datasets.md#mount-vs-download) файлы любого формата из служб хранилища Azure для обучения модели на целевом объекте вычислений. [Узнайте больше о том, как обучить модели машинного обучения с помощью наборов данных](how-to-train-with-datasets.md).

<a name="get"></a>

## <a name="get-datastores-from-your-workspace"></a>Получение хранилищ данных из рабочей области

Чтобы получить конкретное хранилище данных, зарегистрированное в текущей рабочей области, используйте статический метод [`get()`](/python/api/azureml-core/azureml.core.datastore%28class%29#get-workspace--datastore-name-) в классе `Datastore`:

```Python
# Get a named datastore from the current workspace
datastore = Datastore.get(ws, datastore_name='your datastore name')
```
Чтобы получить список хранилищ данных, зарегистрированных в определенной рабочей области, можно использовать свойство [`datastores`](/python/api/azureml-core/azureml.core.workspace%28class%29#datastores) в объекте рабочей области:

```Python
# List all datastores registered in the current workspace
datastores = ws.datastores
for name, datastore in datastores.items():
    print(name, datastore.datastore_type)
```

Чтобы получить хранилище данных рабочей области по умолчанию, используйте следующую строку:

```Python
datastore = ws.get_default_datastore()
```
С помощью следующего кода можно изменить хранилище данных по умолчанию. Эта возможность поддерживается только в пакете SDK. 

```Python
 ws.set_default_datastore(new_default_datastore)
```

## <a name="access-data-during-scoring"></a>Доступ к данным во время оценки

Машинное обучение Azure предоставляет несколько способов использования моделей для оценки. Некоторые из этих методов не предоставляют доступ к хранилищам данных. В следующей таблице приводятся методы, позволяющие получить доступ к хранилищам данных во время оценки.

| Метод | Доступ к хранилищу данных | Описание |
| ----- | :-----: | ----- |
| [Прогнозирование в пакетном режиме](./tutorial-pipeline-batch-scoring-classification.md) | ✔ | Асинхронное создание прогнозов на основе больших объемов данных. |
| [Веб-служба](how-to-deploy-and-where.md) | &nbsp; | Развертывание моделей в качестве веб-служб. |
| [Модуль Azure IoT Edge](how-to-deploy-and-where.md) | &nbsp; | Развертывание моделей на устройствах IoT Edge. |

В случаях, когда пакет SDK не предоставляет доступ к хранилищам данных, можно создать пользовательский код для доступа к данным с помощью соответствующего пакета SDK Azure Например, [пакет SDK для хранилища Azure для Python](https://github.com/Azure/azure-storage-python) является клиентской библиотекой, подходящей для доступа к данным, хранящимся в BLOB-объектах или файлах.

<a name="move"></a>

## <a name="move-data-to-supported-azure-storage-solutions"></a>Перемещение данных в поддерживаемые решения хранилища Azure

Машинное обучение Azure поддерживает доступ к данным из хранилища BLOB-объектов Azure, Файлов Azure, Azure Data Lake Storage 1-го поколения, Azure Data Lake Storage 2-го поколения, базы данных SQL Azure и базы данных Azure для PostgreSQL. Если вы используете неподдерживаемое хранилище, рекомендуется переместить данные в поддерживаемые решения хранилища Azure с помощью [фабрики данных Azure и следующих действий](../data-factory/quickstart-create-data-factory-copy-data-tool.md). Перемещение данных в поддерживаемое хранилище поможет сократить расходы на исходящие данные во время экспериментов машинного обучения. 

Фабрика данных Azure обеспечивает эффективную и стабильную передачу данных с помощью более чем 80 предварительно подготовленных соединителей без дополнительных затрат. К этим соединителям относятся службы данных Azure, локальные источники данных, Amazon S3, Redshift и Google BigQuery.

## <a name="next-steps"></a>Дальнейшие действия

* [Создание набора данных Машинного обучения Azure](how-to-create-register-datasets.md)
* [Обучение модели](how-to-set-up-training-targets.md)
* [Развертывание модели](how-to-deploy-and-where.md)