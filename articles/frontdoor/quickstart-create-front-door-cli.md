---
title: Краткое руководство. Настройка высокого уровня доступности в Azure Front Door — Azure CLI
description: В этом кратком руководстве показано, как использовать Azure Front Door для создания высокодоступного и высокопроизводительного глобального веб-приложения с помощью Azure CLI.
services: front-door
author: duongau
manager: KumudD
ms.assetid: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/21/2020
ms.author: duau
ms.openlocfilehash: a64c91910ba65901a6d1374df9633062398a90e4
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106067662"
---
# <a name="quickstart-create-a-front-door-for-a-highly-available-global-web-application-using-azure-cli"></a>Краткое руководство. Создание Front Door для глобального веб-приложения высокой доступности с помощью Azure CLI

Начните работу с Azure Front Door, используя Azure CLI для создания высокодоступного и высокопроизводительного глобального веб-приложения.

Front Door направляет веб-трафик к конкретным ресурсам в серверном пуле. Определив интерфейсный домен, добавьте ресурсы в серверный пул и создайте правило маршрутизации. В этой статье используется простая конфигурация одного серверного пула с двумя ресурсами веб-приложения и одним правилом маршрутизации с использованием согласования путей по умолчанию "/*".

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Локальная установка Azure CLI или Azure Cloud Shell
- Убедитесь, что в Azure CLI добавлено расширение front-door

```azurecli-interactive 
az extension add --name front-door
```

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать CLI локально, для выполнения инструкций из этого руководства вам потребуется Azure CLI версии 2.0.28 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

В Azure выделите связанные ресурсы группе ресурсов. Вы можете выбрать существующую группу ресурсов или создать новую.

Для целей этого краткого руководства необходимы две группы ресурсов. Один в *центральной части США* и второй — в *центрально-южной части США*.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create):

```azurecli-interactive
az group create \
    --name myRGFDCentral \
    --location centralus

az group create \
    --name myRGFDSouthCentral \
    --location southcentralus
```

## <a name="create-two-instances-of-a-web-app"></a>Создание двух экземпляров веб-приложения

Для этого краткого руководства потребуются два экземпляра веб-приложения, которые выполняются в разных регионах Azure. Оба экземпляра этого веб-приложения работают в режиме активный — активный, то есть любой из них будет принимать трафик службы.

Если у вас нет веб-приложения, создайте два его экземпляра, используя показанный ниже скрипт.

### <a name="create-app-service-plans"></a>Создание планов службы приложений

Прежде чем создавать веб-приложения, вам потребуется два плана службы приложений: один в *центральной части США*, а второй — в *центрально-южной части США*.

Для этого выполните команду [az appservice plan create](/cli/azure/appservice/plan#az_appservice_plan_create&preserve-view=true).

```azurecli-interactive
az appservice plan create \
--name myAppServicePlanCentralUS \
--resource-group myRGFDCentral

az appservice plan create \
--name myAppServicePlanSouthCentralUS \
--resource-group myRGFDSouthCentral
```

### <a name="create-web-apps"></a>Создание веб-приложений

При выполнении следующих команд в каждом из планов службы приложений, упомянутых на предыдущем шаге, будет создано веб-приложение. Имена веб-приложений должны быть глобально уникальными.

Создайте веб-приложение с помощью команды [az webapp create](/cli/azure/webapp#az_webapp_create&preserve-view=true):

```azurecli-interactive
az webapp create \
--name WebAppContoso1 \
--resource-group myRGFDCentral \
--plan myAppServicePlanCentralUS 

az webapp create \
--name WebAppContoso2 \
--resource-group myRGFDSouthCentral \
--plan myAppServicePlanSouthCentralUS
```

Запишите имя узла по умолчанию для каждого веб-приложения, чтобы можно было определить внутренние адреса при развертывании Front Door на следующем шаге.

## <a name="create-the-front-door"></a>Создание Front Door

Создайте базовый экземпляр Front Door с параметрами балансировки нагрузки, проверкой работоспособности и правилами маршрутизации по умолчанию, выполнив следующую команду:

Создайте Front Door с помощью команды [az network front-door create](/cli/azure/ext/front-door/network/front-door#ext_front_door_az_network_front_door_create&preserve-view=true):

```azurecli-interactive
az network front-door create \
--resource-group myRGFDCentral \
--name contoso-frontend \
--accepted-protocols http https \
--backend-address webappcontoso1.azurewebsites.net webappcontoso2.azurewebsites.net 
```

**--resource-group:** Укажите группу ресурсов, в которой требуется развернуть Front Door.

**--name:** Укажите глобально уникальное имя для Azure Front Door. 

**--accepted-protocols:** Допустимые значения: **http** и **https**. Если вы хотите использовать оба варианта, они должны быть разделены пробелом.

**--backend-address:** Определите оба имени узла веб-приложений, разделенные пробелом.

После успешного завершения развертывания запишите имя узла в разделе *frontEndpoints*.

## <a name="test-the-front-door"></a>Тестирование Front Door

Откройте веб-браузер и введите имя узла, полученное из команд. Front Door направляет запрос к одному из внутренних ресурсов.

:::image type="content" source="./media/quickstart-create-front-door-cli/front-door-testing-page.png" alt-text="Страница тестирования Front Door":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с помощью Front Door, удалите обе группы ресурсов. Удалив ее, вы также удалите Front Door и все связанные с ним ресурсы. 

Чтобы удалить группу ресурсов, выполните команду [az group delete](/cli/azure/group#az_group_delete&preserve-view=true).

```azurecli-interactive
az group delete \
--name myRGFDCentral 

az group delete \
--name myRGFDSouthCentral
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать:
* Front Door
* два веб-приложения.

Сведения о добавлении личного домена в Front Door см. в следующих учебниках.

> [!div class="nextstepaction"]
> [Добавить личный домен](front-door-custom-domain.md)
