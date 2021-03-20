---
title: Подготовка и настройка главного образа VHD — Azure
description: Как подготовить, настроить и передать основной образ Windows Virtual Desktop в Azure.
author: Heidilohr
ms.topic: how-to
ms.date: 01/19/2021
ms.author: helohr
manager: lizross
ms.openlocfilehash: ca92e6e888ead6bbd79c7a8d524516ee95e8c45d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101092926"
---
# <a name="prepare-and-customize-a-master-vhd-image"></a>Подготовка и настройка главного образа VHD

В этой статье описывается, как подготовить главный виртуальный жесткий диск (VHD) для отправки в Azure, включая создание виртуальных машин и установку на них программного обеспечения. Эти инструкции предназначены для конфигурации Виртуального рабочего стола Windows, которую можно использовать в имеющихся процессах вашей организации.

>[!IMPORTANT]
>Мы рекомендуем использовать образ из коллекции образов Azure. Однако если необходимо использовать настроенный образ, убедитесь, что на виртуальной машине не установлен агент виртуальных рабочих столов Windows. Использование пользовательского образа с агентом виртуальных рабочих столов Windows может привести к проблемам с образом, например к блокировке регистрации и предотвращению соединений с пользовательским сеансом.  

## <a name="create-a-vm"></a>Создание виртуальной машины

Многосеансовая поддержка Windows 10 Enterprise доступна в коллекции образов Azure. Существует два варианта настройки этого образа.

