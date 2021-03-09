---
title: Добавление репозитория артефактов в лабораторию в Azure DevTest Labs | Документация Майкрософт
description: Узнайте, как указать собственный репозиторий артефактов для лаборатории в Azure DevTest Labs для хранения средств, недоступных в репозитории общедоступных артефактов.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: da4e345b18a46226853d71bbf66af0487f1a761f
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102502201"
---
# <a name="add-an-artifact-repository-to-your-lab-in-devtest-labs"></a>Добавление репозитория артефактов в лабораторию в DevTest Labs
DevTest Labs позволяет указать артефакт, добавляемый в виртуальную машину во время создания виртуальной машины или после ее создания. Этот артефакт может быть средством или приложением, которое необходимо установить на виртуальной машине. Артефакты определяются в JSON-файле, загруженном из репозитория Git в GitHub или Azure DevOps.

[Общедоступный репозиторий артефактов](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts), поддерживаемый DevTest Labs, предоставляет множество стандартных средств для Windows и Linux. Ссылка на этот репозиторий автоматически добавляется в лабораторию. Вы можете создать собственный репозиторий артефактов с конкретными инструментами, которые недоступны в репозитории общедоступных артефактов. Дополнительные сведения о создании пользовательских артефактов см. в статье [Создание пользовательских артефактов](devtest-lab-artifact-author.md).

В этой статье содержатся сведения о том, как добавить пользовательский репозиторий артефактов с помощью портал Azure, шаблонов управления ресурсами Azure и Azure PowerShell. Вы можете автоматизировать Добавление репозитория артефактов в лабораторию, создав скрипты PowerShell или CLI.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования
Чтобы добавить репозиторий в лабораторию, сначала нужно получить из него основные сведения. В следующих разделах описано, как получить необходимые сведения для репозиториев, размещенных в **GitHub** или **Azure DevOps**.

### <a name="get-the-github-repository-clone-url-and-personal-access-token"></a>Получение URL-адреса клона и персонального маркера доступа из репозитория GitHub

1. Перейдите на домашнюю страницу репозитория GitHub, который содержит определения артефактов или шаблонов Resource Manager.
2. Выберите **Clone or download**(Клонировать или скачать).
3. Нажмите кнопку **HTTPS clone url** (URL-адрес клона HTTPS), чтобы скопировать URL-адрес в буфер обмена. Сохраните URL-адрес для последующего использования.
4. В правом верхнем углу GitHub выберите изображение профиля и щелкните **Параметры**.
5. В меню **Личные параметры** слева выберите **Параметры разработчика**.
6. В меню слева выберите **Личные маркеры доступа** .
7. Выберите **Создать новый маркер**.
8. На странице **New personal access token** (Новый личный маркер доступа) заполните поле **Token description** (Описание маркера). Примите элементы по умолчанию в разделе **Выберите области** и щелкните **Создать токен**.
9. Сохраните созданный маркер — он потребуется позже.
10. Закройте GitHub.   

### <a name="get-the-azure-repos-clone-url-and-personal-access-token"></a>Получение URL-адреса клона и персонального маркера доступа из Azure Repos
1. Перейдите на домашнюю страницу коллекции команд (например, `https://contoso-web-team.visualstudio.com` ), а затем выберите свой проект.
2. На домашней странице проекта выберите **Код**.
3. Чтобы увидеть URL-адрес клона, на странице **Код** проекта выберите **Клон**.
4. Сохраните URL-адрес — он потребуется позже.
5. Для создания личного маркера доступа выберите в раскрывающемся меню учетной записи пользователя **Мой профиль**.
6. На странице сведений о профиле выберите **Безопасность**.
7. На вкладке **безопасность > личные маркеры доступа** выберите **+ Новый токен**.
8. На странице **Создание личного маркера доступа** выполните следующие действия.
   1. Введите **имя** токена.
   2. В списке **Организация** выберите **все доступные Организации**.
   3. В списке **срок действия (UTC)** выберите **90 дней** или пользовательский определенный срок действия.
   4. Выберите параметр **полный доступ** для областей.
   5. Нажмите кнопку **создания**.
9. Новый маркер появится в списке **Личные маркеры доступа**. Выберите **Скопировать маркер** и сохраните значение маркера, так как оно вам потребуется позднее.
10. Перейдите к разделу Подключение лаборатории к репозиторию.

