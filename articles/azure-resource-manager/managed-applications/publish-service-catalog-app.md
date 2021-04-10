---
title: Публикация управляемого приложения в каталоге услуг
description: Показано, как создать управляемое приложение Azure, предназначенное для членов вашей организации.
author: tfitzmac
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
ms.date: 04/14/2020
ms.author: tomfitz
ms.openlocfilehash: b255cafb9040f87c902fe6c094c3e0db3d461e4b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101093649"
---
# <a name="quickstart-create-and-publish-a-managed-application-definition"></a>Краткое руководство. Создание и публикация определения управляемого приложения

Это краткое руководство содержит общие сведения о работе со [службой "Управляемые приложения Azure"](overview.md). Вы можете создать и опубликовать управляемое приложение, предназначенное для членов вашей организации.

Чтобы опубликовать управляемое приложение в каталоге услуг, сделайте следующее:

* Создайте шаблон, который определяет ресурсы, развертываемые с управляемым приложением.
* Определите элементы пользовательского интерфейса для портала при развертывании управляемого приложения.
* Создайте _ZIP-файл_ пакета, который содержит нужные файлы шаблона.
* Определите, каким пользователям, группам или приложениям нужен доступ к группе ресурсов в подписке пользователя.
* Создайте определение управляемого приложения, которое указывает на _ZIP-файл_ пакета и запрашивает доступ для получения удостоверения.

## <a name="create-the-arm-template"></a>Создание шаблона ARM

В каждом определении управляемого приложения указан файл с именем _mainTemplate.json_. В нем определяются ресурсы Azure, которые необходимо развернуть. Этот шаблон ничем не отличается от обычного шаблона ARM.

Создайте файл с именем _mainTemplate.json_. В нем учитывается регистр.

Добавьте в файл приведенный ниже код JSON. В нем определяются параметры для создания учетной записи хранения и задаются свойства для нее.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountNamePrefix": {
      "type": "string"
    },
    "storageAccountType": {
      "type": "string"
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {
    "storageAccountName": "[concat(parameters('storageAccountNamePrefix'), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      },
      "kind": "StorageV2",
      "properties": {}
    }
  ],
  "outputs": {
    "storageEndpoint": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2019-06-01').primaryEndpoints.blob]"
    }
  }
}
```

Сохраните файл _mainTemplate.json_.

## <a name="define-your-create-experience"></a>Определение процедуры создания

Как издатель, вы определяете процедуру для создания управляемого приложения на портале. Для создания интерфейса портала используется файл _createUiDefinition.json_. Вы определяете, как пользователи вводят значения для каждого параметра, используя [элементы управления](create-uidefinition-elements.md), в том числе раскрывающиеся списки, текстовые поля и поля для пароля.

Создайте файл с именем _createUiDefinition.json_ (в этом имени учитывается регистр).

Добавьте следующее содержимое JSON в файл и сохраните его.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [
      {}
    ],
    "steps": [
      {
        "name": "storageConfig",
        "label": "Storage settings",
        "subLabel": {
          "preValidation": "Configure the infrastructure settings",
          "postValidation": "Done"
        },
        "bladeTitle": "Storage settings",
        "elements": [
          {
            "name": "storageAccounts",
            "type": "Microsoft.Storage.MultiStorageAccountCombo",
            "label": {
              "prefix": "Storage account name prefix",
              "type": "Storage account type"
            },
            "defaultValue": {
              "type": "Standard_LRS"
            },
            "constraints": {
              "allowedTypes": [
                "Premium_LRS",
                "Standard_LRS",
                "Standard_GRS"
              ]
            }
          }
        ]
      }
    ],
    "outputs": {
      "storageAccountNamePrefix": "[steps('storageConfig').storageAccounts.prefix]",
      "storageAccountType": "[steps('storageConfig').storageAccounts.type]",
      "location": "[location()]"
    }
  }
}
```

