---
title: Получение доступа к Хранилищу BLOB-объектов Azure с помощью Azure Databricks и Azure Key Vault
description: Из этого руководства вы узнаете, как получить доступ к Хранилищу BLOB-объектов Azure из Azure Databricks с использованием секретов, которые хранятся в Azure Key Vault.
author: msmbaldwin
ms.author: mbaldwin
ms.service: key-vault
ms.topic: tutorial
ms.date: 06/16/2020
ms.openlocfilehash: be1b0da23841b5a63ec044e04a5465e29345f9d1
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772205"
---
# <a name="tutorial-access-azure-blob-storage-using-azure-databricks-and-azure-key-vault"></a>Руководство по Получение доступа к Хранилищу BLOB-объектов Azure с помощью Azure Databricks и Azure Key Vault

Из этого руководства вы узнаете, как получить доступ к Хранилищу BLOB-объектов Azure из Azure Databricks с использованием секретов, которые хранятся в Azure Key Vault. 

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создание контейнера больших двоичных объектов и учетной записи хранения с помощью Azure CLI;
> * создание Key Vault и настройка секрета;
> * создание рабочей области Azure Databricks и добавление области секретов Key Vault;
> * получение доступа к контейнеру больших двоичных объектов из рабочей области Azure Databricks.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

Перед началом работы с этим руководством установите [Azure CLI](/cli/azure/install-azure-cli-windows).

## <a name="create-a-storage-account-and-blob-container-with-azure-cli"></a>Создание контейнера больших двоичных объектов и учетной записи хранения с помощью Azure CLI

Для использования больших двоичных объектов сначала необходимо создать учетную запись хранения общего назначения. Если у вас нет [группы ресурсов](/cli/azure/group#az_group_create), создайте ее перед выполнением команды. Приведенная ниже команда позволяет создать и отобразить метаданные контейнера хранилища. Скопируйте указанный здесь **идентификатор**.

```azurecli
az storage account create --name contosoblobstorage5 --resource-group contosoResourceGroup --location eastus --sku Standard_ZRS --encryption-services blob
```

![Вывод в консоль при выполнении указанной выше команды. Идентификатор выделен зеленым цветом для удобства пользователя.](../media/databricks-command-output-1.png)

Прежде чем создать контейнер, в который вы отправите большой двоичный объект, необходимо назначить себе роль [Участник данных BLOB-объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor). В нашем примере эта роль будет назначена ранее созданной учетной записи хранения.

```azurecli
az role assignment create --role "Storage Blob Data Contributor" --assignee t-trtr@microsoft.com --scope "/subscriptions/885e24c8-7a36-4217-b8c9-eed31e110504/resourceGroups/contosoResourceGroup5/providers/Microsoft.Storage/storageAccounts/contosoblobstorage5
```

Теперь, когда вы назначили роль учетной записи хранения, можете создать контейнер для большого двоичного объекта.

```azurecli
az storage container create --account-name contosoblobstorage5 --name contosocontainer5 --auth-mode login
```

После создания контейнера в него можно передать большой двоичный объект (любой файл на ваш выбор). В нашем примере это TXT-файл с текстом helloworld.

```azurecli
az storage blob upload --account-name contosoblobstorage5 --container-name contosocontainer5 --name helloworld --file helloworld.txt --auth-mode login
```

Получите список больших двоичных объектов в контейнере и убедитесь, что этот файл находится в контейнере.

```azurecli
az storage blob list --account-name contosoblobstorage5 --container-name contosocontainer5 --output table --auth-mode login
```

![Вывод в консоль при выполнении указанной выше команды. Она отображает файл, который вы только что сохранили в контейнере.](../media/databricks-command-output-2.png)

Получите значение **key1** для контейнера хранилища с помощью приведенной ниже команды. Скопируйте это значение.

```azurecli
az storage account keys list -g contosoResourceGroup5 -n contosoblobstorage5
```

![Вывод в консоль при выполнении указанной выше команды. Значение key1 выделено зеленым прямоугольником.](../media/databricks-command-output-3.png)

## <a name="create-a-key-vault-and-set-a-secret"></a>Создание Key Vault и настройка секрета

Чтобы создать Key Vault, выполните приведенную ниже команду. Эта команда также выдаст метаданные Key Vault. Скопируйте значения **ID** и **vaultUri**.

```azurecli
az keyvault create --name contosoKeyVault10 --resource-group contosoResourceGroup5 --location eastus
```

![Изображение](../media/databricks-command-output-4.png)
![Вывод в консоль при выполнении указанной выше команды. Значение ID и vaultUri выделены зеленым цветом для удобства пользователя.](../media/databricks-command-output-5.png)

Чтобы создать секрет, выполните приведенную ниже команду. Присвойте секрету значение **key1**, полученное из учетной записи хранения.

```azurecli
az keyvault secret set --vault-name contosoKeyVault10 --name storageKey --value "value of your key1"
```

## <a name="create-an-azure-databricks-workspace-and-add-key-vault-secret-scope"></a>Создание рабочей области Azure Databricks и добавление области секретов Key Vault

Действия, описанные в этом разделе, нельзя выполнить из командной строки. Воспользуйтесь [этим руководством](/azure/databricks/scenarios/store-secrets-azure-key-vault#create-an-azure-databricks-workspace-and-add-a-secret-scope). Вам потребуется применить [портал Azure](https://ms.portal.azure.com/#home) для выполнения следующих действий:

1. создание ресурса Azure Databricks;
1. запуск рабочей области;
1. создание области секретов с поддержкой Key Vault.

## <a name="access-your-blob-container-from-azure-databricks-workspace"></a>Получение доступа к контейнеру больших двоичных объектов из рабочей области Azure Databricks

Действия, описанные в этом разделе, нельзя выполнить из командной строки. Воспользуйтесь [этим руководством](/azure/databricks/scenarios/store-secrets-azure-key-vault#access-your-blob-container-from-azure-databricks). Вам придется применить рабочую область Azure Databricks, чтобы выполнить следующие действия:

1. создание **нового кластера**;
1. создание **нового Notebook**;
1. заполнение полей в скрипте Python;
1. выполнение скрипта Python.

```python
dbutils.fs.mount(
source = "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net",
mount_point = "/mnt/<mount-name>",
extra_configs = {"<conf-key>":dbutils.secrets.get(scope = "<scope-name>", key = "<key-name>")})

df = spark.read.text("/mnt/<mount-name>/<file-name>")

df.show()
```

## <a name="next-steps"></a>Дальнейшие действия

Убедитесь, что хранилище ключей можно восстановить:
> [!div class="nextstepaction"]
> [Очистка ресурсов](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-powershell)