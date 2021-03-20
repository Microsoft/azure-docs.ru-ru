---
title: Управление зависимостями приложений Spark в Azure HDInsight
description: В этой статье приводятся общие сведения об управлении зависимостями Spark в кластере HDInsight Spark для приложений PySpark и Scala.
author: yanancai
ms.author: yanacai
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 09/09/2020
ms.openlocfilehash: f0673523c74a0ea298e7d2d520952c3e98877e91
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98930044"
---
# <a name="manage-spark-application-dependencies"></a>Управление зависимостями приложений Spark

Из этой статьи вы узнаете, как управлять зависимостями для приложений Spark, работающих в HDInsight. Мы рассмотрим как Scala, так и PySpark в приложении Spark и области кластера.

Используйте быстрые ссылки, чтобы перейти к разделу, основанному на пользовательском сценарии:
* [Настройка зависимостей JAR-файлов заданий Spark с помощью Jupyter Notebook](#use-jupyter-notebook)
* [Настройка зависимостей JAR-файлов заданий Spark с помощью Azure Toolkit for IntelliJ](#use-azure-toolkit-for-intellij)
* [Настройка зависимостей JAR для кластера Spark](#jar-libs-for-cluster)
* [Безопасное управление зависимостями JAR](#safely-manage-jar-dependencies)
* [Настройка пакетов Python задания Spark с помощью Jupyter Notebook](#use-jupyter-notebook-1)
* [Безопасное управление пакетами Python для кластера Spark](#python-packages-for-cluster)

## <a name="jar-libs-for-one-spark-job"></a>Библиотеки JAR для одного задания Spark
### <a name="use-jupyter-notebook"></a>Использование Jupyter Notebook
Когда сеанс Spark начинается в Jupyter Notebook в ядре Spark для Scala, можно настроить пакеты из:

* [Репозиторий Maven](https://search.maven.org/)или пакеты, участвующие в сообществе, в [пакетах Spark](https://spark-packages.org/).
* JAR-файлы, хранящиеся в основном хранилище кластера.

С помощью `%%configure` волшебной настройки записной книжки можно настроить использование внешнего пакета. В записных книжках, которые используют внешние пакеты, следует вызывать волшебную команду `%%configure` в первой ячейке кода. Она настраивает ядро для использования пакета до начала сеанса.

>
>[!IMPORTANT]  
>Если вы забыли настроить ядро в первой ячейке, можно использовать `%%configure` с параметром `-f`, но при этом сеанс будет перезапущен и все изменения будут утеряны.

**Пример для пакетов из репозитория Maven или пакетов Spark**

После обнаружения пакета из репозитория Maven Соберите значения для **groupId**, **ArtifactId** и **Version**. Объедините три значения, разделив их двоеточием (**:**).

   ![Объединение схемы пакета](./media/apache-spark-manage-dependencies/spark-package-schema.png "Объединение схемы пакета")

Убедитесь, что собираемые значения соответствуют кластеру. В этом случае мы используем пакет Cosmos DB для соединителя Spark для Scala 2,11 и Spark 2,3 для кластера Spark HDInsight 3,6. Если вы не уверены, запустите `scala.util.Properties.versionString` в ячейке кода в ядре Spark, чтобы получить версию кластера Scala. Выполните команду `sc.version` , чтобы получить версию кластера Spark.

```
%%configure { "conf": {"spark.jars.packages": "com.microsoft.azure:azure-cosmosdb-spark_2.3.0_2.11:1.3.3" }}
```

**Пример для JAR, хранящегося в основном хранилище**

Используйте [схему универсального кода ресурса (URI)](../hdinsight-hadoop-linux-information.md#URI-and-scheme) для JAR-файлов в основном хранилище кластеров. Для службы хранилища Azure это будет `wasb://`, для Azure Data Lake Storage 2-го поколения — `abfs://` или `adl://` для Azure Data Lake Storage 1-го поколения. Если для службы хранилища Azure или Data Lake Storage 2-го поколения включена Безопасная отправка, URI будет иметь значение `wasbs://` или `abfss://` . См. раздел [безопасное перемещение](../../storage/common/storage-require-secure-transfer.md).

Используйте разделенный запятыми список путей JAR для нескольких JAR-файлов, стандартные маски разрешены. JAR включены в пути к классам Driver и выполнителя.

```
%%configure { "conf": {"spark.jars": "wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/azure-cosmosdb-spark_2.3.0_2.11-1.3.3.jar" }}
```

После настройки внешних пакетов можно выполнить импорт в ячейку кода, чтобы проверить правильность размещения пакетов.

```scala
import com.microsoft.azure.cosmosdb.spark._
```

### <a name="use-azure-toolkit-for-intellij"></a>Использование Azure Toolkit for IntelliJ
[Подключаемый модуль Azure Toolkit for IntelliJ](./apache-spark-intellij-tool-plugin.md) предоставляет возможности пользовательского интерфейса для отправки приложения Spark Scala в кластер HDInsight. Он предоставляет `Referenced Jars` `Referenced Files` Свойства и для настройки путей к библиотекам JAR при отправке приложения Spark. См. Дополнительные сведения об [использовании подключаемого модуля Azure Toolkit for IntelliJ для HDInsight](./apache-spark-intellij-tool-plugin.md#run-a-spark-scala-application-on-an-hdinsight-spark-cluster).

![Диалоговое окно Spark Submission (Отправка в Spark)](./media/apache-spark-intellij-tool-plugin/hdi-submit-spark-app-02.png)

## <a name="jar-libs-for-cluster"></a>Библиотеки JAR для кластера
В некоторых случаях может потребоваться настроить зависимости JAR на уровне кластера, чтобы каждое приложение могло быть настроено с одинаковыми зависимостями по умолчанию. Подход заключается в добавлении путей JAR в путь к классу для драйвера и исполнителя Spark.

1. Выполните приведенный ниже пример действий сценария для копирования JAR-файлов из основного хранилища `wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/*` в локальную файловую систему кластера `/usr/libs/sparklibs` . Этот шаг необходим, так как Linux использует `:` для разделения списка путей к классам, но HDInsight поддерживает только пути к хранилищу с такой схемой, как `wasb://` . Путь к удаленному хранилищу будет работать неправильно, если добавить его непосредственно в путь к классу.

    ```bash
    sudo mkdir -p /usr/libs/sparklibs
    sudo hadoop fs -copyToLocal wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/*.* /usr/libs/sparklibs
    ```

2. Чтобы обновить путь к классу, измените конфигурацию службы Spark с Ambari на. Перейдите в раздел **Ambari > Spark > configs > Custom Spark2 — Defaults**. **Добавьте свойство** , как показано ниже. Используйте `:` для разделения путей, если требуется добавить более одного пути. Стандартные маски разрешены.

    ```
    spark.driver.extraClassPath=/usr/libs/sparklibs/*
    spark.executor.extraClassPath=/usr/libs/sparklibs/*
    ```

   ![Изменение конфигурации Spark по умолчанию](./media/apache-spark-manage-dependencies/change-spark-default-config.png "Изменение конфигурации Spark по умолчанию")

3. Сохраните измененные конфигурации и перезапустите затронутые службы.

   ![Перезапуск затронутых служб](./media/apache-spark-manage-dependencies/restart-impacted-services.png "Перезапуск затронутых служб")

Вы можете автоматизировать шаги с помощью [действий сценария](../hdinsight-hadoop-customize-cluster-linux.md). Действие сценария для [добавления пользовательских библиотек Hive](https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh) является хорошим справочником. При изменении настроек службы Spark убедитесь, что вы используете API Ambari, а не изменяете файлы конфигурации напрямую. 

## <a name="safely-manage-jar-dependencies"></a>Безопасное управление зависимостями JAR
Кластер HDInsight имеет встроенные зависимости JAR, и обновления для этих версий JAR происходят время от времени. Чтобы избежать конфликта версий между встроенными JAR и JAR, которые вы хотите использовать для справки, рассмотрите возможность [заливки зависимостей приложения](./safely-manage-jar-dependency.md).

## <a name="python-packages-for-one-spark-job"></a>Пакеты Python для одного задания Spark
### <a name="use-jupyter-notebook"></a>Использование Jupyter Notebook
Ядро HDInsight Jupyter Notebook PySpark не поддерживает установку пакетов Python из репозитория пакетов PyPi или Anaconda напрямую. Если у вас `.zip` есть `.egg` зависимости, или `.py` , и вы хотите сослаться на них для одного сеанса Spark, выполните следующие действия.

1. Выполните приведенный ниже пример действий сценария для копирования `.zip` `.egg` или `.py` файлов из основного хранилища `wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/*` в локальную файловую систему кластера `/usr/libs/pylibs` . Этот шаг необходим, так как Linux использует `:` для разделения списка путей поиска, но HDInsight поддерживает только пути к хранилищу с такой схемой, как `wasb://` . Путь к удаленному хранилищу будет работать неправильно при использовании `sys.path.insert` .

    ```bash
    sudo mkdir -p /usr/libs/pylibs
    sudo hadoop fs -copyToLocal wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/*.* /usr/libs/pylibs
    ```

2. В записной книжке выполните приведенный ниже код в ячейке кода с ядром PySpark:

   ```python
   import sys
   sys.path.insert(0, "/usr/libs/pylibs/pypackage.zip")
   ```

3. Выполните команду `import` , чтобы проверить, были ли успешно добавлены пакеты.  

## <a name="python-packages-for-cluster"></a>Пакеты Python для кластера
Пакеты Python можно установить из Anaconda в кластер с помощью команды conda с помощью действий сценария. Установленные пакеты находятся на уровне кластера и применяются ко всем приложениям. 

В кластере HDInsight Spark есть две встроенные установки Python, Anaconda Python 2,7 и Anaconda Python 3,5. Дополнительные сведения о параметрах Python по умолчанию для служб и о том, как безопасно устанавливать внешние пакеты Python без прерывания работы кластера, см. в разделе более подробное описание [безопасного управления зависимостями Python для кластера](./apache-spark-python-package-installation.md).
