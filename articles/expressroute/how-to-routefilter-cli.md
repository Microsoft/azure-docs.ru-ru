---
title: Руководство по Настройка фильтров маршрутов для пиринга Майкрософт с помощью Azure CLI
description: В этом руководстве показано, как настраивать фильтры маршрутов для пиринга Майкрософт с помощью Azure CLI.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/08/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: ac7fc5af21f11699331d41a074e88ae757170664
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91976000"
---
# <a name="tutorial-configure-route-filters-for-microsoft-peering-azure-cli"></a>Руководство по Настройка фильтров маршрутов для пиринга Майкрософт: Azure CLI

> [!div class="op_single_selector"]
> * [портале Azure](how-to-routefilter-portal.md)
> * [Azure PowerShell](how-to-routefilter-powershell.md)
> * [Azure CLI](how-to-routefilter-cli.md)
> 

Фильтры маршрутов — это способ использовать подмножество поддерживаемых служб через пиринг Майкрософт. Действия, описанные в этой статье, помогут настроить фильтры маршрутов для каналов ExpressRoute и управлять ими.

Службы Microsoft 365, такие как Exchange Online, SharePoint Online и Skype для бизнеса, доступны через пиринг Майкрософт. При настройке пиринга Майкрософт в канале ExpressRoute все префиксы, которые относятся к этим службам, объявляются через установленные сеансы BGP. Значение сообщества BGP подключается к каждому префиксу для идентификации службы, предлагаемой через него. Список значений сообщества BGP и служб, с которыми они сопоставляются, см. в разделе [Поддержка сообществ BGP](expressroute-routing.md#bgp).

Подключение ко всем службам Azure и Microsoft 365 приводит к объявлению большого количества префиксов через BGP. Это значительно увеличивает размер таблиц маршрутов, которые обслуживают маршрутизаторы в вашей сети. Если вы планируете использовать только подмножество служб, предлагаемых через пиринг Майкрософт, размер таблиц маршрутизации можно уменьшить двумя способами. Вы можете:

* Отфильтровывать нежелательные префиксы путем применения фильтров маршрутов к сообществам BGP. Это стандартный подход, который часто используется при работе со многими сетями.

* Определять фильтры маршрутов и применять их к каналу ExpressRoute. Фильтр маршрутов — это новый ресурс, который позволяет выбрать список служб, которые вы собираетесь использовать через пиринг Майкрософт. Маршрутизаторы ExpressRoute отправляют только список префиксов, которые принадлежат службам, определяемым в фильтре маршрутов.

В этом руководстве описано следующее:
> [!div class="checklist"]
> - получение значений сообщества BGP;
> - создание фильтра маршрутов и правила фильтрации;
> - связывание фильтра маршрутов с каналом ExpressRoute.

### <a name="about-route-filters"></a><a name="about"></a>О фильтрах маршрута

При настроенном пиринге Майкрософт для канала ExpressRoute пограничные маршрутизаторы Майкрософт устанавливают пару сеансов BGP с пограничными маршрутизаторами через вашего поставщика услуг подключения. В вашей сети маршруты не объявляются. Чтобы включить объявление маршрутов в своей сети, необходимо связать с ней фильтр маршрутов.

Фильтр маршрутов позволяет вам идентифицировать службы, которые можно использовать через пиринг Майкрософт по каналу ExpressRoute. Это фактически список разрешений для всех значений сообщества BGP. После определения ресурса фильтра маршрутов и его подключения к каналу ExpressRoute все префиксы, которые сопоставляются со значениями сообщества BGP, объявляются в сети.

Чтобы подключить фильтры маршрутов к службам Microsoft 365, у вас должно быть разрешение на использование этих служб через ExpressRoute. Если у вас нет такого разрешения, подключение фильтров маршрутов завершится сбоем. Дополнительные сведения о получении разрешения см. в статье [Azure ExpressRoute для Microsoft 365](/microsoft-365/enterprise/azure-expressroute).

> [!IMPORTANT]
> Пиринг Майкрософт для каналов ExpressRoute, которые были настроены до 1 августа 2017 г., позволяет объявлять все префиксы служб, даже если не настроены фильтры маршрутов. Пиринг Майкрософт для каналов ExpressRoute, настроенных 1 августа 2017 г. или позднее, не будет объявлять префиксы, пока к каналу не будет присоединен фильтр маршрутов.
> 

## <a name="prerequisites"></a>Предварительные требования

Чтобы успешно подключиться к службам через пиринг Майкрософт, необходимо выполнить следующие действия по настройке:

* Необходимо иметь активный канал ExpressRoute с подготовленным пирингом Майкрософт. Для выполнения этих задач можно использовать следующие инструкции:
  * [Создайте канал ExpressRoute](howto-circuit-cli.md). Он должен быть включен на стороне поставщика услуг подключения. Канал ExpressRoute должен находиться в подготовленном и включенном состоянии.
  * [Создайте пиринг Майкрософт](howto-routing-cli.md), если вы управляете сеансом BGP напрямую или если у вашего поставщика услуг подключения подготовлен пиринг Майкрософт для вашего канала.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

Если вы решили установить и использовать CLI локально, для выполнения инструкций из этого руководства вам потребуется Azure CLI версии 2.0.28 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Войдите в учетную запись Azure и выберите подписку.

Чтобы начать настройку, войдите в свою учетную запись Azure. Если вы воспользовались кнопкой "Попробовать", вы вошли в систему автоматически и можете не подтверждать вход в учетную запись. Для подключения используйте следующие примеры:

```azurecli
az login
```

Просмотрите подписки учетной записи.

```azurecli-interactive
az account list
```

Выберите подписку, для которой требуется создать канал ExpressRoute.

```azurecli-interactive
az account set --subscription "<subscription ID>"
```

## <a name="get-a-list-of-prefixes-and-bgp-community-values"></a><a name="prefixes"></a>Получение списка префиксов и значений сообщества BGP

1. Чтобы получить список значений сообщества BGP и префиксов, связанных со службами, которые доступны через пиринг Майкрософт, используйте следующий командлет:

    ```azurecli-interactive
    az network route-filter rule list-service-communities
    ```

1. Создайте список значений сообщества BGP, которые нужно использовать в фильтре маршрута.

## <a name="create-a-route-filter-and-a-filter-rule"></a><a name="filter"></a>Создание фильтра маршрута и правила фильтрации

Фильтр может иметь только одно правило, и оно должно иметь тип "Разрешить". С этим правилом может быть связан список значений сообщества BGP. Команда `az network route-filter create` позволяет создать только ресурс фильтра маршрутов. После создания ресурса нужно создать правило и подключить его к объекту фильтра маршрута.

1. Чтобы создать ресурс фильтра маршрутов, выполните следующую команду:

    ```azurecli-interactive
    az network route-filter create -n MyRouteFilter -g MyResourceGroup
    ```

1. Чтобы создать правило фильтрации, выполните следующую команду:
 
    ```azurecli-interactive
    az network route-filter rule create --filter-name MyRouteFilter -n CRM --communities 12076:5040 --access Allow -g MyResourceGroup
    ```

## <a name="attach-the-route-filter-to-an-expressroute-circuit"></a><a name="attach"></a>Подключения фильтра маршрута к каналу ExpressRoute

Чтобы подключить фильтр маршрутов к каналу ExpressRoute, выполните следующую команду:

```azurecli-interactive
az network express-route peering update --circuit-name MyCircuit -g ExpressRouteResourceGroupName --name MicrosoftPeering --route-filter MyRouteFilter
```

## <a name="common-tasks"></a><a name="tasks"></a>Стандартные задачи

### <a name="to-get-the-properties-of-a-route-filter"></a><a name="getproperties"></a>Получение свойств фильтра маршрута

Чтобы получить свойства фильтра маршрута, выполните следующую команду.

```azurecli-interactive
az network route-filter show -g ExpressRouteResourceGroupName --name MyRouteFilter 
```

### <a name="to-update-the-properties-of-a-route-filter"></a><a name="updateproperties"></a>Обновление свойств фильтра маршрутов

Если фильтр маршрутов уже подключен к каналу, изменения объявлений префикса автоматически распространятся в списке сообщества BGP в рамках установленных сеансов BGP. Вы можете обновить список сообщества BGP вашего фильтра маршрута с помощью следующей команды:

```azurecli-interactive
az network route-filter rule update --filter-name MyRouteFilter -n CRM -g ExpressRouteResourceGroupName --add communities '12076:5040' --add communities '12076:5010'
```

### <a name="to-detach-a-route-filter-from-an-expressroute-circuit"></a><a name="detach"></a>Отсоединение фильтра маршрутов от канала ExpressRoute

Как только фильтр маршрутов отсоединяется от канала ExpressRoute, префиксы перестают объявляться через сеанс BGP. Фильтр маршрутов можно отсоединить от канала ExpressRoute с помощью следующей команды:

```azurecli-interactive
az network express-route peering update --circuit-name MyCircuit -g ExpressRouteResourceGroupName --name MicrosoftPeering --remove routeFilter
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Фильтр маршрутов можно удалить, только если он не подключен к каналу. Убедитесь, что фильтр маршрутов не подключен к каналу, прежде чем пытаться удалить его. Фильтр маршрутов можно удалить, используя следующую команду:

```azurecli-interactive
az network route-filter delete -n MyRouteFilter -g MyResourceGroup
```

## <a name="next-steps"></a>Next Steps

Дополнительные сведения о примерах конфигурации маршрутизаторов см. в следующих статьях:

> [!div class="nextstepaction"]
> [Примеры конфигурации маршрутизатора для настройки и управления маршрутизацией](expressroute-config-samples-routing.md)
