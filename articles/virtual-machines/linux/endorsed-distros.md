---
title: Дистрибутивы Linux, рекомендованные для использования в Azure
description: Узнайте о рекомендованных дистрибутивах Linux в Azure, включая рекомендации по Ubuntu, CentOS, Oracle и SUSE.
services: virtual-machines
documentationcenter: ''
author: danielsollondon
manager: gwallace
tags: azure-service-management,azure-resource-manager
ms.assetid: 2777a526-c260-4cb9-a31a-bdfe1a55fffc
ms.service: virtual-machines
ms.collection: linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: conceptual
ms.date: 01/03/2021
ms.author: guybo
ms.openlocfilehash: 091a06fd608763ac7265670733890e93bf71dde1
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102547413"
---
# <a name="endorsed-linux-distributions-on-azure"></a>Рекомендуемые дистрибутивы Linux в Azure

Партнеры предоставляют образы Linux в Azure Marketplace. Корпорация Майкрософт работает с различными сообществом Linux, чтобы добавить еще больше вариантов в список рекомендуемых дистрибутивов. Для дистрибутивов, недоступных в Marketplace, вы всегда можете использовать собственную Linux, следуя указаниям в руководстве по [созданию и передаче виртуального жесткого диска, содержащего операционную систему Linux](./create-upload-generic.md).

## <a name="supported-distributions-and-versions"></a>Поддерживаемые дистрибутивы и версии

