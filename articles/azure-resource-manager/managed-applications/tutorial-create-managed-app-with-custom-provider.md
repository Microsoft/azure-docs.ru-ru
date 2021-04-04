---
title: Руководство. Использование настраиваемых действий и ресурсов
description: В этом учебнике описано создание управляемого приложения Azure с настраиваемым поставщиком Azure.
ms.topic: tutorial
ms.author: lazinnat
author: lazinnat
ms.date: 06/20/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 50a65583c41dd94b174a33432afcf42b31e67df0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96437054"
---
# <a name="tutorial-create-managed-application-with-custom-actions-and-resources"></a>Руководство по созданию управляемого приложения с настраиваемыми действиями и ресурсами

В рамках этого учебника вы создадите собственное управляемое приложение с настраиваемыми действиями и ресурсами. Управляемое приложение будет содержать настраиваемое действие на странице `Overview`, тип настраиваемого ресурса будет отображаться как отдельный пункт меню в таблице `Table of Content`, а контекст настраиваемого действия — на странице настраиваемых ресурсов.

В этом учебнике описаны следующие шаги:

> [!div class="checklist"]
> * Создание файла определения пользовательского интерфейса для создания экземпляра управляемого приложения.
> * Создание шаблона развертывания с [настраиваемым поставщиком Azure](../custom-providers/overview.md), учетной записью хранения Azure и функцией Azure
> * Создание артефакта определения представления с настраиваемыми действиями и ресурсами.
> * Развертывание определения управляемого приложения.
> * Развертывание экземпляра управляемого приложения.
> * Выполнение настраиваемых действий и создание настраиваемых ресурсов.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником вам необходимо ознакомиться со следующими статьями:

* [Создание и публикация определения управляемого приложения](publish-service-catalog-app.md).
* [Deploy service catalog app through Azure portal](deploy-service-catalog-quickstart.md) (Развертывание приложения из каталога служб с помощью портала Azure).
* [Create Azure portal user interface for your managed application](create-uidefinition-overview.md) (Создание пользовательского интерфейса портала Azure для управляемого приложения).
* [View definition artifact in Azure Managed Applications](concepts-view-definition.md) (Артефакт определения представления в управляемых приложениях).
* [Azure Custom Resource Providers Overview](../custom-providers/overview.md) (Общие сведения о поставщиках настраиваемых ресурсов Azure).

## <a name="user-interface-definition"></a>Определение пользовательского интерфейса

В рамках этого учебника вы создадите управляемое приложение и группу управляемых ресурсов, содержащую экземпляр настраиваемого поставщика, учетную запись хранения и функцию. Используемая в этом примере функция Azure реализует API, который обрабатывает операции настраиваемого поставщика для действий и ресурсов. Учетная запись хранения Azure используется в качестве основного хранилища для ресурсов настраиваемого поставщика.

