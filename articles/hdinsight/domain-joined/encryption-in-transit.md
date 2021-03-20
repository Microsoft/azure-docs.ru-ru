---
title: Шифрование Azure HDInsight при передаче
description: Узнайте о функциях безопасности, которые обеспечивают шифрование при передаче для кластера Azure HDInsight.
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/24/2020
ms.openlocfilehash: fb3761ce7839cb4450997da094646b6604aeb895
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946861"
---
# <a name="ipsec-encryption-in-transit-for-azure-hdinsight"></a>Шифрование IPSec при передаче для Azure HDInsight

В этой статье рассматривается реализация шифрования при передаче для обмена данными между узлами кластера Azure HDInsight.

## <a name="background"></a>Историческая справка

Azure HDInsight предлагает разнообразные функции безопасности для защиты корпоративных данных. Эти решения сгруппированы по основам безопасности периметра, проверки подлинности, авторизации, аудита, шифрования и соответствия. Шифрование может применяться к данным как при хранении, так и при передаче.

Шифрование неактивных данных охватывается шифрованием на стороне сервера в учетных записях хранения Azure, а также шифрование дисков на виртуальных машинах Azure, которые входят в кластер HDInsight.

Шифрование данных при передаче в HDInsight достигается с помощью [протокола TLS](../transport-layer-security.md) для доступа к шлюзам кластера и [протоколу IPsec](https://wikipedia.org/wiki/IPsec) между узлами кластера. IPSec можно дополнительно включить между всеми головными узлами, рабочими узлами, граничными узлами, Zookeeper узлами, а также узлами шлюза и [брокера идентификаторов](./identity-broker.md) .

## <a name="enable-encryption-in-transit"></a>Включить шифрование при передаче

### <a name="azure-portal"></a>Портал Azure

Чтобы создать новый кластер с включенным шифрованием при транзитном транспорте с помощью портал Azure, выполните следующие действия.

1. Начните нормальный процесс создания кластера. См. раздел [Создание кластеров под управлением Linux в HDInsight с помощью портал Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) для создания начальных кластеров.
1. Заполните вкладки **Основные сведения** и **хранилище** . Перейдите на вкладку **безопасность и сеть** .

    :::image type="content" source="media/encryption-in-transit/create-cluster-security-networking-tab.png" alt-text="Вкладка Создание кластера — безопасность и сеть.":::

1. На вкладке **безопасность и сеть** установите флажок **включить шифрование при передаче** .

    :::image type="content" source="media/encryption-in-transit/enable-encryption-in-transit.png" alt-text="Создание кластера — включение шифрования при передаче.":::

### <a name="create-a-cluster-with-encryption-in-transit-enabled-through-the-azure-cli"></a>Создание кластера с шифрованием в транзите, включенным с помощью Azure CLI

Шифрование при передаче включается с помощью `isEncryptionInTransitEnabled` Свойства.

Вы можете [скачать пример шаблона и файл параметров](https://github.com/Azure-Samples/hdinsight-enterprise-security). Перед использованием шаблона и приведенного ниже фрагмента кода Azure CLI Замените следующие заполнители их правильными значениями:

| Заполнитель | Описание |
|---|---|
| `<SUBSCRIPTION_ID>` | Идентификатор подписки Azure |
| `<RESOURCE_GROUP>` | Группа ресурсов, в которой вы хотите создать новый кластер и учетную запись хранения. |
| `<STORAGEACCOUNTNAME>` | Существующая учетная запись хранения, которая будет использоваться с кластером. Имя должно иметь вид `ACCOUNTNAME.blob.core.windows.net` |
| `<CLUSTERNAME>` | Это имя вашего кластера HDInsight. |
| `<PASSWORD>` | Выбранный пароль для входа в кластер с помощью SSH и панели мониторинга Ambari. |
| `<VNET_NAME>` | Виртуальная сеть, в которой будет развернут кластер. |

Приведенный ниже фрагмент кода выполняет следующие начальные действия.

1. Выполняет вход в учетную запись Azure.
1. Задает активную подписку, в которой будут выполняться операции создания.
1. Создает новую группу ресурсов для новых действий развертывания.
1. Разверните шаблон, чтобы создать новый кластер.

```azurecli
az login
az account set --subscription <SUBSCRIPTION_ID>

# Create resource group
az group create --name <RESOURCEGROUPNAME> --location eastus2

az deployment group create --name HDInsightEnterpriseSecDeployment \
    --resource-group <RESOURCEGROUPNAME> \
    --template-file hdinsight-enterprise-security.json \
    --parameters parameters.json
```

## <a name="next-steps"></a>Дальнейшие действия

* [Общие сведения о корпоративной безопасности в Azure HDInsight](hdinsight-security-overview.md)
* [Синхронизация Azure Active Directory пользователей с кластером HDInsight](../disk-encryption.md).
