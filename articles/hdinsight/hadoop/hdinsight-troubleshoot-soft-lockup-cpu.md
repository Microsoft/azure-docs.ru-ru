---
title: Ошибка устройства наблюдения "Мягкая блокировка ЦП" из кластера Azure HDInsight
description: В журналах ядра в кластере Azure HDInsight отображаются счетчики производительности "Мягкая блокировка" для ошибок наблюдения
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/05/2019
ms.openlocfilehash: 5d9d7b0fc21660dd22ff92bbe2de38c759c440ec
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944342"
---
# <a name="scenario-watchdog-bug-soft-lockup---cpu-error-from-an-azure-hdinsight-cluster"></a>Сценарий: ошибка "устройство наблюдения: ошибка: кратковременная блокировка — ЦП" из кластера Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

В системных журналах ядра содержится сообщение об ошибке: `watchdog: BUG: soft lockup - CPU` .

## <a name="cause"></a>Причина

[Ошибка](https://bugzilla.kernel.org/show_bug.cgi?id=199437) в ядре Linux вызывает мягкую зависание ЦП.

## <a name="resolution"></a>Решение

Применить исправление ядра. Приведенный ниже сценарий обновляет ядро Linux и перезагружает компьютеры в разное время более 24 часов. Выполните действие сценария в двух пакетах. Первый пакет находится на всех узлах, кроме головного узла. Второй пакет находится на головном узле. Не запускайте на головном узле и других узлах одновременно.

1. Перейдите к кластеру HDInsight с портал Azure.

1. Перейдите к разделу действия скрипта.

1. Выберите **отправить новый** и введите входные данные, как показано ниже.

    | Свойство. | Значение |
    | --- | --- |
    | Тип скрипта | — Пользовательский |
    | Имя |Исправление проблемы с мягкой блокировкой ядра |
    | URI bash-скрипта |`https://raw.githubusercontent.com/hdinsight/hdinsight.github.io/master/ClusterCRUD/KernelSoftLockFix/scripts/KernelSoftLockIssue_FixAndReboot.sh` |
    | Типы узлов |Worker, Zookeeper |
    | Параметры |Н/Д |

    Выберите **сохранить это действие скрипта...** , если требуется выполнять скрипт при добавлении новых узлов.

1. Нажмите кнопку **создания**.

1. Дождитесь завершения выполнения.

1. Выполните действие скрипта на головном узле, выполнив те же действия, что и на шаге 3, но на этот раз с типами узлов: Head.

1. Дождитесь завершения выполнения.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]