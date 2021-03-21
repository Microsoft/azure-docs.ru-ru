---
title: Устранение неполадок с агентом обновления Linux в службе автоматизации Azure
description: В этой статье содержатся сведения об устранении неполадок с агентом обновления Linux в Управлении обновлениями.
services: automation
ms.date: 01/25/2021
ms.topic: troubleshooting
ms.subservice: update-management
ms.openlocfilehash: da7c0ea670b4c4201930ce5d0f01e7bd9d9835e9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "100581038"
---
# <a name="troubleshoot-linux-update-agent-issues"></a>Устранение неполадок с агентом обновления Linux

Существует множество причин, по которым компьютер в Управлении обновлениями может не отображаться как готовый к работе (работоспособный). Вы можете проверить работоспособность агента гибридной рабочей роли Runbook для Linux, чтобы определить первоначальную проблему. Далее приведены три состояния готовности компьютера.

* Готово: гибридная рабочая роль Runbook развернута и последний раз отображалась менее часа назад.
* Без подключения: гибридная рабочая роль Runbook развернута и последний раз отображалась больше часа назад.
* Не настроено: гибридная рабочая роль Runbook не найдена или не завершила развертывание.

> [!NOTE]
> Между отображением на портале Azure и текущим состоянием компьютера может быть небольшая задержка.

