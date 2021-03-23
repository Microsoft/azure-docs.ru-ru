---
title: Обработка больших наборов данных с помощью фабрики данных и пакетной службы
description: В этой статье описывается процесс обработки больших объемов данных в конвейере фабрики данных Azure с использованием возможности параллельной обработки пакетной службы Azure.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 2568fc2ea133a0cdd49e0a2577ce1e580056a74c
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104783138"
---
# <a name="process-large-scale-datasets-by-using-data-factory-and-batch"></a>Обработка больших наборов данных с помощью фабрики данных и пакетной службы
> [!NOTE]
> Статья относится к версии 1 фабрики данных Azure, которая является общедоступной. Если вы используете текущую версию Фабрики данных, см. статью о [пользовательских действиях в службе "Фабрика данных Azure"](../transform-data-using-dotnet-custom-activity.md).

В этой статье описывается архитектура простого решения, которое автоматически и по расписанию перемещает и обрабатывает большие наборы данных. Здесь также предоставлено полное пошаговое руководство по реализации решения с помощью фабрики данных и пакетной службы Azure.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Эта статья длиннее обычной, так как в ней содержится пошаговое руководство для всего примера решения. Если вы впервые используете пакетную службу и фабрику данных, здесь вы можете узнать об этих службах и их совместном использовании. Если вы немного знакомы с этими службами и проектируете решение, то можете сосредоточиться на разделе об архитектуре этой статьи. Если вы разрабатываете прототип или решение, то можете выполнить пошаговые инструкции из приведенного руководства. Мы приветствуем ваши комментарии об этой статье и использовании ее содержимого.

Сначала давайте рассмотрим, как фабрика данных и пакетная служба могут быть полезны для обработки больших наборов данных в облаке.     


## <a name="why-azure-batch"></a>Сведения о пакетной службе Azure
 С помощью пакетной службы Azure можно эффективно запускать приложения для крупномасштабных параллельных и высокопроизводительных вычислений (HPC) в облаке. Это платформа, которая позволяет запланировать выполнение ресурсоемких вычислений в управляемой коллекции виртуальных машин. Она автоматически масштабирует вычислительные ресурсы, учитывая требования заданий.

С помощью пакетной службы вы можете определять вычислительные ресурсы Azure, необходимые для выполнения приложений в параллельном режиме и в нужном масштабе. Задания можно запускать по требованию или по расписанию. Вам не нужно вручную создавать и настраивать кластер высокопроизводительных вычислений, отдельные виртуальные машины, виртуальные сети или сложную инфраструктуру планировщика задач и заданий, а также управлять всеми этими ресурсами.

 Если вы не знакомы с пакетной службой, изучите приведенные ниже статьи. Это поможет понять архитектуру и схему реализации решения, описанного в этой статье.   

* [Выполнение реальных параллельных рабочих нагрузок с использованием пакетной службы](../../azure-sql/database/sql-database-paas-overview.md)
* [Обзор функций пакетной службы](../../batch/batch-service-workflow-features.md)

Дополнительные сведения о пакетной службе см. в [документации по пакетной службе](../../batch/index.yml).

## <a name="why-azure-data-factory"></a>Сведения о фабрике данных Azure
Фабрика данных представляет собой облачную службу интеграции информации, которая организует и автоматизирует перемещение и преобразование данных. С помощью фабрики данных можно создать управляемые конвейеры данных, которые перемещают данные из локальных и облачных хранилищ данных в централизованное. Например, в хранилище BLOB-объектов Azure. Фабрика данных также позволяет обрабатывать и преобразовывать данные с помощью служб, таких как Azure HDInsight и Машинное обучение Azure. Кроме того, вы можете планировать запуск конвейеров данных по расписанию (например, ежечасно, ежедневно и еженедельно), отслеживать их состояние и управлять ими, чтобы обнаруживать проблемы и принимать меры.

  Если вы не знакомы с фабрикой данных, изучите приведенные ниже статьи. Это поможет понять архитектуру и схему реализации решения, описанного в этой статье.  

* [Введение в службу фабрики данных](data-factory-introduction.md)
* [Построение конвейера данных](data-factory-build-your-first-pipeline.md)   

Дополнительные сведения о Фабрике данных Azure см. в [документации по Фабрике данных Azure](/rest/api/datafactory/v1/data-factory-data-factory).

## <a name="data-factory-and-batch-together"></a>Совместное использование фабрики данных и пакетной службы
Фабрика данных включает в себя встроенные действия. Например, действие копирования используется, чтобы копировать и перемещать данные из исходного хранилища данных в целевое. Действие Hive используется, чтобы обрабатывать данные с использованием кластеров Hadoop (HDInsight) в Azure. Список поддерживаемых действий преобразования см. в [этой статье](data-factory-data-transformation-activities.md).

Вы также можете создать настраиваемые действия .NET для перемещения или обработки данных с помощью собственной логики. Эти действия можно выполнять в кластере HDInsight или в пуле виртуальных машин пакетной службы. При использовании пакетной службы можно настроить автоматическое масштабирование пула (добавление или удаление виртуальных машин в зависимости от рабочей нагрузки) по предоставленной формуле.     

## <a name="architecture-of-a-sample-solution"></a>Архитектура примера решения
  Архитектура, описанная в этой статье, предназначена для простого решения. Она также подходит для таких сложных сценариев, как моделирование рисков по финансовым услугам, обработка и отрисовка изображений и геномный анализ.

