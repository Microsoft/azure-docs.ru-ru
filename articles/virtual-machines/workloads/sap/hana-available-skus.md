---
title: Номера SKU для SAP HANA в Azure (крупные экземпляры) | Документация Майкрософт
description: Номера SKU для SAP HANA в Azure (крупные экземпляры).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: juergent
editor: ''
keywords: ХЛИ, HANA, SKU, S896, S224, S448, S672, Оптане, SAP
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/21/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 25a11bf96ba680608e5bb22835becf80fadee4f3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101668934"
---
# <a name="available-skus-for-hana-large-instances"></a>Доступные номера SKU для крупных экземпляров HANA

SAP HANA в службе Azure (крупные экземпляры), основанной только на метках версии 3, доступна в нескольких конфигурациях в регионах Azure:

- Восточная Австралия
- Юго-Восточная часть Австралии
- Восточная Япония
- Западная Япония

SAP HANA в службе Azure (крупные экземпляры), основанной на метках версии 4, можно найти в нескольких конфигурациях в следующих регионах Azure:

- Западная часть США 2
- Восточная часть США

BareMetal инфраструктура (сертифицированная для рабочих нагрузок SAP HANA), основанная на метках редакции 4,2. Он доступен в нескольких конфигурациях в регионах Azure:
- Западная Европа
- Северная Европа
- восточная часть США 2
- Центрально-южная часть США




Список доступных больших экземпляров Azure, которые предлагают списки, как указано ниже.

> [!IMPORTANT]
> Обратите внимание на первый столбец, представляющий состояние сертификации HANA для каждого из типов крупных экземпляров в списке. Столбец должен сопоставляться с [SAP HANA каталогом оборудования](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) для номеров SKU Azure, начинающихся с буквы **S** .