Определение пользовательского интерфейса для создания экземпляра управляемого приложения включает следующие входные элементы: `funcname` и `storagename`. Имена учетной записи хранения и функции должны быть глобально уникальными. По умолчанию файлы функции будут развернуты из [примера пакета функций](https://github.com/Azure/azure-quickstart-templates/tree/master/101-custom-rp-with-function/artifacts/functionzip), но вы можете изменить его, добавив входной элемент для ссылки на пакет в файл *createUiDefinition.json*:

```json
{
  "name": "funcname",
  "type": "Microsoft.Common.TextBox",
  "label": "Name of the function to be created",
  "toolTip": "Name of the function to be created",
  "visible": true,
  "constraints": {
    "required": true
  }
},
{
  "name": "storagename",
  "type": "Microsoft.Common.TextBox",
  "label": "Name of the storage to be created",
  "toolTip": "Name of the storage to be created",
  "visible": true,
  "constraints": {
    "required": true
  }
},
{
  "name": "zipFileBlobUri",
  "type": "Microsoft.Common.TextBox",
  "defaultValue": "https://github.com/Azure/azure-quickstart-templates/tree/master/101-custom-rp-with-function/artifacts/functionzip/functionpackage.zip",
  "label": "The Uri to the uploaded function zip file",
  "toolTip": "The Uri to the uploaded function zip file",
  "visible": true
}
```

и выходной элемент в файл *createUiDefinition.json*:

```json
  "funcname": "[steps('applicationSettings').funcname]",
  "storageName": "[steps('applicationSettings').storagename]",
  "zipFileBlobUri": "[steps('applicationSettings').zipFileBlobUri]"
```

Полный пример *createUiDefinition.json* можно найти в статье [Reference: User interface elements artifact](reference-createuidefinition-artifact.md) (Справочные материалы. Артефакт элементов пользовательского интерфейса).

## <a name="template-with-custom-provider"></a>Шаблон с настраиваемым поставщиком

Для создания экземпляра управляемого приложения с настраиваемым поставщиком вам потребуется определить в файле **mainTemplate.json** ресурс настраиваемого поставщика с именем **public** и типом **Microsoft.CustomProviders/resourceProviders**. В этом ресурсе определяются типы ресурсов и действия для службы. Чтобы развернуть экземпляры функции Azure и учетной записи хранения Azure, определите ресурсы типа `Microsoft.Web/sites` и `Microsoft.Storage/storageAccounts` соответственно.

В рамках этого учебника вы создадите один тип ресурса `users`, а также настраиваемые действия `ping` и `users/contextAction`, которые будут выполняться в контексте настраиваемого ресурса `users`. Для каждого типа ресурса и действия предоставьте конечную точку, указывающую на функцию с именем, предоставленным в файле [createUiDefinition.json](#user-interface-definition). Укажите для **routingType** значение `Proxy,Cache` для типов ресурсов и `Proxy` — для действий:

```json
{
  "apiVersion": "[variables('customrpApiversion')]",
  "type": "Microsoft.CustomProviders/resourceProviders",
  "name": "[variables('customProviderName')]",
  "location": "[parameters('location')]",
  "properties": {
    "actions": [
      {
        "name": "ping",
        "routingType": "Proxy",
        "endpoint": "[listSecrets(resourceId('Microsoft.Web/sites/functions', parameters('funcname'), 'HttpTrigger1'), '2018-02-01').trigger_url]"
      },
      {
        "name": "users/contextAction",
        "routingType": "Proxy",
        "endpoint": "[listSecrets(resourceId('Microsoft.Web/sites/functions', parameters('funcname'), 'HttpTrigger1'), '2018-02-01').trigger_url]"
      }
    ],
    "resourceTypes": [
      {
        "name": "users",
        "routingType": "Proxy,Cache",
        "endpoint": "[listSecrets(resourceId('Microsoft.Web/sites/functions', parameters('funcname'), 'HttpTrigger1'), '2018-02-01').trigger_url]"
      }
    ]
  },
  "dependsOn": [
    "[concat('Microsoft.Web/sites/',parameters('funcname'))]"
  ]
}
```

Полный пример файла *mainTemplate.json* можно найти на странице [Reference: Deployment template artifact](reference-main-template-artifact.md) (Справочные материалы. Артефакт шаблона развертывания).

## <a name="view-definition-artifact"></a>Просмотр артефакта определения

Чтобы определить пользовательский интерфейс, который содержит настраиваемые действия и ресурсы в управляемом приложении, необходимо создать артефакт **viewDefinition.json**. Дополнительные сведения об артефакте определения представления см. в [этой статье](concepts-view-definition.md).

В рамках этого учебника вы определите:
* Страницу *Обзор* с кнопкой панели инструментов, которая представляет настраиваемое действие `TestAction` с основным текстовым вводом.
* Страницу *Пользователи*, представляющую тип настраиваемого ресурса `users`.
* Действие настраиваемого ресурса `users/contextAction` на странице *Пользователи*, которое будет выполняться в контексте настраиваемого ресурса типа `users`.

В следующем примере показана конфигурация представления для страницы обзора:

```json
{
    "kind": "Overview",
    "properties": {
      "header": "Welcome to your Demo Azure Managed Application",
      "description": "This Managed application with Custom Provider is for demo purposes only.",
      "commands": [{
          "displayName": "Ping Action",
          "path": "/customping",
          "icon": "LaunchCurrent"
      }]
    }
  }
```

Приведенный ниже пример включает конфигурацию страницы ресурсов "Пользователи" с действием настраиваемого ресурса:

```json
{
    "kind": "CustomResources",
    "properties": {
      "displayName": "Users",
      "version": "1.0.0.0",
      "resourceType": "users",
      "createUIDefinition": {
      },
      "commands": [{
        "displayName": "Custom Context Action",
        "path": "users/contextAction",
        "icon": "Start"
      }],
      "columns": [
        { "key": "properties.FullName", "displayName": "Full Name" },
        { "key": "properties.Location", "displayName": "Location", "optional": true }
      ]
    }
  }
```

Полный пример файла *viewDefinition.json* можно найти в статье [Reference: View definition artifact](reference-view-definition-artifact.md) (Справочные материалы. Артефакт определения представления).

## <a name="managed-application-definition"></a>Определение управляемого приложения

Упакуйте следующие артефакты управляемого приложения в ZIP-архив и отправьте его в хранилище:

* createUiDefinition.json;
* mainTemplate.json;
* viewDefinition.json.

Все файлы должны находиться на корневом уровне. Пакет с артефактами можно хранить в любом хранилище, например в хранилище BLOB-объектов на сайте GitHub или учетной записи хранилища BLOB-объектов Azure. Ниже приведен сценарий отправки пакета приложения в учетную запись хранения: 

```powershell
$resourceGroup="appResourcesGroup"
$storageName="mystorageaccount$RANDOM"

# Sign in to your Azure subscription
Connect-AzAccount
# Create resource group for managed application definition and application package
New-AzResourceGroup -Name $resourceGroup -Location eastus

# Create storage account for a package with application artifacts
$storageAccount=New-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -SkuName Standard_LRS `
  -Location eastus `
$ctx=$storageAccount.Context

# Create storage container and upload zip to blob
New-AzStorageContainer -Name appcontainer -Context $ctx -Permission blob
Set-AzStorageBlobContent `
  -File "path_to_your_zip_package" `
  -Container appcontainer `
  -Blob app.zip `
  -Context $ctx 

# Get blob absolute uri
$blobUri=(Get-AzureStorageBlob -Container appcontainer -Blob app.zip -Context $ctx).ICloudBlob.uri.AbsoluteUri
```

Запустите приведенный ниже сценарий в Azure CLI или выполните определенные шаги на портале Azure, чтобы развернуть определение управляемого приложения из каталога услуг:

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli-interactive)

```azurecli-interactive
resourceGroup="appResourcesGroup"
# Select subscription and create resource group (if you have not created yet)
az account set --subscription <subscriptionID>
az group create --name $resourceGroup --location eastus

# Get object ID of your identity
userid=$(az ad user show --upn-or-object-id example@contoso.org --query objectId --output tsv)
# Get role definition ID for the Owner role
roleid=$(az role definition list --name Owner --query [].name --output tsv)

# Create managed application definition resource
az managedapp definition create \
  --name "ManagedUsersAppDefinition" \
  --location "eastus" \
  --resource-group $resourceGroup \
  --lock-level ReadOnly \
  --display-name "Managed users app definition" \
  --description "Managed application with Azure Custom Provider" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "path to your app.zip package"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. На портале Azure щелкните **Все службы**. В списке ресурсов введите и выберите **Центр управляемых приложений**.
2. В **центре управляемых приложений** выберите **Определение приложения из каталога услуг** и щелкните **Добавить**. 
    
    ![Добавление каталога услуг](./media/tutorial-create-managed-app-with-custom-provider/service-catalog-managed-application.png)

3. Укажите значения для создания определения каталога услуг:

    * Укажите уникальное **имя** для определения каталога услуг, **отображаемое имя** и *описание* (необязательно).
    * Выберите **подписку**, **группу ресурсов** и **расположение** для создания определения приложения. Вы можете использовать ту же группу ресурсов, что и для ZIP-архива или же создать новую.
    * В качестве **URI файла пакета** укажите путь к созданному ранее ZIP-файлу.

    ![Определение значений](./media/tutorial-create-managed-app-with-custom-provider/add-service-catalog-managed-application.png)

4. Когда откроется раздел "Аутентификация и уровень блокировки", выберите **Добавить авторизацию**.

    ![Добавление авторизации](./media/tutorial-create-managed-app-with-custom-provider/add-authorization.png)

5. Выберите группу пользователей Azure Active Directory для управления ресурсами и нажмите **ОК**.

   ![Добавление группы авторизации](./media/tutorial-create-managed-app-with-custom-provider/add-auth-group.png)

6. Указав все значения, выберите **Создать**.

   ![Создание определения управляемого приложения](./media/tutorial-create-managed-app-with-custom-provider/create-service-catalog-definition.png)

---

## <a name="managed-application-instance"></a>Экземпляр управляемого приложения

При развертывании определения управляемого приложения запустите приведенный ниже сценарий или выполните шаги на портале Azure, чтобы развернуть экземпляр управляемого приложения с настраиваемым поставщиком:

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli-interactive)

```azurecli-interactive
appResourcesGroup="appResourcesGroup"
applicationGroup="usersApplicationGroup"

# Create resource group for managed application instance
az group create --name $applicationGroup --location eastus

# Get ID of managed application definition
appid=$(az managedapp definition show --name ManagedUsersAppDefinition --resource-group $appResourcesGroup --query id --output tsv)

# Create the managed application
az managedapp create \
  --name ManagedUsersApp \
  --location "eastus" \
  --kind "Servicecatalog" \
  --resource-group $applicationGroup \
  --managedapp-definition-id $appid \
  --managed-rg-id "managedResourcesGroup" \
  --parameters "{\"funcname\": {\"value\": \"managedusersappfunction\"}, \"storageName\": {\"value\": \"managedusersappstorage\"}}"
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. На портале Azure щелкните **Все службы**. В списке ресурсов введите и выберите **Центр управляемых приложений**.
2. В **Центре управляемых приложений** выберите **Приложения каталога услуг** и щелкните **Добавить**. 

    ![Добавление управляемого приложения](./media/tutorial-create-managed-app-with-custom-provider/add-managed-application.png)

3. На странице **Приложения каталога услуг** введите отображаемое имя определения каталога услуг в поле поиска. Выберите созданное на предыдущем шаге определение и щелкните **Создать**.

    ![Выбор каталога услуг](./media/tutorial-create-managed-app-with-custom-provider/select-service-catalog-definition.png)

4. Укажите значения для создания экземпляра управляемого приложения из определения каталога услуг:

    * Выберите **подписку**, **группу ресурсов** и **расположение** для создания экземпляра приложения.
    * Укажите уникальные имена для функции Azure и учетной записи хранения Azure.

    ![Параметры приложения](./media/tutorial-create-managed-app-with-custom-provider/application-settings.png)

5. После выполнения проверки щелкните **OK**, чтобы развернуть экземпляр управляемого приложения. 
    
    ![Развертывание управляемого приложения](./media/tutorial-create-managed-app-with-custom-provider/deploy-managed-application.png)

---

## <a name="custom-actions-and-resources"></a>Настраиваемые действия и ресурсы

После развертывания экземпляра приложения из каталога услуг будут созданы две группы ресурсов. Первая группа ресурсов `applicationGroup` содержит экземпляр управляемого приложения, вторая группа ресурсов `managedResourceGroup` содержит ресурсы для управляемого приложения, включая **настраиваемого поставщика**.

![Группы ресурсов приложения](./media/tutorial-create-managed-app-with-custom-provider/application-resource-groups.png)

Вы можете перейти к экземпляру управляемого приложения и выполнить **настраиваемое действие** на странице обзора, создать настраиваемый ресурс **users** на странице "Пользователи" и запустить **настраиваемое действие контекста** в настраиваемом ресурсе.

* Перейдите на страницу обзора и нажмите кнопку Ping Action (Действие Ping):

![Выполнение настраиваемого действия](./media/tutorial-create-managed-app-with-custom-provider/perform-custom-action.png)

* Перейдите на страницу "Пользователи" и нажмите кнопку "Добавить". Укажите входные данные для создания ресурса и отправьте форму:

![Снимок экрана: кнопка "Добавить", выбранная на странице "Пользователи".](./media/tutorial-create-managed-app-with-custom-provider/create-custom-resource.png)

* Перейдите на страницу "Пользователи", выберите ресурс users и щелкните Custom Context Action (Настраиваемое действие контекста):

![Снимок экрана: выбранное настраиваемое действие контекста.](./media/tutorial-create-managed-app-with-custom-provider/perform-custom-resource-action.png)

[!INCLUDE [clean-up-section-portal](../../../includes/clean-up-section-portal.md)]

## <a name="looking-for-help"></a>Требуется помощь?

Если у вас есть вопросы об Управляемых приложениях Azure, вы можете задать их на [этой странице](https://stackoverflow.com/questions/tagged/azure-managedapps). Подобный вопрос, возможно, уже был задан, поэтому перед его публикацией сначала проверьте наличие ответа. Чтобы быстрее получить ответ, добавьте к вопросу тег `azure-managedapps`!

## <a name="next-steps"></a>Дальнейшие действия

Сведения о том, как опубликовать управляемое приложение в Azure Marketplace, см. в статье [Управляемые приложения Azure в Marketplace](../../marketplace/create-new-azure-apps-offer.md).

Дополнительные сведения о [Настраиваемых поставщиках Azure](../custom-providers/overview.md).
