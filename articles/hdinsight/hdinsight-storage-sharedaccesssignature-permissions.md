---
title: Ограничение доступа с использованием подписанных URL-адресов в Azure HDInsight
description: Узнайте, как использовать подписанные URL-права для ограничения доступа HDInsight к данным, хранящимся в хранилище BLOB-объектов Azure.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020, devx-track-azurecli
ms.date: 04/28/2020
ms.openlocfilehash: 647ac363dd81604f0a5a1d750d98a3349beb19c0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98931716"
---
# <a name="use-azure-blob-storage-shared-access-signatures-to-restrict-access-to-data-in-hdinsight"></a>Использование подписей общего доступа хранилища BLOB-объектов Azure для ограничения доступа к данным в HDInsight

HDInsight имеет полный доступ к данным в учетных записях хранения BLOB-объектов Azure, связанных с кластером. Чтобы ограничить этот доступ, вы можете применить подписанные URL-адреса для контейнера больших двоичных объектов. Подписанные URL-адрес (SAS) — это функция учетных записей хранилища больших двоичных объектов Azure, которая позволяет ограничить доступ к данным. например предоставлять доступ к данным только для чтения.

> [!IMPORTANT]  
> Чтобы создать решение на основе Apache Ranger, попробуйте использовать HDInsight, присоединенный к домену. Дополнительные сведения см. в статье [Настройка присоединенных к домену кластеров HDInsight (предварительная версия)](./domain-joined/apache-domain-joined-configure-using-azure-adds.md).

> [!WARNING]  
> HDInsight следует предоставить полный доступ к хранилищу по умолчанию для кластера.

## <a name="prerequisites"></a>Предварительные требования

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](./hdinsight-hadoop-linux-use-ssh-unix.md).

* Существующий [контейнер хранилища](../storage/blobs/storage-quickstart-blobs-portal.md).  

* При использовании PowerShell вам потребуется [модуль Az](/powershell/azure/).

* Если вы хотите использовать Azure CLI и еще не установили его, обратитесь к разделу [Установка Azure CLI](/cli/azure/install-azure-cli).

