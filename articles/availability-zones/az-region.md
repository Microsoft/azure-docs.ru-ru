---
title: Службы Azure с поддержкой Зон доступности
description: Для создания высокодоступного и отказоустойчивого приложения в Azure Зоны доступности предоставляет физически разделенные зоны, которые можно использовать для выполнения ресурсов.
author: prsandhu
ms.service: azure
ms.topic: conceptual
ms.date: 03/16/2021
ms.author: prsandhu
ms.reviewer: cynthn
ms.custom: fasttrack-edit, mvc, references_regions
ms.openlocfilehash: 7aff8320186ada53772fc4ff2232e8cf55585a77
ms.sourcegitcommit: 87a6587e1a0e242c2cfbbc51103e19ec47b49910
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103573036"
---
# <a name="azure-services-that-support-availability-zones"></a>Службы Azure с поддержкой Зон доступности

Microsoft Azure Глобальная инфраструктура разрабатывается и создается на каждом уровне, чтобы обеспечить высочайший уровень избыточности и устойчивость к своим клиентам. Инфраструктура Azure состоит из географических областей, регионов и Зоны доступности, которые ограничивают радиус сбоя и, таким образом, ограничивают потенциальное воздействие на клиентские приложения и данные. Конструкция Зоны доступности Azure была разработана, чтобы обеспечить программное и сетевое решение для защиты от сбоев центра обработки данных и повышения высокого уровня доступности для наших клиентов.

Зоны доступности — уникальные физические расположения в пределах одного региона Azure. Каждая зона состоит из одного или нескольких центров обработки данных с независимым питанием, охлаждением и сетью. Физическое разделение Зоны доступности в пределах региона ограничивает влияние на приложения и данные из сбоев зоны, таких как крупномасштабное переполнение, основные степени и подмножества, а также другие события, которые могут нарушить доступ к сайту, безопасность, расширенные служебные программы и доступность ресурсов. Зоны доступности и связанные с ними центры обработки данных спроектированы таким образом, что если одна зона скомпрометирована, службы, емкость и доступность поддерживаются другими Зоны доступности в регионе.

Все службы управления Azure спроектированы как устойчивые к сбоям на уровне региона. В спектре сбоев один или несколько сбоев зоны доступности в регионе имеют меньший радиус сбоя по сравнению со всем сбоем региона. Azure может выполнить восстановление после сбоя на уровне зоны для служб управления в пределах региона. Azure выполняет критическое обслуживание по одной зоне за раз в пределах региона, чтобы предотвратить сбои, влияющие на ресурсы клиента, развернутые в Зоны доступности в пределах региона.


![концептуальное представление региона Azure с тремя зонами](./media/az-region/azure-region-availability-zones.png)


Службы Azure, поддерживающие Зоны доступности, делятся на три категории: **зональные**, **избыточные в зонах** и **не региональные** службы. Рабочие нагрузки клиентов можно классифицировать для использования любого из этих сценариев архитектуры в соответствии с производительностью и устойчивостью приложений.

- **Зональные Services** — ресурс можно развернуть в определенной, самостоятельно выбираемой зоне доступности для достижения более строгой задержки или требований к производительности.  Отказоустойчивость — это самостоятельная репликация приложений и данных в одну или несколько зон в пределах региона.  Ресурсы можно закреплять в определенной зоне. Например, виртуальные машины, управляемые диски или стандартные IP-адреса могут быть закреплены в определенной зоне, что обеспечивает повышенную устойчивость путем использования одного или нескольких экземпляров ресурсов, распределенных между зонами.

- **Службы, избыточные** в пределах зоны — Платформа Azure реплицирует ресурсы и данные между зонами.  Корпорация Майкрософт управляет доставкой высокого уровня доступности, так как Azure автоматически реплицирует и распространяет экземпляры в пределах региона.  ZRS, например, реплицирует данные по трем зонам, чтобы сбой зоны не влиял на высокую доступность данных. 

- Службы, **не предназначенные для регионального региона** — службы всегда доступны в географических регионах Azure и устойчивы к простоям в пределах зоны, а также по сбоям в рамках всего региона. 


Чтобы обеспечить всестороннюю непрерывность бизнес-процессов в Azure, создайте архитектуру приложений, используя сочетание зон доступности с парами регионов Azure. Можно синхронно реплицировать приложения и данные с помощью зон доступности в регионе Azure, чтобы обеспечить высокий уровень доступности, и асинхронно реплицировать их между регионами Azure, чтобы обеспечить защиту с помощью аварийного восстановления. Дополнительные сведения см. в статье [Создание решений для обеспечения высокой доступности с помощью зоны доступности](/azure/architecture/high-availability/building-solutions-for-high-availability). 

