---
title: PowerShell — кластер HDInsight с Data Lake Storage 1-го поколения — Azure
description: Используйте Azure PowerShell для создания и использования кластеров Azure HDInsight с Azure Data Lake Storage 1-го поколения.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 5e899f28cf5b3c11ae5f935d7bc273c566214225
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97606782"
---
# <a name="create-hdinsight-clusters-with-azure-data-lake-storage-gen1-as-default-storage-by-using-powershell"></a>Создание кластеров HDInsight, использующих Azure Data Lake Storage 1-го поколения в качестве хранилища по умолчанию, с помощью PowerShell

> [!div class="op_single_selector"]
> * [Использование портала Azure](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Использование PowerShell (для хранилища по умолчанию)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [Использование PowerShell (для дополнительного хранилища)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [Использование Resource Manager](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)

Узнайте, как с помощью Azure PowerShell настроить кластеры Azure HDInsight, использующие Azure Data Lake Storage 1-го поколения в качестве хранилища по молчанию. Инструкции по созданию кластера HDInsight, использующего Data Lake Storage 1-го поколения в качестве дополнительного хранилища, см. в статье [Создание кластера HDInsight с Data Lake Storage 1-го поколения (как дополнительное хранилище) с помощью Azure PowerShell](data-lake-store-hdinsight-hadoop-use-powershell.md).

Ниже приведены некоторые важные сведения об использовании HDInsight с Data Lake Storage 1-го поколения.

* Возможность создавать кластеры HDInsight, использующие Data Lake Storage 1-го поколения в качестве хранилища по умолчанию, поддерживается для HDInsight версий 3.5 и 3.6.

* Возможность создавать кластеры HDInsight, использующие Data Lake Storage 1-го поколения в качестве хранилища по умолчанию, *не поддерживается* для кластеров HDInsight уровня "Премиум".

Чтобы настроить HDInsight для работы с Data Lake Storage 1-го поколения с помощью PowerShell, следуйте инструкциям в следующих пяти разделах.

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Прежде чем приступить к изучению этого руководства, убедитесь, что выполнены следующие требования.

* **Подписка на Azure**. Перейдите на сайт [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1.0 или более поздняя версия**. Инструкции см. в статье [Приступая к работе с командлетами Azure PowerShell](/powershell/azure/).
* **Пакет средств разработки программного обеспечения для Windows (пакет SDK)**. Инструкции по установке пакета Windows SDK приведены на странице [Загружаемые файлы и инструменты для Windows 10](https://dev.windows.com/downloads). Пакет SDK используется для создания сертификата безопасности.
* **Субъект-служба Azure Active Directory**. В этом руководстве описывается создание субъекта-службы в Azure Active Directory (Azure AD). Чтобы создать субъект-службу, необходимо быть администратором Azure AD. Если вы являетесь администратором, то можете пропустить это предварительное требование и продолжить работу с руководством.

    >[!NOTE]
    >Создать субъект-службу может только администратор Azure AD. Администратор Azure AD должен сначала создать субъект-службу, после чего вы сможете создать кластер HDInsight, использующий Data Lake Storage 1-го поколения. При создании субъекта-службы необходимо использовать сертификат, как описано в разделе [Создание субъекта-службы с использованием сертификата](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-certificate-from-certificate-authority).
    >

## <a name="create-an-azure-data-lake-storage-gen1-account"></a>Создание учетной записи Azure Data Lake Storage 1-го поколения

Чтобы создать учетную запись Data Lake Storage 1-го поколения, выполните следующее.

1. На своем компьютере откройте окно PowerShell и введите приведенные ниже фрагменты кода. Когда будет предложено выполнить вход, войдите как один администраторов или владельцев из подписки. 

    ```azurepowershell
    # Sign in to your Azure account
    Connect-AzAccount

    # List all the subscriptions associated to your account
    Get-AzSubscription

    # Select a subscription
    Set-AzContext -SubscriptionId <subscription ID>

    # Register for Data Lake Storage Gen1
    Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
    ```

    > [!NOTE]
    > Если зарегистрировать поставщик ресурсов Data Lake Storage 1-го поколения и получить сообщение об ошибке, аналогичное `Register-AzResourceProvider : InvalidResourceNamespace: The resource namespace 'Microsoft.DataLakeStore' is invalid` , возможно, ваша подписка не утверждена для Data Lake Storage 1-го поколения. Чтобы использовать подписку Azure для Data Lake Storage 1-го поколения, следуйте инструкциям в разделе [Начало работы с Azure Data Lake Storage 1-го поколения с помощью портала Azure](data-lake-store-get-started-portal.md).
    >

2. Учетная запись Data Lake Storage 1-го поколения связывается с группой ресурсов Azure. Для начала создайте группу ресурсов.

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

3. Создайте учетную запись Data Lake Storage 1-го поколения. Имя этой учетной записи должно содержать только строчные буквы и цифры.

    ```azurepowershell
    $dataLakeStorageGen1Name = "<your new Data Lake Storage Gen1 name>"
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

4. При использовании Data Lake Storage 1-го поколения в качестве хранилища по умолчанию необходимо указать путь к корневой папке, в которую будут скопированы связанные с кластером файлы в процессе создания кластера. Используйте приведенные ниже командлеты, чтобы создать корневую папку (в следующем фрагменте это **/clusters/hdiadlcluster**).

    ```azurepowershell
    $myrootdir = "/"
    New-AzDataLakeStoreItem -Folder -AccountName $dataLakeStorageGen1Name -Path $myrootdir/clusters/hdiadlcluster
    ````

## <a name="set-up-authentication-for-role-based-access-to-data-lake-storage-gen1"></a>Настройка проверки подлинности для доступа к Data Lake Storage 1-го поколения на основе ролей
Каждая подписка Azure связана с сущностью Azure AD. Пользователи и службы, обращающиеся к ресурсам подписки через портал Azure или API Azure Resource Manager, сначала должны пройти аутентификацию в Azure AD. Доступ к подпискам и службам Azure предоставляется путем назначения соответствующей роли для ресурса Azure. В случае служб субъект-служба идентифицирует службу в Azure AD.

В этом разделе показано, как предоставить службе приложения, например HDInsight, доступ к ресурсу Azure (учетной записи Data Lake Storage 1-го поколения, созданной ранее). Для этого следует создать субъект-службу для приложения и назначить ему роли с помощью PowerShell.

Чтобы настроить аутентификацию Active Directory для Data Lake Storage 1-го поколения, необходимо выполнить инструкции, приведенные в двух следующих разделах.

### <a name="create-a-self-signed-certificate"></a>Создание самозаверяющего сертификата
Прежде чем выполнять дальнейшие действия, описанные в этом разделе, убедитесь, что на вашем компьютере установлен [пакет SDK Windows](https://dev.windows.com/en-us/downloads). Вам также необходимо создать каталог, например *C:\mycertdir*, для создания сертификата.

1. В окне PowerShell перейдите в расположение, где установлен пакет Windows SDK (обычно это *C:\Program Files (x86)\Windows Kits\10\bin\x86*), и с помощью служебной программы [MakeCert][makecert] создайте самозаверяющий сертификат и закрытый ключ. Используйте следующие команды:

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

    При появлении соответствующего запроса введите указанный ранее пароль для закрытого ключа. Значение, указываемое для параметра **-po**, — это пароль, связанный с PFX-файлом. После успешного выполнения команды в указанном каталоге сертификатов также должен отобразиться файл **CertFile. pfx** .

### <a name="create-an-azure-ad-and-a-service-principal"></a>Создание приложения Azure AD и субъекта-службы
Следуя инструкциям в этом разделе, вы создадите субъект-службу для приложения Azure Active Directory, назначите роль субъекту-службе и пройдете аутентификацию в качестве субъекта-службы, используя сертификат. Чтобы создать приложение в Azure AD, выполните следующие команды.

1. В окне консоли PowerShell вставьте из буфера обмена следующие командлеты. Обязательно укажите для свойства **-DisplayName** уникальное значение. Значения свойств **-HomePage** b **-IdentiferUris** — это заполнители, которые не проверяются.

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

3. Предоставьте субъекту-службе доступ к корневой папке Data Lake Storage 1-го поколения и всем папкам в ранее указанном корневом пути. Используйте следующие командлеты.

    ```azurepowershell
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path / -AceType User -Id $objectId -Permissions All
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path /clusters -AceType User -Id $objectId -Permissions All
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path /clusters/hdiadlcluster -AceType User -Id $objectId -Permissions All
    ```

## <a name="create-an-hdinsight-linux-cluster-with-data-lake-storage-gen1-as-the-default-storage"></a>Создание кластера HDInsight на платформе Linux, использующего Data Lake Storage 1-го поколения в качестве хранилища по умолчанию

Следуя инструкциям в этом разделе, вы создадите кластер HDInsight Hadoop на платформе Linux, использующий Data Lake Storage 1-го поколения в качестве хранилища по умолчанию. При использовании этого выпуска кластер HDInsight и Data Lake Storage 1-го поколения должны находиться в одном расположении.

1. Получите идентификатор клиента для подписки и сохраните его для последующего использования.

    ```azurepowershell
    $tenantID = (Get-AzContext).Tenant.TenantId
    ```

2. Создайте кластер HDInsight с помощью приведенных ниже командлетов.

    ```azurepowershell
    # Set these variables

    $location = "East US 2"
    $storageAccountName = $dataLakeStorageGen1Name    # Data Lake Storage Gen1 account name
        $storageRootPath = "<Storage root path you specified earlier>"     # e.g. /clusters/hdiadlcluster
        $clusterName = "<unique cluster name>"
    $clusterNodes = <ClusterSizeInNodes>            # The number of nodes in the HDInsight cluster
    $httpCredentials = Get-Credential
    $sshCredentials = Get-Credential

    New-AzHDInsightCluster `
           -ClusterType Hadoop `
           -OSType Linux `
           -ClusterSizeInNodes $clusterNodes `
           -ResourceGroupName $resourceGroupName `
           -ClusterName $clusterName `
           -HttpCredential $httpCredentials `
           -Location $location `
           -DefaultStorageAccountType AzureDataLakeStore `
           -DefaultStorageAccountName "$storageAccountName.azuredatalakestore.net" `
           -DefaultStorageRootPath $storageRootPath `
           -Version "3.6" `
           -SshCredential $sshCredentials `
           -AadTenantId $tenantId `
           -ObjectId $objectId `
           -CertificateFilePath $certificateFilePath `
           -CertificatePassword $password
    ```

    После успешного выполнения командлетов должны отобразиться сведения о кластере.

## <a name="run-test-jobs-on-the-hdinsight-cluster-to-use-data-lake-storage-gen1"></a>Выполнение тестовых заданий в кластере HDInsight для использования Data Lake Storage 1-го поколения
После настройки кластера HDInsight выполните в нем тестовые задания, чтобы проверить, может ли он использовать Data Lake Storage 1-го поколения. Для этого выполните пример задания Hive, чтобы создать таблицу, которая использует демонстрационные данные, которые уже доступны в Data Lake Storage 1-го поколения по адресу *\<cluster root> /example/Data/Sample.log*.

Следуя инструкциям в этом разделе, вы подключитесь по протоколу Secure Shell (SSH) к созданному кластеру HDInsight на платформе Linux и выполните пример запроса Hive.

* Если вы используете клиент Windows для подключения к кластеру по протоколу SSH, ознакомьтесь со статьей [Использование SSH с HDInsight (Hadoop) в PuTTY на базе Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md).
* Если вы используете клиент Linux для подключения к кластеру по SSH, ознакомьтесь со статьей [Использование SSH с HDInsight (Hadoop) на платформе Windows, Linux, Unix или OS X](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

1. Установив подключение, запустите интерфейс командной строки Hive, выполнив в командной строке следующую команду.

    ```powershell
    hive
    ```

2. Используя интерфейс командной строки, введите приведенные ниже инструкции, чтобы создать таблицу **vehicles** с помощью демонстрационных данных в Data Lake Storage 1-го поколения.

    ```azurepowershell
    DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'adl:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;
    ```

Результаты запроса появятся в консоли SSH.

>[!NOTE]
>Путь к демонстрационным данных в приведенной выше команде CREATE TABLE — `adl:///example/data/`, где `adl:///` является корнем кластера. Используя пример корня кластера, указанный в этом руководстве, получаем команду `adl://hdiadlstore.azuredatalakestore.net/clusters/hdiadlcluster`. Можно использовать более короткий вариант пути или указать полный путь к корню кластера.
>

## <a name="access-data-lake-storage-gen1-by-using-hdfs-commands"></a>Доступ к Data Lake Storage 1-го поколения с помощью команд HDFS
Настроив кластер HDInsight для работы с Data Lake Storage 1-го поколения, можно использовать для доступа к хранилищу команды оболочки HDFS (распределенная файловая система Hadoop).

Следуя инструкциям в этом разделе, вы подключитесь по протоколу SSH к созданному кластеру HDInsight на платформе Linux и выполните команды HDFS.

* Если вы используете клиент Windows для подключения к кластеру по протоколу SSH, ознакомьтесь со статьей [Использование SSH с HDInsight (Hadoop) в PuTTY на базе Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md).
* Если вы используете клиент Linux для подключения к кластеру по SSH, ознакомьтесь со статьей [Использование SSH с HDInsight (Hadoop) на платформе Windows, Linux, Unix или OS X](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

Когда подключение будет установлено, выведите список файлов в Data Lake Storage 1-го поколения, используя приведенную ниже команду файловой системы HDFS.

```azurepowershell
hdfs dfs -ls adl:///
```

С помощью команды `hdfs dfs -put` вы можете передать несколько файлов в Data Lake Storage 1-го поколения, а затем с помощью команды `hdfs dfs -ls` проверить, успешно ли они передались.

## <a name="see-also"></a>См. также раздел
* [Использование Data Lake Storage 1-го поколения с кластерами Azure HDInsight](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen1.md)
* [Создание кластеров HDInsight, использующих Data Lake Storage 1-го поколения, с помощью портала Azure](data-lake-store-hdinsight-hadoop-use-portal.md)

[makecert]: /windows-hardware/drivers/devtest/makecert
[pvk2pfx]: https://msdn.microsoft.com/library/windows/desktop/ff550672(v=vs.85).aspx