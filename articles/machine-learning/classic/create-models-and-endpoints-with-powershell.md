---
title: 'ML Studio (классическая модель): создание нескольких конечных точек & модели в Azure'
description: Используйте PowerShell для создания нескольких моделей машинного обучения и конечных точек веб-службы с одним алгоритмом, но разными наборами данных для обучения.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 04/04/2017
ms.openlocfilehash: 35b5fe4556f1d557d3fc0420e9069f2fb510eec4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100520516"
---
# <a name="create-multiple-web-service-endpoints-from-one-experiment-with-ml-studio-classic-and-powershell"></a>Создание нескольких конечных точек веб-службы из одного эксперимента с помощью студии машинного обучения (классическая модель) и PowerShell

**ПРИМЕНИМО К:**  ![Применимо к.](../../../includes/media/aml-applies-to-skus/yes.png)Студия машинного обучения (классическая)   ![Неприменимо к.](../../../includes/media/aml-applies-to-skus/no.png)[Машинное обучение Azure](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

При машинном обучении часто возникает следующая задача: требуется создать множество моделей с одинаковым рабочим процессом обучения и одинаковым алгоритмом. Но для их обучения нужно использовать разные входные наборы данных. В этой статье показано, как сделать это в масштабе в Машинное обучение Azure Studio (классическая модель), используя только один эксперимент.

Предположим, что вы являетесь владельцем международного бизнеса по предоставлению франшиз на аренду велосипедов. Вы хотите создать модель регрессии, чтобы прогнозировать спрос, основываясь на исторических данных. У вас есть 1000 расположений в разных частях мира, в которых доступна аренда, и вы собрали набор данных по каждому из них. В этот набор входят такие сведения, как даты, время, данные погоды и трафика, характерные для конкретного расположения.

Вы можете обучить модель один раз, используя совокупную версию всех наборов данных для всех расположений. Но у каждого расположения есть уникальные характеристики среды. Поэтому лучше использовать отдельные наборы данных, чтобы обучить модель регрессии для каждого расположения. В этом случае каждая обученная модель будет учитывать характеристики размера, объема, географического положения, населения, удобства среды для велосипедистов и многое другое.

Это может быть лучшим подходом, но вы не хотите создавать эксперименты обучения 1 000 в Машинное обучение Azure Studio (классическая модель) с каждым из них, представляющим уникальное расположение. Такая задача не только трудоемка, но и малоэффективна, поскольку каждый эксперимент имеет одинаковые компоненты, за исключением набора данных для обучения.

К счастью, это можно сделать с помощью [API-интерфейса переобучения машинное обучение Azure Studio (классическая модель)](./retrain-machine-learning-model.md) и автоматизации задачи с помощью среды [PowerShell для машинное обучение Azure Studio (классическая модель)](powershell-module.md).

> [!NOTE]
> Чтобы ускорить выполнение, мы создадим пример для 10 расположений вместо 1000. Однако в случае с 1000 расположений применяются все те же принципы и процедуры. Но если вы решите обучить модель по 1000 наборов данных, постарайтесь применить параллельное выполнение скриптов PowerShell. Этот вопрос выходит за рамки данной статьи, но примеры многопоточности PowerShell можно найти в Интернете.  
> 
> 

## <a name="set-up-the-training-experiment"></a>настройка обучающего эксперимента
[Обучающий эксперимент](https://gallery.azure.ai/Experiment/Bike-Rental-Training-Experiment-1) для этого примера представлен в [Cortana Intelligence Gallery](https://gallery.azure.ai). Откройте этот эксперимент в рабочей области [машинное обучение Azure Studio (классическая модель)](https://studio.azureml.net) .

> [!NOTE]
> Для работы с этим примером рекомендуется использовать стандартную рабочую область, а не бесплатную. Вы создадите по одной конечной точке для каждого заказчика (всего 10 точек), поэтому вам потребуется стандартная рабочая область. Бесплатная рабочая область поддерживает только 3 конечные точки.
> 
> 

Эксперимент использует модуль **Импорт данных** , чтобы импортировать набор данных для обучения *customer001.csv* из учетной записи хранения Azure. Предположим, что вы собрали наборы данных для обучения по всем расположениям аренды велосипедов и сохранили их в одном хранилище BLOB-объектов с именами от *rentalloc001.csv* до *rentalloc10.csv*.

![Модуль чтения импортирует данные из большого двоичного объекта Azure](./media/create-models-and-endpoints-with-powershell/reader-module.png)

Обратите внимание, что в модуль **Обучение модели** добавлен модуль **Web Service Output** (Выходные данные веб-службы).
Если эксперимент развернут в виде веб-службы, связанная с выходом конечная точка возвращает обученную модель в формате ILEARNER-файла.

Кроме того, вам потребуется определить параметр веб-службы, который содержит URL-адрес для использования в модуле **Импорт данных**. В этом параметре вы сможете указать разные наборы данных для обучения моделей по каждому расположению.
Это можно реализовать и другими способами. Для получения данных из базы данных в базе данных SQL Azure можно использовать запрос SQL с параметром веб-службы. Также можно применить модуль **входных данных для веб-службы**, чтобы передать в веб-службу набор данных.

![Модуль обученной модели выводит данные в модуль вывода веб-службы](./media/create-models-and-endpoints-with-powershell/web-service-output.png)

Теперь давайте запустим этот обучающий эксперимент, используя значение по умолчанию *rental001.csv* в качестве набора данных для обучения. Просмотрев выходные данные модуля **Оценка** (для просмотра щелкните выходные данные и выберите **Визуализировать**), вы заметите неплохую производительность *AUC* = 0,91. На этом этапе вы уже можете развернуть веб-службу из нашего обучающего эксперимента.

## <a name="deploy-the-training-and-scoring-web-services"></a>Развертывание обучающей и оценивающей веб-служб
Чтобы развернуть обучающую веб-службу, нажмите кнопку **Set Up Web Service** (Настройка веб-службы) ниже холста эксперимента и выберите **Deploy Web Service** (Развертывание веб-службы). Присвойте этой веб-службе имя Bike Rental Training (Обучение для аренды велосипедов).

Теперь разверните оценивающую веб-службу.
Для этого нажмите под холстом кнопку **Set Up Web Service** (Настройка веб-службы) и выберите **Predictive Web Service** (Прогнозная веб-служба). При этом создается оценивающий эксперимент.
Чтобы он работал как веб-служба, осталось внести лишь небольшие изменения. Удалите из входных данных столбец меток "cnt" и сократите выходные данных, оставив только идентификатор экземпляра и прогнозируемое значение.

Чтобы не выполнять это самостоятельно, откройте уже подготовленный [прогнозный эксперимент](https://gallery.azure.ai/Experiment/Bike-Rental-Predicative-Experiment-1) из коллекции.

Чтобы развернуть веб-службы, запустите прогнозный эксперимент, а затем щелкните **Deploy Web Service** (Развертывание веб-службы) внизу холста. Присвойте оценивающей веб-службе имя Bike Rental Scoring (Оценка для аренды велосипедов).

## <a name="create-10-identical-web-service-endpoints-with-powershell"></a>Создание 10 идентичных конечных точек веб-службы с помощью PowerShell
Эта веб-служба предоставляется с конечной точкой по умолчанию. Но вам не нужна конечная точка по умолчанию, так как ее нельзя обновить. Вам следует создать 10 дополнительных конечных точек — по одной для каждого расположения. Для этого мы воспользуемся PowerShell.

Сначала настройте среду PowerShell:

```powershell
Import-Module .\AzureMLPS.dll
# Assume the default configuration file exists and is properly set to point to the valid Workspace.
$scoringSvc = Get-AmlWebService | where Name -eq 'Bike Rental Scoring'
$trainingSvc = Get-AmlWebService | where Name -eq 'Bike Rental Training'
```

Затем выполним следующую команду PowerShell:

```powershell
# Create 10 endpoints on the scoring web service.
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    Write-Host ('adding endpoint ' + $endpointName + '...')
    Add-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -Description $endpointName     
}
```

Итак, вы создали 10 конечных точек, и все они содержат одну и ту же модель, обученную по набору данных *customer001.csv*. Их можно просмотреть на портале Azure.

![Просмотр списка обученных моделей на портале](./media/create-models-and-endpoints-with-powershell/created-endpoints.png)

## <a name="update-the-endpoints-to-use-separate-training-datasets-using-powershell"></a>Обновление конечных точек для использования отдельных наборов данных для обучения с помощью PowerShell
Следующим шагом является обновление конечных точек с использованием моделей, обученных по отдельным данным каждого из клиентов. Но сначала вам следует создать эти модели из веб-службы **Bike Rental Training**. Вернемся к веб-службе **Bike Rental Training** . Вызовите ее конечную точку BES 10 раз с 10 разными наборами данных для обучения, чтобы создать 10 разных моделей. В этом вам поможет командлет PowerShell **InovkeAmlWebServiceBESEndpoint**.

Также для `$configContent` нужно предоставить учетные данные для доступа к учетной записи хранения BLOB-объектов. Они передаются в полях `AccountName`, `AccountKey` и `RelativeLocation`. В поле `AccountName` можно указать любое из имен учетных записей в том виде, в котором они представлены на **портале Azure** (на вкладке *Хранилище*). Выберите учетную запись хранения. Чтобы найти ее `AccountKey`, нажмите кнопку **Управление ключами доступа** и скопируйте значение поля *Первичный ключ доступа*. `RelativeLocation` — это путь к хранилищу, в котором будет храниться новая модель. Например, путь `hai/retrain/bike_rental/` в скрипте ниже указывает на контейнер с именем `hai`, а `/retrain/bike_rental/` — это вложенные папки. Сейчас невозможно создать вложенные папки с помощью пользовательского интерфейса портала. Но это можно сделать с помощью [нескольких обозревателей службы хранилища Azure](../../storage/common/storage-explorers.md). Рекомендуем создать для хранения обученных моделей (ILEARNER-файлы) контейнер в хранилище. Для этого в нижней части страницы хранилища нажмите кнопку **Добавить** и введите имя `retrain` для контейнера. Таким образом, в следующем скрипте нужно изменить лишь `AccountName`, `AccountKey` и `RelativeLocation` (:`"retrain/model' + $seq + '.ilearner"`).

```powershell
# Invoke the retraining API 10 times
# This is the default (and the only) endpoint on the training web service
$trainingSvcEp = (Get-AmlWebServiceEndpoint -WebServiceId $trainingSvc.Id)[0];
$submitJobRequestUrl = $trainingSvcEp.ApiLocation + '/jobs?api-version=2.0';
$apiKey = $trainingSvcEp.PrimaryKey;
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $inputFileName = 'https://bostonmtc.blob.core.windows.net/hai/retrain/bike_rental/BikeRental' + $seq + '.csv';
    $configContent = '{ "GlobalParameters": { "URI": "' + $inputFileName + '" }, "Outputs": { "output1": { "ConnectionString": "DefaultEndpointsProtocol=https;AccountName=<myaccount>;AccountKey=<mykey>", "RelativeLocation": "hai/retrain/bike_rental/model' + $seq + '.ilearner" } } }';
    Write-Host ('training regression model on ' + $inputFileName + ' for rental location ' + $seq + '...');
    Invoke-AmlWebServiceBESEndpoint -JobConfigString $configContent -SubmitJobRequestUrl $submitJobRequestUrl -ApiKey $apiKey
}
```

> [!NOTE]
> Конечная точка BES является единственным поддерживаемым режимом для этой операции. RRS использовать для создания обученных моделей нельзя.
> 
> 

Как показано выше, вместо создания 10 разных JSON-файлов, требующихся для настройки заданий BES, можно динамически создать строку настройки. Эта строка передается в параметре *jobConfigString* для командлета **InvokeAmlWebServceBESEndpoint**. Ее даже не нужно сохранять на диске.

Если все пройдет правильно, через некоторое время в вашей учетной записи хранения Azure появятся 10 ILEARNER-файлов: от *model001.ilearner* до *model010.ilearner*. Теперь все готово к передаче этих моделей в 10 конечных точек оценивающей веб-службы с помощью командлета PowerShell **Patch-AmlWebServiceEndpoint**. Не забывайте, что вы можете изменить только конечные точки, созданные ранее программными средствами, но не конечную точку по умолчанию.

```powershell
# Patch the 10 endpoints with respective .ilearner models
$baseLoc = 'http://bostonmtc.blob.core.windows.net/'
$sasToken = '<my_blob_sas_token>'
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    $relativeLoc = 'hai/retrain/bike_rental/model' + $seq + '.ilearner';
    Write-Host ('Patching endpoint ' + $endpointName + '...');
    Patch-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -ResourceName 'Bike Rental [trained model]' -BaseLocation $baseLoc -RelativeLocation $relativeLoc -SasBlobToken $sasToken
}
```

Процесс должен выполняться довольно быстро. Когда он завершится, вы получите 10 конечных точек прогнозной веб-службы. Каждая из них будет содержать модель, обученную по уникальному набору данных для определенного расположения аренды. Все эти модели создаются в одном эксперименте обучения. Чтобы проверить работу системы, обратитесь к этим конечным точкам с помощью командлета **InvokeAmlWebServiceRRSEndpoint**, предоставив одинаковые входные данные. Вы должны получить разные результаты прогнозирования, поскольку модели обучены по разным наборам данных.

## <a name="full-powershell-script"></a>Полный сценарий PowerShell
Ниже приведен полный исходный код.

```powershell
Import-Module .\AzureMLPS.dll
# Assume the default configuration file exists and properly set to point to the valid workspace.
$scoringSvc = Get-AmlWebService | where Name -eq 'Bike Rental Scoring'
$trainingSvc = Get-AmlWebService | where Name -eq 'Bike Rental Training'

# Create 10 endpoints on the scoring web service
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    Write-Host ('adding endpoint ' + $endpontName + '...')
    Add-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -Description $endpointName     
}

# Invoke the retraining API 10 times to produce 10 regression models in .ilearner format
$trainingSvcEp = (Get-AmlWebServiceEndpoint -WebServiceId $trainingSvc.Id)[0];
$submitJobRequestUrl = $trainingSvcEp.ApiLocation + '/jobs?api-version=2.0';
$apiKey = $trainingSvcEp.PrimaryKey;
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $inputFileName = 'https://bostonmtc.blob.core.windows.net/hai/retrain/bike_rental/BikeRental' + $seq + '.csv';
    $configContent = '{ "GlobalParameters": { "URI": "' + $inputFileName + '" }, "Outputs": { "output1": { "ConnectionString": "DefaultEndpointsProtocol=https;AccountName=<myaccount>;AccountKey=<mykey>", "RelativeLocation": "hai/retrain/bike_rental/model' + $seq + '.ilearner" } } }';
    Write-Host ('training regression model on ' + $inputFileName + ' for rental location ' + $seq + '...');
    Invoke-AmlWebServiceBESEndpoint -JobConfigString $configContent -SubmitJobRequestUrl $submitJobRequestUrl -ApiKey $apiKey
}

# Patch the 10 endpoints with respective .ilearner models
$baseLoc = 'http://bostonmtc.blob.core.windows.net/'
$sasToken = '?test'
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    $relativeLoc = 'hai/retrain/bike_rental/model' + $seq + '.ilearner';
    Write-Host ('Patching endpoint ' + $endpointName + '...');
    Patch-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -ResourceName 'Bike Rental [trained model]' -BaseLocation $baseLoc -RelativeLocation $relativeLoc -SasBlobToken $sasToken
}
```