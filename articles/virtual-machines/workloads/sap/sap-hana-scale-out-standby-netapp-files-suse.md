---
title: Горизонтальное увеличение масштаба SAP HANA с использованием резервного узла и Azure NetApp Files в SLES | Документация Майкрософт
description: Развертывание горизонтально масштабируемой системы SAP HANA с резервным узлом на виртуальных машинах Azure с помощью Azure NetApp Files в SUSE Linux Enterprise Server.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/06/2021
ms.author: radeltch
ms.openlocfilehash: 37a3923abf32259499d876074ceda29c27e87d3c
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106551974"
---
# <a name="deploy-a-sap-hana-scale-out-system-with-standby-node-on-azure-vms-by-using-azure-netapp-files-on-suse-linux-enterprise-server"></a>Развертывание горизонтально масштабируемой системы SAP HANA с резервным узлом на виртуальных машинах Azure с помощью Azure NetApp Files в SUSE Linux Enterprise Server 

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

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[suse-ha-guide]:https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md


В этой статье описывается, как развернуть высокодоступную систему SAP HANA в конфигурации с горизонтальным увеличением масштаба и резервированием на виртуальных машинах Azure с помощью [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) для томов общего хранилища.  

В примерах конфигураций, командах установки и т. д. экземпляр системы HANA имеет значение **03**, а ее идентификатор — **HN1**. Примеры основаны на HANA 2.0 SP4 и SUSE Linux Enterprise Server для SAP 12 SP4. 

Прежде чем начать, ознакомьтесь со следующими примечаниями и документацией по SAP.

* [Документация по Azure NetApp Files][anf-azure-doc] 
* В примечании к SAP [1928533] включены:  
  * список размеров виртуальных машин Azure, поддерживаемых для развертывания ПО SAP;
  * важные сведения о доступных ресурсах для каждого размера виртуальной машины Azure;
  * сведения о поддерживаемом программном обеспечении SAP и сочетаниях операционных систем и баз данных;
  * сведения о требуемой версии ядра SAP для Windows и Linux в Microsoft Azure
* В примечании к SAP [2015553] описываются предварительные требования к SAP при развертывании программного обеспечения SAP в Azure
* В примечании к SAP [2205917] содержатся рекомендуемые параметры ОС для SUSE Linux Enterprise Server for SAP Applications
* В примечании к SAP [1944799] содержатся рекомендации по SAP HANA для SUSE Linux Enterprise Server for SAP Applications
* В примечании к SAP [2178632] содержатся подробные сведения обо всех доступных метриках мониторинга для SAP в Azure
* В примечании к SAP [2191498] содержатся сведения о необходимой версии агента SAP Host Agent для Linux в Azure
* В примечании к SAP [2243692] содержатся сведения о лицензировании SAP в Linux в Azure
* В примечании к SAP [1984787] содержатся общие сведения о SUSE Linux Enterprise Server 12
* В примечании к SAP [1999351] содержатся дополнительные сведения об устранении неполадок, связанных с расширением для расширенного мониторинга Azure для SAP
* В примечание к SAP [1900823] содержатся сведения о требованиях к хранилищу SAP HANA
* [Вики-сайт сообщества SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) содержит все необходимые примечания к SAP для Linux
* [SAP NetWeaver на виртуальных машинах Linux. Руководство по планированию и внедрению][planning-guide]
* [Развертывание программного обеспечения SAP на виртуальных машинах Linux в Azure][deployment-guide]
* [Общие сведения о развертывании СУБД на виртуальных машинах Azure для рабочих нагрузок SAP][dbms-guide]
* [Рекомендации профессионалов по использованию SUSE SAP][suse-ha-guide]: содержит все необходимые сведения для настройки высокой доступности и репликации системы SAP HANA локально (для использования в качестве общего базового показателя; они предоставляют намного более подробную информацию)
* [Заметки о выпуске расширения SUSE высокого уровня доступности 12 SP3][suse-ha-12sp3-relnotes]
* [Приложения NetApp SAP в Microsoft Azure. Использование Azure NetApp Files][anf-sap-applications-azure]
* [Тома NFS версии 4.1 в Azure NetApp Files для SAP HANA](./hana-vm-operations-netapp.md)

## <a name="overview"></a>Обзор

Одним из способов достижения высокого уровня доступности HANA является настройка автоматической отработки отказа узла. Чтобы настроить автоматическую отработку отказа узла, добавьте одну или несколько виртуальных машин в систему HANA и настройте их в качестве резервных узлов. При сбое активного узла происходит автоматический переход на резервный узел. В представленной конфигурации с виртуальными машинами Azure вы достигаете автоматической отработки отказа с помощью [NFS на Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).  

> [!NOTE]
> Для резервного узла требуется доступ ко всем томам базы данных. Тома HANA должны быть подключены как тома NFSv4. Улучшенный механизм блокировки на основе аренды файлов в протоколе NFSv4 используется для ограждения `I/O`. 

> [!IMPORTANT]
> Чтобы создать поддерживаемую конфигурацию, необходимо развернуть тома HANA и журнальные тома системе NFSv4.1 и подключить их с помощью протокола NFSv4.1. Конфигурация автоматической отработки отказа узла HANA с резервным узлом не поддерживается для NFSv3.

![Общие сведения о высоком уровне доступности SAP NetWeaver](./media/high-availability-guide-suse-anf/sap-hana-scale-out-standby-netapp-files-suse.png)

На предыдущей схеме, которая соответствует сетевым рекомендациям для SAP HANA, три подсети представлены в одной виртуальной сети Azure: 
* для связи с клиентами;
* для связи с системой хранения данных;
* для обмена данными между узлами SAP HANA.