## <a name="use-azure-portal"></a>Использование портала Azure
В этом разделе приводятся шаги по добавлению репозитория артефактов в лабораторию в портал Azure.

1. Войдите на [портал Azure](https://portal.azure.com).
2. Щелкните **Больше служб**, а затем выберите в списке служб **DevTest Labs**.
3. В списке лабораторий выберите свою лабораторию.
4. В меню слева выберите **Конфигурация и политики** .
5. В меню слева выберите **репозитории** в разделе **внешние ресурсы** .
6. На панели инструментов выберите **+ Добавить** .

    ![Кнопка добавления репозитория](./media/devtest-lab-add-repo/devtestlab-add-repo.png)
5. На странице **репозитории** укажите следующие сведения.
   1. **Имя**. Введите имя репозитория.
   2. **URL-адрес клона Git.** Введите URL-адрес клона HTTPS Git, скопированный ранее из GitHub или Azure DevOps Services.
   3. **Ветвь**. Укажите ветвь для получения определений.
   4. **Личный маркер доступа**. Укажите личный маркер доступа, полученный ранее из GitHub или Azure DevOps Services.
   5. **Путь к папке.** Введите по меньшей мере один путь к папке относительно URL-адреса клона, содержащего определения артефактов или шаблонов Resource Manager. При указании подкаталога обязательно включите косую черту в путь к папке.

        ![Область "Репозитории"](./media/devtest-lab-add-repo/devtestlab-repo-blade.png)
6. Щелкните **Сохранить**.

## <a name="use-azure-resource-manager-template"></a>Использование шаблона Azure Resource Manager
Шаблоны управления ресурсами Azure (Azure Resource Manager) — это JSON-файлы, описывающие ресурсы в Azure, которые вы хотите создать. Дополнительные сведения об этих шаблонах см. в разделе [authoring Azure Resource Manager Templates](../azure-resource-manager/templates/template-syntax.md).

В этом разделе приводятся шаги по добавлению репозитория артефактов в лабораторию с помощью шаблона Azure Resource Manager.  Шаблон создает лабораторию, если он еще не существует.

### <a name="template"></a>Шаблон
Пример шаблона, используемый в этой статье, собирает следующие сведения с помощью параметров. Большинство параметров имеют значения Smart по умолчанию, но необходимо указать несколько значений. Необходимо указать имя лаборатории, URI для репозитория артефактов и маркер безопасности для репозитория.

- Имя лаборатории.
- Отображаемое имя репозитория артефактов в пользовательском интерфейсе DevTest Labs. Значение по умолчанию: `Team Repository` .
- URI репозитория (например: `https://github.com/<myteam>/<nameofrepo>.git` или `"https://MyProject1.visualstudio.com/DefaultCollection/_git/TeamArtifacts"` .
- Ветвь в репозитории, который содержит артефакты. Значение по умолчанию: `master` .
- Имя папки, содержащей артефакты. Значение по умолчанию: `/Artifacts` .
- Тип репозитория. Допустимые значения: `VsoGit` или `GitHub` .
- Маркер доступа для репозитория.

    ```json
    {

        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "labName": {
                "type": "string"
            },
            "artifactRepositoryDisplayName": {
                "type": "string",
                "defaultValue": "Team Repository"
            },
            "artifactRepoUri": {
                "type": "string"
            },
            "artifactRepoBranch": {
                "type": "string",
                "defaultValue": "master"
            },
            "artifactRepoFolder": {
                "type": "string",
                "defaultValue": "/Artifacts"
            },
            "artifactRepoType": {
                "type": "string",
                "allowedValues": ["VsoGit", "GitHub"]
            },
            "artifactRepoSecurityToken": {
                "type": "securestring"
            }
        },
        "variables": {
            "artifactRepositoryName": "[concat('Repo-', uniqueString(subscription().subscriptionId))]"
        },
        "resources": [{
                "apiVersion": "2016-05-15",
                "type": "Microsoft.DevTestLab/labs",
                "name": "[parameters('labName')]",
                "location": "[resourceGroup().location]",
                "resources": [
                    {
                        "apiVersion": "2016-05-15",
                        "name": "[variables('artifactRepositoryName')]",
                        "type": "artifactSources",
                        "dependsOn": [
                            "[resourceId('Microsoft.DevTestLab/labs', parameters('labName'))]"
                        ],
                        "properties": {
                            "uri": "[parameters('artifactRepoUri')]",
                            "folderPath": "[parameters('artifactRepoFolder')]",
                            "branchRef": "[parameters('artifactRepoBranch')]",
                            "displayName": "[parameters('artifactRepositoryDisplayName')]",
                            "securityToken": "[parameters('artifactRepoSecurityToken')]",
                            "sourceType": "[parameters('artifactRepoType')]",
                            "status": "Enabled"
                        }
                    }
                ]
            }
        ]
    }
    ```


### <a name="deploy-the-template"></a>Развертывание шаблона
Существует несколько способов развертывания шаблона в Azure и создания ресурса, если он не существует или не обновлен, если он существует. Подробности см. в следующих статьях:

- [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
- [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../azure-resource-manager/templates/deploy-cli.md)
- [Развертывание ресурсов с использованием шаблонов Resource Manager и портала Azure](../azure-resource-manager/templates/deploy-portal.md)
- [Развертывание ресурсов с помощью шаблонов диспетчер ресурсов и диспетчер ресурсов REST API](../azure-resource-manager/templates/deploy-rest.md)

Давайте посмотрим, как развернуть шаблон в PowerShell. Командлеты, используемые для развертывания шаблона, являются контекстно-зависимыми, поэтому используются текущий клиент и текущая подписка. Используйте [Set-азконтекст](/powershell/module/az.accounts/set-azcontext) перед развертыванием шаблона (при необходимости), чтобы изменить контекст.

Сначала создайте группу ресурсов с помощью команды [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup). Если группа ресурсов, которую вы хотите использовать, уже существует, пропустите этот шаг.

```powershell
New-AzResourceGroup -Name MyLabResourceGroup1 -Location westus
```

Затем создайте развертывание в группе ресурсов с помощью команды [New-азресаурцеграупдеплоймент](/powershell/module/az.resources/new-azresourcegroupdeployment). Этот командлет применяет изменения ресурсов к Azure. Можно выполнить несколько развертываний ресурсов для любой определенной группы ресурсов. Если вы развертываете несколько раз в одной группе ресурсов, убедитесь, что имя каждого развертывания уникально.

```powershell
New-AzResourceGroupDeployment `
    -Name MyLabResourceGroup-Deployment1 `
    -ResourceGroupName MyLabResourceGroup1 `
    -TemplateFile azuredeploy.json `
    -TemplateParameterFile azuredeploy.parameters.json
```

После успешного выполнения New-AzResourceGroupDeployment команда выводит важную информацию, например состояние подготовки (должно быть выполнено успешно), и все выходные данные шаблона.

## <a name="use-azure-powershell"></a>Использование Azure PowerShell
В этом разделе приведен пример скрипта PowerShell, который можно использовать для добавления репозитория артефактов в лабораторию. Если у вас нет Azure PowerShell, см. Дополнительные инструкции по установке [и настройке Azure PowerShell](/powershell/azure/) .

### <a name="full-script"></a>Полный сценарий
Ниже приведен полный сценарий, в том числе некоторые подробные сообщения и комментарии.

**New-DevTestLabArtifactRepository.ps1**:

```powershell

<#

.SYNOPSIS
This script creates a new custom repository and adds it to an existing DevTest Lab.

.PARAMETER LabName
The name of the lab.

.PARAMETER LabResourceGroupName
The name of the resource group that contains the lab.

.PARAMETER ArtifactRepositoryName
Name for the new artifact repository.
Script creates a random name for the repository if it is not specified.

.PARAMETER ArtifactRepositoryDisplayName
Display name for the artifact repository.
This is the name that shows up in the Azure portal (https://portal.azure.com) when viewing all the artifact repositories for a lab.

.PARAMETER RepositoryUri
Uri to the repository.

.PARAMETER RepositoryBranch
Branch in which artifact files can be found. Defaults to 'master'.

.PARAMETER FolderPath
Folder under which artifacts can be found. Defaults to '/Artifacts'

.PARAMETER PersonalAccessToken
Security token for access to GitHub or VSOGit repository.
See https://azure.microsoft.com/documentation/articles/devtest-lab-add-artifact-repo/ for instructions to get personal access token

.PARAMETER SourceType
Whether artifact is VSOGit or GitHub repository.

.EXAMPLE
Set-AzContext -SubscriptionId 11111111-1111-1111-1111-111111111111
.\New-DevTestLabArtifactRepository.ps1 -LabName "mydevtestlab" -LabResourceGroupName "mydtlrg" -ArtifactRepositoryName "MyTeam Repository" -RepositoryUri "https://github.com/<myteam>/<nameofrepo>.git" -PersonalAccessToken "1111...." -SourceType "GitHub"

.NOTES
Script uses the current Az context. To set the context, use the Set-AzContext cmdlet

#>


[CmdletBinding()]
Param(

    [Parameter(Mandatory=$true)]
    $LabName,

    [Parameter(Mandatory=$true)]
    $LabResourceGroupName,
    $ArtifactRepositoryName,
    $ArtifactRepositoryDisplayName  = 'Team Artifact Repository',

    [Parameter(Mandatory=$true)]
    $RepositoryUri,
    $RepositoryBranch = 'master',
    $FolderPath = '/Artifacts',

    [Parameter(Mandatory=$true)]
    $PersonalAccessToken ,

    [Parameter(Mandatory=$true)]
    [ValidateSet('VsoGit', 'GitHub')]
    $SourceType
)


#Set artifact repository internal name,
# if not set by user.

if ($ArtifactRepositoryName -eq $null){
    $ArtifactRepositoryName = "PrivateRepo" + (Get-Random -Maximum 999)
}

# Sign in to Azure
Connect-AzAccount


#Get Lab Resource
$LabResource = Get-AzResource -ResourceType 'Microsoft.DevTestLab/labs' -ResourceName $LabName -ResourceGroupName $LabResourceGroupName

Write-Verbose "Lab Name: $($LabResource.Name)"
Write-Verbose "Lab Resource Group Name: $($LabResource.ResourceGroupName)"
Write-Verbose "Lab Resource Location: $($LabResource.Location)"

Write-Verbose "Artifact Repository Internal Name: $ArtifactRepositoryName"

#Prepare properties object for call to New-AzResource
$propertiesObject = @{
    uri = $RepositoryUri;
    folderPath = $FolderPath;
    branchRef = $RepositoryBranch;
    displayName = $ArtifactRepositoryDisplayName;
    securityToken = $PersonalAccessToken;
    sourceType = $SourceType;
    status = 'Enabled'
}

Write-Verbose @"Properties to be passed to New-AzResource:$($propertiesObject | Out-String)"

#Resource will be added to current subscription.
$resourcetype = 'Microsoft.DevTestLab/labs/artifactSources'
$resourceName = $LabName + '/' + $ArtifactRepositoryName
Write-Verbose "Az ResourceType: $resourcetype"
Write-Verbose "Az ResourceName: $resourceName"

Write-Verbose "Creating artifact repository '$ArtifactRepositoryDisplayName'..."
$result = New-AzResource -Location $LabResource.Location -ResourceGroupName $LabResource.ResourceGroupName -properties $propertiesObject -ResourceType $resourcetype -ResourceName $resourceName -ApiVersion 2016-05-15 -Force


#Alternate implementation:
# Use resourceId rather than resourcetype and resourcename parameters.
# Using resourceId allows you to specify the $SubscriptionId rather than using the
# subscription id of Get-AzContext.
#$resourceId = "/subscriptions/$SubscriptionId/resourceGroups/$($LabResource.ResourceGroupName)/providers/Microsoft.DevTestLab/labs/$LabName/artifactSources/$ArtifactRepositoryName"
#$result = New-AzResource -properties $propertiesObject -ResourceId $resourceId -ApiVersion 2016-05-15 -Force


# Check the result
if ($result.Properties.ProvisioningState -eq "Succeeded") {
    Write-Verbose ("Successfully added artifact repository source '$ArtifactRepositoryDisplayName'")
}
else {
    Write-Error ("Error adding artifact repository source '$ArtifactRepositoryDisplayName'")
}

#Return the newly created resource so it may be used in subsequent scripts
return $result
```

### <a name="run-the-powershell-script"></a>Запуск скрипта PowerShell
В следующем примере показано, как выполнить скрипт:

```powershell
Set-AzContext -SubscriptionId <Your Azure subscription ID>

.\New-DevTestLabArtifactRepository.ps1 -LabName "mydevtestlab" -LabResourceGroupName "mydtlrg" -ArtifactRepositoryName "MyTeam Repository" -RepositoryUri "https://github.com/<myteam>/<nameofrepo>.git" -PersonalAccessToken "1111...." -SourceType "GitHub"
```


### <a name="parameters"></a>Параметры
Пример сценария PowerShell в этой статье принимает следующие параметры:

| Параметр | Описание |
| --------- | ----------- |
| лабнаме | Имя лаборатории. |
| артифактрепоситоринаме | Имя для нового репозитория артефактов. Скрипт создает случайное имя для репозитория, если оно не указано. |
| артифактрепоситоридисплайнаме | Отображаемое имя репозитория артефактов. Это имя, которое отображается в портал Azure ( https://portal.azure.com) при просмотре всех репозиториев артефактов для лаборатории). |
| репоситорюри | URI репозитория. Примеры: `https://github.com/<myteam>/<nameofrepo>.git` или `"https://MyProject1.visualstudio.com/DefaultCollection/_git/TeamArtifacts"` .|
| RepositoryBranch | Ветвь, в которой можно найти файлы артефактов. По умолчанию используется значение master. |
| FolderPath | Папка, в которой можно найти артефакты. По умолчанию используется значение "/Артифактс" |
| персоналакцесстокен | Маркер безопасности для доступа к репозиторию GitHub или Всогит. Инструкции по получению личного маркера доступа см. в разделе Предварительные требования. |
| Тип источника | Является ли артефакт Всогит или репозиторием GitHub. |

Репозиторию требуется внутреннее имя для идентификации, которое отличается от отображаемого имени, которое отображается в портал Azure. Вы не видите внутреннее имя с помощью портал Azure, но видите его при использовании интерфейсов API службы Azure или Azure PowerShell. Сценарий предоставляет имя, если оно не указано пользователем нашего скрипта.

```powershell
#Set artifact repository name, if not set by user
if ($ArtifactRepositoryName -eq $null){
    $ArtifactRepositoryName = "PrivateRepo" + (Get-Random -Maximum 999)
}
```

### <a name="powershell-commands-used-in-the-script"></a>Команды PowerShell, используемые в скрипте

| Команда PowerShell | Примечания |
| ------------------ | ----- |
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Эта команда используется для получения сведений о лаборатории, например о ее расположении. |
| [New-AzResource](/powershell/module/az.resources/new-azresource) | Нет специальной команды для добавления репозиториев артефактов. Универсальный командлет [New-азресаурце](/powershell/module/az.resources/new-azresource) выполняет задание. Для этого командлета требуется пара **ResourceId** или **resourceName** и **ResourceType** , чтобы получить сведения о типе создаваемого ресурса. В этом примере сценария используется имя ресурса и пара типа ресурса. <br/><br/>Обратите внимание, что вы создаете источник репозитория артефактов в том же расположении и в той же группе ресурсов, что и лаборатория.|

Скрипт добавляет новый ресурс в текущую подписку. Чтобы просмотреть эти сведения, используйте [Get-азконтекст](/powershell/module/az.accounts/get-azcontext) . Используйте [Set-азконтекст](/powershell/module/az.accounts/set-azcontext) , чтобы задать текущий клиент и подписку.

Чтобы узнать имя ресурса и сведения о типе ресурса, лучше всего использовать веб-сайт [Azure API для тестового диска](https://azure.github.io/projects/apis/) . Обратитесь к поставщику [DevTest Labs — 2016-05-15](https://aka.ms/dtlrestapis) , чтобы просмотреть доступные интерфейсы API RESTful для поставщика DevTest Labs. В скрипте пользователи заменяют следующий идентификатор ресурса.

```powershell
"/subscriptions/$SubscriptionId/resourceGroups/$($LabResource.ResourceGroupName)/providers/Microsoft.DevTestLab/labs/$LabName/artifactSources/$ArtifactRepositoryName"
```

Тип ресурса — это все, что указывается после "providers" в URI, за исключением элементов, перечисленных в фигурных скобках. Имя ресурса — это все, что отображается в фигурных скобках. Если для имени ресурса ожидается более одного элемента, отделите каждый элемент косой чертой по мере того, как мы сделали это.

```powershell
$resourcetype = 'Microsoft.DevTestLab/labs/artifactSources'
$resourceName = $LabName + '/' + $ArtifactRepositoryName
```


## <a name="next-steps"></a>Дальнейшие действия
- [Укажите обязательные артефакты для лаборатории в Azure DevTest Labs](devtest-lab-mandatory-artifacts.md)
- [Создание пользовательских артефактов для виртуальной машины DevTest Labs](devtest-lab-artifact-author.md)
- [Диагностика сбоев артефактов в лаборатории](devtest-lab-troubleshoot-artifact-failure.md)