| SAP HANA сертификации | Модель | Общий объем памяти | Память DRAM | Оптане памяти | Память | Доступность |
| --- | --- | --- | --- | --- | --- | --- |
| YES <br />[OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2185), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2265) | SAP HANA на виртуальных машинах Azure S96<br /> – 2 x Intel® Xeon® процессор E7-8890 v4 <br /> 48 ядер ЦП и 96 потоков ЦП |  768 ГБ | 768 ГБ | --- | 3 ТБ | Доступно |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2186), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2269) | SAP HANA в Azure S224<br /> — 4 процессора Intel® Xeon® Platinum 8276 <br /> 112 ядер ЦП и 224 ЦП |  3 ТБ | 3 ТБ | --- | 6,3 ТБ | Доступно |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2297) | SAP HANA в Azure S224m<br /> — 4 процессора Intel® Xeon® Platinum 8276 <br /> 112 ядер ЦП и 224 ЦП |  6 ТБ | 6 ТБ | --- | 10,5 ТБ | Доступно |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2381) | SAP HANA в Azure S224om<br /> — 4 процессора Intel® Xeon® Platinum 8276 <br /> 112 ядер ЦП и 224 ЦП | 6 ТБ |  3 ТБ |  3 ТБ | 10,5 ТБ | Доступно |
| NO | SAP HANA в Azure S224oo<br /> — 4 процессора Intel® Xeon® Platinum 8276 <br /> 112 ядер ЦП и 224 ЦП | 4,5 ТБ |  1,5 ТБ |  3 ТБ | 8,4 ТБ | Доступно |
| NO | SAP HANA в Azure S224ooo<br /> — 4 процессора Intel® Xeon® Platinum 8276 <br /> 112 ядер ЦП и 224 ЦП | 7,5 ТБ |  1,5 ТБ |  6 ТБ | 12,7 ТБ | Доступно |
| NO | SAP HANA в Azure S224oom<br /> — 4 процессора Intel® Xeon® Platinum 8276 <br /> 112 ядер ЦП и 224 ЦП | 9,0 ТБ |  3 ТБ |  6 ТБ | 14,8 ТБ | Доступно |
| YES <br />[OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1983), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2268) | SAP HANA на виртуальных машинах Azure S384<br /> — 8 процессоров Intel® Xeon® E7-8890 v4<br /> 192 ядра ЦП и 384 потока ЦП |  4 TБ | 4 TБ | --- | 16 ТБ | Доступно |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2080) | SAP HANA на виртуальных машинах Azure S384m<br /> — 8 процессоров Intel® Xeon® E7-8890 v4<br /> 192 ядра ЦП и 384 потока ЦП |  6 ТБ | 6 ТБ | --- | 18 ТБ |  Доступно  |
| YES <br />[OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1984), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2267) | SAP HANA на виртуальных машинах Azure S384xm<br /> — 8 процессоров Intel® Xeon® E7-8890 v4<br /> 192 ядра ЦП и 384 потока ЦП |  8 TБ | 8 TБ | --- | 22 ТБ | Доступно |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2411), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2378) | SAP HANA в Azure S448<br /> — 8 x процессоров Intel® Xeon® Platinum 8276 <br /> 224 ядер ЦП и 448 ЦП | 6 ТБ |  6 ТБ |  --- | 10,5 ТБ | Доступно (только версия 4) |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2410), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2377) | SAP HANA в Azure S448m<br /> — 8 x процессоров Intel® Xeon® Platinum 8276 <br /> 224 ядер ЦП и 448 ЦП | 12 ТБ |  12 ТБ |  --- | 18,9 ТБ | Доступно (только версия 4) |
| NO | SAP HANA в Azure S448oo<br /> — 8 x процессоров Intel® Xeon® Platinum 8276 <br /> 224 ядер ЦП и 448 ЦП | 9,0 ТБ |  3 ТБ |  6 ТБ | 14,8 ТБ  | Доступно (только версия 4) |
| NO | SAP HANA в Azure S448om<br /> — 8 x процессоров Intel® Xeon® Platinum 8276 <br /> 224 ядер ЦП и 448 ЦП | 12 ТБ |  6 ТБ |  6 ТБ | 18,9 ТБ  | Доступно (только версия 4) |
| NO | SAP HANA в Azure S448ooo<br /> — 8 x процессоров Intel® Xeon® Platinum 8276 <br /> 224 ядер ЦП и 448 ЦП | 15,0 ТБ |  3 ТБ |  12 ТБ | 23,2 ТБ  | Доступно (только версия 4) |
| NO | SAP HANA в Azure S448oom<br /> — 8 x процессоров Intel® Xeon® Platinum 8276 <br /> 224 ядер ЦП и 448 ЦП | 18,0 TБ |  6 ТБ |  12 ТБ | 27,4 ТБ  | Доступно (только версия 4) |
| YES <br /> [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2049) | SAP HANA на виртуальных машинах Azure S576m<br /> — 12 процессоров Intel® Xeon® E7-8890 v4<br /> 288 ядер ЦП и 576 потоков ЦП |  12 ТБ | 12 ТБ | --- | 28 ТБ | Доступно (только версия 4) |
| NO | SAP HANA на виртуальных машинах Azure S576xm<br /> — 12 процессоров Intel® Xeon® E7-8890 v4<br /> 288 ядер ЦП и 576 потоков ЦП |  18,0 TБ | 18.0 | --- |  41 TБ | Доступно |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2409), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2380) | SAP HANA в Azure S672<br /> – 12 x процессоров Intel® Xeon® Platinum 8276 <br /> 336 ядер ЦП и 672 ЦП | 9,0 ТБ |  9,0 ТБ |  --- | 14,7 ТБ | Доступно (только версия 4) |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2408), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2379) | SAP HANA в Azure S672m<br /> – 12 x процессоров Intel® Xeon® Platinum 8276 <br /> 336 ядер ЦП и 672 ЦП | 18,0 TБ |  18,0 TБ |  --- | 27,4 ТБ | Доступно (только версия 4) |
| NO | SAP HANA в Azure S672oo<br /> – 12 x процессоров Intel® Xeon® Platinum 8276 <br /> 336 ядер ЦП и 672 ЦП | 13,5 ТБ |  4,5 ТБ |  9,0 ТБ | 21,1 ТБ  | Доступно (только версия 4) |
| NO | SAP HANA в Azure S672om<br /> – 12 x процессоров Intel® Xeon® Platinum 8276 <br /> 336 ядер ЦП и 672 ЦП | 18,0 TБ |  9,0 ТБ |  9,0 ТБ | 27,4 ТБ  | Доступно (только версия 4) |
| NO | SAP HANA в Azure S672ooo<br /> – 12 x процессоров Intel® Xeon® Platinum 8276 <br /> 336 ядер ЦП и 672 ЦП | 22,5 ТБ |  4,5 ТБ |  18,0 TБ | 33,7 ТБ  | Доступно (только версия 4) |
| NO | SAP HANA в Azure S672oom<br /> – 12 x процессоров Intel® Xeon® Platinum 8276 <br /> 336 ядер ЦП и 672 ЦП | 27,0 ТБ |  9,0 ТБ |  18,0 TБ | 40,0 ТБ  | Доступно (только версия 4) |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1985) | SAP HANA на виртуальных машинах Azure S768m<br /> — 16 процессоров Intel® Xeon® E7-8890 v4<br /> 384 ядра ЦП и 768 потоков ЦП |  16 TБ | 16 TБ | -- | 36 ТБ | Доступно |
| NO | SAP HANA на виртуальных машинах Azure S768xm<br /> — 16 процессоров Intel® Xeon® E7-8890 v4<br /> 384 ядра ЦП и 768 потоков ЦП |  24,0 TБ | 24,0 TБ | --- | 56 ТБ | Доступно |
|  YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2407), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2376)  | SAP HANA в Azure S896<br /> – 16 x процессоров Intel® Xeon® Platinum 8276 <br /> 448 ядер ЦП и 896 ЦП | 12 ТБ |  12 ТБ |  --- | 18,9 ТБ | Доступно (только версия 4) |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2406), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2328) | SAP HANA в Azure S896m<br /> – 16 x процессоров Intel® Xeon® Platinum 8276 <br /> 448 ядер ЦП и 896 ЦП | 24,0 TБ | 24,0 TБ | -- | 35,8 ТБ | Доступно |
| NO | SAP HANA в Azure S896oo<br /> – 16 x процессоров Intel® Xeon® Platinum 8276 <br /> 448 ядер ЦП и 896 ЦП | 18,0 TБ |  6 ТБ |  12 ТБ | 27,4 ТБ  | Доступно (только версия 4) |
| NO | SAP HANA в Azure S896om<br /> – 16 x процессоров Intel® Xeon® Platinum 8276 <br /> 448 ядер ЦП и 896 ЦП | 24,0 TБ |  12 ТБ |  12 ТБ | 35,8 ТБ  | Доступно (только версия 4) |
| NO | SAP HANA в Azure S896ooo<br /> – 16 x процессоров Intel® Xeon® Platinum 8276 <br /> 448 ядер ЦП и 896 ЦП | 30,0 ТБ |  6 ТБ |  24,0 TБ | 44,3 ТБ  | Доступно (только версия 4) |
| NO | SAP HANA в Azure S896oom<br /> – 16 x процессоров Intel® Xeon® Platinum 8276 <br /> 448 ядер ЦП и 896 ЦП | 36,0 ТБ |  12 ТБ |  24,0 TБ | 52,7 ТБ  | Доступно (только версия 4) |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1986) | SAP HANA на виртуальных машинах Azure S960m<br /> — 20 процессоров Intel® Xeon® E7-8890 v4<br /> 480 ядер ЦП и 960 потоков ЦП |  20 ТБ | 20 ТБ | -- | 46 ТБ | Доступно (только версия 4) |


