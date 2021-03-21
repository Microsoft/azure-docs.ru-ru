---
title: Отправка метрик виртуальной машины классической системы Windows в базу данных метрик Azure Monitor
description: Отправка метрик ОС для виртуальной машины Windows (классическая) в хранилище данных Azure Monitor
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: ancav
ms.openlocfilehash: dcb2c0ceb3092bb6905ac1eb9fc58093da935f2f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102048987"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metrics-database-for-a-windows-virtual-machine-classic"></a>Отправка метрик гостевой ОС в базу данных метрик Azure Monitor для виртуальной машины Windows (классическая модель)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[Расширение диагностики](../agents/diagnostics-extension-overview.md) Azure Monitor (также известное как WAD или "Диагностика") позволяет собирать метрики и журналы из гостевой операционной системы (гостевой ОС), работающей на виртуальной машине, в облачной службе или в кластере Service Fabric. Это расширение может отправлять данные телеметрии во [множество различных расположений](../data-platform.md?toc=%2fazure%2fazure-monitor%2ftoc.json).

В этой статье описывается процесс отправки метрик производительности гостевой ОС для виртуальной машины Windows (классической) в базу данных метрик Azure Monitor. Начиная с версии 1.11 расширение диагностики позволяет записывать метрики напрямую в хранилище метрик Azure Monitor, где уже собраны стандартные метрики платформы. 

Хранение их в этом расположении позволяет получить доступ к тем же действиям, которые доступны для метрик платформы. К этим действиям относятся оповещения практически в реальном времени, построение диаграмм, маршрутизация, доступ из REST API и многое другое. Ранее расширение диагностики записывало данные в службу хранилища Azure, а не в хранилище данных Azure Monitor. 

Процесс, описанный в этой статье, применим только к классическим виртуальным машинам под управлением ОС Windows.

## <a name="prerequisites"></a>Предварительные требования

- Вам необходимы права [администратора службы или соадминистратора](../../cost-management-billing/manage/add-change-subscription-administrator.md) в подписке Azure. 

- Подписку необходимо зарегистрировать в [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md). 

- Необходимо установить [Azure PowerShell](/powershell/azure) или [Azure Cloud Shell](../../cloud-shell/overview.md).

