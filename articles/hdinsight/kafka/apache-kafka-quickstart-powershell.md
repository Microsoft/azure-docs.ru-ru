---
title: Краткое руководство. Создание Apache Kafka с помощью Azure PowerShell — HDInsight
description: В этом кратком руководстве вы узнаете, как создать кластер Apache Kafka в Azure HDInsight с помощью Azure PowerShell. Вы также узнаете, что такое разделы, подписчики и потребители в Kafka.
ms.service: hdinsight
ms.custom: mvc
ms.topic: quickstart
ms.date: 06/12/2019
ms.openlocfilehash: f993ffa8d0d141d04ad399c5d1d4f0fc28cc82ac
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102505143"
---
# <a name="quickstart-create-apache-kafka-cluster-in-azure-hdinsight-using-powershell"></a>Краткое руководство. Создание кластера Apache Kafka в Azure HDInsight с помощью PowerShell

[Apache Kafka](https://kafka.apache.org/) — это распределенная платформа потоковой передачи с открытым кодом. Она часто используется как брокер сообщений, предоставляя такие же функциональные возможности, как и очередь сообщений типа "публикация-подписка". 

В этом кратком руководстве вы узнаете, как создать кластер [Apache Kafka](https://kafka.apache.org) с помощью Azure PowerShell. Вы также узнаете, как с помощью предоставленных служебных программ отправлять и получать сообщения, используя Kafka.

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

API Kafka доступен только ресурсам, размещенным в той же виртуальной сети. В этом кратком руководстве вы напрямую обращаетесь к кластеру по протоколу SSH. Чтобы подключить к Kafka другие службы, сети или виртуальные машины, необходимо сначала создать виртуальную сеть, а затем создать в ней эти ресурсы. Дополнительные сведения см. в документе [Подключение к Kafka в HDInsight с помощью виртуальной сети Azure](apache-kafka-connect-vpn-gateway.md).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Установите [модуль Az](/powershell/azure/) для PowerShell.

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в подписку Azure с помощью командлета `Connect-AzAccount` и следуйте инструкциям на экране.

```azurepowershell-interactive
# Login to your Azure subscription
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"
```

## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере запрашиваются имя и расположение, а затем создается группа ресурсов.

```azurepowershell-interactive
$resourceGroup = Read-Host -Prompt "Enter the resource group name"
$location = Read-Host -Prompt "Enter the Azure region to use"

New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

Хотя кластер Kafka в HDInsight использует управляемые диски Azure для хранения данных Kafka, он также использует службу хранилища Azure для хранения различных сведений, например журналов. Используйте командлет [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount), чтобы создать учетную запись хранения.

> [!IMPORTANT]  
> Тип учетной записи хранения `BlobStorage` можно использовать только как дополнительное хранилище кластеров HDInsight.

```azurepowershell-interactive
$storageName = Read-Host -Prompt "Enter the storage account name"

New-AzStorageAccount `
    -ResourceGroupName $resourceGroup `
    -Name $storageName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableHttpsTrafficOnly 1
```

HDInsight сохраняет данные в учетной записи хранения в контейнере больших двоичных объектов. Создайте контейнер с помощью командлета [New-AzStorageContainer](/powershell/module/Az.Storage/New-AzStorageContainer).

```azurepowershell-interactive
$containerName = Read-Host -Prompt "Enter the container name"

$storageKey = (Get-AzStorageAccountKey `
                -ResourceGroupName $resourceGroup `
                -Name $storageName)[0].Value
$storageContext = New-AzStorageContext `
                    -StorageAccountName $storageName `
                    -StorageAccountKey $storageKey

New-AzStorageContainer -Name $containerName -Context $storageContext
```

## <a name="create-an-apache-kafka-cluster"></a>Создание кластера Apache Kafka

Создайте кластер Apache Kafka в HDInsight с помощью командлета [New-AzHDInsightCluster](/powershell/module/az.HDInsight/New-azHDInsightCluster).

```azurepowershell-interactive
# Create a Kafka 1.1 cluster
$clusterName = Read-Host -Prompt "Enter the name of the Kafka cluster"
$httpCredential = Get-Credential -Message "Enter the cluster login credentials" -UserName "admin"
$sshCredentials = Get-Credential -Message "Enter the SSH user credentials" -UserName "sshuser"

$numberOfWorkerNodes = "4"
$clusterVersion = "3.6"
$clusterType="Kafka"
$disksPerNode=2

$kafkaConfig = New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"
$kafkaConfig.Add("kafka", "1.1")

New-AzHDInsightCluster `
        -ResourceGroupName $resourceGroup `
        -ClusterName $clusterName `
        -Location $location `
        -ClusterSizeInNodes $numberOfWorkerNodes `
        -ClusterType $clusterType `
        -OSType "Linux" `
        -Version $clusterVersion `
        -ComponentVersion $kafkaConfig `
        -HttpCredential $httpCredential `
        -DefaultStorageAccountName "$storageName.blob.core.windows.net" `
        -DefaultStorageAccountKey $storageKey `
        -DefaultStorageContainer $clusterName `
        -SshCredential $sshCredentials `
        -DisksPerWorkerNode $disksPerNode
```

Операция создания кластера HDInsight может занять до 20 минут.

Параметр `-DisksPerWorkerNode` позволяет настроить уровень масштабируемости Kafka в HDInsight. Кластер Kafka в HDInsight использует локальный диск виртуальных машин в кластере для хранения данных. Kafka обрабатывает большое количество операций ввода-вывода, поэтому используются [управляемые диски Azure](../../virtual-machines/managed-disks-overview.md), чтобы обеспечить высокую пропускную способность и предоставить дополнительное хранилище для каждого узла.

Управляемый диск может быть двух типов: __Стандартный__ (HDD) или __Премиум__ (SSD). Тип диска зависит от размера виртуальной машины, используемой рабочими узлами (брокерами Kafka). Диски категории "Премиум" автоматически используются для виртуальных машин серий DS и GS. Для всех остальных виртуальных машин используются стандартные управляемые диски. Настроить тип виртуальной машины можно с помощью параметра `-WorkerNodeSize`. Дополнительные сведения о параметрах см. в документации по [New-AzHDInsightCluster](/powershell/module/az.HDInsight/New-azHDInsightCluster).

Если вы планируете использовать более 32 рабочих узлов (при создании кластера или в ходе масштабирования после создания кластера), необходимо использовать параметр `-HeadNodeSize`, чтобы указать размер виртуальной машины, обеспечивающий как минимум 8 процессорных ядер и 14 ГБ ОЗУ. Дополнительные сведения о размерах узлов и их стоимости см. на странице с [ценами на HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

1. Чтобы подключиться к первичному головному узлу кластера Kafka, выполните приведенную ниже команду. Замените `sshuser` именем пользователя SSH. Замените `mykafka` именем кластера Kafka.

    ```bash
    ssh sshuser@mykafka-ssh.azurehdinsight.net
    ```

2. При первом подключении к кластеру клиент SSH может отобразить предупреждение о том, что не удается установить подлинность узла. При появлении запроса введите __yes__ (Да) и нажмите клавишу __ВВОД__, чтобы добавить узел в список доверенных серверов клиента SSH.

3. При появлении запроса введите пароль пользователя SSH.

После подключения отобразятся сведения, аналогичные приведенному ниже тексту.

```output
Authorized uses only. All activity may be monitored and reported.
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-1011-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    https://www.ubuntu.com/business/services/cloud

83 packages can be updated.
37 updates are security updates.



Welcome to Kafka on HDInsight.

Last login: Thu Mar 29 13:25:27 2018 from 108.252.109.241
```

## <a name="get-the-apache-zookeeper-and-broker-host-information"></a><a id="getkafkainfo"></a>Получение сведений об узлах Apache Zookeeper и брокера

Для работы с Kafka необходимы сведения об узле *Apache Zookeeper* и узле *брокера*. Эти узлы используются Kafka API и многими другими служебными программами, поставляемыми с платформой Kafka.

В этом разделе для получения сведений об узле используется REST API Apache Ambari в кластере.

1. Используя SSH-подключение к кластеру, выполните следующую команду, чтобы установить служебную программу `jq`. Эта служебная программа используется для анализа документов JSON. С ее помощью также удобно получать сведения об узлах.
   
    ```bash
    sudo apt -y install jq
    ```

2. Чтобы задать переменную среды с именем кластера, выполните следующую команду.

    ```bash
    read -p "Enter the Kafka on HDInsight cluster name: " CLUSTERNAME
    ```

    При появлении запроса введите имя кластера Kafka.

3. Чтобы задать переменную среды со сведениями об узле Zookeeper, выполните следующую команду. Эта команда извлекает все узлы Zookeeper, а затем возвращает только первые две записи. Причина этого состоит в том, что требуется обеспечить избыточность на случай, если узел станет недоступным.

    ```bash
    export KAFKAZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`
    ```

    При появлении запроса введите пароль учетной записи для входа в кластер (но не учетной записи SSH).

4. Чтобы убедиться, что переменную среды задано верно, выполните следующую команду.

    ```bash
     echo '$KAFKAZKHOSTS='$KAFKAZKHOSTS
    ```

    Эта команда возвращает сведения аналогичные следующим:

    `zk0-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181,zk2-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181`

5. Чтобы задать переменную среды со сведениями об узле брокера Kafka, выполните следующую команду.

    ```bash
    export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    ```

    При появлении запроса введите пароль учетной записи для входа в кластер (но не учетной записи SSH).

6. Чтобы убедиться, что переменную среды задано верно, выполните следующую команду.

    ```bash   
    echo '$KAFKABROKERS='$KAFKABROKERS
    ```

    Эта команда возвращает сведения аналогичные следующим:
   
    `wn1-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092,wn0-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092`

## <a name="manage-apache-kafka-topics"></a>Управление разделами Apache Kafka

Kafka хранит потоки данных в *разделах*. Для управления разделами можно использовать служебную программу `kafka-topics.sh`.

* **Чтобы создать раздел**, в сеансе SSH-подключения выполните следующую команду.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
    ```

    Эта команда создает подключение к Zookeeper, используя хранящиеся в `$KAFKAZKHOSTS` сведения об узле. Затем она создает раздел Kafka с именем **test**. 

    * Данные, хранящиеся в этом разделе, разделены на восемь секций.

    * Каждая секция реплицируются на три рабочих узла в кластере.

        Если кластер был создан в регионе Azure, который предоставляет три домена сбоя, то следует использовать коэффициент репликации, равный 3. В противном случае следует использовать коэффициент репликации, равный 4.
        
        В регионах с тремя доменами сбоя коэффициент репликации, равный 3, обеспечивает распределение реплик по доменам сбоя. В регионах с двумя доменами сбоя коэффициент репликации, равный 4, обеспечивает равномерное распределение реплик по доменам сбоя.
        
        Дополнительные сведения о количестве доменов сбоя в регионе см. в разделе [Использование управляемых дисков для виртуальных машин в группе доступности](../../virtual-machines/availability.md).

        В Kafka нет сведений о доменах сбоя Azure. При создании реплик секций для разделов реплики могут быть неправильно распределены с точки зрения высокой доступности.

        Для обеспечения высокого уровня доступности используйте [средство перераспределения секций Apache Kafka](https://github.com/hdinsight/hdinsight-kafka-tools). Этот инструмент следует запускать из сеанса SSH-подключения на головном узле кластера Kafka.

        Чтобы обеспечить максимально высокий уровень доступности данных Kafka, следует перераспределять реплики секций для раздела в следующих случаях:

        * при создании раздела или секции;

        * при масштабировании кластера.

* **Чтобы вывести список разделов**, введите следующую команду.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
    ```

    Она выводит список разделов на кластере Kafka.

* **Чтобы удалить раздел**, используйте следующую команду.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic topicname --zookeeper $KAFKAZKHOSTS
    ```

    Она удаляет раздел `topicname`.

    > [!WARNING]  
    > Если вы уже удалили раздел `test`, то необходимо создать его заново. Он используется в последующих действиях в этом документе.

Чтобы получить дополнительные сведения о командах, доступных в служебной программе `kafka-topics.sh`, используйте следующую команду.

```bash
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh
```

## <a name="produce-and-consume-records"></a>Создание и использование записей

Kafka хранит *записи* в разделах. Записи создаются *производителями*, а используются *потребителями*. Производители и потребители взаимодействуют со службой *брокера Kafka*. Каждый рабочий узел в кластере HDInsight — это узел брокера Kafka.

Чтобы сохранить записи в созданный ранее раздел test, а затем считать их с помощью потребителя, сделайте следующее:

1. Чтобы записать записи в раздел, используйте служебную программу `kafka-console-producer.sh` из сеанса SSH-подключения.
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic test
    ```
   
    После выполнения этой команды вы перейдете к пустой строке.

2. Введите текстовое сообщение в эту строку и нажмите клавишу ВВОД. Введите несколько таких сообщений, а затем нажмите клавиши **CTRL+C**, чтобы вернуться к обычной командной строке. Каждая строка отправляется в раздел Kafka как отдельная запись.

3. Чтобы считать записи из раздела, используйте служебную программу `kafka-console-consumer.sh` из сеанса SSH-подключения.
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic test --from-beginning
    ```
   
    Эта команда извлекает записи из раздела, а затем отображает их. Параметр `--from-beginning` указывает потребителю считывать данные с самого начала потока, поэтому будут извлечены все записи.

    Если вы используете более раннюю версию Kafka, замените `--bootstrap-server $KAFKABROKERS` на `--zookeeper $KAFKAZKHOSTS`.

4. Нажмите клавиши __Ctrl+C__, чтобы остановить потребитель.

Вы также можете создать отправители и получатели программными средствами. Пример использования этого API см. в руководстве [Использование API производителя и потребителя Apache Kafka](apache-kafka-producer-consumer-api.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов, кластер HDInsight и все связанные с ним ресурсы, выполнив команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroup
```

> [!WARNING]  
> Начисление оплаты начинается после создания кластера HDInsight и прекращается только после его удаления. Кластеры оплачиваются поминутно, поэтому всегда следует удалять кластер, когда он больше не нужен.
> 
> При удалении кластера Kafka в HDInsight удаляются все данные, хранящиеся в Kafka.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Совместное использование Apache Spark и Apache Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md)