---
title: Включение автоматического создания разделов в Apache Kafka — Azure HDInsight
description: Узнайте, как настроить автоматическое создание разделов в Apache Kafka в HDInsight. Можно настроить Kafka, задав для параметра значение `auto.create.topics.enable` true через Ambari. Или во время создания кластера с помощью PowerShell или шаблонов диспетчер ресурсов.
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/28/2020
ms.openlocfilehash: 3766d41959383d802e50aafbf59b9841d1c8d74e
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104870693"
---
# <a name="how-to-configure-apache-kafka-on-hdinsight-to-automatically-create-topics"></a>Настройка автоматического создания разделов в Apache Kafka в HDInsight

По умолчанию Apache Kafka в HDInsight не включает автоматическое создание разделов. Но вы можете включить автоматическое создание разделов для существующих кластеров с помощью Apache Ambari. Также автоматическое создание разделов можно включить при создании нового кластера Kafka с помощью шаблона Azure Resource Manager.

## <a name="apache-ambari-web-ui"></a>Веб-интерфейс Apache Ambari

Чтобы включить автоматическое создание разделов в существующем кластере через пользовательский веб-интерфейс Ambari, выполните следующие действия:

1. В [портал Azure](https://portal.azure.com)выберите кластер Kafka.

1. На **панели мониторинга кластера** выберите **Ambari Home**.

    :::image type="content" source="./media/apache-kafka-auto-create-topics/azure-portal-cluster-dashboard-ambari.png" alt-text="Изображение страницы портала с выбранной панелью мониторинга кластера" border="true":::

    Когда появится запрос, пройдите проверку подлинности, используя учетные данные (администратора) входа для кластера. Вместо этого можно подключиться к Амабри напрямую из `https://CLUSTERNAME.azurehdinsight.net/` расположения, где `CLUSTERNAME` — это имя кластера Kafka.

1. В списке в левой части страницы выберите службу Kafka.

    :::image type="content" source="./media/apache-kafka-auto-create-topics/hdinsight-service-list.png" alt-text="Вкладка списка служб Apache Ambari" border="true":::

1. Выберите "Конфигурации" в середине страницы.

    :::image type="content" source="./media/apache-kafka-auto-create-topics/hdinsight-service-config.png" alt-text="Вкладка &quot;конфигурации&quot; службы Apache Ambari" border="true":::

1. В поле "Фильтр" введите значение параметра `auto.create`.

    :::image type="content" source="./media/apache-kafka-auto-create-topics/hdinsight-filter-field.png" alt-text="Поле фильтра поиска Apache Ambari" border="true":::

    Этот параметр фильтрует список свойств и отображает `auto.create.topics.enable` параметр.

1. Измените значение `auto.create.topics.enable` на `true` , а затем выберите **сохранить**. Добавьте заметку, а затем нажмите кнопку **сохранить** еще раз.

    :::image type="content" source="./media/apache-kafka-auto-create-topics/auto-create-topics-enable.png" alt-text="Изображение ввода значения для параметра auto.create.topics.enable" border="true":::

1. Выберите службу Kafka и щелкните __Перезапустить__, а затем выберите __Restart all affected__ (Перезапустить все затронутые). При появлении запроса выберите __Подтверждение перезапустить все__.

    :::image type="content" source="./media/apache-kafka-auto-create-topics/restart-all-affected.png" alt-text="&quot;Apache Ambari перезапускает все затронутые&quot;" border="true":::

> [!NOTE]  
> Можно также задать значения Ambari с помощью REST API Ambari. Обычно это сложнее, так как необходимо сделать несколько вызовов RESTFUL для получения текущей конфигурации, изменить ее и т. д. Дополнительные сведения см. в статье [Управление кластерами HDInsight с помощью Apache Ambari REST API](../hdinsight-hadoop-manage-ambari-rest-api.md) документе.

## <a name="resource-manager-templates"></a>Шаблоны Resource Manager

При создании кластера Kafka с помощью шаблона Azure Resource Manager вы можете напрямую задать значение параметра `auto.create.topics.enable`, добавив его в `kafka-broker`. В следующем фрагменте JSON для этого параметра устанавливается значение `true`:

```json
"clusterDefinition": {
    "kind": "kafka",
    "configurations": {
    "gateway": {
        "restAuthCredential.isEnabled": true,
        "restAuthCredential.username": "[parameters('clusterLoginUserName')]",
        "restAuthCredential.password": "[parameters('clusterLoginPassword')]"
    },
    "kafka-broker": {
        "auto.create.topics.enable": "true"
    }
}
```

## <a name="next-steps"></a>Next Steps

Из этого документа вы узнали, как включить автоматическое создание разделов для Apache Kafka в HDInsight. Дополнительные сведения о работе с Kafka вы можете получить по следующим ссылкам.

* [Анализ журналов для Apache Kafka в HDInsight](apache-kafka-log-analytics-operations-management.md)
* [Репликация данных между кластерами Apache Kafka](apache-kafka-mirroring.md)
