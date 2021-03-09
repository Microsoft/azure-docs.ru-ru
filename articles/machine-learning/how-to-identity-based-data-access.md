---
title: Доступ к данным на основе удостоверений к службам хранилища в Azure
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать доступ к данным на основе удостоверений для подключения к службам хранилища в Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
ms.reviewer: nibaccam
ms.date: 02/22/2021
ms.custom: how-to, contperf-fy21q1, devx-track-python, data4ml
ms.openlocfilehash: 68d07481e228b1d1b2f4571a783f925add261cff
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102520019"
---
# <a name="connect-to-storage-with-identity-based-data-access-preview"></a>Подключение к хранилищу с доступом к данным на основе удостоверений (Предварительная версия)

>[!IMPORTANT]
> Функциональные возможности, представленные в этой статье, доступны в предварительной версии, и их следует рассматривать как [экспериментальные](/python/api/overview/azure/ml/#stable-vs-experimental) функции предварительной версии, которые могут измениться в любое время.

Из этой статьи вы узнаете, как подключиться к службам хранилища в Azure с помощью доступа к данным на основе удостоверений и Машинное обучение Azure хранилища данных с помощью [пакета SDK для Python для машинное обучение Azure](/python/api/overview/azure/ml/intro).  

Обычно в хранилищах данных используется доступ к данным на основе учетных данных для подтверждения того, что у вас есть разрешение на доступ к службе хранилища. Они сохраняют сведения о подключении, такие как идентификатор подписки и авторизация маркера, в [Key Vault](https://azure.microsoft.com/services/key-vault/) , связанном с рабочей областью. При создании хранилища данных, использующего доступ к данным на основе удостоверений, имя входа Azure ([маркер Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md)) используется для подтверждения того, что у вас есть разрешение на доступ к службе хранилища. В этом сценарии учетные данные для проверки подлинности не сохраняются, и в хранилище данных хранятся только сведения об учетной записи хранения. 

Сведения о создании хранилищ данных, использующих проверку подлинности на основе учетных записей, например ключи доступа или субъекты-службы, см. в статье [Подключение к службам хранилища в Azure](how-to-access-data.md).

## <a name="identity-based-data-access-in-azure-machine-learning"></a>Доступ к данным на основе удостоверений в Машинное обучение Azure

Существует две области применения доступа к данным на основе удостоверений в Машинное обучение Azure. Особенно при работе с конфиденциальными данными и необходимости более детального управления доступом к данным. 

1. Доступ к службам хранилища.
1. Обучение моделей машинного обучения с частными данными.

### <a name="accessing-storage-services"></a>Доступ к службам хранилища

Вы можете подключаться к службам хранилища через доступ к данным на основе удостоверений с помощью Машинное обучение Azure хранилищ данных или [машинное обучение Azure наборов данных](how-to-create-register-datasets.md). 

Обычно учетные данные для проверки подлинности хранятся в хранилище данных, которое используется для обеспечения разрешения на доступ к службе хранилища. Если эти учетные данные зарегистрированы в хранилищах данных, любой пользователь с ролью *читателя* рабочей области сможет получить их, что может быть проблемой безопасности в некоторых организациях. Дополнительные [сведения о роли *читателя* рабочей области](how-to-assign-roles.md#default-roles). 

При использовании доступа к данным на основе удостоверений Машинное обучение Azure запрашивает маркер Azure Active Directory для проверки подлинности доступа к данным, вместо того чтобы хранить учетные данные в хранилище данных. Что позволяет управлять доступом к данным на уровне хранилища и сохранять конфиденциальные учетные данные. 

Такое же поведение применяется при

* [Создайте набор данных непосредственно из URL-адресов хранилища](#use-data-in-storage). 
* Интерактивно работать с данными через записную книжку Jupyter на локальном компьютере или в [вычислительном экземпляре](concept-compute-instance.md).

> [!NOTE]
> Учетные данные, хранимые с помощью проверки подлинности на основе учетных данных: идентификатор подписки, маркеры подписанного URL-адрес (SAS), ключи доступа к хранилищу и сведения о субъекте-службе, например идентификатор клиента и идентификатор

### <a name="model-training-on-private-data"></a>Обучение модели на частных данных

В некоторых сценариях машинного обучения задействованы модели обучения с частными данными. В таких случаях специалисты по обработке и анализу данных должны запускать рабочие процессы обучения без раскрытия конфиденциальных данных. В этом сценарии для проверки подлинности доступа к данным используется управляемое удостоверение учебного вычислений. Таким образом, Администраторы хранилища могут предоставлять доступ **читателя данных BLOB-объекта хранилища** к управляемому удостоверению, используемому в процессе обучения для выполнения учебного задания, а не отдельных специалистов по анализу данных. Узнайте, как [настроить управляемое удостоверение для вычислений](how-to-create-attach-compute-cluster.md#managed-identity).


## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Попробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree).

- Учетная запись хранения Azure с поддерживаемым типом хранилища. В предварительной версии поддерживаются следующие типы хранилищ. 
    - [Хранилище BLOB-объектов Azure](../storage/blobs/storage-blobs-overview.md)
    - [Azure Data Lake Gen 1](../data-lake-store/index.yml)
    - [Azure Data Lake Gen 2](../storage/blobs/data-lake-storage-introduction.md)
    - [База данных SQL Azure](../azure-sql/database/sql-database-paas-overview.md)

- [Пакет SDK для машинное обучение Azure для Python](/python/api/overview/azure/ml/install).

- Рабочая область машинного обучения Azure.
  
  [Создайте машинное обучение Azure рабочую область](how-to-manage-workspace.md) или используйте существующую, [используя пакет SDK для Python](how-to-manage-workspace.md#connect-to-a-workspace). 

## <a name="storage-access-permissions"></a>Разрешения на доступ к хранилищу

Чтобы обеспечить безопасное подключение к службе хранилища в Azure, Машинное обучение Azure требуется разрешение на доступ к соответствующему хранилищу данных.

Доступ к данным на основе удостоверений поддерживает только подключения к следующим службам хранилища:

* хранилище BLOB-объектов Azure
* Поколение Azure Data Lake 1
* Поколение Azure Data Lake 2
* База данных SQL Azure

Для доступа к этим службам хранилища необходимо иметь по крайней мере доступ на **Чтение данных BLOB-объекта хранилища** . См. Дополнительные сведения о [модуле чтения BLOB-объектов хранилища](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Только владельцы учетных записей хранения могут [изменять уровень доступа с помощью портал Azure](../storage/common/storage-auth-aad-rbac-portal.md).

При обучении модели на удаленном целевом объекте вычислений удостоверение вычислений должно быть предоставлено по крайней мере с ролью **читателя данных BLOB-объекта хранилища** из службы хранилища. Узнайте, как [настроить управляемое удостоверение для вычислений](how-to-create-attach-compute-cluster.md#managed-identity).

## <a name="work-with-virtual-networks"></a>Работа с виртуальными сетями

По умолчанию Машинное обучение Azure не может взаимодействовать с учетной записью хранения, которая находится за брандмауэром или в виртуальной сети.

Учетные записи хранения можно настроить для разрешения доступа только из конкретных виртуальных сетей, что требует дополнительных конфигураций, чтобы гарантировать, что данные не будут утечки за пределы сети. Такое поведение одинаково для доступа к данным на основе учетных данных, Узнайте, [какие конфигурации необходимы и как их применять для сценариев виртуальной сети](how-to-access-data.md#virtual-network). 

## <a name="create-and-register-datastores"></a>Создание и регистрация хранилищ данных

При регистрации службы хранилища в Azure в качестве хранилища данных вы автоматически создаете и регистрируете хранилище данных в определенной рабочей области. Дополнительные сведения о подключении к хранилищу данных за счет использования виртуальных сетей см. в следующих разделах: [разрешения на доступ к хранилищу](#storage-access-permissions) для указания требуемых типов разрешений и [Работа с виртуальной сетью](#work-with-virtual-networks) .

В следующем коде обратите внимание на отсутствие параметров проверки подлинности, таких как `sas_token` ,, `account_key` `subscription_id` или субъект-служба `client_id` . Это означает, что Машинное обучение Azure использует доступ к данным на основе удостоверений для проверки подлинности. Так как создание хранилищ данных обычно происходит в интерактивном режиме в записной книжке или в студии, маркер Azure Active Directory используется для проверки подлинности доступа к данным.

> [!NOTE]
> Имена хранилищ данных должны состоять только из строчных букв, цифр и знаков подчеркивания. 

### <a name="azure-blob-container"></a>Контейнер BLOB-объектов Azure

Чтобы зарегистрировать контейнер BLOB-объектов в качестве хранилища данных, используйте [`register_azure_blob_container()`](/python/api/azureml-core/azureml.core.datastore%28class%29#register-azure-blob-container-workspace--datastore-name--container-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false--blob-cache-timeout-none--grant-workspace-access-false--subscription-id-none--resource-group-none-).

Следующий код создает и регистрирует `credentialless_blob` хранилище данных в `ws` рабочей области и присваивает его переменной `blob_datastore` . Это хранилище данных обращается к `my_container_name` контейнеру больших двоичных объектов в `my-account-name` учетной записи хранения.

```Python
# create blob datastore without credentials
blob_datastore = Datastore.register_azure_blob_container(workspace=ws,
                                                      datastore_name='credentialless_blob',
                                                      container_name='my_container_name',
                                                      account_name='my_account_name')
```

### <a name="azure-data-lake-storage-generation-1"></a>Поколение Azure Data Lake Storage 1

Для хранилища данных Azure Data Lake Storage поколения 1 (ADLS Gen 1) используйте [register_azure_data_lake ()](/python/api/azureml-core/azureml.core.datastore.datastore#register-azure-data-lake-workspace--datastore-name--store-name--tenant-id-none--client-id-none--client-secret-none--resource-url-none--authority-url-none--subscription-id-none--resource-group-none--overwrite-false--grant-workspace-access-false-) , чтобы зарегистрировать хранилище, которое подключается к хранилищу данных Azure с поколением 1.

Следующий код создает и регистрирует `credentialless_adls1` хранилище данных в `workspace` рабочей области и присваивает его переменной `adls_dstore` . Это хранилище данных получает доступ к `adls_storage` учетной записи хранения Azure Data Lake Store.

```Python
# create adls gen1 without credentials
adls_dstore = Datastore.register_azure_data_lake(workspace = workspace,
                                                 datastore_name='credentialless_adls1',
                                                 store_name='adls_storage')

```

### <a name="azure-data-lake-storage-generation-2"></a>Azure Data Lake Storage 2-го поколения

Для хранилища данных Azure Data Lake Storage Generation 2 (ADLS Gen 2) используйте [register_azure_data_lake_gen2 ()](/python/api/azureml-core/azureml.core.datastore.datastore#register-azure-data-lake-gen2-workspace--datastore-name--filesystem--account-name--tenant-id--client-id--client-secret--resource-url-none--authority-url-none--protocol-none--endpoint-none--overwrite-false-) , чтобы зарегистрировать хранилище данных, которое подключается к хранилищу Azure Data Lake 2.

Следующий код создает и регистрирует `credentialless_adls2` хранилище данных в `ws` рабочей области и присваивает его переменной `adls2_dstore` . Это хранилище данных обращается к файловой системе `tabular` в `myadls2` учетной записи хранения.  

```python
# createn adls2 datastore without credentials
adls2_dstore = Datastore.register_azure_data_lake_gen2(workspace=ws, 
                                                       datastore_name='credentialless_adls2', 
                                                       filesystem='tabular', 
                                                       account_name='myadls2')
```

## <a name="use-data-in-storage"></a>Использование данных в хранилище

[Машинное обучение Azure наборы](how-to-create-register-datasets.md) данных — это рекомендуемый способ взаимодействия с данными в хранилище с машинное обучение Azure. 

Наборы данных упаковывают ваши данные в отложенно вычисляемый объект для задач машинного обучения, например для обучения. Кроме того, с помощью наборов данных можно [загрузить или подключить](how-to-train-with-datasets.md#mount-vs-download) файлы любого формата из служб хранилища Azure, таких как, хранилище BLOB-объектов Azure и озера данных Azure, к целевому объекту вычислений.


**Для создания наборов данных с доступом к данным на основе удостоверений** доступны следующие варианты. Этот тип создания набора данных использует маркер Azure Active Directory для проверки подлинности доступа к данным. 

*  Пути для ссылок из хранилищ данных, которые также используют доступ к данным на основе удостоверений. 
<br>В следующем примере `blob_datastore` ранее был создан с использованием доступа к данным на основе удостоверений.   

    ```python
    blob_dataset = Dataset.Tabular.from_delimited_files(blob_datastore,'test.csv') 
    ```

* Пропускать создание и создание наборов данных непосредственно из URL-адресов хранилища. В настоящее время эта функция поддерживает только большие двоичные объекты Azure и Azure Data Lake Storage поколения 1 и 2.

    ```python
    blob_dset = Dataset.File.from_files('https://myblob.blob.core.windows.net/may/keras-mnist-fashion/')
    ```

**Однако при отправке учебного задания, использующего набор данных, созданный с доступом к данным на основе удостоверений**, управляемое удостоверение обучающего вычислений используется для проверки подлинности доступа к данным вместо токена Azure Active Directory. Для этого сценария убедитесь, что управляемому удостоверению вычислений предоставлено по крайней мере роль **читателя данных BLOB-объекта хранилища** из службы хранилища. Узнайте, как [настроить управляемое удостоверение для вычислений](how-to-create-attach-compute-cluster.md#managed-identity). 

## <a name="next-steps"></a>Дальнейшие действия

* [Создайте набор данных машинного обучения Azure](how-to-create-register-datasets.md).
* [Обучение с наборами данных](how-to-train-with-datasets.md).
* [Создание хранилища данных с доступом к данным на основе ключей](how-to-access-data.md).
