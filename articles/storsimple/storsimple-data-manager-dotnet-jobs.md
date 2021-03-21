---
title: Использование пакета SDK для .NET для Microsoft Azure StorSimple заданий Диспетчер данных
description: Узнайте, как использовать пакет SDK для .NET в службе Диспетчер данных StorSimple для преобразования данных устройства StorSimple.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: b18627d2806662d6d966af95d51873d5623b2393
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96015542"
---
# <a name="use-the-net-sdk-to-initiate-data-transformation"></a>Запуск преобразования данных с помощью пакета SDK для .NET

## <a name="overview"></a>Обзор

В этой статье объясняется, как использовать функцию преобразования данных в службе диспетчера данных StorSimple для преобразования данных устройства StorSimple. Затем преобразованные данные используются другими службами Azure в облаке.

Задание преобразования данных можно запустить двумя способами:

- Использование пакета SDK для .NET
- Использование модуля runbook службы автоматизации Azure.
 
  В статье содержится подробная информация по созданию примера консольного приложения .NET для запуска задания преобразования данных и отслеживания его завершения. Дополнительные сведения о том, как запустить преобразование данных через службу автоматизации, можно узнать из статьи [Активация задания с помощью службы автоматизации Azure (закрытая предварительная версия)](storsimple-data-manager-job-using-automation.md).

## <a name="prerequisites"></a>Предварительные условия

Перед началом работы убедитесь, что у вас есть следующие компоненты:
*   Компьютер с такими запущенными средами:

    - Visual Studio 2012, 2013, 2015 или 2017.

    - Azure PowerShell. [Скачать Azure PowerShell](/powershell/azure/).
*   Правильно настроенное определение задания в диспетчере данных StorSimple в одной группе ресурсов.
*   Все необходимые библиотеки DLL. Их можно скачать из [репозитория GitHub](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/tree/master/Data_Manager_Job_Run/dlls).
*   [`Get-ConfigurationParams.ps1`](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/blob/master/Data_Manager_Job_Run/Get-ConfigurationParams.ps1) Скрипт из репозитория GitHub.

## <a name="step-by-step-procedure"></a>Пошаговая процедура

Выполните следующие шаги, чтобы запустить задание преобразования данных с помощью .NET.

1. Чтобы получить параметры конфигурации, сделайте следующее:
    1. Скачайте сценарий `Get-ConfigurationParams.ps1` из репозитория GitHub в расположение `C:\DataTransformation`.
    1. Запустите сценарий `Get-ConfigurationParams.ps1` из репозитория GitHub. Введите следующую команду:

        ```
        C:\DataTransformation\Get-ConfigurationParams.ps1 -SubscriptionName "AzureSubscriptionName" -ActiveDirectoryKey "AnyRandomPassword" -AppName "ApplicationName"
         ```
        Вы можете передать любые значения для ActiveDirectoryKey и AppName.

2. Этот скрипт выводит следующие значения:
    * Идентификатор клиента
    * Tenant ID
    * ключ Active Directory (совпадает с указанным выше);
    * Идентификатор подписки

        ![Выходные данные сценария параметров конфигурации](media/storsimple-data-manager-dotnet-jobs/get-config-parameters.png)

3. С помощью Visual Studio 2012, 2013 или 2015 создайте консольное приложение C# .NET.

    1. Запустите **Visual Studio 2012/2013/2015**.
    1. Выберите **Файл > Создать > Проект**.

        ![Создание проекта 1](media/storsimple-data-manager-dotnet-jobs/create-new-project-7.png)        
    2. Выберите **Установленные > Шаблоны > Visual C# > Консольное приложение**.
    3. В поле **Имя** введите **DataTransformationApp**.
    4. В поле **Расположение** выберите **C:\DataTransformation**.
    6. Чтобы создать проект, нажмите кнопку **ОК** .

        ![Создание проекта 2](media/storsimple-data-manager-dotnet-jobs/create-new-project-1.png)

4. Теперь добавьте все библиотеки DLL из папки [dlls](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/tree/master/Data_Manager_Job_Run/dlls) в качестве **ссылочных** в созданный проект. Чтобы добавить DLL-файлы, сделайте следующее:

   1. В Visual Studio выберите **Просмотр > Обозреватель решений**.
   2. Щелкните стрелку слева от проекта приложения преобразования данных. Щелкните **Ссылки**, а затем щелкните этот элемент правой кнопкой мыши и выберите **Добавить ссылку**.
    
       ![Добавление библиотек DLL 1](media/storsimple-data-manager-dotnet-jobs/create-new-project-4.png)

   3. Перейдите в расположение папки с пакетами, выберите все библиотеки DLL, щелкните **Добавить**, а затем нажмите кнопку **ОК**.

       ![Добавление библиотек DLL 2](media/storsimple-data-manager-dotnet-jobs/create-new-project-6.png)

