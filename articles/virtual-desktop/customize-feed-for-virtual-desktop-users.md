---
title: Настройка веб-канала для пользователей виртуальных рабочих столов Windows в Azure
description: Настройка веб-канала для пользователей виртуальных рабочих столов Windows с помощью командлетов PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 09/14/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: e47486f29537cb948aaae7cf17e97bae14b60700
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90084300"
---
# <a name="customize-the-feed-for-windows-virtual-desktop-users"></a>Настройка веб-канала для пользователей виртуальных рабочих столов Windows

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows с объектами Azure Resource Manager для Виртуального рабочего стола Windows. Если вы используете Виртуальный рабочий стол Windows (классический) без объектов Azure Resource Manager, ознакомьтесь с [этой статьей](./virtual-desktop-fall-2019/customize-feed-virtual-desktop-users-2019.md).

Вы можете настроить веб-канал таким образом, чтобы ресурсы RemoteApp и удаленный рабочий стол отображались для пользователей с распознаваемым способом.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что вы уже скачали и установили модуль PowerShell для виртуальных рабочих столов Windows. Если вы этого не сделали, следуйте инструкциям в разделе [Настройка модуля PowerShell](powershell-module.md).

## <a name="customize-the-display-name-for-a-remoteapp"></a>Настройка отображаемого имени для RemoteApp

Вы можете изменить отображаемое имя для опубликованного RemoteApp, задав понятное имя. По умолчанию понятное имя совпадает с именем удаленного приложения RemoteApp.

Чтобы получить список опубликованных RemoteApp для группы приложений, выполните следующий командлет PowerShell:

```powershell
Get-AzWvdApplication -ResourceGroupName <resourcegroupname> -ApplicationGroupName <appgroupname>
```

Чтобы назначить понятное имя RemoteApp, выполните следующий командлет с необходимыми параметрами:

```powershell
Update-AzWvdApplication -ResourceGroupName <resourcegroupname> -ApplicationGroupName <appgroupname> -Name <applicationname> -FriendlyName <newfriendlyname>
```

Например, предположим, что вы получили текущие приложения с помощью следующего примера командлета:

```powershell
Get-AzWvdApplication -ResourceGroupName 0301RG -ApplicationGroupName 0301RAG | format-list
```

Результат будет выглядеть следующим образом:

```powershell
CommandLineArgument :
CommandLineSetting  : DoNotAllow
Description         :
FilePath            : C:\Program Files\Windows NT\Accessories\wordpad.exe
FriendlyName        : Microsoft Word
IconContent         : {0, 0, 1, 0…}
IconHash            : --iom0PS6XLu-EMMlHWVW3F7LLsNt63Zz2K10RE0_64
IconIndex           : 0
IconPath            : C:\Program Files\Windows NT\Accessories\wordpad.exe
Id                  : /subscriptions/<subid>/resourcegroups/0301RG/providers/Microsoft.DesktopVirtualization/applicationgroups/0301RAG/applications/Microsoft Word
Name                : 0301RAG/Microsoft Word
ShowInPortal        : False
Type                : Microsoft.DesktopVirtualization/applicationgroups/applications
```
Чтобы обновить понятное имя, выполните следующий командлет:

```powershell
Update-AzWvdApplication -GroupName 0301RAG -Name "Microsoft Word" -FriendlyName "WordUpdate" -ResourceGroupName 0301RG -IconIndex 0 -IconPath "C:\Program Files\Windows NT\Accessories\wordpad.exe" -ShowInPortal:$true -CommandLineSetting DoNotallow -FilePath "C:\Program Files\Windows NT\Accessories\wordpad.exe"
```

Чтобы подтвердить успешное обновление понятного имени, выполните следующий командлет:

```powershell
Get-AzWvdApplication -ResourceGroupName 0301RG -ApplicationGroupName 0301RAG | format-list FriendlyName
```

Командлет должен предоставить следующие выходные данные:

```powershell
FriendlyName        : WordUpdate
```

## <a name="customize-the-display-name-for-a-remote-desktop"></a>Настройка отображаемого имени для удаленный рабочий стол

Вы можете изменить отображаемое имя для опубликованного удаленного рабочего стола, указав Понятное имя. Если вы вручную создали пул узлов и группу классических приложений с помощью PowerShell, понятное имя по умолчанию — "сеанс рабочего стола". Если вы создали пул узлов и группу классических приложений с помощью шаблона Azure Resource Manager GitHub или предложения Azure Marketplace, понятное имя по умолчанию совпадает с именем пула узлов.

Чтобы получить ресурс удаленного рабочего стола, выполните следующий командлет PowerShell:

```powershell
Get-AzWvdDesktop -ResourceGroupName <resourcegroupname> -ApplicationGroupName <appgroupname> -Name <applicationname>
```

Чтобы назначить понятное имя ресурсу удаленного рабочего стола, выполните следующий командлет PowerShell:

```powershell
Update-AzWvdDesktop -ResourceGroupName <resourcegroupname> -ApplicationGroupName <appgroupname> -Name <applicationname> -FriendlyName <newfriendlyname>
```

## <a name="customize-a-display-name-in-azure-portal"></a>Настройка отображаемого имени в портал Azure

Вы можете изменить отображаемое имя для опубликованного удаленного рабочего стола, указав Понятное имя с помощью портал Azure.

1. Войдите на портал Azure по адресу <https://portal.azure.com>.

2. Выполните поиск по запросу **виртуальный рабочий стол Windows**.

3. В разделе "службы" выберите **Windows виртуальный рабочий стол**.

4. На странице виртуальных рабочих столов Windows в левой части экрана выберите **группы приложений** , а затем выберите имя группы приложений, которую требуется изменить. (Например, если вы хотите изменить отображаемое имя группы приложений для настольных систем, выберите группу приложений с именем **Desktop**.)

5. В меню в левой части экрана выберите **приложения** .

6. Выберите приложение, которое требуется обновить, а затем введите новое **Отображаемое имя**.

7. Щелкните **Сохранить**. Редактируемое приложение теперь должно отображать обновленное имя.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили веб-канал для пользователей, вы можете войти в клиент виртуальных рабочих столов Windows, чтобы протестировать его. Для этого перейдите к инструкциям по подключению к виртуальному рабочему столу Windows.

 * [Подключение с Windows 10 или Windows 7](connect-windows-7-10.md)
 * [Подключение с помощью веб-клиента](connect-web.md)
 * [Подключение к клиенту Android](connect-android.md)
 * [Подключение с помощью клиента iOS](connect-ios.md)
 * [Подключение с помощью клиента macOS](connect-macos.md)
