---
title: Оптимизация Apache HBase с Apache Ambari в Azure HDInsight
description: Используйте веб-интерфейс Apache Ambari для настройки и оптимизации Apache HBase.
ms.service: hdinsight
ms.topic: how-to
ms.date: 02/01/2021
ms.openlocfilehash: 7e54b1347e4c67b99ba87b15c2c15d9d28244ce7
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864777"
---
# <a name="optimize-apache-hbase-with-apache-ambari-in-azure-hdinsight"></a>Оптимизация Apache HBase с Apache Ambari в Azure HDInsight

Apache Ambari — это веб-интерфейс для управления кластерами HDInsight и их мониторинга. Общие сведения о пользовательском веб-интерфейсе Ambari см. [в статье Управление кластерами HDInsight с помощью веб-интерфейса Apache Ambari](hdinsight-hadoop-manage-ambari.md).

Конфигурация Apache HBase изменяется на вкладке **конфигурации HBase** . В следующих разделах описаны некоторые важные параметры конфигурации, влияющие на производительность HBase.

## <a name="set-hbase_heapsize"></a>Настройка параметра HBASE_HEAPSIZE

> [!NOTE]
> Эта статья содержит ссылки на *образец* термина — термин, который корпорация Майкрософт больше не использует. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.

Размер кучи HBase определяет максимальный объем кучи в мегабайтах для сервера *региона* и *главного* сервера. Значение по умолчанию — 1000 МБ. Это значение должно быть настроено для рабочей нагрузки кластера.

1. Чтобы изменить этот параметр, перейдите в область **Advanced HBase-env** (Дополнительные параметры среды HBase) на вкладке **Configs** (Конфигурации) HBase, а затем найдите параметр `HBASE_HEAPSIZE`.

1. Измените значение по умолчанию на 5000 МБ.

    :::image type="content" source="./media/optimize-hbase-ambari/ambari-hbase-heapsize.png" alt-text="&quot;Apache Ambari HBase Memory хеапсизе&quot;" border="true":::

## <a name="optimize-read-heavy-workloads"></a>Оптимизация рабочих нагрузок с интенсивными операциями чтения

Приведенные ниже настройки важны для повышения производительности рабочих нагрузок с интенсивными операциями чтения.

### <a name="block-cache-size"></a>Размер кэша блоков

Кэш блоков является кэшем чтения. Его размер задает параметр `hfile.block.cache.size`. Его значение по умолчанию — 0,4. Это означает 40 % от общего объема памяти сервера региона. Чем больше размер кэша блоков, тем быстрее будут выполняться операции произвольного чтения.

1. Чтобы изменить этот параметр, перейдите на вкладку **Settings** (Параметры) на вкладке **Configs** (Конфигурации) HBase, а затем найдите параметр **% of RegionServer Allocated to Read Buffers** (% памяти сервера региона, выделяемой для буферов чтения).

    :::image type="content" source="./media/optimize-hbase-ambari/hbase-block-cache-size.png" alt-text="Размер кэша блоков памяти Apache HBase" border="true":::

1. Чтобы изменить значение, щелкните значок **Edit** (Изменить).

### <a name="memstore-size"></a>Размер Memstore

Все изменения сохраняются в буфере памяти, который называется *Memstore*. Этот буфер увеличивает общий объем данных, которые могут быть записаны на диск за одну операцию. Он также ускоряет доступ к последним изменениям. Размер Memstore определяют следующие два параметра:

* `hbase.regionserver.global.memstore.UpperLimit`: определяет максимальный процент памяти сервера региона, доступной для объединенного буфера Memstore;

* `hbase.regionserver.global.memstore.LowerLimit`: определяет минимальный процент памяти сервера региона, доступной для объединенного буфера Memstore.

Чтобы оптимизировать операции произвольного чтения, можно уменьшить максимальный и минимальный размеры Memstore.

### <a name="number-of-rows-fetched-when-scanning-from-disk"></a>Число получаемых строк при проверке данных на диске