- Число ядер ЦП = сумма ядер ЦП без поддержки технологии Hyper-Threading из суммы процессоров экземпляра сервера.
- Число потоков ЦП = сумма потоков вычислений, предоставляемых ядрами ЦП с поддержкой технологии Hyper-Threading из суммы процессоров экземпляра сервера. По умолчанию большинство экземпляров настроены на использование технологии Hyper-Threading.
- В соответствии с рекомендациями поставщика, предложения S768m, S768xm и S960m не используют Hyper-Threading для работы с SAP HANA.


> [!IMPORTANT]
> Следующие SKU, хотя и еще поддерживаются, больше не могут быть приобретены: S72, S72m, S144, S144m, S192 и S192m 

Выбираемые конфигурации зависят от рабочей нагрузки, ресурсов ЦП и необходимого объема памяти. Для рабочих нагрузок OLTP можно использовать номера SKU, оптимизированные для рабочих нагрузок OLAP. 

Оборудование делится на два различных класса по номерам SKU:

- S72, S72m, S96, S144, S144m, S192, S192m, S192xm, S224 и S224m, S224oo, S224om, S224ooo, S224oom называются номерами SKU типа I.
- Все остальные номера SKU называются номерами SKU типа II.
- Если вас интересуют номера SKU, которые еще не указаны в каталоге оборудования SAP, обратитесь к группе учетная запись Майкрософт, чтобы получить дополнительные сведения. 


