---
title: Выполнение запросов Apache Hive с использованием пакета SDK .NET для HDInsight — Azure
description: Узнайте, как отправлять задания Apache Hadoop в Azure HDInsight с использованием пакета SDK .NET для HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 12/24/2019
ms.openlocfilehash: 032c070a9485fc4720ae7966a0991bc0c5ae3921
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104867854"
---
# <a name="run-apache-hive-queries-using-hdinsight-net-sdk"></a>Выполнение запросов Apache Hive с использованием пакета SDK .NET для HDInsight

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

Узнайте, как отправлять запросы Apache Hive с использованием пакета SDK .NET для HDInsight. Напишите программу на языке C#, чтобы отправить запрос Hive для перечисления таблиц Hive и отобразить результаты.

> [!NOTE]  
> Действия, описанные в этой статье, необходимо выполнять из клиента Windows. Чтобы получить сведения об использовании клиента Linux, OS X или Unix для работы с Hive, воспользуйтесь выбором вкладок в верхней части статьи.

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этой статьей необходимо иметь следующее:

* Кластер Apache Hadoop в HDInsight. См. статью [Руководство по Hadoop. Начало работы с Hadoop в HDInsight на платформе Linux](apache-hadoop-linux-tutorial-get-started.md).

    > [!IMPORTANT]  
    > Начиная с 15 сентября 2017 г. пакет SDK для HDInsight .NET поддерживает возвращение результатов запроса Hive только из учетных записей хранения Azure. Если использовать этот пример с кластером HDInsight, в котором Azure Data Lake Storage применяется в качестве основного хранилища, вы не сможете получить результаты поиска с помощью пакета SDK для .NET.

* [Visual Studio](https://visualstudio.microsoft.com/vs/community/) 2013 и более поздние. Должна быть установлена по крайней мере Рабочая нагрузка на **разработку классических приложений .NET** .

## <a name="run-a-hive-query"></a>Выполнение запроса Hive

Пакет SDK для HDInsight .NET содержит клиентские библиотеки .NET, которые упрощают работу с кластерами HDInsight из .NET.

1. Создайте в Visual Studio консольное приложение C#.

1. Введите следующую команду в окне консоли диспетчера пакетов NuGet:

    ```console
    Install-Package Microsoft.Azure.Management.HDInsight.Job
    ```

1. Измените приведенный ниже код, чтобы инициализировать значения переменных: `ExistingClusterName, ExistingClusterUsername, ExistingClusterPassword,DefaultStorageAccountName,DefaultStorageAccountKey,DefaultStorageContainerName` . Затем используйте измененный код в качестве всего содержимого **программы. CS** в Visual Studio.

    ```csharp
    using System.Collections.Generic;
    using System.IO;
    using System.Text;
    using System.Threading;
    using Microsoft.Azure.Management.HDInsight.Job;
    using Microsoft.Azure.Management.HDInsight.Job.Models;
    using Hyak.Common;

    namespace SubmitHDInsightJobDotNet
    {
        class Program
        {
            private static HDInsightJobManagementClient _hdiJobManagementClient;

            private const string ExistingClusterName = "<Your HDInsight Cluster Name>";
            private const string ExistingClusterUsername = "<Cluster Username>";
            private const string ExistingClusterPassword = "<Cluster User Password>";

            // Only Azure Storage accounts are supported by the SDK
            private const string DefaultStorageAccountName = "<Default Storage Account Name>";
            private const string DefaultStorageAccountKey = "<Default Storage Account Key>";
            private const string DefaultStorageContainerName = "<Default Blob Container Name>";

            private const string ExistingClusterUri = ExistingClusterName + ".azurehdinsight.net";

            static void Main(string[] args)
            {
                System.Console.WriteLine("The application is running ...");

                var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
                _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);

                SubmitHiveJob();

                System.Console.WriteLine("Press ENTER to continue ...");
                System.Console.ReadLine();
            }

            private static void SubmitHiveJob()
            {
                Dictionary<string, string> defines = new Dictionary<string, string> { { "hive.execution.engine", "tez" }, { "hive.exec.reducers.max", "1" } };
                List<string> args = new List<string> { { "argA" }, { "argB" } };
                var parameters = new HiveJobSubmissionParameters
                {
                    Query = "SHOW TABLES",
                    Defines = defines,
                    Arguments = args
                };

                System.Console.WriteLine("Submitting the Hive job to the cluster...");
                var jobResponse = _hdiJobManagementClient.JobManagement.SubmitHiveJob(parameters);
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
                var storageAccess = new AzureStorageAccess(DefaultStorageAccountName, DefaultStorageAccountKey,
                    DefaultStorageContainerName);
                var output = (jobDetail.ExitValue == 0)
                    ? _hdiJobManagementClient.JobManagement.GetJobOutput(jobId, storageAccess) // fetch stdout output in case of success
                    : _hdiJobManagementClient.JobManagement.GetJobErrorLogs(jobId, storageAccess); // fetch stderr output in case of failure

                System.Console.WriteLine("Job output is: ");

                using (var reader = new StreamReader(output, Encoding.UTF8))
                {
                    string value = reader.ReadToEnd();
                    System.Console.WriteLine(value);
                }
            }
        }
    }
    ```

1. Нажмите клавишу **F5** для запуска приложения.

Выходные данные приложения должны выглядеть следующим образом:

:::image type="content" source="./media/apache-hadoop-use-hive-dotnet-sdk/hdinsight-hadoop-use-hive-net-sdk-output.png" alt-text="Выходные данные задания Hadoop Hive в HDInsight" border="true":::

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как отправлять запросы Apache Hive с помощью пакета SDK для HDInsight .NET. Дополнительные сведения см. в следующих статьях:

* [Приступая к работе с Azure HDInsight](apache-hadoop-linux-tutorial-get-started.md)
* [Создание кластеров Apache Hadoop в HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)
* [Справочник по пакетам SDK HDInsight для .NET](/dotnet/api/overview/azure/hdinsight)
* [Использование Apache Sqoop с HDInsight](apache-hadoop-use-sqoop-mac-linux.md)
* [Создание приложений .NET HDInsight с неинтерактивной проверкой подлинности](../hdinsight-create-non-interactive-authentication-dotnet-applications.md)
