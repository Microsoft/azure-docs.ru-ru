---
title: включить файл
description: включить файл
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 07/08/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: e5a811620de8336abd3e0df6d72db761ce18b2b6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86230996"
---
Все последние поколения виртуальных машин поддерживают шифрование на узле:

|Тип  |Не поддерживается  |Поддерживается  |
|---------|---------|---------|
|Общего назначения     | Dv3, Dav4, Dv2, Av2        | B, DSv2, Dsv3, DC, DCv2, Dasv4        |
|Оптимизированные для вычислений     |         | Fsv2        |
|Оптимизированные для операций в памяти     | Ev3, Eav4        | DSv2, Esv3, M, Mv2, Easv4        |
|Оптимизированные для операций в хранилище     |         | Ls, Lsv2 (незашифрованные диски NVMe)        |
|Графический процессор     | NC, NV        | NCv2, NCv3, ND, NVv3, NVv4, NDv2 (Предварительная версия)        |
|Для высокопроизводительных вычислений     | H        | ХБ, HC, HBv2        |
|Предыдущие поколения     | F, A, D, L, G        | DS, GS, FS, NVv2        |

Обновление размера виртуальной машины приведет к проверке того, поддерживает ли новый размер виртуальной машины функцию Енкриптионасост.
