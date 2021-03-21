---
title: Использование частных конечных точек
titleSuffix: Azure SignalR Service
description: Общие сведения о частных конечных точках для безопасного доступа к службе Azure SignalR из виртуальных сетей.
services: signalr
author: ArchangelSDY
ms.service: signalr
ms.topic: article
ms.date: 05/06/2020
ms.author: dayshen
ms.openlocfilehash: 92e93c3746308d2d6c1a489efc6b5c866b0ad2d9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98682636"
---
# <a name="use-private-endpoints-for-azure-signalr-service"></a>Использование частных конечных точек для службы SignalR Azure

Вы можете использовать [частные конечные точки](../private-link/private-endpoint-overview.md) для службы Azure SignalR, чтобы разрешить клиентам в виртуальной сети (VNet) безопасный доступ к данным по [частной ссылке](../private-link/private-link-overview.md). Частная конечная точка использует IP-адрес из адресного пространства виртуальной сети для службы Azure SignalR. Сетевой трафик между клиентами в виртуальной сети и службой SignalR Azure проходит по частной ссылке на магистральную сеть Майкрософт, что устраняет уязвимость в общедоступном Интернете.

Использование частных конечных точек для службы Azure SignalR позволяет выполнять следующие задачи:

- Защитите службу Azure SignalR с помощью управления доступом к сети, чтобы заблокировать все подключения в общедоступной конечной точке для службы Azure SignalR.
- Увеличьте уровень безопасности для виртуальной сети, позволяя блокировать утечка данных из сети.
- Безопасное подключение к службам Azure SignalR из локальных сетей, подключающихся к виртуальной сети с помощью [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) или [которыми](../expressroute/expressroute-locations.md) с частными пиринга.

## <a name="conceptual-overview"></a>Общие сведения

![Обзор частных конечных точек для службы Azure SignalR](media/howto-private-endpoints/private-endpoint-overview.png)

Частная конечная точка — это специальный сетевой интерфейс для службы Azure в [виртуальной сети](../virtual-network/virtual-networks-overview.md) (VNet). При создании частной конечной точки для службы Azure SignalR обеспечивается безопасное подключение между клиентами в виртуальной сети и службой. Частной конечной точке назначается IP-адрес из диапазона IP-адресов виртуальной сети. Подключение между частной конечной точкой и службой SignalR Azure использует защищенную закрытую ссылку.

Приложения в виртуальной сети могут легко подключаться к службе Azure SignalR через закрытую конечную точку, **используя те же строки подключения и механизмы авторизации, которые используются в противном случае**. Частные конечные точки можно использовать со всеми протоколами, поддерживаемыми службой SignalR Azure, включая REST API.

При создании частной конечной точки для службы Azure SignalR в виртуальной сети запрос согласия отправляется на утверждение владельцу службы Azure SignalR. Если пользователь, запрашивающий создание частной конечной точки, также является владельцем службы Azure SignalR, этот запрос на согласие автоматически утверждается.

