---
title: Миграция в средства Azure Resource Manager для HDInsight
description: Процесс перехода к средствам разработки на основе Azure Resource Manager для кластеров HDInsight
ms.service: hdinsight
ms.custom: hdinsightactive, devx-track-azurecli
ms.topic: how-to
ms.date: 02/21/2018
ms.openlocfilehash: a8f808cd43f96f26db0de28e8059d02d9488320a
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102434667"
---
# <a name="migrating-to-azure-resource-manager-based-development-tools-for-hdinsight-clusters"></a>Переход к средствам разработки на основе Azure Resource Manager для кластеров HDInsight

Средства для HDInsight на основе диспетчера служб Azure (ASM) устарели. Если для работы с кластерами HDInsight вы используете Azure PowerShell, классический Azure CLI или пакет SDK для HDInsight .NET, рекомендуем перейти на версии PowerShell, CLI и пакета SDK для .NET на основе Azure Resource Manager. В этой статье рассказывается, как перейти на средства, основанные на Azure Resource Manager. В этом документе везде, где это применимо, подчеркиваются различия между подходами ASM и Resource Manager для HDInsight.

> [!IMPORTANT]  
> Поддержка PowerShell, интерфейса командной строки и пакета SDK для .NET на основе ASM будет прекращена **1 января 2017 года**.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="migrating-azure-classic-cli-to-azure-resource-manager"></a>Переход с классического Azure CLI на Azure Resource Manager

> [!IMPORTANT]  
> Azure CLI не обеспечивает поддержку работы с кластерами HDInsight. Хотя классический Azure CLI устарел, его все еще можно использовать для HDInsight.

Ниже приведены основные команды для работы с HDInsight посредством классического Azure CLI:

* `azure hdinsight cluster create` — создает кластер HDInsight;
* `azure hdinsight cluster delete` — удаляет кластер HDInsight;
* `azure hdinsight cluster show` — отображает информацию о существующем кластере;
* `azure hdinsight cluster list` — выдает список кластеров HDInsight для подписки Azure.

Проверить, какие параметры и переключатели доступны для каждой команды, позволяет переключатель `-h` .

### <a name="new-commands"></a>Новые команды
В диспетчере ресурсов Azure появились следующие команды:

* `azure hdinsight cluster resize` — динамически изменяет количество рабочих узлов в кластере;
* `azure hdinsight cluster enable-http-access` — обеспечивает HTTPs-доступ к кластеру (по умолчанию включен);
* `azure hdinsight cluster disable-http-access` — отключает HTTPs-доступ к кластеру;
* `azure hdinsight script-action` — предоставляет команды для создания действий сценариев в кластере и управления этими действиями;
* `azure hdinsight config` — предоставляет команды для создания файла конфигурации, который можно использовать с командой `hdinsight cluster create` для предоставления сведений о конфигурации.

### <a name="deprecated-commands"></a>Устаревшие команды
При использовании команд `azure hdinsight job` для отправки заданий в кластер HDInsight эти команды недоступны в списке команд диспетчера ресурсов. Если задания из сценариев необходимо отправлять в HDInsight программными средствами, используйте API REST, предоставляемый в HDInsight. Дополнительные сведения об отправке заданий с использованием API REST см. в следующих документах:

* [Выполнение заданий MapReduce с помощью cURL с использованием Hadoop в HDInsight](hadoop/apache-hadoop-use-mapreduce-curl.md)
* [Выполнение запросов Hive в Apache Hadoop в HDInsight с использованием REST](hadoop/apache-hadoop-use-hive-curl.md)


Сведения о других методах интерактивного выполнения Apache Hadoop MapReduce, Apache Hive и Apache Pig см. в статьях [Использование MapReduce в HDInsight](hadoop/hdinsight-use-mapreduce.md), [Обзор Apache Hive и HiveQL в Azure HDInsight](hadoop/hdinsight-use-hive.md) и [Использование Apache Pig с Apache Hadoop в HDInsight](./index.yml).

