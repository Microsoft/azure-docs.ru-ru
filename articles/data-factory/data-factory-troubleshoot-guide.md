---
title: Устранение неполадок в Фабрике данных Azure | Документация Майкрософт
description: Узнайте, как устранять неполадки с внешними действиями управления в Фабрике данных Azure.
author: nabhishek
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 12/30/2020
ms.author: abnarain
ms.openlocfilehash: 101e55188b8021040e2fd6bd573c1c6330241e72
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100382809"
---
# <a name="troubleshoot-azure-data-factory"></a>Устранение неполадок в Фабрике данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье рассматриваются распространенные методы устранения неполадок для внешних действий управления Фабрики данных Azure.

## <a name="connector-and-copy-activity"></a>Соединитель и действие копирования

Сведения о проблемах с соединителями, например об ошибке при использовании действия копирования, см. в статье [Устранение неполадок с соединителями Фабрики данных Azure](connector-troubleshoot-guide.md).

## <a name="azure-databricks"></a>Azure Databricks

### <a name="error-code-3200"></a>Код ошибки: 3200

- **Сообщение**. Ошибка 403.

- **Причина.** `The Databricks access token has expired.`

- **Рекомендация**. По умолчанию маркер доступа Azure Databricks действителен в течение 90 дней. Создайте маркер и обновите связанную службу.

### <a name="error-code-3201"></a>Код ошибки: 3201

- **Сообщение.** `Missing required field: settings.task.notebook_task.notebook_path.`

- **Причина.** `Bad authoring: Notebook path not specified correctly.`

- **Рекомендация**. Укажите путь к записной книжке в действии Databricks.

<br/> 

- **Сообщение.** `Cluster... does not exist.`

- **Причина.** `Authoring error: Databricks cluster does not exist or has been deleted.`

- **Рекомендация**. Убедитесь, что кластер Databricks существует.

<br/> 

- **Сообщение.** `Invalid Python file URI... Please visit Databricks user guide for supported URI schemes.`

- **Причина.** `Bad authoring.`

- **Рекомендация**. Укажите абсолютные пути для схем адресации рабочей области или `dbfs:/folder/subfolder/foo.py` для файлов, хранящихся в файловой системе Databricks (DFS).

<br/> 

- **Сообщение.** `{0} LinkedService should have domain and accessToken as required properties.`

- **Причина.** `Bad authoring.`