На схеме ниже показано, как фабрика данных управляет перемещением и обработкой данных, а также как пакетная служба параллельно обрабатывает данные. Загрузите и распечатайте схему для удобства (в формате 11 x 17 дюймов или A3). Чтобы распечатать схему, скачайте PDF-файл [HPC and data orchestration using Azure Batch and Data Factory](https://go.microsoft.com/fwlink/?LinkId=717686) (Управление данными и высокопроизводительными вычислениями с помощью пакетной службы и фабрики данных Azure).

[![Схема обработки данных большого объема](./media/data-factory-data-processing-using-batch/image1.png)](https://go.microsoft.com/fwlink/?LinkId=717686)

Ниже приведен список основных этапов процесса. Решение содержит код и указания по созданию комплексного решения.

* **Настройка пула вычислительных узлов (виртуальные машины) для пакетной службы.** Вы можете указать количество узлов и размер каждого из них.

* **Создание экземпляра фабрики данных**, для которого настраиваются сущности, представляющие хранилище BLOB-объектов, службу вычислений пакетной службы, входные и выходные данные, а также рабочий процесс и конвейер с действиями, позволяющими перемещать и преобразовывать данные.

* **Создание настраиваемого действия .NET в конвейере фабрики данных.** Действие является пользовательским кодом, который выполняется в пуле пакетной службы.

* **Сохранение больших объемов входных данных в виде больших двоичных объектов в службе хранилища Azure.** Данные при этом разделяются на логические срезы (обычно по времени).

* **Фабрика данных копирует данные, которые параллельно обрабатываются** и передаются в дополнительное расположение.

* **Фабрика данных запускает настраиваемое действие с помощью пула, выделенного пакетной службой.** Действия в фабрике данных могут выполняться параллельно. В каждом действии обрабатывается один срез данных. Результаты сохраняются в хранилище.

* **Фабрика данных перемещает окончательные результаты в третье расположение** для распространения с помощью приложения или дальнейшей обработки с помощью других инструментов.

## <a name="implementation-of-the-sample-solution"></a>Реализация примера решения
Пример решения намеренно простой. Он предназначен, чтобы продемонстрировать совместное использование фабрики данных и пакетной службы для обработки наборов данных. Решение вычисляет количество вхождений условия поиска Microsoft во входных файлах, упорядоченных по временным рядам. Затем количество выводится в выходные файлы.

**Время.** Если вы знакомы с основами использования Azure, фабрики данных и пакетной службы, а также выполнили указанные ниже предварительные требования, создание решения должно занять 1–2 часа.

### <a name="prerequisites"></a>Предварительные требования
#### <a name="azure-subscription"></a>Подписка Azure.
Если у вас нет подписки Azure, можно быстро создать бесплатную пробную учетную запись. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/pricing/free-trial/).

#### <a name="azure-storage-account"></a>Учетная запись хранения Azure.
Учетная запись хранения используется в этом руководстве для хранения данных. Если у вас нет учетной записи хранения, [создайте ее](../../storage/common/storage-account-create.md). В примере решения используется хранилище BLOB-объектов.

#### <a name="azure-batch-account"></a>Учетная запись пакетной службы Azure
Создайте учетную запись пакетной службы с помощью [портала Azure](https://portal.azure.com/). Дополнительные сведения см. в [этой статье](../../batch/batch-account-create-portal.md). Запишите ключ и имя учетной записи пакетной службы. Для создания учетной записи пакетной службы можно также воспользоваться командлетом [New-AzBatchAccount](/powershell/module/az.batch/new-azbatchaccount). Инструкции по использованию этого командлета см. в статье [Управление ресурсами пакетной службы с помощью командлетов PowerShell](../../batch/batch-powershell-cmdlets-get-started.md).

В примере решения используется пакетная служба (косвенно, через конвейер фабрики данных) для параллельной обработки данных в пуле вычислительных узлов (управляемой коллекции виртуальных машин).

#### <a name="azure-batch-pool-of-virtual-machines"></a>Пул виртуальных машин пакетной службы
Создайте пул пакетной службы с как минимум двумя вычислительными узлами.

1. На [портале Azure](https://portal.azure.com) щелкните **Обзор** в меню слева и выберите **Учетные записи пакетной службы**.

1. Выберите свою учетную запись пакетной службы, чтобы открыть колонку **Учетная запись пакетной службы**.

1. Выберите плитку **Pools** (Пулы).

1. В колонке **Pools** (Пулы) нажмите кнопку **Добавить** на панели инструментов, чтобы добавить пул.

   а. Введите идентификатор для пула (**идентификатор пула**). Запишите идентификатор пула. Он понадобится при создании решения фабрики данных.

   b. В качестве параметра **семейства операционных систем** укажите **Windows Server 2012 R2**.

   c. Выберите **Ценовая категория для узлов**.

   d. Для параметра **Выделенный целевой объект** укажите значение **2**.

   д) Для параметра **Максимальное число заданий на узел** укажите значение **2**.

   е) Нажмите кнопку **ОК**, чтобы создать пул.

#### <a name="azure-storage-explorer"></a>Обозреватель службы хранилища Azure
Для проверки и изменения данных в проектах службы хранилища используется [Обозреватель службы хранилища Azure 6](https://azurestorageexplorer.codeplex.com/) или [CloudXplorer](https://clumsyleaf.com/products/cloudxplorer) (от ClumsyLeaf Software). Кроме того, с помощью этих средств можно проверять и изменять данные в журналах облачных приложений.

1. Создайте контейнер **mycontainer** с закрытым доступом (без анонимного доступа).

1. При использовании CloudXplorer создавайте папки и подпапки со следующей структурой:

   ![Структура папки и подпапки](./media/data-factory-data-processing-using-batch/image3.png)

   `Inputfolder`и `outputfolder` — это папки верхнего уровня в `mycontainer`. Папка `inputfolder` содержит подпапки с метками даты и времени (ГГГГ-ММ-ДД-ЧЧ).

   При использовании Обозревателя службы хранилища на следующем шаге необходимо передать файлы с этими именами: `inputfolder/2015-11-16-00/file.txt`, `inputfolder/2015-11-16-01/file.txt` и т. д. На этом шаге автоматически будут созданы папки.

1. Создайте на компьютере текстовый файл **file.txt** с содержимым, в котором есть ключевое слово **Microsoft**. Например, test custom activity Microsoft test custom activity Microsoft.

1. Отправьте файл в следующие входные папки хранилища BLOB-объектов.

   ![Входные папки](./media/data-factory-data-processing-using-batch/image4.png)

   При использовании Обозревателя службы хранилища передайте файл **file.txt** в контейнер **mycontainer**. На панели инструментов выберите **Копировать**, чтобы создать копию большого двоичного объекта. В диалоговом окне **Копирование большого двоичного объекта** измените **имя целевого большого двоичного объекта** на `inputfolder/2015-11-16-00/file.txt`. Повторите этот шаг, чтобы создать `inputfolder/2015-11-16-01/file.txt`, `inputfolder/2015-11-16-02/file.txt`, `inputfolder/2015-11-16-03/file.txt`, `inputfolder/2015-11-16-04/file.txt` и т. д. Это действие автоматически создает данные папки.

1. Создайте другой контейнер с именем: `customactivitycontainer`. Передайте в этот контейнер ZIP-файл настраиваемого действия.

#### <a name="visual-studio"></a>Visual Studio
Установите Visual Studio 2012 или более поздней версии, чтобы создать настраиваемое действие пакетной службы, которое будет использоваться в решении фабрики данных.

### <a name="high-level-steps-to-create-the-solution"></a>Пошаговое создание решения
1. Создайте настраиваемое действие, содержащее логику обработки данных.

1. Создайте фабрику данных, которая использует настраиваемое действие.

### <a name="create-the-custom-activity"></a>Создание пользовательского действия
Настраиваемое действие фабрики данных составляет основу этого примера решения. Для выполнения этого действия в решении используется пакетная служба. Общие сведения о разработке настраиваемых действий и их использовании в конвейере фабрики данных см. в статье [Использование настраиваемых действий в конвейере фабрики данных Azure](data-factory-use-custom-activities.md).

Чтобы создать настраиваемое действие .NET, которое можно использовать в конвейере фабрики данных, создайте проект библиотеки классов .NET с классом, который реализует интерфейс IDotNetActivity. У этого интерфейса только один метод: Execute. Подпись метода выглядит следующим образом:

```csharp
public IDictionary<string, string> Execute(
            IEnumerable<LinkedService> linkedServices,
            IEnumerable<Dataset> datasets,
            Activity activity,
            IActivityLogger logger)
```

У этого метода есть несколько ключевых компонентов, с которыми нужно разобраться.

* Метод принимает четыре параметра:

  * **linkedServices** — это перечисляемый список связанных служб, которые связывают источники входных и выходных данных (например, хранилище BLOB-объектов) с фабрикой данных. В этом примере присутствует только одна связанная служба (служба хранилища Azure), которая используется для входных и выходных данных.
  * **наборы данных**. Этот параметр является перечисляемым списком наборов данных. Этот параметр можно использовать для получения расположений и схем, которые определяются входными и выходными наборами данных.
  * **activity** — этот параметр представляет собой текущую вычислительную сущность. В этом примере — пакетную службу.
  * **logger** — средство ведения журнала, позволяющее записывать комментарии отладки, которые отображаются в виде пользовательского журнала для конвейера.
* Этот метод возвращает словарь, который можно будет использовать для создания цепочки из настраиваемых действий в будущем. Эта функция еще не реализована, поэтому из метода просто возвращается пустой словарь.

#### <a name="procedure-create-the-custom-activity"></a>Описание процедуры. Создание пользовательского действия
1. Создайте проект библиотеки классов .NET в Visual Studio.

   а. Запустите Visual Studio 2012, 2013 или 2015.

   b. Выберите **File** > **New** > **Project** ( Файл > Создать > Проект).

   c. Разверните раздел **Шаблоны** и выберите **Visual C#\#** . В этом руководстве используется язык C\#, но для создания настраиваемого действия вы можете использовать любой язык .NET.

   d. Выберите тип **Библиотека классов** из списка типов проектов справа.

   д) В поле **Имя** введите **MyDotNetActivity**.

   е) В поле **Расположение\\ выберите** C:**ADF**. Создайте папку **ADF**, если она отсутствует.

   ж. Чтобы создать проект, щелкните **ОК**.

