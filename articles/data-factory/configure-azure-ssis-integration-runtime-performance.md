---
title: Настройка производительности для Azure-SSIS Integration Runtime
description: Узнайте, как настроить свойства среды выполнения интеграции Azure-SSIS для высокой производительности
ms.date: 01/10/2018
ms.topic: conceptual
ms.service: data-factory
author: swinarko
ms.author: sawinark
ms.openlocfilehash: 5d275100124660b901504b7e7f71cf93518fd077
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100364398"
---
# <a name="configure-the-azure-ssis-integration-runtime-for-high-performance"></a>Настройка среды выполнения интеграции Azure-SSIS для высокой производительности

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]


В этой статье описывается настройка среды выполнения интеграции Azure-SSIS для высокой производительности. Azure SSIS IR можно использовать для развертывания и запуска пакетов SQL Server Integration Services (SSIS) в Azure. Дополнительные сведения о среде выполнения интеграции Azure-SSIS IR см. в [этом разделе](concepts-integration-runtime.md#azure-ssis-integration-runtime). Сведения о развертывании и запуске пакетов SQL Server Integration Services (SSIS) в Azure см. в статье [Lift and shift SQL Server Integration Services workloads to the cloud](/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview) (Перенос рабочих нагрузок SQL Server Integration Services в облако).

> [!IMPORTANT]
> Эта статья содержит результаты производительности и наблюдения за внутренним тестированием, выполняемым участниками группы разработки служб SSIS. Результаты могут различаться. Выполните тестирование до подготовки параметров конфигурации, которые могут повлиять на затраты и производительность.

## <a name="properties-to-configure"></a>Свойства, которые нужно настроить

В следующей части сценария конфигурации приведены свойства, которые можно настроить при создании среды выполнения интеграции Azure-SSIS. Полный сценарий и описание PowerShell см. в статье [Развертывание пакетов служб интеграции SQL Server (SSIS) в Azure](tutorial-deploy-ssis-packages-azure-powershell.md).

```powershell
# If your input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$"
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS integration runtime information - This is a Data Factory compute resource for running SSIS packages
$AzureSSISName = "[specify a name for your Azure-SSIS IR]"
$AzureSSISDescription = "[specify a description for your Azure-SSIS IR]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2
# Azure-SSIS IR edition/license info: Standard or Enterprise
$AzureSSISEdition = "Standard" # Standard by default, while Enterprise lets you use advanced/premium features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, while BasePrice lets you bring your existing SQL Server license with Software Assurance to earn cost savings from Azure Hybrid Benefit (AHB) option
# For a Standard_D1_v2 node, up to 4 parallel executions per node are supported, but for other nodes, up to max(2 x number of cores, 8) are currently supported
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info
$SetupScriptContainerSasUri = "" # OPTIONAL to provide SAS URI of blob container where your custom setup script and its associated files are stored
# Virtual network info: Classic or Azure Resource Manager
$VnetId = "[your virtual network resource ID or leave it empty]" # REQUIRED if you use Azure SQL Database with virtual network service endpoints/SQL Managed Instance/on-premises data, Azure Resource Manager virtual network is recommended, Classic virtual network will be deprecated soon
$SubnetName = "[your subnet name or leave it empty]" # WARNING: Please use the same subnet as the one used with your Azure SQL Database with virtual network service endpoints or a different subnet than the one used for your SQL Managed Instance

### SSISDB info
$SSISDBServerEndpoint = "[your server name or managed instance name.DNS prefix].database.windows.net" # WARNING: Please ensure that there is no existing SSISDB, so we can prepare and manage one on your behalf
# Authentication info: SQL or Azure Active Directory (AAD)
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication or leave it empty for AAD authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication or leave it empty for AAD authentication]"
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for Azure SQL Database or leave it empty for SQL Managed Instance]"
```

## <a name="azuressislocation"></a>AzureSSISLocation
**AzureSSISLocation** — расположение для рабочего узла среды выполнения интеграции. Рабочий узел поддерживает постоянное подключение к базе данных каталога SSIS (SSISDB) в базе данных SQL Azure. Задайте для **азурессислокатион** то же расположение, что и для [логического сервера SQL Server](../azure-sql/database/logical-servers.md) , на котором размещена база данных SSISDB, что позволяет среде выполнения интеграции работать как можно эффективнее.

## <a name="azuressisnodesize"></a>AzureSSISNodeSize
Служба "Фабрика данных Azure", включая среду выполнения интеграции MSSQL Integration Services Azure, поддерживает следующие параметры:
-   Standard\_A4\_v2
-   Standard\_A8\_v2
-   Standard\_D1\_v2
-   Standard\_D2\_v2
-   Standard\_D3\_v2
-   Standard\_D4\_v2
-   Стандартный \_ D2 \_ v3
-   Стандартный \_ D4 \_ v3
-   Стандартный \_ D8 \_ v3
-   Стандартный \_ D16 \_ v3
-   Стандартный \_ d32 \_ v3
-   Стандартный \_ D64 \_ v3
-   Стандартный \_ E2 \_ v3
-   Стандартная \_ E4 \_ v3
-   Стандартный \_ E8 \_ v3
-   Стандартный \_ E16 \_ v3
-   Стандартный \_ E32 \_ v3
-   Стандартный \_ E64 \_ v3

В неофициальном внутреннем тестировании командой разработчиков служб SSIS оказалось, что устройства серии D (а не серии A) лучше использовать для выполнения пакета служб SSIS.

-   Коэффициент производительности и цены в серии D выше, чем у серии A, а коэффициент производительности и цены для ряда v3 выше, чем у серии v2.
-   Пропускная способность для ряда D выше, чем у серии A с одинаковой ценой, а пропускная способность для ряда v3 выше, чем у серии V2 по той же цене.
-   Узлы серии v2 Azure-SSIS IR не подходят для выборочной установки, поэтому вместо них следует использовать узлы серии v3. Если вы уже используете узлы серии v2, переключитесь на использование узлов серии v3 как можно скорее.
-   Серия E-это оптимизированные для памяти размеры виртуальных машин, обеспечивающие более высокое соотношение объема памяти и ЦП других компьютеров. Если для пакета требуется много памяти, можно выбрать виртуальную машину серии E.

### <a name="configure-for-execution-speed"></a>Настройка скорости выполнения
Если у вас немного пакетов, и вы хотите, чтобы они выполнялись быстро, используйте информацию на следующей схеме, чтобы выбрать тип виртуальной машины, подходящий для вашего сценария.

Далее предоставлены данные о выполнении одного пакета на одном рабочем узле. Пакет загружает 3 000 000 записей с именами и фамилиями из хранилища BLOB-объектов Azure, создает столбец с полным именем и записывает записи с полным именем длиннее 20 символов в хранилище BLOB-объектов Azure.

Ось y — это количество пакетов, выполненных за один час. Обратите внимание, что это только результат проверки одного пакета, использующего память. Если вы хотите знать пропускную способность пакета, рекомендуется выполнить тест самостоятельно.

![Скорость выполнения пакета среды выполнения интеграции SSIS](media/configure-azure-ssis-integration-runtime-performance/ssisir-execution-speedV2.png)

### <a name="configure-for-overall-throughput"></a>Настройка общей пропускной способности

Если у вас есть много пакетов, и для вас важна общая пропускная способность, используйте информацию на следующей схеме, чтобы выбрать тип виртуальной машины, подходящий для вашего сценария.

Ось y — это количество пакетов, выполненных за один час. Обратите внимание, что это только результат проверки одного пакета, использующего память. Если вы хотите знать пропускную способность пакета, рекомендуется выполнить тест самостоятельно.

![Максимальная общая пропускная способность среды выполнения интеграции SSIS](media/configure-azure-ssis-integration-runtime-performance/ssisir-overall-throughputV2.png)

## <a name="azuressisnodenumber"></a>AzureSSISNodeNumber

**AzureSSISNodeNumber** корректирует масштабируемость среды выполнения интеграции. Пропускная способность среды выполнения интеграции пропорциональна **AzureSSISNodeNumber**. Сначала задайте для **AzureSSISNodeNumber** небольшое значение, контролируйте пропускную способность среды выполнения интеграции, а затем измените значение для своего сценария. Чтобы узнать, как изменить количество рабочих узлов, см. статью [Управление средой выполнения интеграции Azure SSIS](manage-azure-ssis-integration-runtime.md).

## <a name="azuressismaxparallelexecutionspernode"></a>AzureSSISMaxParallelExecutionsPerNode

Если вы уже используете мощный рабочий узел для запуска пакетов, увеличение значения **AzureSSISMaxParallelExecutionsPerNode** может повысить общую пропускную способность среды выполнения интеграции. Если требуется увеличить максимальное значение, необходимо использовать Azure PowerShell для обновления **азурессисмакспараллелексекутионсперноде**. Вы можете определить подходящее значение на основе стоимости пакета и следующих конфигураций для рабочих узлов. Дополнительные сведения см. в статье [Размеры виртуальных машин общего назначения](../virtual-machines/sizes-general.md).

| Размер             | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Максимальная пропускная способность временного хранилища: операций ввода-вывода в секунду / операций чтения в Мбит/с / операций записи в Мбит/с | Максимальное число дисков данных / пропускная способность: ОПЕРАЦИЙ ВВОДА-ВЫВОДА | Максимальное число сетевых адаптеров и ожидаемая производительность сети (Мбит/с) |
|------------------|------|-------------|------------------------|------------------------------------------------------------|-----------------------------------|------------------------------------------------|
| Standard\_D1\_v2 | 1    | 3,5         | 50                     | 3000 / 46 / 23                                             | 2 / 2x500                         | 2 / 750                                        |
| Standard\_D2\_v2 | 2    | 7           | 100                    | 6000 / 93 / 46                                             | 4 / 4x500                         | 2 / 1500                                       |
| Standard\_D3\_v2 | 4    | 14          | 200                    | 12000 / 187 / 93                                           | 8 / 8x500                         | 4 / 3000                                       |
| Standard\_D4\_v2 | 8    | 28          | 400                    | 24000 / 375 / 187                                          | 16 / 16x500                       | 8 / 6000                                       |
| Standard\_A4\_v2 | 4    | 8           | 40                     | 4000 / 80 / 40                                             | 8 / 8x500                         | 4 / 1000                                       |
| Standard\_A8\_v2 | 8    | 16          | 80                     | 8000 / 160 / 80                                            | 16 / 16x500                       | 8 / 2000                                       |
| Стандартный \_ D2 \_ v3 | 2    | 8           | 50                     | 3000 / 46 / 23                                             | 4 и 6x500                         | 2 / 1000                                       |
| Стандартный \_ D4 \_ v3 | 4    | 16          | 100                    | 6000 / 93 / 46                                             | 8 или 12x500                        | 2 / 2000                                       |
| Стандартный \_ D8 \_ v3 | 8    | 32          | 200                    | 12000 / 187 / 93                                           | 16 или 24x500                       | 4 / 4000                                       |
| Стандартный \_ D16 \_ v3| 16   | 64          | 400                    | 24000 / 375 / 187                                          | 32/48x500                        | 8/8000                                       |
| Стандартный \_ d32 \_ v3| 32   | 128         | 800                    | 48000 / 750 / 375                                          | 32/96x500                       | 8 / 16 000                                      |
| Стандартный \_ D64 \_ v3| 64   | 256         | 1600                   | 96000/1000/500                                         | 32/192x500                      | 8/30 000                                      |
| Стандартный \_ E2 \_ v3 | 2    | 16          | 50                     | 3000 / 46 / 23                                             | 4 и 6x500                         | 2 / 1000                                       |
| Стандартная \_ E4 \_ v3 | 4    | 32          | 100                    | 6000 / 93 / 46                                             | 8 или 12x500                        | 2 / 2000                                       |
| Стандартный \_ E8 \_ v3 | 8    | 64          | 200                    | 12000 / 187 / 93                                           | 16 или 24x500                       | 4 / 4000                                       |
| Стандартный \_ E16 \_ v3| 16   | 128         | 400                    | 24000 / 375 / 187                                          | 32/48x500                       | 8/8000                                       |
| Стандартный \_ E32 \_ v3| 32   | 256         | 800                    | 48000 / 750 / 375                                          | 32/96x500                       | 8 / 16 000                                      |
| Стандартный \_ E64 \_ v3| 64   | 432         | 1600                   | 96000/1000/500                                         | 32/192x500                      | 8/30 000                                      |

Ниже приведены рекомендации по настройке правильного значения для свойства **AzureSSISMaxParallelExecutionsPerNode**. 

1. Сначала задайте для него небольшое значение.
2. Немного увеличьте его, чтобы проверить, улучшилась ли общая пропускная способность.
3. Перестаньте увеличивать значение, когда общая пропускная способность достигнет максимального значения.

## <a name="ssisdbpricingtier"></a>SSISDBPricingTier

**СсисдбприЦингтиер** — это ценовая категория для базы данных каталога SSIS (SSISDB) в базе данных SQL Azure. Этот параметр влияет на максимальное число рабочих ролей в экземпляре IR, на скорость помещения операции выполнения пакета в очередь, а также на скорость загрузки журнала выполнения.

-   Если для вас не имеет значения скорость помещения операции выполнения пакета в очередь и загрузки журнала выполнения, вы можете выбрать самый низкий уровень ценовой категории для базы данных. База данных SQL Azure с ценовой категорией "Базовый" поддерживает 8 рабочих ролей в экземпляре среды выполнения интеграции.

-   Выберите более эффективную базу данных, чем базовая, если вам требуется больше 8 рабочих ролей или больше 50 ядер. В противном случае база данных станет узким местом для экземпляра среды выполнения интеграции и отрицательно повлияет на общую производительность.

-   Выберите более мощную базу данных, например S3, если уровень ведения журнала имеет значение подробный. В соответствии с нашим неофициальным тестированием, ценовая категория S3 может поддерживать выполнение пакетов служб SSIS с 2 узлами, 128 параллельных вычислений и подробный уровень ведения журнала.

Вы также можете настроить ценовую категорию базы данных на основе информации об использовании [единицы транзакций базы данных](../azure-sql/database/service-tiers-dtu.md) (DTU), доступной на портале Azure.

## <a name="design-for-high-performance"></a>Проектирование для обеспечения высокой производительности
Проектирование пакета SSIS для запуска в Azure отличается от проектирования пакета для локального выполнения. Для более эффективного выполнения в Azure SSIS IR вместо объединения нескольких независимых задач в том же пакете разделите их на несколько пакетов. Создайте выполнение для каждого пакета, чтобы не приходилось ожидать выполнения каждого из них. Такой подход обеспечивает масштабируемость среды выполнения интеграции Azure-SSIS и улучшает общую пропускную способность.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше о среде выполнения интеграции Azure-SSIS в [этом разделе](concepts-integration-runtime.md#azure-ssis-integration-runtime).