* При использовании [Python](https://www.python.org/downloads/)версии 2,7 или более поздней.

* При использовании C# Visual Studio должна иметь версию 2013 или более позднюю.

* Схема универсального кода ресурса (URI) для вашей учетной записи хранения. Эта схема будет использоваться `wasb://` для хранилища BLOB-объектов Azure, `abfs://` для Azure Data Lake Storage 2-го поколения или `adl://` для Azure Data Lake Storage 1-го поколения. Если для хранилища BLOB-объектов Azure включено безопасное перемещение, URI будет иметь значение `wasbs://` .

* Существующий кластер HDInsight, в который добавляется подписанный URL-доступ. Если нет, создайте кластер с помощью Azure PowerShell и добавьте подписанный URL-адрес в процессе создания кластера.

* Примеры файлов из [https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature) . Этот репозиторий содержит указанные далее элементы.

  * Проект Visual Studio, способный создать контейнер хранилища, хранимую политику и SAS для использования с HDInsight.
  * Сценарий Python, способный создать контейнер хранилища, хранимую политику и SAS для использования с HDInsight.
  * Сценарий PowerShell, способный создать кластер HDInsight и настроить его для использования SAS. Обновленная версия используется далее.
  * Пример файла: `hdinsight-dotnet-python-azure-storage-shared-access-signature-master\sampledata\sample.log`

## <a name="shared-access-signatures"></a>Подписанные URL-адреса

Существуют две формы подписанных URL-адресов:

* `Ad hoc`: Время начала, время окончания срока действия и разрешения для SAS указаны в URI SAS.

* `Stored access policy`: Хранимая политика доступа определяется в контейнере ресурсов, например в контейнере больших двоичных объектов. Она может использоваться для управления ограничениями одного или нескольких подписанных URL-адресов. При сопоставлении подписи общего доступа с хранимой политикой доступа эта подпись наследует ограничения (время начала, время окончания и разрешения), определенные для данной хранимой политики доступа.

Различие между этими двумя формами важно для одного ключевого сценария — отзыва. SAS — это URL-адрес, поэтому любой пользователь, получивший SAS, может его использовать. Не имеет значения, кто из них запросил начало. Размещенный в свободном доступе подписанный URL-адрес сможет использовать любой человек. Распространяемая подпись общего доступа действительна до тех пор, пока не произойдет одно из четырех возможных событий:

1. Наступает время истечения срока действия, указанное для данной подписи общего доступа.

2. Истек срок действия, указанный для хранимой политики доступа, на которую ссылается SAS. Следующие сценарии приводят к достижению времени окончания срока действия:

    * Истек интервал времени.
    * Хранимая политика доступа изменилась, чтобы время окончания срока действия оказалось в прошлом. При изменении времени окончания срока действия отменяется подписанный URL-адрес.

3. Удаляется хранимая политика доступа, на которую ссылается подпись, что является другим способом отозвать подпись общего доступа. При повторном создании хранимой политики доступа с тем же именем все маркеры SAS для предыдущей политики будут действительны (если время окончания срока действия SAS не прошло). Если вы намерены отозвать SAS, обязательно используйте другое имя при повторном создании политики доступа, срок действия которой еще не истек.

4. Повторно создается ключ учетной записи, который был использован для создания подписи общего доступа. Это приведет к тому, что все приложения, использующие прежний ключ, не смогут пройти аутентификацию. Предоставьте всем компонентам новый ключ.

> [!IMPORTANT]  
> URI подписанного URL-адреса связан с ключом учетной записи, который использовался для создания подписи и соответствующей хранимой политики доступа (при наличии таковой). Если хранимая политика доступа не указана, то единственный способ отменить подписанный URL-адрес — изменить ключ учетной записи.

Рекомендуется всегда использовать хранимые политики доступа. При использовании хранимой политики можно отменить подписи или продлить дату окончания срока действия при необходимости. В инструкциях, приведенных в этом документе, для формирования подписанных URL-адресов используются хранимые политики доступа.

Дополнительные сведения о подписанных URL-адресах см. в статье [Общие сведения о модели SAS](../storage/common/storage-sas-overview.md).

## <a name="create-a-stored-policy-and-sas"></a>Создание хранимой политики и SAS

Сохраните маркер SAS, созданный в конце каждого метода. Маркер будет выглядеть примерно так, как показано ниже.

```output
?sv=2018-03-28&sr=c&si=myPolicyPS&sig=NAxefF%2BrR2ubjZtyUtuAvLQgt%2FJIN5aHJMj6OsDwyy4%3D
```

### <a name="using-powershell"></a>Использование PowerShell

Замените `RESOURCEGROUP` , `STORAGEACCOUNT` и `STORAGECONTAINER` соответствующими значениями для существующего контейнера хранилища. Измените каталог на `hdinsight-dotnet-python-azure-storage-shared-access-signature-master` или измените параметр, `-File` чтобы он содержал абсолютный путь для `Set-AzStorageblobcontent` . Введите следующую команду PowerShell:

```powershell
$resourceGroupName = "RESOURCEGROUP"
$storageAccountName = "STORAGEACCOUNT"
$containerName = "STORAGECONTAINER"
$policy = "myPolicyPS"

# Login to your Azure subscription
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

# Get the access key for the Azure Storage account
$storageAccountKey = (Get-AzStorageAccountKey `
                                -ResourceGroupName $resourceGroupName `
                                -Name $storageAccountName)[0].Value

# Create an Azure Storage context
$storageContext = New-AzStorageContext `
                                -StorageAccountName $storageAccountName `
                                -StorageAccountKey $storageAccountKey

# Create a stored access policy for the Azure storage container
New-AzStorageContainerStoredAccessPolicy `
   -Container $containerName `
   -Policy $policy `
   -Permission "rl" `
   -ExpiryTime "12/31/2025 08:00:00" `
   -Context $storageContext

# Get the stored access policy or policies for the Azure storage container
Get-AzStorageContainerStoredAccessPolicy `
    -Container $containerName `
    -Context $storageContext

# Generates an SAS token for the Azure storage container
New-AzStorageContainerSASToken `
    -Name $containerName `
    -Policy $policy `
    -Context $storageContext

<# Removes a stored access policy from the Azure storage container
Remove-AzStorageContainerStoredAccessPolicy `
    -Container $containerName `
    -Policy $policy `
    -Context $storageContext
#>

# upload a file for a later example
Set-AzStorageblobcontent `
    -File "./sampledata/sample.log" `
    -Container $containerName `
    -Blob "samplePS.log" `
    -Context $storageContext
```

### <a name="using-azure-cli"></a>Использование Azure CLI

Использование переменных в этом разделе основано на среде Windows. Для bash или других сред потребуются небольшие вариации.

1. Замените `STORAGEACCOUNT` и `STORAGECONTAINER` соответствующими значениями для существующего контейнера хранилища.

    ```azurecli
    # set variables
    set AZURE_STORAGE_ACCOUNT=STORAGEACCOUNT
    set AZURE_STORAGE_CONTAINER=STORAGECONTAINER

    #Login
    az login

    # If you have multiple subscriptions, set the one to use
    # az account set --subscription SUBSCRIPTION

    # Retrieve the primary key for the storage account
    az storage account keys list --account-name %AZURE_STORAGE_ACCOUNT% --query "[0].{PrimaryKey:value}" --output table
    ```

2. Задайте для полученного первичного ключа переменную для последующего использования. Замените на `PRIMARYKEY` полученное значение на предыдущем шаге, а затем введите следующую команду:

    ```console
    #set variable for primary key
    set AZURE_STORAGE_KEY=PRIMARYKEY
    ```

3. Измените каталог на `hdinsight-dotnet-python-azure-storage-shared-access-signature-master` или измените параметр, `--file` чтобы он содержал абсолютный путь для `az storage blob upload` . Выполните оставшиеся команды:

    ```azurecli
    # Create stored access policy on the containing object
    az storage container policy create --container-name %AZURE_STORAGE_CONTAINER% --name myPolicyCLI --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT% --expiry 2025-12-31 --permissions rl

    # List stored access policies on a containing object
    az storage container policy list --container-name %AZURE_STORAGE_CONTAINER% --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT%

    # Generate a shared access signature for the container
    az storage container generate-sas --name %AZURE_STORAGE_CONTAINER% --policy-name myPolicyCLI --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT%

    # Reversal
    # az storage container policy delete --container-name %AZURE_STORAGE_CONTAINER% --name myPolicyCLI --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT%

    # upload a file for a later example
    az storage blob upload --container-name %AZURE_STORAGE_CONTAINER% --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT% --name sampleCLI.log --file "./sampledata/sample.log"
    ```

### <a name="using-python"></a>Использование Python

Откройте `SASToken.py` файл и замените `storage_account_name` значения, `storage_account_key` и `storage_container_name` соответствующими значениями для существующего контейнера хранилища, а затем запустите скрипт.

Может потребоваться выполнить, `pip install --upgrade azure-storage` Если появится сообщение об ошибке `ImportError: No module named azure.storage` .

### <a name="using-c"></a>Использование C\#

1. Откройте решение в Visual Studio.

2. В обозреватель решений щелкните правой кнопкой мыши проект **сасексампле** и выберите пункт **свойства**.

3. Выберите **Параметры** и добавьте значения для следующих записей:

    |Элемент |Описание |
    |---|---|
    |StorageConnectionString|Строка подключения к учетной записи хранения, для которой необходимо создать хранимую политику и SAS. Требуется формат: `DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey`, где `myaccount` — это имя вашей учетной записи хранения, а `mykey` — ключ к учетной записи хранения.|
    |ContainerName;|Контейнер в учетной записи хранения, доступ к которому необходимо ограничить.|
    |SASPolicyName|Имя, которое будет использоваться для создаваемой хранимой политики.|
    |FileToUpload|Путь к файлу, который будет отправляться в контейнер.|

4. Запустите проект. Сохраните маркер политики SAS, имя учетной записи хранения и имя контейнера. Эти значения используются для связи учетной записи хранения с кластером HDInsight.

## <a name="use-the-sas-with-hdinsight"></a>Использование SAS с HDInsight

При создании кластера HDInsight необходимо указать основную учетную запись хранения. Можно также указать дополнительные учетные записи хранения. Оба способа добавления хранилища требуют полного доступа к соответствующим учетным записям хранения и контейнерам.

Используйте подписанный URL-доступ для ограничения доступа к контейнеру. Добавьте пользовательскую запись в конфигурацию **основного сайта** для кластера. Эту запись можно добавить во время создания кластера с помощью PowerShell или после создания кластера с помощью Ambari.

### <a name="create-a-cluster-that-uses-the-sas"></a>Создание кластера, использующего SAS

Замените `CLUSTERNAME` , `RESOURCEGROUP` , `DEFAULTSTORAGEACCOUNT` , `STORAGECONTAINER` , `STORAGEACCOUNT` и `TOKEN` соответствующими значениями. Введите команды PowerShell:

```powershell
$clusterName = 'CLUSTERNAME'
$resourceGroupName = 'RESOURCEGROUP'

# Replace with the Azure data center you want to the cluster to live in
$location = 'eastus'

# Replace with the name of the default storage account TO BE CREATED
$defaultStorageAccountName = 'DEFAULTSTORAGEACCOUNT'

# Replace with the name of the SAS container CREATED EARLIER
$SASContainerName = 'STORAGECONTAINER'

# Replace with the name of the SAS storage account CREATED EARLIER
$SASStorageAccountName = 'STORAGEACCOUNT'

# Replace with the SAS token generated earlier
$SASToken = 'TOKEN'

# Default cluster size (# of worker nodes), version, and type
$clusterSizeInNodes = "4"
$clusterVersion = "3.6"
$clusterType = "Hadoop"

# Login to your Azure subscription
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

# Create an Azure Storage account and container
New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $defaultStorageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableHttpsTrafficOnly 1

$defaultStorageAccountKey = (Get-AzStorageAccountKey `
                                -ResourceGroupName $resourceGroupName `
                                -Name $defaultStorageAccountName)[0].Value

$defaultStorageContext = New-AzStorageContext `
                                -StorageAccountName $defaultStorageAccountName `
                                -StorageAccountKey $defaultStorageAccountKey

# Create a blob container. This holds the default data store for the cluster.
New-AzStorageContainer `
    -Name $clusterName `
    -Context $defaultStorageContext

# Cluster login is used to secure HTTPS services hosted on the cluster
$httpCredential = Get-Credential `
    -Message "Enter Cluster login credentials" `
    -UserName "admin"

# SSH user is used to remotely connect to the cluster using SSH clients
$sshCredential = Get-Credential `
    -Message "Enter SSH user credentials" `
    -UserName "sshuser"

# Create the configuration for the cluster
$config = New-AzHDInsightClusterConfig

$config = $config | Add-AzHDInsightConfigValue `
    -Spark2Defaults @{} `
    -Core @{"fs.azure.sas.$SASContainerName.$SASStorageAccountName.blob.core.windows.net"=$SASToken}

# Create the HDInsight cluster
New-AzHDInsightCluster `
    -Config $config `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $clusterName `
    -Location $location `
    -ClusterSizeInNodes $clusterSizeInNodes `
    -ClusterType $clusterType `
    -OSType Linux `
    -Version $clusterVersion `
    -HttpCredential $httpCredential `
    -SshCredential $sshCredential `
    -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
    -DefaultStorageAccountKey $defaultStorageAccountKey `
    -DefaultStorageContainer $clusterName

<# REVERSAL
Remove-AzHDInsightCluster `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $clusterName

Remove-AzStorageContainer `
    -Name $clusterName `
    -Context $defaultStorageContext

Remove-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $defaultStorageAccountName

Remove-AzResourceGroup `
    -Name $resourceGroupName
#>
```

> [!IMPORTANT]  
> При появлении запроса на имя и пароль пользователя HTTP/s или SSH необходимо указать пароль, который отвечает следующим условиям:
>
> * содержит не меньше 10 символов;
> * содержит хотя бы одну цифру;
> * содержит хотя бы один специальный символ;
> * содержит хотя бы одну букву в верхнем или нижнем регистре.

Обычно выполнение сценария занимает около 15 минут. Если сценарий выполнен без ошибок, значит, кластер создан.

### <a name="use-the-sas-with-an-existing-cluster"></a>Использование SAS в существующем кластере

Если у вас уже есть кластер, можно добавить SAS в конфигурацию **основного сайта** , выполнив следующие действия.

1. Откройте веб-интерфейс Ambari для вашего кластера. Адрес этой страницы: `https://YOURCLUSTERNAME.azurehdinsight.net`. При появлении запроса пройдите проверку подлинности, указав имя пользователя и пароль администратора, которые использовались при создании кластера.

1. Перейдите в раздел  >  **конфигурации** HDFS  >  **Расширенные**  >  **пользовательские основные компоненты — сайт**.

1. Разверните раздел **пользовательское ядро — сайт** , прокрутите до конца и выберите **Добавить свойство...**. Используйте следующие значения для **ключа** и **значения**:

    * **Ключ:** `fs.azure.sas.CONTAINERNAME.STORAGEACCOUNTNAME.blob.core.windows.net`
    * **Значение**: SAS, возвращенный одним из методов, выполненных ранее.

    Замените на `CONTAINERNAME` имя контейнера, используемого в приложении C# или SAS. Замените на `STORAGEACCOUNTNAME` имя учетной записи хранения, которую вы использовали.

    Выберите **Добавить** , чтобы сохранить этот ключ и значение

1. Нажмите кнопку **сохранить** , чтобы сохранить изменения конфигурации. При появлении запроса добавьте описание изменения (например, "Добавление доступа к хранилищу SAS") и нажмите кнопку **сохранить**.

    После завершения изменений нажмите кнопку **ОК** .

   > [!IMPORTANT]  
   > Чтобы изменения вступили в силу, необходимо перезапустить несколько служб.

1. Появится раскрывающийся список **Перезагрузка** . Выберите **перезапустить все затронутые** в раскрывающемся списке и __Подтвердите перезагрузка__.

    Повторите эту процедуру для **MapReduce2** и **YARN**.

1. Перезапустив службы, выберите одну из них и отключите режим обслуживания в раскрывающемся списке **Учетные записи обслуживания**.

## <a name="test-restricted-access"></a>Тестирование ограниченного доступа

Выполните следующие действия, чтобы убедиться, что вы можете читать и просматривать элементы только в учетной записи хранения SAS.

1. Подключитесь к кластеру. Замените `CLUSTERNAME` именем кластера и введите следующую команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Чтобы вывести содержимое контейнера, выполните следующую команду в командной строке:

    ```bash
    hdfs dfs -ls wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/
    ```

    Замените на `SASCONTAINER` имя контейнера, созданного для учетной записи хранения SAS. Замените `SASACCOUNTNAME` именем учетной записи хранения, используемой для SAS.

    Список включает файл, отправленный при создании контейнера и SAS.

3. Проверьте, можете ли вы прочитать содержимое файла, выполнив следующую команду: Замените `SASCONTAINER` и, `SASACCOUNTNAME` как в предыдущем шаге. Замените на `sample.log` имя файла, отображаемого в предыдущей команде:

    ```bash
    hdfs dfs -text wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/sample.log
    ```

    Эта команда выводит содержимое файла.

4. Загрузите файл в локальную файловую систему, выполнив следующую команду:

    ```bash
    hdfs dfs -get wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/sample.log testfile.txt
    ```

    Эта команда загружает на локальный компьютер файл с именем **testfile.txt**.

5. Передайте локальный файл в новый файл с именем **testupload.txt** в хранилище SAS, выполнив следующую команду:

    ```bash
    hdfs dfs -put testfile.txt wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/testupload.txt
    ```

    Должно появиться сообщение следующего вида:

    ```output
    put: java.io.IOException
    ```

    Эта ошибка возникает из-за того, что расположение хранилища доступно только для чтения и включения в списки. Разместите данные в хранилище по умолчанию доступного для записи кластера, выполнив следующую команду:

    ```bash
    hdfs dfs -put testfile.txt wasbs:///testupload.txt
    ```

    На этот раз операция должна завершиться успешно.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как добавить хранилище с ограниченным доступом в кластер HDInsight, Узнайте о других способах работы с данными в кластере:

* [Использование SSH с HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md)
* [Предоставление пользователям доступа к представлениям Apache Ambari](hdinsight-authorize-users-to-ambari.md)
