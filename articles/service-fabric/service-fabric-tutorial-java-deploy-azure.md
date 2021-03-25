---
title: Развертывание приложения Java в кластере Service Fabric в Azure
description: В этом руководстве вы узнаете, как развертывать приложение Java Service Fabric в кластере Azure Service Fabric.
ms.topic: tutorial
ms.date: 02/26/2018
ms.custom: mvc, devx-track-java, devx-track-azurecli
ms.openlocfilehash: 48cb7df49fb918ebd30883edc573cc63b6e35e4a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98789402"
---
# <a name="tutorial-deploy-a-java-application-to-a-service-fabric-cluster-in-azure"></a>Руководство по Развертывание приложения Java в кластере Service Fabric в Azure

Это руководство является третьей частью цикла. В нем показано, как развернуть приложение Service Fabric в кластере в Azure.

В третьей части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание защищенного кластера Linux в Azure.
> * Развертывание приложения в кластере

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * [Развертывание приложения Java служб Service Fabric Reliable Services в Azure.](service-fabric-tutorial-create-java-app.md)
> * [Развертывание и отладка приложения в локальном кластере.](service-fabric-tutorial-debug-log-local-cluster.md)
> * Развертывание приложения в кластере Azure
> * [Настройка мониторинга и диагностики приложения](service-fabric-tutorial-java-elk.md)
> * [Настройка процесса непрерывной интеграции и доставки](service-fabric-tutorial-java-jenkins.md)

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Установка Azure CLI](/cli/azure/install-azure-cli)
* Установите пакет SDK для Service Fabric для [Mac](service-fabric-get-started-mac.md) или [Linux](service-fabric-get-started-linux.md).
* [Установите Python 3](https://wiki.python.org/moin/BeginnersGuide/Download).

## <a name="create-a-service-fabric-cluster-in-azure"></a>Создание кластера Service Fabric в Azure

При выполнении следующих шагов создаются ресурсы, необходимые для развертывания вашего приложения в кластере Service Fabric. Кроме того, устанавливаются ресурсы, необходимые для мониторинга работоспособности вашего решения с использованием стека ELK (Elasticsearch, Logstash, Kibana). В частности, [Центры событий](https://azure.microsoft.com/services/event-hubs/) используются в качестве приемника для журналов из Service Fabric. Он настраивается для отправки журналов из кластера Service Fabric в ваш экземпляр Logstash.

1. Откройте терминал и загрузите следующий пакет, содержащий необходимые вспомогательные скрипты и шаблоны для создания ресурсов в Azure.

    ```bash
    git clone https://github.com/Azure-Samples/service-fabric-java-quickstart.git
    ```

2. Вход в учетную запись Azure

    ```azurecli
    az login
    ```

3. Установите подписку Azure, которую вы хотите использовать для создания ресурсов.

    ```azurecli
    az account set --subscription [SUBSCRIPTION-ID]
    ```

4. В папке *service-fabric-java-quickstart/AzureCluster* запустите следующую команду для создания сертификата кластера в хранилище ключей. Этот сертификат используется для защиты кластера Service Fabric. Укажите регион (должен быть таким же, как кластер Service Fabric), имя группы ресурсов хранилища ключей, имя хранилища ключей, пароль сертификата и DNS-имя кластера.

    ```bash
    ./new-service-fabric-cluster-certificate.sh [REGION] [KEY-VAULT-RESOURCE-GROUP] [KEY-VAULT-NAME] [CERTIFICATE-PASSWORD] [CLUSTER-DNS-NAME-FOR-CERTIFICATE]

    Example: ./new-service-fabric-cluster-certificate.sh 'westus' 'testkeyvaultrg' 'testkeyvault' '<password>' 'testservicefabric.westus.cloudapp.azure.com'
    ```

    Предыдущая команда возвращает следующие сведения, которые следует отметить для дальнейшего использования.

    ```output
    Source Vault Resource Id: /subscriptions/<subscription_id>/resourceGroups/testkeyvaultrg/providers/Microsoft.KeyVault/vaults/<name>
    Certificate URL: https://<name>.vault.azure.net/secrets/<cluster-dns-name-for-certificate>/<guid>
    Certificate Thumbprint: <THUMBPRINT>
    ```

5. Создайте группу ресурсов для учетной записи хранения, в которой хранятся ваши журналы.

    ```azurecli
    az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

    Example: az group create --location westus --name teststorageaccountrg
    ```

6. Создайте учетную запись хранения, которая будет использоваться для хранения создаваемых журналов.

    ```azurecli
    az storage account create -g [RESOURCE-GROUP-NAME] -l [REGION] --name [STORAGE-ACCOUNT-NAME] --kind Storage

    Example: az storage account create -g teststorageaccountrg -l westus --name teststorageaccount --kind Storage
    ```

7. Войдите на портал [Azure](https://portal.azure.com) и перейдите на вкладку **Подписанный URL-адрес** своей учетной записи хранения. Создайте маркер SAS следующим образом.

    ![Создайте SAS для хранилища.](./media/service-fabric-tutorial-java-deploy-azure/storagesas.png)

8. Скопируйте URL-адрес SAS учетной записи для использования в дальнейшем при создании кластера Service Fabric. Результат будет выглядеть, как следующий URL-адрес:

    ```output
    ?sv=2017-04-17&ss=bfqt&srt=sco&sp=rwdlacup&se=2018-01-31T03:24:04Z&st=2018-01-30T19:24:04Z&spr=https,http&sig=IrkO1bVQCHcaKaTiJ5gilLSC5Wxtghu%2FJAeeY5HR%2BPU%3D
    ```

9. Создайте группу ресурсов, содержащую ресурсы концентратора событий. Центры событий используются для отправки сообщений из Service Fabric на сервер, где запущены ресурсы ELK.

    ```azurecli
    az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

    Example: az group create --location westus --name testeventhubsrg
    ```

10. Создайте ресурс Центров событий, используя следующую команду. Следуя инструкциям, введите данные namespaceName, eventHubName, customerGroupName, sendAuthorizationRule и receiveAuthorizationRule.

    ```azurecli
    az deployment group create -g [RESOURCE-GROUP-NAME] --template-file eventhubsdeploy.json

    Example:
    az deployment group create -g testeventhubsrg --template-file eventhubsdeploy.json
    Please provide string value for 'namespaceName' (? for help): testeventhubnamespace
    Please provide string value for 'eventHubName' (? for help): testeventhub
    Please provide string value for 'consumerGroupName' (? for help): testeventhubconsumergroup
    Please provide string value for 'sendAuthorizationRuleName' (? for help): sender
    Please provide string value for 'receiveAuthorizationRuleName' (? for help): receiver
    ```

    Скопируйте содержимое поля **выходных данных** в выходных данных JSON предыдущей команды. Сведения об отправителе используются при создании кластера Service Fabric. Имя получателя и ключ следует сохранить для использования в следующем руководстве, если служба Logstash настроена для получения сообщений от концентратора событий. Следующий большой двоичный объект является примером выходных данных JSON:

    ```json
    "outputs": {
        "receiver Key": {
            "type": "String",
            "value": "[KEY]"
        },
        "receiver Name": {
            "type": "String",
            "value": "receiver"
        },
        "sender Key": {
            "type": "String",
            "value": "[KEY]"
        },
        "sender Name": {
            "type": "String",
            "value": "sender"
        }
    }
    ```

11. Запустите скрипт *eventhubssastoken.py*, чтобы создать URL-адрес SAS для созданного ресурса EventHubs. Этот URL-адрес SAS используется кластером Service Fabric для отправки журналов в Центры событий. В результате для формирования URL-адреса используется политика **отправителя**. Скрипт возвращает URL-адрес SAS для ресурса Центров событий, который используется на следующем шаге:

    ```python
    python3 eventhubssastoken.py 'testeventhubs' 'testeventhubs' 'sender' '[PRIMARY-KEY]'
    ```

    Скопируйте значение поля **sr** в возвращаемом JSON. Значение поля **sr** — это токен SAS для концентраторов событий. Следующий URL-адрес является примером поля **sr**:

    ```output
    https%3A%2F%testeventhub.servicebus.windows.net%testeventhub&sig=7AlFYnbvEm%2Bat8ALi54JqHU4i6imoFxkjKHS0zI8z8I%3D&se=1517354876&skn=sender
    ```

    URL-адрес SAS концентраторов событий имеет следующую структуру: `https://<namespacename>.servicebus.windows.net/<eventhubsname>?sr=<sastoken>`. Например `https://testeventhubnamespace.servicebus.windows.net/testeventhub?sr=https%3A%2F%testeventhub.servicebus.windows.net%testeventhub&sig=7AlFYnbvEm%2Bat8ALi54JqHU4i6imoFxkjKHS0zI8z8I%3D&se=1517354876&skn=sender`.

12. Откройте файл *sfdeploy.parameters.json* и замените следующее содержимое из предыдущих шагов. [SAS-URL-STORAGE-ACCOUNT] — отмечено на шаге 8. [SAS-URL-EVENT-HUBS] — отмечено на шаге 11.

    ```json
    "applicationDiagnosticsStorageAccountName": {
        "value": "teststorageaccount"
    },
    "applicationDiagnosticsStorageAccountSasToken": {
        "value": "[SAS-URL-STORAGE-ACCOUNT]"
    },
    "loggingEventHubSAS": {
        "value": "[SAS-URL-EVENT-HUBS]"
    }
    ```

13. Открывает **sfdeploy.parameters.json**. Измените следующие параметры, а затем сохраните файл.
    - **clusterName**. Вы можете использовать только строчные буквы и цифры.
    - **adminUserName** (на значение, отличное от пустого).
    - **adminUserName** (на значение, отличное от пустого).

14. Выполните следующую команду, чтобы создать кластер Service Fabric.

    ```azurecli
    az sf cluster create --location 'westus' --resource-group 'testlinux' --template-file sfdeploy.json --parameter-file sfdeploy.parameters.json --secret-identifier <certificate_url_from_step4>
    ```

## <a name="deploy-your-application-to-the-cluster"></a>Развертывание приложения в кластере

1. Перед развертыванием приложения необходимо добавить следующий фрагмент кода в файл *Voting/VotingApplication/ApplicationManifest.xml*. Поле **X509FindValue** — это отпечаток, полученный на шаге 4 раздела **Создание кластера Service Fabric в Azure**. Этот фрагмент кода вложен в поле **ApplicationManifest** (корневое поле).

    ```xml
    <Certificates>
          <SecretsCertificate X509FindType="FindByThumbprint" X509FindValue="[CERTIFICATE-THUMBPRINT]" />
    </Certificates>
    ```

2. Чтобы развернуть приложение в этом кластере, вы должны использовать SFCTL, чтобы установить соединение с кластером. SFCTL требуется PEM-файл с открытым и закрытым ключом для подключения к кластеру. Выполните приведенную ниже команду, чтобы создать PEM-файл с открытым и закрытым ключом. 

    ```bash
    openssl pkcs12 -in <clustername>.<region>.cloudapp.azure.com.pfx -out sfctlconnection.pem -nodes -passin pass:<password>
    ```

3. Выполните следующую команду для подключения к кластеру.

    ```bash
    sfctl cluster select --endpoint https://<clustername>.<region>.cloudapp.azure.com:19080 --pem sfctlconnection.pem --no-verify
    ```

4. Чтобы развернуть приложение, перейдите в папку *Voting/Scripts* и запустите скрипт **install.sh**.

    ```bash
    ./install.sh
    ```

5. Для доступа к Service Fabric Explorer откройте любой веб-браузер и введите `https://testlinuxcluster.westus.cloudapp.azure.com:19080`. Выберите сертификат из хранилища сертификатов, который вы хотите использовать для подключения к этой конечной точке. Если вы используете компьютер Linux, сертификаты, созданные скриптом *new-service-fabric-cluster-certificate.sh*, нужно импортировать в Chrome для просмотра Service Fabric Explorer. Если вы используете компьютер Mac, необходимо установить PFX-файл в цепочку ключей. Обратите внимание, что приложение установлено в кластере.

    ![SFX Java Azure](./media/service-fabric-tutorial-java-deploy-azure/sfxjavaonazure.png)

6. Чтобы получить доступ к приложению, введите `https://testlinuxcluster.westus.cloudapp.azure.com:8080`

    ![Приложение для голосования Java Azure](./media/service-fabric-tutorial-java-deploy-azure/votingappjavaazure.png)

7. Чтобы удалить приложение из кластера, запустите скрипт *uninstall.sh* в папке **Scripts**.

    ```bash
    ./uninstall.sh
    ```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание защищенного кластера Linux в Azure.
> * Создание ресурсов, необходимых для мониторинга с помощью ELK.

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Tutorial: Monitor your Service Fabric applications using ELK](service-fabric-tutorial-java-elk.md) (Руководство. Мониторинг приложений Service Fabric с помощью ELK)
