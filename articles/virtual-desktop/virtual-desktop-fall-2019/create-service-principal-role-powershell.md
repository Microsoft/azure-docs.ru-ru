---
title: Назначение роли субъекту-службе Виртуального рабочего стола Windows (классического) в Azure
description: Узнайте, как создать субъекты-службы и назначить роли с помощью PowerShell в Виртуальном рабочем столе Windows (классическом).
author: Heidilohr
ms.topic: tutorial
ms.date: 05/27/2020
ms.author: helohr
ms.custom: devx-track-azurepowershell
manager: femila
ms.openlocfilehash: 535ee2dace742a57877c9673e77bee64fd81c456
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106445046"
---
# <a name="tutorial-create-service-principals-and-role-assignments-with-powershell-in-windows-virtual-desktop-classic"></a>Руководство по Создание субъектов-служб и назначений ролей в Виртуальном рабочем столе Windows (классическом) с помощью PowerShell

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows (классическому), который не поддерживает объекты Azure Resource Manager для Виртуального рабочего стола Windows.

Субъекты-службы — это удостоверения, которые вы можете создать в Azure Active Directory для назначения ролей и разрешений с определенной целью. В Виртуальном рабочем столе Windows вы можете создать субъект-службу для решения следующих задач:

- автоматизация определенных задач управления Виртуального рабочего стола Windows.
- использование в качестве учетных данных вместо пользователей Многофакторной идентификации при использовании какого-либо из шаблонов Resource Manager в Виртуальном рабочем столе Windows.

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создайте субъект-службу в Azure Active Directory.
> * создание назначения роли в Виртуальном рабочем столе Windows.
> * вход в Виртуальный рабочий стол Windows с помощью субъекта-службы.

## <a name="prerequisites"></a>Предварительные требования

Перед созданием субъектов-служб и назначений ролей вам необходимо будет сделать следующее:

1. Установите модуль Azure AD. Чтобы установить модуль, запустите PowerShell как администратор и запустите следующий командлет:

    ```powershell
    Install-Module AzureAD
    ```

2. [Скачайте и импортируйте модуль PowerShell для Виртуального рабочего стола Windows](/powershell/windows-virtual-desktop/overview/).

3. Вам нужно выполнить все инструкции в этой статье в рамках одного сеанса PowerShell. Этот процесс может не выполняться при прерывании сеанса PowerShell путем закрытия окна и его повторного открытия.

## <a name="create-a-service-principal-in-azure-active-directory"></a>Создание субъекта-службы в Azure Active Directory

Выполнив все предварительные требования в сеансе PowerShell, запустите указанные ниже командлеты PowerShell, чтобы создать мультитенатный субъект-службу в Azure.

```powershell
Import-Module AzureAD
$aadContext = Connect-AzureAD
$svcPrincipal = New-AzureADApplication -AvailableToOtherTenants $true -DisplayName "Windows Virtual Desktop Svc Principal"
$svcPrincipalCreds = New-AzureADApplicationPasswordCredential -ObjectId $svcPrincipal.ObjectId
```
## <a name="view-your-credentials-in-powershell"></a>Просмотр учетных данных в PowerShell

Прежде чем создавать назначение ролей для субъекта-службы, просмотрите учетные данные и запишите их для дальнейшего использования. Особенно это касается пароля, так как вы не сможете получить его после закрытия этого сеанса PowerShell.

Ниже приведены учетные данные, которые вам нужно записать, и командлеты, которые необходимо запустить, чтобы получить их.

- Пароль:

    ```powershell
    $svcPrincipalCreds.Value
    ```

- Идентификатор клиента:

    ```powershell
    $aadContext.TenantId.Guid
    ```

- Идентификатор приложения:

    ```powershell
    $svcPrincipal.AppId
    ```

## <a name="create-a-role-assignment-in-windows-virtual-desktop"></a>создание назначения роли в Виртуальном рабочем столе Windows;

Затем необходимо создать назначение ролей, чтобы субъект-служба мог войти в Виртуальный рабочий стол Windows. Обязательно войдите с учетной записью, у которой есть разрешения создавать назначения ролей.

Сначала [скачайте и импортируйте модуль PowerShell для Виртуального рабочего стола Windows](/powershell/windows-virtual-desktop/overview/) для использования в сеансе PowerShell (если вы еще это не сделали).

Выполните следующие командлеты PowerShell, чтобы подключиться к Виртуальному рабочему столу Windows и отобразить клиенты.

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
Get-RdsTenant
```

Когда вы обнаружите имя клиента, для которого нужно создать назначение ролей, включите это имя в следующий командлет:

```powershell
$myTenantName = "<Windows Virtual Desktop Tenant Name>"
New-RdsRoleAssignment -RoleDefinitionName "RDS Owner" -ApplicationId $svcPrincipal.AppId -TenantName $myTenantName
```

## <a name="sign-in-with-the-service-principal"></a>Вход с помощью субъекта-службы

После создания назначения ролей для субъекта-службы удостоверьтесь, что субъект-служба может войти в Виртуальный рабочий стол Windows, запустив следующий командлет:

```powershell
$creds = New-Object System.Management.Automation.PSCredential($svcPrincipal.AppId, (ConvertTo-SecureString $svcPrincipalCreds.Value -AsPlainText -Force))
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com" -Credential $creds -ServicePrincipal -AadTenantId $aadContext.TenantId.Guid
```

Войдя, убедитесь, что все работает правильно, протестировав несколько командлетов PowerShell в Виртуальном рабочем столе Windows с использованием субъекта-службы.

## <a name="next-steps"></a>Дальнейшие действия

После создания субъекта-службы и назначения ему роли в клиенте Виртуального рабочего стола Windows его можно использовать для создания пула узлов. Дополнительные сведения см. в руководстве по созданию пула узлов в Виртуальном рабочем столе Windows.

 > [!div class="nextstepaction"]
 > [Создание пула узлов с помощью Azure Marketplace](create-host-pools-azure-marketplace-2019.md)
