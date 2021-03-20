---
title: 'ML Studio (классическая модель): повторное обучение веб-службы в Azure'
description: Узнайте, как обновить веб-службу для использования недавно обученной модели машинного обучения в Машинное обучение Azure Studio (классическая модель).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18, devx-track-csharp
ms.date: 02/14/2019
ms.openlocfilehash: a4fe9e54e5e03a8dbf2a727b22f784c36d6c65f9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100517592"
---
# <a name="retrain-and-deploy-a-machine-learning-model"></a>Повторное обучение и развертывание модели машинного обучения

**ПРИМЕНИМО К:**  ![Применимо к.](../../../includes/media/aml-applies-to-skus/yes.png)Студия машинного обучения (классическая)   ![Неприменимо к.](../../../includes/media/aml-applies-to-skus/no.png)[Машинное обучение Azure](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Повторное обучение — один из способов поддерживать точность моделей машинного обучения и учитывать самые актуальные доступные данные. В этой статье показано, как переучить и развернуть модель машинного обучения в качестве новой веб-службы в Studio (классической). Если вас интересует повторное обучение для классической веб-службы, [изучите это практическое руководство.](retrain-classic-web-service.md)

Для работы с этой статьей требуется развернутая прогнозная веб-служба. Если у вас еще нет прогнозной веб-службы, [Узнайте, как развернуть веб-службу Studio (классическая).](deploy-a-machine-learning-web-service.md)

Для повторного обучения и развертывания новой веб-службы машинного обучения вам потребуется выполнить следующие задачи:

1. Развертывание **переобучающей веб-службы**.
1. Обучение новой модели с помощью этой **переобучающей веб-службы**.
1. Обновление существующего **прогнозного эксперимента** для использования новой модели.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="deploy-the-retraining-web-service"></a>Разверните переобучающую веб-службу.

Переобучающая веб-служба позволяет повторно обучать модели с новым набором параметров, например с новыми данными, и сохранить ее для последующего использования. Вы подключаете модуль **Web Service Output** (Выходные данные веб-службы) к модулю **Train Model** (Обучение модели), и обучающий эксперимент создает новую модель для использования.

Следуйте приведенным ниже инструкциям, чтобы развернуть переобучающую веб-службу:

1. Подключите модуль **Web Service Input** (Входные данные веб-службы) к нужному источнику данных. Вы, естественно, захотите убедиться, что входные данные будут обработаны так же, как исходный набор обучающих данных.
1. Подключите модуль **Web Service Output** (Выходные данные веб-службы) к выходу **Train Model** (Обучение модели).
1. Если у вас есть модуль **Evaluate Model** (Оценка модели), вы можете присоединить к нему **Web Service Output** (Выходные данные веб-службы), чтобы получить результаты оценки.
1. Запустите эксперимент.

    После запуска эксперимента рабочий процесс должен выглядеть примерно так:

    ![Итоговый рабочий процесс](media/retrain-machine-learning/machine-learning-retrain-models-programmatically-IMAGE04.png)

    Теперь разверните обучающий эксперимент в качестве обучающей веб-службы, которая выводит обученную модель и результаты ее оценки.

1. В нижней части холста эксперимента щелкните **Set Up Web Service** (Настроить веб-службу).
1. Выберите **Deploy Web Service [New]** (Развернуть веб-службу (новую)). Портал веб-служб Машинного обучения Azure откроется на странице **Deploy Web service** (Развертывание веб-службы).
1. Введите имя веб-службы и выберите план платежей.
1. Выберите **Развернуть**.

## <a name="retrain-the-model"></a>Повторное обучение модели

В этом примере для создания приложения переобучения используется код на языке C#. Кроме того, для этого можно использовать образцы кода на языке R или Python.

Следуйте приведенным ниже инструкциям, чтобы вызвать API переобучения:

1. Создание консольного приложения C# в Visual Studio: **Создание**  >  **проекта**  >  **Visual C#**  >  **классическое**  >  **консольное приложение Windows (платформа .NET Framework)**.
1. Войдите на портал веб-служб Машинного обучения.
1. Щелкните веб-службу, с которой работаете.
1. Щелкните **Consume**(Использование).
1. В нижней части страницы **Consume** (Использование) в разделе **Sample Code** (Пример кода) щелкните **Batch** (Пакет).
1. Скопируйте пример кода на C# для пакетного выполнения и вставьте его в файл Program.cs. Убедитесь, что пространство имен не изменено.

Добавьте пакет NuGet Microsoft.AspNet.WebApi.Client, как указано в комментариях. Чтобы добавить ссылку на файл Microsoft.WindowsAzure.Storage.dll, возможно, потребуется установить [клиентскую библиотеку для служб хранилища Azure](https://www.nuget.org/packages/WindowsAzure.Storage).

На следующем снимке экрана показана станица **Consume** (Использование) на портале веб-служб Машинного обучения Azure.

![Страница Consume (Использование)](media/retrain-machine-learning/machine-learning-retrain-models-consume-page.png)

### <a name="update-the-apikey-declaration"></a>Обновление объявления apiKey

Найдите объявление **apiKey**:

```csharp
const string apiKey = "abc123"; // Replace this with the API key for the web service
```

В разделе **Основные сведения о потреблении** на странице **Использование** выберите первичный ключ и скопируйте его в объявление **apiKey** .

### <a name="update-the-azure-storage-information"></a>Обновление сведений о службе хранилища Azure

В этом примере кода BES файл с локального диска (например, C:\temp\CensusIpnput.csv) отправляется в службу хранилища Azure, где он обрабатывается. Затем результаты записываются обратно в службу хранилища Azure.

1. Войдите на портал Azure.
1. В столбце слева щелкните **More services** (Дополнительные службы), найдите параметр **Учетные записи хранения** и выберите его.
1. Выберите в списке учетную запись хранения, которая будет использоваться для хранения переобученной модели.
1. В левой области навигации щелкните **Ключи доступа**.
1. Скопируйте и сохраните **первичный ключ доступа**.
1. В левом столбце навигации щелкните **Blobs (большие двоичные объекты**).
1. Выберите существующий контейнер или создайте другой, а затем сохраните его имя.

Найдите объявления *StorageAccountName*, *StorageAccountKey* и *StorageContainerName*, а затем обновите их, используя значения с портала.

```csharp
const string StorageAccountName = "mystorageacct"; // Replace this with your Azure storage account name
const string StorageAccountKey = "a_storage_account_key"; // Replace this with your Azure Storage key
const string StorageContainerName = "mycontainer"; // Replace this with your Azure Storage container name
```

Кроме того, необходимо обеспечить доступность файла входных данных в расположении, указанном в коде.

### <a name="specify-the-output-location"></a>Указание расположения выходных данных

Указывая расположение выходных данных для полезных данных запроса, измените расширение файла, заданное в *RelativeLocation*, на `ilearner`.

```csharp
Outputs = new Dictionary<string, AzureBlobDataReference>() {
    {
        "output1",
        new AzureBlobDataReference()
        {
            ConnectionString = storageConnectionString,
            RelativeLocation = string.Format("{0}/output1results.ilearner", StorageContainerName) /*Replace this with the location you want to use for your output file and a valid file extension (usually .csv for scoring results or .ilearner for trained models)*/
        }
    },
```

Ниже приведен пример выходных данных переобучения:

![Выходные данные переобучения](media/retrain-machine-learning/machine-learning-retrain-models-programmatically-IMAGE06.png)

### <a name="evaluate-the-retraining-results"></a>Оценка результатов переобучения

При выполнении приложения выходные данные содержат URL-адрес и подписанные URL-адреса, необходимые для доступа к результатам оценки.

Чтобы увидеть результаты работы переобученной модели, введите в адресную строку браузера полный URL-адрес, составив его из значений параметров *BaseLocation*, *RelativeLocaiton* и *SasBlobToken*, содержащихся в выходных данных *output2*.

Проанализируйте результаты и определите, можно ли только что обученную модель считать лучше предыдущей.

Сохраните значения *BaseLocation*, *RelativeLocation* и *SasBlobToken* из полученных результатов.

## <a name="update-the-predictive-experiment"></a>Обновление прогнозного эксперимента

### <a name="sign-in-to-azure-resource-manager"></a>Вход в Azure Resource Manager

Во-первых, войдите в свою учетную запись Azure из среды PowerShell с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) .

### <a name="get-the-web-service-definition-object"></a>Получение объекта определения веб-службы

Затем получите объект определения веб-службы, вызвав командлет [Get-азмлвебсервице](/powershell/module/az.machinelearning/get-azmlwebservice) .

```azurepowershell
$wsd = Get-AzMlWebService -Name 'RetrainSamplePre.2016.8.17.0.3.51.237' -ResourceGroupName 'Default-MachineLearning-SouthCentralUS'
```

Чтобы определить имя группы ресурсов существующей веб-службы, выполните командлет Get-AzMlWebService без параметров, чтобы отобразить веб-службы в подписке. Найдите необходимую веб-службу и посмотрите ее идентификатор. Имя группы ресурсов — это четвертый элемент в идентификаторе, который следует сразу за элементом *resourceGroups* . В следующем примере имя группы ресурсов — Default-MachineLearning-SouthCentralUS.

```azurepowershell
Properties : Microsoft.Azure.Management.MachineLearning.WebServices.Models.WebServicePropertiesForGraph
Id : /subscriptions/<subscription ID>/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/RetrainSamplePre.2016.8.17.0.3.51.237
Name : RetrainSamplePre.2016.8.17.0.3.51.237
Location : South Central US
Type : Microsoft.MachineLearning/webServices
Tags : {}
```

Кроме того, чтобы определить имя группы ресурсов существующей веб-службы, можно войти на портал веб-служб Машинного обучения Azure. Затем выбрать веб-службу. Имя группы ресурсов — это пятый элемент в URL-адресе веб-службы, который следует сразу за элементом *resourceGroups* . В следующем примере имя группы ресурсов — Default-MachineLearning-SouthCentralUS.

`https://services.azureml.net/subscriptions/<subscription ID>/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/RetrainSamplePre.2016.8.17.0.3.51.237`

### <a name="export-the-web-service-definition-object-as-json"></a>Экспорт объекта определения веб-службы в формате JSON

Чтобы изменить определение обученной модели для использования новой обученной модели, сначала необходимо использовать командлет [Export-азмлвебсервице](/powershell/module/az.machinelearning/export-azmlwebservice) , чтобы экспортировать его в файл формата JSON.

```azurepowershell
Export-AzMlWebService -WebService $wsd -OutputFile "C:\temp\mlservice_export.json"
```

### <a name="update-the-reference-to-the-ilearner-blob"></a>Обновление ссылки на большой двоичный объект ilearner

В ресурсах-контейнерах найдите элемент [trained model], обновите значение *uri* в узле *locationInfo*, заменив его универсальным кодом ресурса (URI) BLOB-объекта ilearner. URI формируется в результате объединения параметров *BaseLocation* и *RelativeLocation* из выходных данных вызова переобучения BES.

```json
"asset3": {
    "name": "Retrain Sample [trained model]",
    "type": "Resource",
    "locationInfo": {
        "uri": "https://mltestaccount.blob.core.windows.net/azuremlassetscontainer/baca7bca650f46218633552c0bcbba0e.ilearner"
    },
    "outputPorts": {
        "Results dataset": {
            "type": "Dataset"
        }
    }
},
```

### <a name="import-the-json-into-a-web-service-definition-object"></a>Импорт JSON-файла в объект определения веб-службы

Используйте командлет [Import-азмлвебсервице](/powershell/module/az.machinelearning/import-azmlwebservice) , чтобы преобразовать измененный файл JSON обратно в объект определения веб-службы, который можно использовать для обновления эксперимента прогнозного.

```azurepowershell
$wsd = Import-AzMlWebService -InputFile "C:\temp\mlservice_export.json"
```

### <a name="update-the-web-service"></a>Обновление веб-службы

Наконец, используйте командлет [Update-азмлвебсервице](/powershell/module/az.machinelearning/update-azmlwebservice) для обновления прогнозного эксперимента.

```azurepowershell
Update-AzMlWebService -Name 'RetrainSamplePre.2016.8.17.0.3.51.237' -ResourceGroupName 'Default-MachineLearning-SouthCentralUS'
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как управлять веб-службами или отслеживать несколько экспериментов, см. в следующих статьях:

* [Обзор портала веб-служб](manage-new-webservice.md)
* [Управление итерациями эксперимента](manage-experiment-iterations.md)