В этой статье описывается, как запустить средство устранения неполадок для компьютеров Azure на портале Azure и компьютеров вне Azure в [сценарии изолированной среды](#troubleshoot-offline).

> [!NOTE]
> В настоящее время сценарий устранения неполадок не направляет трафик через прокси-сервер, если он настроен.

## <a name="start-the-troubleshooter"></a>Запуск средства устранения неполадок

Для компьютеров Azure: на портале выберите ссылку **Устранение неполадок** в столбце **Готовность к обновлению агента**, после чего откроется страница устранения неполадок с агентом обновления. Для устранения неполадок со сторонними компьютерами перейдите к соответствующему разделу в этой статье. Инструкции по устранению неполадок на компьютерах, размещенных вне Azure, приведены в разделе "Автономное устранение неполадок".

![Страница со списком виртуальных машин](../media/update-agent-issues-linux/vm-list.png)

> [!NOTE]
> Проверка выполняется с запущенной виртуальной машиной. Если виртуальная машина не запущена, появится кнопка **Запустить виртуальную машину**.

На странице устранения неполадок с агентом обновления выберите **Run checks** (Выполнить проверки), чтобы запустить средство устранения неполадок. Средство устранения неполадок использует [команду "Выполнить"](../../virtual-machines/linux/run-command.md) для запуска на компьютере сценария, проверяющего зависимости. После того как средство устранения неполадок завершило свою работу, оно возвращает результаты проверок.

![Страница "Устранение неполадок"](../media/update-agent-issues-linux/troubleshoot-page.png)

После завершения проверок результаты отображаются в окне. Разделы о проверках предоставляют сведения о предназначении каждой проверки.

![Страница проверки агента обновлений](../media/update-agent-issues-linux/update-agent-checks.png)

## <a name="prerequisite-checks"></a>Проверка предварительных требований

### <a name="operating-system"></a>Операционная система

Проверка операционной системы обнаруживает, использует ли гибридная рабочая роль Runbook одну из приведенных ниже операционных систем.

|Операционная система  |Примечания  |
|---------|---------|
|CentOS 6 (x86 или x64) и 7 (x64).      | У агентов Linux должен быть доступ к репозиторию обновлений. Для исправления на основе классификации требуется, чтобы программа yum вернула данные безопасности, которых нет в CentOS без дополнительной настройки.         |
|Red Hat Enterprise 6 (x86 или x64) и 7 (x64).     | У агентов Linux должен быть доступ к репозиторию обновлений.        |
|SUSE Linux Enterprise Server 11 (x86 или x64) и 12 (x64)     | У агентов Linux должен быть доступ к репозиторию обновлений.        |
|Ubuntu 14.04 LTS, 16.04 LTS и 18.04 LTS (x86/x64)      |У агентов Linux должен быть доступ к репозиторию обновлений.         |

## <a name="monitoring-agent-service-health-checks"></a>Проверки работоспособности службы агента мониторинга

### <a name="log-analytics-agent"></a>Агент Log Analytics

Проверка позволяет убедиться, что агент Log Analytics для Linux установлен. Инструкции по его установке см. в разделе [Установка агента для Linux](../../azure-monitor/vm/quick-collect-linux-computer.md#install-the-agent-for-linux).

### <a name="log-analytics-agent-status"></a>Состояние агента Log Analytics

Проверка гарантирует, что агент Log Analytics для Linux выполняется. Если агент не выполняется, можно запустить следующую команду, чтобы попытаться перезапустить его. Дополнительные сведения об устранении неполадок агента см. в разделе об [устранении неполадок с гибридными рабочими ролями Runbook для Linux](hybrid-runbook-worker.md#linux).

```bash
sudo /opt/microsoft/omsagent/bin/service_control restart
```

### <a name="multihoming"></a>Множественная адресация

Эта проверка определяет отправление отчетов агентом в несколько рабочих областей. Управление обновлениями не поддерживает множественную адресацию.

### <a name="hybrid-runbook-worker"></a>Гибридная рабочая роль Runbook

Эта проверка позволяет убедиться, что в агенте Log Analytics для Linux установлен пакет гибридной рабочей роли Runbook. Этот пакет необходим для работы службы "Управление обновлениями". Чтобы узнать больше, ознакомьтесь с разделом [Агент Службы анализа журналов для Linux не работает](hybrid-runbook-worker.md#oms-agent-not-running).

Управление обновлениями скачивает пакеты гибридной рабочей роли Runbook с конечной точки операций. Следовательно, если гибридная рабочая роль Runbook не выполняется и проверка [конечной точки](#operations-endpoint) завершается сбоем, то обновление тоже может завершиться сбоем.

### <a name="hybrid-runbook-worker-status"></a>Состояние гибридной рабочей роли Runbook

Эта проверка гарантирует, что гибридная рабочая роль Runbook выполняется на компьютере. Процессы в примере ниже должны отображаться, если гибридная рабочая роль Runbook работает правильно.

```bash
nxautom+   8567      1  0 14:45 ?        00:00:00 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/main.py /var/opt/microsoft/omsagent/state/automationworker/oms.conf rworkspace:<workspaceId> <Linux hybrid worker version>
nxautom+   8593      1  0 14:45 ?        00:00:02 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/hybridworker.py /var/opt/microsoft/omsagent/state/automationworker/worker.conf managed rworkspace:<workspaceId> rversion:<Linux hybrid worker version>
nxautom+   8595      1  0 14:45 ?        00:00:02 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/hybridworker.py /var/opt/microsoft/omsagent/<workspaceId>/state/automationworker/diy/worker.conf managed rworkspace:<workspaceId> rversion:<Linux hybrid worker version>
```

## <a name="connectivity-checks"></a>Проверка подключения

### <a name="general-internet-connectivity"></a>Общее подключения к Интернету

Проверка позволяет убедиться, что у компьютера есть доступ к Интернету.

### <a name="registration-endpoint"></a>Конечная точка регистрации

Эта проверка позволяет определить, может ли гибридная рабочая роль Runbook правильно взаимодействовать со службой автоматизации Azure в рабочей области Log Analytics.

В конфигурации прокси-сервера и брандмауэра необходимо разрешить агенту гибридной рабочей роли Runbook взаимодействовать с конечной точкой регистрации. Список адресов и портов, которые необходимо открыть, приведен в разделе [Настройка сети](../automation-hybrid-runbook-worker.md#network-planning).

### <a name="operations-endpoint"></a>Конечная точка операций

Эта проверка позволяет определить, может ли агент Log Analytics правильно взаимодействовать со службой Job Runtime Data.

В конфигурации прокси-сервера и брандмауэра необходимо разрешить агенту гибридной рабочей роли Runbook взаимодействовать со службой Job Runtime Data. Список адресов и портов, которые необходимо открыть, приведен в разделе [Настройка сети](../automation-hybrid-runbook-worker.md#network-planning).

### <a name="log-analytics-endpoint-1"></a>Конечная точка 1 Log Analytics

Проверка подтверждает, что у компьютера есть доступ к конечным точкам, в которых нуждается агент Log Analytics.

### <a name="log-analytics-endpoint-2"></a>Конечная точка 2 Log Analytics

Проверка подтверждает, что у компьютера есть доступ к конечным точкам, в которых нуждается агент Log Analytics.

### <a name="log-analytics-endpoint-3"></a>Конечная точка 3 Log Analytics

Проверка подтверждает, что у компьютера есть доступ к конечным точкам, в которых нуждается агент Log Analytics.

## <a name="troubleshoot-offline"></a><a name="troubleshoot-offline"></a>Автономное устранение неполадок

Средство устранения неполадок можно использовать в гибридной рабочей роли в автономном режиме, запустив сценарий локально. Сценарий Python, [UM_Linux_Troubleshooter_Offline. Корректировка](https://github.com/Azure/updatemanagement/blob/main/UM_Linux_Troubleshooter_Offline.py), можно найти на сайте GitHub. Ниже показан пример выходных данных этого сценария:

```output
Debug: Machine Information:   Static hostname: LinuxVM2
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 00000000000000000000000000000000
           Boot ID: 00000000000000000000000000000000
    Virtualization: microsoft
  Operating System: Ubuntu 16.04.5 LTS
            Kernel: Linux 4.15.0-1025-azure
      Architecture: x86-64


Passed: Operating system version is supported

Passed: Microsoft Monitoring agent is installed

Debug: omsadmin.conf file contents:
        WORKSPACE_ID=00000000-0000-0000-0000-000000000000
        AGENT_GUID=00000000-0000-0000-0000-000000000000
        LOG_FACILITY=local0
        CERTIFICATE_UPDATE_ENDPOINT=https://00000000-0000-0000-0000-000000000000.oms.opinsights.azure.com/ConfigurationService.Svc/RenewCertificate
        URL_TLD=opinsights.azure.com
        DSC_ENDPOINT=https://scus-agentservice-prod-1.azure-automation.net/Accou            nts/00000000-0000-0000-0000-000000000000/Nodes\(AgentId='00000000-0000-0000-0000-000000000000'\)
        OMS_ENDPOINT=https://00000000-0000-0000-0000-000000000000.ods.opinsights            .azure.com/OperationalData.svc/PostJsonDataItems
        AZURE_RESOURCE_ID=/subscriptions/00000000-0000-0000-0000-000000000000/re            sourcegroups/myresourcegroup/providers/microsoft.compute/virtualmachines/linuxvm            2
        OMSCLOUD_ID=0000-0000-0000-0000-0000-0000-00
        UUID=00000000-0000-0000-0000-000000000000


Passed: Microsoft Monitoring agent is running

Passed: Machine registered with log analytics workspace:['00000000-0000-0000-0000-000000000000']

Passed: Hybrid worker package is present

Passed: Hybrid worker is running

Passed: Machine is connected to internet

Passed: TCP test for {scus-agentservice-prod-1.azure-automation.net} (port 443)             succeeded

Passed: TCP test for {eus2-jobruntimedata-prod-su1.azure-automation.net} (port 4            43) succeeded

Passed: TCP test for {00000000-0000-0000-0000-000000000000.ods.opinsights.azure.            com} (port 443) succeeded

Passed: TCP test for {00000000-0000-0000-0000-000000000000.oms.opinsights.azure.            com} (port 443) succeeded

Passed: TCP test for {ods.systemcenteradvisor.com} (port 443) succeeded

```

## <a name="next-steps"></a>Дальнейшие действия

[Устранение неполадок с гибридной рабочей ролью Runbook](hybrid-runbook-worker.md).
