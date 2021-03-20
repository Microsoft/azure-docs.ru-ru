---
title: Создание определений политик гостевой конфигурации из базового плана групповая политика для Windows
description: Узнайте, как преобразовать групповая политика из базового плана безопасности Windows Server 2019 в определение политики.
ms.date: 08/17/2020
ms.topic: how-to
ms.openlocfilehash: 7f7e2af70efa6771d94d7ceaa14d1408175b1d12
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93348650"
---
# <a name="how-to-create-guest-configuration-policy-definitions-from-group-policy-baseline-for-windows"></a>Создание определений политик гостевой конфигурации из базового плана групповая политика для Windows

Перед созданием определений настраиваемой политики рекомендуется ознакомиться с концептуальными сведениями в [конфигурации гостя политики Azure](../concepts/guest-configuration.md). Дополнительные сведения о создании определений пользовательских политик гостевой конфигурации для Linux см. в статье [Создание политик конфигурации гостей для Linux](./guest-configuration-create-linux.md). Дополнительные сведения о создании определений настраиваемых политик гостевой конфигурации для Windows см. в статье [Создание политик конфигурации гостей для Windows](./guest-configuration-create.md).

При аудите системы Windows гостевая конфигурация применяет модуль ресурсов [Desired State Configuration (DSC)](/powershell/scripting/dsc/overview/overview) для создания файла конфигурации. Эта конфигурация DSC определяет состояние, в котором должен находиться компьютер. Если оценка конфигурации не **соответствует требованиям**, активируется действие политики *помощью параметров auditifnotexists* .
[Гостевая Конфигурация политики Azure](../concepts/guest-configuration.md) выполняет аудит только параметров на виртуальных машинах.

> [!IMPORTANT]
> Для аудита виртуальных машин Azure требуется расширение гостевой конфигурации. Чтобы развернуть расширение в большом масштабе на всех компьютерах Windows, назначьте приведенные ниже определения политик.
> - [Развертывание необходимых компонентов для политики гостевой конфигурации на виртуальных машинах Windows](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0ecd903d-91e7-4726-83d3-a229d7f2e293)
> 
> Не используйте секреты или конфиденциальные сведения в пакетах с нестандартным содержимым.

