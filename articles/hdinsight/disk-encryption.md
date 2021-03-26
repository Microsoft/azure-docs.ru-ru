---
title: Двойное шифрование неактивных данных
titleSuffix: Azure HDInsight
description: В этой статье описываются два уровня шифрования, доступные для неактивных данных в кластерах Azure HDInsight.
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/10/2020
ms.openlocfilehash: 3d4f9e3be02a64efa058ea1f84a3e261cb6166fc
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104867123"
---
# <a name="azure-hdinsight-double-encryption-for-data-at-rest"></a>Двойное шифрование Azure HDInsight для неактивных данных

В этой статье обсуждаются методы шифрования неактивных данных в кластерах Azure HDInsight. Шифрование неактивных данных означает шифрование на управляемых дисках (дисках данных, дисках ОС и временных дисках), подключенных к виртуальным машинам кластера HDInsight. 

Этот документ не поддерживает данные, хранящиеся в вашей учетной записи хранения Azure. Кластеры могут иметь одну или несколько подключенных учетных записей хранения Azure, где ключи шифрования также могут управляться корпорацией Майкрософт или управляемыми клиентом, но служба шифрования отличается. Дополнительные сведения о шифровании службы хранилища Azure см. [в статье шифрование службы хранилища Azure для неактивных данных](../storage/common/storage-service-encryption.md).

## <a name="introduction"></a>Введение

В Azure есть три основные роли управляемых дисков: диск данных, диск операционной системы и временный диск. Дополнительные сведения о различных типах управляемых дисков см. в статье [Введение в управляемые диски Azure](../virtual-machines/managed-disks-overview.md). 

HDInsight поддерживает несколько типов шифрования на двух разных уровнях:

- Шифрование на стороне сервера (SSE) — SSE выполняется службой хранилища. В HDInsight SSE используется для шифрования дисков ОС и дисков данных. По умолчанию он включен. SSE — это служба шифрования уровня 1.
- Шифрование на узле с помощью ключа, управляемого платформой, аналогично SSE, этот тип шифрования выполняется службой хранилища. Однако он предназначен только для временных дисков и не включен по умолчанию. Шифрование на узле также является службой шифрования уровня 1.
- Шифрование неактивных данных с помощью управляемого клиентом ключа. Этот тип шифрования можно использовать на временных дисках. Он не включен по умолчанию и требует, чтобы клиент предоставил собственный ключ через хранилище ключей Azure. Шифрование неактивных компонентов — это служба шифрования уровня 2.

Эти типы приведены в следующей таблице.

