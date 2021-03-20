---
title: Удаление серверов и отключение защиты | Документация Майкрософт
description: В этой статье описывается, как отменить регистрацию серверов в хранилище Site Recovery, а также отключить защиту для виртуальных машин и физических серверов.
author: Sharmistha-Rai
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 06/18/2019
ms.author: sharrai
ms.openlocfilehash: 1356deabd13db1dd2f29ac1b1f088db2120353fe
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101738860"
---
# <a name="remove-servers-and-disable-protection"></a>Удаление серверов и отключение защиты

В этой статье описывается, как отменить регистрацию серверов в хранилище служб восстановления и как отключить защиту для виртуальных машин, защищенных с помощью Site Recovery.


## <a name="unregister-a--configuration-server"></a>Отмена регистрации сервера конфигурации

При репликации виртуальных машин VMware или физических серверов Windows или Linux в Azure можно отменить регистрацию неподключенного сервера конфигурации в хранилище, используя следующие действия:

1. [Отключите защиту для виртуальных машин](#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure).
2. [Отмена связи или удаление](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) политик репликации.
3. [Удаление сервера конфигурации](vmware-azure-manage-configuration-server.md#delete-or-unregister-a-configuration-server)

## <a name="unregister-a-vmm-server"></a>Отмена регистрации сервера VMM

1. Остановите репликацию виртуальных машин в облаках на сервере VMM, который нужно удалить.
2. Удалите все сопоставления сетей, используемые облаками на этом сервере. В **Site Recovery инфраструктура**  >  **для**  >  **сетевого сопоставления** VMM System Center щелкните правой кнопкой мыши сетевое сопоставление > **Удалить**.
3. Запишите идентификатор сервера VMM.
4. Отмените связь с политиками репликации для облаков на сервере VMM, который нужно удалить.  В **Site Recovery инфраструктура**  >  **для**  >   **политик репликации** VMM System Center дважды щелкните соответствующую политику. Щелкните облако правой кнопкой мыши и выберите **Disassociate** (Отменить связь).
5. Удалите сервер VMM или его запущенный узел. В **Site Recovery инфраструктуре**  >  **для**  >  **серверов VMM** System Center VMM щелкните правой кнопкой мыши сервер > **Удалить**.
6. Если сервер VMM был в отключенном состоянии, то скачайте и запустите [сценарий очистки](/samples/browse/?redirectedfrom=TechNet-Gallery) на этом сервере VMM. Откройте PowerShell с параметром **Запуск от имени администратора** , чтобы изменить политику выполнения для области по умолчанию (LocalMachine). В скрипте укажите идентификатор сервера VMM, который требуется удалить. Скрипт удалит с сервера информацию о регистрации и связывании облаков.
5. Запустите сценарий очистки на сервере-получателе VMM.
6. Запустите скрипт очистки на любом другом пассивном узле кластера VMM, на котором установлен поставщик.
7. Вручную удалите поставщика на сервере VMM. При наличии кластера удалите его со всех узлов.
8. В случае репликации виртуальных машин в Azure нужно удалить агент служб восстановления Microsoft с узлов Hyper-V в удаленных облаках.

## <a name="unregister-a-hyper-v-host-in-a-hyper-v-site"></a>Отмена регистрации узла Hyper-V на сайте Hyper-V

Узлы Hyper-V, которые не управляются с помощью VMM, расположены на сайте Hyper-V. Удалите узел на сайте Hyper-V следующим образом:

1. Отключите репликацию для виртуальных машин Hyper-V, расположенных на узле.
2. Отмените политики для сайта Hyper-V. В **Site Recovery инфраструктуре**  >  **для политик репликации сайтов Hyper-V**  >   дважды щелкните соответствующую политику. Щелкните сайт правой кнопкой мыши и выберите **Отменить**.
3. Удалите узлы Hyper-V. В **Site Recovery инфраструктуре** узлов  >  Hyper-v **для сайтов** Hyper — щелкните  >  правой кнопкой мыши сервер > **Удалить**.
4. После того как все узлы на сайте Hyper-V будут удалены, удалите сайт Hyper-V. В **Site Recovery инфраструктуре** для сайтов Hyper-v сайтов на основе  >  **оболочки** щелкните  >  правой кнопкой мыши сайт > **Удалить**.
5. Если узел Hyper-V был **отключен**, то на каждом удаленном узле Hyper-V запустите приведенный ниже сценарий. Этот скрипт очистит параметры на сервере и отменит его регистрацию в хранилище.


```powershell
        pushd .
        try
        {
            $windowsIdentity=[System.Security.Principal.WindowsIdentity]::GetCurrent()
            $principal=new-object System.Security.Principal.WindowsPrincipal($windowsIdentity)
            $administrators=[System.Security.Principal.WindowsBuiltInRole]::Administrator
            $isAdmin=$principal.IsInRole($administrators)
            if (!$isAdmin)
            {
                "Please run the script as an administrator in elevated mode."
                $choice = Read-Host
                return;
            }

            $error.Clear()
            "This script will remove the old Azure Site Recovery Provider related properties. Do you want to continue (Y/N) ?"
            $choice =  Read-Host

            if (!($choice -eq 'Y' -or $choice -eq 'y'))
            {
            "Stopping cleanup."
            return;
            }

            $serviceName = "dra"
            $service = Get-Service -Name $serviceName
            if ($service.Status -eq "Running")
            {
                "Stopping the Azure Site Recovery service..."
                net stop $serviceName
            }

            $asrHivePath = "HKLM:\SOFTWARE\Microsoft\Azure Site Recovery"
            $registrationPath = $asrHivePath + '\Registration'
            $proxySettingsPath = $asrHivePath + '\ProxySettings'
            $draIdvalue = 'DraID'
            $idMgmtCloudContainerId='IdMgmtCloudContainerId'


            if (Test-Path $asrHivePath)
            {
                if (Test-Path $registrationPath)
                {
                    "Removing registration related registry keys."
                    Remove-Item -Recurse -Path $registrationPath
                }

                if (Test-Path $proxySettingsPath)
                {
                    "Removing proxy settings"
                    Remove-Item -Recurse -Path $proxySettingsPath
                }

                $regNode = Get-ItemProperty -Path $asrHivePath
                if($regNode.DraID -ne $null)
                {
                    "Removing DraId"
                    Remove-ItemProperty -Path $asrHivePath -Name $draIdValue
                }
                if($regNode.IdMgmtCloudContainerId -ne $null)
                {
                    "Removing IdMgmtCloudContainerId"
                    Remove-ItemProperty -Path $asrHivePath -Name $idMgmtCloudContainerId
                }
                "Registry keys removed."
            }

            # First retrieve all the certificates to be deleted
            $ASRcerts = Get-ChildItem -Path cert:\localmachine\my | where-object {$_.friendlyname.startswith('ASR_SRSAUTH_CERT_KEY_CONTAINER') -or $_.friendlyname.startswith('ASR_HYPER_V_HOST_CERT_KEY_CONTAINER')}
            # Open a cert store object
            $store = New-Object System.Security.Cryptography.X509Certificates.X509Store("My","LocalMachine")
            $store.Open('ReadWrite')
            # Delete the certs
            "Removing all related certificates"
            foreach ($cert in $ASRcerts)
            {
                $store.Remove($cert)
            }
        }catch
        {
            [system.exception]
            Write-Host "Error occurred" -ForegroundColor "Red"
            $error[0]
            Write-Host "FAILED" -ForegroundColor "Red"
        }
        popd
```


## <a name="disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure"></a>Отключение защиты для виртуальной машины VMware или физического сервера (из VMware в Azure)

1. В поле **защищенные элементы**  >  **реплицированные** элементы щелкните правой кнопкой мыши компьютер, > **отключить репликацию**.
2. На странице **Отключение репликации** выберите один из следующих параметров:
    - **Отключить репликацию и удалить (рекомендуется)** — этот параметр позволяет удалить реплицированный элемент из Azure Site Recovery и репликация для компьютера остановлена. Конфигурация репликации на сервере конфигурации очищается, и выставление счетов за использование Site Recovery для этого защищенного сервера останавливается. Обратите внимание на то, что этот параметр может использоваться, только если сервер конфигурации находится в подключенном состоянии.
    - **Удалить**. Этот параметр должен использоваться только в том случае, если исходная среда удалена или недоступна (не подключена). Он позволяет удалить реплицированный элемент из Azure Site Recovery (выставление счетов останавливается). Конфигурация репликации на сервере конфигурации **не будет** очищена. 

> [!NOTE]
> При использовании обоих параметров служба Mobility Service не будет удалена с защищенных серверов. Ее необходимо удалить вручную. Если вы планируете повторно защитить сервер, используя тот же сервер конфигурации, то удаление службы Mobility Service можно пропустить.

> [!NOTE]
> Если вы уже выполнили отработку отказа для виртуальной машины и она запущена в Azure, обратите внимание, что отключение защиты не удаляет и не влияет на виртуальную машину, на которую выполнен переход.
## <a name="disable-protection-for-a-azure-vm-azure-to-azure"></a>Отключение защиты для виртуальной машины Azure (Azure в Azure)

-  В поле **защищенные элементы**  >  **реплицированные** элементы щелкните правой кнопкой мыши компьютер, > **отключить репликацию**.
> [!NOTE]
> Служба мобильности не будет удалена с защищенных серверов, вам необходимо удалить ее вручную. Если вы планируете повторно защитить сервер, то удаление службы мобильности можно пропустить.

## <a name="disable-protection-for-a-hyper-v-virtual-machine-hyper-v-to-azure"></a>Отключение защиты виртуальной машины Hyper-V (из Hyper-V в Azure)

> [!NOTE]
> Если вы реплицируете виртуальные машины Hyper-V в Azure, не используя сервер VMM, сделайте следующее: Если выполняется репликация виртуальных машин с помощью **System Center VMM в Azure**, следуйте инструкциям в разделе "Отключение защиты для виртуальной машины Hyper-V, реплицируемой в Azure, при использовании сценария репликации из System Center VMM в Azure".

1. В поле **защищенные элементы**  >  **реплицированные** элементы щелкните правой кнопкой мыши компьютер, > **отключить репликацию**.
2. В разделе **Отключение репликации** можно выбрать следующие параметры:
   - **Отключить репликацию и удалить (рекомендуется)**. Этот параметр позволяет удалить реплицированный элемент из Azure Site Recovery и остановить репликацию на компьютере. Конфигурация репликации на локальной виртуальной машине будет очищена, и выставление счетов за использование Site Recovery для этого защищенного сервера будет остановлено.
   - **Удалить**. Этот параметр должен использоваться только в том случае, если исходная среда удалена или недоступна (не подключена). Он позволяет удалить реплицированный элемент из Azure Site Recovery (выставление счетов останавливается). Конфигурация репликации на локальной виртуальной машине **не будет** очищена. 

    > [!NOTE]
    > Если вы выбрали параметр **Удалить**, то выполните приведенный ниже набор сценариев для очистки настроек репликации локального сервера Hyper-V.

    > [!NOTE]
    > Если вы уже выполнили отработку отказа для виртуальной машины и она запущена в Azure, обратите внимание, что отключение защиты не удаляет и не влияет на виртуальную машину, на которую выполнен переход.

1. Удалите репликацию для виртуальной машины на исходном сервере узла Hyper-V. Замените SQLVM1 именем виртуальной машины и запустите сценарий с правами администратора PowerShell.

```powershell
    $vmName = "SQLVM1"
    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'"
    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"
    $replicationService.RemoveReplicationRelationship($vm.__PATH)
```

## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario"></a>Отключение защиты для виртуальной машины Hyper-V, реплицируемой в Azure, при использовании сценария репликации из System Center VMM в Azure

1. В поле **защищенные элементы**  >  **реплицированные** элементы щелкните правой кнопкой мыши компьютер, > **отключить репликацию**.
2. На странице **Отключение репликации** выберите один из следующих параметров:

   - **Отключить репликацию и удалить (рекомендуется)**. Этот параметр позволяет удалить реплицированный элемент из Azure Site Recovery и остановить репликацию на компьютере. Конфигурация репликации на локальной виртуальной машине очищается, и выставление счетов за использование Site Recovery для этого защищенного сервера останавливается.
   - **Удалить**. Этот параметр должен использоваться только в том случае, если исходная среда удалена или недоступна (не подключена). Он позволяет удалить реплицированный элемент из Azure Site Recovery (выставление счетов останавливается). Конфигурация репликации на локальной виртуальной машине **не будет** очищена. 

     > [!NOTE]
     > Если вы выбрали параметр **Удалить**, то выполните приведенные ниже сценарии для очистки настроек репликации локального сервера Hyper-V.
3. Запустите этот сценарий на исходном сервере VMM с помощью PowerShell (необходимы права администратора) из консоли VMM. Замените заполнитель **SQLVM1** именем виртуальной машины.

    ```powershell
    $vm = get-scvirtualmachine -Name "SQLVM1"
    Set-SCVirtualMachine -VM $vm -ClearDRProtection
    ```

4. После выполнения описанных выше действий параметры репликации на сервере VMM будут удалены. Чтобы остановить репликацию для виртуальной машины, работающей на сервере узлов Hyper-V, запустите приведенный ниже скрипт. Замените SQLVM1 именем виртуальной машины и host01.contoso.com именем сервера узлов Hyper-V.

```powershell
    $vmName = "SQLVM1"
    $hostName  = "host01.contoso.com"
    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'" -computername $hostName
    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"  -computername $hostName
    $replicationService.RemoveReplicationRelationship($vm.__PATH)
```

## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-secondary-vmm-server-using-the-system-center-vmm-to-vmm-scenario"></a>Отключение защиты виртуальной машины Hyper-V, реплицируемой на сервер-получатель VMM, при использовании сценария репликации из System Center VMM в VMM

1. В поле **защищенные элементы**  >  **реплицированные** элементы щелкните правой кнопкой мыши компьютер, > **отключить репликацию**.
2. На странице **Отключение репликации** выберите один из следующих параметров:

   - **Отключить репликацию и удалить (рекомендуется)**. Этот параметр позволяет удалить реплицированный элемент из Azure Site Recovery и остановить репликацию на компьютере. Конфигурация репликации на локальной виртуальной машине очищается, и выставление счетов за использование Site Recovery для этого защищенного сервера останавливается.
   - **Удалить**. Этот параметр должен использоваться только в том случае, если исходная среда удалена или недоступна (не подключена). Он позволяет удалить реплицированный элемент из Azure Site Recovery (выставление счетов останавливается). Конфигурация репликации на локальной виртуальной машине **не будет** очищена. Выполните приведенный ниже набор сценариев для очистки параметров репликации локальных виртуальных машин.
     > [!NOTE]
     > Если вы выбрали параметр **Удалить**, то выполните приведенные ниже сценарии для очистки настроек репликации локального сервера Hyper-V.

3. Запустите этот сценарий на исходном сервере VMM с помощью PowerShell (необходимы права администратора) из консоли VMM. Замените заполнитель **SQLVM1** именем виртуальной машины.

    ```powershell
    $vm = get-scvirtualmachine -Name "SQLVM1"
    Set-SCVirtualMachine -VM $vm -ClearDRProtection
    ```

4. На сервере-получателе VMM выполните следующий сценарий, чтобы удалить параметры дополнительной виртуальной машины.

    ```powershell
    $vm = get-scvirtualmachine -Name "SQLVM1"
    Remove-SCVirtualMachine -VM $vm -Force
    ```

5. На дополнительном сервере VMM обновите виртуальные машины на сервере узлов Hyper-V, чтобы дополнительная виртуальная машина была заново обнаружена в консоли VMM.
6. После выполнения описанных выше действий параметры репликации будут удалены на сервере VMM. Если вы хотите остановить репликацию для виртуальной машины, запустите приведенный ниже скрипт на основной и дополнительной виртуальной машинах. Замените SQLVM1 именем виртуальной машины.

    ```powershell
    Remove-VMReplication –VMName "SQLVM1"
    ```