---
title: 'Руководство по Создание конвейера для переноса данных с помощью Azure PowerShell '
description: В этом руководстве вы создадите конвейер фабрики данных Azure с действием копирования с помощью Azure PowerShell.
author: linda33wj
ms.service: data-factory
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 54c296ed8013b9962de9487cfec3e2568c03e738
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100377046"
---
# <a name="tutorial-create-a-data-factory-pipeline-that-moves-data-by-using-azure-powershell"></a>Руководство по Создание конвейера фабрики данных для переноса данных с помощью Azure PowerShell
> [!div class="op_single_selector"]
> * [Обзор и предварительные требования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Мастер копирования](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Шаблон Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API для .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, ознакомьтесь с руководством по [применению действия копирования](../quickstart-create-data-factory-powershell.md). 

В этой статье показано, как создать фабрику данных c конвейером, который копирует данные из Хранилища BLOB-объектов Azure в Базу данных SQL Azure, с помощью Azure PowerShell. Если вы еще не работали с фабрикой данных Azure, перед выполнением действий, описанных в этом руководстве, ознакомьтесь со статьей [Введение в фабрику данных Azure](data-factory-introduction.md).   

В этом руководстве описано, как создать конвейер с одним действием — действием копирования. Действие копирования копирует данные из поддерживаемого хранилища данных в поддерживаемое хранилище данных-приемник. Список хранилищ данных, которые поддерживаются в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Это действие выполняется с помощью глобально доступной службы, обеспечивающей безопасное, надежное и масштабируемое копирование данных между разными хранилищами. Дополнительные сведения о действии копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md).

