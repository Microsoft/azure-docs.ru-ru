---
title: Сертификация Microsoft Azure для SAP | Документация Майкрософт
description: Обновленный список текущих конфигураций и сертификатов SAP на платформе Azure.
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/21/2020
ms.author: juergent
ms.custom: ''
ms.openlocfilehash: 76f67204b41a36d10541974e78a22f7d065e144b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101669700"
---
# <a name="sap-certifications-and-configurations-running-on-microsoft-azure"></a>Сертификаты и конфигурации SAP на платформе Microsoft Azure

Компания SAP и корпорация Майкрософт имеют долгую историю плодотворного сотрудничества, которое обеспечивает взаимные преимущества для их клиентов. Корпорация Майкрософт регулярно обновляет свою платформу и отправляет компании SAP сведения о новых сертификатах. Тем самым гарантируется, что Microsoft Azure остается лучшей платформой для выполнения рабочих нагрузок SAP. В следующих таблицах представлены поддерживаемые конфигурации Azure и список сертификатов SAP, который регулярно пополняется. Этот список содержит обзор, который может отклоняться от официальных списков SAP. Сведения о том, как получить подробные данные, описаны в статье [что такое программное обеспечение SAP поддерживается для развертываний Azure](./sap-supported-product-on-azure.md) .

## <a name="sap-hana-certifications"></a>Сертификаты SAP HANA
Справочные материалы:

- Сведения о поддержке SAP HANA для собственных виртуальных машин Azure и SAP HANA (крупные экземпляры) приведены в [списке сертифицированных платформ IaaS для SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure).

