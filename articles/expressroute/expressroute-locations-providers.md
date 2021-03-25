---
title: Расположения и поставщики услуг подключения для Azure ExpressRoute | Документация Майкрософт
description: В этой статье приведена подробная информация о расположениях, где предлагаются услуги, и способах подключения к регионам Azure. Эта таблица отсортирована по расположению.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 02/10/2021
ms.author: duau
ms.openlocfilehash: a43f95ad65e95db2b69b32c3fe8d62db71a98a17
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105025209"
---
# <a name="expressroute-partners-and-peering-locations"></a>Партнеры и одноранговые расположения ExpressRoute

> [!div class="op_single_selector"]
> * [Расположения по поставщикам](expressroute-locations.md)
> * [Поставщики по расположению](expressroute-locations-providers.md)


В этой статье приведены таблицы со сведениями о географическом покрытии и расположениях ExpressRoute, поставщиках услуг подключения и системных интеграторах ExpressRoute.

> [!Note]
> Регионы Azure и расположения ExpressRoute — это два отдельных понятия с существенными различиями. Понимание этих различий крайне важно для изучения гибридных сетевых подключений Azure. 
>
>

## <a name="azure-regions"></a>Регионы Azure
Регионы Azure — это глобальные центры обработки данных, в которых размещаются ресурсы вычислений, сети и хранилища Azure. При создании ресурса Azure клиенту необходимо выбрать расположение ресурса. Расположение ресурса определяет, в каком центре обработки данных Azure (или в какой зоне доступности) создается этот ресурс.

## <a name="expressroute-locations"></a>Расположения ExpressRoute
Расположения ExpressRoute (которые иногда называют расположениями пиринга) — это объекты совместного размещения, в которых установлены устройства Microsoft Enterprise Edge (MSEE). Расположения ExpressRoute являются точкой входа в сеть корпорации Майкрософт. Они распределены по всему миру, предоставляя клиентам возможность подключаться к сети корпорации Майкрософт в любом регионе. Именно в этих расположениях партнеры ExpressRoute и клиенты ExpressRoute Direct создают перекрестные подключения к сети корпорации Майкрософт. В общем случае расположения ExpressRoute не обязаны соответствовать регионам Azure. Например, клиент может создать канал ExpressRoute в расположении пиринга *Сиэтл* для доступа к ресурсу в регионе *Восточная часть США*.

Вы сможете получить доступ ко всем службам Azure во всех регионах соответствующего геополитического региона, если вы подключены по меньшей мере к одному расположению ExpressRoute в этом регионе. 

## <a name="azure-regions-to-expressroute-locations-within-a-geopolitical-region"></a><a name="locations"></a>Регионы Azure с расположениями ExpressRoute в пределах геополитических регионов
В следующей таблице сопоставлены регионы Azure с расположениями ExpressRoute в пределах геополитических регионов.

| **Геополитический регион** | **Регионы Azure** | **Расположения ExpressRoute** |
| --- | --- | --- |
| **Государственные организации Австралии** | Центральная Австралия, Центральная Австралия 2 |Канберра, Канберра 2 |
| **Европа** | Центральная Франция, Южная Франция, Северная Германия, Центрально-Западная Германия, Северная Европа, Восточная Норвегия, Западная Норвегия, Северная Швейцария, Западная Швейцария, западная часть Соединенного Королевства, южная часть Соединенного Королевства, Западная Европа |Амстердам, Amsterdam2, Берлин, Копенгаген, Дублин, Франкфурт, Frankfurt2, Geneva, Лондон, London2, Мадрид, Марселе, Милан, Мюнхене, Ньюпорт (), Oslo, Париж, Ставанжер, Стокгольм, Цюрих |
| **Северная Америка** | Восточная часть США, западная часть США, восточная часть США 2, западная часть США 2, центральная часть США, центрально-южная часть США, центрально-северная часть США, центрально-западная часть США, Центральная Канада, Восточная Канада |Атланта, Чикаго, Далласе, Денвер, Лас-Йорк, Лос-Анджелес, Лос Angeles2, Майами, Миннеаполисе, Монреаль, Нью-Йорк, Phoenix, Квебек City, Куеретаро (Мексика), Куинци, San Сан Антонио, Сиэтл, Silicon впадин, Silicon Valley2, Торонто, Toronto2, Vancouver, Вашингтон (округ Колумбия), Вашингтон |
| **Азия** | Восточная Азия, Юго-Восточная Азия | Бангкок, Гонконг, Гонконг 2, Джакарта, Куала-Лумпур, Сингапур, Сингапур 2, Тайбэй |
| **Индия** | Западная Индия, Центральная Индия, Южная Индия |Ченнаи, Ченнаи 2, Мумбаи, Мумбаи 2 |
| **Япония** | Западная Япония, Восточная Япония |Осака, Токио, Токио 2 |
| **Океания** | Восточная Австралия, Юго-Восточная Австралия |Окленд, Мельбурн, Перт, Сидней, Сидней 2 | 
| **Южная Корея** | Республика Корея, центральный регион, Республика Корея, южный регион |Пусан, Сеул|
| **ОАЭ** | Центральная часть ОАЭ, Северная часть ОАЭ | Дубаи, Дубаи 2 |
| **ЮАР** | Западная часть ЮАР, Северная часть ЮАР |Кейптаун, Йоханнесбург |
| **Южная Америка** | Южная Бразилия |Богота, Рио de де Жанейро, Сан-Паулу |

