---
title: Использование MapReduce и PowerShell с Apache Hadoop в Azure HDInsight
description: Узнайте, как использовать PowerShell для удаленного выполнения заданий MapReduce с помощью Apache Hadoop в HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/08/2020
ms.openlocfilehash: 16c6c5e317591b70c3a1300453093fc715e213fb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98939680"
---
# <a name="run-mapreduce-jobs-with-apache-hadoop-on-hdinsight-using-powershell"></a>Выполнение заданий MapReduce с помощью PowerShell и Apache Hadoop в HDInsight

[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

В этом документе приведен пример использования Azure PowerShell для выполнения задания MapReduce в Hadoop на кластере HDInsight.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Hadoop в HDInsight. См. [Создание кластеров под управлением Linux в HDInsight с помощью портала Azure](../hdinsight-hadoop-create-linux-clusters-portal.md).

* Установите [модуль Az](/powershell/azure/) для PowerShell.

## <a name="run-a-mapreduce-job"></a>Выполнение задания MapReduce

Azure PowerShell предоставляет *командлеты* , позволяющие удаленно запускать задания MapReduce в HDInsight. На внутреннем уровне PowerShell выполняет вызовы REST к [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) (прежнее название — Templeton) в кластере HDInsight.

При выполнении заданий MapReduce в удаленном кластере HDInsight используются следующие командлеты:

|Командлет | Описание |
|---|---|
|Connect-AzAccount|выполняет аутентификацию Azure PowerShell для подписки Azure.|
|New-AzHDInsightMapReduceJobDefinition|создает *определение задания*, используя заданную информацию MapReduce.|
|Start-AzHDInsightJob|отправляет определение задания в HDInsight и запускает задание. Будет возвращен объект *задания*.|
|Wait-AzHDInsightJob|проверяет состояние задания с помощью объекта-задания. Он ожидает завершения задания или превышения времени ожидания.|
|Get-AzHDInsightJobOutput|используется для получения выходных данных задания.|

Следующие шаги показывают, как использовать эти командлеты для выполнения задания в кластере HDInsight:

1. С помощью редактора Сохраните следующий код как **mapreducejob.ps1**.

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-mapreduce/use-mapreduce.ps1?range=5-69)]

2. Откройте новую командную строку **Azure PowerShell** . Перейдите к расположению файла **mapreducejob.ps1** , а затем используйте следующую команду для запуска сценария:

    ```azurepowershell
    .\mapreducejob.ps1
    ```

    При запуске скрипта будет предложено ввести имя кластера HDInsight и имя входа в кластер. Вам также может быть предложено аутентифицироваться в подписке Azure.

3. По завершении задания выходные данные будут иметь примерно следующий вид.

    ```output
    Cluster         : CLUSTERNAME
    ExitCode        : 0
    Name            : wordcount
    PercentComplete : map 100% reduce 100%
    Query           :
    State           : Completed
    StatusDirectory : f1ed2028-afe8-402f-a24b-13cc17858097
    SubmissionTime  : 12/5/2014 8:34:09 PM
    JobId           : job_1415949758166_0071
    ```

    Такие выходные данные означают, что задание завершено успешно.

    > [!NOTE]  
    > Если значение **ExitCode** не равно 0, см. сведения в разделе [Устранение неполадок](#troubleshooting).

    В этом примере скачанные файлы сохраняются в файле **output.txt** в каталоге, из которого запускается сценарий.

### <a name="view-output"></a>Просмотр выходных данных

Чтобы просмотреть слова и статистику, полученные при выполнении задания, откройте файл **output.txt** в текстовом редакторе.

> [!NOTE]  
> Выходные файлы задания MapReduce являются неизменяемыми. Поэтому при повторном выполнении этого примера потребуется изменить имя выходного файла.

## <a name="troubleshooting"></a>Устранение неполадок

Если при завершении задания данные не возвращаются, просмотрите ошибки задания. Чтобы просмотреть сведения об ошибке для этого задания, добавьте следующую команду в конец файла **mapreducejob.ps1** . Затем сохраните файл и повторно запустите скрипт.

```powershell
# Print the output of the WordCount job.
Write-Host "Display the standard output ..." -ForegroundColor Green
Get-AzHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $wordCountJob.JobId `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
```

Этот командлет возвращает информацию, которая записывалась в STDERR при выполнении задания.

## <a name="next-steps"></a>Дальнейшие действия

Как можно видеть, Azure PowerShell позволяет с легкостью выполнять задания MapReduce в кластере HDInsight, отслеживать состояние задания и получать выходные данные. Дополнительная информация о других способах работы с Hadoop в HDInsight:

* [Использование MapReduce в Hadoop в HDInsight](hdinsight-use-mapreduce.md)
* [Использование Apache Hive с Apache Hadoop в HDInsight](hdinsight-use-hive.md)