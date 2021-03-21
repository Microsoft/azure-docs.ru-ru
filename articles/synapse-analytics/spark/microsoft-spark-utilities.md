---
title: Введение в служебные программы Microsoft Spark
description: Учебник. Мсспаркутилс в записных книжках Azure синапсе Analytics
author: ruixinxu
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: reference
ms.subservice: spark
ms.date: 09/10/2020
ms.author: ruxu
ms.reviewer: ''
zone_pivot_groups: programming-languages-spark-all-minus-sql
ms.openlocfilehash: 58672bd68d9a2ea85f58b3761f3b89098b9f5afc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100368665"
---
# <a name="introduction-to-microsoft-spark-utilities"></a>Введение в служебные программы Microsoft Spark

Microsoft Spark Utilities (Мсспаркутилс) — это встроенный пакет, который упрощает выполнение стандартных задач. Мсспаркутилс можно использовать для работы с файловыми системами, для получения переменных среды и для работы с секретами. Мсспаркутилс доступны в `PySpark (Python)` `Scala` `.NET Spark (C#)` записных книжках,, и в конвейерах синапсе.

## <a name="pre-requisites"></a>Предварительные требования

### <a name="configure-access-to-azure-data-lake-storage-gen2"></a>Настройка доступа к Azure Data Lake Storage 2-го поколения 

Записные книжки синапсе используют сквозной доступ Azure Active Directory (Azure AD) для доступа к учетным записям ADLS 2-го поколения. Для доступа к учетной записи ADLS 2-го поколения (или папке) необходимо быть **участником данных хранилища BLOB-объектов** . 

Для доступа к учетным записям хранения в конвейерах синапсе используется удостоверение рабочей области (MSI). Чтобы использовать Мсспаркутилс в действиях конвейера, удостоверение рабочей области должно быть **участником данных хранилища BLOB-объектов** для доступа к учетной записи ADLS 2-го поколения (или папке).

