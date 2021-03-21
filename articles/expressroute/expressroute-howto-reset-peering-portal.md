---
title: 'Azure ExpressRoute: сброс пиринга цепи с помощью портал Azure'
description: Узнайте, как отключить и включить пирингы канала Azure ExpressRoute с помощью портал Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 11/30/2020
ms.author: duau
ms.openlocfilehash: 432ecedbbb8965926499380eb1165fdf43018426
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97680275"
---
# <a name="reset-expressroute-circuit-peerings-by-using-the-azure-portal"></a>Сброс пиринга канала ExpressRoute с помощью портал Azure

В этой статье описывается, как отключить и включить пирингы канала Azure ExpressRoute с помощью портал Azure. При отключении пиринга выполняется завершение сеанса протокол BGP (BGP) для основного и дополнительного подключений канала ExpressRoute. При включении пиринга выполняется восстановление сеанса BGP на основном и дополнительном подключении канала ExpressRoute.

> [!Note]
> При первой настройке пиринга в канале ExpressRoute пиринг по умолчанию включен.

Сброс параметров пиринга ExpressRoute может быть полезен в следующих сценариях:

* Вы тестируете проект и реализацию аварийного восстановления. Например, предположим, что у вас есть две цепи ExpressRoute. Вы можете отключить одноранговые узлы и принудительно настроить сетевой трафик для использования другого канала.

* Вы хотите включить обнаружение двунаправленной переадресации (БФД) для частного пиринга Azure или пиринга Майкрософт. Если ваш канал ExpressRoute был создан до 1 августа 2018, на частном одноранговом пиринга Azure или до 10 января 2020 на пиринг Майкрософт, БФД не был включен по умолчанию. Сбросьте пиринг, чтобы включить БФД.

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

В браузере перейдите к [портал Azure](https://portal.azure.com), а затем выполните вход с помощью учетной записи Azure.

## <a name="reset-a-peering"></a>Сброс пиринга

Вы можете сбросить пиринг Майкрософт и частный пиринг Azure в канале ExpressRoute независимо друг от друга.

1. Выберите цепь, которую необходимо изменить.

    :::image type="content" source="./media/expressroute-howto-reset-peering-portal/expressroute-circuit-list.png" alt-text="Снимок экрана, показывающий выбор канала в списке канала ExpressRoute.":::

1. Выберите конфигурацию пиринга, которую требуется сбросить.

    :::image type="content" source="./media/expressroute-howto-reset-peering-portal/expressroute-circuit.png" alt-text="Снимок экрана, на котором показано, как выбрать пиринг в обзоре канала ExpressRoute.":::

1. Снимите флажок **включить пиринг** и нажмите кнопку **сохранить** , чтобы отключить конфигурацию пиринга.

    :::image type="content" source="./media/expressroute-howto-reset-peering-portal/disable-peering.png" alt-text="Снимок экрана, на котором показан сброс флажка &quot;включить пиринг&quot;.":::

1. Установите флажок **включить пиринг** и нажмите кнопку **сохранить** , чтобы снова включить конфигурацию пиринга.

## <a name="next-steps"></a>Дальнейшие действия

Сведения об устранении неполадок ExpressRoute см. в следующих статьях:

* [Проверка подключения ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* [Устранение проблем с производительностью сети](expressroute-troubleshooting-network-performance.md)
