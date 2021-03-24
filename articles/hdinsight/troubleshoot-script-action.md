---
title: Устранение неполадок в действиях скриптов в Azure HDInsight
description: Общие шаги по устранению неполадок для действий скриптов в Azure HDInsight.
ms.service: hdinsight
ms.topic: troubleshooting
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: 73b958964db2d0b308dd6dfc34024d61ce5ad8af
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104871441"
---
# <a name="troubleshoot-script-actions-in-azure-hdinsight"></a>Устранение неполадок в действиях скриптов в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="viewing-logs"></a>Просмотр журналов

Вы можете использовать веб-интерфейс Apache Ambari для просмотра информации, регистрируемой действиями сценария. Если во время создания кластера произошел сбой сценария, журналы находятся в учетной записи хранения кластера по умолчанию. Этот раздел содержит сведения о том, как получить журналы обоими этими способами.

### <a name="apache-ambari-web-ui"></a>Веб-интерфейс Apache Ambari

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net`, где `CLUSTERNAME` — это имя вашего кластера.

1. В панели вверху страницы выберите запись **ops**. Появится список текущих и предыдущих операций, выполняемых в кластере с помощью Ambari.

    :::image type="content" source="./media/troubleshoot-script-action/hdi-apache-ambari-nav.png" alt-text="Веб-панель Ambari с выбранной записью ops" border="true":::

1. Найдите записи, для которых в столбце **Операции** указано **run\_customscriptaction**. Такие записи создаются при выполнении действий сценариев.

    :::image type="content" source="./media/troubleshoot-script-action/ambari-script-action.png" alt-text="Операции с действиями сценария Apache Ambari" border="true":::

    Для просмотра выходных данных **STDOUT** и **STDERR** выберите запись **run\customscriptaction** и перейдите по ссылкам. Эти выходные данные формируются при запуске сценария и могут содержать полезные сведения.

### <a name="default-storage-account"></a>Учетная запись хранения по умолчанию

Если из-за ошибки сценария при создании кластера происходит сбой, журналы сохраняются в учетной записи хранения кластера.

* Журналы хранилища находятся в `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\CLUSTER_NAME\DATE`.

    :::image type="content" source="./media/troubleshoot-script-action/script-action-logs-in-storage.png" alt-text="Журналы действий скриптов" border="true":::

    В этом каталоге журналы упорядочены по **головному узлу**, **рабочему узлу** и **узлу zookeeper**. См. следующие примеры.

    * **Головного узла**: `<ACTIVE-HEADNODE-NAME>.cloudapp.net`

    * **Рабочий узел**: `<ACTIVE-WORKERNODE-NAME>.cloudapp.net`

    * **Zookeeper узел**: `<ACTIVE-ZOOKEEPERNODE-NAME>.cloudapp.net`

* Все **stdout** и **stderr** соответствующего узла передаются в учетную запись хранения. Для каждого действия сценария создается файл **output-\*.txt** и файл **errors-\*.txt**. Файл **output-*.txt** содержит сведения об URI сценария, запущенном на узле. Ниже приведен пример таких сведений:

    ```output
    'Start downloading script locally: ', u'https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh'
    ```

* Возможно, повторно создан кластер действия сценария с тем же именем. В этом случае соответствующие журналы можно отличить по имени папки **DATE**. Например, структура папок для кластера **mycluster**, созданного в другие дни, будет похожа на следующие записи журнала:

    `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-04` `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-05`

* При создании кластера действие сценария с таким же именем в тот же день можно использовать уникальный префикс для идентификации соответствующих файлов журнала.

* При создании кластера около полуночи (00:00) файлы журналов могут включать сведения за два дня. В таких случаях для одного кластера будут отображаться две разные папки даты.

* Передача файлов журнала в контейнер по умолчанию может занять до пяти минут, особенно для больших кластеров. Таким образом, если вы хотите получить доступ к журналам, не следует немедленно удалять кластер при сбое действия сценария.

## <a name="ambari-watchdog"></a>Модуль наблюдения Ambari

Не изменяйте пароль модуля наблюдения Ambari (hdinsightwatchdog) в кластере HDInsight под управлением Linux. Изменение пароля нарушает возможность выполнения новых действий сценария в кластере HDInsight.

## <a name="cant-import-name-blobservice"></a>Не удается импортировать BlobService имени

__Симптомы__. Сбой действия скрипта. При просмотре операции в Ambari выводится следующее сообщение об ошибке:

```
Traceback (most recent call list):
  File "/var/lib/ambari-agent/cache/custom_actions/scripts/run_customscriptaction.py", line 21, in <module>
    from azure.storage.blob import BlobService
ImportError: cannot import name BlobService
```

__Причина__. Эта ошибка возникает при обновлении клиента Python службы хранилища Azure, который входит в состав кластера HDInsight. HDInsight требуется клиент службы хранилища Azure версии 0.20.0.

__Решение__. Чтобы устранить эту ошибку, вручную подключитесь к каждому узлу кластера с помощью `ssh`. Выполните следующую команду, чтобы повторно установить требуемую версию клиента хранилища:

```bash
sudo pip install azure-storage==0.20.0
```

Сведения о подключении к кластеру по протоколу SSH см. в статье [Подключение к HDInsight (Apache Hadoop) с помощью SSH](hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="history-doesnt-show-the-scripts-used-during-cluster-creation"></a>Журнал не отображает сценарии, используемые во время создания кластера

Если кластер создан до 15 марта 2016 г., в журнале действий сценариев могут отсутствовать записи. При изменении размера кластера сценарии фиксируются в журнале действий сценариев.

Есть два исключения.

* Кластер создан до 1 сентября 2015 г. Это дата добавления действий сценариев. Если кластер был создан раньше, действия сценариев не могли использоваться при его создании.

* При создании кластера использовано несколько действий сценариев одновременно. Вы использовали или одно и то же имя для разных сценариев, или одно и то же имя и URI, но разные параметры для разных сценариев. В таких случаях происходит следующая ошибка:

    ```
    No new script actions can be run on this cluster because of conflicting script names in existing scripts. Script names provided at cluster creation must be all unique. Existing scripts are run on resize.
    ```

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../includes/hdinsight-troubleshooting-next-steps.md)]