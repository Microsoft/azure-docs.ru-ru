---
title: Настройка общего доступа к реестру
description: Настройте правила IP-адресов, чтобы разрешить доступ к реестру контейнеров Azure из выбранных общедоступных IP-адресов или диапазонов адресов.
ms.topic: article
ms.date: 03/08/2021
ms.openlocfilehash: 727aa1dc028b5f52a022e54c2cd252ae372e78fe
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104773068"
---
# <a name="configure-public-ip-network-rules"></a>Настройка сетевых правил для общедоступных IP-адресов

По умолчанию реестр контейнеров Azure принимает подключения через Интернет от узлов в любой сети. В этой статье показано, как настроить для реестра контейнеров разрешение на доступ только с конкретных общедоступных IP-адресов или диапазонов адресов. Приводятся инструкции для портала Azure и Azure CLI.

Правила IP-сети настраиваются на общедоступной конечной точке реестра. Правила IP-сети не применяются к частным конечным точкам, настроенным для [Приватного канала](container-registry-private-link.md).

Настройка правил доступа с IP-адресов доступна на уровне службы **Премиум** реестра контейнеров. Сведения об уровнях служб реестра и ограничениях см. в статье [Уровни служб Реестра контейнеров Azure](container-registry-skus.md).

Каждый реестр поддерживает не более 100 правил сетевого доступа.

[!INCLUDE [container-registry-scanning-limitation](../../includes/container-registry-scanning-limitation.md)]

## <a name="access-from-selected-public-network---cli"></a>Доступ из выбранной общедоступной сети (интерфейс командной строки)

### <a name="change-default-network-access-to-registry"></a>Изменение сетевого доступа по умолчанию для реестра

Чтобы разрешить доступ только из выбранной общедоступной сети, необходимо прежде всего настроить режим запрета доступа по умолчанию. Подставьте нужное имя реестра в следующей команде [az acr update][az-acr-update]:

```azurecli
az acr update --name myContainerRegistry --default-action Deny
```

### <a name="add-network-rule-to-registry"></a>Добавление правил сети для реестра

Выполните команду [az acr network-rule add][az-acr-network-rule-add], чтобы добавить в реестр правило сетевого доступа, которое разрешает доступ с общедоступного IP-адреса или из диапазона адресов. Например, подставьте здесь нужное имя реестра контейнеров и общедоступный IP-адрес виртуальной машины в виртуальной сети.

```azurecli
az acr network-rule add \
  --name mycontainerregistry \
  --ip-address <public-IP-address>
```

> [!NOTE]
> Новое правило вступает в силу через несколько минут после добавления.

## <a name="access-from-selected-public-network---portal"></a>Доступ из выбранной общедоступной сети (портал)

1. На портале перейдите к нужному реестру контейнеров.
1. В разделе **Параметры** выберите **Сеть**.
1. На вкладке **Открытый доступ** включите разрешение открытого доступа из **выбранных сетей**.
1. В разделе **Брандмауэр** введите общедоступный IP-адрес, например адрес виртуальной машины в виртуальной сети. Или введите диапазон адресов, который содержит нужный IP-адрес виртуальной машины, в нотации CIDR.
1. Щелкните **Сохранить**.

![Настройка правила брандмауэра для реестра контейнеров][acr-access-selected-networks]

> [!NOTE]
> Новое правило вступает в силу через несколько минут после добавления.

> [!TIP]
> Вы также можете включить доступ к реестру с локального клиентского компьютера или из диапазона локальных IP-адресов. Чтобы разрешить такой доступ, вам нужно знать общедоступный IPv4-адрес компьютера. Его можно узнать, просто выполнив поиск по фразе "мой IP-адрес" в Интернет-браузере. Также актуальный IPv4-адрес клиента автоматически отображается при настройке параметров брандмауэра на странице **Сеть** на портале.

## <a name="disable-public-network-access"></a>Отключение доступа из общедоступной сети

При желании вы можете совсем отключить общедоступную конечную точку для реестра. Отключение общедоступной конечной точки переопределяет все конфигурации брандмауэра. Например, вы можете отключить общий доступ к реестру, защищенному в виртуальной сети, с помощью [Приватного канала](container-registry-private-link.md).

> [!NOTE]
> Если реестр настроен в виртуальной сети с [конечной точкой службы](container-registry-vnet.md), отключение доступа к общедоступной конечной точке реестра также приведет к отключению доступа к реестру в виртуальной сети.

