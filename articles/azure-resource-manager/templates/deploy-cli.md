---
title: Развертывание ресурсов с помощью Azure CLI и шаблона
description: Используйте Azure Resource Manager и Azure CLI для развертывания ресурсов в Azure. Ресурсы определяются в шаблоне диспетчер ресурсов или в файле Бицеп.
ms.topic: conceptual
ms.date: 03/04/2021
ms.openlocfilehash: 1fc3397409634f29690af05ca2a85e67c30b6d89
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104954978"
---
# <a name="deploy-resources-with-arm-templates-and-azure-cli"></a>Развертывание ресурсов с помощью шаблонов ARM и Azure CLI

В этой статье объясняется, как использовать Azure CLI с шаблонами Azure Resource Manager (шаблоны ARM) или файлами Бицеп для развертывания ресурсов в Azure. Если вы не знакомы с концепциями развертывания и управления решениями Azure, см. статью [Общие сведения о развертывании шаблонов](overview.md) или [бицеп](bicep-overview.md).

Команды развертывания изменились в Azure CLI версии 2.2.0. Для использования примеров в этой статье требуется Azure CLI версии 2.2.0 или более поздней. Для развертывания файлов Бицеп требуется [Azure CLI версии 2.20.0 или более поздней](/cli/azure/install-azure-cli).

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

Если Azure CLI не установлен, можно использовать Azure Cloud Shell. Дополнительные сведения см. [в статье Развертывание шаблонов ARM из Azure Cloud Shell](deploy-cloud-shell.md).

## <a name="deployment-scope"></a>Область развертывания

Вы можете выбрать для развертывания группу ресурсов, подписку, группу управления или клиент. В зависимости от области развертывания используются разные команды.

