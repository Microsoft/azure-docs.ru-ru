---
title: Включение удаленный рабочий стол для роли с помощью PowerShell
description: Как настроить приложение облачной службы Azure для подключений к удаленному рабочему столу с помощью PowerShell
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 5b1650edb575de8fd59ad2495dafcd628a717c02
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102610406"
---
# <a name="enable-remote-desktop-connection-for-a-role-in-azure-cloud-services-classic-using-powershell"></a>Включение подключение к удаленному рабочему столу для роли в облачных службах Azure (классическая модель) с помощью PowerShell

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

> [!div class="op_single_selector"]
> * [Портал Azure](cloud-services-role-enable-remote-desktop-new-portal.md)
> * [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md)
> * [Visual Studio](cloud-services-role-enable-remote-desktop-visual-studio.md)

С помощью удаленного рабочего стола обеспечивается доступ к рабочему столу экземпляра, работающего в Azure. Подключение к удаленному рабочему столу позволяет диагностировать и устранять неполадки выполняющегося приложения.

В этой статье описывается включение удаленного рабочего стола для ролей облачной службы с помощью PowerShell. Сведения о компонентах, которые потребуются для выполнения инструкций в этой статье, см. в статье [Установка и настройка Azure PowerShell](/powershell/azure/). В PowerShell используется расширение удаленного рабочего стола, поэтому удаленный рабочий стол можно включить даже после развертывания приложения.

## <a name="configure-remote-desktop-from-powershell"></a>Настройка удаленного рабочего стола с помощью PowerShell
Командлет [Set-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure.service/set-azureserviceremotedesktopextension) позволяет включить удаленный рабочий стол для всех или некоторых ролей в развернутой облачной службе. Он позволяет указать имя и пароль пользователя удаленного рабочего стола с помощью параметра *Credential* , который принимает объект PSCredential.

Если вы используете PowerShell в интерактивном режиме, то можете задать объект PSCredential, вызвав командлет [Get-Credentials](/powershell/module/microsoft.powershell.security/get-credential) .

```powershell
$remoteusercredentials = Get-Credential
```

Эта команда открывает диалоговое окно, в котором вы сможете в защищенном режиме указать имя и пароль удаленного пользователя.

Так как PowerShell используется в сценариях автоматизации, объект **PSCredential** можно настроить так, чтобы участие пользователя не требовалось. Сначала необходимо задать надежный пароль. Для начала нужно придумать текстовый пароль и преобразовать его в защищенную строку с помощью командлета [ConvertTo-SecureString](/powershell/module/microsoft.powershell.security/convertto-securestring). Далее следует преобразовать защищенную строку в зашифрованную стандартную строку, используя командлет [ConvertFrom-SecureString](/powershell/module/microsoft.powershell.security/convertfrom-securestring). После этого вы можете сохранить зашифрованную стандартную строку в файл с помощью командлета [Set-Content](/previous-versions/windows/it-pro/windows-powershell-1.0/ee176959(v=technet.10)).

Можно также создать файл с надежным паролем, чтобы не нужно было каждый раз вводить пароль. Кроме того, файл с надежным паролем безопаснее, чем обычный текстовый файл. Для создания файла надежного пароля используйте эту команду PowerShell:

```powershell
ConvertTo-SecureString -String "Password123" -AsPlainText -Force | ConvertFrom-SecureString | Set-Content "password.txt"
```

> [!IMPORTANT]
> При выборе пароля убедитесь, что соблюдены [требования к сложности](/previous-versions/windows/it-pro/windows-server-2003/cc786468(v=ws.10)).

Чтобы создать объект учетных данных из файла с надежным паролем, необходимо считать содержимое файла и преобразовать его обратно в защищенную строку с помощью командлета [ConvertTo-SecureString](/powershell/module/microsoft.powershell.security/convertto-securestring).

Командлет [Set-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure.service/set-azureserviceremotedesktopextension) принимает также параметр *Expiration* , который указывает значение **DateTime** , определяющее дату и время истечения срока действия учетной записи пользователя. Например, можно задать срок действия учетной записи, который истечет через несколько дней.

