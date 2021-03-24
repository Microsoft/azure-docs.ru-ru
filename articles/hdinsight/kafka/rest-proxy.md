---
title: Прокси-сервер REST для Apache Kafka (Azure HDInsight)
description: Сведения о том, как выполнять в Azure HDInsight операции Apache Kafka с помощью прокси-сервера REST для Kafka.
ms.service: hdinsight
ms.topic: how-to
ms.custom: has-adal-ref, devx-track-python
ms.date: 04/03/2020
ms.openlocfilehash: bc3cbe5d0d7cf5e5a78112ae5df63ebb88a97f5a
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864845"
---
# <a name="interact-with-apache-kafka-clusters-in-azure-hdinsight-using-a-rest-proxy"></a>Взаимодействие с кластерами Apache Kafka в Azure HDInsight через прокси-сервер REST

Прокси-сервер Kafka RESTFUL позволяет взаимодействовать с кластером Kafka через REST API по протоколу HTTPS. Это действие означает, что клиенты Kafka могут находиться за пределами виртуальной сети. Клиенты могут выполнять простые и безопасные вызовы HTTPS в кластер Kafka, а не полагаться на библиотеки Kafka. В этой статье показано, как создать кластер Kafka с поддержкой прокси-сервера REST. Также вы получите пример кода для вызовов к прокси-серверу REST.

## <a name="rest-api-reference"></a>Справочник по REST API

Сведения об операциях, поддерживаемых REST API Kafka, см. в [справочнике по API прокси-сервера REST для Kafka в Azure HDInsight](/rest/api/hdinsight-kafka-rest-proxy).

## <a name="background"></a>Историческая справка

:::image type="content" source="./media/rest-proxy/rest-proxy-architecture.png" alt-text="Архитектура прокси-сервера REST для Kafka" border="false":::

Полную спецификацию операций, поддерживаемых API прокси-сервера REST для Apache Kafka, см. [здесь](/rest/api/hdinsight-kafka-rest-proxy).

### <a name="rest-proxy-endpoint"></a>Конечная точка прокси-сервера REST

При создании кластера Kafka в HDInsight с прокси-сервером REST для этого кластера создается новая общедоступная конечная точка, которую можно найти на странице **Свойства** для кластера HDInsight на портале Azure.

### <a name="security"></a>Безопасность

Доступом к прокси-серверу REST для Kafka можно управлять через группы безопасности Azure Active Directory. При создании кластера Kafka предоставьте группе безопасности AAD доступ к конечной точке REST. Клиенты Kafka, которым нужен доступ к прокси-серверу REST, должны быть зарегистрированы в этой группе с ролью владельца. Владелец группы может зарегистрироваться через портал или с помощью PowerShell.

