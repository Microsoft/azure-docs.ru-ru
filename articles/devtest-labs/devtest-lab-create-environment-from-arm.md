---
title: Создание сред с несколькими виртуальными машинами и ресурсов PaaS с помощью шаблонов
description: Узнайте, как в Azure DevTest Labs создавать среды со множеством виртуальных машин и ресурсов PaaS, используя шаблон Azure Resource Manager.
ms.topic: article
ms.date: 08/12/2020
ms.openlocfilehash: 97659d4ab95fdbe75460161d0ceed71a1cb5cf82
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88182414"
---
# <a name="create-multi-vm-environments-and-paas-resources-with-azure-resource-manager-templates"></a>Создание сред со множеством виртуальных машин и ресурсов PaaS с помощью шаблонов Azure Resource Manager

Azure DevTest Labsные среды позволяют пользователям легко развертывать сложные инфраструктуры в пределах лаборатории. [Шаблоны Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md) можно использовать для создания сред с наборами ресурсов в DevTest Labs. Эти среды могут содержать любые ресурсы Azure, которые могут быть созданы диспетчер ресурсов шаблонами.

Вы можете легко [Добавить в лабораторию одну виртуальную машину (ВМ)](devtest-lab-add-vm.md) с помощью [портал Azure](https://portal.azure.com). Однако в таких сценариях, как многоуровневые веб-приложения или ферма SharePoint, необходим механизм создания нескольких виртуальных машин за один шаг. С помощью шаблонов Azure Resource Manager можно определить инфраструктуру и конфигурацию решения Azure и многократно развернуть несколько виртуальных машин в стабильном состоянии.

Azure Resource Manager шаблоны также предоставляют следующие преимущества:

- Шаблоны Azure Resource Manager загружаются непосредственно из репозитория в репозитории GitHub или Azure Repos системы управления версиями.
- Пользователи могут создать среду, выбрав настроенный шаблон Azure Resource Manager из портал Azure так же, как и для других типов [баз виртуальных машин](devtest-lab-comparing-vm-base-image-types.md).
- Вы можете подготавливать ресурсы PaaS Azure, а также виртуальные машины IaaS в среде из шаблона Azure Resource Manager.
- Затраты на среды в лаборатории можно отчислить в дополнение к отдельным виртуальным машинам, созданным другими типами баз. Ресурсы PaaS создаются и отображаются в отслеживании затрат. Однако автоматическое выключение виртуальной машины не применяется к ресурсам PaaS.

Дополнительные сведения о преимуществах использования шаблонов диспетчер ресурсов для развертывания, обновления или удаления нескольких лабораторных ресурсов за одну операцию см. в статье [преимущества использования шаблонов диспетчер ресурсов](../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager).

> [!NOTE]
> При использовании шаблона диспетчер ресурсов в качестве основы для создания виртуальных машин лаборатории существуют некоторые различия между созданием нескольких виртуальных машин или одной виртуальной машины. Дополнительные сведения см. [в статье Использование шаблона Azure Resource Manager виртуальной машины](devtest-lab-use-resource-manager-template.md).
>

## <a name="use-devtest-labs-public-environments"></a>Использование общедоступных сред DevTest Labs
Azure DevTest Labs имеет [общедоступный репозиторий шаблонов Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/Environments) , которые можно использовать для создания сред без необходимости самостоятельного подключения к внешнему источнику GitHub. Этот общедоступный репозиторий аналогичен общедоступному репозиторию артефактов, доступных в портал Azure для каждой создаваемой лабораторной работы. Репозиторий среды позволяет быстро начать работу с предварительно созданными шаблонами среды с небольшими входными параметрами. Эти шаблоны предоставляют возможности плавного запуска для ресурсов PaaS в лабораториях.

В общедоступном репозитории группа DevTest Labs и другие имеют доступ к часто используемым шаблонам, таким как веб-приложения Azure, Service Fabric кластеру и среде разработки фермы SharePoint. Вы можете использовать эти шаблоны непосредственно или настроить их в соответствии со своими потребностями. Дополнительные сведения см. в статье [Настройка и использование общедоступных сред в Azure DevTest Labs](devtest-lab-configure-use-public-environments.md). Создав собственные шаблоны, вы можете сохранить их в этом репозитории, чтобы поделиться ими с другими пользователями, или настроить собственный репозиторий Git.

<a name="configure-your-own-template-repositories"></a>
## <a name="create-your-own-template-repositories"></a>Создание собственных репозиториев шаблонов

В качестве одного из рекомендаций по использованию инфраструктуры как кода и конфигурации как кода следует управлять шаблонами среды в системе управления версиями. Azure DevTest Labs следует на этом практике и загрузить все шаблоны Azure Resource Manager непосредственно из репозиториев GitHub или Azure Repos. В результате можно использовать шаблоны диспетчер ресурсов во всем цикле выпуска, от тестовой среды до рабочей среды.

Существует несколько правил для упорядочения шаблонов Azure Resource Manager в репозитории:

- Необходимо присвоить файлу главного шаблона *azuredeploy.jsзначение on*.

- Если вы хотите использовать значения параметров, определенные в файле параметров, файл параметров должен иметь имя *azuredeploy.parameters.jsв*.

  Для создания значения URI parametersLink можно использовать параметры `_artifactsLocation` и `_artifactsLocationSasToken`, чтобы служба DevTest Labs автоматически управляла вложенными шаблонами. Дополнительные сведения см. в разделе [Развертывание вложенных шаблонов Azure Resource Manager для тестовых сред](deploy-nested-template-environments.md).

- Можно определить метаданные, чтобы указать отображаемое имя шаблона и описание в файле с именем *metadata.js*, как показано ниже.

  ```json
  {
    "itemDisplayName": "<your template name>",
    "description": "<description of the template>"
  }
  ```

![Основные файлы шаблонов Azure Resource Manager](./media/devtest-lab-create-environment-from-arm/master-template.png)

## <a name="add-template-repositories-to-the-lab"></a>Добавление репозиториев шаблонов в лабораторию

После создания и настройки репозитория его можно добавить в лабораторию с помощью портал Azure:

1. Войдите на [портал Azure](https://portal.azure.com).
1. Щелкните **Все службы** и выберите в списке **DevTest Labs**.
1. В списке лабораторий выберите нужную лабораторию.
1. В области **Обзор** лаборатории выберите **Конфигурация и политики**.

   ![Configuration and Policies (Конфигурация и политики)](./media/devtest-lab-create-environment-from-arm/configuration-and-policies-menu.png)

1. В списке параметры **конфигурации и политики** выберите **репозитории**. Репозиторий **общедоступного репозитория артефактов** создается автоматически для всех лабораторий и подключается к [общедоступному репозиторию GitHub DevTest Labs](https://github.com/Azure/azure-devtestlab).

1. Чтобы добавить репозиторий шаблонов Azure Resource Manager, нажмите кнопку **Добавить**.

   ![Общедоступный репозиторий](./media/devtest-lab-create-environment-from-arm/public-repo.png)

1. В области **репозитории** введите следующие сведения.

   - **Имя**: введите имя репозитория для использования в лаборатории.
   - **URL-адрес клона git**: введите URL-адрес клона HTTPS Git из GitHub или Azure Repos.
   - **Branch** (необязательно): введите имя ветви, чтобы получить доступ к определениям шаблона Azure Resource Manager.
   - **Личный маркер доступа**. Введите личный маркер доступа, который используется для безопасного доступа к репозиторию.
     - Чтобы получить маркер из Azure Repos, в своем профиле выберите **Параметры пользователя**  >  **Безопасность**  >  **Личные маркеры доступа**.
     - Чтобы получить токен из GitHub, в разделе Профиль выберите **Параметры**  >  **разработчика параметры**  >  **личные права доступа**.
   - **Пути к папкам**. Введите путь к папке относительно URI клона Git для определений артефактов или определений шаблонов Azure Resource Manager.

1. Щелкните **Сохранить**.

   ![Добавить новый репозиторий](./media/devtest-lab-create-environment-from-arm/repo-values.png)

После добавления шаблона Azure Resource Manager в лабораторию пользователи лаборатории могут создавать среды с помощью шаблона.

## <a name="configure-access-rights-for-lab-users"></a>Настройка прав доступа для пользователей лаборатории

Пользователи лаборатории имеют роль **читателя** по умолчанию, поэтому они не могут изменять ресурсы в группе ресурсов среды. Например, они не могут останавливаться или запускать свои ресурсы.

Чтобы предоставить пользователям рабочей среды роль **участника** , чтобы они могли изменять ресурсы в своих средах, выполните следующие действия.

1. В [портал Azure](https://portal.azure.com)в области **обзора** лаборатории выберите **Конфигурация и политики**, а затем выберите **Параметры лаборатории**.

1. В области **Параметры лаборатории** выберите **участник** и нажмите кнопку **сохранить** , чтобы предоставить пользователям лаборатории разрешения на запись.

   ![Настройка прав доступа пользователей лаборатории](./media/devtest-lab-create-environment-from-arm/config-access-rights.png)

В следующем разделе описано создание сред на основе шаблона Azure Resource Manager.

## <a name="create-environments-from-templates-in-the-azure-portal"></a>Создание сред на основе шаблонов в портал Azure

После добавления шаблона Azure Resource Manager в лабораторию пользователи лаборатории могут создавать среды в портал Azure, выполнив следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).

1. Щелкните **Все службы** и выберите в списке **DevTest Labs**.

1. В списке лабораторий выберите нужную лабораторию.

1. На странице лаборатории нажмите кнопку **Добавить**.

1. В области **Выбор базы** отображаются базовые образы, которые можно использовать, с шаблонами Azure Resource Manager, указанными в первую очередь. Выберите нужный шаблон Azure Resource Manager.

   ![Выбор основы](./media/devtest-lab-create-environment-from-arm/choose-a-base.png)

1. В области **Добавить** введите значение **имени среды** , которое будет отображаться для пользователей среды.

   Шаблон Azure Resource Manager определяет остальные поля ввода. Если шаблон *azuredeploy.parameter.jsв* файле определяет значения по умолчанию, поля ввода показывают эти значения.

   Для параметров типа " *защищенная строка*" можно использовать секреты из Azure Key Vault. Сведения о хранении секретов в хранилище ключей и их использовании при создании лабораторных ресурсов см. [в статье хранение секретов в Azure Key Vault](devtest-lab-store-secrets-in-key-vault.md).  

   ![Добавление области](./media/devtest-lab-create-environment-from-arm/add.png)

   > [!NOTE]
   > Следующие значения параметров не отображаются в полях ввода, даже если они указаны в шаблоне. Вместо этого в форме отображаются пустые поля ввода, где пользователи лаборатории должны вводить значения при создании среды.
   >
   > - GEN-UNIQUE
   > - GEN-UNIQUE-[N]
   > - GEN-SSH-PUB-KEY
   > - GEN-PASSWORD

1. Нажмите кнопку **Добавить**, чтобы создать среду.

   Среда начнет подготовку немедленно и отобразит состояние в списке **мои виртуальные машины** . Лаборатория автоматически создает новую группу ресурсов для инициализации всех ресурсов, определенных в шаблоне Azure Resource Manager.

1. После создания среды выберите окружение в списке **мои виртуальные машины** , чтобы открыть панель группа ресурсов и просмотреть все ресурсы, подготовленные средой.

   ![Ресурсы среды](./media/devtest-lab-create-environment-from-arm/all-environment-resources.png)

   Можно также развернуть среду, чтобы просмотреть только список виртуальных машин, подготовленных средой.

   ![Список "My virtual machines" (Мои виртуальные машины)](./media/devtest-lab-create-environment-from-arm/my-vm-list.png)

1. Выберите любую из сред, чтобы просмотреть доступные действия, такие как применение артефактов, присоединение дисков данных, изменение времени автоматического завершения работы и многое другое.

   ![Действия среды](./media/devtest-lab-create-environment-from-arm/environment-actions.png)

<a name="automate-deployment-of-environments"></a>
## <a name="automate-environment-creation-with-powershell"></a>Автоматизация создания среды с помощью PowerShell

Можно использовать портал Azure для добавления одной среды в лабораторию, но когда сценарий разработки или тестирования должен создать несколько сред, автоматическое развертывание будет более эффективным.

Прежде чем продолжать, убедитесь, что у вас есть шаблон Azure Resource Manager, определяющий создаваемые ресурсы. [Добавьте и настройте шаблон в репозитории Git](#configure-your-own-template-repositories), а затем [добавьте репозиторий в лабораторию](#add-template-repositories-to-the-lab).

Следующий пример скрипта создает среду в лаборатории. Комментарии помогут лучше понять принципы работы сценария.

1. Сохраните следующий пример сценария PowerShell на жестком диске как *deployenv.ps1*.

   [!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

   ```powershell
   #Requires -Module Az.Resources

   [CmdletBinding()]

   param (
   # ID of the Azure Subscription for the lab
   [string] [Parameter(Mandatory=$true)] $SubscriptionId,

   # Name of the existing lab in which to create the environment
   [string] [Parameter(Mandatory=$true)] $LabName,

   # Name of the connected repository in the lab
   [string] [Parameter(Mandatory=$true)] $RepositoryName,

   # Name of the template (folder name in the Git repository)
   [string] [Parameter(Mandatory=$true)] $TemplateName,

   # Name of the environment to be created in the lab
   [string] [Parameter(Mandatory=$true)] $EnvironmentName,

   # The parameters to be passed to the template. Each parameter is prefixed with "-param_".
   # For example, if the template has a parameter named "TestVMName" with a value of "MyVMName",
   # the string in $Params will have the form: -param_TestVMName MyVMName.
   # This convention allows the script to dynamically handle different templates.
   [Parameter(ValueFromRemainingArguments=$true)]
       $Params
   )

   # Sign in to Azure.
   # Comment out the following statement to completely automate the environment creation.
   Connect-AzAccount

   # Select the subscription that has the lab.  
   Set-AzContext -SubscriptionId $SubscriptionId | Out-Null

   # Get information about the user, specifically the user ID, which is used later in the script.  
   $UserId = $((Get-AzADUser -UserPrincipalName ((Get-AzContext).Account).Id).Id)

   # Get information about the lab, such as lab location.
   $lab = Get-AzResource -ResourceType "Microsoft.DevTestLab/labs" -Name $LabName
   if ($lab -eq $null) { throw "Unable to find lab $LabName in subscription $SubscriptionId." }

   # Get information about the repository in the lab.
   $repository = Get-AzResource -ResourceGroupName $lab.ResourceGroupName `
       -ResourceType 'Microsoft.DevTestLab/labs/artifactsources' `
       -ResourceName $LabName `
       -ApiVersion 2016-05-15 `
       | Where-Object { $RepositoryName -in ($_.Name, $_.Properties.displayName) } `
       | Select-Object -First 1
   if ($repository -eq $null) { throw "Unable to find repository $RepositoryName in lab $LabName." }

   # Get information about the Resource Manager template base for the environment.
   $template = Get-AzResource -ResourceGroupName $lab.ResourceGroupName `
       -ResourceType "Microsoft.DevTestLab/labs/artifactSources/armTemplates" `
       -ResourceName "$LabName/$($repository.Name)" `
       -ApiVersion 2016-05-15 `
       | Where-Object { $TemplateName -in ($_.Name, $_.Properties.displayName) } `
       | Select-Object -First 1
   if ($template -eq $null) { throw "Unable to find template $TemplateName in lab $LabName." }

   # Build the template parameters with parameter name and values.  
   $parameters = Get-Member -InputObject $template.Properties.contents.parameters -MemberType NoteProperty | Select-Object -ExpandProperty Name
   $templateParameters = @()

   # Extract the custom parameters from $Params and format as name/value pairs.
   $Params | ForEach-Object {
       if ($_ -match '^-param_(.*)' -and $Matches[1] -in $parameters) {
           $name = $Matches[1]                
       } elseif ( $name ) {
           $templateParameters += @{ "name" = "$name"; "value" = "$_" }
           $name = $null #reset name variable
       }
   }

   # Once name/value pairs are isolated, create an object to hold the necessary template properties.
   $templateProperties = @{ "deploymentProperties" = @{ "armTemplateId" = "$($template.ResourceId)"; "parameters" = $templateParameters }; }

   # Now, create or deploy the environment in the lab by using the New-AzResource command.
   New-AzResource -Location $Lab.Location `
       -ResourceGroupName $lab.ResourceGroupName `
       -Properties $templateProperties `
       -ResourceType 'Microsoft.DevTestLab/labs/users/environments' `
       -ResourceName "$LabName/$UserId/$EnvironmentName" `
       -ApiVersion '2016-05-15' -Force

   Write-Output "Environment $EnvironmentName completed."
   ```

1. Выполните сценарий следующим образом, используя ваши значения для SubscriptionId, Лабнаме, ResourceGroupName, RepositoryName, TemplateName (папка в репозитории Git) и EnvironmentName.

   ```powershell
   ./deployenv.ps1 -SubscriptionId "000000000-0000-0000-0000-0000000000000" -LabName "mydevtestlab" -ResourceGroupName "mydevtestlabRG000000" -RepositoryName "myRepository" -TemplateName "My Environment template name" -EnvironmentName "myGroupEnv"
   ```

Можно также использовать Azure CLI для развертывания ресурсов с помощью шаблонов диспетчер ресурсов. Дополнительные сведения см. в статье [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../azure-resource-manager/templates/deploy-cli.md).

> [!NOTE]
> Только пользователь с правами владельца лаборатории может создать виртуальные машины из шаблона Resource Manager с помощью Azure PowerShell. Если вы хотите автоматизировать создание виртуальной машины с помощью шаблона диспетчер ресурсов и у вас есть только разрешения пользователя, можно использовать команду CLI [AZ Lab VM Create](/cli/azure/lab/vm#az-lab-vm-create).

## <a name="resource-manager-template-limitations-in-devtest-labs"></a>Ограничения шаблонов диспетчер ресурсов в DevTest Labs

Учитывайте эти ограничения при использовании шаблонов диспетчер ресурсов в DevTest Labs:

- Нельзя создавать формулы или пользовательские образы из виртуальных машин лаборатории, созданных на основе шаблона диспетчер ресурсов.

- Большинство политик не оцениваются при развертывании шаблонов диспетчер ресурсов.

Например, у вас может быть политика лаборатории, с помощью которой пользователь может создавать только пять виртуальных машин. Тем не менее пользователь может развернуть шаблон Resource Manager, который создает десятки виртуальных машин. Ниже перечислены политики, которые не оцениваются.

  - число виртуальных машин на пользователя;

  - число виртуальных машин уровня Premium на пользователя лаборатории;

  - число дисков уровня Premium на пользователя лаборатории.

## <a name="next-steps"></a>Дальнейшие действия
- После создания виртуальной машины можно подключиться к ней, нажав кнопку **подключить** на панели управления виртуальной машины.
- Для просмотра и администрирования ресурсов в среде выберите среду в списке **Мои виртуальные машины** в вашей лаборатории.
- Изучите [шаблоны Azure Resource Manager из коллекции шаблонов](https://github.com/Azure/azure-quickstart-templates)быстрого запуска Azure.
