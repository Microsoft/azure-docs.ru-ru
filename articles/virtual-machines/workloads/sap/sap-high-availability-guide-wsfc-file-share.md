---
title: Кластер SAP ASCS/SCS в WSFC с использованием файлового ресурса в Azure | Документация Майкрософт
description: Сведения о кластеризации экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows с помощью файлового ресурса в Azure.
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
ms.date: 03/15/2021
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a51f874d09aebfcb2c0b73e0b484f68042d1bb6d
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103496207"
---
# <a name="cluster-an-sap-ascsscs-instance-on-a-windows-failover-cluster-by-using-a-file-share-in-azure"></a>Кластеризация экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows с помощью файлового ресурса в Azure

> ![Эмблема Windows.][Logo_Windows] Windows
>

Отказоустойчивая кластеризация Windows Server является основой для установки высокодоступных приложений SAP ASCS/SCS и СУБД в Windows.

Отказоустойчивый кластер представляет собой группу из 1 + n независимых серверов (узлов), работающих совместно для повышения доступности приложений и служб. В случае отказа узла отказоустойчивый кластер Windows Server вычисляет допустимое количество сбоев, при котором сохранится работоспособность кластера для предоставления приложений и служб. Возможность отказоустойчивой кластеризации можно добавить, используя разные режимы кворума.

## <a name="prerequisites"></a>Предварительные требования
Прежде чем перейти к задачам, приведенным в этой статье, ознакомьтесь с этой статьей.

* [Архитектура высокого уровня доступности и сценарии для SAP NetWeaver на виртуальных машинах Azure][sap-high-availability-architecture-scenarios]

> [!IMPORTANT]
> Кластеризация экземпляров SAP ASCS/SCS с файловым ресурсом поддерживается для продуктов SAP NetWeaver 7.40 (и более поздней версии) с ядром SAP 7.49 (и более поздней версии).
>


## <a name="windows-server-failover-clustering-in-azure"></a>Отказоустойчивый кластер Windows Server в Azure

На виртуальных машинах Azure нужно выполнить больше действий для настройки WSFC по сравнению с развертываниями без операционной системы и развертываниями в частных облаках. При создании кластера вы должны назначить несколько IP-адресов и имен виртуальных узлов экземпляру SAP ASCS/SCS.

### <a name="name-resolution-in-azure-and-the-cluster-virtual-host-name"></a>Разрешение имен в Azure и имя виртуального узла кластера

Облачная платформа Azure не предоставляет возможность настройки виртуальных IP-адресов, т. е. плавающих IP-адресов. Для настройки виртуального IP-адреса, используемого для доступа к кластерным ресурсам в облаке, вам потребуется альтернативное решение. 

Azure Load Balancer выполняет роль *внутренней подсистемы балансировки нагрузки* для Azure. С его помощью клиенты могут обращаться к кластеру через виртуальный IP-адрес кластера. 

Разверните внутреннюю подсистему балансировки нагрузки в группе ресурсов, которая содержит узлы кластера. Затем настройте все необходимые правила перенаправления портов, используя порты проб внутренней подсистемы балансировки нагрузки. Клиенты могут подключаться через имя виртуального узла. DNS-сервер разрешает IP-адрес кластера. Внутренний балансировщик нагрузки выполняет перенаправление на активный узел кластера.

![Рис. 1. Конфигурация отказоустойчивого кластера Windows Server в Azure без общего диска][sap-ha-guide-figure-1001]

_**Рис. 1.** Конфигурация отказоустойчивого кластера Windows Server без общего диска в Azure_

## <a name="sap-ascsscs-ha-with-file-share"></a>Экземпляр SAP ASCS/SCS высокой доступности с файловым ресурсом

В SAP разработан новый подход и альтернатива общим дискам кластера для кластеризации экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows. Вместо использования общих дисков кластера можно использовать файловый ресурс SMB для развертывания файлов глобального узла SAP.

> [!NOTE]
> Файловый ресурс SMB является альтернативным вариантом общих дисков кластера для кластеризации экземпляров SAP ASCS/SCS.  
>

