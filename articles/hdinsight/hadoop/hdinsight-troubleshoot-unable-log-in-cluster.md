---
title: Не удалось войти в кластер Azure HDInsight.
description: Устранение неполадок, почему не удается войти в кластер Apache Hadoop в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/31/2019
ms.openlocfilehash: 03657606f7e24436ff5a851fe7eef1e0c4d5dc24
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944304"
---
# <a name="scenario-unable-to-log-into-azure-hdinsight-cluster"></a>Сценарий: не удается войти в кластер Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

Не удалось войти в кластер Azure HDInsight.

## <a name="cause"></a>Причина

Причины могут отличаться. Помните, что при входе на панели мониторинга кластера или приложения используйте учетные данные для входа в кластер или HTTP. При удаленном подключении используйте учетные данные SSH или удаленного рабочего стола.

## <a name="resolution"></a>Решение

Устранить распространенные неполадки можно с помощью одного или нескольких описанных методов.

* Попробуйте открыть панель мониторинга кластера в новой вкладке браузера в режиме конфиденциальности.

* Если вы не можете отозвать учетные данные SSH, вы можете [сбросить учетные данные в пользовательском интерфейсе Ambari](../hdinsight-administer-use-portal-linux.md#change-passwords).

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]