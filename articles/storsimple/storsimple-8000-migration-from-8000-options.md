---
title: Варианты переноса данных с устройств StorSimple 8000
description: Содержит общие сведения о параметрах переноса данных из StorSimple 8000 Series.
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 03/25/2020
ms.author: alkohli
ms.openlocfilehash: 4839f8211e678f5fc2fb3572c7eaa545fbee6c6c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94961198"
---
# <a name="options-to-migrate-data-from-storsimple-8000-series"></a>Параметры для переноса данных из серии StorSimple 8000

> [!IMPORTANT]
> 31 декабря 2022. Серия StorSimple 8000 пойдет в состояние прекращения поддержки (EOS). Мы рекомендуем перенести пользователей серии StorSimple 8000 в один из вариантов, описанных в документе.

Для ряда StorSimple 8000 достигнут [конец поддержки](https://support.microsoft.com/lifecycle/search?alpha=Azure%20StorSimple%208000%20Series) в декабре 2022. Клиенты, которые работают с серией StorSimple 8000, имеют возможность обновления до других гибридных служб первого разработчика Azure. В этой статье описываются гибридные варианты, доступные для переноса данных в Azure.

## <a name="migration-options"></a>Варианты переноса

У клиентов, использующих серии StorSimple 8000, есть варианты Azure или сторонние параметры.

### <a name="azure-options"></a>Возможности Azure

#### <a name="migrate-to-azure-file-sync"></a>Миграция в службу "Синхронизация файлов Azure"

Этот новый вариант миграции позволяет клиентам хранить файловые ресурсы своей организации в службе файлов Azure. Затем эти файловые ресурсы централизуются для локального доступа с помощью службы "Синхронизация файлов Azure" (AFS). AFS можно развернуть на узле Windows Server. Фактический перенос данных выполняется с помощью средства миграции или путем перемещения копии узла.

Дополнительные сведения о том, как выполнить миграцию данных в Синхронизация файлов Azure, см. в [Синхронизация файлов Azureах StorSimple 8100 и 8600](../storage/files/storage-files-migration-storsimple-8000.md).

### <a name="third-party-options"></a>Сторонние решения

#### <a name="migrate-to-panzura-freedom-nas"></a>Перенос в Panzura Freedom NAS

Пользователи серии StorSimple 5000-7000 и StorSimple 8000 могут перейти на Панзура свободу NAS для сохранения данных в Azure. Решение Панзура свобода предоставляет решение NAS, охватывающее центры обработки данных, офисы, общедоступные и частные облака. Решение позволяет выполнять локальные, гибридные и облачные рабочие процессы с данными для NFS, SMB и мобильных клиентов.

Такой вариант миграции поддерживается Panzura, и пользователи могут начать с создания запроса в техническую поддержку по миграции на веб-сайте [Panzura](https://panzura.com/migrate-storsimple-panzura/).

#### <a name="migrate-to-nasuni"></a>Переход на Nasuni

Перенос всей среды StorSimple на стабильную, безопасную и высокопроизводительную платформу файловых служб легко благодаря Nasuni. Nasuni обеспечивает безопасность и производительность локальной файловой системы, а также объединяет ее с масштабируемостью и устойчивостью Azure.  В качестве ведущего поставщика независимого программного обеспечения Azure Nasuni предоставляет все средства, необходимые для переноса данных StorSimple на современные платформы, которые позволяют обмениваться файлами и совместно работать с ними в нескольких расположениях.

Начните работу уже сегодня: [веб-сайт Nasuni](https://info.nasuni.com/storsimple8000-webinar).

<!-- 04/09/2020 v-grpr (priestlg) - As per request, commenting out this section because the information that will go into this section is forthcoming
#### Migrate to Cohesity

Cohesity enables you to migrate data from your current StorSimple 5000–7000 to the Cohesity Data Platform on Azure. The Cohesity Data Platform is a software-defined web-scale solution that consolidates files, backups, objects, and VMs onto a single cloud-native solution. After migration to the Data Platform, you can manage, protect, and provision data and apps from cloud to core through a single pane of glass. With Cohesity, start with as few as three nodes. 

Learn more on [migration to the Cohesity Data Platform](https://info.cohesity.com/migrate-from-storsimple-to-cohesity.html).

#### Migrate to Nasuni

Nasuni makes it easy for StorSimple 5000-7000 customers to migrate and keep their data in Azure.  Nasuni is a leading Azure-based NAS storage solution, giving customers the performance and security they expect from on-prem solutions, with cloud economics and scale.  In addition to high performance file storage, Nasuni and Azure handle backup and DR, while allowing you to share and collaborate on your data around the globe with centralized file storage management. 

Nasuni has the experience to make your migration easy – get started today: https://info.nasuni.com/nasuni-storsimple-migration

#### Migrate to Talon FAST

Talon makes it easy for StorSimple 5000-7000 customers to continue to leverage the benefits they valued so much in the StorSimple platform (small on-site footprint backed by unlimited cloud resources) with even greater function.  With the Talon FAST solution, customers can migrate and keep their data in Azure, while now having an even smaller software-only onsite footprint and adding benefits such as global file locking, global namespace, and multi-site collaboration.  Talon is a leading Azure ecosystem solution, working with global customers to migrate their on-premises file server workloads into a consolidated, Azure-based footprint without compromising user workflow or experience.  

Learn more about how to evolve to a cloud-consolidated enterprise at https://www.talonstorage.com/alliances/microsoft-storsimple.
-->

## <a name="migration---frequently-asked-questions"></a>Часто задаваемые вопросы о переносе

### <a name="q-when-do-the-storsimple-8000-series-devices-reach-end-of-service"></a>У. Когда вы достигли конца обслуживания устройствам серии StorSimple 8000?

A. [Прекращение поддержки](https://support.microsoft.com/[lifecycle/search?alpha=Azure%20StorSimple%208000%20Series) серии StorSimple 8000 за декабрь 2022. Прекращение поддержки подразумевает, что корпорация Майкрософт больше не сможет предоставлять поддержку оборудования и программного обеспечения этих устройств после декабря 2022. Мы настоятельно рекомендуем начать разработку плана по переносу данных с ваших устройств.

### <a name="q-what-happens-to-the-data-i-have-stored-in-azure"></a>У. Что происходит с данными, которые хранятся в Azure?  

A. Вы можете продолжать использовать данные в Azure после их перемещения в новую службу.

### <a name="q-what-happens-to-the-data-i-have-stored-locally-on-my-storsimple-device"></a>У. Что происходит с данными, которые хранятся локально на устройстве StorSimple?

A. Данные, которые хранятся локально на устройстве, можно скопировать в новую службу, как описано в документах о переносе.

### <a name="q-what-happens-if-i-want-to-keep-my-storsimple-8000-series-appliance"></a>У. Что произойдет, если я хочу разместить мое устройство StorSimple серии 8000?

A. Хотя службы могут продолжать работу, корпорация Майкрософт больше не будет обеспечивать поддержку оборудования и программного обеспечения. Для обеспечения непрерывности бизнес-процессов настоятельно рекомендуется выполнить миграцию.

### <a name="q-what-options-are-available-to-migrate-data-from-storsimple-8000-series-devices"></a>У. Какие варианты доступны для переноса данных с устройств StorSimple 8000?

A. В зависимости от своего сценария пользователи серии StorSimple 8000 имеют следующие варианты миграции:

* **Перенос в службу "Синхронизация файлов Azure"**. Используйте этот вариант, если необходимо переключиться в собственный формат Azure. Службу "Синхронизация файлов Azure" можно использовать для централизованного управления файловыми ресурсами.

* **Другие параметры**. Вы можете обратиться в Служба поддержки Майкрософт, чтобы обсудить параметры миграции, не указанные здесь.

### <a name="q-is-migration-to-other-storage-solutions-supported"></a>У. Поддерживается ли перенос данных в другие решения для хранения?

A. Да. Поддерживается перенос на другие решения для хранения с использованием копии узла данных.

### <a name="q-is-migration-supported-by-microsoft"></a>У. Поддерживается ли перенос корпорацией Майкрософт?

A. Переход с серии 8000 является полностью поддерживаемой операцией. Более того, корпорация Майкрософт рекомендует обратиться в службу поддержки перед началом миграции. Сейчас миграции — это интерактивная операция. Если вы собираетесь перенести данные с устройства StorSimple 8000, обратитесь в [службу поддержки storsimple](mailto:storsimp@microsoft.com).

### <a name="q-what-is-the-pricing-model-for-migration-to-azure-file-sync"></a>У. Какова модель ценообразования для миграции на Синхронизация файлов Azure?

A. При использовании Синхронизация файлов Azure может взиматься плата за подписку для службы. Кроме того, клиентам придется платить за текущие затраты на хранение. Для оценки см. [цены на AFS]( https://azure.microsoft.com/pricing/details/storage/files/) .

### <a name="q-how-long-does-it-take-to-complete-a-migration"></a>У. Сколько времени занимает процесс переноса данных?

A. Время переноса данных зависит от объема данных и выбранного варианта обновления.

## <a name="next-steps"></a>Дальнейшие действия

* [Перенос данных из серии StorSimple 8000 в Синхронизация файлов Azure](../storage/files/storage-files-migration-storsimple-8000.md)