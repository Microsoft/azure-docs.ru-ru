---
title: Обновление модулей Azure PowerShell в службе автоматизации Azure
description: В этой статье описывается, как обновлять популярные модули Azure PowerShell, предоставляемые по умолчанию в службе автоматизации Azure.
services: automation
ms.subservice: process-automation
ms.date: 06/14/2019
ms.topic: conceptual
ms.openlocfilehash: f05c3247ecdd77fe4cf02db70c7ba4b2c534f959
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99055354"
---
# <a name="update-azure-powershell-modules"></a>Обновление модулей Azure PowerShell

Самые популярные модули PowerShell предоставляются по умолчанию в каждой учетной записи службы автоматизации. См. раздел [Модули по умолчанию](shared-resources/modules.md#default-modules). Специалисты Azure регулярно обновляют модули Azure, и такие изменения могут затронуть включенные командлеты. Например, может произойти переименование параметра или прекращение поддержки командлета, что помешает работе ваших последовательностей runbook. 

> [!NOTE]
> Вы не можете удалять глобальные модули, которые служба автоматизации предоставляет в стандартной конфигурации.

## <a name="set-up-an-automation-account"></a>Настройка учетной записи службы автоматизации

Чтобы избежать негативного влияния на последовательности runbook и автоматизируемые процессы, выполняйте тестирование и проверку при любых обновлениях. Если у вас нет выделенной учетной записи службы автоматизации, предназначенной для этой цели, то рекомендуется ее создать, чтобы протестировать различные сценарии в процессе разработки модулей Runbook. Тестирование должно включать итеративные изменения, такие как обновление модулей PowerShell.

Убедитесь, что учетная запись службы автоматизации имеет созданную [учетную запись запуска от имени Azure](automation-security-overview.md#run-as-accounts) .

Если вы разрабатываете скрипты локально, для тестирования рекомендуется сохранить локально те же версии модуля, которые используются в учетной записи службы автоматизации. Так вы гарантированно получите те же результаты. После проверки результатов и применения необходимых изменений можно перейти к процессу переноса их в рабочую среду.

> [!NOTE]
> Новая учетная запись службы автоматизации может не содержать последние модули.

## <a name="obtain-a-runbook-to-use-for-updates"></a>Получение последовательности runbook для использования при обновлении

Для обновления модулей Azure в учетной записи службы автоматизации необходимо использовать последовательность runbook [Update-AutomationAzureModulesForAccount](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update) с открытым кодом. Чтобы начать обновление модулей Azure с помощью этой последовательности runbook, скачайте ее из репозитория GitHub. После этого вы сможете импортировать ее в учетную запись службы автоматизации или выполнить в качестве скрипта. Инструкции по импорту runbook в учетную запись службы автоматизации см. в статье [Импорт runbook](manage-runbooks.md#import-a-runbook).

Последовательность runbook **Update-AutomationAzureModulesForAccount** по умолчанию поддерживает обновление модулей Azure, AzureRM и AZ. Дополнительные сведения об обновлении модулей Az.Automation с помощью этой последовательности runbook см. в [этом файле сведений](https://github.com/microsoft/AzureAutomation-Account-Modules-Update/blob/master/README.md). При использовании модулей Az в учетной записи службы автоматизации необходимо учитывать дополнительные важные факторы. Дополнительные сведения см. в статье [Администрирование модулей в службе автоматизации Azure](shared-resources/modules.md).

## <a name="use-update-runbook-code-as-a-regular-powershell-script"></a>Использование кода runbook обновления в качестве обычного скрипта PowerShell

Вы можете выполнить код runbook в качестве обычного скрипта PowerShell. Для этого войдите в Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount), а затем передайте `-Login $false` в скрипт.

## <a name="use-the-update-runbook-on-sovereign-clouds"></a>Использование runbook обновления в национальных облаках

Если вы хотите выполнять эту последовательность runbook в национальных облаках, используйте параметр `AzEnvironment`, чтобы передать в runbook нужную среду. Приемлемые значения: AzureCloud (общедоступное облако Azure), AzureChinaCloud, AzureGermanCloud и AzureUSGovernment. Эти значения можно получить с помощью `Get-AzEnvironment | select Name`. Если вы не передаете значение этому командлету, по умолчанию runbook использует AzureCloud.

## <a name="use-the-update-runbook-to-update-a-specific-module-version"></a>Использование runbook обновления с конкретной версией модуля

Если вам важно использовать определенную версию модуля Azure, а не последнюю доступную в коллекции PowerShell, передайте все нужные версии в необязательном параметре `ModuleVersionOverrides` в runbook **Update-AutomationAzureModulesForAccount**. Примеры см. в модуле runbook [Update-AutomationAzureModulesForAccount.ps1](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update/blob/master/Update-AutomationAzureModulesForAccount.ps1). Модули Azure PowerShell, которые не упомянуты в параметре `ModuleVersionOverrides`, обновляются до последних версий из коллекции PowerShell. Если в параметр `ModuleVersionOverrides` не передано ничего, все модули обновляются последними версиями из коллекции PowerShell, как при нажатии кнопки **Обновить модули Azure** на портале Azure.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в статье [Администрирование модулей в службе автоматизации Azure](shared-resources/modules.md).
* Дополнительные сведения о runbook обновления см. на [странице репозитория этой последовательности runbook](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update).
