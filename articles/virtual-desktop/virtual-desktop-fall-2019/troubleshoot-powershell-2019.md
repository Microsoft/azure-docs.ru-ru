---
title: Windows Virtual Desktop (классическая модель) PowerShell — Azure
description: Устранение неполадок с PowerShell при настройке среды клиента Windows для виртуальных рабочих столов (классическая модель).
author: Heidilohr
ms.topic: troubleshooting
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 08c9f7a6c5f8fd4a51c464018438bf6e7db119fb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100374904"
---
# <a name="windows-virtual-desktop-classic-powershell"></a>Оболочка виртуальных рабочих столов Windows (классическая модель)

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows (классическому), который не поддерживает объекты Azure Resource Manager для Виртуального рабочего стола Windows. Сведения об обеспечении управления объектами Azure Resource Manager для Виртуального рабочего стола Windows см. в [этой статье](../troubleshoot-powershell.md).

Используйте эту статью для устранения ошибок и проблем при использовании PowerShell с виртуальным рабочим столом Windows. Дополнительные сведения о службы удаленных рабочих столов PowerShell см. в статье [Windows Virtual Desktop PowerShell](/powershell/windows-virtual-desktop/overview).

## <a name="provide-feedback"></a>Отзывы

Посетите [техническое сообщество Виртуального рабочего стола Windows](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop), чтобы обсудить службу "Виртуальный рабочий стол Windows" с группой разработчиков и активными членами сообщества.

## <a name="powershell-commands-used-during-windows-virtual-desktop-setup"></a>Команды PowerShell, используемые при установке виртуальных рабочих столов Windows

В этом разделе перечислены команды PowerShell, которые обычно используются при настройке виртуальных рабочих столов Windows, а также способы решения проблем, которые могут возникнуть при их использовании.

### <a name="error-add-rdsappgroupuser-command----the-specified-userprincipalname-is-already-assigned-to-a-remoteapp-app-group-in-the-specified-host-pool"></a>Ошибка: Add-RdsAppGroupUser команда — указанное значение UserPrincipalName уже назначено группе приложений RemoteApp в указанном пуле узлов.

```Powershell
Add-RdsAppGroupUser -TenantName <TenantName> -HostPoolName <HostPoolName> -AppGroupName 'Desktop Application Group' -UserPrincipalName <UserName>
```

**Причина:** Используемое имя пользователя уже назначено группе приложений другого типа. Пользователи не могут быть назначены как для удаленного рабочего стола, так и для группы удаленных приложений в одном пуле узлов сеансов.

**Исправление:** Если пользователю нужны как удаленные приложения, так и удаленный рабочий стол, создайте разные пулы узлов или предоставьте пользователям доступ к удаленному рабочему столу, что позволит использовать любое приложение на виртуальной машине узла сеансов.

### <a name="error-add-rdsappgroupuser-command----the-specified-userprincipalname-doesnt-exist-in-the-azure-active-directory-associated-with-the-remote-desktop-tenant"></a>Ошибка: Add-RdsAppGroupUser команда — указанное значение UserPrincipalName не существует в Azure Active Directory, связанном с клиентом удаленный рабочий стол

```PowerShell
Add-RdsAppGroupUser -TenantName <TenantName> -HostPoolName <HostPoolName> -AppGroupName "Desktop Application Group" -UserPrincipalName <UserPrincipalName>
```

**Причина:** Пользователь, указанный параметром-UserPrincipalName, не может быть найден в Azure Active Directory, привязанном к клиенту виртуальных рабочих столов Windows.

**Исправление:** Подтвердите элементы в следующем списке.

- Пользователь синхронизирован с Azure Active Directory.
- Пользователь не связан с бизнес-клиентами (B2C) или бизнес-предприятием (B2B).
- Клиент виртуальных рабочих столов Windows привязан к правильному Azure Active Directory.

### <a name="error-get-rdsdiagnosticactivities----user-isnt-authorized-to-query-the-management-service"></a>Ошибка: Get-RdsDiagnosticActivities--пользователь не имеет права на запрос к службе управления

```PowerShell
Get-RdsDiagnosticActivities -ActivityId <ActivityId>
```

**Причина:** параметр-TenantName

**Исправление:** Выдача Get-RdsDiagnosticActivities с помощью-TenantName \<TenantName> .

### <a name="error-get-rdsdiagnosticactivities----the-user-isnt-authorized-to-query-the-management-service"></a>Ошибка: Get-RdsDiagnosticActivities--пользователь не имеет права на запрос к службе управления

```PowerShell
Get-RdsDiagnosticActivities -Deployment -username <username>
```

**Причина:** Использование параметра-Deployment.

**Исправление.** параметр-Deployment может использоваться только администраторами развертывания. Эти администраторы обычно являются членами группы службы удаленных рабочих столов или Windows Virtual Desktop. Замените параметр-Deployment на-TenantName \<TenantName> .

### <a name="error-new-rdsroleassignment----the-user-isnt-authorized-to-query-the-management-service"></a>Ошибка: New-RdsRoleAssignment--пользователь не имеет права на запрос к службе управления

**Причина 1:** Используемая учетная запись не имеет службы удаленных рабочих столов разрешений владельца на клиенте.

**Исправление 1:** Пользователю с разрешениями службы удаленных рабочих столов Owner необходимо выполнить назначение ролей.

**Причина 2:** Используемая учетная запись имеет службы удаленных рабочих столов разрешений владельца, но не является частью Azure Active Directory клиента или не имеет разрешений на запрос Azure Active Directory, где находится пользователь.

**Исправление 2:** Пользователю с разрешениями Active Directory необходимо выполнить назначение ролей.

>[!Note]
>New-RdsRoleAssignment не может предоставить разрешения пользователю, который не существует в Azure Active Directory (AD).

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения об устранении неполадок с Виртуальным рабочим столом Windows и о путях эскалации см. в статье [Общие сведения об устранении неисправностей, отзывы и поддержка](troubleshoot-set-up-overview-2019.md).
- Инструкции по устранению неполадок при создании клиента и пула узлов в среде Виртуального рабочего стола Windows см. в статье [Создание клиента и пула узлов](troubleshoot-set-up-issues-2019.md).
- Инструкции по устранению неполадок при настройке виртуальной машины через Виртуальный рабочий стол Windows см. в статье [Конфигурация виртуальной машины узла сеанса](troubleshoot-vm-configuration-2019.md).
- Сведения об устранении неполадок подключения клиентов к виртуальным рабочим столам Windows см. в статье [подключения к службе виртуальных рабочих столов Windows](troubleshoot-service-connection-2019.md).
- Сведения об устранении неполадок, связанных с удаленный рабочий стол клиентами, см. [в разделе Устранение неполадок клиента удаленный рабочий стол](../troubleshoot-client.md)
- Дополнительные сведения о службе см. в разделе [Среда виртуальных рабочих столов Windows](environment-setup-2019.md).
- Сведения об устранении неполадок см. в статье [Tutorial: Troubleshoot Resource Manager template deployments](../../azure-resource-manager/templates/template-tutorial-troubleshoot.md) (Руководство. Устранение неполадок развертывания шаблонов Resource Manager)
- Сведения о действиях аудита см. в статье [Операции аудита с помощью Resource Manager](../../azure-resource-manager/management/view-activity-logs.md).
- Дополнительные сведения об определении ошибок во время развертывания см. в статье [Просмотр операций развертывания с помощью портала Azure](../../azure-resource-manager/templates/deployment-history.md).
