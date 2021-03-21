---
title: Не удалось создать Jupyter Notebook в Azure HDInsight
description: Сведения о действиях по устранению неполадок и возможных способах устранения проблем при взаимодействии с кластерами Azure HDInsight.
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 02/11/2020
ms.openlocfilehash: d5f6612e22522452efb5eecd14c6c825bf29e6d5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98938669"
---
# <a name="unable-to-create-jupyter-notebook-in-azure-hdinsight"></a>Не удалось создать Jupyter Notebook в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

При запуске Jupyter Notebook появляется сообщение об ошибке, содержащее следующее:

```error
Cannot convert notebook to v5 because that version doesn't exist
```

## <a name="cause"></a>Причина

Несовпадение версий.

## <a name="resolution"></a>Решение

1. С помощью команды [ssh command](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Откройте `_version.py` , выполнив следующую команду:

    ```bash
    sudo nano /usr/bin/anaconda/lib/python2.7/site-packages/nbformat/_version.py
    ```

1. Измените значение **5** на **4** , чтобы измененная строка выявлялась следующим образом:

    ```python
    version_info = (4, 0, 3)
    ```

    Сохраните изменения, введя **CTRL + X**, **Y**, **введите**.

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net/#/main/services/JUPYTER`, где `CLUSTERNAME` — это имя вашего кластера.

1. Перезапустите службу Jupyter.

## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите своего варианта проблемы или вам не удается ее устранить, дополнительные сведения можно получить, посетив один из следующих каналов.

* Получите ответы специалистов Azure на [сайте поддержки сообщества пользователей Azure](https://azure.microsoft.com/support/community/).

* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Вступайте в сообщество Azure для получения нужных ресурсов: ответов, поддержки и советов экспертов.

* Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки на [портале Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Выберите **Поддержка** в строке меню или откройте центр **Справка и поддержка**. Дополнительные сведения см. в статье [Создание запроса на поддержку Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Доступ к управлению подписками и поддержкой выставления счетов уже включен в вашу подписку Microsoft Azure, а техническая поддержка предоставляется в рамках одного из [планов Службы поддержки Azure](https://azure.microsoft.com/support/plans/).