Дополнительные сведения см. в статье [CreateUiDefitinion.json for Azure managed application's create experience](create-uidefinition-overview.md) (Файл CreateUiDefitinion.json для создания интерфейсов управляемых приложений Azure).

## <a name="package-the-files"></a>Упаковка файлов

Добавьте два файла в _ZIP-файл_ с именем _app.zip_. Приведенные выше два файла должны быть расположены на корневом уровне _ZIP-файла_. Если поместить их в папку, то при создании определения управляемого приложения появится сообщение об ошибке из-за отсутствия необходимых файлов.

Передайте пакет в доступное расположение, из которого его можно будет использовать. Вам потребуется указать уникальное имя учетной записи хранения.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name storageGroup -Location eastus

$storageAccount = New-AzStorageAccount `
  -ResourceGroupName storageGroup `
  -Name "mystorageaccount" `
  -Location eastus `
  -SkuName Standard_LRS `
  -Kind StorageV2

$ctx = $storageAccount.Context

New-AzStorageContainer -Name appcontainer -Context $ctx -Permission blob

Set-AzStorageBlobContent `
  -File "D:\myapplications\app.zip" `
  -Container appcontainer `
  -Blob "app.zip" `
  -Context $ctx
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create --name storageGroup --location eastus

az storage account create \
    --name mystorageaccount \
    --resource-group storageGroup \
    --location eastus \
    --sku Standard_LRS \
    --kind StorageV2

az storage container create \
    --account-name mystorageaccount \
    --name appcontainer \
    --public-access blob

az storage blob upload \
    --account-name mystorageaccount \
    --container-name appcontainer \
    --name "app.zip" \
    --file "D:\myapplications\app.zip"

```

---

## <a name="create-the-managed-application-definition"></a>Создание определения управляемого приложения

### <a name="create-an-azure-active-directory-user-group-or-application"></a>Создание группы пользователей или приложения Azure Active Directory

Следующий шаг — выбор группы пользователей, пользователя или приложения для управления ресурсами для клиента. Этому удостоверению предоставляются разрешения в управляемой группе ресурсов в соответствии с назначенной ролью. Это может быть любая встроенная роль Azure, например "Владелец" или "Участник". Чтобы создать группу пользователей Active Directory, ознакомьтесь с разделом [Создание группы и добавление в нее пользователей в Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Требуется идентификатор объекта группы пользователей для управления ресурсами.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$groupID=(Get-AzADGroup -DisplayName mygroup).Id
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
groupid=$(az ad group show --group mygroup --query objectId --output tsv)
```

---

### <a name="get-the-role-definition-id"></a>Получение идентификатора определения роли

Далее необходимо получить идентификатор определения встроенной роли Azure, к которой вы хотите предоставить доступ пользователю, группе пользователей или приложению. Обычно это роль "Владелец", "Участник" или "Читатель". Следующая команда позволяет получить идентификатор определения роли "Владелец".

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$ownerID=(Get-AzRoleDefinition -Name Owner).Id
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
ownerid=$(az role definition list --name Owner --query [].name --output tsv)
```

---

### <a name="create-the-managed-application-definition"></a>Создание определения управляемого приложения

Если у вас еще нет группы ресурсов для хранения определения управляемого приложения, создайте ее.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name appDefinitionGroup -Location westcentralus
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create --name appDefinitionGroup --location westcentralus
```

---

Теперь создайте ресурс определения управляемого приложения.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$blob = Get-AzStorageBlob -Container appcontainer -Blob app.zip -Context $ctx

New-AzManagedApplicationDefinition `
  -Name "ManagedStorage" `
  -Location "westcentralus" `
  -ResourceGroupName appDefinitionGroup `
  -LockLevel ReadOnly `
  -DisplayName "Managed Storage Account" `
  -Description "Managed Azure Storage Account" `
  -Authorization "${groupID}:$ownerID" `
  -PackageFileUri $blob.ICloudBlob.StorageUri.PrimaryUri.AbsoluteUri
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
blob=$(az storage blob url --account-name mystorageaccount --container-name appcontainer --name app.zip --output tsv)

az managedapp definition create \
  --name "ManagedStorage" \
  --location "westcentralus" \
  --resource-group appDefinitionGroup \
  --lock-level ReadOnly \
  --display-name "Managed Storage Account" \
  --description "Managed Azure Storage Account" \
  --authorizations "$groupid:$ownerid" \
  --package-file-uri "$blob"
```

---

После выполнения команды в вашей группе ресурсов будет определение управляемого приложения.

Далее приведены некоторые параметры, которые использовались в предыдущем примере:

* **Группа ресурсов**. Имя группы ресурсов, в которой создается определение управляемого приложения.
* **Уровень блокировки**. Тип блокировки группы управляемых ресурсов. Этот параметр предотвращает выполнение нежелательных операций в группе ресурсов. Сейчас поддерживается только тип блокировки ReadOnly. Если указать этот тип, пользователь будет иметь только доступ на чтение к ресурсам в управляемой группе ресурсов. Удостоверения издателей, которым предоставлен доступ к группе управляемых ресурсов, исключаются из блокировки.
* **authorizations**. Содержит идентификатор субъекта и идентификатор определения роли, используемые для предоставления разрешения группе управляемых ресурсов. Это свойство указывается в формате `<principalId>:<roleDefinitionId>`. Если требуется более одного значения, укажите их в формате `<principalId1>:<roleDefinitionId1>,<principalId2>:<roleDefinitionId2>`. Значения разделяются запятыми.
* **URI файла пакета**. Расположение _ZIP-файла_ пакета, содержащего необходимые файлы.

## <a name="bring-your-own-storage-for-the-managed-application-definition"></a>Предоставление собственного хранилища для определения управляемого приложения

Вы можете сохранить определение управляемого приложения в учетной записи хранения, которую вы указали во время создания. Это позволит полностью контролировать расположение приложения и доступ к нему в соответствии с вашими нормативными требованиями.

> [!NOTE]
> Подключение к собственному хранилищу поддерживается только с шаблоном ARM или развертываниями REST API для определения управляемого приложения.

### <a name="select-your-storage-account"></a>Выбор учетной записи хранения

Необходимо [создать учетную запись хранения](../../storage/common/storage-account-create.md), в которой будет содержаться определение управляемого приложения для использования с каталогом услуг.

Скопируйте идентификатор ресурса учетной записи хранения. Он будет использоваться позже при развертывании определения.

### <a name="set-the-role-assignment-for-appliance-resource-provider-in-your-storage-account"></a>Задание назначения роли "Поставщик ресурсов устройства" в учетной записи хранения

Перед развертыванием определения управляемого приложения в учетной записи хранения необходимо предоставить разрешения участника для роли **Поставщик ресурсов устройства**, чтобы предоставить разрешение на запись файлов определений в контейнер вашей учетной записи хранения.

1. Войдите в свою учетную запись хранения на [портале Azure](https://portal.azure.com).
1. Выберите **Управление доступом (IAM)** , чтобы отобразить параметры управления доступом для учетной записи хранения. Выберите вкладку **Назначения ролей**, чтобы просмотреть список назначений ролей.
1. В окне **Добавить назначение ролей** выберите роль **Участник**.
1. Из поля **Назначение доступа к** выберите **пользователя, группу или субъект-службу Azure AD**.
1. В разделе **Выбрать** найдите роль **Appliance Resource Provider** (Поставщик ресурсов устройств) и выберите ее.
1. Сохраните назначение роли.

### <a name="deploy-the-managed-application-definition-with-an-arm-template"></a>Развертывание определения управляемого приложения с помощью шаблона ARM

Используйте следующий шаблон ARM для развертывания упакованного управляемого приложения в качестве нового определения управляемого приложения в каталоге услуг, файлы определения которого хранятся и обслуживаются в вашей учетной записи хранения:

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "applicationName": {
      "type": "string",
      "metadata": {
        "description": "Managed Application name"
      }
    },
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ],
      "metadata": {
        "description": "Storage Account type"
      }
    },
    "definitionStorageResourceID": {
      "type": "string",
      "metadata": {
        "description": "Storage account resource ID for where you're storing your definition"
      }
    },
    "_artifactsLocation": {
      "type": "string",
      "metadata": {
        "description": "The base URI where artifacts required by this template are located."
      }
    }
  },
  "variables": {
    "lockLevel": "None",
    "description": "Sample Managed application definition",
    "displayName": "Sample Managed application definition",
    "managedApplicationDefinitionName": "[parameters('applicationName')]",
    "packageFileUri": "[parameters('_artifactsLocation')]",
    "defLocation": "[parameters('definitionStorageResourceID')]",
    "managedResourceGroupId": "[concat(subscription().id,'/resourceGroups/', concat(parameters('applicationName'),'_managed'))]",
    "applicationDefinitionResourceId": "[resourceId('Microsoft.Solutions/applicationDefinitions',variables('managedApplicationDefinitionName'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Solutions/applicationDefinitions",
      "apiVersion": "2020-08-21-preview",
      "name": "[variables('managedApplicationDefinitionName')]",
      "location": "[parameters('location')]",
      "properties": {
        "lockLevel": "[variables('lockLevel')]",
        "description": "[variables('description')]",
        "displayName": "[variables('displayName')]",
        "packageFileUri": "[variables('packageFileUri')]",
        "storageAccountId": "[variables('defLocation')]"
      }
    }
  ],
  "outputs": {}
}
```

Мы добавили новое свойство с именем `storageAccountId` в свойства `applicationDefinitions` и предоставляем идентификатор учетной записи хранения, в качестве значения которого вы хотите сохранить определение:

Вы можете убедиться, что файлы определения приложения сохранены в предоставленной учетной записи хранения в контейнере, под именем `applicationDefinitions`.

> [!NOTE]
> Для повышения безопасности можно создать определение управляемых приложений, чтобы сохранить его в [большом двоичном объекте учетной записи хранения Azure, где включено шифрование](../../storage/common/storage-service-encryption.md). Содержимое определений шифруется с помощью параметров шифрования учетной записи хранения. Определение в каталоге услуг могут видеть только пользователи с правами доступа к файлу.

## <a name="make-sure-users-can-see-your-definition"></a>Убедитесь, что пользователи могут видеть ваше определение.

У вас есть доступ к определению управляемого приложения, но вы хотите, чтобы другие пользователи в вашей организации могли получить к нему доступ. Предоставьте им роль читателя в определении. Они могут унаследовать этот уровень доступа из группы подписки или ресурса. Сведения о том, как проверить, кто имеет доступ к определению, и добавить пользователей или группы, см. в статье [Назначение ролей Azure с помощью портала Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Дальнейшие действия

Вы опубликовали определение управляемого приложения. Теперь можно приступать к развертыванию экземпляра этого определения.

> [!div class="nextstepaction"]
> [Краткое руководство. Развертывание приложения из каталога служб](deploy-service-catalog-quickstart.md)