Сообщество DSC опубликовало [модуль баселинеманажемент](https://github.com/microsoft/BaselineManagement) для преобразования экспортированных шаблонов групповая политика в формат DSC. Вместе с командлетом Гуестконфигуратион модуль Баселинеманажемент создает гостевой пакет конфигурации политики Azure для Windows из содержимого групповая политика. Дополнительные сведения об использовании модуля Баселинеманажемент см. в статье [Краткое руководство. преобразование групповая политика в DSC](/powershell/scripting/dsc/quickstarts/gpo-quickstart).

В этом руководстве мы рассмотрим процесс создания пакета гостевой конфигурации политики Azure на основе объекта групповая политика (GPO). Хотя в этом пошаговом руководстве рассматривается преобразование базовых показателей безопасности Windows Server 2019, один и тот же процесс можно применить к другим объектам групповой политики.  

## <a name="download-windows-server-2019-security-baseline-and-install-related-powershell-modules"></a>Загрузите базовый план безопасности Windows Server 2019 и установите связанные модули PowerShell.

Чтобы установить **DSC**, **гуестконфигуратион**, **базовое управление** и связанные модули Azure в PowerShell:

1. В командной строке PowerShell выполните следующую команду:

   ```azurepowershell-interactive
   # Install the BaselineManagement module, Guest Configuration DSC resource module, and relevant Azure modules from PowerShell Gallery
   Install-Module az.resources, az.policyinsights, az.storage, guestconfiguration, gpregistrypolicyparser, securitypolicydsc, auditpolicydsc, baselinemanagement -scope currentuser -Repository psgallery -AllowClobber
   ```

1. Создайте каталог для и скачайте базовый уровень безопасности Windows Server 2019 с помощью набора средств обеспечения соответствия требованиям безопасности для Windows.

   ```azurepowershell-interactive
   # Download the 2019 Baseline files from https://docs.microsoft.com/windows/security/threat-protection/security-compliance-toolkit-10
   New-Item -Path 'C:\git\policyfiles\downloads' -Type Directory
   Invoke-WebRequest -Uri 'https://download.microsoft.com/download/8/5/C/85C25433-A1B0-4FFA-9429-7E023E7DA8D8/Windows%2010%20Version%201909%20and%20Windows%20Server%20Version%201909%20Security%20Baseline.zip' -Out C:\git\policyfiles\downloads\Server2019Baseline.zip
   ```

1. Разблокируйте и разверните загруженный базовый план сервера 2019.

   ```azurepowershell-interactive
   Unblock-File C:\git\policyfiles\downloads\Server2019Baseline.zip
   Expand-Archive -Path C:\git\policyfiles\downloads\Server2019Baseline.zip -DestinationPath C:\git\policyfiles\downloads\
   ```

1. Проверка содержимого базового сервера 2019 с помощью **MapGuidsToGpoNames.ps1**.

   ```azurepowershell-interactive
   # Show content details of downloaded GPOs
   C:\git\policyfiles\downloads\Scripts\Tools\MapGuidsToGpoNames.ps1 -rootdir C:\git\policyfiles\downloads\GPOs\ -Verbose
   ```

## <a name="convert-from-group-policy-to-azure-policy-guest-configuration"></a>Преобразование из групповая политика в гостевую конфигурацию политики Azure

Далее мы преобразуем загруженный базовый план сервера 2019 в пакет конфигурации гостя с помощью модуля настройки гостевой системы и базовых модулей управления.

1. Преобразуйте групповая политика в конфигурацию требуемого состояния с помощью модуля управления базовыми показателями.

   ```azurepowershell-interactive
   ConvertFrom-GPO -Path 'C:\git\policyfiles\downloads\GPOs\{3657C7A2-3FF3-4C21-9439-8FDF549F1D68}\' -OutputPath 'C:\git\policyfiles\' -OutputConfigurationScript -Verbose
   ```

1. Переименование, переформатирование и выполнение преобразованных скриптов перед созданием пакета содержимого политики.

   ```azurepowershell-interactive
   Rename-Item -Path C:\git\policyfiles\DSCFromGPO.ps1 -NewName C:\git\policyfiles\Server2019Baseline.ps1
   (Get-Content -Path C:\git\policyfiles\Server2019Baseline.ps1).Replace('DSCFromGPO', 'Server2019Baseline') | Set-Content -Path C:\git\policyfiles\Server2019Baseline.ps1
   (Get-Content -Path C:\git\policyfiles\Server2019Baseline.ps1).Replace('PSDesiredStateConfiguration', 'PSDscResources') | Set-Content -Path C:\git\policyfiles\Server2019Baseline.ps1
   C:\git\policyfiles\Server2019Baseline.ps1
   ```

1. Создайте пакет содержимого гостевой конфигурации политики Azure.

   ```azurepowershell-interactive
   New-GuestConfigurationPackage -Name Server2019Baseline -Configuration c:\git\policyfiles\localhost.mof -Verbose
   ```

## <a name="create-azure-policy-guest-configuration"></a>Создание гостевой конфигурации политики Azure

1. Следующим шагом является публикация файла в хранилище BLOB-объектов Azure. Для команды `Publish-GuestConfigurationPackage` требуется `Az.Storage` модуль.

   ```azurepowershell-interactive
   Publish-GuestConfigurationPackage -Path ./AuditBitlocker.zip -ResourceGroupName  myResourceGroupName -StorageAccountName myStorageAccountName
   ```

1. После создания и отправки пакета настраиваемой политики гостевой конфигурации создайте определение политики гостевой конфигурации. Используйте `New-GuestConfigurationPolicy` командлет, чтобы создать конфигурацию гостя.

   ```azurepowershell-interactive
    $NewGuestConfigurationPolicySplat = @{
        ContentUri = $Uri 
        DisplayName = 'Server 2019 Configuration Baseline' 
        Description 'Validation of using a completely custom baseline configuration for Windows VMs' 
        Path = 'C:\git\policyfiles\policy'  
        Platform = Windows 
        }
   New-GuestConfigurationPolicy @NewGuestConfigurationPolicySplat
   ```
    
1. Опубликуйте определения политик с помощью `Publish-GuestConfigurationPolicy` командлета. Этот командлет принимает только параметр **Path** с расположением JSON-файлов, созданных `New-GuestConfigurationPolicy`. Чтобы выполнить команду публикации, необходимо получить доступ к созданию определений политик в Azure. Требования к авторизации описаны на странице [Общие сведения о Политике Azure](../overview.md#getting-started). Наиболее подходящей встроенной ролью является **Участник политики ресурсов**.

   ```azurepowershell-interactive
   Publish-GuestConfigurationPolicy -Path C:\git\policyfiles\policy\ -Verbose
   ```

## <a name="assign-guest-configuration-policy-definition"></a>Назначение определения политики конфигурации гостя

После создания политики в Azure последним шагом является назначение инициативы. Узнайте о том, как назначить инициативу с помощью [портала](../assign-policy-portal.md), [Azure CLI](../assign-policy-azurecli.md) и [Azure PowerShell](../assign-policy-powershell.md).

> [!IMPORTANT]
> Определения политик гостевой конфигурации **всегда** должны быть назначены с помощью инициативы, объединяющей политики _помощью параметров auditifnotexists_ и _DeployIfNotExists_ . Если назначена только политика _AuditIfNotExists_, необходимые компоненты не развертываются, а в сведениях о политике всегда показано, что "0" серверов соответствуют требованиям.

Для назначения определения политики с помощью _DeployIfNotExists_ требуется дополнительный уровень доступа. Чтобы предоставить минимальные разрешения, можно создать пользовательское определение роли, расширяющее возможности роли **Участник политики ресурсов**. В приведенном ниже примере создается роль с именем **Resource Policy Contributor DINE** с дополнительным разрешением _Microsoft.Authorization/roleAssignments/write_.

   ```azurepowershell-interactive
   $subscriptionid = '00000000-0000-0000-0000-000000000000'
   $role = Get-AzRoleDefinition "Resource Policy Contributor"
   $role.Id = $null
   $role.Name = "Resource Policy Contributor DINE"
   $role.Description = "Can assign Policies that require remediation."
   $role.Actions.Clear()
   $role.Actions.Add("Microsoft.Authorization/roleAssignments/write")
   $role.AssignableScopes.Clear()
   $role.AssignableScopes.Add("/subscriptions/$subscriptionid")
   New-AzRoleDefinition -Role $role
   ```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте подробнее об аудите виртуальных машин с помощью [гостевой конфигурации](../concepts/guest-configuration.md).
- Узнайте о [программном создании политик](./programmatically-create.md).
- Узнайте, как [получать данные о соответствии](./get-compliance-data.md).