Тома NetApp для Azure находятся в отдельной подсети, [делегированной Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

Для этого примера конфигурации подсетями являются следующие адреса:  

  - `client` 10.23.0.0/24  
  - `storage` 10.23.2.0/24  
  - `hana` 10.23.3.0/24  
  - `anf` 10.23.1.0/26  

## <a name="set-up-the-azure-netapp-files-infrastructure"></a>Настройка инфраструктуры Azure NetApp Files 

Прежде чем продолжить настройку инфраструктуры Azure NetApp Files, ознакомьтесь с документацией по [Azure NetApp Files][anf-azure-doc]. 

Служба Azure NetApp Files доступна в нескольких [регионах Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Проверьте, предлагает ли выбранный регион Azure службу Azure NetApp Files.  

Перейдите по приведенной ниже ссылке, чтобы узнать о доступности службы Azure NetApp Files по регионам Azure: [Доступность Azure NetApp Files по региону Azure][anf-avail-matrix].  

Перед развертыванием Azure NetApp Files запросите подключение к Azure NetApp Files, следуя инструкциям в статье [Регистрация в службе Azure NetApp Files][anf-register]. 

### <a name="deploy-azure-netapp-files-resources"></a>Развертывание ресурсов Azure NetApp Files  

В следующих инструкциях предполагается, что вы уже развернули [виртуальную сеть Azure](../../../virtual-network/virtual-networks-overview.md). Ресурсы Azure NetApp Files и виртуальные машины, к которым будут подключаться ресурсы Azure NetApp Files, должны быть развернуты в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure.  

1. Если вы еще не сделали этого, запросите [подключение к Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).  

2. Создайте учетную запись NetApp в выбранном регионе Azure, следуя инструкциям в разделе [Создание учетной записи NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).  

3. Настройте пул емкости Azure NetApp Files, следуя [инструкциям по настройке пула емкости Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).  

   В архитектуре HANA, представленной в этой статье, используется один пул емкости Azure NetApp Files на уровне *Ultra Service*. Для рабочих нагрузок HANA в Azure рекомендуется использовать [уровень обслуживания](../../../azure-netapp-files/azure-netapp-files-service-levels.md)Azure NetApp Files *Ultra* или *Premium*.  

4. Делегируйте подсеть в Azure NetApp Files согласно [инструкциям по делегированию подсети в Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

5. Разверните тома Azure NetApp Files, следуя [инструкциям по созданию тома NFS для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  

   При развертывании томов обязательно выберите версию **NFSv4.1**. В настоящее время для доступа к протоколу NFSv4.1 требуется добавить его в список разрешений. Разверните тома в выделенной [ подсети](/rest/api/virtualnetwork/subnets) Azure NetApp Files. IP-адреса томов Azure NetApp Files назначаются автоматически. 
   
   Учтите, что ресурсы Azure NetApp Files и виртуальные машины Azure должны находиться в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure. Например, **HN1**-Data-Mnt00001, **HN1**-log-mnt00001 и т. д. — это имена томов, а NFS://10.23.1.5/**HN1**-Data-mnt00001, NFS://10.23.1.4/**HN1**-log-mnt00001 и т. д. — это пути к файлам на томах Azure NetApp Files.  

   * volume **HN1**-data-mnt00001 (nfs://10.23.1.5/**HN1**-data-mnt00001)
   * volume **HN1**-data-mnt00002 (nfs://10.23.1.6/**HN1**-data-mnt00002)
   * volume **HN1**-log-mnt00001 (nfs://10.23.1.4/**HN1**-log-mnt00001)
   * volume **HN1**-log-mnt00002 (nfs://10.23.1.6/**HN1**-log-mnt00002)
   * volume **HN1**-shared (nfs://10.23.1.4/**HN1**-shared)
   
   В этом примере мы использовали отдельный том Azure NetApp Files для каждого тома HANA и журнального тома. Для более производительной конфигурации, оптимизированной для небольших или непродуктивных систем, можно разместить все подключения к данным и все журналы на одном томе.  

### <a name="important-considerations"></a>Важные замечания

При создании Azure NetApp Files для архитектуры с высоким уровнем доступности SAP NetWeaver в SUSE учитывайте следующие важные моменты.

- Минимальный размер пула емкости равен 4 ТиБ.  
- Минимальный размер тома составляет 100 ГиБ.
- Azure NetApp Files и все виртуальные машины, на которых будут монтироваться тома Azure NetApp Files, должны быть развернуты в одной виртуальной сети Azure или в [одноранговых виртуальных сетях](../../../virtual-network/virtual-network-peering-overview.md) в одном регионе.  
- В выбранной виртуальной сети должна быть подсеть, делегированная службе Azure NetApp Files.
- Пропускная способность тома Azure NetApp является функцией квоты тома и уровня обслуживания, как описано в статье [Уровни обслуживания для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). При определении размера томов HANA в Azure NetApp убедитесь, что полученная пропускная способность соответствует требованиям к системе HANA.  
- Azure NetApp Files предлагает [политику экспорта](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md), которая позволяет управлять разрешенными клиентами, типом доступа (чтение и запись, доступ только для чтения и т. д.). 
- Azure NetApp Files пока еще не поддерживает зоны. В настоящее время эта функция развернута не во всех зонах доступности региона Azure. Учитывайте возможную задержку в некоторых регионах Azure.  
-  

> [!IMPORTANT]
> Для рабочих нагрузок SAP HANA низкий показатель задержки чрезвычайно важен. Пообщайтесь с представителями корпорации Майкрософт, чтобы убедиться, что виртуальные машины и тома Azure NetApp Files развернуты близко друг от друга.  

### <a name="sizing-for-hana-database-on-azure-netapp-files"></a>Выбор размера базы данных HANA в Azure NetApp Files

Пропускная способность тома Azure NetApp Files является функцией размера тома и уровня обслуживания, как описано в статье [Уровни обслуживания для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). 

При проектировании инфраструктуры для SAP в Azure следует учитывать минимальные требования к пропускной способности хранилища от SAP, которые соотносятся с минимальными характеристиками пропускной способности следующего.

- Включение чтения и записи в/Hana/log с 250 МБ в секунду (МБ/с) с размером операций ввода-вывода в 1 МБ.  
- Включение действия чтения с пропускной способностью не менее 400 МБ/с для /hana/data для размеров операций ввода-вывода 16 МБ и 64 МБ.  
- Включение действия записи с пропускной способностью не менее 250 МБ/с для /hana/data для размеров операций ввода-вывода 16 МБ и 64 МБ. 

Ниже представлены [ограничения пропускной способности Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md) на 1 ТиБ квоты тома.
- Уровень хранилища "Premium" — 64 МиБ/с  
- Уровень хранилища "Ультра" — 128 МиБ/с  

Для удовлетворения минимальных требований SAP к пропускной способности для данных и файлов журналов, а также в соответствии с рекомендациями для /hana/shared рекомендованные размеры будут обозначены следующим образом.

| Громкость | Размер:<br>Хранилище класса "Premium" | Размер:<br>Хранилище класса "Ультра" | Поддерживаемый протокол NFS |
| --- | --- | --- | --- |
| /hana/log/ | 4 ТиБ | 2 ТиБ | Версия 4.1 |
| hana/data; | 6.3 ТиБ | 3.2 ТиБ | Версия 4.1 |
| /hana/shared | Максимум (512 ГБ, 1xRAM) на 4 рабочих узла | Максимум (512 ГБ, 1xRAM) на 4 рабочих узла | Версия 3 или версия 4.1 |

Конфигурация SAP HANA для макета, представленного в этой статье с использованием Azure NetApp Files уровня хранения Ultra, будет выглядеть следующим образом.

| Громкость | Размер:<br>Хранилище класса "Ультра" | Поддерживаемый протокол NFS |
| --- | --- | --- |
| /hana/log/mnt00001 | 2 ТиБ | Версия 4.1 |
| /hana/log/mnt00002 | 2 ТиБ | Версия 4.1 |
| /hana/data/mnt00001 | 3.2 ТиБ | Версия 4.1 |
| /hana/data/mnt00002 | 3.2 ТиБ | Версия 4.1 |
| /hana/shared | 2 ТиБ | Версия 3 или версия 4.1 |

> [!NOTE]
> Приведенные рекомендации по размерам для Azure NetApp Files нацелены на минимальные требования, которые рекомендуются SAP в отношении своих поставщиков инфраструктуры. В реальных сценариях развертывания и рабочих нагрузок такие значения могут оказаться недостаточными. Поэтому примите эти рекомендации в качестве отправной точки и адаптируйтесь в соответствии с требованиями своей рабочей нагрузки.  

> [!TIP]
> Вы можете динамически изменять размер томов Azure NetApp Files без необходимости *отключения* тома, прекращать работу виртуальных машин или прерывать работу SAP HANA. Это позволяет создать гибкий подход, который поможет справиться как с ожидаемыми, так и с непредвиденными обстоятельствами, связанными с пропускной способностью приложения.

## <a name="deploy-linux-virtual-machines-via-the-azure-portal"></a>Развертывание виртуальных машин Linux с помощью портала Azure

Сначала необходимо создать тома Azure NetApp Files. Затем сделайте следующее.
1. Создайте [подсети виртуальной сети Azure](../../../virtual-network/virtual-network-manage-subnet.md) в [виртуальной сети Azure](../../../virtual-network/virtual-networks-overview.md). 
1. Разверните виртуальные машины. 
1. Создайте дополнительные сетевые интерфейсы и подключите сетевые интерфейсы к соответствующим виртуальным машинам.  

   Каждая виртуальная машина имеет три сетевых интерфейса, которые соответствуют трем подсетям виртуальной сети Azure (`client`, `storage` и `hana`). 

   Дополнительные сведения см. в статье [Создание виртуальной машины Linux в Azure с несколькими сетевыми картами](../../linux/multiple-nics.md).  

> [!IMPORTANT]
> Для рабочих нагрузок SAP HANA низкий показатель задержки чрезвычайно важен. Чтобы добиться низких значений задержки, пообщайтесь с представителями корпорации Майкрософт. Так виртуальные машины и тома Azure NetApp Files развернуты близко друг от друга. При [подключении новой системы SAP HANA](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxjSlHBUxkJBjmARn57skvdUQlJaV0ZBOE1PUkhOVk40WjZZQVJXRzI2RC4u), которая использует SAP HANA Azure NetApp Files, отправьте необходимые сведения. 
 
В следующих инструкциях предполагается, что вы уже создали группу ресурсов, виртуальную сеть Azure и три подсети виртуальной сети Azure: `client`, `storage` и `hana`. При развертывании виртуальных машин выберите подсеть клиента, чтобы сетевой интерфейс клиента был основным интерфейсом на виртуальных машинах. Кроме того, необходимо настроить явный маршрут к делегированной подсети Azure NetApp Files через шлюз подсети хранилища. 

> [!IMPORTANT]
> Убедитесь, что выбранная операционная система была сертифицирована для использования SAP HANA на конкретных типах виртуальных машин. Список сертифицированных для SAP HANA типов виртуальных машин и выпусков ОС см. в разделе [SAP HANA сертифицированные платформы IAAS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Чтобы просмотреть подробные сведения о поддерживаемых SAP HANA на определенных типах виртуальных машин выпусках ОС, щелкните необходимый тип виртуальной машины.  

1. Создайте группу доступности для SAP HANA. Убедитесь, что задан максимальный домен обновления.  

2. Создайте три виртуальные машины (**hanadb1**, **hanadb2** и **hanadb3**), выполнив следующие действия.  

   a. Используйте образ SLES4SAP из коллекции Azure, который поддерживается для SAP HANA. В этом примере мы использовали образ SLES4SAP 12 с пакетом обновления 4 (SP4).  

   b. Выберите ранее созданную группу доступности для SAP HANA.  

   c. Выберите подсеть виртуальной сети Azure для клиента. Выберите [Ускоренная сеть](../../../virtual-network/create-vm-accelerated-networking-cli.md).  

   При развертывании виртуальных машин имя сетевого интерфейса создается автоматически. В этих инструкциях мы будем называть автоматически создаваемые сетевые интерфейсы, подключенные к подсети виртуальной сети Azure клиента, такие как **hanadb1-Client**, **hanadb2-Client** и **hanadb3-Client**. 

3. Создайте три сетевых интерфейса, по одному для каждой виртуальной машины, для `storage` подсети виртуальной сети (в этом примере — **hanadb1-storage**, **hanadb2-storage** и **hanadb3-storage**).  

4. Создайте три сетевых интерфейса, по одному для каждой виртуальной машины, для `hana` подсети виртуальной сети (в этом примере — **hanadb1-hana**, **hanadb2-hana** и **hanadb3-hana**).  

5. Подключите вновь созданные виртуальные сетевые интерфейсы к соответствующим виртуальным машинам, выполнив следующие действия.  

    a. Перейдите к списку виртуальных машин на [портале Azure](https://portal.azure.com/#home).  

    b. В меню слева выберите **Виртуальные машины**. Выполните фильтрацию по имени виртуальной машины (например, **hanadb1**), а затем выберите виртуальную машину.  

    c. В области **Обзор** выберите пункт **Прервать**, чтобы освободить виртуальную машину.  

    d. Выберите **Сеть**, а затем подключите сетевой интерфейс. В раскрывающемся списке **Подключить сетевой интерфейс** выберите уже созданные сетевые интерфейсы для подсетей `storage` и `hana`.  
    
    д) Щелкните **Сохранить**. 
 
    е) Повторите шаги с b по д для оставшихся виртуальных машин (в нашем примере это **hanadb2** и **hanadb3**).
 
    ж. Пока оставьте виртуальные машины в остановленном состоянии. Далее мы разберем функцию [ускорения сети](../../../virtual-network/create-vm-accelerated-networking-cli.md) для всех вновь подключенных сетевых интерфейсов.  

6. Включите ускоренную сеть для дополнительных сетевых интерфейсов для подсетей `storage` и `hana`, выполнив следующие действия.  

    a. Открытие [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) на [портале Azure](https://portal.azure.com/#home).  

    b. Выполните следующие команды, чтобы включить ускоренную сеть для дополнительных сетевых интерфейсов, подключенных к подсетям `storage` и `hana`.  

    <pre><code>
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb1-storage</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb2-storage</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb3-storage</b> --accelerated-networking true
    
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb1-hana</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb2-hana</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb3-hana</b> --accelerated-networking true

    </code></pre>

7. Запустите виртуальные машины, выполнив следующие действия.  

    a. В меню слева выберите **Виртуальные машины**. Выполните фильтрацию по имени виртуальной машины (например, **hanadb1**), а затем выберите ее.  

    b. В области **Обзор** выберите **Запуск**.  

## <a name="operating-system-configuration-and-preparation"></a>Настройка и подготовка операционной системы

Инструкции в следующих разделах начинаются с одной из следующих версий.
* **[A]** : применимо ко всем узлам
* **[1]** : применимо только к узлу 1
* **[2]** : применимо только к узлу 2
* **[3]** : применимо только к узлу 3

Настройте и подготовьте ОС, выполнив следующие действия.

1. **[A]** Сохраните файлы узлов на виртуальных машинах. Включите записи для всех подсетей. Для этого примера были добавлены следующие записи `/etc/hosts`.  

    <pre><code>
    # Storage
    10.23.2.4   hanadb1-storage
    10.23.2.5   hanadb2-storage
    10.23.2.6   hanadb3-storage
    # Client
    10.23.0.5   hanadb1
    10.23.0.6   hanadb2
    10.23.0.7   hanadb3
    # Hana
    10.23.3.4   hanadb1-hana
    10.23.3.5   hanadb2-hana
    10.23.3.6   hanadb3-hana
    </code></pre>

2. **[A]** Измените параметры конфигурации DHCP и Cloud для сетевого интерфейса для хранения, чтобы избежать непреднамеренного изменения имени узла.  

    В следующих инструкциях предполагается, что сетевой интерфейс хранилища имеет значение `eth1`. 

    <pre><code>
    vi /etc/sysconfig/network/dhcp
    # Change the following DHCP setting to "no"
    DHCLIENT_SET_HOSTNAME="no"
    vi /etc/sysconfig/network/ifcfg-<b>eth1</b>
    # Edit ifcfg-eth1 
    #Change CLOUD_NETCONFIG_MANAGE='yes' to "no"
    CLOUD_NETCONFIG_MANAGE='no'
    </code></pre>

2. **[A]** Добавьте сетевой маршрут, чтобы связь с Azure NetApp Files проходила через сетевой интерфейс хранилища.  

    В следующих инструкциях предполагается, что сетевой интерфейс хранилища имеет значение `eth1`.  

    <pre><code>
    vi /etc/sysconfig/network/ifroute-<b>eth1</b>
    # Add the following routes 
    # RouterIPforStorageNetwork - - -
    # ANFNetwork/cidr RouterIPforStorageNetwork - -
    <b>10.23.2.1</b> - - -
    <b>10.23.1.0/26</b> <b>10.23.2.1</b> - -
    </code></pre>

    Перезагрузите виртуальную машину, чтобы активировать изменения.  

3. **[A]** Подготовьте ОС для запуска SAP HANA в NETAPP Systems с NFS, как описано в разделе [Приложения NetApp SAP в Microsoft Azure с использованием Azure NetApp Files][anf-sap-applications-azure]. Создайте файл конфигурации */etc/sysctl.d/netapp-hana.conf* для параметров конфигурации NetApp.  

    <pre><code>
    vi /etc/sysctl.d/netapp-hana.conf
    # Add the following entries in the configuration file
    net.core.rmem_max = 16777216
    net.core.wmem_max = 16777216
    net.core.rmem_default = 16777216
    net.core.wmem_default = 16777216
    net.core.optmem_max = 16777216
    net.ipv4.tcp_rmem = 65536 16777216 16777216
    net.ipv4.tcp_wmem = 65536 16777216 16777216
    net.core.netdev_max_backlog = 300000
    net.ipv4.tcp_slow_start_after_idle=0
    net.ipv4.tcp_no_metrics_save = 1
    net.ipv4.tcp_moderate_rcvbuf = 1
    net.ipv4.tcp_window_scaling = 1
    net.ipv4.tcp_timestamps = 1
    net.ipv4.tcp_sack = 1
    </code></pre>

4. **[A]** Создайте файл конфигурации */etc/sysctl.d/ms-az.conf* с параметрами конфигурации Microsoft для Azure.  

    <pre><code>
    vi /etc/sysctl.d/ms-az.conf
    # Add the following entries in the configuration file
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv4.tcp_max_syn_backlog = 16348
    net.ipv4.conf.all.rp_filter = 0
    sunrpc.tcp_slot_table_entries = 128
    vm.swappiness=10
    </code></pre>

> [!TIP]
> Не следует явно задавать net.ipv4.ip_local_port_range и net.ipv4.ip_local_reserved_ports в файлах конфигурации sysctl, чтобы разрешить агенту узла SAP управлять диапазонами портов. Дополнительные сведения см. в примечании к SAP [2382421](https://launchpad.support.sap.com/#/notes/2382421).  

4. **[A]** Настройте параметры sunrpc, как рекомендуется в разделе [Приложения SAP NetApp на Microsoft Azure с использованием Azure NetApp Files][anf-sap-applications-azure].  

    <pre><code>
    vi /etc/modprobe.d/sunrpc.conf
    # Insert the following line
    options sunrpc tcp_max_slot_table_entries=128
    </code></pre>

## <a name="mount-the-azure-netapp-files-volumes"></a>Монтируйте тома Azure NetApp Files

1. **[A]** Создайте точки подключения для томов базы данных HANA.  

    <pre><code>
    mkdir -p /hana/data/<b>HN1</b>/mnt00001
    mkdir -p /hana/data/<b>HN1</b>/mnt00002
    mkdir -p /hana/log/<b>HN1</b>/mnt00001
    mkdir -p /hana/log/<b>HN1</b>/mnt00002
    mkdir -p /hana/shared
    mkdir -p /usr/sap/<b>HN1</b>
    </code></pre>

2. **[1]** Создайте каталоги узлов для/usr/sap на **HN1**-shared.  

    <pre><code>
    # Create a temporary directory to mount <b>HN1</b>-shared
    mkdir /mnt/tmp
    # if using NFSv3 for this volume, mount with the following command
    mount <b>10.23.1.4</b>:/<b>HN1</b>-shared /mnt/tmp
    # if using NFSv4.1 for this volume, mount with the following command
    mount -t nfs -o sec=sys,vers=4.1 <b>10.23.1.4</b>:/<b>HN1</b>-shared /mnt/tmp
    cd /mnt/tmp
    mkdir shared usr-sap-<b>hanadb1</b> usr-sap-<b>hanadb2</b> usr-sap-<b>hanadb3</b>
    # unmount /hana/shared
    cd
    umount /mnt/tmp
    </code></pre>

3. **[A]** Проверьте параметры домена NFS. Убедитесь, что домен настроен в качестве домена Azure NetApp Files по умолчанию, т. е. **`defaultv4iddomain.com`** , а для сопоставления установлено значение **Никто**.  

    > [!IMPORTANT]
    > Обязательно укажите домен NFS в `/etc/idmapd.conf` виртуальной машине в соответствии с конфигурацией домена по умолчанию в Azure NetApp Files: **`defaultv4iddomain.com`** . В случае несоответствия между конфигурацией домена на клиенте NFS (т. е. виртуальной машине) и NFS-сервере (т. е. конфигурацией Azure NetApp) разрешения для файлов на томах NetApp в Azure, подключенных к виртуальным машинам, будут отображаться как `nobody`.  

    <pre><code>
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Verbosity = 0
    Pipefs-Directory = /var/lib/nfs/rpc_pipefs
    Domain = <b>defaultv4iddomain.com</b>
    [Mapping]
    Nobody-User = <b>nobody</b>
    Nobody-Group = <b>nobody</b>
    </code></pre>

4. **[A]** Проверьте параметр `nfs4_disable_idmapping`. Ему должно быть задано значение **Y**. Чтобы создать структуру каталогов, в которой находится параметр `nfs4_disable_idmapping`, выполните команду mount. Вы не сможете вручную создать каталог в /sys/modules, так как доступ зарезервирован для ядра или драйверов.  

    <pre><code>
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    # If you need to set nfs4_disable_idmapping to Y
    mkdir /mnt/tmp
    mount 10.23.1.4:/HN1-shared /mnt/tmp
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    </code></pre>

5. **[A]** Создайте группу SAP HANA и пользователя вручную. Для идентификаторов групп sapsys и пользователя **hn1** adm должны быть заданы одинаковые идентификаторы, которые предоставляются во время подключения. (В этом примере идентификаторы имеют значение **1001**.) Если идентификаторы заданы неправильно, доступ к томам будет невозможен. Идентификаторы групп sapsys и учетных записей пользователей **hn1** adm и sapadm должны быть одинаковыми на всех виртуальных машинах.  

    <pre><code>
    # Create user group 
    sudo groupadd -g 1001 sapsys
    # Create  users 
    sudo useradd <b>hn1</b>adm -u 1001 -g 1001 -d /usr/sap/<b>HN1</b>/home -c "SAP HANA Database System" -s /bin/sh
    sudo useradd sapadm -u 1002 -g 1001 -d /home/sapadm -c "SAP Local Administrator" -s /bin/sh
    # Set the password  for both user ids
    sudo passwd hn1adm
    sudo passwd sapadm
    </code></pre>

6. **[A]** Подключите общие тома Azure NetApp Files.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.5:/<b>HN1</b>-data-mnt00001 /hana/data/<b>HN1</b>/mnt00001  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.23.1.6:/<b>HN1</b>-data-mnt00002 /hana/data/<b>HN1</b>/mnt00002  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.23.1.4:/<b>HN1</b>-log-mnt00001 /hana/log/<b>HN1</b>/mnt00001  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.23.1.6:/<b>HN1</b>-log-mnt00002 /hana/log/HN1/mnt00002  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.23.1.4:/<b>HN1</b>-shared/shared /hana/shared  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount all volumes
    sudo mount -a 
    </code></pre>

7. **[1]** Подключите тома конкретного узла к **hanadb1**.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb1</b> /usr/sap/<b>HN1</b>  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    </code></pre>

8. **[2]** Подключите тома конкретного узла к **hanadb2**.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb2</b> /usr/sap/<b>HN1</b>  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    </code></pre>

9. **[3]** Подключите тома конкретного узла к **hanadb3**.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb3</b> /usr/sap/<b>HN1</b>  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    </code></pre>

10. **[A]** Убедитесь, что все тома HANA подключены с помощью протокола NFS версии **NFSv4**.  

    <pre><code>
    sudo nfsstat -m
    # Verify that flag vers is set to <b>4.1</b> 
    # Example from <b>hanadb1</b>
    /hana/data/<b>HN1</b>/mnt00001 from 10.23.1.5:/<b>HN1</b>-data-mnt00001
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.5
    /hana/log/<b>HN1</b>/mnt00002 from 10.23.1.6:/<b>HN1</b>-log-mnt00002
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.6
    /hana/data/<b>HN1</b>/mnt00002 from 10.23.1.6:/<b>HN1</b>-data-mnt00002
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.6
    /hana/log/<b>HN1</b>/mnt00001 from 10.23.1.4:/<b>HN1</b>-log-mnt00001
    Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.4
    /usr/sap/<b>HN1</b> from 10.23.1.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb1</b>
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.4
    /hana/shared from 10.23.1.4:/<b>HN1</b>-shared/shared
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.2.4,local_lock=none,addr=10.23.1.4
    </code></pre>

## <a name="installation"></a>Установка  

В этом примере для развертывания SAP HANA в конфигурации с горизонтальным увеличением масштаба с резервным узлом Azure мы использовали HANA 2.0 SP4.  

### <a name="prepare-for-hana-installation"></a>Подготовка к установке SAP HANA

1. **[A]** Перед установкой HANA задайте корневой пароль. После завершения установки можно отключить корневой пароль. Выполните как `root` команду`passwd`.  

2. **[1]** Убедитесь, что вы можете войти с помощью SSH в **hanadb2** и **hanadb3** без запроса пароля.  

    <pre><code>
    ssh root@<b>hanadb2</b>
    ssh root@<b>hanadb3</b>
    </code></pre>

3. **[A]** Установите дополнительные пакеты, необходимые для HANA 2.0 SP4. Дополнительные сведения см. в примечании к SAP [2593824](https://launchpad.support.sap.com/#/notes/2593824). 

    <pre><code>
    sudo zypper install libgcc_s1 libstdc++6 libatomic1 
    </code></pre>

4. **[2], [3]** Измените владельца каталогов SAP HANA `data` и `log` на **hn1** adm.   

    <pre><code>
    # Execute as root
    sudo chown hn1adm:sapsys /hana/data/<b>HN1</b>
    sudo chown hn1adm:sapsys /hana/log/<b>HN1</b>
    </code></pre>

### <a name="hana-installation"></a>Установка HANA

1. **[1]** Установите SAP Hana, следуя инструкциям из руководства по [установке и обновлению SAP HANA 2.0](https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.04/en-US/7eb0167eb35e4e2885415205b8383584.html). В этом примере мы устанавливаем SAP HANA с горизонтальным увеличением масштаба с главным, одним рабочим и одним резервным узлом.  

   a. Запустите программу **hdblcm** из каталога с установочным программным обеспечением HANA. Используйте параметр `internal_network` и передайте адресное пространство для подсети, используемое для обмена данными между узлами HANA.  

    <pre><code>
    ./hdblcm --internal_network=10.23.3.0/24
    </code></pre>

   b. Введите следующие значения на запрос в командной строке.

     * На запрос **Выберите действие**: введите **1** (установка)
     * На запрос **Выберите дополнительные компоненты для установки**: введите **2, 3**
     * Для пути установки: нажмите клавишу ВВОД (по умолчанию — /hana/shared).
     * На запрос **Имя локального узла**: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Желаете добавить узлы в систему?** введите **y**
     * На запрос **Добавить имена узлов с разделителями-запятыми**: введите **hanadb2, hanadb3**
     * На запрос **Имя привилегированного пользователя** [root]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Пароль привилегированного пользователя**: введите пароль привилегированного пользователя
     * Для ролей узла hanadb2: введите **1** (рабочий узел)
     * На запрос **Группа отработки отказа узла** для узла hanadb2 [default]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Номер раздела хранилища** для узла hanadb2 [<<assign automatically>>]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Рабочая группа** для узла hanadb2 [default]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Выберите роли** для узла hanadb3: введите **2** (режим ожидания)
     * На запрос **Группа отработки отказа узла** для узла hanadb3 [default]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Рабочая группа** для узла hanadb3 [default]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Идентификатор системы SAP HANA** : введите **HN1**
     * На запрос **Номер экземпляра** [00]: введите **03**
     * На запрос **Рабочая группа локального узла** [default]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Выберите использование системы/введите индекс [4]** : введите **4** (пользовательское значение)
     * На запрос **Расположение томов данных** [/hana/hata/HN1]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Расположение журнальных томов** [/hana/hata/HN1]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Ограничить максимальное выделение памяти?** [n]: введите **n**
     * На запрос **Имя узла сертификата для узла hanadb1** [hanadb1]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Имя узла сертификата для узла hanadb2** [hanadb2]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Имя узла сертификата для узла hanadb3** [hanadb3]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * На запрос **Пароль для системного администратора (hn1adm)** : введите пароль
     * На запрос **Пароль пользователя системной базы данных**: введите пароль системы
     * На запрос **Подтвердите пароль пользователя системной базы данных**: введите пароль системы
     * На запрос **Перезапустить систему после перезагрузки компьютера?** [n]: введите **n** 
     * На запрос **Продолжить (y/n)?** : проверьте сводку и, если все выглядит хорошо, введите **y**


2. **[1]** Проверьте файл global.ini  

   Откройте файл global.ini и убедитесь, что конфигурация внутреннего SAP HANA взаимодействия между узлами выполнена. Проверьте раздел **communication**. В нем должно иметься адресное пространство для подсети `hana`, а `listeninterface` должно иметь значение `.internal`. Проверьте раздел **internal_hostname_resolution**. Он должен иметь IP-адреса виртуальных машин HANA, принадлежащих подсети `hana`.  

   <pre><code>
    sudo cat /usr/sap/<b>HN1</b>/SYS/global/hdb/custom/config/global.ini
    # Example 
    #global.ini last modified 2019-09-10 00:12:45.192808 by hdbnameserve
    [communication]
    internal_network = <b>10.23.3/24</b>
    listeninterface = .internal
    [internal_hostname_resolution]
    <b>10.23.3.4</b> = <b>hanadb1</b>
    <b>10.23.3.5</b> = <b>hanadb2</b>
    <b>10.23.3.6</b> = <b>hanadb3</b>
   </code></pre>

3. **[1]** Добавьте сопоставление узлов, чтобы обеспечить использование IP-адресов клиента для связи между клиентами. Добавьте раздел `public_host_resolution` и добавьте соответствующие IP-адреса из подсети клиента.  

   <pre><code>
    sudo vi /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
    #Add the section
    [public_hostname_resolution]
    map_<b>hanadb1</b> = <b>10.23.0.5</b>
    map_<b>hanadb2</b> = <b>10.23.0.6</b>
    map_<b>hanadb3</b> = <b>10.23.0.7</b>
   </code></pre>

4. **[1]** Перезапустите SAP HANA, чтобы активировать изменения.  

   <pre><code>
    sudo -u <b>hn1</b>adm /usr/sap/hostctrl/exe/sapcontrol -nr <b>03</b> -function StopSystem HDB
    sudo -u <b>hn1</b>adm /usr/sap/hostctrl/exe/sapcontrol -nr <b>03</b> -function StartSystem HDB
   </code></pre>

5. **[1]** Убедитесь, что клиентский интерфейс будет использовать IP-адреса из подсети `client` для обмена данными.  

   <pre><code>
    sudo -u hn1adm /usr/sap/HN1/HDB03/exe/hdbsql -u SYSTEM -p "<b>password</b>" -i 03 -d SYSTEMDB 'select * from SYS.M_HOST_INFORMATION'|grep net_publicname
    # Expected result
    "<b>hanadb3</b>","net_publicname","<b>10.23.0.7</b>"
    "<b>hanadb2</b>","net_publicname","<b>10.23.0.6</b>"
    "<b>hanadb1</b>","net_publicname","<b>10.23.0.5</b>"
   </code></pre>

   Сведения о том, как проверить конфигурацию, см. в примечании к SAP [2183363 — настройка внутренней сети SAP HANA](https://launchpad.support.sap.com/#/notes/2183363).  

6. Чтобы оптимизировать SAP HANA под базовое хранилище Azure NetApp Files, задайте следующие параметры SAP HANA.

   - `max_parallel_io_requests` **128**
   - `async_read_submit` **on**
   - `async_write_submit_active` **on**
   - `async_write_submit_blocks` **all**

   Дополнительные сведения см. в разделе[Приложения NetApp SAP в Microsoft Azure с использованием Azure NetApp Files][anf-sap-applications-azure]. 

   Начиная с систем SAP HANA 2.0 параметры можно задать в `global.ini`. Дополнительные сведения см. в примечании к SAP [ 1999930](https://launchpad.support.sap.com/#/notes/1999930).  
   
   Для систем SAP HANA 1.0 версии SPS12 и более ранних эти параметры можно настроить во время установки, как описано в примечании к SAP [2267798](https://launchpad.support.sap.com/#/notes/2267798).  

7. Хранилище, используемое Azure NetApp Files, имеет ограничение размера файла, равное 16 терабайтам (ТБ). SAP HANA не осведомлена неявно об ограничениях размера хранилища и не будет создавать новый файл данных при достижении предельного размера файла в 16 ТБ. Так как SAP HANA попытается увеличивать размер файла за пределы 16 ТБ, эта попытка приведет к ошибкам и, в конце концов, к сбою сервера индексации. 

   > [!IMPORTANT]
   > Чтобы SAP HANA не пыталась увеличивать размер файлов данных за [пределы 16 ТБ](../../../azure-netapp-files/azure-netapp-files-resource-limits.md) подсистемы хранения, задайте следующие параметры в `global.ini`.  
   > - datavolume_striping = true
   > - datavolume_striping_size_gb = 15000 дополнительные сведения см. в примечании к SAP [2400005](https://launchpad.support.sap.com/#/notes/2400005).
   > Следует учитывать примечание к SAP [2631285](https://launchpad.support.sap.com/#/notes/2631285). 

## <a name="test-sap-hana-failover"></a>Тестовая отработка отказа SAP HANA 

> [!NOTE]
> В этой статье упоминаются термины *slave (ведомый)* и *master (главный)* , которые больше не используются Майкрософт. Когда эти термины будут удалены из программных продуктов, мы удалим их и из этой статьи.

1. Имитация сбоя узла на рабочем узле SAP HANA. Выполните следующие действия: 

   a. Прежде чем имитировать сбой узла, выполните следующие команды в качестве **hn1** adm, чтобы записать состояние среды.  

   <pre><code>
    # Check the landscape status
    python /usr/sap/<b>HN1</b>/HDB<b>03</b>/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | yes    | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
    # Check the instance status
    sapcontrol -nr <b>03</b>  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
   </code></pre>

   b. Для имитации сбоя узла выполните следующую команду в качестве привилегированного пользователя на рабочем узле, **hanadb2** в этом случае.  
   
   <pre><code>
    echo b > /proc/sysrq-trigger
   </code></pre>

   c. Наблюдайте за системой до завершения отработки отказа. После завершения отработки отказа запишите состояние, которое должно выглядеть следующим образом.  

    <pre><code>
    # Check the instance status
    sapcontrol -nr <b>03</b>  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GRAY
    # Check the landscape status
    /usr/sap/HN1/HDB03/exe/python_support> python landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | no     | info   |          |        |         2 |         0 | default  | default  | master 2   | slave      | worker      | standby     | worker  | standby | default | -       |
    | hanadb3 | yes    | info   |          |        |         0 |         2 | default  | default  | master 3   | slave      | standby     | slave       | standby | worker  | default | default |
   </code></pre>

   > [!IMPORTANT]
   > При возникновении аварии ядра не следует задерживать отработку отказа SAP HANA. Установите значение `kernel.panic` на 20 секунд на *всех* виртуальных машинах HANA. Настройка выполняется в `/etc/sysctl`. Перезагрузите виртуальные машины, чтобы активировать изменение. Если это изменение не выполняется, отработка отказа может занять 10 или более минут, если на узле возникла авария ядра.  

2. Удалите сервер имен, выполнив следующие действия.

   a. Перед тестированием проверьте состояние среды, выполнив следующие команды в качестве **hn1** adm:  

   <pre><code>
    #Landscape status 
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | no     | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GRAY
   </code></pre>

   b. Выполните следующие команды в качестве **hn1** adm на активном главном узле, **hanadb1** в этом случае.  

    <pre><code>
        hn1adm@hanadb1:/usr/sap/HN1/HDB03> HDB kill
    </code></pre>
    
    Резервный узел **hanadb3** войдет в работу в качестве главного узла. Ниже приведено состояние ресурса после завершения теста отработки отказа.  

    <pre><code>
        # Check the instance status
        sapcontrol -nr 03 -function GetSystemInstanceList
        GetSystemInstanceList
        OK
        hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
        hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
        hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GRAY
        hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
        # Check the landscape status
        python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
        | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
        |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
        |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
        | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
        | hanadb1 | no     | info   |          |        |         1 |         0 | default  | default  | master 1   | slave      | worker      | standby     | worker  | standby | default | -       |
        | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
        | hanadb3 | yes    | info   |          |        |         0 |         1 | default  | default  | master 3   | master     | standby     | master      | standby | worker  | default | default |
    </code></pre>

   c. Перезапустите экземпляр HANA на **hanadb1** (то есть на той же виртуальной машине, где был уничтожен сервер имен). Узел **hanadb1** повторно присоединится к среде и сохранит свою резервную роль.  

   <pre><code>
    hn1adm@hanadb1:/usr/sap/HN1/HDB03> HDB start
   </code></pre>

   После запуска SAP HANA в **hanadb1** должно быть указано следующее состояние.  

   <pre><code>
    # Check the instance status
    sapcontrol -nr 03 -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | info   |          |        |         1 |         0 | default  | default  | master 1   | slave      | worker      | standby     | worker  | standby | default | -       |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | yes    | info   |          |        |         0 |         1 | default  | default  | master 3   | master     | standby     | master      | standby | worker  | default | default |
   </code></pre>

   d. Опять же, остановите сервер имен на активном в данный момент главном узле (то есть на узле **hanadb3**).  
   
   <pre><code>
    hn1adm@hanadb3:/usr/sap/HN1/HDB03> HDB kill
   </code></pre>

   Узел **hanadb1** возобновит роль главного узла. После завершения теста отработки отказа состояние будет выглядеть следующим образом.

   <pre><code>
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList & python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GRAY
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | no     | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
   </code></pre>

   д) Запустите SAP HANA в **hanadb3**, который будет готов к работе в качестве резервного узла.  

   <pre><code>
    hn1adm@hanadb3:/usr/sap/HN1/HDB03> HDB start
   </code></pre>

   После запуска SAP HANA в **hanadb3** состояние выглядит следующим образом.  

   <pre><code>
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList & python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GRAY
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | no     | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
   </code></pre>

## <a name="next-steps"></a>Дальнейшие действия

* [Планирование и реализация виртуальных машин Azure для SAP][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
* [Тома NFS версии 4.1 в Azure NetApp Files для SAP HANA](./hana-vm-operations-netapp.md)
* Дополнительные сведения об установке высокого уровня доступности и плана для аварийного восстановления SAP HANA на виртуальных машинах Azure см. в статье [Высокий уровень доступности SAP HANA на виртуальных машинах Azure][sap-hana-ha].