Полный стек крупных экземпляров HANA не выделяется монопольно для одного клиента. Это также относится к стойкам ресурсов вычислений и хранения, соединенным через сетевую структуру, развернутую в Azure. Инфраструктура крупных экземпляров HANA, такая как Azure, развертывает &quot;клиенты&quot; другого пользователя, которые изолированы друг от друга на следующих трех уровнях:

- **Сеть**: изоляция через виртуальные сети в рамках метки крупных экземпляров Hana.
- **Хранилище**: изоляция с помощью виртуальных машин хранилища, которые имеют назначенные тома хранилища и изолируют тома хранилища между клиентами.
- **Вычисление**: выделенное назначение серверных единиц для одного клиента. Нет секционирования оборудования или программного обеспечения для экземпляров сервера. Один экземпляр сервера или узла не может одновременно использоваться несколькими клиентами. 

Развертывания крупных экземпляров HANA на различных клиентах невидимы друг для друга. Крупные экземпляры HANA, развернутые в разных клиентах, не могут взаимодействовать напрямую друг с другом на уровне стека крупных экземпляров HANA. Взаимодействовать друг с другом на уровне стека крупных экземпляров HANA могут только крупные экземпляры HANA внутри одного клиента.

Развернутый клиент в стеке крупных экземпляров назначается одной подписке Azure для выставления счетов. Однако с точки зрения сети к нему можно получить доступ из виртуальных сетей других подписок Azure в рамках одной регистрации Azure. При развертывании в другой подписке Azure в том же регионе Azure вы также можете попросить отдельный клиент крупных экземпляров HANA.

Существуют значительные различия между запуском SAP HANA в крупных экземплярах HANA и на виртуальных машинах, развернутых в Azure:

- Для SAP HANA в Azure (крупные экземпляры) не предусмотрен уровень виртуализации. Вы получаете производительность базового физического оборудования.
- В отличие от Azure сервер решения "SAP HANA в Azure (крупные экземпляры)" выделяется для конкретного клиента. Исключена вероятность того, что на экземпляре сервера или узле будет использоваться секционирование оборудования или программного обеспечения. В результате крупный экземпляр HANA используется как в целом назначено для клиента и, следовательно, для вас. Перезагрузка или завершение работы сервера не влечет за собой автоматически развертывание операционной системы и SAP HANA на другом сервере. (Единственное исключение для SKU 1 класса — когда на сервере могут возникнуть проблемы и необходимо выполнить повторное развертывание на другой сервер.)
- В отличие от Azure, где типы процессоров узла выбираются на основе лучшего соотношения цены и производительности, для SAP HANA в Azure (крупные экземпляры) выбираются самые высокопроизводительные процессоры из линейки Intel E7v3 и E7v4.

**Дальнейшие действия**
- См. раздел [HLI Sizing](hana-sizing.md) (Определение размера HLI)