Владельцы службы Azure SignalR могут управлять запросами на согласие и закрытыми конечными точками через вкладку "*частные конечные точки*" для службы Azure signalr в [портал Azure](https://portal.azure.com).

> [!TIP]
> Если вы хотите ограничить доступ к службе Azure SignalR только через закрытую конечную точку, [Настройте контроль доступа к сети](howto-network-access-control.md#managing-network-access-control) для запрета или управления доступом через общедоступную конечную точку.

### <a name="connecting-to-private-endpoints"></a>Подключение к частным конечным точкам

Клиенты в виртуальной сети, использующие частную конечную точку, должны использовать одну и ту же строку подключения для службы SignalR Azure, так как клиенты подключаются к общедоступной конечной точке. Мы будем использовать разрешение DNS для автоматического направления подключений из виртуальной сети в службу Azure SignalR по частному каналу.

> [!IMPORTANT]
> Используйте одну и ту же строку подключения для подключения к службе Azure SignalR с помощью частных конечных точек, как и в противном случае. Не подключайтесь к службе Azure SignalR, используя `privatelink` URL-адрес его поддомена.

Мы создадим [закрытую зону DNS](../dns/private-dns-overview.md) , подключенную к виртуальной сети, с необходимыми обновлениями для частных конечных точек по умолчанию. Однако если вы используете собственный DNS-сервер, может потребоваться внести дополнительные изменения в конфигурацию DNS. В разделе об [изменениях DNS](#dns-changes-for-private-endpoints) ниже описаны обновления, необходимые для частных конечных точек.

## <a name="dns-changes-for-private-endpoints"></a>Изменения DNS для частных конечных точек

При создании частной конечной точки запись ресурса DNS CNAME для службы Azure SignalR обновляется до псевдонима в поддомене с префиксом `privatelink` . По умолчанию также создается [Частная зона DNS](../dns/private-dns-overview.md), соответствующая `privatelink` поддомену, с записями ресурсов DNS a для частных конечных точек.

При разрешении доменного имени службы Azure SignalR из виртуальной сети с закрытой конечной точкой она разрешается в общедоступную конечную точку службы Azure SignalR. При разрешении из виртуальной сети, в которой размещается частная конечная точка, доменное имя разрешается в IP-адрес частной конечной точки.

В приведенном выше примере записи ресурсов DNS для службы Azure SignalR "Foobar", разрешенные за пределами виртуальной сети, в которой размещается частная конечная точка, будут:

| Имя                                                  | Type  | Значение                                                 |
| :---------------------------------------------------- | :---: | :---------------------------------------------------- |
| ``foobar.service.signalr.net``                        | CNAME. | ``foobar.privatelink.service.signalr.net``            |
| ``foobar.privatelink.service.signalr.net``            | A     | \<Azure SignalR Service public IP address\>           |

Как упоминалось ранее, вы можете запретить или контролировать доступ клиентов за пределами виртуальной сети через общедоступную конечную точку, используя контроль доступа к сети.

Записи ресурсов DNS для "Foobar" при разрешении клиентом в виртуальной сети, где размещается частная конечная точка, будут:

| Имя                                                  | Type  | Значение                                                 |
| :---------------------------------------------------- | :---: | :---------------------------------------------------- |
| ``foobar.service.signalr.net``                        | CNAME. | ``foobar.privatelink.service.signalr.net``            |
| ``foobar.privatelink.service.signalr.net``            | A     | 10.1.1.5                                              |

Такой подход обеспечивает доступ к службе Azure SignalR, **используя ту же строку подключения** для клиентов в виртуальной сети, где размещаются частные конечные точки, а также клиенты за пределами виртуальной сети.

Если в сети используется пользовательский DNS-сервер, клиенты должны иметь возможность разрешить полное доменное имя для конечной точки службы Azure SignalR на IP-адрес частной конечной точки. Необходимо настроить DNS-сервер для делегирования поддомена частной связи в частную зону DNS для виртуальной сети или настроить записи A для `foobar.privatelink.service.signalr.net` с IP-адресом частной конечной точки.

> [!TIP]
> При использовании пользовательского или локального DNS-сервера следует настроить DNS-сервер для разрешения имени службы SignalR Azure в `privatelink` поддомене на IP-адрес частной конечной точки. Это можно сделать, делегируя `privatelink` поддомен частной зоне DNS виртуальной сети, или настроив зону DNS на DNS-сервере и добавив записи A DNS.

Рекомендуемое имя зоны DNS для частных конечных точек службы SignalR Azure: `privatelink.service.signalr.net` .

Дополнительные сведения о настройке собственного DNS-сервера для поддержки частных конечных точек см. в следующих статьях:

- [Разрешение имен ресурсов в виртуальных сетях Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)
- [Конфигурация DNS для частных конечных точек](../private-link/private-endpoint-overview.md#dns-configuration)

## <a name="create-a-private-endpoint"></a>Создание частной конечной точки

### <a name="create-a-private-endpoint-along-with-a-new-azure-signalr-service-in-the-azure-portal"></a>Создайте частную конечную точку и новую службу Azure SignalR в портал Azure

1. При создании новой службы Azure SignalR выберите вкладку **сеть** . Выберите **Частная конечная точка** в качестве метода подключения.

    ![Создание службы Azure SignalR — вкладка "сеть"](media/howto-private-endpoints/portal-create-blade-networking-tab.png)

1. Нажмите кнопку **Добавить**. Заполните поля подписка, Группа ресурсов, расположение и имя для новой частной конечной точки. Выберите виртуальную сеть и подсеть.

    ![Создание службы Azure SignalR — Добавление частной конечной точки](media/howto-private-endpoints/portal-create-blade-add-private-endpoint.png)

1. Щелкните **Просмотреть и создать**.

### <a name="create-a-private-endpoint-for-an-existing-azure-signalr-service-in-the-azure-portal"></a>Создайте закрытую конечную точку для существующей службы Azure SignalR в портал Azure

1. Перейдите в службу Azure SignalR.

1. Щелкните меню Параметры **подключения к частным конечным точкам**.

1. В верхней части щелкните кнопку **+ Частная конечная точка** .

    ![Колонка "подключения частной конечной точки"](media/howto-private-endpoints/portal-private-endpoint-connections-blade.png)

1. Заполните поля подписка, Группа ресурсов, имя ресурса и регион для новой частной конечной точки.
    
    ![Создание частной конечной точки — основные сведения](media/howto-private-endpoints/portal-create-private-endpoint-basics.png)

1. Выберите целевой ресурс службы SignalR Azure.

    ![Создание частной конечной точки — ресурс](media/howto-private-endpoints/portal-create-private-endpoint-resource.png)

1. Выберите целевую виртуальную сеть

    ![Создание частной конечной точки — конфигурация](media/howto-private-endpoints/portal-create-private-endpoint-configuration.png)

1. Щелкните **Просмотреть и создать**.

### <a name="create-a-private-endpoint-using-azure-cli"></a>Создание частной конечной точки с помощью Azure CLI

1. Вход в Azure CLI
    ```azurecli
    az login
    ```
1. Выбор подписки Azure
    ```azurecli
    az account set --subscription {AZURE SUBSCRIPTION ID}
    ```
1. Создание новой группы ресурсов
    ```azurecli
    az group create -n {RG} -l {AZURE REGION}
    ```
1. Регистрация Microsoft. Сигналрсервице в качестве поставщика
    ```azurecli
    az provider register -n Microsoft.SignalRService
    ```
1. Создание новой службы Azure SignalR
    ```azurecli
    az signalr create --name {NAME} --resource-group {RG} --location {AZURE REGION} --sku Standard_S1
    ```
1. Создайте виртуальную сеть
    ```azurecli
    az network vnet create --resource-group {RG} --name {vNet NAME} --location {AZURE REGION}
    ```
1. Добавление подсети
    ```azurecli
    az network vnet subnet create --resource-group {RG} --vnet-name {vNet NAME} --name {subnet NAME} --address-prefixes {addressPrefix}
    ```
1. Отключение политик виртуальной сети
    ```azurecli
    az network vnet subnet update --name {subnet NAME} --resource-group {RG} --vnet-name {vNet NAME} --disable-private-endpoint-network-policies true
    ```
1. Добавление частной зоны DNS
    ```azurecli
    az network private-dns zone create --resource-group {RG} --name privatelink.service.signalr.net
    ```
1. Связывание частной зоны DNS с виртуальной сетью
    ```azurecli
    az network private-dns link vnet create --resource-group {RG} --virtual-network {vNet NAME} --zone-name privatelink.service.signalr.net --name {dnsZoneLinkName} --registration-enabled true
    ```
1. Создание частной конечной точки (автоматическое утверждение)
    ```azurecli
    az network private-endpoint create --resource-group {RG} --vnet-name {vNet NAME} --subnet {subnet NAME} --name {Private Endpoint Name}  --private-connection-resource-id "/subscriptions/{AZURE SUBSCRIPTION ID}/resourceGroups/{RG}/providers/Microsoft.SignalRService/SignalR/{NAME}" --group-ids signalr --connection-name {Private Link Connection Name} --location {AZURE REGION}
    ```
1. Создание частной конечной точки (запрос утверждения вручную)
    ```azurecli
    az network private-endpoint create --resource-group {RG} --vnet-name {vNet NAME} --subnet {subnet NAME} --name {Private Endpoint Name}  --private-connection-resource-id "/subscriptions/{AZURE SUBSCRIPTION ID}/resourceGroups/{RG}/providers/Microsoft.SignalRService/SignalR/{NAME}" --group-ids signalr --connection-name {Private Link Connection Name} --location {AZURE REGION} --manual-request
    ```
1. Отображение строк подключения
    ```azurecli
    az network private-endpoint show --resource-group {RG} --name {Private Endpoint Name}
    ```

## <a name="pricing"></a>Цены

Дополнительные сведения о ценах см. на [странице цен на службу "Приватный канал" Azure](https://azure.microsoft.com/pricing/details/private-link).

## <a name="known-issues"></a>Известные проблемы

Учитывайте следующие известные проблемы, связанные с частными конечными точками для службы SignalR Azure.

### <a name="free-tier"></a>Уровень служб "Бесплатный"

Вы не можете создать закрытую конечную точку для службы Azure SignalR уровня "бесплатный".

### <a name="access-constraints-for-clients-in-vnets-with-private-endpoints"></a>Ограничения доступа для клиентов в виртуальных сетей с частными конечными точками

При доступе к другим экземплярам службы SignalR, имеющим частные конечные точки, клиенты в виртуальных сетей с существующими закрытыми конечными точками имеют ограничения. Например, предположим, что у виртуальной сети N1 есть частная конечная точка для экземпляра службы Azure SignalR S1. Если служба Azure SignalR S2 имеет закрытую конечную точку в виртуальной сети N2, клиенты в виртуальной сети N1 также должны получить доступ к службе Azure SignalR S2 с помощью частной конечной точки. Если в службе Azure SignalR S2 нет частных конечных точек, клиенты в виртуальной сети N1 могут получить доступ к службе Azure SignalR в этой учетной записи без закрытой конечной точки.

Это ограничение является результатом изменений DNS, внесенных при создании частной конечной точки службой Azure SignalR S2.

### <a name="network-security-group-rules-for-subnets-with-private-endpoints"></a>Правила групп безопасности сети для сетей с частными конечными точками

Сейчас нельзя настроить правила [группы безопасности сети](../virtual-network/network-security-groups-overview.md) (NSG) и определяемые пользователем маршруты для частных конечных точек. Правила NSG, применяемые к подсети, в которой размещается частная конечная точка, применяются к частной конечной точке. Для этой проблемы ограниченный обходной путь заключается в реализации правил доступа для частных конечных точек в исходных подсетях, хотя этот подход может потребовать более высоких затрат на управление.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка контроля доступа к сети](howto-network-access-control.md)