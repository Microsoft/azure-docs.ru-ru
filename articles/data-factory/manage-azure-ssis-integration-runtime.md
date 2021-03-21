---
title: Перенастройка среды выполнения интеграции SQL Server Integration Services Azure
description: Узнайте, как перенастроить среду выполнения интеграции Azure-SSIS в фабрике данных Azure после ее подготовки.
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/03/2020
author: swinarko
ms.author: sawinark
ms.openlocfilehash: 724411dc12654aec1614230c943923062b334cd2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100370688"
---
# <a name="reconfigure-the-azure-ssis-integration-runtime"></a>Перенастройка среды выполнения интеграции SQL Server Integration Services Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Из этой статьи вы узнаете, как перенастроить существующую среду выполнения интеграции SQL Server Integration Services Azure. Сведения о создании среды выполнения интеграции SQL Server Integration Services Azure в Фабрике данных Azure см. в [этой статье](create-azure-ssis-integration-runtime.md).  

## <a name="data-factory-ui"></a>Пользовательский интерфейс Фабрики данных 
Пользовательский интерфейс фабрики данных можно использовать для остановки, изменения, повторной настройки и удаления среды выполнения интеграции Azure SSIS. 

1. Откройте пользовательский интерфейс фабрики данных, щелкнув плитку " **автор & монитор** " на домашней странице фабрики данных.
2. Выберите **Управление** концентратором ниже **Главная**, **изменить** и **мониторинг** концентраторов, чтобы отображалась область **подключения** .

### <a name="to-reconfigure-an-azure-ssis-ir"></a>Перенастройка среды выполнения интеграции Azure SSIS
На панели **Подключения** центра **Управление** перейдите на страницу **Среды выполнения интеграции** и выберите **Обновить**. 

   ![Панель "Подключения"](./media/tutorial-create-azure-ssis-runtime-portal/connections-pane.png)

   Вы можете изменить или перенастроить свою Azure-SSIS IR, выбрав ее имя. С помощью соответствующих кнопок можно выполнять мониторинг, запуск, остановку или удаление Azure-SSIS IR, автоматически создавать конвейер ADF с помощью действия "Выполнение пакета SSIS" для запуска в Azure-SSIS IR, а также просматривать код JSON и полезные данные Azure-SSIS IR.  Изменить или удалить Azure-SSIS IR можно только после ее остановки.

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

После подготовки и запуска экземпляра среды выполнения Azure SSIS его можно перенастроить, выполнив последовательность командлетов PowerShell `Stop` - `Set` - `Start`. Например, следующий сценарий PowerShell изменяет количество узлов, выделенных для экземпляра среды выполнения интеграции Azure-SSIS, до пяти.

### <a name="reconfigure-an-azure-ssis-ir"></a>Перенастройка Azure-SSIS IR

1. Сначала завершите работу среды выполнения интеграции Azure SSIS с помощью командлета [-AzDataFactoryV2IntegrationRuntime](/powershell/module/az.datafactory/stop-Azdatafactoryv2integrationruntime) . Эта команда освобождает все узлы и прекращает выставление счетов.

   ```powershell
   Stop-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName 
   ```
2. Затем перенастройте Azure-SSIS IR с помощью командлета [Set-AzDataFactoryV2IntegrationRuntime](/powershell/module/az.datafactory/set-Azdatafactoryv2integrationruntime) . В следующем примере команды развертывается среда выполнения интеграции Azure-SSIS на пять узлов.

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -NodeCount 5
   ```  
3. Затем запустите среду выполнения интеграции Azure SSIS с помощью командлета [Start-AzDataFactoryV2IntegrationRuntime](/powershell/module/az.datafactory/start-Azdatafactoryv2integrationruntime) . Эта команда выделяет все ее узлы для запуска пакетов SSIS.   

   ```powershell
   Start-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName
   ```

### <a name="delete-an-azure-ssis-ir"></a>Удаление среды выполнения интеграции Azure-SSIS
1. Сначала перечислите все имеющиеся среды выполнения интеграции Azure-SSIS в фабрике данных.

   ```powershell
   Get-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName -Status
   ```
2. Затем остановите все имеющиеся среды выполнения интеграции Azure-SSIS в фабрике данных.

   ```powershell
   Stop-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Force
   ```
3. После чего удалите все имеющиеся среды выполнения интеграции Azure-SSIS в фабрике данных одну за другой.

   ```powershell
   Remove-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Force
   ```
4. Наконец, удалите фабрику данных.

   ```powershell
   Remove-AzDataFactoryV2 -Name $DataFactoryName -ResourceGroupName $ResourceGroupName -Force
   ```
5. Если вы создавали группу ресурсов, удалите ее.

   ```powershell
   Remove-AzResourceGroup -Name $ResourceGroupName -Force 
   ```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о среде выполнения Azure SSIS см. в следующих разделах: 

- [Azure-SSIS Integration Runtime](concepts-integration-runtime.md#azure-ssis-integration-runtime). В этой статье содержатся общие сведения о средах выполнения интеграции в целом, включая Azure SSIS IR. 
- [Развертывание пакетов служб интеграции SQL Server (SSIS) в Azure](./tutorial-deploy-ssis-packages-azure.md). В этой статье приводятся пошаговые инструкции по созданию Azure-SSIS IR и использованию базы данных SQL Azure для размещения каталога служб SSIS. 
- [Создание среды выполнения интеграции Azure SSIS](create-azure-ssis-integration-runtime.md). Эта статья разворачивается в этом руководстве и содержит инструкции по использованию Управляемый экземпляр Azure SQL и присоединению IR к виртуальной сети. 
- [Присоединение среды выполнения интеграции Azure SSIS к виртуальной сети](join-azure-ssis-integration-runtime-virtual-network.md) В этой статье содержатся общие сведения о присоединении среды выполнения интеграции Azure–SSIS к виртуальной сети Azure. В ней также показано, как настроить виртуальную сеть, чтобы присоединить среду выполнения интеграции Azure–SSIS к виртуальной сети с помощью портала Azure. 
- [Мониторинг среды выполнения интеграции в фабрике данных Azure](monitor-integration-runtime.md#azure-ssis-integration-runtime). В этом статье показано, как извлечь сведения о среде выполнения интеграции Azure SSIS и описания состояний из возвращаемых данных.