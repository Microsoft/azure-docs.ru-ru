---
title: Управление Azure Data Lake Analytics с помощью Python
description: В этой статье описано, как с помощью Python управлять учетными записями, источниками данных, пользователями и заданиями Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/08/2018
ms.custom: devx-track-python
ms.openlocfilehash: fda09047d138e404eeca87ed3eba9bb72ff62d56
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92220233"
---
# <a name="manage-azure-data-lake-analytics-using-python"></a>Управление Azure Data Lake Analytics с помощью Python
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

В этой статье описано, как управлять учетными записями, источниками данных, пользователями и заданиями Azure Data Lake Analytics с помощью Python.

## <a name="supported-python-versions"></a>Поддерживаемые версии Python

* Используйте 64-разрядную версию Python.
* Можно использовать стандартную распространяемую версию Python, опубликованную в **[разделе загрузок на сайте Python.org](https://www.python.org/downloads/)**. 
* Многим разработчикам удобнее использовать **[распространяемую версию Anaconda Python](https://www.anaconda.com/download/)**.  
* При написании данной статьи использовалась версия 3.6 стандартного распространяемого пакета Python

## <a name="install-azure-python-sdk"></a>Установка пакета Azure SDK для Python

Установите следующие модули:

* Модуль **Azure-руководства-Resource** включает другие модули azure для Active Directory и т. д.
* Модуль **Azure-Lake-Store** включает Azure Data Lake Store операций файловой системы. 
* Модуль **Azure-Management-Lake-Store** включает Azure Data Lake Store операции управления учетными записями.
* Модуль **azure-mgmt-datalake-analytics** содержит операции Azure Data Lake Analytics. 

Во-первых, убедитесь, что установлена актуальная версия компонента `pip`, выполнив следующую команду:

```console
python -m pip install --upgrade pip
```

При написании этого документа использовался компонент `pip version 9.0.1`.

Чтобы установить модули, используйте следующие команды `pip` в командной строке.

```console
pip install azure-mgmt-resource
pip install azure-datalake-store
pip install azure-mgmt-datalake-store
pip install azure-mgmt-datalake-analytics
```

## <a name="create-a-new-python-script"></a>Создание сценария Python

Скопируйте приведенный ниже код и вставьте его в сценарий.

```python
# Use this only for Azure AD service-to-service authentication
#from azure.common.credentials import ServicePrincipalCredentials

# Use this only for Azure AD end-user authentication
#from azure.common.credentials import UserPassCredentials

# Required for Azure Resource Manager
from azure.mgmt.resource.resources import ResourceManagementClient
from azure.mgmt.resource.resources.models import ResourceGroup

# Required for Azure Data Lake Store account management
from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
from azure.mgmt.datalake.store.models import DataLakeStoreAccount

# Required for Azure Data Lake Store filesystem management
from azure.datalake.store import core, lib, multithread

# Required for Azure Data Lake Analytics account management
from azure.mgmt.datalake.analytics.account import DataLakeAnalyticsAccountManagementClient
from azure.mgmt.datalake.analytics.account.models import DataLakeAnalyticsAccount, DataLakeStoreAccountInformation

# Required for Azure Data Lake Analytics job management
from azure.mgmt.datalake.analytics.job import DataLakeAnalyticsJobManagementClient
from azure.mgmt.datalake.analytics.job.models import JobInformation, JobState, USqlJobProperties

# Required for Azure Data Lake Analytics catalog management
from azure.mgmt.datalake.analytics.catalog import DataLakeAnalyticsCatalogManagementClient

# Use these as needed for your application
import logging
import getpass
import pprint
import uuid
import time
```

Запустите этот сценарий, чтобы проверить, можно ли импортировать модули.

## <a name="authentication"></a>Аутентификация

### <a name="interactive-user-authentication-with-a-pop-up"></a>Интерактивная аутентификация пользователей с помощью всплывающего окна

Этот метод не поддерживается.

### <a name="interactive-user-authentication-with-a-device-code"></a>Интерактивная аутентификация пользователей с помощью кода устройства

```python
user = input(
    'Enter the user to authenticate with that has permission to subscription: ')
password = getpass.getpass()
credentials = UserPassCredentials(user, password)
```

### <a name="noninteractive-authentication-with-spi-and-a-secret"></a>Неинтерактивная аутентификация с помощью SPI и секрета

```python
credentials = ServicePrincipalCredentials(
    client_id='FILL-IN-HERE', secret='FILL-IN-HERE', tenant='FILL-IN-HERE')
```

### <a name="noninteractive-authentication-with-api-and-a-certificate"></a>Неинтерактивная аутентификация с помощью API и секрета

Этот метод не поддерживается.

## <a name="common-script-variables"></a>Общие переменные сценария

Эти переменные используются в примерах.

```python
subid = '<Azure Subscription ID>'
rg = '<Azure Resource Group Name>'
location = '<Location>'  # i.e. 'eastus2'
adls = '<Azure Data Lake Store Account Name>'
adla = '<Azure Data Lake Analytics Account Name>'
```

## <a name="create-the-clients"></a>Создание клиентов

```python
resourceClient = ResourceManagementClient(credentials, subid)
adlaAcctClient = DataLakeAnalyticsAccountManagementClient(credentials, subid)
adlaJobClient = DataLakeAnalyticsJobManagementClient(
    credentials, 'azuredatalakeanalytics.net')
```

## <a name="create-an-azure-resource-group"></a>Создание группы ресурсов Azure

```python
armGroupResult = resourceClient.resource_groups.create_or_update(
    rg, ResourceGroup(location=location))
```

## <a name="create-data-lake-analytics-account"></a>Создание учетной записи аналитики озера данных

Сначала создайте учетную запись хранилища.

```python
adlsAcctResult = adlsAcctClient.account.create(
    rg,
    adls,
    DataLakeStoreAccount(
        location=location)
    )
).wait()
```
Затем создайте учетную запись ADLA, которая использует это хранилище.

```python
adlaAcctResult = adlaAcctClient.account.create(
    rg,
    adla,
    DataLakeAnalyticsAccount(
        location=location,
        default_data_lake_store_account=adls,
        data_lake_store_accounts=[DataLakeStoreAccountInformation(name=adls)]
    )
).wait()
```

## <a name="submit-a-job"></a>Отправка задания

```python
script = """
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
"""

jobId = str(uuid.uuid4())
jobResult = adlaJobClient.job.create(
    adla,
    jobId,
    JobInformation(
        name='Sample Job',
        type='USql',
        properties=USqlJobProperties(script=script)
    )
)
```

## <a name="wait-for-a-job-to-end"></a>Ожидание завершения задания

```python
jobResult = adlaJobClient.job.get(adla, jobId)
while(jobResult.state != JobState.ended):
    print('Job is not yet done, waiting for 3 seconds. Current state: ' +
          jobResult.state.value)
    time.sleep(3)
    jobResult = adlaJobClient.job.get(adla, jobId)

print('Job finished with result: ' + jobResult.result.value)
```

## <a name="list-pipelines-and-recurrences"></a>Вывод списка конвейеров и повторений
В зависимости от того, вложены ли в задания метаданные конвейеров или повторений, можно отобразить список конвейеров и повторений.

```python
pipelines = adlaJobClient.pipeline.list(adla)
for p in pipelines:
    print('Pipeline: ' + p.name + ' ' + p.pipelineId)

recurrences = adlaJobClient.recurrence.list(adla)
for r in recurrences:
    print('Recurrence: ' + r.name + ' ' + r.recurrenceId)
```

## <a name="manage-compute-policies"></a>Управление политиками вычислений

Объект DataLakeAnalyticsAccountManagementClient предоставляет методы для управления политиками вычислений для учетной записи Data Lake Analytics.

### <a name="list-compute-policies"></a>Вывод списка политик вычислений

Следующий код извлекает список политик вычислений для учетной записи Data Lake Analytics.

```python
policies = adlaAccountClient.computePolicies.listByAccount(rg, adla)
for p in policies:
    print('Name: ' + p.name + 'Type: ' + p.objectType + 'Max AUs / job: ' +
          p.maxDegreeOfParallelismPerJob + 'Min priority / job: ' + p.minPriorityPerJob)
```

### <a name="create-a-new-compute-policy"></a>Создание новой политики вычислений

Следующий код создает новую политику вычислений для учетной записи Data Lake Analytics, задавая максимальное количество AU, доступных для указанного пользователя, равным 50 и минимальный приоритет задания равным 250.

```python
userAadObjectId = "3b097601-4912-4d41-b9d2-78672fc2acde"
newPolicyParams = ComputePolicyCreateOrUpdateParameters(
    userAadObjectId, "User", 50, 250)
adlaAccountClient.computePolicies.createOrUpdate(
    rg, adla, "GaryMcDaniel", newPolicyParams)
```

## <a name="next-steps"></a>Дальнейшие действия

- Для просмотра учебника с помощью других средств используйте вкладки-селекторы в верхней части страницы.
- Для знакомства с U-SQL см. статью о [начале работы с языком U-SQL для Azure Data Lake Analytics](data-lake-analytics-u-sql-get-started.md).
- Задачи управления описываются в руководстве по [управлению Azure Data Lake Analytics с помощью портала Azure](data-lake-analytics-manage-use-portal.md).

