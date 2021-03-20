---
title: Предварительная версия PowerShell для подключения к приложению MSIX для виртуальных рабочих столов Windows — Azure
description: Как настроить присоединение приложения MSIX для виртуальных рабочих столов Windows с помощью PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 12/14/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 1196982fedc7321805e36cceed27c90e43a6e705
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99558334"
---
# <a name="set-up-msix-app-attach-preview-using-powershell"></a>Настройка присоединения приложения MSIX (Предварительная версия) с помощью PowerShell

> [!IMPORTANT]
> Присоединение приложения MSIX в настоящее время находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Помимо портал Azure можно также настроить подключение приложения MSIX (Предварительная версия) вручную с помощью PowerShell. В этой статье описано, как использовать PowerShell для настройки подключения приложения MSIX.

## <a name="requirements"></a>Требования

>[!IMPORTANT]
>Прежде чем приступить к работе, обязательно заполните и отправьте [эту форму](https://aka.ms/enablemsixappattach) , чтобы включить присоединение приложения MSIX в вашей подписке. Если у вас нет утвержденного запроса, присоединение приложения MSIX не будет работать. Утверждение запросов может занять до 24 часов в рабочие дни. После принятия и завершения запроса вы получите сообщение электронной почты.

Вот что необходимо для настройки присоединения приложения MSIX:

- Работающее развертывание виртуального рабочего стола Windows. Сведения о развертывании виртуальных рабочих столов Windows (классическая модель) см. в статье [Создание клиента в виртуальном рабочем столе Windows](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md). Чтобы узнать, как развернуть виртуальный рабочий стол Windows с Azure Resource Managerной интеграцией, см. раздел [Создание пула узлов с портал Azure](./create-host-pools-azure-marketplace.md).
- Пул узлов виртуальных рабочих столов Windows, который имеет хотя бы один активный узел сеансов.
- Этот пул узлов должен находиться в среде проверки.
- Группа удаленных приложений для настольных компьютеров.
- Средство упаковки MSIX.
- MSIX приложение, развернутое в образе MSIX, передаваемый в общую папку.
- Файловый ресурс в развертывании виртуальных рабочих столов Windows, где будет храниться пакет MSIX.
- Общая папка, в которую вы перегрузили образ MSIX, также должна быть доступна для всех виртуальных машин в пуле узлов. Для доступа к образу пользователям потребуются разрешения только на чтение.
- Скачайте и установите PowerShell Core.
- Скачайте общедоступный модуль предварительной версии Azure PowerShell и разверните его в локальной папке.
- Установите модуль Azure, выполнив следующий командлет:

    ```powershell
    Install-Module -Name Az -Force
    ```

## <a name="sign-in-to-azure-and-import-the-module"></a>Войдите в Azure и импортируйте модуль.

Когда все требования будут готовы, откройте PowerShell Core в командной строке с повышенными привилегиями и выполните следующий командлет:

```powershell
Connect-AzAccount
```

После запуска выполните проверку подлинности учетной записи с помощью учетных данных. В этом случае может появиться запрос на ввод URL-адреса устройства или маркера.

## <a name="import-the-azwindowsvirtualdesktop-module"></a>Импортируйте модуль AZ. Виндовсвиртуалдесктоп

Для выполнения инструкций, описанных в этой статье, потребуется модуль AZ. Десктопвиртуализатион.

>[!NOTE]
>Для общедоступной предварительной версии модуль будет предоставляться как отдельные ZIP-файлы, которые необходимо импортировать вручную.

Прежде чем начать, можно выполнить следующий командлет, чтобы узнать, установлен ли модуль AZ. Десктопвиртуализатион в вашем сеансе или на виртуальной машине:

```powershell
Get-Module | Where-Object { $_.Name -Like "desktopvirtualization" }
```

Если вы хотите удалить существующую копию модуля в глобальной сети и начать заново, выполните следующий командлет:

```powershell
Uninstall-Module Az.DesktopVirtualization
```

Если модуль заблокирован на виртуальной машине, запустите этот командлет, чтобы разблокировать его:

```powershell
Unblock-File "<path>\Az.DesktopVirtualization.psm1"
```

Благодаря такой очистке, пора импортировать модуль.

1. Выполните следующий командлет, а затем нажмите клавишу **R** , когда появится запрос на согласие на запуск пользовательского кода.

   ```powershell
   Import-Module -Name "<path>\Az.DesktopVirtualization.psm1" -Verbose
   ```

2. После выполнения командлета Import проверьте, есть ли у него командлеты для MSIX, выполнив следующий командлет:

   ```powershell
   Get-Command -Module Az.DesktopVirtualization | Where-Object { $_.Name -match "MSIX" }
   ```

   Если командлеты существуют, выходные данные должны выглядеть следующим образом:

   ```powershell
   CommandType     Name                                               Version    Source

   -----------     ----                                               -------    ------

   Function        Expand-AzWvdMsixImage                              0.0        Az.DesktopVirtualization

   Function        Get-AzWvdMsixPackage                               0.0        Az.DesktopVirtualization

   Function        New-AzWvdMsixPackage                               0.0        Az.DesktopVirtualization

   Function        Remove-AzWvdMsixPackage                            0.0        Az.DesktopVirtualization

   Function        Update-AzWvdMsixPackage                            0.0        Az.DesktopVirtualization
   ```

   Если вы не видите эти выходные данные, закройте все сеансы PowerShell и PowerShell Core и повторите попытку.

## <a name="set-up-helper-variables"></a>Настройка вспомогательных переменных

После импорта модуля вам потребуется настроить вспомогательные переменные. В следующих примерах показано, как выполнять каждое из них.

Чтобы получить идентификатор подписки, сделайте следующее:

```powershell
Get-AzContext -ListAvailable | fl
```

Чтобы выбрать контекст клиента Azure и подписку с именем:

```powershell
$obj = Select-AzContext -Name "<Name>"
```

Чтобы задать переменную подписки, сделайте следующее:

```powershell
$subId = $obj.Subscription.Id
```

Чтобы задать имя рабочей области, сделайте следующее:

```powershell
$ws = "<WorksSpaceName>"
```

Чтобы задать имя пула узлов, сделайте следующее:

```powershell
$hp = "<HostPoolName>"
```

Чтобы настроить группу ресурсов, в которой настроены виртуальные машины узла сеансов, выполните следующие действия.

```powershell
$rg = "<ResourceGroupName>"
```

И, наконец, для подтверждения правильности установки всех переменных:

```powershell
Get-AzWvdWorkspace -Name $ws -ResourceGroupName $rg -SubscriptionId $subID
```

## <a name="add-an-msix-package-to-a-host-pool"></a>Добавление пакета MSIX в пул узлов

Когда все будет настроено, пора добавить пакет MSIX в пул узлов. Для этого сначала необходимо получить UNC-путь к образу MSIX.

Используя UNC-путь, выполните этот командлет, чтобы развернуть образ MSIX:

```powershell
$obj = Expand-AzWvdMsixImage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subID -Uri <UNCPath>
```

Выполните этот командлет, чтобы добавить пакет MSIX в нужный пул узлов:

```powershell
New-AzWvdMsixPackage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subId -PackageAlias $obj.PackageAlias -DisplayName <DisplayName> -ImagePath <UNCPath> -IsActive:$true
```

Когда все будет готово, убедитесь, что пакет был создан с помощью этого командлета:

```powershell
Get-AzWvdMsixPackage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subId | Where-Object {$_.PackageFamilyName -eq $obj.PackageFamilyName}
```

## <a name="remove-an-msix-package-from-a-host-pool"></a>Удаление пакета MSIX из пула узлов

Чтобы удалить пакет из пула узлов, выполните следующие действия.

Получите список всех пакетов, связанных с пулом узлов с помощью этого командлета, а затем найдите имя пакета, который необходимо удалить в выходных данных:

```powershell
Get-AzWvdMsixPackage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subId 
```

Кроме того, можно получить определенный пакет на основе его отображаемого имени с помощью этого командлета:

```powershell
Get-AzWvdMsixPackage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subId | Where-Object { $_.Name -like "Power" }
```

Чтобы удалить пакет, выполните следующий командлет:

```powershell
Remove-AzWvdMsixPackage -FullName $obj.PackageFullName -HostPoolName $hp -ResourceGroupName $rg
```

## <a name="publish-msix-apps-to-an-app-group"></a>Публикация приложений MSIX в группе приложений

Вы можете следовать инструкциям, приведенным в этом разделе, если вы завершили действия, описанные в предыдущих разделах. Если у вас есть пул узлов с активным узлом сеансов, по крайней мере одна группа приложений, и в пул узлов добавлен пакет MSIX, можно приступать к работе.

Чтобы опубликовать приложение из пакета MSIX в группе приложений, необходимо найти его имя, а затем использовать это имя в командлете публикации.

Чтобы опубликовать приложение, выполните следующие действия.

Выполните этот командлет, чтобы вывести список всех доступных групп приложений:

```powershell
Get-AzWvdApplicationGroup -ResourceGroupName $rg -SubscriptionId $subId
```

Когда вы нашли имя группы приложений, в которую хотите опубликовать приложения, используйте ее имя в этом командлете:

```powershell
$grName = "<AppGroupName>"
```

Наконец, необходимо опубликовать приложение.

- Чтобы опубликовать приложение MSIX в группе классических приложений, выполните следующий командлет:

   ```powershell
   New-AzWvdApplication -ResourceGroupName $rg -SubscriptionId $subId -Name PowerBi -ApplicationType MsixApplication -ApplicationGroupName $grName -MsixPackageFamilyName $obj.PackageFamilyName -CommandLineSetting 0
   ```

- Чтобы опубликовать приложение в удаленной группе приложений, выполните этот командлет:

   ```powershell
   New-AzWvdApplication -ResourceGroupName $rg -SubscriptionId $subId -Name PowerBi -ApplicationType MsixApplication -ApplicationGroupName $grName -MsixPackageFamilyName $obj.PackageFamilyName -CommandLineSetting 0 -MsixPackageApplicationId $obj.PackageApplication.AppId
   ```

>[!NOTE]
>Если пользователь назначен как в удаленную группу приложений, так и в группу классических приложений в одном пуле узлов, то при подключении пользователя к удаленному рабочему столу они увидят приложения MSIX из обеих групп.

## <a name="next-steps"></a>Дальнейшие действия

Запросите наши вопросы сообщества об этой функции в [Windows Virtual Desktop течкоммунити](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

Вы также можете оставить отзыв о виртуальном рабочем столе Windows в [центре обратной связи по виртуальным рабочим столам Windows](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).

Ниже приведены некоторые другие статьи, которые могут оказаться полезными.

- [Глоссарий по присоединению приложений MSIX](app-attach-glossary.md)
- [Часто задаваемые вопросы о присоединении приложения MSIX](app-attach-faq.md)
