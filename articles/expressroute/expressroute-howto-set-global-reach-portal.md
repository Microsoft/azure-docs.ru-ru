---
title: 'Azure ExpressRoute: Настройка Global Reach с помощью портал Azure'
description: Эта статья поможет вам связать каналы ExpressRoute вместе, чтобы создать частную сеть между локальными сетями и включить Global Reach с помощью портал Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 03/05/2021
ms.author: duau
ms.openlocfilehash: 336bd4aaf881b7315921ef374c92a2ac95ff3c8c
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102431321"
---
# <a name="configure-expressroute-global-reach-using-the-azure-portal"></a>Настройка Global Reach ExpressRoute с помощью портал Azure

Эта статья поможет вам настроить ExpressRoute Global Reach с помощью PowerShell. Дополнительные сведения см. в разделе [Связывание каналов ExpressRoute для включения ExpressRoute Global Reach (предварительная версия)](expressroute-global-reach.md).

 ## <a name="before-you-begin"></a>Подготовка к работе

Прежде чем начать настройку, подтвердите следующие критерии.

* Вы понимаете [рабочие процессы](expressroute-workflows.md)подготовки канала ExpressRoute.
* Каналы ExpressRoute находятся в подготовленном состоянии.
* Частный пиринг Azure настроен для каналов ExpressRoute.
* Если вы хотите запустить PowerShell локально, убедитесь, что на компьютере установлена последняя версия Azure PowerShell.

## <a name="identify-circuits"></a>Выявление цепей

1. В браузере откройте [портал Azure](https://portal.azure.com) и выполните вход с помощью учетной записи Azure.

2. Укажите каналы ExpressRoute, которые вы хотите использовать. Вы можете включить ExpressRoute Global Reach между частным пирингом двух каналов ExpressRoute, если они находятся в поддерживаемых странах и регионах. Каналы должны быть созданы в разных расположениях пиринга. 

   * Если подписке принадлежат оба канала, вы можете выбрать канал для выполнения конфигурации в следующих разделах.
   * Если два канала находятся в разных подписках Azure, может потребоваться выполнить авторизацию из одной подписки Azure. Затем необходимо передать ключ авторизации при выполнении команды конфигурации в другой подписке Azure.

    :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/expressroute-circuit-global-reach-list.png" alt-text="Снимок экрана со списком каналов ExpressRoute.":::

## <a name="enable-connectivity"></a>Включение соединения

Обеспечение подключения между локальными сетями. Существуют отдельные наборы инструкций для каналов, которые находятся в одной подписке Azure, и цепи, которые являются разными подписками.

### <a name="expressroute-circuits-in-the-same-azure-subscription"></a>Каналы ExpressRoute в одной подписке Azure

1. Выберите конфигурацию **частного пиринга Azure** . 

    :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/expressroute-circuit-private-peering.png" alt-text="Снимок экрана со страницей обзора ExpressRoute.":::

1. Щелкните **добавить Global REACH** , чтобы открыть страницу *Добавление конфигурации Global REACH* .

    :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/private-peering-enable-global-reach.png" alt-text="Включить глобальный доступ от частного пиринга":::

1. На странице *Добавление конфигурации Global REACH* присвойте имя этой конфигурации. Выберите *канал ExpressRoute* , к которому нужно подключить этот канал, и введите адрес **IPv4** для *Global REACH подсети*. Мы используем IP-адреса в этой подсети, чтобы установить подключение между двумя каналами ExpressRoute. Не используйте адреса из этой подсети в виртуальных сетях Azure или в локальной сети. Нажмите кнопку **Добавить** , чтобы добавить канал в конфигурацию частного пиринга.

    :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/add-global-reach-configuration.png" alt-text="Снимок экрана добавления Global Reach в частный пиринг.":::

1. Нажмите кнопку **сохранить** , чтобы завершить настройку Global REACH. По завершении операции вы получите подключение между двумя локальными сетями с помощью обеих каналов ExpressRoute.

    :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/save-private-peering-configuration.png" alt-text="Снимок экрана: сохранение конфигураций частных пиринга.":::

### <a name="expressroute-circuits-in-different-azure-subscriptions"></a>Каналы ExpressRoute в разных подписках Azure

Если две цепи находятся не в одной подписке Azure, вам потребуется авторизация. В следующей конфигурации авторизация создается из подписки канала 2. Затем ключ авторизации передается цепи 1.

1. Создайте ключ авторизации.

   :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/create-authorization-expressroute-circuit.png" alt-text="Снимок экрана создания ключа авторизации."::: 

   Запишите идентификатор ресурса канала 2 и ключ авторизации.

1. Выберите конфигурацию **частного пиринга Azure** . 

    :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/expressroute-circuit-private-peering.png" alt-text="Снимок экрана частного пиринга на странице обзора.":::

1. Щелкните **добавить Global REACH** , чтобы открыть страницу *Добавление конфигурации Global REACH* .

    :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/private-peering-enable-global-reach.png" alt-text="Снимок экрана добавления Global Reach в частном пиринга.":::

1. На странице *Добавление конфигурации Global REACH* присвойте имя этой конфигурации. Установите флажок **авторизации активации** . Введите **ключ авторизации** и **идентификатор канала ExpressRoute** , созданный и полученный на шаге 1. Затем укажите **/29 IPv4** для *подсети Global REACH*. Мы используем IP-адреса в этой подсети, чтобы установить подключение между двумя каналами ExpressRoute. Не используйте адреса из этой подсети в виртуальных сетях Azure или в локальной сети. Нажмите кнопку **Добавить** , чтобы добавить канал в конфигурацию частного пиринга.

    :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/add-global-reach-configuration-with-authorization.png" alt-text="Снимок экрана: Добавление Global Reach с ключом авторизации.":::

1. Нажмите кнопку **сохранить** , чтобы завершить настройку Global REACH. По завершении операции вы получите подключение между двумя локальными сетями с помощью обеих каналов ExpressRoute.

    :::image type="content" source="./media/expressroute-howto-set-global-reach-portal/save-private-peering-configuration.png" alt-text="Снимок экрана: сохранение конфигурации частного пиринга с помощью Global Reach.":::

## <a name="verify-the-configuration"></a>Проверка конфигурации

Проверьте конфигурацию Global Reach, выбрав *частный пиринг* в конфигурации канала ExpressRoute. При правильной настройке конфигурация должна выглядеть следующим образом:

:::image type="content" source="./media/expressroute-howto-set-global-reach-portal/verify-global-reach-configuration.png" alt-text="Снимок экрана Global Reach настроена.":::

## <a name="disable-connectivity"></a>Отключение возможности подключения

Чтобы отключить подключение между отдельными каналами, нажмите кнопку Удалить рядом с *именем Global REACH* , чтобы удалить связь между ними. Затем нажмите кнопку **сохранить** , чтобы завершить операцию.

:::image type="content" source="./media/expressroute-howto-set-global-reach-portal/disable-global-reach-configuration.png" alt-text="Снимок экрана, показывающий, как отключить Global Reach.":::

После завершения операции вы больше не сможете подключиться к локальной сети через каналы ExpressRoute.

## <a name="next-steps"></a>Дальнейшие действия
- [Получение дополнительных сведений об ExpressRoute Global Reach](expressroute-global-reach.md).
- [Проверка подключения ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
- [Подключение виртуальной сети к каналу ExpressRoute](expressroute-howto-linkvnet-arm.md).