В следующей таблице перечислены дистрибутивы и версии Linux, поддерживаемые в Azure. Дополнительные сведения см. [в разделе Поддержка образов Linux в Microsoft Azure](https://support.microsoft.com/help/2941892/support-for-linux-and-open-source-technology-in-azure).

Драйверы Linux Integration Services (LIS) для Hyper-V и Azure представляют из себя модули ядра, которые Майкрософт встраивает непосредственно в основное ядро Linux. Некоторые драйверы LIS встроены в ядро дистрибутива по умолчанию. Более старые дистрибутивы на основе Red Hat Enterprise (RHEL) и CentOS можно отдельно скачать на странице [служб интеграции Linux (LIS) 4.2 для Hyper-V и Azure](https://www.microsoft.com/download/details.aspx?id=55106). Дополнительные сведения см. в статье [требования к ядру Linux](create-upload-generic.md#linux-kernel-requirements).

Агент Linux для Azure уже предварительно установлен в образах Azure Marketplace и обычно доступен в репозитории пакетов дистрибутива. Исходный код можно найти на сайте [GitHub](https://github.com/azure/walinuxagent).

| Distribution | Версия | драйверы, | Агент |
| --- | --- | --- | --- |
| CentOS по мошенническому звуковому программному обеспечению |CentOS 6. x, 7. x, 8. x |CentOS 6,3: [скачивание LIS](https://www.microsoft.com/download/details.aspx?id=55106)<p>CentOS 6.4 и более поздних версий: в ядре |Пакет: в [репозитории](http://olcentgbl.trafficmanager.net/openlogic/6/openlogic/x86_64/RPMS/) в разделе "WALinuxAgent" <br/>Исходный код: [GitHub](https://github.com/Azure/WALinuxAgent) |
| [CoreOS](https://coreos.com/docs/running-coreos/cloud-providers/azure/)<p> CoreOS теперь имеет [окончание жизни](https://coreos.com/os/eol/) от 26 мая 2020 г. |Больше не доступно | | |
| Debian от Credativ |8. x, 9. x, 10. x |В ядре |Пакет: в репозитории в разделе WAAgent  <br/>Исходный код: [GitHub](https://github.com/Azure/WALinuxAgent) |
|Контейнер флаткар Linux от Кинволк| Профессиональная, стабильная, бета-версия| В ядре | WA-Linux-агент уже установлен в/УСР/шаре/ОЕМ/бин/ваажент |
| Oracle Linux по Oracle |6.x, 7.x, 8.x |В ядре |Пакет: в репозитории в разделе WALinuxAgent  <br/>Исходный код: [GitHub](https://go.microsoft.com/fwlink/p/?LinkID=250998) |
| [Red Hat Enterprise Linux с помощью Red Hat](../workloads/redhat/overview.md) |6.x, 7.x, 8.x |В ядре |Пакет: в репозитории в разделе WALinuxAgent  <br/>Исходный код: [GitHub](https://github.com/Azure/WALinuxAgent) |
| SUSE Linux Enterprise по SUSE |SLES/SLES для SAP 11. x, 12. x, 15. x <br/> [Жизненный цикл образа общедоступного облака SUSE](https://www.suse.com/c/suse-public-cloud-image-life-cycle/) |В ядре |Пакет:<p> для версии 11: в репозитории [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools);<br>для версии 12: входит в состав модуля Public Cloud в python-azure-agent.<br/>Исходный код: [GitHub](https://go.microsoft.com/fwlink/p/?LinkID=250998) |
| openSUSE по SUSE |openSUSE Leap 15.x |В ядре |Пакет: в репозитории [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools) под именем python-azure-agent <br/>Исходный код: [GitHub](https://github.com/Azure/WALinuxAgent) |
| Ubuntu на каноническом |Ubuntu Server и Pro. 16. x, 18. x, 20. x<p>Сведения о расширенной поддержке для Ubuntu 12,04 и 14,04 см. в разделе [Расширенная безопасность обслуживания Ubuntu](https://www.ubuntu.com/esm). |В ядре |Пакет: в репозитории в разделе WALinuxAgent  <br/>Исходный код: [GitHub](https://github.com/Azure/WALinuxAgent) |

## <a name="image-update-cadence"></a>Ритмичность обновления образа

Azure требует, чтобы издатели рекомендуемых дистрибутивов Linux регулярно выполняли обновление образов в Azure Marketplace с последними исправлениями и исправлениями безопасности с частотой квартала или более высокой. Обновленные образы в Marketplace автоматически становятся доступными клиентам в виде новых версий SKU образа. Дополнительные сведения о поиске образов Linux см. [в статье Поиск образов виртуальных машин Linux в Azure Marketplace](./cli-ps-findimage.md).

## <a name="azure-tuned-kernels"></a>Ядра, настроенные Azure

Azure тесно работает с различными предустановочными дистрибутивами Linux для оптимизации образов, опубликованных в Azure Marketplace. Одним из аспектов этой совместной работы является разработка "настроенных" ядер Linux, оптимизированных для платформы Azure и поставляемых как полностью Поддерживаемые компоненты дистрибутива Linux. Azure-Tuned ядра включают новые функции и улучшения производительности, а также более быстрый (обычно ежеквартальный) ритмичность по сравнению с ядром по умолчанию или универсальными ядрами, доступными в дистрибутиве.

В большинстве случаев эти ядра будут предварительно установлены на образы по умолчанию в Azure Marketplace, поэтому клиенты сразу же получат преимущества этих оптимизированных ядер. Дополнительные сведения об этих Azure-Tuned ядрах можно найти по следующим ссылкам:

- [CentOS Azure-Tuned ядро — доступно через подпись виртуализации CentOS](https://wiki.centos.org/SpecialInterestGroup/Virtualization)
- [Ядро Debian Cloud — доступно в образе "однопортовый" Debian 10 и Debian 9 "в Azure](https://wiki.debian.org/Cloud/MicrosoftAzure)
- [Ядро Azure-Tuned SLES](https://www.suse.com/c/a-different-builtin-kernel-for-azure-on-demand-images/)
- [Ядро Azure-Tuned Ubuntu](https://blog.ubuntu.com/2017/09/21/microsoft-and-canonical-increase-velocity-with-azure-tailored-kernel)
- [Контейнер флаткар Linux Pro](https://azuremarketplace.microsoft.com/marketplace/apps/kinvolk.flatcar_pro)

## <a name="partners"></a>Участники

### <a name="coreos"></a>CoreOS

CoreOS по [окончании срока жизни](https://coreos.com/os/eol/) составляет 26 мая 2020 г.
Корпорация Майкрософт имеет два (2) канала миграции для CoreOS пользователей.

- Флаткар by Кинволк (см. запись Флаткар Container Linux by Кинволк).
- [ОС Fedora Core](https://docs.fedoraproject.org/en-US/fedora-coreos/provisioning-azure/) (клиенты должны передать собственный образ. Ниже приведена [Документация по миграции](https://docs.fedoraproject.org/en-US/fedora-coreos/migrate-cl/).

### <a name="credativ"></a>credativ

[https://www.credativ.de/en/portfolio/support/open-source-support-center/](https://www.credativ.de/en/portfolio/support/open-source-support-center/)

credativ — это независимая компания консультации и услуг, которая специализируется на разработке и реализации профессиональных решений с помощью бесплатного программного обеспечения. Как ведущие специалисты по открытым источникам, credativ имеет международное распознавание с множеством ИТ-отделов, использующих их поддержку. В сочетании с Майкрософт credativ готовит образы Debian. Образы специально предназначены для работы в Azure и могут легко управляться с помощью платформы. credativ также будет поддерживать долгосрочное обслуживание и обновление образов Debian для Azure с помощью центров поддержки с открытым кодом.

### <a name="kinvolk"></a>кинволк
[https://www.kinvolk.io/flatcar-container-linux/](https://www.kinvolk.io/flatcar-container-linux/)

Кинволк — это компания, за которую Флаткар контейнер Linux, продолжая первоначальную концепцию CoreOS для минимального, неизменяемого и автоматического обновления для контейнерных приложений. В качестве минимального дистрибутив Флаткар содержит только те пакеты, которые необходимы для развертывания контейнеров. Неизменяемая файловая система гарантирует согласованность и безопасность, в то время как возможности автоматического обновления позволяют всегда обновляться последними исправлениями безопасности. 

Флаткар Container Linux создается с помощью глобальной группы разработчиков Linux и специалистов по ИТ-контейнерам, которые предлагают необязательную коммерческую подписку на техническую поддержку, включающую Круглосуточная отклики, безопасность и технические оповещения, а также эксклюзивные образы, оптимизированные для Azure, включая канал долгосрочной поддержки.


### <a name="oracle"></a>Oracle;

[https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html)

Стратегия Oracle — это предоставление широкого портфеля решений для общедоступных и частных облаков. Это дает клиентам свободу выбора и гибкость при развертывании программного обеспечения Oracle в облаках Oracle, а также в других облаках. Партнерство Oracle с корпорацией Майкрософт позволяет клиентам развертывать программное обеспечение Oracle в общедоступных и частных облаках Майкрософт с достоверностью сертификации и поддержки из Oracle.  Обязательства и инвестиции Oracle в решениях общедоступного и частного облаков Oracle не меняются.

### <a name="red-hat"></a>Red Hat

[https://www.redhat.com/en/partners/strategic-alliance/microsoft](https://www.redhat.com/en/partners/strategic-alliance/microsoft)

Ведущим поставщиком в мире решений с открытым исходным кодом, Red Hat помогает более чем 90% компаний из списка Fortune 500 решать бизнес-задачи, вынуждовать стратегии ИТ и бизнеса и подготовиться к будущему технологии. Red Hat достигает этого, предоставляя безопасные решения с помощью открытой бизнес-модели и доступной, прогнозируемой модели подписки.

### <a name="suse"></a>SUSE

[https://www.suse.com/suse-linux-enterprise-server-on-azure](https://www.suse.com/suse-linux-enterprise-server-on-azure)

SUSE Linux Enterprise Server в Azure — проверенная платформа, предоставляющая высокую степень надежности и безопасности для работы в облаке. Универсальная платформа SUSE Linux легко интегрируется с облачными службами Azure, создавая облачную среду с простым управлением. Свыше 9200 сертифицированных приложений для SUSE Linux Enterprise Server от более чем 1800 независимых поставщиков программного обеспечения гарантируют, что рабочие нагрузки, поддерживаемые в центре обработки данных, могут быть беспрепятственно развернуты в Azure.

### <a name="canonical"></a>Canonical

[https://www.ubuntu.com/cloud/azure](https://www.ubuntu.com/cloud/azure)

Инженерно-техническое открытое сообщество Canonical способствует успешному распространению Ubuntu в клиентских, серверных и облачных средах, включая персональные облачные службы для потребителей. Canonical представляет Ubuntu как единую бесплатную платформу, используемую в различных средах, от телефона до облака, с семейством согласованных интерфейсов для телефонов, планшетов, телевизоров и настольных компьютеров. Это делает Ubuntu лучшим вариантом для различных учреждений, от поставщиков общедоступных облаков до создателей бытовой электроники, и фаворитом среди частных технологических решений.

Благодаря центрам разработки и проектирования по всему миру Canonical имеет уникальные возможности для партнерства с производителями оборудования, поставщиками содержимого и разработчиками программного обеспечения для вывода на рынок решений Ubuntu для разных сред, от компьютеров до серверов и портативных устройств.
