---
title: Jupyter 404 ошибка-"Блокировка кросс-источника API" — Azure HDInsight
description: Ошибка Jupyter Server 404 "не найдена" из-за блокировки межисточникового API в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/29/2019
ms.openlocfilehash: 27cd3aff859fd46679679ac12d3acc03fa6da158
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929446"
---
# <a name="scenario-jupyter-server-404-not-found-error-due-to-blocking-cross-origin-api-in-azure-hdinsight"></a>Сценарий: Jupyter Server 404 "не найдено" из-за "блокировки межисточникового API" в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы исправления проблем, возникающих при использовании компонентов Apache Spark в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

При доступе к службе Jupyter в HDInsight отображается окно ошибки с сообщением "не найдено". Если проверить журналы Jupyter, вы увидите нечто вроде:

```log
[W 2018-08-21 17:43:33.352 NotebookApp] 404 PUT /api/contents/PySpark/notebook.ipynb (10.16.0.144) 4504.03ms referer=https://pnhr01hdi-corpdir.msappproxy.net/jupyter/notebooks/PySpark/notebook.ipynb
Blocking Cross Origin API request.  
Origin: https://xxx.xxx.xxx, Host: pnhr01.j101qxjrl4zebmhb0vmhg044xe.ax.internal.cloudapp.net:8001
```

Вы также можете увидеть IP-адрес в поле "источник" в журнале Jupyter.

## <a name="cause"></a>Причина

Эта ошибка может быть вызвана несколькими причинами:

- Если вы настроили правила группы безопасности сети (NSG), чтобы ограничивать доступ к кластеру. При ограничении доступа с помощью правил NSG вы сможете получить прямой доступ к Apache Ambari и другим службам, используя IP-адрес, а не имя кластера. Однако при доступе к Jupyter может появиться ошибка 404 "не найдено".

- Если вы задаете шлюзу HDInsight настраиваемое DNS-имя, отличное от стандартного `xxx.azurehdinsight.net` .

## <a name="resolution"></a>Решение

1. Измените файлы jupyter.py в следующих двух местах:

    ```bash
    /var/lib/ambari-server/resources/common-services/JUPYTER/1.0.0/package/scripts/jupyter.py
    /var/lib/ambari-agent/cache/common-services/JUPYTER/1.0.0/package/scripts/jupyter.py
    ```

1. Найдите строку, в которой написано: `NotebookApp.allow_origin='\"https://{2}.{3}\"'` и измените ее на: `NotebookApp.allow_origin='\"*\"'` .

1. Перезапустите службу Jupyter из Ambari.

1. При вводе `ps aux | grep jupyter` в командной строке должна отобразиться возможность подключения к любому URL-адресу.

Это менее безопасно, чем уже установленный параметр. Но предполагается, что доступ к кластеру ограничен, и что у него из внешней сети разрешено подключаться к кластеру, так как мы NSG на месте.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]