---
title: Действие скрипта для пакетов Python с Jupyter в Azure HDInsight
description: Пошаговые инструкции по использованию действия сценария для настройки записных книжек Jupyter, доступных в кластерах HDInsight Spark для использования внешних пакетов Python.
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020, devx-track-python
ms.date: 04/29/2020
ms.openlocfilehash: c3f912b4f4c2e78c44425f489927cee185b3d312
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104868721"
---
# <a name="safely-manage-python-environment-on-azure-hdinsight-using-script-action"></a>Безопасное управление средой Python в Azure HDInsight с помощью действия скрипта

В HDInsight есть две встроенные установки Python в кластере Spark: Anaconda Python 2.7 и Python 3.5. Клиентам может потребоваться настроить среду Python, например установку внешних пакетов Python. В этой статье мы покажем лучшую методику безопасного управления средами Python для кластеров Apache Spark в HDInsight.

## <a name="prerequisites"></a>Предварительные требования

Кластер Apache Spark в HDInsight. Инструкции см. в статье [Начало работы. Создание кластера Apache Spark в HDInsight на платформе Linux и выполнение интерактивных запросов с помощью SQL Spark](apache-spark-jupyter-spark-sql.md). Если у вас еще нет кластера Spark в HDInsight, можно выполнить действия скрипта во время его создания. Обратитесь к документации по [использованию настраиваемых действий сценария](../hdinsight-hadoop-customize-cluster-linux.md).

## <a name="support-for-open-source-software-used-on-hdinsight-clusters"></a>Поддержка программного обеспечения с открытым исходным кодом, используемого в кластере HDInsight

