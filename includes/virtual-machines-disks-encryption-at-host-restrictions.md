---
title: включить файл
description: включить файл
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 07/10/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 7700b1118c4f04643607f44a474338ffdd5c09e0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86231023"
---
- Не поддерживает Ultra Disks.
- Невозможно включить, если на виртуальных машинах или масштабируемых наборах виртуальных машин включено шифрование дисков Azure (с помощью BitLocker или ВИРТУАЛЬНОЙ машины).
- Шифрование дисков Azure не может быть включено на дисках с включенным шифрованием на узле.
- Шифрование можно включить в существующем масштабируемом наборе виртуальных машин. Однако автоматически шифруются только новые виртуальные машины, созданные после включения шифрования.
- Для шифрования существующие виртуальные машины должны быть освобождены и перераспределены.