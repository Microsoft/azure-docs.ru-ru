---
title: Управление агентом мобильности для VMware или физических серверов с помощью Azure Site Recovery
description: Управление агентом службы Mobility Service для аварийного восстановления виртуальных машин VMware и физических серверов в Azure с помощью службы Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: ramamill
ms.openlocfilehash: d921bddf90c415cb244e2cc9ad98354392a537ee
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90530171"
---
# <a name="manage-the-mobility-agent"></a>Управление агентом службы "Мобильность" 

Агент мобильности настраивается на сервере при использовании Azure Site Recovery для аварийного восстановления виртуальных машин VMware и физических серверов в Azure. Агент мобильности координирует обмен данными между защищенным компьютером, сервером конфигурации и сервером обработки масштабирования и управляет репликацией данных. В этой статье перечислены распространенные задачи по управлению агентом мобильности после его развертывания.

>[!TIP]
>Чтобы скачать установщик для определенной ОС или дистрибутив Linux, см. Руководство [здесь](vmware-physical-mobility-service-overview.md#locate-installer-files). Для автоматического обновления с портала не требуется загружать установщик. [ASR автоматически извлекает установщик с сервера конфигурации и обновляет агент](#update-mobility-service-from-azure-portal).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="update-mobility-service-from-azure-portal"></a>Обновление службы Mobility Service с портал Azure

1. Прежде чем приступать к обновлению службы Mobility Service на защищенных компьютерах, убедитесь, что серверы конфигурации, обработки масштабирования и все основные целевые серверы, входящие в состав вашего развертывания, обновлены.
    1. Начиная с версии 9,36, для SUSE Linux Enterprise Server 11 SP3, RHEL 5, CentOS 5, Debian 7 убедитесь, что последний установщик [доступен на сервере конфигурации и на сервере обработки масштабирования](vmware-physical-mobility-service-overview.md#download-latest-mobility-agent-installer-for-suse-11-sp3-rhel-5-debian-7-server).
1. На портале откройте хранилище и выберите **Реплицированные элементы**.
1. Если сервер конфигурации последней версии, отобразится уведомление: "Доступно обновление для агента репликации Site Recovery. Щелкните, чтобы установить его."

     ![Окно "Реплицированные элементы"](./media/vmware-azure-install-mobility-service/replicated-item-notif.png)

4. Щелкните уведомление и в разделе **Обновление агента** выберите компьютеры, на которых нужно обновить службу Mobility Service. Нажмите кнопку **ОК**.

     ![Список виртуальных машин "Реплицированные элементы"](./media/vmware-azure-install-mobility-service/update-okpng.png)

5. Для каждого выбранного компьютера будет запущено задание обновления службы Mobility Service. Агент мобильности обновлен до версии сервера конфигурации. Например, если сервер конфигурации работает в версии 9,33, агент мобильности на защищенной виртуальной машине также обновляется до версии 9,33.

## <a name="update-mobility-service-through-powershell-script-on-windows-server"></a>Обновление службы Mobility Service с помощью сценария PowerShell в Windows Server

Прежде чем приступать к обновлению службы Mobility Service на защищенных компьютерах, убедитесь, что серверы конфигурации, обработки масштабирования и все основные целевые серверы, входящие в состав вашего развертывания, обновлены.

Используйте следующий скрипт для обновления службы Mobility Service на сервере с помощью командлета PowerShell.

```azurepowershell
Update-AzRecoveryServicesAsrMobilityService -ReplicationProtectedItem $rpi -Account $fabric.fabricSpecificDetails.RunAsAccounts[0]
```

## <a name="update-mobility-service-manually-on-each-protected-server"></a>Обновление службы Mobility Service вручную на каждом защищенном сервере

1. Прежде чем приступать к обновлению службы Mobility Service на защищенных компьютерах, убедитесь, что серверы конфигурации, обработки масштабирования и все основные целевые серверы, входящие в состав вашего развертывания, обновлены.

2. [Откройте установщик агента](vmware-physical-mobility-service-overview.md#locate-installer-files) на основе операционной системы сервера.

>[!IMPORTANT]
> Если вы реплицируете виртуальную машину Azure IaaS из одного региона Azure в другой, не используйте этот метод. Сведения обо всех доступных вариантах см. в [нашем руководстве](azure-to-azure-autoupdate.md) .

3. Скопируйте файл установки на защищенный компьютер и запустите его, чтобы обновить агент мобильности.

## <a name="update-account-used-for-push-installation-of-mobility-service"></a>Обновление учетной записи, используемой для принудительной установки службы Mobility Service

При развертывании Site Recovery, чтобы включить принудительную установку Mobility Service, вы указали учетную запись, которую сервер обработки Site Recovery использует для доступа к компьютерам и установки службы (если на компьютере включена репликация). Если вы хотите обновить учетные данные для этой учетной записи, следуйте [этим инструкциям](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation).

## <a name="uninstall-mobility-service"></a>Удаление службы Mobility Service

### <a name="on-a-windows-machine"></a>На компьютере с Windows

Выполните удаление из пользовательского интерфейса или командной строки.

- **Из пользовательского интерфейса**: на панели управления компьютера выберите **Программы**. Выберите **Microsoft Azure Site Recovery удаление службы Mobility Service/главного целевого сервера**  >  .
- **Из командной строки**: откройте окно командной строки с правами администратора компьютера. Выполните следующую команду. 
    ```
    MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1} /L+*V "C:\ProgramData\ASRSetupLogs\UnifiedAgentMSIUninstall.log"
    ```

### <a name="on-a-linux-machine"></a>На компьютере с Linux
1. На компьютере с Linux войдите в систему как **привилегированный** пользователь.
2. В терминале перейдите по адресу/usr/local/ASR.
3. Выполните следующую команду.
    ```
    uninstall.sh -Y
   ```
   
## <a name="install-site-recovery-vss-provider-on-source-machine"></a>Установка Site Recovery поставщика VSS на исходном компьютере

Для создания точек согласованности приложений на исходном компьютере требуется поставщик VSS Azure Site Recovery. Если установка поставщика завершилась неудачно с помощью принудительной установки, следуйте приведенным ниже указаниям, чтобы установить его вручную.

1. Откройте окно "Администратор cmd".
2. Перейдите к папке установки службы Mobility Service. (Например, C:\Program Files (x86) \Microsoft Azure site Рековери\ажент)
3. Запустите скрипт InMageVSSProvider_Uninstall. cmd. Служба будет удалена, если она уже существует.
4. Выполните скрипт InMageVSSProvider_Install. cmd, чтобы установить поставщик VSS вручную.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка аварийного восстановления для виртуальных машин VMware](vmware-azure-tutorial.md)
- [Настройка аварийного восстановления в Azure для локальных физических серверов](physical-azure-disaster-recovery.md)