Служба Microsoft Azure HDInsight использует сформированную вокруг Apache Hadoop среду технологий с открытым кодом. Microsoft Azure предоставляет общий уровень поддержки для технологий с открытым исходным кодом. Дополнительные сведения см. на [веб-сайте вопросов и ответов службы поддержки Azure](https://azure.microsoft.com/support/faq/). Служба HDInsight предоставляет дополнительный уровень поддержки для встроенных компонентов.

В службе HDInsight доступно два типа компонентов с открытым исходным кодом.

|Компонент |Описание |
|---|---|
|Встроено|Эти компоненты предварительно установлены в кластерах HDInsight и предоставляют его базовые функциональные возможности. Например, к этой категории относится диспетчер ресурсов Apache Hadoop YARN, язык запросов Apache Hive (HiveQL) и библиотека Mahout. Полный список компонентов кластера доступен в статье [Что представляют собой компоненты и версии Apache Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md)|
|Другой|Как пользователь кластера, вы можете установить или использовать в рабочей нагрузке любой компонент, полученный от сообщества или созданный самостоятельно.|

> [!IMPORTANT]
> Компоненты, поставляемые с кластером HDInsight, полностью поддерживаются. Служба поддержки Майкрософт помогает выявлять и устранять проблемы, связанные с этими компонентами.
>
> Настраиваемые компоненты получают ограниченную коммерчески оправданную поддержку, способствующую дальнейшей диагностике проблемы. Служба поддержки Майкрософт может устранить проблему ИЛИ вас могут попросить обратиться к специалистам по технологиям с открытым исходным кодом, используя доступные каналы связи. Вы можете использовать сайты сообществ, например: [Страница вопросов и ответов Майкрософт для HDInsight](/answers/topics/azure-hdinsight.html), `https://stackoverflow.com`. Кроме того, для проектов Apache есть соответствующие сайты на `https://apache.org`.

## <a name="understand-default-python-installation"></a>Общие сведения об установке Python по умолчанию

Кластер HDInsight Spark создается с установкой Anaconda. В кластере имеются два установленных компонента Python: Anaconda Python 2.7 и Python 3.5. В таблице ниже приведены параметры Python по умолчанию для Spark, Livy и Jupyter.

|Параметр |Python 2,7|Python 3.5|
|----|----|----|
|путь|/usr/bin/anaconda/bin|/usr/bin/anaconda/envs/py35/bin|
|Версия Spark|По умолчанию 2.7|Может изменить конфигурацию на 3,5|
|Версия Livy|По умолчанию 2.7|Может изменить конфигурацию на 3,5|
|Jupyter|Ядро PySpark|Ядро PySpark3|

## <a name="safely-install-external-python-packages"></a>Безопасная установка внешних пакетов Python

Кластер HDInsight зависит от встроенной среды Python (как Python 2.7, так и Python 3.5). Установка пользовательских пакетов непосредственно в этих встроенных средах по умолчанию может привести к непредвиденным изменениям версий библиотек и нарушить работу кластера. Чтобы безопасно установить пользовательские внешние пакеты Python для приложений Spark, выполните указанные ниже действия.

1. Создайте виртуальную среду Python с помощью conda. Виртуальная среда обеспечивает изолированное пространство для проектов, которое не нарушает работу других проектов. При создании виртуальной среды Python можно указать нужную версию Python. Создавать виртуальную среду требуется даже в том случае, если вы хотите использовать Python 2.7 и 3.5. Это необходимо для того, чтобы не нарушилась работа среды кластера по умолчанию. Чтобы создать виртуальную среду Python, выполните действия скрипта в кластере для всех узлов, используя приведенный ниже скрипт.

    -   `--prefix` задает путь, где находится виртуальная среда conda. Существует несколько конфигураций, в которые необходимо внести дальнейшие изменения в зависимости от указанного здесь пути. В этом примере используется py35new, так как в кластере уже есть виртуальная среда с именем py35.
    -   `python=` задает версию Python для виртуальной среды. В этом примере используется версия 3.5, то есть та же версия, что и встроенная версия кластера. Для создания виртуальной среды можно также использовать другие версии Python.
    -   `anaconda` задает anaconda в качестве package_spec для установки пакетов Anaconda в виртуальной среде.
    
    ```bash
    sudo /usr/bin/anaconda/bin/conda create --prefix /usr/bin/anaconda/envs/py35new python=3.5 anaconda --yes
    ```

2. При необходимости установите внешние пакеты Python в созданной виртуальной среде. Чтобы установить внешние пакеты Python, выполните действия скрипта в кластере для всех узлов, используя приведенный ниже скрипт. Для записи файлов в папку виртуальной среды необходимо иметь права sudo.

    Полный список доступных пакетов можно найти в [указателе пакетов](https://pypi.python.org/pypi). Его также можно получить из других источников. Например, можно установить пакеты, предоставляемые посредством [conda-forge](https://conda-forge.org/feedstocks/).

    Используйте приведенную ниже команду, если вы хотите установить библиотеку последней версии.

    - Используйте канал conda:

        -   `seaborn` — это имя пакета, который нужно установить.
        -   `-n py35new` — укажите имя только что созданной виртуальной среды. Измените имя в соответствии с созданной виртуальной средой.

        ```bash
        sudo /usr/bin/anaconda/bin/conda install seaborn -n py35new --yes
        ```

    - Либо используйте репозиторий PyPi, изменив `seaborn` и `py35new` соответствующим образом:
        ```bash
        sudo /usr/bin/anaconda/envs/py35new/bin/pip install seaborn
        ```

    Используйте приведенную ниже команду, чтобы установить библиотеку определенной версии.

    - Используйте канал conda:

        -   `numpy=1.16.1` — это имя и версия пакета, который нужно установить.
        -   `-n py35new` — укажите имя только что созданной виртуальной среды. Измените имя в соответствии с созданной виртуальной средой.

        ```bash
        sudo /usr/bin/anaconda/bin/conda install numpy=1.16.1 -n py35new --yes
        ```

    - Либо используйте репозиторий PyPi, изменив `numpy==1.16.1` и `py35new` соответствующим образом:

        ```bash
        sudo /usr/bin/anaconda/envs/py35new/bin/pip install numpy==1.16.1
        ```

    Если вы не знаете имя виртуальной среды, вы можете подключиться по SSH к головному узлу кластера и выполнить команду `/usr/bin/anaconda/bin/conda info -e`, чтобы получить список всех виртуальных сред.

3. Измените конфигурации Spark и Livy и укажите созданную виртуальную среду.

    1. Откройте пользовательский интерфейс Ambari, перейдите на страницу Spark2 и выберите вкладку Configs (Конфигурации).

        :::image type="content" source="./media/apache-spark-python-package-installation/ambari-spark-and-livy-config.png" alt-text="Изменение конфигурации Spark и Livy с помощью Ambari" border="true":::

    2. Разверните Advanced livy2-env и добавьте приведенные ниже инструкции в конце. Если вы установили виртуальную среду с другим префиксом, измените путь соответствующим образом.

        ```bash
        export PYSPARK_PYTHON=/usr/bin/anaconda/envs/py35new/bin/python
        export PYSPARK_DRIVER_PYTHON=/usr/bin/anaconda/envs/py35new/bin/python
        ```

        :::image type="content" source="./media/apache-spark-python-package-installation/ambari-livy-config.png" alt-text="Изменение конфигурации Livy с помощью Ambari" border="true":::

    3. Разверните Advanced spark2-env и замените существующую инструкцию экспорта PYSPARK_PYTHON в нижней части. Если вы установили виртуальную среду с другим префиксом, измените путь соответствующим образом.

        ```bash
        export PYSPARK_PYTHON=${PYSPARK_PYTHON:-/usr/bin/anaconda/envs/py35new/bin/python}
        ```

        :::image type="content" source="./media/apache-spark-python-package-installation/ambari-spark-config.png" alt-text="Изменение конфигурации Spark с помощью Ambari" border="true":::

    4. Сохраните изменения и перезапустите затронутые службы. Эти изменения требуют перезапуска службы Spark2. В пользовательском интерфейсе Ambari появится напоминание о необходимости перезапуска. Нажмите кнопку Restart (Перезапустить), чтобы перезапустить все затронутые службы.

        :::image type="content" source="./media/apache-spark-python-package-installation/ambari-restart-services.png" alt-text="Перезапуск служб" border="true":::

    5. Задайте для сеанса Spark два свойства, чтобы убедиться, что задание указывает на обновленную конфигурацию Spark: `spark.yarn.appMasterEnv.PYSPARK_PYTHON` и `spark.yarn.appMasterEnv.PYSPARK_DRIVER_PYTHON` . 

        Используя терминал или записную книжку, используйте `spark.conf.set` функцию.

        ```spark
        spark.conf.set("spark.yarn.appMasterEnv.PYSPARK_PYTHON", "/usr/bin/anaconda/envs/py35/bin/python")
        spark.conf.set("spark.yarn.appMasterEnv.PYSPARK_DRIVER_PYTHON", "/usr/bin/anaconda/envs/py35/bin/python")
        ```

        Если вы используете Livy, добавьте следующие свойства в текст запроса:

        ```
        “conf” : {
        “spark.yarn.appMasterEnv.PYSPARK_PYTHON”:”/usr/bin/anaconda/envs/py35/bin/python”,
        “spark.yarn.appMasterEnv.PYSPARK_DRIVER_PYTHON”:”/usr/bin/anaconda/envs/py35/bin/python”
        }
        ```

4. Если вы хотите использовать созданную виртуальную среду в Jupyter, измените конфигурации Jupyter и перезапустите Jupyter. Выполните действия скрипта для всех головных узлов, используя приведенную ниже инструкцию для указания новой виртуальной среды в Jupyter. Не забудьте изменить в пути префикс, заданный для виртуальной среды. После выполнения этого действия скрипта перезапустите службу Jupyter в пользовательском интерфейсе Ambari, чтобы это изменение стало доступно.

    ```bash
    sudo sed -i '/python3_executable_path/c\ \"python3_executable_path\" : \"/usr/bin/anaconda/envs/py35new/bin/python3\"' /home/spark/.sparkmagic/config.json
    ```

    Можно еще раз подтвердить среду Python в Jupyter Notebook, выполнив приведенный ниже код.

    :::image type="content" source="./media/apache-spark-python-package-installation/check-python-version-in-jupyter.png" alt-text="Проверка версии Python в Jupyter Notebook" border="true":::

## <a name="known-issue"></a>Известная проблема

Существует известная ошибка в Anaconda версий `4.7.11`, `4.7.12` и `4.8.0`. Если вы видите, что действия сценария перестают отвечать на запросы `"Collecting package metadata (repodata.json): ...working..."` и завершаются с ошибкой `"Python script has been killed due to timeout after waiting 3600 secs"` . можно скачать [этот скрипт](https://gregorysfixes.blob.core.windows.net/public/fix-conda.sh) и выполнить его как действия скрипта во всех узлах, чтобы устранить проблему.

Чтобы проверить версию Anaconda, можно подключиться по SSH к головному узлу кластера и выполнить команду `/usr/bin/anaconda/bin/conda --v`.

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор: Spark в Azure HDInsight](apache-spark-overview.md)
* [Внешние пакеты с записными книжками Jupyter в Apache Spark](apache-spark-jupyter-notebook-use-external-packages.md)
* [Отслеживание и отладка заданий в кластере Apache Spark в HDInsight на платформе Linux](apache-spark-job-debugging.md)
