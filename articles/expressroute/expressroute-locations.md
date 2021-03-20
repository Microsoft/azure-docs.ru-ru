---
title: Поставщики услуг подключения и расположения для Azure ExpressRoute | Документация Майкрософт
description: В этой статье приведена подробная информация о расположениях, где предлагаются услуги, и способах подключения к регионам Azure. Эта таблица отсортирована по поставщику услуг подключения.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 02/10/2021
ms.author: duau
ms.openlocfilehash: 4a75931017bbde24c17ecdc0da498547c6088682
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104581530"
---
# <a name="expressroute-connectivity-partners-and-peering-locations"></a>Партнеры по подключению и одноранговые расположения ExpressRoute

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

## <a name="azure-regions-to-expressroute-locations-within-a-geopolitical-region"></a><a name="locations"></a>Регионы Azure с расположениями ExpressRoute в пределах геополитических регионов.
В следующей таблице сопоставлены регионы Azure с расположениями ExpressRoute в пределах геополитических регионов.

| **Геополитический регион** | **Регионы Azure** | **Расположения ExpressRoute** |
| --- | --- | --- |
| **Государственные организации Австралии** |Центральная Австралия, Центральная Австралия 2 |Канберра, Канберра 2 |
| **Европа** | Центральная Франция, Южная Франция, Северная Германия, Центрально-Западная Германия, Северная Европа, Восточная Норвегия, Западная Норвегия, Северная Швейцария, Западная Швейцария, западная часть Соединенного Королевства, южная часть Соединенного Королевства, Западная Европа |Амстердам, Amsterdam2, Берлин, Копенгаген, Дублин, Франкфурт, Frankfurt2, Geneva, Лондон, London2, Мадрид, Марселе, Милан, Мюнхене, Ньюпорт (), Oslo, Париж, Ставанжер, Стокгольм, Цюрих |
| **Северная Америка** |Восточная часть США, западная часть США, восточная часть США 2, западная часть США 2, центральная часть США, центрально-южная часть США, центрально-северная часть США, центрально-западная часть США, Центральная Канада, Восточная Канада |Атланта, Чикаго, Далласе, Денвер, Лас-Йорк, Лос-Анджелес, Лос Angeles2, Майами, Миннеаполисе, Монреаль, Нью-Йорк, Phoenix, Квебек City, Куеретаро (Мексика), Куинци, San Сан Антонио, Сиэтл, Silicon впадин, Silicon Valley2, Торонто, Toronto2, Vancouver, Вашингтон (округ Колумбия), Вашингтон |
| **Азия** | Восточная Азия, Юго-Восточная Азия | Бангкок, Гонконг, Гонконг 2, Джакарта, Куала-Лумпур, Сингапур, Сингапур 2, Тайбэй |
| **Индия** | Западная Индия, Центральная Индия, Южная Индия |Ченнаи, Ченнаи 2, Мумбаи, Мумбаи 2 |
| **Япония** | Западная Япония, Восточная Япония |Осака, Токио, Токио 2 |
| **Океания** | Восточная Австралия, Юго-Восточная Австралия |Окленд, Мельбурн, Перт, Сидней, Сидней 2 |
| **Южная Корея** | Республика Корея, центральный регион, Республика Корея, южный регион |Пусан, Сеул|
| **ОАЭ** | Центральная часть ОАЭ, Северная часть ОАЭ | Дубаи, Дубаи 2 |
| **ЮАР** | Западная часть ЮАР, Северная часть ЮАР |Кейптаун, Йоханнесбург |
| **Южная Америка** | Южная Бразилия |Богота, Рио de де Жанейро, Сан-Паулу |


## <a name="regions-and-geopolitical-boundaries-for-national-clouds"></a>Регионы и геополитические границы для национальных облаков
В таблице ниже содержатся сведения о регионах и геополитических границах для национальных облаков.

| **Геополитический регион** | **Регионы Azure** | **Расположения ExpressRoute** |
| --- | --- | --- |
| **Облако US Gov** |US Gov (Аризона), US Gov (Айова), US Gov (Техас), US Gov (Вирджиния), центральный регион US DoD, восточный регион US DoD  |Атланта, Чикаго, Даллас, Нью-Йорк, Феникс, Сан-Антонио, Сиэтл, Кремниевая долина, Вашингтон (округ Колумбия) |
| **Восточный Китай** |"Восточный Китай", "Восточный Китай 2" |Шанхай, Шанхай 2 |
| **Северный Китай** |"Северный Китай", "Северный Китай 2" |Пекин, Пекин 2 |
| **Германия** |Центральная Германия, восточная Германия |Берлин, Франкфурт |

В стандартном номере SKU ExpressRoute подключение между геополитическими регионами не поддерживается. Для поддержки глобальных подключений необходимо включить надстройку ExpressRoute класса "Премиум". Подключение к национальным облачным средам не поддерживается. При необходимости вы можете работать с поставщиками услуг подключения.

## <a name="expressroute-connectivity-providers"></a><a name="partners"></a>Поставщики услуг подключения ExpressRoute

В следующей таблице показаны расположения по поставщикам услуг. См. дополнительные сведения о [службах доступных поставщиков по расположению](expressroute-locations-providers.md).


### <a name="global-commercial-azure"></a>Глобальная коммерческая версия Azure

