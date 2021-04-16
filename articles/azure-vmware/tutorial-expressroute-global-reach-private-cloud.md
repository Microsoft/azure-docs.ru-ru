---
title: Учебник по настройке пиринга между локальной средой и частным облаком
description: Создание пиринга ExpressRoute Global Reach к частному облаку в решении Azure VMware.
ms.topic: tutorial
ms.date: 03/17/2021
ms.openlocfilehash: 798b822989127ccbb00e971de2cc4147ac234259
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106449568"
---
# <a name="tutorial-peer-on-premises-environments-to-a-private-cloud"></a>Руководство по Настройка пиринга между локальной средой и частным облаком

ExpressRoute Global Reach подключает локальную среду к частному облаку решения Azure VMware. Подключение ExpressRoute Global Reach устанавливается между локальными средами и каналом ExpressRoute частного облака с имеющимся подключением ExpressRoute. 

Для канала ExpressRoute, используемого при [настройке сети между частным облаком VMware в Azure](tutorial-configure-networking.md), необходимо создать и использовать ключи авторизации.  Вы уже использовали один ключ авторизации из канала ExpressRoute, поэтому для настройки пиринга с каналом ExpressRoute локальной среды вам нужно будет создать другой ключ.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * как создать второй ключ авторизации для _канала 2_ (канала ExpressRoute частного облака);
> * как использовать портал Azure или Azure CLI в Cloud Shell в подписке _канала 1_ для настройки пиринга ExpressRoute Global Reach между локальной средой и частный облаком.


## <a name="before-you-begin"></a>Перед началом

Перед включением подключения между двумя каналами ExpressRoute с помощью ExpressRoute Global Reach ознакомьтесь с документацией по [включению подключения в разных подписках Azure](../expressroute/expressroute-howto-set-global-reach-cli.md#enable-connectivity-between-expressroute-circuits-in-different-azure-subscriptions).  

## <a name="prerequisites"></a>Предварительные требования

- Установленное подключение к частному облаку решения Azure VMware и из него с помощью канала ExpressRoute, подключенного к шлюзу ExpressRoute в виртуальной сети Azure (VNet) — канал 2 из процедур пиринга.
- Отдельный работоспособный канал ExpressRoute, используемый для подключения локальных сред к Azure — канал 1 с точки зрения процедур пиринга.
- [блок неперекрывающихся сетевых адресов](../expressroute/expressroute-routing.md#ip-addresses-used-for-peerings) /29 для пиринга ExpressRoute Global Reach.
- Убедитесь, что все шлюзы, включая службу поставщика ExpressRoute, поддерживают 4-байтовые номера автономной системы (ASN). Решение Azure VMware использует 4-байтовые общедоступные номера ASN для объявления маршрутов.

>[!IMPORTANT]
>В рамках этих предварительных требований считается, что канал ExpressRoute локальной среды называется _канал 1_, а канал ExpressRoute частного облака находится в другой подписке и обозначается как _канал 2_.

## <a name="create-an-expressroute-authorization-key-in-the-private-cloud-expressroute-circuit"></a>Создание ключа авторизации ExpressRoute в канале ExpressRoute частного облака

[!INCLUDE [request-authorization-key](includes/request-authorization-key.md)]
 
## <a name="peer-private-cloud-to-on-premises-with-authorization-key"></a>Настройка пиринга между частным облаком и локальными средами с помощью ключа авторизации
Теперь, когда вы создали ключ авторизации для канала ExpressRoute частного облака, вы можете настроить его пиринг с каналом ExpressRoute локальной среды. С точки зрения локального канала ExpressRoute пиринг выполняется на **портале Azure** или с помощью **Azure CLI в Cloud Shell**. Оба этих метода предусматривают использование идентификатора ресурса и ключа авторизации для канала ExpressRoute частного облака, чтобы завершить пиринг.

### <a name="portal"></a>[Портал](#tab/azure-portal)
 
1. Войдите на [портал Azure](https://portal.azure.com), используя ту же подписку, к которой относится канал ExpressRoute локальный среды.

1. На странице **Обзор** частного облака в разделе "Управление" выберите **Подключение** > **ExpressRoute Global Reach** > **Добавить**.

    :::image type="content" source="./media/expressroute-global-reach/expressroute-global-reach-tab.png" alt-text="Снимок экрана: вкладка ExpressRoute Global Reach в частном облаке Решения Azure VMware.":::

1. Создайте подключение между локальной средой и облаком. Выполните одно из следующих действий, а затем щелкните **Создать**:

   - выберите в списке **канал ExpressRoute**;
   - Если у вас есть идентификатор канала, вставьте его в поле и укажите созданный ключ авторизации.

   :::image type="content" source="./media/expressroute-global-reach/on-premises-cloud-connections.png" alt-text="Ввод идентификатора ExpressRoute и ключа авторизации, а также выбор элемента &quot;Создать&quot;.":::   
   
   Новое подключение отобразится в списке локальных облачных подключений.

>[!TIP]
>Вы можете удалить или отключить подключение в списке, выбрав элемент **Еще**.  
>
> :::image type="content" source="./media/expressroute-global-reach/on-premises-connection-disconnect.png" alt-text="Отключение или удаление локального подключения":::

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Команды CLI](../expressroute/expressroute-howto-set-global-reach-cli.md) дополнены подробными сведениями и примерами, которые помогут настроить пиринг ExpressRoute Global Reach между локальными средами и частным облаком Решения Azure VMware.

>[!TIP]
>Чтобы сократить выходные данные команд Azure CLI и отобразить только необходимые результаты, в этих инструкциях для выполнения запроса JMESPath можно использовать [аргумент `–query`](/cli/azure/query-azure-cli).

1. Войдите на [портал Azure](https://portal.azure.com), используя ту же подписку, к которой относится канал ExpressRoute локальный среды. 

1. Откройте Cloud Shell и оставьте оболочку Bash.

   :::image type="content" source="media/expressroute-global-reach/azure-portal-cloud-shell.png" alt-text="Снимок экрана: Cloud Shell на портале Azure.":::

1. Создайте пиринг для канала 1, передав идентификатор ресурса канала 2 и ключ авторизации. 

   ```azurecli-interactive
   az network express-route peering connection create -g <ResourceGroupName> --circuit-name <Circuit1Name> --peering-name AzurePrivatePeering -n <ConnectionName> --peer-circuit <Circuit2ResourceID> --address-prefix <__.__.__.__/29> --authorization-key <authorizationKey>
   ```

   :::image type="content" source="media/expressroute-global-reach/azure-command-with-results.png" alt-text="Снимок экрана: команда и результаты успешной настройки пиринга между каналом 1 и каналом 2.":::

Теперь можно подключаться из локальных сред к частному облаку при помощи пиринга ExpressRoute Global Reach.

>[!TIP]
>Вы можете удалить пиринг, выполнив инструкции из раздела [Отключение подключения между локальными сетями](../expressroute/expressroute-howto-set-global-reach-cli.md#disable-connectivity-between-your-on-premises-networks).


---

## <a name="next-steps"></a>Дальнейшие действия

С помощь этого учебника вы узнали, как включить пиринг Global Reach ExpressRoute между локальной средой и частным облаком. 

Перейдите к следующему руководству, чтобы узнать, как развернуть и настроить решение VMware HCX для частного облака решения Azure VMware.

> [!div class="nextstepaction"]
> [Развертывание и настройка VMware HCX](tutorial-deploy-vmware-hcx.md)


<!-- LINKS - external-->

<!-- LINKS - internal -->
