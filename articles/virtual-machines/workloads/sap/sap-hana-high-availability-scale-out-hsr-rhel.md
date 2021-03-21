---
title: SAP HANA горизонтального масштабирования с помощью HSR и Pacemaker на RHEL | Документация Майкрософт
description: SAP HANA горизонтальное масштабирование с помощью HSR и Pacemaker на RHEL
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
ms.date: 02/01/2021
ms.author: radeltch
ms.openlocfilehash: dd000ca36bb0ad586b973b2bcdc638d8992b1a73
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101668645"
---
# <a name="high-availability-of-sap-hana-scale-out-system-on-red-hat-enterprise-linux"></a>Высокий уровень доступности SAP HANA масштабируемой системы на Red Hat Enterprise Linux 

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


В этой статье описывается, как развернуть высокодоступную систему SAP HANA в конфигурации с горизонтальным масштабированием с помощью репликации системы HANA (HSR) и Pacemaker на виртуальных машинах Azure Red Hat Enterprise Linux. Общие файловые системы в представленной архитектуре предоставляются [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) и подключены через NFS.  

В примерах конфигураций, команд установки и т. д. экземпляр HANA имеет значение **03** , а идентификатор системы Hana — **HN1**. Примеры основаны на HANA 2,0 SP4 и Red Hat Enterprise Linux для SAP 7,6. 

Прежде чем начать, ознакомьтесь со следующими примечаниями и документацией по SAP:

* Примечание SAP [1928533] включает в себя:  
  * Список размеров виртуальных машин Azure, которые поддерживаются при развертывании программного обеспечения SAP.
  * важные сведения о доступных ресурсах для каждого размера виртуальной машины Azure;
  * сведения о поддерживаемом программном обеспечении SAP и сочетаниях операционных систем и баз данных;
  * Требуемая версия ядра SAP для Windows и Linux на Microsoft Azure
