---
title: Руководство. Развертывание частного облака Решения Azure VMware
description: Сведения о том, как создать и развернуть частное облако Решения Azure VMware.
ms.topic: tutorial
ms.date: 02/22/2021
ms.openlocfilehash: 89a44ce7e5910609068f72c321971ced2e3646b4
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107374851"
---
# <a name="tutorial-deploy-an-azure-vmware-solution-private-cloud"></a>Руководство. Развертывание частного облака Решения Azure VMware

Решение Azure VMware позволяет развернуть в Azure кластер vSphere. Минимальное первоначальное развертывание должно включать три узла. Далее можно по одному добавлять дополнительные узлы, но не более 16 узлов на кластер.

Так как Решение Azure VMware не позволяет при запуске управлять частным облаком из локального экземпляра vCenter, нужно выполнить дополнительную настройку. Такие процедуры и соответствующие требования описаны в этом учебнике.

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание частного облака Решения Azure VMware
> * проверка развертывания частного облака.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Соответствующие права администратора и разрешение на создание частного облака. У вас должны быть разрешения уровня "Участник" или выше для подписки.
- Используйте сведения, которые вы собрали при изучении статьи по [планированию](production-ready-deployment-steps.md) для развертывания Решения Azure VMware.
- Убедитесь, что настроены необходимые сетевые подключения, как описано в статье [Руководство. Контрольный список для сети](tutorial-network-checklist.md).
- Узлы были подготовлены, а поставщик ресурсов Microsoft.AVS зарегистрирован, как описано в статье [Запрос квоты узлов и включение Решения Azure VMware](enable-azure-vmware-solution.md).

## <a name="create-a-private-cloud"></a>Создание частного облака

Вы можете создать частное облако Решения Azure VMware с помощью [портала Azure](#azure-portal) или [Azure CLI](#azure-cli).

### <a name="azure-portal"></a>Портал Azure

[!INCLUDE [create-avs-private-cloud-azure-portal](includes/create-private-cloud-azure-portal-steps.md)]

### <a name="azure-cli"></a>Azure CLI

Вместо создания частного облака Решения Azure VMware на портале Azure можно использовать Azure CLI в Azure Cloud Shell.  Список команд, которые можно использовать с Решением Azure VMware, см. в статье [Команды Azure VMware](/cli/azure/ext/vmware/vmware).

#### <a name="open-azure-cloud-shell"></a>Открытие Azure Cloud Shell

Нажмите кнопку **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/bash](https://shell.azure.com/bash). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу **ВВОД**, чтобы выполнить его.

#### <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive

az group create --name myResourceGroup --location eastus
```

#### <a name="create-a-private-cloud"></a>Создание частного облака

Укажите имя для группы ресурсов и частного облака, расположение и размер кластера.

| Свойство  | Описание  |
| --------- | ------------ |
| **-g** — имя группы ресурсов.     | Имя группы ресурсов для ресурсов частного облака.        |
| **-n** — имя частного облака.     | Имя частного облака Решения Azure VMware.        |
| **--location**     | Расположение для частного облака.         |
| **--cluster-size**     | Размер кластера. Минимальное значение — 3.         |
| **--network-block**     | IP-адрес блока сети CIDR, используемый для частного облака. Блок адресов не должен пересекаться с блоками адресов, используемыми в других виртуальных сетях, находящихся в вашей подписке и локальных сетях.        |
| **--sku** | Значение SKU: AV36 |

```azurecli-interactive
az vmware private-cloud create -g myResourceGroup -n myPrivateCloudName --location eastus --cluster-size 3 --network-block xx.xx.xx.xx/22 --sku AV36
```

## <a name="azure-vmware-commands"></a>Команды Azure VMware

Список команд, которые можно использовать с Решением Azure VMware, см. в статье [Команды Azure VMware](/cli/azure/ext/vmware/vmware).

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание частного облака Решения Azure VMware
> * проверка развертывания частного облака.
> * Удаление частного облака Решения Azure VMware

Перейдите к следующему учебнику, чтобы узнать, как создать переходную среду. Вы будете использовать переходную среду для подключения к своей среде, чтобы вы могли управлять частным облаком локально.


> [!div class="nextstepaction"]
> [Доступ к частному облаку Решения Azure VMware](tutorial-access-private-cloud.md)
