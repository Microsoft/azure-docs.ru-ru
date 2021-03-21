---
title: Преобразование данных в облаке с помощью PowerShell
description: Этот скрипт PowerShell преобразовывает данные в облаке путем выполнения программы Spark на кластере Azure HDInsight Spark.
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/12/2017
ms.openlocfilehash: 2e1178f91344038fc86f1411fca1beafbe901d05
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100389830"
---
# <a name="powershell-script---transform-data-in-cloud-using-azure-data-factory"></a>Сценарий PowerShell. Преобразование данных в облаке с помощью фабрики данных Azure

Этот пример сценария PowerShell создает конвейер, который преобразовывает данные в облаке путем выполнения программы Spark на кластере Azure HDInsight Spark. 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Предварительные требования
* **Учетная запись хранения Azure.** Создайте входной файл и сценарий Python и передайте их в службу хранилища Azure. Выходные данные программы Spark хранятся в этой учетной записи хранения. Кластер Spark по запросу использует ту же учетную запись хранения, что и его основное хранилище.  

### <a name="upload-python-script-to-your-blob-storage-account"></a>Отправка скрипта Python в учетную запись хранилища BLOB-объектов
1. Создайте файл Python с именем **WordCount_Spark.py** со следующим содержимым: 

    ```python
    import sys
    from operator import add
    
    from pyspark.sql import SparkSession
    
    def main():
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
            
        lines = spark.read.text("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/inputfiles/minecraftstory.txt").rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' ')) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(add)
        counts.saveAsTextFile("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/outputfiles/wordcount")
        
        spark.stop()
    
    if __name__ == "__main__":
        main()
    ```
2. Замените свойство **storageAccountName&lt;&gt;** именем своей учетной записи хранения Azure. Затем сохраните файл. 
3. В хранилище BLOB-объектов Azure создайте контейнер с именем **adftutorial**, если он не существует. 
4. Создайте папку с именем **spark**.
5. Создайте вложенную папку с именем **script** в папке **spark**. 
6. Отправьте файл **WordCount_Spark.py** во вложенную папку **script**. 


### <a name="upload-the-input-file"></a>Отправка входного файла
1. Создайте файл с определенным текстом и назовите его **minecraftstory.txt**. Программа Spark подсчитывает количество слов в этом тексте. 
2. Создайте вложенную папку `inputfiles` в папке `spark` в контейнере больших двоичных объектов. 
3. Отправьте файл `minecraftstory.txt` во вложенную папку `inputfiles`. 

## <a name="sample-script"></a>Пример скрипта
> [!IMPORTANT]
> Этот скрипт создает JSON-файлы, определяющие сущности фабрики данных (связанную службу, набор данных и конвейер) на жестком диске в папке c:\.

[!code-powershell[main](../../../powershell_scripts/data-factory/transform-data-using-spark/transform-data-using-spark.ps1 "Transform data using Spark")]

## <a name="clean-up-deployment"></a>Очистка развертывания

После выполнения примера скрипта можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```
Чтобы удалить фабрику данных из группы ресурсов, выполните следующую команду: 

```powershell
Remove-AzDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды:

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [Set-AzDataFactoryV2](/powershell/module/az.datafactory/set-Azdatafactoryv2) | Создали фабрику данных. |
| [Set-AzDataFactoryV2LinkedService](/powershell/module/az.datafactory/set-Azdatafactoryv2linkedservice) | Создает в этой фабрике данных связанную службу. Связанная служба вычисляет или привязывает хранилище данных к фабрике данных. |
| [Set-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/set-Azdatafactoryv2pipeline) | Создает конвейер в фабрике данных. Конвейер содержит одно или несколько действий, выполняющих определенную операцию. В этом конвейере действие Spark преобразовывает данные путем выполнения программы в кластере Azure HDInsight Spark. |
| [Invoke-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/invoke-Azdatafactoryv2pipeline) | Создает выполнение для конвейера. Другими словами, запускает конвейер. |
| [Get-AzDataFactoryV2ActivityRun](/powershell/module/az.datafactory/get-Azdatafactoryv2activityrun) | Получает сведения о выполнении действия в конвейере. 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для Azure Data Factory приведены [здесь](../samples-powershell.md).