- **Рекомендация**. Проверьте [определение связанной службы](compute-linked-services.md#azure-databricks-linked-service).

<br/> 

- **Сообщение.** `{0} LinkedService should specify either existing cluster ID or new cluster information for creation.`

- **Причина.** `Bad authoring.`

- **Рекомендация**. Проверьте [определение связанной службы](compute-linked-services.md#azure-databricks-linked-service).

<br/> 

- **Сообщение.** `Node type Standard_D16S_v3 is not supported. Supported node types: Standard_DS3_v2, Standard_DS4_v2, Standard_DS5_v2, Standard_D8s_v3, Standard_D16s_v3, Standard_D32s_v3, Standard_D64s_v3, Standard_D3_v2, Standard_D8_v3, Standard_D16_v3, Standard_D32_v3, Standard_D64_v3, Standard_D12_v2, Standard_D13_v2, Standard_D14_v2, Standard_D15_v2, Standard_DS12_v2, Standard_DS13_v2, Standard_DS14_v2, Standard_DS15_v2, Standard_E8s_v3, Standard_E16s_v3, Standard_E32s_v3, Standard_E64s_v3, Standard_L4s, Standard_L8s, Standard_L16s, Standard_L32s, Standard_F4s, Standard_F8s, Standard_F16s, Standard_H16, Standard_F4s_v2, Standard_F8s_v2, Standard_F16s_v2, Standard_F32s_v2, Standard_F64s_v2, Standard_F72s_v2, Standard_NC12, Standard_NC24, Standard_NC6s_v3, Standard_NC12s_v3, Standard_NC24s_v3, Standard_L8s_v2, Standard_L16s_v2, Standard_L32s_v2, Standard_L64s_v2, Standard_L80s_v2.`

- **Причина.** `Bad authoring.`

- **Рекомендация**. См. сообщение об ошибке.

<br/>

### <a name="error-code-3202"></a>Код ошибки: 3202

- **Сообщение.** `There were already 1000 jobs created in past 3600 seconds, exceeding rate limit: 1000 job creations per 3600 seconds.`

- **Причина.** `Too many Databricks runs in an hour.`

- **Рекомендация**. Проверьте все конвейеры, в которых для скорости создания заданий используется эта рабочая область Databricks. Если конвейеры запустили слишком много Databricks, работающих в статистическом выражении, перенесите некоторые конвейеры в новую рабочую область.

<br/> 

- **Сообщение.** `Could not parse request object: Expected 'key' and 'value' to be set for JSON map field base_parameters, got 'key: "..."' instead.`

- **Причина.** `Authoring error: No value provided for the parameter.`

- **Рекомендация**. Проверьте конвейер JSON и убедитесь, что все параметры в записной книжке baseParameters указывают непустое значение.

<br/> 

- **Сообщение**. `User: `SimpleUserContext{userId=..., name=user@company.com, orgId=...}` is not authorized to access cluster.`

- **Причина.** Пользователю, создавшему маркер доступа, не разрешен доступ к кластеру Databricks, указанному в связанной службе.

- **Рекомендация**. Убедитесь, что у пользователя есть необходимые разрешения в рабочей области.

### <a name="error-code-3203"></a>Код ошибки: 3203

- **Сообщение.** `The cluster is in Terminated state, not available to receive jobs. Please fix the cluster or retry later.`

- **Причина.** Работа кластера завершена. В случае с интерактивными кластерами эта проблема может быть состоянием гонки.

- **Рекомендация**. Во избежание этой ошибки используйте кластеры заданий.

### <a name="error-code-3204"></a>Код ошибки: 3204

- **Сообщение.** `Job execution failed.`

- **Причина.** Сообщения об ошибках указывают на различные проблемы, такие как непредвиденное состояние кластера или определенное действие. Зачастую сообщение об ошибке не отображается.

- **Рекомендация**. Недоступно

### <a name="error-code-3208"></a>Код ошибки: 3208

- **Сообщение.** `An error occurred while sending the request.`

- **Причина**: было прервано сетевое подключение к службе "кирпичы".

- **Рекомендация**. Если вы используете локальную среду выполнения интеграции, убедитесь, что сетевое подключение надежно от узлов среды выполнения интеграции. Если вы используете среду выполнения интеграции Azure, обычно используется повторная попытка.
 
## <a name="azure-data-lake-analytics"></a>Аналитика озера данных Azure

Приведенная ниже таблица относится к U-SQL.
 
### <a name="error-code-2709"></a>Код ошибки: 2709

- **Сообщение.** `The access token is from the wrong tenant.`

- **Причина.** Недопустимый клиент Azure Active Directory (Azure AD).

- **Рекомендация**. Недопустимый клиент Azure Active Directory (Azure AD).

<br/>

- **Сообщение.** `We cannot accept your job at this moment. The maximum number of queued jobs for your account is 200. `

- **Причина.** Эта ошибка вызвана регулированием в Data Lake Analytics.

- **Рекомендация**. Сократите число отправленных заданий в Data Lake Analytics. Измените триггеры Фабрики данных и параметры параллелизма для действий или увеличьте пределы в Data Lake Analytics.

<br/> 

- **Сообщение.** `This job was rejected because it requires 24 AUs. This account's administrator-defined policy prevents a job from using more than 5 AUs.`

- **Причина.** Эта ошибка вызвана регулированием в Data Lake Analytics.

- **Рекомендация**. Сократите число отправленных заданий в Data Lake Analytics. Измените триггеры Фабрики данных и параметры параллелизма для действий или увеличьте пределы в Data Lake Analytics.

### <a name="error-code-2705"></a>Код ошибки: 2705

- **Сообщение.** `Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.<br/> <br/> User is not able to access Data Lake Store. <br/> <br/> User is not authorized to use Data Lake Analytics.`

- **Причина.** У субъекта-службы или сертификата нет доступа к файлу в хранилище.

- **Рекомендация**. Убедитесь, что субъект-служба или сертификат, предоставленные пользователем для заданий Data Lake Analytics, имеют доступ как к учетной записи Data Lake Analytics, так и к экземпляру Data Lake Storage по умолчанию из корневой папки.

### <a name="error-code-2711"></a>Код ошибки: 2711

- **Сообщение.** `Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.<br/> <br/> User is not able to access Data Lake Store. <br/> <br/> User is not authorized to use Data Lake Analytics.`

- **Причина.** У субъекта-службы или сертификата нет доступа к файлу в хранилище.

- **Рекомендация**. Убедитесь, что субъект-служба или сертификат, предоставленные пользователем для заданий Data Lake Analytics, имеют доступ как к учетной записи Data Lake Analytics, так и к экземпляру Data Lake Storage по умолчанию из корневой папки.

<br/> 

- **Сообщение.** `Cannot find the 'Azure Data Lake Store' file or folder.`

- **Причина.** Неверный путь к файлу U-SQL или учетные данные связанной службы не имеют права доступа.

- **Рекомендация**. Проверьте путь и учетные данные, предоставленные в связанной службе.

### <a name="error-code-2704"></a>Код ошибки: 2704

- **Сообщение.** `Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.<br/> <br/> User is not able to access Data Lake Store. <br/> <br/> User is not authorized to use Data Lake Analytics.`

- **Причина.** У субъекта-службы или сертификата нет доступа к файлу в хранилище.

- **Рекомендация**. Убедитесь, что субъект-служба или сертификат, предоставленные пользователем для заданий Data Lake Analytics, имеют доступ как к учетной записи Data Lake Analytics, так и к экземпляру Data Lake Storage по умолчанию из корневой папки.

### <a name="error-code-2707"></a>Код ошибки: 2707

- **Сообщение.** `Cannot resolve the account of AzureDataLakeAnalytics. Please check 'AccountName' and 'DataLakeAnalyticsUri'.`

- **Причина.** Неправильная учетная запись Data Lake Analytics в связанной службе.

- **Рекомендация**. Убедитесь, что указана правильная учетная запись.

### <a name="error-code-2703"></a>Код ошибки: 2703

- **Сообщение.** `Error Id: E_CQO_SYSTEM_INTERNAL_ERROR (or any error that starts with "Error Id:").`

- **Причина.** Ошибка из Data Lake Analytics.

- **Рекомендация**. Задание, отправленное в Data Lake Analytics, и сценарий не выполнены. Выполните исследование в Data Lake Analytics. На портале перейдите к учетной записи Data Lake Analytics и найдите задание, используя идентификатор выполнения действия Фабрики данных (не используйте идентификатор запуска конвейера). Задание там содержит больше сведений об ошибке и поможет устранить неполадки.

   Если разрешение неясно, обратитесь в группу поддержки Data Lake Analytics и предоставьте URL-адрес задания, которое включает имя учетной записи и идентификатор задания.
 
## <a name="azure-functions"></a>Функции Azure

### <a name="error-code-3602"></a>Код ошибки: 3602

- **Сообщение.** `Invalid HttpMethod: '%method;'.`

- **Причина.** Метод Httpmethod, указанный в полезных данных действия, не поддерживается действием функции Azure.

- **Рекомендация**. Поддерживаемые методы Httpmethod: PUT, POST, GET, DELETE, OPTIONS, HEAD и TRACE.

### <a name="error-code-3603"></a>Код ошибки: 3603

- **Сообщение.** `Response Content is not a valid JObject.`

- **Причина.** Вызванная функция Azure не вернула в ответе полезные данные JSON. Действие функции Фабрики данных Azure (ADF) поддерживает только содержимое ответов JSON.

- **Рекомендация**. Обновите функцию Azure, чтобы она возвращала допустимые полезные данные JSON, например функция C# может возвращать `(ActionResult)new OkObjectResult("{\"Id\":\"123\"}");`.

### <a name="error-code-3606"></a>Код ошибки: 3606

- **Сообщение**. В функции Azure отсутствует ключ функции.

- **Причина.** Определение действия функции Azure не завершено.

- **Рекомендация**. Убедитесь, что у входного определения JSON действия функции Azure есть свойство с именем `functionKey`.

### <a name="error-code-3607"></a>Код ошибки: 3607

- **Сообщение.** `Azure function activity missing function name.`

- **Причина.** Определение действия функции Azure не завершено.

- **Рекомендация**. Убедитесь, что у входного определения JSON действия функции Azure есть свойство с именем `functionName`.

### <a name="error-code-3608"></a>Код ошибки: 3608

- **Сообщение.** `Call to provided Azure function '%FunctionName;' failed with status-'%statusCode;' and message - '%message;'.`

- **Причина.** Сведения о функции Azure в определении действия могут быть неправильными.

- **Рекомендация**. Исправьте сведения о функции Azure и повторите попытку.

### <a name="error-code-3609"></a>Код ошибки: 3609

- **Сообщение.** `Azure function activity missing functionAppUrl.`

- **Причина.** Определение действия функции Azure не завершено.

- **Рекомендация**. Убедитесь, что у входного определения JSON действия функции Azure есть свойство с именем `functionAppUrl`.

### <a name="error-code-3610"></a>Код ошибки: 3610

- **Сообщение.** `There was an error while calling endpoint.`

- **Причина.** URL-адрес функции может быть неправильным.

- **Рекомендация**. Убедитесь, что значение для `functionAppUrl` в действии JSON является правильным, и повторите попытку.

### <a name="error-code-3611"></a>Код ошибки: 3611

- **Сообщение.** `Azure function activity missing Method in JSON.`

- **Причина.** Определение действия функции Azure не завершено.

- **Рекомендация**. Убедитесь, что у входного определения JSON действия функции Azure есть свойство с именем `method`.

### <a name="error-code-3612"></a>Код ошибки: 3612

- **Сообщение.** `Azure function activity missing LinkedService definition in JSON.`

- **Причина.** Определение действия функции Azure не завершено.

- **Рекомендация**. Убедитесь, что входное определение JSON действия функции Azure содержит сведения о связанной службе.

## <a name="azure-machine-learning"></a>Машинное обучение Azure

### <a name="error-code-4101"></a>Код ошибки: 4101

- **Сообщение.** `AzureMLExecutePipeline activity '%activityName;' has invalid value for property '%propertyName;'.`

- **Причина.** Недопустимый формат или отсутствует определение свойства `%propertyName;`.

- **Рекомендация**. Проверьте, есть ли у действия `%activityName;` свойство `%propertyName;` с правильными данными.

### <a name="error-code-4110"></a>Код ошибки: 4110

- **Сообщение.** `AzureMLExecutePipeline activity missing LinkedService definition in JSON.`

- **Причина.** Определение действия AzureMLExecutePipeline не завершено.

- **Рекомендация**. Убедитесь, что во входном определении JSON действия AzureMLExecutePipeline правильно указаны сведения о связанной службе.

### <a name="error-code-4111"></a>Код ошибки: 4111

- **Сообщение.** `AzureMLExecutePipeline activity has wrong LinkedService type in JSON. Expected LinkedService type: '%expectedLinkedServiceType;', current LinkedService type: Expected LinkedService type: '%currentLinkedServiceType;'.`

- **Причина.** Неправильное определение действия.

- **Рекомендация**. Убедитесь, что во входном определении JSON действия AzureMLExecutePipeline правильно указаны сведения о связанной службе.

### <a name="error-code-4112"></a>Код ошибки: 4112

- **Сообщение.** `AzureMLService linked service has invalid value for property '%propertyName;'.`

- **Причина.** Недопустимый формат или отсутствует определение свойства %propertyName;.

- **Рекомендация**. Проверьте, имеет ли связанная служба свойство `%propertyName;` с правильными данными.

### <a name="error-code-4121"></a>Код ошибки: 4121

- **Сообщение.** `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Причина.** Срок действия учетных данных, используемых для доступа к Машинному обучению Azure, истек.

- **Рекомендация**. Проверьте действительность учетных данных и повторите попытку.

### <a name="error-code-4122"></a>Код ошибки: 4122

- **Сообщение.** `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Причина.** Учетные данные, предоставленные в связанной службе Машинного обучения Azure, недопустимы или не имеют разрешения для операции.

- **Рекомендация**. Убедитесь, что учетные данные в связанной службе допустимы и имеют разрешение на доступ к Машинному обучению Azure.

### <a name="error-code-4123"></a>Код ошибки: 4123

- **Сообщение.** `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Причина.** Свойства действия, такие как `pipelineParameters`, недопустимы для конвейера Машинного обучения Azure.

- **Рекомендация**. Убедитесь, что значение свойств действия соответствует ожидаемым полезным данным опубликованного конвейера Машинного обучения Azure, указанного в связанной службе.

### <a name="error-code-4124"></a>Код ошибки: 4124

- **Сообщение.** `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Причина.** Конечная точка опубликованного конвейера Машинного обучения Azure не существует.

- **Рекомендация**. Убедитесь, что конечная точка опубликованного конвейера Машинного обучения Azure, указанная в связанной службе, существует в Машинном обучении Azure.

### <a name="error-code-4125"></a>Код ошибки: 4125

- **Сообщение.** `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Причина.** Произошла ошибка сервера в Машинном обучении Azure.

- **Рекомендация**. Повторите попытку позже. Свяжитесь с командой Машинного обучения Azure, если проблема не исчезнет.

### <a name="error-code-4126"></a>Код ошибки: 4126

- **Сообщение.** `Azure ML pipeline run failed with status: '%amlPipelineRunStatus;'. Azure ML pipeline run Id: '%amlPipelineRunId;'. Please check in Azure Machine Learning for more error logs.`

- **Причина.** Сбой выполнения конвейера Машинного обучения Azure.

- **Рекомендация**. Проверьте Машинное обучение Azure на наличие дополнительных журналов ошибок, а затем исправьте конвейер Машинного обучения.

## <a name="common"></a>Распространенные

### <a name="error-code-2103"></a>Код ошибки: 2103

- **Сообщение.** `Please provide value for the required property '%propertyName;'.`

- **Причина.** Не указано требуемое значение для свойства.

- **Рекомендация**. Укажите значение из сообщения и повторите попытку.

### <a name="error-code-2104"></a>Код ошибки: 2104

- **Сообщение.** `The type of the property '%propertyName;' is incorrect.`

- **Причина.** Указано неправильный тип свойства.

- **Рекомендация**. Исправьте тип свойства и повторите попытку.

### <a name="error-code-2105"></a>Код ошибки: 2105

- **Сообщение.** `An invalid json is provided for property '%propertyName;'. Encountered an error while trying to parse: '%message;'.`

- **Причина.** Значение свойства недопустимо или не соответствует ожидаемому формату.

- **Рекомендация**. Ознакомьтесь с документацией по этому свойству и убедитесь, что указанное значение имеет правильный формат и тип.

### <a name="error-code-2106"></a>Код ошибки: 2106

- **Сообщение.** `The storage connection string is invalid. %errorMessage;`

- **Причина.** Строка подключения для хранилища недопустима или имеет неправильный формат.

- **Рекомендация**. Перейдите на портал Azure и найдите хранилище, затем скопируйте и вставьте строку подключения в связанную службу и повторите попытку.

### <a name="error-code-2108"></a>Код ошибки: 2108

- **Сообщение.** `Error calling the endpoint '%url;'. Response status code: '%code;'`

- **Причина.** Произошел сбой запроса из-за основной проблемы, такой как сетевое подключение, сбой DNS, проверка сертификата сервера или превышение времени ожидания.

- **Рекомендация**. Используйте Fiddler или Postman для проверки запроса.

### <a name="error-code-2110"></a>Код ошибки: 2110

- **Сообщение.** `The linked service type '%linkedServiceType;' is not supported for '%executorType;' activities.`

- **Причина.** Указанная в действии связанная служба неверна.

- **Рекомендация**. Убедитесь, что связанная служба имеет один из поддерживаемых типов для действия. Например, тип связанной службы для действий HDI может быть HDInsight или HDInsightOnDemand.

### <a name="error-code-2111"></a>Код ошибки: 2111

- **Сообщение.** `The type of the property '%propertyName;' is incorrect. The expected type is %expectedType;.`

- **Причина.** Указан неправильный тип предоставленного свойства.

- **Рекомендация**. Исправьте тип свойства и повторите попытку.

### <a name="error-code-2112"></a>Код ошибки: 2112

- **Сообщение.** `The cloud type is unsupported or could not be determined for storage from the EndpointSuffix '%endpointSuffix;'.`

- **Причина.** Тип облака не поддерживается или не может быть определен для хранилища из EndpointSuffix.

- **Рекомендация**. Используйте хранилище в другом облаке и повторите попытку.

### <a name="error-code-2128"></a>Код ошибки: 2128

- **Сообщение.** `No response from the endpoint. Possible causes: network connectivity, DNS failure, server certificate validation or timeout.`

- **Причина.** Сетевое подключение, сбой DNS, проверка сертификата сервера или превышение времени ожидания.

- **Рекомендация**. Убедитесь, что конечная точка, к которой вы пытаетесь подключиться, реагирует на запросы. Вы можете использовать такие инструменты, как Fiddler или Postman.

## <a name="custom"></a>Другой

Приведенная ниже таблица относится к пакетной службе Azure.
 
### <a name="error-code-2500"></a>Код ошибки: 2500

- **Сообщение.** `Hit unexpected exception and execution failed.`

- **Причина.** `Can't launch command, or the program returned an error code.`

- **Рекомендация**. Убедитесь, что исполняемый файл существует. Если программа запущена, убедитесь, что *stdout.txt* и *stderr.txt* переданы в учетную запись хранения. Рекомендуется включить журналы в код для отладки.

### <a name="error-code-2501"></a>Код ошибки: 2501

- **Сообщение.** `Cannot access user batch account; please check batch account settings.`

- **Причина.** Неверный ключ доступа или имя пула пакетной службы.

- **Рекомендация**. Проверьте в связанной службе имя пула и ключ доступа пакетной службы.

### <a name="error-code-2502"></a>Код ошибки: 2502

- **Сообщение.** `Cannot access user storage account; please check storage account settings.`

- **Причина.** Неверное имя учетной записи хранения или ключ доступа.

- **Рекомендация**. Проверьте в связанной службе имя учетной записи хранения и ключ доступа.

### <a name="error-code-2504"></a>Код ошибки: 2504

- **Сообщение.** `Operation returned an invalid status code 'BadRequest'.`

- **Причина.** Слишком много файлов в `folderPath` настраиваемого действия. Общий размер `resourceFiles` не может превышать 32 768 символов.

- **Рекомендация**. Удалите ненужные файлы или заархивируйте и добавьте команду unzip, чтобы извлечь их.
   
   Например, используйте `powershell.exe -nologo -noprofile -command "& { Add-Type -A 'System.IO.Compression.FileSystem'; [IO.Compression.ZipFile]::ExtractToDirectory($zipFile, $folder); }" ; $folder\yourProgram.exe`.

### <a name="error-code-2505"></a>Код ошибки: 2505

- **Сообщение.** `Cannot create Shared Access Signature unless Account Key credentials are used.`

- **Причина.** Настраиваемые действия поддерживают только те учетные записи хранения, которые используют ключ доступа.

- **Рекомендация**. Ознакомьтесь с описанием ошибки.

### <a name="error-code-2507"></a>Код ошибки: 2507

- **Сообщение.** `The folder path does not exist or is empty: ...`

- **Причина.** Отсутствуют файлы в учетной записи хранения по указанному пути.

- **Рекомендация**. Путь к папке должен содержать исполняемые файлы, которые необходимо запустить.

### <a name="error-code-2508"></a>Код ошибки: 2508

- **Сообщение.** `There are duplicate files in the resource folder.`

- **Причина**: несколько файлов с одинаковыми именами находятся в разных вложенных папках FolderPath.

- **Рекомендация**. Настраиваемые действия преобразовывают структуру папок в folderPath в плоскую структуру. Если вам нужно сохранить структуру папок, заархивируйте файлы и извлеките их в пакетной службе Azure с помощью команды unzip.
   
   Например, используйте `powershell.exe -nologo -noprofile -command "& { Add-Type -A 'System.IO.Compression.FileSystem'; [IO.Compression.ZipFile]::ExtractToDirectory($zipFile, $folder); }" ; $folder\yourProgram.exe`.

### <a name="error-code-2509"></a>Код ошибки: 2509

- **Сообщение.** `Batch url ... is invalid; it must be in Uri format.`

- **Причина.** URL-адреса пакетной службы должны выглядеть приблизительно так: `https://mybatchaccount.eastus.batch.azure.com`.

- **Рекомендация**. Ознакомьтесь с описанием ошибки.

### <a name="error-code-2510"></a>Код ошибки: 2510

- **Сообщение.** `An error occurred while sending the request.`

- **Причина.** Недопустимый URL-адрес пакетной службы.

- **Рекомендация**. Проверьте URL-адрес пакетной службы.
 
## <a name="hdinsight"></a>HDInsight

### <a name="error-code-206"></a>Код ошибки: 206

- **Сообщение.** `The batch ID for Spark job is invalid. Please retry your job.`

- **Причина.** Возникла внутренняя проблема со службой, которая вызвала эту ошибку.

- **Рекомендация**. Эта проблема может быть временной. Повторите задание через некоторое время.

### <a name="error-code-207"></a>Код ошибки: 207

- **Сообщение.** `Could not determine the region from the provided storage account. Please try using another primary storage account for the on demand HDI.`

- **Причина.** Возникла внутренняя ошибка при попытке определить регион из основной учетной записи хранения.

- **Рекомендация**. Попробуйте использовать другое хранилище. 

### <a name="error-code-208"></a>Код ошибки: 208

- **Сообщение.** `Service Principal or the MSI authenticator are not instantiated. Please consider providing a Service Principal in the HDI on demand linked service which has permissions to create an HDInsight cluster in the provided subscription and try again.`

- **Причина.** Возникла внутренняя ошибка при попытке чтения субъекта-службы или при создании экземпляра проверки подлинности MSI.

- **Рекомендация**. Предоставьте субъект-службу, у которой есть разрешения на создание кластера HDInsight в указанной подписке, и повторите попытку. Проверьте [правильность настройки Управления удостоверениями](../hdinsight/hdinsight-managed-identities.md).

### <a name="error-code-2300"></a>Код ошибки: 2300

- **Сообщение.** `Failed to submit the job '%jobId;' to the cluster '%cluster;'. Error: %errorMessage;.`

- **Причина.** Сообщение об ошибке содержит текст, подобный этому: `The remote name could not be resolved.`. Указанный URI кластера может быть недопустимым.

- **Рекомендация**. Убедитесь, что кластер не был удален и что указан правильный URI. Когда вы открываете URI в браузере, должен отобразиться пользовательский интерфейс Ambari. Если кластер находится в виртуальной сети, URI должен быть частным. Чтобы открыть его, используйте виртуальную машину, которая является частью той же виртуальной сети.

   Дополнительные сведения см. в разделе [Прямое соединение со службами Apache Hadoop](../hdinsight/hdinsight-plan-virtual-network-deployment.md#directly-connect-to-apache-hadoop-services).
 
 </br>

- **Причина.** Если сообщение об ошибке содержит текст, подобный `A task was canceled.`, время отправки задания истекло.

- **Рекомендация**. Проблема может быть связана с общим подключением к HDInsight или с сетевым подключением. Сначала убедитесь, что пользовательский интерфейс Ambari в HDInsight доступен из любого браузера. Затем проверьте, действительны ли учетные данные.
   
   Если вы используете локальную среду выполнения интеграции (IR), выполните этот шаг из виртуальной машины или компьютера, на котором установлена локальная среда IR. Затем попробуйте снова отправить задание из Фабрики данных.

   Дополнительные сведения см. в разделе [Веб-интерфейс Ambari](../hdinsight/hdinsight-hadoop-manage-ambari.md#ambari-web-ui).

 </br>

- **Причина.** Если сообщение об ошибке содержит текст, похожий на `User admin is locked out in Ambari` или `Unauthorized: Ambari user name or password is incorrect`, учетные данные для HDInsight неправильные или устарели.

- **Рекомендация**. Исправьте учетные данные и повторите развертывание связанной службы. Сначала убедитесь, что учетные данные работают в HDInsight. Для этого откройте URI кластера в любом браузере и попытайтесь выполнить вход. Если учетные данные не работают, их можно сбросить на портале Azure.

   Для кластера ESP сбросьте пароль с помощью функции [самостоятельного сброса пароля](../active-directory/user-help/active-directory-passwords-update-your-own-password.md).

 </br>

- **Причина.** Если в сообщении об ошибке содержится текст, подобный `502 - Web server received an invalid response while acting as a gateway or proxy server`, эта ошибка возвращается службой HDInsight.

- **Рекомендация**. Ошибка 502 часто возникает при остановке процесса сервера Ambari. Службы Ambari можно перезапустить, перезагрузив головной узел.

    1. Подключитесь к одному из узлов в HDInsight, используя SSH.
    1. Определите активный узел головного узла, запустив `ping headnodehost`.
    1. Подключитесь к активному головному узлу, так как сервер Ambari находится на активном головном узле, используя SSH. 
    1. Перезагрузите активный головной узел.

       Дополнительные сведения см. в документации по устранению неполадок с Azure HDInsight. Пример:

       * [Ambari UI 502 error](https://hdinsight.github.io/ambari/ambari-ui-502-error.html) (Ошибка 502 интерфейса Ambari)
       * [Сценарий. RpcTimeoutException для сервера Thrift Apache Spark в Azure HDInsight](../hdinsight/spark/apache-spark-troubleshoot-rpctimeoutexception.md)
       * [Устранение ошибок "Недопустимый шлюз" в шлюзе приложений](../application-gateway/application-gateway-troubleshooting-502.md)

 </br>

- **Причина.** Если сообщение об ошибке содержит текст, похожий на `Unable to service the submit job request as templeton service is busy with too many submit job requests` или `Queue root.joblauncher already has 500 applications, cannot accept submission of application`, в HDInsight одновременно отправляется слишком много заданий.

- **Рекомендация**. Ограничьте количество одновременных заданий, отправляемых в HDInsight. Ознакомьтесь со сведениями о параллелизме действий Фабрики данных, если задания отправляются одним и тем же действием. Измените триггеры так, чтобы параллельные запуски конвейера распределялись по времени.

   Ознакомьтесь со статьей [Понимание и устранение ошибок, полученных из WebHCat в HDInsight](../hdinsight/hdinsight-hadoop-templeton-webhcat-debug-errors.md), чтобы настроить `templeton.parallellism.job.submit` как предполагается в ошибке.

### <a name="error-code-2301"></a>Код ошибки: 2301

- **Сообщение.** `Could not get the status of the application '%physicalJobId;' from the HDInsight service. Received the following error: %message;. Please refer to HDInsight troubleshooting documentation or contact their support for further assistance.`

- **Причина.** Возникли проблемы с кластером или службой HDInsight.

- **Рекомендация**. Эта ошибка возникает, если при попытке запросить состояние выполняемого задания ADF не получает ответ от кластера HDInsight. Эта проблема может быть связана с самим кластером, или это следствие сбоя службы HDInsight.

   Ознакомьтесь с документацией по устранению неполадок в HDInsight по адресу https://docs.microsoft.com/azure/hdinsight/hdinsight-troubleshoot-guide или обратитесь в службу поддержки за дополнительной помощью.

### <a name="error-code-2302"></a>Код ошибки: 2302

- **Сообщение.** `Hadoop job failed with exit code '%exitCode;'. See '%logPath;/stderr' for more details. Alternatively, open the Ambari UI on the HDI cluster and find the logs for the job '%jobId;'. Contact HDInsight team for further support.`

- **Причина.** Задание было отправлено в кластер HDI и не выполнено.

- **Рекомендация**. 

 1. Проверьте пользовательский интерфейс Ambari, выполнив приведенные ниже действия.
    1. Убедитесь, что все службы по-прежнему работают.
    1. В пользовательском интерфейсе Ambari на панели мониторинга проверьте раздел оповещений.
       1. Дополнительные сведения об оповещениях и разрешениях для них см. на странице [Мониторинг кластера и управление им](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.5.0/managing-and-monitoring-ambari/content/amb_predefined_alerts.html).
    1. Проверьте память YARN. Если у вас большой объем памяти YARN, обработка заданий может быть отложена. Если у вас недостаточно ресурсов для размещения приложения или задания Spark, вертикально увеличьте масштаб кластера, чтобы в нем стало достаточно памяти и ядер. 
 1. Выполните пример тестового задания.
    1. Если вы выполняете то же задание на серверной части HDInsight, убедитесь, что оно выполнено успешно. Дополнительные сведения см. в статье [Выполнение примеров MapReduce, включенных в HDInsight](../hdinsight/hadoop/apache-hadoop-run-samples-linux.md). 
 1. Если по-прежнему происходит сбой выполнения задания в HDInsight, проверьте журналы приложений и просмотрите сведения, которые следует предоставить в службу поддержки:
    1. Проверьте, было ли задание отправлено в YARN. Если задание не было отправлено в YARN, используйте `--master yarn`.
    1. Если приложение завершило выполнение, соберите данные о времени начала и окончания приложения YARN. Если приложение не завершило выполнение, соберите данные о времени начала или запуска.
    1. Проверьте журнал приложений и соберите данные с помощью `yarn logs -applicationId <Insert_Your_Application_ID>`.
    1. Проверьте журналы YARN Resource Manager и соберите из них данные в каталоге `/var/log/hadoop-yarn/yarn`.
    1. Если с помощью этих шагов не удалось устранить проблему, обратитесь в службу поддержки Azure HDInsight и предоставьте указанные журналы и метки времени.

### <a name="error-code-2303"></a>Код ошибки: 2303

- **Сообщение.** `Hadoop job failed with transient exit code '%exitCode;'. See '%logPath;/stderr' for more details. Alternatively, open the Ambari UI on the HDI cluster and find the logs for the job '%jobId;'. Try again or contact HDInsight team for further support.`

- **Причина.** Задание было отправлено в кластер HDI и не выполнено.

- **Рекомендация**. 

 1. Проверьте пользовательский интерфейс Ambari, выполнив приведенные ниже действия.
    1. Убедитесь, что все службы по-прежнему работают.
    1. В пользовательском интерфейсе Ambari на панели мониторинга проверьте раздел оповещений.
       1. Дополнительные сведения об оповещениях и разрешениях для них см. на странице [Мониторинг кластера и управление им](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.5.0/managing-and-monitoring-ambari/content/amb_predefined_alerts.html).
    1. Проверьте память YARN. Если у вас большой объем памяти YARN, обработка заданий может быть отложена. Если у вас недостаточно ресурсов для размещения приложения или задания Spark, вертикально увеличьте масштаб кластера, чтобы в нем стало достаточно памяти и ядер. 
 1. Выполните пример тестового задания.
    1. Если вы выполняете то же задание на серверной части HDInsight, убедитесь, что оно выполнено успешно. Дополнительные сведения см. в статье [Выполнение примеров MapReduce, включенных в HDInsight](../hdinsight/hadoop/apache-hadoop-run-samples-linux.md). 
 1. Если по-прежнему происходит сбой выполнения задания в HDInsight, проверьте журналы приложений и просмотрите сведения, которые следует предоставить в службу поддержки:
    1. Проверьте, было ли задание отправлено в YARN. Если задание не было отправлено в YARN, используйте `--master yarn`.
    1. Если приложение завершило выполнение, соберите данные о времени начала и окончания приложения YARN. Если приложение не завершило выполнение, соберите данные о времени начала или запуска.
    1. Проверьте журнал приложений и соберите данные с помощью `yarn logs -applicationId <Insert_Your_Application_ID>`.
    1. Проверьте журналы YARN Resource Manager и соберите из них данные в каталоге `/var/log/hadoop-yarn/yarn`.
    1. Если с помощью этих шагов не удалось устранить проблему, обратитесь в службу поддержки Azure HDInsight и предоставьте указанные журналы и метки времени.

### <a name="error-code-2304"></a>Код ошибки: 2304

- **Сообщение.** `MSI authentication is not supported on storages for HDI activities.`

- **Причина.** Связанные службы хранения, используемые в связанной службе HDInsight (HDI) или действии HDI, настроены с проверкой подлинности MSI, которая не поддерживается.

- **Рекомендация**. Укажите полные строки подключения для учетных записей хранения, используемых в связанной службе HDI или действии HDI.

### <a name="error-code-2305"></a>Код ошибки: 2305

- **Сообщение.** `Failed to initialize the HDInsight client for the cluster '%cluster;'. Error: '%message;'`

- **Причина.** Сведения о подключении для кластера HDI неверны, у предоставленного пользователя нет прав на выполнение требуемого действия или у службы HDInsight есть проблемы с ответами на запросы от ADF.

- **Рекомендация**. Убедитесь, что информация о пользователе правильная, а также что пользовательский интерфейс Ambari для кластера HDI можно открыть в браузере из виртуальной машины, на которой установлена среда IR (для локальной среды IR), или с любого компьютера (для Azure IR).

### <a name="error-code-2306"></a>Код ошибки: 2306

- **Сообщение.** `An invalid json is provided for script action '%scriptActionName;'. Error: '%message;'`

- **Причина.** JSON, предоставленный для действия сценария, имеет недопустимый формат.

- **Рекомендация**. Сообщение об ошибке должно помочь определить проблему. Исправьте конфигурацию JSON и повторите попытку.

   Дополнительные сведения см. в разделе [Связанная служба Azure HDInsight по запросу](./compute-linked-services.md#azure-hdinsight-on-demand-linked-service).

### <a name="error-code-2310"></a>Код ошибки: 2310

- **Сообщение.** `Failed to submit Spark job. Error: '%message;'`

- **Причина.** Служба ADF попыталась создать пакет в кластере Spark с помощью API Livy (livy/batch), но произошла ошибка.

- **Рекомендация**. Следуйте указаниям в сообщении об ошибке, чтобы устранить проблему. Если для решения проблемы недостаточно сведений, обратитесь в группу HDI и предоставьте им идентификатор пакета и идентификатор задания (их можно найти в выходных данных выполнения действия на странице мониторинга ADF). Чтобы продолжить устранение неполадок, соберите данные полного журнала пакетного задания.

   Дополнительные сведения о сборе данных полного журнала см. в разделе [Получение полного журнала пакетного задания](/rest/api/hdinsightspark/hdinsight-spark-batch-job#get-the-full-log-of-a-batch-job).

### <a name="error-code-2312"></a>Код ошибки: 2312

- **Сообщение.** `Spark job failed, batch id:%batchId;. Please follow the links in the activity run Output from ADF Monitoring page to troubleshoot the run on HDInsight Spark cluster. Please contact HDInsight support team for further assistance.`

- **Причина.** Сбой задания в кластере HDInsight Spark.

- **Рекомендация**. Перейдите по ссылкам в выходных данных выполнения действия на странице мониторинга ADF, чтобы устранить неполадки в работе кластера HDInsight Spark. Обратитесь в службу поддержки HDInsight за дополнительной помощью.

   Дополнительные сведения о сборе данных полного журнала см. в разделе [Получение полного журнала пакетного задания](/rest/api/hdinsightspark/hdinsight-spark-batch-job#get-the-full-log-of-a-batch-job).

### <a name="error-code-2313"></a>Код ошибки: 2313

- **Сообщение.** `The batch with ID '%batchId;' was not found on Spark cluster. Open the Spark History UI and try to find it there. Contact HDInsight support for further assistance.`

- **Причина.** Пакет был удален в кластере HDInsight Spark.

- **Рекомендация**. Устраните неполадки в пакетах в кластере HDInsight Spark. За дополнительной помощью обратитесь в службу поддержки HDInsight. 

   Дополнительные сведения о том, как собрать данные полного журнала, см. в разделе [Получение полного журнала пакетного задания](/rest/api/hdinsightspark/hdinsight-spark-batch-job#get-the-full-log-of-a-batch-job). Предоставьте общий доступ к полному журналу службе поддержки HDInsight для получения дополнительной помощи.

### <a name="error-code-2328"></a>Код ошибки: 2328

- **Сообщение.** `Failed to create the on demand HDI cluster. Cluster or linked service name: '%clusterName;', error: '%message;'`

- **Причина.** В сообщении об ошибке должны быть указаны сведения о причине ошибки.

- **Рекомендация**. Сообщение об ошибке должно помочь устранить проблему.

### <a name="error-code-2329"></a>Код ошибки: 2329

- **Сообщение.** `Failed to delete the on demand HDI cluster. Cluster or linked service name: '%clusterName;', error: '%message;'`

- **Причина.** В сообщении об ошибке должны быть указаны сведения о причине ошибки.

- **Рекомендация**. Сообщение об ошибке должно помочь устранить проблему.

### <a name="error-code-2331"></a>Код ошибки: 2331

- **Сообщение.** `The file path should not be null or empty.`

- **Причина.** Указан пустой путь к файлу.

- **Рекомендация**. Укажите путь к существующему файлу.

### <a name="error-code-2340"></a>Код ошибки: 2340

- **Сообщение.** `HDInsightOnDemand linked service does not support execution via SelfHosted IR. Your IR name is '%IRName;'. Please select an Azure IR instead.`

- **Причина.** Связанная служба HDInsightOnDemand не поддерживает выполнение через среду IR SelfHosted.

- **Рекомендация**. Выберите Azure IR и повторите попытку.

### <a name="error-code-2341"></a>Код ошибки: 2341

- **Сообщение.** `HDInsight cluster URL '%clusterUrl;' is incorrect, it must be in URI format and the scheme must be 'https'.`

- **Причина.** Предоставленный URL-адрес имеет неправильный формат.

- **Рекомендация**. Исправьте URL-адрес кластера и повторите попытку.

### <a name="error-code-2342"></a>Код ошибки: 2342

- **Сообщение.** `Failed to connect to HDInsight cluster: '%errorMessage;'.`

- **Причина.** Предоставленные учетные данные неверны для кластера, либо возникла проблема конфигурации сети или проблема с подключением, либо в среде IR возникли проблемы с подключением к кластеру.

- **Рекомендация**. 
    1. Проверьте правильность учетных данных, открыв в браузере пользовательский интерфейс Ambari кластера HDInsight.
    1. Если кластер находится в виртуальной сети (VNet) и используется локальная среда IR, URL-адрес HDI должен быть частным URL-адресом в виртуальных сетях. В нем также должно быть указано "-int" после имени кластера.
    
       Например, измените `https://mycluster.azurehdinsight.net/` на `https://mycluster-int.azurehdinsight.net/`. Обратите внимание на `-int` после `mycluster`, но перед `.azurehdinsight.net`.
    1. Если кластер находится в виртуальной сети, используется локальная среда IR, а также использовался частный URL-адрес, но выполнить подключение все равно не удалось, значит у виртуальной машины, на которой установлена среда IR, возникли проблемы с подключением к HDI. 
    
       Подключитесь к виртуальной машине, на которой установлена среда IR, и откройте в браузере пользовательский интерфейс Ambari. Используйте частный URL-адрес для кластера. Это подключение должно работать из браузера. В противном случае обратитесь в службу поддержки HDInsight за дополнительной помощью.
    1. Если локальная среда IR не используется, кластер HDI должен быть общедоступным. Откройте в браузере пользовательский интерфейс Ambari и убедитесь, что он открывается. Если с кластером или службами на нем возникают какие-либо проблемы, обратитесь за помощью в службу поддержки HDInsight.

       URL-адрес кластера HDI, используемый в связанной службе ADF, должен быть доступен для среды IR службы ADF (локально размещенного или Azure), чтобы выполнить проверку подключения и чтобы выполнялись запуски. Это состояние можно проверить, открыв URL-адрес из браузера с виртуальной машины или с любого общедоступного компьютера.

### <a name="error-code-2343"></a>Код ошибки: 2343

- **Сообщение.** `User name and password cannot be null or empty to connect to the HDInsight cluster.`

- **Причина**: имя пользователя или пароль пусты.

- **Рекомендация**. Укажите правильные учетные данные для подключения к HDI и повторите попытку.

### <a name="error-code-2345"></a>Код ошибки: 2345

- **Сообщение.** `Failed to read the content of the hive script. Error: '%message;'`

- **Причина.** Файл сценария не существует или служба ADF не может подключиться к расположению сценария.

- **Рекомендация**. Убедитесь, что сценарий существует и что соответствующая связанная служба имеет правильные учетные данные для подключения.

### <a name="error-code-2346"></a>Код ошибки: 2346

- **Сообщение.** `Failed to create ODBC connection to the HDI cluster with error message '%message;'.`

- **Причина.** Служба ADF попыталась установить подключение ODBC к кластеру HDI, но произошла ошибка.

- **Рекомендация**. 

   1. Убедитесь, что вы правильно настроили подключение ODBC или Java Database Connectivity (JDBC).
      1. Если в JDBC вы используете ту же виртуальную сеть, это подключение можно получить из:<br>
        `Hive -> Summary -> HIVESERVER2 JDBC URL`
      1. Чтобы убедиться, что вы правильно настроили JDBC, ознакомьтесь со статьей [Отправка запросов в Apache Hive с помощью драйвера JDBC в HDInsight](../hdinsight/hadoop/apache-hadoop-connect-hive-jdbc-driver.md).
      1. Ознакомьтесь со статьей [Руководство по отправке запросов в Apache Hive с помощью PowerShell и драйвера ODBC](../hdinsight/interactive-query/apache-hive-query-odbc-driver-powershell.md), чтобы убедиться в правильности настройки ODB. 
   1. Убедитесь, что Hiveserver2, хранилище метаданных Hive и Hiveserver2 Interactive активны и работают. 
   1. Проверьте пользовательский интерфейс Ambari:
      1. Убедитесь, что все службы по-прежнему работают.
      1. В пользовательском интерфейсе Ambari на панели мониторинга проверьте раздел оповещений.
         1. Дополнительные сведения об оповещениях и разрешениях для них см. на странице [Мониторинг кластера и управление им](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.5.0/managing-and-monitoring-ambari/content/amb_predefined_alerts.html).
   1. Если с помощью этих шагов не удалось устранить проблему, обратитесь в службу поддержки Azure HDInsight.

### <a name="error-code-2347"></a>Код ошибки: 2347

- **Сообщение.** `Hive execution through ODBC failed with error message '%message;'.`

- **Причина.** Служба ADF передала сценарий Hive для выполнения в кластер HDI через подключение ODBC, и в сценарии возникла ошибка на HDI.

- **Рекомендация**. 

   1. Убедитесь, что вы правильно настроили подключение ODBC или Java Database Connectivity (JDBC).
      1. Если в JDBC вы используете ту же виртуальную сеть, это подключение можно получить из:<br>
        `Hive -> Summary -> HIVESERVER2 JDBC URL`
      1. Чтобы убедиться, что вы правильно настроили JDBC, ознакомьтесь со статьей [Отправка запросов в Apache Hive с помощью драйвера JDBC в HDInsight](../hdinsight/hadoop/apache-hadoop-connect-hive-jdbc-driver.md).
      1. Ознакомьтесь со статьей [Руководство по отправке запросов в Apache Hive с помощью PowerShell и драйвера ODBC](../hdinsight/interactive-query/apache-hive-query-odbc-driver-powershell.md), чтобы убедиться в правильности настройки ODB. 
   1. Убедитесь, что Hiveserver2, хранилище метаданных Hive и Hiveserver2 Interactive активны и работают. 
   1. Проверьте пользовательский интерфейс Ambari:
      1. Убедитесь, что все службы по-прежнему работают.
      1. В пользовательском интерфейсе Ambari на панели мониторинга проверьте раздел оповещений.
         1. Дополнительные сведения об оповещениях и разрешениях для них см. на странице [Мониторинг кластера и управление им](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.5.0/managing-and-monitoring-ambari/content/amb_predefined_alerts.html).
   1. Если с помощью этих шагов не удалось устранить проблему, обратитесь в службу поддержки Azure HDInsight.

### <a name="error-code-2348"></a>Код ошибки: 2348

- **Сообщение.** `The main storage has not been initialized. Please check the properties of the storage linked service in the HDI linked service.`

- **Причина.** Свойства связанной службы хранения установлены неправильно.

- **Рекомендация**. В связанной службе основного хранилища для действий HDI поддерживаются только полные строки подключения. Убедитесь, что вы не используете авторизацию или приложения MSI.

### <a name="error-code-2350"></a>Код ошибки: 2350

- **Сообщение.** `Failed to prepare the files for the run '%jobId;'. HDI cluster: '%cluster;', Error: '%errorMessage;'`

- **Причина.** Учетные данные, предоставленные для подключения к хранилищу, где должны находиться файлы, неверны или файлы там не существуют.

- **Рекомендация**. Эта ошибка возникает, когда служба ADF готовится к выполнению действий HDI и пытается перед отправкой задания в HDI скопировать файлы в основное хранилище. Убедитесь, что файлы существуют в указанном расположении и подключение к хранилищу настроено правильно. Действия HDI службы ADF не поддерживают проверку подлинности MSI для учетных записей хранения, связанных с действиями HDI. Убедитесь, что эти связанные службы имеют полные ключи или используют Azure Key Vault.

### <a name="error-code-2351"></a>Код ошибки: 2351

- **Сообщение.** `Could not open the file '%filePath;' in container/fileSystem '%container;'.`

- **Причина.** Файл не существует по указанному пути.

- **Рекомендация**. Проверьте, существует ли файл на самом деле и имеет ли связанная служба со сведениями о подключении, указывающими на этот файл, правильные учетные данные.

### <a name="error-code-2352"></a>Код ошибки: 2352

- **Сообщение.** `The file storage has not been initialized. Please check the properties of the file storage linked service in the HDI activity.`

- **Причина.** Свойства связанной службы хранилища файлов установлены неправильно.

- **Рекомендация**. Убедитесь, что свойства связанной службы хранилища файлов настроены правильно.

### <a name="error-code-2353"></a>Код ошибки: 2353

- **Сообщение.** `The script storage has not been initialized. Please check the properties of the script storage linked service in the HDI activity.`

- **Причина.** Свойства связанной службы хранилища сценариев установлены неправильно.

- **Рекомендация**. Убедитесь, что свойства связанной службы хранилища сценариев настроены правильно.

### <a name="error-code-2354"></a>Код ошибки: 2354

- **Сообщение.** `The storage linked service type '%linkedServiceType;' is not supported for '%executorType;' activities for property '%linkedServicePropertyName;'.`

- **Причина.** Тип связанной службы хранилища не поддерживается действием.

- **Рекомендация**. Убедитесь, что выбранная связанная служба имеет один из поддерживаемых типов для действия. Действия HDI поддерживают связанные службы AzureBlobStorage и AzureBlobFSStorage.

   Дополнительные сведения см. в статье [Сравнение вариантов хранения для использования с кластерами Azure HDInsight](../hdinsight/hdinsight-hadoop-compare-storage-options.md).

### <a name="error-code-2355"></a>Код ошибки: 2355

- **Сообщение.** `The '%value' provided for commandEnvironment is incorrect. The expected value should be an array of strings where each string has the format CmdEnvVarName=CmdEnvVarValue.`

- **Причина.** Указано неверное значение для `commandEnvironment`.

- **Рекомендация**. Убедитесь, что предоставленное значение похоже на:
 
    ``` \"commandEnvironment\": [
    \"variableName=variableValue\"
    ]
    ```

    Также убедитесь, что каждая переменная появляется в списке только один раз.

### <a name="error-code-2356"></a>Код ошибки: 2356

- **Сообщение.** `The commandEnvironment already contains a variable named '%variableName;'.`

- **Причина.** Указано неверное значение для `commandEnvironment`.

- **Рекомендация**. Убедитесь, что предоставленное значение похоже на:
 
    ``` \"commandEnvironment\": [
    \"variableName=variableValue\"
    ]
    ```

    Также убедитесь, что каждая переменная появляется в списке только один раз.

### <a name="error-code-2357"></a>Код ошибки: 2357

- **Сообщение.** `The certificate or password is wrong for ADLS Gen 1 storage.`

- **Причина.** Предоставлены неправильные учетные данные.

- **Рекомендация**. Убедитесь, что сведения о подключении в ADLS 1-го поколения связаны со службой и что тестирование подключения прошло успешно.

### <a name="error-code-2358"></a>Код ошибки: 2358

- **Сообщение.** `The value '%value;' for the required property 'TimeToLive' in the on demand HDInsight linked service '%linkedServiceName;' has invalid format. It should be a timespan between '00:05:00' and '24:00:00'.`

- **Причина.** Предоставленное значение для обязательного свойства `TimeToLive` имеет недопустимый формат. 

- **Рекомендация**. Обновите значение до рекомендуемого диапазона и повторите попытку.

### <a name="error-code-2359"></a>Код ошибки: 2359

- **Сообщение.** `The value '%value;' for the property 'roles' is invalid. Expected types are 'zookeeper', 'headnode', and 'workernode'.`

- **Причина.** Предоставленное значение для свойства `roles` является недопустимым.

- **Рекомендация**. Обновите значение, чтобы оно стало одним из предложений, и повторите попытку.

### <a name="error-code-2360"></a>Код ошибки: 2360

- **Сообщение.** `The connection string in HCatalogLinkedService is invalid. Encountered an error while trying to parse: '%message;'.`

- **Причина.** Предоставленная строка подключения для `HCatalogLinkedService` является недопустимой.

- **Рекомендация**. Измените значение на правильную строку подключения SQL Azure и повторите попытку.

### <a name="error-code-2361"></a>Код ошибки: 2361

- **Сообщение.** `Failed to create on demand HDI cluster. Cluster name is '%clusterName;'.`

- **Причина.** Произошел сбой при создании кластера, и служба ADF не получила сообщение об ошибке от службы HDInsight.

- **Рекомендация**. Откройте портал Azure и попробуйте найти ресурс HDI с указанным именем, а затем проверьте состояние подготовки. Обратитесь в службу поддержки HDInsight за дополнительной помощью.

### <a name="error-code-2362"></a>Код ошибки: 2362

- **Сообщение.** `Only Azure Blob storage accounts are supported as additional storages for HDInsight on demand linked service.`

- **Причина.** Предоставленное дополнительное хранилище не являлось хранилищем BLOB-объектов Azure.

- **Рекомендация**. Предоставьте учетную запись хранилища BLOB-объектов Azure в качестве дополнительного хранилища для связанной службы HDInsight по запросу.

### <a name="ssl-error-when-adf-linked-service-using-hdinsight-esp-cluster"></a>Ошибка SSL при связанной службе ADF с помощью кластера HDInsight ESP

- **Сообщение.** `Failed to connect to HDInsight cluster: 'ERROR [HY000] [Microsoft][DriverSupport] (1100) SSL certificate verification failed because the certificate is missing or incorrect.`

- **Причина**. скорее всего, эта неполадка связана с надежным хранилищем системы.

- **Решение**. можно перейти по пути **Microsoft Integration RUNTIME\4.0\SHARED\ODBC Дриверс\микрософт Hive ODBC дривер\либ** и открыть DriverConfiguration64.exe, чтобы изменить этот параметр.

    ![Снимите флажок использовать системное хранилище доверия](./media/connector-troubleshoot-guide/system-trust-store-setting.png)

## <a name="web-activity"></a>Веб-действие

### <a name="error-code-2128"></a>Код ошибки: 2128

- **Сообщение.** `No response from the endpoint. Possible causes: network connectivity, DNS failure, server certificate validation or timeout.`

- **Причина.** Эта проблема связана с сетевым подключением, сбоем DNS, проверкой сертификата сервера или превышением времени ожидания.

- **Рекомендация**. Убедитесь, что конечная точка, к которой вы пытаетесь подключиться, реагирует на запросы. Вы можете использовать такие инструменты, как **Fiddler или Postman**.

### <a name="error-code-2108"></a>Код ошибки: 2108

- **Сообщение.** `Error calling the endpoint '%url;'. Response status code: '%code;'`

- **Причина.** Не удалось выполнить запрос из-за такой проблемы, как сетевое подключение, сбой DNS, проверка сертификата сервера или превышение времени ожидания.

- **Рекомендация**. Используйте Fiddler или Postman для проверки запроса.

#### <a name="more-details"></a>Дополнительные сведения
Чтобы использовать **Fiddler** для создания HTTP-сеанса отслеживаемого веб-приложения, выполните приведенные ниже действия.

1. Скачайте, установите и откройте [Fiddler](https://www.telerik.com/download/fiddler).

1. Если веб-приложение использует протокол HTTPS, последовательно выберите **Инструменты** > **Параметры Fiddler** > **HTTPS**.

   1. На вкладке HTTPS выберите **Capture HTTPS CONNECTs** (Захват подключений HTTPS) и **Decrypt HTTPS traffic** (Расшифровка трафика HTTPS).

      ![Параметры Fiddler](media/data-factory-troubleshoot-guide/fiddler-options.png)

1. Если в приложении используются TLS/SSL-сертификаты, добавьте сертификат Fiddler на устройство.

   Справочные ресурсы: **Инструменты** > **Параметры Fiddler** > **HTTPS** > **Действия** > **Export Root Certificate to Desktop** (Экспортировать корневой сертификат на рабочий стол).

1. Отключите запись, выбрав **Файл** > **Capture Traffic** (Запись трафика). Или нажмите клавишу **F12**.

1. Очистите кэш браузера, чтобы удалить все кэшированные элементы и скачать их снова.

1. Создайте запрос:

1. Выберите вкладку **Композитор**.

   1. Задайте метод HTTP и URL-адрес.
 
   1. При необходимости добавьте заголовки и текст запроса.

   1. Нажмите кнопку **Выполнить**.

1. Снова включите запись трафика и завершите проблемную транзакцию на странице.

1. Справочные ресурсы: **Файл** > **Сохранить** > **Все сеансы**.

Дополнительные сведения см. на странице [начала работы с Fiddler](https://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/ConfigureFiddler).

## <a name="general"></a>Общие сведения

### <a name="activity-stuck-issue"></a>Проблемы с задержанным действием

Если вы следите за тем, чтобы действие выполнялось намного дольше обычного выполнения, не выполняя никаких действий, может произойти зависание. Вы можете попробовать отменить его и повторить попытку, чтобы узнать, помогает ли он. Если это действие копирования, можно узнать о мониторинге производительности и устранении неполадок при [устранении неполадок, связанных с действиями копирования](copy-activity-performance-troubleshooting.md). Если это поток данных, ознакомьтесь с руководством по настройке [производительности потоков данных](concepts-data-flow-performance.md) .

### <a name="payload-is-too-large"></a>Полезная нагрузка слишком велика

**Сообщение об ошибке:**`The payload including configurations on activity/dataSet/linked service is too large. Please check if you have settings with very large value and try to reduce its size.`

**Причина:** Полезные данные для каждого выполнения действия включают конфигурацию действия, связанные наборы данных и конфигурации связанных служб, если таковые имеются, и небольшую часть системных свойств, созданных для каждого типа действия. Ограничение такого размера полезной нагрузки составляет 896 КБ, как упоминалось в разделе [ограничения фабрики данных](../azure-resource-manager/management/azure-subscription-service-limits.md#data-factory-limits) .

**Рекомендации:** Это может быть вызвано тем, что вы передаете одно или несколько значений больших параметров из вышестоящего или внешнего выходного действия, особенно если передаются фактические данные между действиями в потоке управления. Проверьте, можно ли уменьшить размер значений больших параметров, или настройте логику конвейера, чтобы избежать передачи таких значений между действиями и обработки их внутри действия.

## <a name="next-steps"></a>Дальнейшие действия

Для получения дополнительных сведений об устранении неполадок воспользуйтесь следующими ресурсами:

* [Блог о Фабрике данных](https://azure.microsoft.com/blog/tag/azure-data-factory/)
* [Запросы на добавление функции в Фабрику данных](https://feedback.azure.com/forums/270578-data-factory)
* [Форум Stack Overflow по Фабрике данных](https://stackoverflow.com/questions/tagged/azure-data-factory)
* [Сведения о Фабрике данных в Twitter](https://twitter.com/hashtag/DataFactory)
* [Видео по Azure](https://azure.microsoft.com/resources/videos/index/)
* [Раздел вопросов и ответов на сайте Майкрософт](/answers/topics/azure-data-factory.html)
