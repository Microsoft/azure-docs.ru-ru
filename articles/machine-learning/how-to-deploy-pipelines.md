---
title: Публикация конвейеров ML
titleSuffix: Azure Machine Learning
description: Запуск рабочих процессов машинного обучения с помощью конвейеров машинного обучения и пакета SDK для Машинное обучение Azure для Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.author: laobri
author: lobrien
ms.date: 8/25/2020
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q1
ms.openlocfilehash: efedb21a1ec1ed53a8c6bfadf337d23a89c04383
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102520182"
---
# <a name="publish-and-track-machine-learning-pipelines"></a>Публикация и мониторинг конвейеров машинного обучения



В этой статье показано, как совместно использовать конвейер машинного обучения с вашими коллегами или клиентами.

Конвейеры машинного обучения — это многократно используемые рабочие процессы для задач машинного обучения. Одним из преимуществ конвейеров является повышение совместной работы. Вы также можете создавать конвейеры версий, что позволяет клиентам использовать текущую модель при работе с новой версией. 

## <a name="prerequisites"></a>Предварительные условия

* Создание [рабочей области машинное обучение Azure](how-to-manage-workspace.md) для хранения всех ресурсов конвейера

* [Настройка среды разработки](how-to-configure-environment.md) для установки пакета SDK машинное обучение Azure или использование [машинное обучение Azure вычислительного экземпляра](concept-compute-instance.md) с уже установленным пакетом SDK

* Создайте и запустите конвейер машинного обучения, как описано в следующем [руководстве. создание конвейера машинное обучение Azure для пакетной оценки](tutorial-pipeline-batch-scoring-classification.md). Другие варианты см. [в статье Создание и запуск конвейеров машинного обучения с помощью пакета SDK для машинное обучение Azure](./how-to-create-machine-learning-pipelines.md) .

## <a name="publish-a-pipeline"></a>Публикация конвейера

После создания и запуска конвейера можно опубликовать конвейер, чтобы он выполнялся с разными входными данными. Чтобы конечная точка RESTFUL уже опубликованного конвейера принимала параметры, необходимо настроить конвейер на использование `PipelineParameter` объектов для аргументов, которые будут отличаться.

1. Чтобы создать параметр конвейера, используйте объект [PipelineParameter](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.pipelineparameter) со значением по умолчанию.

   ```python
   from azureml.pipeline.core.graph import PipelineParameter
   
   pipeline_param = PipelineParameter(
     name="pipeline_arg",
     default_value=10)
   ```

2. Добавьте этот объект `PipelineParameter` в качестве параметра в любой из шагов в конвейере следующим образом.

   ```python
   compareStep = PythonScriptStep(
     script_name="compare.py",
     arguments=["--comp_data1", comp_data1, "--comp_data2", comp_data2, "--output_data", out_data3, "--param1", pipeline_param],
     inputs=[ comp_data1, comp_data2],
     outputs=[out_data3],
     compute_target=compute_target,
     source_directory=project_folder)
   ```

3. Опубликуйте этот конвейер, который будет принимать параметр при вызове.

   ```python
   published_pipeline1 = pipeline_run1.publish_pipeline(
        name="My_Published_Pipeline",
        description="My Published Pipeline Description",
        version="1.0")
   ```

## <a name="run-a-published-pipeline"></a>Запустите опубликованный конвейер.

Все опубликованные конвейеры используют конечную точку REST. Конечная точка конвейера позволяет запускать конвейер из любых внешних систем, включая клиентов, не относящихся к Python. Эта конечная точка обеспечивает управляемую повторяемость для сценариев пакетной оценки и переобучения.