Для этой архитектуры характерно следующее:

* Центральные службы SAP (с собственной структурой файлов и процессами передачи сообщений и постановки в очередь) отделены от файлов глобального узла SAP.
* Центральные службы SAP выполняются в экземпляре SAP ASCS/SCS.
* Экземпляр SAP ASCS/SCS является кластерным и доступен с помощью \<ASCS/SCS virtual host name\> имени виртуального узла.
* Глобальные файлы SAP помещаются в общую папку SMB и доступны с помощью \<SAP global host\> имени узла: \\ \\ &lt; SAP Global host &gt; \sapmnt \\ &lt; SID &gt; \SYS \. ..
* Экземпляр SAP ASCS/SCS устанавливается на локальный диск на обоих узлах кластера.
* \<ASCS/SCS virtual host name\>Сетевое имя отличается от &lt; глобального узла SAP &gt; .

![Рис. 2. Архитектура высокодоступной системы SAP ASCS/SCS с файловым ресурсом SAP.][sap-ha-guide-figure-8004]

_**Рис. 2.** Новая архитектура высокодоступной системы SAP ASCS/SCS с файловым ресурсом SMB_

Предварительные требования для файлового ресурса SMB:

* Протокол SMB 3.0 (или более поздней версии).
* Возможность установить список управления доступом Active Directory для групп пользователей Active Directory и объект-компьютер `computer$`.
* Файловый ресурс должен быть высокодоступным:
    * Диски, используемые для хранения файлов, не должны представлять собой единую точку отказа.
    * Простой серверов или виртуальных машин не является причиной простоя файлового ресурса.

\<SID\>Роль кластера SAP не содержит общие диски кластера или общий ресурс кластера общего файлового ресурса.


![Рис. 3. Ресурсы роли кластера SAP \<SID\> для использования файлового ресурса][sap-ha-guide-figure-8005]

_**Рис. 3.** Ресурсы роли кластера SAP &lt;SID&gt; для использования файлового ресурса_


## <a name="scale-out-file-shares-with-storage-spaces-direct-in-azure-as-an-sapmnt-file-share"></a>Масштабируемый файловый ресурс с Локальными дисковыми пространствами в Azure в качестве файлового ресурса SAPMNT

Вы можете использовать масштабируемый файловый ресурс для размещения и защиты файлов глобального узла SAP. Масштабируемый файловый ресурс также предоставляет высокодоступную службу файлового ресурса SAPMNT.

![Рис. 4. Масштабируемый файловый ресурс, используемый для защиты файлов глобального узла SAP][sap-ha-guide-figure-8006]

_**Рис. 4.** Масштабируемый файловый ресурс, используемый для защиты файлов глобального узла SAP_

> [!IMPORTANT]
> Масштабируемый файловый ресурс полностью поддерживается в облаке Microsoft Azure, а также в локальных средах.
>

Масштабируемый файловый ресурс предлагает высокодоступный файловый ресурс SAPMNT с возможностью горизонтального масштабирования.

Локальные дисковые пространства используются в качестве общего диска для масштабируемого файлового ресурса. Они также позволяют создать высокодоступное масштабируемое хранилище с использованием серверов с локальным хранилищем. Общее хранилище, используемое для масштабируемого файлового ресурса, например для файлов глобального узла SAP, не представляет собой единую точку отказа.

При выборе Локальные дисковые пространства рассмотрите следующие варианты использования:

