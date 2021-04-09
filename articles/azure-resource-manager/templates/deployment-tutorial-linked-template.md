---
title: Учебник. Развертывание связанного шаблона
description: Узнайте как выполнять развертывание связанного шаблона
ms.date: 02/12/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 8f2bbd327adca6eef62d5e79f422f61d460ea7a5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100589270"
---
# <a name="tutorial-deploy-a-linked-template"></a>Руководство по развертыванию связанного шаблона

В [предыдущих учебниках](./deployment-tutorial-local-template.md) вы узнали, как развернуть шаблон, хранящийся на локальном компьютере. Для развертывания сложных решений вы можете разбить один шаблон на несколько и развернуть их с помощью основного шаблона. В этом учебнике вы узнаете, как развернуть основной шаблон, содержащий ссылку на связанный шаблон. При развертывании главного шаблона запускается развертывание связанного шаблона. Вы также узнаете, как сохранить и защитить шаблоны с помощью маркера SAS. Это занимает около **12 минут**.

## <a name="prerequisites"></a>Предварительные требования

Советуем выполнить инструкции из предыдущего учебника, но это необязательно.

## <a name="review-template"></a>Проверка шаблона

В предыдущих учебниках вы развернули шаблон, который создает учетную запись хранения, план Службы приложений и веб-приложение. Использовался шаблон:

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/local-template/azuredeploy.json":::

## <a name="create-a-linked-template"></a>Создание связанного шаблона

Ресурс учетной записи хранения можно разделить на связанный шаблон:

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/linked-template/linkedStorageAccount.json":::

Следующий шаблон является основным. Выделенный объект `Microsoft.Resources/deployments` показывает, как вызвать связанный шаблон. Связанный шаблон нельзя сохранять в виде локального файла или файла, который доступен только в вашей локальной сети. Вы можете указать значение универсального кода ресурса (URI) связанного шаблона, который содержит протокол HTTP или HTTPS, или использовать свойство _relativePath_ для развертывания удаленного связанного шаблона в расположении относительно родительского шаблона. Один из вариантов — разместить основной и связанный шаблон в учетной записи хранения.

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/linked-template/azuredeploy.json" highlight="34-52":::

## <a name="store-the-linked-template"></a>Сохранение связанного шаблона

Основной и связанный шаблон хранятся в репозитории GitHub:

Указанный ниже сценарий PowerShell создает учетную запись хранения, контейнер и копирует два шаблона из репозитория GitHub в контейнер. Вот эти два шаблона:

- Основной шаблон: https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/azuredeploy.json.
- Связанный шаблон: https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/linkedStorageAccount.json.

Выберите **Попробовать** чтобы открыть Cloud Shell, выберите **Копировать**, чтобы скопировать скрипт PowerShell, и щелкните правой кнопкой мыши панель оболочки, чтобы вставить скрипт:

> [!IMPORTANT]
> Имя учетной записи хранения должно содержать от 3 до 24 символов и состоять только из цифр и букв нижнего регистра. Имя должно быть уникальным. В шаблоне имя учетной записи хранения — это имя проекта с добавлением **store**, которое должно содержать от 3 до 11 символов. Поэтому имя проекта должно соответствовать требованиям к имени учетной записи хранения и быть длиной менее 11 символов.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name:"   # This name is used to generate names for Azure resources, such as storage account name.
$location = Read-Host -Prompt "Enter a location (i.e. centralus)"

$resourceGroupName = $projectName + "rg"
$storageAccountName = $projectName + "store"
$containerName = "templates" # The name of the Blob container to be created.

$mainTemplateURL = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/azuredeploy.json"
$linkedTemplateURL = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/linkedStorageAccount.json"

$mainFileName = "azuredeploy.json" # A file name used for downloading and uploading the main template.Add-PSSnapin
$linkedFileName = "linkedStorageAccount.json" # A file name used for downloading and uploading the linked template.

# Download the templates
Invoke-WebRequest -Uri $mainTemplateURL -OutFile "$home/$mainFileName"
Invoke-WebRequest -Uri $linkedTemplateURL -OutFile "$home/$linkedFileName"