## <a name="azure-services-supporting-availability-zones"></a>Службы Azure, поддерживающие Зоны доступности

 - Виртуальные машины старого поколения не перечислены. Дополнительные сведения см. в статье [Размеры виртуальных машин предыдущих поколений](../virtual-machines/sizes-previous-gen.md).
 - Как упоминалось в [регионах и зоны доступности в Azure](az-overview.md), некоторые службы не являются региональными. Эти службы не имеют зависимости от конкретного региона Azure, так как они устойчивы к простоям на уровне зоны, а также к простоям в рамках всего региона.  Список нерегиональный службы можно найти в списке [продуктов, доступных по регионам](https://azure.microsoft.com/global-infrastructure/services/).


## <a name="azure-regions-with-availability-zones"></a>Регионы Azure с Зоны доступности


| Северная и Южная Америка           | Европа               | Африка              | Азиатско-Тихоокеанский регион   |
|--------------------|----------------------|---------------------|----------------|
|                    |                      |                     |                |
| Brazil South       | Центральная Франция       | Южная Африка, Северный * | Japan East     |
| Центральная Канада     | Центрально-Западная Германия |                     | Southeast Asia |
| Центральная часть США         | Северная Европа         |                     | Восточная Австралия |
| Восточная часть США            | южная часть Соединенного Королевства             |                     |                |
| восточная часть США 2          | Западная Европа          |                     |                |
| Юго-Центральный регион США |                      |                     |                |
| US Gov (Вирджиния)     |                      |                     |                |
| Западная часть США 2        |                      |                     |                |


\* Чтобы узнать больше о Зоны доступности и доступных службах в этих регионах, обратитесь к специалистам Майкрософт по продажам или клиентам. Сведения о следующих регионах, которые будут поддерживать Зоны доступности, см. в разделе [географические диаграммы Azure](https://azure.microsoft.com/en-us/global-infrastructure/geographies/).


## <a name="azure-services-supporting-availability-zones"></a>Службы Azure, поддерживающие Зоны доступности

- Виртуальные машины старого поколения не перечислены ниже. Дополнительные сведения см. в статье [предыдущие поколения размеров виртуальных машин](../virtual-machines/sizes-previous-gen.md).

- Некоторые службы не являются региональными. Дополнительные сведения см. [в разделе регионы и зоны доступности в Azure](az-overview.md) . Эти службы не имеют зависимости от конкретного региона Azure, что делает их устойчивыми к простоям на уровне зоны и по всему региону.  Список нерегиональный службы можно найти в списке [продуктов, доступных по регионам](https://azure.microsoft.com/global-infrastructure/services/).


### <a name="zone-resilient-services"></a>Отказоустойчивые службы зоны 

: globe_with_meridians: службы, не являющиеся региональными, — службы всегда доступны в географических регионах Azure и устойчивы к простоям на уровне зоны, а также к сбоям в рамках всего региона.

: large_blue_diamond: устойчивость к простоям в пределах зоны 

**Базовые службы**

|     Продукты                                                    | Устойчивость             |
|-----------------------------------------------------------------|:----------------------------:|
|     Учетная запись хранения                                           | : large_blue_diamond:  |
|     Шлюз приложений (v2)                                  | : large_blue_diamond:  |
|     Azure Backup                                                | : large_blue_diamond:  |
|     Azure Cosmos DB                                           | : large_blue_diamond:  |
|     Azure Data Lake Storage Gen 2                             | : large_blue_diamond:  |
|     Azure Express Route                                       | : large_blue_diamond:  |
|     Общедоступный IP-адрес Azure                                           | : large_blue_diamond:  |
|     База данных SQL Azure (уровень общего назначения)                 | : large_blue_diamond:  |
|     База данных SQL Azure (уровень критически важный для бизнеса Premium &)     | : large_blue_diamond:  |
|     Хранилище дисков                                                | : large_blue_diamond:  |
|     Центры событий                                                  | : large_blue_diamond:  |
|     Key Vault                                                   | : large_blue_diamond:  |
|     Load Balancer                                               | : large_blue_diamond:  |
|     Cлужебная шина                                                 | : large_blue_diamond:  |
|     Service Fabric                                            | : large_blue_diamond:  |
|     Хранилище: уровни "горячий" и "холодного" хранилища BLOB-объектов                      | : large_blue_diamond:  |
|     Хранилище: управляемые диски                                    | : large_blue_diamond:  |
|     Масштабируемые наборы виртуальных машин                               | : large_blue_diamond:  |
|     Виртуальные машины                                          | : large_blue_diamond:  |
|     Виртуальные машины: Av2-Series                              | : large_blue_diamond:  |
|     Виртуальные машины: Bs-Series                               | : large_blue_diamond:  |
|     Виртуальные машины: DSv2-Series                             | : large_blue_diamond:  |
|     Виртуальные машины: DSv3-Series                             | : large_blue_diamond:  |
|     Виртуальные машины: Dv2-Series                              | : large_blue_diamond:  |
|     Виртуальные машины: Dv3-Series                              | : large_blue_diamond:  |
|     Виртуальные машины: ESv3-Series                             | : large_blue_diamond:  |
|     Виртуальные машины: Ev3-Series                              | : large_blue_diamond:  |
|     Виртуальные машины: серии F                                | : large_blue_diamond:  |
|     Виртуальные машины: FS-Series                               | : large_blue_diamond:  |
|     Виртуальная сеть                                           | : large_blue_diamond:  |
|     VPN-шлюз                                                 | : large_blue_diamond:  |


**Самые распространенные службы**

| Продукты                                        | Устойчивость |
|-------------------------------------------------|:------------:|
| Среда службы приложений                        |      : large_blue_diamond:  |
| Доменные службы Azure Active Directory          |      : large_blue_diamond:  |
| Бастион Azure                                   |      : large_blue_diamond:  |
| Кэш Redis для Azure                           |      : large_blue_diamond:  |
| Azure Cognitive Services: Анализ текста        |      : large_blue_diamond:  |
| Azure Data Explorer                             |      : large_blue_diamond:  |
| База данных Azure для MySQL — гибкий сервер      |      : large_blue_diamond:  |
| База данных Azure для PostgreSQL — гибкий сервер |      : large_blue_diamond:  |
| Защита от атак DDoS Azure                           |      : large_blue_diamond:  |
| Шифрование дисков Azure                           |      : large_blue_diamond:  |
| Брандмауэр Azure                                  |      : large_blue_diamond:  |
| Диспетчер брандмауэра Azure                          |      : large_blue_diamond:  |
| служба Azure Kubernetes (AKS);                  |      : large_blue_diamond:  |
| Приватный канал Azure                              |      : large_blue_diamond:  |
| Azure Red Hat OpenShift                         |      : large_blue_diamond:  |
| Azure Site Recovery                             |      : large_blue_diamond:  |
| Azure SQL: виртуальная машина                      |      : large_blue_diamond:  |
| Поиск Azure                                    |      : large_blue_diamond:  |
| Брандмауэр веб-приложения Azure                  |      : large_blue_diamond:  |
| Cognitive Services: Анализ текста              |      : large_blue_diamond:  |
| Реестр контейнеров                              |      : large_blue_diamond:  |
| Сетка событий                                      |      : large_blue_diamond:  |
| Наблюдатель за сетями                                 |      : large_blue_diamond:  |
| Наблюдатель за сетями: Аналитика трафика              |      : large_blue_diamond:  |
| Power BI Embedded                               |      : large_blue_diamond:  |
| Хранилище BLOB-объектов класса Premium                            |      : large_blue_diamond:  |
| Хранилище: файлы Azure Premium                    |      : large_blue_diamond:  |
| Виртуальные машины: выделенный узел Azure          |      : large_blue_diamond:  |
| Виртуальные машины: Ddsv4-Series                  |      : large_blue_diamond:  |
| Виртуальные машины: Ddv4-Series                   |      : large_blue_diamond:  |
| Виртуальные машины: Dsv4-Series                   |      : large_blue_diamond:  |
| Виртуальные машины: Dv4-Series                    |      : large_blue_diamond:  |
| Виртуальные машины: Edsv4-Series                  |      : large_blue_diamond:  |
| Виртуальные машины: Edv4-Series                   |      : large_blue_diamond:  |
| Виртуальные машины: Esv4-Series                   |      : large_blue_diamond:  |
| Виртуальные машины: Ev4-Series                    |      : large_blue_diamond:  |
| Виртуальные машины: Fsv2-Series                   |      : large_blue_diamond:  |
| Виртуальные машины: серия M                      |      : large_blue_diamond:  |
| Виртуальная глобальная сеть                                     |      : large_blue_diamond:  |
| Виртуальная глобальная сеть: ExpressRoute                       |      : large_blue_diamond:  |
| Виртуальная глобальная сеть: VPN-шлюз типа "точка — сеть"          |      : large_blue_diamond:  |
| Виртуальная глобальная сеть: VPN-шлюз типа "сеть — сеть"           |      : large_blue_diamond:  |


**Не региональный**

|     Продукты                                  |     Устойчивость    |
|-----------------------------------------------|:-------------------:|
|     Azure DNS                                 |     : globe_with_meridians:             |
|     Azure Active Directory                  |     : globe_with_meridians:             |
|     Расширенная защита от угроз Azure          |     : globe_with_meridians:             |
|     Помощник по Azure                             |     : globe_with_meridians:             |
|     Azure Blueprints                          |     : globe_with_meridians:             |
|     Служба Azure Bot                        |     : globe_with_meridians:             |
|     Azure Front Door                          |     : globe_with_meridians:             |
|     Защитник Azure для IoT                  |     : globe_with_meridians:             |
|     Azure Front Door                           |     : globe_with_meridians:             |
|     Azure Information Protection            |     : globe_with_meridians:             |
|     Azure Лигхсаусе                        |     : globe_with_meridians:             |
|     Управляемые приложения Azure              |     : globe_with_meridians:             |
|     Azure Maps                                |     : globe_with_meridians:             |
|     Политика Azure                              |     : globe_with_meridians:             |
|     Граф ресурсов Azure                    |     : globe_with_meridians:             |
|     Azure Sentinel                            |     : globe_with_meridians:             |
|     Azure Stack                               |     : globe_with_meridians:             |
|     Azure Stackная граница                        |     : globe_with_meridians:             |
|     Cloud Shell                               |     : globe_with_meridians:             |
|     Сеть доставки содержимого (CDN)                  |     : globe_with_meridians:             |
|     Управление затратами                           |     : globe_with_meridians:             |
|     защищенное хранилище для Microsoft Azure    |     : globe_with_meridians:             |
|     Intune                                    |     : globe_with_meridians:             |
|     Служба пиринга Microsoft Azure         |     : globe_with_meridians:             |
|     портал Microsoft Azure                  |     : globe_with_meridians:             |
|     Microsoft Cloud App Security              |     : globe_with_meridians:             |
|     Microsoft Graph                           |     : globe_with_meridians:             |
|     Центр безопасности                         |     : globe_with_meridians:             |
|     Диспетчер трафика                         |     : globe_with_meridians:             |


## <a name="pricing-for-vms-in-availability-zones"></a>Цены на виртуальные машины в Зоны доступности

За виртуальные машины, развернутые в зоне доступности, не взимается дополнительная плата. Дополнительные сведения см. на [странице с ценами на пропускную способность](https://azure.microsoft.com/pricing/details/bandwidth/).


## <a name="get-started-with-availability-zones"></a>Приступая к работе с зонами доступности

- [Создание виртуальной машины](../virtual-machines/windows/create-portal-availability-zone.md)
- [Добавление управляемого диска с помощью PowerShell](../virtual-machines/windows/attach-disk-ps.md#add-an-empty-data-disk-to-a-virtual-machine)
- [Создание избыточного между зонами масштабируемого набора виртуальных машин](../virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones.md)
- [Распределение нагрузки виртуальных машин по зонам с помощью службы Load Balancer уровня "Стандартный" с избыточным между зонами внешним интерфейсом](../load-balancer/quickstart-load-balancer-standard-public-cli.md?tabs=option-1-create-load-balancer-standard).
- [Распределение нагрузки виртуальных машин в пределах зоны с помощью Load Balancer уровня "Стандартный" с зональным внешним интерфейсом](../load-balancer/quickstart-load-balancer-standard-public-cli.md?tabs=option-1-create-load-balancer-standard).
- [Хранилище, избыточное между зонами](../storage/common/storage-redundancy.md)
- [Уровень общего назначения базы данных SQL](../azure-sql/database/high-availability-sla.md#general-purpose-service-tier-zone-redundant-availability-preview)
- [Географическое аварийное восстановление в Центрах событий](../event-hubs/event-hubs-geo-dr.md#availability-zones)
- [Географическое аварийное восстановление в служебной шине](../service-bus-messaging/service-bus-geo-dr.md#availability-zones)
- [Создание шлюза виртуальной сети, избыточного между зонами](../vpn-gateway/create-zone-redundant-vnet-gateway.md)
- [Добавление области, избыточной в пределах зоны, для Azure Cosmos DB](../cosmos-db/high-availability.md#availability-zone-support)
- [начало работы кэша Azure для Redis Зоны доступности](https://gist.github.com/JonCole/92c669ea482bbb7996f6428fb6c3eb97#file-redisazgettingstarted-md)
- [Создание экземпляра доменных служб Azure Active Directory](../active-directory-domain-services/tutorial-create-instance.md)
- [Создание кластера Azure Kubernetes Service (AKS), использующего Зоны доступности](../aks/availability-zones.md)


## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Регионы и зоны доступности в Azure](az-overview.md)
