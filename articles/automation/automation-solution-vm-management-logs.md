---
title: Запрос журналов Запуска и остановки виртуальных машин в нерабочее время в службе автоматизации Azure
description: В этой статье объясняется, как использовать Azure Monitor для запроса данных журналов, создаваемых решением "Запуск и остановка виртуальных машин в нерабочее время".
services: automation
ms.subservice: process-automation
ms.date: 04/01/2020
ms.topic: conceptual
ms.openlocfilehash: 3e9e924d6626d9f0dcd2db8a5e8b8f90a0aa01ce
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100593842"
---
# <a name="query-logs-from-startstop-vms-during-off-hours"></a>Журналы запросов Запуска и остановки виртуальных машин в нерабочее время

Служба автоматизации направляет в связанную рабочую область Log Analytics записи двух типов: журналы заданий и потоки заданий. В этой статье рассматриваются данные, доступные для [запроса](../azure-monitor/logs/log-query-overview.md) в Azure Monitor.

## <a name="job-logs"></a>Журналы заданий

|Свойство | Описание|
|----------|----------|
|Caller |  Сторона, инициировавшая операцию. Допустимые значения: электронный адрес или system для запланированных заданий.|
|Категория | Классификация типа данных. Для службы автоматизации значением является JobLogs.|
|CorrelationId | GUID, представляющий собой идентификатор корреляции задания runbook.|
|JobId | GUID, представляющий собой идентификатор задания runbook.|
|operationName | Указывает тип операции, выполняемой в Azure. Для службы автоматизации значением является Job.|
|resourceId | Указывает тип ресурса в Azure. Для службы автоматизации значением является учетная запись службы автоматизации, связанная с Runbook.|
|ResourceGroup | Указывает имя группы ресурсов задания Runbook.|
|ResourceProvider | Указывает службу Azure, которая предоставляет ресурсы для развертывания и управления. Для службы автоматизации значением будет Azure Automation.|
|ResourceType | Указывает тип ресурса в Azure. Для службы автоматизации значением является учетная запись службы автоматизации, связанная с Runbook.|
|resultType | Состояние задания Runbook. Возможны следующие значения:<br>Started<br>- Остановлена<br>Приостановлено<br>Сбой<br>- Succeeded.|
|resultDescription | Описывает состояние результата задания Runbook. Возможны следующие значения:<br>— Job is started;<br>— Job Failed;<br>- Job Completed.|
|RunbookName | Указывает имя модуля Runbook.|
|SourceSystem | Указывает исходную систему для отправленных данных. Для службы автоматизации значением является OpsManager.|
|StreamType | Задает тип события. Возможны следующие значения:<br>- Verbose.<br>Выходные данные<br>error<br>Предупреждение|
|SubscriptionId | Указывает идентификатор подписки задания.
|Time | Дата и время выполнения задания Runbook.|

## <a name="job-streams"></a>Потоки заданий

|Свойство | Описание|
|----------|----------|
|Caller |  Сторона, инициировавшая операцию. Допустимые значения: электронный адрес или system для запланированных заданий.|
|Категория | Классификация типа данных. Для службы автоматизации значением является JobStreams.|
|JobId | GUID, представляющий собой идентификатор задания runbook.|
|operationName | Указывает тип операции, выполняемой в Azure. Для службы автоматизации значением является Job.|
|ResourceGroup | Указывает имя группы ресурсов задания Runbook.|
|resourceId | Указывает идентификатор ресурса в Azure. Для службы автоматизации значением является учетная запись службы автоматизации, связанная с Runbook.|
|ResourceProvider | Указывает службу Azure, которая предоставляет ресурсы для развертывания и управления. Для службы автоматизации значением будет Azure Automation.|
|ResourceType | Указывает тип ресурса в Azure. Для службы автоматизации значением является учетная запись службы автоматизации, связанная с Runbook.|
|resultType | Результат задания Runbook во время создания события. Возможное значение:<br>- InProgress.|
|resultDescription | Включает в себя выходной поток из Runbook.|
|RunbookName | Имя Runbook.|
|SourceSystem | Указывает исходную систему для отправленных данных. Для службы автоматизации значением является OpsManager.|
|StreamType | Тип потока задания. Возможны следующие значения:<br>- Progress<br>Выходные данные<br>Предупреждение<br>error<br>debug<br>- Verbose.|
|Time | Дата и время выполнения задания Runbook.|

При выполнении поиска по журналам, в результате которого возвращаются записи категории **JobLogs** или **JobStreams**, можно выбрать представление **JobLogs** или **JobStreams**. Эти представления отображают набор элементов с перечнем обновлений, возвращенных в результатах поиска.

## <a name="sample-log-searches"></a>Пример поисков журналов

Приведенная ниже таблица содержит примеры поиска по журналам для получения записей заданий, собранных решением "Запуск и остановка виртуальных машин в нерабочее время".

|Запрос | Описание|
|----------|----------|
|Поиск успешно выполненных заданий runbook ScheduledStartStop_Parent. | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "ScheduledStartStop_Parent" ) <br>&#124;  where ( ResultType == "Completed" )  <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|
|Поиск заданий runbook ScheduledStartStop_Parent, которые не были выполнены успешно. | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "ScheduledStartStop_Parent" ) <br>&#124;  where ( ResultType == "Failed" )  <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|
|Поиск успешно выполненных заданий runbook SequencedStartStop_Parent. | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "SequencedStartStop_Parent" ) <br>&#124;  where ( ResultType == "Completed" ) <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|
|Поиск заданий runbook SequencedStartStop_Parent, которые не были выполнены успешно. | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "SequencedStartStop_Parent" ) <br>&#124;  where ( ResultType == "Failed" ) <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы настроить возможность Запуска и остановки виртуальных машин в нерабочее время, см. [эту статью](automation-solution-vm-management-config.md).
* Сведения о предупреждениях журнала во время развертывания компонентов см. в статье [Создание оповещений журнала с помощью Azure Monitor](../azure-monitor/alerts/alerts-log.md).
* Сведения об устранении ошибок с этой возможностью см. в [этой статье](troubleshoot/start-stop-vm.md).