# Create a resource group
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create a storage account
$storageAccount = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -Location $location `
    -SkuName "Standard_LRS"

$context = $storageAccount.Context

# Create a container
New-AzStorageContainer -Name $containerName -Context $context -Permission Container

# Upload the templates
Set-AzStorageBlobContent `
    -Container $containerName `
    -File "$home/$mainFileName" `
    -Blob $mainFileName `
    -Context $context

Set-AzStorageBlobContent `
    -Container $containerName `
    -File "$home/$linkedFileName" `
    -Blob $linkedFileName `
    -Context $context

Write-Host "Press [ENTER] to continue ..."
```

## <a name="deploy-template"></a>Развертывание шаблона

Чтобы развернуть шаблоны в учетной записи хранения, создайте маркер SAS и задайте его в параметре _-QueryString_. Задайте срок действия, достаточный для выполнения развертывания. Большие двоичные объекты, содержащие шаблоны, будут доступны только владельцу учетной записи. Тем не менее, если создать маркер SAS для этого большого двоичного объекта, то он будет доступен любому пользователю, обладающему этим маркером SAS. Если другой пользователь перехватит универсальный код ресурса (URI) и маркер SAS, то сможет получить доступ к шаблону. Маркер SAS — хороший способ ограничить доступ к своим шаблонам, но не следует указывать конфиденциальные данные, например пароли, непосредственно в шаблоне.

Если вы еще не создали группу ресурсов, см. [этот раздел](./deployment-tutorial-local-template.md#create-resource-group).

> [!NOTE]
> В приведенном ниже коде Azure CLI параметр `date` (`-d`) будет недопустимым аргументом в macOS. Поэтому чтобы добавить два часа к текущему времени в терминале macOS, пользователям macOS следует использовать `-v+2H`.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive

$projectName = Read-Host -Prompt "Enter the same project name:"   # This name is used to generate names for Azure resources, such as storage account name.

$resourceGroupName="${projectName}rg"
$storageAccountName="${projectName}store"
$containerName = "templates"

$key = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
$context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $key

$mainTemplateUri = $context.BlobEndPoint + "$containerName/azuredeploy.json"
$sasToken = New-AzStorageContainerSASToken `
    -Context $context `
    -Container $containerName `
    -Permission r `
    -ExpiryTime (Get-Date).AddHours(2.0)
$newSas = $sasToken.substring(1)


New-AzResourceGroupDeployment `
  -Name DeployLinkedTemplate `
  -ResourceGroupName $resourceGroupName `
  -TemplateUri $mainTemplateUri `
  -QueryString $newSas `
  -projectName $projectName `
  -verbose

Write-Host "Press [ENTER] to continue ..."
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
echo "Enter a project name that is used to generate resource names:" &&
read projectName &&

resourceGroupName="${projectName}rg" &&
storageAccountName="${projectName}store" &&
containerName="templates" &&

key=$(az storage account keys list -g $resourceGroupName -n $storageAccountName --query [0].value -o tsv) &&

sasToken=$(az storage container generate-sas \
  --account-name $storageAccountName \
  --account-key $key \
  --name $containerName \
  --permissions r \
  --expiry `date -u -d "120 minutes" '+%Y-%m-%dT%H:%MZ'`) &&
sasToken=$(echo $sasToken | sed 's/"//g')&&

blobUri=$(az storage account show -n $storageAccountName -g $resourceGroupName -o tsv --query primaryEndpoints.blob) &&
templateUri="${blobUri}${containerName}/azuredeploy.json" &&

az deployment group create \
  --name DeployLinkedTemplate \
  --resource-group $resourceGroupName \
  --template-uri $templateUri \
  --parameters projectName=$projectName \
  --query-string $sasToken \
  --verbose
```

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Очистите развернутые ресурсы, удалив группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

Вы узнали как выполнять развертывание связанного шаблона. Из следующего руководства вы узнаете, как создать конвейер DevOps для развертывания шаблона.

> [!div class="nextstepaction"]
> [Создать конвейер](./deployment-tutorial-pipeline.md)