- Виртуальные машины, используемые для создания кластера Локальные дисковые пространства, необходимо развернуть в группе доступности Azure.
- Для аварийного восстановления кластера Локальные дисковые пространства можно использовать [службы Azure Site Recovery](../../../site-recovery/azure-to-azure-support-matrix.md#replicated-machines---storage).
- Не поддерживается растяжение прямого кластера дискового пространства в разных Зоны доступности Azure.

### <a name="sap-prerequisites-for-scale-out-file-shares-in-azure"></a>Предварительные требования SAP для масштабируемого файлового ресурса в Azure

Чтобы использовать масштабируемый файловый ресурс, система должна отвечать следующим требованиям:

* По крайней мере два узла кластера для масштабируемого файлового ресурса.
* Каждый узел должен иметь по крайней мере два локальных диска.
* Для поддержки высокой производительности необходимо реализовать *устойчивость за счет зеркального отображения*:
    * Двустороннее зеркальное отображение для масштабируемого файлового ресурса с двумя узлами кластера.
    * Двустороннее зеркальное отображение для масштабируемого файлового ресурса с тремя (или более) узлами кластера.
* Мы рекомендуем три (или более) узла кластера для масштабируемого файлового ресурса с трехсторонним зеркальным отображением.
    Этот вариант обеспечивает большую масштабируемость и отказоустойчивость хранилища, чем вариант масштабируемого файлового ресурса с двумя узлами кластера и двусторонним зеркальным отображением.
* Необходимо использовать диск Azure уровня "Премиум".
* Рекомендуется использовать управляемые диски Azure.
* Рекомендуется отформатировать тома с помощью новой системы Resilient File System (ReFS).
    * См. [[Примечание к SAP 1869038 о поддержке файловой системы ReFs в SAP][1869038]][1869038] и главу [Выбор файловой системы][planning-volumes-s2d-choosing-filesystem] статьи "Планирование томов в локальных дисковых пространствах".
    * Не забудьте установить [накопительное обновление Майкрософт KB4025334][kb4025334].
* Вы можете использовать виртуальные машины Azure серии DS или DSv2.
* Чтобы обеспечить оптимальную производительность внутри сети виртуальных машин, необходимую для синхронизации дисков Локальных дисковых пространств, используйте тип виртуальной машины с высокой пропускной способностью сети.
    Дополнительные сведения см. в разделах спецификации [Серия Dv2][dv2-series] и [Серия DS][ds-series].
* Рекомендуется зарезервировать нераспределенную часть емкости в пуле хранения. Таким образом тома получат пространство для локального восстановления в случае отказа диска. В результате безопасность и производительность данных повысится.  Дополнительные сведения см. в разделе [Выбор размера томов][choosing-the-size-of-volumes-s2d].
* Вам не нужно настраивать внутренний балансировщик нагрузки Azure для сетевого имени масштабируемого файлового ресурса, например для \<SAP global host\> . Это делается для \<ASCS/SCS virtual host name\> экземпляра SAP ASCS/SCS или для СУБД. Масштабируемый файловый ресурс масштабирует нагрузку на всех узлах кластера. \<SAP global host\> использует локальный IP-адрес для всех узлов кластера.


> [!IMPORTANT]
> Нельзя переименовать общую папку SAPMNT, которая указывает на \<SAP global host\> . SAP поддерживает только имя общего ресурса "sapmnt".
>
> Дополнительные сведения см. в [примечании к SAP № 2492395 о том, можно ли изменить имя общей папки sapmnt?][2492395]

### <a name="configure-sap-ascsscs-instances-and-a-scale-out-file-share-in-two-clusters"></a>Настройка экземпляров SAP ASCS/SCS и масштабируемый файловый ресурс в двух кластерах

Экземпляры SAP ASCS/SCS необходимо развернуть в отдельном кластере с собственной \<SID\> ролью кластера SAP. В этом случае Масштабируемый файловый ресурс настраивается в другом кластере с другой ролью кластера.


> [!IMPORTANT]
> Программа установки должна соответствовать следующим требованиям: экземпляры SAP ASCS/SCS и общий ресурс SOFS должны быть развернуты в отдельных кластерах.    
>
> [!IMPORTANT] 
> В этом сценарии экземпляр SAP ASCS/SCS настроен для доступа к глобальному узлу SAP по UNC-пути \\\\&lt;глобальный узел SAP&gt;\sapmnt\\&lt;SID&gt;\SYS\..
>

![Рис. 5. Экземпляр SAP ASCS/SCS и масштабируемый файловый ресурс, развернутые в двух кластерах][sap-ha-guide-figure-8007]

_**Рис. 5.** Экземпляр SAP ASCS/SCS и масштабируемый файловый ресурс, развернутые в двух кластерах_

> [!IMPORTANT]
> В облаке Azure каждый кластер, используемый для SAP и масштабируемых файловых ресурсов, должен быть развернут в отдельной группе доступности Azure или в Зоны доступности Azure. Это гарантирует распределенное размещение этих виртуальных машин кластера в базовой инфраструктуре Azure. Эта технология поддерживает развертывания зон доступности.
>

## <a name="generic-file-share-with-sios-datakeeper-as-cluster-shared-disks"></a>Универсальный файловый ресурс с SIOS DataKeeper в качестве общих дисков кластера


Универсальный файловый ресурс — это еще один вариант получения высокодоступного файлового ресурса.

В таком случае в качестве общего диска кластера вы можете использовать стороннее решение SIOS.

## <a name="next-steps"></a>Дальнейшие действия

* [Подготовка инфраструктуры Azure для SAP высокого уровня доступности с использованием отказоустойчивого кластера Windows и файлового ресурса для экземпляра SAP (A)SCS][sap-high-availability-infrastructure-wsfc-file-share]
* [Установка SAP NetWeaver высокого уровня доступности в Azure с использованием отказоустойчивого кластера Windows и общего диска для экземпляра SAP (A)SCS][sap-high-availability-installation-wsfc-shared-disk]
* [Развертывание масштабируемого файлового сервера Локальных дисковых пространств с двумя узлами для хранения дисков UPD в Azure][deploy-sofs-s2d-in-azure]
* [Локальные дисковые пространства в Windows Server 2016][s2d-in-win-2016]
* [Deep Dive: Volumes in Storage Spaces Direct][deep-dive-volumes-in-s2d] (Подробный обзор. Тома в Локальных дисковых пространствах)

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[kb4025334]:https://support.microsoft.com/help/4025334/windows-10-update-kb4025334

[dv2-series]:../../dv2-dsv2-series.md
[ds-series]:https://docs.microsoft.com/azure/virtual-machines/windows/sizes-general

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-volumes-s2d-choosing-filesystem]:https://docs.microsoft.com/windows-server/storage/storage-spaces/plan-volumes#choosing-the-filesystem
[choosing-the-size-of-volumes-s2d]:https://docs.microsoft.com/windows-server/storage/storage-spaces/plan-volumes#choosing-the-size-of-volumes
[deploy-sofs-s2d-in-azure]:https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-storage-spaces-direct-deployment
[s2d-in-win-2016]:https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview
[deep-dive-volumes-in-s2d]:https://blogs.technet.microsoft.com/filecab/2016/08/29/deep-dive-volumes-in-spaces-direct/

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-ha-guide]:sap-high-availability-guide.md
[sap-ha-guide-2]:#42b8f600-7ba3-4606-b8a5-53c4f026da08
[sap-ha-guide-4]:#8ecf3ba0-67c0-4495-9c14-feec1a2255b7
[sap-ha-guide-8]:#78092dbe-165b-454c-92f5-4972bdbef9bf
[sap-ha-guide-8.1]:#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-ha-guide-8.9]:#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.11]:#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-ha-guide-8.12]:#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-ha-guide-8.12.1]:#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-ha-guide-8.12.3]:#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-ha-guide-8.12.3.1]:#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-ha-guide-8.12.3.2]:#dd41d5a2-8083-415b-9878-839652812102
[sap-ha-guide-8.12.3.3]:#d9c1fc8e-8710-4dff-bec2-1f535db7b006
[sap-ha-guide-9]:#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-ha-guide-9.1.1]:#a97ad604-9094-44fe-a364-f89cb39bf097

