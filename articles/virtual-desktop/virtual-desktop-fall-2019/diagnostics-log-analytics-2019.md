---
title: Служба анализа журналов диагностики виртуальных рабочих столов Windows (классическая модель) — Azure
description: Использование log Analytics с функцией диагностики виртуальных рабочих столов Windows (классическая модель).
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: bfe8026f099c341b68a1aa1fc5e3fac425e29403
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100578896"
---
# <a name="use-log-analytics-for-the-diagnostics-feature-in-windows-virtual-desktop-classic"></a>Использование Log Analytics для компонента "Диагностика" в виртуальном рабочем столе Windows (классическая модель)

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows (классическому), который не поддерживает объекты Azure Resource Manager для Виртуального рабочего стола Windows. Сведения об обеспечении управления объектами Azure Resource Manager для Виртуального рабочего стола Windows см. в [этой статье](../diagnostics-log-analytics.md).

Виртуальный рабочий стол Windows включает функцию диагностики, которая позволяет администратору выявлять разные проблемы через единый интерфейс. Эта функция регистрирует диагностические сведения каждый раз, когда назначенная ей роль виртуального рабочего стола Windows использует эту службу. Каждый журнал содержит сведения о том, какая роль виртуальных рабочих столов Windows была вовлечена в действие, любые сообщения об ошибках, отображаемые во время сеанса, сведения о клиенте и сведения о пользователе. Функция диагностики создает журналы действий как для пользователей, так и для административных действий. Каждый журнал действий разделяется на три основные категории:

- Действия подписки веб-канала: когда пользователь пытается подключиться к каналу через Удаленный рабочий стол (Майкрософт) приложения.
- Действия подключения: когда пользователь пытается подключиться к рабочему столу или удаленному приложению RemoteApp через Удаленный рабочий стол (Майкрософт) приложения.
- Действия управления: когда администратор выполняет в системе операции управления, такие как создание пулов узлов, назначение пользователей группам приложений и создание назначений ролей.

Подключения, которые не достигают Виртуального рабочего стола Windows, не отображаются в результатах диагностики, так как служба роли диагностики является частью Виртуального рабочего стола Windows. Проблемы с подключением к виртуальным рабочим столам Windows могут возникнуть, если у пользователя возникли проблемы с сетевым подключением.

## <a name="why-you-should-use-log-analytics"></a>Зачем использовать Log Analytics

Мы рекомендуем использовать Log Analytics для анализа диагностических данных в клиенте Azure, который выходит за пределы одного пользователя. Так как вы можете извлечь счетчики производительности виртуальной машины в Log Analytics у вас есть одно средство для сбора информации о развертывании.

## <a name="before-you-get-started"></a>Необходимые условия

Прежде чем можно будет использовать Log Analytics с функцией диагностики, необходимо [создать рабочую область](../../azure-monitor/vm/quick-collect-windows-computer.md#create-a-workspace).

После создания рабочей области следуйте инструкциям в статье [Подключение компьютеров Windows к Azure Monitor](../../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key) , чтобы получить следующие сведения.

- ИДЕНТИФИКАТОР рабочей области
- Первичный ключ рабочей области

Эти сведения понадобятся вам позже в процессе установки.

## <a name="push-diagnostics-data-to-your-workspace"></a>Отправка данных диагностики в рабочую область

Вы можете отправить диагностические данные из клиента виртуальных рабочих столов Windows в Log Analytics рабочей области. Эту функцию можно настроить сразу после создания клиента путем связывания рабочей области с клиентом или настроить позже с помощью существующего клиента.

Чтобы связать клиент с рабочей областью Log Analytics во время настройки нового клиента, выполните следующий командлет, чтобы войти в виртуальный рабочий стол Windows с помощью учетной записи пользователя Тенанткреатор:

```powershell
Add-RdsAccount -DeploymentUrl https://rdbroker.wvd.microsoft.com
```

Если вы собираетесь связать существующий клиент, а не новый клиент, выполните этот командлет.

```powershell
Set-RdsTenant -Name <TenantName> -AzureSubscriptionId <SubscriptionID> -LogAnalyticsWorkspaceId <String> -LogAnalyticsPrimaryKey <String>
```

Эти командлеты необходимо выполнить для каждого клиента, который необходимо связать с Log Analytics.

>[!NOTE]
>Если вы не хотите связывать рабочую область Log Analytics при создании клиента, выполните `New-RdsTenant` командлет.

## <a name="cadence-for-sending-diagnostic-events"></a>Ритмичность для отправки диагностических событий

События диагностики отправляются в Log Analytics по завершении.

## <a name="example-queries"></a>Примеры запросов

В следующих примерах запросов показано, как функция диагностики создает отчет для наиболее частых действий в системе:

В первом примере показаны действия подключения, инициированные пользователями с поддерживаемыми клиентами удаленного рабочего стола:

```powershell
WVDActivityV1_CL

| where Type_s == "Connection"

| join kind=leftouter (

    WVDErrorV1_CL

    | summarize Errors = makelist(pack('Time', Time_t, 'Code', ErrorCode_s , 'CodeSymbolic', ErrorCodeSymbolic_s, 'Message', ErrorMessage_s, 'ReportedBy', ReportedBy_s , 'Internal', ErrorInternal_s )) by ActivityId_g

    ) on $left.Id_g  == $right.ActivityId_g 

| join  kind=leftouter (

    WVDCheckpointV1_CL

    | summarize Checkpoints = makelist(pack('Time', Time_t, 'ReportedBy', ReportedBy_s, 'Name', Name_s, 'Parameters', Parameters_s) ) by ActivityId_g

    ) on $left.Id_g  == $right.ActivityId_g

|project-away ActivityId_g, ActivityId_g1
```

Следующий пример запроса показывает действия по управлению администраторами на клиентах:

```powershell
WVDActivityV1_CL

| where Type_s == "Management"

| join kind=leftouter (

    WVDErrorV1_CL

    | summarize Errors = makelist(pack('Time', Time_t, 'Code', ErrorCode_s , 'CodeSymbolic', ErrorCodeSymbolic_s, 'Message', ErrorMessage_s, 'ReportedBy', ReportedBy_s , 'Internal', ErrorInternal_s )) by ActivityId_g

    ) on $left.Id_g  == $right.ActivityId_g 

| join  kind=leftouter (

    WVDCheckpointV1_CL

    | summarize Checkpoints = makelist(pack('Time', Time_t, 'ReportedBy', ReportedBy_s, 'Name', Name_s, 'Parameters', Parameters_s) ) by ActivityId_g

    ) on $left.Id_g  == $right.ActivityId_g

|project-away ActivityId_g, ActivityId_g1
```

## <a name="stop-sending-data-to-log-analytics"></a>Запретить отправку данных в Log Analytics

Чтобы запретить отправку данных из существующего клиента в Log Analytics, выполните следующий командлет и задайте пустые строки:

```powershell
Set-RdsTenant -Name <TenantName> -AzureSubscriptionId <SubscriptionID> -LogAnalyticsWorkspaceId <String> -LogAnalyticsPrimaryKey <String>
```

Этот командлет потребуется выполнить для каждого клиента, с которого необходимо запретить отправку данных.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы просмотреть распространенные сценарии ошибок, которые может определить функция диагностики, см. раздел [Определение и диагностика проблем](diagnostics-role-service-2019.md#common-error-scenarios).
