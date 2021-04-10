---
title: Добавление сегмента сети NSX-T
description: Инструкции по добавлению сегмента сети NSX-T в Решении Azure VMware.
ms.topic: include
ms.date: 03/13/2021
ms.openlocfilehash: 14d698413d31af2dcbbdea5f37ec7f24f65199ad
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103462136"
---
<!-- Used in manage-dhcp.md and tutorial-nsx-t-network-segment.md -->

1. В NSX-T **Networking** > **Segments** (Manager > Сеть > Сегменты), а затем выберите **Add Segment** (Добавить сегмент). 

   :::image type="content" source="../media/nsxt/nsxt-segments-overview.png" alt-text="Снимок экрана, на котором показано, как добавить новый сегмент":::

1. Введите имя сегмента.

1. Выберите шлюз уровня 1 (TNTxx-T1) в качестве **подключенного шлюза** и оставьте значение Flexible (Гибкий) в поле **Type** (Тип).

1. Выберите предварительно настроенную **зону транспортировки** для наложения (TNTxx-OVERLAY-TZ), а затем выберите **Set Subnets** (Задать подсети). 

   :::image type="content" source="../media/nsxt/nsxt-create-segment-specs.png" alt-text="Укажите имя сегмента, подключенный шлюз, тип и зону транспортировки, а затем выберите Set Subnets (Задать подсети).":::

1. Введите IP-адрес шлюза и выберите **Добавить**. 

   >[!IMPORTANT]
   >IP-адрес должен находиться в неперекрывающемся блоке адресов RFC1918, что обеспечит подключение к виртуальным машинам в новом сегменте.

   :::image type="content" source="../media/nsxt/nsxt-create-segment-gateway.png" alt-text="Укажите IP-адрес шлюза для нового сегмента, а затем выберите ADD (Добавить).":::

1. Нажмите кнопку **Apply** (Применить), а затем — **Save** (Сохранить).

1. Выберите **No** (Нет), чтобы отказаться от дальнейшей настройки сегмента. 

   :::image type="content" source="../media/nsxt/nsxt-create-segment-continue-no.png" alt-text="Отклонение настройки недавно созданного сегмента сети, выбрав &quot;Нет&quot;.":::

1. Убедитесь в наличии нового сегмента сети. В этом примере новый сегмент сети — это **ls01**.

   1. В NSX-T Manager выберите **Networking** > **Segments** (Сеть > Сегменты). 

      :::image type="content" source="../media/nsxt/nsxt-new-segment-overview-2.png" alt-text="Проверка наличия нового сегмента сети в NSX-T.":::

   1. В vCenter выберите **Networking > SDDC-Datacenter** (Сеть > Центр обработки данных SDDC).

      :::image type="content" source="../media/nsxt/vcenter-with-ls01-2.png" alt-text="Проверка наличия нового сегмента сети в vCenter.":::