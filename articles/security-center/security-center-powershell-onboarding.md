---
title: Подключение к Центру безопасности Azure с помощью PowerShell
description: В этом документе рассматривается процесс включения Центра безопасности Azure с помощью командлетов PowerShell.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: quickstart
ms.date: 01/24/2021
ms.author: memildin
ms.openlocfilehash: 25639509cfb689e025ea4388f302378648af8ba6
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102439499"
---
# <a name="automate-onboarding-of-azure-security-center-using-powershell"></a>Автоматизированное подключение центра безопасности Azure с помощью PowerShell

Вы можете защитить рабочие нагрузки Azure программным способом с помощью модуля PowerShell центра безопасности Azure.
PowerShell позволяет автоматизировать задачи и избежать человеческих ошибок, возникающих в результате выполнения задач вручную. Это особенно полезно в крупномасштабных развертываниях, включающих десятки подписок с сотнями и тысячами ресурсов, которые должны быть защищены с самого начала.

Подключение центра безопасности Azure с помощью PowerShell позволяет программным образом автоматизировать подключение и управление вашими ресурсами Azure, а также добавить необходимые средства управления безопасностью.

В этой статье содержится пример сценария PowerShell, который можно изменить и использовать в вашей среде, чтобы развернуть центр безопасности для подписок. 

В рамках этого примера мы включим Центр безопасности для подписки с идентификатором d07c0080-170c-4c24-861d-9c817742786c и применим рекомендуемые параметры, которые обеспечивают высокий уровень защиты. Для этого мы используем службу Azure Defender, которая предоставляет дополнительные возможности защиты и обнаружения угроз.

1. Включите [Azure Defender](azure-defender.md). 
 
2. Задайте рабочую область Log Analytics, в которую агент Log Analytics будет отправлять данные, собранные на виртуальных машинах в этой подписке. В рамках примера мы используем существующую определенную пользователем рабочую область myWorkspace.

3. Активируйте автоматическую подготовку агента Центра безопасности, во время которой [развертывается агент Log Analytics](security-center-enable-data-collection.md#auto-provision-mma).

5. Назначьте [руководителя по информационной безопасности организации в качестве контактного лица по вопросам безопасности, которому будут отправляться оповещения Центра безопасности и сведения о важных событиях](security-center-provide-security-contact-details.md).

6. Присвойте [политики безопасности по умолчанию](tutorial-security-policy.md) центра безопасности.

## <a name="prerequisites"></a>Предварительные требования

Эти действия следует выполнять перед запуском командлетов центра безопасности:

1. Запустите PowerShell от имени администратора.

1. В PowerShell выполните следующие команды:
      
    ```Set-ExecutionPolicy -ExecutionPolicy AllSigned```

    ```Install-Module -Name Az.Security -Force```

## <a name="onboard-security-center-using-powershell"></a>Подключение центра безопасности с помощью PowerShell

1. Зарегистрируйте свои подписки в поставщике ресурсов центра безопасности:

    ```Set-AzContext -Subscription "d07c0080-170c-4c24-861d-9c817742786c"```

    ```Register-AzResourceProvider -ProviderNamespace 'Microsoft.Security'```

1. Необязательно: задайте уровень покрытия подписок (включение или отключение Azure Defender ). Если значение не указано, служба Defender отключена:

    ```Set-AzContext -Subscription "d07c0080-170c-4c24-861d-9c817742786c"```

    ```Set-AzSecurityPricing -Name "default" -PricingTier "Standard"```

1. Настройте рабочую область Log Analytics, куда агенты будут отправлять отчеты. У вас должна быть уже созданная рабочая область Log Analytics, куда виртуальные машины подписки будут отправлять отчеты. Вы можете определить несколько подписок для отправки отчетов в ту же рабочую область. Если не определить рабочую область, будет использоваться установленная по умолчанию.

    ```Set-AzSecurityWorkspaceSetting -Name "default" -Scope "/subscriptions/d07c0080-170c-4c24-861d-9c817742786c" -WorkspaceId"/subscriptions/d07c0080-170c-4c24-861d-9c817742786c/resourceGroups/myRg/providers/Microsoft.OperationalInsights/workspaces/myWorkspace"```

1. Запустите автоматическую подготовку установки агента Log Analytics на виртуальные машины Azure:
    
    ```Set-AzContext -Subscription "d07c0080-170c-4c24-861d-9c817742786c"```
    
    ```Set-AzSecurityAutoProvisioningSetting -Name "default" -EnableAutoProvision```

    > [!NOTE]
    > Рекомендуется включить автоматическую подготовку, чтобы убедиться, что виртуальные машины Azure автоматически защищены с помощью центра безопасности Azure.
    >

1. (Необязательно.) Настоятельно рекомендуется указать сведения о контактном лице по вопросам безопасности для подключаемых подписок. Эта информация будет использоваться для отправки оповещений и уведомлений, созданных центром безопасности:

    ```Set-AzSecurityContact -Name "default1" -Email "CISO@my-org.com" -Phone "2142754038" -AlertAdmin -NotifyOnAlert```

1. Назначьте инициативу политики центра безопасности по умолчанию:

    ```Register-AzResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'```

    ```$Policy = Get-AzPolicySetDefinition | where {$_.Properties.displayName -EQ 'Azure Security Benchmark'} New-AzPolicyAssignment -Name 'ASC Default <d07c0080-170c-4c24-861d-9c817742786c>' -DisplayName 'Security Center Default <subscription ID>' -PolicySetDefinition $Policy -Scope '/subscriptions/d07c0080-170c-4c24-861d-9c817742786c'```

Вы успешно подключили Центр безопасности Azure с помощью PowerShell.

Теперь вы можете использовать эти командлеты PowerShell со сценариями автоматизации, чтобы программно повторить эти операции для подписок и ресурсов. Это экономит время и уменьшает вероятность человеческой ошибки. Вы можете использовать этот [пример сценария](https://github.com/Microsoft/Azure-Security-Center/blob/master/quickstarts/ASC-Samples.ps1) в качестве ориентира.




## <a name="see-also"></a>См. также
Дополнительные сведения об использовании PowerShell для автоматизации подключения к центру безопасности см. в следующей статье:

* [Az.Security](/powershell/module/az.security)

Дополнительные сведения о центре безопасности см. в следующих статьях:

* [Настройка политик безопасности в Центре безопасности Azure](tutorial-security-policy.md) — узнайте, как настроить политики безопасности для подписок и групп ресурсов Azure.
* [Управление оповещениями безопасности в Центре безопасности Azure и реагирование на них](security-center-managing-and-responding-alerts.md) — узнайте, как управлять оповещениями системы безопасности и реагировать на них.