[sap-ha-multi-sid-guide]:sap-high-availability-multi-sid.md (Конфигурации высокой доступности SAP с несколькими SID)

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[sap-ha-guide-figure-1000]:./media/virtual-machines-shared-sap-high-availability-guide/1000-wsfc-for-sap-ascs-on-azure.png
[sap-ha-guide-figure-1001]:./media/virtual-machines-shared-sap-high-availability-guide/1001-wsfc-on-azure-ilb.png
[sap-ha-guide-figure-1002]:./media/virtual-machines-shared-sap-high-availability-guide/1002-wsfc-sios-on-azure-ilb.png
[sap-ha-guide-figure-2000]:./media/virtual-machines-shared-sap-high-availability-guide/2000-wsfc-sap-as-ha-on-azure.png
[sap-ha-guide-figure-2001]:./media/virtual-machines-shared-sap-high-availability-guide/2001-wsfc-sap-ascs-ha-on-azure.png
[sap-ha-guide-figure-2003]:./media/virtual-machines-shared-sap-high-availability-guide/2003-wsfc-sap-dbms-ha-on-azure.png
[sap-ha-guide-figure-2004]:./media/virtual-machines-shared-sap-high-availability-guide/2004-wsfc-sap-ha-e2e-archit-template1-on-azure.png
[sap-ha-guide-figure-2005]:./media/virtual-machines-shared-sap-high-availability-guide/2005-wsfc-sap-ha-e2e-arch-template2-on-azure.png

