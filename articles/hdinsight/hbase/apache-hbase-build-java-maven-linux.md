---
title: Создание клиента Java HBase для Azure HDInsight с помощью Apache Maven
description: Сведения об использовании Apache Maven для создания приложения Java для Apache HBase и его последующем развертывании в HBase в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seodec18, devx-track-java
ms.date: 12/24/2019
ms.openlocfilehash: 13616cf99db39ec4dac1d13e3dcd2cefc5a44614
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98942966"
---
# <a name="build-java-applications-for-apache-hbase"></a>Создание приложений Java для Apache HBase

Узнайте, как создать приложение [Apache HBase](https://hbase.apache.org/) в среде Java. Затем вы будете использовать приложение с HBase в Azure HDInsight.

В этом руководстве для создания и сборки проекта используется [Apache Maven](https://maven.apache.org/). Maven — это инструмент для управления и повышения обозримости проектов программного обеспечения, позволяющий создавать ПО, документацию и отчеты для проектов Java.

## <a name="prerequisites"></a>Предварительные условия

* Кластер Apache HBase в HDInsight. См. статью [Начало работы с Apache HBase](./apache-hbase-tutorial-get-started-linux.md).

* [Java Developer Kit (JDK) версии 8](/azure/developer/java/fundamentals/java-jdk-long-term-support).

* Средство [Apache Maven](https://maven.apache.org/download.cgi), [установленное](https://maven.apache.org/install.html) согласно инструкций Apache.  Maven — система сборки проектов Java.

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

* При использовании PowerShell вам потребуется [модуль AZ](/powershell/azure/).

* Текстовый редактор. В этой статье используется Блокнот Microsoft Notepad.

## <a name="test-environment"></a>Тестовая среда

Среда, используемая для этой статьи, была компьютер под Windows 10.  Команды были выполнены в командной строке, а различные файлы были изменены с помощью блокнота. Измените соответствующим образом для своей среды.

В командной строке введите приведенные ниже команды, чтобы создать рабочую среду.

```cmd
IF NOT EXIST C:\HDI MKDIR C:\HDI
cd C:\HDI
```

## <a name="create-a-maven-project"></a>Создание проекта Maven

1. Введите следующую команду, чтобы создать проект Maven с именем **hbaseapp**:

    ```cmd
    mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

    cd hbaseapp
    mkdir conf
    ```

    Эта команда создает каталог с именем `hbaseapp` в текущем расположении, содержащий базовый проект Maven. Вторая команда изменяет рабочий каталог на `hbaseapp` . Третья команда создает новый каталог, `conf` который будет использоваться позже. Каталог `hbaseapp` содержит следующие элементы:

    * `pom.xml`: Объектная модель проекта ([POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)) содержит сведения и сведения о конфигурации, используемые для построения проекта.
    * `src\main\java\com\microsoft\examples` содержит код приложения;
    * `src\test\java\com\microsoft\examples` содержит тесты для приложения.

2. Удалите созданный код примера. Удалите созданные файлы тестов и приложений `AppTest.java` , а затем `App.java` введите приведенные ниже команды.

    ```cmd
    DEL src\main\java\com\microsoft\examples\App.java
    DEL src\test\java\com\microsoft\examples\AppTest.java
    ```

## <a name="update-the-project-object-model"></a>Обновление модели объекта проекта

Все справочные материалы по файлу pom.xml см. по адресу https://maven.apache.org/pom.html.  Откройте `pom.xml` , введя следующую команду:

```cmd
notepad pom.xml
```

### <a name="add-dependencies"></a>Добавление зависимостей

В `pom.xml` добавьте в раздел следующий текст `<dependencies>` :

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-shaded-client</artifactId>
    <version>1.1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-core</artifactId>
    <version>4.14.1-HBase-1.1</version>
</dependency>
```  

В этом разделе показано, что для проекта требуются компоненты **hbase-client** и **phoenix-core**. При компиляции эти зависимости скачиваются из репозитория Maven по умолчанию. Можно воспользоваться [поиском в центральном репозитории Maven](https://search.maven.org/artifact/org.apache.hbase/hbase-client/1.1.2/jar) , чтобы получить дополнительную информацию об этой зависимости.

> [!IMPORTANT]  
> Номер версии hbase-client должен соответствовать версии Apache HBase, которая поставляется с кластером HDInsight. Воспользуйтесь следующей таблицей, чтобы найти правильный номер версии.

| Версия кластера HDInsight | Используемая версия Apache HBase |
| --- | --- |
| 3,6 | 1.1.2 |
| 4.0 | 2.0.0 |

Дополнительные сведения о версиях и компонентах HDInsight см. в статье [Что представляют собой различные компоненты Apache Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md)

### <a name="build-configuration"></a>Конфигурация построения

Подключаемые модули Maven позволяют настроить этапы сборки проекта. Этот раздел используется для добавления подключаемых модулей, ресурсов и других параметров конфигурации сборки.

Добавьте в файл следующий код `pom.xml` , а затем сохраните и закройте файл. Эти строки должны находиться в файле внутри тегов `<project>...</project>` (например, между тегами `</dependencies>` и `</project>`).

```xml
<build>
    <sourceDirectory>src</sourceDirectory>
    <resources>
    <resource>
        <directory>${basedir}/conf</directory>
        <filtering>false</filtering>
        <includes>
        <include>hbase-site.xml</include>
        </includes>
    </resource>
    </resources>
    <plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
        </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.2.1</version>
        <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
            </transformer>
        </transformers>
        </configuration>
        <executions>
        <execution>
            <phase>package</phase>
            <goals>
            <goal>shade</goal>
            </goals>
        </execution>
        </executions>
    </plugin>
    </plugins>
</build>
```

В этом разделе настраивается ресурс (`conf/hbase-site.xml`), который содержит информацию о конфигурации для HBase.

> [!NOTE]  
> Также можно настроить значения конфигурации непосредственно из кода. Ознакомьтесь с комментариями к примеру `CreateTable`.

В этом разделе также будут настроены подключаемые модули [Apache Maven Compiler](https://maven.apache.org/plugins/maven-compiler-plugin/) и [Apache Maven Shade](https://maven.apache.org/plugins/maven-shade-plugin/). Подключаемый модуль компилятора используется для компиляции топологии. Подключаемый модуль shade используется для предотвращения дублирования лицензии в JAR-файле, собранном Maven. Этот подключаемый модуль используется для предотвращения ошибки с дублированием файлов лицензий, которая появляется во время выполнения на кластере HDInsight. Использование maven-shade-plugin с реализацией `ApacheLicenseResourceTransformer` позволяет избежать этой ошибки.

maven-shade-plugin также создает так называемый uber jar, который содержит все зависимости, требуемые для приложения.

### <a name="download-the-hbase-sitexml"></a>Скачивание файла hbase-site.xml

Для копирования конфигурации HBase из кластера HBase в каталог `conf` используйте следующую команду. Замените `CLUSTERNAME` именем кластера HDInsight, а затем введите команду:

```cmd
scp sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/etc/hbase/conf/hbase-site.xml ./conf/hbase-site.xml
```

## <a name="create-the-application"></a>Создание приложения

### <a name="implement-a-createtable-class"></a>Реализация класса CreateTable

Введите следующую команду, чтобы создать и открыть новый файл `CreateTable.java` . Выберите **Да** в командной строке, чтобы создать новый файл.

```cmd
notepad src\main\java\com\microsoft\examples\CreateTable.java
```

Затем скопируйте и вставьте приведенный ниже код Java в новый файл. Затем закройте файл.

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.util.Bytes;

public class CreateTable {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Example of setting zookeeper values for HDInsight
    // in code instead of an hbase-site.xml file
    //
    // config.set("hbase.zookeeper.quorum",
    //            "zookeepernode0,zookeepernode1,zookeepernode2");
    //config.set("hbase.zookeeper.property.clientPort", "2181");
    //config.set("hbase.cluster.distributed", "true");
    //
    //NOTE: Actual zookeeper host names can be found using Ambari:
    //curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts"

    //Linux-based HDInsight clusters use /hbase-unsecure as the znode parent
    config.set("zookeeper.znode.parent","/hbase-unsecure");

    // create an admin object using the config
    HBaseAdmin admin = new HBaseAdmin(config);

    // create the table...
    HTableDescriptor tableDescriptor = new HTableDescriptor(TableName.valueOf("people"));
    // ... with two column families
    tableDescriptor.addFamily(new HColumnDescriptor("name"));
    tableDescriptor.addFamily(new HColumnDescriptor("contactinfo"));
    admin.createTable(tableDescriptor);

    // define some people
    String[][] people = {
        { "1", "Marcel", "Haddad", "marcel@fabrikam.com"},
        { "2", "Franklin", "Holtz", "franklin@contoso.com" },
        { "3", "Dwayne", "McKee", "dwayne@fabrikam.com" },
        { "4", "Rae", "Schroeder", "rae@contoso.com" },
        { "5", "Rosalie", "burton", "rosalie@fabrikam.com"},
        { "6", "Gabriela", "Ingram", "gabriela@contoso.com"} };

    HTable table = new HTable(config, "people");

    // Add each person to the table
    //   Use the `name` column family for the name
    //   Use the `contactinfo` column family for the email
    for (int i = 0; i< people.length; i++) {
        Put person = new Put(Bytes.toBytes(people[i][0]));
        person.add(Bytes.toBytes("name"), Bytes.toBytes("first"), Bytes.toBytes(people[i][1]));
        person.add(Bytes.toBytes("name"), Bytes.toBytes("last"), Bytes.toBytes(people[i][2]));
        person.add(Bytes.toBytes("contactinfo"), Bytes.toBytes("email"), Bytes.toBytes(people[i][3]));
        table.put(person);
    }
    // flush commits and close the table
    table.flushCommits();
    table.close();
    }
}
```

Этот код представляет `CreateTable` собой класс, который создает таблицу с именем `people` и заполняет ее некоторыми предопределенными пользователями.

### <a name="implement-a-searchbyemail-class"></a>Реализация класса SearchByEmail

Введите следующую команду, чтобы создать и открыть новый файл `SearchByEmail.java` . Выберите **Да** в командной строке, чтобы создать новый файл.

```cmd
notepad src\main\java\com\microsoft\examples\SearchByEmail.java
```

Затем скопируйте и вставьте приведенный ниже код Java в новый файл. Затем закройте файл.

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.filter.RegexStringComparator;
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter;
import org.apache.hadoop.hbase.filter.CompareFilter.CompareOp;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.util.GenericOptionsParser;

public class SearchByEmail {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Use GenericOptionsParser to get only the parameters to the class
    // and not all the parameters passed (when using WebHCat for example)
    String[] otherArgs = new GenericOptionsParser(config, args).getRemainingArgs();
    if (otherArgs.length != 1) {
        System.out.println("usage: [regular expression]");
        System.exit(-1);
    }

    // Open the table
    HTable table = new HTable(config, "people");

    // Define the family and qualifiers to be used
    byte[] contactFamily = Bytes.toBytes("contactinfo");
    byte[] emailQualifier = Bytes.toBytes("email");
    byte[] nameFamily = Bytes.toBytes("name");
    byte[] firstNameQualifier = Bytes.toBytes("first");
    byte[] lastNameQualifier = Bytes.toBytes("last");

    // Create a regex filter
    RegexStringComparator emailFilter = new RegexStringComparator(otherArgs[0]);
    // Attach the regex filter to a filter
    //   for the email column
    SingleColumnValueFilter filter = new SingleColumnValueFilter(
        contactFamily,
        emailQualifier,
        CompareOp.EQUAL,
        emailFilter
    );

    // Create a scan and set the filter
    Scan scan = new Scan();
    scan.setFilter(filter);

    // Get the results
    ResultScanner results = table.getScanner(scan);
    // Iterate over results and print  values
    for (Result result : results ) {
        String id = new String(result.getRow());
        byte[] firstNameObj = result.getValue(nameFamily, firstNameQualifier);
        String firstName = new String(firstNameObj);
        byte[] lastNameObj = result.getValue(nameFamily, lastNameQualifier);
        String lastName = new String(lastNameObj);
        System.out.println(firstName + " " + lastName + " - ID: " + id);
        byte[] emailObj = result.getValue(contactFamily, emailQualifier);
        String email = new String(emailObj);
        System.out.println(firstName + " " + lastName + " - " + email + " - ID: " + id);
    }
    results.close();
    table.close();
    }
}
```

`SearchByEmail`Класс может использоваться для запроса строк по адресу электронной почты. При использовании класса можно задавать либо строку, либо регулярное выражение, так как используется фильтр регулярных выражений.

### <a name="implement-a-deletetable-class"></a>Реализация класса DeleteTable

Введите следующую команду, чтобы создать и открыть новый файл `DeleteTable.java` . Выберите **Да** в командной строке, чтобы создать новый файл.

```cmd
notepad src\main\java\com\microsoft\examples\DeleteTable.java
```

Затем скопируйте и вставьте приведенный ниже код Java в новый файл. Затем закройте файл.

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HBaseAdmin;

public class DeleteTable {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Create an admin object using the config
    HBaseAdmin admin = new HBaseAdmin(config);

    // Disable, and then delete the table
    admin.disableTable("people");
    admin.deleteTable("people");
    }
}
```

`DeleteTable`Класс очищает таблицы HBase, созданные в этом примере, отключая и удаляя таблицу, созданную `CreateTable` классом.

## <a name="build-and-package-the-application"></a>Сборка и создание пакета приложения

1. Выполните следующую команду из каталога `hbaseapp`, чтобы собрать JAR-файл, содержащий приложение:

    ```cmd
    mvn clean package
    ```

    Эта команда создает и упаковывает приложение в JAR-файл.

2. После выполнения команды каталог `hbaseapp/target` будет содержать файл с именем `hbaseapp-1.0-SNAPSHOT.jar`.

   > [!NOTE]  
   > Файл `hbaseapp-1.0-SNAPSHOT.jar` относится к типу uber jar. Он содержит все зависимости, необходимые для запуска приложения.

## <a name="upload-the-jar-and-run-jobs-ssh"></a>Передача JAR-файла и запуск заданий (SSH)

В следующих действиях используется команда `scp` для копирования JAR-файла в головной узел Apache HBase в кластере HDInsight. С помощью команды `ssh` выполняется подключение к кластеру; пример запускается непосредственно на головном узле.

1. Отправьте JAR-файл в кластер. Замените `CLUSTERNAME` именем кластера HDInsight, а затем введите следующую команду:

    ```cmd
    scp ./target/hbaseapp-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:hbaseapp-1.0-SNAPSHOT.jar
    ```

2. Подключитесь к кластеру HBase. Замените `CLUSTERNAME` именем кластера HDInsight, а затем введите следующую команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

3. Чтобы создать таблицу HBase с помощью приложения Java, используйте следующую команду в открытом SSH-подключении:

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.CreateTable
    ```

    Эта команда создает таблицу HBase с именем **people** и заполняет ее данными.

4. Для поиска адресов электронной почты, хранящихся в этой таблице, воспользуйтесь следующей командой:

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.SearchByEmail contoso.com
    ```

    Вы получите следующие результаты:

    ```console
    Franklin Holtz - ID: 2
    Franklin Holtz - franklin@contoso.com - ID: 2
    Rae Schroeder - ID: 4
    Rae Schroeder - rae@contoso.com - ID: 4
    Gabriela Ingram - ID: 6
    Gabriela Ingram - gabriela@contoso.com - ID: 6
    ```

5. Чтобы удалить таблицу, используйте следующую команду:

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.DeleteTable
    ```

## <a name="upload-the-jar-and-run-jobs-powershell"></a>Передача JAR-файла и запуск заданий (PowerShell)

В следующих шагах используется [модуль Azure PowerShell AZ](/powershell/azure/new-azureps-module-az) для передачи JAR-файла в хранилище по умолчанию для кластера Apache HBase. Затем командлеты HDInsight используются для удаленного запуска примеров.

1. После установки и настройки модуля AZ создайте файл с именем `hbase-runner.psm1` . В качестве содержимого файла добавьте следующий текст:

   ```powershell
    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.CreateTable"
    -clusterName "MyHDInsightCluster"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "contoso.com"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "^r" -showErr
    #>

    function Start-HBaseExample {
    [CmdletBinding(SupportsShouldProcess = $true)]
    param(
    #The class to run
    [Parameter(Mandatory = $true)]
    [String]$className,

    #The name of the HDInsight cluster
    [Parameter(Mandatory = $true)]
    [String]$clusterName,

    #Only used when using SearchByEmail
    [Parameter(Mandatory = $false)]
    [String]$emailRegex,

    #Use if you want to see stderr output
    [Parameter(Mandatory = $false)]
    [Switch]$showErr
    )

    Set-StrictMode -Version 3

    # Is the Azure module installed?
    FindAzure

    # Get the login for the HDInsight cluster
    $creds=Get-Credential -Message "Enter the login for the cluster" -UserName "admin"

    # The JAR
    $jarFile = "wasb:///example/jars/hbaseapp-1.0-SNAPSHOT.jar"

    # The job definition
    $jobDefinition = New-AzHDInsightMapReduceJobDefinition `
        -JarFile $jarFile `
        -ClassName $className `
        -Arguments $emailRegex

    # Get the job output
    $job = Start-AzHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds
    Write-Host "Wait for the job to complete ..." -ForegroundColor Green
    Wait-AzHDInsightJob `
        -ClusterName $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds
    if($showErr)
    {
    Write-Host "STDERR"
    Get-AzHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds `
                -DisplayOutputType StandardError
    }
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds
    }

    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    -Container "MyContainer"
    #>

    function Add-HDInsightFile {
        [CmdletBinding(SupportsShouldProcess = $true)]
        param(
            #The path to the local file.
            [Parameter(Mandatory = $true)]
            [String]$localPath,

            #The destination path and file name, relative to the root of the container.
            [Parameter(Mandatory = $true)]
            [String]$destinationPath,

            #The name of the HDInsight cluster
            [Parameter(Mandatory = $true)]
            [String]$clusterName,

            #If specified, overwrites existing files without prompting
            [Parameter(Mandatory = $false)]
            [Switch]$force
        )

        Set-StrictMode -Version 3

        # Is the Azure module installed?
        FindAzure

        # Get authentication for the cluster
        $creds=Get-Credential

        # Does the local path exist?
        if (-not (Test-Path $localPath))
        {
            throw "Source path '$localPath' does not exist."
        }

        # Get the primary storage container
        $storage = GetStorage -clusterName $clusterName

        # Upload file to storage, overwriting existing files if -force was used.
        Set-AzStorageBlobContent -File $localPath `
            -Blob $destinationPath `
            -force:$force `
            -Container $storage.container `
            -Context $storage.context
    }

    function FindAzure {
        # Is there an active Azure subscription?
        $sub = Get-AzSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Connect-AzAccount
        }
    }

    function GetStorage {
        param(
            [Parameter(Mandatory = $true)]
            [String]$clusterName
        )
        $hdi = Get-AzHDInsightCluster -ClusterName $clusterName
        # Does the cluster exist?
        if (!$hdi)
        {
            throw "HDInsight cluster '$clusterName' does not exist."
        }
        # Create a return object for context & container
        $return = @{}
        $storageAccounts = @{}

        # Get storage information
        $resourceGroup = $hdi.ResourceGroup
        $storageAccountName=$hdi.DefaultStorageAccount.split('.')[0]
        $container=$hdi.DefaultStorageContainer
        $storageAccountKey=(Get-AzStorageAccountKey `
            -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value
        # Get the resource group, in case we need that
        $return.resourceGroup = $resourceGroup
        # Get the storage context, as we can't depend
        # on using the default storage context
        $return.context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
        # Get the container, so we know where to
        # find/store blobs
        $return.container = $container
        # Return storage accounts to support finding all accounts for
        # a cluster
        $return.storageAccount = $storageAccountName
        $return.storageAccountKey = $storageAccountKey

        return $return
    }
    # Only export the verb-phrase things
    export-modulemember *-*
   ```

    Этот файл содержит два модуля:

   * **Add-HDInsightFile** — используется для загрузки файлов в кластер;
   * **Start-HBaseExample** — используется для запуска классов, созданных ранее.

2. Сохраните `hbase-runner.psm1` файл в `hbaseapp` каталоге.

3. Зарегистрируйте модули с помощью Azure PowerShell. Откройте новое окно Azure PowerShell и измените приведенную ниже команду, заменив `CLUSTERNAME` именем кластера. Затем введите следующие команды:

    ```powershell
    cd C:\HDI\hbaseapp
    $myCluster = "CLUSTERNAME"
    Import-Module .\hbase-runner.psm1
    ```

4. Воспользуйтесь следующей командой, чтобы отправить файл `hbaseapp-1.0-SNAPSHOT.jar` в ваш кластер.

    ```powershell
    Add-HDInsightFile -localPath target\hbaseapp-1.0-SNAPSHOT.jar -destinationPath example/jars/hbaseapp-1.0-SNAPSHOT.jar -clusterName $myCluster
    ```

    При появлении запроса введите имя для входа и пароль администратора кластера. Команда отправляет файл `hbaseapp-1.0-SNAPSHOT.jar` в расположение `example/jars` в главном хранилище кластера.

5. Чтобы создать таблицу с помощью `hbaseapp`, используйте следующую команду:

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.CreateTable -clusterName $myCluster
    ```

    При появлении запроса введите имя для входа и пароль администратора кластера.

    Эта команда создает таблицу с именем **people** в HBase в кластере HDInsight. Эта команда не отображает какие-либо выходные данные в окне консоли.

6. Для осуществления поиска записей таблицы используйте следующую команду:

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.SearchByEmail -clusterName $myCluster -emailRegex contoso.com
    ```

    При появлении запроса введите имя для входа и пароль администратора кластера.

    Будет использован класс `SearchByEmail` для поиска всех строк, у которых значение семейства столбцов `contactinformation` и столбца `email` содержит строку `contoso.com`. Вы получите следующие результаты:

    ```output
    Franklin Holtz - ID: 2
    Franklin Holtz - franklin@contoso.com - ID: 2
    Rae Schroeder - ID: 4
    Rae Schroeder - rae@contoso.com - ID: 4
    Gabriela Ingram - ID: 6
    Gabriela Ingram - gabriela@contoso.com - ID: 6
    ```

    Использование **fabrikam.com** для значения `-emailRegex` вернет список пользователей, у которых имеется строка **fabrikam.com** в поле электронного адреса. В качестве поискового запроса также можно использовать регулярные выражения. Например, **^r** возвращает адреса электронной почты, начинающиеся с буквы r.

7. Чтобы удалить таблицу, используйте следующую команду:

    ```PowerShell
    Start-HBaseExample -className com.microsoft.examples.DeleteTable -clusterName $myCluster
    ```

### <a name="no-results-or-unexpected-results-when-using-start-hbaseexample"></a>При использовании Start-HBaseExample результаты отсутствуют или не получено каких-либо неожиданных результатов.

Используйте параметр `-showErr` для просмотра стандартной ошибки (STDERR), выдаваемой при выполнении задания.

## <a name="next-steps"></a>Дальнейшие действия

[Узнайте, как использовать SQLLine с Apache HBase.](apache-hbase-query-with-phoenix.md)