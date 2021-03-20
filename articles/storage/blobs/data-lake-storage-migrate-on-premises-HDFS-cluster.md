---
title: Перенос из локального хранилища HDFS в службу хранилища Azure с помощью Azure Data Box
description: Перенос данных из локального хранилища HDFS в службу хранилища Azure (хранилище BLOB-объектов или Data Lake Storage 2-го поколения) с помощью устройства Data Box.
author: normesta
ms.service: storage
ms.date: 02/14/2019
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: jamesbak
ms.openlocfilehash: e58137dd680ff9a2be2bd657f0969304b526873f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95913119"
---
# <a name="migrate-from-on-prem-hdfs-store-to-azure-storage-with-azure-data-box"></a>Перенос из локального хранилища HDFS в службу хранилища Azure с помощью Azure Data Box

Вы можете перенести данные из локального хранилища HDFS кластера Hadoop в службу хранилища Azure (хранилище BLOB-объектов или Data Lake Storage 2-го поколения) с помощью устройства Data Box. Вы можете выбрать один из Диск Data Box, Data Box 80 ТБ или 770 ТБ Data Box Heavy.

Эта статья поможет вам выполнить следующие задачи:

> [!div class="checklist"]
> * Подготовка к переносу данных.
> * Скопируйте данные на Диск Data Box, Data Box или на устройство Data Box Heavy.
> * Отправьте устройство обратно в корпорацию Майкрософт.
> * Применение разрешений доступа к файлам и каталогам (только Data Lake Storage 2-го поколения)

## <a name="prerequisites"></a>Предварительные условия

Эти действия необходимы для завершения миграции.

* Учетная запись хранения Azure.

* Локальный кластер Hadoop, содержащий исходные данные.

