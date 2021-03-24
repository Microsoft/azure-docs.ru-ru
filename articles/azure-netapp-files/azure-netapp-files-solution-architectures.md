---
title: Архитектуры решений с использованием Azure NetApp Files | Документация Майкрософт
description: Содержит ссылки на рекомендации по архитектуре решений с помощью Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/19/2021
ms.author: b-juche
ms.openlocfilehash: abbae7f8cb047f2f2ad3d998966bc00b02a002d4
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864029"
---
# <a name="solution-architectures-using-azure-netapp-files"></a>Архитектура решений, использующих Azure NetApp Files
В этой статье содержатся ссылки на рекомендации, которые помогут вам понять архитектуру решения для использования Azure NetApp Files.  

На следующей диаграмме перечислены категории архитектур решений, которые Azure NetApp Files предлагает:

![Категории архитектуры решения](../media/azure-netapp-files/solution-architecture-categories.png)

## <a name="linux-oss-apps-and-database-solutions"></a>ОС Linux и решения для баз данных

В этом разделе содержатся ссылки на решения для приложений и баз данных ОС Linux. 

### <a name="oracle"></a>Oracle;

* [Рекомендации по развертыванию Oracle в Azure с помощью Azure NetApp Files](https://www.netapp.com/us/media/tr-4780.pdf)
* [Образы виртуальных машин Oracle и их развертывание на Microsoft Azure: параметры конфигурации общего хранилища](../virtual-machines/workloads/oracle/oracle-vm-solutions.md#shared-storage-configuration-options)
* [Производительность базы данных Oracle в отдельных томах Azure NetApp Files](performance-oracle-single-volumes.md)
* [Преимущества использования Azure NetApp Files с Oracle Database](solutions-benefits-azure-netapp-files-oracle-database.md)

### <a name="machine-learning"></a>Машинное обучение
*   [Cloudera Машинное обучение](https://docs.cloudera.com/machine-learning/cloud/requirements-azure/topics/ml-requirements-azure.html)

## <a name="windows-apps-and-sql-server-solutions"></a>Приложения Windows и решения SQL Server

В этом разделе содержатся ссылки на приложения Windows и SQL Server решения.

### <a name="file-sharing-and-global-file-caching"></a>Общий доступ к файлам и глобальное кэширование файлов

* [Создаете собственную службу Azure NFS? Решение файловые ресурсы Linux в облако](https://cloud.netapp.com/blog/ma-anf-blg-build-your-own-linux-nfs-file-shares)
* [Развертывание глобального кэша файлов/Azure NetApp Files](https://youtu.be/91LKb1qsLIM)
* [Соответствие требованиям облака для Azure NetApp Files](https://cloud.netapp.com/hubfs/Cloud%20Compliance%20for%20Azure%20NetApp%20Files%20-%20November%202020.pdf)

### <a name="sql-server"></a>SQL Server

* [Развертывание SQL Server через SMB с Azure NetApp Files](https://www.youtube.com/watch?v=x7udfcYbibs)
* [Развертывание SQL Server Always-On отказоустойчивого кластера по протоколу SMB с Azure NetApp Files](https://www.youtube.com/watch?v=zuNJ5E07e8Q) 
* [Развертывание Always-On групп доступности с помощью Azure NetApp Files](https://www.youtube.com/watch?v=y3VQmzzeyvc) 
* [Преимущества использования Azure NetApp Files для развертывания SQL Server](solutions-benefits-azure-netapp-files-sql-server.md)

## <a name="sap-on-azure-solutions"></a>Решения SAP в Azure

В этом разделе содержатся ссылки на решения SAP в Azure. 

### <a name="generic-sap-and-sap-netweaver"></a>Универсальные SAP и SAP NetWeaver 

* [Приложения SAP на Microsoft Azure с помощью Azure NetApp Files](https://www.netapp.com/us/media/tr-4746.pdf)
* [Обеспечение высокого уровня доступности виртуальных машин Azure для SAP NetWeaver на SUSE Linux Enterprise Server с помощью Azure NetApp Files для приложений SAP](../virtual-machines/workloads/sap/high-availability-guide-suse-netapp-files.md)
* [Высокий уровень доступности SAP NetWeaver на виртуальных машинах Azure на Red Hat Enterprise Linux с Azure NetApp Files для приложений SAP](../virtual-machines/workloads/sap/high-availability-guide-rhel-netapp-files.md)
* [Высокий уровень доступности SAP NetWeaver на виртуальных машинах Azure в Windows с Azure NetApp Files (SMB) для приложений SAP](../virtual-machines/workloads/sap/high-availability-guide-windows-netapp-files-smb.md)
* [Высокий уровень доступности SAP NetWeaver на виртуальных машинах Azure на Red Hat Enterprise Linux для многоид безопасности приложений SAP](../virtual-machines/workloads/sap/high-availability-guide-rhel-multi-sid.md)

### <a name="sap-hana"></a>SAP HANA 

* [Конфигурации хранилища виртуальных машин SAP HANA в Azure](../virtual-machines/workloads/sap/hana-vm-operations-storage.md)
* [Тома NFS версии 4.1 в Azure NetApp Files для SAP HANA](../virtual-machines/workloads/sap/hana-vm-operations-netapp.md)
* [Высокий уровень доступности SAP HANA с Azure NetApp Files на Red Hat Enterprise Linux](../virtual-machines/workloads/sap/sap-hana-high-availability-netapp-files-red-hat.md)
* [SAP HANA горизонтальное масштабирование с помощью резервного узла на виртуальных машинах Azure с Azure NetApp Files на SUSE Linux Enterprise Server](../virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-suse.md)
* [SAP HANA горизонтальное масштабирование с помощью резервного узла на виртуальных машинах Azure с Azure NetApp Files на Red Hat Enterprise Linux](../virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-rhel.md)
* [SAP HANA горизонтального масштабирования с помощью HSR и Pacemaker на RHEL виртуальных машинах Azure](../virtual-machines/workloads/sap/sap-hana-high-availability-scale-out-hsr-rhel.md)
* [Средство создания моментальных снимков с единообразием приложений Azure (Азакснап)](azacsnap-introduction.md)

### <a name="sap-anydb"></a>Сертифицированную SAP

* [Развертывание СУБД Oracle виртуальные машины Azure для рабочей нагрузки SAP — виртуальные машины Azure](../virtual-machines/workloads/sap/dbms_guide_oracle.md#oracle-configuration-guidelines-for-sap-installations-in-azure-vms-on-linux)
* [Развертывание SAP сертифицированную (Oracle 19c) с Azure NetApp Files](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/deploy-sap-anydb-oracle-19c-with-azure-netapp-files/ba-p/2064043)

### <a name="sap-iq-nls"></a>SAP IQ — NLS

*   [Развертывание решения с высоким уровнем доступности SAP IQ с помощью Azure NetApp Files на SUSE Linux Enterprise Server](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/deploy-sap-iq-nls-ha-solution-using-azure-netapp-files-on-suse/ba-p/1651172#.X2tDfpNzBh4.linkedin)
* [Как управлять лицензией SAP IQ в сценарии высокой доступности](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/how-to-manage-sap-iq-license-in-ha-scenario/ba-p/2052583)

### <a name="sap-tech-community-and-blog-posts"></a>Технические сообщества SAP и записи в блоге 

* [Azure NetApp Files — резервное копирование SAP HANA в секундах](https://blog.netapp.com/azure-netapp-files-sap-hana-backup-in-seconds/)
* [Azure NetApp Files — восстановление базы данных HANA из резервной копии моментальных снимков](https://blog.netapp.com/azure-netapp-files-backup-sap-hana)
* [Azure NetApp Files — SAP HANA разгрузка резервной копии с синхронизацией в облаке](https://blog.netapp.com/azure-netapp-files-sap-hana)
* [Ускорение копирования SAP HANA системы с помощью Azure NetApp Files](https://blog.netapp.com/sap-hana-faster-using-azure-netapp-files/)
* [Облачные тома ONTAP и Azure NetApp Files: упрощена миграция SAP HANA системы](https://blog.netapp.com/cloud-volumes-ontap-and-azure-netapp-files-sap-hana-system-migration-made-easy/)
* [Архитектурные решения для максимизации использовании инвестиций в HANA N + M Scale-Out архитектуру. часть 1](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/architectural-decisions-to-maximize-anf-investment-in-hana-n-m/ba-p/2078737)
* [Архитектурные решения для максимизации использовании инвестиций в HANA N + M Scale-Out архитектура. часть 2](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/architectural-decisions-to-maximize-anf-investment-in-hana-n-m/ba-p/2117130)

## <a name="azure-vmware-solutions"></a>Решения Azure VMware

* [Azure NetApp Files с помощью решения VMware для Azure. Подключение гостевых ОС](../azure-vmware/netapp-files-with-azure-vmware-solution.md)

## <a name="virtual-desktop-infrastructure-solutions"></a>Решения инфраструктуры виртуальных рабочих столов

В этом разделе содержатся ссылки на решения инфраструктуры виртуальных рабочих столов.

### <a name="windows-virtual-desktop"></a>Виртуальный рабочий стол Windows

* [Преимущества использования Azure NetApp Files с Виртуальным рабочим столом Windows](solutions-windows-virtual-desktop.md)
* [Варианты хранения для контейнеров профилей Фслогикс в виртуальном рабочем столе Windows](../virtual-desktop/store-fslogix-profile.md#azure-platform-details)
* [Создание контейнера профиля Фслогикс для пула узлов с помощью Azure NetApp Files](../virtual-desktop/create-fslogix-profile-container.md)
* [Виртуальный рабочий стол Windows в масштабе предприятия](/azure/architecture/example-scenario/wvd/windows-virtual-desktop)
* [Рекомендации Microsoft Фслогикс для корпоративного Azure NetApp Files](/azure/architecture/example-scenario/wvd/windows-virtual-desktop-fslogix#azure-netapp-files-best-practices)
* [Настройка Azure NetApp Files для подключения к приложению MSIX](https://techcommunity.microsoft.com/t5/windows-virtual-desktop/setting-up-azure-netapp-files-for-msix-app-attach-step-by-step/m-p/1990021)

## <a name="hpc-solutions"></a>Высокопроизводительные решения

В этом разделе содержатся ссылки на решения для высокопроизводительных вычислений (HPC). 

### <a name="generic-hpc"></a>Универсальный HPC

* [Azure NetApp Files: максимально эффективное получение из облачного хранилища](https://cloud.netapp.com/hubfs/Resources/ANF%20PERFORMANCE%20TESTING%20IN%20TEMPLATE.pdf)
* [Выполнение рабочих нагрузок MPI с помощью пакетной службы Azure и Azure NetApp Files](https://azure.microsoft.com/resources/run-mpi-workloads-with-azure-batch-and-azure-netapp-files/)
* [Циклическое облако Azure: среды Циклеклауд HPC на Azure NetApp Files](/azure/cyclecloud/overview)

### <a name="oil-and-gas"></a>нефть и газ;

* [Высокопроизводительные вычислительные системы (HPC): топливо и газ в Azure](https://techcommunity.microsoft.com/t5/azure-global/high-performance-computing-hpc-oil-and-gas-in-azure/ba-p/824926)
* [Запуск программного обеспечения моделирования молекулярного в Azure](/azure/architecture/example-scenario/infrastructure/reservoir-simulation)

### <a name="electronic-design-automation-eda"></a>Автоматизация разработки электронных макетов (EDA)

* [Преимущества использования Azure NetApp Files для автоматизации работы с электронными макетами](solutions-benefits-azure-netapp-files-electronic-design-automation.md)
* [Azure Циклеклауд: EDA HPC Lab с Azure NetApp Files](https://github.com/Azure/cyclecloud-hands-on-labs/blob/master/EDA/README.md)
* [Azure для полупроводниковой индустрии](https://azurecomcdn.azureedge.net/cvt-f40f39cd9de2d875ab0c198a8d7b186350cf0bca161e80d7896941389685d012/mediahandler/files/resourcefiles/azure-for-the-semiconductor-industry/Azure_for_the_Semiconductor_Industry.pdf)

### <a name="analytics"></a>Analytics

* [Azure NetApp Files: Общая файловая система, используемая в сетке SAS на Microsoft Azure](https://communities.sas.com/t5/Administration-and-Deployment/Azure-NetApp-Files-A-shared-file-system-to-use-with-SAS-Grid-on/m-p/705192)
* [Azure NetApp Files: Общая файловая система для использования с сеткой SAS в MS Azure — RHEL 8.3/нконнект UPDATE](https://communities.sas.com/t5/Administration-and-Deployment/Azure-NetApp-Files-A-shared-file-system-to-use-with-SAS-Grid-on/m-p/722261#M21648)
* [Рекомендации по использованию Microsoft Azure с SAS®](https://communities.sas.com/t5/Administration-and-Deployment/Best-Practices-for-Using-Microsoft-Azure-with-SAS/m-p/676833#M19680)

## <a name="azure-platform-services-solutions"></a>Решения для служб платформы Azure

В этом разделе приводятся решения для служб платформы Azure. 

### <a name="azure-kubernetes-services-and-kubernetes"></a>Azure Kubernetes Services и Kubernetes

* [Интеграция Azure NetApp Files со службой Kubernetes Azure](../aks/azure-netapp-files.md)
* [Kubernetes производительность в Azure с помощью Azure NetApp Files](https://cloud.netapp.com/blog/ma-anf-blg-configure-kubernetes-openshift)
* [Azure NetApp Files + Trident = динамическое и постоянное хранилище для Kubernetes](https://anfcommunity.com/2021/02/16/azure-netapp-files-trident-dynamic-and-persistent-storage-for-kubernetes/)
* [Trident — Orchestrator Storage для контейнеров](https://netapp-trident.readthedocs.io/en/stable-v20.04/kubernetes/operations/tasks/backends/anf.html)
* [Платформа электронной коммерции Magento в службе Kubernetes Azure (AKS)](/azure/architecture/example-scenario/magento/magento-azure)

### <a name="azure-batch"></a>Пакетная служба Azure

* [Выполнение рабочих нагрузок MPI с помощью пакетной службы Azure и Azure NetApp Files](https://azure.microsoft.com/resources/run-mpi-workloads-with-azure-batch-and-azure-netapp-files/)