В этом примере показано, как установить расширение удаленного рабочего стола для облачной службы с помощью PowerShell:

```powershell
$servicename = "cloudservice"
$username = "RemoteDesktopUser"
$securepassword = Get-Content -Path "password.txt" | ConvertTo-SecureString
$expiry = $(Get-Date).AddDays(1)
$credential = New-Object System.Management.Automation.PSCredential $username,$securepassword
Set-AzureServiceRemoteDesktopExtension -ServiceName $servicename -Credential $credential -Expiration $expiry
```
При необходимости вы также можете указать слот развертывания и роли, для которых необходимо включить удаленный рабочий стол. Если эти параметры не указаны, командлет по умолчанию включит использование удаленного рабочего стола для всех ролей в слоте развертывания **рабочей среды** .

Расширение удаленного рабочего стола привязывается к развернутой службе. В случае создания нового развертывания службы потребуется включить использование удаленного рабочего стола для этого развертывания. Если необходимо, чтобы использование удаленного рабочего стола включалось всегда, рассмотрите возможность интеграции соответствующих сценариев PowerShell в рабочий процесс развертывания.

## <a name="remote-desktop-into-a-role-instance"></a>Подключение к экземпляру роли по протоколу удаленного рабочего стола

Командлет [Get-AzureRemoteDesktopFile](/powershell/module/servicemanagement/azure.service/get-azureremotedesktopfile) можно использовать, чтобы добавить удаленный рабочий стол для определенного экземпляра роли облачной службы. Можно использовать параметр *LocalPath* , чтобы скачать RDP-файл на локальный компьютер. Вы можете также использовать параметр *Launch* , чтобы открыть диалоговое окно "Подключение к удаленному рабочему столу" для доступа к экземпляру роли облачной службы.

```powershell
Get-AzureRemoteDesktopFile -ServiceName $servicename -Name "WorkerRole1_IN_0" -Launch
```

## <a name="check-if-remote-desktop-extension-is-enabled-on-a-service"></a>Убедитесь, что расширение удаленного рабочего стола включено в службе.

Командлет [Get-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure.service/get-azureremotedesktopfile) показывает, включен ли удаленный рабочий стол в развернутой службе. Он возвращает имя пользователя удаленного рабочего стола и роли, для которых включено расширение удаленного рабочего стола. По умолчанию это выполняется в слоте развертывания, и вы можете выбрать слот промежуточного развертывания.

```powershell
Get-AzureServiceRemoteDesktopExtension -ServiceName $servicename
```

## <a name="remove-remote-desktop-extension-from-a-service"></a>Удаление расширения удаленного рабочего стола из службы

Если вы уже включили расширение удаленного рабочего стола в развернутой службе и хотите обновить параметры удаленного рабочего стола, то сначала удалите расширение удаленного рабочего стола. Затем снова включите его с новыми параметрами. Например, если вы хотите задать новый пароль для учетной записи удаленного пользователя или ее срок действия истек. Данное действие необходимо только для существующих развертываний, в которых включено расширение удаленного рабочего стола. К новым развертываниям можно непосредственно применить расширение.

Чтобы удалить расширение удаленного рабочего стола из развернутой службы, используйте командлет [Remove-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure.service/remove-azureserviceremotedesktopextension) . При необходимости можно также указать слот развертывания и роль, для которой необходимо удалить удаленный рабочий стол.

```powershell
Remove-AzureServiceRemoteDesktopExtension -ServiceName $servicename -UninstallConfiguration
```

> [!NOTE]
> Чтобы полностью удалить конфигурацию расширения, выполните командлет *remove* с параметром **UninstallConfiguration** .
>
> Параметр **UninstallConfiguration** удаляет все конфигурации расширения, примененные к службе. Каждая конфигурация расширения связана с конфигурацией службы. Вызов командлета *remove* без параметра **UninstallConfiguration** разорвет связь <mark>развертывания</mark> с конфигурацией расширения, фактически удаляя это расширение. Однако конфигурация расширения будет по-прежнему связана со службой.

## <a name="additional-resources"></a>Дополнительные ресурсы

[Настройка облачных служб](cloud-services-how-to-configure-portal.md)