Первый вариант — подготовить виртуальную машину в Azure, выполнив инструкции из раздела Создание ВИРТУАЛЬНОЙ машины на [основе управляемого образа](../virtual-machines/windows/create-vm-generalized-managed.md), а затем перейти к [подготовке и установке программного обеспечения](set-up-customize-master-image.md#software-preparation-and-installation).

Второй вариант — создать образ локально, загрузив образ, подготавливая виртуальную машину Hyper-V и настроив ее в соответствии с вашими потребностями, которые мы рассмотрим в следующем разделе.

### <a name="local-image-creation"></a>Создание локального образа

После загрузки образа в локальное расположение откройте **Диспетчер Hyper-V** , чтобы создать виртуальную машину с скопированным VHD-диском. Ниже приведены инструкции по простой версии, но более подробные инструкции см. в статье [Создание виртуальной машины в Hyper-V](/windows-server/virtualization/hyper-v/get-started/create-a-virtual-machine-in-hyper-v/).

Чтобы создать виртуальную машину с скопированным виртуальным жестким диском:

1. Откройте **Мастер создания виртуальной машины**.

2. На странице Указание поколения выберите **поколение 1**.

    > [!div class="mx-imgBorder"]
    > ![Снимок экрана со страницей указания поколения. Выбран параметр "поколение 1".](media/a41174fd41302a181e46385e1e701975.png)

3. В разделе тип контрольной точки отключите контрольные точки, отключив флажок.

    > [!div class="mx-imgBorder"]
    > ![Снимок экрана: раздел типа контрольной точки на странице контрольные точки.](media/20c6dda51d7cafef33251188ae1c0c6a.png)

Можно также выполнить следующий командлет в PowerShell, чтобы отключить контрольные точки.

```powershell
Set-VM -Name <VMNAME> -CheckpointType Disabled
```

### <a name="fixed-disk"></a>Фиксированный диск

При создании виртуальной машины из существующего виртуального жесткого диска по умолчанию создается динамический диск. Его можно изменить на фиксированный диск, нажав кнопку **изменить диск...** , как показано на следующем рисунке. Более подробные инструкции см. [в статье Подготовка VHD или VHDX Windows к отправке в Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md).

> [!div class="mx-imgBorder"]
> ![Снимок экрана с параметром "изменить диск".](media/35772414b5a0f81f06f54065561d1414.png)

Можно также выполнить следующий командлет PowerShell, чтобы изменить диск на фиксированный.

```powershell
Convert-VHD –Path c:\test\MY-VM.vhdx –DestinationPath c:\test\MY-NEW-VM.vhd -VHDType Fixed
```

## <a name="software-preparation-and-installation"></a>Подготовка и установка программного обеспечения

В этом разделе описывается подготовка и установка Фслогикс и защитника Windows, а также некоторые основные параметры конфигурации для приложений и реестра образа.

Если вы устанавливаете Microsoft 365 приложения для Enterprise и OneDrive на виртуальной машине, перейдите к разделу [Установка Office в главном образе VHD](install-office-on-wvd-master-image.md) и следуйте инструкциям по установке приложений. После этого вернитесь к этой статье.

Если пользователям требуется доступ к определенным бизнес-приложениям, рекомендуется установить их после выполнения инструкций этого раздела.

### <a name="set-up-user-profile-container-fslogix"></a>Настройка контейнера профиля пользователя (Фслогикс)

Чтобы включить контейнер Фслогикс как часть образа, следуйте инструкциям в разделе [Создание контейнера профилей для пула узлов с помощью общей папки](create-host-pools-user-profile.md#configure-the-fslogix-profile-container). С помощью [этого краткого руководства](/fslogix/configure-cloud-cache-tutorial/)вы можете протестировать функциональные возможности контейнера фслогикс.

### <a name="configure-windows-defender"></a>Настройка защитника Windows

Если в виртуальной машине настроен защитник Windows, убедитесь, что он не просматривает все содержимое VHD-и VHDX-файлов во время вложения.

Эта конфигурация удаляет только сканирование VHD-и VHDX-файлов во время вложения, но не влияет на проверку в реальном времени.

Более подробные инструкции по настройке защитника Windows в Windows Server см. в разделе [Настройка исключений антивирусной программы защитника Windows в Windows Server](/windows/security/threat-protection/windows-defender-antivirus/configure-server-exclusions-windows-defender-antivirus/).

Дополнительные сведения о настройке защитника Windows для исключения определенных файлов из проверки см. в разделе [Настройка и проверка исключений на основе расширения файла и расположения папки](/windows/security/threat-protection/windows-defender-antivirus/configure-extension-file-exclusions-windows-defender-antivirus/).

### <a name="disable-automatic-updates"></a>Отключить автоматическое обновление

Чтобы отключить автоматическое обновление через локальные групповая политика:

1. Откройте **редактор локальных групповых политик \\ Административные шаблоны \\ компонентов Windows \\ Центр обновления Windows**.
2. Щелкните правой кнопкой мыши пункт **настроить автоматическое обновление** и установите для него значение **отключено**.

Можно также выполнить следующую команду в командной строке, чтобы отключить автоматическое обновление.

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
```

### <a name="specify-start-layout-for-windows-10-pcs-optional"></a>Укажите начальный макет для компьютеров с Windows 10 (необязательно)

Выполните эту команду, чтобы указать начальный макет для компьютеров с Windows 10.

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer" /v SpecialRoamingOverrideAllowed /t REG_DWORD /d 1 /f
```

### <a name="set-up-time-zone-redirection"></a>Настройка перенаправления часового пояса

Перенаправление часового пояса можно применить на уровне групповая политика, так как все виртуальные машины в пуле узлов входят в одну группу безопасности.

Перенаправление часовых поясов:

1. На Active Directory сервере откройте **консоль управления групповыми политиками**.
2. Разверните домен и групповая политика объекты.
3. Щелкните правой кнопкой мыши **объект Групповая политика** , созданный для параметров групповой политики, и выберите **изменить**.
4. В **редактор "Управление групповыми политиками"** перейдите к разделу **политики конфигурации компьютера**  >    >  **Административные шаблоны**  >  **компоненты Windows**  >  **службы удаленных рабочих столов**  >  **Удаленный рабочий стол устройство узла сеансов**  >  **и перенаправление ресурсов**.
5. Включите параметр **Разрешить перенаправление** часового пояса.

Эту команду также можно выполнить на главном образе для перенаправления часовых поясов:

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
```

### <a name="disable-storage-sense"></a>Отключение датчика хранилища

Для узла сеансов Виртуальных рабочих столов Windows, использующего Windows 10 Корпоративная или Windows 10 Корпоративная с несколькими сеансами, рекомендуется отключить Контроль памяти. Вы можете отключить контроль хранилища в меню "Параметры" в разделе " **хранилище**", как показано на следующем снимке экрана:

> [!div class="mx-imgBorder"]
> ![Снимок экрана меню "хранилище" в разделе "Параметры". Параметр "контроль хранилища" отключен.](media/storagesense.png)

Кроме того, этот параметр можно изменить с помощью реестра, выполнив следующую команду:

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
```

### <a name="include-additional-language-support"></a>Включить дополнительную поддержку языка

В этой статье не рассматривается настройка языковой и региональной поддержки. Дополнительные сведения см. в следующих статьях:

- [Добавление языков в образы Windows](/windows-hardware/manufacture/desktop/add-language-packs-to-windows/)
- [Функции по запросу](/windows-hardware/manufacture/desktop/features-on-demand-v2--capabilities/)
- [Возможности языка и региона по запросу (FOD)](/windows-hardware/manufacture/desktop/features-on-demand-language-fod/)

### <a name="other-applications-and-registry-configuration"></a>Другие приложения и конфигурация реестра

В этом разделе описывается конфигурация приложений и операционной системы. Все настройки в этом разделе выполняются с помощью записей реестра, которые можно выполнять с помощью средств командной строки и средства regedit.

>[!NOTE]
>Рекомендации можно реализовать в конфигурации с помощью групповая политика объектов (GPO) или импорта реестра. Администратор может выбрать любой вариант в зависимости от требований Организации.

Для сбора данных телеметрии центра отзывов в Windows 10 Корпоративная с несколькими сеансами выполните следующую команду:

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 3 /f
```

Выполните следующую команду для исправления сбоев программы Watson:

```cmd
remove CorporateWerServer* from Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Windows Error Reporting
```

Введите следующие команды в редакторе реестра, чтобы устранить 5 КБ разрешения. Перед включением параллельного стека необходимо выполнить эти команды.

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MaxMonitors /t REG_DWORD /d 4 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MaxXResolution /t REG_DWORD /d 5120 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MaxYResolution /t REG_DWORD /d 2880 /f

reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\rdp-sxs" /v MaxMonitors /t REG_DWORD /d 4 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\rdp-sxs" /v MaxXResolution /t REG_DWORD /d 5120 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\rdp-sxs" /v MaxYResolution /t REG_DWORD /d 2880 /f
```

## <a name="prepare-the-image-for-upload-to-azure"></a>Подготовка образа к отправке в Azure

После завершения настройки и установки всех приложений следуйте инструкциям в статье [Подготовка VHD-или VHDX Windows к отправке в Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md) для подготовки образа.

После подготовки образа для передачи убедитесь, что виртуальная машина остается в состоянии OFF или освобождено.

## <a name="upload-master-image-to-a-storage-account-in-azure"></a>Отправка основного образа в учетную запись хранения в Azure

Этот раздел применим только в том случае, если главный образ был создан локально.

Приведенные ниже инструкции помогут вам отправить основной образ в учетную запись хранения Azure. Если у вас еще нет учетной записи хранения Azure, следуйте инструкциям в [этой статье](/azure/developer/javascript/tutorial-vscode-static-website-node-03) , чтобы создать ее.

1. Преобразуйте образ виртуальной машины (VHD) в исправленный, если он еще не создан. Если вы не преобразуете образ в Fixed, вы не сможете успешно создать образ.

2. Отправьте виртуальный жесткий диск в контейнер больших двоичных объектов в вашей учетной записи хранения. Можно быстро отправить с помощью [средства Обозреватель службы хранилища](https://azure.microsoft.com/features/storage-explorer/). Дополнительные сведения о средстве Обозреватель службы хранилища см. в [этой статье](../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows).

    > [!div class="mx-imgBorder"]
    > ![Снимок экрана окна поиска Обозреватель службы хранилища Microsoft Azure инструмента. Установлен флажок "отправлять файлы VHD или VHDX как страничные BLOB-объекты (рекомендуется)".](media/897aa9a9b6acc0aa775c31e7fd82df02.png)

3. Затем перейдите к портал Azure в браузере и выполните поиск по фразе "изображения". Поиск должен привести к **созданию страницы создания образа** , как показано на следующем снимке экрана:

    > [!div class="mx-imgBorder"]
    > ![Снимок экрана страницы «Создание изображения» портал Azure, заполненный примерами значений для изображения.](media/d3c840fe3e2430c8b9b1f44b27d2bf4f.png)

4. После создания образа на следующем снимке экрана вы увидите такое уведомление:

    > [!div class="mx-imgBorder"]
    > ![Снимок экрана уведомления об успешно созданном образе.](media/1f41b7192824a2950718a2b7bb9e9d69.png)

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда у вас есть образ, можно создать или обновить пулы узлов. Дополнительные сведения о создании и обновлении пулов узлов см. в следующих статьях:

- [Create a host pool with an Azure Resource Manager template](./virtual-desktop-fall-2019/create-host-pools-arm-template.md) (Создание пула узлов с помощью шаблона Azure Resource Manager)
- [Руководство. Создание пула узлов с помощью Azure Marketplace](create-host-pools-azure-marketplace.md)
- [Create a host pool with PowerShell](create-host-pools-powershell.md) (Создание пула узлов с помощью PowerShell)
- [Создание контейнера профиля для пула узлов с помощью общей папки](create-host-pools-user-profile.md)
- [Настройка метода балансировки нагрузки виртуального рабочего стола Windows](configure-host-pool-load-balancing.md)

Если после подготовки или настройки образа VHD возникла проблема с подключением, ознакомьтесь с [руководством по устранению неполадок](troubleshoot-agent.md#your-issue-isnt-listed-here-or-wasnt-resolved) .