* [Устройство Azure Data Box](https://azure.microsoft.com/services/storage/databox/).

  * [Закажите Data Box](../../databox/data-box-deploy-ordered.md) или [Data Box Heavy](../../databox/data-box-heavy-deploy-ordered.md). 

  * Кабель и подключите [Data Box](../../databox/data-box-deploy-set-up.md) или [Data Box Heavy](../../databox/data-box-heavy-deploy-set-up.md) к локальной сети.

Если все готово, начнем.

## <a name="copy-your-data-to-a-data-box-device"></a>Копирование данных на устройство Data Box

Если данные помещаются в одно устройство Data Box, данные будут скопированы на устройство Data Box. 

Если размер данных превышает емкость устройства Data Box, используйте [необязательную процедуру, чтобы разделить данные между несколькими Data Box устройствами](#appendix-split-data-across-multiple-data-box-devices) , а затем выполните этот шаг. 

Чтобы скопировать данные из локального хранилища HDFS на устройство Data Box, необходимо установить несколько элементов, а затем использовать средство [DistCp](https://hadoop.apache.org/docs/stable/hadoop-distcp/DistCp.html) .

Выполните следующие действия, чтобы скопировать данные с помощью интерфейсов API для хранилища BLOB-объектов или объекта на устройство Data Box. Интерфейс REST API сделает устройство отображаемым в кластере в качестве хранилища HDFS.

1. Прежде чем копировать данные через остальное, укажите примитивы безопасности и подключения для подключения к интерфейсу RESTFUL на Data Box или Data Box Heavy. Войдите в локальный веб-интерфейс Data Box и перейдите на страницу **подключение и копирование** . В учетной записи хранения Azure для устройства в разделе **параметры доступа** выберите, а затем — **остальные**.

    ![Страница "подключение и копирование"](media/data-lake-storage-migrate-on-premises-HDFS-cluster/data-box-connect-rest.png)

2. В диалоговом окне доступ к учетной записи хранения и отправка данных скопируйте **конечную точку службы BLOB-объектов** и **ключ учетной записи хранения**. В конечной точке службы BLOB-объектов опустите `https://` и замыкающую косую черту.

    В этом случае конечной точкой является: `https://mystorageaccount.blob.mydataboxno.microsoftdatabox.com/` . Часть URI, которую вы будете использовать, имеет следующие узлы: `mystorageaccount.blob.mydataboxno.microsoftdatabox.com` . Пример см. в разделе [Подключение к оставшимся по протоколу HTTP](../../databox/data-box-deploy-copy-data-via-rest.md). 

     ![Диалоговое окно "доступ к учетной записи хранения и передача данных"](media/data-lake-storage-migrate-on-premises-HDFS-cluster/data-box-connection-string-http.png)

3. Добавьте конечную точку и Data Box или IP-адрес узла Data Box Heavy на `/etc/hosts` каждый узел.

    ```    
    10.128.5.42  mystorageaccount.blob.mydataboxno.microsoftdatabox.com
    ```

    Если вы используете другой механизм для DNS, следует убедиться, что конечная точка Data Box может быть разрешена.

4. Задайте для переменной оболочки `azjars` расположение `hadoop-azure` `azure-storage` JAR-файлов и. Эти файлы можно найти в каталоге установки Hadoop.

    Чтобы определить, существуют ли эти файлы, используйте следующую команду: `ls -l $<hadoop_install_dir>/share/hadoop/tools/lib/ | grep azure` . Замените `<hadoop_install_dir>` заполнитель на путь к каталогу, в котором установлен Hadoop. Обязательно используйте полные пути.

    Примеры:

    `azjars=$hadoop_install_dir/share/hadoop/tools/lib/hadoop-azure-2.6.0-cdh5.14.0.jar` `azjars=$azjars,$hadoop_install_dir/share/hadoop/tools/lib/microsoft-windowsazure-storage-sdk-0.6.0.jar`

5. Создайте контейнер хранилища, который будет использоваться для копирования данных. Также следует указать целевой каталог в составе этой команды. На этом этапе это может быть фиктивный каталог назначения.

    ```
    hadoop fs -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.<blob_service_endpoint>=<account_key> \
    -mkdir -p  wasb://<container_name>@<blob_service_endpoint>/<destination_directory>
    ```

    * Замените `<blob_service_endpoint>` заполнитель именем конечной точки службы BLOB-объектов.

    * Замените `<account_key>` заполнитель ключом доступа вашей учетной записи.

    * Замените `<container-name>` заполнитель именем своего контейнера.

    * Замените `<destination_directory>` заполнитель именем каталога, в который необходимо скопировать данные.

6. Выполните команду list, чтобы убедиться, что контейнер и каталог были созданы.

    ```
    hadoop fs -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.<blob_service_endpoint>=<account_key> \
    -ls -R  wasb://<container_name>@<blob_service_endpoint>/
    ```

   * Замените `<blob_service_endpoint>` заполнитель именем конечной точки службы BLOB-объектов.

   * Замените `<account_key>` заполнитель ключом доступа вашей учетной записи.

   * Замените `<container-name>` заполнитель именем своего контейнера.

7. Скопируйте данные из хранилища HDFS в Data Box хранилище BLOB-объектов в созданный ранее контейнер. Если каталог, в который выполняется копирование, не найден, команда автоматически создаст его.

    ```
    hadoop distcp \
    -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.<blob_service_endpoint<>=<account_key> \
    -filters <exclusion_filelist_file> \
    [-f filelist_file | /<source_directory> \
           wasb://<container_name>@<blob_service_endpoint>/<destination_directory>
    ```

    * Замените `<blob_service_endpoint>` заполнитель именем конечной точки службы BLOB-объектов.

    * Замените `<account_key>` заполнитель ключом доступа вашей учетной записи.

    * Замените `<container-name>` заполнитель именем своего контейнера.

    * Замените `<exlusion_filelist_file>` заполнитель именем файла, который содержит список исключений файлов.

    * Замените `<source_directory>` заполнитель именем каталога, содержащего данные, которые необходимо скопировать.

    * Замените `<destination_directory>` заполнитель именем каталога, в который необходимо скопировать данные.

    `-libjars`Параметр используется, чтобы сделать `hadoop-azure*.jar` и зависимые `azure-storage*.jar` файлы доступными для `distcp` . Это может быть уже для некоторых кластеров.

    В следующем примере показано, как `distcp` команда используется для копирования данных.

    ```
     hadoop distcp \
    -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.mystorageaccount.blob.mydataboxno.microsoftdatabox.com=myaccountkey \
    -filter ./exclusions.lst -f /tmp/copylist1 -m 4 \
    /data/testfiles \
    wasb://hdfscontainer@mystorageaccount.blob.mydataboxno.microsoftdatabox.com/data
    ```
  
    Чтобы повысить скорость копирования, сделайте следующее:

    * Попробуйте изменить число модулей сопоставления. (В приведенном выше примере используются модули `m` сопоставления = 4).

    * Попробуйте выполнить несколько `distcp` параллельно.

    * Помните, что большие файлы работают лучше, чем небольшие файлы.

## <a name="ship-the-data-box-to-microsoft"></a>Доставка Data Box в корпорацию Майкрософт

Выполните следующие действия, чтобы подготовить и доставить устройство Data Box в корпорацию Майкрософт.

1. Сначала  [Подготовка к отправке на Data Box или Data Box Heavy](../../databox/data-box-deploy-copy-data-via-rest.md).

2. После завершения подготовки устройства Скачайте файлы спецификации. Эти файлы BOM или manifest будут использоваться позже для проверки данных, передаваемых в Azure.

3. Завершите работу устройства и удалите кабели.

4. Вызовите курьера для отправки посылки службой доставки UPS.

    * Сведения об устройствах Data Box см. [в разделе доставка Data Box](../../databox/data-box-deploy-picked-up.md).

    * Сведения об устройствах Data Box Heavy см. [в разделе доставка Data Box Heavy](../../databox/data-box-heavy-deploy-picked-up.md).

5. После получения устройства Корпорация Майкрософт подключается к сети центра обработки данных, и данные передаются в учетную запись хранения, указанную при помещении заказа устройства. Проверьте соответствие файлов спецификации с файлами спецификаций, которые передаются в Azure. 

## <a name="apply-access-permissions-to-files-and-directories-data-lake-storage-gen2-only"></a>Применение разрешений доступа к файлам и каталогам (только Data Lake Storage 2-го поколения)

У вас уже есть данные в учетной записи хранения Azure. Теперь вы будете применять разрешения на доступ к файлам и каталогам.

> [!NOTE]
> Этот шаг необходим только в том случае, если в качестве хранилища данных используется Azure Data Lake Storage 2-го поколения. Если вы используете только учетную запись хранения BLOB-объектов без иерархического пространства имен в качестве хранилища данных, этот раздел можно пропустить.

### <a name="create-a-service-principal-for-your-azure-data-lake-storage-gen2-account"></a>Создание субъекта-службы для учетной записи Azure Data Lake Storage 2-го поколения

Сведения о создании субъекта-службы см. в статье [как использовать портал для создания приложения Azure AD и субъекта-службы, которые могут получать доступ к ресурсам](../../active-directory/develop/howto-create-service-principal-portal.md).

* При выполнении действий, описанных в разделе [Назначение приложению роли](../../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application) этой статьи, не забудьте назначить субъекту-службе роль **участника данных BLOB-объектов хранилища**.

* При выполнении действий, описанных в разделе [Получение значений для входа в](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) статье, сохраните идентификатор приложения и значения секрета клиента в текстовом файле. Они вам скоро понадобятся.

### <a name="generate-a-list-of-copied-files-with-their-permissions"></a>Создать список скопированных файлов с разрешениями

Выполните следующую команду из локального кластера Hadoop:

```bash

sudo -u hdfs ./copy-acls.sh -s /{hdfs_path} > ./filelist.json
```

Эта команда создает список скопированных файлов с разрешениями.

> [!NOTE]
> В зависимости от числа файлов в HDFS выполнение этой команды может занять много времени.

### <a name="generate-a-list-of-identities-and-map-them-to-azure-active-directory-add-identities"></a>Создание списка удостоверений и их сопоставьте с Azure Active Directory (добавление) удостоверениями

1. Скачайте `copy-acls.py` скрипт. См. раздел [Загрузка вспомогательных сценариев и настройка пограничных узлов для их запуска в](#download-helper-scripts) этой статье.

2. Выполните эту команду, чтобы создать список уникальных удостоверений.

   ```bash
   
   ./copy-acls.py -s ./filelist.json -i ./id_map.json -g
   ```

   Этот скрипт создает файл с именем `id_map.json` , содержащий удостоверения, которые необходимо сопоставлять с удостоверениями на основе надстроек.

3. Откройте файл `id_map.json` в текстовом редакторе.

4. Для каждого объекта JSON, который отображается в файле, обновите `target` атрибут имени участника-пользователя AAD (UPN) или ObjectID (OID) с соответствующим сопоставленным удостоверением. Когда все будет готово, сохраните файл. Этот файл потребуется на следующем шаге.

### <a name="apply-permissions-to-copied-files-and-apply-identity-mappings"></a>Применить разрешения к скопированным файлам и применить сопоставления удостоверений

Выполните следующую команду, чтобы применить разрешения к данным, скопированным в учетную запись Data Lake Storage 2-го поколения:

```bash
./copy-acls.py -s ./filelist.json -i ./id_map.json  -A <storage-account-name> -C <container-name> --dest-spn-id <application-id>  --dest-spn-secret <client-secret>
```

* Замените заполнитель `<storage-account-name>` именем вашей учетной записи хранения.

* Замените `<container-name>` заполнитель именем своего контейнера.

* Замените `<application-id>` `<client-secret>` заполнители и на идентификатор приложения и секрет клиента, собранные при создании субъекта-службы.

## <a name="appendix-split-data-across-multiple-data-box-devices"></a>Приложение. разбиение данных между несколькими Data Box устройствами

Перед переносом данных на устройство Data Box необходимо скачать некоторые вспомогательные скрипты, убедиться, что данные упорядочены в соответствии с устройством Data Box, и исключить все ненужные файлы.

<a id="download-helper-scripts"></a>

### <a name="download-helper-scripts-and-set-up-your-edge-node-to-run-them"></a>Скачивание вспомогательных скриптов и настройка пограничных узлов для их запуска

1. На пограничном или головном узле локального кластера Hadoop выполните следующую команду:

   ```bash
   
   git clone https://github.com/jamesbak/databox-adls-loader.git
   cd databox-adls-loader
   ```

   Эта команда создает клон репозитория GitHub, содержащего вспомогательные скрипты.

2. Убедитесь, что на локальном компьютере установлен пакет [JQ](https://stedolan.github.io/jq/) .

   ```bash
   
   sudo apt-get install jq
   ```

3. Установите пакет [запросов](https://2.python-requests.org/en/master/) Python.

   ```bash
   
   pip install requests
   ```

4. Задайте разрешения на выполнение для необходимых скриптов.

   ```bash
   
   chmod +x *.py *.sh

   ```

### <a name="ensure-that-your-data-is-organized-to-fit-onto-a-data-box-device"></a>Убедитесь, что данные организованы для размещения на Data Box устройстве.

Если размер данных превышает размер одного Data Box устройства, можно разделить файлы в группы, которые можно хранить на нескольких Data Box устройствах.

Если данные не превышают размер одного устройства Data Box, можно перейти к следующему разделу.

1. С повышенными разрешениями запустите `generate-file-list` сценарий, скачанный, следуя инструкциям, приведенным в предыдущем разделе.

   Ниже приведено описание параметров команды.

   ```
   sudo -u hdfs ./generate-file-list.py [-h] [-s DATABOX_SIZE] [-b FILELIST_BASENAME]
                    [-f LOG_CONFIG] [-l LOG_FILE]
                    [-v {DEBUG,INFO,WARNING,ERROR}]
                    path

   where:
   positional arguments:
   path                  The base HDFS path to process.

   optional arguments:
   -h, --help            show this help message and exit
   -s DATABOX_SIZE, --databox-size DATABOX_SIZE
                        The size of each Data Box in bytes.
   -b FILELIST_BASENAME, --filelist-basename FILELIST_BASENAME
                        The base name for the output filelists. Lists will be
                        named basename1, basename2, ... .
   -f LOG_CONFIG, --log-config LOG_CONFIG
                        The name of a configuration file for logging.
   -l LOG_FILE, --log-file LOG_FILE
                        Name of file to have log output written to (default is
                        stdout/stderr)
   -v {DEBUG,INFO,WARNING,ERROR}, --log-level {DEBUG,INFO,WARNING,ERROR}
                        Level of log information to output. Default is 'INFO'.
   ```

2. Скопируйте созданные списки файлов в HDFS, чтобы они были доступны для задания [DistCp](https://hadoop.apache.org/docs/stable/hadoop-distcp/DistCp.html) .

   ```
   hadoop fs -copyFromLocal {filelist_pattern} /[hdfs directory]
   ```

### <a name="exclude-unnecessary-files"></a>Исключить ненужные файлы

Необходимо исключить некоторые каталоги из задания Дискп. Например, исключите каталоги, содержащие сведения о состоянии, которые сохраняют работу кластера.

В локальном кластере Hadoop, где планируется инициировать задание DistCp, создайте файл, указывающий список каталогов, которые необходимо исключить.

Ниже приведен пример:

```
.*ranger/audit.*
.*/hbase/data/WALs.*
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как Data Lake Storage 2-го поколения работает с кластерами HDInsight. См. раздел [Use Azure Data Lake Storage Gen2 with Azure HDInsight clusters](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md) (Использование хранилища Azure Data Lake поколения 2 с кластерами Azure HDInsight).