> [!IMPORTANT]
> Если вы используете управление доступом на основе ролей Azure (Azure RBAC) для управления доступом к конвейеру, [Задайте разрешения для сценария конвейера (обучение или оценка)](how-to-assign-roles.md#common-scenarios).

Чтобы вызвать выполнение предыдущего конвейера, необходим маркер заголовка проверки подлинности Azure Active Directory. Этот маркер описан в справочнике по [классам азуреклиаусентикатион](/python/api/azureml-core/azureml.core.authentication.azurecliauthentication) и в [проверке подлинности в машинное обучение Azure](https://aka.ms/pl-restep-auth) записной книжке.

```python
from azureml.pipeline.core import PublishedPipeline
import requests

response = requests.post(published_pipeline1.endpoint,
                         headers=aad_token,
                         json={"ExperimentName": "My_Pipeline",
                               "ParameterAssignments": {"pipeline_arg": 20}})
```

`json`Аргумент для запроса POST должен содержать, для `ParameterAssignments` ключа, словарь, содержащий параметры конвейера и их значения. Кроме того, `json` аргумент может содержать следующие ключи:

| Клавиши | Описание |
| --- | --- | 
| `ExperimentName` | Имя эксперимента, связанного с этой конечной точкой |
| `Description` | Произвольный текст, описывающий конечную точку | 
| `Tags` | Произвольные пары "ключ-значение", которые могут использоваться для добавления меток и создания заметок к запросам  |
| `DataSetDefinitionValueAssignments` | Словарь, используемый для изменения наборов данных без повторного обучения (см. обсуждение ниже) | 
| `DataPathAssignments` | Словарь, используемый для изменения путей к контурам без повторного обучения (см. обсуждение ниже) | 

### <a name="run-a-published-pipeline-using-c"></a>Запуск опубликованного конвейера с помощью C # 

В следующем коде показано, как асинхронно вызывать конвейер из C#. Частичный фрагмент кода просто показывает структуру вызова и не является частью примера Microsoft. В нем не отображаются полные классы и обработка ошибок. 

```csharp
[DataContract]
public class SubmitPipelineRunRequest
{
    [DataMember]
    public string ExperimentName { get; set; }

    [DataMember]
    public string Description { get; set; }

    [DataMember(IsRequired = false)]
    public IDictionary<string, string> ParameterAssignments { get; set; }
}

// ... in its own class and method ... 
const string RestEndpoint = "your-pipeline-endpoint";

using (HttpClient client = new HttpClient())
{
    var submitPipelineRunRequest = new SubmitPipelineRunRequest()
    {
        ExperimentName = "YourExperimentName", 
        Description = "Asynchronous C# REST api call", 
        ParameterAssignments = new Dictionary<string, string>
        {
            {
                // Replace with your pipeline parameter keys and values
                "your-pipeline-parameter", "default-value"
            }
        }
    };

    string auth_key = "your-auth-key"; 
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", auth_key);

    // submit the job
    var requestPayload = JsonConvert.SerializeObject(submitPipelineRunRequest);
    var httpContent = new StringContent(requestPayload, Encoding.UTF8, "application/json");
    var submitResponse = await client.PostAsync(RestEndpoint, httpContent).ConfigureAwait(false);
    if (!submitResponse.IsSuccessStatusCode)
    {
        await WriteFailedResponse(submitResponse); // ... method not shown ...
        return;
    }

    var result = await submitResponse.Content.ReadAsStringAsync().ConfigureAwait(false);
    var obj = JObject.Parse(result);
    // ... use `obj` dictionary to access results
}
```

### <a name="run-a-published-pipeline-using-java"></a>Запуск опубликованного конвейера с помощью Java

В следующем коде показан вызов конвейера, требующего проверки подлинности (см. раздел [Настройка проверки подлинности для машинное обучение Azure ресурсов и рабочих процессов](how-to-setup-authentication.md)). Если конвейер развернут в общедоступном виде, вызовы, создающие, не нужны `authKey` . В частичном фрагменте кода не показан стандартный класс Java и шаблон обработки исключений. Код использует `Optional.flatMap` для объединения функций, которые могут возвращать пустое значение `Optional` . Использование `flatMap` сокращений и разъяснений кода, но обратите внимание, что `getRequestBody()` поглощает исключения.

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.Optional;
// JSON library
import com.google.gson.Gson;

String scoringUri = "scoring-endpoint";
String tenantId = "your-tenant-id";
String clientId = "your-client-id";
String clientSecret = "your-client-secret";
String resourceManagerUrl = "https://management.azure.com";
String dataToBeScored = "{ \"ExperimentName\" : \"My_Pipeline\", \"ParameterAssignments\" : { \"pipeline_arg\" : \"20\" }}";

HttpClient client = HttpClient.newBuilder().build();
Gson gson = new Gson();

HttpRequest tokenAuthenticationRequest = tokenAuthenticationRequest(tenantId, clientId, clientSecret, resourceManagerUrl);
Optional<String> authBody = getRequestBody(client, tokenAuthenticationRequest);
Optional<String> authKey = authBody.flatMap(body -> Optional.of(gson.fromJson(body, AuthenticationBody.class).access_token);;
Optional<HttpRequest> scoringRequest = authKey.flatMap(key -> Optional.of(scoringRequest(key, scoringUri, dataToBeScored)));
Optional<String> scoringResult = scoringRequest.flatMap(req -> getRequestBody(client, req));
// ... etc (`scoringResult.orElse()`) ... 

static HttpRequest tokenAuthenticationRequest(String tenantId, String clientId, String clientSecret, String resourceManagerUrl)
{
    String authUrl = String.format("https://login.microsoftonline.com/%s/oauth2/token", tenantId);
    String clientIdParam = String.format("client_id=%s", clientId);
    String resourceParam = String.format("resource=%s", resourceManagerUrl);
    String clientSecretParam = String.format("client_secret=%s", clientSecret);

    String bodyString = String.format("grant_type=client_credentials&%s&%s&%s", clientIdParam, resourceParam, clientSecretParam);

    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create(authUrl))
        .POST(HttpRequest.BodyPublishers.ofString(bodyString))
        .build();
    return request;
}

static HttpRequest scoringRequest(String authKey, String scoringUri, String dataToBeScored)
{
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create(scoringUri))
        .header("Authorization", String.format("Token %s", authKey))
        .POST(HttpRequest.BodyPublishers.ofString(dataToBeScored))
        .build();
    return request;

}

static Optional<String> getRequestBody(HttpClient client, HttpRequest request) {
    try {
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        if (response.statusCode() != 200) {
            System.out.println(String.format("Unexpected server response %d", response.statusCode()));
            return Optional.empty();
        }
        return Optional.of(response.body());
    }catch(Exception x)
    {
        System.out.println(x.toString());
        return Optional.empty();
    }
}

class AuthenticationBody {
    String access_token;
    String token_type;
    int expires_in;
    String scope;
    String refresh_token;
    String id_token;
    
    AuthenticationBody() {}
}
```

### <a name="changing-datasets-and-datapaths-without-retraining"></a>Изменение наборов данных и путей к ним без повторного обучения

Может потребоваться обучение и вывод для различных наборов данных и путей к ним. Например, вы можете обучить набор данных меньшего размера, но выводить его в полном наборе данных. Вы можете переключать наборы данных с помощью `DataSetDefinitionValueAssignments` ключа в `json` аргументе запроса. Пути к данным переключаются с помощью `DataPathAssignments` . Метод для обоих методов аналогичен:

1. В скрипте определения конвейера создайте `PipelineParameter` для набора данных. Создайте `DatasetConsumptionConfig` или `DataPath` из `PipelineParameter` :

    ```python
    tabular_dataset = Dataset.Tabular.from_delimited_files('https://dprepdata.blob.core.windows.net/demo/Titanic.csv')
    tabular_pipeline_param = PipelineParameter(name="tabular_ds_param", default_value=tabular_dataset)
    tabular_ds_consumption = DatasetConsumptionConfig("tabular_dataset", tabular_pipeline_param)
    ```

1. В скрипте ML получите доступ к динамически заданному набору данных с помощью `Run.get_context().input_datasets` :

    ```python
    from azureml.core import Run
    
    input_tabular_ds = Run.get_context().input_datasets['tabular_dataset']
    dataframe = input_tabular_ds.to_pandas_dataframe()
    # ... etc ...
    ```

    Обратите внимание, что скрипт ML обращается к значению, заданному для `DatasetConsumptionConfig` ( `tabular_dataset` ), а не к значению `PipelineParameter` ( `tabular_ds_param` ).

1. В скрипте определения конвейера присвойте `DatasetConsumptionConfig` параметру значение `PipelineScriptStep` :

    ```python
    train_step = PythonScriptStep(
        name="train_step",
        script_name="train_with_dataset.py",
        arguments=["--param1", tabular_ds_consumption],
        inputs=[tabular_ds_consumption],
        compute_target=compute_target,
        source_directory=source_directory)
    
    pipeline = Pipeline(workspace=ws, steps=[train_step])
    ```

1. Чтобы динамически переключать наборы данных в вызове RESTFUL, используйте `DataSetDefinitionValueAssignments` :
    
    ```python
    tabular_ds1 = Dataset.Tabular.from_delimited_files('path_to_training_dataset')
    tabular_ds2 = Dataset.Tabular.from_delimited_files('path_to_inference_dataset')
    ds1_id = tabular_ds1.id
    d22_id = tabular_ds2.id
    
    response = requests.post(rest_endpoint, 
                             headers=aad_token, 
                             json={
                                "ExperimentName": "MyRestPipeline",
                               "DataSetDefinitionValueAssignments": {
                                    "tabular_ds_param": {
                                        "SavedDataSetReference": {"Id": ds1_id #or ds2_id
                                    }}}})
    ```

В записных книжках, демонстрирующих [набор данных и пипелинепараметер](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-dataset-and-pipelineparameter.ipynb) и демонстрирующих [пути к](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-datapath-and-pipelineparameter.ipynb) данным и пипелинепараметер, приведены полные примеры этого метода.

## <a name="create-a-versioned-pipeline-endpoint"></a>Создание конечной точки конвейера с версией

Вы можете создать конечную точку конвейера с несколькими опубликованными конвейерами. Эта методика предоставляет фиксированную конечную точку RESTFUL при итерации и обновлении конвейеров машинного обучения.

```python
from azureml.pipeline.core import PipelineEndpoint

published_pipeline = PipelineEndpoint.get(workspace=ws, name="My_Published_Pipeline")
pipeline_endpoint = PipelineEndpoint.publish(workspace=ws, name="PipelineEndpointTest",
                                            pipeline=published_pipeline, description="Test description Notebook")
```

## <a name="submit-a-job-to-a-pipeline-endpoint"></a>Отправка задания в конечную точку конвейера

Вы можете отправить задание в версию по умолчанию конечной точки конвейера:

```python
pipeline_endpoint_by_name = PipelineEndpoint.get(workspace=ws, name="PipelineEndpointTest")
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment")
print(run_id)
```

Вы также можете отправить задание в определенную версию:

```python
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment", pipeline_version="0")
print(run_id)
```

То же самое можно сделать с помощью REST API:

```python
rest_endpoint = pipeline_endpoint_by_name.endpoint
response = requests.post(rest_endpoint, 
                         headers=aad_token, 
                         json={"ExperimentName": "PipelineEndpointExperiment",
                               "RunSource": "API",
                               "ParameterAssignments": {"1": "united", "2":"city"}})
```

## <a name="use-published-pipelines-in-the-studio"></a>Использование опубликованных конвейеров в студии

Вы также можете запустить опубликованный конвейер из студии:

1. Войдите в [Студию машинного обучения Azure](https://ml.azure.com).

1. [Просмотрите рабочую область](how-to-manage-workspace.md#view).

1. В левой части выберите **конечные точки**.

1. В верхней части окна выберите **конечные точки конвейера**.
 ![Список опубликованных конвейеров машинного обучения](./media/how-to-create-your-first-pipeline/pipeline-endpoints.png)

1. Выберите конкретный конвейер для выполнения, использования или просмотра результатов предыдущих запусков конечной точки конвейера.

## <a name="disable-a-published-pipeline"></a>Отключение опубликованного конвейера

Чтобы скрыть конвейер из списка опубликованных конвейеров, отключите его либо в студии, либо в пакете SDK:

```python
# Get the pipeline by using its ID from Azure Machine Learning studio
p = PublishedPipeline.get(ws, id="068f4885-7088-424b-8ce2-eeb9ba5381a6")
p.disable()
```

Его можно включить снова с помощью `p.enable()` . Дополнительные сведения см. в разделе Справочник по [классам публишедпипелине](/python/api/azureml-pipeline-core/azureml.pipeline.core.publishedpipeline) .

## <a name="next-steps"></a>Дальнейшие действия

- Используйте [эти записные книжки Jupyter на сайте GitHub](https://aka.ms/aml-pipeline-readme), чтобы подробнее изучить конвейеры машинного обучения.
- См. справочную справку по пакету [azureml-конвейеры-Core](/python/api/azureml-pipeline-core/) и пакету [azureml-конвейеры-этапов](/python/api/azureml-pipeline-steps/) .
- Советы по отладке и устранению неполадок конвейеров см. [в этой статье](how-to-debug-pipelines.md) .