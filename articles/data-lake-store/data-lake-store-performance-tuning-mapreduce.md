---
title: Настройка производительности Azure Data Lake Storage 1-го поколения — MapReduce
description: Сведения о настройке производительности MapReduce в Azure Data Lake Storage 1-го поколения, включая параметры, рекомендации, пример вычисления и ограничения.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 12/19/2016
ms.author: twooley
ms.openlocfilehash: 2549413241e422fb1e0e5e1f079c287e0b7cf005
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97723734"
---
# <a name="performance-tuning-guidance-for-mapreduce-on-hdinsight-and-azure-data-lake-storage-gen1"></a>Рекомендации по настройке производительности для MapReduce в HDInsight и Azure Data Lake Storage 1-го поколения

## <a name="prerequisites"></a>Предварительные условия

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Учетная запись Azure Data Lake Storage 1-го поколения**. Инструкции по ее созданию см. в статье [Приступая к работе с Azure Data Lake Storage 1-го поколения](data-lake-store-get-started-portal.md)
* **Кластер Azure HDInsight** с доступом к учетной записи Data Lake Storage 1-го поколения. Дополнительные сведения см. в статье [Создание кластеров HDInsight, использующих Data Lake Store, с помощью портала Azure](data-lake-store-hdinsight-hadoop-use-portal.md). Убедитесь, что вы включили удаленный рабочий стол для кластера.
* **Использование MapReduce в HDInsight**. См. дополнительные сведения об [использовании MapReduce в Hadoop и HDInsight](../hdinsight/hadoop/hdinsight-use-mapreduce.md)
* **Ознакомьтесь с рекомендациями по настройке производительности для Data Lake Storage 1-го поколения**. Общие понятия производительности см. в разделе [Data Lake Storage 1-го поколения рекомендации по настройке производительности](./data-lake-store-performance-tuning-guidance.md) .

## <a name="parameters"></a>Параметры

Ниже перечислены наиболее важные параметры, которые можно настроить, чтобы оптимизировать производительность Data Lake Storage 1-го поколения при выполнении заданий MapReduce.

|Параметр      | Описание  |
|---------|---------|
|`Mapreduce.map.memory.mb`  |  Объем памяти, выделяемой для каждого модуля сопоставления.  |
|`Mapreduce.job.maps`     |  Число задач Map на задание.  |
|`Mapreduce.reduce.memory.mb`     |  Объем памяти, выделяемой для каждого объекта reduce.  |
|`Mapreduce.job.reduces`    |   Число задач уменьшения на задание.  |

### <a name="mapreducemapmemory--mapreducereducememory"></a>MapReduce. Map. Memory/MapReduce. reduce. Memory

Измените это число в зависимости от объема памяти, необходимой для задачи Map или reduce. Вы можете просмотреть значения по умолчанию `mapreduce.map.memory` и `mapreduce.reduce.memory` в Ambari с помощью конфигурации Yarn. В Ambari перейдите по адресу YARN и просмотрите вкладку **configs (конфигурации** ). Будет отображена память YARN.

### <a name="mapreducejobmaps--mapreducejobreduces"></a>MapReduce. job. Maps/MapReduce. job. reduces

Определяет максимальное количество модулей сопоставления или модулей сжатия для создания. Число разбиений определяет, сколько модулей сопоставления создается для задания MapReduce. Таким образом, вы можете получить меньшее число модулей сопоставления, чем было запрошено, если количество разделений меньше числа запрошенных модулей сопоставления.

## <a name="guidance"></a>Руководство

### <a name="step-1-determine-number-of-jobs-running"></a>Шаг 1. Определение числа выполняемых заданий

По умолчанию MapReduce будет использовать весь кластер для обработки задания. Вы можете использовать меньше кластеров, используя меньшее число модулей сопоставления, чем есть доступные контейнеры. В этой статье предполагается, что ваше приложение — единственное, которое выполняется в кластере.

### <a name="step-2-set-mapreducemapmemorymapreducereducememory"></a>Шаг 2. Задание MapReduce. Map. Memory/MapReduce. reduce. Memory

Объем памяти для задач сопоставления и уменьшения будет зависеть от конкретного задания. Можно уменьшить объем памяти, если нужно повысить параллелизм. Число одновременно выполняемых задач зависит от числа контейнеров. Сокращая объем памяти для каждого модуля сопоставления или уменьшения, можно создавать дополнительные контейнеры, обеспечивая одновременный запуск большего числа модулей сопоставления или уменьшения. Слишком сильное снижение объема памяти может вызвать нехватку памяти при выполнении некоторых процессов. Если при выполнении задания возникнет ошибка кучи, увеличьте объем памяти для каждого модуля сопоставления или уменьшения. Учтите, что добавление дополнительных контейнеров увеличивает дополнительную нагрузку на каждый дополнительный контейнер, что может привести к снижению производительности. Альтернатива — обеспечить больший объем памяти, используя кластер с большим объемом памяти или увеличивая число узлов в кластере. Больший объем памяти позволит использовать дополнительные контейнеры, обеспечивая возможность параллельной обработки.

### <a name="step-3-determine-total-yarn-memory"></a>Шаг 3. Определение общего объема памяти YARN

