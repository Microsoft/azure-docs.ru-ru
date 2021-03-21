---
title: Механизм интеграции Cloud Foundry с Azure | Документация Майкрософт
description: В этой статье описывается, как Cloud Foundry может использовать службы Azure для повышения эффективности работы предприятия.
services: virtual-machines-linux
documentationcenter: ''
author: ningk
tags: Cloud-Foundry
ms.assetid: 00c76c49-3738-494b-b70d-344d8efc0853
ms.service: virtual-machines
ms.topic: conceptual
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/11/2018
ms.author: ningk
ms.openlocfilehash: ff2a6618b60ff2cfa5faa74c905e140466a14359
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102563325"
---
# <a name="integrate-cloud-foundry-with-azure"></a>Интеграция Cloud Foundry и Azure

[Cloud Foundry](https://docs.cloudfoundry.org/) — это платформа PaaS, работающая поверх платформы IaaS поставщиков облачных служб. Она обеспечивает согласованный процесс развертывания приложений между поставщиками облачных услуг. Она также может интегрироваться с различными службами Azure с высоким уровнем доступности, масштабируемостью и экономией корпоративного класса.
Существует [6 подсистем Cloud Foundry](https://docs.cloudfoundry.org/concepts/architecture/), которые можно гибко масштабировать в сети, включая маршрутизацию, проверку подлинности, управление жизненным циклом приложений, Управление службами, Обмен сообщениями и мониторинг. Для каждой из подсистем можно настроить Cloud Foundry для использования корреспондентской службы Azure. 

![Архитектура интеграции Cloud Foundry и Azure](media/CFOnAzureEcosystem-colored.png)

## <a name="1-high-availability-and-scalability"></a>1. высокий уровень доступности и масштабируемость
### <a name="managed-disk"></a>Управляемые диски
Bosh использует Azure CPI (интерфейс облачного поставщика) для создания и удаления подподпрограмм диска. По умолчанию используются неуправляемые диски. Клиенту при этом нужно вручную создавать учетные записи хранения, а затем настраивать их в файлах манифестов CF. Это связано с ограничением на количество дисков на одну учетную запись хранения.
Теперь можно использовать [управляемые диски](https://azure.microsoft.com/services/managed-disks/), которые обеспечивают управляемое защищенное и надежное дисковое хранилище для виртуальных машин. Клиенту больше не нужно работать с учетной записью хранения для обеспечения масштабирования и высокого уровня доступности. Azure упорядочивает диски автоматически. Будь то новое или существующее развертывание, в Azure Иос будет осуществляться создание или миграция управляемого диска во время развертывания CF. Он поддерживается в PCF 1,11. Дополнительные сведения см. в [руководстве по управляемым дискам](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/managed-disks) Cloud Foundry с открытым исходным кодом. 
### <a name="availability-zone-"></a>Зона доступности *
Cloud Foundry — это платформа приложений, созданных для облака. Для нее предусмотрено [четыре уровня обеспечения высокой доступности](https://docs.pivotal.io/pivotalcf/2-1/concepts/high-availability.html). Первые три уровня сбоев программного обеспечения могут обрабатываться самой системой CF, а отказоустойчивость платформы обеспечивается поставщиками облачных служб. Ключевые компоненты CF должны быть защищены решением для обеспечения высокого уровня доступности платформы поставщика облачных служб. К таким решениям относятся GoRouters, Diego Brains, плитки баз данных и служб CF. По умолчанию для обеспечения отказоустойчивости между кластерами в центре обработки данных используются [группы доступности Azure](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/deploy-cloudfoundry-with-availability-sets).
Ситуацию улучшил выпуск [зон доступности Azure](../availability-zones/az-overview.md) с оптимизацией отказоустойчивости, реализованной как избыточность с низкой задержкой между центрами обработки данных.
Зоны доступности Azure позволяет достичь высокого уровня доступности благодаря включению набора виртуальных машин в два или большее количество центров обработки данных. Каждый набор виртуальных машин избыточен по отношению к другим наборам. Если одна зона выйдет из строя, остальные наборы останутся доступными и изолированными от сбоя.
> [!NOTE] 
> Зона доступности Azure пока предлагается не для всех регионов — см. последнее [объявление о списке поддерживаемых регионов](../availability-zones/az-overview.md). Также см. [руководство по зонам доступности Azure для Cloud Foundry с открытым исходным кодом](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/availability-zone).

## <a name="2-network-routing"></a>2. сетевая маршрутизация
По умолчанию базовая подсистема балансировки нагрузки Azure используется для входящих запросов к API и приложениям CF, перенаправляя их в Gorouters. Компоненты CF, например Diego Brain, MySQL, ERT, могут также с помощью подсистемы балансировки нагрузки балансировать трафик для обеспечения высокой доступности. Azure также предоставляет набор полностью управляемых решений балансировки нагрузки. Если вы ищете завершение TLS/SSL ("разгрузка SSL") или обработку уровня приложения для запросов HTTP/HTTPS, рассмотрите возможность использования шлюза приложений. Для балансировки нагрузки с обеспечением высокой доступности и масштабируемости на четвертом уровне рекомендуется использовать стандартную подсистему балансировки нагрузки.
### <a name="azure-application-gateway-"></a>Шлюз приложений Azure *
[Шлюз приложений Azure](../application-gateway/overview.md) предлагает различные возможности балансировки нагрузки уровня 7, включая разгрузку SSL, СКВОЗНОЙ протокол TLS, брандмауэр веб-приложения, сходство сеансов на основе файлов cookie и многое другое. Вы можете [настроить шлюз приложений в Cloud Foundry с открытым исходным кодом](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/application-gateway). Дополнительные сведения см. в [примечаниях к выпуску PCF 2.1](https://docs.pivotal.io/pivotalcf/2-1/pcf-release-notes/opsmanager-rn.html#azure-application-gateway) для теста POC.

### <a name="azure-standard-load-balancer-"></a>Azure Load Balancer уровня "Стандартный" *
Azure Load Balancer — это подсистема балансировки нагрузки четвертого уровня. Он используется для распределения трафика между экземплярами служб в наборе с балансировкой нагрузки. Стандартная версия включает [дополнительные возможности](../load-balancer/load-balancer-overview.md) наряду с возможностями базовой версии, например: 1. Максимальный размер внутреннего пула повышен с 100 до 1000 виртуальных машин.  2. Конечные точки теперь поддерживают несколько групп доступности вместо одной.  3. Дополнительные функции, такие как порты HA, Расширенные данные мониторинга и т. д. Если вы переходите в зону доступности Azure, требуется стандартная подсистема балансировки нагрузки. При работе с новым развертыванием рекомендуется начать с Azure Load Balancer уровня "Стандартный". 

## <a name="3-authentication"></a>3. Проверка подлинности 
[Cloud Foundry User Account and Authentication](https://docs.cloudfoundry.org/concepts/architecture/uaa.html) (UAA) — это служба централизованного управления удостоверениями для платформы CF и ее компонентов. [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) — это облачный каталог и служба управления удостоверениями корпорации Майкрософт. По умолчанию для проверки подлинности в Cloud Foundry используется служба UAA. UAA также позволяет использовать Azure AD в качестве внешнего пользовательского хранилища. Пользователи Azure AD могут получить доступ к Cloud Foundry с помощью удостоверения LDAP без учетной записи Cloud Foundry. См. дополнительные сведения о [настройке Azure AD для UAA в PCF](https://docs.pivotal.io/p-identity/1-6/azure/index.html).

## <a name="4-data-storage-for-cloud-foundry-runtime-system"></a>4. хранилище данных для системы среды выполнения Cloud Foundry
Благодаря своей расширяемости, Cloud Foundry позволяет использовать хранилища BLOB-объектов Azure или службы Azure MySQL или PostgreSQL в качестве хранилищ системы среды выполнения приложений.
### <a name="azure-blobstore-for-cloud-foundry-cloud-controller-blobstore"></a>Использование хранилища BLOB-объектов Azure в качестве хранилища BLOB-объектов Cloud Foundry Cloud Controller
Хранилище BLOB-объектов Cloud Controller — это критически важное хранилище данных для пакетов сборок, дроплетов, пакетов и пулов ресурсов. По умолчанию для хранилища BLOB-объектов Cloud Controller используется сервер NFS. Чтобы избежать реализации единой точки отказа, используйте хранилище BLOB-объектов Azure в качестве внешнего хранилища. Дополнительные сведения см. в [документации по Cloud Foundry](https://docs.cloudfoundry.org/deploying/common/cc-blobstore-config.html) и [описании параметров в Pivotal Cloud Foundry](https://docs.pivotal.io/pivotalcf/2-0/customizing/azure.html).

### <a name="mysqlpostgresql-as-cloud-foundry-elastic-run-time-database-"></a>Использование MySQL или PostgreSQL в качестве базы данных Cloud Foundry Elastic Runtime *
Для CF Elastic Runtime требуются две основные системные базы данных:
#### <a name="ccdb"></a>CCDB 
База данных Cloud Controller.  Cloud Controller предоставляет конечные точки REST API клиентам для доступа к системе. В базе CCDB хранятся таблицы организаций, пространств, служб, ролей пользователей и других компонентов Cloud Controller.
#### <a name="uaadb"></a>UAADB 
База данных службы User Account and Authentication. В ней хранятся данные, связанные с аутентификацией пользователей, например зашифрованные имена пользователей и пароли.

По умолчанию можно использовать локальную системную базу данных (MySQL). Для обеспечения высокой доступности и масштабирования используйте преимущества управляемых Azure служб MySQL или PostgreSQL. См. дополнительные сведения о [включении Azure MySQL или PostgreSQL для CCDB, UAADB и других системных баз данных в Cloud Foundry с открытым исходным кодом](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/configure-cf-external-databases-using-azure-mysql-postgres-service).

## <a name="5-open-service-broker"></a>5. Откройте Service Broker
Azure Service Broker обеспечивает унифицированный интерфейс для управления доступом приложений к службам Azure. Новый [проект Open Service Broker для Azure](https://github.com/Azure/open-service-broker-azure) предоставляет единый и простой способ доставки служб приложениям через Cloud Foundry, OpenShift и Kubernetes. Дополнительные сведения о развертывании в PCF см. на [плитке Azure Open Service Broker для PCF](https://pivotal.io/platform/services-marketplace/data-management/microsoft-azure).

## <a name="6-metrics-and-logging"></a>6. метрики и ведение журнала
Log Analytics сопла Azure — это Cloud Foundry компонент, который перенаправляет метрики из [Cloud Foundry loggregator firehose](https://docs.cloudfoundry.org/loggregator/architecture.html) в [журналы Azure Monitor](https://azure.microsoft.com/services/log-analytics/). С помощью Nozzle можно собирать, просматривать и анализировать метрики производительности и работоспособности системы CF в нескольких развертываниях.
Щелкните [здесь](./cloudfoundry-oms-nozzle.md) , чтобы узнать, как развернуть сопло log Analytics Azure в среде с открытым исходным кодом и в сводном Cloud Foundry, а затем получить доступ к данным из консоли Azure Monitor журналы. 
> [!NOTE]
> В PCF 2,0 метрики работоспособности BOSH для виртуальных машин пересылаются в Loggregator firehose по умолчанию и интегрируются в консоль Azure Monitor журналов.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="7-cost-saving"></a>7. Сохранение затрат
### <a name="cost-saving-for-devtest-environments"></a>Сокращение затрат на среды разработки и тестирования
#### <a name="b-series-"></a>Серия B: *
Раньше для использования в производственной среде Pivotal Cloud Foundry часто рекомендовались виртуальные машины серии F и D. Сегодня новая [серия B](https://azure.microsoft.com/blog/introducing-b-series-our-new-burstable-vm-size/) с накапливаемыми ресурсами предоставляет новые возможности. Виртуальные машины серии B идеально подходят для рабочих нагрузок, которым не требуется непрерывная полная производительность ЦП, например веб-серверов, небольших баз данных и сред разработки и тестирования. Обычно для этих рабочих нагрузок требуется накапливаемая производительность. Ее стоимость составляет 0,012 долл. США в час для B1 по сравнению с 0,05 долл. США в час для F1. См. дополнительные сведения о [размерах виртуальных машин](../virtual-machines/sizes-general.md) и [ценах на них](https://azure.microsoft.com/pricing/details/virtual-machines/linux/). 
#### <a name="managed-standard-disk"></a>Управляемый диск уровня "Стандартный" 
Раньше для обеспечения надежной производительности в рабочей среде рекомендовались диски класса "Премиум".  Благодаря использованию [управляемых дисков](https://azure.microsoft.com/services/managed-disks/) стандартное хранилище обеспечивает аналогичную надежность с разными уровнями производительности. Для рабочих нагрузок без учета производительности, таких как разработка и тестирование или Некритическая среда, управляемые диски Standard предлагают альтернативный вариант с более низкими затратами.  
### <a name="cost-saving-in-general"></a>Сокращение общих затрат 
#### <a name="significant-vm-cost-saving-with-azure-reservations"></a>Значительное сокращение затрат на виртуальные машины за счет резервирований Azure: 
Сейчас счета за использование всех виртуальных машин CF начисляются по требованию, хотя среды обычно работают в непрерывном режиме. Теперь вы можете зарезервировать емкость виртуальных машин на один или три года и получить скидку в размере 45–65 %. Скидки применяются в системе выставления счетов без внесения изменений в вашу среду. Дополнительные сведения см. на странице [Зарезервированные экземпляры виртуальных машин Azure](https://azure.microsoft.com/pricing/reserved-vm-instances/). 
#### <a name="managed-premium-disk-with-smaller-sizes"></a>Управляемые диски уровня "Премиум" меньших размеров 
Управляемые диски поддерживают и меньшие размеры, например P4 (32 Гбайт) и P6 (64 Гбайт) для уровней "Премиум" и "Стандартный". При небольших рабочих нагрузках можно сократить расходы, выполнив перенос со стандартных дисков уровня "Премиум" на управляемые диски уровня "Премиум".
#### <a name="use-azure-first-party-services"></a>Использование служб первого производителя Azure: 
Преимущество собственных служб Azure заключается в снижении долгосрочных затрат на администрирование в дополнение к высокому уровню доступности и надежности, о котором говорилось выше. 

Компанией Pivotal была запущена [среда Small Footprint ERT](https://docs.pivotal.io/pivotalcf/2-0/customizing/small-footprint.html) для клиентов PCF. Компоненты совместно располагаются всего в четырех виртуальных машинах, на которых выполняются до 2500 экземпляров приложений. Пробная версия сейчас доступна в [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry).

## <a name="next-steps"></a>Дальнейшие действия
Функции интеграции Azure в первую очередь доступны в [Cloud Foundry с открытым исходным кодом](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/), прежде чем они будут доступны в сводных Cloud Foundry. Компоненты, отмеченные звездочкой *, все еще недоступны в PCF. Интеграция Cloud Foundry с Azure Stack не рассматривается в этом документе.
Для получения последних сведений о состоянии и помощи по PCF в компонентах, отмеченных звездочкой *, или по интеграции Cloud Foundry со стеком Azure Stack обратитесь к менеджеру по работе с клиентами Pivotal и Майкрософт.