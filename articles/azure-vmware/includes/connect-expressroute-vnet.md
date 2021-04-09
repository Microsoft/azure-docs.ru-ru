---
title: Подключение ExpressRoute к шлюзу виртуальной сети
description: Инструкции по подключению ExpressRoute к шлюзу виртуальной сети.
ms.topic: include
ms.date: 12/08/2020
ms.openlocfilehash: 6e2e3748dbfd8d69b53dcc4c3a09809756ac48dc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103494374"
---
<!-- Used in deploy-azure-vmware-solution.md and tutorial-configure-networking.md -->

1. Перейдите к частному облаку, созданному вами при работе с учебником [Развертывание кластера vSphere в Azure](../tutorial-create-private-cloud.md). Выберите **Возможность подключения** в разделе **Управление** и откройте вкладку **ExpressRoute**.

1. Скопируйте ключ авторизации. Если ключ авторизации отсутствует, его необходимо создать, выбрав **+ Запросить ключ авторизации**.

   :::image type="content" source="../media/expressroute-global-reach/start-request-authorization-key.png" alt-text="Скопируйте ключ авторизации. Если ключ авторизации отсутствует, его необходимо создать, выбрав &quot;+ Запросить ключ авторизации&quot;" border="true" lightbox="../media/expressroute-global-reach/start-request-authorization-key.png":::.

1. Перейдите к шлюзу виртуальной сети, который вы создали на предыдущем шаге, и в разделе **Параметры** выберите **Подключения**. На странице **Подключения** выберите **+ Добавить**.

1. На странице **Добавление подключения** укажите значения в полях и нажмите кнопку **OК**. 

   | Поле | Значение |
   | --- | --- |
   | **имя**;  | Введите имя для подключения.  |
   | **Тип подключения:**  | Выберите **ExpressRoute**.  |
   | **Активировать авторизацию**  | Обязательно установите этот флажок.  |
   | **Шлюз виртуальной сети** | Это шлюз виртуальной сети, который вы создали ранее.  |
   | **Ключ авторизации**  | Скопируйте и вставьте сюда ключ авторизации, полученный на вкладке ExpressRoute для группы ресурсов. |
   | **URI одноранговой цепи**  | Скопируйте и вставьте сюда идентификатор ExpressRoute, полученный на вкладке ExpressRoute для группы ресурсов.  |

   :::image type="content" source="../media/expressroute-global-reach/expressroute-add-connection.png" alt-text="Снимок экрана: страница &quot;Добавление подключения&quot; для подключения ExpressRoute к шлюзу виртуальной сети.":::

В результате создается подключение между каналом ExpressRoute и виртуальной сетью.

:::image type="content" source="../media/expressroute-global-reach/virtual-network-gateway-connections.png" alt-text="Снимок экрана: подключения шлюза виртуальной сети.":::