### <a name="disable-public-access---cli"></a>Отключение общего доступа (интерфейс командной строки)

Чтобы отключить общий доступ с помощью Azure CLI, воспользуйтесь командой [az acr update][az-acr-update] и задайте для `--public-network-enabled` значение `false`. Для использования аргумента `public-network-enabled` нужна версия Azure CLI не ниже 2.6.0. 

```azurecli
az acr update --name myContainerRegistry --public-network-enabled false
```

### <a name="disable-public-access---portal"></a>Отключение общего доступа (портал)

1. На портале перейдите к нужному реестру контейнеров, затем выберите **Параметры > Сеть**.
1. На вкладке **Открытый доступ** выберите для параметра **Разрешение доступа из общедоступных сетей** значение **Отключено**. Затем нажмите кнопку **Save** (Сохранить).

![Отключение общего доступа][acr-access-disabled]


## <a name="restore-public-network-access"></a>Восстановление доступа из общедоступной сети

Чтобы восстановить общедоступную конечную точку, измените параметры сети, чтобы разрешить общий доступ. Включение общедоступной конечной точки переопределяет все конфигурации брандмауэра. 

### <a name="restore-public-access---cli"></a>Восстановление общего доступа (интерфейс командной строки)

Выполните команду [az acr update][az-acr-update] и присвойте параметру `--public-network-enabled` значение `true`. 

> [!NOTE]
> Для использования аргумента `public-network-enabled` нужна версия Azure CLI не ниже 2.6.0. 

```azurecli
az acr update --name myContainerRegistry --public-network-enabled true
```

### <a name="restore-public-access---portal"></a>Восстановление общего доступа (портал)

1. На портале перейдите к нужному реестру контейнеров, затем выберите **Параметры > Сеть**.
1. На вкладке **Открытый доступ** выберите для параметра **Разрешение доступа из общедоступных сетей** значение **Все сети**. Затем нажмите кнопку **Save** (Сохранить).

![Общий доступ из всех сетей][acr-access-all-networks]

## <a name="troubleshoot"></a>Диагностика

Если задано правило общедоступной сети или запрещен общий доступ к реестру, попытки входа в реестр из неразрешенной общедоступной сети завершатся сбоем. Клиентский доступ через прокси-сервер HTTPS также завершится ошибкой, если не задано правило доступа для прокси-сервера. Вы увидите сообщение об ошибке, аналогичное `Error response from daemon: login attempt failed with status: 403 Forbidden` или `Looks like you don't have access to registry` .

Эти ошибки также могут возникать, если используется прокси-сервер HTTPS, разрешенный правилом доступа к сети, но прокси-сервер в клиентской среде не настроен должным образом. Убедитесь, что клиент DOCKER и управляющая программа DOCKER настроены для поведения прокси-сервера. Дополнительные сведения см. в разделе [http/https proxy](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy) в документации по DOCKER.


## <a name="next-steps"></a>Дальнейшие действия

* Чтобы ограничить доступ к реестру, используя частную конечную точку в виртуальной сети, см. статью [Настройка Приватного канала Azure для реестра контейнеров Azure](container-registry-private-link.md).
* Если вам нужно настроить правила для доступа к реестру, расположенному за клиентским брандмауэром, воспользуйтесь руководством [Настройка правил для доступа к реестру контейнеров Azure за брандмауэром](container-registry-firewall-access-rules.md).

[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-network-rule-add]: /cli/azure/acr/network-rule/#az-acr-network-rule-add
[az-acr-network-rule-remove]: /cli/azure/acr/network-rule/#az-acr-network-rule-remove
[az-acr-network-rule-list]: /cli/azure/acr/network-rule/#az-acr-network-rule-list
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-acr-update]: /cli/azure/acr#az-acr-update
[quickstart-portal]: container-registry-get-started-portal.md
[quickstart-cli]: container-registry-get-started-azure-cli.md
[azure-portal]: https://portal.azure.com

[acr-access-selected-networks]: ./media/container-registry-access-selected-networks/acr-access-selected-networks.png
[acr-access-disabled]: ./media/container-registry-access-selected-networks/acr-access-disabled.png
[acr-access-all-networks]: ./media/container-registry-access-selected-networks/acr-access-all-networks.png
