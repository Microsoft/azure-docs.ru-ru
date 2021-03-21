---
title: Управление DHCP для решения VMware Azure
description: Узнайте, как создать DHCP для частного облака решения Azure VMware и управлять им.
ms.topic: how-to
ms.custom: contperf-fy21q2
ms.date: 11/09/2020
ms.openlocfilehash: bcaba4274b0e6b423e9fa490c80fc57204d4e153
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97708557"
---
# <a name="manage-dhcp-for-azure-vmware-solution"></a>Управление DHCP для решения VMware Azure

Приложениям и рабочим нагрузкам, работающим в среде частного облака, требуются службы DHCP для назначения IP-адресов.  В этой статье показано, как создать DHCP и управлять им в решении Azure VMware двумя способами:

- Если вы используете NSX-T для размещения DHCP-сервера, необходимо [создать DHCP-сервер](#create-a-dhcp-server) и [ретранслятор для этого сервера](#create-dhcp-relay-service). При создании DHCP-сервера также добавляется сегмент сети и указывается диапазон IP-адресов DHCP.   

- Если вы используете сторонний внешний DHCP-сервер в сети, необходимо [создать службу ретранслятора DHCP](#create-dhcp-relay-service). При создании ретранслятора на DHCP-сервере (НСКС-T или стороннего производителя для размещения DHCP-сервера) необходимо указать диапазон IP-адресов DHCP.

>[!IMPORTANT]
>DHCP не работает для виртуальных машин в сети VMware ХККС на уровне 2, если DHCP-сервер находится в локальном центре обработки данных.  По умолчанию НСКС блокирует все DHCP-запросы от обхода перетягивания L2. Сведения о решении см. в разделе [Отправка запросов DHCP в локальную процедуру DHCP-сервера](#send-dhcp-requests-to-the-on-premises-dhcp-server) .


## <a name="create-a-dhcp-server"></a>Создание DHCP-сервера

Если вы хотите использовать НСКС-T для размещения DHCP-сервера, вы создадите DHCP-сервер. Затем добавьте сегмент сети и укажите диапазон IP-адресов DHCP.

1. В диспетчере НСКС-T выберите **сети**  >  **DHCP** и нажмите кнопку **Добавить сервер**.

1. Выберите **DHCP** для **типа сервера**, укажите имя и IP-адрес сервера, а затем нажмите кнопку **сохранить**.

   :::image type="content" source="./media/manage-dhcp/dhcp-server-settings.png" alt-text="добавить DHCP-сервер" border="true":::

1. Выберите **шлюзы уровня 1**, щелкните вертикальное многоточие на шлюзе уровня 1, а затем выберите **изменить**.

   :::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway.png" alt-text="Выберите шлюз для использования" border="true":::

1. Выберите **не устанавливать выделение IP-адресов** , чтобы добавить подсеть.

   :::image type="content" source="./media/manage-dhcp/add-subnet.png" alt-text="Добавление подсети" border="true":::

1. В качестве **типа** выберите **локальный сервер DHCP**. 
   
1. Для **DHCP-сервера** выберите **по умолчанию DHCP** и нажмите кнопку **сохранить**.

1. Нажмите кнопку **сохранить** еще раз и выберите **Закрыть редактирование**.

### <a name="add-a-network-segment"></a>Добавление сегмента сети

[!INCLUDE [add-network-segment-steps](includes/add-network-segment-steps.md)]


## <a name="create-dhcp-relay-service"></a>Создание службы ретранслятора DHCP

Если вы хотите использовать внешний DHCP-сервер стороннего производителя, необходимо создать службу ретранслятора DHCP. Вы также укажете диапазон IP-адресов DHCP в НСКС-T Manager. 

1. В диспетчере НСКС-T выберите **сети**  >  **DHCP** и нажмите кнопку **Добавить сервер**.

1. Выберите **ретранслятор DHCP** для **типа сервера**, укажите имя и IP-адрес сервера, а затем нажмите кнопку **сохранить**.

   :::image type="content" source="./media/manage-dhcp/create-dhcp-relay.png" alt-text="Создание службы ретранслятора DHCP" border="true":::

1. Выберите **шлюзы уровня 1**, щелкните вертикальное многоточие на шлюзе уровня 1, а затем выберите **изменить**.

   :::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway-relay.png" alt-text="Изменение шлюза уровня 1" border="true":::

1. Выберите **не устанавливать выделение IP-** адресов, чтобы определить выделение IP-адреса.

   :::image type="content" source="./media/manage-dhcp/edit-ip-address-allocation.png" alt-text="изменить выделение IP-адресов" border="true":::

1. В качестве **типа** выберите **DHCP-сервер**. 
   
1. Для **DHCP-сервера** выберите **ретранслятор DHCP** и нажмите кнопку **сохранить**.

1. Нажмите кнопку **сохранить** еще раз и выберите **Закрыть редактирование**.


## <a name="specify-the-dhcp-ip-address-range"></a>Укажите диапазон IP-адресов DHCP

1. В NSX-T Manager выберите **Networking** > **Segments** (Сеть > Сегменты). 
   
1. Щелкните вертикальное многоточие в названии сегмента и выберите **изменить**.
   
1. Выберите **задать подсети** , чтобы указать IP-адрес DHCP для подсети. 
   
   :::image type="content" source="./media/manage-dhcp/network-segments.png" alt-text="сегменты сети" border="true":::
      
1. При необходимости измените IP-адрес шлюза и введите диапазон DHCP IP. 
      
   :::image type="content" source="./media/manage-dhcp/edit-subnet.png" alt-text="изменение подсетей" border="true":::
      
1. Нажмите кнопку **Применить**, а затем **сохранить**. Сегменту назначается пул DHCP-серверов.
      
   :::image type="content" source="./media/manage-dhcp/assigned-to-segment.png" alt-text="Пул DHCP-серверов, назначенный сегменту" border="true":::


## <a name="send-dhcp-requests-to-the-on-premises-dhcp-server"></a>Отправка запросов DHCP на локальный DHCP-сервер

Если вы хотите отправить DHCP-запросы из виртуальных машин Azure VMware в расширенном сегменте L2 на локальный DHCP-сервер, создайте профиль сегмента безопасности. 

1. Войдите в vCenter в локальной среде и в разделе **Home** (Домашняя страница) выберите **HCX**.

1. Выберите **расширение сети** в разделе **службы**.

1. Выберите расширение сети, которое будет поддерживать запросы DHCP из решения VMware для Azure в локальную среду. 

1. Запишите имя целевой сети.  

   :::image type="content" source="media/manage-dhcp/hcx-find-destination-network.png" alt-text="Снимок экрана расширения сети в VMware vSphere Client" lightbox="media/manage-dhcp/hcx-find-destination-network.png":::

1. В диспетчере решений Azure VMware НСКС-T выберите **сети**  >    >  **сегмент профили** сегментов. 

1. Выберите **Добавить профиль сегмента** , а затем **сегмент безопасность**.

   :::image type="content" source="media/manage-dhcp/add-segment-profile.png" alt-text="Снимок экрана добавления профиля сегмента в НСКС-T" lightbox="media/manage-dhcp/add-segment-profile.png":::

1. Укажите имя и тег, а затем установите переключатель **фильтра бпду** в положение вкл., а все ПЕРЕКЛЮЧАТЕЛи DHCP — выключены.

   :::image type="content" source="media/manage-dhcp/add-segment-profile-bpdu-filter-dhcp-options.png" alt-text="Снимок экрана с включенным фильтром БПДУ и выключением DHCP" lightbox="media/manage-dhcp/add-segment-profile-bpdu-filter-dhcp-options.png":::

1. Удалите все MAC-адреса (если они есть) в **списке разрешений фильтра бпду**.  Затем нажмите кнопку **Save** (Сохранить).

   :::image type="content" source="media/manage-dhcp/add-segment-profile-bpdu-filter-allow-list.png" alt-text="Снимок экрана, показывающий MAC-адреса в списке разрешений фильтра БПДУ":::

1. В разделе сегменты сегментов **сети**  >    >  в области поиска введите сетевое имя определения.

   :::image type="content" source="media/manage-dhcp/networking-segments-search.png" alt-text="Снимок экрана: поле фильтра &quot;Сетевые > сегментов&quot;":::

1. Щелкните вертикальное многоточие в названии сегмента и выберите **изменить**.

   :::image type="content" source="media/manage-dhcp/edit-network-segment.png" alt-text="Снимок экрана: кнопка &quot;Изменить&quot; для сегмента" lightbox="media/manage-dhcp/edit-network-segment.png":::

1. Измените **Безопасность сегмента** на созданный ранее профиль сегмента.

   :::image type="content" source="media/manage-dhcp/edit-segment-security.png" alt-text="Снимок экрана: поле &quot;безопасность сегмента&quot;" lightbox="media/manage-dhcp/edit-segment-security.png":::

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [обслуживании узла и управлении жизненным циклом](concepts-private-clouds-clusters.md#host-maintenance-and-lifecycle-management).