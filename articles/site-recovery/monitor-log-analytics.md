---
title: Мониторинг Azure Site Recovery с помощью журналов Azure Monitor
description: Сведения о мониторинге Azure Site Recovery с помощью журналов Azure Monitor (Log Analytics)
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/15/2019
ms.author: raynew
ms.openlocfilehash: 308e1bcf042feb15179d32844d8c569af6166619
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100571673"
---
# <a name="monitor-site-recovery-with-azure-monitor-logs"></a>Мониторинг Site Recovery с помощью журналов Azure Monitor

В этой статье описывается мониторинг компьютеров, реплицированных с помощью Azure [Site Recovery](site-recovery-overview.md), с помощью [журналов Azure Monitor](../azure-monitor/logs/data-platform-logs.md)и [log Analytics](../azure-monitor/logs/log-query-overview.md).

Журналы Azure Monitor предоставляют платформу данных журнала, которая собирает журналы действий и ресурсов, а также другие данные мониторинга. В журналах Azure Monitor используется Log Analytics для написания и тестирования запросов журналов, а также для интерактивного анализа данных журнала. Вы можете визуализировать и запрашивать результаты журнала, а также настраивать оповещения для выполнения действий на основе наблюдаемых данных.

Для Site Recovery можно использовать журналы Azure Monitor, которые помогут выполнить следующие действия.

- **Наблюдение за работоспособностью и состоянием Site Recovery**. Например, можно наблюдать за работоспособностью репликации, состоянием тестовой отработки отказа, Site Recovery событиями, целями точки восстановления (RPO) для защищенных компьютеров и частотой изменения дисковых данных.
- **Настройте оповещения для Site Recovery**. Например, можно настроить оповещения о работоспособности компьютера, состоянии тестовой отработки отказа или Site Recovery состояния задания.

Использование журналов Azure Monitor с Site Recovery поддерживается для репликации **Azure в Azure** , а также для репликации **виртуальных машин и физических серверов VMware в Azure** .

> [!NOTE]
> Для получения журналов обработанных данных и журналов скорости передачи для VMware и физических компьютеров необходимо установить Microsoft Monitoring Agent на сервере обработки. Этот агент отправляет журналы реплицируемых компьютеров в рабочую область. Эта возможность доступна только для версии агента мобильности 9,30.

## <a name="before-you-start"></a>Перед началом работы

Вам потребуется следующее.

- По крайней мере один компьютер, защищенный в хранилище служб восстановления.
- Рабочая область Log Analytics для хранения журналов Site Recovery. [Сведения о](../azure-monitor/logs/quick-create-workspace.md) настройке рабочей области.
- Основные сведения о том, как писать, выполнять и анализировать запросы журналов в Log Analytics. [Подробнее](../azure-monitor/logs/log-analytics-tutorial.md).

Перед началом работы рекомендуется ознакомиться с [общими вопросами мониторинга](monitoring-common-questions.md) .

## <a name="configure-site-recovery-to-send-logs"></a>Настройка Site Recovery для отправки журналов

1. В хранилище щелкните **параметры диагностики**  >  **Добавить параметр диагностики**.

    ![Снимок экрана, показывающий параметр "добавить параметр диагностики".](./media/monitoring-log-analytics/add-diagnostic.png)

2. В окне **параметры диагностики** укажите имя и установите флажок **отправить на log Analytics**.
3. Выберите подписку на Azure Monitor журналы и рабочую область Log Analytics.
4. Выберите **система диагностики Azure** в переключателе.
5. В списке Журнал выберите все журналы с префиксом **азуреситерековери**. Нажмите кнопку **ОК**.

    ![Снимок экрана с экраном параметров диагностики.](./media/monitoring-log-analytics/select-workspace.png)

Журналы Site Recovery начинается с перевода в таблицу (**AzureDiagnostics**) в выбранной рабочей области.

## <a name="configure-microsoft-monitoring-agent-on-the-process-server-to-send-churn-and-upload-rate-logs"></a>Настройте Microsoft Monitoring Agent на сервере обработки для отправки журналов обновлений и скорости отправки