Для запросов к конечной точке прокси-сервера REST клиентским приложениям нужно получить токен OAuth. Этот же токен используется для проверки членства в группе безопасности. Изучите ниже [пример клиентского приложения](#client-application-sample), в котором демонстрируется получение токена OAuth. Клиентское приложение передает маркер OAuth в запросе HTTPS на прокси-сервер RESTFUL.

> [!NOTE]
> В статье [Управление доступом к приложениям и ресурсам с помощью групп Azure Active Directory](../../active-directory/fundamentals/active-directory-manage-groups.md) вы найдете дополнительные сведения о группах безопасности AAD. Дополнительные сведения о работе токенов OAuth см. в статье [Авторизация доступа к веб-приложениям Azure Active Directory с помощью потока предоставления кода OAuth 2.0](../../active-directory/azuread-dev/v1-protocols-oauth-code.md).

## <a name="kafka-rest-proxy-with-network-security-groups"></a>Работа прокси-сервера REST для Kafka с группами безопасности сети
Если вы используете собственную виртуальную сеть и управляете сетевым трафиком с помощью групп безопасности сети, разрешите **входящий трафик** через порт **9400** в дополнение к стандартному порту 443. Это обеспечит доступность прокси-сервера REST для Kafka.

## <a name="prerequisites"></a>Предварительные требования

1. зарегистрировать приложение в Azure AD; Клиентские приложения, которые вы создаете для взаимодействия с прокси-сервером REST для Kafka, будут использовать эти значения идентификатора и секрета приложения для проверки подлинности в Azure.

1. Создайте группу безопасности Azure AD. Добавьте приложение, которое вы зарегистрировали в Azure AD, в группу безопасности в качестве **члена** группы. Эта группа безопасности будет использоваться для управления тем, какие приложения могут взаимодействовать с прокси-сервером REST. Ознакомьтесь со статьей [Создание простой группы и добавление в нее участников с помощью Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md), чтобы получить сведения о создании групп AAD.

    Убедитесь, что для этой группы установлен тип **Безопасность**.
    :::image type="content" source="./media/rest-proxy/rest-proxy-group.png" alt-text="Группа безопасности" border="true":::

    Убедитесь, что приложение является членом группы.
    :::image type="content" source="./media/rest-proxy/rest-proxy-membergroup.png" alt-text="Проверка членства" border="true":::

## <a name="create-a-kafka-cluster-with-rest-proxy-enabled"></a>Создание кластера Kafka с включенным прокси-сервером REST

Для описанных ниже действий используется портал Azure. Пример работы с Azure CLI см. в статье [Создание кластера Apache Kafka с прокси-сервером REST с помощью Azure CLI](tutorial-cli-rest-proxy.md).

1. В процессе создания кластера Kafka установите на вкладке **Безопасность и сеть** флажок **Включить прокси-сервер REST для Kafka**.

     :::image type="content" source="./media/rest-proxy/azure-portal-cluster-security-networking-kafka-rest.png" alt-text="На снимке экрана показана страница Создание кластера H D Insights с выбранной безопасностью и сетью." border="true":::

1. Щелкните **Выбор группы безопасности**. Из предложенного списка выберите ту группу безопасности, которая должна иметь доступ к прокси-серверу REST. Чтобы найти нужную группу безопасности, можно воспользоваться полем поиска. В нижней части страницы нажмите кнопку **Выбрать**.

     :::image type="content" source="./media/rest-proxy/azure-portal-cluster-security-networking-kafka-rest2.png" alt-text="На снимке экрана показана страница Создание кластера H D Insights с возможностью выбора группы безопасности." border="true":::

1. Выполните остальные действия для создания кластера, как описано в статье [Создание кластера Apache Kafka в Azure HDInsight с помощью портала Azure](./apache-kafka-get-started.md).

1. После создания кластера перейдите к свойствам кластера, чтобы записать URL-адрес прокси-сервера REST для Kafka.

     :::image type="content" source="./media/rest-proxy/apache-kafka-rest-proxy-view-proxy-url.png" alt-text="Просмотр URL-адреса прокси-сервера REST" border="true":::

## <a name="client-application-sample"></a>Пример клиентского приложения

Приведенный ниже код Python можно использовать для взаимодействия с прокси-сервером REST в кластере Kafka. Чтобы использовать этот пример кода, выполните следующие шаги:

1. Сохраните пример кода на компьютере, где установлен Python.
1. Установите необходимые зависимости Python, выполнив команду `pip3 install msal`.
1. Измените раздел **Configure these properties** (Настройте эти свойства) в коде, указав для следующих свойства значения из вашей среды:

    |Свойство |Описание |
    |---|---|
    |Tenant ID|Клиент Azure, на котором находится ваша подписка.|
    |Идентификатор клиента|Идентификатор приложения, зарегистрированного в группе безопасности.|
    |Секрет клиента|Секрет приложения, зарегистрированного в группе безопасности.|
    |Kafkarest_endpoint|Получите это значение на вкладке **Свойства** в разделе сводной информации о кластере, как описано в [разделе о развертывании](#create-a-kafka-cluster-with-rest-proxy-enabled). Оно должно быть указано в формате `https://<clustername>-kafkarest.azurehdinsight.net`.|

1. В командной строке выполните файл Python с помощью команды `sudo python3 <filename.py>`.

Этот код делает следующее:

1. Получает токен OAuth из Azure AD.
1. Демонстрирует выполнение запроса к прокси-серверу REST для Kafka.

Дополнительные сведения о получении маркеров OAuth в Python см. в разделе [Python AuthenticationContext Class](/python/api/adal/adal.authentication_context.authenticationcontext). Вы можете заметить некоторую задержку в отображении состояния `topics` при создании или удалении не через прокси-сервер REST для Kafka. Она обусловлена обновлением кэша. Улучшено поле **значения** API-интерфейса производителя. Теперь он принимает объекты JSON и любую сериализованную форму.

```python
#Required python packages
#pip3 install msal

import json
import msal
import random
import requests
import string
import sys
import time

def get_random_string():
    letters = string.ascii_letters
    random_string = ''.join(random.choice(letters) for i in range(7))

    return random_string


#--------------------------Configure these properties-------------------------------#
# Tenant ID for your Azure Subscription
tenant_id = 'ABCDEFGH-1234-1234-1234-ABCDEFGHIJKL'
# Your Client Application Id
client_id = 'XYZABCDE-1234-1234-1234-ABCDEFGHIJKL'
# Your Client Credentials
client_secret = 'password'
# kafka rest proxy -endpoint
kafkarest_endpoint = "https://<clustername>-kafkarest.azurehdinsight.net"
#--------------------------Configure these properties-------------------------------#

# Get access token
# Scope
scope = 'https://hib.azurehdinsight.net/.default'
#Authority
authority = 'https://login.microsoftonline.com/' + tenant_id

app = msal.ConfidentialClientApplication(
    client_id , client_secret, authority,
    #cache - For details on how look at this example: https://github.com/Azure-Samples/ms-identity-python-webapp/blob/master/app.py
)

# The pattern to acquire a token looks like this.
result = None
result = app.acquire_token_for_client(scopes=[scope])
accessToken = result['access_token']
verify_https = True
request_timeout = 10

# Print access token
print("Access token: " + accessToken)

# API format
api_version = 'v1'
api_format = kafkarest_endpoint + '/{api_version}/{rest_api}'
get_topic_api = 'metadata/topics'
topic_api_format = 'topics/{topic_name}'
producer_api_format = 'producer/topics/{topic_name}'
consumer_api_format = 'consumer/topics/{topic_name}/partitions/{partition_id}/offsets/{offset}?count={count}'  # by default count = 1

# Request header
headers = {
    'Authorization': 'Bearer ' + accessToken,
    'Content-type': 'application/json'          # set Content-type to 'application/json'
}

# New topic
new_topic = 'hello_topic_' + get_random_string()
print("Topic " + new_topic + " is going to be used for demo.")

topics = []

# Create a  new topic
# Example of topic config
topic_config = {
    "partition_count": 1,
    "replication_factor": 1,
    "topic_properties": {
        "retention.ms": 604800000,
        "min.insync.replicas": "1"
    }
}

create_topic_url = api_format.format(api_version=api_version, rest_api=topic_api_format.format(topic_name=new_topic))
response = requests.put(create_topic_url, headers=headers, json=topic_config, timeout=request_timeout, verify=verify_https)
print(response.content)

if response.ok:
    while new_topic not in topics:
        print("The new topic " + new_topic + " is not visible yet. sleep 30 seconds...")
        time.sleep(30)
        # List Topic
        get_topic_url = api_format.format(api_version=api_version, rest_api=get_topic_api)

        response = requests.get(get_topic_url, headers={'Authorization': 'Bearer ' + accessToken}, timeout=request_timeout, verify=verify_https)
        topic_list = response.json()
        topics = topic_list.get("topics", [])
else:
    print("Topic " + new_topic + " was created. Exit.")
    sys.exit(1)

# Produce messages to new_topic
# Example payload of Producer REST API
payload_json = {
    "records": [
        {
            "key": "key1",
            "value": "**********"         # A string                              
        },
        {
            "partition": 0,
            "value": 5                    # An integer
        },
        {
            "value": 3.14                 # A floating number
        },
        {
            "value": {                    # A JSON object
                "id": 1,
                "name": "HDInsight Kafka REST proxy"
            }
        },
        {
            "value": [                    # A list of JSON objects
                {
                    "id": 1,
                    "name": "HDInsight Kafka REST proxy 1"
                },
                {
                    "id": 2,
                    "name": "HDInsight Kafka REST proxy 2"
                },
                {
                    "id": 3,
                    "name": "HDInsight Kafka REST proxy 3"
                }
            ]
        },
        {
            "value": {                  # A nested JSON object
                "group id": 1,
                "HDI Kafka REST": {
                    "id": 1,
                    "name": "HDInsight Kafka REST proxy 1"
                },
                "HDI Kafka REST server info": {
                    "id": 1,
                    "name": "HDInsight Kafka REST proxy 1",
                    "servers": [
                        {
                            "server id": 1,
                            "server name": "HDInsight Kafka REST proxy server 1"
                        },
                        {
                            "server id": 2,
                            "server name": "HDInsight Kafka REST proxy server 2"
                        },
                        {
                            "server id": 3,
                            "server name": "HDInsight Kafka REST proxy server 3"
                        }
                    ]
                }
            }
        }
    ]
}

print("Payloads in a Producer request: \n", payload_json)
producer_url = api_format.format(api_version=api_version, rest_api=producer_api_format.format(topic_name=new_topic))
response = requests.post(producer_url, headers=headers, json=payload_json, timeout=request_timeout, verify=verify_https)
print(response.content)

# Consume messages from the topic
partition_id = 0
offset = 0
count = 2

while True:
    consumer_url = api_format.format(api_version=api_version, rest_api=consumer_api_format.format(topic_name=new_topic, partition_id=partition_id, offset=offset, count=count))
    print("Consuming " + str(count) + " messages from offset " + str(offset))

    response = requests.get(consumer_url, headers=headers, timeout=request_timeout, verify=verify_https)

    if response.ok:
        messages = response.json()
        print("Consumed messages: \n" + json.dumps(messages, indent=2))
        next_offset = response.headers.get("NextOffset")
        if offset == next_offset or not messages.get("records", []):
            print("Consumer caught up with producer. Exit for now...")
            break

        offset = next_offset

    else:
        print("Error " + str(response.status_code))
        break
        
# List partitions
get_partitions_url = api_format.format(api_version=api_version, rest_api=partitions_api_format.format(topic_name=new_topic))
print("Fetching partitions from  " + get_partitions_url)

response = requests.get(get_partitions_url, headers={'Authorization': 'Bearer ' + accessToken}, timeout=request_timeout, verify=verify_https)
partition_list = response.json()
print("Partition list: \n" + json.dumps(partition_list, indent=2))

# List a partition
get_partition_url = api_format.format(api_version=api_version, rest_api=partition_api_format.format(topic_name=new_topic, partition_id=partition_id))
print("Fetching metadata of a partition from  " + get_partition_url)

response = requests.get(get_partition_url, headers={'Authorization': 'Bearer ' + accessToken}, timeout=request_timeout, verify=verify_https)
partition = response.json()
print("Partition metadata: \n" + json.dumps(partition, indent=2))

```

Ниже приведен еще один пример того, как можно получить токен из Azure для прокси-сервера REST с помощью команды curl. **Обратите внимание, что при получении токена нужно указать `scope=https://hib.azurehdinsight.net/.default`.**

```cmd
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=<clientid>&client_secret=<clientsecret>&grant_type=client_credentials&scope=https://hib.azurehdinsight.net/.default' 'https://login.microsoftonline.com/<tenantid>/oauth2/v2.0/token'
```

## <a name="next-steps"></a>Дальнейшие действия

* [Справочные документы по API прокси-сервера REST для Kafka](/rest/api/hdinsight-kafka-rest-proxy/)
