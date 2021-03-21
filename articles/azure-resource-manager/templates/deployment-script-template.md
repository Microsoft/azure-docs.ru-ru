---
title: Использование сценариев развертывания в шаблонах | Документация Майкрософт
description: Использование сценариев развертывания в шаблонах Azure Resource Manager.
services: azure-resource-manager
author: mumian
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 03/18/2021
ms.author: jgao
ms.openlocfilehash: 130deea4e5998d696065df4854a47bf7ffd1183c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104594248"
---
# <a name="use-deployment-scripts-in-arm-templates"></a>Использование скриптов развертывания в шаблонах ARM

Узнайте, как использовать скрипты развертывания в шаблонах ресурсов Azure (шаблоны ARM). С помощью нового типа ресурсов `Microsoft.Resources/deploymentScripts` пользователи могут выполнять сценарии в развертываниях шаблонов и просматривать результаты выполнения. Эти сценарии можно использовать для выполнения настраиваемых действий, в том числе с их помощью можно:

- добавить пользователей в каталог;
- выполнить операции плоскости данных, например копирование больших двоичных объектов или базы данных начальных значений;
- найти и проверить ключ лицензии;
- создать самозаверяющий сертификат;
- создать объект в Azure AD;
- найти блоки IP-адресов из пользовательской системы.

Преимущества сценария развертывания:

- Простое программирование, использование и отладка. Сценарии развертывания можно разрабатывать в средах разработки на свой выбор. Эти сценарии можно внедрять в шаблоны или во внешние файлы сценариев.
- Можно указать язык и платформу сценария. В настоящее время поддерживаются сценарии развертывания Azure PowerShell и Azure CLI в среде Linux.
- В сценарий можно передавать аргументы командной строки.
- Можно указать выходные данные сценария и передать их в развертывание.

Ресурс сценария развертывания доступен только в регионах, где доступен экземпляр контейнера Azure.  Ознакомьтесь с разделом [Доступность ресурсов службы "Экземпляры контейнеров Azure" в регионах Azure](../../container-instances/container-instances-region-availability.md).

