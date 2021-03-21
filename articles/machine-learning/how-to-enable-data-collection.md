---
title: Сбор данных рабочих моделей
titleSuffix: Azure Machine Learning
description: Узнайте, как получать данные из Машинное обучение Azureной модели, развернутой в кластере Azure Kubernetes Service (AKS).
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.author: copeters
author: lostmygithubaccount
ms.date: 07/14/2020
ms.topic: conceptual
ms.custom: how-to, data4ml
ms.openlocfilehash: f42a2f9e606eaa0475f2e35c479e99545ecf8193
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102521906"
---
# <a name="collect-data-from-models-in-production"></a>Получение данных из моделей в рабочей среде

В этой статье показано, как получить данные из Машинное обучение Azureной модели, развернутой в кластере Azure Kubernetes Service (AKS). Собранные данные затем хранятся в хранилище BLOB-объектов Azure.

После включения сбора собранные данные помогут вам:

* [Отслеживание смещения данных](how-to-monitor-datasets.md) для собранных рабочих данных.

* Анализ собранных данных с помощью [Power BI](#powerbi) или [Azure Databricks](#databricks)

* Принятие более эффективных решений о том, когда следует переучить или оптимизировать модель.

* Переобучение модели с помощью собранных данных.

## <a name="what-is-collected-and-where-it-goes"></a>Собираемые и расположенные

Вы можете собирать следующие данные:

* Входные данные модели из веб-служб, развернутых в кластере AKS. Голосовые аудио, изображения и видео *не* собираются.
  
* Прогнозирование моделей с использованием входных данных рабочей среды.

>[!NOTE]
> Предварительная агрегация и предварительные вычисления над этими данными в настоящее время не являются частью службы сбора.

Выходные данные сохраняются в хранилище BLOB-объектов. Так как данные добавляются в хранилище BLOB-объектов, можно выбрать предпочтительный инструмент для выполнения анализа.

Для пути к выходным данным в BLOB-объекте используется следующий синтаксис:

```
/modeldata/<subscriptionid>/<resourcegroup>/<workspace>/<webservice>/<model>/<version>/<designation>/<year>/<month>/<day>/data.csv
# example: /modeldata/1a2b3c4d-5e6f-7g8h-9i10-j11k12l13m14/myresourcegrp/myWorkspace/aks-w-collv9/best_model/10/inputs/2018/12/31/data.csv
```

>[!NOTE]
> В версиях пакета SDK Машинное обучение Azure для Python, предшествующего версии 0.1.0 A16, `designation` аргумент имеет имя `identifier` . Если вы разработали код с более ранней версией, необходимо соответствующим образом обновить его.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://aka.ms/AMLFree), прежде чем начинать работу.

- Необходимо установить Машинное обучение Azure рабочую область, локальный каталог, содержащий скрипты, и пакет SDK для Машинное обучение Azure для Python. Сведения об их установке см. в статье [Настройка среды разработки](how-to-configure-environment.md).

- Для развертывания в AKS требуется обученная модель машинного обучения. Если у вас нет модели, см. Руководство [обучение модели классификации изображений](tutorial-train-models-with-aml.md) .

- Необходим кластер AKS. Сведения о том, как создать и развернуть в нем, см. [в разделе Развертывание и размещение](how-to-deploy-and-where.md).

- [Настройте среду](how-to-configure-environment.md) и установите [пакет SDK для машинное обучение Azure мониторинга](/python/api/overview/azure/ml/install).

## <a name="enable-data-collection"></a>Включение сбора данных

[Сбор данных](/python/api/azureml-monitoring/azureml.monitoring.modeldatacollector.modeldatacollector) можно включить независимо от модели, развертываемой с помощью машинное обучение Azure или других средств.

Чтобы включить сбор данных, необходимо выполнить следующие действия.

1. Откройте файл оценки.

1. Добавьте следующий код в начало файла.

   ```python 
   from azureml.monitoring import ModelDataCollector
   ```

1. Объявите переменные коллекции данных в функции `init`:

    ```python
    global inputs_dc, prediction_dc
    inputs_dc = ModelDataCollector("best_model", designation="inputs", feature_names=["feat1", "feat2", "feat3", "feat4", "feat5", "feat6"])
    prediction_dc = ModelDataCollector("best_model", designation="predictions", feature_names=["prediction1", "prediction2"])
    ```

    *CorrelationId* является необязательным параметром. Вам не нужно использовать его, если модель не требует ее. Использование *correlationId* позволяет легко сопоставляться с другими данными, такими как *лоаннумбер* или *CustomerID*.
    
    Параметр *identifier* позже используется для создания структуры папок в большом двоичном объекте. Его можно использовать для различения обработанных данных от обработанных.

1. Добавьте в функцию `run(input_df)` следующие строки кода.

    ```python
    data = np.array(data)
    result = model.predict(data)
    inputs_dc.collect(data) #this call is saving our input data into Azure Blob
    prediction_dc.collect(result) #this call is saving our input data into Azure Blob
    ```

1. При развертывании службы в AKS сбор данных *не* устанавливается автоматически в **значение true** . Обновите файл конфигурации, как показано в следующем примере:

    ```python
    aks_config = AksWebservice.deploy_configuration(collect_model_data=True)
    ```

    Вы также можете включить Application Insights для наблюдения за службами, изменив эту конфигурацию:

    ```python
    aks_config = AksWebservice.deploy_configuration(collect_model_data=True, enable_app_insights=True)
    ```

1. Сведения о создании нового образа и развертывании модели машинного обучения см. в разделе [развертывание и использование](how-to-deploy-and-where.md).

1. Добавьте пакет PIP "Azure-Monitoring" в conda-Dependencies среды веб-службы:
  ```Python
    env = Environment('webserviceenv')
    env.python.conda_dependencies = CondaDependencies.create(conda_packages=['numpy'],pip_packages=['azureml-defaults','azureml-monitoring','inference-schema[numpy-support]'])
  ```


## <a name="disable-data-collection"></a>Отключение сбора данных

Сбор данных можно отключить в любое время. Используйте код Python для отключения сбора данных.

  ```python 
  ## replace <service_name> with the name of the web service
  <service_name>.update(collect_model_data=False)
  ```

## <a name="validate-and-analyze-your-data"></a>Проверка и анализ данных

Вы можете выбрать инструмент для анализа данных, собранных в хранилище BLOB-объектов.

### <a name="quickly-access-your-blob-data"></a>Быстрый доступ к данным большого двоичного объекта

1. Войдите на [портал Azure](https://portal.azure.com).

1. Перейдите в рабочую область.

1. Выберите **Хранилище**.

    [![Выберите вариант хранения](./media/how-to-enable-data-collection/StorageLocation.png)](././media/how-to-enable-data-collection/StorageLocation.png#lightbox)

1. Выполните путь к выходным данным большого двоичного объекта, используя следующий синтаксис:

   ```
   /modeldata/<subscriptionid>/<resourcegroup>/<workspace>/<webservice>/<model>/<version>/<designation>/<year>/<month>/<day>/data.csv
   # example: /modeldata/1a2b3c4d-5e6f-7g8h-9i10-j11k12l13m14/myresourcegrp/myWorkspace/aks-w-collv9/best_model/10/inputs/2018/12/31/data.csv
   ```

### <a name="analyze-model-data-using-power-bi"></a><a id="powerbi"></a> Анализ данных модели с помощью Power BI

1. Скачайте и откройте [Power BI Desktop](https://www.powerbi.com).

1. Выберите **получить данные** и выберите [**хранилище BLOB-объектов Azure**](/power-bi/desktop-data-sources).

    [![Настройка Power BI большого двоичного объекта](./media/how-to-enable-data-collection/PBIBlob.png)](././media/how-to-enable-data-collection/PBIBlob.png#lightbox)

1. Добавьте имя учетной записи хранения и введите ключ к хранилищу данных. Эти сведения можно найти, выбрав **Параметры**  >  **ключи доступа** в большом двоичном объекте.

1. Выберите контейнер **данных модели** и нажмите кнопку **изменить**.

    [![Навигатор Power BI](./media/how-to-enable-data-collection/pbiNavigator.png)](././media/how-to-enable-data-collection/pbiNavigator.png#lightbox)

1. В редакторе запросов щелкните в столбце **имя** и добавьте учетную запись хранения.

1. Введите путь к модели в фильтре. Если вы хотите просматривать файлы только из определенного года или месяца, просто разверните путь фильтра. Например, чтобы просмотреть только данные в марте, используйте следующий путь фильтра:

   /моделдата/ \<subscriptionid> / \<resourcegroupname> / \<workspacename> / \<webservicename> / \<modelname> / \<modelversion> / \<designation> / \<year> /3

1. Отфильтруйте данные, относящиеся к вам, на основе значений **имен** . Если вы сохранили прогнозы и входные данные, необходимо создать запрос для каждого из них.

1. Щелкните двойные стрелки вниз рядом с заголовком столбца **содержимого** , чтобы объединить файлы.

    [![Power BI содержимое](./media/how-to-enable-data-collection/pbiContent.png)](././media/how-to-enable-data-collection/pbiContent.png#lightbox)

1. Щелкните **ОК**. Предварительная загрузка данных.

    [![Power BI объединение файлов](./media/how-to-enable-data-collection/pbiCombine.png)](././media/how-to-enable-data-collection/pbiCombine.png#lightbox)

1. Выберите **Закрыть и применить**.

1. Если были добавлены входные данные и прогнозы, таблицы автоматически упорядочиваются по значениям **RequestId** .

1. Приступите к созданию пользовательских отчетов о данных модели.

### <a name="analyze-model-data-using-azure-databricks"></a><a id="databricks"></a> Анализ данных модели с помощью Azure Databricks

1. Создайте [рабочую область Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal).

1. Перейдите к рабочей области Databricks.

1. В рабочей области "сведения о модулях **данных" выберите Отправить данные**.

    [![Выбор параметра отправки данных для кирпичов](./media/how-to-enable-data-collection/dbupload.png)](././media/how-to-enable-data-collection/dbupload.png#lightbox)

1. Выберите **создать новую таблицу** и выберите **другие источники данных**  >  **хранилище BLOB-объектов Azure**  >  **создать таблицу в записной книжке**.

    [![Создание таблицы кирпичей данных](./media/how-to-enable-data-collection/dbtable.PNG)](././media/how-to-enable-data-collection/dbtable.PNG#lightbox)

1. Обновите расположение данных. Пример:

    ```
    file_location = "wasbs://mycontainer@storageaccountname.blob.core.windows.net/modeldata/1a2b3c4d-5e6f-7g8h-9i10-j11k12l13m14/myresourcegrp/myWorkspace/aks-w-collv9/best_model/10/inputs/2018/*/*/data.csv" 
    file_type = "csv"
    ```

    [![Установка кирпичей](./media/how-to-enable-data-collection/dbsetup.png)](././media/how-to-enable-data-collection/dbsetup.png#lightbox)

1. Выполните действия, описанные в шаблоне, чтобы просмотреть и проанализировать данные.

## <a name="next-steps"></a>Дальнейшие действия

[Обнаружение смещения данных](how-to-monitor-datasets.md) для собранных данных.