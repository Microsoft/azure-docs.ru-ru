---
title: Руководство по созданию подключения "сеть — сеть" с помощью Виртуальной глобальной сети Azure
description: В руководстве показано, как создать VPN-подключение типа "сеть — сеть" к Azure с помощью Виртуальной глобальной сети Azure.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 03/05/2021
ms.author: cherylmc
ms.openlocfilehash: 365952beca96b91d312eab209c5332fca2d4842b
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106061882"
---
# <a name="tutorial-create-a-site-to-site-connection-using-azure-virtual-wan"></a>Руководство по Создание подключения "сеть — сеть" с помощью Виртуальной глобальной сети Azure

В этом руководстве объясняется, как создать подключение к ресурсам в Azure через VPN-соединение IPsec/IKE (IKEv1 и IKEv2) с помощью Виртуальной глобальной сети. Для этого типа подключения требуется локальное VPN-устройство, которому назначен внешний общедоступный IP-адрес. Дополнительные сведения о Виртуальной глобальной сети см. в [этой статье](virtual-wan-about.md).

Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * Создание виртуальной глобальной сети
> * Создание концентратора.
> * Создание сайта.
> * Подключение сайта к концентратору.
> * Подключение сайта VPN к концентратору.
> * Подключение виртуальной сети к концентратору.
> * Скачивание файла конфигурации.
> * Настройка VPN-шлюза

> [!NOTE]
> Обычно при наличии большого количества сайтов для создания этой конфигурации используется [партнер Виртуальной глобальной сети](https://aka.ms/virtualwan). Однако вы можете создать эту конфигурацию самостоятельно, если вы знакомы с работой с сетью и умеете настраивать собственное VPN-устройство.
>

![Схема Виртуальной глобальной сети](./media/virtual-wan-about/virtualwan.png)

## <a name="prerequisites"></a>Предварительные требования

Перед началом настройки убедитесь, что удовлетворены следующие требования:

[!INCLUDE [Before you begin](../../includes/virtual-wan-before-include.md)]

## <a name="create-a-virtual-wan"></a><a name="openvwan"></a>Создание виртуальной глобальной сети

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-create-vwan-include.md)]

## <a name="create-a-hub"></a><a name="hub"></a>Создание концентратора

Концентратор — это виртуальная сеть, которая может содержать шлюзы для подключений "сеть — сеть", "точка — сеть" или каналов ExpressRoute. За созданный концентратор будет взиматься плата, даже если вы не присоединяете сайты. Создание VPN-шлюза типа "сеть — сеть" в виртуальном концентраторе занимает 30 минут.

[!INCLUDE [Create a hub](../../includes/virtual-wan-tutorial-s2s-hub-include.md)]

## <a name="create-a-site"></a><a name="site"></a>Создание сайта

С помощью инструкций из этого раздела вы создадите сайт. Сайты соответствуют вашим физическим расположениям. Создайте столько сайтов, сколько необходимо. Например, если у вас есть филиал в Нью-Йорке, в Лондоне, а также в Лос-Анджелесе, создайте три отдельных сайта. Эти сайты содержат конечные точки локального VPN-устройства. В виртуальной глобальной сети можно создать до 1000 сайтов на виртуальный концентратор. Если у вас несколько концентраторов, можно создать по 1000 сайтов для каждого из них. Если у вас есть устройство CPE от партнера виртуальной глобальной сети, ознакомьтесь с ним, чтобы узнать о его автоматизации в Azure. Как правило, автоматизация подразумевает простой процесс для экспорта крупномасштабных сведений о филиалах в Azure и настройку подключения CPE к VPN-шлюзу Виртуальной глобальной сети Azure. Дополнительные сведения см. в разделе [Участники Виртуальной глобальной сети](virtual-wan-configure-automation-providers.md).

[!INCLUDE [Create a site](../../includes/virtual-wan-tutorial-s2s-site-include.md)]

## <a name="connect-the-vpn-site-to-the-hub"></a><a name="connectsites"></a>Подключение сайта VPN к концентратору

На этом шаге вы подключаете сайт VPN к концентратору.

[!INCLUDE [Connect VPN sites](../../includes/virtual-wan-tutorial-s2s-connect-vpn-site-include.md)]

## <a name="connect-the-vnet-to-the-hub"></a><a name="vnet"></a>Подключение виртуальной сети к концентратору

[!INCLUDE [Connect](../../includes/virtual-wan-connect-vnet-hub-include.md)]

## <a name="download-vpn-configuration"></a><a name="device"></a>Скачивание конфигурации VPN