* Для развертывания в **группе ресурсов** используйте команду [AZ Deployment Group Create](/cli/azure/deployment/group#az-deployment-group-create):

  ```azurecli-interactive
  az deployment group create --resource-group <resource-group-name> --template-file <path-to-template-or-bicep>
  ```

* Для развертывания в **подписке** используйте команду [AZ Deployment подсоздание](/cli/azure/deployment/sub#az-deployment-sub-create):

  ```azurecli-interactive
  az deployment sub create --location <location> --template-file <path-to-template-or-bicep>
  ```

  Дополнительные сведения о развертываниях на уровне подписки см. в статье [Создание групп ресурсов и ресурсов на уровне подписки](deploy-to-subscription.md).

* Для развертывания в **группе управления** используйте команду [AZ Deployment mg Create](/cli/azure/deployment/mg#az-deployment-mg-create):

  ```azurecli-interactive
  az deployment mg create --location <location> --template-file <path-to-template-or-bicep>
  ```

  Дополнительные сведения о развертываниях на уровне групп управления см. в статье [Создание ресурсов на уровне группы управления](deploy-to-management-group.md).

* Чтобы выполнить развертывание в **клиенте**, используйте [AZ Deployment клиент Create](/cli/azure/deployment/tenant#az-deployment-tenant-create):

  ```azurecli-interactive
  az deployment tenant create --location <location> --template-file <path-to-template-or-bicep>
  ```

  Дополнительные сведения о развертываниях на уровне клиента см. в статье [Создание ресурсов на уровне клиента](deploy-to-tenant.md).

Для каждой области пользователь, развертывающий шаблон или файл Бицеп, должен иметь необходимые разрешения для создания ресурсов.

## <a name="deploy-local-template-or-bicep-file"></a>Развертывание локального шаблона или файла Бицеп

Вы можете развернуть шаблон с локального компьютера или из него, который хранится извне. В этом разделе описывается развертывание локального шаблона.

Если развертывание выполняется в несуществующей группе ресурсов, создайте группу ресурсов. Имя группы ресурсов может содержать только буквы, цифры, точки, знаки подчеркивания, дефисы и скобки. Оно может содержать до 90 знаков и Имя не может заканчиваться точкой.

```azurecli-interactive
az group create --name ExampleGroup --location "Central US"
```

Чтобы развернуть локальный шаблон или файл Бицеп, используйте `--template-file` параметр в команде Deployment. В следующем примере также показано, как задать значение параметра.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file <path-to-template-or-bicep> \
  --parameters storageAccountType=Standard_GRS
```

Развертывание может занять несколько минут. По завершении появится сообщение, содержащее результат:

```output
"provisioningState": "Succeeded",
```

## <a name="deploy-remote-template"></a>Развертывание удаленного шаблона

> [!NOTE]
> В настоящее время Azure CLI не поддерживает развертывание удаленных файлов Бицеп. Используйте [БИЦЕП CLI](./bicep-install.md#development-environment) для компиляции файла бицеп в шаблон JSON, а затем загрузите JSON-файл в удаленное расположение.

Вместо того чтобы хранить шаблоны ARM на локальном компьютере, вы можете хранить их во внешнем расположении. Вы можете хранить шаблоны в репозитории системы управления версиями (например, GitHub). А также их можно хранить в учетной записи хранения Azure для общего доступа в организации.

Если развертывание выполняется в несуществующей группе ресурсов, создайте группу ресурсов. Имя группы ресурсов может содержать только буквы, цифры, точки, знаки подчеркивания, дефисы и скобки. Оно может содержать до 90 знаков и Имя не может заканчиваться точкой.

```azurecli-interactive
az group create --name ExampleGroup --location "Central US"
```

Для развертывания внешнего шаблона используйте параметр `template-uri`.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json" \
  --parameters storageAccountType=Standard_GRS
```

В предыдущем примере для шаблона требуется общедоступный код URI, который подходит для большинства сценариев, так как шаблон не должен содержать конфиденциальные данные. Если необходимо указать конфиденциальные данные (например, пароль администратора), то передайте это значение с помощью безопасного параметра. Однако если вы хотите управлять доступом к шаблону, рассмотрите возможность использования [спецификаций шаблонов](#deploy-template-spec).

Чтобы развернуть удаленные связанные шаблоны с относительным путем, хранящимся в учетной записи хранения, используйте `query-string` для указания маркера SAS:

```azurecli-interactive
az deployment group create \
  --name linkedTemplateWithRelativePath \
  --resource-group myResourceGroup \
  --template-uri "https://stage20210126.blob.core.windows.net/template-staging/mainTemplate.json" \
  --query-string $sasToken
```

Дополнительные сведения см. в разделе [использование относительного пути для связанных шаблонов](./linked-templates.md#linked-template).

## <a name="deployment-name"></a>Deployment name (Имя развертывания)

При развертывании шаблона ARM можно присвоить имя развертыванию. Это имя может помочь получить развертывание из журнала развертывания. Если имя для развертывания не указано, используется имя файла шаблона. Например, если развернуть шаблон с именем `azuredeploy.json` и не указывать имя развертывания, то развертывание будет называться `azuredeploy` .

Каждый раз при выполнении развертывания в журнал развертывания группы ресурсов добавляется запись с именем развертывания. Если запустить другое развертывание и присвоить ему такое же имя, то Предыдущая запись будет заменена текущим развертыванием. Если требуется хранить уникальные записи в журнале развертывания, присвойте каждому развертыванию уникальное имя.

Чтобы создать уникальное имя, можно назначить случайное число.

```azurecli-interactive
deploymentName='ExampleDeployment'$RANDOM
```

Или добавьте значение даты.

```azurecli-interactive
deploymentName='ExampleDeployment'$(date +"%d-%b-%Y")
```

При выполнении параллельных развертываний в одной группе ресурсов с тем же именем развертывания завершается только Последнее развертывание. Все развертывания с тем же именем, которые не были завершены, заменяются последним развертыванием. Например, если вы запускаете развертывание с именем, `newStorage` которое развертывает учетную запись хранения с именем `storage1` и в то же время запускает другое развертывание с именем, `newStorage` которое развертывает учетную запись хранения с именем `storage2` , развертывается только одна учетная запись хранения. Результирующая учетная запись хранения называется `storage2` .

Однако при запуске развертывания с именем, `newStorage` которое развертывает учетную запись хранения с именем `storage1` , и сразу после завершения работы вы запускаете другое развертывание с именем, `newStorage` которое развертывает учетную запись хранения с именем `storage2` , а затем у вас есть две учетные записи хранения. Одна из них называется `storage1` , а другая называется `storage2` . Но в журнале развертывания имеется только одна запись.

При указании уникального имени для каждого развертывания их можно запускать параллельно без конфликтов. Если вы запускаете развертывание с именем, `newStorage1` которое развертывает учетную запись хранения с именем `storage1` и в то же время запускает другое развертывание с именем, `newStorage2` которое развертывает учетную запись хранения с именем `storage2` , то в журнале развертывания есть две учетные записи хранения и две записи.

Чтобы избежать конфликтов с параллельными развертываниями и обеспечить уникальность записей в журнале развертывания, присвойте каждому развертыванию уникальное имя.

## <a name="deploy-template-spec"></a>Развертывание спецификации шаблона

> [!NOTE]
> В настоящее время Azure CLI не поддерживает создание спецификаций шаблонов, предоставляя файлы Бицеп. Однако можно создать файл Бицеп с ресурсом [Microsoft. Resources/темплатеспекс](/azure/templates/microsoft.resources/templatespecs) , чтобы развернуть спецификацию шаблона. Ниже приведен [Пример](https://github.com/Azure/azure-docs-json-samples/blob/master/create-template-spec-using-template/azuredeploy.bicep).

Вместо развертывания локального или удаленного шаблона можно создать [спецификацию шаблона](template-specs.md). Спецификация шаблона — это ресурс в подписке Azure, который содержит шаблон ARM. Это позволяет легко обеспечить безопасный общий доступ к шаблону для пользователей в вашей организации. Используйте управление доступом на основе ролей Azure (Azure RBAC), чтобы предоставить доступ к спецификации шаблона. Сейчас эта функция доступна в предварительной версии.

В следующих примерах показано, как создать и развернуть спецификацию шаблона.

Сначала создайте шаблон Spec, указав шаблон ARM.

```azurecli
az ts create \
  --name storageSpec \
  --version "1.0" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "./mainTemplate.json"
```

Затем получите идентификатор для спецификации шаблона и разверните его.

```azurecli
id = $(az ts show --name storageSpec --resource-group templateSpecRG --version "1.0" --query "id")

az deployment group create \
  --resource-group demoRG \
  --template-spec $id
```

Дополнительные сведения см. в разделе [спецификации шаблонов Azure Resource Manager (Предварительная версия)](template-specs.md).

## <a name="preview-changes"></a>Предварительный просмотр изменений

Перед развертыванием шаблона можно просмотреть изменения, которые шаблон будет вносить в вашу среду. Используйте [операцию "что если](template-deploy-what-if.md) ", чтобы убедиться, что шаблон вносит необходимые изменения. Что если также проверяет шаблон на наличие ошибок.

## <a name="parameters"></a>Параметры

Чтобы передать значения параметров, можно использовать встроенные параметры или файл параметров.

### <a name="inline-parameters"></a>Встроенные параметры

Чтобы передать встроенные параметры, укажите значения в `parameters`. Например, чтобы передать строку и массив в шаблон в оболочке bash, используйте:

```azurecli-interactive
az deployment group create \
  --resource-group testgroup \
  --template-file <path-to-template-or-bicep> \
  --parameters exampleString='inline string' exampleArray='("value1", "value2")'
```

Если вы используете Azure CLI в командной строке Windows (CMD) или PowerShell, передайте массив в формате: `exampleArray="['value1','value2']"` .

Можно также получить содержимое файла и предоставлять его в виде встроенного параметра.

```azurecli-interactive
az deployment group create \
  --resource-group testgroup \
  --template-file <path-to-template-or-bicep> \
  --parameters exampleString=@stringContent.txt exampleArray=@arrayContent.json
```

Получение значения параметра из файла полезно в тех случаях, когда есть необходимость предоставить значения конфигурации. Например, вы можете предоставить [значения cloud-init для виртуальной машины Linux](../../virtual-machines/linux/using-cloud-init.md).

_arrayContent.jsв_ формате:

```json
[
    "value1",
    "value2"
]
```

Чтобы передать объект, например для установки тегов, используйте JSON. Например, шаблон может содержать параметр, подобный приведенному ниже:

```json
    "resourceTags": {
      "type": "object",
      "defaultValue": {
        "Cost Center": "IT Department"
      }
    }
```

В этом случае можно передать строку JSON, чтобы задать параметр, как показано в следующем скрипте Bash:

```bash
tags='{"Owner":"Contoso","Cost Center":"2345-324"}'
az deployment group create --name addstorage  --resource-group myResourceGroup \
--template-file $templateFile \
--parameters resourceName=abcdef4556 resourceTags="$tags"
```

Используйте двойные кавычки вокруг JSON, который необходимо передать в объект.

### <a name="parameter-files"></a>Файлы параметров

Вместо передачи параметров в виде встроенных значений в сценарии вам может быть проще использовать JSON-файл, содержащий значения параметров. Файл параметров должен находиться в локальной среде. Внешние файлы параметров не поддерживаются в Azure CLI. Как шаблон ARM, так и файл Бицеп используют файлы параметров JSON.

Дополнительные сведения о файле параметров см. в статье [Создание файла параметров Resource Manager](parameter-files.md).

Чтобы передать локальный файл параметров, используйте `@` команду, чтобы указать локальный файл с именем _storage.parameters.json_.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

## <a name="handle-extended-json-format"></a>Обработку расширенного формата JSON

Чтобы развернуть шаблон с многострочными строками или комментариями с помощью Azure CLI с версией 2.3.0 или более ранней, необходимо использовать `--handle-extended-json-format` параметр.  Пример:

```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2018-10-01",
  "name": "[variables('vmName')]", // to customize name, change it in variables
  "location": "[
    parameters('location')
    ]", //defaults to resource group location
  /*
    storage account and network interface
    must be deployed first
  */
  "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
  ],
```

## <a name="next-steps"></a>Дальнейшие действия

* Если возникнет ошибка при развертывании, выполните откат по инструкциям из статьи [Откат к работоспособному развертыванию в случае ошибки](rollback-on-error.md).
* Сведения о том, как указать способ обработки ресурсов, которые существуют в группе ресурсов, но не определены в шаблоне, см. в [описании режимов развертывания с помощью Azure Resource Manager](deployment-modes.md).
* Сведения о том, как определить параметры в шаблоне, см. [в разделе Общие сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Советы по устранению распространенных ошибок развертывания см. в разделе [Устранение распространенных ошибок развертывания в Azure с помощью Azure Resource Manager](common-deployment-errors.md).
