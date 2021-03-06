---
title: Обзор решения Azure VMware по Клаудсимпле — узлы
description: Узнайте об основных понятиях Клаудсимпле, включая узлы, подготовленные узлы, частное облако и решение VMware по номерам SKU узлов Клаудсимпле.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 7de4b5faa997d909089daedab7e48e5d5a6de2e7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88140842"
---
# <a name="cloudsimple-nodes-overview"></a>Обзор узлов Клаудсимпле

Узлы — это стандартные блоки частного облака. Узел:

* Выделенный компьютер для вычислений без операционной системы, на котором установлена низкоуровневая оболочка VMware ESXi  
* Единицу вычислений, которую можно подготавливать или резервировать для создания частных облаков
* Доступно для инициализации или резервирования в регионе, где доступна служба Клаудсимпле

Частное облако создается на основе подготовленных узлов. Чтобы создать частное облако, требуется как минимум три узла одного номера SKU. Чтобы развернуть частное облако, добавьте дополнительные узлы.  Можно добавить узлы в существующий кластер или создать новый кластер путем подготовки узлов в портал Azure и связывания их со службой Клаудсимпле.  Все подготовленные узлы отображаются в службе Клаудсимпле.  

## <a name="provisioned-nodes"></a>Подготовленные узлы

Подготовленные узлы обеспечивают емкость с оплатой по мере использования. Узлы подготовки помогают быстро масштабировать кластер VMware по требованию. При необходимости можно добавить узлы или удалить подготовленный узел для масштабирования кластера VMware. Счета за подготовленные узлы выставляются ежемесячно и взимается в подписку, в которой они подготавливаются.

* Если вы платите за подписку Azure по кредитной карте, плата оплачивается немедленно.
* Если счет выставляется по счету, плата отображается в следующем счете.

## <a name="vmware-solution-by-cloudsimple-nodes-sku"></a>SKU решения VMware по Клаудсимпле для узлов

Для подготовки или резервирования доступны следующие типы узлов.

| SKU           | CS28-node                 | CS36-node                 | CS36m-node                |
|---------------|-----------------------------|-----------------------------|-----------------------------|
| Регион        | Восточная часть США, западная часть США            | Восточная часть США, западная часть США            | Западная Европа                 |
| ЦП           | 2 2,2 ГГц, 28 ядер (56 HT) | 2 ядра с частотой 2.3 ГГц, 36 ядер (72 HT) | 2 ядра с частотой 2.3 ГГц, 36 ядер (72 HT) |
| ОЗУ           | 256 ГБ                      | 512 ГБ                      | 576 ГБ                      |
| Диск кэша    | 1,6-ТБ NVMe                 | 3,2-ТБ NVMe                 | 3,2-ТБ NVMe                 |
| Диск емкости | 5,625 ТБ необработанных                | 11,25 ТБ необработанных                | 15,36 ТБ необработанных                |
| Тип хранения  | All Flash                   | All Flash                   | All Flash                   |

## <a name="limits"></a>Ограничения

Следующие ограничения узла применяются к частным облакам.

| Ресурс | Ограничение |
|----------|-------|
| Минимальное число узлов для создания частного облака | 3 |
| Максимальное число узлов в кластере в частном облаке | 16 |
| Максимальное число узлов в частном облаке | 64 |
| Минимальное число узлов в новом кластере | 3 |

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о [подготовке узлов](create-nodes.md)
* Сведения о [частных облаках](cloudsimple-private-cloud.md)
