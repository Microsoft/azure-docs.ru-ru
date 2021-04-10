---
title: Определение размера SAP HANA в Azure (крупные экземпляры) | Документация Майкрософт
description: Определение размера SAP HANA в Azure (крупные экземпляры).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/04/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d49191abbba9c189672be4cd8bad4346e9689bf6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101675422"
---
# <a name="sizing"></a>Определение размера

В целом определение размера для крупных экземпляров HANA ничем не отличается от определения размера для HANA. Для существующих и развернутых систем необходимо перенести данные из других реляционных СУБД в HANA, так как SAP предоставляет ряд отчетов, которые выполняются в имеющихся системах SAP. Эти отчеты проверяют данные и определяют требования к памяти при перемещении базы данных в HANA. Прочтите следующие примечания SAP, чтобы получить дополнительные сведения о выполнении этих отчетов и информацию о последних исправлениях и версиях этих отчетов:

- [SAP Note #1793345 — Sizing for SAP Suite on HANA (Примечание SAP № 1793345. Определение размера для SAP Suite в HANA)](https://launchpad.support.sap.com/#/notes/1793345);
- [SAP Note #1872170 — Suite on HANA and S/4 HANA sizing report (Примечание SAP № 1872170. Отчет для определения размера Suite в HANA и S/4 HANA)](https://launchpad.support.sap.com/#/notes/1872170);
- [SAP Note #2121330 — FAQ: SAP BW on HANA Sizing Report](https://launchpad.support.sap.com/#/notes/2121330) (Примечание SAP № 2121330. Часто задаваемые вопросы: отчет для определения размера SAP BW в HANA);
- [SAP Note #1736976 — Sizing Report for BW on HANA](https://launchpad.support.sap.com/#/notes/1736976) (Примечание SAP № 1736976. Отчет для определения размера BW в HANA);
- [SAP Note #2296290 — New Sizing Report for BW on HANA](https://launchpad.support.sap.com/#/notes/2296290) (Примечание SAP № 2296290. Новый отчет для определения размера BW в HANA).

Для новых развертываний доступно средство для определения требований к памяти для реализации программного обеспечения SAP на базе HANA — SAP Quick Sizer.

Требования к памяти для HANA повышаются по мере роста объема данных. Вам нужно следить за потреблением памяти, чтобы спрогнозировать будущие требования. В зависимости от требований к памяти вы можете определить подходящий SKU для крупных экземпляров HANA.

**Дальнейшие действия**
- См. раздел [Onboarding requirements](hana-onboarding-requirements.md) (Требования для внедрения)