Конвейер может содержать сразу несколько действий. Два действия можно объединить в цепочку (выполнить одно действие вслед за другим), настроив выходной набор данных одного действия как входной набор данных другого действия. Дополнительные сведения см. в разделе [Несколько действий в конвейере](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

> [!NOTE]
> В этой статье рассматриваются не все командлеты фабрики данных. Полную документацию по этим командлетам см. в [справочнике](/powershell/module/az.datafactory).
> 
> В этом руководстве конвейер данных копирует данные из исходного хранилища данных в целевое. Инструкции по преобразованию данных с помощью Фабрики данных Azure см. в [руководстве по созданию конвейера для преобразования данных с использованием кластера Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

- Выполните предварительные требования, перечисленные в [этом руководстве](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
- Установите **Azure PowerShell**. Следуйте инструкциям по [установке и настройке Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="steps"></a>Шаги
Ниже приведены шаги, которые вы выполните в процессе работы с этим руководством.

1. Создайте **фабрику данных** Azure. На этом этапе вы создадите фабрику данных Azure с именем ADFTutorialDataFactoryPSH. 
1. Создайте в этой фабрике данных **связанные службы**. На этом шаге создайте две связанные службы: службу хранилища Azure и Базу данных SQL Azure. 

    Связанная служба хранилища Azure связывает учетную запись хранения Azure с фабрикой данных. Вы создали контейнер и отправили данные в эту учетную запись хранения в ходе выполнения предварительных [требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

    Связанная служба SQL Azure (AzureSqlLinkedService) связывает Базу данных SQL Azure с фабрикой данных. В этой базе данных хранятся данные, скопированные из хранилища BLOB-объектов. Вы создали таблицу SQL в этой базе данных в ходе выполнения [предварительных требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   
1. Создайте в фабрике данных входные и выходные **наборы данных**.  

    Связанная служба хранилища Azure указывает строку подключения, которую фабрика данных использует во время выполнения, чтобы подключиться к учетной записи хранения Azure. А входной набор данных больших двоичных объектов определяет контейнер и папку с входными данными.  

    Аналогичным образом связанная служба Базы данных SQL Azure указывает строку подключения, которую служба Фабрики данных использует во время выполнения, чтобы подключиться к базе данных. А выходной набор данных таблицы SQL определяет таблицу в базе данных, в которую копируются данные из хранилища BLOB-объектов.
1. Создайте **конвейер** в фабрике данных. На этом этапе вы создадите конвейер с действием копирования.   

    Действие копирования копирует данные из большого двоичного объекта в Хранилище BLOB-объектов Azure в таблицу в Базе данных SQL Azure. Его можно использовать, чтобы копировать данные из любого поддерживаемого источника в любое расположение. Список поддерживаемых хранилищ данных см. в [этом разделе](data-factory-data-movement-activities.md#supported-data-stores-and-formats). 
1. Выполните мониторинг конвейера. На этом этапе вы **отследите** срезы входных и выходных наборов данных с помощью PowerShell.

## <a name="create-a-data-factory"></a>Создание фабрики данных
> [!IMPORTANT]
> Выполните [предварительные требования, необходимые для работы с этим руководством](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), если вы еще этого не сделали.   

Фабрика данных может иметь один или несколько конвейеров. Конвейер может содержать одно или несколько действий. Это может быть, например, действие копирования данных из исходного хранилища в целевое или действие HDInsight Hive для выполнения скрипта Hive, преобразующего входные данные в выходные данные продукта. Начнем с создания фабрики данных.

1. Запустите **PowerShell**. Не закрывайте Azure PowerShell, пока выполняются описанные в учебнике инструкции. Если закрыть и снова открыть это окно, то придется вновь выполнять эти команды.

    Выполните следующую команду и введите имя пользователя и пароль, которые используются для входа на портал Azure.

    ```powershell
    Connect-AzAccount
    ```   

    Чтобы просмотреть все подписки для этой учетной записи, выполните следующую команду:

    ```powershell
    Get-AzSubscription
    ```

    Выполните следующую команду, чтобы выбрать подписку, с которой вы собираетесь работать. Замените **&lt;NameOfAzureSubscription**&gt; именем своей подписки Azure.

    ```powershell
    Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
    ```
1. Создайте группу ресурсов Azure с именем **ADFTutorialResourceGroup** , выполнив следующую команду:

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

    Некоторые действия, описанные в этом учебнике, предполагают, что вы используете группу ресурсов с именем **ADFTutorialResourceGroup**. Если вы используете другую группу ресурсов, укажите ее вместо ADFTutorialResourceGroup.
1. Выполните командлет **New-AzDataFactory**, чтобы создать фабрику данных с именем **ADFTutorialDataFactoryPSH**.  

    ```powershell
    $df=New-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH –Location "West US"
    ```
    Это имя уже может использоваться. В этом случае присвойте фабрике данных уникальное имя, добавив к нему префикс или суффикс (например, ADFTutorialDataFactoryPSH05152017) и выполните команду еще раз.  

Обратите внимание на следующие моменты.

* Имя фабрики данных Azure должно быть глобально уникальным. При возникновении следующей ошибки измените имя (например, на ваше_имя_ADFTutorialDataFactoryPSH). Используйте это имя вместо ADFTutorialFactoryPSH при выполнении шагов в этом руководстве. Ознакомьтесь с [правилами именования](data-factory-naming-rules.md) артефактов фабрики данных.

    ```
    Data factory name "ADFTutorialDataFactoryPSH" is not available
    ```
* Чтобы создавать экземпляры фабрики данных, вы должны быть участником или администратором подписки Azure.
* В будущем имя фабрики данных может быть зарегистрировано в качестве DNS-имени и, следовательно, стать отображаемым.
* Может появиться следующее сообщение об ошибке: **This subscription is not registered to use namespace Microsoft.DataFactory** (Подписка не зарегистрирована для использования пространства имен Microsoft.DataFactory). В этом случае выполните одно из следующих действий и повторите попытку публикации.

  * Чтобы зарегистрировать поставщик фабрики данных Azure, выполните следующую команду в Azure PowerShell:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

    Чтобы убедиться, что поставщик фабрики данных зарегистрирован, выполните следующую команду:

    ```powershell
    Get-AzResourceProvider
    ```
  * Войдите на [портал Azure](https://portal.azure.com), используя свою подписку Azure. Откройте колонку фабрики данных или создайте фабрику данных на портале Azure. Поставщик будет зарегистрирован автоматически.

## <a name="create-linked-services"></a>Создание связанных служб
Связанная служба в фабрике данных связывает хранилища данных и службы вычислений с фабрикой данных. В этом руководстве не используются службы вычислений, например Azure HDInsight или Azure Data Lake Analytics. Вы используете два хранилища данных — служба хранилища Azure (источник) и база данных SQL Azure (конечное хранилище). 

Поэтому нужно создать две связанные службы с именами AzureStorageLinkedService и AzureSqlLinkedService и такими типами: AzureStorage и AzureSqlDatabase.  

Связанная служба хранилища Azure связывает учетную запись хранения Azure с фабрикой данных. В этой учетной записи хранения вы создали контейнер и отправили в нее данные в ходе выполнения предварительных [требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

Связанная служба SQL Azure (AzureSqlLinkedService) связывает Базу данных SQL Azure с фабрикой данных. В этой базе данных хранятся данные, скопированные из хранилища BLOB-объектов. Вы создали пустую таблицу в этой базе данных в ходе выполнения [предварительных требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). 

### <a name="create-a-linked-service-for-an-azure-storage-account"></a>Создание связанной службы для учетной записи хранения Azure
На этом шаге вы свяжете учетную запись хранения Azure с фабрикой данных.

1. Создайте JSON-файл **AzureStorageLinkedService.json** а папке **C:\ADFGetStartedPSH** со следующим содержимым: (Если папки ADFGetStartedPSH нет, создайте ее.)

    > [!IMPORTANT]
    > Перед сохранением файла замените значения &lt;accountname&gt; и &lt;accountkey&gt; на имя вашей учетной записи хранения Azure и ее ключ. 

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        }
     }
    ``` 
1. В **Azure PowerShell** перейдите в папку **ADFGetStartedPSH**.
1. Чтобы создать связанную службу, выполните командлет **New-AzDataFactoryLinkedService**: **AzureStorageLinkedService**. В этом и в других командлетах фабрики данных, которые используются в этом руководстве, требуется передача значений для параметров **ResourceGroupName** и **DataFactoryName**. Кроме того, вы можете передать объект DataFactory, возвращенный командлетом New-AzDataFactory, не вводя параметры ResourceGroupName и DataFactoryName при каждом запуске командлета. 

    ```powershell
    New-AzDataFactoryLinkedService $df -File .\AzureStorageLinkedService.json
    ```
    Пример выходных данных:

    ```
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ``` 

    Связанную службу можно создать и другим способом. Вместо объекта DataFactory укажите имя группы ресурсов и фабрики данных.  

    ```powershell
    New-AzDataFactoryLinkedService -ResourceGroupName ADFTutorialResourceGroup -DataFactoryName <Name of your data factory> -File .\AzureStorageLinkedService.json
    ```

### <a name="create-a-linked-service-for-azure-sql-database"></a>Создание связанной службы для Базы данных SQL Azure
На этом шаге вы свяжете Базу данных SQL Azure с фабрикой данных.

1. Создайте в папке C:\ADFGetStartedPSH файл JSON с именем AzureSqlLinkedService.json и добавьте в него приведенное ниже содержимое.

    > [!IMPORTANT]
    > Вместо &lt;servername&gt;, &lt;databasename&gt;, &lt;username@servername&gt; и &lt;password&gt; укажите имя своего сервера, имя базы данных, имя учетной записи пользователя и пароль.

    ```json
    {
        "name": "AzureSqlLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<server>.database.windows.net,1433;Database=<databasename>;User ID=<user>@<server>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
     }
    ```
1. Чтобы создать связанную службу, выполните следующую команду:

    ```powershell
    New-AzDataFactoryLinkedService $df -File .\AzureSqlLinkedService.json
    ```

    Пример выходных данных:

    ```
    LinkedServiceName : AzureSqlLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ```

   Убедитесь, что параметр **Разрешить доступ к службам Azure** включен для вашего сервера. Чтобы проверить и при необходимости включить этот параметр, выполните следующие действия.

    1. Войдите на [портал Azure](https://portal.azure.com)
    1. Щелкните **Больше служб** слева и выберите **Серверы SQL** в категории **Базы данных**.
    1. Выберите в списке свой сервер SQL Server.
    1. В колонке SQL Server щелкните ссылку **Показать параметры брандмауэра**.
    1. В колонке **Параметры брандмауэра** щелкните **ВКЛ** для параметра **Разрешить доступ к службам Azure**.
    1. Нажмите кнопку **Сохранить** на панели инструментов. 

## <a name="create-datasets"></a>Создание наборов данных
На предыдущем шаге вы создали связанные службы, связывающие учетную запись хранения Azure и Базу данных SQL Azure с фабрикой данных. На этом этапе вы определите два набора данных (InputDataset и OutputDataset), представляющие входные и выходные данные в хранилищах данных, на которые ссылаются службы AzureStorageLinkedService и AzureSqlLinkedService соответственно.

Связанная служба хранилища Azure указывает строку подключения, которую фабрика данных использует во время выполнения, чтобы подключиться к учетной записи хранения Azure. А входной набор данных больших двоичных объектов (InputDataset) определяет контейнер и папку с входными данными.  

Аналогичным образом связанная служба Базы данных SQL Azure указывает строку подключения, которую служба Фабрики данных использует во время выполнения, чтобы подключиться к базе данных. А выходной набор данных таблицы SQL (OututDataset) определяет таблицу в базе данных, в которую копируются данные из хранилища BLOB-объектов. 

### <a name="create-an-input-dataset"></a>Создание входного набора данных
На этом этапе вы создадите набор данных с именем InputDataset. Он указывает на файл большого двоичного объекта (emp.txt) в корневой папке контейнера больших двоичных объектов в службе хранилища Azure, которая представлена связанной службой AzureStorageLinkedService. Если не указать значение fileName (или пропустить его), данные из всех больших двоичных объектов в папке входных данных копируются в целевое расположение. В этом руководстве вы укажете значение параметра fileName.  

1. Создайте файл JSON с именем **InputDataset.json** в папке **C:\ADFGetStartedPSH** со следующим содержимым:

    ```json
    {
        "name": "InputDataset",
        "properties": {
            "structure": [
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "emp.txt",
                "folderPath": "adftutorial/",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
                }
            },
            "external": true,
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
     }
    ```

    В следующей таблице приведены описания свойств JSON, используемых в этом фрагменте кода.

    | Свойство | Описание |
    |:--- |:--- |
    | type | Для свойства типа задано значение **AzureBlob**, так как данные хранятся в хранилище BLOB-объектов Azure. |
    | linkedServiceName | Ссылается на созданную ранее службу **AzureStorageLinkedService**. |
    | folderPath | Определяет **контейнер** больших двоичных объектов и **папку**, которая содержит входные большие двоичные объекты. В этом руководстве adftutorial — это контейнер больших двоичных объектов, а созданная папка является корневой. | 
    | fileName | Это необязательное свойство. Если это свойство не указано, выбираются все файлы из папки folderPath. В этом руководстве для свойства fileName указывается значение **emp.txt**, чтобы обрабатывался только этот файл. |
    | format -> type |Входной файл имеет текстовый формат, поэтому укажите значение **TextFormat**. |
    | columnDelimiter | Столбцы во входном файле разделяются **запятыми (`,`)** . |
    | frequency/interval | Для свойства frequency задано значение **Hour**, а для свойства interval — значение **1**. Это означает, что срезы входных данных будут создаваться **каждый час**. Иными словами, служба фабрики данных будет искать входные данные в корневой папке указанного контейнера BLOB-объектов (**adftutorial**) каждый час. Поиск данных осуществляется в пределах времени начала и времени окончания для конвейера, но не перед этим периодом или после него.  |
    | external | Это свойство имеет значение **true**, если этот конвейер не создает данные. В этом руководстве входные данные находятся в файле emp.txt, который не создается этим конвейером, поэтому мы присвоим этому свойству значение true. |

    Дополнительные сведения об этих свойствах JSON см. в [этом разделе](data-factory-azure-blob-connector.md#dataset-properties).
1. Выполните следующую команду для создания набора данных фабрики данных.

    ```powershell  
    New-AzDataFactoryDataset $df -File .\InputDataset.json
    ```
    Пример выходных данных:

    ```
    DatasetName       : InputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureBlobDataset
    Policy            : Microsoft.Azure.Management.DataFactories.Common.Models.Policy
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

### <a name="create-an-output-dataset"></a>Создание выходного набора данных
На этом этапе мы создадим выходной набор данных с именем **OutputDataset**. Этот набор данных указывает на таблицу SQL в Базе данных SQL Azure (представляется **AzureSqlLinkedService**). 

1. Создайте файл JSON с именем **OutputDataset.json** в папке **C:\ADFGetStartedPSH** со следующим содержимым:

    ```json
    {
        "name": "OutputDataset",
        "properties": {
            "structure": [
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": {
                "tableName": "emp"
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```

    В следующей таблице приведены описания свойств JSON, используемых в этом фрагменте кода.

    | Свойство | Описание |
    |:--- |:--- |
    | type | Свойство type имеет значение **AzureSqlTable**, так как данные копируются в таблицу в Базе данных SQL Azure. |
    | linkedServiceName | Ссылается на созданную ранее службу **AzureSqlLinkedService**. |
    | tableName | Указывает **таблицу**, в которую копируются данные. | 
    | frequency/interval | Для свойства frequency задано значение **Hour**, а для interval — **1**. Это означает, что срезы выходных данных создаются **каждый час** в пределах времени начала и времени окончания для конвейера, но не перед этим периодом или после него.  |

    В таблице emp в базе данных есть три столбца: **ID**, **FirstName** и **LastName**. ID — это столбец для идентификаторов, поэтому здесь вам нужно указать только значения **FirstName** и **LastName**.

    Дополнительные сведения об этих свойствах JSON см. в [этом разделе](data-factory-azure-sql-connector.md#dataset-properties).
1. Выполните следующую команду для создания набора данных фабрики данных.

    ```powershell   
    New-AzDataFactoryDataset $df -File .\OutputDataset.json
    ```

    Пример выходных данных:

    ```
    DatasetName       : OutputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureSqlTableDataset
    Policy            :
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

## <a name="create-a-pipeline"></a>Создание конвейера
На этом этапе вы создадите конвейер с **действием копирования**, которое использует **InputDataset** в качестве входных данных и **OutputDataset** в качестве выходных.

Сейчас на основе этого набора настраивается расписание. В этом руководстве выходной набор данных создает срез раз в час. Для конвейера настроено время начала и время окончания с разницей в сутки. Таким образом, конвейер создает 24 среза для выходного набора данных. 

1. Создайте файл JSON с именем **ADFTutorialPipeline.json** в папке **C:\ADFGetStartedPSH** со следующим содержимым:

    ```json   
    {
      "name": "ADFTutorialPipeline",
      "properties": {
        "description": "Copy data from a blob to Azure SQL table",
        "activities": [
          {
            "name": "CopyFromBlobToSQL",
            "type": "Copy",
            "inputs": [
              {
                "name": "InputDataset"
              }
            ],
            "outputs": [
              {
                "name": "OutputDataset"
              }
            ],
            "typeProperties": {
              "source": {
                "type": "BlobSource"
              },
              "sink": {
                "type": "SqlSink",
                "writeBatchSize": 10000,
                "writeBatchTimeout": "60:00:00"
              }
            },
            "Policy": {
              "concurrency": 1,
              "executionPriorityOrder": "NewestFirst",
              "retry": 0,
              "timeout": "01:00:00"
            }
          }
        ],
        "start": "2017-05-11T00:00:00Z",
        "end": "2017-05-12T00:00:00Z"
      }
    } 
    ```
    Обратите внимание на следующие моменты.

   - В разделе действий доступно только одно действие, параметр **type** которого имеет значение **Copy**. Дополнительные сведения о действии копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md). В решениях фабрики данных можно также использовать [действия преобразования данных](data-factory-data-transformation-activities.md).
   - Для этого действия параметру input присвоено значение **InputDataset**, а параметру output — значение **OutputDataset**. 
   - В разделе **typeProperties** в качестве типа источника указано **BlobSource**, а в качестве типа приемника — **SqlSink**. Список хранилищ данных, поддерживаемых действием копирования в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Чтобы узнать, как использовать конкретное хранилище данных в качестве источника или приемника, щелкните ссылку в таблице.  

     Замените значение свойства **start** текущей датой, а значение свойства **end** — датой следующего дня. Можно указать только часть даты и пропустить временную часть указанной даты и времени. Например, 2016-02-03, что эквивалентно 2016-02-03T00:00:00Z.

     Даты начала и окончания должны быть в [формате ISO](https://en.wikipedia.org/wiki/ISO_8601). Пример: 2016-10-14T16:32:41Z. Время **окончания** указывать не обязательно, однако в этом примере мы будем его использовать. 

     Если не указать значение свойства **end**, оно вычисляется по формуле "**время начала + 48 часов**". Чтобы запустить конвейер в течение неопределенного срока, укажите значение **9999-09-09** в качестве значения свойства **end**.

     В примере выше получено 24 среза данных, так как они создаются каждый час.

     Описание свойств JSON в определении конвейера см. в статье [Конвейеры и действия в фабрике данных Azure](data-factory-create-pipelines.md). Описание свойств JSON в определении действия копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md). Описание свойств JSON, поддерживаемых BlobSource, см. в статье о [соединителе больших двоичных объектов Azure](data-factory-azure-blob-connector.md). Описание свойств JSON, поддерживаемых SqlSink, см. в статье о [соединителе базы данных SQL Azure](data-factory-azure-sql-connector.md).
1. Выполните следующую команду для создания таблицы фабрики данных.

    ```powershell   
    New-AzDataFactoryPipeline $df -File .\ADFTutorialPipeline.json
    ```

    Пример выходных данных: 

    ```
    PipelineName      : ADFTutorialPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.PipelinePropertie
    ProvisioningState : Succeeded
    ```

**Поздравляем!** Фабрика данных Azure с конвейером, который копирует данные из Хранилища BLOB-объектов Azure в Базу данных SQL Azure, успешно создана. 

## <a name="monitor-the-pipeline"></a>Мониторинг конвейера
На этом шаге Azure PowerShell будет использоваться для мониторинга процессов в фабрике данных Azure.

1. Замените &lt;DataFactoryName&gt; именем своей фабрики данных. Выполните командлет **Get-AzDataFactory** и назначьте выходные данные переменной $df.

    ```powershell  
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name <DataFactoryName>
    ```

    Пример:
    ```powershell
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH0516
    ```

    Затем выведите содержимое $df и просмотрите следующие выходные данные: 

    ```
    PS C:\ADFGetStartedPSH> $df

    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DataFactoryId     : 6f194b34-03b3-49ab-8f03-9f8a7b9d3e30
    ResourceGroupName : ADFTutorialResourceGroup
    Location          : West US
    Tags              : {}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DataFactoryProperties
    ProvisioningState : Succeeded
    ```
1. Выполните командлет **Get-AzDataFactorySlice**, чтобы получить сведения обо всех срезах набора данных **OutputDataset**, который является выходным набором конвейера.  

    ```powershell   
    Get-AzDataFactorySlice $df -DatasetName OutputDataset -StartDateTime 2017-05-11T00:00:00Z
    ```

   Это значение должно соответствовать значению параметра **Start** в JSON-файле конвейера. Вы увидите 24 среза — по одному для каждого часа с 00:00 текущего дня до 00:00 следующего дня.

   Ниже приведены три примера срезов из выходных данных. 

    ``` 
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 11:00:00 PM
    End               : 5/12/2017 12:00:00 AM
    RetryCount        : 0
    State             : Ready
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 9:00:00 PM
    End               : 5/11/2017 10:00:00 PM
    RetryCount        : 0
    State             : InProgress
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 8:00:00 PM
    End               : 5/11/2017 9:00:00 PM
    RetryCount        : 0
    State             : Waiting
    SubState          : ConcurrencyLimit
    LatencyStatus     :
    LongRetryCount    : 0
    ```
1. Выполните командлет **Get-AzDataFactoryRun**, чтобы получить сведения о действиях, выполняемых для **конкретного** среза. Скопируйте значение даты и времени в выходных данных предыдущей команды, чтобы задать значение параметра StartDateTime. 

    ```powershell  
    Get-AzDataFactoryRun $df -DatasetName OutputDataset -StartDateTime "5/11/2017 09:00:00 PM"
    ```

   Пример выходных данных: 

    ```
    Id                  : c0ddbd75-d0c7-4816-a775-704bbd7c7eab_636301332000000000_636301368000000000_OutputDataset
    ResourceGroupName   : ADFTutorialResourceGroup
    DataFactoryName     : ADFTutorialDataFactoryPSH0516
    DatasetName         : OutputDataset
    ProcessingStartTime : 5/16/2017 8:00:33 PM
    ProcessingEndTime   : 5/16/2017 8:01:36 PM
    PercentComplete     : 100
    DataSliceStart      : 5/11/2017 9:00:00 PM
    DataSliceEnd        : 5/11/2017 10:00:00 PM
    Status              : Succeeded
    Timestamp           : 5/16/2017 8:00:33 PM
    RetryAttempt        : 0
    Properties          : {}
    ErrorMessage        :
    ActivityName        : CopyFromBlobToSQL
    PipelineName        : ADFTutorialPipeline
    Type                : Copy  
    ```

Полную документацию по командлетам фабрики данных см. в [этом справочнике](/powershell/module/az.datafactory).

## <a name="summary"></a>Сводка
В этом руководстве показано, как создать фабрику данных Azure для копирования данных из большого двоичного объекта Azure в Базу данных SQL Azure. Вы использовали PowerShell для создания фабрики данных, связанных служб, наборов данных и конвейера. Вот обобщенные действия, которые вы выполнили в этом руководстве:  

1. создание **фабрики данных Azure**;
1. Создание **связанных служб**.

   а. **Служба хранилища Azure** — создание связанной службы для связи с учетной записью хранения Azure, которая содержит входные данные.     
   b. **SQL Azure** — создание связанной службы для связи с Базой данных SQL Azure, которая содержит выходные данные.
1. Создание **наборов данных** , которые описывают входные и выходные данные для конвейеров.
1. Создание **конвейера** с **BlobSource** в качестве источника и **SqlSink** в качестве приемника с помощью **действия копирования**.

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве показано, как использовать Хранилище BLOB-объектов Azure и Базу данных SQL Azure как исходное и целевое хранилища данных в операции копирования. В следующей таблице приведен список хранилищ данных, которые поддерживаются в качестве источников и целевых расположений для действия копирования. 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Чтобы получить дополнительные сведения о том, как скопировать данные в хранилище данных или из него, щелкните ссылку для хранилища данных в таблице. 