### <a name="examples"></a>Примеры
**Создание кластера**

* Старая команда (ASM) — `azure hdinsight cluster create myhdicluster --location northeurope --osType linux --storageAccountName mystorage --storageAccountKey <storagekey> --storageContainer mycontainer --userName admin --password mypassword --sshUserName sshuser --sshPassword mypassword`
* Новая команда — `azure hdinsight cluster create myhdicluster -g myresourcegroup --location northeurope --osType linux --clusterType hadoop --defaultStorageAccountName mystorage --defaultStorageAccountKey <storagekey> --defaultStorageContainer mycontainer --userName admin -password mypassword --sshUserName sshuser --sshPassword mypassword`

**Удаление кластера**

* Старая команда (ASM) — `azure hdinsight cluster delete myhdicluster`
* Новая команда — `azure hdinsight cluster delete mycluster -g myresourcegroup`

**Получение списка кластеров**

* Старая команда (ASM) — `azure hdinsight cluster list`
* Новая команда — `azure hdinsight cluster list`

> [!NOTE]  
> Для команды списка при добавлении `-g` к группе ресурсов возвращаются только кластеры, входящие в указанную группу ресурсов.

**Отображение данных кластера**

* Старая команда (ASM) — `azure hdinsight cluster show myhdicluster`
* Новая команда — `azure hdinsight cluster show myhdicluster -g myresourcegroup`

## <a name="migrating-azure-powershell-to-azure-resource-manager"></a>Переход с Azure PowerShell на диспетчер ресурсов Azure
Общие сведения об Azure PowerShell в режиме Azure Resource Manager (ARM) см. в статье [Управление ресурсами с помощью Azure PowerShell](../azure-resource-manager/management/manage-resources-powershell.md).

Командлеты диспетчера ресурсов Azure PowerShell могут устанавливаться параллельно с командлетами ASM. Командлеты двух режимов можно различать по именам.  Диспетчер ресурсов режим имеет *аздинсигхт* в именах командлетов, которые сравниваются с *AzureHDInsight* в более старом режиме управления службами Azure.  Например, *New-аздинсигхтклустер* и *New-азурехдинсигхтклустер*. Параметры и переключатели могут иметь новые имена, а при использовании диспетчера ресурсов появляется доступ к множеству новых параметров.  Например, для некоторых командлетов требуется новый переключатель *-ResourceGroupName*.

Перед использованием командлетов HDInsight необходимо подключиться к учетной записи Azure и создать новую группу ресурсов:

* [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount)
* [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)

### <a name="renamed-cmdlets"></a>Переименованные командлеты
Получение списка командлетов HDInsight ASM в консоли Windows PowerShell:

```powershell
help *azurehdinsight*
```

В следующей таблице перечислены командлеты ASM и их имена в режиме диспетчера ресурсов:

| Командлеты ASM | Командлеты диспетчера ресурсов |
| --- | --- |
| Add-AzureHDInsightConfigValue |[Add-Аздинсигхтконфигвалуе](/powershell/module/az.hdinsight/add-azhdinsightconfigvalue) |
| Add-AzureHDInsightMetastore |[Add-Аздинсигхтметасторе](/powershell/module/az.hdinsight/add-azhdinsightmetastore) |
| Add-AzureHDInsightScriptAction |[Add-Аздинсигхтскриптактион](/powershell/module/az.hdinsight/add-azhdinsightscriptaction) |
| Add-AzureHDInsightStorage |[Add-Аздинсигхтстораже](/powershell/module/az.hdinsight/add-azhdinsightstorage) |
| Get-AzureHDInsightCluster |[Get-Аздинсигхтклустер](/powershell/module/az.hdinsight/get-azhdinsightcluster) |
| Get-AzureHDInsightJob |[Get-Аздинсигхтжоб](/powershell/module/az.hdinsight/get-azhdinsightjob) |
| Get-AzureHDInsightJobOutput |[Get-Аздинсигхтжобаутпут](/powershell/module/az.hdinsight/get-azhdinsightjoboutput) |
| Get-AzureHDInsightProperty |[Get-Аздинсигхтпроперти](/powershell/module/az.hdinsight/get-azhdinsightproperty) |
| Grant-AzureHDInsightHttpServicesAccess |[Grant-AzureRmHDInsightHttpServicesAccess](/powershell/module/azurerm.hdinsight/grant-azurermhdinsighthttpservicesaccess) |
| Grant-AzureHdinsightRdpAccess |[Grant-AzHDInsightRdpServicesAccess](/powershell/module/az.hdinsight/grant-azhdinsightrdpservicesaccess) |
| Invoke-AzureHDInsightHiveJob |[Invoke-Аздинсигхсивежоб](/powershell/module/az.hdinsight/invoke-azhdinsighthivejob) |
| New-AzureHDInsightCluster |[New-Аздинсигхтклустер](/powershell/module/az.hdinsight/new-azhdinsightcluster) |
| New-AzureHDInsightClusterConfig |[New-Аздинсигхтклустерконфиг](/powershell/module/az.hdinsight/new-azhdinsightclusterconfig) |
| New-AzureHDInsightHiveJobDefinition |[New-Аздинсигхсивежобдефинитион](/powershell/module/az.hdinsight/new-azhdinsighthivejobdefinition) |
| New-AzureHDInsightMapReduceJobDefinition |[New-Аздинсигхтмапредуцежобдефинитион](/powershell/module/az.hdinsight/new-azhdinsightmapreducejobdefinition) |
| New-AzureHDInsightPigJobDefinition |[New-Аздинсигхтпигжобдефинитион](/powershell/module/az.hdinsight/new-azhdinsightpigjobdefinition) |
| New-AzureHDInsightSqoopJobDefinition |[New-Аздинсигхтскупжобдефинитион](/powershell/module/az.hdinsight/new-azhdinsightsqoopjobdefinition) |
| New-AzureHDInsightStreamingMapReduceJobDefinition |[New-Аздинсигхтстреамингмапредуцежобдефинитион](/powershell/module/az.hdinsight/new-azhdinsightstreamingmapreducejobdefinition) |
| Remove-AzureHDInsightCluster |[Remove-Аздинсигхтклустер](/powershell/module/az.hdinsight/remove-azhdinsightcluster) |
| Revoke-AzureHDInsightHttpServicesAccess |[Revoke-AzHDInsightHttpServicesAccess](/powershell/module/azurerm.hdinsight/revoke-azurermhdinsighthttpservicesaccess) |
| Revoke-AzureHdinsightRdpAccess |[Revoke-AzHDInsightRdpServicesAccess](/powershell/module/az.hdinsight/revoke-azhdinsightrdpservicesaccess) |
| Set-AzureHDInsightClusterSize |[Set-Аздинсигхтклустерсизе](/powershell/module/az.hdinsight/set-azhdinsightclustersize) |
| Set-AzureHDInsightDefaultStorage |[Set-Аздинсигхтдефаултстораже](/powershell/module/az.hdinsight/set-azhdinsightdefaultstorage) |
| Start-AzureHDInsightJob |[Start-Аздинсигхтжоб](/powershell/module/az.hdinsight/start-azhdinsightjob) |
| Stop-AzureHDInsightJob |[Аздинсигхтжоб](/powershell/module/az.hdinsight/stop-azhdinsightjob) |
| Use-AzureHDInsightCluster |[Use-Аздинсигхтклустер](/powershell/module/az.hdinsight/use-azhdinsightcluster) |
| Wait-AzureHDInsightJob |[Wait-Аздинсигхтжоб](/powershell/module/az.hdinsight/wait-azhdinsightjob) |

### <a name="new-cmdlets"></a>Новые командлеты
Ниже перечислены новые командлеты, доступные только в режиме диспетчера ресурсов. 