## <a name="azure-regions-and-geopolitical-boundaries-for-national-clouds"></a>Регионы Azure и геополитические границы для национальных облаков
В таблице ниже содержатся сведения о регионах и геополитических границах для национальных облаков.

| **Геополитический регион** | **Регионы Azure** | **Расположения ExpressRoute** |
| --- | --- | --- |
| **Облако US Gov** |US Gov (Аризона), US Gov (Айова), US Gov (Техас), US Gov (Вирджиния), центральный регион US DoD, восточный регион US DoD  |Атланта, Чикаго, Даллас, Нью-Йорк, Феникс, Сан-Антонио, Сиэтл, Кремниевая долина, Вашингтон (округ Колумбия) |
| **Восточный Китай** |"Восточный Китай", "Восточный Китай 2" |Шанхай, Шанхай 2 |
| **Северный Китай** |"Северный Китай", "Северный Китай 2" |Пекин, Пекин 2 |
| **Германия** |Центральная Германия, восточная Германия |Берлин, Франкфурт |

В стандартном номере SKU ExpressRoute подключение между геополитическими регионами не поддерживается. Для поддержки глобальных подключений необходимо включить надстройку ExpressRoute класса "Премиум". Подключение к национальным облачным средам не поддерживается. При необходимости вы можете работать с поставщиками услуг подключения.

## <a name="expressroute-connectivity-providers"></a><a name="partners"></a>Поставщики услуг подключения ExpressRoute

В таблице ниже приведены расположения для подключения и поставщики услуг в каждом расположении. Поставщики услуг и расположения, в которых они предоставляют услуги, перечислены в разделе [Расположения поставщиков услуг подключения](expressroute-locations.md).

* **Локальные регионы Azure** — это те, к которым могут получить доступ сети [ExpressRoute ценовой категории "Локальный"](expressroute-faqs.md) в каждом расположении пиринга. **н/д** указывает, что локальная сеть ExpressRoute недоступна в этом расположении пиринга.

