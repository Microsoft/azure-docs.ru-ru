---
title: PowerShell — HDInsight с хранилищем Data Lake Storage 1-го поколения — Azure
description: Узнайте, как использовать Azure PowerShell для настройки кластера HDInsight с Azure Data Lake Storage 1-го поколения в качестве дополнительного хранилища.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: b7cb9d5c5c2ca850678d3f3194a9af8de526ada4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92103396"
---
# <a name="use-azure-powershell-to-create-an-hdinsight-cluster-with-azure-data-lake-storage-gen1-as-additional-storage"></a>Использование Azure PowerShell для создания кластера HDInsight с Azure Data Lake Storage 1-го поколения (в качестве дополнительного хранилища)

> [!div class="op_single_selector"]
> * [Использование портала](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Использование PowerShell (для хранилища по умолчанию)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [Использование PowerShell (для дополнительного хранилища)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [Использование Resource Manager](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)
>
>

Узнайте, как с помощью Azure PowerShell настроить кластер HDInsight с Azure Data Lake Storage 1-го поколения в качестве **дополнительного хранилища**. Инструкции по созданию кластера HDInsight с Data Lake Storage 1-го поколения в качестве хранилища по умолчанию см. в статье [Создание кластеров, использующих Data Lake Storage 1-го в качестве хранилища по умолчанию, с помощью PowerShell](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md).

> [!NOTE]
> Если вы собираетесь добавить Data Lake Storage 1-го поколения в качестве дополнительного хранилища для кластера HDInsight, настоятельно рекомендуется сделать это во время создания кластера, как описано в этой статье. Добавление Data Lake Storage 1-го поколения в качестве дополнительного хранилища для существующего кластера HDInsight — очень сложный процесс, который может привести к ошибкам.
>

В поддерживаемых типах кластеров Data Lake Storage 1-го поколения можно использовать в качестве хранилища по умолчанию или дополнительной учетной записи хранения. Если Data Lake Storage 1-го поколения используется в качестве дополнительного хранилища, учетная запись хранения по умолчанию для кластеров по-прежнему будет храниться в хранилище BLOB-объектов Azure (WASB), а файлы, связанные с кластером (такие как журналы и т. д.), по-прежнему будут записываться в хранилище по умолчанию, а данные, которые требуется обработать, могут храниться в Data Lake Storage 1-го поколения. Использование Data Lake Storage 1-го поколения в качестве дополнительной учетной записи хранения не влияет на производительность или возможность выполнять чтение и запись в хранилище из кластера.

## <a name="using-data-lake-storage-gen1-for-hdinsight-cluster-storage"></a>Использование Data Lake Storage 1-го поколения в качестве хранилища кластера HDInsight

Ниже приведены некоторые важные сведения об использовании HDInsight с Data Lake Storage 1-го поколения.

* Возможность создавать кластеры HDInsight с доступом к Data Lake Storage 1-го поколения в качестве дополнительного хранилища поддерживается для версий HDInsight 3.2, 3.4, 3.5 и 3.6.

Настройка HDInsight для использования Data Lake Storage 1-го поколения с помощью PowerShell состоит из нескольких этапов.

* Создание учетной записи Data Lake Storage 1-го поколения 
* Настройка проверки подлинности для доступа к Data Lake Storage 1-го поколения на основе ролей
* Создание кластера HDInsight с проверкой подлинности в Data Lake Storage 1-го поколения
* выполнение тестового задания в кластере.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Перед началом работы с этим учебником необходимо иметь следующее:

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1,0 или более поздней** версии. Ознакомьтесь со статьей [Установка и настройка Azure PowerShell](/powershell/azure/).
* **Windows SDK**. Его можно установить [отсюда](https://dev.windows.com/en-us/downloads). Пакет используется для создания сертификата безопасности.
* **Субъект-служба Azure Active Directory**. В этом учебнике приведены инструкции по созданию субъекта-службы в Azure AD. Однако, чтобы создать субъект-службу, необходимо быть администратором Azure AD. Если вы являетесь администратором Azure AD, то можете пропустить это предварительное требование и продолжить работу с учебником.

    **Если вы не являетесь администратором Azure AD**, то вы не сможете выполнить шаги, необходимые для создания субъекта-службы. В этом случае администратор Azure AD должен сначала создать субъект-службу, после чего вы сможете создать кластер HDInsight с Data Lake Storage 1-го поколения. При создании субъекта-службы также необходимо использовать сертификат, как описано в разделе [Create a service principal with certificate](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-certificate-from-certificate-authority) (Создание субъекта-службы с сертификатом).

## <a name="create-a-data-lake-storage-gen1-account"></a>Создание учетной записи Data Lake Storage 1-го поколения 
Чтобы создать учетную запись Data Lake Storage 1-го поколения, сделайте следующее.

1. На своем компьютере откройте новое окно Azure PowerShell и введите следующий фрагмент кода. Когда вам будет предложено войти, введите учетные данные администратора или владельца подписки:

    ```azurepowershell
    # Log in to your Azure account
    Connect-AzAccount

    # List all the subscriptions associated to your account
    Get-AzSubscription

    # Select a subscription
    Set-AzContext -SubscriptionId <subscription ID>

    # Register for Data Lake Storage Gen1
    Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
    ```

   > [!NOTE]
   > Если при регистрации поставщика ресурсов Data Lake Storage 1-го поколения появляется сообщение об ошибке, похожее на `Register-AzResourceProvider : InvalidResourceNamespace: The resource namespace 'Microsoft.DataLakeStore' is invalid` , то возможно, что подписка не утверждена для Data Lake Storage 1-го поколения. Убедитесь, что подписка Azure для Data Lake Storage 1-го поколения включена, выполнив указанные [инструкции](data-lake-store-get-started-portal.md).
   >
   >
2. Учетная запись хранения с Data Lake Storage 1-го поколения связана с группой ресурсов Azure. Для начала создайте группу ресурсов Azure.

    ```azurepowershell
    $resourceGroupName = "<your new resource group name>"
    New-AzResourceGroup -Name $resourceGroupName -Location "East US 2"
    ```

    Вы должны увидеть такие выходные данные:

    ```output
    ResourceGroupName : hdiadlgrp
    Location          : eastus2
    ProvisioningState : Succeeded
    Tags              :
    ResourceId        : /subscriptions/<subscription-id>/resourceGroups/hdiadlgrp
    ```

3. Создайте учетную запись хранения с Data Lake Storage 1-го поколения. Имя новой учетной записи должно содержать только строчные буквы и цифры.

    ```azurepowershell
    $dataLakeStorageGen1Name = "<your new storage account with Data Lake Storage Gen1 name>"
    New-AzDataLakeStoreAccount -ResourceGroupName $resourceGroupName -Name $dataLakeStorageGen1Name -Location "East US 2"
    ```

    Вы должны увидеть подобные выходные данные:

    ```output
    ...
    ProvisioningState           : Succeeded
    State                       : Active
    CreationTime                : 5/5/2017 10:53:56 PM
    EncryptionState             : Enabled
    ...
    LastModifiedTime            : 5/5/2017 10:53:56 PM
    Endpoint                    : hdiadlstore.azuredatalakestore.net
    DefaultGroup                :
    Id                          : /subscriptions/<subscription-id>/resourceGroups/hdiadlgrp/providers/Microsoft.DataLakeStore/accounts/hdiadlstore
    Name                        : hdiadlstore
    Type                        : Microsoft.DataLakeStore/accounts
    Location                    : East US 2
    Tags                        : {}
    ```

5. Отправьте пример данных в Data Lake Storage 1-го поколения. Позже мы проверим, доступны ли эти данные из кластера HDInsight. Если у вас нет под рукой подходящих для этих целей данных, передайте папку **Ambulance Data** из [репозитория Git для озера данных Azure](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData).

    ```azurepowershell
    $myrootdir = "/"
    Import-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name -Path "C:\<path to data>\vehicle1_09142014.csv" -Destination $myrootdir\vehicle1_09142014.csv
    ```

## <a name="set-up-authentication-for-role-based-access-to-data-lake-storage-gen1"></a>Настройка проверки подлинности для доступа к Data Lake Storage 1-го поколения на основе ролей

Каждая подписка Azure связана со службой Azure Active Directory. Пользователи и службы, получающие доступ к ресурсам подписки через портал Azure или API Azure Resource Manager, сначала должны пройти аутентификацию в соответствующей службе Azure Active Directory. Доступ к подпискам и службам Azure предоставляется путем назначения соответствующей роли для ресурса Azure.  Для служб субъект-служба идентифицирует службу в Azure Active Directory (Azure AD). В этом разделе показано, как предоставить службе приложений, например HDInsight, доступ к ресурсу Azure (учетную запись хранения с Data Lake Storage 1-го поколения, созданным ранее), создав субъект-службу для приложения и назначив ему роли с помощью Azure PowerShell.

Чтобы настроить для Data Lake Storage 1-го поколения проверку подлинности в Active Directory, вам необходимо сделать следующее.

* Создание самозаверяющего сертификата
* создать приложение в Azure Active Directory и субъект-службу.

### <a name="create-a-self-signed-certificate"></a>Создание самозаверяющего сертификата

Прежде чем выполнять дальнейшие действия, описанные в этом разделе, убедитесь, что на вашем компьютере установлен [пакет SDK Windows](https://dev.windows.com/en-us/downloads). Вам также необходимо создать каталог, например **C:\mycertdir**, в котором будет создан сертификат.

1. В окне PowerShell перейдите в расположение, где установлен пакет SDK Windows (обычно это `C:\Program Files (x86)\Windows Kits\10\bin\x86`), и с помощью служебной программы [MakeCert][makecert] создайте самозаверяющий сертификат и закрытый ключ. Используйте такие команды:

    ```azurepowershell
    $certificateFileDir = "<my certificate directory>"
    cd $certificateFileDir

    makecert -sv mykey.pvk -n "cn=HDI-ADL-SP" CertFile.cer -r -len 2048
    ```

    Вам будет предложено ввести пароль для закрытого ключа. После успешного выполнения команды в указанном каталоге сертификатов должны появиться файлы **CertFile.cer** и **mykey.pvk**.
2. С помощью служебной программы [Pvk2Pfx][pvk2pfx] преобразуйте созданные программой MakeCert PVK- и CER-файлы в PFX-файл. Выполните следующую команду.

    ```azurepowershell
    pvk2pfx -pvk mykey.pvk -spc CertFile.cer -pfx CertFile.pfx -po <password>
    ```

    При появлении соответствующего запроса введите указанный ранее пароль для закрытого ключа. Значение, указываемое для параметра **-po** — это пароль, связанный с PFX-файлом. После успешного выполнения команды вы должны увидеть в указанном каталоге сертификата файл CertFile.pfx.

### <a name="create-an-azure-active-directory-and-a-service-principal"></a>Создание приложения в Azure Active Directory и субъекта-службы

Следуя описаниям, приведенным в этом разделе, вы создадите субъект-службу для приложения Azure Active Directory, назначите роль субъекту-службе и пройдете проверку подлинности в качестве субъекта-службы, используя сертификат. Чтобы создать приложение в Azure Active Directory, выполните следующие команды.

1. В окне консоли PowerShell вставьте из буфера обмена следующие командлеты. Укажите для свойства **-DisplayName** уникальное значение. Значения свойств **-HomePage** b **-IdentiferUris** — это заполнители, которые не проверяются.

    ```azurepowershell
    $certificateFilePath = "$certificateFileDir\CertFile.pfx"

    $password = Read-Host -Prompt "Enter the password" # This is the password you specified for the .pfx file

    $certificatePFX = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($certificateFilePath, $password)

    $rawCertificateData = $certificatePFX.GetRawCertData()

    $credential = [System.Convert]::ToBase64String($rawCertificateData)

    $application = New-AzADApplication `
        -DisplayName "HDIADL" `
        -HomePage "https://contoso.com" `
        -IdentifierUris "https://mycontoso.com" `
        -CertValue $credential  `
        -StartDate $certificatePFX.NotBefore  `
        -EndDate $certificatePFX.NotAfter

    $applicationId = $application.ApplicationId
    ```

2. С помощью идентификатора приложения создайте субъект-службу.

    ```azurepowershell
    $servicePrincipal = New-AzADServicePrincipal -ApplicationId $applicationId

     $objectId = $servicePrincipal.Id
    ```

3. Предоставьте субъекту-службе доступ к файлу и папке Data Lake Storage 1-го поколения, к которым будете обращаться из кластера HDInsight. Приведенный ниже фрагмент предоставляет доступ к корню учетной записи хранения с Data Lake Storage 1-го поколения (куда был скопирован образец файла данных) и самим файлом.

    ```azurepowershell
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path / -AceType User -Id $objectId -Permissions All
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path /vehicle1_09142014.csv -AceType User -Id $objectId -Permissions All
    ```

## <a name="create-an-hdinsight-linux-cluster-with-data-lake-storage-gen1-as-additional-storage"></a>Создание кластера HDInsight на платформе Linux с Data Lake Storage 1-го поколения в качестве дополнительного хранилища

В этом разделе показано, как создать кластер HDInsight Hadoop на платформе Linux с Data Lake Storage 1-го поколения в качестве дополнительного хранилища. В этом выпуске кластер HDInsight и учетная запись хранения с Data Lake Storage 1-го поколения должны находиться в одном расположении.

1. Сначала получите идентификатор клиента подписки. Позже он вам понадобится.

    ```azurepowershell
    $tenantID = (Get-AzContext).Tenant.TenantId
    ```

2. В этом выпуске в кластере Hadoop Data Lake Storage 1-го поколения может использоваться только как дополнительное хранилище кластера. Хранилище по умолчанию по-прежнему будет хранилищем BLOB-объектов Azure (WASB). Поэтому мы сначала создадим учетную запись хранения и контейнеры хранилища, необходимые для кластера.

    ```azurepowershell
    # Create an Azure storage account
    $location = "East US 2"
    $storageAccountName = "<StorageAccountName>"   # Provide a Storage account name

    New-AzStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName -Location $location -Type Standard_GRS

    # Create an Azure Blob Storage container
    $containerName = "<ContainerName>"              # Provide a container name
    $storageAccountKey = (Get-AzStorageAccountKey -Name $storageAccountName -ResourceGroupName $resourceGroupName)[0].Value
    $destContext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
    New-AzStorageContainer -Name $containerName -Context $destContext
    ```

3. Создайте кластер HDInsight, выполнив следующие командлеты.

    ```azurepowershell
    # Set these variables
    $clusterName = $containerName                   # As a best practice, have the same name for the cluster and container
    $clusterNodes = <ClusterSizeInNodes>            # The number of nodes in the HDInsight cluster
    $httpCredentials = Get-Credential
    $sshCredentials = Get-Credential

    New-AzHDInsightCluster -ClusterName $clusterName -ResourceGroupName $resourceGroupName -HttpCredential $httpCredentials -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.windows.net" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainer $containerName  -ClusterSizeInNodes $clusterNodes -ClusterType Hadoop -Version "3.4" -OSType Linux -SshCredential $sshCredentials -ObjectID $objectId -AadTenantId $tenantID -CertificateFilePath $certificateFilePath -CertificatePassword $password
    ```

    В случае успешного выполнения командлета должен отобразиться список сведений о кластере.


## <a name="run-test-jobs-on-the-hdinsight-cluster-to-use-the-data-lake-storage-gen1"></a>Запуск тестовых заданий в кластере HDInsight для использования Data Lake Storage 1-го поколения
После настройки кластера HDInsight выполните в нем тестовые задания, чтобы проверить, доступно ли ему Data Lake Storage 1-го поколения. Для этого мы выполним пример задания Hive, которое создает таблицу, используя демонстрационные данные, отправленные ранее в учетную запись хранения с Data Lake Storage 1-го поколения.

В этом разделе вы подключитесь к созданному кластеру HDInsight под управлением Linux по протоколу SSH и выполните пример запроса Hive.

* Если вы используете клиент Windows для подключения к кластеру по протоколу SSH, ознакомьтесь со статьей [Использование SSH с HDInsight (Hadoop) в PuTTY на базе Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).
* Если вы используете клиент Linux для подключения к кластеру по SSH, ознакомьтесь со статьей [Использование SSH с HDInsight (Hadoop) на платформе Windows, Linux, Unix или OS X](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

1. После подключения запустите интерфейс командной строки Hive с помощью следующей команды:

    ```azurepowershell
    hive
    ```

2. Используя интерфейс командной строки, введите следующие инструкции, чтобы создать таблицу с именем **vehicles** с помощью примера данных в Data Lake Storage 1-го поколения:

    ```azurepowershell
    DROP TABLE vehicles;
    CREATE EXTERNAL TABLE vehicles (str string) LOCATION 'adl://<mydatalakestoragegen1>.azuredatalakestore.net:443/';
    SELECT * FROM vehicles LIMIT 10;
    ```

    Должен отобразиться результат, аналогичный приведенному ниже:

    ```output
    1,1,2014-09-14 00:00:03,46.81006,-92.08174,51,S,1
    1,2,2014-09-14 00:00:06,46.81006,-92.08174,13,NE,1
    1,3,2014-09-14 00:00:09,46.81006,-92.08174,48,NE,1
    1,4,2014-09-14 00:00:12,46.81006,-92.08174,30,W,1
    1,5,2014-09-14 00:00:15,46.81006,-92.08174,47,S,1
    1,6,2014-09-14 00:00:18,46.81006,-92.08174,9,S,1
    1,7,2014-09-14 00:00:21,46.81006,-92.08174,53,N,1
    1,8,2014-09-14 00:00:24,46.81006,-92.08174,63,SW,1
    1,9,2014-09-14 00:00:27,46.81006,-92.08174,4,NE,1
    1,10,2014-09-14 00:00:30,46.81006,-92.08174,31,N,1
    ```

## <a name="access-data-lake-storage-gen1-using-hdfs-commands"></a>Доступ к Data Lake Storage 1-го поколения с помощью команд HDFS
Настроив в кластере HDInsight параметры для работы с Data Lake Storage 1-го поколения, используйте для доступа к хранилищу команды оболочки HDFS.

В этом разделе вы подключитесь к созданному кластеру HDInsight под управлением Linux по протоколу SSH и выполните команды HDFS.

* Если вы используете клиент Windows для подключения к кластеру по протоколу SSH, ознакомьтесь со статьей [Использование SSH с HDInsight (Hadoop) в PuTTY на базе Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).
* Если вы используете клиент Linux для подключения к кластеру по SSH, ознакомьтесь со статьей [Использование SSH с HDInsight (Hadoop) на платформе Windows, Linux, Unix или OS X](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

После подключения используйте следующую команду файловой системы HDFS, чтобы вывести список файлов в учетной записи хранения с Data Lake Storage 1-го поколения.

```azurepowershell
hdfs dfs -ls adl://<storage account with Data Lake Storage Gen1 name>.azuredatalakestore.net:443/
```

Эта команда должна показать файл, который вы ранее отправили в Data Lake Storage 1-го поколения.

```output
15/09/17 21:41:15 INFO web.CaboWebHdfsFileSystem: Replacing original urlConnectionFactory with org.apache.hadoop.hdfs.web.URLConnectionFactory@21a728d6
Found 1 items
-rwxrwxrwx   0 NotSupportYet NotSupportYet     671388 2015-09-16 22:16 adl://mydatalakestoragegen1.azuredatalakestore.net:443/mynewfolder
```

С помощью команды `hdfs dfs -put` вы можете передать несколько файлов в Data Lake Storage 1-го поколения, а затем с помощью команды `hdfs dfs -ls` проверить, успешно ли они передались.

## <a name="see-also"></a>См. также:
* [Использование Data Lake Storage 1-го поколения с кластерами Azure HDInsight](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen1.md)
* [Создание кластера HDInsight, использующего Data Lake Storage 1-го поколения, с помощью портала](data-lake-store-hdinsight-hadoop-use-portal.md)

[makecert]: /windows-hardware/drivers/devtest/makecert
[pvk2pfx]: /windows-hardware/drivers/devtest/pvk2pfx