| Продукт SAP | Поддерживаемая ОС | Предложения Azure |
| --- | --- | --- |
| SAP HANA, версия для разработчиков (включая клиентское ПО HANA с драйверами SQLODBC, ODBO (только для ОС Windows), ODBC и JDBC, среду разработки HANA Studio и базу данных HANA) | Red Hat Enterprise Linux, SUSE Linux Enterprise | Семейство виртуальных машин серии D |
| Business One в HANA | SUSE Linux Enterprise | DS14_v2, M32ts, M32ls, M64ls, M64s <br /> [SAP HANA сертифицированные платформы IaaS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure%23SAP%20Business%20One) |
| SAP S/4 HANA | Red Hat Enterprise Linux, SUSE Linux Enterprise | Управляемая доступность для GS5. Полная поддержка M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, M416s_v2, M416ms_v2, <br /> SAP HANA (крупные экземпляры) в Azure. [Платформы IaaS, сертифицированные для SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |
| Suite on HANA, OLTP | Red Hat Enterprise Linux, SUSE Linux Enterprise | M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, <br /> M416s_v2, M416ms_v2, SAP HANA в Azure (крупные экземпляры) [SAP HANA сертифицированные платформы IaaS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |
| HANA Enterprise для BW, OLAP | Red Hat Enterprise Linux, SUSE Linux Enterprise | GS5, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, <br /> M416s_v2, M416ms_v2, SAP HANA в Azure (крупные экземпляры) [SAP HANA сертифицированные платформы IaaS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |
| SAP BW/4 HANA | Red Hat Enterprise Linux, SUSE Linux Enterprise | GS5, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, <br /> M416s_v2, M416ms_v2, SAP HANA в Azure (крупные экземпляры) <br /> [SAP HANA сертифицированные платформы IaaS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |

Обратите внимание, что термин "кластеризация" на странице с [платформами IaaS, сертифицированными для SAP HAN](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure), используется как синоним слова "горизонтальное масштабирование". Здесь не имеется в виду кластеризация с высоким уровнем доступности.

## <a name="sap-netweaver-certifications"></a>Сертификация SAP NetWeaver
Платформа Microsoft Azure сертифицирована для следующих продуктов SAP при полной поддержке Майкрософт и SAP.
Справочные материалы:

- Список всех приложений на основе SAP NetWeaver, включая SAP TREX, SAP LiveCache и сервер содержимого SAP, приведен в [примечании SAP № 1928533 о поддерживаемых продуктах и типах виртуальных машин Azure](https://launchpad.support.sap.com/#/notes/1928533). Здесь также содержится список всех баз данных, кроме SAP HANA.


| Продукт SAP | Гостевая ОС | Реляционная СУБД | Типы виртуальных машин |
| --- | --- | --- | --- |
| ПО SAP Business Suite | Windows, SUSE Linux Enterprise, Red Hat Enterprise Linux, Oracle Linux |SQL Server, Oracle (только Windows и Oracle Linux), DB2, SAP ASE |От A5 до A11, D11 до D14, DS11 в DS14, DS11_v2ся к DS15_v2, GS1 до GS5, D2s_v3 D64s_v3, D2as_v4 D64as_v4, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, E2s_v3, E64s_v3, E2as_v4, E64as_v4 |
| SAP Business All-in-One | Windows, SUSE Linux Enterprise, Red Hat Enterprise Linux, Oracle Linux |SQL Server, Oracle (только Windows и Oracle Linux), DB2, SAP ASE |От A5 до A11, D11 до D14, DS11 в DS14, DS11_v2ся к DS15_v2, GS1 до GS5, D2s_v3 D64s_v3, D2as_v4 D64as_v4, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, E2s_v3, E64s_v3, E2as_v4, E64as_v4 |
| Бизнес-аналитика SAP BusinessObjects | Windows |Н/Д |От A5 до A11, D11 до D14, DS11 в DS14, DS11_v2ся к DS15_v2, GS1 до GS5, D2s_v3 D64s_v3, D2as_v4 D64as_v4, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, E2s_v3, E64s_v3, E2as_v4, E64as_v4 |
| SAP NetWeaver | Windows, SUSE Linux Enterprise, Red Hat Enterprise Linux, Oracle Linux |SQL Server, Oracle (только Windows и Oracle Linux), DB2, SAP ASE |От A5 до A11, D11 до D14, DS11 в DS14, DS11_v2ся к DS15_v2, GS1 до GS5, D2s_v3 D64s_v3, D2as_v4 D64as_v4, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, E2s_v3, E64s_v3, E2as_v4, E64as_v4 |

## <a name="other-sap-workload-supported-on-azure"></a>Другие рабочие нагрузки SAP, поддерживаемые в Azure

| Продукт SAP | Гостевая ОС | Реляционная СУБД | Типы виртуальных машин |
| --- | --- | --- | --- |
| SAP Business One на SQL Server | Windows  | SQL Server | Все типы виртуальных машин, сертифицированные NetWeaver<br /> [Примечание к SAP № 928839](https://launchpad.support.sap.com/#/notes/928839) |
| SAP BPC 10.01 MS SP08 | Windows и Linux | | Все типы виртуальных машин, сертифицированные NetWeaver<br /> Примечание к SAP № 2451795 |
| Платформа SAP BusinessObjects BI | Windows и Linux | | Примечание к SAP № 2145537 |
| SAP Data Services 4.2 | | | Примечание к SAP № 2288344 |
| Платформа SAP Hybris Commerce  | Windows | SQL Server, Oracle | Все типы виртуальных машин, сертифицированные NetWeaver <br /> [Документация по Hybris](https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/6.7.0.0/en-US/8c71300f866910149b40c88dfc0de431.html) |
| Платформа SAP Hybris Commerce  | SLES 12 или более поздней версии | SAP HANA | Все типы виртуальных машин, сертифицированные NetWeaver <br /> [Документация по Hybris](https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/6.7.0.0/en-US/8c71300f866910149b40c88dfc0de431.html) |
| Платформа SAP Hybris Commerce  | RHEL 7 или более поздней версии | SAP HANA | Все типы виртуальных машин, сертифицированные NetWeaver <br /> [Документация по Hybris]https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/6.7.0.0/en-US/8c71300f866910149b40c88dfc0de431.html) |
| SAP (Hybris) Commerce Platform 1811 и более поздних версий  | Windows, SLES или RHEL | SQL Azure DB | Все типы виртуальных машин, сертифицированные NetWeaver <br /> [Документация по Hybris](https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/1811/en-US/8c71300f866910149b40c88dfc0de431.html) |
