---
title: Устранение неполадок при обновлении поставщика Microsoft Azure Site Recovery
description: Устранение распространенных проблем, возникающих при обновлении поставщика Microsoft Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: troubleshooting
ms.date: 11/10/2019
ms.author: raynew
ms.openlocfilehash: 5a6e4b415a9fe8ea80a84e415879df9d2f359478
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84434369"
---
# <a name="troubleshoot-microsoft-azure-site-recovery-provider-upgrade-failures"></a>Устранение неполадок с обновлением поставщика службы Microsoft Azure Site Recovery

Эта статья поможет устранить неполадки, которые могут возникнуть во время обновления поставщика службы Microsoft Azure Site Recovery.

## <a name="the-upgrade-fails-reporting-that-the-latest-site-recovery-provider-is-already-installed"></a>Сбой обновления с сообщением о том, что последний поставщик службы Azure Site Recovery уже установлен

Во время обновления поставщика службы Microsoft Azure Site Recovery (DRA) происходит сбой обновления средства унифицированной установки и отображается такое сообщение об ошибке:

Upgrade is not supported as a higher version of the software is already installed. (Обновление не поддерживается, так как уже установлена более поздняя версия программного обеспечения.)

Чтобы обновить поставщика службы, выполните следующие действия:

1. Скачайте средство унифицированной установки Microsoft Azure Site Recovery:
   1. В разделе "Ссылки на поддерживаемые накопительные пакеты обновления" статьи [Обновления службы в Azure Site Recovery](service-updates-how-to.md#links-to-currently-supported-update-rollups) выберите поставщика, который вы обновляете.
   2. На открывшейся странице перейдите в раздел **Update information** (Сведения об обновлении) и скачайте накопительный пакет обновлений для средства унифицированной установки Microsoft Azure Site Recovery.

2. Откройте командную строку и перейдите к папке, куда вы скачали пакет. Извлеките файлы средства установки из скачанного пакета с помощью такой команды: MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:&lt;путь к папке для извлеченных файлов&gt;.
    
    Пример команды:

    MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:C:\Temp\Extracted

3. В командной строке перейдите в папку с извлеченными файлами и запустите такие команды для установки:
   
    CX_THIRDPARTY_SETUP.EXE /VERYSILENT /SUPPRESSMSGBOXES /NORESTART  UCX_SERVER_SETUP.EXE /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /UPGRADE

1. Вернитесь в папку со скачанным пакетом и запустите команду MicrosoftAzureSiteRecoveryUnifiedSetup.exe, чтобы завершить обновление. 

## <a name="upgrade-failure-due-to-the-3rd-party-folder-being-renamed"></a>Сбой обновления из-за переименования сторонней папки

Для успешности обновления папка стороннего производителя не должна быть переименована.

Устранение проблемы:

1. Запустите редактор реестра (regedit.exe) и откройте ветвь HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\InMage Systems\Installed Products\10.
1. Проверьте значение ключа `Build_Version`. Если установлено значение последней версии, измените версию на более старую. Например, если последняя версия — 9.22.\* и ключ `Build_Version` имеет это значение, измените версию на 9.21.\*.
1. Скачайте последнюю версию средства унифицированной установки Microsoft Azure Site Recovery:
   1. В разделе "Ссылки на поддерживаемые накопительные пакеты обновления" статьи [Обновления службы в Azure Site Recovery](service-updates-how-to.md#links-to-currently-supported-update-rollups) выберите поставщика, который вы обновляете.
   2. На открывшейся странице перейдите в раздел **Update information** (Сведения об обновлении) и скачайте накопительный пакет обновлений для средства унифицированной установки Microsoft Azure Site Recovery.
1. Откройте командную строку, перейдите в папку со скачанным пакетом и извлеките файлы средства установки из скачанного пакета с помощью такой команды: MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:&lt;путь к папке для извлеченных файлов&gt;.

    Пример команды:

    MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:C:\Temp\Extracted

1. В командной строке перейдите в папку с извлеченными файлами и запустите такие команды для установки:
   
    CX_THIRDPARTY_SETUP.EXE /VERYSILENT /SUPPRESSMSGBOXES /NORESTART

1. Для отслеживания процесса установки используйте диспетчер задач. Когда процесс CX_THIRDPARTY_SETUP.EXE больше не будет отображаться в диспетчере задач, перейдите к следующему этапу.
1. Убедитесь, что путь C:\thirdparty существует и папка содержит библиотеки RRD.
1. Вернитесь в папку со скачанным пакетом и запустите команду MicrosoftAzureSiteRecoveryUnifiedSetup.exe, чтобы завершить обновление.

## <a name="upgrade-failure-due-to-master-target-installation-failure"></a>Сбой обновления из-за сбоя установки главного целевого сервера

При обновлении поставщика Microsoft Azure Site Recovery (DRA) установка главного целевого сервера завершается сбоем с ошибкой "расположение установки не существует и не содержит 1 ГБ свободного места и/или не существует на фиксированном диске".

Это может быть вызвано значением NULL для параметра в разделе реестра. Для устранения проблемы —

1. Запустите редактор реестра (regedit.exe) и откройте ветвь HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\4.
1. Проверьте значение ключа "InstallDirectory". Если он имеет значение null, добавьте текущее значение каталога установки.
1. Аналогичным образом откройте ветвь HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\5 в редакторе реестра.
1. Проверьте значение ключа "InstallDirectory" и добавьте текущее значение каталога установки.
1. Повторно запустите единый установщик установки.
