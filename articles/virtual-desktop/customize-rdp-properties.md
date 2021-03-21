---
title: Настройка свойств RDP с помощью PowerShell — Azure
description: Настройка свойств RDP для виртуальных рабочих столов Windows с помощью командлетов PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 10/09/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: cc3a08f383368b189e41bebd204707f2483e77c0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95018313"
---
# <a name="customize-remote-desktop-protocol-rdp-properties-for-a-host-pool"></a>Настройка свойств протокол удаленного рабочего стола (RDP) для пула узлов

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows с объектами Azure Resource Manager для Виртуального рабочего стола Windows. Если вы используете Виртуальный рабочий стол Windows (классический) без объектов Azure Resource Manager, ознакомьтесь с [этой статьей](./virtual-desktop-fall-2019/customize-rdp-properties-2019.md).

Настройка свойств протокол удаленного рабочего стола пула узлов (RDP), таких как использование нескольких мониторов и перенаправление звука, позволяет обеспечить оптимальную работу пользователей в зависимости от их потребностей. Свойства RDP можно настроить в виртуальном рабочем столе Windows с помощью портал Azure или с помощью параметра *-кустомрдппроперти* в командлете **Update-азввдхостпул** .

Полный список поддерживаемых свойств и их значений по умолчанию см. в разделе [Поддерживаемые параметры RDP-файла](/windows-server/remote/remote-desktop-services/clients/rdp-files?context=%2fazure%2fvirtual-desktop%2fcontext%2fcontext) .

## <a name="default-rdp-file-properties"></a>Свойства RDP-файла по умолчанию

По умолчанию RDP-файлы имеют следующие свойства:

|Свойство RDP|На рабочем столе|Как RemoteApp|
|---|---|---|
|Режим с несколькими мониторами|Выключено|Активировано|
|Перенаправление дисков включено|Диски, буфер обмена, принтеры, COM-порты и смарт-карты|Диски, буфер обмена и принтеры|
|Режим удаленного звука|Воспроизвести локально|Воспроизвести локально|

## <a name="prerequisites"></a>Предварительные требования

Прежде чем начать работу, выполните эти [ инструкции](powershell-module.md), чтобы настроить модуль PowerShell для Виртуального рабочего стола Windows и войти в Azure.

## <a name="configure-rdp-properties-in-the-azure-portal"></a>Настройка свойств RDP в портал Azure

Чтобы настроить свойства RDP в портал Azure:

1. Войдите в Azure по адресу <https://portal.azure.com>.
2. В строке поиска введите **Windows Virtual Desktop** .
3. В разделе "службы" выберите **Windows виртуальный рабочий стол**.
4. На странице виртуальных рабочих столов Windows в меню в левой части экрана выберите **Пулы узлов** .
5. Выберите **имя пула узлов** , который требуется обновить.
6. Выберите **свойства RDP** в меню в левой части экрана.
7. Задайте нужное свойство.
   - Кроме того, можно открыть вкладку **Дополнительно** и добавить свойства RDP в формате, разделенном точкой с запятой, например в примерах PowerShell в следующих разделах.
8. Когда все будет готово, нажмите кнопку **сохранить** , чтобы сохранить изменения.

В следующих разделах вы узнаете, как изменить настраиваемые свойства RDP вручную в PowerShell.

## <a name="add-or-edit-a-single-custom-rdp-property"></a>Добавление или изменение одного настраиваемого свойства RDP

Чтобы добавить или изменить одно настраиваемое свойство RDP, выполните следующий командлет PowerShell:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -CustomRdpProperty <property>
```

Чтобы проверить, обновлялся ли только что запущенный командлет, выполните следующий командлет:

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, CustomRdpProperty

Name              : <hostpoolname>
CustomRdpProperty : <customRDPpropertystring>
```

Например, если вы проводили проверку свойства "аудиокаптуремоде" в пуле узлов с именем 0301HP, необходимо ввести следующий командлет:

```powershell
Get-AzWvdHostPool -ResourceGroupName 0301rg -Name 0301hp | format-list Name, CustomRdpProperty

Name              : 0301HP
CustomRdpProperty : audiocapturemode:i:1;
```

## <a name="add-or-edit-multiple-custom-rdp-properties"></a>Добавление или изменение нескольких настраиваемых свойств RDP

Чтобы добавить или изменить несколько настраиваемых свойств RDP, выполните следующие командлеты PowerShell, предоставив настраиваемые свойства RDP в виде строки с разделителями-запятыми:

```powershell
$properties="<property1>;<property2>;<property3>"
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -CustomRdpProperty $properties
```

Чтобы проверить, было ли Добавлено свойство RDP, выполните следующий командлет:

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, CustomRdpProperty

Name              : <hostpoolname>
CustomRdpProperty : <customRDPpropertystring>
```

Если вы настроили несколько свойств RDP в пуле узлов 0301HP, то в соответствии с предыдущим примером командлет будет выглядеть следующим образом:

```powershell
Get-AzWvdHostPool -ResourceGroupName 0301rg -Name 0301hp | format-list Name, CustomRdpProperty

Name              : 0301HP
CustomRdpProperty : audiocapturemode:i:1;audiomode:i:0;
```

## <a name="reset-all-custom-rdp-properties"></a>Сброс всех настраиваемых свойств RDP

Можно восстановить значения по умолчанию для отдельных настраиваемых свойств RDP, следуя инструкциям в разделе [Добавление или изменение одного настраиваемого](#add-or-edit-a-single-custom-rdp-property)свойства RDP или сброс всех настраиваемых свойств RDP для пула узлов, выполнив следующий командлет PowerShell:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -CustomRdpProperty ""
```

Чтобы убедиться, что этот параметр успешно удален, введите следующий командлет:

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, CustomRdpProperty

Name              : <hostpoolname>
CustomRdpProperty : <CustomRDPpropertystring>
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили свойства протокола удаленного рабочего стола для определенного пула узлов, вы можете войти в клиент виртуальных рабочих столов Windows, чтобы протестировать их в рамках сеанса пользователя. В следующих руководствах вы узнаете, как подключиться к сеансу с помощью выбранного клиента:

- [Подключение с использованием клиента классических приложений Windows](connect-windows-7-10.md)
- [Подключение с помощью веб-клиента](connect-web.md)
- [Подключение к клиенту Android](connect-android.md)
- [Подключение с помощью клиента macOS](connect-macos.md)
- [Подключение с помощью клиента iOS](connect-ios.md)