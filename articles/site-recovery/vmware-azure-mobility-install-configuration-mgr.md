---
title: Автоматизируйте службу Mobility Service для аварийного восстановления установки в Azure Site Recovery
description: Как автоматически установить службу Mobility Service для аварийного восстановления сервера VMware/фисикал с помощью Azure Site Recovery.
author: Rajeswari-Mamilla
ms.topic: how-to
ms.date: 2/5/2020
ms.author: ramamill
ms.openlocfilehash: 2159ab8c2639f0f87fd53e8559dad518a3daa663
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92544823"
---
# <a name="automate-mobility-service-installation"></a>Автоматизация установки службы Mobility Service

В этой статье описывается, как автоматизировать установку и обновление агента службы Mobility Service в [Azure Site Recovery](site-recovery-overview.md).

При развертывании Site Recovery для аварийного восстановления локальных виртуальных машин VMware и физических серверов в Azure необходимо установить агент службы Mobility Service на каждом компьютере, который необходимо реплицировать. Служба Mobility Service захватывает операции записи данных на компьютере и перенаправляет их на сервер обработки Site Recovery для репликации. Вы можете развернуть службу Mobility Service несколькими способами:

- **Принудительная установка**: Позвольте Site Recovery установить агент службы Mobility Service при включении репликации для компьютера в портал Azure.
- **Установка вручную**. вручную установите службу Mobility Service на каждом компьютере. [Дополнительные сведения](vmware-physical-mobility-service-overview.md) о принудительной установке.
- **Автоматическое развертывание**. Автоматизируйте установку с помощью средств развертывания программного обеспечения, таких как Microsoft Endpoint Configuration Manager или сторонних средств, таких как жетпатч.

Автоматическая установка и обновление предоставляют решение, если:

- Ваша организация не разрешает принудительную установку на защищенных серверах.
- Политика компании требует периодического изменения паролей. Необходимо указать пароль для принудительной установки.
- Политика безопасности не допускают Добавление исключений брандмауэра для конкретных компьютеров.
- Вы являетесь поставщиком услуг размещения и не хотите предоставлять учетные данные компьютера клиента, необходимые для принудительной установки с Site Recovery.
- Необходимо масштабировать установки агента на множество серверов одновременно.
- Необходимо запланировать установку и обновление во время плановых периодов обслуживания.

## <a name="prerequisites"></a>Предварительные условия

Для автоматизации установки необходимы следующие компоненты:

