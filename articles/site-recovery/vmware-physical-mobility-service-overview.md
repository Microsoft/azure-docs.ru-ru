---
title: Сведения о службе Mobility Service для аварийного восстановления виртуальных машин VMware и физических серверов с помощью Azure Site Recovery | Документация Майкрософт
description: Узнайте о агенте службы Mobility Service для аварийного восстановления виртуальных машин VMware и физических серверов в Azure с помощью службы Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: how-to
ms.date: 04/10/2020
ms.author: ramamill
ms.openlocfilehash: 8c90427e2dd4ecf23cb803c4f75dbee34c38f992
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98898585"
---
# <a name="about-the-mobility-service-for-vmware-vms-and-physical-servers"></a>Сведения о службе Mobility Service на виртуальных машинах и физических серверах VMware

При настройке аварийного восстановления для виртуальных машин VMware и физических серверов с помощью [Azure Site Recovery](site-recovery-overview.md)вы устанавливаете службу Site Recovery Mobility Service на каждой локальной виртуальной машине VMware и физическом сервере. Служба Mobility Service фиксирует операции записи данных на компьютере и перенаправляет их на сервер обработки Site Recovery. Служба Mobility Service устанавливается программным обеспечением агента Mobility Service, которое можно развернуть с помощью следующих методов.

