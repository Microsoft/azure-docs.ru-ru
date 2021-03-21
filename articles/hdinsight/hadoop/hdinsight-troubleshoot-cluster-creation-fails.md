---
title: Устранение сбоев при создании кластера с помощью Azure HDInsight
description: Узнайте, как устранять неполадки при создании кластера Apache для Azure HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: troubleshooting
ms.date: 04/14/2020
ms.openlocfilehash: f909f5714a049f63032c8a23ca1731777a40d332
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101702867"
---
# <a name="troubleshoot-cluster-creation-failures-with-azure-hdinsight"></a>Устранение сбоев при создании кластера с помощью Azure HDInsight

Ниже перечислены основные причины сбоев при создании кластера.

- Проблемы с разрешениями
- Ограничения политики ресурсов
- брандмауэры;
- Блокировки ресурсов
- Неподдерживаемые версии компонентов
- Ограничения имен учетных записей хранения
- Простои службы

## <a name="permissions-issues"></a>Проблемы с разрешениями

Если вы используете Azure Data Lake Storage 2-го поколения и получаете сообщение об ошибке `AmbariClusterCreationFailedErrorCode` " :::no-loc text="Internal server error occurred while processing the request. Please retry the request or contact support."::: ", откройте портал Azure, перейдите к своей учетной записи хранения и в разделе Управление доступом (IAM) убедитесь, что **участник данных BLOB-объектов хранилища** или роль **владельца данных BLOB-объекта хранилища** назначили пользователю доступ к **управляемому удостоверению** подписки. Подробные инструкции см. в разделе [Настройка разрешений для управляемого удостоверения на Data Lake Storage 2-го поколения](../hdinsight-hadoop-use-data-lake-storage-gen2-portal.md#set-up-permissions-for-the-managed-identity-on-the-data-lake-storage-gen2) .

Если вы используете Azure Data Lake Storage 1-го поколения, см. инструкции по установке и настройке. [используйте Azure Data Lake Storage 1-го поколения с кластерами Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen1.md). Data Lake Storage 1-го поколения не поддерживается для кластеров HBase и не поддерживается в HDInsight версии 4,0.

При использовании службы хранилища Azure убедитесь, что имя учетной записи хранения допустимо во время создания кластера.

## <a name="resource-policy-restrictions"></a>Ограничения политики ресурсов

Политики Azure на основе подписки могут запрещать создание общедоступных IP-адресов. Для создания кластера HDInsight требуется два общедоступных IP-адреса.  

Как правило, следующие политики могут повлиять на создание кластера:

* Политики, препятствующие созданию IP-адресов & подсистемы балансировки нагрузки в рамках подписки.
* Политика, препятствующая созданию учетной записи хранения.
* Политика, препятствующая удалению сетевых ресурсов (подсистемы балансировки IP-адресов/лоад).

## <a name="firewalls"></a>брандмауэры;

Брандмауэры в виртуальной сети или учетной записи хранения могут запретить обмен данными с IP-адресами управления HDInsight.

Разрешить трафик с IP-адресов, указанных в таблице ниже.

| Исходный IP-адрес | Назначение | Направление |
|---|---|---|
| 168.61.49.99 | *: 443 | Входящий трафик |
| 23.99.5.239 | *: 443 | Входящий трафик |
| 168.61.48.131 | *: 443 | Входящий трафик |
| 138.91.141.162 | *: 443 | Входящий трафик |

Также добавьте IP-адреса, относящиеся к региону, в котором создается кластер. Список адресов для каждого региона Azure см. в разделе [IP-адреса управления HDInsight](../hdinsight-management-ip-addresses.md) .

Если вы используете Express Route или собственный DNS-сервер, см. статью [планирование виртуальной сети для Azure HDInsight: подключение нескольких сетей](../hdinsight-plan-virtual-network-deployment.md#multinet).

## <a name="resources-locks"></a>Блокировки ресурсов  

Убедитесь, что [в виртуальной сети и группе ресурсов нет блокировок](../../azure-resource-manager/management/lock-resources.md). Кластеры не могут быть созданы или удалены, если группа ресурсов заблокирована. 

## <a name="unsupported-component-versions"></a>Неподдерживаемые версии компонентов

Убедитесь, что вы используете [поддерживаемую версию Azure HDInsight и Apache Hadoop компонент](../hdinsight-component-versioning.md) в решении.  

## <a name="storage-account-name-restrictions"></a>Ограничения имен учетных записей хранения

Длина имен учетных записей хранения не может превышать 24 символов и не может содержать специальный символ. Эти ограничения также касаются имени контейнера по умолчанию в учетной записи хранения.

Другие ограничения именования также применяются при создании кластера. Дополнительные сведения см. в разделе [ограничения имен кластеров](../hdinsight-hadoop-provision-linux-clusters.md#cluster-name).

## <a name="service-outages"></a>Простои службы

Проверьте [состояние Azure](https://status.azure.com) на наличие потенциальных сбоев или проблем со службой.

## <a name="next-steps"></a>Дальнейшие действия

* [Расширение возможностей HDInsight с помощью виртуальной сети Azure](../hdinsight-plan-virtual-network-deployment.md)
* [Использование Azure Data Lake Storage Gen2 с кластерами Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen2.md)  
* [Использование службы хранилища Azure с кластерами Azure HDInsight](../hdinsight-hadoop-use-blob-storage.md)
* [Установка кластеров в HDInsight с использованием Apache Hadoop, Apache Spark, Apache Kafka и других технологий](../hdinsight-hadoop-provision-linux-clusters.md)
