---
title: Устранение проблем c общими ресурсами службы автоматизации Azure
description: В этой статье рассказывается, как устранять проблемы с общими ресурсами службы автоматизации Azure.
services: automation
ms.subservice: ''
ms.date: 01/27/2021
ms.topic: troubleshooting
ms.openlocfilehash: 1a822166ae4c2bf793e0fa50e93018f499fcc27a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99053626"
---
# <a name="troubleshoot-shared-resource-issues"></a>Устранение проблем с общими ресурсами

В этой статье обсуждаются проблемы, которые могут возникать при использовании [общих ресурсов](../automation-intro.md#shared-resources) в службе автоматизации Azure.

## <a name="modules"></a>Модули

### <a name="scenario-a-module-is-stuck-during-import"></a><a name="module-stuck-importing"></a>Сценарий. Модуль завис во время импорта

#### <a name="issue"></a>Проблема

При импорте или обновлении модуля в службе автоматизации Azure обнаружилось, что модуль завис в состоянии *Импорт*.

#### <a name="cause"></a>Причина

Так как импорт модулей PowerShell — это сложный многоэтапный процесс, модуль может быть неправильно импортирован и может зависнуть в переходном состоянии. Подробнее о процессе импорта см. в статье [Импорт модуля PowerShell](/powershell/scripting/developer/module/importing-a-powershell-module#the-importing-process).

#### <a name="resolution"></a>Решение

Чтобы устранить эту проблему, необходимо удалить зависший модуль с помощью командлета [Remove-AzAutomationModule](/powershell/module/Az.Automation/Remove-AzAutomationModule). Затем можно повторить импорт модуля.

```azurepowershell-interactive
Remove-AzAutomationModule -Name ModuleName -ResourceGroupName ExampleResourceGroup -AutomationAccountName ExampleAutomationAccount -Force
```

### <a name="scenario-azurerm-modules-are-stuck-during-import-after-an-update-attempt"></a><a name="update-azure-modules-importing"></a>Сценарий. Модули AzureRM зависают во время импорта после попытки обновления

#### <a name="issue"></a>Проблема

После попытки обновления модулей AzureRM в вашей учетной записи остается баннер со следующим сообщением.

```error
Azure modules are being updated
```

#### <a name="cause"></a>Причина

Это известная проблема, которая может возникать при обновлении модулей AzureRM в учетной записи службы автоматизации. Эта проблема, в частности, возникает, если модули находятся в группе ресурсов, имя которой представляет собой число и начинается с 0.

#### <a name="resolution"></a>Решение

Чтобы была возможность обновить модули AzureRM в учетной записи службы автоматизации, учетная запись должна находиться в группе ресурсов с буквенно-цифровым именем. В настоящее время в группах ресурсов с цифровыми именами, начинающимися с 0, модули AzureRM обновить невозможно.

### <a name="scenario-module-fails-to-import-or-cmdlets-cant-be-executed-after-importing"></a><a name="module-fails-to-import"></a>Сценарий. Не удается импортировать модуль, или после импорта не выполняются командлеты

#### <a name="issue"></a>Проблема

Модулю не удается осуществить импорт, либо импорт выполнен успешно, но командлеты не извлекаются.

#### <a name="cause"></a>Причина

Ниже приведены наиболее распространенные причины, из-за которых модуль не может успешно завершить импорт в службу автоматизации Azure.

* Структура не соответствует той, которая требуется для службы автоматизации.
* Модуль зависит от другого модуля, который не был развернут в учетной записи службы автоматизации.
* В папке отсутствуют зависимости модуля.
* Командлет [New-AzAutomationModule](/powershell/module/Az.Automation/New-AzAutomationModule) используется для передачи модуля, при этом полный путь для хранения не указан или модуль не загружен с помощью общедоступного URL-адреса.

#### <a name="resolution"></a>Решение

Для решения проблемы используйте одно из следующих решений.

* Убедитесь, что модуль имеет следующий формат: имя_модуля.zip -> имя_модуля или номер_версии -> (имя_модуля.psm1, имя_модуля.psd1).
* Откройте файл **PSD1** и проверьте, есть ли у модуля зависимости. Если зависимости есть, отправьте эти модули в учетную запись службы автоматизации.
* Убедитесь, что все указанные **DLL**-файлы находятся в папке модуля.

### <a name="scenario-update-azuremoduleps1-suspends-when-updating-modules"></a><a name="all-modules-suspended"></a>Сценарий. Update-AzureModule.ps1 приостанавливается при обновлении модулей

#### <a name="issue"></a>Проблема

Если для обновления модулей Azure используется модуль runbook [Update-AzureModule.ps1](https://github.com/azureautomation/runbooks/blob/master/Utility/ARM/Update-AzureModule.ps1), процесс обновления приостанавливается.

#### <a name="cause"></a>Причина

Для данного модуля runbook значение, определяющее количество одновременно обновляемых модулей, по умолчанию равно 10. Процесс обновления подвержен ошибкам, если одновременно обновляются слишком много модулей.

#### <a name="resolution"></a>Решение

Очень редко все модули AzureRM или Az требуются в одной учетной записи службы автоматизации. Следует импортировать только необходимые модули.

> [!NOTE]
> Не импортируйте весь модуль `Az.Automation` или `AzureRM.Automation`, который импортирует все автономные модули.

Если процесс обновления приостанавливается, добавьте параметр `SimultaneousModuleImportJobCount` в сценарий **Update-AzureModules.ps1** и укажите более низкое значение, чем значение по умолчанию, равное 10. Если вы реализуете эту логику, попробуйте начать со значения 3 или 5. `SimultaneousModuleImportJobCount` — параметр системного модуля runbook **Update-AutomationAzureModulesForAccount**, который используется для обновления модулей Azure. Если вы внесете указанное изменение, процесс обновления будет выполняться дольше, но вероятность его завершения будет более высокой. В следующем примере показан параметр и его место в runbook:

 ```powershell
         $Body = @"
            {
               "properties":{
               "runbook":{
                   "name":"Update-AutomationAzureModulesForAccount"
               },
               "parameters":{
                    ...
                    "SimultaneousModuleImportJobCount":"3",
                    ... 
               }
              }
           }
"@
```

## <a name="run-as-accounts"></a>Учетная запись запуска от имени

### <a name="scenario-youre-unable-to-create-or-update-a-run-as-account"></a><a name="unable-create-update"></a>Сценарий. Не удается создать или обновить учетную запись запуска от имени

#### <a name="issue"></a>Проблема

При попытке создать или обновить учетную запись запуска от имени появляется сообщение об ошибке примерно следующего содержания.

```error
You do not have permissions to create…
```

#### <a name="cause"></a>Причина

У вас нет разрешений, необходимых для создания или обновления учетной записи запуска от имени, или ресурс заблокирован на уровне группы ресурсов.

#### <a name="resolution"></a>Решение

Чтобы создать или обновить учетную запись запуска от имени, необходимо иметь соответствующие [разрешения](../automation-security-overview.md#permissions) на различные ресурсы, используемые этой учетной записью.

Если проблема связана с блокировкой, убедитесь, что эту блокировку можно снять. Затем на портале Azure перейдите к ресурсу, который заблокирован, щелкните блокировку правой кнопкой мыши и выберите **Удалить**.

### <a name="scenario-you-receive-the-error-unable-to-find-an-entry-point-named-getperadapterinfo-in-dll-iplpapidll-when-executing-a-runbook"></a><a name="iphelper"></a>Сценарий. При выполнении runbook появляется сообщение об ошибке "Не удается найти точку входа GetPerAdapterInfo в библиотеке DLL iplpapi.dll"

#### <a name="issue"></a>Проблема

При выполнении модуля runbook возникает следующее исключение.

```error
Unable to find an entry point named 'GetPerAdapterInfo' in DLL 'iplpapi.dll'
```

#### <a name="cause"></a>Причина

Скорее всего, эта ошибка возникает из-за неправильной настройки [учетной записи запуска от имени](../automation-security-overview.md).

#### <a name="resolution"></a>Решение

Убедитесь, что учетная запись запуска от имени настроена правильно. Затем убедитесь, что в модуле runbook у вас правильный код проверки подлинности в Azure. В следующем примере показан фрагмент кода модуля runbook для проверки подлинности в Azure с использованием учетной записи запуска от имени.

```powershell
$connection = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $connection.TenantID `
-ApplicationID $connection.ApplicationID -CertificateThumbprint $connection.CertificateThumbprint
```

## <a name="next-steps"></a>Дальнейшие действия

Если эта статья не помогла вам решить проблему, попробуйте использовать один из следующих каналов для получения дополнительной поддержки.

* Получите ответы специалистов Azure на [форумах Azure](https://azure.microsoft.com/support/forums/).
* Обратитесь за помощью в Твиттер — [@AzureSupport](https://twitter.com/azuresupport). Это официальная учетная запись Microsoft Azure для доступа к ресурсам сообщества Azure (ответы на вопросы, поддержка и консультации специалистов).
* Отправьте запрос в службу поддержки Azure Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и выберите **Получить поддержку**.