| **Поставщик услуг** | **Microsoft Azure** | **Microsoft 365**  | **Расположения** |
| --- | --- | --- | --- |
| **[AARNet](https://www.aarnet.edu.au/network-and-services/connectivity-services/azure-expressroute)** |Поддерживается |Поддерживается |Мельбурн, Сидней |
| **[Airtel](https://www.airtel.in/business/#/)** | Поддерживается | Поддерживается | Ченнаи 2, Мумбаи 2 |
| **[AIS](https://business.ais.co.th/solution/en/azure-expressroute.html)** | Поддерживается | Поддерживается | Бангкок |
| **[Aryaka Networks](https://www.aryaka.com/)** |Поддерживается |Поддерживается |Амстердам, Чикаго, Даллас, Гонконг (САР), Сан-Паулу, Сиэтл, Кремниевая долина, Сингапур, Токио, Вашингтон (округ Колумбия) |
| **[Ascenty Data Centers](https://www.ascenty.com/en/cloud/microsoft-express-route)** |Поддерживается |Поддерживается |Сан-Паулу |
| **[AT&T NetBond](https://www.synaptic.att.com/clouduser/html/productdetail/ATT_NetBond.htm)** |Поддерживается |Поддерживается |Амстердам, Чикаго, Даллас, Франкфурт, Лондон, Кремниевая долина, Сингапур, Сидней, Токио, Торонто, Вашингтон (округ Колумбия) |
| **[AT TOKYO](https://www.attokyo.com/connectivity/azure.html)** | Поддерживается | Поддерживается | Осака, Tokyo2 |
| **[BBIX](https://www.bbix.net/en/service/ix/)** | Поддерживается | Поддерживается | Токио |
| **[BCX](https://www.bcx.co.za/solutions/connectivity/data-networks)** |Поддерживается |Поддерживается |Кейптаун, Йоханнесбург|
| **[Bell Canada](https://business.bell.ca/shop/enterprise/cloud-connect-access-to-cloud-partner-services)** |Поддерживается |Поддерживается |Квебек, Монреаль, Торонто |
| **[British Telecom](https://www.globalservices.bt.com/en/solutions/products/cloud-connect-azure)** |Поддерживается |Поддерживается |Амстердам, Amsterdam2, Чикаго, Гонконг, САР, Йоханнесбург, Лондон, London2, Ньюпорт (штат Вашингтон), Париж, Сан-Паулу, Silicon впадина, Сингапур, Сидней, Токио, Вашингтон (округ Колумбия) |
| **[BSNL](https://www.bsnl.co.in/opencms/bsnl/BSNL/services/enterprises/cloudway.html)** |Поддерживается |Поддерживается |Ченнаи, Мумбаи |
| **[C3ntro](https://www.c3ntro.com/data1/express-route1.php)** |Поддерживается |Поддерживается |Майами |
| **CDC** | Поддерживается | Поддерживается | Канберра, Канберра 2 |
| **[CenturyLink Cloud Connect](https://www.centurylink.com/cloudconnect)** |Поддерживается |Поддерживается |Amsterdam2, Чикаго, Дублин, Франкфурт, Гонконг, Лас деньги, London2, Нью Йорк, Париж, Сан Сан Антонио, Silicon впадина, Токио, Торонто, Вашингтон (округ Колумбия), штат Вашингтон |
| **[Chief Telecom](https://www.chief.com.tw/)** |Поддерживается |Поддерживается |Гонконг, Тайбэй |
| **China Mobile International** |Поддерживается |Поддерживается | Гонконг, Гонконг, Kong2, Сингапур |
| **China Telecom Global** |Поддерживается |Поддерживается |Гонконг, Гонконг 2 |
| **China Unicom Global** |Поддерживается |Поддерживается | Гонконг, Сингапур 2 |
| **[Chunghwa Telecom](https://www.cht.com.tw/en/home/cht/about-cht/products-and-services/International/Cloud-Service)** |Поддерживается |Поддерживается |Тайбэй |
| **[Claro](https://www.usclaro.com/enterprise-mnc/connectivity/mpls/)** |Поддерживается |Поддерживается |Майами |
| **[Cologix](https://www.cologix.com/hyperscale/microsoft-azure/)** |Поддерживается |Поддерживается |Чикаго, Далласе, Миннеаполисе, Монреаль, Торонто, Vancouver, Вашингтон (округ Колумбия) |
| **[Colt](https://www.colt.net/direct-connect/azure/)** |Поддерживается |Поддерживается |Амстердам, Amsterdam2, Чикаго, Дублин, Франкфурт, Лондон, London2, Милан, Ньюпорт, Нью Йорк, Осака, Париж, впадина, Silicon Valley2, Сингапур 2, Токио, Вашингтон (округ Колумбия) |
| **[Comcast](https://business.comcast.com/landingpage/microsoft-azure)** |Поддерживается |Поддерживается |Чикаго, Кремниевая долина, Вашингтон, округ Колумбия |
| **[CoreSite](https://www.coresite.com/solutions/cloud-services/public-cloud-providers/microsoft-azure-expressroute)** |Поддерживается |Поддерживается |Чикаго, Денвер, Лос-Анджелес, Нью-Йорк, Кремниевая долина, Кремниевая долина 2, Вашингтон (округ Колумбия), Вашингтон (округ Колумбия) 2 |
| **[DE-CIX](https://www.de-cix.net/en/de-cix-service-world/cloud-exchange/find-a-cloud-service/detail/microsoft-azure)** | Поддерживается |Поддерживается |Amsterdam2, Dubai2, Франкфурт, Марселе, Мумбаи, Мюнхене, Нью Йорк |
| **[Devoli](https://devoli.com/expressroute)** | Поддерживается |Поддерживается | Окленд, Мельбурн, Сидней |
| **du datamena** |Поддерживается |Поддерживается | Дубаи 2 |
| **eir** |Поддерживается |Поддерживается |Дублин|
| **[Epsilon Global Communications](https://www.epsilontel.com/solutions/direct-cloud-connect)** |Поддерживается |Поддерживается |Сингапур, Сингапур 2 |
| **[Equinix](https://www.equinix.com/partners/microsoft-azure/)** |Поддерживается |Поддерживается |Амстердам, Amsterdam2, Атланта, Берлин, Богота, Canberra2, Чикаго, Далласе, Dubai2, Дублин, Франкфурт, Frankfurt2, Geneva, Гонконг, САР, Лондон, London2, Лос-Анджелес, Лос Angeles2, Мельбурн, Майами, Милан, Нью-Йорк, Осака, Париж, Рио де де Жанейро, Сан-Паулу, Сиэтл, Сеул, Торонто, Сингапур, Вашингтон, округ Колумбия, Цюрих |
| **Etisalat UAE** |Поддерживается |Поддерживается |Дубаи|
| **[euNetworks](https://eunetworks.com/services/solutions/cloud-connect/microsoft-azure-expressroute/)** |Поддерживается |Поддерживается |Амстердам, Amsterdam2, Дублин, Франкфурт, Лондон |
| **[FarEasTone](https://www.fetnet.net/corporate/en/Enterprise.html)** |Поддерживается |Поддерживается |Тайбэй|
| **[фаствеб](https://www.fastweb.it/grandi-aziende/cloud/scheda-prodotto/fastcloud-interconnect/)** | Поддерживается |Поддерживается |Milan|
| **[Fibrenoire](https://fibrenoire.ca/en/services/cloudextn-2/)** |Поддерживается |Поддерживается |Монреаль|
| **[GÉANT](https://www.geant.org/Networks)** |Поддерживается |Поддерживается |Амстердам, Amsterdam2, Дублин, Франкфурт, Марселе |
| **[глобалконнект]()** | Поддерживается |Поддерживается | Oslo, Ставанжер | 
| **гтт** |Поддерживается |Поддерживается |Лондон 2 |
| **[Global Cloud Xchange (GCX)](https://globalcloudxchange.com/cloud-platform/cloud-x-fusion/)** | Поддерживается| Поддерживается | Ченнаи, Мумбаи |
| **[иадвантаже](https://www.scx.sunevision.com/)** | Поддерживается | Поддерживается | Гонконг 2 |
| **Intelsat** | Поддерживается | Поддерживается | Вашингтон (округ Колумбия) 2 |
| **[InterCloud](https://www.intercloud.com/)** |Поддерживается |Поддерживается |Амстердам, Чикаго, Франкфурт, Гонконг, Лондон, Нью Йорк, Париж, Silicon впадина, Сингапур, Вашингтон, округ Колумбия, Цюрих |
| **[Internet2](https://internet2.edu/services/cloud-connect/#service-cloud-connect)** |Поддерживается |Поддерживается |Чикаго, Даллас, Кремниевая долина, Вашингтон (округ Колумбия) |
| **[Internet Initiative Japan Inc. - IIJ](https://www.iij.ad.jp/en/news/pressrelease/2015/1216-2.html)** |Поддерживается |Поддерживается |Осака, Токио |
| **[Internet Solutions - Cloud Connect](https://www.is.co.za/solution/cloud-connect/)** |Поддерживается |Поддерживается |Кейптаун, Йоханнесбург, Лондон |
| **[Interxion](https://www.interxion.com/why-interxion/colocate-with-the-clouds/Microsoft-Azure/)** |Поддерживается |Поддерживается |Амстердам, Амстердам 2, Копенгаген, Дублин, Франкфурт, Лондон, Марсель, Париж, Цюрих |
| **[IRIDEOS](https://irideos.it/)** |Поддерживается |Поддерживается |Milan |
| **Железное горное** | Поддерживается |Поддерживается |Вашингтон, округ Колумбия |
| **[IX Reach](https://www.ixreach.com/partners/cloud-partners/microsoft-azure/)**|Поддерживается |Поддерживается | Амстердам, Лондон 2, Кремниевая долина, Торонто, Вашингтон (округ Колумбия) |
| **Jaguar Network** |Поддерживается |Поддерживается |Марселе, Париж |
| **[Jisc](https://www.jisc.ac.uk/microsoft-azure-expressroute)** |Поддерживается |Поддерживается |Лондон, Ньюпорт (то есть) |
| **[KINX](https://www.kinx.net/service/network/cloudhub/ms-expressroute/?lang=en)** |Поддерживается |Поддерживается |Сеул |
| **[Kordia](https://www.kordia.co.nz/cloudconnect)** | Поддерживается |Поддерживается |Окленд, Сидней |
| **[KPN](https://www.kpn.com/zakelijk/cloud/connect.htm)** | Поддерживается | Поддерживается | Амстердам |
| **[KT](https://cloud.kt.com/)** | Поддерживается | Поддерживается | Сеул |
| **[Level 3 Communications](http://your.level3.com/LP=882?WT.tsrc=02192014LP882AzureVanityAzureText)** |Поддерживается |Поддерживается |Амстердам, Вашингтон (округ Колумбия), Даллас, Кремниевая долина, Лондон, Ньюпорт (Уэльс), Сан-Паулу, Сингапур, Сиэтл, Чикаго |
| **LG CNS** |Поддерживается |Поддерживается |Пусан, Сеул |
| **[Liquid Telecom](https://www.liquidtelecom.com/products-and-services/cloud.html)** |Поддерживается |Поддерживается |Кейптаун, Йоханнесбург |
| **[Megaport](https://www.megaport.com/services/microsoft-expressroute/)** |Поддерживается |Поддерживается |Амстердам, Атланта, Ченнаи, Чикаго, Далласе, Денвер, Dubai2, Дублин, Франкфурт, Geneva, Гонконг, Гонконг Kong2, Лас-деньги, Лондон, London2, Лос-Анджелес, Мельбурн, Майами, Миннеаполисе, Монреаль, Нью-Йорк, Осака, Oslo, Париж, Перт, город Квебек, Сан-Сан Антонио, Сиэтл,, Токио, Сингапур 2, ставанжер, Стокгольм, Сидней, Вашингтон |
| **[MTN](https://www.mtnbusiness.co.za/en/Cloud-Solutions/Pages/microsoft-express-route.aspx)** |Поддерживается |Поддерживается |London |
| **[Neutrona Networks](https://www.neutrona.com/index.php/azure-expressroute/)** |Поддерживается |Поддерживается |Даллас, Лос-Анджелес, Майами, Сан-Паулу, Вашингтон (округ Колумбия) |
| **[Next Generation Data](https://vantage-dc-cardiff.co.uk/)** |Поддерживается |Поддерживается |Ньюпорт (Уэльс) |
| **[NEXTDC](https://www.nextdc.com/services/axon-ethernet/microsoft-expressroute)** |Поддерживается |Поддерживается |Мельбурн, Перт, Сидней, Сидней 2 |
| **[Нумерация](https://www.nos.pt/empresas/corporate/cloud/cloud/Pages/nos-cloud-connect.aspx)** |Поддерживается |Поддерживается |Амстердам 2 |
| **[NTT Communications](https://www.ntt.com/en/services/network/virtual-private-network.html)** |Поддерживается |Поддерживается |Амстердам, Гонконг (САР), Лондон, Лос-Анджелес, Осака, Сингапур, Сидней, Токио, Вашингтон (округ Колумбия) |
| **[NTT EAST](https://business.ntt-east.co.jp/service/crossconnect/)** |Поддерживается |Поддерживается |Токио |
| **[Глобальные центры обработки данных NTT в регионе "Европа, Ближний Восток и Африка"](https://hello.global.ntt/)** |Поддерживается |Поддерживается |Amsterdam2, Берлин |
| **[NTT SmartConnect](https://cloud.nttsmc.com/cxc/azure.html)** |Поддерживается |Поддерживается |Осака |
| **[Ooredoo Cloud Connect](https://www.ooredoo.qa/portal/OoredooQatar/cloud-connect-expressroute)** |Поддерживается |Поддерживается |Марсель |
| **[Optus](https://www.optus.com.au/enterprise/)** |Поддерживается |Поддерживается |Мельбурн, Сидней |
| **[Orange](https://www.orange-business.com/en/products/business-vpn-galerie)** |Поддерживается |Поддерживается |Амстердам, Амстердам 2, Дубаи 2, Франкфурт, Гонконг (САР), Йоханнесбург, Лондон, Париж, Сан-Паулу, Кремниевая долина, Сингапур, Сидней, Токио, Вашингтон (округ Колумбия) |
| **[Orixcom](https://www.orixcom.com/cloud-solutions/)** | Поддерживается | Поддерживается | Дубаи 2 |
| **[PacketFabric](https://www.packetfabric.com/cloud-connectivity/microsoft-azure)** |Поддерживается |Поддерживается |Чикаго, Далласе, Лас — деньги, полупроводниковый округ, округ Колумбия |
| **[PCCW Global Limited](https://consoleconnect.com/clouds/#azureRegions)** |Поддерживается |Поддерживается |Чикаго, Гонконг, Гонконг 2, Лондон, Сингапур 2 |
| **[реаннз](https://www.reannz.co.nz/products-and-services/cloud-connect/)** | Поддерживается | Поддерживается | Окленд |
| **[ретелит](https://www.retelit.it/EN/Home.aspx)** | Поддерживается | Поддерживается | Milan | 
| **[Sejong Telecom](https://www.sejongtelecom.net/en/pages/service/cloud_ms)** |Поддерживается |Поддерживается |Сеул |
| **[SES](https://www.ses.com/networks/signature-solutions/signature-cloud/ses-and-azure-expressroute)** | Поддерживается |Поддерживается | London2, Вашингтон (округ Колумбия) |
| **[SIFY](http://telecom.sify.com/azure-expressroute.html)** |Поддерживается |Поддерживается |Ченнаи, Мумбаи 2 |
| **[SingTel](https://www.singtel.com/about-us/news-releases/singtel-provide-secure-private-access-microsoft-azure-public-cloud)** |Поддерживается |Поддерживается |Гонконг, Kong2, Сингапур, Сингапур 2 |
| **[Softbank](https://www.softbank.jp/biz/cloud/cloud_access/direct_access_for_az/)** |Поддерживается |Поддерживается |Осака, Токио |
| **[сохонет](https://www.sohonet.com/fastlane/)** |Поддерживается |Поддерживается |Лондон 2 |
| **[Spark NZ](https://www.sparkdigital.co.nz/solutions/connectivity/cloud-connect/)** |Поддерживается |Поддерживается |Окленд, Сидней |
| **[Спринт](https://business.sprint.com/solutions/cloud-networking/)** |Поддерживается |Поддерживается |Чикаго, Кремниевая долина, Вашингтон, округ Колумбия |
| **[Swisscom](https://www.swisscom.ch/en/business/enterprise/offer/cloud-data-center/microsoft-cloud-services/microsoft-azure-von-swisscom.html)** | Поддерживается | Поддерживается | Geneva, Цюрих |
| **[Tata Communications](https://www.tatacommunications.com/lp/izo/azure/azure_index.html)** |Поддерживается |Поддерживается |Амстердам, Ченнаи, Гонконг (САР), Лондон, Мумбаи, Сан-Паулу, Кремниевая долина, Сингапур, Вашингтон (округ Колумбия) |
| **[Telefonica](https://www.business-solutions.telefonica.com/es/enterprise/solutions/efficient-infrastructure/managed-voice-data-connectivity/)** |Поддерживается |Поддерживается |Амстердам, Сан-Паулу |
| **[Telehouse — KDDI](https://www.telehouse.net/solutions/cloud-services/cloud-link)** |Поддерживается |Поддерживается |Лондон, London2, Сингапур 2 |
| **Telenor** |Поддерживается |Поддерживается |Амстердам, Лондон, Осло |
| **[Telia Carrier](https://www.teliacarrier.com/)** | Поддерживается | Поддерживается |Амстердам, Чикаго, Даллас, Франкфурт, Гонконг, Лондон, Осло, Париж, Кремниевая долина, Стокгольм, Вашингтон (округ Колумбия) |
| **[телин](https://www.telin.net/)** | Поддерживается | Поддерживается |Джакарта |
| **Telmex Uninet**| Поддерживается | Поддерживается | Даллас |
| **[Telstra Corporation](https://www.telstra.com.au/business-enterprise/network-services/networks/cloud-direct-connect/)** |Поддерживается |Поддерживается |Мельбурн, Сингапур, Сидней |
| **[Telus](https://www.telus.com)** |Поддерживается |Поддерживается |Монреаль, Сиэтл, Квебек City, Торонто, Vancouver |
| **[Teraco](https://www.teraco.co.za/services/africa-cloud-exchange/)** |Поддерживается |Поддерживается |Кейптаун, Йоханнесбург |
| **[TIME dotCom](https://www.time.com.my/enterprise/connectivity/direct-cloud)** | Поддерживается | Поддерживается | Куала-Лумпур |
| **[Токаи связь](https://www.tokai-com.co.jp/en/)** | Поддерживается | Поддерживается | Осака, Tokyo2 |
| **[Transtelco](https://transtelco.net/enterprise-services/)** |Поддерживается |Поддерживается |Далласе, Куеретаро (Мексика)|
| **T-Systems** |Поддерживается |Поддерживается |Франкфурт|
| **[UOLDIVEO](https://www.uoldiveo.com.br/)** |Поддерживается |Поддерживается |Сан-Паулу |
| **[уих](https://www.uih.co.th/en/network-solutions/global-network/cloud-direct-for-microsoft-azure-expressroute)** | Поддерживается | Поддерживается | Бангкок |
| **[Verizon](https://enterprise.verizon.com/products/network/application-enablement/secure-cloud-interconnect/)** |Поддерживается |Поддерживается |Амстердам, Чикаго, Даллас, Гонконг (САР), Лондон, Мумбаи, Кремниевая долина, Сингапур, Сидней, Токио, Торонто, Вашингтон (округ Колумбия) |
| **[Viasat](http://www.directcloud.viasatbusiness.com/)** | Поддерживается | Поддерживается | Вашингтон (округ Колумбия) 2 |
| **[Vocus Group NZ](https://www.vocus.co.nz/business/cloud-data-centres)** | Поддерживается | Поддерживается | Окленд, Сидней |
| **[Vodafone](https://www.vodafone.com/business/global-enterprise/global-connectivity/vodafone-ip-vpn-cloud-connect)** |Поддерживается |Поддерживается |Амстердам 2, Лондон, Сингапур |
| **[Vodafone Idea](https://www.vodafone.in/business/enterprise-solutions/connectivity/vpn-extended-connect)** | Поддерживается | Поддерживается | Мумбаи 2 |
| **[Zayo](https://www.zayo.com/solutions/industries/cloud-connectivity/microsoft-expressroute)** |Поддерживается |Поддерживается |Амстердам, Чикаго, Даллас, Денвер, Лондон, Лос-Анджелес, Монреаль, Нью-Йорк, Париж, Сиэтл, Кремниевая долина, Торонто, Вашингтон (округ Колумбия), Вашингтон (округ Колумбия) 2 |

 **+** означает "скоро"

### <a name="national-cloud-environment"></a>Национальная облачная среда

Национальные облака Azure изолированы друг от друга и от глобальной коммерческой службы Azure. ExpressRoute для одного облака Azure не может подключаться к регионам Azure в других облаках. 

### <a name="us-government-cloud"></a>Облако US Gov

| **Поставщик услуг** | **Microsoft Azure** | **Office 365** | **Расположения** |
| --- | --- | --- | --- |
| **[AT&T NetBond](https://www.synaptic.att.com/clouduser/html/productdetail/ATT_NetBond.htm)** |Поддерживается |Поддерживается |Чикаго, Феникс, Кремниевая долина, Вашингтон (округ Колумбия) |
| **[CenturyLink Cloud Connect](https://www.centurylink.com/cloudconnect)** |Поддерживается |Поддерживается |Нью Йорк, Феникс, Сан-Антонио, Вашингтон (округ Колумбия) |
| **[Equinix](https://www.equinix.com/partners/microsoft-azure/)** |Поддерживается |Поддерживается |Атланта, Чикаго, Далласе, Нью Йорк, Сиэтл, Silicon впадина, Вашингтон (округ Колумбия) |
| **[Level 3 Communications](http://your.level3.com/LP=882?WT.tsrc=02192014LP882AzureVanityAzureText)** |Поддерживается |Поддерживается |Чикаго, Кремниевая долина, Вашингтон, округ Колумбия |
| **[Megaport](https://www.megaport.com/services/microsoft-expressroute/)** |Поддерживается | Поддерживается | Чикаго, Даллас, Сан-Антонио, Сиэтл, Вашингтон (округ Колумбия) |
| **[Verizon](http://news.verizonenterprise.com/2014/04/secure-cloud-interconnect-solutions-enterprise/)** |Поддерживается |Поддерживается |Чикаго, Даллас, Нью-Йорк, Кремниевая долина, Вашингтон (округ Колумбия) |

### <a name="china"></a>Китай

| **Поставщик услуг** | **Microsoft Azure** | **Office 365** | **Расположения** |
| --- | --- | --- | --- |
| **China Telecom** |Поддерживается |Не поддерживается |Пекин, Пекин 2, Шанхай, Шанхай 2 |
| **China Unicom** | Поддерживается | Не поддерживается | Пекин 2, Шанхай 2 |
| **[GDS](http://www.gds-services.com/en/about_2.html)** |Поддерживается |Не поддерживается |Пекин 2, Шанхай 2 |

Дополнительные сведения см. на странице [ExpressRoute в Китае](http://www.windowsazure.cn/home/features/expressroute/).

### <a name="germany"></a>Германия

| **Поставщик услуг** | **Microsoft Azure** | **Office 365** | **Расположения** |
| --- | --- | --- | --- |
| **[Colt](https://www.colt.net/direct-connect/azure/)** |Поддерживается |Не поддерживается |Франкфурт |
| **[Equinix](https://www.equinix.com/partners/microsoft-azure/)** |Поддерживается |Не поддерживается |Франкфурт |
| **[e-shelter](https://www.e-shelter.de/en/microsoft-expressroute)** |Поддерживается |Не поддерживается |Берлин |
| **Interxion** |Поддерживается |Не поддерживается |Франкфурт |
| **[Megaport](https://www.megaport.com/services/microsoft-expressroute/)** |Поддерживается  | Не поддерживается | Берлин |
| **T-Systems** |Поддерживается |Не поддерживается |Берлин |

## <a name="connectivity-through-exchange-providers"></a>Подключение через поставщиков Exchange

Вы можете создать подключение, даже если ваш поставщик услуг подключения не указан в предыдущих разделах.

* Узнайте у своего поставщика услуг подключения, подключен ли он к какому-либо Exchange, указанному в таблице выше. Дополнительные сведения об услугах, предлагаемых поставщиками Exchange, см. по ссылкам ниже. Несколько поставщиков услуг подключения уже подключены к серверам Ethernet Exchange.
  * [Cologix](https://www.cologix.com/)
  * [CoreSite](https://www.coresite.com/)
  * [DE-CIX](https://www.de-cix.net/en/de-cix-service-world/cloud-exchange)
  * [Equnix Cloud Exchange](https://www.equinix.com/services/interconnection-connectivity/cloud-exchange/)
  * [Interxion](https://www.interxion.com/products/interconnection/cloud-connect/)
  * [IX Reach](https://www.ixreach.com/partners/cloud-partners/microsoft-azure/)
  * [Megaport](https://www.megaport.com/services/microsoft-expressroute/)
  * [NextDC](https://www.nextdc.com/)
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

## <a name="connectivity-through-additional-service-providers"></a>Подключение через дополнительных поставщиков услуг

| **Поставщик услуг подключения** | **Exchange** | **Расположения** |
| --- | --- | --- |
| **[1CLOUDSTAR](https://www.1cloudstar.com/service/cloudconnect-azure-expressroute/)** |Equinix |Сингапур |
| **[Airgate Technologies, Inc.](https://www.airgate.ca/)** | Equinix, Cologix | Торонто, Монреаль |
| **[Alaska Communications](https://www.alaskacommunications.com/Business)** |Equinix |Seattle |
| **[Altice Business](https://golightpath.com/transport)** |Equinix |Нью-Йорк, Вашингтон (округ Колумбия) |
| **[Arteria Networks Corporation](https://www.arteria-net.com/business/service/cloud/sca/)** |Equinix |Токио |
| **[Axtel](https://alestra.mx/landing/expressrouteazure/)** |Equinix |Даллас|
| **[Beanfield Metroconnect](https://www.beanfield.com/business/cloud-exchange)** |Megaport |Toronto|
| **[Bezeq International Ltd.](https://www.bezeqint.net/english)** | euNetworks | London |
| **[BICS](https://bics.com/bics-solutions-suite/cloud-connect/)** | Equinix | Амстердам, Франкфурт, Лондон, Сингапур, Вашингтон (округ Колумбия) |https://www.chief.com.tw/
| **[BroadBand Tower, Inc.](https://www.bbtower.co.jp/product-service/data-center/network/dcconnect-for-azure/)** | Equinix | Токио |
| **[C3ntro Telecom](https://www.c3ntro.com/data/express-route)** | Equinix, Megaport | Даллас |
| **[Chief]()** | Equinix | Гонконг, САР |
| **[Cinia](https://www.cinia.fi/en/services/connectivity-services/direct-public-cloud-connection.html)** | Equinix, Megaport | Франкфурт, Германия |
| **[CloudXpress](https://www2.telenet.be/fr/business/produits-services/internet/cloudxpress/)** | Equinix | Амстердам | 
| **[CMC Telecom](https://cmctelecom.vn/san-pham/value-added-service-and-it/cmc-telecom-cloud-express-en/)** | Equinix | Сингапур | 
| **[Aptum Technologies](https://aptum.com/services/cloud/managed-azure/)**| Equinix | Монреаль, Торонто |
| **[CoreAzure](https://www.coreazure.com/)**| Equinix | London |
| **[Cox Business](https://www.cox.com/business/networking/cloud-connectivity.html)**| Equinix | Даллас, Кремниевая долина, Вашингтон, округ Колумбия |
| **[Crown Castle](https://fiber.crowncastle.com/solutions/added/cloud-connect)**| Equinix | Атланта, Чикаго, Даллас, Лос-Анджелес, Нью-Йорк, Вашингтон (округ Колумбия) |
| **[Data Foundry](https://www.datafoundry.com/services/cloud-connect)** | Megaport | Даллас |
| **[Epsilon Telecommunications Limited](https://www.epsilontel.com/solutions/cloud-connect/)** | Equinix | Лондон, Сингапур, Вашингтон, округ Колумбия |
| **[Eurofiber](https://eurofiber.nl/microsoft-azure/)** | Equinix | Амстердам |
| **[Exponential E](https://www.exponential-e.com/services/connectivity-services/cloud-connect-exchange)** | Equinix | London |
| **[Fastweb S.p.A](https://www.fastweb.it/grandi-aziende/connessione-voce-e-wifi/scheda-prodotto/rete-privata-virtuale/)** | Equinix | Амстердам |
| **[Fibrenoire](https://www.fibrenoire.ca/en/cloudextn)** | Megaport | Город Квебек |
| **[Gtt Communications Inc](https://www.gtt.net)** |Equinix | Вашингтон, округ Колумбия |
| **[Gulf Bridge International](https://www.gbiinc.com/microsoft-azure-expressroute/)** | Equinix | Амстердам |
| **[HSO](https://www.hso.co.uk/products/cloud-direct)** |Equinix | Лондон, Слау |
| **[IVedha Inc](http://www.ivedha.com/cloud/manage-azure-cloud/express-route-4/)**| Equinix | Toronto |
| **[Kaalam Telecom Bahrain B.S.C](http://www.kalaam-telecom.com/azure/)**| Level 3 Communications |Амстердам |
| **LGA Telecom** |Equinix |Сингапур|
| **[Macroview Telecom](http://www.macroview.com/en/scripts/catitem.php?catid=solution&sectionid=expressroute)** |Equinix |Гонконг, САР 
| **[Macquarie Telecom Group](https://macquariegovernment.com/secure-cloud/secure-cloud-exchange/)** | Megaport | Сидней |
| **[MainOne](https://www.mainone.net/services/connectivity/cloud-connect/)** |Equinix | Амстердам |
| **[Masergy](https://www.masergy.com/solutions/hybrid-networking/cloud-marketplace/microsoft-azure)** | Equinix | Вашингтон, округ Колумбия |
| **[MTN](https://www.mtnbusiness.co.za/en/Cloud-Solutions/Pages/microsoft-express-route.aspx)** | Teraco | Кейптаун, Йоханнесбург |
| **[NexGen Networks](https://www.nexgen-net.com/nexgen-networks-direct-connect-microsoft-azure-expressroute.html)** | Interxion | London |
| **[Nianet](https://nianet.dk/produkter/internet/microsoft-expressroute)** |Equinix | Амстердам, Франкфурт |
| **[Основная облачная служба Inc](https://www.oncore.cloud/services/ue-for-expressroute)**| Equinix | Toronto |
| **[POST Telecom Luxembourg](https://www.teralinksolutions.com/cloud-connectivity/cloudbridge-to-azure-expressroute/)**|Equinix | Амстердам |
| **[Proximus](https://www.proximus.be/en/id_b_cl_proximus_external_cloud_connect/companies-and-public-sector/discover/magazines/expert-blog/proximus-external-cloud-connect.html)**|Equinix | Амстердам, Дублин, Лондон, Париж |
| **[QSC AG](https://www.qsc.de/de/produkte-loesungen/cloud-services-und-it-outsourcing/pure-enterprise-cloud/multi-cloud-management/azure-expressroute/)** |Interxion | Франкфурт |  
| **[RETN](https://retn.net/services/cloud-connect/)** | Equinix | Амстердам |
| **Rogers** | Cologix, Equinix | Монреаль, Торонто |
| **[Spectrum Enterprise](https://enterprise.spectrum.com/services/cloud/cloud-connect.html)** | Equinix | Чикаго, Даллас, Лос-Анджелес, Нью-Йорк, Кремниевая долина | 
| **[Tamares Telecom](http://www.tamarestelecom.com/our-services/#Connectivity)** | Equinix | London | 
| **[TDC Erhverv](https://tdc.dk/Produkter/cloudaccessplus)** | Equinix | Амстердам | 
| **[Telecom Italia Sparkle](https://www.tisparkle.com/our-platform/corporate-platform/sparkle-cloud-connect#catalogue)**| Equinix | Амстердам |
| **[Telekom Deutschland GmbH](https://cloud.telekom.de/de/infrastruktur/managed-it-services/managed-hybrid-infrastructure-mit-microsoft-azure)** | Interxion | Амстердам, Франкфурт |
| **[Telia](https://www.telia.se/foretag/losningar/produkter-tjanster/datanet)** | Equinix | Амстердам |
| **[ThinkTel](https://www.thinktel.ca/services/agile-ix-data/expressroute/)** | Equinix | Toronto | 
| **[United Information Highway (UIH)](https://www.uih.co.th/en/internet-solution/cloud-direct/uih-cloud-direct-for-microsoft-azure-expressroute)**| Equinix | Сингапур |
| **[Venha Pra Nuvem](https://venhapranuvem.com.br/)** | Equinix | Сан-Паулу |
| **[Webair](https://www.webair.com/microsoft-express-route-partnership/)**| Megaport | Нью-Йорк |
| **[Windstream](https://www.windstreambusiness.com/solutions/cloud-services/cloud-and-managed-hosting-services)**| Equinix | Чикаго, Кремниевая долина, Вашингтон, округ Колумбия |
| **[X2nsat Inc.](https://www.x2nsat.com/expressroute/)** |Coresite |Кремниевая долина, Кремниевая долина 2|
| **Zain** |Equinix |London|
| **[Zertia](https://www.zertia.es)**| Level 3 | Мадрид |
| **[Zirro](https://zirro.com/services/)**| Cologix, Equinix | Монреаль, Торонто |

## <a name="connectivity-through-datacenter-providers"></a>Подключение с помощью поставщиков центров обработки данных

| **Поставщик** | **Exchange** |
| --- | --- |
| **[CyrusOne](https://cyrusone.com/enterprise-data-center-services/connectivity-and-interconnection/cloud-connectivity-reaching-amazon-microsoft-google-and-more/microsoft-azure-expressroute/?doing_wp_cron=1498512235.6733090877532958984375)** | Megaport, PacketFabric |
| **[Cyxtera](https://www.cyxtera.com/data-center-services/interconnection)** | Megaport, PacketFabric |
| **[Databank](https://www.databank.com/platforms/connectivity/cloud-direct-connect/)** | Megaport |
| **[DataFoundry](https://www.datafoundry.com/services/cloud-connect/)** | Megaport |
| **[Digital Realty](https://www.digitalrealty.com/services/interconnection/service-exchange/)** | IX Reach, Megaport PacketFabric |
| **[EdgeConnex](https://www.edgeconnex.com/services/edge-data-centers-proximity-matters/)** | Megaport, PacketFabric |
| **[Flexential](https://www.flexential.com/connectivity/cloud-connect-microsoft-azure-expressroute)** | IX Reach, Megaport, PacketFabric |
| **[QTS Data Centers](https://www.qtsdatacenters.com/hybrid-solutions/connectivity/azure-cloud )** | Megaport, PacketFabric |
| **[Stream Data Centers]( https://www.streamdatacenters.com/products-services/network-cloud/ )** | Megaport |
| **[RagingWire Data Centers](https://www.ragingwire.com/wholesale/wholesale-data-centers-worldwide-nexcenters)** | IX Reach, Megaport, PacketFabric |
| **[vXchnge](https://www.vxchnge.com/colocation-services/interconnection)** | IX Reach, Megaport |
| **[T5 Datacenters](https://t5datacenters.com/)** | IX Reach |

## <a name="connectivity-through-national-research-and-education-networks-nren"></a>Подключение через национальные исследовательские и образовательные сети (NREN)

| **Поставщик**|
| --- |
| **AARNET**| 
| **DeIC, через GÉANT**|
| **GARR, через GÉANT**|
| **GÉANT**|
| **HEAnet, через GÉANT**|
| **Internet2**|
| **JISC**|
| **RedIRIS, через GÉANT**|
| **SINET**|
| **Surfnet, через GÉANT**|

* Если вашего поставщика услуг подключения нет в списке, проверьте, подключены ли они к любому из перечисленных выше партнеров ExpressRoute Exchange.

## <a name="expressroute-system-integrators"></a>Системные интеграторы ExpressRoute
Возможность частного подключения, соответствующего вашим потребностям, будет зависеть от масштаба сети. Чтобы упростить переход на ExpressRoute, вы можете обратиться к одному из системных интеграторов, указанных в таблице ниже.

| **Системный интегратор** | **Континент** |
| --- | --- |
| **[Altogee](https://altogee.be/diensten/express-route/)** | Европа |
| **[Avanade Inc.](https://www.avanade.com/)** | Азия, Европа, Северная Америка, Южная Америка |
| **[Bright Skies GmbH](https://bskies.io/expressroute)** | Европа
| **[Ensyst](https://www.ensyst.com.au)** | Азия
| **[Equinix Professional Services](https://www.equinix.com/services/consulting/)** | Северная Америка |
| **[FlexManage](https://www.flexmanage.com/cloud)** | Северная Америка |
| **[Lightstream](https://www.lightstream.tech/partners/microsoft-azure/)** | Северная Америка |
| **[The IT Consultancy Group](https://itconsult.com.au/)** | Австралия |
| **[MOQdigital](https://www.moqdigital.com/insights)** | Австралия |
| **[MSG Services](https://www.msg-services.de/it-services/managed-services/cloud-outsourcing/)** | Европа (Германия) |
| **[Nelite](https://www.exakis-nelite.com/offres/)** | Европа |
| **[New Signature](https://newsignature.com/technologies/express-route/)** | Европа |
| **[OneAs1a](https://www.oneas1a.com/connectivity.html)** | Азия |
| **[Orange Networks](https://orange-networks.com/blog/88-azureexpressroute)** | Европа |
| **[Perficient](https://www.perficient.com/Partners/Microsoft/Cloud/Azure-ExpressRoute)** | Северная Америка |
| **[Presidio](https://www.presidio.com/subpage/1107/microsoft-azure)** | Северная Америка |
| **[sol-tec](https://www.sol-tec.com/what-we-do/)** | Европа |
| **[Venha Pra Nuvem](https://venhapranuvem.com.br/)** | Южная Америка |
| **[Vigilant.IT](https://vigilant.it/expressroute)** | Австралия |

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения об ExpressRoute см. в статье [Вопросы и ответы по ExpressRoute](expressroute-faqs.md).
* Убедитесь, что выполнены все необходимые условия. Ознакомьтесь с разделом [Предварительные требования и контрольный список для ExpressRoute](expressroute-prerequisites.md).

<!--Image References-->
[0]: ./media/expressroute-locations/expressroute-locations-map.png "Карта расположения"
