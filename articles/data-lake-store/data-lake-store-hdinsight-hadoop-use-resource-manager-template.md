---
title: Шаблон — кластер HDInsight с Data Lake Storage 1-го поколения
description: Используйте шаблоны Azure Resource Manager для создания и использования кластеров Azure HDInsight с Azure Data Lake Storage 1-го поколения.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: a7283ad4c4c61ecc293a55ffc4cb9626bb28d630
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92108734"
---
# <a name="create-an-hdinsight-cluster-with-azure-data-lake-storage-gen1-using-azure-resource-manager-template"></a>Создание кластера HDInsight с Azure Data Lake Storage 1-го поколения с помощью шаблона Azure Resource Manager
> [!div class="op_single_selector"]
> * [Использование портала](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Использование PowerShell (для хранилища по умолчанию)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [Использование PowerShell (для дополнительного хранилища)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [Использование Resource Manager](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)
>
>

Узнайте, как с помощью Azure PowerShell настроить кластер HDInsight с Azure Data Lake Storage 1-го поколения в качестве **дополнительного хранилища**.

Для поддерживаемых типов кластеров Data Lake Storage 1-го поколения можно использовать в качестве хранилища по умолчанию или в качестве дополнительной учетной записи хранения. Если Data Lake Storage 1-го поколения используется в качестве дополнительного хранилища, учетная запись хранения по умолчанию для кластеров по-прежнему будет храниться в хранилище BLOB-объектов Azure (WASB), а файлы, связанные с кластером (такие как журналы и т. д.), сохраняются в хранилище по умолчанию, а данные, которые требуется обработать, могут храниться в учетной записи Data Lake Storage 1-го поколения. Использование Data Lake Storage 1-го поколения в качестве дополнительной учетной записи хранения не влияет на производительность или возможность выполнять чтение и запись в хранилище из кластера.

## <a name="using-data-lake-storage-gen1-for-hdinsight-cluster-storage"></a>Использование Data Lake Storage 1-го поколения в качестве хранилища кластера HDInsight

Ниже приведены некоторые важные сведения об использовании HDInsight с Data Lake Storage 1-го поколения.

* Возможность создавать кластеры HDInsight, использующие Data Lake Storage 1-го поколения в качестве хранилища по умолчанию, поддерживается для HDInsight версий 3.5 и 3.6.

* Возможность создавать кластеры HDInsight с доступом к Data Lake Storage 1-го поколения в качестве дополнительного хранилища поддерживается для версий HDInsight 3.2, 3.4, 3.5 и 3.6.

В этой статье мы подготовим кластер Hadoop, в котором Data Lake Storage 1-го поколения будет дополнительным хранилищем. Инструкции по созданию кластера Hadoop с Data Lake Storage 1-го поколения в качестве хранилища по умолчанию см. в статье [Создание кластера HDInsight с Data Lake Storage 1-го поколения с помощью портал Azure](data-lake-store-hdinsight-hadoop-use-portal.md).

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Перед началом работы с этим учебником необходимо иметь следующее:

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1,0 или более поздней** версии. Ознакомьтесь со статьей [Установка и настройка Azure PowerShell](/powershell/azure/).
* **Субъект-служба Azure Active Directory**. В этом учебнике приведены инструкции по созданию субъекта-службы в Azure AD. Однако, чтобы создать субъект-службу, необходимо быть администратором Azure AD. Если вы являетесь администратором Azure AD, то можете пропустить это предварительное требование и продолжить работу с учебником.

    **Если вы не являетесь администратором Azure AD**, то вы не сможете выполнить шаги, необходимые для создания субъекта-службы. В этом случае администратор Azure AD должен сначала создать субъект-службу, после чего вы сможете создать кластер HDInsight с Data Lake Storage 1-го поколения. При создании субъекта-службы также необходимо использовать сертификат, как описано в разделе [Create a service principal with certificate](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-certificate-from-certificate-authority) (Создание субъекта-службы с сертификатом).

## <a name="create-an-hdinsight-cluster-with-data-lake-storage-gen1"></a>Создание кластера HDInsight с Data Lake Storage 1-го поколения
Шаблон Resource Manager и предварительные требования для использования шаблона см. в статье [Deploy a HDInsight Linux cluster with new Data Lake Store](https://github.com/Azure/azure-quickstart-templates/tree/master/201-hdinsight-datalake-store-azure-storage) (Развертывание кластера HDInsight Linux с помощью нового Data Lake Storage 1-го поколения) на GitHub. Следуйте инструкциям по этой ссылке, чтобы создать кластер HDInsight с Data Lake Storage 1-го поколения в качестве дополнительного хранилища.

Инструкции по ссылке выше требуют PowerShell. Прежде чем начать работу с ними, обязательно войдите в учетную запись Azure. На своем компьютере откройте новое окно Azure PowerShell и введите следующий фрагмент кода. Когда вам будет предложено войти, введите учетные данные администратора или владельца подписки.

```
# Log in to your Azure account
Connect-AzAccount

# List all the subscriptions associated to your account
Get-AzSubscription

# Select a subscription
Set-AzContext -SubscriptionId <subscription ID>
```

Шаблон развертывает ресурсы следующих типов:

* [Microsoft. Data Lake Store/учетные записи](/azure/templates/microsoft.datalakestore/accounts)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft. HDInsight/кластеры](/azure/templates/microsoft.hdinsight/clusters)

## <a name="upload-sample-data-to-data-lake-storage-gen1"></a>Отправка примера данных в Data Lake Storage 1-го поколения
Шаблон диспетчер ресурсов создает новую учетную запись хранения с Data Lake Storage 1-го поколения и связывает ее с кластером HDInsight. На этом этапе необходимо отправить пример данных в Data Lake Storage 1-го поколения. Эти данные понадобятся позже в этом руководстве для запуска заданий из кластера HDInsight, которые обращаются к данным в учетной записи хранения с Data Lake Storage 1-го поколения. Инструкции по отправке данных см. в разделе [Отправка файла в Data Lake Storage 1-го поколения](data-lake-store-get-started-portal.md#uploaddata). Если у вас нет под рукой подходящих для этих целей данных, передайте папку **Ambulance Data** из [репозитория Git для озера данных Azure](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData).

## <a name="set-relevant-acls-on-the-sample-data"></a>Установка соответствующих списков управления доступом для примера данных
Чтобы обеспечить доступ к отправляемым примерам данных для кластера HDInsight, необходимо убедиться, что у приложения Azure AD, используемого для определения удостоверения между кластером HDInsight и Data Lake Storage 1-го поколения, есть доступ к файлу или папке, к которой вы пытаетесь получить доступ. Для этого сделайте следующее.

1. Найдите имя приложения Azure AD, связанного с кластером HDInsight, и учетную запись хранения с Data Lake Storage 1-го поколения. Один из способов поиска имени — открыть колонку кластера HDInsight, созданную с помощью шаблона диспетчер ресурсов, перейдите на вкладку **удостоверение Azure AD** и найдите значение **Отображаемое имя субъекта-службы**.
2. Теперь предоставьте этому приложению Azure AD доступ к файлу или папке, к которым необходимо получить доступ из кластера HDInsight. Чтобы установить подходящие списки управления доступом для файла или папки в Data Lake Storage 1-го поколения, см. статью [Защита данных, хранимых в Data Lake Storage 1-го поколения](data-lake-store-secure-data.md#filepermissions).

## <a name="run-test-jobs-on-the-hdinsight-cluster-to-use-data-lake-storage-gen1"></a>Выполнение тестовых заданий в кластере HDInsight для использования Data Lake Storage 1-го поколения
После настройки кластера HDInsight выполните в нем тестовые задания, чтобы проверить, доступно ли ему Data Lake Storage 1-го поколения. Для этого мы выполним пример задания Hive, которое создает таблицу, используя демонстрационные данные, отправленные ранее в учетную запись хранения с Data Lake Storage 1-го поколения.

В этом разделе вы подключитесь к кластеру HDInsight под управлением Linux по протоколу SSH и выполните пример запроса Hive. При использовании клиента Windows мы рекомендуем использовать служебную программу **PuTTY**, которую можно скачать на странице [https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Дополнительные сведения об использовании PuTTY см. в разделе [Использование SSH с Hadoop на основе Linux в HDInsight из Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

1. После подключения запустите интерфейс командной строки Hive с помощью следующей команды:

   ```
   hive
   ```
2. Используя интерфейс командной строки, введите следующие инструкции, чтобы создать таблицу с именем **vehicles** с помощью примера данных в Data Lake Storage 1-го поколения:

   ```
   DROP TABLE vehicles;
   CREATE EXTERNAL TABLE vehicles (str string) LOCATION 'adl://<mydatalakestoragegen1>.azuredatalakestore.net:443/';
   SELECT * FROM vehicles LIMIT 10;
   ```

   Выходные данные должны иметь следующий вид.

   ```
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

В этом разделе вы подключитесь к кластеру HDInsight под управлением Linux по протоколу SSH и выполните команды HDFS. При использовании клиента Windows мы рекомендуем использовать служебную программу **PuTTY**, которую можно скачать на странице [https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Дополнительные сведения об использовании PuTTY см. в разделе [Использование SSH с Hadoop на основе Linux в HDInsight из Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

После подключения используйте следующую команду файловой системы HDFS, чтобы вывести список файлов в учетной записи хранения с Data Lake Storage 1-го поколения.

```
hdfs dfs -ls adl://<storage account with Data Lake Storage Gen1 name>.azuredatalakestore.net:443/
```

Эта команда должна показать файл, который вы ранее отправили в Data Lake Storage 1-го поколения.

```
15/09/17 21:41:15 INFO web.CaboWebHdfsFileSystem: Replacing original urlConnectionFactory with org.apache.hadoop.hdfs.web.URLConnectionFactory@21a728d6
Found 1 items
-rwxrwxrwx   0 NotSupportYet NotSupportYet     671388 2015-09-16 22:16 adl://mydatalakestoragegen1.azuredatalakestore.net:443/mynewfolder
```

С помощью команды `hdfs dfs -put` вы можете передать несколько файлов в Data Lake Storage 1-го поколения, а затем с помощью команды `hdfs dfs -ls` проверить, успешно ли они передались.


## <a name="next-steps"></a>Дальнейшие действия
* [Копирование данных из больших двоичных объектов хранилища Azure в хранилище озера данных](data-lake-store-copy-data-wasb-distcp.md)
* [Использование Data Lake Storage 1-го поколения с кластерами Azure HDInsight](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen1.md)