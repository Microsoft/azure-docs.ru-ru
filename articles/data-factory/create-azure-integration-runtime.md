---
title: Создание среды выполнения интеграции Azure в фабрике данных Azure
description: Узнайте, как создать среду выполнения интеграции Azure в фабрике данных Azure, используемую для копирования данных и подготовки к отправке действий преобразования.
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/09/2020
author: nabhishek
ms.author: abnarain
ms.openlocfilehash: ecf2a74a16234084fbac4d1c26157d1703b56a13
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100373068"
---
# <a name="how-to-create-and-configure-azure-integration-runtime"></a>Создание и настройка среды выполнения интеграции Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Среда выполнения интеграции (IR) — это инфраструктура вычислений, которую фабрика данных Azure использует для обеспечения интеграции данных в разных сетевых средах. Дополнительные сведения о среде выполнения интеграции см. в статье [Integration runtime in Azure Data Factory](concepts-integration-runtime.md) (Среда выполнения интеграции в фабрике данных Azure).

Среда выполнения интеграции Azure обеспечивает полностью управляемый вычислительный процесс для выполнения операций перемещения данных и подготовки действий преобразования данных в службах вычислений, например HDInsight. Она размещается в среде Azure и поддерживает подключение к ресурсам в общей сетевой среде с общедоступными конечными точками.

В этом документе вы узнаете, как можно создать и настроить среду выполнения интеграции Azure. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="default-azure-ir"></a>Среда выполнения интеграции Azure по умолчанию
По умолчанию в каждой фабрике данных на сервере есть среда выполнения интеграции Azure, которая поддерживает операции в облачном хранилище данных и вычислительные службы в общедоступной сети. Расположение этого Azure IR разрешается Авторазрешение. Если свойство **connectVia** не задано в определении связанной службы, используется среда выполнения интеграции Azure по умолчанию. Среду выполнения интеграции Azure необходимо явно создавать, если нужно явно определить ее расположение или если нужно виртуально группировать выполнения действий в разных средах выполнения интеграции для управления. 

## <a name="create-azure-ir"></a>Создание среды выполнения интеграции Azure

Чтобы создать и настроить Azure IR, можно использовать следующие процедуры.

### <a name="create-an-azure-ir-via-azure-powershell"></a>Создание Azure IR с помощью Azure PowerShell
Integration Runtime можно создать с помощью командлета PowerShell **Set-AzDataFactoryV2IntegrationRuntime** . Чтобы создать Azure IR, укажите имя, расположение и тип команды. Ниже приведен пример команды для создания среды выполнения интеграции Azure с заданным расположением "Западная Европа":

```powershell
Set-AzDataFactoryV2IntegrationRuntime -DataFactoryName "SampleV2DataFactory1" -Name "MySampleAzureIR" -ResourceGroupName "ADFV2SampleRG" -Type Managed -Location "West Europe"
```  
Для типа среды выполнения интеграции Azure необходимо задать значение **Управляемый**. Указывать детальные сведения вычисления не нужно, так как служба эластично полностью управляется в облаке. Если нужно создать среду выполнения интеграции Azure SSIS, укажите такие сведения о вычислении, как размер и число узлов. Дополнительные сведения см. в статье [Создание среды выполнения интеграции Azure SSIS в фабрике данных Azure](create-azure-ssis-integration-runtime.md).

Можно настроить существующий Azure IR для изменения его расположения с помощью командлета Set-AzDataFactoryV2IntegrationRuntime PowerShell. Дополнительные сведения о расположении среды выполнения интеграции Azure см. в статье [Integration runtime in Azure Data Factory](concepts-integration-runtime.md) (Среда выполнения интеграции в фабрике данных Azure).

### <a name="create-an-azure-ir-via-azure-data-factory-ui"></a>Создание Azure IR с помощью пользовательского интерфейса фабрики данных Azure
Чтобы создать Azure IR с помощью пользовательского интерфейса фабрики данных Azure, выполните следующие действия.

1. На странице **Начало работы** в пользовательском интерфейсе Фабрики данных Azure выберите вкладку [управления](./author-management-hub.md) в крайней области слева.

   ![Кнопка управления на домашней странице](media/doc-common-process/get-started-page-manage-button.png)

1. Выберите элемент **Integration runtimes** (Среды выполнения интеграции) на панели слева. Затем выберите команду **+Создать**.

   ![Снимок экрана, на котором показаны среды выполнения интеграции на левой панели и кнопка + Создать.](media/doc-common-process/manage-new-integration-runtime.png)

1. На странице **Настройка среды выполнения интеграции** выберите **Azure,** локальное размещение и нажмите кнопку **продолжить**. 

1. На следующей странице выберите **Azure** , чтобы создать Azure IR, а затем нажмите кнопку **продолжить**.
   ![Создание среды выполнения интеграции](media/create-azure-integration-runtime/new-azure-integration-runtime.png)

1. Введите имя Azure IR и нажмите кнопку **создать**.
   ![Создание Azure IR](media/create-azure-integration-runtime/create-azure-integration-runtime.png)

1. После завершения создания появится всплывающее уведомление. На странице **среды выполнения интеграции** убедитесь, что в списке отображается только что созданный IR.

## <a name="use-azure-ir"></a>Использование среды выполнения интеграции Azure

После создания среды выполнения интеграции Azure ее можно использовать для определения связанной службы. Ниже представлен пример использования ссылки на среду выполнения интеграции Azure, созданную выше, из связанной службы хранилища Azure.

```json
{
    "name": "MyStorageLinkedService",
    "properties": {
      "type": "AzureStorage",
      "typeProperties": {
        "connectionString": "DefaultEndpointsProtocol=https;AccountName=myaccountname;AccountKey=..."
      },
      "connectVia": {
        "referenceName": "MySampleAzureIR",
        "type": "IntegrationRuntimeReference"
      }   
    }
}

```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о создании других типов среды выполнения интеграции см. в следующих статьях:

- [Создание и настройка локальной среды выполнения интеграции](create-self-hosted-integration-runtime.md)
- [Создание среды выполнения интеграции Azure SSIS в фабрике данных Azure](create-azure-ssis-integration-runtime.md)
