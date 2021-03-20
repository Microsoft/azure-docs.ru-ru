---
title: Управление группами приложений для виртуальных рабочих столов Windows PowerShell — Azure
description: Управление группами приложений виртуальных рабочих столов Windows с помощью PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: a6f24dea00a174aa0276a9b30add0854c3694056
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88008650"
---
# <a name="manage-app-groups-using-powershell"></a>Управление группами приложений с помощью PowerShell

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows с объектами Azure Resource Manager для Виртуального рабочего стола Windows. Если вы используете Виртуальный рабочий стол Windows (классический) без объектов Azure Resource Manager, ознакомьтесь с [этой статьей](./virtual-desktop-fall-2019/manage-app-groups-2019.md).

Группа приложений по умолчанию, созданная для нового пула узлов Виртуального рабочего стола Windows, также позволяет опубликовать полные классические приложения. Кроме того, вы можете создать одну или несколько групп приложений RemoteApp для пула узлов. Выполните инструкции, приведенные в этом учебнике, чтобы создать группу приложений RemoteApp и опубликовать отдельные приложения в меню **Пуск**.

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * создание группы приложений RemoteApp;
> * предоставление доступа к программам RemoteApp.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что вы использовали инструкции в разделе [Настройка модуля PowerShell](powershell-module.md) для настройки модуля PowerShell и входа в учетную запись Azure.

## <a name="create-a-remoteapp-group"></a>Создание группы приложений RemoteApp

Чтобы создать группу RemoteApp с помощью PowerShell, выполните следующие действия.

1. Выполните этот командлет PowerShell, чтобы создать пустую группу приложений RemoteApp.

   ```powershell
   New-AzWvdApplicationGroup -Name <appgroupname> -ResourceGroupName <resourcegroupname> -ApplicationGroupType "RemoteApp" -HostPoolArmPath '/subscriptions/SubscriptionId/resourcegroups/ResourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/HostPoolName'-Location <azureregion>
   ```

2. (Необязательно) Чтобы проверить, была ли создана группа приложений, выполните указанный ниже командлет и просмотрите список всех групп приложений для пула узлов.

   ```powershell
   Get-AzWvdApplicationGroup -Name <appgroupname> -ResourceGroupName <resourcegroupname>
   ```

3. Выполните указанный ниже командлет, чтобы получить список приложений в меню **Пуск**, которые присутствуют в образе виртуальной машины из пула узлов. Запишите значения свойств **FilePath**, **IconPath**, **IconIndex** и другие важные данные для приложения, которое вы хотите опубликовать.

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName <appgroupname> -ResourceGroupName <resourcegroupname> | Format-List | more
   ```

   Выходные данные должны отображать все пункты меню "Пуск" в следующем формате:

   ```powershell
   AppAlias            : access
   CommandLineArgument :
   FilePath            : C:\Program Files\Microsoft Office\root\Office16\MSACCESS.EXE
   FriendlyName        :
   IconIndex           : 0
   IconPath            : C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\accicons.exe
   Id                  : /subscriptions/resourcegroups/providers/Microsoft.DesktopVirtualization/applicationgroups/startmenuitems/Access
   Name                : 0301RAG/Access
   Type                : Microsoft.DesktopVirtualization/applicationgroups/startmenuitems

   AppAlias            : charactermap
   CommandLineArgument :
   FilePath            : C:\windows\system32\charmap.exe
   FriendlyName        :
   IconIndex           : 0
   IconPath            : C:\windows\system32\charmap.exe
   Id                  : /subscriptions/resourcegroups/providers/Microsoft.DesktopVirtualization/applicationgroups/startmenuitems/Character Map
   Name                : 0301RAG/Character Map
   Type                : Microsoft.DesktopVirtualization/applicationgroups/startmenuitems
   ```

4. Выполните следующий командлет, чтобы установить приложение на основе свойства `AppAlias`. Свойство `AppAlias` отображается после обработки выходных данных, полученных на шаге 3.

   ```powershell
   New-AzWvdApplication -AppAlias <appalias> -GroupName <appgroupname> -Name <remoteappname> -ResourceGroupName <resourcegroupname> -CommandLineSetting <DoNotAllow|Allow|Require>
   ```

5. Выполните следующий командлет, чтобы опубликовать новую программу RemoteApp в группе приложений, созданной на шаге 1.

   ```powershell
   New-AzWvdApplication -GroupName <appgroupname> -Name <remoteappname> -ResourceGroupName <resourcegroupname> -Filepath <filepath> -IconPath <iconpath> -IconIndex <iconindex> -CommandLineSetting <DoNotAllow|Allow|Require>
   ```

6. Чтобы убедиться в том, что приложение было опубликовано, выполните указанный ниже командлет.

   ```powershell
   Get-AzWvdApplication -GroupName <appgroupname> -ResourceGroupName <resourcegroupname>
   ```

7. Повторите шаги 1–5 для каждого приложения, которое вы хотите опубликовать в этой группе приложений.
8. Выполните указанный ниже командлет, чтобы предоставить пользователям доступ к программам RemoteApp в этой группе приложений.

   ```powershell
   New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

## <a name="next-steps"></a>Дальнейшие действия

Если вы приступили к этому руководству в наших руководствах, ознакомьтесь со [статьей Создание пула узлов для проверки обновлений службы](create-validation-host-pool.md). Вы может использовать пул узлов проверки, чтобы отслеживать обновления служб перед их развертыванием в рабочую среду.