**Командлеты, связанные с действиями сценариев:**

* **Get-аздинсигхтперсистедскриптактион**: возвращает сохраненные действия скрипта для кластера и перечисляет их в хронологическом порядке или получает сведения для указанного сохраненного действия скрипта. 
* **Get-аздинсигхтскриптактионхистори**: Получает журнал действий скрипта для кластера и перечисляет его в обратный хронологический порядок или получает сведения о ранее выполненном действии сценария. 
* **Remove-аздинсигхтперсистедскриптактион**: удаляет сохраненное действие скрипта из кластера HDInsight.
* **Set-аздинсигхтперсистедскриптактион**: задает ранее выполненное действие сценария как сохраненное действие скрипта.
* **Submit-аздинсигхтскриптактион**: отправляет новое действие скрипта в кластер Azure HDInsight. 

Дополнительные сведения об использовании см. в статье [Настройка кластеров HDInsight под управлением Linux с помощью действия сценария](hdinsight-hadoop-customize-cluster-linux.md).

**Командлеты, связанные с идентификатором кластера:**

* **Add-аздинсигхтклустеридентити**: добавляет удостоверение кластера в объект конфигурации кластера, чтобы кластер HDInsight мог получить доступ к Azure Data Lake Storage. См. статью [Создание кластера HDInsight с Data Lake Storage с помощью Azure PowerShell](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell.md).

### <a name="examples"></a>Примеры
**Создать кластер**

Старая команда (ASM): 

```azurepowershell
New-AzureHDInsightCluster `
    -Name $clusterName `
    -Location $location `
    -DefaultStorageAccountName "$storageAccountName.blob.core.windows.net" `
    -DefaultStorageAccountKey $storageAccountKey `
    -DefaultStorageContainerName $containerName `
    -ClusterSizeInNodes 2 `
    -ClusterType Hadoop `
    -OSType Linux `
    -Version "3.2" `
    -Credential $httpCredential `
    -SshCredential $sshCredential
```

Новая команда:

```azurepowershell
New-AzHDInsightCluster `
    -ClusterName $clusterName `
    -ResourceGroupName $resourceGroupName `
    -Location $location `
    -DefaultStorageAccountName "$storageAccountName.blob.core.windows.net" `
    -DefaultStorageAccountKey $storageAccountKey `
    -DefaultStorageContainer $containerName  `
    -ClusterSizeInNodes 2 `
    -ClusterType Hadoop `
    -OSType Linux `
    -Version "3.2" `
    -HttpCredential $httpcredentials `
    -SshCredential $sshCredentials
