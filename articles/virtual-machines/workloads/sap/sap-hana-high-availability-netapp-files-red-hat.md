---
title: Высокий уровень доступности SAP HANA с помощью использовании на RHEL | Документация Майкрософт
description: Установите высокий уровень доступности SAP HANA использовании на виртуальных машинах Azure.
services: virtual-machines-linux
documentationcenter: ''
author: rdeltcheva
manager: juergent
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/17/2021
ms.author: radeltch
ms.openlocfilehash: c5f94329920f8c850c0a47dd607ade8e83658b29
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599924"
---
# <a name="high-availability-of-sap-hana-scale-up-with-azure-netapp-files-on-red-hat-enterprise-linux"></a>Высокий уровень доступности SAP HANA с Azure NetApp Files на Red Hat Enterprise Linux

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all 
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736
[1900823]:https://launchpad.support.sap.com/#/notes/1900823
[2292690]:https://launchpad.support.sap.com/#/notes/2292690
[2455582]:https://launchpad.support.sap.com/#/notes/2455582
[2593824]:https://launchpad.support.sap.com/#/notes/2593824
[2009879]:https://launchpad.support.sap.com/#/notes/2009879

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md

В этой статье описывается, как настроить репликацию системы SAP HANA в масштабном развертывании, если файловые системы HANA подключены через NFS с помощью Azure NetApp Files (использовании). В примерах конфигураций и команд установки, номер экземпляра **03** и идентификатор системы Hana **HN1** используются. Для репликации SAP HANA используются один основной узел и по крайней мере один вторичный узел.

Если шаги в этом документе помечены следующими префиксами, то это значение выглядит следующим образом:

- **[A]**: шаг применяется ко всем узлам.
- **[1]**: шаг применяется только к NODE1
- **[2]**: шаг применяется только к NODE2
 
Прежде всего прочитайте следующие примечания и документы SAP:

- примечание к SAP [1928533](https://launchpad.support.sap.com/#/notes/1928533), которое содержит:
    - список размеров виртуальных машин Azure, поддерживаемых для развертывания ПО SAP;
    - важные сведения о доступных ресурсах для каждого размера виртуальной машины Azure;
    - сведения о поддерживаемом программном обеспечении SAP и сочетаниях операционных систем и баз данных;
    - сведения о требуемой версии ядра SAP для Windows и Linux в Microsoft Azure.
- примечание к SAP [2015553](https://launchpad.support.sap.com/#/notes/2015553), в котором описываются предварительные требования к SAP при развертывании программного обеспечения SAP в Azure;
- В примечании SAP [405827](https://launchpad.support.sap.com/#/notes/405827) приведена рекомендуемая файловая система для среды Hana.
- В SAP Note [2002167](https://launchpad.support.sap.com/#/notes/2002167) Рекомендуемые параметры ос для Red Hat Enterprise Linux.
- В примечании SAP [2009879](https://launchpad.support.sap.com/#/notes/2009879) приводятся SAP HANA рекомендации по Red Hat Enterprise Linux.
- примечание к SAP [2178632](https://launchpad.support.sap.com/#/notes/2178632), содержащее подробные сведения обо всех доступных метриках мониторинга для SAP в Azure;
- примечание к SAP [2191498](https://launchpad.support.sap.com/#/notes/2191498), содержащее сведения о необходимой версии агента SAP Host Agent для Linux в Azure;
- примечание к SAP [2243692](https://launchpad.support.sap.com/#/notes/2243692), содержащее сведения о лицензировании SAP в Linux в Azure;
- примечание к SAP [1999351](https://launchpad.support.sap.com/#/notes/1999351), содержащее дополнительные сведения об устранении неполадок, связанных с расширением для расширенного мониторинга Azure для SAP;
- [Вики-сайт сообщества SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) содержит все необходимые заметки SAP для Linux.
- [SAP NetWeaver на виртуальных машинах Linux. Руководство по планированию и внедрению][planning-guide]
- [Развертывание программного обеспечения SAP на виртуальных машинах Linux в Azure][deployment-guide]
- [Общие сведения о развертывании СУБД на виртуальных машинах Azure для рабочих нагрузок SAP][dbms-guide]
- [SAP HANA репликацию системы в кластере Pacemaker.](https://access.redhat.com/articles/3004101)
- Общая документация по RHEL
    - [Общие сведения о надстройке для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
    - [Add-On администрирования высокого уровня доступности.](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
    - [Справочник по Add-On высокого уровня доступности.](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
    - [Настройка SAP HANA репликации системы в Scale-Up в кластере Pacemaker, если файловые системы HANA находятся в общих папках NFS](https://access.redhat.com/solutions/5156571)
- Документация по RHEL для Azure:
    - [Политики поддержки для кластеров высокой доступности RHEL — Виртуальные машины Microsoft Azure как члены кластера.](https://access.redhat.com/articles/3131341)
    - [Установка и настройка High-Availability кластера Red Hat Enterprise Linux 7,4 (и более поздних версий) на Microsoft Azure.](https://access.redhat.com/articles/3252491)
    - [Установите SAP HANA на Red Hat Enterprise Linux для использования в Microsoft Azure.](https://access.redhat.com/solutions/3193782)
    - [Настройка SAP HANA масштабируемой репликации системы в кластере Pacemaker, если файловые системы HANA находятся в общих папках NFS](https://access.redhat.com/solutions/5156571)
- [Приложения NetApp SAP в Microsoft Azure. Использование Azure NetApp Files](https://www.netapp.com/us/media/tr-4746.pdf)
- [Тома NFS версии 4.1 в Azure NetApp Files для SAP HANA](./hana-vm-operations-netapp.md)

## <a name="overview"></a>Обзор

Традиционно в среде масштабирования все файловые системы для SAP HANA подключаются из локального хранилища. Настройка высокого уровня доступности SAP HANA репликации системы на Red Hat Enterprise Linux опубликована в разделе Guide [настройка SAP HANA репликации системы в RHEL](./sap-hana-high-availability-rhel.md)

Чтобы обеспечить SAP HANA высокую доступность системы масштабирования на [Azure NetApp Files](../../../azure-netapp-files/index.yml) общих ресурсах NFS, нам потребуется дополнительная настройка ресурсов в кластере, чтобы ресурсы Hana можно было восстановить, когда один узел теряет доступ к общим папкам NFS в использовании.  Кластер управляет подключениями NFS, что позволяет отслеживать работоспособность ресурсов. Применяются зависимости между подключением файловой системы и SAP HANAными ресурсами.  

![SAP HANA масштабируемости высокой доступности в использовании](./media/sap-hana-high-availability-rhel/sap-hana-scale-up-netapp-files-red-hat.png)

SAP HANA файловые системы подключаются к общим ресурсам NFS с помощью Azure NetApp Files на каждом узле. Файловые системы/Hana/Data,/Hana/log и/Hana/Shared уникальны для каждого узла. 

Подключено к NODE1 (**hanadb1**)

- 10.32.2.4:/**hanadb1**-Data-mnt00001 на/Hana/Data
- 10.32.2.4:/**hanadb1**-mnt00001 в/Hana/log
- 10.32.2.4:/**hanadb1**-Shared-mnt00001 на/Hana/Shared

Подключено к NODE2 (**hanadb2**)

- 10.32.2.4:/**hanadb2**-Data-mnt00001 на/Hana/Data
- 10.32.2.4:/**hanadb2**-mnt00001 в/Hana/log
- 10.32.2.4:/**hanadb2**-Shared-mnt00001 на/Hana/Shared

> [!NOTE]
> Файловые системы/Hana/Shared,/Hana/Data и/Hana/log не являются общими для двух узлов. Каждый узел кластера имеет собственные, отдельные файловые системы.   

SAP HANA Конфигурация репликации системы использует выделенное виртуальное имя узла и виртуальные IP-адреса. Балансировщику нагрузки в Azure нужен виртуальный IP-адрес. Ниже показана конфигурация подсистемы балансировки нагрузки.

- Конфигурация внешнего интерфейса: IP-адрес 10.32.0.10 для HN1-DB
- Внутренняя конфигурация: подключена к основным сетевым интерфейсам всех виртуальных машин, которые должны входить в репликацию системы HANA.
- Порт пробы: Порт 62503
- Правила балансировки нагрузки: 30313 TCP, 30315 TCP, 30317 TCP, 30340 TCP, 30341 TCP, 30342 TCP (если используется базовая подсистема балансировки нагрузки Azure)  

## <a name="set-up-the-azure-netapp-file-infrastructure"></a>Настройка инфраструктуры файлов Azure NetApp

Прежде чем продолжить настройку для Azure NetApp Filesной инфраструктуры, ознакомьтесь с [документацией по файлам Azure NetApp](../../../azure-netapp-files/index.yml).

Azure NetApp Files доступен в нескольких [регионах Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Проверьте, предлагает ли выбранный регион Azure Azure NetApp Files.

Сведения о доступности Azure NetApp Files по регионам Azure см. в разделе [доступность Azure NetApp Files по регионам Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all).

Перед развертыванием Azure NetApp Files запросите подключение к Azure NetApp Files, перейдя к [инструкциям Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).

### <a name="deploy-azure-netapp-files-resources"></a>Развертывание ресурсов Azure NetApp Files

В следующих инструкциях предполагается, что вы уже развернули [виртуальную сеть Azure](../../../virtual-network/virtual-networks-overview.md). Azure NetApp Files ресурсы и виртуальные машины, на которых будут подключены Azure NetApp Files ресурсы, должны быть развернуты в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure.

1. Если вы еще не развернули ресурсы, запросите подключение [к Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).

2. Создайте учетную запись NetApp в выбранном регионе Azure, следуя инструкциям в разделе [Создание учетной записи NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).

3.  Настройте пул емкости Azure NetApp Files, следуя инструкциям в разделе [Настройка пула емкости Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).

    Архитектура HANA, представленная в этой статье, использует один пул емкости Azure NetApp Files на уровне *Ultra* Service. Для рабочих нагрузок HANA в Azure рекомендуется использовать [уровень обслуживания](../../../azure-netapp-files/azure-netapp-files-service-levels.md)Azure NetApp Files *Ultra* или *Premium* .

4.  Делегируйте подсеть Azure NetApp Files, как описано в инструкциях в разделе [Делегирование подсети для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).

5.  Разверните Azure NetApp Files тома, следуя инструкциям в разделе [Создание тома NFS для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).

    При развертывании томов обязательно выберите версию Нфсв 4.1. Разверните тома в выделенной  подсети Azure NetApp Files. IP-адреса томов Azure NetApp Files назначаются автоматически.

    Помните, что ресурсы Azure NetApp Files и виртуальные машины Azure должны находиться в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure. Например, hanadb1-Data-mnt00001, hanadb1-log-mnt00001 и т. д. — это имена томов и nfs://10.32.2.4/hanadb1-data-mnt00001, nfs://10.32.2.4/hanadb1-log-mnt00001 и т. д. — это пути к файлам Azure NetApp Files.
    
    На **hanadb1**

    - Volume hanadb1-Data-mnt00001 (nfs://10.32.2.4:/hanadb1-data-mnt00001) 
    - Volume hanadb1-log-mnt00001 (nfs://10.32.2.4:/hanadb1-log-mnt00001)
    - Volume hanadb1-Shared-mnt00001 (nfs://10.32.2.4:/hanadb1-shared-mnt00001)
    
    На **hanadb2**

    - Volume hanadb2-Data-mnt00001 (nfs://10.32.2.4:/hanadb2-data-mnt00001)
    - Volume hanadb2-log-mnt00001 (nfs://10.32.2.4:/hanadb2-log-mnt00001)
    - Volume hanadb2-Shared-mnt00001 (nfs://10.32.2.4:/hanadb2-shared-mnt00001)

### <a name="important-considerations"></a>Важные замечания

При создании Azure NetApp Files для SAP HANA систем масштабирования необходимо учитывать следующее.

- Пул минимальных мощностей — 4 тебибитес (тиб).
- Минимальный размер тома — 100 гибибайтах (гиб).
- Azure NetApp Files и все виртуальные машины, на которых будут подключаться Azure NetApp Filesные тома, должны находиться в одной виртуальной сети Azure или в [одноранговой виртуальной сети](../../../virtual-network/virtual-network-peering-overview.md) в том же регионе.
- Выбранная виртуальная сеть должна иметь подсеть, делегированную Azure NetApp Files.
- Пропускная способность тома Azure NetApp Files является функцией квоты тома и уровня обслуживания, как описано в статье [об уровне обслуживания для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). При изменении размера томов HANA Azure NetApp убедитесь, что полученная пропускная способность соответствует требованиям к системе HANA.
- С помощью [политики экспорта](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md)Azure NetApp Files можно управлять разрешенными клиентами, типом доступа (чтение и запись, только чтение и т. д.).
- Azure NetApp Files функция еще не поддерживает зону. Сейчас эта функция не развернута во всех зонах доступности в регионе Azure. Учитывайте возможную задержку в некоторых регионах Azure.

> [!IMPORTANT]
> Для рабочих нагрузок SAP HANA низкий показатель задержки чрезвычайно важен. Поработайте с представителями корпорации Майкрософт, чтобы убедиться, что виртуальные машины и Azure NetApp Filesные тома развернуты в близком к ней.

### <a name="sizing-of-hana-database-on-azure-netapp-files"></a>Изменение размера базы данных HANA на Azure NetApp Files

Пропускная способность Azure NetApp Filesого тома — это функция размера тома и уровня обслуживания, как описано в статье [об уровне обслуживания для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md).

При проектировании инфраструктуры для SAP в Azure необходимо учитывать некоторые минимальные требования к хранилищу SAP, которые переводятся в характеристики минимальной пропускной способности.

- Чтение и запись в/Hana/log из 250 МБ в секунду (МБ/с) с размером операций ввода-вывода в 1 МБ.
- Прочтите действие не менее 400 МБ/с для/Hana/Data для размеров операций ввода-вывода 16 МБ и 64-МБ.
- Операция записи по крайней мере 250 МБ/с для/Hana/Data с размерами операций ввода-вывода 16 МБ и 64-МБ.

Ниже представлены [ограничения пропускной способности Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md) на 1 ТиБ квоты тома.

- Уровень хранилища Premium — 64 MiB/с.
- Ultra Storage уровня — 128 MiB/с.

Чтобы удовлетворить минимальные требования к пропускной способности SAP для/Hana/Data и/Hana/log и рекомендации для/Hana/Shared, рекомендуется использовать следующие размеры:

|    Том    | Размер уровня хранилища Premium | Размер Ultra Storage уровня | Поддерживаемый протокол NFS |
| :----------: | :--------------------------: | :------------------------: | :--------------------: |
|  hana/log;   |            4 ТиБ             |           2 ТиБ            |          Версия 4.1          |
|  hana/data;  |           6.3 ТиБ            |          3.2 ТиБ           |          Версия 4.1          |
| /hana/shared |           1 x RAM            |          1 x RAM           |          Версия 3 или версия 4.1    |


> [!NOTE]
> Azure NetApp Files рекомендации по выбору размера, указанные здесь, предназначены для удовлетворения минимальных требований, которые SAP рекомендует для поставщиков инфраструктуры. В реальных развертываниях клиентов и сценариях рабочей нагрузки эти размеры могут быть недостаточно. Поэтому примите эти рекомендации в качестве отправной точки и адаптируйтесь в соответствии с требованиями своей рабочей нагрузки.

> [!TIP]
> Вы можете динамически изменять размер Azure NetApp Files томов без *отключения* томов, отключения виртуальных машин или завершения SAP HANA. Такой подход обеспечивает гибкость для удовлетворения ожидаемых и непредвиденных требований к пропускной способности приложения.

> [!NOTE]
> Все команды для подключения/Hana/Shared в этой статье представлены для томов/Hana/Shared Нфсв 4.1.
> Если вы развернули тома/Hana/Shared в качестве томов NFSv3, не забудьте настроить команды подключения для/Hana/Shared для NFSv3.


## <a name="deploy-linux-virtual-machine-via-azure-portal"></a>Развертывание виртуальной машины Linux с помощью портал Azure 

Сначала необходимо создать тома Azure NetApp Files. Затем выполните следующие действия.

1.  Создайте группу ресурсов.
2.  Создайте виртуальную сеть.
3.  Создайте группу доступности. Настройте максимальное число доменов обновления.
4.  Создайте подсистему балансировки нагрузки (внутреннюю). Мы рекомендуем подсистему балансировки нагрузки ценовой категории "Стандартный".
    Выберите виртуальную сеть, созданную на шаге 2.
5.  Создайте виртуальную машину 1 (**hanadb1**). 
6.  Создайте виртуальную машину 2 (**hanadb2**).  
7.  При создании виртуальной машины не будет добавляться ни один диск, так как все точки подключения будут находиться на общих ресурсах NFS от Azure NetApp Files. 

> [!IMPORTANT]
> Плавающий IP-адрес не поддерживается для вторичной IP-конфигурации NIC в сценариях балансировки нагрузки. Дополнительные сведения см. в статье [ограничения балансировщика нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, разверните вторую сетевую карту.    

> [!NOTE] 
> Если в серверный пул внутреннего (без общедоступного IP-адреса) Azure Load Balancer ценовой категории "Стандартный" помещаются виртуальные машины без общедоступных IP-адресов, у них не будет исходящего подключения к Интернету без дополнительной настройки, разрешающей маршрутизацию к общедоступным конечным точкам. Подробные сведения о такой настройке см. в статье [Подключение к общедоступной конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. категория "Стандартный") в сценариях обеспечения высокого уровня доступности SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).

8.  Если вы используете подсистему балансировки нагрузки ценовой категории "Стандартный", выполните следующие шаги по настройке.
    1.  Сначала создайте пула IP-адресов для интерфейсной части.
        1.  Откройте подсистему балансировки нагрузки, выберите **пул интерфейсных IP-адресов** и щелкните **Добавить**.
        1.  Введите имя нового пула интерфейсных IP-адресов (например, **hana-frontend**).
        1.  Задайте для **назначения** значение **static** и введите IP-адрес (например, **10.32.0.10**).
        1.  Щелкните **ОК**.
        1.  Когда пул интерфейсных IP-адресов будет создан, запишите его IP-адрес.
    1.  Теперь создайте серверный пул.
        1.  Откройте подсистему балансировки нагрузки, выберите **серверные пулы** и щелкните **Добавить**.
        1.  Введите имя нового серверного пула (например, **hana-backend**).
        1.  Щелкните **Добавить виртуальную машину**.
        1.  Выберите **Виртуальная машина**.
        1.  Выберите виртуальные машины кластера SAP HANA и IP-адреса для них.
        1.  Выберите **Добавить**.
    1.  Создайте зонд работоспособности.
        1.  Откройте подсистему балансировки нагрузки, выберите **Зонды работоспособности** и щелкните **Добавить**.
        1.  Введите имя нового зонда работоспособности (например, **hana-hp**).
        1.  Выберите протокол TCP и порт 625 **03**. Сохраните значение "5" для параметра **Интервал** и значение "2" для параметра **Порог состояния неработоспособности**.
        1.  Щелкните **ОК**.
    1.  Теперь создайте правила балансировки нагрузки.
        1.  Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
        1.  Введите имя нового правила балансировки нагрузки (например, **hana-lb**).
        1.  Выберите интерфейсный пул IP-адресов, серверный пул и зонд работоспособности, который вы создали ранее (например, **hana-frontend**, **hana-backend** и **hana-hp**).
        1.  Выберите **Порты высокой доступности**.
        1.  Увеличьте **время ожидания** до 30 минут.
        1.  Не забудьте **включить плавающий IP-адрес**.
        1.  Щелкните **ОК**.


9. Если же ваш сценарий требует использовать подсистему балансировки нагрузки ценовой категории "Базовый", выполните следующие шаги по настройке.
    1.  Настройте подсистему балансировки нагрузки. Сначала создайте пула IP-адресов для интерфейсной части.
        1.  Откройте подсистему балансировки нагрузки, выберите **пул интерфейсных IP-адресов** и щелкните **Добавить**.
        1.  Введите имя нового пула интерфейсных IP-адресов (например, **hana-frontend**).
        1.  Задайте для **назначения** значение **static** и введите IP-адрес (например, **10.32.0.10**).
        1.  Щелкните **ОК**.
        1.  Когда пул интерфейсных IP-адресов будет создан, запишите его IP-адрес.
    1.  Теперь создайте серверный пул.
        1.  Откройте подсистему балансировки нагрузки, выберите **серверные пулы** и щелкните **Добавить**.
        1.  Введите имя нового серверного пула (например, **hana-backend**).
        1.  Щелкните **Добавить виртуальную машину**.
        1.  Выберите группу доступности, созданную на шаге 3.
        1.  Выберите виртуальные машины кластера SAP HANA.
        1.  Щелкните **ОК**.
    1.  Создайте зонд работоспособности.
        1.  Откройте подсистему балансировки нагрузки, выберите **Зонды работоспособности** и щелкните **Добавить**.
        1.  Введите имя нового зонда работоспособности (например, **hana-hp**).
        1.  Выберите протокол **TCP** и порт 625 **03**. Сохраните значение "5" для параметра **Интервал** и значение "2" для параметра **Порог состояния неработоспособности**.
        1.  Щелкните **ОК**.
    1.  Если используется SAP HANA 1.0, создайте правила балансировки нагрузки.
        1.  Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
        1.  Введите имя нового правила балансировки нагрузки (например, hana-lb-3 **03** 15).
        1.  Выберите интерфейсный пул IP-адресов, серверный пул и зонд работоспособности, который вы создали ранее (например, **hana-frontend**).
        1.  Для параметра **Протокол** сохраните значение **TCP** и введите порт 3 **03** 15.
        1.  Увеличьте **время ожидания** до 30 минут.
        1.  Не забудьте **включить плавающий IP-адрес**.
        1.  Щелкните **ОК**.
        1.  Повторите эти шаги для порта 3 **03** 17.
    1.  Если используется SAP HANA 2.0, создайте правила балансировки нагрузки для системной базы данных.
        1.  Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
        1.  Введите имя нового правила балансировки нагрузки (например, hana-lb-3 **03** 13).
        1.  Выберите интерфейсный пул IP-адресов, серверный пул и зонд работоспособности, который вы создали ранее (например, **hana-frontend**).
        1.  Для параметра **Протокол** сохраните значение **TCP** и введите порт 3 **03** 13.
        1.  Увеличьте **время ожидания** до 30 минут.
        1.  Не забудьте **включить плавающий IP-адрес**.
        1.  Щелкните **ОК**.
        1.  Повторите эти шаги для порта 3 **03** 14.
    1.  Если используется SAP HANA 2.0, создайте правила балансировки нагрузки для базы данных клиента.
        1.  Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
        1.  Введите имя нового правила балансировки нагрузки (например, hana-lb-3 **03** 40).
        1.  Выберите пул внешних IP-адресов, внутренний пул и зонд работоспособности, созданные ранее (например, **hana-frontend**).
        1.  Для параметра **Протокол** сохраните значение **TCP** и введите порт 3 **03** 40.
        1.  Увеличьте **время ожидания** до 30 минут.
        1.  Не забудьте **включить плавающий IP-адрес**.
        1.  Щелкните **ОК**.
        1.  Повторите эти шаги для портов 3 **03** 41 и 3 **03** 42.

Дополнительные сведения о портах, требуемых для SAP HANA, см. в разделе [подключения к базам данных клиента](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6/latest/en-US/7a9343c9f2a2436faa3cfdb5ca00c052.html) в руководству по [базам данных клиента SAP HANA](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6) или в статье о SAP [2388694](https://launchpad.support.sap.com/#/notes/2388694).

> [!IMPORTANT]
> Не включайте метки времени TCP на виртуальных машинах Azure, размещенных за Azure Load Balancer. Включение меток времени TCP помешает работе проб работоспособности. Установите для параметра **net.ipv4.tcp_timestamps** значение **0**. Дополнительные сведения см. в статье [Пробы работоспособности Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md). Также см. [примечание к SAP 2382421](https://launchpad.support.sap.com/#/notes/2382421).

## <a name="mount-the-azure-netapp-files-volume"></a>Подключение Azure NetApp Filesного тома

1. **[A]** Создайте точки подключения для томов базы данных Hana. 

    ```
    mkdir -p /hana/data
    mkdir -p /hana/log
    mkdir -p /hana/shared
    ```

2. **[A]** проверьте параметр домена NFS. Убедитесь, что домен настроен в качестве домена Azure NetApp Files по умолчанию, т. е. **defaultv4iddomain.com** , а для сопоставления установлено значение **никто**.

    ```
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Domain = defaultv4iddomain.com
    [Mapping]
    Nobody-User = nobody
    Nobody-Group = nobody
    ```

    > [!IMPORTANT]    
    > Не забудьте установить домен NFS в/ЕТК/идмапд.конф на виртуальной машине в соответствии с конфигурацией домена по умолчанию в Azure NetApp Files: **defaultv4iddomain.com**. В случае несоответствия между конфигурацией домена NFS-клиента (например, виртуальной машиной) и сервером NFS, т. е. конфигурацией Azure NetApp, разрешения на доступ к файлам на томах NetApp в Azure, подключенных к виртуальным машинам, будут отображаться как никто.
    

3. **[1]** Подключите тома конкретного узла в NODE1 (**hanadb1**) 

    ```
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb1-shared-mnt00001 /hana/shared
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb1-log-mnt00001 /hana/log
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb1-data-mnt00001 /hana/data
    ```
    
4.  **[2]** Подключите тома конкретного узла в NODE2 (**hanadb2**)
    
    ```
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb2-shared-mnt00001 /hana/shared
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb2-log-mnt00001 /hana/log
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb2-data-mnt00001 /hana/data
    ```

5. **[A]** убедитесь, что все тома Hana подключены с помощью протокола NFS версии NFSv4.

    ```
    sudo nfsstat -m
    
    # Verify that flag vers is set to 4.1 
    # Example from hanadb1
    
    /hana/log from 10.32.2.4:/hanadb1-log-mnt00001
    Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.32.0.4,local_lock=none,addr=10.32.2.4
    /hana/data from 10.32.2.4:/hanadb1-data-mnt00001
    Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.32.0.4,local_lock=none,addr=10.32.2.4
    /hana/shared from 10.32.2.4:/hanadb1-shared-mnt00001
    Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.32.0.4,local_lock=none,addr=10.32.2.4
    ```

6. **[A]** проверьте **nfs4_disable_idmapping**. Для него должно быть задано значение **Y**. Чтобы создать структуру каталогов, в которой находится **nfs4_disable_idmapping** , выполните команду mount. Вы не сможете вручную создать каталог в /sys/modules, так как доступ зарезервирован для ядра или драйверов.

    ```
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    
    # If you need to set nfs4_disable_idmapping to Y
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    ```

   Дополнительные сведения об изменении параметра **nfs_disable_idmapping** см. в разделе [https://access.redhat.com/solutions/1749883](https://access.redhat.com/solutions/1749883) . 


## <a name="sap-hana-installation"></a>Установка SAP HANA

1. **[A]** Настройка разрешения имен узлов для всех узлов.

   Вы можете использовать DNS-сервер или внести изменения в файл /etc/hosts на всех узлах. В этом примере используется файл /etc/hosts. Замените IP-адрес и имя узла в следующих командах:

   ```
   sudo vi /etc/hosts
   # Insert the following lines in the /etc/hosts file. Change the IP address and hostname to match your environment  
   10.32.0.4   hanadb1
   10.32.0.5   hanadb2
   ```

2. **[A]** RHEL для конфигурации Hana

   Настройте RHEL, как описано в разделе Примечание SAP на основе версии RHEL.

   - [2292690 - SAP HANA DB: Recommended OS settings for RHEL 7](https://launchpad.support.sap.com/#/notes/2292690) (Примечание по поддержке SAP № 2292690. База данных SAP HANA: рекомендуемые параметры операционной системы для RHEL 7).
   - [2777782-SAP HANA DB: Рекомендуемые параметры ОС для RHEL 8](https://launchpad.support.sap.com/#/notes/2777782)
   - [2455582-Linux: запуск приложений SAP, скомпилированных с помощью GCC 6. x](https://launchpad.support.sap.com/#/notes/2455582)
   - [2593824-Linux: запуск приложений SAP, скомпилированных с помощью GCC 7. x](https://launchpad.support.sap.com/#/notes/2593824) 
   - [2886607-Linux: запуск приложений SAP, скомпилированных с помощью GCC 9. x](https://launchpad.support.sap.com/#/notes/2886607)

3. **[A]** Установка SAP HANA

   При работе с HANA 2,0 SPS 01 параметр MDC используется по умолчанию. При установке системы HANA СИСТЕМДБ и клиент с одинаковым идентификатором безопасности будут созданы вместе. В некоторых случаях вы не хотите использовать клиент по умолчанию. Если вы не хотите создавать первоначальный клиент вместе с установкой, можно использовать Примечание SAP [2629711](https://launchpad.support.sap.com/#/notes/2629711)

    Запустите программу **hdblcm** с DVD-диска HANA. Введите следующие значения на запрос в командной строке:  
    Выберите установку: введите **1** (для установки)  
    Select additional components for installation (Выберите дополнительные компоненты для установки). Введите **1**.  
    Введите путь установки [/Hana/Shared]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    Введите имя локального узла [..]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    "Do you want to add additional hosts to the system? (y/n)" (y/n) [n]: **n**  
    Введите SAP HANA идентификатор системы: введите **HN1**.  
    Введите номер экземпляра [00]: введите **03**  
    Выбор режима базы данных/ввод индекса [1]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    Выберите использование системы/введите индекс [4]: введите **4** (для настраиваемых)  
    Введите расположение томов данных [/Hana/Data]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    Введите расположение томов журнала [/Hana/log]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    "Restrict maximum memory allocation?" [n]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    Введите имя узла сертификата для узла "..." [...]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    Введите пароль пользователя агента узла SAP (sapadm): введите пароль пользователя агента узла.  
    Подтверждение пароля пользователя агента узла SAP (sapadm): введите пароль пользователя агента узла еще раз для подтверждения  
    Введите пароль системного администратора (hn1adm): введите пароль системного администратора.  
    Подтверждение пароля системного администратора (hn1adm): введите пароль системного администратора еще раз для подтверждения  
    Введите корневой каталог системного администратора [/usr/sap/HN1/home]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    Введите системный администратор для входа в систему [/bin/sh]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    Введите идентификатор пользователя системного администратора [1001]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    Введите идентификатор группы пользователей (sapsys) [79]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    Введите пароль пользователя базы данных (системный): введите пароль пользователя базы данных.  
    Подтверждение пароля пользователя базы данных (системный): введите пароль пользователя базы данных еще раз для подтверждения  
    "Restart system after machine reboot?" [n]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию  
    "Do you want to continue? (y/n)" (Вы действительно хотите продолжить? (Да/нет)). Проверьте сводку. Чтобы продолжить, введите **y**  

4. **[A]** Обновите агент узла SAP.

   Скачайте последний архив агента узла SAP на сайте [центра программного обеспечения SAP](https://launchpad.support.sap.com/#/softwarecenter) и выполните следующую команду, чтобы обновить этот агент. Замените путь к архиву, чтобы он указывал на скачанный файл.

   ```
   sudo /usr/sap/hostctrl/exe/saphostexec -upgrade -archive <path to SAP Host Agent SAR>
   ```

5. **[A]** Настройка брандмауэра

   Создайте правило брандмауэра для порта пробы подсистемы балансировки нагрузки Azure.

   ```
   sudo firewall-cmd --zone=public --add-port=62503/tcp
   sudo firewall-cmd --zone=public --add-port=62503/tcp –permanent
   ```

## <a name="configure-sap-hana-system-replication"></a>Configuring SAP HANA System Replication (Настройка репликации системы SAP HANA)

Выполните действия, описанные в разделе Настройка [репликации системы SAP HANA](./sap-hana-high-availability-rhel.md#configure-sap-hana-20-system-replication) , чтобы настроить репликацию системы SAP HANA. 

## <a name="cluster-configuration"></a>Конфигурация кластера

В этом разделе описаны необходимые действия, необходимые для эффективной работе кластера при установке SAP HANA в общие папки NFS с помощью Azure NetApp Files. 

### <a name="create-a-pacemaker-cluster"></a>Создание кластера Pacemaker

Выполните действия, описанные в разделе [Настройка Pacemaker на Red Hat Enterprise Linux](./high-availability-guide-rhel-pacemaker.md) в Azure, чтобы создать базовый кластер Pacemaker для этого сервера Hana.

### <a name="configure-filesystem-resources"></a>Настройка ресурсов файловой системы

В этом примере каждый узел кластера имеет собственные файловые системы HANA NFS/Hana/Shared,/Hana/Data и/Hana/log.   

1. **[1]** переведите кластер в режим обслуживания.

   ```
   pcs property set maintenance-mode=true
   ```

2. **[1]** создайте ресурсы файловой системы для подключений **hanadb1** .

    ```
    pcs resource create hana_data1 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb1-data-mnt00001 directory=/hana/data fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb1_nfs
    pcs resource create hana_log1 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb1-log-mnt00001 directory=/hana/log fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb1_nfs
    pcs resource create hana_shared1 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb1-shared-mnt00001 directory=/hana/shared fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb1_nfs
    ```

3. **[2]** создайте ресурсы файловой системы для подключений **hanadb2** .

    ```
    pcs resource create hana_data2 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb2-data-mnt00001 directory=/hana/data fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb2_nfs
    pcs resource create hana_log2 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb2-log-mnt00001 directory=/hana/log fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb2_nfs
    pcs resource create hana_shared2 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb2-shared-mnt00001 directory=/hana/shared fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb2_nfs
    ```

    `OCF_CHECK_LEVEL=20` атрибут добавляется к операции монитора, чтобы каждый монитор выполнял тестовую файловую систему на чтение и запись. Без этого атрибута операция мониторинга проверяет только то, что файловая система подключена. Это может быть проблемой, поскольку при потере подключения файловая система может остаться подключенной, несмотря на недоступность.

    `on-fail=fence` атрибут также добавляется в операцию Monitor. Если этот параметр включен, то в случае сбоя операции мониторинга на узле этот узел немедленно помещается в ограждение. Без этого параметра поведение по умолчанию — отключить все ресурсы, которые зависят от ресурса, на котором произошел сбой, а затем перезапустить неисправный ресурс, а затем запустить все ресурсы, которые зависят от ресурса, на котором произошел сбой. Это не только может занять много времени, когда ресурс SAPHana зависит от неисправного ресурса, но также может завершиться сбоем. Невозможно успешно завершить работу ресурса SAPHana, если сервер NFS, на котором находятся исполняемые файлы HANA, недоступен.

4. **[1]** Настройка ограничений расположения

   Настройте ограничения расположения, чтобы ресурсы, управляющие уникальными подключениями hanadb1, никогда не выполнялись в hanadb2, и наоборот.

    ```
    pcs constraint location hanadb1_nfs rule score=-INFINITY resource-discovery=never \#uname eq hanadb2
    pcs constraint location hanadb2_nfs rule score=-INFINITY resource-discovery=never \#uname eq hanadb1
    ```

    `resource-discovery=never`Параметр задан, так как уникальные подключения для каждого узла имеют одну и ту же точку подключения. Например, `hana_data1` использует точку подключения `/hana/data` , а `hana_data2` также точку подключения `/hana/data` . Это может вызвать ложное срабатывание операции проверки, когда состояние ресурса проверяется при запуске кластера, и это может привести к ненужному поведению восстановления. Это можно избежать, задав `resource-discovery=never`

5. **[1]** Настройка ресурсов атрибутов

   Настройка ресурсов атрибутов. Эти атрибуты будут установлены в значение true, если все подключения NFS узла (/Hana/Data,/Hana/log и/Hana/Data) подключены и в противном случае будут иметь значение false.

   ```
   pcs resource create hana_nfs1_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs1_active
   pcs resource create hana_nfs2_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs2_active
   ```

6. **[1]** Настройка ограничений расположения

   Настройте ограничения расположения, чтобы гарантировать, что ресурс атрибута hanadb1's никогда не будет выполняться в hanadb2 и наоборот.

   ```
   pcs constraint location hana_nfs1_active avoids hanadb2
   pcs constraint location hana_nfs2_active avoids hanadb1
   ```

7. **[1]** создание ограничений упорядочивания

   Настройте ограничения упорядочивания, чтобы ресурсы атрибута узла начинались только после подключения всех подключений NFS узла.
   
    ```
    pcs constraint order hanadb1_nfs then hana_nfs1_active
    pcs constraint order hanadb2_nfs then hana_nfs2_active
    ```

   > [!TIP]
   > Если в конфигурацию входят файловые системы, за пределами группы `hanadb1_nfs` или `hanadb2_nfs` , включите `sequential=false` параметр, чтобы в файловых системах не было зависимостей упорядочения. Все файловые системы должны начинаться раньше `hana_nfs1_active` , но им не нужно начинать ни в каком порядке относительно друг друга. Дополнительные сведения см. [в разделе разделы справки Настройка репликации системы SAP HANA в Scale-Up в кластере Pacemaker, если файловые системы Hana находятся в общих папках NFS](https://access.redhat.com/solutions/5156571) .

### <a name="configure-sap-hana-cluster-resources"></a>Настройка ресурсов кластера SAP HANA

1. Выполните действия, описанные в разделе [Создание ресурсов кластера SAP HANA](./sap-hana-high-availability-rhel.md#create-sap-hana-cluster-resources) , чтобы создать ресурсы SAP HANA в кластере. После создания SAP HANA ресурсов необходимо создать ограничение для правила расположения между SAP HANAными ресурсами и FileSystem (подключения NFS).

2. **[1]** Настройка ограничений между ресурсами SAP HANA и ПОДКЛЮЧЕНИЯми NFS

   Ограничения правил расположения задаются таким образом, чтобы ресурсы SAP HANA можно было запускать на узле только в том случае, если все подключения NFS узла подключены.

    ```
    pcs constraint location SAPHanaTopology_HN1_03-clone rule score=-INFINITY hana_nfs1_active ne true and hana_nfs2_active ne true
    # On RHEL 7.x
    pcs constraint location SAPHana_HN1_03-master rule score=-INFINITY hana_nfs1_active ne true and hana_nfs2_active ne true
    # On RHEL 8.x
    pcs constraint location SAPHana_HN1_03-clone rule score=-INFINITY hana_nfs1_active ne true and hana_nfs2_active ne true
    # Take the cluster out of maintenance mode
    sudo pcs property set maintenance-mode=false
    ```

   Проверка состояния кластера и всех ресурсов
   > [!NOTE]
   > Эта статья содержит ссылки на термин « *Ведомый*» термин, который корпорация Майкрософт больше не использует. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.
   
    ```
    sudo pcs status
    
    Online: [ hanadb1 hanadb2 ]
    
    Full list of resources:
    
    rsc_hdb_azr_agt(stonith:fence_azure_arm):  Started hanadb1
    
    Resource Group: hanadb1_nfs
    hana_data1 (ocf::heartbeat:Filesystem):Started hanadb1
    hana_log1  (ocf::heartbeat:Filesystem):Started hanadb1
    hana_shared1   (ocf::heartbeat:Filesystem):Started hanadb1
    
    Resource Group: hanadb2_nfs
    hana_data2 (ocf::heartbeat:Filesystem):Started hanadb2
    hana_log2  (ocf::heartbeat:Filesystem):Started hanadb2
    hana_shared2   (ocf::heartbeat:Filesystem):Started hanadb2
    
    hana_nfs1_active   (ocf::pacemaker:attribute): Started hanadb1
    hana_nfs2_active   (ocf::pacemaker:attribute): Started hanadb2
    
    Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hanadb1 hanadb2 ]
    
    Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hanadb1 ]
    Slaves: [ hanadb2 ]
    
    Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):  Started hanadb1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):   Started hanadb1
    ```

## <a name="configure-hana-activeread-enabled-system-replication-in-pacemaker-cluster"></a>Настройка репликации системы HANA Active/Read с поддержкой в кластере Pacemaker

Начиная с SAP HANA 2,0 SPS 01 SAP позволяет выполнять активные и доступные для чтения программы установки для репликации системы SAP HANA, где вторичные системы SAP HANA репликации системы можно использовать для интенсивных рабочих нагрузок чтения. Для поддержки такой настройки в кластере требуется второй виртуальный IP-адрес, который позволяет клиентам получать доступ к дополнительной базе данных SAP HANA, доступной для чтения. Чтобы гарантировать, что вторичный сайт репликации по-прежнему будет доступен после перенаправление, кластеру необходимо переместить виртуальный IP-адрес, сопоставленный с дополнительным ресурсом SAPHana.

Дополнительная конфигурация, необходимая для управления репликацией системы HANA Active/Read с поддержкой в высокодоступном кластере Red Hat с вторым виртуальным IP-адресом, описана в разделе [Настройка репликации в режиме "Active/Read" с поддержкой Hana в кластере Pacemaker](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability-rhel#configure-hana-activeread-enabled-system-replication-in-pacemaker-cluster).  

Прежде чем продолжить, убедитесь, что вы полностью настроили кластер с высоким уровнем доступности Red Hat, управляющий SAP HANA базой данных, как описано в приведенных выше сегментах документации.    


## <a name="test-the-cluster-setup"></a>Проверка настройки кластера

В этом разделе описано, как проверить настроенную систему. 

1. Перед запуском теста убедитесь, что в Pacemaker нет невыполненных действий (через состояние ПК), не существует непредвиденных ограничений на расположение (например, оставшиеся части теста миграции), а репликация системы HANA — это состояние синхронизации, например с Системрепликатионстатус:

    ```
    sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
    ```

2. Проверка конфигурации кластера на случай сбоя, когда узел теряет доступ к общему ресурсу NFS (/Hana/Shared)

   SAP HANA агенты ресурсов зависят от двоичных файлов, `/hana/shared` которые хранятся в для выполнения операций во время отработки отказа. Файловая система  `/hana/shared` подключена через NFS в представленном сценарии.  
   Трудно имитировать сбой, когда один из серверов теряет доступ к общему ресурсу NFS. Тест, который можно выполнить, — это повторное подключение файловой системы с правами только для чтения.
   Такой подход подтверждает, что кластер будет иметь возможность отработки отказа, если доступ к `/hana/shared` будет потерян на активном узле.     


   **Ожидаемый результат:** При создании `/hana/shared` файловой системы только для чтения `OCF_CHECK_LEVEL` атрибут ресурса, `hana_shared1` выполняющего операции чтения/записи в файловой системе, завершится ошибкой, так как не сможет записать что-либо в файловую систему и будет выполнять отработку отказа ресурса Hana.  Такой же результат ожидается, когда узел HANA теряет доступ к общим ресурсам NFS. 

   Состояние ресурсов перед запуском теста:

    ```
    sudo pcs status
    # Example output
    Full list of resources:
     rsc_hdb_azr_agt        (stonith:fence_azure_arm):      Started hanadb1

     Resource Group: hanadb1_nfs
         hana_data1 (ocf::heartbeat:Filesystem):    Started hanadb1
         hana_log1  (ocf::heartbeat:Filesystem):    Started hanadb1
         hana_shared1       (ocf::heartbeat:Filesystem):    Started hanadb1

    Resource Group: hanadb2_nfs
         hana_data2 (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_log2  (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_shared2       (ocf::heartbeat:Filesystem):    Started hanadb2

     hana_nfs1_active       (ocf::pacemaker:attribute):     Started hanadb1
     hana_nfs2_active       (ocf::pacemaker:attribute):     Started hanadb2

     Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
         Started: [ hanadb1 hanadb2 ]

     Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
         Masters: [ hanadb1 ]
         Slaves: [ hanadb2 ]

     Resource Group: g_ip_HN1_03
         nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hanadb1
         vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hanadb1
    ```

   /Hana/Shared можно поместить в режиме только для чтения на активном узле кластера, используя приведенную ниже команду.

    ```
    sudo mount -o ro 10.32.2.4:/hanadb1-shared-mnt00001 /hana/shared
    ```

   hanadb1 будет перезагружена или выключена на основе действия, заданного для stonith ( `pcs property show stonith-action` ).  После отключения сервера (hanadb1) ресурс HANA переходит в hanadb2. Состояние кластера можно проверить с hanadb2.

    ```
    pcs status

    Full list of resources:

     rsc_hdb_azr_agt        (stonith:fence_azure_arm):      Started hanadb2

     Resource Group: hanadb1_nfs
         hana_data1 (ocf::heartbeat:Filesystem):    Stopped
         hana_log1  (ocf::heartbeat:Filesystem):    Stopped
         hana_shared1       (ocf::heartbeat:Filesystem):    Stopped

     Resource Group: hanadb2_nfs
         hana_data2 (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_log2  (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_shared2       (ocf::heartbeat:Filesystem):    Started hanadb2

     hana_nfs1_active       (ocf::pacemaker:attribute):     Stopped
     hana_nfs2_active       (ocf::pacemaker:attribute):     Started hanadb2

     Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
         Started: [ hanadb2 ]
         Stopped: [ hanadb1 ]

     Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
         Masters: [ hanadb2 ]
         Stopped: [ hanadb1 ]

     Resource Group: g_ip_HN1_03
         nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hanadb2
         vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hanadb2
    ```

   Рекомендуется тщательно протестировать конфигурацию кластера SAP HANA, выполняя тесты, описанные в разделе [установка SAP HANA репликацию системы на RHEL](./sap-hana-high-availability-rhel.md#test-the-cluster-setup).

## <a name="next-steps"></a>Дальнейшие действия

* [Планирование и реализация виртуальных машин Azure для SAP][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
* [Тома NFS версии 4.1 в Azure NetApp Files для SAP HANA](./hana-vm-operations-netapp.md)