5. Добавьте следующие операторы **using** в файл исходного кода (Program.cs) в проекте.

    ```
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using Microsoft.Azure.Management.HybridData.Models;
    using Microsoft.Internal.Dms.DmsWebJob;
    using Microsoft.Internal.Dms.DmsWebJob.Contracts;
    ```
    
6. Представленный ниже код инициализирует экземпляр задания преобразования данных. Добавьте его в **метод Main**. Замените значения параметров конфигурации полученными ранее значениями. Добавьте значения параметров **ResourceGroupName** и **ResourceName**. Значение **ResourceGroupName** связано с диспетчером данных StorSimple, на котором было настроено определение задания. **ResourceName** — имя службы данных диспетчера StorSimple.

    ```
    // Setup the configuration parameters.
    var configParams = new ConfigurationParams
    {
        ClientId = "client-id",
        TenantId = "tenant-id",
        ActiveDirectoryKey = "active-directory-key",
        SubscriptionId = "subscription-id",
        ResourceGroupName = "resource-group-name",
        ResourceName = "resource-name"
    };

    // Initialize the Data Transformation Job instance.
    DataTransformationJob dataTransformationJob = new DataTransformationJob(configParams);
    ```
   
7. Укажите параметры, с которыми необходимо выполнить определение задания.

    ```
    string jobDefinitionName = "job-definition-name";

    DataTransformationInput dataTransformationInput = dataTransformationJob.GetJobDefinitionParameters(jobDefinitionName);
    ```

    (ИЛИ)

    Если параметры определения задания нужно изменить во время выполнения, добавьте следующий код:

    ```
    string jobDefinitionName = "job-definition-name";
    // Must start with a '\'
    var rootDirectories = new List<string> {@"\root"};

    // Name of the volume on the StorSimple device.
    var volumeNames = new List<string> {"volume-name"};

    var dataTransformationInput = new DataTransformationInput
    {
        // If you require the latest existing backup to be picked else use TakeNow to trigger a new backup.
        BackupChoice = BackupChoice.UseExistingLatest.ToString(),
        // Name of the StorSimple device.
        DeviceName = "device-name",
        // Name of the container in Azure storage where the files will be placed after execution.
        ContainerName = "container-name",
        // File name filter (search pattern) to be applied on files under the root directory. * - Match all files.
        FileNameFilter = "*",
        // List of root directories.
        RootDirectories = rootDirectories,
        // Name of the volume on StorSimple device on which the relevant data is present. 
        VolumeNames = volumeNames
    };
    ```

8. После инициализации добавьте следующий код, чтобы активировать задание преобразования данных в определении задания. Добавьте соответствующее **имя определения задания**.

    ```
    // Trigger a job, retrieve the jobId and the retry interval for polling.
    int retryAfter;
    string jobId = dataTransformationJob.RunJobAsync(jobDefinitionName, 
    dataTransformationInput, out retryAfter);
    Console.WriteLine("jobid: ", jobId);
    Console.ReadLine();

    ```
    Выполните сборку решения сразу после вставки кода. Ниже приведен снимок экрана с фрагментом кода для инициализации экземпляра задания преобразования данных.

   ![Фрагмент кода для инициализации задания преобразования данных](media/storsimple-data-manager-dotnet-jobs/start-dotnet-job-code-snippet-1.png)

9. Это задание преобразует данные, которые соответствуют корневому каталогу и фильтрам файла, в пределах тома StorSimple и помещает их в указанный контейнер или общий файловый ресурс. После преобразования файла в очередь хранилища передается сообщение (в той же учетной записи хранения, что и контейнер или общий файловый ресурс) с тем же именем, что и у определения задания. Это сообщение можно использовать в качестве триггера для запуска дальнейшей обработки файла.

10. После активации задания используйте следующий код для отслеживания завершения задания. Добавлять следующий код для выполнения задания необязательно.

    ```
    Job jobDetails = null;

    // Poll the job.
    do
    {
        jobDetails = dataTransformationJob.GetJob(jobDefinitionName, jobId);

        // Wait before polling for the status again.
        Thread.Sleep(TimeSpan.FromSeconds(retryAfter));

    } while (jobDetails.Status == JobStatus.InProgress);

    // Completion status of the job.
    Console.WriteLine("JobStatus: {0}", jobDetails.Status);
    
    // To hold the console before exiting.
    Console.Read();

    ```
    Ниже приведен снимок экрана со всем примером кода, используемым для запуска задания с помощью .NET.

    ![Полный фрагмент кода для запуска задания .NET](media/storsimple-data-manager-dotnet-jobs/start-dotnet-job-code-snippet.png)

## <a name="next-steps"></a>Дальнейшие действия

[Используйте диспетчер данных StorSimple пользовательский интерфейс для преобразования данных](storsimple-data-manager-ui.md).