|Тип кластера |Диск ОС (управляемый диск) |Диск данных (управляемый диск) |Диск временных данных (локальный SSD) |
|---|---|---|---|
|Kafka, HBase с ускоренной записью|Layer1: [Шифрование SSE](../virtual-machines/managed-disks-overview.md#encryption) по умолчанию|Layer1: [Шифрование SSE](../virtual-machines/managed-disks-overview.md#encryption) по умолчанию, Layer2: Необязательное шифрование неактивных с помощью CMK|Layer1: Необязательное шифрование на узле с помощью PMK, Layer2: Необязательное шифрование неактивных с помощью CMK|
|Все остальные кластеры (Spark, Interactive, Hadoop, HBase без ускоренной записи)|Layer1: [Шифрование SSE](../virtual-machines/managed-disks-overview.md#encryption) по умолчанию|Недоступно|Layer1: Необязательное шифрование на узле с помощью PMK, Layer2: Необязательное шифрование неактивных с помощью CMK|

## <a name="encryption-at-rest-using-customer-managed-keys"></a>Шифрование неактивных с помощью управляемых клиентом ключей

Шифрование с помощью управляемого клиентом ключа — это одноэтапный процесс, который обрабатывается при создании кластера без дополнительных затрат. Вам нужно только авторизовать управляемое удостоверение с Azure Key Vault и добавить ключ шифрования при создании кластера.

Как диски данных, так и временные диски на каждом узле кластера шифруются с помощью симметричного ключа шифрования данных (DEK). Ключ DEK защищен с помощью ключа шифрования ключей (KEK) из вашего хранилища ключей. Процессы шифрования и расшифровки полностью обрабатываются Azure HDInsight.

Для дисков ОС, подключенных к виртуальным машинам кластера, доступен только один уровень шифрования (PMK). Пользователям рекомендуется избегать копирования конфиденциальных данных на диски ОС, если для их сценариев требуется шифрование CMK.

Если брандмауэр хранилища ключей включен в хранилище ключей, где хранится ключ шифрования диска, IP-адреса поставщика региональных ресурсов HDInsight для региона, в котором будет развернут кластер, необходимо добавить в конфигурацию брандмауэра хранилища ключей. Это необходимо потому, что HDInsight не является доверенной службой хранилища ключей Azure.

Для безопасной смены ключей в хранилище ключей можно использовать портал Azure или Azure CLI. При повороте ключа кластер HDInsight начинает использовать новый ключ в течение нескольких минут. Включите функции защиты ключа [обратимого удаления](../key-vault/general/soft-delete-overview.md) , чтобы защититься от сценариев атаки с помощью и случайного удаления. Хранилища ключей без этой функции защиты не поддерживаются.

### <a name="get-started-with-customer-managed-keys"></a>Начало работы с ключами, управляемыми клиентом

Чтобы создать кластер HDInsight с управляемым клиентом ключом, выполните следующие действия.

1. Создание управляемых удостоверений для ресурсов Azure
1. Создание хранилища Azure Key Vault
1. Создание ключа
1. Создание политики доступа
1. Создание кластера HDInsight с включенным ключом "Управление клиентами"
1. Вращение ключа шифрования

Каждый шаг подробно описан в одном из следующих разделов.

### <a name="create-managed-identities-for-azure-resources"></a>Создание управляемых удостоверений для ресурсов Azure

Создание управляемого удостоверения, назначенного пользователем, для проверки подлинности в Key Vault.

Конкретные шаги см. [в статье Создание управляемого удостоверения, назначаемого пользователем](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) . Дополнительные сведения о работе управляемых удостоверений в Azure HDInsight см. [в статье управляемые удостоверения в Azure hdinsight](hdinsight-managed-identities.md). Не забудьте сохранить идентификатор ресурса управляемого удостоверения для его добавления в политику доступа Key Vault.

### <a name="create-azure-key-vault"></a>Создание хранилища Azure Key Vault

Создать хранилище ключей. Конкретные действия см. в разделе [Create Azure Key Vault](../key-vault/general/quick-create-portal.md) .

HDInsight поддерживает только Azure Key Vault. Если у вас есть собственное хранилище ключей, вы можете импортировать ключи в Azure Key Vault. Помните, что для хранилища ключей должно быть включено **обратимое удаление** . Дополнительные сведения об импорте существующих ключей см. в статье [Сведения о ключах, секретах и сертификатах](../key-vault/general/about-keys-secrets-certificates.md).

### <a name="create-key"></a>Создание ключа

1. В новом хранилище ключей перейдите к   >  **разделу Параметры ключи**  >  **+ создать/импорт**.

    :::image type="content" source="./media/disk-encryption/create-new-key.png" alt-text="Создание нового ключа в хранилище ключей Azure":::

1. Укажите имя, а затем выберите **создать**. Ведение **типа ключа** по умолчанию **RSA**.

    :::image type="content" source="./media/disk-encryption/create-key.png" alt-text="создает имя ключа":::

1. Когда вы вернетесь на страницу **ключи** , выберите созданный ключ.

    :::image type="content" source="./media/disk-encryption/key-vault-key-list.png" alt-text="список ключей хранилища ключей":::

1. Выберите версию, чтобы открыть страницу **ключа версии** . При использовании собственного ключа для шифрования кластера HDInsight необходимо указать URI ключа. Скопируйте **идентификатор ключа** и сохраните его, пока не будете готовы к созданию кластера.

    :::image type="content" source="./media/disk-encryption/get-key-identifier.png" alt-text="получение идентификатора ключа":::

### <a name="create-access-policy"></a>Создание политики доступа

1. В новом хранилище ключей перейдите к разделу **Параметры**  >  **политики доступа**  >  **+ Добавить политику доступа**.

    :::image type="content" source="./media/disk-encryption/key-vault-access-policy.png" alt-text="Создание политики доступа Azure Key Vault.":::

1. На странице **Добавление политики доступа** укажите следующие сведения.

    |Свойство |Описание|
    |---|---|
    |Основные разрешения|Выберите **Получение**, **расперенос ключа** и **Ключ переноса**.|
    |Разрешения секретов|Выберите **Get**, **Set** и **Delete**.|
    |Выбор субъекта|Выберите назначенное пользователем управляемое удостоверение, созданное ранее.|

    :::image type="content" source="./media/disk-encryption/azure-portal-add-access-policy.png" alt-text="Задание выбранного субъекта для политики доступа Azure Key Vault":::

1. Выберите **Добавить**.

1. Щелкните **Сохранить**.

    :::image type="content" source="./media/disk-encryption/add-key-vault-access-policy-save.png" alt-text="Сохранить политику доступа Azure Key Vault":::

### <a name="create-cluster-with-customer-managed-key-disk-encryption"></a>Создание кластера с ключом шифрования диска, управляемым клиентом

Теперь можно создать кластер HDInsight. Ключи, управляемые клиентом, можно применять только к новым кластерам во время создания кластера. Невозможно удалить шифрование из управляемых клиентом ключей, а ключи, управляемые клиентом, невозможно добавить в существующие кластеры.

Начиная с выпуска за ноябрь 2020, HDInsight поддерживает создание кластеров с помощью универсальных кодов ресурсов (URI) с управлением версиями и без версий. При создании кластера с универсальным кодом ресурса (URI) без версии кластер HDInsight попытается выполнить автоматический поворот ключа при обновлении ключа в Azure Key Vault. При создании кластера с универсальным кодом ресурса (URI) с версией ключа необходимо выполнить Ручное вращение ключа, как описано в разделе Смена [ключа шифрования](#rotating-the-encryption-key).

Для кластеров, созданных до выпуска за ноябрь 2020, необходимо вручную выполнить смену ключа с помощью URI ключа версии.

#### <a name="using-the-azure-portal"></a>Использование портала Azure

Во время создания кластера можно либо использовать ключ версии, либо ключ, не являющийся версией, следующим образом:

- С **отслеживанием версий** — при создании кластера укажите полный **идентификатор ключа**, включая версию ключа. Например, `https://contoso-kv.vault.azure.net/keys/myClusterKey/46ab702136bc4b229f8b10e8c2997fa4`.
- **Безверсия** — во время создания кластера укажите только **идентификатор ключа**. Например, `https://contoso-kv.vault.azure.net/keys/myClusterKey`.

Также необходимо назначить управляемое удостоверение кластеру.

:::image type="content" source="./media/disk-encryption/create-cluster-portal.png" alt-text="Создать новый кластер":::

#### <a name="using-azure-cli"></a>Использование Azure CLI

В следующем примере показано, как использовать Azure CLI для создания нового кластера Apache Spark с включенным шифрованием дисков. Дополнительные сведения см. в разделе [Azure CLI AZ hdinsight Create](/cli/azure/hdinsight#az-hdinsight-create). Параметр `encryption-key-version` является необязательным.

```azurecli
az hdinsight create -t spark -g MyResourceGroup -n MyCluster \
-p "HttpPassword1234!" --workernode-data-disks-per-node 2 \
--storage-account MyStorageAccount \
--encryption-key-name SparkClusterKey \
--encryption-key-version 00000000000000000000000000000000 \
--encryption-vault-uri https://MyKeyVault.vault.azure.net \
--assign-identity MyMSI
```

#### <a name="using-azure-resource-manager-templates"></a>Использование шаблонов Azure Resource Manager

В следующем примере показано, как использовать шаблон Azure Resource Manager для создания нового кластера Apache Spark с включенным шифрованием дисков. Дополнительные сведения см. в статье [что такое шаблоны ARM?](../azure-resource-manager/templates/overview.md). Свойство шаблона Resource Manager `diskEncryptionKeyVersion` является необязательным.

В этом примере для вызова шаблона используется PowerShell.

```powershell
$templateFile = "azuredeploy.json"
$ResourceGroupName = "MyResourceGroup"
$clusterName = "MyCluster"
$password = ConvertTo-SecureString 'HttpPassword1234!' -AsPlainText -Force
$diskEncryptionVaultUri = "https://MyKeyVault.vault.azure.net"
$diskEncryptionKeyName = "SparkClusterKey"
$diskEncryptionKeyVersion = "00000000000000000000000000000000"
$managedIdentityName = "MyMSI"

New-AzResourceGroupDeployment `
  -Name mySpark `
  -TemplateFile $templateFile `
  -ResourceGroupName $ResourceGroupName `
  -clusterName $clusterName `
  -clusterLoginPassword $password `
` -sshPassword $password `
  -diskEncryptionVaultUri $diskEncryptionVaultUri `
  -diskEncryptionKeyName $diskEncryptionKeyName `
  -diskEncryptionKeyVersion $diskEncryptionKeyVersion `
  -managedIdentityName $managedIdentityName
```

Содержимое шаблона управления ресурсами `azuredeploy.json` :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "0.9.0.0",
  "parameters": {
    "clusterName": {
      "type": "string",
      "metadata": {
        "description": "The name of the HDInsight cluster to create."
      }
    },
    "clusterLoginUserName": {
      "type": "string",
      "defaultValue": "admin",
      "metadata": {
        "description": "These credentials can be used to submit jobs to the cluster and to log into cluster dashboards."
      }
    },
    "clusterLoginPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "The location where all azure resources will be deployed."
      }
    },
    "sshUserName": {
      "type": "string",
      "defaultValue": "sshuser",
      "metadata": {
        "description": "These credentials can be used to remotely access the cluster."
      }
    },
    "sshPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "headNodeSize": {
      "type": "string",
      "defaultValue": "Standard_D12_v2",
      "metadata": {
        "description": "The VM size of the head nodes."
      }
    },
    "workerNodeSize": {
      "type": "string",
      "defaultValue": "Standard_D13_v2",
      "metadata": {
        "description": "The VM size of the worker nodes."
      }
    },
    "diskEncryptionVaultUri": {
      "type": "string",
      "metadata": {
        "description": "The Key Vault DNSname."
      }
    },
    "diskEncryptionKeyName": {
      "type": "string",
      "metadata": {
        "description": "The Key Vault key name."
      }
    },
    "diskEncryptionKeyVersion": {
      "type": "string",
      "metadata": {
        "description": "The Key Vault key version for the selected key."
      }
    },
    "managedIdentityName": {
      "type": "string",
      "metadata": {
        "description": "The user-assigned managed identity."
      }
    }
  },
  "variables": {
    "defaultStorageAccount": {
      "name": "[uniqueString(resourceGroup().id)]",
      "type": "Standard_LRS"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('defaultStorageAccount').name]",
      "location": "[parameters('location')]",
      "apiVersion": "2019-06-01",
      "sku": {
        "name": "[variables('defaultStorageAccount').type]"
      },
      "kind": "Storage",
      "properties": {}
    },
    {
      "apiVersion": "2018-06-01-preview",
      "name": "[parameters('clusterName')]",
      "type": "Microsoft.HDInsight/clusters",
      "location": "[parameters('location')]",
      "properties": {
        "clusterVersion": "3.6",
        "osType": "Linux",
        "tier": "standard",
        "clusterDefinition": {
          "kind": "spark",
          "componentVersion": {
            "Spark": "2.3"
          },
          "configurations": {
            "gateway": {
              "restAuthCredential.isEnabled": true,
              "restAuthCredential.username": "[parameters('clusterLoginUserName')]",
              "restAuthCredential.password": "[parameters('clusterLoginPassword')]"
            }
          }
        },
        "storageProfile": {
          "storageaccounts": [
            {
              "name": "[replace(replace(reference(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name), '2019-06-01').primaryEndpoints.blob,'https://',''),'/','')]",
              "isDefault": true,
              "container": "[parameters('clusterName')]",
              "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name), '2019-06-01').keys[0].value]"
            }
          ]
        },
        "computeProfile": {
          "roles": [
            {
              "name": "headnode",
              "minInstanceCount": 1,
              "targetInstanceCount": 2,
              "hardwareProfile": {
                "vmSize": "[parameters('headNodeSize')]"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                },
              },
            },
            {
              "name": "workernode",
              "targetInstanceCount": 1,
              "hardwareProfile": {
                "vmSize": "[parameters('workerNodeSize')]"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                },
              },
            }
          ]
        },
        "minSupportedTlsVersion": "1.2",
        "diskEncryptionProperties": {
          "vaultUri": "[parameters('diskEncryptionVaultUri')]",
          "keyName": "[parameters('diskEncryptionKeyName')]",
          "keyVersion": "[parameters('diskEncryptionKeyVersion')]",
          "msiResourceId": "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('managedIdentityName'))]"
        }
      },
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('managedIdentityName'))]": {}
        }
      }
    }
  ]
}
```

### <a name="rotating-the-encryption-key"></a>Вращение ключа шифрования

Ключи шифрования, используемые в работающем кластере, можно изменить с помощью портал Azure или Azure CLI. Для этой операции кластер должен иметь доступ как к текущему ключу, так и к новому ключу, в противном случае операция вращения ключа завершится ошибкой. Для кластеров, созданных после выпуска за ноябрь 2020, можно выбрать, нужно ли использовать новый ключ в качестве версии. Для кластеров, созданных до выпуска за ноябрь 2020, при смене ключа шифрования необходимо использовать ключ версии.

#### <a name="using-the-azure-portal"></a>Использование портала Azure

Для смены ключа требуется универсальный код ресурса (URI) основного хранилища ключей. После этого перейдите в раздел свойств кластера HDInsight на портале и нажмите кнопку **изменить ключ** в разделе **URL-адрес ключа шифрования диска**. Введите новый URL-адрес ключа и отправьте его, чтобы повернуть ключ.

:::image type="content" source="./media/disk-encryption/change-key.png" alt-text="повернуть ключ шифрования диска":::

#### <a name="using-azure-cli"></a>Использование Azure CLI

В следующем примере показано, как поворачивать ключ шифрования диска для существующего кластера HDInsight. Дополнительные сведения см. в разделе [Azure CLI AZ hdinsight сдвиг-диск — ключ шифрования](/cli/azure/hdinsight#az-hdinsight-rotate-disk-encryption-key).

```azurecli
az hdinsight rotate-disk-encryption-key \
--encryption-key-name SparkClusterKey \
--encryption-key-version 00000000000000000000000000000000 \
--encryption-vault-uri https://MyKeyVault.vault.azure.net \
--name MyCluster \
--resource-group MyResourceGroup
```

## <a name="faq-for-customer-managed-key-encryption"></a>Часто задаваемые вопросы о шифровании ключей, управляемых клиентом

**Как кластер HDInsight обращается к моему хранилищу ключей?**

HDInsight обращается к вашему экземпляру Azure Key Vault, используя управляемое удостоверение, которое вы связываете с кластером HDInsight. Это управляемое удостоверение можно создать до или во время создания кластера. Вам также потребуется предоставить управляемому удостоверению доступ к хранилищу ключей, где хранится ключ.

**Эта функция доступна для всех кластеров в HDInsight?**

Шифрование с управляемым клиентом ключом доступно для всех типов кластеров, кроме Spark 2,1 и 2,2.

**Можно ли использовать несколько ключей для шифрования разных дисков или папок?**

Нет, все управляемые диски и диски ресурсов шифруются с помощью одного и того же ключа.

**Что произойдет, если кластер теряет доступ к хранилищу ключей или ключу?**

Если кластер теряет доступ к ключу, на портале Apache Ambari отображаются предупреждения. В этом состоянии операция **изменения ключа** завершится ошибкой. После восстановления доступа к ключам Ambari предупреждения отправляются, а такие операции, как смена ключа, могут быть успешно выполнены.

:::image type="content" source="./media/disk-encryption/ambari-alert.png" alt-text="оповещение о доступе к Key Ambari":::

**Как восстановить кластер, если ключи удалены?**

Так как поддерживаются только ключи с поддержкой обратимого удаления, если ключи восстановлены в хранилище ключей, кластер должен восстановить доступ к ключам. Чтобы восстановить ключ Azure Key Vault, см. раздел [Undo-азкэйваулткэйремовал](/powershell/module/az.keyvault/Undo-AzKeyVaultKeyRemoval) или [AZ-keyvault-Key-Recover](/cli/azure/keyvault/key#az-keyvault-key-recover).


**Если кластер масштабируется, будут ли новые узлы поддерживать ключи, управляемые клиентом, без проблем?**

Да. Кластеру требуется доступ к ключу в хранилище ключей во время масштабирования. Один и тот же ключ используется для шифрования управляемых дисков и дисков ресурсов в кластере.

**Являются ли ключи, управляемые клиентом, доступными в моем расположении?**

Ключи, управляемые клиентом HDInsight, доступны во всех общедоступных облаках и национальных облаках.

## <a name="encryption-at-host-using-platform-managed-keys"></a>Шифрование на узле с использованием ключей, управляемых платформой

### <a name="enable-in-the-azure-portal"></a>Включение на портале Azure

Шифрование на узле можно включить во время создания кластера в портал Azure.

> [!Note]
> Если шифрование на узле включено, вы не сможете добавлять приложения в кластер HDInsight из Azure Marketplace.

:::image type="content" source="media/disk-encryption/encryption-at-host.png" alt-text="Включите шифрование на узле.":::

Этот параметр включает [Шифрование на узле](../virtual-machines/disks-enable-host-based-encryption-portal.md) для временных дисков с данными виртуальных машин HDInsight с помощью PMK. Шифрование на узле поддерживается только [для определенных номеров SKU виртуальных машин в ограниченном](../virtual-machines/disks-enable-host-based-encryption-portal.md) регионе, и HDInsight поддерживает [следующие конфигурации узла и номера SKU](./hdinsight-supported-node-configuration.md).

Сведения о правильном размере виртуальной машины для кластера HDInsight см. в статье [Выбор правильного размера виртуальной машины для кластера Azure hdinsight](hdinsight-selecting-vm-size.md). Номер SKU виртуальной машины по умолчанию для узла Zookeeper при включенном шифровании на узле будет размера DS2V2.

### <a name="enable-using-powershell"></a>Включение с помощью PowerShell

В следующем фрагменте кода показано, как можно создать новый кластер Azure HDInsight с шифрованием на узле с помощью PowerShell. `-EncryptionAtHost $true`Для включения этой функции используется параметр.

```powershell
$storageAccountResourceGroupName = "Group"
$storageAccountName = "yourstorageacct001"
$storageAccountKey = Get-AzStorageAccountKey `
    -ResourceGroupName $storageAccountResourceGroupName `
    -Name $storageAccountName | %{ $_.Key1 }
$storageContainer = "container002"
# Cluster configuration info
$location = "East US 2"
$clusterResourceGroupName = "Group"
$clusterName = "your-hadoop-002"
$clusterCreds = Get-Credential
# If the cluster's resource group doesn't exist yet, run:
# New-AzResourceGroup -Name $clusterResourceGroupName -Location $location
# Create the cluster
New-AzHDInsightCluster `
    -ClusterType Hadoop `
    -ClusterSizeInNodes 4 `
    -ResourceGroupName $clusterResourceGroupName `
    -ClusterName $clusterName `
    -HttpCredential $clusterCreds `
    -Location $location `
    -DefaultStorageAccountName "$storageAccountName.blob.core.contoso.net" `
    -DefaultStorageAccountKey $storageAccountKey `
    -DefaultStorageContainer $storageContainer `
    -SshCredential $clusterCreds `
    -EncryptionAtHost $true `
```

### <a name="enable-using-azure-cli"></a>Включение мониторинга с помощью Azure CLI

В следующем фрагменте кода показано, как можно создать новый кластер Azure HDInsight с включенным шифрованием на узле с помощью Azure CLI. `--encryption-at-host true`Для включения этой функции используется параметр.

```azurecli
az hdinsight create -t spark -g MyResourceGroup -n MyCluster \\
-p "yourpass" \\
--storage-account MyStorageAccount --encryption-at-host true
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о Azure Key Vault см. в разделе [что такое Azure Key Vault](../key-vault/general/overview.md).
* [Обзор корпоративной безопасности в Azure HDInsight](./domain-joined/hdinsight-security-overview.md).
