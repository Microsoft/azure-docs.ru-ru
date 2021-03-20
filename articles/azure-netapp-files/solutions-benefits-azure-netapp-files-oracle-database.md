---
title: Преимущества использования Azure NetApp Files с Oracle Database | Документация Майкрософт
description: Описывает технологию и обеспечивает сравнение производительности между Oracle Direct NFS (Днфс) и традиционным клиентом NFS. Показывает преимущества использования Днфс с Azure NetApp Files.
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
ms.date: 04/23/2020
ms.author: b-juche
ms.openlocfilehash: 0f431015caf5bb3364fe5628a599f760c6318c47
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91932504"
---
# <a name="benefits-of-using-azure-netapp-files-with-oracle-database"></a>Преимущества использования Azure NetApp Files с Oracle Database

Oracle Direct NFS (Днфс) делает возможным более высокую производительность, чем драйвер NFS операционной системы. В этой статье описывается технология и приводится сравнение производительности между Днфс и традиционным клиентом NFS (ядро NFS). Он также показывает преимущества и простоту использования Днфс с Azure NetApp Files.  

## <a name="how-oracle-direct-nfs-works"></a>Как работает Direct NFS Oracle

В следующей сводке объясняется, как работает Direct NFS Oracle на высоком уровне:

* Oracle Direct NFS обходит буферный кэш операционной системы. Данные кэшируются в пространстве пользователя только один раз, устраняя нагрузку на копии памяти.  

* Традиционный клиент NFS использует один сетевой поток, как показано ниже:    

    ![Клиент традиционной NFS, использующий один сетевой поток](../media/azure-netapp-files/solutions-traditional-nfs-client-using-single-network-flow.png)

    Oracle Direct NFS улучшает производительность за счет балансировки нагрузки сетевого трафика в нескольких сетевых потоках. Как протестировано и показано ниже, 650 различающиеся сетевые подключения динамически устанавливаются Oracle Database:  

    ![Oracle Direct NFS улучшает производительность](../media/azure-netapp-files/solutions-oracle-direct-nfs-performance-load-balancing.png)

[Вопросы и ответы по Oracle для Direct NFS](http://www.orafaq.com/wiki/Direct_NFS) показывают, что Oracle днфс является оптимизированным клиентом NFS. Он обеспечивает быстрый и масштабируемый доступ к хранилищу NFS, расположенному на устройствах хранения NAS (доступных по протоколу TCP/IP). Днфс встроена в ядро базы данных, как ASM, который используется главным образом с DAS или хранилищем SAN. Поэтому *рекомендуется использовать днфс при реализации хранилища NAS и использовать ASM при реализации хранилища SAN.*

Днфс является параметром по умолчанию в Oracle 18C.

Днфс доступен, начиная с Oracle Database 11g. На схеме ниже сравниваются Днфс с собственными NFS. При использовании Днфс база данных Oracle, которая работает на виртуальной машине Azure, может выполнять больше операций ввода-вывода, чем собственный клиент NFS.

![Сравнение Днфс с Oracle и Azure NetApp Files с помощью встроенной NFS](../media/azure-netapp-files/solutions-oracle-azure-netapp-files-comparing-dnfs-native-nfs.png)

Вы можете включить или отключить Днфс, выполнив две команды и перезапустив базу данных.

Чтобы включить:  
`cd $ORACLE_HOME/rdbms/lib ; make -f ins_rdbms.mk dnfs_on`

Для отключения:  
`cd $ORACLE_HOME/rdbms/lib ; make -f ins_rdbms.mk dnfs_off`

## <a name="azure-netapp-files-combined-with-oracle-direct-nfs"></a>Azure NetApp Files в сочетании с Direct NFS Oracle

Можно повысить производительность Oracle Днфс со службой Azure NetApp Files. Служба предоставляет полный контроль над производительностью приложения. Он может отвечать очень требовательным приложениям. Сочетание Oracle Днфс с Azure NetApp Files предоставляет большие преимущества для рабочих нагрузок.

## <a name="next-steps"></a>Дальнейшие действия

- [Архитектура решений, использующих Azure NetApp Files](azure-netapp-files-solution-architectures.md)
- [Обзор приложений и решений Oracle в Azure](../virtual-machines/workloads/oracle/oracle-overview.md)