Используйте конфигурацию VPN-устройства для настройки локального VPN-устройства.

1. На странице своей виртуальной глобальной сети щелкните **Обзор**.
2. В верхней части страницы **Концентратор > VPNSite** (Сайт VPN) щелкните **Скачать конфигурацию VPN**. Azure создает учетную запись хранения в группе ресурсов microsoft-network-[расположение], где используется расположение глобальной сети. Применив конфигурацию к VPN-устройствам, вы можете удалить эту учетную запись хранения.
3. После завершения создания файла вы можете скачать его, щелкнув ссылку.
4. Примените конфигурацию к локальному VPN-устройству.

### <a name="about-the-vpn-device-configuration-file"></a>О файле конфигурации VPN-устройства

Файл конфигурации устройства содержит параметры, которые необходимо использовать при настройке локального VPN-устройства. При просмотре этого файла обратите внимание на следующие сведения:

* **vpnSiteConfiguration**. В этом разделе указаны сведения об устройстве, настроенные как сайт, подключенный к виртуальной глобальной сети. Они содержат имя и общедоступный IP-адрес устройства филиала.
* **vpnSiteConnections**. В этом разделе представлены следующие параметры:

    * **Адресное пространство** виртуальной сети виртуальных концентраторов.<br>Пример:
 
        ```
        "AddressSpace":"10.1.0.0/24"
        ```
    * **Адресное пространство** виртуальных сетей, подключенных к концентратору.<br>Пример

         ```
        "ConnectedSubnets":["10.2.0.0/16","10.3.0.0/16"]
         ```
    * **IP-адреса** VPN-шлюзов виртуального концентратора. Так как каждое подключение VPN-шлюза состоит из двух туннелей в конфигурации "активный — активный", в этом файле будут указаны оба IP-адреса. В данном примере вы видите Instance0 и Instance1 для каждого сайта.<br>Пример

        ``` 
        "Instance0":"104.45.18.186"
        "Instance1":"104.45.13.195"
        ```
    * **Сведения о конфигурации подключения к Vpngateway**, такие как протокол BGP, общий ключ и т. д. PSK — это автоматически созданный общий ключ. Вы всегда можете изменить подключение на странице обзора для настраиваемого ключа PSK.
  
### <a name="example-device-configuration-file"></a>Пример файла конфигурации устройства

  ```
  { 
      "configurationVersion":{ 
         "LastUpdatedTime":"2018-07-03T18:29:49.8405161Z",
         "Version":"r403583d-9c82-4cb8-8570-1cbbcd9983b5"
      },
      "vpnSiteConfiguration":{ 
         "Name":"testsite1",
         "IPAddress":"73.239.3.208"
      },
      "vpnSiteConnections":[ 
         { 
            "hubConfiguration":{ 
               "AddressSpace":"10.1.0.0/24",
               "Region":"West Europe",
               "ConnectedSubnets":[ 
                  "10.2.0.0/16",
                  "10.3.0.0/16"
               ]
            },
            "gatewayConfiguration":{ 
               "IpAddresses":{ 
                  "Instance0":"104.45.18.186",
                  "Instance1":"104.45.13.195"
               }
            },
            "connectionConfiguration":{ 
               "IsBgpEnabled":false,
               "PSK":"bkOWe5dPPqkx0DfFE3tyuP7y3oYqAEbI",
               "IPsecParameters":{ 
                  "SADataSizeInKilobytes":102400000,
                  "SALifeTimeInSeconds":3600
               }
            }
         }
      ]
   },
   { 
      "configurationVersion":{ 
         "LastUpdatedTime":"2018-07-03T18:29:49.8405161Z",
         "Version":"1f33f891-e1ab-42b8-8d8c-c024d337bcac"
      },
      "vpnSiteConfiguration":{ 
         "Name":" testsite2",
         "IPAddress":"66.193.205.122"
      },
      "vpnSiteConnections":[ 
         { 
            "hubConfiguration":{ 
               "AddressSpace":"10.1.0.0/24",
               "Region":"West Europe"
            },
            "gatewayConfiguration":{ 
               "IpAddresses":{ 
                  "Instance0":"104.45.18.187",
                  "Instance1":"104.45.13.195"
               }
            },
            "connectionConfiguration":{ 
               "IsBgpEnabled":false,
               "PSK":"XzODPyAYQqFs4ai9WzrJour0qLzeg7Qg",
               "IPsecParameters":{ 
                  "SADataSizeInKilobytes":102400000,
                  "SALifeTimeInSeconds":3600
               }
            }
         }
      ]
   },
   { 
      "configurationVersion":{ 
         "LastUpdatedTime":"2018-07-03T18:29:49.8405161Z",
         "Version":"cd1e4a23-96bd-43a9-93b5-b51c2a945c7"
      },
      "vpnSiteConfiguration":{ 
         "Name":" testsite3",
         "IPAddress":"182.71.123.228"
      },
      "vpnSiteConnections":[ 
         { 
            "hubConfiguration":{ 
               "AddressSpace":"10.1.0.0/24",
               "Region":"West Europe"
            },
            "gatewayConfiguration":{ 
               "IpAddresses":{ 
                  "Instance0":"104.45.18.187",
                  "Instance1":"104.45.13.195"
               }
            },
            "connectionConfiguration":{ 
               "IsBgpEnabled":false,
               "PSK":"YLkSdSYd4wjjEThR3aIxaXaqNdxUwSo9",
               "IPsecParameters":{ 
                  "SADataSizeInKilobytes":102400000,
                  "SALifeTimeInSeconds":3600
               }
            }
         }
      ]
   }
  ```

