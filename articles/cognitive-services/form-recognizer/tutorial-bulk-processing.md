---
title: Руководство. Пакетное извлечение данных форм с помощью Фабрики данных Azure — Распознаватель документов
titleSuffix: Azure Cognitive Services
description: Настройте действия Фабрики данных Azure, чтобы активировать обучение и выполнение моделей Распознавателя документов, а также оцифровку большого пакета документов.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: tutorial
ms.date: 01/04/2021
ms.author: pafarley
ms.openlocfilehash: 5b220652009f54482c757f01232517569596c562
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102423902"
---
# <a name="tutorial-extract-form-data-in-bulk-by-using-azure-data-factory"></a>Руководство. Пакетное извлечение данных форм с помощью Фабрики данных Azure

Из этого руководства вы узнаете, как с помощью служб Azure выполнить прием большого пакета форм в файлы мультимедиа. В этом руководстве показано, как автоматизировать прием данных документов из озера данных Azure в базу данных SQL Azure. Вы сможете быстро обучать модели и обрабатывать новые документы несколькими щелчками мыши.

## <a name="business-need"></a>Бизнес-потребности

Большинство организаций осознают ценность своих данных, существующих в разных форматах (PDF, изображения, видео). Компаниям нужны рекомендации и сведения о наиболее экономичных способах оцифровки данных.

Также нашим клиентам часто нужно использовать разные формы, заполняемые клиентами и партнерами. В отличие от других [кратких руководств](./quickstarts/client-library.md), в этом руководстве показано, как автоматически обучить модель с помощью форм разных новых типов, используя подход на основе метаданных. Если у вас нет модели для форм определенного типа, система создаст ее автоматически и предоставит вам идентификатор модели. 

Извлекая данные из форм и объединяя их с существующими операционными системами и хранилищами данных, компании могут получать ценные сведения и предоставлять преимущества своим клиентам и бизнес-пользователям.