Чтобы настроить MapReduce. job. Maps/MapReduce. job. reduces, учитывайте объем общей YARN памяти, доступной для использования. Эта информация доступна в Ambari. Перейдите по адресу YARN и просмотрите вкладку **configs (конфигурации** ). В этом окне отображается память YARN. Умножьте объем памяти YARN на количество узлов в кластере, чтобы получить общий объем памяти YARN.

`Total YARN memory = nodes * YARN memory per node`

Если вы используете пустой кластер, то память может быть общей YARN памятью для кластера. Если другие приложения используют память, вы можете ограничить выделение памяти кластера, сократив число модулей сопоставления или уменьшения в соответствии с числом контейнеров, которые вы хотите использовать.

### <a name="step-4-calculate-number-of-yarn-containers"></a>Шаг 4. Вычисление числа контейнеров YARN

Контейнеры YARN определяют уровень параллелизма для задания. Разделите общий объем памяти YARN на значение mapreduce.map.memory.

`# of YARN containers = total YARN memory / mapreduce.map.memory`

### <a name="step-5-set-mapreducejobmapsmapreducejobreduces"></a>Шаг 5. Установка mapreduce.job.maps/mapreduce.job.reduces

Установите для mapreduce.job.maps/mapreduce.job.reduces по крайней мере число доступных контейнеров. Вы можете поэкспериментировать, увеличивая число модулей сопоставления и уменьшения, чтобы понять, помогает ли это оптимизировать производительность. Имейте в виду, что большое число модулей сопоставления может увеличить нагрузку, что, в свою очередь, может привести к снижению производительности.

Возможности изоляции и планирования использования ресурсов ЦП отключены по умолчанию, следовательно, число контейнеров YARN ограничено объемом памяти.

## <a name="example-calculation"></a>Пример вычисления

Предположим, у вас есть кластер, который состоит из 8 узлов D14 и вы хотите запустить задание с большим количеством операций ввода-вывода. Ниже представлены примеры вычислений, которые нужно выполнить.

### <a name="step-1-determine-number-of-jobs-running"></a>Шаг 1. Определение числа выполняемых заданий

В нашем примере предполагается, что задание является единственным работающим.

### <a name="step-2-set-mapreducemapmemorymapreducereducememory"></a>Шаг 2. Задание MapReduce. Map. Memory/MapReduce. reduce. Memory

В нашем примере вы запускаете задание с интенсивным вводом-выводом и решаете, что достаточно 3 ГБ памяти для задач Map.

`mapreduce.map.memory = 3GB`

### <a name="step-3-determine-total-yarn-memory"></a>Шаг 3. Определение общего объема памяти YARN

`total memory from the cluster is 8 nodes * 96GB of YARN memory for a D14 = 768GB`

### <a name="step-4-calculate--of-yarn-containers"></a>Шаг 4. Расчет числа контейнеров YARN

`# of YARN containers = 768 GB of available memory / 3 GB of memory = 256`

### <a name="step-5-set-mapreducejobmapsmapreducejobreduces"></a>Шаг 5. Установка mapreduce.job.maps/mapreduce.job.reduces

`mapreduce.map.jobs = 256`

## <a name="limitations"></a>Ограничения

**Регулирование Data Lake Storage 1-го поколения**

Как мультитенантная служба, Data Lake Storage 1-го поколения определяет ограничения пропускной способности на уровне учетной записи. При превышении этих ограничений происходит сбой задач. Это можно определить, отслеживая ошибки регулирования в журналах задач. Если для обработки задания требуется большая пропускная способность, свяжитесь с нами.

Чтобы проверить, применяется ли для вас регулирование, включите ведение журнала отладки на стороне клиента. Вот как это сделать.

1. Включите следующее свойство в библиотеку Log4j в Ambari, последовательно выбрав YARN > Config > Advanced yarn-log4j: log4j.logger.com.microsoft.azure.datalake.store=DEBUG.

2. Перезапустите все узлы и службы, чтобы изменения конфигурации вступили в силу.

3. В случае регулирования вы увидите код ошибки HTTP 429 в файле журнала YARN. Файл журнала YARN расположен здесь: /tmp/&lt;пользователь&gt;/yarn.log

## <a name="examples-to-run"></a>Примеры выполнения кода

Чтобы продемонстрировать выполнение MapReduce на Data Lake Storage 1-го поколения, ниже приведен пример кода, который был запущен в кластере со следующими параметрами:

* 16 узлов D14v2;
* кластер Hadoop под управлением HDI 3.6.

Для начала ознакомьтесь с примерами команд для запуска MapReduce Teragen, Terasort и Teravalidate. Эти команды можно настроить в соответствии с имеющимися ресурсами.

### <a name="teragen"></a>Teragen

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teragen -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 10000000000 adl://example/data/1TB-sort-input
```

### <a name="terasort"></a>Terasort

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar terasort -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 -Dmapreduce.job.reduces=512 -Dmapreduce.reduce.memory.mb=3072 adl://example/data/1TB-sort-input adl://example/data/1TB-sort-output
```

### <a name="teravalidate"></a>Teravalidate

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teravalidate -Dmapreduce.job.maps=512 -Dmapreduce.map.memory.mb=3072 adl://example/data/1TB-sort-output adl://example/data/1TB-sort-validate
```