Выполните следующие действия, чтобы убедиться, что у MSI Azure AD и рабочей области есть доступ к учетной записи ADLS 2-го поколения.
1. Откройте [портал Azure](https://portal.azure.com/) и учетную запись хранения, к которой требуется получить доступ. Можно перейти к конкретному контейнеру, к которому требуется получить доступ.
2. На левой панели выберите **элемент управления доступом (IAM)** .
3. Назначьте **учетной записи Azure AD** и **удостоверению рабочей области** (то же, что и имя рабочей области) роль **участника данных BLOB-объекта хранилища** в учетной записи хранения, если она еще не назначена. 
4. Щелкните **Сохранить**.

Вы можете получить доступ к данным в ADLS 2-го поколения с помощью синапсе Spark по следующему URL-адресу:

<code>abfss://<container_name>@<storage_account_name>.dfs.core.windows.net/<path></code>

### <a name="configure-access-to-azure-blob-storage"></a>Настройка доступа к хранилищу BLOB-объектов Azure  

Синапсе использовать **подписанный URL-адрес (SAS)** для доступа к хранилищу больших двоичных объектов Azure. Чтобы избежать предоставления ключей SAS в коде, мы рекомендуем создать новую связанную службу в рабочей области синапсе в учетной записи хранилища BLOB-объектов Azure, к которой вы хотите получить доступ.

Чтобы добавить новую связанную службу для учетной записи хранилища BLOB-объектов Azure, выполните следующие действия.

1. Откройте [Azure синапсе Studio](https://web.azuresynapse.net/).
2. Выберите **Управление** на левой панели и выберите **связанные службы** в списке **внешние подключения**.
3. Найдите **хранилище BLOB-объектов Azure** на **новой панели связанная служба** справа.
4. Выберите **Continue** (Продолжить).
5. Выберите учетную запись хранилища BLOB-объектов Azure для доступа и настройте имя связанной службы. Рекомендуется использовать **ключ учетной записи** для **метода проверки подлинности**.
6. Выберите **проверить подключение** , чтобы проверить правильность параметров.
7. Сначала выберите **создать** , а затем щелкните **опубликовать все** , чтобы сохранить изменения. 

Вы можете получить доступ к данным в хранилище больших двоичных объектов Azure с помощью синапсе Spark по следующему URL-адресу:

<code>wasb[s]://<container_name>@<storage_account_name>.blob.core.windows.net/<path></code>

Ниже приведен пример кода:

:::zone pivot = "programming-language-python"

```python
from pyspark.sql import SparkSession

# Azure storage access info
blob_account_name = 'Your account name' # replace with your blob name
blob_container_name = 'Your container name' # replace with your container name
blob_relative_path = 'Your path' # replace with your relative folder path
linked_service_name = 'Your linked service name' # replace with your linked service name

blob_sas_token = mssparkutils.credentials.getConnectionStringOrCreds(linked_service_name)

# Allow SPARK to access from Blob remotely

wasb_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)

spark.conf.set('fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name), blob_sas_token)
print('Remote blob path: ' + wasb_path)
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val blob_account_name = "" // replace with your blob name
val blob_container_name = "" //replace with your container name
val blob_relative_path = "/" //replace with your relative folder path
val linked_service_name = "" //replace with your linked service name


val blob_sas_token = mssparkutils.credentials.getConnectionStringOrCreds(linked_service_name)

val wasbs_path = f"wasbs://$blob_container_name@$blob_account_name.blob.core.windows.net/$blob_relative_path"
spark.conf.set(f"fs.azure.sas.$blob_container_name.$blob_account_name.blob.core.windows.net",blob_sas_token)

```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var blob_account_name = "";  // replace with your blob name
var blob_container_name = "";     // replace with your container name
var blob_relative_path = "";  // replace with your relative folder path
var linked_service_name = "";    // replace with your linked service name
var blob_sas_token = Credentials.GetConnectionStringOrCreds(linked_service_name);

spark.SparkContext.GetConf().Set($"fs.azure.sas.{blob_container_name}.{blob_account_name}.blob.core.windows.net", blob_sas_token);

var wasbs_path = $"wasbs://{blob_container_name}@{blob_account_name}.blob.core.windows.net/{blob_relative_path}";

Console.WriteLine(wasbs_path);

```

::: zone-end 
 
###  <a name="configure-access-to-azure-key-vault"></a>Настройка доступа к Azure Key Vault

Вы можете добавить Azure Key Vault в качестве связанной службы для управления учетными данными в синапсе. Чтобы добавить Azure Key Vault в качестве связанной службы синапсе, выполните следующие действия:
1. Откройте [Azure синапсе Studio](https://web.azuresynapse.net/).
2. Выберите **Управление** на левой панели и выберите **связанные службы** в списке **внешние подключения**.
3. **Azure Key Vault** поиска на **новой связанной сервисной** панели справа.
4. Выберите учетную запись Azure Key Vault для доступа и настройки имени связанной службы.
5. Выберите **проверить подключение** , чтобы проверить правильность параметров.
6. Сначала выберите **создать** , а затем щелкните **опубликовать все** , чтобы сохранить изменения. 

Записные книжки синапсе используют сквозной доступ к Azure Active Directory (Azure AD) для доступа к Azure Key Vault. Для доступа к Azure Key Vault конвейеры синапсе используют удостоверение рабочей области (MSI). Чтобы код работал как в записной книжке, так и в конвейере синапсе, мы рекомендуем предоставить секретное разрешение на доступ как для учетной записи Azure AD, так и для удостоверения рабочей области.

Чтобы предоставить секретный доступ к удостоверению рабочей области, выполните следующие действия:
1. Откройте [портал Azure](https://portal.azure.com/) и Azure Key Vault, к которым требуется получить доступ. 
2. Выберите **политики доступа** на панели слева.
3. Выберите **Добавить политику доступа**: 
    - Выберите **ключ, секрет, & Управление сертификатами** в качестве шаблона конфигурации.
    - Выберите **учетную запись Azure AD** и **удостоверение рабочей области** (то же, что и имя рабочей области) в поле Выбор участника или убедитесь, что оно уже назначено. 
4. Выберите **выбрать** и **Добавить**.
5. Нажмите кнопку **сохранить** , чтобы зафиксировать изменения.  

## <a name="file-system-utilities"></a>Служебные программы файловой системы

`mssparkutils.fs` предоставляет служебные программы для работы с различными файловыми системами, в том числе Azure Data Lake Storage 2-го поколения (ADLS 2-го поколения) и хранилищем BLOB-объектов Azure. Убедитесь, что вы правильно настроили доступ к [Azure Data Lake Storage 2-го поколения](#configure-access-to-azure-data-lake-storage-gen2) и [хранилищу BLOB-объектов Azure](#configure-access-to-azure-blob-storage) .

Чтобы получить общие сведения о доступных методах, выполните следующие команды:

:::zone pivot = "programming-language-python"

```python
from notebookutils import mssparkutils
mssparkutils.fs.help()
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.fs.help()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
using Microsoft.Spark.Extensions.Azure.Synapse.Analytics.Notebook.MSSparkUtils;
FS.Help()
```

::: zone-end

Результат будет иметь такой вид:
```
mssparkutils.fs provides utilities for working with various FileSystems.

Below is overview about the available methods:

cp(from: String, to: String, recurse: Boolean = false): Boolean -> Copies a file or directory, possibly across FileSystems
mv(from: String, to: String, recurse: Boolean = false): Boolean -> Moves a file or directory, possibly across FileSystems
ls(dir: String): Array -> Lists the contents of a directory
mkdirs(dir: String): Boolean -> Creates the given directory if it does not exist, also creating any necessary parent directories
put(file: String, contents: String, overwrite: Boolean = false): Boolean -> Writes the given String out to a file, encoded in UTF-8
head(file: String, maxBytes: int = 1024 * 100): String -> Returns up to the first 'maxBytes' bytes of the given file as a String encoded in UTF-8
append(file: String, content: String, createFileIfNotExists: Boolean): Boolean -> Append the content to a file
rm(dir: String, recurse: Boolean = false): Boolean -> Removes a file or directory

Use mssparkutils.fs.help("methodName") for more info about a method.

```

### <a name="list-files"></a>Перечень файлов
Перечисление содержимого каталога.


:::zone pivot = "programming-language-python"

```python
mssparkutils.fs.ls('Your directory path')
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.fs.ls("Your directory path")
```
::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
FS.Ls("Your directory path")
```

::: zone-end


### <a name="view-file-properties"></a>Просмотр свойств файла.
Возвращает свойства файла, включая имя файла, путь к файлу, размер файла и принадлежность к каталогу и файлу.

:::zone pivot = "programming-language-python"

```python
files = mssparkutils.fs.ls('Your directory path')
for file in files:
    print(file.name, file.isDir, file.isFile, file.path, file.size)
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val files = mssparkutils.fs.ls("/")
files.foreach{
    file => println(file.name,file.isDir,file.isFile,file.size)
}
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var Files = FS.Ls("/");
foreach(var File in Files) {
    Console.WriteLine(File.Name+" "+File.IsDir+" "+File.IsFile+" "+File.Size);
}
```

::: zone-end

### <a name="create-new-directory"></a>Создать новый каталог

Создает указанный каталог, если он не существует и все необходимые родительские каталоги.

:::zone pivot = "programming-language-python"

```python
mssparkutils.fs.mkdirs('new directory name')
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.fs.mkdirs("new directory name")
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
FS.Mkdirs("new directory name")
```

::: zone-end

### <a name="copy-file"></a>Копировать файл

Копирует файл или каталог. Поддерживает копирование в файловых системах.

:::zone pivot = "programming-language-python"

```python
mssparkutils.fs.cp('source file or directory', 'destination file or directory', True)# Set the third parameter as True to copy all files and directories recursively
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.fs.cp("source file or directory", "destination file or directory", true) // Set the third parameter as True to copy all files and directories recursively
```
::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
FS.Cp("source file or directory", "destination file or directory", true) // Set the third parameter as True to copy all files and directories recursively
```

::: zone-end

### <a name="preview-file-content"></a>Просмотр содержимого файла

Возвращает до первого байта (Максбитес) заданного файла в виде строки в кодировке UTF-8.

:::zone pivot = "programming-language-python"

```python
mssparkutils.fs.head('file path', maxBytes to read)
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.fs.head("file path", maxBytes to read)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
FS.Head("file path", maxBytes to read)
```

::: zone-end

### <a name="move-file"></a>Переместить файл

Перемещает файл или каталог. Поддерживает перемещение по файловым системам.

:::zone pivot = "programming-language-python"

```python
mssparkutils.fs.mv('source file or directory', 'destination directory', True) # Set the last parameter as True to firstly create the parent directory if it does not exist
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.fs.mv("source file or directory", "destination directory", true) // Set the last parameter as True to firstly create the parent directory if it does not exist
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
FS.Mv("source file or directory", "destination directory", true)
```

::: zone-end

### <a name="write-file"></a>Запись файла

Записывает заданную строку в файл в кодировке UTF-8.

:::zone pivot = "programming-language-python"

```python
mssparkutils.fs.put("file path", "content to write", True) # Set the last parameter as True to overwrite the file if it existed already
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.fs.put("file path", "content to write", true) // Set the last parameter as True to overwrite the file if it existed already
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
FS.Put("file path", "content to write", true) // Set the last parameter as True to overwrite the file if it existed already
```

::: zone-end

### <a name="append-content-to-a-file"></a>Добавление содержимого в файл

Добавляет заданную строку в файл в кодировке UTF-8.

:::zone pivot = "programming-language-python"

```python
mssparkutils.fs.append('file path','content to append',True) # Set the last parameter as True to create the file if it does not exist
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.fs.append("file path","content to append",true) // Set the last parameter as True to create the file if it does not exist
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
FS.Append("file path","content to append",true) // Set the last parameter as True to create the file if it does not exist
```

::: zone-end

### <a name="delete-file-or-directory"></a>Удалить файл или каталог

Удаляет файл или каталог.

:::zone pivot = "programming-language-python"

```python
mssparkutils.fs.rm('file path', True) # Set the last parameter as True to remove all files and directories recursively 
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.fs.rm("file path", true) // Set the last parameter as True to remove all files and directories recursively 
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
FS.Rm("file path", true) // Set the last parameter as True to remove all files and directories recursively 
```

::: zone-end


## <a name="credentials-utilities"></a>Служебные программы для учетных данных

Вы можете использовать служебные программы Мсспаркутилс Credential для получения маркеров доступа к связанным службам и управления секретами в Azure Key Vault. 

Чтобы получить общие сведения о доступных методах, выполните следующую команду:

:::zone pivot = "programming-language-python"

```python
mssparkutils.credentials.help()
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.credentials.help()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Credentials.Help()
```

::: zone-end

Получить результат:

```
getToken(audience, name): returns AAD token for a given audience, name (optional)
isValidToken(token): returns true if token hasn't expired
getConnectionStringOrCreds(linkedService): returns connection string or credentials for linked service
getSecret(akvName, secret, linkedService): returns AKV secret for a given AKV linked service, akvName, secret key
getSecret(akvName, secret): returns AKV secret for a given akvName, secret key
putSecret(akvName, secretName, secretValue, linkedService): puts AKV secret for a given akvName, secretName
putSecret(akvName, secretName, secretValue): puts AKV secret for a given akvName, secretName
```

### <a name="get-token"></a>Получение токена

Возвращает маркер Azure AD для данной аудитории, имя (необязательно). В следующей таблице перечислены все доступные типы аудиторий. 

|Тип аудитории|Ключ аудитории|
|--|--|
|Тип разрешения аудитории|Компиляцию|
|Ресурс аудитории хранилища|Объема|
|Ресурс аудитории хранилища данных|DW|
|Ресурс аудитории Data Lake|"Азуреманажемент"|
|Ресурс аудитории хранилища|Data Lake Store|
|Ресурс "аудитория ОССДБ Azure"|"Азуреоссдб"|
|Ресурс Azure синапсе|"Синапсе"|
|Ресурс фабрики данных Azure|ФАЙЛАХ|

:::zone pivot = "programming-language-python"

```python
mssparkutils.credentials.getToken('audience Key')
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.credentials.getToken("audience Key")
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Credentials.GetToken("audience Key")
```

::: zone-end


### <a name="validate-token"></a>Проверка токена

Возвращает значение true, если срок действия токена не истек.

:::zone pivot = "programming-language-python"

```python
mssparkutils.credentials.isValidToken('your token')
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.credentials.isValidToken("your token")
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Credentials.IsValidToken("your token")
```

::: zone-end


### <a name="get-connection-string-or-credentials-for-linked-service"></a>Получение строки подключения или учетных данных для связанной службы

Возвращает строку подключения или учетные данные для связанной службы. 

:::zone pivot = "programming-language-python"

```python
mssparkutils.credentials.getConnectionStringOrCreds('linked service name')
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.credentials.getConnectionStringOrCreds("linked service name")
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Credentials.GetConnectionStringOrCreds("linked service name")
```

::: zone-end


### <a name="get-secret-using-workspace-identity"></a>Получение секрета с помощью удостоверения рабочей области

Возвращает Azure Key Vault секрет для указанного Azure Key Vault имени, имени секрета и имени связанной службы с помощью удостоверения рабочей области. Убедитесь, что вы настроили доступ к [Azure Key Vault](#configure-access-to-azure-key-vault) соответствующим образом.

:::zone pivot = "programming-language-python"

```python
mssparkutils.credentials.getSecret('azure key vault name','secret name','linked service name')
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.credentials.getSecret("azure key vault name","secret name","linked service name")
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Credentials.GetSecret("azure key vault name","secret name","linked service name")
```

::: zone-end


### <a name="get-secret-using-user-credentials"></a>Получение секрета с помощью учетных данных пользователя

Возвращает Azure Key Vault секрет для указанного Azure Key Vault имени, имени секрета и имени связанной службы с помощью учетных данных пользователя. 

:::zone pivot = "programming-language-python"

```python
mssparkutils.credentials.getSecret('azure key vault name','secret name')
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.credentials.getSecret("azure key vault name","secret name")
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Credentials.GetSecret("azure key vault name","secret name")
```

::: zone-end

<!-- ### Put secret using workspace identity

Puts Azure Key Vault secret for a given Azure Key Vault name, secret name, and linked service name using workspace identity. Make sure you configure the access to [Azure Key Vault](#configure-access-to-azure-key-vault) appropriately. -->

:::zone pivot = "programming-language-python"

### <a name="put-secret-using-workspace-identity"></a>Размещение секрета с помощью удостоверения рабочей области

Помещает Azure Key Vault секрет для указанного Azure Key Vault имени, имени секрета и имени связанной службы с помощью удостоверения рабочей области. Убедитесь, что вы настроили доступ к [Azure Key Vault](#configure-access-to-azure-key-vault) соответствующим образом.

```python
mssparkutils.credentials.putSecret('azure key vault name','secret name','secret value','linked service name')
```
::: zone-end

:::zone pivot = "programming-language-scala"

### <a name="put-secret-using-workspace-identity"></a>Размещение секрета с помощью удостоверения рабочей области

Помещает Azure Key Vault секрет для указанного Azure Key Vault имени, имени секрета и имени связанной службы с помощью удостоверения рабочей области. Убедитесь, что вы настроили доступ к [Azure Key Vault](#configure-access-to-azure-key-vault) соответствующим образом.

```scala
mssparkutils.credentials.putSecret("azure key vault name","secret name","secret value","linked service name")
```

::: zone-end

<!-- :::zone pivot = "programming-language-csharp"

```csharp

```

::: zone-end -->


<!-- ### Put secret using user credentials

Puts Azure Key Vault secret for a given Azure Key Vault name, secret name, and linked service name using user credentials.  -->

:::zone pivot = "programming-language-python"

### <a name="put-secret-using-user-credentials"></a>Размещение секрета с использованием учетных данных пользователя

Помещает Azure Key Vault секрет для указанного Azure Key Vault имени, имени секрета и имени связанной службы, используя учетные данные пользователя. 

```python
mssparkutils.credentials.putSecret('azure key vault name','secret name','secret value')
```
::: zone-end

:::zone pivot = "programming-language-scala"

### <a name="put-secret-using-user-credentials"></a>Размещение секрета с использованием учетных данных пользователя

Помещает Azure Key Vault секрет для указанного Azure Key Vault имени, имени секрета и имени связанной службы, используя учетные данные пользователя. 

```scala
mssparkutils.credentials.putSecret("azure key vault name","secret name","secret value")
```

::: zone-end

<!-- :::zone pivot = "programming-language-csharp"

```csharp

```

::: zone-end -->


## <a name="environment-utilities"></a>Служебные программы 

Выполните следующие команды, чтобы получить общие сведения о доступных методах.

:::zone pivot = "programming-language-python"

```python
mssparkutils.env.help()
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.env.help()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Env.Help()
```

::: zone-end

Получить результат:
```
GetUserName(): returns user name
GetUserId(): returns unique user id
GetJobId(): returns job id
GetWorkspaceName(): returns workspace name
GetPoolName(): returns Spark pool name
GetClusterId(): returns cluster id
```

### <a name="get-user-name"></a>Получить имя пользователя

Возвращает имя текущего пользователя.

:::zone pivot = "programming-language-python"

```python
mssparkutils.env.getUserName()
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.env.getUserName()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Env.GetUserName()
```

::: zone-end

### <a name="get-user-id"></a>Получить идентификатор пользователя

Возвращает идентификатор текущего пользователя.

:::zone pivot = "programming-language-python"

```python
mssparkutils.env.getUserId()
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.env.getUserId()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Env.GetUserId()
```

::: zone-end

### <a name="get-job-id"></a>Получение идентификатора задания

Возвращает идентификатор задания.

:::zone pivot = "programming-language-python"

```python
mssparkutils.env.getJobId()
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.env.getJobId()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Env.GetJobId()
```

::: zone-end

### <a name="get-workspace-name"></a>Получить имя рабочей области

Возвращает имя рабочей области.

:::zone pivot = "programming-language-python"

```python
mssparkutils.env.getWorkspaceName()
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.env.getWorkspaceName()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Env.GetWorkspaceName()
```

::: zone-end

### <a name="get-pool-name"></a>Получение имени пула

Возвращает имя пула Spark.

:::zone pivot = "programming-language-python"

```python
mssparkutils.env.getPoolName()
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.env.getPoolName()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Env.GetPoolName()
```

::: zone-end

### <a name="get-cluster-id"></a>Получить идентификатор кластера

Возвращает идентификатор текущего кластера.

:::zone pivot = "programming-language-python"

```python
mssparkutils.env.getClusterId()
```
::: zone-end

:::zone pivot = "programming-language-scala"

```scala
mssparkutils.env.getClusterId()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
Env.GetClusterId()
```

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

- [Ознакомьтесь с примерами записных книжек синапсе](https://github.com/Azure-Samples/Synapse/tree/master/Notebooks)
- [Краткое руководство. Создание пула Apache Spark в Azure Synapse Analytics с помощью веб-инструментов](../quickstart-apache-spark-notebook.md)
- [Что такое Apache Spark в Azure Synapse Analytics](apache-spark-overview.md)
- [Azure Synapse Analytics](../index.yml)
