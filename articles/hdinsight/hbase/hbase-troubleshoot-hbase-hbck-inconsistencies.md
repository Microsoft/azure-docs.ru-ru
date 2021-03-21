---
title: hbck HBase возвращает несоответствия в Azure HDInsight
description: hbck HBase возвращает несоответствия в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/08/2019
ms.openlocfilehash: cbe4231bbecdf279c637cd334336437a020188d4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98936992"
---
# <a name="scenario-hbase-hbck-command-returns-inconsistencies-in-azure-hdinsight"></a>Сценарий: `hbase hbck` команда возвращает несоответствия в Azure HDInsight.

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue-region-is-not-in-hbasemeta"></a>Вопрос. регион не находится в `hbase:meta`

Регион XXX в HDFS, но он отсутствует в списке `hbase:meta` или развернут на любом сервере регионов.

### <a name="cause"></a>Причина

Возможны разные варианты.

### <a name="resolution"></a>Решение

1. Исправьте таблицу meta, выполнив команду:

    ```
    hbase hbck -ignorePreCheckPermission –fixMeta
    ```

1. Назначьте регионы для Регионсерверс, выполнив:

    ```
    hbase hbck -ignorePreCheckPermission –fixAssignment
    ```
---

## <a name="issue-region-is-offline"></a>Ошибка: регион находится в автономном режиме

Регион XXX не развернут ни на одном Регионсервер. Это означает, что регион находится в `hbase:meta` , но вне сети.

### <a name="cause"></a>Причина

Возможны разные варианты.

### <a name="resolution"></a>Решение

Подключите регионы по сети, выполнив:

```
hbase hbck -ignorePreCheckPermission –fixAssignment
```

---

## <a name="issue-regions-have-the-same-startend-keys"></a>Вопрос. регионы имеют одинаковые ключи начала и окончания

### <a name="cause"></a>Причина

Возможны разные варианты.

### <a name="resolution"></a>Решение

Вручную объединить эти регионы. Перейдите к разделу "Таблица пользовательского веб-интерфейса HBase HMaster" и выберите ссылку на таблицу, в которой возникла эта ошибка. Вы увидите начальный ключ или конечный ключ каждого региона, принадлежащего этой таблице. Затем объедините эти регионы. В оболочке HBase выполните `merge_region 'xxxxxxxx','yyyyyyy', true` . Пример:

```
RegionA, startkey:001, endkey:010,

RegionB, startkey:001, endkey:080,

RegionC, startkey:010, endkey:080.
```

В этом сценарии необходимо объединить Region и Регионк и получить область с тем же диапазоном ключей, что и Регионб, а затем объединить Регионб и регион. XXXXXXX и ииииии — это хэш-строка в конце каждого имени региона. Будьте внимательны, чтобы не выполнять слияние двух несмежных регионов. После каждого слияния, например слияния A и C, HBase запустит сжатие по регионам. Дождитесь завершения сжатия, прежде чем выполнять другое слияние с областью. Сведения о состоянии сжатия можно найти на странице сервера этого региона в пользовательском интерфейсе HBase HMaster.

---

## <a name="issue-cant-load-regioninfo"></a>Ошибка: не удается загрузить `.regioninfo`

Не удается загрузить `.regioninfo` для региона `/hbase/data/default/tablex/regiony` .

### <a name="cause"></a>Причина

Скорее всего, это происходит из-за частичного удаления области при Регионсервер сбоях или перезагрузке виртуальных машин. В настоящее время хранилище Azure — это плоская файловая система больших двоичных объектов, а некоторые операции с файлами не являются атомарными.

### <a name="resolution"></a>Решение

Вручную очистите оставшиеся файлы и папки:

1. Выполните `hdfs dfs -ls /hbase/data/default/tablex/regiony` , чтобы проверить, какие папки и файлы все еще находятся в нем.

1. Выполните `hdfs dfs -rmr /hbase/data/default/tablex/regiony/filez` , чтобы удалить все дочерние файлы и папки

1. Выполните процедуру `hdfs dfs -rmr /hbase/data/default/tablex/regiony` , чтобы удалить папку Region.

---

## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите своего варианта проблемы или вам не удается ее устранить, дополнительные сведения можно получить, посетив один из следующих каналов.

* Получите ответы специалистов Azure на [сайте поддержки сообщества пользователей Azure](https://azure.microsoft.com/support/community/).

* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Вступайте в сообщество Azure для получения нужных ресурсов: ответов, поддержки и советов экспертов.

* Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки на [портале Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Выберите **Поддержка** в строке меню или откройте центр **Справка и поддержка**. Дополнительные сведения см. в статье [Создание запроса на поддержку Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Доступ к управлению подписками и поддержкой выставления счетов уже включен в вашу подписку Microsoft Azure, а техническая поддержка предоставляется в рамках одного из [планов Службы поддержки Azure](https://azure.microsoft.com/support/plans/).