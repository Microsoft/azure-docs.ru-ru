---
title: Создание кластеров Apache Hadoop с помощью REST API Azure в Azure
description: Узнайте, как создавать кластеры HDInsight, отправив шаблоны Azure Resource Manager в Azure REST API.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-azurecli
ms.date: 12/10/2019
ms.openlocfilehash: fa5675104d9614e1bd917585ea537c92dddd88cc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98945786"
---
# <a name="create-apache-hadoop-clusters-using-the-azure-rest-api"></a>Создание кластеров Apache Hadoop с помощью REST API Azure

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Узнайте, как создавать кластеры HDInsight с помощью шаблонов Azure Resource Manager и Azure REST API.

Azure REST API позволяет управлять службами, размещенными на платформе Azure, в том числе создавать ресурсы, например кластеры HDInsight.

> [!NOTE]  
> В приведенных в этом документе инструкциях для связи с Azure REST API используется служебная программа [curl (https://curl.haxx.se/)](https://curl.haxx.se/).

## <a name="create-a-template"></a>Создание шаблона

Azure Resource Manager шаблоны — это документы JSON, описывающие **группу ресурсов** и все ресурсы в ней (например, HDInsight). Этот подход на основе шаблонов позволяет определить ресурсы, необходимые для HDInsight, в одном шаблоне.

Следующий документ JSON — это слияние файлов шаблонов и параметров из [https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-ssh-password](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-ssh-password) , которое создает кластер под управлением Linux с использованием пароля для защиты учетной записи пользователя SSH.

   ```json
   {
       "properties": {
           "template": {
               "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
               "contentVersion": "1.0.0.0",
               "parameters": {
                   "clusterType": {
                       "type": "string",
                       "allowedValues": ["hadoop",
                       "hbase",
                       "storm",
                       "spark"],
                       "metadata": {
                           "description": "The type of the HDInsight cluster to create."
                       }
                   },
                   "clusterName": {
                       "type": "string",
                       "metadata": {
                           "description": "The name of the HDInsight cluster to create."
                       }
                   },
                   "clusterLoginUserName": {
                       "type": "string",
                       "metadata": {
                           "description": "These credentials can be used to submit jobs to the cluster and to log into cluster dashboards."
                       }
                   },
                   "clusterLoginPassword": {
                       "type": "securestring",
                       "metadata": {
                           "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
                       }
                   },
                   "sshUserName": {
                       "type": "string",
                       "metadata": {
                           "description": "These credentials can be used to remotely access the cluster."
                       }
                   },
                   "sshPassword": {
                       "type": "securestring",
                       "metadata": {
                           "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
                       }
                   },
                   "clusterStorageAccountName": {
                       "type": "string",
                       "metadata": {
                           "description": "The name of the storage account to be created and be used as the cluster's storage."
                       }
                   },
                   "clusterWorkerNodeCount": {
                       "type": "int",
                       "defaultValue": 4,
                       "metadata": {
                           "description": "The number of nodes in the HDInsight cluster."
                       }
                   }
               },
               "variables": {
                   "defaultApiVersion": "2015-05-01-preview",
                   "clusterApiVersion": "2015-03-01-preview"
               },
               "resources": [{
                   "name": "[parameters('clusterStorageAccountName')]",
                   "type": "Microsoft.Storage/storageAccounts",
                   "location": "[resourceGroup().location]",
                   "apiVersion": "[variables('defaultApiVersion')]",
                   "dependsOn": [],
                   "tags": {

                   },
                   "properties": {
                       "accountType": "Standard_LRS"
                   }
               },
               {
                   "name": "[parameters('clusterName')]",
                   "type": "Microsoft.HDInsight/clusters",
                   "location": "[resourceGroup().location]",
                   "apiVersion": "[variables('clusterApiVersion')]",
                   "dependsOn": ["[concat('Microsoft.Storage/storageAccounts/',parameters('clusterStorageAccountName'))]"],
                   "tags": {

                   },
                   "properties": {
                       "clusterVersion": "3.6",
                       "osType": "Linux",
                       "clusterDefinition": {
                           "kind": "[parameters('clusterType')]",
                           "configurations": {
                               "gateway": {
                                   "restAuthCredential.isEnabled": true,
                                   "restAuthCredential.username": "[parameters('clusterLoginUserName')]",
                                   "restAuthCredential.password": "[parameters('clusterLoginPassword')]"
                               }
                           }
                       },
                       "storageProfile": {
                           "storageaccounts": [{
                               "name": "[concat(parameters('clusterStorageAccountName'),'.blob.core.windows.net')]",
                               "isDefault": true,
                               "container": "[parameters('clusterName')]",
                               "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('clusterStorageAccountName')), variables('defaultApiVersion')).key1]"
                           }]
                       },
                       "computeProfile": {
                           "roles": [{
                               "name": "headnode",
                               "targetInstanceCount": "2",
                               "hardwareProfile": {
                                   "vmSize": "{}" 
                               },
                               "osProfile": {
                                   "linuxOperatingSystemProfile": {
                                       "username": "[parameters('sshUserName')]",
                                       "password": "[parameters('sshPassword')]"
                                   }
                               }
                           },
                           {
                               "name": "workernode",
                               "targetInstanceCount": "[parameters('clusterWorkerNodeCount')]",
                               "hardwareProfile": {
                                   "vmSize": "{}"
                               },
                               "osProfile": {
                                   "linuxOperatingSystemProfile": {
                                       "username": "[parameters('sshUserName')]",
                                       "password": "[parameters('sshPassword')]"
                                   }
                               }
                           }]
                       }
                   }
               }],
               "outputs": {
                   "cluster": {
                       "type": "object",
                       "value": "[reference(resourceId('Microsoft.HDInsight/clusters',parameters('clusterName')))]"
                   }
               }
           },
           "mode": "incremental",
           "Parameters": {
               "clusterName": {
                   "value": "newclustername"
               },
               "clusterType": {
                   "value": "hadoop"
               },
               "clusterStorageAccountName": {
                   "value": "newstoragename"
               },
               "clusterLoginUserName": {
                   "value": "admin"
               },
               "clusterLoginPassword": {
                   "value": "changeme"
               },
               "sshUserName": {
                   "value": "sshuser"
               },
               "sshPassword": {
                   "value": "changeme"
               }
           }
       }
   }
   ```

Этот пример используется при выполнении действий в этом документе. Замените пример *значений* в разделе **Параметры** значениями для своего кластера.

> [!IMPORTANT]  
> Шаблон использует стандартное количество рабочих узлов (4) для кластера HDInsight. Если вы планируете использовать более 32 рабочих узлов, для головного узла потребуется минимум 8-ядерный процессор и 14 ГБ ОЗУ.
>
> Дополнительные сведения о размерах узлов и их стоимости см. на странице с [ценами на HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

## <a name="sign-in-to-your-azure-subscription"></a>Войдите в свою подписку Azure.

Выполните действия, описанные в статье [Приступая к работе с Azure CLI](/cli/azure/get-started-with-az-cli2), и подключитесь к подписке, используя команду `az login`.

## <a name="create-a-service-principal"></a>Создание субъекта-службы

> [!NOTE]  
> Приведенные действия представляют собой сокращенную версию раздела *Создание субъекта-службы с использованием пароля* статьи [Использование интерфейса командной строки Azure для создания субъекта-службы и доступа к ресурсам](/cli/azure/create-an-azure-service-principal-azure-cli). Выполнив эти действия, вы создадите субъект-службу, используемый для аутентификации в Azure REST API.

1. Используйте следующую команду в командной строке, чтобы вывести список подписок Azure.

   ```azurecli
   az account list --query '[].{Subscription_ID:id,Tenant_ID:tenantId,Name:name}'  --output table
   ```

    Из списка выберите подписку, которую нужно использовать, и обратите внимание на столбцы **Subscription_ID** и __Tenant_ID__. Сохраните эти значения.

2. Используйте следующую команду, чтобы создать приложение в Azure Active Directory.

   ```azurecli
   az ad app create --display-name "exampleapp" --homepage "https://www.contoso.org" --identifier-uris "https://www.contoso.org/example" --password <Your password> --query 'appId'
   ```

    Замените значения `--display-name`, `--homepage` и `--identifier-uris` собственными. Введите пароль для новой записи Active Directory.

   > [!NOTE]  
   > Значения `--home-page` и `--identifier-uris` не обязательно должны ссылаться на фактическую веб-страницу, размещенную в Интернете. Это должны быть уникальные универсальные коды ресурса.

   Эта команда возвращает значение __App ID__ для нового приложения. Сохраните его.

3. Используйте следующую команду, чтобы создать субъект-службу с помощью **App ID**:

   ```azurecli
   az ad sp create --id <App ID> --query 'objectId'
   ```

     Эта команда возвращает значение __Object ID__. Сохраните его.

4. Назначьте роль **Владелец** субъекту-службе, используя значение **Object ID**. Используйте **идентификатор подписки**, полученный ранее.

   ```azurecli
   az role assignment create --assignee <Object ID> --role Owner --scope /subscriptions/<Subscription ID>/
   ```

## <a name="get-an-authentication-token"></a>Получение маркера проверки подлинности

Используйте следующую команду, чтобы получить маркер аутентификации:

```bash
curl -X "POST" "https://login.microsoftonline.com/$TENANTID/oauth2/token" \
-H "Cookie: flight-uxoptin=true; stsservicecookie=ests; x-ms-gateway-slice=productionb; stsservicecookie=ests" \
-H "Content-Type: application/x-www-form-urlencoded" \
--data-urlencode "client_id=$APPID" \
--data-urlencode "grant_type=client_credentials" \
--data-urlencode "client_secret=$PASSWORD" \
--data-urlencode "resource=https://management.azure.com/"
```

Задайте для `$TENANTID`, `$APPID` и `$PASSWORD` полученные или указанные ранее значения.

Если этот запрос завершится успешно, будет получен ответ серии 200, и тело ответа будет содержать документ JSON.

Документ JSON, возвращаемый этим запросом, содержит элемент **access_token**. Значение **access_token** используется для проверки подлинности запросов API REST.

```json
{
    "token_type":"Bearer",
    "expires_in":"3599",
    "expires_on":"1463409994",
    "not_before":"1463406094",
    "resource":"https://management.azure.com/","access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWoNBVGZNNXBPWWlKSE1iYTlnb0VLWSIsImtpZCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSJ9.eyJhdWQiOiJodHRwczovL21hbmFnZW1lbnQuYXp1cmUuY29tLyIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI2Ny8iLCJpYXQiOjE0NjM0MDYwOTQsIm5iZiI6MTQ2MzQwNjA5NCwiZXhwIjoxNDYzNDA5OTk5LCJhcHBpZCI6IjBlYzcyMzM0LTZkMDMtNDhmYi04OWU1LTU2NTJiODBiZDliYiIsImFwcGlkYWNyIjoiMSIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0Ny8iLCJvaWQiOiJlNjgxZTZiMi1mZThkLTRkZGUtYjZiMS0xNjAyZDQyNWQzOWYiLCJzdWIiOiJlNjgxZTZiMi1mZThkLTRkZGUtYjZiMS0xNjAyZDQyNWQzOWYiLCJ0aWQiOiI3MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDciLCJ2ZXIiOiIxLjAifQ.nJVERbeDHLGHn7ZsbVGBJyHOu2PYhG5dji6F63gu8XN2Cvol3J1HO1uB4H3nCSt9DTu_jMHqAur_NNyobgNM21GojbEZAvd0I9NY0UDumBEvDZfMKneqp7a_cgAU7IYRcTPneSxbD6wo-8gIgfN9KDql98b0uEzixIVIWra2Q1bUUYETYqyaJNdS4RUmlJKNNpENllAyHQLv7hXnap1IuzP-f5CNIbbj9UgXxLiOtW5JhUAwWLZ3-WMhNRpUO2SIB7W7tQ0AbjXw3aUYr7el066J51z5tC1AK9UC-mD_fO_HUP6ZmPzu5gLA6DxkIIYP3grPnRVoUDltHQvwgONDOw"
}
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Чтобы создать группу ресурсов, сделайте следующее:

* Задайте `$SUBSCRIPTIONID` для идентификатора подписки, полученного при создании субъекта-службы.
* Для маркера доступа, полученного на предыдущем этапе, задайте `$ACCESSTOKEN`.
* Замените `DATACENTERLOCATION` центром обработки данных, в котором вы хотите создать группу ресурсов и ресурсы. Например, "Центрально-южная часть США".
* Задайте для `$RESOURCEGROUPNAME` имя, которое хотите использовать для этой группы:

```bash
curl -X "PUT" "https://management.azure.com/subscriptions/$SUBSCRIPTIONID/resourcegroups/$RESOURCEGROUPNAME?api-version=2015-01-01" \
    -H "Authorization: Bearer $ACCESSTOKEN" \
    -H "Content-Type: application/json" \
    -d $'{
"location": "DATACENTERLOCATION"
}'
```

Если этот запрос завершится успешно, будет получен ответ серии 200, и тело ответа будет включать в себя документ JSON, содержащий информацию о группе. Элемент `"provisioningState"` будет содержать значение `"Succeeded"`.

## <a name="create-a-deployment"></a>Создание развертывания

Используйте следующую команду, чтобы развернуть шаблон в группе ресурсов.

* Задайте для `$DEPLOYMENTNAME` имя, которое хотите использовать для этого развертывания.

```bash
curl -X "PUT" "https://management.azure.com/subscriptions/$SUBSCRIPTIONID/resourcegroups/$RESOURCEGROUPNAME/providers/microsoft.resources/deployments/$DEPLOYMENTNAME?api-version=2015-01-01" \
-H "Authorization: Bearer $ACCESSTOKEN" \
-H "Content-Type: application/json" \
-d "{set your body string to the template and parameters}"
```

> [!NOTE]  
> При сохранении шаблона в файл вместо `-d "{ template and parameters}"` можно использовать следующую команду:
>
> `--data-binary "@/path/to/file.json"`

Если этот запрос завершится успешно, будет получен ответ серии 200, и тело ответа будет включать в себя документ JSON, содержащий информацию об операции развертывания.

> [!IMPORTANT]  
> Развертывание было отправлено, но не завершено. Для завершения развертывания может потребоваться несколько минут, обычно около 15.

## <a name="check-the-status-of-a-deployment"></a>Проверка состояния развертывания

Чтобы проверить состояние развертывания, используйте следующую команду:

```bash
curl -X "GET" "https://management.azure.com/subscriptions/$SUBSCRIPTIONID/resourcegroups/$RESOURCEGROUPNAME/providers/microsoft.resources/deployments/$DEPLOYMENTNAME?api-version=2015-01-01" \
-H "Authorization: Bearer $ACCESSTOKEN" \
-H "Content-Type: application/json"
```

Эта команда возвращает документ JSON, содержащий сведения об операции развертывания. Элемент `"provisioningState"` содержит сведения о состоянии развертывания. Если элемент содержит значение `"Succeeded"`, развертывание завершилось успешно.

## <a name="troubleshoot"></a>Диагностика

Если при создании кластеров HDInsight возникли проблемы, см. раздел [Создание кластеров](./hdinsight-hadoop-customize-cluster-linux.md#access-control).

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы успешно создали кластер HDInsight, используйте следующую информацию, чтобы узнать, как работать с кластером.

### <a name="apache-hadoop-clusters"></a>Кластеры Apache Hadoop

* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hadoop/hdinsight-use-hive.md)
* [Использование MapReduce в Hadoop в HDInsight](hadoop/hdinsight-use-mapreduce.md)

### <a name="apache-hbase-clusters"></a>Кластеры Apache HBase

* [Начало работы с Apache HBase в HDInsight](hbase/apache-hbase-tutorial-get-started-linux.md)
* [Разработка приложений Java для Apache HBase в HDInsight](hbase/apache-hbase-build-java-maven-linux.md)

### <a name="apache-storm-clusters"></a>Кластеры Apache Storm

* [Создание топологий для Apache Storm в HDInsight на языке Java](storm/apache-storm-develop-java-topology.md)
* [Использование компонентов Python в Apache Storm в HDInsight](storm/apache-storm-develop-python-topology.md)
* [Развертывание и мониторинг топологии с помощью Apache Storm в HDInsight.](storm/apache-storm-deploy-monitor-topology-linux.md)
