---
title: Основные сведения о Cloud-init
description: Глубокое изучение подготовки виртуальной машины Azure с помощью Cloud-init.
author: danielsollondon
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: 94dd57310375febb4bc9a55efa704a5fbf4e80e8
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102559449"
---
# <a name="diving-deeper-into-cloud-init"></a>Углубленная погружение в Cloud-init
Чтобы узнать больше о [Cloud-init](https://cloudinit.readthedocs.io/en/latest/index.html) или устранить проблему на более глубоком уровне, необходимо понять, как это работает. В этом документе описываются важные компоненты и объясняются особенности Azure.

Если Cloud-init включен в обобщенный образ, а виртуальная машина создается из этого образа, она будет обрабатывать конфигурации и выполнять до 5 этапов во время начальной загрузки. Эти этапы важны, так как они показывают, в какой точке Cloud-init будет применять конфигурации. 


## <a name="understand-cloud-init-configuration"></a>Общие сведения о конфигурации Cloud-Init
Настройка виртуальной машины для работы на платформе означает, что Cloud-init необходимо применить несколько конфигураций, как потребитель образа. Основные конфигурации, с которыми вы будете взаимодействовать, — это `User data` (CustomData), которые поддерживают несколько форматов, описаны [здесь](https://cloudinit.readthedocs.io/en/latest/topics/format.html#user-data-formats). Вы также можете добавлять и запускать скрипты (/Вар/либ/Клауд/скриптс) для дополнительной настройки. ниже это подробно описано в этой статье.

Некоторые конфигурации уже помогут в образы Azure Marketplace, которые входят в состав Cloud-init, например:

1. **Cloud Data Source** — Cloud-init содержит код, который может взаимодействовать с облачными платформами, которые называются "источниками данных". При создании виртуальной машины из образа Cloud-init в [Azure](https://cloudinit.readthedocs.io/en/latest/topics/datasources/azure.html#azure)Cloud-init загружает источник данных Azure, который будет взаимодействовать с конечными точками метаданных Azure, чтобы получить конфигурацию виртуальной машины.
2. **Конфигурация среды выполнения** (/рун/Клауд-ИНИТ)
3. **Конфигурация образа** (/ЕТК/Клауд), например `/etc/cloud/cloud.cfg` , `/etc/cloud/cloud.cfg.d/*.cfg` . Например, в Azure для образов ОС Linux, использующих Cloud-init, есть директива Azure DataSource, которая указывает, какой источник данных следует использовать в Cloud-init, это экономит время инициализации облака:

   ```bash
   /etc/cloud/cloud.cfg.d# cat 90_dpkg.cfg
   # to update this file, run dpkg-reconfigure cloud-init
   datasource_list: [ Azure ]
   ```


## <a name="cloud-init-boot-stages-processing-configuration"></a>Этапы загрузки Cloud-init (конфигурация обработки)

При подготовке с помощью Cloud-init существует 5 этапов загрузки, конфигурации процессов и отображения в журналах.

1. [Этап генератора](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#generator). запускается системный генератор Cloud-init, который определяет, что Cloud-init следует включить в цели загрузки, и, если это так, он включает Cloud-init. 

2. [Облако — инициализация локального этапа](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#local). здесь Cloud-init будет искать локальный источник данных Azure, который позволит Cloud-init взаимодействовать с Azure и применить конфигурацию сети, включая откат.

3. [Этап инициализации Cloud-init (сеть)](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#network): Сетевые подключения должны быть в сети, а также должны быть созданы сведения о таблице маршрутов и сетевую карту. На этом этапе будут запущены модули, перечисленные в `cloud_init_modules` в/ЕТК/Клауд/Клауд.КФГ. Виртуальная машина в Azure будет подключена, временный диск отформатирован, имя узла задается вместе с другими задачами.

   Вот некоторые из них `cloud_init_modules` :
   
   ```bash
   - migrator
   - seed_random
   - bootcmd
   - write-files
   - growpart
   - resizefs
   - disk_setup
   - mounts
   - set_hostname
   - update_hostname
   - ssh
   ```
   
   После этого этапа Cloud-init сообщит платформе Azure о том, что виртуальная машина успешно подготовлена. Возможно, некоторые модули завершились сбоем, но не все сбои модулей приведут к сбою подготовки.

4. [Этап настройки Cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#config). на этом этапе `cloud_config_modules` будут запущены модули, указанные и перечисленные в/ЕТК/Клауд/Клауд.КФГ.


5. [Заключительный этап Cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#final): на этом заключительном этапе `cloud_final_modules` будут запущены модули в, перечисленные в/ЕТК/Клауд/Клауд.КФГ. Здесь приведены модули, которые необходимо запускать в конце процесса загрузки, такие как установка пакета и запуск сценариев и т. д. 

   -   На этом этапе можно выполнять сценарии, помещая их в каталоги в папке `/var/lib/cloud/scripts` :
   - `per-boot` — скрипты в этом каталоге, выполняются при каждой перезагрузке
   - `per-instance` — скрипты в этом каталоге запускаются при первой загрузке нового экземпляра
   - `per-once` — скрипты в этом каталоге выполняются только один раз

## <a name="next-steps"></a>Дальнейшие действия

[Устранение неполадок Cloud-init](cloud-init-troubleshooting.md).