- Ресурс виртуальной машины должен находиться в [регионе, поддерживающем пользовательские метрики](./metrics-custom-overview.md#supported-regions).

## <a name="create-a-classic-virtual-machine-and-storage-account"></a>Создание классической виртуальной машины и учетной записи хранения

1. Создайте классическую виртуальную машину на портале Azure.
   ![Создание классической виртуальной машины](./media/collect-custom-metrics-guestos-vm-classic/create-classic-vm.png)

1. При создании виртуальной машины выберите вариант создания новой классической учетной записи хранения. Мы воспользуемся этой учетной записью хранения на следующих шагах.

1. На портале Azure перейдите в колонку ресурсов **Учетные записи хранения**. Выберите **Ключи** и запишите имя учетной записи хранения и ключ к ней. Они понадобятся для выполнения последующих действий.
   ![Ключи доступа к хранилищу](./media/collect-custom-metrics-guestos-vm-classic/storage-access-keys.png)

## <a name="create-a-service-principal"></a>Создание субъекта-службы

Создайте субъект-службу в клиенте Azure Active Directory, выполнив инструкции в разделе [Создание субъекта-службы](../../active-directory/develop/howto-create-service-principal-portal.md). Выполняя это действие, обратите внимание на следующее. 
- Создайте новый секрет клиента для этого приложения.
- Сохраните ключ и идентификатор клиента, чтобы использовать их в дальнейшем.

Предоставьте этому приложению права "Издатель метрик мониторинга" для доступа к ресурсу, из которого вы хотите передавать метрики. Вы можете использовать группу ресурсов или целую подписку.  

> [!NOTE]
> Расширение диагностики будет использовать субъект-службу для проверки подлинности в Azure Monitor и передавать метрики для классической виртуальной машины.

## <a name="author-diagnostics-extension-configuration"></a>Создание конфигурации для расширения диагностики

1. Подготовьте файл конфигурации для расширения диагностики. Этот файл определяет, какие журналы и счетчики производительности для классической виртуальной машины должно собирать расширение диагностики. Ниже приведен пример:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        <WadCfg>
        <DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="applicationInsights.errors">
            <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" />
            <Directories scheduledTransferPeriod="PT1M">
                <IISLogs containerName="wad-iis-logfiles" />
                <FailedRequestLogs containerName="wad-failedrequestlogs" />
            </Directories>
            <PerformanceCounters scheduledTransferPeriod="PT1M">
                <PerformanceCounterConfiguration counterSpecifier="\Processor(*)\% Processor Time" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\Available Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\% Committed Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(*)\Disk Read Bytes/sec" sampleRate="PT15S" />
            </PerformanceCounters>
            <WindowsEventLog scheduledTransferPeriod="PT1M">
                <DataSource name="Application!*[System[(Level=1 or Level=2 or Level=3)]]" />
                <DataSource name="Windows Azure!*[System[(Level=1 or Level=2 or Level=3 or Level=4)]]" />
            </WindowsEventLog>
            <CrashDumps>
                <CrashDumpConfiguration processName="WaIISHost.exe" />
                <CrashDumpConfiguration processName="WaWorkerHost.exe" />
                <CrashDumpConfiguration processName="w3wp.exe" />
            </CrashDumps>
            <Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Error" />
            <Metrics resourceId="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.ClassicCompute/virtualMachines/MyClassicVM">
                <MetricAggregation scheduledTransferPeriod="PT1M" />
                <MetricAggregation scheduledTransferPeriod="PT1H" />
            </Metrics>
        </DiagnosticMonitorConfiguration>
        <SinksConfig>
        </SinksConfig>
        </WadCfg>
        <StorageAccount />
    </PublicConfig>
    <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        <StorageAccount name="" endpoint="" />
    </PrivateConfig>
    <IsEnabled>true</IsEnabled>
    </DiagnosticsConfiguration>
    ```
1. В разделе SinksConfig файла диагностики укажите новый приемник Azure Monitor, как показано ниже:

    ```xml
    <SinksConfig>
        <Sink name="AzMonSink">
            <AzureMonitor>
                <ResourceId>Provide the resource ID of your classic VM </ResourceId>
                <Region>The region your VM is deployed in</Region>
            </AzureMonitor>
        </Sink>
    </SinksConfig>
    ```

1. В разделе файла конфигурации, где указан список счетчиков производительности, с которых собираются данные, направьте данные счетчиков производительности в приемник Azure Monitor AzMonSink.

    ```xml
    <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="AzMonSink">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" />
    ...
    </PerformanceCounters>
    ```

1. В закрытой конфигурации укажите учетную запись Azure Monitor. Затем добавьте сведения о субъекте-службе, который используется для генерации метрик.

    ```xml
    <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <StorageAccount name="" endpoint="" />
        <AzureMonitorAccount>
            <ServicePrincipalMeta>
                <PrincipalId>clientId for your service principal</PrincipalId>
                <Secret>client secret of your service principal</Secret>
            </ServicePrincipalMeta>
        </AzureMonitorAccount>
    </PrivateConfig>
    ```

1. Сохраните файл локально.

## <a name="deploy-the-diagnostics-extension-to-your-cloud-service"></a>Развертывание расширения диагностики в облачной службе

1. Запустите PowerShell и войдите в систему.

    ```powershell
    Login-AzAccount
    ```

1. Начните с настройки контекста для классической виртуальной машины.

    ```powershell
    $VM = Get-AzureVM -ServiceName <VM’s Service_Name> -Name <VM Name>
    ```

1. Задайте контекст классической учетной записи хранения, созданной в виртуальной машине.

    ```powershell
    $StorageContext = New-AzStorageContext -StorageAccountName <name of your storage account from earlier steps> -storageaccountkey "<storage account key from earlier steps>"
    ```

1.  Укажите путь к файлу диагностики в переменной с помощью следующей команды:

    ```powershell
    $diagconfig = “<path of the diagnostics configuration file with the Azure Monitor sink configured>”
    ```

1.  Подготовьте обновление для классической виртуальной машины, содержащее файл диагностики с настроенным приемником Azure Monitor.

    ```powershell
    $VM_Update = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $diagconfig -VM $VM -StorageContext $Storage_Context
    ```

1.  Разверните это обновление на виртуальной машине, выполнив следующую команду:

    ```powershell
    Update-AzureVM -ServiceName "ClassicVMWAD7216" -Name "ClassicVMWAD" -VM $VM_Update.VM
    ```

> [!NOTE]
> В процессе установки расширения диагностики по-прежнему необходимо указывать учетную запись хранения. Все журналы и счетчики производительности, указанные в файле конфигурации диагностики, будут записываться в указанную учетную запись хранения.

## <a name="plot-the-metrics-in-the-azure-portal"></a>График метрик на портале Azure

1.  Перейдите на портал Azure. 

1.  В меню слева выберите **мониторинг.**

1.  В колонке **Монитор** выберите **Метрики**.

    ![Переход к метрикам](./media/collect-custom-metrics-guestos-vm-classic/navigate-metrics.png)

1. В раскрывающемся меню ресурсов выберите требуемую классическую виртуальную машину.

1. В раскрывающемся меню пространства имен выберите **Azure. ВМ. Windows. Guest**.

1. В раскрывающемся меню метрики выберите **"память \ используемые байты**.
   ![График метрик](./media/collect-custom-metrics-guestos-vm-classic/plot-metrics.png)


## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о настраиваемых метриках см. в [этой статье](./metrics-custom-overview.md).