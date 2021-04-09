---
title: Защита учетных данных для доступа с помощью связанных служб в Apache Spark для Azure Synapse Analytics
description: В этой статье приведены основные сведения о безопасной интеграции Apache Spark для Azure Synapse Analytics с другими службами с помощью связанных служб и библиотеки маркеров.
services: synapse-analytics
author: mlee3gsd
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: spark
ms.date: 11/19/2020
ms.author: martinle
ms.reviewer: nirav
zone_pivot_groups: programming-languages-spark-all-minus-sql
ms.openlocfilehash: a588b37b270917524453419619fdad6f88f92338
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101693583"
---
# <a name="secure-credentials-with-linked-services-using-the-tokenlibrary"></a>Защита учетных данных с помощью связанных служб с использованием библиотеки TokenLibrary

Доступ к данным из внешних источников является распространенным шаблоном. Если внешний источник данных разрешает анонимный доступ, то, вероятно, потребуется защитить подключение с помощью учетных данных, секрета или строки подключения.  

Для проверки подлинности между ресурсами Azure Synapse по умолчанию использует сквозную аутентификацию Azure Active Directory.  Если вам нужно подключиться к ресурсу с использованием других учетных данных, используйте TokenLibrary напрямую.  TokenLibrary упрощает процесс извлечения маркеров SAS, маркеров Azure AD, строк подключения и секретов, хранящихся в связанной службе или в Azure Key Vault.

Для получения секретов из Azure Key Vault мы рекомендуем создать службу, связанную с Azure Key Vault.  При этом нужно предоставить удостоверению управляемой службы рабочей области Synapse привилегии на получение секретов из такого хранилища Azure Key Vault.  Чтобы аутентифицироваться в Azure Key Vault, Synapse будет использовать удостоверение управляемой службы рабочей области Synapse. Если вы подключаетесь напрямую к Azure Key Vault без использования связанной службы, вы будете выполнять аутентификацию с учетными данными пользователя Azure Active Directory.