Параметр `hbase.client.scanner.caching` определяет количество строк, считываемых с диска, когда для сканера вызывается метод `next`.  Значение по умолчанию — 100. Чем больше значение этого параметра, тем меньше удаленных вызовов выполняется от клиента к серверу региона, что позволяет ускорить проверку. Однако этот параметр также приведет к увеличению нехватки памяти на клиенте.

:::image type="content" source="./media/optimize-hbase-ambari/hbase-num-rows-fetched.png" alt-text="Выборка количества строк Apache HBase" border="true":::

> [!IMPORTANT]  
> Не задавайте значение таким образом, чтобы время между вызовом следующего метода для сканера превышало время ожидания сканера. Время ожидания сканера определяет свойство `hbase.regionserver.lease.period`.

## <a name="optimize-write-heavy-workloads"></a>Оптимизация рабочих нагрузок с интенсивными операциями записи

Приведенные ниже настройки важны для повышения производительности рабочих нагрузок с интенсивными операциями записи.

### <a name="maximum-region-file-size"></a>Максимальный размер файла для региона

HBase хранит данные во внутреннем формате файла, *HFile*. Свойство `hbase.hregion.max.filesize` определяет размер одного файла HFile для региона.  Регион разделяется на два региона, если сумма размеров всех файлов HFile в регионе превышает значение этого параметра.

:::image type="content" source="./media/optimize-hbase-ambari/hbase-hregion-max-filesize.png" alt-text="&quot;Apache HBase Хрегион Max размер файла&quot;" border="true":::

Чем больше размер файла для региона, тем меньше число разделений. Можно увеличить размер файла, чтобы определить значение, обеспечивающее максимальную производительность записи.

### <a name="avoid-update-blocking"></a>Избегание блокировки обновлений

* Свойство `hbase.hregion.memstore.flush.size` определяет размер, по достижении которого содержимое Memstore освобождается на диск. Его значение по умолчанию составляет 128 МБ.

* Множитель блока HBase региона определяется `hbase.hregion.memstore.block.multiplier` . Значение по умолчанию — 4. Максимально допустимое значение равно 8.

* HBase блокирует обновления, если Memstore содержит (`hbase.hregion.memstore.flush.size` * `hbase.hregion.memstore.block.multiplier`) байтов.

    При значениях размера освобождения и множителя блокирования по умолчанию обновления блокируются, когда размер Memstore составляет 128 * 4 = 512 МБ. Чтобы уменьшить количество блокируемых обновлений, увеличьте значение `hbase.hregion.memstore.block.multiplier`.

:::image type="content" source="./media/optimize-hbase-ambari/hbase-hregion-memstore-block-multiplier.png" alt-text="Множитель блока для Apache HBase" border="true":::

## <a name="define-memstore-size"></a>Определение размера Memstore

Размер Memstore определяют параметры `hbase.regionserver.global.memstore.upperLimit` и `hbase.regionserver.global.memstore.lowerLimit`. Если задать для них одинаковое значение, это уменьшит паузы между операциям записи (а также приведет к более частому сбросу данных на диск) и повысит производительность записи.

## <a name="set-memstore-local-allocation-buffer"></a>Настройка локального буфера выделения Memstore

Использование локального буфера выделения Memstore определяет свойство `hbase.hregion.memstore.mslab.enabled`. Если этот параметр включен (true), то во время интенсивной операции записи предотвращается Фрагментация кучи. Значение по умолчанию — true.

:::image type="content" source="./media/optimize-hbase-ambari/hbase-hregion-memstore-mslab-enabled.png" alt-text="hbase.hregion.memstore.mslab.enabled" border="true":::

## <a name="next-steps"></a>Дальнейшие действия

* [Управление кластерами HDInsight с помощью веб-интерфейса Ambari](hdinsight-hadoop-manage-ambari.md)
* [Управление кластерами HDInsight с помощью REST API Ambari](hdinsight-hadoop-manage-ambari-rest-api.md)
* [Оптимизация кластеров](./hdinsight-changing-configs-via-ambari.md)
* [Оптимизация Apache Hive](./optimize-hive-ambari.md)
* [Optimize Apache Pig](./optimize-pig-ambari.md)
