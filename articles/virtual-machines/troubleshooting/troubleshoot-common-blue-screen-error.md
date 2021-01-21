---
title: Ошибки "синий экран" при загрузке виртуальной машины Azure | Документация Майкрософт
description: Узнайте, как решить проблему получения ошибки "синий экран" при загрузке | Документация Майкрософт
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 09/28/2018
ms.author: genli
ms.openlocfilehash: 9a95ddf882e5edba9daa8ff91c02d1df1f50bceb
ms.sourcegitcommit: 484f510bbb093e9cfca694b56622b5860ca317f7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/21/2021
ms.locfileid: "98632982"
---
# <a name="windows-shows-blue-screen-error-when-booting-an-azure-vm"></a>Отображение ошибки "синий экран" при загрузке виртуальной машины Azure под управлением Windows
В этой статье описываются ошибки "синий экран", которые могут возникнуть при загрузке виртуальной машины Windows в Microsoft Azure. Представляем шаги, которые помогут вам при сборе данных для запроса в службу поддержки. 


## <a name="symptom"></a>Симптом 

Виртуальная машина Windows не запускается. При проверке снимков экрана загрузки в окне [Диагностика загрузки](./boot-diagnostics.md) появляется одно из следующих сообщений об ошибке на "синем экране":

- "На вашем ПК возникла проблема, и его необходимо перезагрузить. Мы лишь собираем некоторые сведения об ошибке, а затем вы сможете выполнить перезагрузку".
- "На вашем ПК возникла проблема, и его необходимо перезагрузить".

В этом разделе перечислены распространенные сообщения об ошибках, которые могут возникать при управлении виртуальной машиной.

## <a name="cause"></a>Причина

Ошибка с остановкой работы может возникать по нескольким причинам. Ниже перечислены наиболее распространенные из них.

- Проблема с драйвером.
- Поврежденный системный файл или память.
- Приложение обращается к запрещенному сектору памяти.

## <a name="collect-memory-dump-file"></a>Сбор файла дампа памяти

> [!TIP]
> Если у вас есть недавняя резервная копия виртуальной машины, можно попытаться [восстановить виртуальную машину из резервной копии](../../backup/backup-azure-arm-restore-vms.md) , чтобы устранить проблему загрузки.

Чтобы устранить эту проблему, для начала нужно собрать файл дампа аварийного завершения, а затем обратиться с ним в службу поддержки. Чтобы собрать файл дампа, выполните следующие действия.

### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Подключите диск ОС к виртуальной машине восстановления.

1. Сделайте снимок диска ОС затронутой виртуальной машины в качестве резервной копии. Дополнительные сведения см. в статье [Создание моментального снимка](../windows/snapshot-copy-managed-disk.md).
2. [Подключите диск операционной системы к виртуальной машине восстановления](./troubleshoot-recovery-disks-portal-windows.md). 
3. Подключитесь по протоколу удаленного рабочего стола к виртуальной машине восстановления.

### <a name="locate-dump-file-and-submit-a-support-ticket"></a>Найдите файл дампа и отправьте запрос в службу поддержки

1. На виртуальной машине восстановления в подключенном диске ОС перейдите в папку Windows. Если подключенному диску ОС присвоена буква F, то необходимо перейти в F:\Windows.
2. Найдите файл Memory. dmp и отправьте запрос в [службу поддержки](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) с файлом дампа. 

Если не удается найти файл дампа, перейдите на следующий шаг для включения журнала дампа и последовательной консоли.

### <a name="enable-dump-log-and-serial-console"></a>Включение журнала дампа и последовательной консоли

Чтобы включить журнал дампа и последовательную консоль, выполните следующий сценарий.

1. Откройте сеанс командной строки с повышенными привилегиями (выполнив запуск от имени администратора).
2. Выполните следующий скрипт:

    В этом сценарии мы предполагаем, что подключенному диску ОС присвоена буква F. Замените ее соответствующим значением на своей виртуальной машине.

    ```powershell
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REM Enable Serial Console
    bcdedit /store F:\boot\bcd /set {bootmgr} displaybootmenu yes
    bcdedit /store F:\boot\bcd /set {bootmgr} timeout 5
    bcdedit /store F:\boot\bcd /set {bootmgr} bootems yes
    bcdedit /store F:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    bcdedit /store F:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200

    REM Suggested configuration to enable OS Dump
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    reg unload HKLM\BROKENSYSTEM
    ```

    1. Убедитесь, что на диске достаточно места для выделения объема памяти, соответствующего объему ОЗУ, который зависит от выбираемого вами размера для виртуальной машины.
    2. Если места недостаточно или используется виртуальная машина большого размера (серии G, GS или E), то можно затем изменить расположение создания этого файла и указать любой другой диск данных, который присоединен к виртуальной машине. Для этого необходимо изменить следующий раздел.

    ```config-reg
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f

    reg unload HKLM\BROKENSYSTEM
    ```

3. [Отсоедините диск ОС и снова подключите его к необходимой виртуальной машине](./troubleshoot-recovery-disks-portal-windows.md).
4. Запустите виртуальную машину, чтобы воспроизвести проблему. Затем файл дампа будет создан.
5. Подключите диск ОС к виртуальной машине для восстановления, соберите файл дампа, а затем [отправьте запрос в службу поддержки](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) с этим файлом.
