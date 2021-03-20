---
title: Отправка заданий MapReduce с использованием пакета SDK HDInsight для .NET в Azure
description: Узнайте, как отправлять задания MapReduce в Apache Hadoop для Azure HDInsight с помощью пакета SDK для HDInsight .NET.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 01/15/2020
ms.openlocfilehash: 8fbcb66be11c7c77a9bfaf0e6ec790622dcbbda7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98932185"
---
# <a name="run-mapreduce-jobs-using-hdinsight-net-sdk"></a>Выполнение заданий MapReduce с использованием пакета SDK для HDInsight .NET

[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

Узнайте, как отправлять задания MapReduce с использованием пакета SDK для HDInsight .NET. В кластерах HDInsight предусмотрен JAR-файл с несколькими примерами MapReduce. JAR-файл — `/example/jars/hadoop-mapreduce-examples.jar` .  Один из примеров — **wordcount** (подсчет слов). Вы разрабатываете консольное приложение на C# для отправки задания по подсчету слов.  Задание считывает `/example/data/gutenberg/davinci.txt` файл и выводит результаты в `/example/data/davinciwordcount` .  Чтобы снова запустить приложение, необходимо очистить папку выходных данных.

> [!NOTE]  
> Действия, описанные в этой статье, необходимо выполнять из клиента Windows. Чтобы получить сведения об использовании клиента Linux, OS X или Unix для работы с Hive, воспользуйтесь выбором вкладок в верхней части статьи.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Hadoop в HDInsight. См. [Создание кластеров под управлением Linux в HDInsight с помощью портала Azure](../hdinsight-hadoop-create-linux-clusters-portal.md).

* [Visual Studio](https://visualstudio.microsoft.com/vs/community/).

## <a name="submit-mapreduce-jobs-using-hdinsight-net-sdk"></a>Отправка заданий MapReduce с использованием пакета SDK для HDInsight .NET

Пакет SDK для HDInsight .NET предоставляет клиентские библиотеки .NET, которые упрощают работу с кластерами HDInsight из .NET.

1. Запустите Visual Studio и создайте консольное приложение C#.

1. Последовательно выберите **инструменты**  >  **Диспетчер пакетов NuGet**  >  **консоль диспетчера пакетов** и введите следующую команду:

    ```   
    Install-Package Microsoft.Azure.Management.HDInsight.Job
    ```

1. Скопируйте приведенный ниже код в **Program. CS**. Затем измените код, задав значения для: `existingClusterName` ,,, `existingClusterPassword` `defaultStorageAccountName` `defaultStorageAccountKey` и `defaultStorageContainerName` .

    ```csharp
    using System.Collections.Generic;
    using System.IO;
    using System.Text;
    using System.Threading;
    using Microsoft.Azure.Management.HDInsight.Job;
    using Microsoft.Azure.Management.HDInsight.Job.Models;
    using Hyak.Common;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    
    namespace SubmitHDInsightJobDotNet
    {
        class Program
        {
            private static HDInsightJobManagementClient _hdiJobManagementClient;
    
            private const string existingClusterName = "<Your HDInsight Cluster Name>";
            private const string existingClusterPassword = "<Cluster User Password>";
            private const string defaultStorageAccountName = "<Default Storage Account Name>"; 
            private const string defaultStorageAccountKey = "<Default Storage Account Key>";
            private const string defaultStorageContainerName = "<Default Blob Container Name>";
    
            private const string existingClusterUsername = "admin";
            private const string existingClusterUri = existingClusterName + ".azurehdinsight.net";
            private const string sourceFile = "/example/data/gutenberg/davinci.txt";
            private const string outputFolder = "/example/data/davinciwordcount";
    
            static void Main(string[] args)
            {
                System.Console.WriteLine("The application is running ...");
    
                var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = existingClusterUsername, Password = existingClusterPassword };
                _hdiJobManagementClient = new HDInsightJobManagementClient(existingClusterUri, clusterCredentials);
    
                SubmitMRJob();
    
                System.Console.WriteLine("Press ENTER to continue ...");
                System.Console.ReadLine();
            }
    
            private static void SubmitMRJob()
            {
                List<string> args = new List<string> { { "/example/data/gutenberg/davinci.txt" }, { "/example/data/davinciwordcount" } };
    
                var paras = new MapReduceJobSubmissionParameters
                {
                    JarFile = @"/example/jars/hadoop-mapreduce-examples.jar",
                    JarClass = "wordcount",
                    Arguments = args
                };
    
                System.Console.WriteLine("Submitting the MR job to the cluster...");
                var jobResponse = _hdiJobManagementClient.JobManagement.SubmitMapReduceJob(paras);
                var jobId = jobResponse.JobSubmissionJsonResponse.Id;
                System.Console.WriteLine("Response status code is " + jobResponse.StatusCode);
                System.Console.WriteLine("JobId is " + jobId);
    
                System.Console.WriteLine("Waiting for the job completion ...");
    
                // Wait for job completion
                var jobDetail = _hdiJobManagementClient.JobManagement.GetJob(jobId).JobDetail;
                while (!jobDetail.Status.JobComplete)
                {
                    Thread.Sleep(1000);
                    jobDetail = _hdiJobManagementClient.JobManagement.GetJob(jobId).JobDetail;
                }
    
                // Get job output
                System.Console.WriteLine("Job output is: ");
                var storageAccess = new AzureStorageAccess(defaultStorageAccountName, defaultStorageAccountKey,
                    defaultStorageContainerName);
    
                if (jobDetail.ExitValue == 0)
                {
                    // Create the storage account object
                    CloudStorageAccount storageAccount = CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName=" +
                        defaultStorageAccountName +
                        ";AccountKey=" + defaultStorageAccountKey);
    
                    // Create the blob client.
                    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    
                    // Retrieve reference to a previously created container.
                    CloudBlobContainer container = blobClient.GetContainerReference(defaultStorageContainerName);
    
                    CloudBlockBlob blockBlob = container.GetBlockBlobReference(outputFolder.Substring(1) + "/part-r-00000");
    
                    using (var stream = blockBlob.OpenRead())
                    {
                        using (StreamReader reader = new StreamReader(stream))
                        {
                            while (!reader.EndOfStream)
                            {
                                System.Console.WriteLine(reader.ReadLine());
                            }
                        }
                    }
                }
                else
                {
                    // fetch stderr output in case of failure
                    var output = _hdiJobManagementClient.JobManagement.GetJobErrorLogs(jobId, storageAccess);
    
                    using (var reader = new StreamReader(output, Encoding.UTF8))
                    {
                        string value = reader.ReadToEnd();
                        System.Console.WriteLine(value);
                    }
    
                }
            }
        }
    }

    ```

1. Нажмите клавишу **F5** для запуска приложения.

Чтобы снова запустить задание, необходимо изменить имя папки выходных данных задания в примере `/example/data/davinciwordcount` .

После успешного завершения задания приложение выводит содержимое выходного файла `part-r-00000` .

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы ознакомились с несколькими способами создания кластера HDInsight. Дополнительные сведения см. в следующих статьях:

* Сведения об отправке задания Hive см. в статье [Выполнение запросов Apache Hive с использованием пакета SDK .NET для HDInsight](apache-hadoop-use-hive-dotnet-sdk.md).
* Сведения о создании кластеров HDInsight см. в статье [Установка кластеров в HDInsight с использованием Hadoop, Spark, Kafka и других технологий](../hdinsight-hadoop-provision-linux-clusters.md).
* Сведения об управлении кластерами в HDInsight см. в статье [Управление кластерами Apache Hadoop в HDInsight с помощью портала Azure](../hdinsight-administer-use-portal-linux.md).
* Подробные сведения о пакете SDK для HDInsight .NET см. в [соответствующей справке](/dotnet/api/overview/azure/hdinsight).
* Сведения о неинтерактивной проверке подлинности в Azure см. в статье [Создание приложений .NET HDInsight с неинтерактивной проверкой подлинности](../hdinsight-create-non-interactive-authentication-dotnet-applications.md).