[sap-ha-guide-figure-3000]:./media/virtual-machines-shared-sap-high-availability-guide/3000-template-parameters-sap-ha-arm-on-azure.png
[sap-ha-guide-figure-3001]:./media/virtual-machines-shared-sap-high-availability-guide/3001-configuring-dns-servers-for-Azure-vnet.png
[sap-ha-guide-figure-3002]:./media/virtual-machines-shared-sap-high-availability-guide/3002-configuring-static-IP-address-for-network-card-of-each-vm.png
[sap-ha-guide-figure-3003]:./media/virtual-machines-shared-sap-high-availability-guide/3003-setup-static-ip-address-ilb-for-ascs-instance.png
[sap-ha-guide-figure-3004]:./media/virtual-machines-shared-sap-high-availability-guide/3004-default-ascs-scs-ilb-balancing-rules-for-azure-ilb.png
[sap-ha-guide-figure-3005]:./media/virtual-machines-shared-sap-high-availability-guide/3005-changing-ascs-scs-default-ilb-rules-for-azure-ilb.png
[sap-ha-guide-figure-3006]:./media/virtual-machines-shared-sap-high-availability-guide/3006-adding-vm-to-domain.png
[sap-ha-guide-figure-3007]:./media/virtual-machines-shared-sap-high-availability-guide/3007-config-wsfc-1.png
[sap-ha-guide-figure-3008]:./media/virtual-machines-shared-sap-high-availability-guide/3008-config-wsfc-2.png
[sap-ha-guide-figure-3009]:./media/virtual-machines-shared-sap-high-availability-guide/3009-config-wsfc-3.png
[sap-ha-guide-figure-3010]:./media/virtual-machines-shared-sap-high-availability-guide/3010-config-wsfc-4.png
[sap-ha-guide-figure-3011]:./media/virtual-machines-shared-sap-high-availability-guide/3011-config-wsfc-5.png
[sap-ha-guide-figure-3012]:./media/virtual-machines-shared-sap-high-availability-guide/3012-config-wsfc-6.png
[sap-ha-guide-figure-3013]:./media/virtual-machines-shared-sap-high-availability-guide/3013-config-wsfc-7.png
[sap-ha-guide-figure-3014]:./media/virtual-machines-shared-sap-high-availability-guide/3014-config-wsfc-8.png
[sap-ha-guide-figure-3015]:./media/virtual-machines-shared-sap-high-availability-guide/3015-config-wsfc-9.png
[sap-ha-guide-figure-3016]:./media/virtual-machines-shared-sap-high-availability-guide/3016-config-wsfc-10.png
[sap-ha-guide-figure-3017]:./media/virtual-machines-shared-sap-high-availability-guide/3017-config-wsfc-11.png
[sap-ha-guide-figure-3018]:./media/virtual-machines-shared-sap-high-availability-guide/3018-config-wsfc-12.png
[sap-ha-guide-figure-3019]:./media/virtual-machines-shared-sap-high-availability-guide/3019-assign-permissions-on-share-for-cluster-name-object.png
[sap-ha-guide-figure-3020]:./media/virtual-machines-shared-sap-high-availability-guide/3020-change-object-type-include-computer-objects.png
[sap-ha-guide-figure-3021]:./media/virtual-machines-shared-sap-high-availability-guide/3021-check-box-for-computer-objects.png
[sap-ha-guide-figure-3022]:./media/virtual-machines-shared-sap-high-availability-guide/3022-set-security-attributes-for-cluster-name-object-on-file-share-quorum.png
[sap-ha-guide-figure-3023]:./media/virtual-machines-shared-sap-high-availability-guide/3023-call-configure-cluster-quorum-setting-wizard.png
[sap-ha-guide-figure-3024]:./media/virtual-machines-shared-sap-high-availability-guide/3024-selection-screen-different-quorum-configurations.png
[sap-ha-guide-figure-3025]:./media/virtual-machines-shared-sap-high-availability-guide/3025-selection-screen-file-share-witness.png
[sap-ha-guide-figure-3026]:./media/virtual-machines-shared-sap-high-availability-guide/3026-define-file-share-location-for-witness-share.png
[sap-ha-guide-figure-3027]:./media/virtual-machines-shared-sap-high-availability-guide/3027-successful-reconfiguration-cluster-file-share-witness.png
[sap-ha-guide-figure-3028]:./media/virtual-machines-shared-sap-high-availability-guide/3028-install-dot-net-framework-35.png
[sap-ha-guide-figure-3029]:./media/virtual-machines-shared-sap-high-availability-guide/3029-install-dot-net-framework-35-progress.png
[sap-ha-guide-figure-3030]:./media/virtual-machines-shared-sap-high-availability-guide/3030-sios-installer.png
[sap-ha-guide-figure-3031]:./media/virtual-machines-shared-sap-high-availability-guide/3031-first-screen-sios-data-keeper-installation.png
[sap-ha-guide-figure-3032]:./media/virtual-machines-shared-sap-high-availability-guide/3032-data-keeper-informs-service-be-disabled.png
[sap-ha-guide-figure-3033]:./media/virtual-machines-shared-sap-high-availability-guide/3033-user-selection-sios-data-keeper.png
[sap-ha-guide-figure-3034]:./media/virtual-machines-shared-sap-high-availability-guide/3034-domain-user-sios-data-keeper.png
[sap-ha-guide-figure-3035]:./media/virtual-machines-shared-sap-high-availability-guide/3035-provide-sios-data-keeper-license.png
[sap-ha-guide-figure-3036]:./media/virtual-machines-shared-sap-high-availability-guide/3036-data-keeper-management-config-tool.png
[sap-ha-guide-figure-3037]:./media/virtual-machines-shared-sap-high-availability-guide/3037-tcp-ip-address-first-node-data-keeper.png
[sap-ha-guide-figure-3038]:./media/virtual-machines-shared-sap-high-availability-guide/3038-create-replication-sios-job.png
[sap-ha-guide-figure-3039]:./media/virtual-machines-shared-sap-high-availability-guide/3039-define-sios-replication-job-name.png
[sap-ha-guide-figure-3040]:./media/virtual-machines-shared-sap-high-availability-guide/3040-define-sios-source-node.png
[sap-ha-guide-figure-3041]:./media/virtual-machines-shared-sap-high-availability-guide/3041-define-sios-target-node.png
[sap-ha-guide-figure-3042]:./media/virtual-machines-shared-sap-high-availability-guide/3042-define-sios-synchronous-replication.png
[sap-ha-guide-figure-3043]:./media/virtual-machines-shared-sap-high-availability-guide/3043-enable-sios-replicated-volume-as-cluster-volume.png
[sap-ha-guide-figure-3044]:./media/virtual-machines-shared-sap-high-availability-guide/3044-data-keeper-synchronous-mirroring-for-SAP-gui.png
[sap-ha-guide-figure-3045]:./media/virtual-machines-shared-sap-high-availability-guide/3045-replicated-disk-by-data-keeper-in-wsfc.png
[sap-ha-guide-figure-3046]:./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ascs-virtual-name-ip.png
[sap-ha-guide-figure-3047]:./media/virtual-machines-shared-sap-high-availability-guide/3047-dns-manager.png
[sap-ha-guide-figure-3048]:./media/virtual-machines-shared-sap-high-availability-guide/3048-default-cluster-probe-port.png
[sap-ha-guide-figure-3049]:./media/virtual-machines-shared-sap-high-availability-guide/3049-cluster-probe-port-after.png
[sap-ha-guide-figure-3050]:./media/virtual-machines-shared-sap-high-availability-guide/3050-service-type-ers-delayed-automatic.png
[sap-ha-guide-figure-5000]:./media/virtual-machines-shared-sap-high-availability-guide/5000-wsfc-sap-sid-node-a.png
[sap-ha-guide-figure-5001]:./media/virtual-machines-shared-sap-high-availability-guide/5001-sios-replicating-local-volume.png
[sap-ha-guide-figure-5002]:./media/virtual-machines-shared-sap-high-availability-guide/5002-wsfc-sap-sid-node-b.png
[sap-ha-guide-figure-5003]:./media/virtual-machines-shared-sap-high-availability-guide/5003-sios-replicating-local-volume-b-to-a.png