* **Зоны** определяют [цены](https://azure.microsoft.com/pricing/details/expressroute/).


### <a name="global-commercial-azure"></a>Глобальная коммерческая версия Azure
| **Расположение** | **Адрес** | **Зона** | **Локальные регионы Azure** | **ER Direct** | **Поставщики услуг** |
| --- | --- | --- | --- | --- | --- |
| **Амстердам** | [Equinix AM5](https://www.equinix.com/locations/europe-colocation/netherlands-colocation/amsterdam-data-centers/am5/) | 1 | Западная Европа | 10G, 100G | Aryaka Networks, AT&T NetBond, British Telecom, Colt, Equinix, euNetworks, GÉANT, InterCloud, Interxion, KPN, IX Reach, Level 3 Communications, Megaport, NTT Communications, Orange, Tata Communications, Telefonica, Telenor, Telia Carrier, Verizon, Zayo |
| **Амстердам 2** | [Interxion AMS8](https://www.interxion.com/Locations/amsterdam/schiphol/) | 1 | Западная Европа | 10G, 100G | Британская телекоммуникации, CenturyLink Cloud Connect, Colt, DE-ЦИКС, Equinix, Еунетворкс, ГÉАНТ, Interxion, серия, NTT глобальные центры обработки данных EMEA, оранжевый, Vodafone |
| **Атланта** | [Equinix AT2](https://www.equinix.com/locations/americas-colocation/united-states-colocation/atlanta-data-centers/at2/) | 1 | Недоступно | 10G, 100G | Equinix, Megaport |
| **Окленд** | [Vocus Group NZ Albany](https://www.vocus.co.nz/business/cloud-data-centres) | 2 | Недоступно | 10G | Деволи, Кордиа, Orange, РЕАННЗ, Spark NZ, Вокус Group NZ |
| **Бангкок** | [AIS](https://business.ais.co.th/solution/en/azure-expressroute.html) | 2 | Недоступно | 10G | AIS, УИХ |
| **Берлин** | [NTT GDC](https://www.e-shelter.de/en/location/berlin-1-data-center) | 1 | Северная Германия | 10G | Equinix, NTT Global Datacenters EMEA|
| **Богота** | [Equinix BG1](https://www.equinix.com/locations/americas-colocation/colombia-colocation/bogota-data-centers/bg1/) | 4 | Недоступно | 10G | Equinix |
| **Пусан** | [LG CNS](https://www.lgcns.com/En/Service/DataCenter) | 2 | Республика Корея, южный регион | Недоступно | LG CNS |
| **Канберра** | [CDC](https://cdcdatacentres.com.au/content/about-cdc) | 1 | Центральная Австралия | 10G, 100G | CDC |
| **Канберра 2** | [CDC](https://cdcdatacentres.com.au/content/about-cdc) | 1 | Центральная Австралия 2| 10G, 100G | CDC, Equinix |
| **Кейптаун** | [Teraco CT1](https://www.teraco.co.za/data-centre-locations/cape-town/) | 3 | Западная часть ЮАР | 10G | BCX, Internet Solutions — Cloud Connect, Liquid Telecom, Teraco |
| **Ченнай** | Tata Communications | 2 | Южная Индия | 10G | BSNL, Global Клаудксчанже (ГККС), СИФИ, Tata Communications, Водафонеидеа |
| **Chennai2** | Airtel | 2 | Южная Индия | 10G | Airtel |
| **Чикаго** | [Equinix CH1](https://www.equinix.com/locations/americas-colocation/united-states-colocation/chicago-data-centers/ch1/) | 1 | Центрально-северная часть США | 10G, 100G | Aryaka Networks, AT&T NetBond, Британская телекоммуникации, CenturyLink Cloud Connect, Кологикс, Colt, Comcast, Коресите, Equinix, recloud, Internet2, уровень 3, взаимодействие, Orange, Паккетфабрик, ПККВ Global Limited, спринт, Телиа, Verizon, Zayo |
| **Копенгаген** | [Interxion CPH1](https://www.interxion.com/Locations/copenhagen/) | 1 | Недоступно | 10G | Interxion |
| **Даллас** | [Equinix DA3](https://www.equinix.com/locations/americas-colocation/united-states-colocation/dallas-data-centers/da3/) | 1 | Недоступно | 10G, 100G | Aryaka Networks, AT&T NetBond, Кологикс, Equinix, Internet2, уровень 3 связи, Orange, Неутрона Networks, Паккетфабрик, Телмекс Унинет, Телиа,. Telco, Verizon, Zayo|
| **Денвер** | [CoreSite DE1](https://www.coresite.com/data-centers/locations/denver/de1) | 1 | центрально-западная часть США | Недоступно | CoreSite, Megaport, Zayo |
| **Дубаи** | [PCCS](https://www.pacificcontrols.net/cloudservices/index.html) | 3 | Северная часть ОАЭ; | Недоступно | Etisalat UAE |
| **Дубаи 2** | [du datamena](http://datamena.com/solutions/data-centre) | 3 | Северная часть ОАЭ; | Недоступно | DE-ЦИКС, Du датамена, Equinix, Orange, оранжевый, Орикском |
| **Дублин** | [Equinix DB3](https://www.equinix.com/locations/europe-colocation/ireland-colocation/dublin-data-centers/db3/) | 1 | Северная Европа | 10G, 100G | CenturyLink Cloud Connect, Colt, еир, Equinix, ЖЕАНТ, Еунетворкс, Interxion, Orange |
| **Франкфурт** | [Interxion FRA11](https://www.interxion.com/Locations/frankfurt/) | 1 | Центрально-Западная Германия | 10G, 100G | В&T NetBond, CenturyLink Cloud Connect, Colt, DE-ЦИКС, Equinix, Еунетворкс, ЖЕАНТ, recloud, Interxion, Orange, оранжевый, Телиа перевозчика, T-Systems |
| **Frankfurt2** | [Equinix FR7](https://www.equinix.com/locations/europe-colocation/germany-colocation/frankfurt-data-centers/fr7/) | 1 | Центрально-Западная Германия | 10G, 100G | Equinix |
| **Женева** | [Equinix GV2](https://www.equinix.com/locations/europe-colocation/switzerland-colocation/geneva-data-centers/gv2/) | 1 | Западная Швейцария | 10G, 100G | Equinix, Orange, компания Swisscom |
| **Специальный административный регион Гонконг** | [Equinix HK1](https://www.equinix.com/locations/asia-colocation/hong-kong-colocation/hong-kong-data-center/hk1/) | 2 | Восточная Азия | 10G | Aryaka сети, Британская телекоммуникации, CenturyLink Cloud Connect, главный телекоммуникации, Международная телекоммуникации, Глобальная, Китайская, Уником, Equinix, взаимосвязь, Orange, NTT, оранжевый, ПККВ Global Limited, Tata, Телиа, Verizon |
| **Гонконг 2** | [MEGA-i](https://www.iadvantage.net/index.php/locations/mega-i) | 2 | Восточная Азия | 10G | Китайская, Международная, Китайская, Глобальная, Иадвантаже, Orange, ПККВ Global Limited, SingTel |
| **Джакарта** | Телин, для абонентов Telkom Индонезия | 4 | Недоступно | 10G | телин |
| **Йоханнесбург** | [Teraco JB1](https://www.teraco.co.za/data-centre-locations/johannesburg/#jb1) | 3 | Северная часть ЮАР; | 10G | BCX, British Telecom, Internet Solutions — Cloud Connect, Liquid Telecom, Orange, Teraco |
| **Куала-Лумпур** | [TIME dotCom Menara AIMS](https://www.time.com.my/enterprise/connectivity/direct-cloud) | 2 | Недоступно | Недоступно | TIME dotCom |
| **Лас-Вегас** | [Switch LV](https://www.switch.com/las-vegas) | 1 | Недоступно | 10G, 100G | CenturyLink Cloud Connect, Orange, Паккетфабрик |
| **Лондон** | [Equinix LD5](https://www.equinix.com/locations/europe-colocation/united-kingdom-colocation/london-data-centers/ld5/) | 1 | южная часть Соединенного Королевства | 10G, 100G | AT&T NetBond, British Telecom, Colt, Equinix, euNetworks, InterCloud, Internet Solutions - Cloud Connect, Interxion, Jisc, Level 3 Communications, Megaport, MTN, NTT Communications, Orange, PCCW Global Limited, Tata Communications, Telehouse - KDDI, Telenor, Telia Carrier, Verizon, Vodafone, Zayo |
| **Лондон 2** | [Telehouse North Two](https://www.telehouse.net/data-centres/emea/uk-data-centres/london-data-centres/north-two) | 1 | южная часть Соединенного Королевства | 10G, 100G | Британская телекоммуникации, CenturyLink Cloud Connect, Colt, ГТТ, IX REACH, Equinix, Orange, SES, Сохонет, штат дома-KDDI |
| **Лос-Анджелес** | [CoreSite LA1](https://www.coresite.com/data-centers/locations/los-angeles/one-wilshire) | 1 | Недоступно | 10G, 100G | CoreSite, Equinix, Megaport, Neutrona Networks, NTT, Zayo |
| **Лос-Angeles2** | [Equinix LA1](https://www.equinix.com/locations/americas-colocation/united-states-colocation/los-angeles-data-centers/la1/) | 1 | Недоступно | 10G, 100G | Equinix |
| **Мадрид** | [Interxion MAD1](https://www.interxion.com/es/donde-estamos/europa/madrid) | 1 | Западная Европа | 10G, 100G | |
| **Марсель** |[Interxion MRS1](https://www.interxion.com/Locations/marseille/) | 1 | Южная Франция | Недоступно | DE-CIX, GEANT, Interxion, Jaguar Network, Ooredoo Cloud Connect |
| **Мельбурн** | [NextDC M1](https://www.nextdc.com/data-centres/m1-melbourne-data-centre) | 2 | Юго-Восточная часть Австралии | 10G, 100G | AARNet, Devoli, Equinix, Megaport, NEXTDC, Optus, Telstra Corporation, TPG Telecom |
| **Майами** | [Equinix MI1](https://www.equinix.com/locations/americas-colocation/united-states-colocation/miami-data-centers/mi1/) | 1 | Недоступно | 10G, 100G | Claro, C3ntro, Equinix, Megaport, Neutrona Networks |
| **Милан** | [IRIDEOS](https://irideos.it/en/data-centers/) | 1 | Недоступно | 10G | Colt, Equinix, Фаствеб, ИРИДЕОС, Ретелит |
| **Миннеаполисе** | [Кологикс MIN1](https://www.cologix.com/data-centers/minneapolis/min1/) | 1 | Недоступно | 10G, 100G | Кологикс, Orange |
| **Монреаль** | [Cologix MTL3](https://www.cologix.com/data-centers/montreal/mtl3/) | 1 | Недоступно | 10G, 100G | Bell для Канады, Кологикс, Фибреноире, Orange, Телус, Zayo |
| **Мумбай** | Tata Communications | 2 | Западная Индия | 10G | BSNL, DE-ЦИКС, Global Клаудксчанже (ГККС), зависимость ЖИО, Сифи, Tata Communications, Verizon |
| **Мумбаи 2** | Airtel | 2 | Западная Индия | 10G | Airtel, Sify, Vodafone Idea |
| **Мюнхен** | [EdgeConneX](https://www.edgeconnex.com/locations/europe/munich/) | 1 | Недоступно | 10G | DE-CIX |
| **Нью-Йорк** | [Equinix NY9](https://www.equinix.com/locations/americas-colocation/united-states-colocation/new-york-data-centers/ny9/) | 1 | Недоступно | 10G, 100G | CenturyLink Cloud Connect, Colt, Коресите, DE-ЦИКС, Equinix, recloud, Orange, пакет, Zayo |
| **Ньюпорт (Уэльс)** | [Next Generation Data](https://www.nextgenerationdata.co.uk) | 1 | западная часть Соединенного Королевства | Недоступно | Британская телекоммуникации, Colt, JISC, связь уровня 3, данные следующего поколения |
| **Осака** | [Equinix OS1](https://www.equinix.com/locations/asia-colocation/japan-colocation/osaka-data-centers/os1/) | 2 | Западная Япония | 10G, 100G | В Токио, Colt, Equinix, Интернет-инициатива Япония Inc. — IIJ, Orange, NTT Communications, NTT Смартконнект, Softbank ", Токаи Communications |
| **Осло** | [DigiPlex Ulven](https://www.digiplex.com/locations/oslo-datacentre) | 1 | Восточная Норвегия; | 10G, 100G | Глобалконнект, Orange, Telenor, оператор Телиа |
| **Париж** | [Interxion PAR5](https://www.interxion.com/Locations/paris/) | 1 | Центральная Франция | 10G, 100G | Британская телекоммуникации, CenturyLink Cloud Connect, Colt, Equinix, в облаке, Interxion, Жагуар Network, Orange, оранжевый, Телиа, Zayo |
| **Перт** | [NextDC P1](https://www.nextdc.com/data-centres/p1-perth-data-centre) | 2 | Недоступно | 10G | Megaport, NextDC |
| **Phoenix** | [Еджеконнекс PHX01](https://www.edgeconnex.com/locations/north-america/phoenix-az/) | 1 | Недоступно | 10G, 100G | |
| **Квебек** | [Vantage](https://vantage-dc.com/data_centers/quebec-city-data-center-campus/) | 1 | Восточная Канада | 10G, 100G | Bell для Канады, Orange, Телус |
| **Куеретаро (Мексика)** | [Кио сети QR01](https://www.kionetworks.com/es-mx/) | 4 | Недоступно | 10G | Transtelco|
| **куинци** | [Сабэй Datacenter — создание](https://sabeydatacenters.com/data-center-locations/central-washington-data-centers/quincy-data-center) | 1 | западная часть США 2 | 10G, 100G | |
| **Rio de Janeiro** | [Equinix — RJ2](https://www.equinix.com/locations/americas-colocation/brazil-colocation/rio-de-janeiro-data-centers/rj2/) | 3 | Юго-Восточная Бразилия | 10G | Equinix |
| **Сан-Антонио** | [CyrusOne SA1](https://cyrusone.com/locations/texas/san-antonio-texas/) | 1 | Центрально-южная часть США | 10G, 100G | CenturyLink Cloud Connect, Megaport |
| **Сан-Паулу** | [Equinix SP2](https://www.equinix.com/locations/americas-colocation/brazil-colocation/sao-paulo-data-centers/sp2/) | 3 | Южная Бразилия | 10G, 100G | Aryaka Networks, Ascenty Data Centers, British Telecom, Equinix, Level 3 Communications, Neutrona Networks, Orange, Tata Communications, Telefonica, UOLDIVEO |
| **Сиэтл;** | [Equinix SE2](https://www.equinix.com/locations/americas-colocation/united-states-colocation/seattle-data-centers/se2/) | 1 | западная часть США 2 | 10G, 100G | Aryaka Networks, Equinix, Level 3 Communications, Megaport, Telus, Zayo |
| **Сеул** | [KINX Gasan IDC](https://www.kinx.net/?lang=en) | 2 | Республика Корея, центральный регион | 10G, 100G | КИНКС, KT, LG CNS, Equinix, Сежонг телекоммуникации |
| **Кремниевая долина** | [Equinix SV1](https://www.equinix.com/locations/americas-colocation/united-states-colocation/silicon-valley-data-centers/sv1/) | 1 | западная часть США | 10G, 100G | Aryaka Networks, AT&T NetBond, British Telecom, CenturyLink Cloud Connect, Colt, Comcast, Coresite, Equinix, InterCloud, Internet2, IX Reach, Packet, PacketFabric, Level 3 Communications, Megaport, Orange, Sprint, Tata Communications, Telia Carrier, Verizon, Zayo |
| **Кремниевая долина 2** | [Coresite SV7](https://www.coresite.com/data-centers/locations/silicon-valley/sv7) | 1 | западная часть США | 10G, 100G | Colt, Coresite | 
| **Сингапур** | [Equinix SG1](https://www.equinix.com/locations/asia-colocation/singapore-colocation/singapore-data-center/sg1/) | 2 | Юго-Восточная Азия | 10G, 100G | Aryaka Networks, AT&T NetBond, British Telecom, China Mobile International, Epsilon Global Communications, Equinix, InterCloud, Level 3 Communications, Megaport, NTT Communications, Orange, SingTel, Tata Communications, Telstra Corporation, Verizon, Vodafone |
| **Сингапур 2** | [Global Switch Tai Seng](https://www.globalswitch.com/locations/singapore-data-centres/) | 2 | Юго-Восточная Азия | 10G, 100G | Уником: глобальное, Colt, Эпсилон, глобальная связь, Orange, ПККВ глобальные ограничения, SingTel, дом — KDDI |
| **Ставангер** | [Green Mountain DC1](https://greenmountain.no/dc1-stavanger/) | 1 | Западная Норвегия | 10G, 100G |Глобалконнект, Orange |
| **Стокгольм** | [Equinix SK1](https://www.equinix.com/locations/europe-colocation/sweden-colocation/stockholm-data-centers/sk1/) | 1 | Недоступно | 10G | Equinix, Telia Carrier |
| **Сидней** | [Equinix SY2](https://www.equinix.com/locations/asia-colocation/australia-colocation/sydney-data-centers/sy2/) | 2 | Восточная Австралия | 10G, 100G | AARNet, AT&T NetBond, British Telecom, Devoli, Equinix, Kordia, Megaport, NEXTDC, NTT Communications, Optus, Orange, Spark NZ, Telstra Corporation, TPG Telecom, Verizon, Vocus Group NZ |
| **Сидней 2** | [NextDC S1](https://www.nextdc.com/data-centres/s1-sydney-data-centre) | 2 | Восточная Австралия | 10G, 100G | Megaport, NextDC |
| **Тайбэй** | Chief Telecom | 2 | Недоступно | 10G | Chief Telecom, Chunghwa Telecom, FarEasTone |
| **Токио** | [Equinix TY4](https://www.equinix.com/locations/asia-colocation/japan-colocation/tokyo-data-centers/ty4/) | 2 | Восточная Япония | 10G, 100G | Aryaka Networks, AT&T NetBond, BBIX, British Telecom, CenturyLink Cloud Connect, Colt, Equinix, Internet Initiative Japan Inc. — IIJ, Megaport, NTT Communications, NTT EAST, Orange, Softbank, Verizon |
| **Токио 2** | [AT TOKYO](https://www.attokyo.com/) | 2 | Восточная Япония | 10G, 100G | В Токио, Orange, Токаи Communications |
| **Торонто** | [Cologix TOR1](https://www.cologix.com/data-centers/toronto/tor1/) | 1 | Центральная Канада | 10G, 100G | AT&T NetBond, Bell Canada, CenturyLink Cloud Connect, Cologix, Equinix, IX Reach Megaport, Telus, Verizon, Zayo |
| **Toronto2** | [Allied РЕИТ](https://www.alliedreit.com/property/905-king-st-w/) | 1 | Центральная Канада | 10G, 100G | |
| **Ванкувер** | [Cologix VAN1](https://www.cologix.com/data-centers/vancouver/van1/) | 1 | Недоступно | 10G | Кологикс, Orange, Телус |
| **Вашингтон, округ Колумбия** | [Equinix DC2](https://www.equinix.com/locations/americas-colocation/united-states-colocation/washington-dc-data-centers/dc2/) | 1 | Восточная часть США, восточная часть США 2 | 10G, 100G | Aryaka Networks, AT&T NetBond, Британская телекоммуникации, CenturyLink Cloud Connect, Кологикс, Colt, Comcast, Коресите, Equinix, Internet2, в облаке, железо,, доступ уровня 3, Orange, Неутрона Networks, NTT Communications, оранжевый, паккетфабрик, SES, спринт, Tata, Телиа,, Verizon, Zayo |
| **Вашингтон, округ Колумбия 2** | [Coresite Reston](https://www.coresite.com/data-center-locations/northern-virginia-washington-dc) | 1 | Восточная часть США, восточная часть США 2 | 10G, 100G | CenturyLink Cloud Connect, Коресите, ИНТЕЛСАТ, Orange, ВИАСАТ, Zayo | 
| **Цюрих** | [Interxion ZUR2](https://www.interxion.com/Locations/zurich/) | 1 | Северная Швейцария | 10G, 100G | Equinix, Interxion, Orange, компания Swisscom |

 **+** означает "скоро"

### <a name="national-cloud-environments"></a>Национальные облачные среды

Национальные облака Azure изолированы друг от друга и от глобальной коммерческой службы Azure. ExpressRoute для одного облака Azure не может подключаться к регионам Azure в других облаках.

### <a name="us-government-cloud"></a>Облако US Gov
| **Расположение** | **Адрес** | **Локальные регионы Azure**| **ER Direct** | **Поставщики услуг** |
| --- | --- | --- | --- | --- |
| **Атланта** | [Equinix AT1](https://www.equinix.com/locations/americas-colocation/united-states-colocation/atlanta-data-centers/at1/) | Недоступно | 10G, 100G | Equinix |
| **Чикаго** | [Equinix CH1](https://www.equinix.com/locations/americas-colocation/united-states-colocation/chicago-data-centers/ch1/) | Недоступно | 10G, 100G | В&T NetBond, Британская телекоммуникации, Equinix, связь уровня 3, Verizon |
| **Даллас** | [Equinix DA3](https://www.equinix.com/locations/americas-colocation/united-states-colocation/dallas-data-centers/da3/) | Недоступно | 10G, 100G | Equinix, Megaport, Verizon |
| **Нью-Йорк** | [Equinix NY5](https://www.equinix.com/locations/americas-colocation/united-states-colocation/new-york-data-centers/ny5/) | Недоступно | 10G, 100G | Equinix, CenturyLink Cloud Connect, Verizon |
| **Phoenix** | [CyrusOne Chandler](https://cyrusone.com/locations/arizona/phoenix-arizona-chandler/) | US Gov (Аризона) | 10G, 100G | AT&T NetBond, CenturyLink Cloud Connect, Megaport |
| **Сан-Антонио** | [CyrusOne SA2](https://cyrusone.com/locations/texas/san-antonio-texas-ii/) | US Gov (Техас) | 10G, 100G | CenturyLink Cloud Connect, Megaport |
| **Кремниевая долина** | [Equinix SV4](https://www.equinix.com/locations/americas-colocation/united-states-colocation/silicon-valley-data-centers/sv4/) | Недоступно | 10G, 100G | AT&T, Equinix, Level 3 Communications, Verizon |
| **Сиэтл;** | [Equinix SE2](https://www.equinix.com/locations/americas-colocation/united-states-colocation/seattle-data-centers/se2/) | Недоступно | 10G, 100G | Equinix, Megaport |
| **Вашингтон, округ Колумбия** | [Equinix DC2](https://www.equinix.com/locations/americas-colocation/united-states-colocation/washington-dc-data-centers/dc2/) | Восточный регион US DoD, US Gov (Вирджиния) | 10G, 100G | AT&T NetBond, CenturyLink Cloud Connect, Equinix, Level 3 Communications, Megaport, Verizon |

### <a name="china"></a>Китай
| **Расположение** | **Поставщики услуг** |
| --- | --- |
| **Пекин** |China Telecom |
| **Пекин 2** | China Telecom, China Unicom, GDS |
| **Шанхай** |China Telecom |
| **Шанхай 2** | China Telecom, China Unicom, GDS |

Дополнительные сведения см. на странице [ExpressRoute в Китае](http://www.windowsazure.cn/home/features/expressroute/).

### <a name="germany"></a>Германия
| **Расположение** | **Поставщики услуг** |
| --- | --- |
| **Берлин** |e-shelter, Megaport +, T-Systems |
| **Франкфурт** |Colt, Equinix, Interxion |

## <a name="connectivity-through-exchange-providers"></a><a name="c1partners"></a>Подключение через поставщиков Exchange
Вы можете создать подключение, даже если ваш поставщик услуг подключения не указан в предыдущих разделах.

* Узнайте у своего поставщика услуг подключения, подключен ли он к какому-либо Exchange, указанному в таблице выше. Дополнительные сведения об услугах, предлагаемых поставщиками Exchange, см. по ссылкам ниже. Несколько поставщиков услуг подключения уже подключены к серверам Ethernet Exchange.
  * [Cologix](https://www.cologix.com/)
  * [CoreSite](https://www.coresite.com/)
  * [DE-CIX](https://www.de-cix.net/en/de-cix-service-world/cloud-exchange)
  * [Equnix Cloud Exchange](https://www.equinix.com/services/interconnection-connectivity/cloud-exchange/)
  * [InterXion](https://www.interxion.com/)
  * [NextDC](https://www.nextdc.com/)
  * [Megaport](https://www.megaport.com/services/microsoft-expressroute/)
  * [PacketFabric](https://www.packetfabric.com/cloud-connectivity/microsoft-azure)
  * [Teraco](https://www.teraco.co.za/platform-teraco/africa-cloud-exchange/)
  
* Обратитесь к своему поставщику услуг подключения, чтобы он расширил вашу сеть, добавив необходимое пиринговое расположение.
  * Убедитесь, что поставщик услуг подключения расширяет границы вашего подключения, сохраняя высокую доступность во избежание влияния единых точек отказа.
* Закажите канал ExpressRoute с Exchange у вашего поставщика услуг подключения для связи с Майкрософт.
  * Для настройки подключения выполните действия, описанные в статье [Создание и изменение канала ExpressRoute с помощью PowerShell](expressroute-howto-circuit-classic.md) .

## <a name="connectivity-through-satellite-operators"></a>Подключение через операторов спутниковой связи
Если вам нужно удаленное подключение там, где недоступна оптоволоконная технология, или вы просто интересуетесь новыми вариантами подключения, изучите предложения операторов спутниковой связи. 

* Intelsat
* [SES](https://www.ses.com/networks/signature-solutions/signature-cloud/ses-and-azure-expressroute)
* [Viasat](http://www.directcloud.viasatbusiness.com/)

## <a name="connectivity-through-additional-service-providers"></a><a name="c1partners"></a>Подключение через дополнительных поставщиков услуг
| **Расположение** | **Exchange** | **Поставщики услуг подключения** |
| --- | --- | --- |
| **Амстердам** | Equinix, Interxion, Level 3 Communications | BICS, CloudXpress, Eurofiber, Fastweb S.p.A, Gulf Bridge International, Kalaam Telecom Bahrain B.S.C, MainOne, Nianet, POST Telecom Luxembourg, Proximus, RETN, TDC Erhverv, Telecom Italia Sparkle, Telekom Deutschland GmbH, Telia |
| **Атланта** | Equinix| Crown Castle
| **Кейптаун** | Teraco | MTN |
| **Чикаго** | Equinix| Crown Castle, Spectrum Enterprise, Windstream |
| **Даллас** | Equinix, Megaport | Axtel, C3ntro Telecom, Cox Business, Crown Castle, Data Foundry, Spectrum Enterprise, Transtelco |
| **Франкфурт** | Interxion | BICS, Cinia, Equinix, Nianet, QSC AG, Telekom Deutschland GmbH |
| **Гамбург** | Equinix | Cinia |
| **Гонконг (САР)** | Equinix | Chief, Macroview Telecom |
| **Йоханнесбург** | Teraco | MTN |
| **Лондон** | BICS, Equinix, euNetworks| Bezeq International Ltd., CoreAzure, Epsilon Telecommunications Limited, Exponential E, HSO, NexGen Networks, Proximus, Tamares Telecom, Zain |
| **Лос-Анджелес** | Equinix |Crown Castle, Spectrum Enterprise, Transtelco |
| **Мадрид** | Level3 | Zertia |
| **Монреаль** | Cologix| Airgate Technologies, Inc. Aptum Technologies, Rogers, Zirro |
| **Нью-Йорк** |Equinix, Megaport | Altice Business, Crown Castle, Spectrum Enterprise, Webair |
| **Париж** | Equinix | Proximus |
| **Квебек** | Megaport | Fibrenoire |
| **Сан-Паулу** | Equinix | Venha Pra Nuvem |
| **Сиэтл;** |Equinix | Alaska Communications |
| **Кремниевая долина** |Coresite, Equinix | Cox Business, Spectrum Enterprise, Windstream, X2nsat Inc. |
| **Сингапур** |Equinix |1CLOUDSTAR, BICS, CMC Telecom, Epsilon Telecommunications Limited, LGA Telecom, United Information Highway (UIH) |
| **Slough** | Equinix | HSO|
| **Сидней** | Megaport | Macquarie Telecom Group|
| **Токио** | Equinix | ARTERIA Networks Corporation, BroadBand Tower, Inc. |
| **Торонто** | Equinix, Megaport | Технологии аиргате Inc., Беанфиелд Метроконнект, Аптум Technologies, Иведха Inc, oncore облачные службы Inc., Рожерс, Синктел, Зирро|
| **Вашингтон, округ Колумбия** |Equinix | Altice Business, BICS, Cox Business, Crown Castle, Gtt Communications Inc, Epsilon Telecommunications Limited, Masergy, Windstream |

## <a name="expressroute-system-integrators"></a>Системные интеграторы ExpressRoute
Возможность частного подключения, соответствующего вашим потребностям, будет зависеть от масштаба сети. Чтобы упростить переход на ExpressRoute, вы можете обратиться к одному из системных интеграторов, указанных в таблице ниже.

| **Континент** | **Системные интеграторы** |
| --- | --- |
| **Азия** |Avanade Inc., OneAs1a |
| **Австралия** | Ensyst, IT Consultancy, MOQdigital, Vigilant.IT |
| **Европа** |Avanade Inc., Altogee, Bright Skies GmbH, Inframon, MSG Services, New Signature, Nelite, Orange Networks, sol-tec |
| **Северная Америка** |Avanade Inc., Equinix Professional Services, FlexManage, Lightstream, Perficient, Presidio |
| **Южная Америка** |Avanade Inc., Venha Pra Nuvem |
## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения об ExpressRoute см. в статье [Вопросы и ответы по ExpressRoute](expressroute-faqs.md).
* Убедитесь, что выполнены все необходимые условия. Ознакомьтесь с разделом [Предварительные требования и контрольный список для ExpressRoute](expressroute-prerequisites.md).

<!--Image References-->
[0]: ./media/expressroute-locations/expressroute-locations-map.png "Карта расположения"