- [Принудительная установка](#push-installation). Если защита включена с помощью портал Azure, Site Recovery устанавливает службу Mobility Service на сервере.
- Установка вручную. службу Mobility Service можно установить вручную на каждом компьютере с помощью пользовательского [интерфейса](#install-the-mobility-service-using-ui) или [командной строки](#install-the-mobility-service-using-command-prompt).
- [Автоматическое развертывание](vmware-azure-mobility-install-configuration-mgr.md). Вы можете автоматизировать установку службы Mobility Service с помощью средств развертывания программного обеспечения, таких как Configuration Manager.

> [!NOTE]
> Служба Mobility Service использует примерно 6%-10% памяти на исходных компьютерах для виртуальных машин VMware или физических компьютеров.

## <a name="antivirus-on-replicated-machines"></a>Антивирусная программа на реплицированных компьютерах

Если на компьютерах, на которых требуется выполнить репликацию, выполняется антивирусная программа, исключите папку установки службы Mobility Service _к:\програмдата\аср\ажент_ из антивирусных операций. Это исключение гарантирует, что репликация будет работать должным образом.

## <a name="push-installation"></a>Принудительная установка

Принудительная установка является неотъемлемой частью задания, которое запускается из портал Azure для [включения репликации](vmware-azure-enable-replication.md#enable-replication). После выбора набора виртуальных машин, которые требуется защитить, и включения репликации сервер конфигурации отправляет агент службы Mobility Service на серверы, устанавливает агент и завершает регистрацию агента на сервере конфигурации.

### <a name="prerequisites"></a>Предварительные условия

- Убедитесь, что выполнены все [необходимые условия](vmware-azure-install-mobility-service.md) для принудительной установки.
- Убедитесь, что все конфигурации сервера соответствуют критериям в [матрице поддержки для аварийного восстановления виртуальных машин VMware и физических серверов в Azure](vmware-physical-azure-support-matrix.md).
- Начиная с версии 9,36, для SUSE Linux Enterprise Server 11 SP3, RHEL 5, CentOS 5, Debian 7 убедитесь, что последний установщик [доступен на сервере конфигурации и на сервере обработки масштабирования](#download-latest-mobility-agent-installer-for-suse-11-sp3-rhel-5-debian-7-server) .

Рабочий процесс принудительной установки описан в следующих разделах:

### <a name="mobility-service-agent-version-923-and-higher"></a>Агент службы Mobility Service версии 9,23 и выше

Дополнительные сведения о версии 9,23 см. в разделе [накопительный пакет обновления 35 для Azure Site Recovery](https://support.microsoft.com/help/4494485/update-rollup-35-for-azure-site-recovery).

Во время принудительной установки службы Mobility Service выполняются следующие действия.

1. Агент передается на исходный компьютер. Копирование агента на исходный компьютер может завершиться сбоем из-за нескольких ошибок среды. Ознакомьтесь с [нашим руководством](vmware-azure-troubleshoot-push-install.md) по устранению сбоев при принудительной установке.
1. После успешного копирования агента на сервер на сервере выполняется проверка необходимых компонентов.
   - Если все предварительные условия выполнены, установка начнется.
   - Установка завершается сбоем, если не выполнено одно или несколько [необходимых условий](vmware-physical-azure-support-matrix.md) .
1. В ходе установки агента устанавливается поставщик служба теневого копирования томов (VSS) для Azure Site Recovery. Поставщик VSS используется для создания точек восстановления, совместимых с приложениями. Если установка поставщика VSS завершается сбоем, этот шаг пропускается и установка агента будет продолжена.
1. Если установка агента завершается успешно, но происходит сбой установки поставщика VSS, то состояние задания помечается как **предупреждение**. Это не влияет на создание точек восстановления с постоянным сбоем.

    - Чтобы создать точки восстановления с последовательным применением, ознакомьтесь с [нашим руководством](vmware-physical-manage-mobility-service.md#install-site-recovery-vss-provider-on-source-machine) , чтобы выполнить ручную установку поставщика Site Recovery VSS.
    - Если вы не хотите создавать точки восстановления, соответствующие приложениям, [измените политику репликации](vmware-azure-set-up-replication.md#create-a-policy) , чтобы отключить точки восстановления, совместимые с приложениями.

### <a name="mobility-service-agent-version-922-and-below"></a>Агент службы Mobility Service версии 9,22 и ниже

1. Агент передается на исходный компьютер. Копирование агента на исходный компьютер может завершиться сбоем из-за нескольких ошибок среды. Ознакомьтесь с [нашим руководством](vmware-azure-troubleshoot-push-install.md) по устранению сбоев при принудительной установке.
1. После успешного копирования агента на сервер на сервере выполняется проверка необходимых компонентов.
   - Если все предварительные условия выполнены, установка начнется.
   - Установка завершается сбоем, если не выполнено одно или несколько [необходимых условий](vmware-physical-azure-support-matrix.md) .

1. В ходе установки агента устанавливается поставщик служба теневого копирования томов (VSS) для Azure Site Recovery. Поставщик VSS используется для создания точек восстановления, совместимых с приложениями.
   - В случае сбоя установки поставщика VSS Установка агента завершится ошибкой. Чтобы избежать сбоев при установке агента, используйте [версию 9,23](https://support.microsoft.com/help/4494485/update-rollup-35-for-azure-site-recovery) или более позднюю, чтобы создать отказоустойчивые точки восстановления и выполнить установку поставщика VSS вручную.

## <a name="install-the-mobility-service-using-ui"></a>Установка службы Mobility Service с помощью пользовательского интерфейса

### <a name="prerequisites"></a>Предварительные условия

- Убедитесь, что все конфигурации сервера соответствуют критериям в [матрице поддержки для аварийного восстановления виртуальных машин VMware и физических серверов в Azure](vmware-physical-azure-support-matrix.md).
- [Поиск установщика](#locate-installer-files) для операционной системы сервера.

>[!IMPORTANT]
> Не используйте метод установки пользовательского интерфейса, если вы реплицируете виртуальную машину Azure "инфраструктура как услуга" (IaaS) из одного региона Azure в другой. Используйте установку из [командной строки](#install-the-mobility-service-using-command-prompt) .

1. Скопируйте файл установки на компьютер и запустите его.
1. В колонке **параметров установки** выберите **Install Mobility Service** (Установить Mobility Service).
1. Выберите расположение установки и нажмите кнопку **установить**.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility1.png" alt-text="Страница параметров установки службы Mobility Service.":::

1. Ход состояния установки можно отслеживать в окне **Ход выполнения установки**. После завершения установки нажмите кнопку **Proceed to Configuration** (Перейти к конфигурации), чтобы зарегистрировать службу на сервере конфигурации.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility3.png" alt-text="Снимок экрана, показывающий ход выполнения установки, и кнопка &quot;активный переход к конфигурации&quot; после завершения установки.":::

1. На странице **сведения о сервере конфигурации** укажите НАСТРОЕННЫЙ IP-адрес и парольную фразу.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility4.png" alt-text="Страница регистрации службы Mobility Service.":::

1. Выберите **Зарегистрировать**, чтобы завершить регистрацию.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility5.png" alt-text="Последняя страница регистрации службы Mobility Service.":::

## <a name="install-the-mobility-service-using-command-prompt"></a>Установка службы Mobility Service с помощью командной строки

### <a name="prerequisites"></a>Предварительные условия

- Убедитесь, что все конфигурации сервера соответствуют критериям в [матрице поддержки для аварийного восстановления виртуальных машин VMware и физических серверов в Azure](vmware-physical-azure-support-matrix.md).
- [Поиск установщика](#locate-installer-files) для операционной системы сервера.

### <a name="windows-machine"></a>Компьютер Windows

- В командной строке выполните следующие команды, чтобы скопировать установщик в локальную папку, например _C:\temp_, на сервер, который требуется защитить. Замените имя файла установщика фактическим именем файла.

  ```cmd
  cd C:\Temp
  ren Microsoft-ASR_UA*Windows*release.exe MobilityServiceInstaller.exe
  MobilityServiceInstaller.exe /q /x:C:\Temp\Extracted
  cd C:\Temp\Extracted
  ```

- Выполните эту команду, чтобы установить агент.

  ```cmd
  UnifiedAgent.exe /Role "MS" /InstallLocation "C:\Program Files (x86)\Microsoft Azure Site Recovery" /Platform "VmWare" /Silent
  ```

- Выполните следующие команды, чтобы зарегистрировать агент на сервере конфигурации.

  ```cmd
  cd C:\Program Files (x86)\Microsoft Azure Site Recovery\agent
  UnifiedAgentConfigurator.exe  /CSEndPoint <CSIP> /PassphraseFilePath <PassphraseFilePath>
  ```

#### <a name="installation-settings"></a>Параметры установки

Параметр | Сведения
--- | ---
Синтаксис | `UnifiedAgent.exe /Role \<MS/MT> /InstallLocation \<Install Location> /Platform "VmWare" /Silent`
Журналы установки | `%ProgramData%\ASRSetupLogs\ASRUnifiedAgentInstaller.log`
`/Role` | Параметр, обязательный для установки. Указывает, следует ли устанавливать службу Mobility Service или главный целевой сервер.
`/InstallLocation`| Необязательный параметр. Указывает расположение установки (любая папка).
`/Platform` | Mandatory. Указывает платформу, на которой установлена служба Mobility Service: <br/> **VMware** для виртуальных машин VMware или физических серверов. <br/> **Azure** для виртуальных машин Azure.<br/><br/> Если виртуальные машины Azure разрабатывается как физические компьютеры, укажите **VMware**.
`/Silent`| Необязательный параметр. Указывает, разрешен ли запуск установщика в автоматическом режиме.

#### <a name="registration-settings"></a>Параметры регистрации

Параметр | Сведения
--- | ---
Синтаксис | `UnifiedAgentConfigurator.exe  /CSEndPoint \<CSIP> /PassphraseFilePath \<PassphraseFilePath>`
Журналы конфигурации агента | `%ProgramData%\ASRSetupLogs\ASRUnifiedAgentConfigurator.log`
`/CSEndPoint` | Обязательный параметр. `<CSIP>` Указывает IP-адрес сервера конфигурации. Используйте любой допустимый IP-адрес.
`/PassphraseFilePath` |  Mandatory. Расположение файла с парольной фразой. Используйте любой допустимый локальный путь к файлу или UNC.

### <a name="linux-machine"></a>Компьютер Linux

1. В сеансе терминала скопируйте установщик в локальную папку, например _/tmp_ , на сервере, который требуется защитить. Замените имя файла установщика на фактическое имя файла дистрибутива Linux, а затем выполните команды.

   ```shell
   cd /tmp ;
   tar -xvf Microsoft-ASR_UA_version_LinuxVersion_GA_date_release.tar.gz
   ```

2. Выполните установку следующим образом.

   ```shell
   sudo ./install -d <Install Location> -r MS -v VmWare -q
   ```

3. После завершения установки служба Mobility Service должна быть зарегистрирована на сервере конфигурации. Выполните следующую команду, чтобы зарегистрировать службу Mobility Service на сервере конфигурации.

   ```shell
   /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i <CSIP> -P /var/passphrase.txt
   ```

#### <a name="installation-settings"></a>Параметры установки

Параметр | Сведения
--- | ---
Синтаксис | `./install -d \<Install Location> -r \<MS/MT> -v VmWare -q`
`-r` | Параметр, обязательный для установки. Указывает, следует ли устанавливать службу Mobility Service или главный целевой сервер.
`-d` | Необязательный параметр. Указывает расположение установки службы Mobility Service: `/usr/local/ASR` .
`-v` | Mandatory. Указывает платформу, на которой установлена служба Mobility Service. <br/> **VMware** для виртуальных машин VMware или физических серверов. <br/> **Azure** для виртуальных машин Azure.
`-q` | Необязательный параметр. Указывает, разрешен ли запуск установщика в автоматическом режиме.

#### <a name="registration-settings"></a>Параметры регистрации

Параметр | Сведения
--- | ---
Синтаксис | `cd /usr/local/ASR/Vx/bin<br/><br/> UnifiedAgentConfigurator.sh -i \<CSIP> -P \<PassphraseFilePath>`
`-i` | Обязательный параметр. `<CSIP>` Указывает IP-адрес сервера конфигурации. Используйте любой допустимый IP-адрес.
`-P` |  Mandatory. Полный путь к файлу, в котором хранится парольная фраза. Используйте любую допустимую папку.

## <a name="azure-virtual-machine-agent"></a>Агент виртуальной машины Azure

- **Виртуальные машины Windows**. Начиная с версии 9.7.0.0 Mobility Service, [агент виртуальной машины Azure](../virtual-machines/extensions/features-windows.md#azure-vm-agent) устанавливает установщик Mobility Service. Это гарантирует, что при отработки отказа компьютера в Azure виртуальная машина Azure будет соответствовать необходимым условиям установки агента для использования любого расширения виртуальной машины.
- **Виртуальные машины Linux**. [WALinuxAgent](../virtual-machines/extensions/update-linux-agent.md) следует установить вручную на виртуальной машине Azure после отработки отказа.

## <a name="locate-installer-files"></a>Обнаружение файлов установщика

На сервере конфигурации перейдите в папку _%програмдата%\аср\хоме\свсистемс\пушинсталлсвк\репоситори_. Проверьте, какой установщик требуется в зависимости от операционной системы. В следующей таблице перечислены файлы установщика для каждой виртуальной машины VMware и физической серверной операционной системы. Перед началом работы можно ознакомиться с [поддерживаемыми операционными системами](vmware-physical-azure-support-matrix.md#replicated-machines).

> [!NOTE]
> Имена файлов используют синтаксис, приведенный в следующей таблице с _версиями_ и _датами_ в качестве заполнителей для реальных значений. Фактические имена файлов будут выглядеть, как в следующих примерах:
> - `Microsoft-ASR_UA_9.30.0.0_Windows_GA_22Oct2019_release.exe`
> - `Microsoft-ASR_UA_9.30.0.0_UBUNTU-16.04-64_GA_22Oct2019_release.tar.gz`

Файл установщика | Операционная система (только 64-разрядная версия)
--- | ---
`Microsoft-ASR_UA_version_Windows_GA_date_release.exe` | Windows Server 2016 </br> Windows Server 2012 R2 </br> Windows Server 2012 </br> Windows Server 2008 R2 с пакетом обновления 1 (SP1)
[Загрузить и поместить в эту папку вручную](#rhel-5-or-centos-5-server) | Red Hat Enterprise Linux (RHEL) 5 </br> CentOS 5
`Microsoft-ASR_UA_version_RHEL6-64_GA_date_release.tar.gz` | Red Hat Enterprise Linux (RHEL) 6 </br> CentOS 6
`Microsoft-ASR_UA_version_RHEL7-64_GA_date_release.tar.gz` | Red Hat Enterprise Linux (RHEL) 7 </br> CentOS 7
`Microsoft-ASR_UA_version_RHEL8-64_GA_date_release.tar.gz` | Red Hat Enterprise Linux (RHEL) 8 </br> CentOS 8
`Microsoft-ASR_UA_version_SLES12-64_GA_date_release.tar.gz` | SUSE Linux Enterprise Server 12 SP1 </br> Включает пакет обновления 2 (SP2) и SP3.
[Загрузить и поместить в эту папку вручную](#suse-11-sp3-server) | SUSE Linux Enterprise Server 11 SP3
`Microsoft-ASR_UA_version_SLES11-SP4-64_GA_date_release.tar.gz` | SUSE Linux Enterprise Server 11 SP4
`Microsoft-ASR_UA_version_SLES15-64_GA_date_release.tar.gz` | SUSE Linux Enterprise Server 15 
`Microsoft-ASR_UA_version_OL6-64_GA_date_release.tar.gz` | Oracle Enterprise Linux 6,4 </br> Oracle Enterprise Linux 6,5
`Microsoft-ASR_UA_version_OL7-64_GA_date_release.tar.gz` | Oracle Enterprise Linux 7 
`Microsoft-ASR_UA_version_OL8-64_GA_date_release.tar.gz` | Oracle Enterprise Linux 8
`Microsoft-ASR_UA_version_UBUNTU-14.04-64_GA_date_release.tar.gz` | Ubuntu Linux 14.04
`Microsoft-ASR_UA_version_UBUNTU-16.04-64_GA_date_release.tar.gz` | Сервер Ubuntu Linux 16.04 LTS
`Microsoft-ASR_UA_version_UBUNTU-18.04-64_GA_date_release.tar.gz` | Ubuntu Linux 18,04 LTS Server
`Microsoft-ASR_UA_version_UBUNTU-20.04-64_GA_date_release.tar.gz` | Ubuntu Linux 20,04 LTS Server
[Загрузить и поместить в эту папку вручную](#debian-7-server) | Debian 7
`Microsoft-ASR_UA_version_DEBIAN8-64_GA_date_release.tar.gz` | Debian 8
`Microsoft-ASR_UA_version_DEBIAN9-64_GA_date_release.tar.gz` | Debian 9

## <a name="download-latest-mobility-agent-installer-for-suse-11-sp3-rhel-5-debian-7-server"></a>Скачайте последнюю версию установщика агента мобильности для SUSE 11 SP3, RHEL 5, Debian 7 Server

### <a name="suse-11-sp3-server"></a>Сервер SUSE 11 SP3

В качестве **необходимого компонента для обновления или защиты SUSE Linux Enterprise Server 11 SP3 компьютеров** с версии 9,36 и выше:

1. Убедитесь, что последний установщик агента мобильности скачан из центра загрузки Майкрософт и размещен в репозитории Push Installer на сервере конфигурации и всех серверах обработки Scale out.
2. [Скачайте](site-recovery-whats-new.md) последнюю версию установщика агента SUSE Linux Enterprise Server 11 SP3.
3. Перейдите к серверу конфигурации, скопируйте установщик агента SUSE Linux Enterprise Server 11 SP3 на путь — INSTALL_DIR \home\svsystems\pushinstallsvc\repository
1. После копирования последнего установщика перезапустите службу Inmage PushInstall Service. 
1. Теперь перейдите к связанным серверам обработки масштабирования, повторите шаг 3 и шаг 4.
1. **Например**, если путь установки — C:\Program Files (x86) \Microsoft Azure Site Recovery, то упомянутые выше каталоги будут
    1. C:\Program Files (x86) \Microsoft Azure site Recovery\home\svsystems\pushinstallsvc\repository

### <a name="rhel-5-or-centos-5-server"></a>Сервер RHEL 5 или CentOS 5

В качестве **необходимого компонента для обновления или защиты компьютеров RHEL 5** с версии 9,36 и выше:

1. Убедитесь, что последний установщик агента мобильности скачан из центра загрузки Майкрософт и размещен в репозитории Push Installer на сервере конфигурации и всех серверах обработки Scale out.
2. [Скачайте](site-recovery-whats-new.md) последнюю версию УСТАНОВЩИКА агента RHEL 5 или CentOS 5. 
3. Перейдите на сервер конфигурации, скопируйте установщик агента RHEL 5 или CentOS 5 на страницу Path-INSTALL_DIR \home\svsystems\pushinstallsvc\repository
1. После копирования последнего установщика перезапустите службу Inmage PushInstall Service. 
1. Теперь перейдите к связанным серверам обработки масштабирования, повторите шаг 3 и шаг 4.
1. **Например**, если путь установки — C:\Program Files (x86) \Microsoft Azure Site Recovery, то упомянутые выше каталоги будут
    1. C:\Program Files (x86) \Microsoft Azure site Recovery\home\svsystems\pushinstallsvc\repository

## <a name="debian-7-server"></a>Сервер Debian 7

В качестве **необходимого компонента для обновления или защиты компьютеров Debian 7** с версии 9,36 и выше:

1. Убедитесь, что последний установщик агента мобильности скачан из центра загрузки Майкрософт и размещен в репозитории Push Installer на сервере конфигурации и всех серверах обработки Scale out.
2. [Скачайте](site-recovery-whats-new.md) последнюю версию установщика агента Debian 7.
3. Перейдите на сервер конфигурации, скопируйте установщик агента Debian 7 на путь-INSTALL_DIR \home\svsystems\pushinstallsvc\repository
1. После копирования последнего установщика перезапустите службу Inmage PushInstall Service. 
1. Теперь перейдите к связанным серверам обработки масштабирования, повторите шаг 3 и шаг 4.
1. **Например**, если путь установки — C:\Program Files (x86) \Microsoft Azure Site Recovery, то упомянутые выше каталоги будут
    1. C:\Program Files (x86) \Microsoft Azure site Recovery\home\svsystems\pushinstallsvc\repository

## <a name="next-steps"></a>Дальнейшие действия

[Установка службы Mobility Service для аварийного восстановления виртуальных машин VMware и физических серверов](vmware-azure-install-mobility-service.md)