> [!IMPORTANT]
> Учетная запись хранения и экземпляр контейнера необходимы для выполнения сценария и устранения неполадок. Вы можете указать существующую учетную запись хранения. В противном случае учетная запись хранения вместе с экземпляром контейнера автоматически создается службой сценариев. Эти два создаваемых автоматически ресурса обычно удаляются службой сценариев, когда выполнение сценария развертывания достигает состояния завершения. Плата взимается за ресурсы, пока они не будут удалены. Дополнительные сведения в разделе об [очистке ресурсов сценариев развертывания](#clean-up-deployment-script-resources).

> [!IMPORTANT]
> API ресурсов deploymentScripts версии 2020-10-01 поддерживает [онбехалфофтокенс (OBO)](../../active-directory/develop/v2-oauth2-on-behalf-of-flow.md). С помощью OBO служба скрипта развертывания использует маркер участника развертывания для создания базовых ресурсов для выполнения скриптов развертывания, включая экземпляр контейнера Azure, учетную запись хранения Azure и назначения ролей для управляемого удостоверения. В более старой версии API для создания этих ресурсов используется управляемое удостоверение.
> Логика повторных попыток входа Azure теперь встроена в скрипт-оболочку. При предоставлении разрешений в том же шаблоне, где выполняются скрипты развертывания. Служба скрипта развертывания повторяет попытку входа в течение 10 минут с интервалом 10 секунд, пока не будет реплицирована назначение управляемой роли идентификации.

## <a name="configure-the-minimum-permissions"></a>Настройка минимальных разрешений

Для API скрипта развертывания версии 2020-10-01 или более поздней субъект развертывания используется для создания базовых ресурсов, необходимых для выполнения ресурса скрипта развертывания — учетной записи хранения и экземпляра контейнера Azure. Если скрипту требуется пройти проверку подлинности в Azure и выполнить действия, связанные с Azure, рекомендуется предоставить скрипту управляемое удостоверение, назначаемое пользователем. Управляемое удостоверение должно иметь необходимый доступ для завершения операции в скрипте.

Чтобы настроить разрешения с минимальными правами доступа, вам потребуется:

- Назначьте субъекту развертывания пользовательскую роль со следующими свойствами:

  ```json
  {
    "roleName": "deployment-script-minimum-privilege-for-deployment-principal",
    "description": "Configure least privilege for the deployment principal in deployment script",
    "type": "customRole",
    "IsCustom": true,
    "permissions": [
      {
        "actions": [
          "Microsoft.Storage/storageAccounts/*",
          "Microsoft.ContainerInstance/containerGroups/*",
          "Microsoft.Resources/deployments/*",
          "Microsoft.Resources/deploymentScripts/*"
        ],
      }
    ],
    "assignableScopes": [
      "[subscription().id]"
    ]
  }
  ```

  Если служба хранилища Azure и поставщики ресурсов экземпляров контейнеров Azure не зарегистрированы, необходимо также добавить `Microsoft.Storage/register/action` и `Microsoft.ContainerInstance/register/action` .

- Если используется управляемое удостоверение, участнику развертывания требуется роль **управляемого оператора идентификации** (встроенная роль), назначенная управляемому ресурсу удостоверений.

## <a name="sample-templates"></a>Примеры шаблонов

Ниже приведен пример JSON. Дополнительные сведения см. в последней [схеме шаблона](/azure/templates/microsoft.resources/deploymentscripts).

```json
{
  "type": "Microsoft.Resources/deploymentScripts",
  "apiVersion": "2020-10-01",
  "name": "runPowerShellInline",
  "location": "[resourceGroup().location]",
  "kind": "AzurePowerShell", // or "AzureCLI"
  "identity": {
    "type": "userAssigned",
    "userAssignedIdentities": {
      "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myID": {}
    }
  },
  "properties": {
    "forceUpdateTag": "1",
    "containerSettings": {
      "containerGroupName": "mycustomaci"
    },
    "storageAccountSettings": {
      "storageAccountName": "myStorageAccount",
      "storageAccountKey": "myKey"
    },
    "azPowerShellVersion": "3.0",  // or "azCliVersion": "2.0.80",
    "arguments": "-name \\\"John Dole\\\"",
    "environmentVariables": [
      {
        "name": "UserName",
        "value": "jdole"
      },
      {
        "name": "Password",
        "secureValue": "jDolePassword"
      }
    ],
    "scriptContent": "
      param([string] $name)
      $output = 'Hello {0}. The username is {1}, the password is {2}.' -f $name,${Env:UserName},${Env:Password}
      Write-Output $output
      $DeploymentScriptOutputs = @{}
      $DeploymentScriptOutputs['text'] = $output
    ", // or "primaryScriptUri": "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-helloworld.ps1",
    "supportingScriptUris":[],
    "timeout": "PT30M",
    "cleanupPreference": "OnSuccess",
    "retentionInterval": "P1D"
  }
}
```

> [!NOTE]
> Пример предназначен для демонстрационных целей. Свойства `scriptContent` и `primaryScriptUri` не могут сосуществовать в шаблоне.

Сведения о значениях свойств.

- `identity`. Для API скрипта развертывания версии 2020-10-01 или более поздней назначенное пользователем управляемое удостоверение является необязательным, если в скрипт не требуется выполнять какие-либо действия, связанные с Azure.  Для API версии 2019-10-01-Preview требуется управляемое удостоверение, так как служба скрипта развертывания использует ее для выполнения скриптов. Сейчас поддерживаются только управляемые удостоверения, назначаемые пользователем.
- `kind`. Укажите тип скрипта. В настоящее время поддерживаются сценарии Azure PowerShell и Azure CLI. Допустимые значения: **AzurePowerShell** и **AzureCLI**.
- `forceUpdateTag`. Изменение этого значения между развертываниями шаблона приводит к повторному выполнению скрипта развертывания. Если используются `newGuid()` `utcNow()` функции или, обе функции можно использовать только в значении по умолчанию для параметра. Дополнительные сведения о выполнении скрипта несколько раз см. [здесь](#run-script-more-than-once).
- `containerSettings`: Укажите параметры для настройки экземпляра контейнера Azure.  `containerGroupName` параметр предназначен для указания имени группы контейнеров. Если не указано, имя группы создается автоматически.
- `storageAccountSettings`: Укажите параметры для использования существующей учетной записи хранения. Если этот параметр не указан, учетная запись хранения будет создана автоматически. Ознакомьтесь с [использованием имеющейся учетной записи хранения](#use-existing-storage-account).
- `azPowerShellVersion`/`azCliVersion`: Укажите версию модуля для использования. См. список [поддерживаемых версий Azure PowerShell](https://mcr.microsoft.com/v2/azuredeploymentscripts-powershell/tags/list). См. список [поддерживаемых версий Azure CLI](https://mcr.microsoft.com/v2/azure-cli/tags/list).

  >[!IMPORTANT]
  > Скрипт развертывания использует доступные образы CLI из реестра контейнеров Майкрософт (мкр). Сертификация образа CLI для сценария развертывания занимает около одного месяца. Не используйте версии CLI, выпущенные менее 30 дней назад. Чтобы найти даты выпуска образов, ознакомьтесь с [заметками о выпуске Azure CLI](/cli/azure/release-notes-azure-cli). Если используется неподдерживаемая версия, в сообщении об ошибке выводится список поддерживаемых версий.

- `arguments`. Укажите значения параметров. Значения разделяются пробелами.

  Скрипты развертывания разбивают аргументы на массив строк путем вызова системного вызова [коммандлинетоаргвв ](/windows/win32/api/shellapi/nf-shellapi-commandlinetoargvw) . Этот шаг необходим, так как аргументы передаются в качестве [Свойства команды](/rest/api/container-instances/containergroups/createorupdate#containerexec) в экземпляр контейнера Azure, а свойство Command — в виде массива строк.

  Если аргументы содержат escape-символы, используйте [жсонескапер](https://www.jsonescaper.com/) для двойного экранирования символов. Вставьте исходную экранированную строку в средство, а затем выберите **escape**.  Средство выводит строку с двойным экранированием. Например, в предыдущем примере шаблона аргумент имеет значение `-name \"John Dole\"` . Экранированная строка имеет значение `-name \\\"John Dole\\\"` .

  Чтобы передать параметр шаблона ARM типа Object в качестве аргумента, преобразуйте объект в строку с помощью функции [String ()](./template-functions-string.md#string) , а затем используйте функцию [Replace ()](./template-functions-string.md#replace) , чтобы заменить Any на `\"` `\\\"` . Пример:

  ```json
  replace(string(parameters('tables')), '\"', '\\\"')
  ```

  Дополнительные сведения см. в [примере шаблона](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-jsonEscape.json).

- `environmentVariables`: Укажите переменные среды для передачи в скрипт. Дополнительные сведения см. в статье [Разработка сценариев развертывания](#develop-deployment-scripts).
- `scriptContent`. Укажите содержимое скрипта. Чтобы запустить внешний сценарий, используйте `primaryScriptUri`. Примеры см. в разделах [Использование встроенных сценариев](#use-inline-scripts) и [Использование внешних сценариев](#use-external-scripts).
  > [!NOTE]
  > Портал Azure не может выполнить синтаксический анализ скрипта развертывания с несколькими строками. Чтобы развернуть шаблон с помощью скрипта развертывания из портал Azure, можно либо создать цепочку команд PowerShell с помощью точек с запятой в одну строку, либо использовать `primaryScriptUri` свойство с внешним файлом скрипта.

- `primaryScriptUri`: Укажите общедоступный URL-адрес основного скрипта развертывания с поддерживаемыми расширениями файлов. Дополнительные сведения см. в разделе [использование внешних скриптов](#use-external-scripts).
- `supportingScriptUris`: Укажите массив общедоступных URL-адресов для поддержки файлов, которые вызываются в либо `scriptContent` `primaryScriptUri` . Дополнительные сведения см. в разделе [использование внешних скриптов](#use-external-scripts).
- `timeout`. Укажите максимально допустимое время выполнения скрипта в [формате ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). Значение по умолчанию — **P1D**.
- `cleanupPreference`. Укажите параметры очистки ресурсов развертывания после перехода сценария в состояние завершения. Значение по умолчанию — **Always**, что означает удаление ресурсов независимо от состояния завершения (Succeeded, Failed, Canceled). Дополнительные сведения в разделе об [очистке ресурсов скриптов развертывания](#clean-up-deployment-script-resources).
- `retentionInterval`: Укажите интервал, в течение которого служба будет хранить ресурсы скрипта развертывания после того, как выполнение скрипта развертывания достигнет состояния терминала. По истечении этого времени ресурсы сценария развертывания будут удалены. Длительность основывается на [методике ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). Интервал хранения составляет от 1 до 26 часов (PT26H). Это свойство используется, если для параметра `cleanupPreference` установлено значение **OnExpiration**. Дополнительные сведения в разделе об [очистке ресурсов скриптов развертывания](#clean-up-deployment-script-resources).

### <a name="additional-samples"></a>Дополнительные примеры

- [Пример 1](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault.json). Создание хранилища ключей и использование скрипта развертывания для назначения сертификата в хранилище ключей.
- [Пример 2](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault-subscription.json). Создание группы ресурсов на уровне подписки, создание хранилища ключей в группе ресурсов, а затем использование скрипта развертывания для назначения сертификата в хранилище ключей.
- [Пример 3](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault-mi.json). Создание назначаемого пользователем управляемого удостоверения, назначение роли участника для удостоверения на уровне группы ресурсов, создание хранилища ключей, а затем использование скрипта развертывания для назначения сертификата в хранилище ключей.

## <a name="use-inline-scripts"></a>Использование встроенных сценариев

В приведенном ниже шаблоне определен один ресурс с типом `Microsoft.Resources/deploymentScripts`. Выделенная часть — встроенный сценарий.

:::code language="json" source="~/resourcemanager-templates/deployment-script/deploymentscript-helloworld.json" range="1-44" highlight="24-30":::

> [!NOTE]
> Поскольку Встроенные скрипты развертывания заключены в двойные кавычки, строки внутри скриптов развертывания должны быть экранированы с помощью обратной косой черты (**&#92;**) или заключены в одинарные кавычки. Можно также использовать подстановку строк, как показано в предыдущем примере JSON.

Сценарий принимает один параметр и выводит значение параметра. `DeploymentScriptOutputs` используется для хранения выходных данных. В разделе выходные данные в `value` строке показано, как получить доступ к сохраненным значениям. `Write-Output` используется для отладки. Сведения о том, как получить доступ к выходному файлу, см. в статье [мониторинг и устранение неполадок сценариев развертывания](#monitor-and-troubleshoot-deployment-scripts). Описания свойств см. в разделе [Примеры шаблонов](#sample-templates).

Чтобы выполнить сценарий, выберите **Попробовать**. Откроется Cloud Shell. Вставьте приведенный ниже сценарий PowerShell в окно оболочки.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the name of the resource group to be created"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location $location

New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-helloworld.json"

Write-Host "Press [ENTER] to continue ..."
```

Он возвращает примерно такие выходные данные:

![Выходные данные сценария развертывания шаблона Resource Manager Hello World](./media/deployment-script-template/resource-manager-template-deployment-script-helloworld-output.png)

## <a name="use-external-scripts"></a>Использование внешних сценариев

Помимо встроенных сценариев можно также использовать внешние файлы сценария. Поддерживаются только основные сценарии PowerShell с расширением файла _PS1_. Для CLI основные сценарии могут иметь любое расширение (или не иметь его), если это допустимые сценарии Bash. Чтобы использовать внешние файлы сценария, замените `scriptContent` на `primaryScriptUri`. Пример:

```json
"primaryScriptUri": "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-helloworld.ps1",
```

Дополнительные сведения см. в [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/deployment-script/deploymentscript-helloworld-primaryscripturi.json).

Внешние файлы сценария должны быть доступны. Чтобы защитить файлы скриптов, хранящиеся в учетных записях хранения Azure, создайте маркер SAS и включите его в универсальный код ресурса (URI) для шаблона. Задайте срок действия, достаточный для выполнения развертывания. Дополнительные сведения см. [в статье Развертывание закрытого шаблона ARM с помощью маркера SAS](./secure-template-with-sas-token.md).

Вы несете ответственность за обеспечение целостности скриптов, на которые ссылается скрипт развертывания, `primaryScriptUri` или `supportingScriptUris` . Указывайте только те надежные сценарии.

## <a name="use-supporting-scripts"></a>Использование вспомогательных сценариев

Можно разделить сложную логику и поместить ее в один или несколько вспомогательных файлов сценария. Свойство `supportingScriptUris` позволяет при необходимости указать массив универсальных кодов ресурса (URI) для вспомогательных файлов сценария.

```json
"scriptContent": "
    ...
    ./Create-Cert.ps1
    ...
"

"supportingScriptUris": [
  "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/create-cert.ps1"
],
```

Вспомогательные файлы сценария можно вызывать как из встроенных сценариев, так и из основных файлов сценария. Поддержка файлов сценария не накладывает ограничений на расширение файла.

Вспомогательные файлы копируются в `azscripts/azscriptinput` среду выполнения. Используйте относительный путь, чтобы указать вспомогательные файлы в файлах встроенных сценариев и основных сценариев.

## <a name="work-with-outputs-from-powershell-script"></a>Использование выходных данных сценария PowerShell

В следующем шаблоне показано, как передавать значения между двумя `deploymentScripts` ресурсами:

:::code language="json" source="~/resourcemanager-templates/deployment-script/deploymentscript-basic.json" range="1-68" highlight="30-31,50":::

В первом ресурсе определяется переменная с именем `$DeploymentScriptOutputs` и используется для хранения выходных значений. Чтобы получить доступ к выходному значению из другого ресурса в шаблоне, используйте следующий код.

```json
reference('<ResourceName>').output.text
```

## <a name="work-with-outputs-from-cli-script"></a>Использование выходных данных сценария CLI

В отличие от сценария развертывания PowerShell, поддержка интерфейса командной строки или Bash не предоставляет общую переменную для хранения выходных данных сценария, вместо этого существует переменная среды `AZ_SCRIPTS_OUTPUT_PATH` , в которой хранится расположение, в котором находится файл выходных данных сценария. Если сценарий развертывания запускается из шаблона Resource Manager, эту переменную среды автоматически задает оболочка Bash.

Выходные данные скрипта развертывания должны быть сохранены в `AZ_SCRIPTS_OUTPUT_PATH` расположении, а выходные данные должны быть допустимым строковым объектом JSON. Содержимое файла должно быть сохранено в виде пар "ключ-значение". Например, массив строк хранится в виде `{ "MyResult": [ "foo", "bar"] }` .  Например, сохранение только результатов массива `[ "foo", "bar" ]` является недопустимым.

:::code language="json" source="~/resourcemanager-templates/deployment-script/deploymentscript-basic-cli.json" range="1-44" highlight="32":::

В предыдущем примере используется компонент [jq](https://stedolan.github.io/jq/). Он поставляется с образами контейнеров. Ознакомьтесь с [настройкой среды разработки](#configure-development-environment).

## <a name="use-existing-storage-account"></a>Использовать существующую учетную запись хранения

Учетная запись хранения и экземпляр контейнера необходимы для выполнения сценария и устранения неполадок. Вы можете указать существующую учетную запись хранения. В противном случае учетная запись хранения вместе с экземпляром контейнера автоматически создается службой сценариев. Требования к использованию существующей учетной записи хранения.

- Поддерживаются следующие типы учетных записей хранения:

    | номер SKU             | Поддерживаемый тип     |
    |-----------------|--------------------|
    | Premium_LRS     | FileStorage        |
    | Premium_ZRS     | FileStorage        |
    | Standard_GRS    | Хранилище, StorageV2 |
    | Standard_GZRS   | Хранилище версии 2          |
    | Standard_LRS    | Хранилище, StorageV2 |
    | Standard_RAGRS  | Хранилище, StorageV2 |
    | Standard_RAGZRS | Хранилище версии 2          |
    | Standard_ZRS    | Хранилище версии 2          |

    Эти комбинации поддерживают общие файловые ресурсы. Дополнительные сведения см. в статье [Создание общего файлового ресурса Azure](../../storage/files/storage-how-to-create-file-share.md) и [типов учетных записей хранения](../../storage/common/storage-account-overview.md).

- Правила брандмауэра учетной записи хранения еще не поддерживаются. Дополнительные сведения см. в статье [Настройка брандмауэров службы хранилища Azure и виртуальных сетей](../../storage/common/storage-network-security.md).
- Участник развертывания должен иметь разрешения на управление учетной записью хранения, включая чтение, создание и удаление файловых ресурсов.

Чтобы указать существующую учетную запись хранения, добавьте следующий код JSON в элемент Property объекта `Microsoft.Resources/deploymentScripts` :

```json
"storageAccountSettings": {
  "storageAccountName": "myStorageAccount",
  "storageAccountKey": "myKey"
},
```

- `storageAccountName`: укажите имя учетной записи хранения.
- `storageAccountKey`: укажите один из ключей учетной записи хранения. Для получения ключа можно использовать функцию [listKeys ()](./template-functions-resource.md#listkeys) . Пример:

    ```json
    "storageAccountSettings": {
        "storageAccountName": "[variables('storageAccountName')]",
        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
    }
    ```

Полный пример определения `Microsoft.Resources/deploymentScripts` приведен в разделе [Примеры шаблонов](#sample-templates).

Если используется существующая учетная запись хранения, служба сценариев создает общую папку с уникальным именем. Сведения о том, как служба сценариев очищает файловый ресурс, см. в разделе [Очистка ресурсов сценария развертывания](#clean-up-deployment-script-resources).

## <a name="develop-deployment-scripts"></a>Разработка сценариев развертывания

### <a name="handle-non-terminating-errors"></a>Обработка непрерывающих ошибок

Вы можете управлять тем, как PowerShell реагирует на устранимые ошибки, используя `$ErrorActionPreference` переменную в скрипте развертывания. Если переменная не задана в скрипте развертывания, служба скриптов использует значение по умолчанию **Continue**.

Служба скриптов задает для состояния подготовки ресурсов значение **Failed** , если скрипт встречает ошибку, несмотря на параметр `$ErrorActionPreference` .

### <a name="pass-secured-strings-to-deployment-script"></a>Передача защищенных строк в сценарий развертывания

Используя переменные среды (EnvironmentVariable) в экземплярах контейнеров, вы можете динамически изменять конфигурацию приложения или сценария, запущенного в контейнере. Сценарий развертывания обрабатывает незащищенные и защищенные переменные среды так же, как и экземпляр контейнера Azure. Узнайте больше об [определении переменных среды для экземпляров контейнеров](../../container-instances/container-instances-environment-variables.md#secure-values). Пример см. в разделе [примеры шаблонов](#sample-templates).

Максимальный допустимый размер для переменных среды составляет 64 КБ.

## <a name="monitor-and-troubleshoot-deployment-scripts"></a>Мониторинг и устранение неполадок сценариев развертывания

Для выполнения сценария служба сценариев создает [учетную запись хранения](../../storage/common/storage-account-overview.md) (если не указана существующая учетная запись хранения) и [экземпляр контейнера](../../container-instances/container-instances-overview.md). Если служба сценариев автоматически создает эти ресурсы, то оба `azscripts` ресурса имеют суффикс в именах ресурсов.

![Имена ресурсов сценария развертывания шаблона Resource Manager](./media/deployment-script-template/resource-manager-template-deployment-script-resources.png)

Пользовательский сценарий, результаты выполнения и файл stdout хранятся в общих папках учетной записи хранения. Существует папка с именем `azscripts` . В папке есть еще две папки для входных и выходных файлов: `azscriptinput` и `azscriptoutput` .

Выходная папка содержит файл _executionresult.json_ и выходной файл скрипта. Сообщение об ошибке выполнения сценария можно просмотреть в файле _executionresult.json_. Выходной файл создается только при успешном выполнении сценария. Входная папка содержит системный файл скрипта PowerShell и пользовательские файлы скрипта развертывания. Вы можете заменить файл пользовательского сценария развертывания измененным файлом и повторно запустить сценарий развертывания из экземпляра контейнера Azure.

### <a name="use-the-azure-portal"></a>Использование портала Azure

После развертывания ресурса скрипта развертывания ресурс отобразится в группе ресурсов в портал Azure. На следующем снимке экрана показана страница **обзора** ресурса скрипта развертывания.

![Общие сведения о портале сценариев развертывания шаблона диспетчер ресурсов](./media/deployment-script-template/resource-manager-deployment-script-portal.png)

На странице Обзор отображаются некоторые важные сведения о ресурсе, такие как **состояние подготовки**, **учетная запись хранения**, **экземпляр контейнера** и **журналы**.

В меню слева можно просмотреть содержимое скрипта развертывания, аргументы, передаваемые в скрипт, и выходные данные. Также можно экспортировать шаблон для скрипта развертывания, включая скрипт развертывания.

### <a name="use-powershell"></a>Использование PowerShell

С помощью Azure PowerShell можно управлять сценариями развертывания в области действия подписки или группы ресурсов:

- [Get-аздеплойментскрипт](/powershell/module/az.resources/get-azdeploymentscript): Получает или перечисляет скрипты развертывания.
- [Get-аздеплойментскриптлог](/powershell/module/az.resources/get-azdeploymentscriptlog): Получает журнал выполнения скрипта развертывания.
- [Remove-аздеплойментскрипт](/powershell/module/az.resources/remove-azdeploymentscript): Удаляет скрипт развертывания и связанные с ним ресурсы.
- [Save-аздеплойментскриптлог](/powershell/module/az.resources/save-azdeploymentscriptlog): сохраняет журнал выполнения скрипта развертывания на диске.

`Get-AzDeploymentScript`Выходные данные похожи на:

```output
Name                : runPowerShellInlineWithOutput
Id                  : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0618rg/providers/Microsoft.Resources/deploymentScripts/runPowerShellInlineWithOutput
ResourceGroupName   : myds0618rg
Location            : centralus
SubscriptionId      : 01234567-89AB-CDEF-0123-456789ABCDEF
ProvisioningState   : Succeeded
Identity            : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/mydentity1008rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuami
ScriptKind          : AzurePowerShell
AzPowerShellVersion : 3.0
StartTime           : 6/18/2020 7:46:45 PM
EndTime             : 6/18/2020 7:49:45 PM
ExpirationDate      : 6/19/2020 7:49:45 PM
CleanupPreference   : OnSuccess
StorageAccountId    : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0618rg/providers/Microsoft.Storage/storageAccounts/ftnlvo6rlrvo2azscripts
ContainerInstanceId : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0618rg/providers/Microsoft.ContainerInstance/containerGroups/ftnlvo6rlrvo2azscripts
Outputs             :
                      Key                 Value
                      ==================  ==================
                      text                Hello John Dole

RetentionInterval   : P1D
Timeout             : PT1H
```

### <a name="use-azure-cli"></a>Использование Azure CLI

С помощью Azure CLI можно управлять сценариями развертывания в области действия подписки или группы ресурсов:

- [AZ Deployment — скрипты Delete](/cli/azure/deployment-scripts#az-deployment-scripts-delete): Удаление скрипта развертывания.
- [AZ Deployment-Script List](/cli/azure/deployment-scripts#az-deployment-scripts-list): вывод списка всех скриптов развертывания.
- [AZ Deployment — скрипты](/cli/azure/deployment-scripts#az-deployment-scripts-show): получение скрипта развертывания.
- [AZ Deployment-Scripts показывать-log](/cli/azure/deployment-scripts#az-deployment-scripts-show-log): показывать журналы скрипта развертывания.

Выходные данные команды List похожи на:

```json
[
  {
    "arguments": "-name \\\"John Dole\\\"",
    "azPowerShellVersion": "3.0",
    "cleanupPreference": "OnSuccess",
    "containerSettings": {
      "containerGroupName": null
    },
    "environmentVariables": null,
    "forceUpdateTag": "20200625T025902Z",
    "id": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Resources/deploymentScripts/runPowerShellInlineWithOutput",
    "identity": {
      "tenantId": "01234567-89AB-CDEF-0123-456789ABCDEF",
      "type": "userAssigned",
      "userAssignedIdentities": {
        "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myidentity1008rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuami": {
          "clientId": "01234567-89AB-CDEF-0123-456789ABCDEF",
          "principalId": "01234567-89AB-CDEF-0123-456789ABCDEF"
        }
      }
    },
    "kind": "AzurePowerShell",
    "location": "centralus",
    "name": "runPowerShellInlineWithOutput",
    "outputs": {
      "text": "Hello John Dole"
    },
    "primaryScriptUri": null,
    "provisioningState": "Succeeded",
    "resourceGroup": "myds0624rg",
    "retentionInterval": "1 day, 0:00:00",
    "scriptContent": "\r\n          param([string] $name)\r\n          $output = \"Hello {0}\" -f $name\r\n          Write-Output $output\r\n          $DeploymentScriptOutputs = @{}\r\n          $DeploymentScriptOutputs['text'] = $output\r\n        ",
    "status": {
      "containerInstanceId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.ContainerInstance/containerGroups/64lxews2qfa5uazscripts",
      "endTime": "2020-06-25T03:00:16.796923+00:00",
      "error": null,
      "expirationTime": "2020-06-26T03:00:16.796923+00:00",
      "startTime": "2020-06-25T02:59:07.595140+00:00",
      "storageAccountId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Storage/storageAccounts/64lxews2qfa5uazscripts"
    },
    "storageAccountSettings": null,
    "supportingScriptUris": null,
    "systemData": {
      "createdAt": "2020-06-25T02:59:04.750195+00:00",
      "createdBy": "someone@contoso.com",
      "createdByType": "User",
      "lastModifiedAt": "2020-06-25T02:59:04.750195+00:00",
      "lastModifiedBy": "someone@contoso.com",
      "lastModifiedByType": "User"
    },
    "tags": null,
    "timeout": "1:00:00",
    "type": "Microsoft.Resources/deploymentScripts"
  }
]
```

### <a name="use-rest-api"></a>Использование API-интерфейса RESTful

Сведения о развертывании ресурсов сценария развертывания можно получить на уровне группы ресурсов и на уровне подписки с помощью REST API.

```rest
/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/microsoft.resources/deploymentScripts/<DeploymentScriptResourceName>?api-version=2020-10-01
```

```rest
/subscriptions/<SubscriptionID>/providers/microsoft.resources/deploymentScripts?api-version=2020-10-01
```

В следующем примере используется [ARMClient](https://github.com/projectkudu/ARMClient).

```azurepowershell
armclient login
armclient get /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourcegroups/myrg/providers/microsoft.resources/deploymentScripts/myDeployementScript?api-version=2020-10-01
```

Выходные данные должны быть следующего вида.

```json
{
  "kind": "AzurePowerShell",
  "identity": {
    "type": "userAssigned",
    "tenantId": "01234567-89AB-CDEF-0123-456789ABCDEF",
    "userAssignedIdentities": {
      "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myidentity1008rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuami": {
        "principalId": "01234567-89AB-CDEF-0123-456789ABCDEF",
        "clientId": "01234567-89AB-CDEF-0123-456789ABCDEF"
      }
    }
  },
  "location": "centralus",
  "systemData": {
    "createdBy": "someone@contoso.com",
    "createdByType": "User",
    "createdAt": "2020-06-25T02:59:04.7501955Z",
    "lastModifiedBy": "someone@contoso.com",
    "lastModifiedByType": "User",
    "lastModifiedAt": "2020-06-25T02:59:04.7501955Z"
  },
  "properties": {
    "provisioningState": "Succeeded",
    "forceUpdateTag": "20200625T025902Z",
    "azPowerShellVersion": "3.0",
    "scriptContent": "\r\n          param([string] $name)\r\n          $output = \"Hello {0}\" -f $name\r\n          Write-Output $output\r\n          $DeploymentScriptOutputs = @{}\r\n          $DeploymentScriptOutputs['text'] = $output\r\n        ",
    "arguments": "-name \\\"John Dole\\\"",
    "retentionInterval": "P1D",
    "timeout": "PT1H",
    "containerSettings": {},
    "status": {
      "containerInstanceId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.ContainerInstance/containerGroups/64lxews2qfa5uazscripts",
      "storageAccountId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Storage/storageAccounts/64lxews2qfa5uazscripts",
      "startTime": "2020-06-25T02:59:07.5951401Z",
      "endTime": "2020-06-25T03:00:16.7969234Z",
      "expirationTime": "2020-06-26T03:00:16.7969234Z"
    },
    "outputs": {
      "text": "Hello John Dole"
    },
    "cleanupPreference": "OnSuccess"
  },
  "id": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Resources/deploymentScripts/runPowerShellInlineWithOutput",
  "type": "Microsoft.Resources/deploymentScripts",
  "name": "runPowerShellInlineWithOutput"
}

```

Приведенный ниже REST API возвращает журнал.

```rest
/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/microsoft.resources/deploymentScripts/<DeploymentScriptResourceName>/logs?api-version=2020-10-01
```

Он работает только до удаления ресурсов сценария развертывания.

Выберите **Показать скрытые типы**, чтобы просмотреть список ресурсов deploymentScripts на портале.

![Сценарий развертывания шаблона Resource Manager, показать скрытые типы, портал](./media/deployment-script-template/resource-manager-deployment-script-portal-show-hidden-types.png)

## <a name="clean-up-deployment-script-resources"></a>Очистка ресурсов сценария развертывания

Учетная запись хранения и экземпляр контейнера необходимы для выполнения сценария и устранения неполадок. Вы можете указать существующую учетную запись хранения. В противном случае учетная запись хранения вместе с экземпляром контейнера автоматически создается службой сценариев. Эти два создаваемых автоматически ресурса удаляются службой сценариев, когда выполнение сценария развертывания достигает состояния завершения. Плата взимается за ресурсы, пока они не будут удалены. Сведения о ценах приведены на страницах цен на [экземпляры контейнеров](https://azure.microsoft.com/pricing/details/container-instances/) и [службу хранилища Azure](https://azure.microsoft.com/pricing/details/storage/).

Жизненный цикл этих ресурсов определяется приведенными ниже свойствами в шаблоне.

- `cleanupPreference`: Очистка параметров при выполнении скрипта в состоянии терминала. Поддерживаются такие значения:

  - **Always**. Удаление созданных автоматически ресурсов после завершения выполнения сценария. Если используется существующая учетная запись хранения, то служба сценариев удаляет общую папку, созданную в этой учетной записи хранения. Так как `deploymentScripts` ресурс по-прежнему может присутствовать после очистки ресурсов, служба скриптов сохраняет результаты выполнения скрипта, например stdout, Outputs и возвращаемое значение, перед удалением ресурсов.
  - **OnSuccess**. Удаление созданных автоматически ресурсов только после успешного выполнения сценария. Если используется существующая учетная запись хранения, то служба сценариев удаляет файловый ресурс только после успешного выполнения сценария. Вы по-прежнему сможете получить доступ к ресурсам для поиска отладочной информации.
  - **Onexpir**: удаляет автоматически созданные ресурсы только после `retentionInterval` истечения срока действия параметра. Если используется существующая учетная запись хранения, служба скриптов удаляет файловый ресурс, но оставляет учетную запись хранения.

- `retentionInterval`: Укажите интервал времени, в течение которого ресурс скрипта будет храниться, после чего он будет удален.

> [!NOTE]
> Не рекомендуется использовать учетную запись хранения и экземпляр контейнера, созданные службой сценария для других целей. Эти два ресурса могут быть удалены в зависимости от жизненного цикла сценария.

Экземпляр контейнера и учетная запись хранения удаляются в соответствии с `cleanupPreference` . Однако если сценарий завершается ошибкой и `cleanupPreference` не имеет значение **Always**, то процесс развертывания автоматически удерживает контейнер в течение одного часа. Этот час можно использовать для устранения неполадок сценария. Если вы хотите, чтобы контейнер выполнялся после успешного развертывания, добавьте в сценарий шаг спящего режима. Например, добавьте [Start-Sleep](/powershell/module/microsoft.powershell.utility/start-sleep) в конец скрипта. Если вы не добавите шаг спящего режима, контейнер будет установлен в состояние терминала и к нему нельзя получить доступ, даже если он еще не был удален.

## <a name="run-script-more-than-once"></a>Многократное выполнение сценария

Выполнение сценария развертывания является идемпотентной операцией. Если ни одно из `deploymentScripts` свойств ресурсов (включая встроенный скрипт) не изменилось, скрипт не выполняется при повторном развертывании шаблона. Служба сценариев развертывания сравнивает имена ресурсов в шаблоне с существующими ресурсами в той же группе ресурсов. Существуют два варианта многократного выполнения одного сценария.

- Измените имя `deploymentScripts` ресурса. Например, используйте функцию шаблона [utcNow](./template-functions-date.md#utcnow) в качестве имени ресурса или части его имени. При изменении имени ресурса создается новый `deploymentScripts` ресурс. Рекомендуется хранить историю выполнения скрипта.

    > [!NOTE]
    > `utcNow`Функцию можно использовать только в значении по умолчанию для параметра.

- Укажите другое значение в свойстве шаблона `forceUpdateTag`. Например, используйте `utcNow` в качестве значения.

> [!NOTE]
> Напишите идемпотентные сценарии развертывания. Это гарантирует, что если они случайно будут выполнены повторно, это не приведет к изменениям системы. Например, если сценарий развертывания используется для создания ресурса Azure, убедитесь, что этот ресурс не существует, прежде чем создавать его, чтобы выполнение сценария могло завершиться успешно и вы не создали ресурс повторно.

## <a name="configure-development-environment"></a>Настройка среды разработки

Вы можете использовать предварительно настроенный образ контейнера в качестве среды разработки скриптов развертывания. Дополнительные сведения см. [в статье Настройка среды разработки для скриптов развертывания в шаблонах](./deployment-script-template-configure-dev.md).

После успешного тестирования скрипта его можно использовать в качестве скрипта развертывания в шаблонах.

## <a name="deployment-script-error-codes"></a>Коды ошибок скрипта развертывания

| Код ошибки | Описание |
|------------|-------------|
| деплойментскриптинвалидоператион | Определение ресурса скрипта развертывания в шаблоне содержит недопустимые имена свойств. |
| деплойментскриптресаурцеконфликт | Не удается удалить ресурс скрипта развертывания, который находится в состоянии, отличном от терминала, и выполнение не превысило 1 час. Или не может повторно запустить тот же скрипт развертывания с тем же идентификатором ресурса (подписку, имя группы ресурсов и имя ресурса), но одновременно с другим содержимым тела скрипта. |
| деплойментскриптоператионфаилед | Ошибка при выполнении операции скрипта развертывания. Обратитесь в службу поддержки Майкрософт. |
| деплойментскриптсторажеаккаунтакцесскэйнотспеЦифиед | Для существующей учетной записи хранения не указан ключ доступа.|
| деплойментскриптконтаинерграупконтаинсинвалидконтаинерс | Группа контейнеров, созданная службой скриптов развертывания, была изменена извне, и добавлены недопустимые контейнеры. |
| деплойментскриптконтаинерграупиннонтерминалстате | Два или несколько ресурсов сценария развертывания используют одно и то же имя экземпляра контейнера Azure в одной группе ресурсов, и один из них еще не завершил выполнение. |
| деплойментскриптсторажеаккаунтинвалидкинд | Существующая учетная запись хранения типа Блобблобстораже или Блобстораже не поддерживает общие файловые ресурсы и не может использоваться. |
| деплойментскриптсторажеаккаунтинвалидкиндандску | Существующая учетная запись хранения не поддерживает файловые ресурсы. Список поддерживаемых типов учетных записей хранения см. в статье [использование существующей учетной записи хранения](#use-existing-storage-account). |
| деплойментскриптсторажеаккаунтнотфаунд | Учетная запись хранения не существует или удалена внешним процессом или инструментом. |
| деплойментскриптсторажеаккаунтвиссервицеендпоинтенаблед | Указанная учетная запись хранения имеет конечную точку службы. Учетная запись хранения с конечной точкой службы не поддерживается. |
| деплойментскриптсторажеаккаунтинвалидакцесскэй | Для существующей учетной записи хранения указан недопустимый ключ доступа. |
| деплойментскриптсторажеаккаунтинвалидакцесскэйформат | Недопустимый формат ключа учетной записи хранения. См. [раздел Управление ключами доступа учетной записи хранения](../../storage/common/storage-account-keys-manage.md). |
| деплойментскриптексцеедедмаксалловедтиме | Время выполнения скрипта развертывания превысило значение времени ожидания, указанное в определении ресурса скрипта развертывания. |
| деплойментскриптинвалидаутпутс | Выходные данные скрипта развертывания не являются допустимым объектом JSON. |
| деплойментскриптконтаинеринстанцессервицелогинфаилуре | Назначенному пользователем управляемому удостоверению не удалось войти в систему после 10 попыток с интервалом в 1 минуту. |
| деплойментскриптконтаинерграупнотфаунд | Группа контейнеров, созданная службой скриптов развертывания, удалена внешним инструментом или процессом. |
| деплойментскриптдовнлоадфаилуре | Не удалось скачать вспомогательный скрипт. См. раздел [Использование вспомогательного скрипта](#use-supporting-scripts).|
| деплойментскриптеррор | Пользовательский скрипт выдал ошибку. |
| деплойментскриптбутстрапскриптексекутионфаилед | Скрипт начальной загрузки выдал ошибку. Сценарий начальной загрузки — это системный сценарий, координирующий выполнение скрипта развертывания. |
| деплойментскриптексекутионфаилед | Неизвестная ошибка во время выполнения скрипта развертывания. |
| деплойментскриптконтаинеринстанцессервицеунаваилабле | При создании экземпляра контейнера Azure (ACI) ACI вызвал ошибку "Служба недоступна". |
| деплойментскриптконтаинерграупиннонтерминалстате | При создании экземпляра контейнера Azure (ACI) другой сценарий развертывания использует то же имя ACI в той же области (в той же подписке, в имени группы ресурсов и имени ресурса). |
| деплойментскриптконтаинерграупнамеинвалид | Указанное имя экземпляра контейнера Azure (ACI) не соответствует требованиям ACI. См. раздел [Устранение распространенных неполадок в службе "экземпляры контейнеров Azure](../../container-instances/container-instances-troubleshooting.md#issues-during-container-group-deployment)".|

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как использовать сценарии развертывания. Теперь вы можете изучить учебник по сценариям развертывания:

> [!div class="nextstepaction"]
> [Руководство. Использование сценариев развертывания в шаблонах Azure Resource Manager](./template-tutorial-deployment-script.md)

> [!div class="nextstepaction"]
> [Изучение модуля: расширение шаблонов ARM с помощью скриптов развертывания](/learn/modules/extend-resource-manager-template-deployment-scripts/)