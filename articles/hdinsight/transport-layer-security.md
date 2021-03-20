---
title: Безопасность транспортного уровня в Azure HDInsight
description: Протоколы TLS и SSL — это криптографические алгоритмы, обеспечивающие безопасность обмена данными через сеть компьютера.
ms.service: hdinsight
ms.topic: conceptual
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: 648f24e87e7d8a49c9c495a78e56afb8ae9e8aae
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944697"
---
# <a name="transport-layer-security-in-azure-hdinsight"></a>Безопасность транспортного уровня в Azure HDInsight

Подключение к кластеру HDInsight через конечную точку общедоступного кластера `https://CLUSTERNAME.azurehdinsight.net` осуществляется через узлы шлюза кластера. Эти подключения защищаются с помощью протокола, называемого TLS. Применение более высоких версий TLS в шлюзах повышает безопасность этих подключений. Дополнительные сведения о том, почему следует использовать более новые версии протокола TLS, см. в разделе [решение проблемы tls 1,0](/security/solving-tls1-problem).

По умолчанию кластеры Azure HDInsight принимают TLS 1,2 подключений на общедоступных конечных точках HTTPS и более ранние версии для обеспечения обратной совместимости. Вы можете управлять минимальной версией TLS, поддерживаемой на узлах шлюза во время создания кластера, с помощью портал Azure или шаблона диспетчер ресурсов. На портале выберите версию TLS на вкладке **безопасность и сеть** во время создания кластера. Для шаблона диспетчер ресурсов во время развертывания используйте свойство **минсуппортедтлсверсион** . Пример шаблона см. в статье шаблон быстрого запуска для [HDInsight минимум TLS 1,2](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-minimum-tls). Это свойство поддерживает три значения: "1,0", "1,1" и "1,2", которые соответствуют TLS 1.0 +, TLS 1.1 + и TLS 1.2 + соответственно.


## <a name="next-steps"></a>Дальнейшие действия

* [Планирование виртуальной сети для Azure HDInsight](./hdinsight-plan-virtual-network-deployment.md)
* [Создайте виртуальные сети для кластеров Azure HDInsight](hdinsight-create-virtual-network.md).
* [Группы безопасности сети](../virtual-network/network-security-groups-overview.md).