[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png


[sap-ha-guide-figure-8001]:./media/virtual-machines-shared-sap-high-availability-guide/8001.png
[sap-ha-guide-figure-8002]:./media/virtual-machines-shared-sap-high-availability-guide/8002.png
[sap-ha-guide-figure-8003]:./media/virtual-machines-shared-sap-high-availability-guide/8003.png
[sap-ha-guide-figure-8004]:./media/virtual-machines-shared-sap-high-availability-guide/8004.png
[sap-ha-guide-figure-8005]:./media/virtual-machines-shared-sap-high-availability-guide/8005.png
[sap-ha-guide-figure-8006]:./media/virtual-machines-shared-sap-high-availability-guide/8006.png
[sap-ha-guide-figure-8007]:./media/virtual-machines-shared-sap-high-availability-guide/8007.png
[sap-ha-guide-figure-8008]:./media/virtual-machines-shared-sap-high-availability-guide/8008.png
[sap-ha-guide-figure-8009]:./media/virtual-machines-shared-sap-high-availability-guide/8009.png
[sap-ha-guide-figure-8010]:./media/virtual-machines-shared-sap-high-availability-guide/8010.png
[sap-ha-guide-figure-8011]:./media/virtual-machines-shared-sap-high-availability-guide/8011.png
[sap-ha-guide-figure-8012]:./media/virtual-machines-shared-sap-high-availability-guide/8012.png
[sap-ha-guide-figure-8013]:./media/virtual-machines-shared-sap-high-availability-guide/8013.png
[sap-ha-guide-figure-8014]:./media/virtual-machines-shared-sap-high-availability-guide/8014.png
[sap-ha-guide-figure-8015]:./media/virtual-machines-shared-sap-high-availability-guide/8015.png
[sap-ha-guide-figure-8016]:./media/virtual-machines-shared-sap-high-availability-guide/8016.png
[sap-ha-guide-figure-8017]:./media/virtual-machines-shared-sap-high-availability-guide/8017.png
[sap-ha-guide-figure-8018]:./media/virtual-machines-shared-sap-high-availability-guide/8018.png
[sap-ha-guide-figure-8019]:./media/virtual-machines-shared-sap-high-availability-guide/8019.png
[sap-ha-guide-figure-8020]:./media/virtual-machines-shared-sap-high-availability-guide/8020.png
[sap-ha-guide-figure-8021]:./media/virtual-machines-shared-sap-high-availability-guide/8021.png
[sap-ha-guide-figure-8022]:./media/virtual-machines-shared-sap-high-availability-guide/8022.png
[sap-ha-guide-figure-8023]:./media/virtual-machines-shared-sap-high-availability-guide/8023.png
[sap-ha-guide-figure-8024]:./media/virtual-machines-shared-sap-high-availability-guide/8024.png
[sap-ha-guide-figure-8025]:./media/virtual-machines-shared-sap-high-availability-guide/8025.png


[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md

[1869038]:https://launchpad.support.sap.com/#/notes/1869038 