### <a name="configuring-your-vpn-device"></a>Настройка VPN-устройства

>[!NOTE]
> Если вы работаете с партнерским решением Виртуальной глобальной сети, VPN-устройство будет настроено автоматически. Контроллер устройства получает файл конфигурации из Azure и применяет его к устройству, чтобы настроить подключение к Azure. Это означает, что вам не нужно вручную настраивать VPN-устройство.
>

Если вам нужны инструкции по настройке устройства, см. раздел [Загрузка сценариев конфигурации VPN-устройства из Azure](~/articles/vpn-gateway/vpn-gateway-about-vpn-devices.md#configscripts). При этом учтите следующее:

* Инструкции на странице VPN-устройств не предназначены для Виртуальной глобальной сети, но вы можете использовать значения Виртуальной глобальной сети из файла конфигурации для настройки VPN-устройства вручную. 
* Загружаемые скрипты конфигурации устройства, предназначенные для VPN-шлюза, не подходят для Виртуальной глобальной сети, так как ее конфигурация отличается.
* Новая Виртуальная глобальная сеть поддерживает IKEv1 и IKEv2.
* Виртуальная глобальная сеть может использовать политики VPN-устройства на основе маршрута и инструкции устройства.

## <a name="configure-your-vpn-gateway"></a><a name="gateway-config"></a>Настройка VPN-шлюза

Вы можете просмотреть и настроить параметры VPN-шлюза в любое время, выбрав **Просмотр и настройка**.

:::image type="content" source="media/virtual-wan-site-to-site-portal/view-configuration-1.png" alt-text="Снимок экрана: страница &quot;VPN (сеть-сеть)&quot;, на которой стрелка указывает на действие &quot;Просмотр и настройка&quot;" lightbox="media/virtual-wan-site-to-site-portal/view-configuration-1-expand.png":::

На странице **Изменение VPN-шлюза** можно посмотреть следующие параметры:

* Общедоступный IP-адрес VPN-шлюза (назначен Azure).
* Частный IP-адрес VPN-шлюза (назначен Azure).
* IP-адрес BGP по умолчанию для VPN-шлюза (назначен Azure).
* Параметр конфигурации для настраиваемого IP-адреса BGP — это поле зарезервировано для APIPA (автоматическое назначение частных IP-адресов). Azure поддерживает IP-адреса BGP в диапазонах 169.254.21.* и 169.254.22.*. Azure принимает подключения по протоколу BGP в этих диапазонах, но инициирует соединение по IP-адресу BGP по умолчанию.

   :::image type="content" source="media/virtual-wan-site-to-site-portal/view-configuration-2.png" alt-text="Просмотр конфигурации" lightbox="media/virtual-wan-site-to-site-portal/view-configuration-2-expand.png":::

## <a name="clean-up-resources"></a><a name="cleanup"></a>Очистка ресурсов

Если созданные ресурсы вам больше не нужны, удалите их. Некоторые ресурсы Виртуальной глобальной сети необходимо удалять в определенном порядке с учетом зависимостей. Удаление может занять около 30 минут.

[!INCLUDE [Delete resources](../../includes/virtual-wan-resource-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Виртуальной глобальной сети см. в следующей статье:

> [!div class="nextstepaction"]
> * [Вопросы и ответы о Виртуальной глобальной сети](virtual-wan-faq.md)