* Примечание SAP [2015553]. список необходимых компонентов для развертываний программного обеспечения SAP, поддерживаемых SAP в Azure
* В примечании SAP [2002167] рекомендуются параметры ОС для Red Hat Enterprise Linux
* примечание к SAP [2009879], содержащее рекомендации по SAP HANA для Red Hat Enterprise Linux;
* Примечание SAP [2178632]: содержит подробные сведения обо всех метриках мониторинга, сообщаемых для SAP в Azure.
* Примечание SAP [2191498]: содержит обязательную версию агента узла SAP для Linux в Azure.
* Примечание SAP [2243692]: содержит сведения о лицензировании SAP в Linux в Azure.
* Примечание SAP [1999351]. содержит дополнительные сведения об устранении неполадок для расширения расширенного мониторинга Azure для SAP.
* Примечание SAP [1900823]: содержит сведения о требованиях к хранилищу SAP HANA
* [Вики-сайт сообщества SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes)содержит все необходимые примечания SAP для Linux.
* [SAP NetWeaver на виртуальных машинах Linux. Руководство по планированию и внедрению][planning-guide]
* [Развертывание программного обеспечения SAP на виртуальных машинах Linux в Azure][deployment-guide]
* [Общие сведения о развертывании СУБД на виртуальных машинах Azure для рабочих нагрузок SAP][dbms-guide]
* [Требования к сети SAP HANA](https://www.sap.com/documents/2016/08/1cd2c2fb-807c-0010-82c7-eda71af511fa.html)
* Общая документация по RHEL
  * [Общие сведения о надстройке для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Администрирование надстройки для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Справочник по надстройке для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
  * [Red Hat Enterprise Linux сетевого руководством](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide)
  * [Разделы справки настроить репликацию SAP HANA Scale-Out системы в кластере Pacemaker с файловыми системами HANA в общих папках NFS](https://access.redhat.com/solutions/5423971)
* Документация по RHEL для Azure:
  * [Установка SAP HANA в Red Hat Enterprise Linux для использования в Microsoft Azure](https://access.redhat.com/public-cloud/microsoft-azure)
  * [Red Hat Enterprise Linux решение для SAP HANA Scale-Out и репликации системы](https://access.redhat.com/solutions/4386601)
* [Приложения NetApp SAP в Microsoft Azure. Использование Azure NetApp Files][anf-sap-applications-azure]
* [Документация по Azure NetApp Files][anf-azure-doc] 
* [Тома NFS версии 4.1 в Azure NetApp Files для SAP HANA](./hana-vm-operations-netapp.md)

## <a name="overview"></a>Обзор

Одним из способов достижения высокого уровня доступности HANA для масштабных установок HANA является настройка репликации системы HANA и защита решения с помощью кластера Pacemaker для обеспечения автоматической отработки отказа. При сбое активного узла кластер отключается от ресурсов HANA к другому сайту.  
В представленной конфигурации показаны три узла HANA на каждом сайте, а также узел "большинство разработчиков" для предотвращения сценария разбиения на страницы. Инструкции можно адаптировать, чтобы включить дополнительные виртуальные машины в качестве узлов базы данных HANA.  

Общая файловая система HANA `/hana/shared` в представленной архитектуре обеспечивается [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md). Он монтируется через Нфсв 4.1 на каждом узле HANA на одном сайте репликации системы HANA. Файловые системы `/hana/data` и `/hana/log` являются локальными файловыми системами и не являются общими для узлов базы данных Hana. SAP HANA будет установлен в режиме без общего доступа. 

> [!TIP]
> Рекомендуемые конфигурации хранилища SAP HANA см. в статье [SAP HANA конфигурации хранилища виртуальных машин Azure](./hana-vm-operations-storage.md).   

[![SAP HANA масштабирование с помощью кластера HSR и Pacemaker](./media/sap-hana-high-availability-rhel/sap-hana-high-availability-scale-out-hsr-rhel.png)](./media/sap-hana-high-availability-rhel/sap-hana-high-availability-scale-out-hsr-rhel-detail.png#lightbox)

На предыдущей схеме три подсети представлены в одной виртуальной сети Azure, следуя рекомендациям SAP HANA сети: 
* для связи с клиентом — `client` 10.23.0.0/24.  
* для внутренних взаимодействий между узлами HANA — `inter` 10.23.1.128/26.  
* для репликации системы HANA — `hsr` 10.23.1.192/26.  

Как `/hana/data` и `/hana/log` развертываются на локальных дисках, нет необходимости развертывать отдельную подсеть и отдельные виртуальные сетевые карты для связи с хранилищем.  

Тома NetApp для Azure развертываются в отдельной подсети, [делегировано Azure NetApp Files] ( https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-delegate-subnet: `anf` 10.23.1.0/26.   

## <a name="set-up-the-infrastructure"></a>Настройка инфраструктуры

В приведенных ниже инструкциях предполагается, что вы уже создали группу ресурсов, виртуальную сеть Azure с тремя сетевыми подсетями Azure `client` : `inter` и `hsr` .

### <a name="deploy-linux-virtual-machines-via-the-azure-portal"></a>Развертывание виртуальных машин Linux с помощью портал Azure
1. Разверните виртуальные машины Azure.  
Для конфигурации, представленной в этом документе, разверните семь виртуальных машин: 
   - три виртуальных машины, которые служат узлами базы данных HANA для репликации HANA Site 1: **Hana-S1-DB1**, **Hana-S1-DB2** и **Hana-S1-db3**  
   - три виртуальных машины, которые служат узлами базы данных HANA для репликации HANA Site 2: **Hana-S2-DB1**, **Hana-S2-DB2** и **Hana-S2-db3**  
   - небольшая виртуальная машина, которая будет служить в качестве *большинства изготовителей*: **Hana-s-mm**

   Виртуальные машины, развернутые в качестве узлов SAP DB HANA, должны быть сертифицированы SAP для HANA как опубликованные в [каталоге SAP HANA оборудования](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). При развертывании узлов базы данных HANA убедитесь, что выбран параметр [Ускоренная сеть](../../../virtual-network/create-vm-accelerated-networking-cli.md) .  
  
   Для узла "большинство разработчиков" можно развернуть небольшую виртуальную машину, так как эта виртуальная машина не запускает ресурсы SAP HANA. Виртуальная машина "большинство разработчиков" используется в конфигурации кластера для достижения нечетного числа узлов кластера в сценарии раздельного мозгового случая. В этом примере виртуальной машине с производителем большинства нужен только один виртуальный сетевой интерфейс в `client` подсети.        

   Развертывание локальных управляемых дисков для `/hana/data` и `/hana/log` . Минимальная рекомендуемая конфигурация хранилища для `/hana/data` и `/hana/log` описана в статье [SAP HANA конфигурации хранилища виртуальных машин Azure](./hana-vm-operations-storage.md).

   Разверните основной сетевой интерфейс для каждой виртуальной машины в `client` подсети виртуальной сети.  
   При развертывании виртуальной машины с помощью портал Azure имя сетевого интерфейса создается автоматически. В этих инструкциях мы будем называть автоматически создаваемые основные сетевые интерфейсы, подключенные к `client` подсети виртуальной сети Azure, как **Hana-S1-DB1-Client**, Hana-S1- **DB2**-Client, **Hana-S1 — db3-** Client и т. д.  


   > [!IMPORTANT]
   > Убедитесь, что выбранная ОС является сертифицированной SAP для SAP HANA на конкретных типах виртуальных машин, которые вы используете. Список сертифицированных типов виртуальных машин и выпусков ОС для этих типов SAP HANA см. на веб-сайте [SAP HANA сертифицированные платформы IaaS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) . Щелкните сведения о списке типа виртуальной машины, чтобы получить полный список поддерживаемых SAP HANA выпусков ОС для этого типа.  
  

2. Создайте шесть сетевых интерфейсов, по одному для каждой виртуальной машины базы данных HANA, в `inter` подсети виртуальной сети (в этом примере это **Hana-S1-DB1-внутрихолдинговый**, Hana **-S1-DB2**-Внутрихолдинговый, Hana- **S1-db3-** **Внутрихолдинговый, Hana**-S2- **DB2**-Внутрихолдинговый, Hana – косайтового уровня) и Hana- **S2-db3-** in).  

3. Создайте шесть сетевых интерфейсов, по одному для каждой виртуальной машины базы данных HANA, в `hsr` подсети виртуальной сети (в этом примере это **Hana-** S1-DB1-HSR, **Hana-S1-DB2-HSR**, Hana- **S1-db3-HSR**, Hana-S2- **DB2**-HSR и Hana-S2- **HSR-db3**).   

4. Подключите только что созданные виртуальные сетевые интерфейсы к соответствующим виртуальным машинам.  

    а. Перейдите к виртуальной машине в [портал Azure](https://portal.azure.com/#home).  

    b. В левой области выберите **виртуальные машины**. Выполните фильтрацию по имени виртуальной машины (например, **Hana-S1-DB1**), а затем выберите виртуальную машину.  

    c. В области **Обзор** выберите пункт **прерывать** , чтобы освободить виртуальную машину.  

    d. Выберите **сеть**, а затем подключите сетевой интерфейс. В раскрывающемся списке **Подключить сетевой интерфейс** выберите уже созданные сетевые интерфейсы для `inter` `hsr` подсетей и.  
    
    д) Щелкните **Сохранить**. 
 
    е) Повторите шаги с b по e для оставшихся виртуальных машин (в нашем примере это  **Hana-S1-DB2**, **Hana-S1-db3**, **Hana-S2-DB1**, **Hana-S2-DB2** и **Hana-S2-db3**).
 
    ж. Оставить виртуальные машины в остановленном состоянии сейчас. Далее мы разберем функцию [ускорения сети](../../../virtual-network/create-vm-accelerated-networking-cli.md) для всех вновь подключенных сетевых интерфейсов.  

5. Включите ускоренную сеть для дополнительных сетевых интерфейсов для `inter` `hsr` подсетей и, выполнив следующие действия.  

    а. Откройте [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) в [портал Azure](https://portal.azure.com/#home).  

    b. Выполните следующие команды, чтобы включить ускоренную сеть для дополнительных сетевых интерфейсов, подключенных к `inter` `hsr` подсетям и.  

    ```azurecli
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db1-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db2-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db3-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db1-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db2-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db3-inter --accelerated-networking true
    
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db1-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db2-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db3-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db1-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db2-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db3-hsr --accelerated-networking true
    ```

7. Запуск виртуальных машин базы данных HANA

### <a name="deploy-azure-load-balancer"></a>Развертывание Azure Load Balancer

1. Мы рекомендуем использовать стандартный балансировщик нагрузки. Выполните следующие действия по настройке для развертывания стандартной подсистемы балансировки нагрузки.
   1. Сначала создайте пула IP-адресов для интерфейсной части.

      1. Откройте подсистему балансировки нагрузки, выберите **пул интерфейсных IP-адресов** и щелкните **Добавить**.
      1. Введите имя нового пула интерфейсных IP-адресов (например, **hana-frontend**).
      1. Задайте для **назначения** значение **static** и введите IP-адрес (например, **10.23.0.18**).
      1. Щелкните **ОК**.
      1. Когда пул интерфейсных IP-адресов будет создан, запишите его IP-адрес.

   1. Затем создайте внутренний пул и добавьте все виртуальные машины кластера в серверный пул:

      1. Откройте подсистему балансировки нагрузки, выберите **серверные пулы** и щелкните **Добавить**.
      1. Введите имя нового серверного пула (например, **hana-backend**).
      1. Щелкните **Добавить виртуальную машину**.
      1. Выберите **Виртуальная машина**.
      1. Выберите виртуальные машины кластера SAP HANA и их IP-адреса для `client` подсети.
      1. Выберите **Добавить**.

   1. Создайте зонд работоспособности.

      1. Откройте подсистему балансировки нагрузки, выберите **Зонды работоспособности** и щелкните **Добавить**.
      1. Введите имя нового зонда работоспособности (например, **hana-hp**).
      1. Выберите протокол **TCP** и порт 625 **03**. Сохраните значение "5" для параметра **Интервал** и значение "2" для параметра **Порог состояния неработоспособности**.
      1. Щелкните **ОК**.

   1. Теперь создайте правила балансировки нагрузки.
   
      1. Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
      1. Введите имя нового правила балансировки нагрузки (например, **hana-lb**).
      1. Выберите интерфейсный пул IP-адресов, серверный пул и зонд работоспособности, который вы создали ранее (например, **hana-frontend**, **hana-backend** и **hana-hp**).
      1. Выберите **Порты высокой доступности**.
      1. Увеличьте **время ожидания** до 30 минут.
      1. Не забудьте **включить плавающий IP-адрес**.
      1. Щелкните **ОК**.

   > [!IMPORTANT]
   > Плавающий IP-адрес не поддерживается для вторичной IP-конфигурации NIC в сценариях балансировки нагрузки. Дополнительные сведения см. в статье [ограничения балансировщика нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, разверните вторую сетевую карту.    
   
   > [!Note]
   > Если в серверный пул внутреннего (без общедоступного IP-адреса) Azure Load Balancer ценовой категории "Стандартный" помещаются виртуальные машины без общедоступных IP-адресов, у них не будет исходящего подключения к Интернету без дополнительной настройки, разрешающей маршрутизацию к общедоступным конечным точкам. Подробные сведения о такой настройке см. в статье [Подключение к общедоступной конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. категория "Стандартный") в сценариях обеспечения высокого уровня доступности SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  


   > [!IMPORTANT]
   > Не включайте метки времени TCP на виртуальных машинах Azure, размещенных за Azure Load Balancer. Включение меток времени TCP помешает работе проб работоспособности. Установите для параметра **net.ipv4.tcp_timestamps** значение **0**. Дополнительные сведения см. в статье [Пробы работоспособности Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md).
   > Также см. [примечание к SAP 2382421](https://launchpad.support.sap.com/#/notes/2382421).  

### <a name="deploy-the-azure-netapp-files-infrastructure"></a>Развертывание инфраструктуры Azure NetApp Files 

Разверните тома использовании для `/hana/shared` файловой системы. Вам потребуется отдельный `/hana/shared` том для каждого сайта репликации системы Hana. Дополнительные сведения см. [в разделе Настройка инфраструктуры Azure NetApp Files](./sap-hana-scale-out-standby-netapp-files-rhel.md#set-up-the-azure-netapp-files-infrastructure).

В этом примере использовались следующие Azure NetApp Files тома: 

* Volume **HN1**-Shared-s1 (NFS://10.23.1.7/**HN1**-Shared-S1)
* Volume **HN1**-Shared-s2 (NFS://10.23.1.7/**HN1**-Shared-S2)

## <a name="operating-system-configuration-and-preparation"></a>Настройка и подготовка операционной системы

Инструкции в следующих разделах начинаются с одной из следующих сокращений:
* **[A]**: применимо ко всем узлам
* **[AH]**: применимо ко всем узлам базы данных Hana
* **[M]**: применимо к узлу "специалисты по большинству"
* **[AH1]**: применимо ко всем узлам базы данных Hana на сайте 1
* **[AH2]**: применимо ко всем узлам базы данных Hana на сайте 2
* **[1]**: применимо только к узлу 1 базы данных Hana, сайт 1
* **[2]**: применимо только к узлу 1 базы данных Hana, сайт 2


Настройте и подготовьте ОС, выполнив следующие действия.

1. **[A]** сохраните файлы узлов на виртуальных машинах. Включить записи для всех подсетей. Для этого примера были добавлены следующие записи `/etc/hosts` .  

    ```bash
     # Client subnet
     10.23.0.11 hana-s1-db1
     10.23.0.12 hana-s1-db1
     10.23.0.13 hana-s1-db2
     10.23.0.14 hana-s2-db1
     10.23.0.15 hana-s2-db2
     10.23.0.16 hana-s2-db3
     10.23.0.17 hana-s-mm
     # Internode subnet
     10.23.1.138 hana-s1-db1-inter
     10.23.1.139 hana-s1-db2-inter
     10.23.1.140 hana-s1-db3-inter
     10.23.1.141 hana-s2-db1-inter
     10.23.1.142 hana-s2-db2-inter
     10.23.1.143 hana-s2-db3-inter
     # HSR subnet
     10.23.1.202 hana-s1-db1-hsr
     10.23.1.203 hana-s1-db2-hsr
     10.23.1.204 hana-s1-db3-hsr
     10.23.1.205 hana-s2-db1-hsr
     10.23.1.206 hana-s2-db2-hsr
     10.23.1.207 hana-s2-db3-hsr
    ```


2. **[A]** установите клиентский пакет NFS.  

    ```yum install nfs-utils ```


3. **[AH]** Настройка Red Hat для HANA.  

    Настройте RHEL, как описано в <https://access.redhat.com/solutions/2447641> и в следующих примечаниях SAP:  
   - [2292690 - SAP HANA DB: Recommended OS settings for RHEL 7](https://launchpad.support.sap.com/#/notes/2292690) (Примечание по поддержке SAP № 2292690. База данных SAP HANA: рекомендуемые параметры операционной системы для RHEL 7).
   - [2777782-SAP HANA DB: Рекомендуемые параметры ОС для RHEL 8](https://launchpad.support.sap.com/#/notes/2777782)
   - [2455582-Linux: запуск приложений SAP, скомпилированных с помощью GCC 6. x](https://launchpad.support.sap.com/#/notes/2455582)
   - [2593824-Linux: запуск приложений SAP, скомпилированных с помощью GCC 7. x](https://launchpad.support.sap.com/#/notes/2593824) 
   - [2886607-Linux: запуск приложений SAP, скомпилированных с помощью GCC 9. x](https://launchpad.support.sap.com/#/notes/2886607)

## <a name="prepare-the-file-systems"></a>Подготовка файловых систем
### <a name="mount-the-shared-file-systems"></a>Подключение общих файловых систем

В этом примере общие файловые системы HANA развертываются на Azure NetApp Files и подключены через NFSv4.  

1. **[AH]** Создайте точки подключения для томов базы данных HANA.  

    ```bash
    mkdir -p /hana/shared
    ```

2. **[AH]** Проверьте параметр домена NFS. Убедитесь, что домен настроен в качестве домена Azure NetApp Files по умолчанию, т. е **`defaultv4iddomain.com`** . для сопоставления установлено значение **никто**.  
   Этот шаг необходим, если используется Azure Нетаппфилес Нфсв 4.1.  

    > [!IMPORTANT]
    > Обязательно укажите домен NFS в `/etc/idmapd.conf` виртуальной машине в соответствии с конфигурацией домена по умолчанию в Azure NetApp Files: **`defaultv4iddomain.com`** . В случае несоответствия между конфигурацией домена на клиенте NFS (т. е. виртуальной машине) и NFS-сервере (т. е. конфигурацией Azure NetApp) разрешения для файлов на томах NetApp в Azure, подключенных к виртуальным машинам, будут отображаться как `nobody`.  

    ```bash
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Domain = defaultv4iddomain.com
    [Mapping]
    Nobody-User = nobody
    Nobody-Group = nobody
    ```

3. **[AH]** Проверьте `nfs4_disable_idmapping` . Ему должно быть задано значение **Y**. Чтобы создать структуру каталогов, в которой находится параметр `nfs4_disable_idmapping`, выполните команду mount. Вы не сможете вручную создать каталог в /sys/modules, так как доступ зарезервирован для ядра или драйверов.  
   Этот шаг необходим, если используется Azure Нетаппфилес Нфсв 4.1.  

    ```bash
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    # If you need to set nfs4_disable_idmapping to Y
    mkdir /mnt/tmp
    mount 10.9.0.4:/HN1-shared /mnt/tmp
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    ```

   Дополнительные сведения о том, как изменить `nfs4_disable_idmapping` параметр, см https://access.redhat.com/solutions/1749883 . в разделе.

4. **[AH1]** Подключите общие Azure NetApp Files тома к виртуальным машинам базы данных сайту SITE1 HANA.  

    ```bash
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.23.1.7:/HN1-shared-s1 /hana/shared
    ```

5. **[AH2]** Подключите общие Azure NetApp Files тома к виртуальным машинам базы данных SITE2 HANA.  

    ```bash
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.23.1.7:/HN1-shared-s2 /hana/shared
    ```


10. **[AH]** Убедитесь, что соответствующие `/hana/shared/` файловые системы подключены ко всем виртуальным машинам базы данных Hana с протоколом NFS версии **NFSv4**.  

    ```bash
    sudo nfsstat -m
    # Verify that flag vers is set to 4.1 
    # Example from SITE 1, hana-s1-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s1
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.11,local_lock=none,addr=10.23.1.7
    # Example from SITE 2, hana-s2-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s2
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.14,local_lock=none,addr=10.23.1.7
    ```

### <a name="prepare-the-data-and-log-local-file-systems"></a>Подготовка локальных файловых систем данных и журналов
В представленной конфигурации файловые системы `/hana/data` и `/hana/log` развертываются на управляемом диске и локально подключены к каждой ВИРТУАЛЬНОЙ машине Hana DB. Вам потребуется выполнить действия по созданию локальных данных и томов журналов на каждой виртуальной машине HANA DB. 

Настройте структуру дисков с помощью  **диспетчера логических томов (LVM)**. В следующем примере предполагается, что к каждой виртуальной машине HANA подключены три диска данных, которые используются для создания двух томов.

1. **[AH]** Список всех доступных дисков:
    ```bash
    ls /dev/disk/azure/scsi1/lun*
    ```

   Выходные данные примера:

    ```bash
    /dev/disk/azure/scsi1/lun0  /dev/disk/azure/scsi1/lun1  /dev/disk/azure/scsi1/lun2 
    ```

2. **[AH]** Создайте физические тома для всех дисков, которые вы хотите использовать.
    ```bash
    sudo pvcreate /dev/disk/azure/scsi1/lun0
    sudo pvcreate /dev/disk/azure/scsi1/lun1
    sudo pvcreate /dev/disk/azure/scsi1/lun2
    ```

3. **[AH]** Создайте группу томов для файлов данных. Используйте одну группу томов для файлов журналов и другую — для общей папки SAP HANA.
    ```bash
    sudo vgcreate vg_hana_data_HN1 /dev/disk/azure/scsi1/lun0 /dev/disk/azure/scsi1/lun1
    sudo vgcreate vg_hana_log_HN1 /dev/disk/azure/scsi1/lun2
    ```

4. **[AH]** Создайте логические тома. 
   Линейный том создается при использовании `lvcreate` без параметра `-i`. Мы рекомендуем создать чередующийся том для повышения производительности операций ввода-вывода, определяя размеры блоков чередования по значениям из статьи [Конфигурации хранилища виртуальных машин SAP HANA в Azure](./hana-vm-operations-storage.md). Аргумент `-i` обозначает число базовых физических томов, а аргумент `-I` — размер блоков чередования. В этом документе для тома данных используется 2 физических тома, поэтому аргумент `-i` имеет значение **2**. Размер чередования для тома данных составляет **256 КИБ**. Журнальный том использует один физический том, поэтому параметры `-i` и `-I` не используются явным образом в командах для журнального тома.  

   > [!IMPORTANT]
   > Используйте `-i` параметр и задайте для него номер базового физического тома, если для каждого тома данных или журнала используется более одного физического тома. Используйте параметр `-I`, чтобы указать размер блока чередования при создании чередующегося тома.  
   > Сведения о рекомендуемых конфигурациях хранилища, включая размеры блоков чередования и количество дисков, см. в статье [Конфигурации хранилища виртуальных машин SAP HANA в Azure](./hana-vm-operations-storage.md).  

    ```bash
    sudo lvcreate -i 2 -I 256 -l 100%FREE -n hana_data vg_hana_data_HN1
    sudo lvcreate -l 100%FREE -n hana_log vg_hana_log_HN1
    sudo mkfs.xfs /dev/vg_hana_data_HN1/hana_data
    sudo mkfs.xfs /dev/vg_hana_log_HN1/hana_log
    ```

5. **[AH]** Создайте каталоги подключения и скопируйте UUID всех логических томов:
    ```bash
    sudo mkdir -p /hana/data/HN1
    sudo mkdir -p /hana/log/HN1
    # Write down the ID of /dev/vg_hana_data_HN1/hana_data and /dev/vg_hana_log_HN1/hana_log
    sudo blkid
    ```

6. **[AH]** Создайте `fstab` записи для логических томов и подключения:
    ```bash
    sudo vi /etc/fstab
    ```

   Вставьте следующую строку в файл `/etc/fstab`:

    ```bash
    /dev/disk/by-uuid/UUID of /dev/mapper/vg_hana_data_HN1-hana_data /hana/data/HN1 xfs  defaults,nofail  0  2
    /dev/disk/by-uuid/UUID of /dev/mapper/vg_hana_log_HN1-hana_log /hana/log/HN1 xfs  defaults,nofail  0  2
    ```

   Подключите новые тома:

    ```bash
    sudo mount -a
    ```

## <a name="installation"></a>Установка  

В этом примере для развертывания SAP HANA в конфигурации с горизонтальным масштабированием с помощью HSR на виртуальных машинах Azure мы использовали HANA 2,0 SP4.  

### <a name="prepare-for-hana-installation"></a>Подготовка к установке HANA

1. **[AH]** Перед установкой HANA задайте корневой пароль. После завершения установки можно отключить корневой пароль. Выполните `root` команду AS `passwd` .  

2. **[1, 2]** измените разрешения для `/hana/shared` 
    ```bash
    chmod 775 /hana/shared
    ```

3. **[1]** убедитесь, что вы можете войти через SSH в виртуальные машины базы данных Hana на этом сайте **Hana-S1-DB2** и **Hana-S1-db3** без запроса пароля.  
   Если это не так, ключи SSH Exchange, как описано в разделе [использование проверки подлинности на основе ключей](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s2-ssh-configuration-keypairs).  
    ```bash
    ssh root@hana-s1-db2
    ssh root@hana-s1-db3
    ```

4. **[2]** убедитесь, что вы можете войти через SSH в виртуальные машины базы данных Hana на этом сайте **Hana-S2-DB2** и **Hana-S2-db3** без запроса пароля.  
   Если это не так, ключи SSH Exchange, как описано в разделе [использование проверки подлинности на основе ключей](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s2-ssh-configuration-keypairs).  
    ```bash
    ssh root@hana-s2-db2
    ssh root@hana-s2-db3
    ```

5. **[AH]** Установите дополнительные пакеты, необходимые для HANA 2,0 SP4. Дополнительные сведения см. в разделе SAP Note [2593824](https://launchpad.support.sap.com/#/notes/2593824) для RHEL 7. 

    ```bash
    # If using RHEL 7
    yum install libgcc_s1 libstdc++6 compat-sap-c++-7 libatomic1
    # If using RHEL 8
    yum install libatomic libtool-ltdl.x86_64
    ```


6. **[A]** временно отключите брандмауэр, чтобы он не влиял на установку Hana. После завершения установки HANA ее можно включить повторно. 
    ```bash
    # Execute as root
    systemctl stop firewalld
    systemctl disable firewalld
    ```

### <a name="hana-installation-on-the-first-node-on-each-site"></a>Установка HANA на первом узле на каждом сайте

1. **[1]** установите SAP Hana, следуя инструкциям из руководства по [установке и обновлению SAP HANA 2,0](https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.04/en-US/7eb0167eb35e4e2885415205b8383584.html). В приведенных ниже инструкциях мы покажем установку SAP HANA на первом узле сайта 1.   

   а. Запустите программу **hdblcm** , как `root` в каталоге с установочным программным обеспечением Hana. Используйте `internal_network` параметр и передайте адресное пространство для подсети, используемое для обмена данными между УЗЛАМИ Hana.  

    ```bash
    ./hdblcm --internal_network=10.23.1.128/26
    ```

   b. В командной строке введите следующие значения:

     * Для **выберите действие**: введите **1** (для установки).
     * Дополнительные **компоненты для установки**: введите **2, 3**
     * Для пути установки: нажмите клавишу ВВОД (по умолчанию —/Hana/Shared).
     * Для **имени локального узла**: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * **Чтобы добавить узлы в систему?**: введите **n**
     * Для **SAP HANA идентификатора системы**: введите **HN1** .
     * **Номер экземпляра** [00]: введите **03**
     * Для **рабочей группы локального узла** [default]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **выбора использования системы/введите индекс [4]**: введите **4** (для настраиваемого)
     * Для **расположения томов данных** [/Hana/Data/HN1]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **расположения томов журнала** [/Hana/log/HN1]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **ограничения максимального выделения памяти?** [n]: введите **n**
     * Для **имени узла сертификата для узла Hana-S1-DB1** [Hana-S1-DB1]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **пароля пользователя агента узла SAP (sapadm)** введите пароль.
     * Чтобы **подтвердить пароль пользователя агента узла SAP (sapadm)**, введите пароль.
     * **Пароль для системного администратора (hn1adm)**: введите пароль.
     * Для **корневого каталога системного администратора** [/usr/SAP/HN1/Home]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **системного администратора для входа в систему** [/bin/sh]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **идентификатора пользователя системного администратора** [1001]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **Введите идентификатор группы пользователей (sapsys)** [79]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **пользователя системной базы данных (системный) пароль**: введите пароль системы.
     * Для **подтверждения системного пароля пользователя системной базы данных**: введите пароль системы
     * **Перезапустить систему после перезагрузки компьютера?** [n]: введите **n** 
     * **Чтобы продолжить (y/n)**: Проверьте сводку и, если все выглядит хорошо, введите **y** .

2. **[2]** повторите предыдущий шаг, чтобы установить SAP HANA на первом узле на сайте 2.   

3. **[1, 2]** проверьте global.ini  

   Отобразите global.ini и убедитесь, что конфигурация внутреннего SAP HANA взаимодействия между узлами выполнена. Проверьте раздел **связи** . Оно должно иметь адресное пространство для `inter` подсети и `listeninterface` должно иметь значение `.internal` . Проверьте раздел **internal_hostname_resolution** . Он должен иметь IP-адреса виртуальных машин HANA, принадлежащих к `inter` подсети.  

   ```bash
     sudo cat /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
     # Example from SITE1 
     [communication]
     internal_network = 10.23.1.128/26
     listeninterface = .internal
     [internal_hostname_resolution]
     10.23.1.138 = hana-s1-db1
     10.23.1.139 = hana-s1-db2
     10.23.1.140 = hana-s1-db3
   ```

4. **[1, 2]** подготовка `global.ini` к установке в среде без общего доступа, как описано в примечании к SAP [2080991](https://launchpad.support.sap.com/#/notes/0002080991).  

   ```bash
    sudo vi /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
    [persistence]
    basepath_shared = no
   ```

4. **[1, 2]** перезапустите SAP Hana, чтобы активировать изменения.  

   ```bash
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StopSystem
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StartSystem
   ```

6. **[1, 2]** убедитесь, что клиентский интерфейс будет использовать IP-адреса из `client` подсети для обмена данными.  

    ```bash
    # Execute as hn1adm
    /usr/sap/HN1/HDB03/exe/hdbsql -u SYSTEM -p "password" -i 03 -d SYSTEMDB 'select * from SYS.M_HOST_INFORMATION'|grep net_publicname
    # Expected result - example from SITE 2
    "hana-s2-db1","net_publicname","10.23.0.14"
   ```

   Сведения о том, как проверить конфигурацию, см. в разделе SAP Note [2183363 — Настройка внутренней сети SAP HANA](https://launchpad.support.sap.com/#/notes/2183363).  

7. **[AH]** Измените разрешения для каталогов данных и журналов, чтобы избежать ошибок установки HANA.  

   ```bash
    sudo chmod o+w -R /hana/data /hana/log
   ```

8. **[1]** установите вторичные узлы Hana. Примеры инструкций на этом шаге приведены для сайта 1.  
   а. Запустите резидентную программу **hdblcm** как `root` .    
    ```bash
     cd /hana/shared/HN1/hdblcm
     ./hdblcm 
    ```

   b. В командной строке введите следующие значения:

     * Для **выберите действие**: введите **2** (для добавления узлов).
     * **Чтобы добавить имена узлов с разделителями-запятыми**: Hana-S1-DB2, Hana-S1-db3
     * Дополнительные **компоненты для установки**: введите **2, 3**
     * Для **введите имя привилегированного пользователя [Root]**: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **выбора ролей для узла "Hana-S1-DB2" [1]**: 1 (для рабочего процесса)
     * Для **входа в группу отработки отказа узла "Hana-S1-DB2" [default]**: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **Введите номер раздела хранилища для узла Hana-S1-DB2 [<<assign automatically>>]**: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **Введите рабочую группу для узла "Hana-S1-DB2" [default]**: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **выбора ролей для узла Hana-S1-db3 ' [1]**: 1 (для рабочего процесса)
     * Для **входа в группу отработки отказа узла для узла Hana-S1-db3 ' [default]**: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **Введите номер раздела хранилища для узла Hana-S1-db3 ' [<<assign automatically>>]**: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **входа в группу рабочей группы для узла Hana-S1-db3 ' [default]**: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * **Пароль для системного администратора (hn1adm)**: введите пароль.
     * Для **Введите пароль пользователя агента узла SAP (sapadm)**: введите пароль.
     * Чтобы **подтвердить пароль пользователя агента узла SAP (sapadm)**, введите пароль.
     * Для **имени узла сертификата для узла Hana-S1-DB2** [Hana-S1-DB2]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию
     * Для **имени узла сертификата для узла Hana-S1-db3** [Hana-S1-db3]: нажмите клавишу ВВОД, чтобы принять значение по умолчанию.
     * **Чтобы продолжить (y/n)**: Проверьте сводку и, если все выглядит хорошо, введите **y** .

9. **[2]** повторите предыдущий шаг, чтобы установить вторичные SAP HANA узлы на сайте 2.   

## <a name="configure-sap-hana-20-system-replication"></a>Настройка репликации системы SAP HANA 2.0

1. **[1]** настройте репликацию системы на сайте 1:

   Создайте резервную копию баз данных как **HN1** ADM:

    ```bash
    hdbsql -d SYSTEMDB -u SYSTEM -p "passwd" -i 03 "BACKUP DATA USING FILE ('initialbackupSYS')"
    hdbsql -d HN1 -u SYSTEM -p "passwd" -i 03 "BACKUP DATA USING FILE ('initialbackupHN1')"
    ```

   Скопируйте системные файлы PKI на вторичный сайт.

    ```bash
    scp /usr/sap/HN1/SYS/global/security/rsecssfs/data/SSFS_HN1.DAT hana-s2-db1:/usr/sap/HN1/SYS/global/security/rsecssfs/data/
    scp /usr/sap/HN1/SYS/global/security/rsecssfs/key/SSFS_HN1.KEY  hana-s2-db1:/usr/sap/HN1/SYS/global/security/rsecssfs/key/
    ```

   Создайте основной сайт:

    ```bash
    hdbnsutil -sr_enable --name=HANA_S1
    ```

2. **[2]** настройте репликацию системы на сайте 2:
    
   Зарегистрируйте второй сайт, чтобы запустить репликацию системы. Выполните следующую команду от имени <hanasid\>adm.

    ```bash
    sapcontrol -nr 03 -function StopWait 600 10
    hdbnsutil -sr_register --remoteHost=hana-s1-db1 --remoteInstance=03 --replicationMode=sync --name=HANA_S2
    sapcontrol -nr 03 -function StartSystem
    ```

3. **[1]** Проверка состояния репликации

   Проверьте состояние репликации и подождите, пока все базы данных синхронизируются.

    ```bash
    sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
    # | Database | Host          | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary     | Secondary | Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
    # |          |               |       |              |           |         |           | Host          | Port      | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
    # | -------- | ------------- | ----- | ------------ | --------- | ------- | --------- | ------------- | --------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
    # | HN1      | hana-s1-db3   | 30303 | indexserver  |         5 |       1 | HANA_S1   | hana-s2-db3   |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | SYSTEMDB | hana-s1-db1   | 30301 | nameserver   |         1 |       1 | HANA_S1   | hana-s2-db1   |     30301 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | HN1      | hana-s1-db1   | 30307 | xsengine     |         2 |       1 | HANA_S1   | hana-s2-db1   |     30307 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | HN1      | hana-s1-db1   | 30303 | indexserver  |         3 |       1 | HANA_S1   | hana-s2-db1   |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | HN1      | hana-s1-db2   | 30303 | indexserver  |         4 |       1 | HANA_S1   | hana-s2-db2   |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    #
    # status system replication site "2": ACTIVE
    # overall system replication status: ACTIVE
    #
    # Local System Replication State
    #   
    # mode: PRIMARY
    # site id: 1
    # site name: HANA_S1
    ```

4. **[1, 2]** измените конфигурацию Hana таким образом, чтобы связь для репликации системы Hana, если она направляется через интерфейс виртуальной сети репликации системы Hana.   
   - Останавливает HANA на обоих сайтах
    ```bash
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StopSystem HDB
    ```

   - Измените global.ini, чтобы добавить сопоставление узла для репликации системы HANA. Используйте IP-адреса из `hsr` подсети.  
    ```bash
    sudo vi /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
    #Add the section
    [system_replication_hostname_resolution]
    10.23.1.202 = hana-s1-db1
    10.23.1.203 = hana-s1-db2
    10.23.1.204 = hana-s1-db3
    10.23.1.205 = hana-s2-db1
    10.23.1.206 = hana-s2-db2
    10.23.1.207 = hana-s2-db3
    ```

   - Запуск HANA на обоих сайтах
   ```bash
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StartSystem HDB
   ```

   Дополнительные сведения см. в разделе [разрешение имен узлов для репликации системы](https://help.sap.com/viewer/eb3777d5495d46c5b2fa773206bbfb46/1.0.12/en-US/c0cba1cb2ba34ec89f45b48b2157ec7b.html).  

5. **[AH]** Снова включите брандмауэр.  
   - Повторное включение брандмауэра
       ```bash
       # Execute as root
       systemctl start firewalld
       systemctl enable firewalld
       ```

   - Откройте необходимые порты брандмауэра. Необходимо настроить порты для номера экземпляра HANA.  

       > [!IMPORTANT]
       > Создайте правила брандмауэра, чтобы разрешить обмен данными между узлами HANA и клиентский трафик. Требуемые порты перечислены в разделе [Порты TCP/IP для всех продуктов SAP](https://help.sap.com/viewer/ports). Приведенные ниже команды являются всего лишь примером. В этом сценарии используется номер системы 03.

       ```bash
        # Execute as root
        sudo firewall-cmd --zone=public --add-port=30301/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30301/tcp
        sudo firewall-cmd --zone=public --add-port=30303/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30303/tcp
        sudo firewall-cmd --zone=public --add-port=30306/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30306/tcp
        sudo firewall-cmd --zone=public --add-port=30307/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30307/tcp
        sudo firewall-cmd --zone=public --add-port=30313/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30313/tcp
        sudo firewall-cmd --zone=public --add-port=30315/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30315/tcp
        sudo firewall-cmd --zone=public --add-port=30317/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30317/tcp
        sudo firewall-cmd --zone=public --add-port=30340/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30340/tcp
        sudo firewall-cmd --zone=public --add-port=30341/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30341/tcp
        sudo firewall-cmd --zone=public --add-port=30342/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30342/tcp
        sudo firewall-cmd --zone=public --add-port=1128/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=1128/tcp
        sudo firewall-cmd --zone=public --add-port=1129/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=1129/tcp
        sudo firewall-cmd --zone=public --add-port=40302/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40302/tcp
        sudo firewall-cmd --zone=public --add-port=40301/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40301/tcp
        sudo firewall-cmd --zone=public --add-port=40307/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40307/tcp
        sudo firewall-cmd --zone=public --add-port=40303/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40303/tcp
        sudo firewall-cmd --zone=public --add-port=40340/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40340/tcp
        sudo firewall-cmd --zone=public --add-port=50313/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=50313/tcp
        sudo firewall-cmd --zone=public --add-port=50314/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=50314/tcp
        sudo firewall-cmd --zone=public --add-port=30310/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30310/tcp
        sudo firewall-cmd --zone=public --add-port=30302/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30302/tcp
       ```

## <a name="create-a-pacemaker-cluster"></a>Создание кластера Pacemaker

Следуйте указаниям в статье [Настройка кластера Pacemaker в Red Hat Enterprise Linux в Azure](high-availability-guide-rhel-pacemaker.md), чтобы создать базовый кластер Pacemaker для этого сервера HANA.
Включите в кластер все виртуальные машины, включая большинства.  

> [!IMPORTANT]
> Не устанавливайте значение `quorum expected-votes` 2, так как это не кластер из двух узлов.  
> Убедитесь, что свойство кластера `concurrent-fencing`  включено, чтобы десериализоваться разграничение узлов.   

## <a name="create-file-system-resources"></a>Создание ресурсов файловой системы

1. **[1, 2]** прерывать SAP HANA на обоих сайтах репликации. Выполните как <SID \> ADM.  

    ```bash
    sapcontrol -nr 03 -function StopSystem
    ```

2. **[AH]** Отключите файловую систему `/hana/shared` , которая была временно подключена для установки на всех виртуальных машинах базы данных Hana. Перед отключением всех процессов и сеансов, использующих файловую систему, необходимо будет отключить их. 
 
    ```bash
    umount /hana/shared 
    ```

3. **[1]** создайте ресурсы кластера файловой системы для `/hana/shared` в отключенном состоянии. Ресурсы создаются с параметром `--disabled` , поскольку необходимо определить ограничения расположения перед включением подключений.  

    ```bash
    # /hana/shared file system for site 1
    pcs resource create fs_hana_shared_s1 --disabled ocf:heartbeat:Filesystem device=10.23.1.7:/HN1-shared-s1  directory=/hana/shared \
    fstype=nfs options='defaults,rw,hard,timeo=600,rsize=262144,wsize=262144,proto=tcp,intr,noatime,sec=sys,vers=4.1,lock,_netdev' op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 \
    op start interval=0 timeout=120 op stop interval=0 timeout=120

    # /hana/shared file system for site 2   
    pcs resource create fs_hana_shared_s2 --disabled ocf:heartbeat:Filesystem device=10.23.1.7:/HN1-shared-s1 directory=/hana/shared \
    fstype=nfs options='defaults,rw,hard,timeo=600,rsize=262144,wsize=262144,proto=tcp,intr,noatime,sec=sys,vers=4.1,lock,_netdev' op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 \
    op start interval=0 timeout=120 op stop interval=0 timeout=120

    # clone the /hana/shared file system resources for both site1 and site2
    pcs resource clone fs_hana_shared_s1 meta clone-node-max=1 interleave=true
    pcs resource clone fs_hana_shared_s2 meta clone-node-max=1 interleave=true
    ```
 
   `OCF_CHECK_LEVEL=20` атрибут добавляется к операции Monitor, поэтому операции мониторинга выполняют проверку файловой системы на чтение и запись. Без этого атрибута операция мониторинга проверяет только то, что файловая система подключена. Это может быть проблемой, поскольку при потере подключения файловая система может остаться подключенной, несмотря на то, что она недоступна.  

   `on-fail=fence` атрибут также добавляется в операцию Monitor. Если этот параметр включен, то в случае сбоя операции мониторинга на узле этот узел немедленно помещается в ограждение. Без этого параметра поведение по умолчанию — отключить все ресурсы, которые зависят от ресурса, на котором произошел сбой, а затем перезапустить неисправный ресурс, а затем запустить все ресурсы, которые зависят от ресурса, на котором произошел сбой. Это не только может занять много времени, когда ресурс SAPHana зависит от неисправного ресурса, но также может завершиться сбоем. Ресурс SAPHana не может быть успешно приостановлен, если общая папка NFS, в которой хранятся двоичные файлы HANA, недоступна.  

4. **[1]** настройте и проверьте атрибуты узла. Всем узлам SAP HANA DB на сайте репликации 1 назначается атрибут `S1` , а всем узлам SAP HANA DB на сайте репликации 2 назначается атрибут `S2` .  

    ```bash
    # HANA replication site 1
    pcs node attribute hana-s1-db1 NFS_SID_SITE=S1
    pcs node attribute hana-s1-db2 NFS_SID_SITE=S1
    pcs node attribute hana-s1-db3 NFS_SID_SITE=S1
    # HANA replication site 2
    pcs node attribute hana-s2-db1 NFS_SID_SITE=S2
    pcs node attribute hana-s2-db2 NFS_SID_SITE=S2
    pcs node attribute hana-s2-db3 NFS_SID_SITE=S2
    #To verify the attribute assignment to nodes execute
    pcs node attribute
    ```

5. **[1]** настройте ограничения, которые определяют, где будут подключены файловые системы NFS и включают ресурсы файловой системы.  
    ```bash
    # Configure the constraints
    pcs constraint location fs_hana_shared_s1-clone rule resource-discovery=never score=-INFINITY NFS_SID_SITE ne S1
    pcs constraint location fs_hana_shared_s2-clone rule resource-discovery=never score=-INFINITY NFS_SID_SITE ne S2
    # Enable the file system resources
    pcs resource enable fs_hana_shared_s1
    pcs resource enable fs_hana_shared_s2
    ```

   При включении ресурсов файловой системы кластер будет подключать `/hana/shared` файловые системы.
 
6. **[AH]** Убедитесь, что тома использовании подключены `/hana/shared` ко всем виртуальным машинам базы данных Hana на обоих сайтах.

    ```bash
    sudo nfsstat -m
    # Verify that flag vers is set to 4.1 
    # Example from SITE 1, hana-s1-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s1
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.11,local_lock=none,addr=10.23.1.7
    # Example from SITE 2, hana-s2-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s2
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.14,local_lock=none,addr=10.23.1.7
    ```

7. **[1]** настройте ресурсы атрибута. Настройте ограничения, которые будут устанавливать атрибуты в `true` , если подключения NFS для `hana/shared` подключены.  

    ```bash
    # Configure the attribure resources
    pcs resource create hana_nfs_s1_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs_s1_active
    pcs resource create hana_nfs_s2_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs_s2_active
    # Clone the attribure resources
    pcs resource clone hana_nfs_s1_active meta clone-node-max=1 interleave=true
    pcs resource clone hana_nfs_s2_active meta clone-node-max=1 interleave=true
    # Configure the constraints, which will set the attribute values
    pcs constraint order fs_hana_shared_s1-clone then hana_nfs_s1_active-clone
    pcs constraint order fs_hana_shared_s2-clone then hana_nfs_s2_active-clone
    ```

   > [!TIP]
   > Если в конфигурацию включены другие файловые системы, помимо/ `hana/shared` , которые подключены к NFS, включите `sequential=false` параметр, чтобы не учитывать зависимости между файловыми системами. Все подключенные файловые системы NFS должны запускаться до соответствующего ресурса атрибута, но им не нужно начинать ни в каком порядке относительно друг друга. Дополнительные сведения см. [в разделе разделы справки configure SAP HANA Scale-Out HSR в кластере Pacemaker, если файловые системы Hana являются общими папками NFS](https://access.redhat.com/solutions/5423971).  

8. **[1]** разместите Pacemaker в режиме обслуживания при подготовке к созданию ресурсов кластера Hana.  
    ```bash
    pcs property set maintenance-mode=true
    ```

## <a name="create-sap-hana-cluster-resources"></a>Создание ресурсов кластера SAP HANA

1. **[A]** установите агент ресурсов масштабирования Hana на всех узлах кластера, включая самого производителя.    

    ```bash
    yum install -y resource-agents-sap-hana-scaleout 
    ```


   > [!NOTE]
   > Ознакомьтесь с [политиками поддержки для кластеров RHEL ha. управление SAP HANA в кластере](https://access.redhat.com/articles/3397471) для минимальной поддерживаемой версии пакета `resource-agents-sap-hana-scaleout` для вашего выпуска ОС.  

2. **[1, 2]** установите для Hana "системный обработчик репликации". Этот обработчик должен быть установлен на одном узле базы данных HANA на каждом системном сайте репликации. SAP HANA должен быть по-прежнему отключен.        

   1. Подготовка обработчика как `root` 
    ```bash
     mkdir -p /hana/shared/myHooks
     cp /usr/share/SAPHanaSR-ScaleOut/SAPHanaSR.py /hana/shared/myHooks
     chown -R hn1adm:sapsys /hana/shared/myHooks
    ```

   2. Становк `global.ini`
    ```bash
    # add to global.ini
    [ha_dr_provider_SAPHanaSR]
    provider = SAPHanaSR
    path = /hana/shared/myHooks
    execution_order = 1
    
    [trace]
    ha_dr_saphanasr = info
    ```

3. **[AH]** Для кластера требуется конфигурация "Sudo" на узле кластера для <SID \> ADM. В этом примере это достигается путем создания нового файла. Выполните команды как `root` .    
    ```bash
    cat << EOF > /etc/sudoers.d/20-saphana
    # SAPHanaSR-ScaleOut needs for srHook
     Cmnd_Alias SOK = /usr/sbin/crm_attribute -n hana_hn1_glob_srHook -v SOK -t crm_config -s SAPHanaSR
     Cmnd_Alias SFAIL = /usr/sbin/crm_attribute -n hana_hn1_glob_srHook -v SFAIL -t crm_config -s SAPHanaSR
     hn1adm ALL=(ALL) NOPASSWD: SOK, SFAIL
     EOF
    ```

4. **[1, 2]** запустите SAP HANA на обоих сайтах репликации. Выполните как <SID \> ADM.  

    ```bash
    sapcontrol -nr 03 -function StartSystem 
    ```

5. **[1]** Проверьте установку обработчика. Выполните команду AS <SID \> ADM на активном сайте репликации системы Hana.   

    ```bash
    cdtrace
     awk '/ha_dr_SAPHanaSR.*crm_attribute/ \
     { printf "%s %s %s %s\n",$2,$3,$5,$16 }' nameserver_*

     # Example entries
     # 2020-07-21 22:04:32.364379 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:04:46.905661 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:04:52.092016 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:04:52.782774 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:04:53.117492 ha_dr_SAPHanaSR SFAIL
     # 2020-07-21 22:06:35.599324 ha_dr_SAPHanaSR SOK
    ```

6. **[1]** создайте ресурсы кластера Hana. Выполните следующие команды в виде `root` .    
   1. Убедитесь, что кластер уже находится в режиме обслуживания.  
    
   2. Затем создайте ресурс топологии HANA.  
      При создании кластера RHEL **7. x** используйте следующие команды:  
      ```bash
      pcs resource create SAPHanaTopology_HN1_HDB03 SAPHanaTopologyScaleOut \
       SID=HN1 InstanceNumber=03 \
       op start timeout=600 op stop timeout=300 op monitor interval=10 timeout=600

      pcs resource clone SAPHanaTopology_HN1_HDB03 meta clone-node-max=1 interleave=true
      ```

      При создании кластера RHEL **8. x** используйте следующие команды:  
      ```bash
      pcs resource create SAPHanaTopology_HN1_HDB03 SAPHanaTopology \
       SID=HN1 InstanceNumber=03 meta clone-node-max=1 interleave=true \
       op methods interval=0s timeout=5 \
       op start timeout=600 op stop timeout=300 op monitor interval=10 timeout=600

      pcs resource clone SAPHanaTopology_HN1_HDB03 meta clone-node-max=1 interleave=true
      ```

   3. Затем создайте ресурс экземпляра HANA.  
      > [!NOTE]
      > Эта статья содержит ссылки на термин « *Ведомый*» термин, который корпорация Майкрософт больше не использует. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.  
 
      При создании кластера RHEL **7. x** используйте следующие команды:    
      ```bash
      pcs resource create SAPHana_HN1_HDB03 SAPHanaController \
       SID=HN1 InstanceNumber=03 PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=false \
       op start interval=0 timeout=3600 op stop interval=0 timeout=3600 op promote interval=0 timeout=3600 \
       op monitor interval=60 role="Master" timeout=700 op monitor interval=61 role="Slave" timeout=700
     
      pcs resource master msl_SAPHana_HN1_HDB03 SAPHana_HN1_HDB03 \
       meta master-max="1" clone-node-max=1 interleave=true
      ```

      При создании кластера RHEL **8. x** используйте следующие команды:  
      ```bash
      pcs resource create SAPHana_HN1_HDB03 SAPHanaController \
       SID=HN1 InstanceNumber=03 PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=false \
       op demote interval=0s timeout=320 op methods interval=0s timeout=5 \
       op start interval=0 timeout=3600 op stop interval=0 timeout=3600 op promote interval=0 timeout=3600 \
       op monitor interval=60 role="Master" timeout=700 op monitor interval=61 role="Slave" timeout=700
     
      pcs resource promotable SAPHana_HN1_HDB03 \
       meta master-max="1" clone-node-max=1 interleave=true
      ```
      > [!IMPORTANT]
      > Рекомендуется установить для AUTOMATED_REGISTER значение **нет**, при выполнении тщательного тестирования при сбое, чтобы предотвратить сбой основного экземпляра, автоматически регистрируемого в качестве получателя. После успешного завершения тестов отработки отказа задайте для параметра AUTOMATED_REGISTER значение **Да**, чтобы после того, как репликация системы перенаправление может возобновить работу автоматически. 

   4. Создайте виртуальный IP-адрес и связанные ресурсы.  
      ```bash
      pcs resource create vip_HN1_03 ocf:heartbeat:IPaddr2 ip=10.23.0.18 op monitor interval="10s" timeout="20s"
      sudo pcs resource create nc_HN1_03 azure-lb port=62503
      sudo pcs resource group add g_ip_HN1_03 nc_HN1_03 vip_HN1_03
      ```

   5. Создание ограничений кластера  
      При создании кластера RHEL **7. x** используйте следующие команды:  
      ```bash
      #Start HANA topology, before the HANA instance
      pcs constraint order SAPHanaTopology_HN1_HDB03-clone then msl_SAPHana_HN1_HDB03

      pcs constraint colocation add g_ip_HN1_03 with master msl_SAPHana_HN1_HDB03 4000
      #HANA resources are only allowed to run on a node, if the node's NFS file systems are mounted. The constraint also avoids the majority maker node
      pcs constraint location SAPHanaTopology_HN1_HDB03-clone rule resource-discovery=never score=-INFINITY hana_nfs_s1_active ne true and hana_nfs_s2_active ne true
      ```
 
      При создании кластера RHEL **8. x** используйте следующие команды:  
      ```bash
      #Start HANA topology, before the HANA instance
      pcs constraint order SAPHanaTopology_HN1_HDB03-clone then SAPHana_HN1_HDB03-clone

      pcs constraint colocation add g_ip_HN1_03 with master SAPHana_HN1_HDB03-clone 4000
      #HANA resources are only allowed to run on a node, if the node's NFS file systems are mounted. The constraint also avoids the majority maker node
      pcs constraint location SAPHanaTopology_HN1_HDB03-clone rule resource-discovery=never score=-INFINITY hana_nfs_s1_active ne true and hana_nfs_s2_active ne true
      ```

7. **[1]** расположить кластер в режиме обслуживания. Убедитесь, что состоянию кластера соответствует значение ОК и все ресурсы запущены.  
    ```bash
    sudo pcs property set maintenance-mode=false
    #If there are failed cluster resources, you may need to run the next command
    pcs resource cleanup
    ```
  
   > [!NOTE]
   > Время ожидания в приведенной выше конфигурации указано только для примера. Его нужно настроить для конкретной установки HANA. Например, время ожидания при запуске лучше увеличить, если запуск базы данных SAP HANA выполняется долго.
  
## <a name="test-sap-hana-failover"></a>Тестовая отработка отказа SAP HANA 

1. Перед началом теста проверьте кластер и SAP HANA состояние репликации системы.  

   а. Убедитесь в отсутствии невыполненных действий кластера.  
     ```bash
     #Verify that there are no failed cluster actions
     pcs status
     # Example
     #Stack: corosync
     #Current DC: hana-s-mm (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
     #Last updated: Thu Sep 24 06:00:20 2020
     #Last change: Thu Sep 24 05:59:17 2020 by root via crm_attribute on hana-s1-db1
     #
     #7 nodes configured
     #45 resources configured
     #
     #Online: [ hana-s-mm hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #
     #Active resources:
     #
     #rsc_st_azure    (stonith:fence_azure_arm):      Started hana-s-mm
     #Clone Set: fs_hana_shared_s1-clone [fs_hana_shared_s1]
     #    Started: [ hana--s1-db1 hana-s1-db2 hana-s1-db3 ]
     #Clone Set: fs_hana_shared_s2-clone [fs_hana_shared_s2]
     #    Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #Clone Set: hana_nfs_s1_active-clone [hana_nfs_s1_active]
     #    Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
     #Clone Set: hana_nfs_s2_active-clone [hana_nfs_s2_active]
     #    Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #Clone Set: SAPHanaTopology_HN1_HDB03-clone [SAPHanaTopology_HN1_HDB03]
     #    Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #Master/Slave Set: msl_SAPHana_HN1_HDB03 [SAPHana_HN1_HDB03]
     #    Masters: [ hana-s1-db1 ]
     #    Slaves: [ hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #Resource Group: g_ip_HN1_03
     #    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hana-s1-db1
     #    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hana-s1-db1
     ```

   b. Проверка синхронизации SAP HANA репликации системы

      ```bash
      # Verify HANA HSR is in sync
      sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
      #| Database | Host        | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary     | Secondary| Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
      #|          |             |       |              |           |         |           | Host          | Port     | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
      #| -------- | ----------- | ----- | ------------ | --------- | ------- | --------- | ------------- | -------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
      #| HN1      | hana-s1-db3 | 30303 | indexserver  |         5 |       2 | HANA_S1   | hana-s2-db3 |     30303  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
      #| HN1      | hana-s1-db2 | 30303 | indexserver  |         4 |       2 | HANA_S1   | hana-s2-db2 |     30303  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |  
      #| SYSTEMDB | hana-s1-db1 | 30301 | nameserver   |         1 |       2 | HANA_S1   | hana-s2-db1 |     30301  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
      #| HN1      | hana-s1-db1 | 30307 | xsengine     |         2 |       2 | HANA_S1   | hana-s2-db1 |     30307  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
      #| HN1      | hana-s1-db1 | 30303 | indexserver  |         3 |       2 | HANA_S1   | hana-s2-db1 |     30303  |         1 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |

      #status system replication site "1": ACTIVE
      #overall system replication status: ACTIVE

      #Local System Replication State
      #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

      #mode: PRIMARY
      #site id: 1
      #site name: HANA_S1
      ```

2. Проверьте конфигурацию кластера на случай сбоя, когда узел теряет доступ к общему ресурсу NFS ( `/hana/shared` ).  

   SAP HANA агенты ресурсов зависят от двоичных файлов, `/hana/shared` которые хранятся в для выполнения операций во время отработки отказа. Файловая система `/hana/shared` подключена через NFS в представленной конфигурации. Тест, который можно выполнить, — это повторное подключение `/hana/shared` файловой системы только для *чтения*. При таком подходе выполняется проверка отработки отказа кластера, если доступ к `/hana/shared` теряется на активном сайте репликации системы.  

   **Ожидаемый результат**: при повторном подключении `/hana/shared` как *доступ только для чтения* операция наблюдения, которая выполняет операции чтения/записи в файловой системе, завершится ошибкой, так как она не сможет выполнить запись в файловую систему и вызовет отработку отказа ресурса Hana. Такой же результат ожидается, когда узел HANA теряет доступ к общему ресурсу NFS.  
     
   Состояние ресурсов кластера можно проверить, выполнив `crm_mon` или `pcs status` . Состояние ресурсов перед запуском теста:
      ```bash
      # Output of crm_mon
      #7 nodes configured
      #45 resources configured

      #Online: [ hana-s-mm hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      #
      #Active resources:

      #rsc_st_azure    (stonith:fence_azure_arm):      Started hana-s-mm
      # Clone Set: fs_hana_shared_s1-clone [fs_hana_shared_s1]
      #    Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
      # Clone Set: fs_hana_shared_s2-clone [fs_hana_shared_s2]
      #     Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Clone Set: hana_nfs_s1_active-clone [hana_nfs_s1_active]
      #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
      # Clone Set: hana_nfs_s2_active-clone [hana_nfs_s2_active]
      #     Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Clone Set: SAPHanaTopology_HN1_HDB03-clone [SAPHanaTopology_HN1_HDB03]
      #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Master/Slave Set: msl_SAPHana_HN1_HDB03 [SAPHana_HN1_HDB03]
      #     Masters: [ hana-s1-db1 ]
      #     Slaves: [ hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Resource Group: g_ip_HN1_03
      #     nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hana-s1-db1
      #     vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hana-s1-db1
      ```

   Чтобы смоделировать сбой для `/hana/shared` одной из виртуальных машин сайта репликации, выполните следующую команду:
      ```bash
      # Execute as root 
      mount -o ro /hana/shared
      # Or if the above command returns an error
      sudo mount -o ro 10.23.1.7/HN1-shared-s1 /hana/shared
      ```
  
   Виртуальная машина HANA, которой был потерян доступ к, `/hana/shared` должна перезапускаться или останавливаться в зависимости от конфигурации кластера. Ресурсы кластера переносятся на другой сайт репликации системы HANA.  
         
   Если кластер не был запущен на виртуальной машине, он был перезапущен, запустите кластер, выполнив следующее: 

      ```bash
      # Start the cluster 
      pcs cluster start
      ```
   
   При запуске кластера файловая система `/hana/shared` будет автоматически подключена.     
   Если задать AUTOMATED_REGISTER = "false", необходимо будет настроить репликацию системы SAP HANA на вторичном сайте. В этом случае можно выполнить следующие команды, чтобы перенастроить SAP HANA как вторичную.   

      ```bash
      # Execute on the secondary 
      su - hn1adm
      # Make sure HANA is not running on the secondary site. If it is started, stop HANA
      sapcontrol -nr 03 -function StopWait 600 10
      # Register the HANA secondary site
      hdbnsutil -sr_register --name=HANA_S1 --remoteHost=hana-s2-db1 --remoteInstance=03 --replicationMode=sync
      # Switch back to root and cleanup failed resources
      pcs resource cleanup SAPHana_HN1_HDB03
      ```

   Состояние ресурсов после теста: 

      ```bash
      # Output of crm_mon
      #7 nodes configured
      #45 resources configured
    
      #Online: [ hana-s-mm hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      
      #Active resources:
    
      #rsc_st_azure    (stonith:fence_azure_arm):      Started hana-s-mm
      # Clone Set: fs_hana_shared_s1-clone [fs_hana_shared_s1]
      #    Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
      # Clone Set: fs_hana_shared_s2-clone [fs_hana_shared_s2]
      #     Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Clone Set: hana_nfs_s1_active-clone [hana_nfs_s1_active]
      #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 ]
      # Clone Set: hana_nfs_s2_active-clone [hana_nfs_s2_active]
      #     Started: [ hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Clone Set: SAPHanaTopology_HN1_HDB03-clone [SAPHanaTopology_HN1_HDB03]
      #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
      # Master/Slave Set: msl_SAPHana_HN1_HDB03 [SAPHana_HN1_HDB03]
      #     Masters: [ hana-s2-db1 ]
      #     Slaves: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db2 hana-s2-db3 ]
      # Resource Group: g_ip_HN1_03
      #     nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hana-s2-db1
      #     vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hana-s2-db1
      ```


Рекомендуется тщательно протестировать конфигурацию кластера SAP HANA, а также выполнить тесты, документированные в [HA для SAP HANA на виртуальных машинах Azure в RHEL](./sap-hana-high-availability-rhel.md#test-the-cluster-setup).


## <a name="next-steps"></a>Дальнейшие действия

* [Планирование и реализация виртуальных машин Azure для SAP][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
* [Тома NFS версии 4.1 в Azure NetApp Files для SAP HANA](./hana-vm-operations-netapp.md)
* Сведения о том, как установить высокий уровень доступности и спланировать аварийное восстановление SAP HANA на виртуальных машинах Azure, см. в статье [высокий уровень доступности SAP HANA на виртуальные машины Azure][sap-hana-ha].