Дополнительные сведения см. в статье о [связанных службах](../../data-factory/concepts-linked-services.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="usage"></a>Использование

### <a name="tokenlibraryhelp"></a>TokenLibrary.help()
Эта функция отображает справочную документацию по TokenLibrary.

::: zone pivot = "programming-language-scala"

```scala
TokenLibrary.help()
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
TokenLibrary.help()
```

::: zone-end

::: zone pivot = "programming-language-csharp"

```csharp
Console.WriteLine(TokenLibrary.help());
```

::: zone-end

## <a name="tokenlibrary-for-azure-data-lake-storage-gen2"></a>TokenLibrary для Azure Data Lake Storage 2-го поколения

#### <a name="adls-gen2-primary-storage"></a>Основное хранилище Azure Data Lake Storage 2-го поколения

При доступе к файлам из основного хранилища Azure Data Lake Storage по умолчанию используется сквозная аутентификация Azure Active Directory, для которой не требуется явное использование TokenLibrary.

::: zone pivot = "programming-language-scala"

```scala
val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")
display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>')
display(df.limit(10))
```

::: zone-end

#### <a name="adls-gen2-storage-with-linked-services"></a>Хранилище ADLS 2-го поколения со связанными службами

Synapse предоставляет интегрированную среду связанных служб для подключения к Azure Data Lake Storage 2-го поколения.  Для связанных служб можно настроить аутентификацию с использованием **ключа учетной записи**, **субъекта-службы**, **управляемого удостоверения** или **учетных данных**.

Если в качестве метода аутентификации для связанной службы выбран **ключ учетной записи**, связанная служба будет выполнять аутентификацию с использованием предоставленного ключа учетной записи хранения, запрашивать ключ SAS и автоматически применять его к запросу хранилища через поставщика **LinkedServiceBasedSASProvider**.

::: zone pivot = "programming-language-scala"

```scala
val sc = spark.sparkContext
spark.conf.set("spark.storage.synapse.linkedServiceName", "<LINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedSASProvider")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
# Python code
spark.conf.set("spark.storage.synapse.linkedServiceName", "<lINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedSASProvider")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<DIRECTORY PATH>')

df.show()
```

::: zone-end

Если в качестве метода аутентификации для связанной службы выбрано **управляемое удостоверение** или **субъект-служба**, связанная служба будет использовать управляемое удостоверение или маркер субъекта-службы с поставщиком **LinkedServiceBasedTokenProvider**.  


::: zone pivot = "programming-language-scala"

```scala
val sc = spark.sparkContext
spark.conf.set("spark.storage.synapse.linkedServiceName", "<LINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.oauth.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedTokenProvider") 
val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
# Python code
spark.conf.set("spark.storage.synapse.linkedServiceName", "<lINKED SERVICE NAME>")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedTokenProvider")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<DIRECTORY PATH>')

df.show()
```

::: zone-end

#### <a name="adls-gen2-storage-without-linked-services"></a>Хранилище ADLS 2-го поколения без связанных служб

Вы можете подключиться к хранилищу ADLS 2-го поколения напрямую с помощью ключа SAS через поставщика **ConfBasedSASProvider**. Для этого также укажите ключ SAS в параметре конфигурации **spark.storage.synapse.sas**.

::: zone pivot = "programming-language-scala"

```scala
%%spark
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.ConfBasedSASProvider")
spark.conf.set("spark.storage.synapse.sas", "<SAS KEY>")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark

spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.ConfBasedSASProvider")
spark.conf.set("spark.storage.synapse.sas", "<SAS KEY>")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>')

display(df.limit(10))
```

::: zone-end

#### <a name="adls-gen2-storage-with-azure-key-vault"></a>Хранилище ADLS 2-го поколения с Azure Key Vault

Вы можете подключиться к хранилищу ADLS 2-го поколения с помощью маркера SAS, сохраненного в секрете Azure Key Vault.  

::: zone pivot = "programming-language-scala"

```scala
%%spark
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.AkvBasedSASProvider")
spark.conf.set("spark.storage.synapse.akv", "<AZURE KEY VAULT NAME>")
spark.conf.set("spark.storage.akv.secret", "<SECRET KEY>")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.AkvBasedSASProvider")
spark.conf.set("spark.storage.synapse.akv", "<AZURE KEY VAULT NAME>")
spark.conf.set("spark.storage.akv.secret", "<SECRET KEY>")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>')

display(df.limit(10))
```

::: zone-end

## <a name="tokenlibrary-for-other-linked-services"></a>TokenLibrary для других связанных служб

Чтобы подключиться к другим связанным службам, можно выполнить прямой вызов TokenLibrary.

#### <a name="getconnectionstring"></a>getConnectionString()

 Чтобы получить строку подключения, используйте функцию **getConnectionString** и передайте **имя связанной службы**.

::: zone pivot = "programming-language-scala"

```scala
%%spark
// retrieve connectionstring from TokenLibrary

import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val connectionString: String = TokenLibrary.getConnectionString("<LINKED SERVICE NAME>")
println(connectionString)
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
# retrieve connectionstring from TokenLibrary

from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary
connection_string = token_library.getConnectionString("<LINKED SERVICE NAME>")
print(connection_string)
```

::: zone-end

::: zone pivot = "programming-language-csharp"

```csharp
%%csharp
// retrieve connectionstring from TokenLibrary

using Microsoft.Spark.Extensions.Azure.Synapse.Analytics.Utils;

string connectionString = TokenLibrary.GetConnectionString(<LINKED SERVICE NAME>);
Console.WriteLine(connectionString);
```

::: zone-end

#### <a name="getconnectionstringasmap"></a>getConnectionStringAsMap()

getConnectionStringAsMap — это вспомогательная функция, доступная в Scala и Python для анализа конкретных значений из пар _ключ=значение_ в строке подключения, например следующих:

_`DefaultEndpointsProtocol=https;AccountName=<ACCOUNT NAME>;AccountKey=<ACCOUNT KEY>`_

используйте функцию **getConnectionStringAsMap** и передайте ключ, чтобы вернуть значение.  В приведенном выше примере строки подключения 

_**TokenLibrary.getConnectionStringAsMap("DefaultEndpointsProtocol")**_

возвращает

**_"https"_**

::: zone pivot = "programming-language-scala"

```scala
// Linked services can be used for storing and retrieving credentials (e.g, account key)
// Example connection string (for storage): "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val accountKey: String = TokenLibrary.getConnectionStringAsMap("<LINKED SERVICE NAME">).get("<KEY NAME>")
println(accountKey)
```
::: zone-end

::: zone pivot = "programming-language-python"

```python
# Linked services can be used for storing and retrieving credentials (e.g, account key)
# Example connection string (for storage): "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary
accountKey = token_library.getConnectionStringAsMap("<LINKED SERVICE NAME>").get("<KEY NAME>")
print(accountKey)
```

::: zone-end

#### <a name="getsecret"></a>getSecret()

Для получения секрета, хранящегося в Azure Key Vault, мы рекомендуем создать связанную службу для Azure Key Vault в рабочей области Synapse. При этом удостоверению управляемой службы рабочей области Synapse потребуется предоставить права на **получение** секретов из Azure Key Vault.  Связанная служба будет использовать удостоверение управляемой службы для подключения к службе Azure Key Vault, чтобы получить секрет.  В противном случае при подключении напрямую к Azure Key Vault будут использоваться учетные данные пользователя Azure Active Directory.  В такой ситуации необходимо будет предоставить пользователю разрешения на получение секретов из Azure Key Vault.

`TokenLibrary.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>" [, <LINKED SERVICE NAME>])`

Чтобы получить секрет из Azure Key Vault, используйте функцию **TokenLibrary.getSecret()** .

::: zone pivot = "programming-language-scala"

```scala
import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val connectionString: String = TokenLibrary.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>", "<LINKED SERVICE NAME>")
println(connectionString)
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
import sys
from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

connection_string = token_library.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>", "<LINKED SERVICE NAME>")
print(connection_string)
```

::: zone-end

::: zone pivot = "programming-language-csharp"

```csharp
using Microsoft.Spark.Extensions.Azure.Synapse.Analytics.Utils;

string connectionString = TokenLibrary.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>", "<LINKED SERVICE NAME>");
Console.WriteLine(connectionString);
```

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

- [Запись в выделенный пул SQL](./synapse-spark-sql-pool-import-export.md)