Распознаватель документов Azure помогает организациям использовать данные, автоматизировать процессы (платежи по счетам, обработку налогов и т. д.), экономить деньги и время и повышать точность данных.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка Azure Data Lake для хранения форм;
> * создание таблицы параметризации с помощью базы данных Azure;
> * хранение конфиденциальных учетных данных с использованием Azure Key Vault;
> * обучение модели Распознавателя документов в записной книжке Azure Databricks;
> * извлечение данных из формы с помощью записной книжки Databricks;
> * автоматизация обучения и извлечения форм с помощью Фабрики данных Azure.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. [Создайте бесплатно](https://azure.microsoft.com/free/cognitive-services/).
* Получив подписку Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="создание ресурса Распознавателя документов"  target="_blank">создайте ресурс Распознавателя документов</a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания ресурса нажмите **Перейти к ресурсу**.
    * Для подключения приложения к API "Распознаватель документов" потребуется ключ и конечная точка созданного ресурса. Позже вы добавите значения ключа и конечной точки в свой код.
    * Пока вы можете использовать ценовую категорию "Бесплатный" (F0) для работы со службой. Позже вы сможете повысить этот уровень, чтобы переключиться на рабочую среду.
* Минимум пять документов одного типа. В идеале этот рабочий процесс предназначен для больших наборов документов. Советы и рекомендации по объединению данных в набор данных для обучения см. в статье [Создание набора данных для обучения для пользовательской модели](./build-training-data-set.md). При работе с этим руководством можно использовать файлы в папке Train из [тестового набора данных](https://go.microsoft.com/fwlink/?linkid=2128080).


## <a name="project-architecture"></a>Архитектура проекта 

Этот проект настраивает набор конвейеров Фабрики данных Azure для активации записных книжек Python, которые обучают модели, анализируют документы в учетной записи хранения Azure Data Lake и извлекают из них данные.

В качестве входных данных REST API Распознавателя документов требует определенные параметры. Из соображений безопасности некоторые из этих параметров будут храниться в хранилище ключей Azure. Другие параметры, которые не относятся к конфиденциальным данным, например, имя папки для большого двоичного объекта, будут храниться в таблице параметризации в базе данных SQL Azure.

Для каждого типа анализируемых форм специалисты по обработке и анализу данных создают и заполняют строку в таблице параметров. Затем с помощью Фабрики данных Azure они просматривают список обнаруженных типов форм и передают соответствующие параметры в записную книжку Databricks для обучения или переобучения моделей Распознавателя документов. Кроме того, здесь можно использовать одну из функций Azure.

Записная книжка Azure Databricks применяет обученные модели для извлечения данных из форм. Затем она экспортирует эти данные в базу данных SQL Azure.

:::image type="content" source="./media/tutorial-bulk-processing/architecture.png" alt-text="Диаграмма: архитектура проекта.":::


## <a name="set-up-azure-data-lake"></a>Настройка Azure Data Lake

Пакет форм для обработки можно разместить в локальной среде или на сервере FTP или SFTP. При работе с этим руководством используются формы в учетной записи хранения Azure Data Lake Storage 2-го поколения. Вы можете передать файлы с помощью Фабрики данных Azure, Обозревателя службы хранилища Azure или AzCopy. Наборы данных для обучения и оценки могут находиться в разных контейнерах, но наборы данных для обучения для форм всех типов должны размещаться в одном контейнере. Они могут находиться в разных папках.

Для создания озера данных выполните инструкции, приведенные в статье [Создание учетной записи хранения для использования с Azure Data Lake Storage 2-го поколения](../../storage/blobs/create-data-lake-storage-account.md).

## <a name="create-a-parameterization-table"></a>Создание таблицы параметризации

Далее мы создадим таблицу метаданных в базе данных SQL Azure. Эта таблица будет содержать неконфиденциальные данные, необходимые REST API "Распознаватель документов". Встретив форму нового типа в наборе данных, мы добавим новую запись в эту таблицу и активируем конвейеры обучения и оценки. Эти конвейеры будут реализованы позже.

В этой таблице данные хранятся в следующих полях:

* **form_description.** Не является обязательным в рамках обучения. Это поле содержит описание типа форм, для которых обучается модель (например, "формы клиента А", "формы гостиницы Б").
* **training_container_name.** Имя контейнера учетной записи хранения, в котором мы сохранили набор данных для обучения. Это может быть тот же контейнер, что и **scoring_container_name**.
* **training_blob_root_folder.** папка в учетной записи хранения, где будут храниться файлы для обучения модели.
* **scoring_container_name.** Это имя контейнера учетной записи хранения, где сохранены файлы, из которых мы будем извлекать пары "ключ — значение". Это может быть тот же контейнер, что и **training_container_name**.
* **scoring_input_blob_folder.** папка в учетной записи хранения, в которой будут храниться файлы для извлечения данных.
* **model_id.** Идентификатор модели, которую нужно обучить повторно. Для первого запуска здесь нужно указать значение -1, чтобы создать в учебной записной книжке новую настраиваемую модель для обучения. Записная книжка вернет идентификатор новой модели в экземпляр Фабрики данных Azure. Мы будем обновлять это значение в базе данных SQL Azure с помощью действия хранимой процедуры.

  Каждый раз, когда потребуется принять форму нового типа, необходимо вручную присвоить идентификатору модели значение -1, прежде чем обучать ее.

* **file_type.** Поддерживаемые типы форм: `application/pdf`, `image/jpeg`, `image/png` и `image/tif`.

  При наличии форм в файлах другого типа нужно изменить это значение и **model_id** при обучении формы нового типа.
* **form_batch_group_id.** Со временем у вас может накопиться несколько типов форм, обучаемых с использованием одной и той же модели. Параметр **form_batch_group_id** позволяет указать все типы форм, которые обучаются с помощью определенной модели.

### <a name="create-the-table"></a>Создание таблицы


[Создайте базу данных SQL Azure](https://ms.portal.azure.com/#create/Microsoft.SQLDatabase), а затем выполните этот скрипт SQL в [редакторе запросов](../../azure-sql/database/connect-query-portal.md), чтобы создать необходимую таблицу.


```sql
CREATE TABLE dbo.ParamFormRecogniser(
    form_description varchar(50) NULL,
  training_container_name varchar(50) NOT NULL,
    training_blob_root_folder varchar(50) NULL,
    scoring_container_name varchar(50) NOT NULL,
    scoring_input_blob_folder varchar(50) NOT NULL,
    scoring_output_blob_folder varchar(50) NOT NULL,
    model_id varchar(50) NULL,
    file_type varchar(50) NULL
) ON PRIMARY
GO
```

Запустите этот скрипт, чтобы создать процедуру, которая автоматически обновляет значение **model_id** после обучения.

```SQL
CREATE PROCEDURE [dbo].[update_model_id] ( @form_batch_group_id  varchar(50),@model_id varchar(50))
AS
BEGIN 
    UPDATE [dbo].[ParamFormRecogniser]   
        SET [model_id] = @model_id  
    WHERE form_batch_group_id =@form_batch_group_id
END
```

## <a name="use-azure-key-vault-to-store-sensitive-credentials"></a>Хранение конфиденциальных учетных данных с использованием Azure Key Vault

Из соображений безопасности не следует хранить определенные конфиденциальные данные в таблице параметризации в базе данных SQL Azure. Конфиденциальные параметры будут храниться в виде секретов Azure Key Vault.

### <a name="create-an-azure-key-vault"></a>Создание хранилища ключей Azure

[Создайте ресурс Key Vault](https://ms.portal.azure.com/#create/Microsoft.KeyVault). Перейдите к созданному ресурсу Key Vault и в разделе **Параметр** выберите элемент **Секреты**, чтобы добавить параметры.

Откроется новое окно. Щелкните **Создать или импортировать**. Введите имя параметра и его значение, а затем щелкните **Создать**. Повторите эти действия для каждого из следующих параметров:

* **CognitiveServiceEndpoint.** URL-адрес конечной точки API "Распознаватель документов".
* **CognitiveServiceSubscriptionKey.** ключ доступа для службы "Распознаватель документов". 
* **StorageAccountName.** Учетная запись хранения, в которой хранятся набор данных для обучения и формы, из которых вы будете извлекать пары "ключ — значение". Если эти элементы находятся в разных учетных записях, введите имя каждой из них в качестве отдельного секрета. Не забывайте, что наборы данных для обучения для всех типов форм должны храниться в одном контейнере. Но они могут находиться в разных папках.
* **StorageAccountSasKey.** Подписанный URL-адрес (SAS) учетной записи хранения. Чтобы получить подписанный URL-адрес, перейдите к своему ресурсу хранилища. На вкладке **Обозреватель службы хранилища** найдите нужный контейнер, щелкните его правой кнопкой мыши и выберите **Получить подписанный URL-адрес**. Важно получить SAS для вашего контейнера, а не для самой учетной записи хранения. Убедитесь, что выбраны разрешения **Чтение** и **Список**, а затем щелкните **Создать**. Затем скопируйте значение в разделе **URL-адрес**. Он должен иметь следующий формат: `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.

## <a name="train-your-form-recognizer-model-in-a-databricks-notebook"></a>Обучение модели Распознавателя документов в записной книжке Databricks

Для хранения и запуска кода Python, взаимодействующего со службой "Распознаватель документов", мы будем использовать Azure Databricks.

### <a name="create-a-notebook-in-databricks"></a>Создание записной книжки в Databricks

[Создайте ресурс Azure Databricks](https://ms.portal.azure.com/#create/Microsoft.Databricks) на портале Azure. Перейдите к созданному ресурсу и откройте рабочую область.

### <a name="create-a-secret-scope-backed-by-azure-key-vault"></a>Создание области секретов на основе Azure Key Vault


Чтобы создать ссылку на секреты в созданном ранее хранилище ключей Azure, нужно создать область секретов в Databricks. Выполните инструкции из статьи [Создание области секретов с поддержкой Azure Key Vault](/azure/databricks/security/secrets/secret-scopes#--create-an-azure-key-vault-backed-secret-scope).

### <a name="create-a-databricks-cluster"></a>Создание кластера Databricks

Кластер — это коллекция вычислительных ресурсов Databricks. Чтобы создать кластер, сделайте следующее:

1. На панели слева нажмите кнопку **Кластеры**.
1. На странице **Кластеры** щелкните **Создать кластер**.
1. На странице **Создание кластера** укажите имя кластера и выберите элемент **7.2 (Scala 2.12, Spark 3.0.0)** в раскрывающемся списке **Версия Databricks Runtime**.
1. Выберите **Создать кластер**.

### <a name="write-a-settings-notebook"></a>Создание записной книжки параметров

Итак, вы готовы к добавлению записных книжек Python. Для начала создайте записную книжку с именем **Settings**. Эта записная книжка будет присваивать значения из таблицы параметризации соответствующим переменным в скрипте. Позже Фабрика данных Azure передаст эти значения в качестве параметров. Кроме того, мы присвоим переменным значения, сохраненные в виде секретов в хранилище ключей. 

1. Чтобы создать записную книжку **Settings**, нажмите кнопку **Рабочая область**. На новой вкладке щелкните раскрывающийся список и выберите **Создать** и **Записная книжка**.
1. В появившемся всплывающем окне введите имя записной книжки и выберите язык по умолчанию **Python**. Выберите кластер Databricks и щелкните **Создать**.
1. В первой ячейке записной книжки мы извлекаем параметры, переданные Фабрикой данных Azure.

    ```python 
    dbutils.widgets.text("form_batch_group_id", "","")
    dbutils.widgets.get("form_batch_group_id")
    form_batch_group_id = getArgument("form_batch_group_id")
    
    dbutils.widgets.text("model_id", "","")
    dbutils.widgets.get("model_id")
    model_id = getArgument("model_id")
    
    dbutils.widgets.text("training_container_name", "","")
    dbutils.widgets.get("training_container_name")
    training_container_name = getArgument("training_container_name")
    
    dbutils.widgets.text("training_blob_root_folder", "","")
    dbutils.widgets.get("training_blob_root_folder")
    training_blob_root_folder=  getArgument("training_blob_root_folder")
    
    dbutils.widgets.text("scoring_container_name", "","")
    dbutils.widgets.get("scoring_container_name")
    scoring_container_name=  getArgument("scoring_container_name")
    
    dbutils.widgets.text("scoring_input_blob_folder", "","")
    dbutils.widgets.get("scoring_input_blob_folder")
    scoring_input_blob_folder=  getArgument("scoring_input_blob_folder")
    
    
    dbutils.widgets.text("file_type", "","")
    dbutils.widgets.get("file_type")
    file_type = getArgument("file_type")
    
    dbutils.widgets.text("file_to_score_name", "","")
    dbutils.widgets.get("file_to_score_name")
    file_to_score_name=  getArgument("file_to_score_name")
    ```

1. Во второй ячейке мы извлекаем секреты из Key Vault и присваиваем их переменным.

    ```python 
    cognitive_service_subscription_key = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveserviceSubscriptionKey")
    cognitive_service_endpoint = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveServiceEndpoint")
    
    training_storage_account_name = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountName")
    storage_account_sas_key= dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountSasKey")  
    
    ScoredFile = file_to_score_name+ "_output.json"
    training_storage_account_url="https://"+training_storage_account_name+".blob.core.windows.net/"+training_container_name+storage_account_sas_key 
    ```

### <a name="write-a-training-notebook"></a>Создание учебной записной книжки

Теперь, когда записная книжка **Settings** готова, мы можем создать записную книжку для обучения модели. Как упоминалось выше, мы будем использовать файлы, хранящиеся в папке в учетной записи хранения Azure Data Lake Storage 2-го поколения с именем **training_blob_root_folder**. Имя папки передано в качестве переменной. Каждый набор типов форм будет занимать одну папку. По мере прохода по таблице параметров мы обучим модель, используя все эти типы форм. 

1. Создайте записную книжку с именем **TrainFormRecognizer**. 
1. В ее первой ячейке выполните записную книжку **Settings**.

    ```python
    %run "./Settings"
    ```

1. В следующей ячейке назначьте переменные из файла Settings и динамически обучите модель для каждого типа формы, применив код из [краткого руководства по REST](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-train-extract.md#get-training-results%20).

    ```python
    import json
    import time
    from requests import get, post
    
    post_url = cognitive_service_endpoint + r"/formrecognizer/v2.0/custom/models"
    source = training_storage_account_url
    prefix= training_blob_root_folder
    
    includeSubFolders=True
    useLabelFile=False
    headers = {
        # Request headers.
        'Content-Type': file_type,
        'Ocp-Apim-Subscription-Key': cognitive_service_subscription_key,
    }
    body =     {
        "source": source
        ,"sourceFilter": {
            "prefix": prefix,
            "includeSubFolders": includeSubFolders
       },
    }
    if model_id=="-1": # If you don't already have a model you want to retrain. In this case, we create a model and use it to extract the key/value pairs.
      try:
          resp = post(url = post_url, json = body, headers = headers)
          if resp.status_code != 201:
              print("POST model failed (%s):\n%s" % (resp.status_code, json.dumps(resp.json())))
              quit()
          print("POST model succeeded:\n%s" % resp.headers)
          get_url = resp.headers["location"]
          model_id=get_url[get_url.index('models/')+len('models/'):]     
          
      except Exception as e:
          print("POST model failed:\n%s" % str(e))
          quit()
    else :# If you already have a model you want to retrain, we reuse it and (re)train with the new form types.  
      try:
        get_url =post_url+r"/"+model_id
          
      except Exception as e:
          print("POST model failed:\n%s" % str(e))
          quit()
    ```

1. Заключительный этап процесса обучения — получение результата обучения в формате JSON.

    ```python
    n_tries = 10
    n_try = 0
    wait_sec = 5
    max_wait_sec = 5
    while n_try < n_tries:
        try:
            resp = get(url = get_url, headers = headers)
            resp_json = resp.json()
            print (resp.status_code)
            if resp.status_code != 200:
                print("GET model failed (%s):\n%s" % (resp.status_code, json.dumps(resp_json)))
                n_try += 1
                quit()
            model_status = resp_json["modelInfo"]["status"]
            print (model_status)
            if model_status == "ready":
                print("Training succeeded:\n%s" % json.dumps(resp_json))
                n_try += 1
                quit()
            if model_status == "invalid":
                print("Training failed. Model is invalid:\n%s" % json.dumps(resp_json))
                n_try += 1
                quit()
            # Training still running. Wait and retry.
            time.sleep(wait_sec)
            n_try += 1
            wait_sec = min(2*wait_sec, max_wait_sec)     
            print (n_try)
        except Exception as e:
            msg = "GET model failed:\n%s" % str(e)
            print(msg)
            quit()
    print("Train operation did not complete within the allocated time.")
    ```

## <a name="extract-form-data-by-using-a-notebook"></a>Извлечение данных из формы с помощью записной книжки

### <a name="mount-the-azure-data-lake-storage"></a>Подключение Azure Data Lake Storage

Следующий этап — оценка разных форм с использованием обученной модели. Мы подключим учетную запись хранения Azure Data Lake Storage к Databricks и будем использовать это подключение на этапе приема данных.

Как и ранее на этапе обучения, для извлечения пар "ключ — значение" из форм мы будем использовать Фабрику данных Azure. Мы выполним перебор всех форм в папках, которые указаны в таблице параметров.

1. Создайте записную книжку, чтобы подключить учетную запись хранения к Databricks. Присвойте ей имя **MountDataLake**. 
1. Сначала необходимо вызвать записную книжку **Settings**:

    ```python
    %run "./Settings"
    ```

1. Во второй ячейке определите переменные для конфиденциальных параметров, которые мы получим из секретов, сохраненных в Key Vault.

    ```python
    cognitive_service_subscription_key = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveserviceSubscriptionKey")
    cognitive_service_endpoint = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveServiceEndpoint")
    
    scoring_storage_account_name = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountName")
    scoring_storage_account_sas_key= dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountSasKey")  
    
    scoring_mount_point = "/mnt/"+scoring_container_name 
    scoring_source_str = "wasbs://{container}@{storage_acct}.blob.core.windows.net/".format(container=scoring_container_name, storage_acct=scoring_storage_account_name) 
    scoring_conf_key = "fs.azure.sas.{container}.{storage_acct}.blob.core.windows.net".format(container=scoring_container_name, storage_acct=scoring_storage_account_name)
    
    ```

1. Попробуйте отключить учетную запись хранения, если она была ранее подключена.

    ```python
    try:
      dbutils.fs.unmount(scoring_mount_point) # Use this to unmount as needed
    except:
      print("{} already unmounted".format(scoring_mount_point))
    
    ```

1. Подключите учетную запись хранения.


    ```python
    try: 
      dbutils.fs.mount( 
        source = scoring_source_str, 
        mount_point = scoring_mount_point, 
        extra_configs = {scoring_conf_key: scoring_storage_account_sas_key} 
      ) 
    except Exception as e: 
      print("ERROR: {} already mounted. Run previous cells to unmount first".format(scoring_mount_point))
    
    ```

    > [!NOTE]
    > Мы подключили только учетную запись хранения для обучения. Это означает, что файлы для обучения и файлы, из которых требуется извлечь пары "ключ — значение", находятся в одной учетной записи хранения. Если для оценки и обучения вы используете разные учетные записи хранения, вам потребуется подключить обе учетные записи хранения. 

### <a name="write-the-scoring-notebook"></a>Создание записной книжки оценки

Теперь мы можем создать записную книжку оценки. Как и для записной книжки для обучения, мы будем использовать файлы, хранящиеся в папках в учетной записи хранения Azure Data Lake Storage, которую мы только что подключили. Имя папки передается в виде переменной. Мы выполним перебор всех форм в указанной папке и извлечем из них пары "ключ — значение". 

1. Создайте записную книжку и присвойте ей имя **ScoreFormRecognizer**. 
1. Выполните записные книжка **Settings** и **MountDataLake**.

    ```python
    %run "./Settings"
    %run "./MountDataLake"
    ```

1. Добавьте следующий код, который вызывает [API анализа](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm):

    ```python
    ########### Python Form Recognizer Async Analyze #############
    import json
    import time
    from requests import get, post 
    
    
    #prefix= TrainingBlobFolder
    post_url = cognitive_service_endpoint + "/formrecognizer/v2.0/custom/models/%s/analyze" % model_id
    source = r"/dbfs/mnt/"+scoring_container_name+"/"+scoring_input_blob_folder+"/"+file_to_score_name
    output = r"/dbfs/mnt/"+scoring_container_name+"/scoringforms/ExtractionResult/"+os.path.splitext(os.path.basename(source))[0]+"_output.json"
    
    params = {
        "includeTextDetails": True
    }
    
    headers = {
        # Request headers.
        'Content-Type': file_type,
        'Ocp-Apim-Subscription-Key': cognitive_service_subscription_key,
    }
    
    with open(source, "rb") as f:
        data_bytes = f.read()
    
    try:
        resp = post(url = post_url, data = data_bytes, headers = headers, params = params)
        if resp.status_code != 202:
            print("POST analyze failed:\n%s" % json.dumps(resp.json()))
            quit()
        print("POST analyze succeeded:\n%s" % resp.headers)
        get_url = resp.headers["operation-location"]
    except Exception as e:
        print("POST analyze failed:\n%s" % str(e))
        quit() 
    ```

1. В следующей ячейке мы получим результаты извлечения пар "ключ — значение". В этой ячейке выводится результат. Нам нужен результат в формате JSON, чтобы обработать его в Базе данных SQL Azure или Azure Cosmos DB. Поэтому мы запишем результат в JSON-файл. Имя выходного файла состоит из имени оцениваемого файла и постфикса _output.json. Файл будет храниться в той же папке, что и исходный файл.

    ```python
    n_tries = 10
    n_try = 0
    wait_sec = 5
    max_wait_sec = 5
    while n_try < n_tries:
       try:
           resp = get(url = get_url, headers = {"Ocp-Apim-Subscription-Key": cognitive_service_subscription_key})
           resp_json = resp.json()
           if resp.status_code != 200:
               print("GET analyze results failed:\n%s" % json.dumps(resp_json))
               n_try += 1
               quit()
           status = resp_json["status"]
           if status == "succeeded":
               print("Analysis succeeded:\n%s" % json.dumps(resp_json))
               n_try += 1
               quit()
           if status == "failed":
               print("Analysis failed:\n%s" % json.dumps(resp_json))
               n_try += 1
               quit()
           # Analysis still running. Wait and retry.
           time.sleep(wait_sec)
           n_try += 1
           wait_sec = min(2*wait_sec, max_wait_sec)     
       except Exception as e:
           msg = "GET analyze results failed:\n%s" % str(e)
           print(msg)
           n_try += 1
           print("Analyze operation did not complete within the allocated time.")
           quit()
    
    ```
1. Выполните процедуру записи файла в новой ячейке:

    ```python
    import requests
    file = open(output, "w")
    file.write(str(resp_json))
    file.close()
    ```

## <a name="automate-training-and-scoring-by-using-azure-data-factory"></a>Автоматизация обучения и оценки с использованием Фабрики данных Azure

Остался один этап — настроить автоматизацию обучения и оценки в службе Фабрики данных Azure. Сначала выполните действия, описанные в разделе [Создание фабрики данных](../../data-factory/quickstart-create-data-factory-portal.md#create-a-data-factory). После создания ресурса Фабрики данных Azure необходимо создать три конвейера: один для обучения и два для оценки. (Подробнее мы их опишем далее.)

### <a name="training-pipeline"></a>Конвейер обучения

Первое действие в конвейере обучения — это запрос на поиск для чтения и возврата значений из таблицы параметризации в базе данных SQL Azure. Все наборы данных для обучения мы сохраним в одной учетной записи хранения и в одном контейнере (можно в разных папках). Поэтому для параметра Lookup activity (Действие поиска) следует сохранить значение по умолчанию **First row only** (Только для первой строки). Для каждого типа формы, используемого для обучения модели, мы применим все файлы в папке **training_blob_root_folder**.

:::image type="content" source="./media/tutorial-bulk-processing/training-pipeline.png" alt-text="Снимок экрана: конвейер обучения в Фабрике данных.":::

Хранимая процедура принимает два параметра: **model_id** и **form_batch_group_id**. Код для возврата идентификатора модели из записной книжки Databricks выглядит так: `dbutils.notebook.exit(model_id)`. Код для чтения кода в действии хранимой процедуры в Фабрике данных выглядит так: `@activity('GetParametersFromMetadaTable').output.firstRow.form_batch_group_id`.

### <a name="scoring-pipelines"></a>Конвейеры оценки

Чтобы извлечь пары "ключ — значение", мы просканируем все папки в таблице параметризации. Для каждой папки мы извлечем пары "ключ — значение" для всех файлов в ней. Фабрика данных Azure сейчас не поддерживает вложенные циклы ForEach. Поэтому мы создадим два конвейера. Первый конвейер выполняет запрос на поиск в таблице параметризации и передает список папок в качестве параметра во второй конвейер.

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-1a.png" alt-text="Снимок экрана: первый конвейер оценки в Фабрике данных.":::

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-1b.png" alt-text="Снимок экрана: подробные сведения о первом конвейере оценки в Фабрике данных.":::

Второй конвейер с помощью действия GetMeta извлекает список файлов в папке и передает его в качестве параметра в записную книжку Databricks для оценки.

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-2a.png" alt-text="Снимок экрана: второй конвейер оценки в Фабрике данных.":::

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-2b.png" alt-text="Снимок экрана: подробные сведения о втором конвейере оценки в Фабрике данных.":::

### <a name="specify-a-degree-of-parallelism"></a>Указание степени параллелизма

В конвейерах обучения и оценки можно настроить степень параллелизма, чтобы обрабатывать одновременно нескольких форм.

Чтобы настроить степень параллелизма для конвейера Фабрики данных Azure, сделайте следующее:

1. Выберите действие **ForEach**.
1. Снимите флажок **Sequential** (Последовательно).
1. Задайте степень параллелизма в поле **Batch count** (Число пакетов). Для оценки рекомендуется использовать не более 15 пакетов.

:::image type="content" source="./media/tutorial-bulk-processing/parallelism.png" alt-text="Снимок экрана: настройка параллелизма для действия оценки в Фабрике данных Azure.":::

## <a name="how-to-use-the-pipeline"></a>Использование конвейера

Теперь у вас есть автоматизированный конвейер для оцифровки невыполненной работы с формами и последующего выполнения аналитики на основе полученных данных. Когда вы будете добавлять новые формы уже известного типа в существующую папку хранения, достаточно повторно вызвать конвейеры оценки. Они обновят все выходные файлы, в том числе для новых форм. 

Если вы добавляете новые формы нового типа, в соответствующий контейнер необходимо также передать набор данных для обучения. Затем добавьте новую строку в таблицу параметризации, указав расположения новых документов и набор данных для обучения. Введите значение -1 для параметра **model_ID**, чтобы указать, что для этих форм нужно обучить новую модель. Затем запустите конвейер обучения в Фабрике данных Azure. Он получит данные из таблицы, обучит модель и перезапишет идентификатор модели в таблице. Теперь вы можете вызвать конвейеры оценки, чтобы начать запись выходных файлов.

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого руководства вы настроили конвейеры Фабрики данных Azure, чтобы активировать обучение и запуск моделей Распознавателя документов для оцифровки большого пакета файлов. Теперь изучите API "Распознаватель документов", чтобы узнать о других его возможностях.

* [REST API "Распознаватель документов"](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeBusinessCardAsync)
