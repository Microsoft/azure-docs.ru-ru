---
title: Краткое руководство. Развертывание шаблона Resource Manager для создания виртуальной машины
description: Из этого краткого руководства вы узнаете, как с помощью шаблона Resource Manager создать виртуальную машину Windows.
author: cynthn
ms.service: virtual-machines
ms.collection: windows
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 06/04/2020
ms.author: cynthn
ms.custom: subject-armqs
ms.openlocfilehash: b2856c51ccc31b57ea567c5d0940cd260797f770
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102560758"
---
# <a name="quickstart-create-a-windows-virtual-machine-using-an-arm-template"></a>Краткое руководство. Создание виртуальной машины Windows с помощью шаблона ARM

В этом кратком руководстве показано, как с помощью шаблона Azure Resource Manager (шаблон ARM) развернуть в Azure виртуальную машину Windows.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-vm-simple-windows%2fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/).

:::code language="json" source="~/quickstart-templates/101-vm-simple-windows/azuredeploy.json":::


В шаблоне определены следующие ресурсы.

- [**Microsoft.Network/virtualNetworks/subnets**](/azure/templates/Microsoft.Network/virtualNetworks/subnets): создание подсети.
- [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts): создание учетной записи хранения.
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/Microsoft.Network/publicIPAddresses): создание общедоступного IP-адреса.
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/Microsoft.Network/networkSecurityGroups): создание группы безопасности сети.
- [**Microsoft.Network/virtualNetworks**](/azure/templates/Microsoft.Network/virtualNetworks): создание виртуальной сети.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/Microsoft.Network/networkInterfaces): создание сетевой карты.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/Microsoft.Compute/virtualMachines): создание виртуальной машины.



## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Шаблон создает хранилище ключей и секрет.

    [![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-vm-simple-windows%2fazuredeploy.json)

1. Введите или выберите следующие значения. По возможности используйте значения по умолчанию.

    - **Подписка**. Выберите нужную подписку Azure.
    - **Группа ресурсов**. Выберите группу ресурсов в раскрывающемся списке или нажмите **Создать**, введите уникальное имя группы ресурсов и нажмите кнопку **ОК**.
    - **Расположение**. Выберите расположение.  Например, **центральная часть США**.
    - **Имя пользователя администратора**. Укажите имя пользователя, например *azureuser*.
    - **Пароль администратора**. Укажите пароль, который будет использоваться для учетной записи администратора. Пароль должен включать минимум 12 символов и соответствовать [определенным требованиям к сложности](faq.md#what-are-the-password-requirements-when-creating-a-vm).
    - **Префикс метки DNS**. Введите уникальный идентификатор, который будет использоваться как префикс метки DNS.
    - **Версия ОС Windows**. Выберите версию Windows, которую требуется запустить на виртуальной машине.
    - **Размер виртуальной машины**. Выберите [размер](../sizes.md), который будет использоваться для виртуальной машины.
    - **Расположение**. По умолчанию совпадает с расположением группы ресурсов, если она уже существует.
1. Выберите **Review + create** (Просмотреть и создать). После завершения проверки нажмите **Создать**, чтобы создать и развернуть виртуальную машину.


Для развертывания шаблона используется портал Azure. В дополнение к порталу Azure можно также использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Вы можете использовать портал Azure для проверки виртуальной машины и других созданных ресурсов. После завершения развертывания нажмите **Перейти к группе ресурсов**, чтобы просмотреть виртуальную машину и другие ресурсы.


## <a name="clean-up-resources"></a>Очистка ресурсов

Когда группа ресурсов станет не нужна, удалите ее. При этом будет удалена виртуальная машина и все ресурсы из этой группы ресурсов. 

1. Выберите пункт **Группа ресурсов**.
1. На странице этой группы ресурсов нажмите **Удалить**.
1. При появлении запроса введите имя группы ресурсов, которую необходимо удалить, и нажмите **Удалить**.


## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как развернуть простую виртуальную машину с помощью шаблона ARM. Дополнительные сведения о виртуальных машинах Azure см. в руководстве для виртуальных машин Linux.


> [!div class="nextstepaction"]
> [Создание виртуальных машин Windows и управление ими с помощью модуля Azure PowerShell](./tutorial-manage-vm.md)