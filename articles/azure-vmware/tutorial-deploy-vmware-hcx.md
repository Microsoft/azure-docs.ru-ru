---
title: Учебник по развертыванию и настройке VMware HCX
description: Узнайте, как развернуть и настроить решение VMware HCX для частного облака Решения Azure VMware.
ms.topic: tutorial
ms.date: 03/13/2021
ms.openlocfilehash: ab59b4a29f8ba372c1892613d5e0461bf3fdc512
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104869095"
---
# <a name="deploy-and-configure-vmware-hcx"></a>Развертывание и настройка VMware HCX

В этой статье показано, как развернуть и настроить локальный соединитель VMware HCX для частного облака Решения Azure VMware. С помощью VMware HCX вы можете перенести рабочие нагрузки VMware в Решение Azure VMware и на другие подключенные сайты, используя разные типы миграции. Решение Azure VMware развертывает и настраивает HCX Cloud Manager, поэтому вам нужно скачать, активировать и настроить соединитель HCX в локальном центре обработки данных VMware.

Расширенный соединитель VMware HCX уже развернут в Решении Azure VMware. Он поддерживает до трех подключений к сайтам (из локальной среды в облако или из облака в облако). Если вам нужно более трех подключений к сайтам, создайте [запрос в службу поддержки](https://portal.azure.com/#create/Microsoft.Support), чтобы включить надстройку [VMware HCX Enterprise](https://cloud.vmware.com/community/2019/08/08/introducing-hcx-enterprise/).  

>[!TIP]
>Хотя в средстве VMware Configuration Manager максимально допустимо 25 пар сайтов между локальным соединителем и диспетчером облачных сред, лицензии ограничивают это число до 3 для выпуска HCX Advanced и до 10 для выпуска HCX Enterprise.

>[!NOTE]
>VMware HCX Enterprise предоставляется в составе Решения Azure VMware в предварительной версии службы. Эту службу можно использовать бесплатно на условиях предварительной версии. Когда служба VMware HCX Enterprise станет общедоступной, вы получите уведомление о том, что через 30 дней начнут выставляться счета. Вы сможете отключить эту службу или отказаться от ее использования. Простого способа перейти с версии VMware HCX Enterprise к версии VMware HCX Advanced нет. Если вы решите перейти на использование более ранней версии, потребуется выполнить повторное развертывание, что предполагает простой.

Прежде всего внимательно изучите разделы [Перед началом работы](#before-you-begin), [Требования к версиям программного обеспечения](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-54E5293B-8707-4D29-BFE8-EE63539CC49B.html) и [Предварительные требования](#prerequisites). 

Затем мы последовательно разберем все необходимые процедуры для выполнения следующих задач:

> [!div class="checklist"]
> * скачивание OVA-файла соединителя VMware HCX;
> * развертывание локального OVA-файла VMware HCX (соединитель VMware HCX);
> * активация соединителя VMware HCX;
> * связывание локального соединителя VMware HCX с диспетчером облаков HCX в Решении Azure VMware;
> * настройка межсетевого соединения (сетевой профиль, профиль вычислений и сетка служб);
> * завершение установки путем проверки состояния устройства и проверки возможности миграции.

Завершив работу, переходите к дальнейшим действиям, описанным в конце этой статьи.  

## <a name="before-you-begin"></a>Перед началом

При подготовке развертывания мы рекомендуем вам ознакомиться со следующей документацией по VMware:

* [Пользовательское руководством по VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-E456F078-22BE-494B-8E4B-076EF33A9CF4.html)
* [Миграция виртуальных машин с помощью VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-D0CD0CC6-3802-42C9-9718-6DA5FEC246C6.html?hWord=N4IghgNiBcIBIGEAaACAtgSwOYCcwBcMB7AOxAF8g)
* [Рекомендации по развертыванию VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/install-checklist/GUID-C0A0E820-D5D0-4A3D-AD8E-EEAA3229F325.html)
* [Серия блогов, посвященных VMware и миграции в облако](https://blogs.vmware.com/vsphere/2019/10/cloud-migration-series-part-2.html) 
* [Сетевые порты, требуемые для работы VMware HCX](https://ports.vmware.com/home/VMware-HCX)


## <a name="prerequisites"></a>Предварительные требования

Если вы планируете использовать VMware HCX Enterprise, потребуется запросить активацию через каналы поддержки Решения Azure VMware.


### <a name="on-premises-vsphere-environment"></a>Локальная среда vSphere

Убедитесь, что локальная среда vSphere (исходная среда) соответствует [минимальным требованиям](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-54E5293B-8707-4D29-BFE8-EE63539CC49B.html). 

### <a name="network-and-ports"></a>Сеть и порты

* [ExpressRoute Global Reach](tutorial-expressroute-global-reach-private-cloud.md) настраивается между каналами ExpressRoute в локальной среде и частным облаком Решения Azure VMware.

* Открыты [все требуемые порты](https://ports.vmware.com/home/VMware-HCX) для обмена данными между локальными компонентами и частным облаком Решения Azure VMware.

### <a name="ip-addresses"></a>IP-адреса

[!INCLUDE [hcx-network-segments](includes/hcx-network-segments.md)]
   
## <a name="download-the-vmware-hcx-connector-ova"></a>Скачивание OVA-файла соединителя VMware HCX

Перед развертыванием виртуального устройства в локальном решении vCenter необходимо скачать OVA-файл соединителя VMware HCX.  

1. На портале Azure выберите частное облако Решения Azure VMware. 

1. Чтобы получить IP-адрес диспетчера Решения Azure VMware, выберите **Управление** > **Подключение** и перейдите на вкладку **HCX**. 

   :::image type="content" source="media/tutorial-vmware-hcx/find-hcx-ip-address.png" alt-text="Снимок экрана: получение IP-адреса VMware HCX." lightbox="media/tutorial-vmware-hcx/find-hcx-ip-address.png":::

1. Выберите **Управление** > **Удостоверение**. 

   Отобразятся URL-адреса и учетные данные пользователя в vCenter и NSX-T Manager частного облака.

   > [!TIP]
   > Пароль vCenter был определен при настройке частного облака. Это тот же пароль, который вы будете использовать для входа в диспетчер HCX Решения Azure VMware. Вы можете выбрать **Создать пароль**, чтобы создать новые пароли для vCenter и NSX-T.

   :::image type="content" source="media/tutorial-access-private-cloud/ss4-display-identity.png" alt-text="Отобразите URL-адреса и учетные данные частного облака vCenter и NSX Manager." border="true":::


1. Откройте окно браузера, войдите в диспетчер HCX Решения Azure VMware по адресу `https://x.x.x.9`, используя порт 443 и учетные данные данными пользователя **cloudadmin\@vsphere.local**.

1. Выберите **Администрирование** > **Обновление системы**, а затем **Запросить ссылку для скачивания**.

1. Выберите предпочтительный вариант для скачивания OVA-файла соединителя VMware HCX.

## <a name="deploy-the-vmware-hcx-connector-ova-on-premises"></a>Развертывание OVA-файла соединителя VMware HCX в локальной среде

1. В локальном экземпляре vCenter выберите [шаблон OVF-файла](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vm_admin.doc/GUID-17BEDA21-43F6-41F4-8FB2-E01D275FE9B4.html), чтобы развернуть соединитель HCX VMware в локальном экземпляре vCenter. 

   > [!TIP]
   > Выберите OVA-файл, скачанный ранее.  

   :::image type="content" source="media/tutorial-vmware-hcx/select-ovf-template.png" alt-text="Снимок экрана: обзор папок для выбора шаблона OVF" lightbox="media/tutorial-vmware-hcx/select-ovf-template.png":::


1. Выберите имя и расположение, а также ресурс или кластер для развертывания соединителя VMware HCX. Проверьте сведения и необходимые ресурсы, а затем щелкните **Next** (Далее).  

1. Ознакомьтесь с условиями лицензии. Если вы с ними согласны, выберите нужное хранилище и сеть, а затем щелкните **Далее**.

1. Выберите хранилище и щелкните **Next** (Далее).

1. Выберите сегмент сети управления VMware HCX, который вы ранее определили согласно [предварительным требованиям для IP-адресов](#ip-addresses).  Затем нажмите кнопку **Далее**.

1. В разделе **Customize template** (Настройка шаблона) введите все необходимые данные и щелкните **Next** (Далее). 

   :::image type="content" source="media/tutorial-vmware-hcx/customize-template.png" alt-text="Снимок экрана: поля для настройки шаблона" lightbox="media/tutorial-vmware-hcx/customize-template.png":::

1. Проверьте конфигурацию и щелкните **Finish** (Готово), чтобы развернуть OVA-файл соединителя VMware HCX.
   
   > [!IMPORTANT]
   > Возможно, вам придется вручную включить виртуальное устройство.  После включения подождите 10–15 минут, прежде чем переходить к следующему шагу.

Полный обзор этой процедуры см. в видео [Azure VMware Solution: HCX Appliance Deployment](https://www.youtube.com/embed/UKmSTYrL6AY) (Решение Azure VMware — развертывание устройства HCX). 


## <a name="activate-vmware-hcx"></a>Активация VMware HCX

Завершив развертывание OVA-файла соединителя VMware HCX в локальной среде и запустив устройство, можно переходить к активации. Для этого необходимо получить лицензионный ключ на портале Решения Azure VMware.

1. На портале Решения Azure VMware выберите **Manage** > **Connectivity** (Управление — Подключение), перейдите на вкладку **HCX**, а затем щелкните **Add** (Добавить).

1. Войдите в VMware HCX Manager в локальной среде по адресу `https://HCXManagerIP:9443`, используя учетные данные **администратора**. 

   > [!TIP]
   > Пароль **администратора** определяется во время развертывания OVA-файла для диспетчера VMware HCX.

   > [!IMPORTANT]
   > Не забудьте указать номер порта `9443` вместе с IP-адресом VMware HCX Manager.

1. В разделе **Licensing** (Лицензирование) введите свой ключ в поле **HCX Advanced Key** (Расширенный ключ HCX), затем щелкните **Activate** (Активировать).  
   
    > [!NOTE]
    > VMware HCX Manager должен иметь открытый доступ к Интернету или настроенный прокси-сервер.

1. В поле **Datacenter Location** (Расположение центра обработки данных) укажите ближайшее расположение к локальной среде с VMware HCX Manager. Затем выберите **Continue** (Продолжить).

1. Измените имя в поле **System Name** (Имя системы) или сохраните значение по умолчанию, а затем щелкните **Continue** (Продолжить).
   
1. Щелкните **Yes, Continue** (Да, продолжить).

1. В разделе **Connect your vCenter** (Подключение к vCenter) укажите полное доменное имя или IP-адрес сервера vCenter и соответствующие учетные данные, а затем выберите **Continue** (Продолжить).
   
   > [!TIP]
   > Сервер vCenter используется для развертывания соединителя VMware HCX в центре обработки данных.

1. В разделе **Configure SSO/PSC** (Настройка единого входа или PSC) укажите полное доменное имя или IP-адрес Platform Services Controller, а затем щелкните **Continue** (Продолжить).
   
   > [!NOTE]
   > Обычно это значение полного доменного имени или IP-адреса vCenter.

1. Убедитесь, что указаны правильные сведения, и выберите **Restart** (Перезапустить).
    
   > [!NOTE]
   > После перезапуска будет наблюдаться определенная задержка с отображением запроса следующего шага.

После перезапуска служб откроется экран, где vCenter отмечен зеленым цветом. Параметры конфигурации vCenter и единого входа должны быть заданы надлежащим образом и совпадать с теми, которые показаны на предыдущем экране.

:::image type="content" source="media/tutorial-vmware-hcx/activation-done.png" alt-text="Снимок экрана: панель мониторинга, где состояние vCenter обозначено зеленым цветом" lightbox="media/tutorial-vmware-hcx/activation-done.png":::  

Полный обзор этой процедуры см. в видео [Azure VMware Solution: Activate HCX](https://www.youtube.com/embed/PnVg6SZkQsY?rel=0&amp;vq=hd720) (Решение Azure VMware — активация HCX).

   > [!IMPORTANT]
   > Независимо от того, используете ли вы HCX Advanced или HCX Enterprise, может потребоваться установка исправления, указанного в [статье 81558 базы знаний](https://kb.vmware.com/s/article/81558) VMware. 

## <a name="configure-the-vmware-hcx-connector"></a>Настройка соединителя VMware HCX

Теперь вы можете добавить связывание сайтов, создать профиль сети и вычислений и включить службы миграции, расширения сети или аварийного восстановления. 

### <a name="add-a-site-pairing"></a>Добавление связывания сайта

Вы можете подключить диспетчер облаков VMware HCX в Решении Azure VMware к соединителю VMware HCX в центре обработки данных (связать их между собой). 

1. Войдите в vCenter в локальной среде и в разделе **Home** (Домашняя страница) выберите **HCX**.

1. В разделе **Infrastructure** (Инфраструктура) выберите **Site Pairing** (Связывание сайтов), а затем в середине экрана щелкните **Connect To Remote Site** (Подключиться к удаленному сайту). 

1. Введите URL-адрес или IP-адрес (`https://x.x.x.9`) диспетчера облака HCX Решения Azure VMware, который вы сохранили ранее, имя пользователя (cloudadmin\@vsphere.local) и пароль для Решения Azure VMware. В этом случае выберите **Подключиться**.

   > [!NOTE]
   > Условия для успешного связывания сайтов:
   > * Соединитель VMware HCX должен иметь возможность маршрутизировать IP-адрес диспетчера облака HCX через порт 443.
   >
   > * Используйте тот же пароль, что и для входа в vCenter. Этот пароль был определен на начальном экране развертывания.

   На этом экране будет показано, что диспетчер облаков VMware HCX в Решении Azure VMware и соединитель VMware HCX в локальной среде успешно подключены (связаны друг с другом).

   :::image type="content" source="media/tutorial-vmware-hcx/site-pairing-complete.png" alt-text="Снимок экрана: связывание диспетчера HCX в Решении Azure VMware и соединителя VMware HCX":::.

Полный обзор этой процедуры см. в видео [Azure VMware Solution: HCX Site Pairing](https://www.youtube.com/embed/jXOmYUnbWZY?rel=0&amp;vq=hd720) (Решение Azure VMware — связывание сайтов HCX).

### <a name="create-network-profiles"></a>Создание сетевых профилей

Соединитель VMware HCX развертывает подмножество виртуальных устройств (автоматизированных), которым требуется несколько сегментов IP-адресов. При создании сетевых профилей вы задаете сегменты IP-адресов, которые определили ранее на [этапе предварительного планирования развертывания сегментов сети VMware HCX и подготовки к его реализации](production-ready-deployment-steps.md#vmware-hcx-network-segments).

Вы создадите четыре сетевых профиля:

   - Управление
   - vMotion
   - Репликация
   - Исходящая связь

1. Выберите **Infrastructure**, select **Interconnect** > **Multi-Site Service Mesh** > **Network Profiles** > **Create Network Profile** (Инфраструктура > Межсоединение > Сетка многосайтовой службы > Сетевые профили > Создать сетевой профиль).

   :::image type="content" source="media/tutorial-vmware-hcx/network-profile-start.png" alt-text="Снимок экрана: параметры, настроенные для создания сетевого профиля" lightbox="media/tutorial-vmware-hcx/network-profile-start.png":::

1. Для каждого сетевого профиля выберите сеть и группу портов, укажите имя и создайте пул IP-адресов сегмента. Щелкните **Создать**. 

   :::image type="content" source="media/tutorial-vmware-hcx/example-configurations-network-profile.png" alt-text="Снимок экрана: сведения о новом сетевом профиле":::

Полный обзор этой процедуры см. в видео [Azure VMware Solution: HCX Network Profile](https://www.youtube.com/embed/O0rU4jtXUxc) (Решение Azure VMware — сетевой профиль HCX).


### <a name="create-a-compute-profile"></a>Создание вычислительного профиля

1. В разделе **Infrastructure** (Инфраструктура) последовательно выберите **Interconnect** > **Compute Profiles** > **Create Compute Profile** (Межсоединение — Вычислительные профили — Создать вычислительный профиль).

   :::image type="content" source="media/tutorial-vmware-hcx/compute-profile-create.png" alt-text="Снимок экрана: выбранные параметры для создания вычислительного профиля" lightbox="media/tutorial-vmware-hcx/compute-profile-create.png":::

1. Введите имя профиля и нажмите кнопку **Continue** (Продолжить).  

   :::image type="content" source="media/tutorial-vmware-hcx/name-compute-profile.png" alt-text="Снимок экрана: поле с введенным именем вычислительного профиля и кнопка Continue (Продолжить)" lightbox="media/tutorial-vmware-hcx/name-compute-profile.png":::

1. Выберите службы, которые нужно включить, такие как службы миграции, сетевого расширения или аварийного восстановления, а затем нажмите кнопку **Continue** (Продолжить).
  
   > [!NOTE]
   > Обычно здесь не нужно ничего изменять.

1. В разделе **Select Service Resources** (Выбор ресурсов служб) выберите один или несколько ресурсов служб (кластеры), для которых должны быть включены выбранные службы VMware HCX.  

1. Если кластеры отображаются в локальном центре обработки данных, выберите **Continue** (Продолжить).

   :::image type="content" source="media/tutorial-vmware-hcx/select-service-resource.png" alt-text="Снимок экрана: выбор ресурсов служб и кнопка Continue (Продолжить)" lightbox="media/tutorial-vmware-hcx/select-service-resource.png":::

1. На странице **Select Datastore** (Выбор хранилища данных) выберите ресурс хранилища данных для развертывания устройств VMware HCX Interconnect. Затем выберите **Continue** (Продолжить).

   Когда выбрано несколько ресурсов, VMware HCX использует первый выбранный ресурс, пока его емкость не будет исчерпана.   

   :::image type="content" source="media/tutorial-vmware-hcx/deployment-resources-and-reservations.png" alt-text="Снимок экрана: выбранный ресурс хранилища данных и кнопка Continue (Продолжить)" lightbox="media/tutorial-vmware-hcx/deployment-resources-and-reservations.png":::  

1. В разделе **Select Management Network Profile** (Выбор сетевого профиля управления) выберите профиль, созданный на предыдущих шагах. Затем выберите **Continue** (Продолжить).  

   :::image type="content" source="media/tutorial-vmware-hcx/select-management-network-profile.png" alt-text="Снимок экрана: выбор сетевого профиля управления и кнопка Continue (Продолжить)" lightbox="media/tutorial-vmware-hcx/select-management-network-profile.png":::

1. В разделе **Select Uplink Network Profile** (Выбор сетевого профиля канала исходящей связи) выберите сетевой профиль, созданный ранее. Затем выберите **Continue** (Продолжить).

   :::image type="content" source="media/tutorial-vmware-hcx/select-uplink-network-profile.png" alt-text="Снимок экрана: выбор сетевого профиля канала исходящей связи и кнопка Continue (Продолжить)" lightbox="media/tutorial-vmware-hcx/select-uplink-network-profile.png":::

1. В разделе **Select vMotion Network Profile** (Выбор сетевого профиля vMotion) выберите сетевой профиль, созданный на предыдущих шагах. Затем выберите **Continue** (Продолжить).

   :::image type="content" source="media/tutorial-vmware-hcx/select-vmotion-network-profile.png" alt-text="Снимок экрана: выбор сетевого профиля vMotion и кнопка Continue (Продолжить)" lightbox="media/tutorial-vmware-hcx/select-vmotion-network-profile.png":::

1. В разделе **Select vSphere Replication Network Profile** (Выбор сетевого профиля репликации vSphere) выберите сетевой профиль, созданный на предыдущих шагах. Затем выберите **Continue** (Продолжить).

   :::image type="content" source="media/tutorial-vmware-hcx/select-replication-network-profile.png" alt-text="Снимок экрана: выбор сетевого профиля репликации и кнопка Continue (Продолжить)" lightbox="media/tutorial-vmware-hcx/select-replication-network-profile.png":::

1. В разделе **Select Distributed Switches for Network Extensions** (Выбор распределенных коммутаторов для сетевых расширений) выберите коммутаторы, через которые подключаются виртуальные машины, переносимые в Решение Azure VMware из расширенной сети уровня 2. Затем выберите **Continue** (Продолжить).

   > [!NOTE]
   > Если вы не выполняете миграцию виртуальных машин в расширенных сетях уровня 2 (L2), этот шаг можно пропустить.
   
   :::image type=" content" source="media/tutorial-vmware-hcx/select-layer-2-distributed-virtual-switch.png" alt-text="Снимок экрана: выбор распределенных виртуальных коммутаторов и кнопка Continue (Продолжить)" lightbox="media/tutorial-vmware-hcx/select-layer-2-distributed-virtual-switch.png":::

1. Проверьте правила подключения и нажмите **Continue** (Продолжить).  

   :::image type="content" source="media/tutorial-vmware-hcx/review-connection-rules.png" alt-text="Снимок экрана: правила подключения и кнопка Continue (Продолжить)" lightbox="media/tutorial-vmware-hcx/review-connection-rules.png":::

1. Нажмите **Готово**, чтобы создать вычислительный профиль.


   :::image type="content" source="media/tutorial-vmware-hcx/compute-profile-done.png" alt-text="Снимок экрана: сведения о вычислительном профиле" lightbox="media/tutorial-vmware-hcx/compute-profile-done.png":::

Полный обзор этой процедуры см. в видео [Azure VMware Solution: Compute Profile](https://www.youtube.com/embed/e02hsChI3b8) (Решение Azure VMware — вычислительный профиль).

### <a name="create-a-service-mesh"></a>Создание сетки служб

Теперь нужно настроить сетку служб между локальной средой и частным облаком Решения Azure VMware.



> [!NOTE]
> Чтобы связать сетку служб с Решением Azure VMware, требуется следующее:
>
> * открытые порты UDP 500/4500 между локальными адресами сетевого профиля исходящей связи, определенными в локальном соединителе VMware HCX, и адресами сетевого профиля исходящей связи облака HCX в Решении Azure VMware.
>
> * Не забудьте проверить [требуемые порты для VMware HCX](https://ports.vmware.com/home/VMware-HCX).

1. В разделе **Infrastructure** (Инфраструктура) выберите **Interconnect** > **Service Mesh** > **Create Service Mesh** (Межсоединение > Сетка служб > Создать сетку служб).    

   :::image type="content" source="media/tutorial-vmware-hcx/create-service-mesh.png" alt-text="Снимок экрана: выбор параметров для создания сетки служб" lightbox="media/tutorial-vmware-hcx/create-service-mesh.png":::

1. Просмотрите предварительно заполненные сайты, а затем выберите **Continue** (Продолжить). 

   > [!NOTE]
   > Если сетка служб настраивается впервые, вам не потребуется ничего менять на этом экране.  

1. Выберите значения для вычислительных профилей исходной и удаленной среды в раскрывающихся списках, а затем щелкните **Continue** (Продолжить).  

   От выбранных значений зависит, для каких ресурсов виртуальные машины смогут использовать службы VMware HCX.  

   :::image type="content" source="media/tutorial-vmware-hcx/select-compute-profile-source.png" alt-text="Снимок экрана: выбор исходного вычислительного профиля" lightbox="media/tutorial-vmware-hcx/select-compute-profile-source.png":::

   :::image type="content" source="media/tutorial-vmware-hcx/select-compute-profile-remote.png" alt-text="Снимок экрана: выбор удаленного вычислительного профиля" lightbox="media/tutorial-vmware-hcx/select-compute-profile-remote.png":::

1. Просмотрите службы, которые будут включены, а затем щелкните **Continue** (Продолжить).  

1. В разделе **Advanced Configuration - Override Uplink Network profiles** (Расширенная конфигурация — переопределение сетевых профилей каналов исходящей связи) щелкните **Continue** (Продолжить).  

   Сетевые профили каналов исходящей связи обеспечивают подключение к сети, через которую можно обращаться к устройствам Interconnect удаленного сайта.  
  
1. Проверьте параметры на экране **Advanced Configuration — Network Extension Appliance Scale Out** (Расширенная конфигурация: горизонтальное увеличение масштаба устройство расширения сети) и выберите **Continue** (Продолжить). 

   Вы можете использовать до восьми виртуальных локальных сетей на устройство. Вы также можете развернуть другое устройство, чтобы добавить еще восемь виртуальных локальных сетей. Кроме того, в учетной записи необходимо иметь IP-адреса для дополнительных устройств (один IP-адрес на устройство).  Дополнительные сведения см. на странице со сведениями об [ограничениях конфигурации VMware HCX](https://configmax.vmware.com/guest?vmwareproduct=VMware%20HCX&release=VMware%20HCX&categories=41-0,42-0,43-0,44-0,45-0).
   
   :::image type="content" source="media/tutorial-vmware-hcx/extend-networks-increase-vlan.png" alt-text="Снимок экрана, на котором показано, где можно увеличить число виртуальных локальных сетей." lightbox="media/tutorial-vmware-hcx/extend-networks-increase-vlan.png":::

1. На экране **Advanced Configuration — Traffic Engineering** (Расширенная конфигурация: проектирование трафика) проверьте параметры и внесите необходимые изменения, а затем щелкните **Continue** (Продолжить).

1. Проверьте предварительную версию топологии и нажмите **Продолжить**.

1. Введите понятное имя для этой сетки служб и нажмите **Finish** (Готово) для завершения.  

1. Выберите **View Tasks** (Просмотр задач) для мониторинга развертывания. 

   :::image type="content" source="media/tutorial-vmware-hcx/monitor-service-mesh.png" alt-text="Снимок экрана: кнопка для просмотра задач":::

   После успешного завершения развертывания сетки служб вы увидите, что службы обозначены зеленым цветом.

   :::image type="content" source="media/tutorial-vmware-hcx/service-mesh-green.png" alt-text="Снимок экрана: зеленый индикатор состояния служб" lightbox="media/tutorial-vmware-hcx/service-mesh-green.png":::

1. Проверьте работоспособность сетки службы, проверив состояние устройства. 

1. Для этого выберите **Interconnect** > **Appliances** (Interconnect > Устройства).

   :::image type="content" source="media/tutorial-vmware-hcx/interconnect-appliance-state.png" alt-text="Снимок экрана: выбор действий для проверки состояния устройства" lightbox="media/tutorial-vmware-hcx/interconnect-appliance-state.png":::

Полный обзор этой процедуры см. в видео [Azure VMware Solution: Service Mesh](https://www.youtube.com/embed/COY3oIws108) (Решение Azure VMware — сетка служб).

### <a name="optional-create-a-network-extension"></a>Создание расширения сети (необязательно)

Чтобы расширить какие-либо сети из локальной среды, включив в них Решение Azure VMware, сделайте следующее:

1. В разделе **Services** (Службы) последовательно выберите **Network Extension** > **Create a Network Extension** (Расширение сети > Создать расширение сети).

   :::image type="content" source="media/tutorial-vmware-hcx/create-network-extension.png" alt-text="Снимок экрана: выбранные параметры для создания расширения сети" lightbox="media/tutorial-vmware-hcx/create-network-extension.png":::

1. Выберите все сети, которые нужно расширить в Решение Azure VMware, и щелкните **Next** (Далее).

   :::image type="content" source="media/tutorial-vmware-hcx/select-extend-networks.png" alt-text="Снимок экрана: выбор сети":::

1. Введите IP-адрес локального шлюза для каждой из расширяемых сетей, а затем нажмите кнопку **Submit** (Отправить). 

   :::image type="content" source="media/tutorial-vmware-hcx/extend-networks-gateway.png" alt-text="Снимок экрана: ввод IP-адреса шлюза":::

   Расширение сети занимает несколько минут. Когда оно будет выполнено, отобразится новое состояние **Extension complete** (Расширение завершено).

   :::image type="content" source="media/tutorial-vmware-hcx/extension-complete.png" alt-text="Снимок экрана: состояние завершения расширения" lightbox="media/tutorial-vmware-hcx/extension-complete.png":::

Полный обзор этой процедуры см. в видео [Azure VMware Solution: Network Extension](https://www.youtube.com/embed/gYR0nftKui0) (Решение Azure VMware — расширение сети).


## <a name="next-steps"></a>Дальнейшие действия

Если туннель подключения HCX имеет состояние **UP** (Активно) и обозначен зеленым цветом, вы можете перенести и защитить виртуальные машины Решения Azure VMware с помощью VMware HCX. Решение Azure VMware поддерживает миграцию рабочих нагрузок (с расширением сети или без). Вы по-прежнему можете переносить рабочие нагрузки в среду vSphere, создавать сети в локальной среде и развертывать виртуальные машины в этих сетях.  

Дополнительные сведения об использовании HCX см. в технической документации по VMware:

* [Документация по VMware HCX](https://docs.vmware.com/en/VMware-HCX/index.html)
* [Миграция виртуальных машин с помощью VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-D0CD0CC6-3802-42C9-9718-6DA5FEC246C6.html?hWord=N4IghgNiBcIBIGEAaACAtgSwOYCcwBcMB7AOxAF8g)
* [Требуемые порты HCX](https://ports.vmware.com/home/VMware-HCX)
* [Настройка прокси-сервера HCX перед утверждением лицензионного ключа](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-920242B3-71A3-4B24-9ACF-B20345244AB2.html?hWord=N4IghgNiBcIA4CcD2APAngAgBIGEAaIAvkA)