- Развернутое решение установки программного обеспечения, например [Configuration Manager](/configmgr/) или [жетпатч](https://jetpatch.com/microsoft-azure-site-recovery/).
- Необходимые условия для развертывания в [Azure](tutorial-prepare-azure.md) и [локальной среде](vmware-azure-tutorial-prepare-on-premises.md) для аварийного восстановления VMware или для аварийного восстановления [физического сервера](physical-azure-disaster-recovery.md) . Ознакомьтесь с [требованиями к поддержке](vmware-physical-azure-support-matrix.md) аварийного восстановления.

## <a name="prepare-for-automated-deployment"></a>Подготовка к автоматическому развертыванию

В следующей таблице перечислены средства и процессы для автоматизации развертывания службы Mobility Service.

**Инструмент** | **Сведения** | **Инструкции**
--- | --- | ---
**диспетчер конфигураций** | 1. Убедитесь, что выполнены указанные выше [условия](#prerequisites) . <br/><br/> 2. Разверните аварийное восстановление, настроив исходную среду, включая загрузку файла OVA, чтобы развернуть сервер конфигурации Site Recovery в качестве виртуальной машины VMware с помощью шаблона OVF.<br/><br/> 3. Вы регистрируете сервер конфигурации в службе Site Recovery, настраиваете целевую среду Azure и настраиваете политику репликации.<br/><br/> 4. для автоматического развертывания службы Mobility Service создайте общую сетевую папку, содержащую файлы установки парольной фразы сервера конфигурации и службы Mobility Service.<br/><br/> 5. вы создаете пакет Configuration Manager, содержащий установку или обновления, а также готовитесь к развертыванию службы Mobility Service.<br/><br/> 6. После этого можно включить репликацию в Azure для компьютеров, на которых установлена служба Mobility Service. | [Автоматизация с помощью Configuration Manager](#automate-with-configuration-manager)
**жетпатч** | 1. Убедитесь, что выполнены указанные выше [условия](#prerequisites) . <br/><br/> 2. Разверните аварийное восстановление, настроив исходную среду, включая загрузку и развертывание Жетпатч диспетчер агентов для Azure Site Recovery в среде Site Recovery с помощью шаблона OVF.<br/><br/> 3. Вы регистрируете сервер конфигурации в Site Recovery, настраиваете целевую среду Azure и настраиваете политику репликации.<br/><br/> 4. для автоматического развертывания инициализируйте и завершите настройку Жетпатч диспетчер агентов.<br/><br/> 5. в Жетпатч можно создать политику Site Recovery для автоматизации развертывания и обновления агента службы Mobility Service. <br/><br/> 6. После этого можно включить репликацию в Azure для компьютеров, на которых установлена служба Mobility Service. | [Автоматизация с помощью Жетпатч диспетчер агентов](https://jetpatch.com/microsoft-azure-site-recovery-deployment-guide/)<br/><br/> [Устранение неполадок установки агента в Жетпатч](https://kc.jetpatch.com/hc/articles/360035981812)

## <a name="automate-with-configuration-manager"></a>Автоматизация с помощью Configuration Manager

### <a name="prepare-the-installation-files"></a>Подготовка файлов установки

1. Убедитесь, что выполнены необходимые условия.
1. Создайте защищенную сетевую общую папку (общую папку SMB), к которой может получить доступ компьютер, на котором работает сервер конфигурации.
1. В Configuration Manager [категории серверов](/sccm/core/clients/manage/collections/automatically-categorize-devices-into-collections) , на которых требуется установить или обновить службу Mobility Service. Одна коллекция должна содержать все серверы Windows, другие серверы Linux.
1. В общей сетевой папке создайте папку:

   - Для установки на компьютерах под Windows создайте папку с именем _Каталог mobsvcwindows_.
   - Для установки на компьютерах Linux создайте папку с именем _Каталог mobsvclinux_.

1. Войдите на компьютер сервера конфигурации.
1. На компьютере сервера конфигурации откройте командную строку администрирования.
1. Чтобы создать файл парольной фразы, выполните следующую команду:

    ```Console
    cd %ProgramData%\ASR\home\svsystems\bin
    genpassphrase.exe -v > MobSvc.passphrase
    ```

1. Скопируйте файл _файл mobsvc. пароль_ в папку Windows и папку Linux.
1. Чтобы перейти к папке, содержащей файлы установки, выполните следующую команду:

    ```Console
    cd %ProgramData%\ASR\home\svsystems\pushinstallsvc\repository
    ```

1. Скопируйте эти файлы установки в общую сетевую папку:

   - Для Windows скопируйте _Microsoft-ASR_UA_version_Windows_GA_date_Release.exe_ в _Каталог mobsvcwindows_.
   - Для Linux скопируйте следующие файлы в _Каталог mobsvclinux_:
     - _Microsoft-ASR_UARHEL6 -64release. tar. gz_
     - _Microsoft-ASR_UARHEL7 -64release. tar. gz_
     - _Microsoft-ASR_UASLES11-SP3-64release. tar. gz_
     - _Microsoft-ASR_UASLES11-SP4-64release. tar. gz_
     - _Microsoft-ASR_UAOL6 -64release. tar. gz_
     - _Microsoft-ASR_UAUBUNTU-14.04 -64release. tar. gz_

1. Как описано в следующих процедурах, скопируйте код в папки Windows или Linux. Предполагается, что:

   - IP-адрес сервера конфигурации — `192.168.3.121` .
   - Безопасной сетевой общей папкой является `\\ContosoSecureFS\MobilityServiceInstallers` .

### <a name="copy-code-to-the-windows-folder"></a>Копирование кода в папку Windows

Скопируйте следующий код:

- Сохраните код в папке _Каталог mobsvcwindows_ как _install.bat_.
- Замените `[CSIP]` заполнители в этом скрипте фактическими значениями IP-адреса сервера конфигурации.
- Скрипт поддерживает новые установки агента службы Mobility Service и обновления уже установленных агентов.

```DOS
Time /t >> C:\Temp\logfile.log
REM ==================================================
REM ==== Clean up the folders ========================
RMDIR /S /q %temp%\MobSvc
MKDIR %Temp%\MobSvc
MKDIR C:\Temp
REM ==================================================

REM ==== Copy new files ==============================
COPY M*.* %Temp%\MobSvc
CD %Temp%\MobSvc
REN Micro*.exe MobSvcInstaller.exe
REM ==================================================

REM ==== Extract the installer =======================
MobSvcInstaller.exe /q /x:%Temp%\MobSvc\Extracted
REM ==== Wait 10s for extraction to complete =========
TIMEOUT /t 10
REM =================================================

REM ==== Perform installation =======================
REM =================================================

CD %Temp%\MobSvc\Extracted
whoami >> C:\Temp\logfile.log
SET PRODKEY=HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
REG QUERY %PRODKEY%\{275197FC-14FD-4560-A5EB-38217F80CBD1}
IF NOT %ERRORLEVEL% EQU 0 (
    echo "Product is not installed. Goto INSTALL." >> C:\Temp\logfile.log
    GOTO :INSTALL
) ELSE (
    echo "Product is installed." >> C:\Temp\logfile.log

    echo "Checking for Post-install action status." >> C:\Temp\logfile.log
    GOTO :POSTINSTALLCHECK
)

:POSTINSTALLCHECK
    REG QUERY "HKLM\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\5" /v "PostInstallActions" | Find "Succeeded"
    If %ERRORLEVEL% EQU 0 (
        echo "Post-install actions succeeded. Checking for Configuration status." >> C:\Temp\logfile.log
        GOTO :CONFIGURATIONCHECK
    ) ELSE (
        echo "Post-install actions didn't succeed. Goto INSTALL." >> C:\Temp\logfile.log
        GOTO :INSTALL
    )

:CONFIGURATIONCHECK
    REG QUERY "HKLM\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\5" /v "AgentConfigurationStatus" | Find "Succeeded"
    If %ERRORLEVEL% EQU 0 (
        echo "Configuration has succeeded. Goto UPGRADE." >> C:\Temp\logfile.log
        GOTO :UPGRADE
    ) ELSE (
        echo "Configuration didn't succeed. Goto CONFIGURE." >> C:\Temp\logfile.log
        GOTO :CONFIGURE
    )


:INSTALL
    echo "Perform installation." >> C:\Temp\logfile.log
    UnifiedAgent.exe /Role MS /InstallLocation "C:\Program Files (x86)\Microsoft Azure Site Recovery" /Platform "VmWare" /Silent
    IF %ERRORLEVEL% EQU 0 (
        echo "Installation has succeeded." >> C:\Temp\logfile.log
        (GOTO :CONFIGURE)
    ) ELSE (
        echo "Installation has failed." >> C:\Temp\logfile.log
        GOTO :ENDSCRIPT
    )

:CONFIGURE
    echo "Perform configuration." >> C:\Temp\logfile.log
    cd "C:\Program Files (x86)\Microsoft Azure Site Recovery\agent"
    UnifiedAgentConfigurator.exe  /CSEndPoint "[CSIP]" /PassphraseFilePath %Temp%\MobSvc\MobSvc.passphrase
    IF %ERRORLEVEL% EQU 0 (
        echo "Configuration has succeeded." >> C:\Temp\logfile.log
    ) ELSE (
        echo "Configuration has failed." >> C:\Temp\logfile.log
    )
    GOTO :ENDSCRIPT

:UPGRADE
    echo "Perform upgrade." >> C:\Temp\logfile.log
    UnifiedAgent.exe /Platform "VmWare" /Silent
    IF %ERRORLEVEL% EQU 0 (
        echo "Upgrade has succeeded." >> C:\Temp\logfile.log
    ) ELSE (
        echo "Upgrade has failed." >> C:\Temp\logfile.log
    )
    GOTO :ENDSCRIPT

:ENDSCRIPT
    echo "End of script." >> C:\Temp\logfile.log
```

### <a name="copy-code-to-the-linux-folder"></a>Копирование кода в папку Linux

Скопируйте следующий код:

- Сохраните код в папке _Каталог mobsvclinux_ как _install_linux. sh_.
- Замените `[CSIP]` заполнители в этом скрипте фактическими значениями IP-адреса сервера конфигурации.
- Скрипт поддерживает новые установки агента службы Mobility Service и обновления уже установленных агентов.

```Bash
#!/usr/bin/env bash

rm -rf /tmp/MobSvc
mkdir -p /tmp/MobSvc
INSTALL_DIR='/usr/local/ASR'
VX_VERSION_FILE='/usr/local/.vx_version'

echo "=============================" >> /tmp/MobSvc/sccm.log
echo `date` >> /tmp/MobSvc/sccm.log
echo "=============================" >> /tmp/MobSvc/sccm.log

if [ -f /etc/oracle-release ] && [ -f /etc/redhat-release ]; then
    if grep -q 'Oracle Linux Server release 6.*' /etc/oracle-release; then
        if uname -a | grep -q x86_64; then
            OS="OL6-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *OL6*.tar.gz /tmp/MobSvc
        fi
    fi
elif [ -f /etc/redhat-release ]; then
    if grep -q 'Red Hat Enterprise Linux Server release 6.* (Santiago)' /etc/redhat-release || \
        grep -q 'CentOS Linux release 6.* (Final)' /etc/redhat-release || \
        grep -q 'CentOS release 6.* (Final)' /etc/redhat-release; then
        if uname -a | grep -q x86_64; then
            OS="RHEL6-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *RHEL6*.tar.gz /tmp/MobSvc
        fi
    elif grep -q 'Red Hat Enterprise Linux Server release 7.* (Maipo)' /etc/redhat-release || \
        grep -q 'CentOS Linux release 7.* (Core)' /etc/redhat-release; then
        if uname -a | grep -q x86_64; then
            OS="RHEL7-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *RHEL7*.tar.gz /tmp/MobSvc
                fi
    fi
elif [ -f /etc/SuSE-release ] && grep -q 'VERSION = 11' /etc/SuSE-release; then
    if grep -q "SUSE Linux Enterprise Server 11" /etc/SuSE-release && grep -q 'PATCHLEVEL = 3' /etc/SuSE-release; then
        if uname -a | grep -q x86_64; then
            OS="SLES11-SP3-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *SLES11-SP3*.tar.gz /tmp/MobSvc
        fi
    elif grep -q "SUSE Linux Enterprise Server 11" /etc/SuSE-release && grep -q 'PATCHLEVEL = 4' /etc/SuSE-release; then
        if uname -a | grep -q x86_64; then
            OS="SLES11-SP4-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *SLES11-SP4*.tar.gz /tmp/MobSvc
        fi
    fi
elif [ -f /etc/lsb-release ] ; then
    if grep -q 'DISTRIB_RELEASE=14.04' /etc/lsb-release ; then
       if uname -a | grep -q x86_64; then
           OS="UBUNTU-14.04-64"
           echo $OS >> /tmp/MobSvc/sccm.log
           cp *UBUNTU-14*.tar.gz /tmp/MobSvc
       fi
    fi
else
    exit 1
fi

if [ -z "$OS" ]; then
    exit 1
fi

Install()
{
    echo "Perform Installation." >> /tmp/MobSvc/sccm.log
    ./install -q -d ${INSTALL_DIR} -r MS -v VmWare
    RET_VAL=$?
    echo "Installation Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Installation has succeeded. Proceed to configuration." >> /tmp/MobSvc/sccm.log
        Configure
    else
        echo "Installation has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

Configure()
{
    echo "Perform configuration." >> /tmp/MobSvc/sccm.log
    ${INSTALL_DIR}/Vx/bin/UnifiedAgentConfigurator.sh -i [CSIP] -P MobSvc.passphrase
    RET_VAL=$?
    echo "Configuration Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Configuration has succeeded." >> /tmp/MobSvc/sccm.log
    else
        echo "Configuration has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

Upgrade()
{
    echo "Perform Upgrade." >> /tmp/MobSvc/sccm.log
    ./install -q -v VmWare
    RET_VAL=$?
    echo "Upgrade Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Upgrade has succeeded." >> /tmp/MobSvc/sccm.log
    else
        echo "Upgrade has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

cp MobSvc.passphrase /tmp/MobSvc
cd /tmp/MobSvc

tar -zxvf *.tar.gz

if [ -e ${VX_VERSION_FILE} ]; then
    echo "${VX_VERSION_FILE} exists. Checking for configuration status." >> /tmp/MobSvc/sccm.log
    agent_configuration=$(grep ^AGENT_CONFIGURATION_STATUS "${VX_VERSION_FILE}" | cut -d"=" -f2 | tr -d " ")
    echo "agent_configuration=$agent_configuration" >> /tmp/MobSvc/sccm.log
     if [ "$agent_configuration" == "Succeeded" ]; then
        echo "Agent is already configured. Proceed to Upgrade." >> /tmp/MobSvc/sccm.log
        Upgrade
    else
        echo "Agent is not configured. Proceed to Configure." >> /tmp/MobSvc/sccm.log
        Configure
    fi
else
    Install
fi

cd /tmp

```

### <a name="create-a-package"></a>Создание пакета

1. Войдите в консоль Configuration Manager и последовательно выберите **Библиотека программного обеспечения**  >  **Управление приложениями**  >  **пакеты**.
1. Щелкните правой кнопкой мыши **пакеты**  >  **создать пакет**.
1. Укажите сведения о пакете, включая имя, описание, изготовителя, язык и версию.
1. Выберите **Этот пакет содержит исходные файлы**.
1. Нажмите кнопку **Обзор** и выберите общую сетевую папку, содержащую соответствующий установщик (_Каталог mobsvcwindows_ или _Каталог mobsvclinux_). Затем нажмите кнопку **Далее**.

   ![Снимок экрана с мастером создания пакета и программы](./media/vmware-azure-mobility-install-configuration-mgr/create_sccm_package.png)

1. На странице **выберите тип программы** , которую необходимо создать, выберите **Стандартная программа**  >  **Далее**.

   ![Снимок экрана мастера создания пакетов и программ, который показывает стандартный параметр программы.](./media/vmware-azure-mobility-install-configuration-mgr/sccm-standard-program.png)

1. На странице **Укажите сведения об этой стандартной программе** укажите следующие значения:

    **Параметр** | **Значение Windows** | **Значение Linux**
    --- | --- | ---
    **имя**; | Установка Microsoft Azure Mobility Service (Windows) | Установите Microsoft Azure Mobility Service (Linux).
    **командная строка;** | install.bat | ./install_linux.sh
    **Программа может выполняться** | Независимо от входа пользователя в систему | Независимо от входа пользователя в систему
    **Другие параметры** | Использовать параметр по умолчанию | Использовать параметр по умолчанию

   ![Снимок экрана, на котором показаны сведения, которые можно указать для стандартной программы.](./media/vmware-azure-mobility-install-configuration-mgr/sccm-program-properties.png)

1. В **поле Укажите требования для этой стандартной программы** выполните следующие действия.

   - Для компьютеров Windows выберите **Эта программа может запускаться только на указанных платформах**. Затем выберите [Поддерживаемые операционные системы Windows](vmware-physical-azure-support-matrix.md#replicated-machines) и нажмите кнопку **Далее**.
   - Для компьютеров Linux выберите **эту программу можно запускать на любой платформе**. Выберите **Далее**.

1. Завершите мастер.

### <a name="deploy-the-package"></a>Развертывание пакета

1. В консоли Configuration Manager щелкните пакет правой кнопкой мыши и выберите команду **распространить содержимое**.

   ![Снимок экрана с консолью Configuration Manager](./media/vmware-azure-mobility-install-configuration-mgr/sccm_distribute.png)

1. Выберите Точки распространения, на которые следует скопировать эти пакеты. [Подробнее](/sccm/core/servers/deploy/configure/install-and-configure-distribution-points).
1. Завершите работу мастера. Теперь пакет будет реплицироваться на выбранные точки распространения.
1. После завершения распространения пакета щелкните правой кнопкой мыши пакет, > **развернуть**.

   ![Снимок экрана консоли Configuration Manager, в которой отображается пункт меню "развернуть".](./media/vmware-azure-mobility-install-configuration-mgr/sccm_deploy.png)

1. Выберите созданную ранее коллекцию устройств Windows или Linux.
1. На странице **Укажите место назначения содержимого** выберите **точки распространения**.
1. В окне **Укажите параметры для управления развертыванием этого программного обеспечения** установите для параметра **назначение** значение **обязательно**.

   ![Снимок экрана с мастером развертывания программного обеспечения](./media/vmware-azure-mobility-install-configuration-mgr/sccm-deploy-select-purpose.png)

1. В поле **укажите расписание для этого развертывания** настройте расписание. [Подробнее](/sccm/apps/deploy-use/deploy-applications#bkmk_deploy-sched).

   - Служба Mobility Service устанавливается в соответствии с заданным вами расписанием.
   - Чтобы избежать лишних перезагрузок, запланируйте установку пакета на период ежемесячного обслуживания или обновления программного обеспечения.

1. На странице **точки распространения** настройте параметры и завершите работу мастера.
1. Мониторинг хода выполнения развертывания в консоли Configuration Manager. Перейдите к разделу **мониторинг**  >  **развертываний**  >  _\<your package name\>_ .

### <a name="uninstall-the-mobility-service"></a>Удаление службы Mobility Service

Вы можете создать Configuration Manager пакеты, чтобы удалить службу Mobility Service. Например, следующий сценарий удаляет службу Mobility Service:

```DOS
Time /t >> C:\logfile.log
REM ==================================================
REM ==== Check if Mob Svc is already installed =======
REM ==== If not installed no operation required ========
REM ==== Else run uninstall command =====================
REM ==== {275197FC-14FD-4560-A5EB-38217F80CBD1} is ====
REM ==== guid for Mob Svc Installer ====================
whoami >> C:\logfile.log
NET START | FIND "InMage Scout Application Service"
IF  %ERRORLEVEL% EQU 1 (GOTO :INSTALL) ELSE GOTO :UNINSTALL
:NOOPERATION
                echo "No Operation Required." >> c:\logfile.log
                GOTO :ENDSCRIPT
:UNINSTALL
                echo "Uninstall" >> C:\logfile.log
                MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1} /L+*V "C:\ProgramData\ASRSetupLogs\UnifiedAgentMSIUninstall.log"
:ENDSCRIPT
```

## <a name="next-steps"></a>Дальнейшие действия

[Включите репликацию](vmware-azure-enable-replication.md) для виртуальных машин.
