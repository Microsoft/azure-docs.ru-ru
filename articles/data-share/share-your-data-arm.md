---
title: Совместное использование данных за пределами организации с помощью шаблонов Resource Manager и Azure Data Share — краткое руководство
description: Научитесь совместному использованию данных с клиентами и партнерами с помощью Azure Data Share и шаблона Azure Resource Manager.
author: mumian
ms.author: jgao
ms.service: data-share
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 08/19/2020
ms.openlocfilehash: 5abe92120c8b822ac86ced90658869a0858d4ff4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92487693"
---
# <a name="quickstart-share-data-using-azure-data-share-and-arm-template"></a>Краткое руководство. Совместное использование данных с помощью Azure Data Share и шаблона Resource Manager

Узнайте, как настроить Azure Data Share из учетной записи хранения Azure с помощью шаблона Azure Resource Manager, а также начните обмениваться данными с клиентами и партнерами за пределами организации Azure. См. список поддерживаемых источников данных в [Azure Data Share](./supported-data-stores.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-share-share-storage-account%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-data-share-share-storage-account/).

:::code language="json" source="~/quickstart-templates/101-data-share-share-storage-account/azuredeploy.json":::

В шаблоне определены следующие ресурсы:

* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts):
* [Microsoft.Storage/storageAccounts/blobServices/containers](/azure/templates/microsoft.storage/storageaccounts/blobservices/containers)
* [Microsoft.DataShare/accounts](/azure/templates/microsoft.datashare/accounts)
* [Microsoft.DataShare/accounts/shares](/azure/templates/microsoft.datashare/accounts/shares)
* [Microsoft.Storage/storageAccounts/providers/roleAssignments](/azure/templates/microsoft.authorization/roleassignments)
* [Microsoft.DataShare/accounts/shares/dataSets](/azure/templates/microsoft.datashare/accounts/shares/datasets)
* [Microsoft.DataShare/accounts/shares/invitations](/azure/templates/microsoft.datashare/accounts/shares/invitations)
* [Microsoft.DataShare/accounts/shares/synchronizationSettings](/azure/templates/microsoft.datashare/accounts/shares/synchronizationsettings)

В этом примере шаблона выполняются следующие действия:

* Создание учетной записи хранения и контейнера, используемого в качестве источника данных Data Share.
* Создание учетной записи Data Share и Data Share.
* Создание назначения роли для предоставления роли читателя данных Storage Blob исходному ресурсу общих данных. См. [Роли и требования для Azure Data Share](./concepts-roles-permissions.md).
* Добавление набора данных в Data Share.
* Добавление получателей в Data Share.
* Включение расписания моментальных снимков для Data Share.

Этот шаблон создается c целью обучения. На практике обычно используются данные имеющейся учетной записи хранения. Необходимо создать назначение роли перед запуском шаблона или скрипта для создания набора данных. Иногда при развертывании шаблона может появиться следующее сообщение об ошибке:

```plaintext
"Missing permissions for DataShareAcccount on resource 'subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>' (Code: 5006)"
```

Это связано с тем, что развертывание пытается создать набор данных перед завершением назначения роли в Azure. Несмотря на сообщение об ошибке, развертывание может быть успешным. Вы по-прежнему сможете [просмотреть развернутые ресурсы](#review-deployed-resources).

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон.

    [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-share-share-storage-account%2Fazuredeploy.json)
1. Введите или выберите следующие значения.

    * **Подписка**. Выберите подписку Azure, используемую для создания общего ресурса данных и других ресурсов.
    * **Группа ресурсов**. Выберите **Создать**, чтобы создать новую группу ресурсов или выберите имеющуюся.
    * **Расположение**. Выберите расположение группы ресурсов.
    * **Имя проекта**. Введите имя проекта.  Имя проекта используется для создания имен ресурсов.  См. определения переменных в предыдущем шаблоне.
    * **Расположение**. Выберите расположение для ресурсов.  Вы можете выбрать то же расположение для группы ресурсов.
    * **Invitation Email** (Приглашение по электронной почте). Введите адрес электронной почты для входа в Azure получателя общего ресурса данных.  Псевдоним электронной почты не подходит.

    Для остальных параметров используйте значение по умолчанию.
1. Установите флажок **Я принимаю указанные выше условия** и нажмите кнопку **Приобрести**.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

1. Войдите на [портал Azure](https://portal.azure.com).
1. Откройте созданную учетную запись общего ресурса данных.
1. В меню слева выберите **Send Shares** (Отправить общие папки).  Отобразится одна учетная запись хранения.
1. Выберите учетную запись хранения.  В разделе **Сведения** вы увидите параметр синхронизации, настроенный в шаблоне.

    ![Параметры синхронизации учетной записи хранения Azure Data Share](./media/share-your-data-arm/azure-data-share-storage-account-synchronization-settings.png)
1. В верхней части экрана выберите **Приглашения**. Отобразится адрес электронной почты, указанный при развертывании шаблона. В разделе **Состояние** должно быть указано значение **Ожидание**.

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите ненужную группу ресурсов. Ресурсы в ней будут также удалены.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать Azure Data Share и пригласить получателей. Дополнительные сведения о том, как потребитель данных может получить и принять общий ресурс данных, см. на [этой странице](subscribe-to-data-share.md).