1. Выберите **Инструменты** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.

1. В консоли диспетчера пакетов выполните следующую команду, чтобы импортировать пакет Microsoft.Azure.Management.DataFactories.

    ```powershell
    Install-Package Microsoft.Azure.Management.DataFactories
    ```
1. Импортируйте пакет NuGet для **службы хранилища Azure** в проект. Он нужен, так как в этом примере используется API хранилища BLOB-объектов.

    ```powershell
    Install-Package Az.Storage
    ```
1. Добавьте следующие директивы с using в исходный файл в проекте.

    ```csharp
    using System.IO;
    using System.Globalization;
    using System.Diagnostics;
    using System.Linq;

    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Runtime;

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```
1. Измените имя пространства имен на **MyDotNetActivityNS**.

    ```csharp
    namespace MyDotNetActivityNS
    ```
1. Измените имя класса на **MyDotNetActivity** и извлеките класс из интерфейса **IDotNetActivity**, как показано далее.

    ```csharp
    public class MyDotNetActivity : IDotNetActivity
    ```
1. Реализуйте (добавьте) метод **Execute** интерфейса **IDotNetActivity** для класса **MyDotNetActivity**. Скопируйте указанный ниже пример кода в этот метод. Объяснение логики, используемой в этом методе, см. в разделе [Метод Execute](#execute-method).

    ```csharp
    /// <summary>
    /// The Execute method is the only method of IDotNetActivity interface you must implement.
    /// In this sample, the method invokes the Calculate method to perform the core logic.  
    /// </summary>
    public IDictionary<string, string> Execute(
       IEnumerable<LinkedService> linkedServices,
       IEnumerable<Dataset> datasets,
       Activity activity,
       IActivityLogger logger)
    {

       // Declare types for the input and output data stores.
       AzureStorageLinkedService inputLinkedService;

       Dataset inputDataset = datasets.Single(dataset => dataset.Name == activity.Inputs.Single().Name);

       foreach (LinkedService ls in linkedServices)
           logger.Write("linkedService.Name {0}", ls.Name);

       // Use the First method instead of Single because we are using the same
       // Azure Storage linked service for input and output.
       inputLinkedService = linkedServices.First(
           linkedService =>
           linkedService.Name ==
           inputDataset.Properties.LinkedServiceName).Properties.TypeProperties
           as AzureStorageLinkedService;

       string connectionString = inputLinkedService.ConnectionString; // To create an input storage client.
       string folderPath = GetFolderPath(inputDataset);
       string output = string.Empty; // for use later.

       // Create the storage client for input. Pass the connection string.
       CloudStorageAccount inputStorageAccount = CloudStorageAccount.Parse(connectionString);
       CloudBlobClient inputClient = inputStorageAccount.CreateCloudBlobClient();

       // Initialize the continuation token before using it in the do-while loop.
       BlobContinuationToken continuationToken = null;
       do
       {   // get the list of input blobs from the input storage client object.
           BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,
                                    true,
                                    BlobListingDetails.Metadata,
                                    null,
                                    continuationToken,
                                    null,
                                    null);

           // The Calculate method returns the number of occurrences of
           // the search term "Microsoft" in each blob associated
           // with the data slice.
           //
           // The definition of the method is shown in the next step.
           output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

       } while (continuationToken != null);

       // Get the output dataset by using the name of the dataset matched to a name in the Activity output collection.
       Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);

       folderPath = GetFolderPath(outputDataset);

       logger.Write("Writing blob to the folder: {0}", folderPath);

       // Create a storage object for the output blob.
       CloudStorageAccount outputStorageAccount = CloudStorageAccount.Parse(connectionString);
       // Write the name of the file.
       Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));

       logger.Write("output blob URI: {0}", outputBlobUri.ToString());
       // Create a blob and upload the output text.
       CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
       logger.Write("Writing {0} to the output blob", output);
       outputBlob.UploadText(output);

       // The dictionary can be used to chain custom activities together in the future.
       // This feature is not implemented yet, so just return an empty dictionary.
       return new Dictionary<string, string>();
    }
    ```
1. Добавьте в класс следующие вспомогательные методы. Их вызывает метод **Execute** . Самое главное, метод **Calculate** изолирует код, который выполняет итерацию каждого большого двоичного объекта.

    ```csharp
    /// <summary>
    /// Gets the folderPath value from the input/output dataset.
    /// </summary>
    private static string GetFolderPath(Dataset dataArtifact)
    {
       if (dataArtifact == null || dataArtifact.Properties == null)
       {
           return null;
       }

       AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
       if (blobDataset == null)
       {
           return null;
       }

       return blobDataset.FolderPath;
    }

    /// <summary>
    /// Gets the fileName value from the input/output dataset.
    /// </summary>

    private static string GetFileName(Dataset dataArtifact)
    {
       if (dataArtifact == null || dataArtifact.Properties == null)
       {
           return null;
       }

       AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
       if (blobDataset == null)
       {
           return null;
       }

       return blobDataset.FileName;
    }

    /// <summary>
    /// Iterates through each blob (file) in the folder, counts the number of instances of the search term in the file,
    /// and prepares the output text that is written to the output blob.
    /// </summary>

    public static string Calculate(BlobResultSegment Bresult, IActivityLogger logger, string folderPath, ref BlobContinuationToken token, string searchTerm)
    {
       string output = string.Empty;
       logger.Write("number of blobs found: {0}", Bresult.Results.Count<IListBlobItem>());
       foreach (IListBlobItem listBlobItem in Bresult.Results)
       {
           CloudBlockBlob inputBlob = listBlobItem as CloudBlockBlob;
           if ((inputBlob != null) && (inputBlob.Name.IndexOf("$$$.$$$") == -1))
           {
               string blobText = inputBlob.DownloadText(Encoding.ASCII, null, null, null);
               logger.Write("input blob text: {0}", blobText);
               string[] source = blobText.Split(new char[] { '.', '?', '!', ' ', ';', ':', ',' }, StringSplitOptions.RemoveEmptyEntries);
               var matchQuery = from word in source
                                where word.ToLowerInvariant() == searchTerm.ToLowerInvariant()
                                select word;
               int wordCount = matchQuery.Count();
               output += string.Format("{0} occurrences(s) of the search term \"{1}\" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
           }
       }
       return output;
    }
    ```
    Метод GetFolderPath возвращает путь к папке, на которую указывает набор данных, а метод GetFileName — имя большого двоичного объекта или файла, на который указывает набор данных.

    ```csharp

    "name": "InputDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "fileName": "file.txt",
            "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
    ```

    Метод Calculate вычисляет количество экземпляров ключевого слова Microsoft во входных файлах (в больших двоичных объектах в папке). Условие поиска Microsoft жестко задано в коде.

1. Скомпилируйте проект. В меню выберите **Построить**, а затем — **Построить решение**.

1. Запустите проводник и перейдите к папке **bin\\debug** или **bin\\release**. Выбор папки зависит от типа сборки.

1. Создайте ZIP-файл **MyDotNetActivity.zip**, который содержит все двоичные файлы из папки **\\bin\\Debug**. Вы также можете добавить файл MyDotNetActivity.**pdb**, чтобы в случае сбоя получить дополнительные сведения, например номер строки в исходном коде, вызвавшей ошибку.

   ![Список папки bin\Debug](./media/data-factory-data-processing-using-batch/image5.png)

1. Передайте архив **MyDotNetActivity.zip** в качестве большого двоичного объекта в контейнер больших двоичных объектов `customactivitycontainer` в хранилище BLOB-объектов (используется связанной службой StorageLinkedService в ADFTutorialDataFactor). Если контейнер больших двоичных объектов `customactivitycontainer` еще не существует, создайте его.

#### <a name="execute-method"></a>Метод Execute
В этом разделе содержатся дополнительные сведения о коде в методе Execute.

1. Элементы для выполнения итерации входной коллекции находятся в пространстве имен [Microsoft.WindowsAzure.Storage.Blob](/java/api/com.microsoft.azure.storage.blob). Для итерации коллекции больших двоичных объектов необходимо использовать класс **BlobContinuationToken**. В сущности, необходимо использовать цикл do-while и маркер в качестве механизма для выхода из цикла. Дополнительные сведения см. в статье [Приступая к работе с хранилищем BLOB-объектов Azure с помощью .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md). Базовый цикл выглядит так:

    ```csharp
    // Initialize the continuation token.
    BlobContinuationToken continuationToken = null;
    do
    {
    // Get the list of input blobs from the input storage client object.
    BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,

                         true,
                                   BlobListingDetails.Metadata,
                                   null,
                                   continuationToken,
                                   null,
                                   null);
    // Return a string derived from parsing each blob.

     output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

    } while (continuationToken != null);

    ```
   Дополнительные сведения см. в документации по методу [ListBlobsSegmented](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmented).

1. Код для работы с набором больших двоичных объектов логически находится в цикле do-while. В методе **Execute** цикл do-while передает список больших двоичных объектов в метод **Calculate**. Этот метод возвращает строковую переменную с именем **output** , которая является результатом итерации всех больших двоичных объектов в сегменте.

   Он возвращает число вхождений условия поиска Microsoft в большом двоичном объекте, переданном в метод **Calculate**.

    ```csharp
    output += string.Format("{0} occurrences of the search term \"{1}\" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
    ```
1. После завершения работы метода **Calculate** его результаты записываются в новый большой двоичный объект. Для каждого обработанного набора больших двоичных объектов можно записать новый большой двоичный объект с результатами. Чтобы записать новый большой двоичный объект, сначала найдите выходной набор данных.

    ```csharp
    // Get the output dataset by using the name of the dataset matched to a name in the Activity output collection.
    Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);
    ```
1. Код также вызывает вспомогательный метод **GetFolderPath**, чтобы получить путь к папке (имя контейнера хранилища).

    ```csharp
    folderPath = GetFolderPath(outputDataset);
    ```
   Метод GetFolderPath приводит объект DataSet к AzureBlobDataSet со свойством FolderPath.

    ```csharp
    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

    return blobDataset.FolderPath;
    ```
1. Код вызывает метод **GetFileName** , чтобы получить имя файла (имя большого двоичного объекта). Используемый код аналогичен приведенному выше коду, используемому для получения пути к папке.

    ```csharp
    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

    return blobDataset.FileName;
    ```
1. Чтобы записать имя файла, создается объект универсального кода ресурса (URI). Для возврата имени контейнера в конструкторе URI используется свойство **BlobEndpoint** . Для создания URI выходного большого двоичного объекта сочетаются путь к папке и имя файла.  

    ```csharp
    // Write the name of the file.
    Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));
    ```
1. После записи имени файла вы можете записать выводимую строку из метода **Calculate** в новый большой двоичный объект.

    ```csharp
    // Create a blob and upload the output text.
    CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
    logger.Write("Writing {0} to the output blob", output);
    outputBlob.UploadText(output);
    ```

### <a name="create-the-data-factory"></a>Создание фабрики данных
В разделе [Создание настраиваемого действия](#create-the-custom-activity) вы создали настраиваемое действие и отправили ZIP-файл с двоичными файлами и PDB-файлом в контейнер больших двоичных объектов. В этом разделе вы создадите фабрику данных с конвейером, использующим пользовательское действие.

Входной набор данных для пользовательского действия представляет собой большие двоичные объекты (файлы) во входной папке (`mycontainer\\inputfolder`) в хранилище BLOB-объектов. Выходной набор данных для пользовательского действия представляет собой выходные большие двоичные объекты в папке выходных данных (`mycontainer\\outputfolder`) в хранилище BLOB-объектов.

Перенесите в папки входных данных один или несколько файлов.

```
mycontainer -\> inputfolder
    2015-11-16-00
    2015-11-16-01
    2015-11-16-02
    2015-11-16-03
    2015-11-16-04
```

Например, перенесите один файл (file.txt) со следующим содержимым в каждую папку.

```
test custom activity Microsoft test custom activity Microsoft
```

Каждая входная папка соответствует одному срезу в фабрике данных, даже если она содержит два файла или более. При обработке каждого среза конвейером пользовательское действие выполняет итерацию всех больших двоичных объектов во входной папке для этого среза.

Отображаются пять выходных файлов с одинаковым содержимым. Например, выходной файл, полученный в ходе обработки файла в папке 2015-11-16-00, включает в себя следующее содержимое.

```
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
```

Если в папку входных данных перенести несколько файлов (file.txt, file2.txt, file3.txt) с одинаковым содержимым, то в выходном файле будет следующее содержимое. В этом примере каждая папка (2015-11-16-00 и т. д.) соответствует одному срезу, несмотря на то, что она содержит несколько входных файлов.

```csharp
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file2.txt.
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file3.txt.
```

Теперь во входном файле появилось три строки, по одной на каждый входной файл (большой двоичный объект) в папке, связанный со срезом (2015-11-16-00).

Такая задача создается для каждого запуска действия. В этом примере в конвейере есть только одно действие. При обработке среза конвейером в пакетной службе выполняется настраиваемое действие для обработки среза. Так как имеется пять срезов (каждому срезу может соответствовать несколько больших двоичных объектов или файлов), в пакетной службе создано пять задач. При запуске задачи в пакетной службе выполняется настраиваемое действие.

В следующем пошаговом руководстве содержатся дополнительные сведения.

#### <a name="step-1-create-the-data-factory"></a>Шаг 1. Создание фабрики данных
1. После входа на [портал Azure](https://portal.azure.com/) сделайте следующее:

   а. В меню слева выберите **Создать**.

   b. В колонке **Создание** выберите **Данные+аналитика**.

   c. Выберите **Фабрика данных** в колонке **Аналитика данных**.

1. В колонке **Новая фабрика данных** введите **CustomActivityFactory** в качестве имени. Имя фабрики данных должно быть глобально уникальным. Если появится сообщение об ошибке "Имя CustomActivityFactory фабрики данных недоступно", измените имя фабрики данных. Например, используйте "ваше_имя_CustomActivityFactory" и снова создайте фабрику данных.

1. Выберите **Имя группы ресурсов**, чтобы создать группу ресурсов или использовать имеющуюся.

1. Убедитесь, что подписка и регион, в которых необходимо создать фабрику данных, указаны верно.

1. В колонке **Новая фабрика данных** нажмите кнопку **Создать**.

1. На панели мониторинга портала будет создана фабрика данных.

1. Ее содержимое отобразится на странице **Фабрика данных**.

   ![Страница "Фабрика данных"](./media/data-factory-data-processing-using-batch/image6.png)

#### <a name="step-2-create-linked-services"></a>Шаг 2. Создание связанных служб
Связанные службы связывают хранилища данных или службы вычислений с фабрикой данных. На этом этапе учетная запись службы хранилища и учетная запись пакетной службы будут связаны с фабрикой данных.

#### <a name="create-an-azure-storage-linked-service"></a>Создание связанной службы хранилища Azure
1. В колонке **Фабрика данных** для **CustomActivityFactory** щелкните элемент **Создать и развернуть**. Отобразится редактор фабрики данных.

1. На панели команд выберите **Новое хранилище данных** и выберите **Служба хранилища Azure**. В редакторе отобразится скрипт JSON, используемый для создания связанной службы хранилища.

   ![Новое хранилище данных](./media/data-factory-data-processing-using-batch/image7.png)

1. Замените **account name** именем своей учетной записи хранения. Замените **ключ учетной записи** ключом вашей учетной записи хранения. Сведения о том, как получить ключ доступа к хранилищу, см. в статье [Управление ключами доступа к хранилищу](../../storage/common/storage-account-keys-manage.md).

1. Чтобы развернуть связанную службу, выберите **Развернуть** на панели команд.

   ![Развертывание](./media/data-factory-data-processing-using-batch/image8.png)

#### <a name="create-an-azure-batch-linked-service"></a>Создание связанной пакетной службы Azure
На этом шаге вы создаете связанную службу для учетной записи пакетной службы, в которой будет выполняться настраиваемое действие фабрики данных.

1. Выберите **Новое вычисление** в командной строке, а затем — **Пакет Azure**. В редакторе отобразится скрипт JSON, используемый для создания связанной пакетной службы.

1. В сценарии JSON выполните следующее:

   а. Замените **account name** именем учетной записи пакетной службы.

   b. Замените **access key** ключом доступа к учетной записи пакетной службы.

   c. Введите идентификатор пула для свойства **poolName**. Для этого свойства можно задать имя или идентификатор пула.

   d. Введите URI пакета для свойства JSON **batchUri** .

      > [!IMPORTANT]
      > URL-адрес в колонке **учетной записи пакетной службы** имеет следующий формат: `<accountname>.<region>.batch.azure.com` . Для `batchUri` свойства в скрипте JSON необходимо удалить `<accountname>.` из URL-адреса. Например, `"batchUri": "https://eastus.batch.azure.com"`.

      ![Колонка учетной записи пакетной службы](./media/data-factory-data-processing-using-batch/image9.png)

      Для свойства **poolName** можно также указать идентификатор пула вместо его имени.

      > [!NOTE]
      > Служба фабрики данных не поддерживает параметр по требованию для пакетной службы, как для HDInsight. В фабрике данных можно использовать только собственный пул пакетной службы.
      >
      >

   д) В качестве параметра семейства операционных систем укажите **StorageLinkedService** for the **linkedServiceName** . Эта связанная служба была создана на предыдущем шаге. Это хранилище используется в качестве промежуточной области для файлов и журналов.

1. Чтобы развернуть связанную службу, выберите **Развернуть** на панели команд.

#### <a name="step-3-create-datasets"></a>Шаг 3. Создание наборов данных
На этом шаге вы создадите наборы данных, которые представляют входные и выходные данные.

#### <a name="create-the-input-dataset"></a>Создание входного набора данных
1. В редакторе фабрики данных на панели инструментов нажмите кнопку **Новый набор данных**. В раскрывающемся списке выберите **Хранилище BLOB-объектов Azure**.

1. Замените скрипт JSON в правой области на следующий фрагмент кода JSON:

    ```json
    {
       "name": "InputDataset",
       "properties": {
           "type": "AzureBlob",
           "linkedServiceName": "AzureStorageLinkedService",
           "typeProperties": {
               "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
               "format": {
                   "type": "TextFormat"
               },
               "partitionedBy": [
                   {
                       "name": "Year",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "yyyy"
                       }
                   },
                   {
                       "name": "Month",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "MM"
                       }
                   },
                   {
                       "name": "Day",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "dd"
                       }
                   },
                   {
                       "name": "Hour",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "HH"
                       }
                   }
               ]
           },
           "availability": {
               "frequency": "Hour",
               "interval": 1
           },
           "external": true,
           "policy": {}
       }
    }
    ```

    Позже в этом пошаговом руководстве вы создадите конвейер с временем начала 2015-11-16T00:00:00Z и временем окончания 2015-11-16T05:00:00Z. Данные будут создаваться почасово, поэтому мы получим пять входных и выходных срезов (от **00**:00:00 до\> **05**:00:00).

    Для параметров **frequency** и **interval** входного набора данных установлены значения **Hour** и **1**. Это означает, что входной срез данных будет создаваться каждый час.

    Значения времени начала для каждого среза, представленные системной переменной **SliceStart** в приведенном выше фрагменте кода JSON. Ниже приведены значения времени начала для каждого среза.

    | **Срез** | **Время начала**          |
    |-----------|-------------------------|
    | 1         | 2015-11-16T **00**:00:00 |
    | 2         | 2015-11-16T **01**:00:00 |
    | 3         | 2015-11-16T **02**:00:00 |
    | 4         | 2015-11-16T **03**:00:00 |
    | 5         | 2015-11-16T **04**:00:00 |

    Значение параметра **folderPath** вычисляется с использованием года, месяца, дня и часа значения времени начала среза (**SliceStart**). Вот как входная папка сопоставляется со срезом.

    | **Срез** | **Время начала**          | **Входная папка**  |
    |-----------|-------------------------|-------------------|
    | 1         | 2015-11-16T **00**:00:00 | 2015-11-16-**00** |
    | 2         | 2015-11-16T **01**:00:00 | 2015-11-16-**01** |
    | 3         | 2015-11-16T **02**:00:00 | 2015-11-16-**02** |
    | 4         | 2015-11-16T **03**:00:00 | 2015-11-16-**03** |
    | 5         | 2015-11-16T **04**:00:00 | 2015-11-16-**04** |

1. На панели инструментов выберите **Развернуть**, чтобы создать и развернуть таблицу **InputDataset**.

#### <a name="create-the-output-dataset"></a>Создание выходного набора данных
На этом шаге создается еще один набор данных типа AzureBlob для представления выходных данных.

1. В редакторе фабрики данных на панели инструментов нажмите кнопку **Новый набор данных**. В раскрывающемся списке выберите **Хранилище BLOB-объектов Azure**.

1. Замените скрипт JSON в правой области на следующий фрагмент кода JSON:

    ```json
    {
       "name": "OutputDataset",
       "properties": {
           "type": "AzureBlob",
           "linkedServiceName": "AzureStorageLinkedService",
           "typeProperties": {
               "fileName": "{slice}.txt",
               "folderPath": "mycontainer/outputfolder",
               "partitionedBy": [
                   {
                       "name": "slice",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "yyyy-MM-dd-HH"
                       }
                   }
               ]
           },
           "availability": {
               "frequency": "Hour",
               "interval": 1
           }
       }
    }
    ```

    Для каждого входного среза данных создается выходной большой двоичный объект или файл. Ниже в таблице приведены имена, которые даются выходным файлам для каждого среза. Все выходные файлы создаются в одной папке выходных данных: `mycontainer\\outputfolder`.

    | **Срез** | **Время начала**          | **Выходной файл**       |
    |-----------|-------------------------|-----------------------|
    | 1         | 2015-11-16T **00**:00:00 | 2015-11-16-**00.txt** |
    | 2         | 2015-11-16T **01**:00:00 | 2015-11-16-**01.txt** |
    | 3         | 2015-11-16T **02**:00:00 | 2015-11-16-**02.txt** |
    | 4         | 2015-11-16T **03**:00:00 | 2015-11-16-**03.txt** |
    | 5         | 2015-11-16T **04**:00:00 | 2015-11-16-**04.txt** |

    Помните, что все файлы во входной папке (например, 2015-11-16-00) являются частью среза со значениями времени начала (2015-11-16-00). Во время обработки этого среза пользовательское действие сканирует каждый файл и создает строку в выходном файле с количеством вхождений условия поиска Microsoft. Если в папке 2015-11-16-00 есть три файла, то в выходном файле 2015-11-16-00.txt будет три строки.

1. На панели инструментов выберите **Развернуть**, чтобы создать и развернуть **OutputDataset**.

#### <a name="step-4-create-and-run-the-pipeline-with-a-custom-activity"></a>Шаг 4. Создание и запуск конвейера с настраиваемым действием
На этом шаге создается конвейер с одним настраиваемым действием, созданным ранее.

> [!IMPORTANT]
> Если файл **file.txt** не отправлен в папки входных данных в контейнере больших двоичных объектов, сделайте это, прежде чем создавать конвейер. В конвейере JSON для свойства **IsPaused** задано значение false, поэтому конвейер запустится сразу по прошествии даты, указанной в параметре **start**.
>
>

1. В редакторе фабрики данных выберите **Новый конвейер** на панели команд. Если команда не отображается, щелкните символ многоточия, чтобы ее отобразить.

1. Замените скрипт JSON в правой области на следующий фрагмент кода JSON:

    ```json
    {
       "name": "PipelineCustom",
       "properties": {
           "description": "Use custom activity",
           "activities": [
               {
                   "type": "DotNetActivity",
                   "typeProperties": {
                       "assemblyName": "MyDotNetActivity.dll",
                       "entryPoint": "MyDotNetActivityNS.MyDotNetActivity",
                       "packageLinkedService": "AzureStorageLinkedService",
                       "packageFile": "customactivitycontainer/MyDotNetActivity.zip"
                   },
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
                   "policy": {
                       "timeout": "00:30:00",
                       "concurrency": 5,
                       "retry": 3
                   },
                   "scheduler": {
                       "frequency": "Hour",
                       "interval": 1
                   },
                   "name": "MyDotNetActivity",
                   "linkedServiceName": "AzureBatchLinkedService"
               }
           ],
           "start": "2015-11-16T00:00:00Z",
           "end": "2015-11-16T05:00:00Z",
           "isPaused": false
      }
    }
    ```
   Обратите внимание на следующие моменты.

   * В конвейере содержится одно действие типа **DotNetActivity**.
   * В **AssemblyName** задано имя библиотеки DLL **MyDotNetActivity.dll**.
   * В **EntryPoint** — **MyDotNetActivityNS.MyDotNetActivity**. По сути, это \<namespace\> .\<classname\> в коде.
   * Для параметра **PackageLinkedService** задано значение **StorageLinkedService**, которое указывает на хранилище BLOB-объектов, содержащее настраиваемый ZIP-файл. Если для входных и выходных файлов и ZIP-файла настраиваемого действия используются разные учетные записи хранения, то необходимо создать другую связанную службу хранилища. В этой статье предполагается, что вы используете одну и ту же учетную запись хранения.
   * Для **PackageFile** установлено значение **customactivitycontainer/MyDotNetActivity.zip**. Он имеет формат \<containerforthezip\> / \<nameofthezip.zip\> .
   * Настраиваемое действие принимает **InputDataset** в качестве входных данных и **OutputDataset** — в качестве выходных данных.
   * Свойство **linkedServiceName** настраиваемого действия указывает на **AzureBatchLinkedService**, которое сообщает фабрике данных, что необходимо запустить настраиваемое действие в пакетной службе.
   * Обратите внимание на параметр **concurrency**. Если используется значение по умолчанию, т. е. 1, то даже если в пуле пакетной службы есть два или больше вычислительных узлов, срезы обрабатываются по очереди. Таким образом игнорируется возможность параллельной обработки пакетной службы. Если задать для параметра **concurrency** большее значение, например 2, это означает, что одновременно могут обрабатываться два среза (соответствующие двум задачам в пакетной службе). В этом случае используются обе виртуальные машины в пуле пакетной службы. Укажите соответствующее значение для свойства concurrency.
   * По умолчанию на виртуальной машине в любой момент времени будет выполняться только одна задача (один срез). По умолчанию для параметра **Maximum tasks per VM** (Максимальное количество задач на виртуальную машину) задано значение 1 для пула пакетной службы. В ходе выполнения необходимых требований вы создали пул со значением 2 для этого свойства. Поэтому на виртуальной машине могут выполняться два среза фабрики данных одновременно.
     - Для свойства **isPaused** по умолчанию установлено значение false. В этом примере конвейер запускается незамедлительно, потому что срезы приходятся на прошлое. Для этого свойства можно задать значение **true**, чтобы приостановить работу конвейера. Чтобы перезапустить его, нужно снова установить значение **false**.
     -   Разница между временем **начала** и временем **окончания** составляет пять часов. Срезы создаются каждый час. Таким образом конвейер создает пять срезов.

1. Чтобы развернуть конвейер, нажмите кнопку **Развернуть** на панели команд.

#### <a name="step-5-test-the-pipeline"></a>Шаг 5. Тестирование конвейера
На этом шаге вы протестируете конвейер. Для этого файлы будут перенесены в папки входных данных. Сначала перенесем один файл в каждую входную папку.

1. На портале Azure в колонке **Фабрика данных** выберите **Схема**.

   ![Схема](./media/data-factory-data-processing-using-batch/image10.png)

1. В представлении **Схема** дважды щелкните входной набор данных **InputDataset**.

   ![InputDataset](./media/data-factory-data-processing-using-batch/image11.png)

1. Отобразится колонка **InputDataset** со всеми пятью срезами. Обратите внимание на значения в столбцах **Время начала среза** и **Время окончания среза** для каждого среза.

   ![Время начала и окончания входного среза](./media/data-factory-data-processing-using-batch/image12.png)

1. Теперь в представлении **Схема** выберите **OutputDataset**.

1. Если срезы уже созданы, пять выходных срезов отобразятся в состоянии **готовности**.

   ![Время начала и окончания выходного среза](./media/data-factory-data-processing-using-batch/image13.png)

1. На портале просмотрите задачи, связанные со срезами, а также сведения о том, на какой виртуальной машине выполнялся каждый срез. Дополнительные сведения см. в разделе [Интеграция фабрики данных и пакетной службы](#data-factory-and-batch-integration).

1. Выходные файлы появятся в папке `mycontainer` контейнера `outputfolder` в хранилище BLOB-объектов.

   ![Выходные файлы в хранилище](./media/data-factory-data-processing-using-batch/image15.png)

   Перечислено пять выходных файлов, по одному на каждый входной срез. Каждый из выходных файлов имеет содержимое, аналогичное приведенному ниже.

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
    ```
   На указанной ниже схеме показано сопоставление срезов данных фабрики и задач в пакетной службе. В этом примере срез запускался всего один раз.

   ![Схема сопоставления срезов](./media/data-factory-data-processing-using-batch/image16.png)

1. Теперь попробуем использовать несколько файлов в папке. Создайте файлы **file2.txt**, **file3.txt**, **file4.txt** и **file5.txt** с тем же содержимым, что и в файле file.txt в папке **2015-11-06-01**.

1. В папке выходных данных удалите выходной файл **2015-11-16-01.txt**.

1. В колонке **OutputDataset** щелкните правой кнопкой мыши срез, для которого в столбце **Время начала среза** задано значение **11/16/2015 01:00:00 AM**. Выберите **Запустить** для повторного выполнения и обработки среза. Теперь в срезе пять файлов вместо одного.

    ![Run (Запустить)](./media/data-factory-data-processing-using-batch/image17.png)

1. Когда после запуска среза его состояние изменится на **Готов**, проверьте содержимое в выходном файле для этого среза (**2015-11-16-01.txt**). Выходные файлы появятся в папке `mycontainer` контейнера `outputfolder` в хранилище BLOB-объектов. В выходном файле каждому файлу среза должна соответствовать одна строка.

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file2.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file3.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file4.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file5.txt.
    ```

> [!NOTE]
> Если не удалить выходной файл 2015-11-16-01.txt перед использованием пяти входных файлов, появится одна строка из предыдущего запуска среза и пять строк из текущего. По умолчанию содержимое добавляется в выходной файл, если он уже создан.
>
>

#### <a name="data-factory-and-batch-integration"></a>Интеграция фабрики данных и пакетной службы
Служба фабрики данных создает в пакетной службе задание с именем `adf-poolname:job-xxx`.

![Пакетные задания](media/data-factory-data-processing-using-batch/data-factory-batch-jobs.png)

Для каждого запуска действия среза в задании создается задача. Если 10 срезов готовы к обработке, в рамках этого задания создаются 10 задач. Если в пуле доступны несколько вычислительных узлов, можно обрабатывать параллельно сразу несколько срезов. Если для максимального количества задач на вычислительный узел установлено значение больше одного, можно также выполнять сразу несколько срезов в одном цикле вычислений.

В этом примере имеется пять срезов, поэтому в пакетной службе есть пять задач. Так как для свойства **concurrency** задано значение **5** в конвейере JSON в фабрике данных, а для параметра **Maximum tasks per VM** (Максимальное число задач на виртуальную машину) — **2** в пуле пакетной службы с **2** виртуальными машинами, задачи будут выполняться очень быстро. (Просмотрите время начала и завершения задач.)

На портале просмотрите задание пакетной службы и его задачи, связанные со срезами, а также сведения о том, на какой виртуальной машине выполнялся каждый срез.

![Задачи пакетного задания](media/data-factory-data-processing-using-batch/data-factory-batch-job-tasks.png)

### <a name="debug-the-pipeline"></a>Выполнение отладки конвейера
Отладка состоит из нескольких базовых методов.

1. Если для входного среза данных не установлено значение **Готов**, убедитесь, что структура входной папки правильная и что в ней имеется файл file.txt.

   ![Структура входной папки](./media/data-factory-data-processing-using-batch/image3.png)

1. В методе **Execute** настраиваемого действия используйте объект **IActivityLogger**, чтобы записывать в журнал сведения, которые помогут устранить неполадки. Сообщения, записанные в журнал, сохраняются в файле user\_0.log.

   В колонке **OutputDataset** выберите срез, чтобы открыть для него колонку **Срез данных**. В разделе **Выполнения действий** видно, что выполняется лишь одно действие. Если в командной строке выбрать **Запустить**, можно запустить другое действие для этого среза.

   Если выбрать выполнение действия, появится колонка **Подробности о выполнении операции** со списком файлов журнала. Записанные в журнал сообщения можно просмотреть в файле user\_0.log. Если возникнет ошибка, действие будет выполнено три раза, так как для количества повторных попыток в JSON-файле конвейера или действия задано значение 3. Если выбрать выполнение действия, отобразятся файлы журнала, которые можно использовать для устранения ошибки.

   ![Колонки OutputDataset и "Срез данных"](./media/data-factory-data-processing-using-batch/image18.png)

   В списке файлов журнала выберите **user-0.log**. На панели справа отображаются результаты использования метода **IActivityLogger.Write**.

   ![Колонка "Подробности о выполнении операции"](./media/data-factory-data-processing-using-batch/image19.png)

   Проверьте файл system-0.log на наличие любых системных сообщений об ошибках или исключениях.

    ```
    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Loading assembly file MyDotNetActivity...

    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Creating an instance of MyDotNetActivityNS.MyDotNetActivity from assembly file MyDotNetActivity...

    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Executing Module

    Trace\_T\_D\_12/6/2015 1:43:38 AM\_T\_D\_\_T\_D\_Information\_T\_D\_0\_T\_D\_Activity e3817da0-d843-4c5c-85c6-40ba7424dce2 finished successfully
    ```
1. Добавьте **PDB-файл** в ZIP-файл, чтобы в случае возникновения ошибки сведения об ошибке содержали, например, информацию о стеке вызовов.

1. Все файлы в ZIP-файле для пользовательского действия должны находиться на верхнем уровне без подпапок.

   ![Список ZIP-файлов настраиваемого действия](./media/data-factory-data-processing-using-batch/image20.png)

1. Убедитесь, что для параметров **assemblyName** (MyDotNetActivity.dll), **entryPoint** (MyDotNetActivityNS.MyDotNetActivity), **packageFile** (customactivitycontainer/MyDotNetActivity.zip) и **packageLinkedService** (должен указывать на хранилище BLOB-объектов, содержащее ZIP-файл) установлены правильные значения.

1. Если ошибки устранены и необходимо повторно обработать срез, в колонке **OutputDataset** щелкните срез правой кнопкой мыши и выберите **Запустить**.

   ![Параметр "Запустить" в колонке OutputDataset](./media/data-factory-data-processing-using-batch/image21.png)

   > [!NOTE]
   > В хранилище BLOB-объектов появится контейнер с именем `adfjobs`. Этот контейнер не удаляется автоматически, но его можно безопасно удалить после завершения тестирования решения. Аналогичным образом решение фабрики данных создает задание пакетной службы с именем `adf-\<pool ID/name\>:job-0000000001`. При необходимости после выполнения тестирования решения это задание можно удалить.
   >
   >
1. Настраиваемое действие не использует файл **app.config** из пакета. Поэтому, если код считывает какие-либо строки подключения из файла конфигурации, это не сработает во время выполнения. При использовании пакетной службы все секреты рекомендуется хранить в Azure Key Vault. Для защиты хранилища ключей используйте субъект-службу на основе сертификата, который помещен в пул пакетной службы. В этом случае настраиваемое действие .NET в среде выполнения сможет использовать секреты из хранилища ключей. Это общее решение можно использовать для любого типа секретов, а не только строк подключения.

    Имеется и более простое (но не самое лучшее) решение. Можно создать связанную службу базы данных SQL с параметрами строки подключения. Затем вы можете создать набор данных, использующий связанную службу, и привязать его как фиктивный набор входных данных к настраиваемому действию .NET. После этого вы сможете получить доступ к строке подключения связанной службы из кода пользовательского действия. В среде выполнения она должна работать нормально.  

#### <a name="extend-the-sample"></a>Расширение примера
Для получения дополнительных сведений о функциях фабрики данных и пакетной службы этот пример можно расширить. Например, чтобы обрабатывать срезы с использованием другого диапазона времени, сделайте следующее.

1. Добавьте в папку `inputfolder` подпапки: 2015-11-16-05, 2015-11-16-06, 201-11-16-07, 2011-11-16-08 и 2015-11-16-09. Поместите в них входные файлы. Измените время окончания для конвейера с `2015-11-16T05:00:00Z` на `2015-11-16T10:00:00Z`. В представлении **схемы** дважды щелкните **InputDataset** и убедитесь, что для входных срезов отображается состояние "Готово". Дважды щелкните **OuptutDataset**, чтобы просмотреть состояние выходных срезов. Если отображается состояние **Готово**, проверьте выходные файлы в выходной папке.

1. Укажите для параметра **concurrency** большее или меньшее значение, чтобы понять, как это влияет на производительность решения, особенно на обработку в пакетной службе. Дополнительные сведения о **параллелизме** параметров см. в разделе "Шаг 4. Создание и запуск конвейера с настраиваемым действием".

1. Создайте пул с большим или меньшим значением для параметра **Maximum tasks per VM** (Максимальное число задач на виртуальную машину). Чтобы воспользоваться созданным пулом, обновите связанную службу пакетной службы в решении фабрики данных. Дополнительные сведения о параметре **Maximum tasks per VM** (Максимальное число задач на виртуальную машину) см. в разделе "Шаг 4. Создание и запуск конвейера с настраиваемым действием".

1. Создайте пул пакетной службы с использованием функции **автомасштабирования**. Автоматическое масштабирование вычислительных узлов в пуле пакетной службы представляет собой динамическую настройку вычислительной мощности, используемой вашим приложением.

    Приведенный здесь пример формулы обеспечивает следующее поведение. Созданный пул изначально содержит одну виртуальную машину. Метрика $PendingTasks определяет количество задач в состоянии выполнения и активном состоянии (в очереди). Формула находит среднее число ожидающих выполнения задач за последние 180 секунд и соответствующим образом задает значение TargetDedicated. Благодаря этому значение TargetDedicated никогда не превысит 25 виртуальных машин. По мере добавления новых задач пул автоматически расширяется. При завершении задач виртуальные машины поочередно высвобождаются, и функция автомасштабирования сжимает виртуальные машины. Вы можете настроить значения startingNumberOfVMs и maxNumberofVMs в соответствии со своими потребностями.

    Формула автоматического масштабирования:

    ```
    startingNumberOfVMs = 1;
    maxNumberofVMs = 25;
    pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
    pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
    $TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
    ```

   Дополнительные сведения см. в статье [Создание формулы автоматического масштабирования для масштабирования вычислительных узлов в пуле пакетной службы](../../batch/batch-automatic-scaling.md).

   Если в пуле используется [autoScaleEvaluationInterval](/rest/api/batchservice/pool/enableautoscale) (значение по умолчанию), пакетной службе может потребоваться 15–30 минут на подготовку виртуальной машины перед выполнением настраиваемого действия. Если пул использует другое значение autoScaleEvaluationInterval, пакетная служба может затрачивать autoScaleEvaluationInterval плюс 10 минут.

1. В примере решения метод **Execute** вызывает метод **Calculate**, который обрабатывает входной срез данных для создания выходного среза данных. Вы можете написать собственный метод для обработки входных данных и заменить вызов метода **Calculate** в методе **Execute** его вызовом.

### <a name="next-steps-consume-the-data"></a>Дальнейшие действия: Использование данных
После обработки данных их можно использовать в интерактивных средах, например в Power BI. Ниже приведены ссылки на статьи, в которых представлены общие сведения о Power BI и информация о том, как использовать это средство в Azure.

* [Изучение набора данных в Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-data/)
* [Начало работы с Power BI Desktop](/power-bi/fundamentals/desktop-getting-started)
* [Обновление данных в Power BI](https://powerbi.microsoft.com/documentation/powerbi-refresh-data/)
* [Azure Power BI](https://powerbi.microsoft.com/documentation/powerbi-azure-and-power-bi/) (Azure и Power BI)

## <a name="references"></a>Ссылки
* [Фабрика данных Azure](https://azure.microsoft.com/documentation/services/data-factory/).

  * [Введение в фабрику данных Azure](data-factory-introduction.md)
  * [Учебник. Создание первого конвейера для преобразования данных с помощью кластера Hadoop](data-factory-build-your-first-pipeline.md)
  * [Использование настраиваемых действий в конвейере фабрики данных Azure](data-factory-use-custom-activities.md)
* [Пакетная служба Azure](https://azure.microsoft.com/documentation/services/batch/)

  * [Выполнение реальных параллельных рабочих нагрузок с использованием пакетной службы](../../azure-sql/database/sql-database-paas-overview.md)
  * [Обзор функций пакетной](../../batch/batch-service-workflow-features.md)службы)
  * [Создание учетной записи пакетной службы на портале Azure](../../batch/batch-account-create-portal.md)
  * [Приступая к созданию решений с помощью клиентской библиотеки пакетной службы для .NET](../../batch/quick-run-dotnet.md)

[batch-explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[batch-explorer-walkthrough]: /archive/blogs/windowshpc/azure-batch-explorer-sample-walkthrough