Вы можете записывать сведения о скорости обработки данных и скорости отправки исходных данных для виртуальных машин VMware или физических компьютеров в локальной среде. Чтобы это сделать, на сервере обработки должен быть установлен агент наблюдения (Майкрософт).

1. Перейдите в рабочую область Log Analytics и щелкните **Дополнительные параметры**.
2. Щелкните страницу **подключенные источники** и выберите **серверы Windows**.
3. Скачайте агент Windows (64 bit) на сервере обработки. 
4. [Получение идентификатора и ключа рабочей области](../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key)
5. [Настройка агента для использования TLS 1,2](../azure-monitor/agents/agent-windows.md#configure-agent-to-use-tls-12)
6. [Завершите установку агента](../azure-monitor/agents/agent-windows.md#install-agent-using-setup-wizard) , указав полученный идентификатор и ключ рабочей области.
7. После завершения установки перейдите в Log Analytics рабочую область и щелкните **Дополнительные параметры**. Перейдите на страницу **данные** и щелкните **счетчики производительности Windows** еще больше. 
8. Щелкните **"+"** , чтобы добавить следующие два счетчика с интервалом выборки в 300 секунд:

    - Асраналитикс (*) \Саурцевмчурнрате
    - Асраналитикс (*) \Саурцевмсрпрате

Данные о скорости обработки и передачи начнут поступать в рабочую область.


## <a name="query-the-logs---examples"></a>Запрос журналов — примеры

Данные из журналов извлекаются с помощью запросов журналов, написанных на [языке запросов Kusto](../azure-monitor/logs/get-started-queries.md). В этом разделе приводится несколько примеров распространенных запросов, которые можно использовать для мониторинга Site Recovery.

> [!NOTE]
> В некоторых примерах используется **replicationProviderName_s** задано значение **A2A**. При этом будут извлечены виртуальные машины Azure, которые реплицируются в дополнительный регион Azure, с помощью Site Recovery. В этих примерах можно заменить **A2A** на **InMageAzureV2**, если вы хотите получить локальные виртуальные машины VMware или физические серверы, которые реплицируются в Azure с помощью Site Recovery.


### <a name="query-replication-health"></a>Запрос работоспособности репликации

Этот запрос строит круговую диаграмму о текущей работоспособности репликации всех защищенных виртуальных машин Azure, разбитую на три состояния: обычная, предупреждение или критическая.

```
AzureDiagnostics  
| where replicationProviderName_s == "A2A"   
| where isnotempty(name_s) and isnotnull(name_s)  
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s  
| project name_s , replicationHealth_s  
| summarize count() by replicationHealth_s  
| render piechart   
```
### <a name="query-mobility-service-version"></a>Запрос версии службы Mobility Service

Этот запрос строит круговую диаграмму виртуальных машин Azure, реплицированных с Site Recovery, с разбивкой по версии агента мобильности, в которой они выполняются.

```
AzureDiagnostics  
| where replicationProviderName_s == "A2A"   
| where isnotempty(name_s) and isnotnull(name_s)  
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s  
| project name_s , agentVersion_s  
| summarize count() by agentVersion_s  
| render piechart 
```

### <a name="query-rpo-time"></a>Время RPO запроса

Этот запрос строит линейчатую диаграмму виртуальных машин Azure, реплицированных с Site Recovery, разделенных на целевую точку восстановления (RPO): менее 15 минут, от 15-30 минут до 30 минут.

```
AzureDiagnostics 
| where replicationProviderName_s == "A2A"   
| where isnotempty(name_s) and isnotnull(name_s)  
| extend RPO = case(rpoInSeconds_d <= 900, "<15Min",   
rpoInSeconds_d <= 1800, "15-30Min", ">30Min")  
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s  
| project name_s , RPO  
| summarize Count = count() by RPO  
| render barchart 
```

![Снимок экрана, показывающий линейчатую диаграмму виртуальных машин Azure, реплицированную с помощью Site Recovery.](./media/monitoring-log-analytics/example1.png)

### <a name="query-site-recovery-jobs"></a>Запрос Site Recovery заданий

Этот запрос получает все задания Site Recovery (для всех сценариев аварийного восстановления), которые запускаются за последние 72 часов и их состояние завершения.

```
AzureDiagnostics  
| where Category == "AzureSiteRecoveryJobs"  
| where TimeGenerated >= ago(72h)   
| project JobName = OperationName , VaultName = Resource , TargetName = affectedResourceName_s, State = ResultType  
```

### <a name="query-site-recovery-events"></a>События Site Recovery запросов

Этот запрос получает все события Site Recovery (для всех сценариев аварийного восстановления), которые возникают за последние 72 часов и их серьезность. 

```
AzureDiagnostics   
| where Category == "AzureSiteRecoveryEvents"   
| where TimeGenerated >= ago(72h)   
| project AffectedObject=affectedResourceName_s , VaultName = Resource, Description_s = healthErrors_s , Severity = Level  
```

### <a name="query-test-failover-state-pie-chart"></a>Состояние тестовой отработки отказа запроса (круговая диаграмма)

Этот запрос строит круговую диаграмму для состояния тестовой отработки отказа виртуальных машин Azure, реплицированных с помощью Site Recovery.

```
AzureDiagnostics  
| where replicationProviderName_s == "A2A"   
| where isnotempty(name_s) and isnotnull(name_s)  
| where isnotempty(failoverHealth_s) and isnotnull(failoverHealth_s)  
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s  
| project name_s , Resource, failoverHealth_s  
| summarize count() by failoverHealth_s  
| render piechart 
```

### <a name="query-test-failover-state-table"></a>Состояние тестовой отработки отказа запроса (таблица)

Этот запрос отображает таблицу для состояния тестовой отработки отказа виртуальных машин Azure, реплицированных с помощью Site Recovery.

```
AzureDiagnostics   
| where replicationProviderName_s == "A2A"   
| where isnotempty(name_s) and isnotnull(name_s)   
| where isnotempty(failoverHealth_s) and isnotnull(failoverHealth_s)   
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s   
| project VirtualMachine = name_s , VaultName = Resource , TestFailoverStatus = failoverHealth_s 
```

### <a name="query-machine-rpo"></a>Запросить ЦЕЛЕВое значение Machine

Этот запрос строит граф тенденций, который отслеживает RPO определенной виртуальной машины Azure (ContosoVM123) за последние 72 часов.

```
AzureDiagnostics   
| where replicationProviderName_s == "A2A"   
| where TimeGenerated > ago(72h)  
| where isnotempty(name_s) and isnotnull(name_s)   
| where name_s == "ContosoVM123"  
| project TimeGenerated, name_s , RPO_in_seconds = rpoInSeconds_d   
| render timechart 
```
![Снимок экрана с диаграммой тренда, отслеживающей целевую точку восстановления для определенной виртуальной машины Azure.](./media/monitoring-log-analytics/example2.png)

### <a name="query-data-change-rate-churn-and-upload-rate-for-an-azure-vm"></a>Скорость изменения данных запроса (обновление) и скорость передачи для виртуальной машины Azure

Этот запрос строит диаграмму тренда для конкретной виртуальной машины Azure (ContosoVM123), которая представляет скорость изменения данных (байт в секунду) и скорость передачи данных. 

```
AzureDiagnostics   
| where Category in ("AzureSiteRecoveryProtectedDiskDataChurn", "AzureSiteRecoveryReplicationDataUploadRate")   
| extend CategoryS = case(Category contains "Churn", "DataChurn",   
Category contains "Upload", "UploadRate", "none")  
| extend InstanceWithType=strcat(CategoryS, "_", InstanceName_s)   
| where TimeGenerated > ago(24h)   
| where InstanceName_s startswith "ContosoVM123"   
| project TimeGenerated , InstanceWithType , Churn_MBps = todouble(Value_s)/1048576   
| render timechart  
```
![снимок экрана с диаграммой тенденций для определенной виртуальной машины Azure.](./media/monitoring-log-analytics/example3.png)

### <a name="query-data-change-rate-churn-and-upload-rate-for-a-vmware-or-physical-machine"></a>Скорость изменения данных запросов (обновление) и скорость передачи для VMware или физического компьютера

> [!Note]
> Убедитесь, что на сервере обработки настроен агент мониторинга для выборки этих журналов. [Инструкции по настройке агента мониторинга](#configure-microsoft-monitoring-agent-on-the-process-server-to-send-churn-and-upload-rate-logs)см. в статье.

Этот запрос строит диаграмму тренда для конкретного диска **disk0** реплицированного элемента **Win-9r7sfh9qlru**, который представляет скорость изменения данных (записи в байтах в секунду) и скорость передачи данных. Вы можете найти **в колонке** имя диска в реплицированном элементе в хранилище служб восстановления. Имя экземпляра, используемое в запросе, — это DNS-имя компьютера, за которым следует имя диска, как в этом примере.

```
Perf
| where ObjectName == "ASRAnalytics"
| where InstanceName contains "win-9r7sfh9qlru_disk0"
| where TimeGenerated >= ago(4h) 
| project TimeGenerated ,CounterName, Churn_MBps = todouble(CounterValue)/5242880 
| render timechart
```
Сервер обработки отправляет эти данные каждые 5 минут в Log Analytics рабочую область. Эти точки данных представляют среднее значение, вычисленное в течение 5 минут.

### <a name="query-disaster-recovery-summary-azure-to-azure"></a>Запрос сводки аварийного восстановления (Azure в Azure)

Этот запрос строит сводную таблицу для виртуальных машин Azure, реплицированных в дополнительный регион Azure.  В нем отображаются имя виртуальной машины, репликация и состояние защиты, RPO, состояние тестовой отработки отказа, версия агента мобильности, все активные ошибки репликации и исходное расположение.

```
AzureDiagnostics 
| where replicationProviderName_s == "A2A"   
| where isnotempty(name_s) and isnotnull(name_s)   
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s   
| project VirtualMachine = name_s , Vault = Resource , ReplicationHealth = replicationHealth_s, Status = protectionState_s, RPO_in_seconds = rpoInSeconds_d, TestFailoverStatus = failoverHealth_s, AgentVersion = agentVersion_s, ReplicationError = replicationHealthErrors_s, SourceLocation = primaryFabricName_s 
```

### <a name="query-disaster-recovery-summary-vmwarephysical-servers"></a>Запрос сводки аварийного восстановления (VMware или физические серверы)

Этот запрос строит сводную таблицу для виртуальных машин VMware и физических серверов, реплицированных в Azure.  В нем отображаются имя компьютера, репликация и состояние защиты, RPO, состояние тестовой отработки отказа, версия агента мобильности, все активные ошибки репликации и соответствующий сервер обработки.

```
AzureDiagnostics  
| where replicationProviderName_s == "InMageAzureV2"   
| where isnotempty(name_s) and isnotnull(name_s)   
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s   
| project VirtualMachine = name_s , Vault = Resource , ReplicationHealth = replicationHealth_s, Status = protectionState_s, RPO_in_seconds = rpoInSeconds_d, TestFailoverStatus = failoverHealth_s, AgentVersion = agentVersion_s, ReplicationError = replicationHealthErrors_s, ProcessServer = processServerName_g  
```

## <a name="set-up-alerts---examples"></a>Настройка оповещений — примеры

Вы можете настроить Site Recovery оповещения на основе Azure Monitor данных. Дополнительные [сведения](../azure-monitor/alerts/alerts-log.md#create-a-log-alert-rule-with-the-azure-portal) о настройке оповещений журнала. 

> [!NOTE]
> В некоторых примерах используется **replicationProviderName_s** задано значение **A2A**. Этот параметр задает оповещения для виртуальных машин Azure, которые реплицируются в дополнительный регион Azure. В этих примерах можно заменить **A2A** на **InMageAzureV2** , если вы хотите настроить оповещения для локальных виртуальных машин VMware или физических серверов, реплицированных в Azure.

### <a name="multiple-machines-in-a-critical-state"></a>Несколько компьютеров в критическом состоянии

Настройте оповещение, если более 20 реплицированных виртуальных машин Azure переходит в критическое состояние.

```
AzureDiagnostics   
| where replicationProviderName_s == "A2A"   
| where replicationHealth_s == "Critical"  
| where isnotempty(name_s) and isnotnull(name_s)   
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s   
| summarize count() 
```
Для предупреждения установите **пороговое значение** 20.

### <a name="single-machine-in-a-critical-state"></a>Один компьютер в критическом состоянии

Настройте оповещение, если конкретная реплицированная виртуальная машина Azure переходит в критическое состояние.

```
AzureDiagnostics   
| where replicationProviderName_s == "A2A"   
| where replicationHealth_s == "Critical"  
| where name_s == "ContosoVM123"  
| where isnotempty(name_s) and isnotnull(name_s)   
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s   
| summarize count()  
```
Для предупреждения установите **пороговое значение** 1.

### <a name="multiple-machines-exceed-rpo"></a>Превышение RPO для нескольких компьютеров

Настройте оповещение, если значение RPO для более 20 виртуальных машин Azure превышает 30 минут.
```
AzureDiagnostics   
| where replicationProviderName_s == "A2A"   
| where isnotempty(name_s) and isnotnull(name_s)   
| where rpoInSeconds_d > 1800  
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s   
| project name_s , rpoInSeconds_d   
| summarize count()  
```
Для предупреждения установите **пороговое значение** 20.

### <a name="single-machine-exceeds-rpo"></a>Один компьютер превышает RPO

Настройте оповещение, если значение RPO для одной виртуальной машины Azure превышает 30 минут.

```
AzureDiagnostics   
| where replicationProviderName_s == "A2A"   
| where isnotempty(name_s) and isnotnull(name_s)   
| where name_s == "ContosoVM123"  
| where rpoInSeconds_d > 1800  
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s   
| project name_s , rpoInSeconds_d   
| summarize count()  
```
Для предупреждения установите **пороговое значение** 1.

### <a name="test-failover-for-multiple-machines-exceeds-90-days"></a>Тестовая отработка отказа для нескольких компьютеров превышает 90 дней

Настройте оповещение, если последняя успешная тестовая отработка отказа превысила 90 дней, для более чем 20 виртуальных машин. 

```
AzureDiagnostics  
| where replicationProviderName_s == "A2A"   
| where Category == "AzureSiteRecoveryReplicatedItems"  
| where isnotempty(name_s) and isnotnull(name_s)   
| where lastSuccessfulTestFailoverTime_t <= ago(90d)   
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s   
| summarize count()  
```
Для предупреждения установите **пороговое значение** 20.

### <a name="test-failover-for-single-machine-exceeds-90-days"></a>Тестовая отработка отказа для одного компьютера превышает 90 дней

Настройте оповещение, если последняя успешная тестовая отработка отказа для конкретной виртуальной машины была более 90 дней назад.
```
AzureDiagnostics  
| where replicationProviderName_s == "A2A"   
| where Category == "AzureSiteRecoveryReplicatedItems"  
| where isnotempty(name_s) and isnotnull(name_s)   
| where lastSuccessfulTestFailoverTime_t <= ago(90d)   
| where name_s == "ContosoVM123"  
| summarize hint.strategy=partitioned arg_max(TimeGenerated, *) by name_s   
| summarize count()  
```
Для предупреждения установите **пороговое значение** 1.

### <a name="site-recovery-job-fails"></a>Сбой задания Site Recovery

Настройте оповещение, если задание Site Recovery (в данном случае задание повторной защиты) завершается сбоем для любого сценария Site Recovery, в течение последнего дня. 
```
AzureDiagnostics   
| where Category == "AzureSiteRecoveryJobs"   
| where OperationName == "Reprotect"  
| where ResultType == "Failed"  
| summarize count()  
```

Для предупреждения установите **пороговое значение** 1, а **период** — 1440 минут, чтобы проверить ошибки за последний день.

## <a name="next-steps"></a>Дальнейшие действия

[Дополнительные сведения о](site-recovery-monitor-and-troubleshoot.md) встроенном мониторинге Site Recovery.