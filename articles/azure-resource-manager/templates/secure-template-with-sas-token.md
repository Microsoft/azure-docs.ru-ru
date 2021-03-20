---
title: Безопасное развертывание шаблона с маркером SAS
description: Развертывание ресурсов в Azure с помощью шаблона Azure Resource Manager, защищенного маркером SAS. Показывает Azure PowerShell и Azure CLI.
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 8b35e82da8ebca98ec9fe1fb7441612bf61fb142
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88855658"
---
# <a name="deploy-private-arm-template-with-sas-token"></a>Развертывание закрытого шаблона ARM с маркером SAS

Если шаблон Azure Resource Manager (шаблон ARM) находится в учетной записи хранения, можно ограничить доступ к шаблону, чтобы не предоставлять его открытым. Доступ к защищенному шаблону осуществляется путем создания маркера подписанного URL-адрес (SAS) для шаблона и предоставления маркера во время развертывания. В этой статье объясняется, как использовать Azure PowerShell или Azure CLI для развертывания шаблона с маркером SAS.

> [!IMPORTANT]
> Вместо защиты шаблона с помощью маркера SAS рассмотрите возможность использования [спецификаций шаблонов](template-specs.md). С помощью спецификаций шаблонов можно предоставить общий доступ к шаблонам другим пользователям в Организации и управлять доступом к шаблонам с помощью Azure RBAC.

## <a name="create-storage-account-with-secured-container"></a>Создание учетной записи хранения с защищенным контейнером

Следующий скрипт создает учетную запись хранения и контейнер с отключенным общим доступом.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup `
  -Name ExampleGroup `
  -Location "Central US"
New-AzStorageAccount `
  -ResourceGroupName ExampleGroup `
  -Name {your-unique-name} `
  -Type Standard_LRS `
  -Location "Central US"
Set-AzCurrentStorageAccount `
  -ResourceGroupName ExampleGroup `
  -Name {your-unique-name}
New-AzStorageContainer `
  -Name templates `
  -Permission Off
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create \
  --name "ExampleGroup" \
  --location "Central US"
az storage account create \
    --resource-group ExampleGroup \
    --location "Central US" \
    --sku Standard_LRS \
    --kind Storage \
    --name {your-unique-name}
connection=$(az storage account show-connection-string \
    --resource-group ExampleGroup \
    --name {your-unique-name} \
    --query connectionString)
az storage container create \
    --name templates \
    --public-access Off \
    --connection-string $connection
```

---

## <a name="upload-template-to-storage-account"></a>Отправить шаблон в учетную запись хранения

Теперь вы готовы отправить шаблон в учетную запись хранения. Укажите путь к шаблону, который вы хотите использовать.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Set-AzStorageBlobContent `
  -Container templates `
  -File c:\Templates\azuredeploy.json
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az storage blob upload \
    --container-name templates \
    --file azuredeploy.json \
    --name azuredeploy.json \
    --connection-string $connection
```

---

## <a name="provide-sas-token-during-deployment"></a>Предоставление маркера SAS во время развертывания

Чтобы развернуть частный шаблон в учетной записи хранения, создайте маркер SAS и добавьте его в универсальный код ресурса (URI) для шаблона. Задайте срок действия, достаточный для выполнения развертывания.

> [!IMPORTANT]
> Большой двоичный объект, содержащий шаблон, будет доступен только владельцу учетной записи. Тем не менее, если создать маркер SAS для этого большого двоичного объекта, то он будет доступен любому пользователю, обладающему этим универсальным кодом ресурса (URI). Если другой пользователь перехватит этот универсальный код ресурса (URI), то сможет получить доступ к шаблону. Маркер SAS — хороший способ ограничить доступ к своим шаблонам, но не следует указывать конфиденциальные данные, например пароли, непосредственно в шаблоне.
>

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
# get the URI with the SAS token
$templateuri = New-AzStorageBlobSASToken `
  -Container templates `
  -Blob azuredeploy.json `
  -Permission r `
  -ExpiryTime (Get-Date).AddHours(2.0) -FullUri

# provide URI with SAS token during deployment
New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri $templateuri
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Следующий пример работает с окружением Bash в Cloud Shell. В других средах может потребоваться другой синтаксис для создания срока действия маркера SAS.

```azurecli-interactive
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group ExampleGroup \
    --name {your-unique-name} \
    --query connectionString)
token=$(az storage blob generate-sas \
    --container-name templates \
    --name azuredeploy.json \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
    --container-name templates \
    --name azuredeploy.json \
    --output tsv \
    --connection-string $connection)
az deployment group create \
  --resource-group ExampleGroup \
  --template-uri $url?$token
```

---

Пример использования маркера SAS со связанными шаблонами см. в статье [Использование связанных шаблонов в Azure Resource Manager](linked-templates.md).


## <a name="next-steps"></a>Дальнейшие действия
* Общие сведения о развертывании шаблонов см. в статье [развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](deploy-powershell.md).
* Сведения об определении параметров в шаблоне см. в разделе [Создание шаблонов](template-syntax.md#parameters).
