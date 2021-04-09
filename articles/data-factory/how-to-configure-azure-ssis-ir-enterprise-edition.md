---
title: Подготовка выпуска Enterprise Edition для среды выполнения интеграции Azure Integration Services
description: В этой статье описаны возможности выпуска Enterprise Edition для среды выполнения интеграции Azure Integration Services, а также процедура его подготовки.
ms.service: data-factory
ms.topic: conceptual
ms.date: 07/09/2020
author: swinarko
ms.author: sawinark
ms.openlocfilehash: f700729c2d059648b1c3a7e451526aefcb436818
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100387552"
---
# <a name="provision-enterprise-edition-for-the-azure-ssis-integration-runtime"></a>Подготовка выпуска Enterprise Edition для среды выполнения интеграции Azure Integration Services

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Выпуск Enterprise Edition среды выполнения интеграции Azure SSIS позволяет использовать следующие расширенные возможности и возможности уровня "Премиум":
-   компоненты отслеживания измененных данных;
-   соединители Oracle, Teradata и SAP BW;
-   соединители и преобразования SQL Server Analysis Services и Azure Analysis Services;
-   Преобразования нечетких группирований и нечетких уточняющих запросов
-   Преобразования извлечения терминов и поиска по терминам

Некоторым из этих возможностей требуется установка дополнительных компонентов для настройки среды выполнения интеграции Azure SSIS. Дополнительные сведения об установке дополнительных компонентов см. в статье [Пользовательская установка для среды выполнения интеграции Azure SSIS](how-to-configure-azure-ssis-ir-custom-setup.md).

## <a name="enterprise-features"></a>Корпоративные функции

| **Корпоративные функции** | **Описания** |
|---|---|
| Компоненты CDC | Источник CDC, задача управления и преобразование "Разделитель" предварительно установлены в среде выполнения интеграции Azure SSIS Enterprise Edition. Чтобы подключиться к Oracle, необходимо установить CDC Designer и службу на другом компьютере. |
| Соединители Oracle | Диспетчер подключений, источник и назначение Oracle предварительно установлены в среде выполнения интеграции Azure SSIS Enterprise Edition. Также необходимо установить драйвер интерфейса OCI (Oracle Call Interface) и при необходимости настроить Oracle Transport Network Substrate в среде выполнения интеграции Azure SSIS. Дополнительные сведения см. в разделе [Выборочная установка среды выполнения интеграции Azure-SSIS](how-to-configure-azure-ssis-ir-custom-setup.md). |
| Соединители Teradata | Необходимо установить диспетчер подключений, источник и назначение Teradata, а также API параллельного транспортера Teradata и драйвер ODBC Teradata в среде выполнения интеграции Azure SSIS Enterprise Edition. Дополнительные сведения см. в разделе [Выборочная установка среды выполнения интеграции Azure-SSIS](how-to-configure-azure-ssis-ir-custom-setup.md). |
| Соединители SAP BW | Диспетчер подключений, источник и назначение SAP BW предварительно установлены в среде выполнения интеграции Azure SSIS Enterprise Edition. Необходимо также установить драйвер SAP BW в среде выполнения интеграции Azure SSIS. Эти соединители поддерживают SAP BW версии 7.0 или более ранних. Чтобы подключиться к более поздним версиям SAP BW или другим продуктам SAP, можно приобрести и установить соединители SAP от сторонних независимых поставщиков программного обеспечения в среде выполнения интеграции Azure SSIS. Дополнительные сведения об установке дополнительных компонентов см. в статье [Пользовательская установка для среды выполнения интеграции Azure SSIS](how-to-configure-azure-ssis-ir-custom-setup.md). |
| Компоненты Analysis Services               | Назначение обучения модели интеллектуального анализа данных, назначение обработки измерений, назначение обработки секций, а также преобразование запросов интеллектуального анализа данных предварительно установлены в среде выполнения интеграции Azure SSIS. Все эти компоненты поддерживают SQL Server Analysis Services, однако Azure Analysis Services поддерживается только назначением обработки секций. Чтобы подключиться к SQL Server Analysis Services, необходимо [настроить учетные данные проверки подлинности Windows в SSISDB](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth). Помимо этих компонентов, задача "DDL выполнения Analysis Services", задача "Обработка средствами Analysis Services" и задача "Запрос интеллектуального анализа данных" также предварительное установлены в среде выполнения интеграции Azure SSIS выпуска Standard и Enterprise Edition. |
| Преобразования нечетких группирований и нечетких уточняющих запросов  | Преобразования нечетких группирований и нечетких уточняющих запросов предварительно установлены в среде выполнения интеграции Azure SSIS Enterprise Edition. При хранении эталонных данных эти компоненты поддерживают и сервер SQL Server, и Базу данных SQL Azure. |
| Преобразования извлечения терминов и поиска по терминам | Преобразования извлечения терминов и поиска по терминам предварительно установлены в среде выполнения интеграции Azure SSIS Enterprise Edition. При хранении эталонных данных эти компоненты поддерживают и сервер SQL Server, и Базу данных SQL Azure. |

## <a name="instructions"></a>Instructions

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

1.  Скачайте и установите [Azure PowerShell](/powershell/azure/install-az-ps).

2.  При подготовке или перенастройке среды выполнения интеграции Azure SSIS с помощью PowerShell, прежде чем ее запускать, выполните команду `Set-AzDataFactoryV2IntegrationRuntime` с **Enterprise** в качестве значения для параметра **Edition**. Ниже приведен пример скрипта:

    ```powershell
    $MyAzureSsisIrEdition = "Enterprise"

    Set-AzDataFactoryV2IntegrationRuntime -DataFactoryName $MyDataFactoryName
                                               -Name $MyAzureSsisIrName
                                               -ResourceGroupName $MyResourceGroupName
                                               -Edition $MyAzureSsisIrEdition

    Start-AzDataFactoryV2IntegrationRuntime -DataFactoryName $MyDataFactoryName
                                                 -Name $MyAzureSsisIrName
                                                 -ResourceGroupName $MyResourceGroupName
    ```

## <a name="next-steps"></a>Дальнейшие действия

-   [Пользовательская установка для среды выполнения интеграции Azure SSIS](how-to-configure-azure-ssis-ir-custom-setup.md)

-   [Разработка платных или лицензионных пользовательских компонентов для среды выполнения интеграции Azure SSIS](how-to-develop-azure-ssis-ir-licensed-components.md)