```

**Удаление кластера**

Старая команда (ASM):

```azurepowershell
Remove-AzureHDInsightCluster -name $clusterName 
```

Новая команда:

```azurepowershell
Remove-AzHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $clusterName
```

**Список кластеров**

Старая команда (ASM):

```azurepowershell
Get-AzureHDInsightCluster
```

Новая команда:

```azurepowershell
Get-AzHDInsightCluster
```

**Отображение кластеров**

Старая команда (ASM):

```azurepowershell
Get-AzureHDInsightCluster -Name $clusterName
```

Новая команда:

```azurepowershell
Get-AzHDInsightCluster -ResourceGroupName $resourceGroupName -clusterName $clusterName
```

#### <a name="other-samples"></a>Другие примеры
* [Создание кластеров HDInsight](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)
* [Отправка заданий Apache Hive](hadoop/apache-hadoop-use-hive-powershell.md)
* [Отправка заданий Apache Sqoop](hadoop/apache-hadoop-use-sqoop-powershell.md)

## <a name="migrating-to-the-new-hdinsight-net-sdk"></a>Переход на новый пакет SDK для HDInsight .NET
[Пакет SDK для HDInsight .NET (ASM)](/previous-versions/azure/reference/mt416619(v=azure.100)) на основе управления службами Azure устарел. Настоятельно рекомендуем использовать [пакет SDK для HDInsight .NET на основе диспетчера ресурсов](/dotnet/api/overview/azure/hdinsight) на базе Azure Resource Management. Следующие пакеты HDInsight на основе ASM устарели:

* `Microsoft.WindowsAzure.Management.HDInsight`
* `Microsoft.Hadoop.Client`

Этот раздел содержит ссылки на дополнительные сведения о том, как выполнять определенные задачи, используя пакет SDK на основе диспетчера ресурсов.

| Инструкции по выполнению операций с помощью пакета SDK HDInsight на базе диспетчера ресурсов | Ссылки |
| --- | --- |
| Пакет SDK Azure HDInsight для .NET|См. раздел [Azure HDINSIGHT SDK для .NET](/dotnet/api/overview/azure/hdinsight) . |
| Интерактивная проверка подлинности приложений с Azure Active Directory c использованием пакета SDK для .NET |См. статью о [выполнении запросов Apache Hive с помощью пакета SDK для .NET](hadoop/apache-hadoop-use-hive-dotnet-sdk.md). Во фрагменте кода, представленном в этой статье, используется метод интерактивной проверки подлинности. |
| Неинтерактивная проверка подлинности приложений с Azure Active Directory c использованием пакета SDK для .NET |См. статью [Создание приложений .NET HDInsight с неинтерактивной проверкой подлинности](hdinsight-create-non-interactive-authentication-dotnet-applications.md). |
| Отправка задания Hive с помощью пакета SDK для .NET |См. статью об [отправке заданий Apache Hive](hadoop/apache-hadoop-use-hive-dotnet-sdk.md). |
| Отправка задания Sqoop с помощью пакета SDK для .NET |См. статью об [отправке заданий Apache Sqoop](hadoop/apache-hadoop-use-sqoop-dotnet-sdk.md). |
| Получение списка кластеров HDInsight с помощью пакета SDK для .NET |См. раздел [Получение списка кластеров](hdinsight-administer-use-dotnet-sdk.md#list-clusters). |
| Масштабирование кластеров HDInsight с помощью пакета SDK для .NET |См. раздел [Масштабирование кластеров](hdinsight-administer-use-dotnet-sdk.md#scale-clusters). |
| Предоставление и отмена доступа к кластерам HDInsight с помощью пакета SDK для .NET |См. раздел [Предоставление и отмена доступа](hdinsight-administer-use-dotnet-sdk.md#grantrevoke-access). |
| Обновление учетных данных пользователя HTTP для кластеров HDInsight с помощью пакета SDK для .NET |См. раздел [Обновление учетных данных пользователя HTTP](hdinsight-administer-use-dotnet-sdk.md#update-http-user-credentials). |
| Поиск учетной записи хранения по умолчанию для кластеров HDInsight с помощью пакета SDK для .NET |См. раздел [Поиск учетной записи хранения по умолчанию](hdinsight-administer-use-dotnet-sdk.md#find-the-default-storage-account). |
| Удаление кластеров HDInsight с помощью пакета SDK для .NET |См. раздел [Удаление кластеров](hdinsight-administer-use-dotnet-sdk.md#delete-clusters). |

### <a name="examples"></a>Примеры
Ниже представлены некоторые примеры выполнения операций с использованием пакета SDK на основе ASM и фрагмент аналогичного кода для пакета SDK на основе диспетчера ресурсов.

**Создание CRUD-клиента кластера**

* Старая команда (ASM)

  ```azurecli
  //Certificate auth
  //This logs the application in using a subscription administration certificate, which is not offered in Azure Resource Manager
  
  const string subid = "454467d4-60ca-4dfd-a556-216eeeeeeee1";
  var cred = new HDInsightCertificateCredential(new Guid(subid), new X509Certificate2(@"path\to\certificate.cer"));
  var client = HDInsightClient.Connect(cred);
  ```
* Новая команда (авторизация субъекта-службы)

  ```azurecli
  //Service principal auth
  //This will log the application in as itself, rather than on behalf of a specific user.
  //For details, including how to set up the application, see:
  //   https://azure.microsoft.com/documentation/articles/hdinsight-create-non-interactive-authentication-dotnet-applications/
  
  var authFactory = new AuthenticationFactory();
  
  var account = new AzureAccount { Type = AzureAccount.AccountType.ServicePrincipal, Id = clientId };
  
  var env = AzureEnvironment.PublicEnvironments[EnvironmentName.AzureCloud];
  
  var accessToken = authFactory.Authenticate(account, env, tenantId, secretKey, ShowDialog.Never).AccessToken;
  
  var creds = new TokenCloudCredentials(subId.ToString(), accessToken);
  
  _hdiManagementClient = new HDInsightManagementClient(creds);
  ```

* Новая команда (авторизация пользователя)

  ```azurecli
  //User auth
  //This will log the application in on behalf of the user.
  //The end-user will see a login popup.
  
  var authFactory = new AuthenticationFactory();
  
  var account = new AzureAccount { Type = AzureAccount.AccountType.User, Id = username };
  
  var env = AzureEnvironment.PublicEnvironments[EnvironmentName.AzureCloud];
  
  var accessToken = authFactory.Authenticate(account, env, AuthenticationFactory.CommonAdTenant, password, ShowDialog.Auto).AccessToken;
  
  var creds = new TokenCloudCredentials(subId.ToString(), accessToken);
  
  _hdiManagementClient = new HDInsightManagementClient(creds);
  ```

**Создание кластера**

* Старая команда (ASM)

  ```azurecli
  var clusterInfo = new ClusterCreateParameters
              {
                  Name = dnsName,
                  DefaultStorageAccountKey = key,
                  DefaultStorageContainer = defaultStorageContainer,
                  DefaultStorageAccountName = storageAccountDnsName,
                  ClusterSizeInNodes = 1,
                  ClusterType = type,
                  Location = "West US",
                  UserName = "admin",
                  Password = "*******",
                  Version = version,
                  HeadNodeSize = NodeVMSize.Large,
              };
  clusterInfo.CoreConfiguration.Add(new KeyValuePair<string, string>("config1", "value1"));
  client.CreateCluster(clusterInfo);
  ```

* Новая команда

  ```azurecli
  var clusterCreateParameters = new ClusterCreateParameters
      {
          Location = "West US",
          ClusterType = "Hadoop",
          Version = "3.1",
          OSType = OSType.Linux,
          DefaultStorageAccountName = "mystorage.blob.core.windows.net",
          DefaultStorageAccountKey =
              "O9EQvp3A3AjXq/W27rst1GQfLllhp0gUeiUUn2D8zX2lU3taiXSSfqkZlcPv+nQcYUxYw==",
          UserName = "hadoopuser",
          Password = "*******",
          HeadNodeSize = "ExtraLarge",
          RdpUsername = "hdirp",
          RdpPassword = ""*******",
          RdpAccessExpiry = new DateTime(2025, 3, 1),
          ClusterSizeInNodes = 5
      };
  var coreConfigs = new Dictionary<string, string> {{"config1", "value1"}};
  clusterCreateParameters.Configurations.Add(ConfigurationKey.CoreSite, coreConfigs);
  ```

**Включение доступа по протоколу HTTP**

* Старая команда (ASM)

  ```azurecli
  client.EnableHttp(dnsName, "West US", "admin", "*******");
  ```

* Новая команда
  
  ```azurecli
  var httpParams = new HttpSettingsParameters
  {
         HttpUserEnabled = true,
         HttpUsername = "admin",
         HttpPassword = "*******",
  };
  client.Clusters.ConfigureHttpSettings(resourceGroup, dnsname, httpParams);
  ```

**Удаление кластера**

* Старая команда (ASM)

  ```azurecli
  client.DeleteCluster(dnsName);
  ```

* Новая команда

  ```azurecli
  client.Clusters.Delete(resourceGroup, dnsname);
  ```
