---
title: Виртуальные машины Azure серии DC
description: Спецификации виртуальных машин серии DC.
author: susaxen
ms.service: virtual-machines
ms.subservice: vm-sizes-general
ms.topic: conceptual
ms.date: 02/20/2020
ms.author: jushiman
ms.openlocfilehash: 677f4df0873f8b72d40dd373035111e2e0002491
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102549236"
---
# <a name="dcsv2-series"></a>Серия DCsv2


Серия DCsv2 может помочь защитить конфиденциальность и целостность данных и кода при его работе в общедоступном облаке. Эти машины поддерживают последнее поколение процессора Intel XEON E-2288G с технологией SGX. С помощью технологии Intel Turbo Boost эти компьютеры могут перейти на 5 ГГц. Экземпляры серии DCsv2 позволяют клиентам во время использования создавать защищенные приложения на основе анклава для защиты своего кода и данных.

Примеры использования: обмен конфиденциальными данными с участием нескольких сторон, обнаружение мошенничества, защита от отмывания денег, блокчейн, анализ использования конфиденциальных данных, анализ оперативной информации и конфиденциальное машинное обучение.

[Хранилище класса Premium](premium-storage-performance.md): поддерживаемое *<br> 
 [кэширование хранилища класса Premium](premium-storage-performance.md): поддерживаемые <br> 
 [Динамическая миграция](maintenance-and-updates.md): неподдерживаемые <br> 
 [обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается <br> 
 [Поддержка создания виртуальных машин](generation-2.md): поколение 2 с <br> 
 [ускоренной сетью](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (* требуется минимум 4 виртуальных ЦП *) <br>
[Временные диски ОС](ephemeral-os-disks.md): поддерживаются <br>

* За исключением Standard_DC8_v2 <br>

| Размер             | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Максимальное число дисков данных | Максимальная пропускная способность временного хранилища с кэшированием: операций ввода-вывода / Мбит/с (размер кэша в Гиб) | Максимальное количество сетевых адаптеров/ожидаемая пропускная способность сети (Мбит/с) | Память EPC (МиБ) |
|------------------|------|-------------|------------------------|----------------|-------------------------------------------------------------------------|----------------------------------------------|---------------------|
| Standard_DC1s_v2 | 1    | 4           | 50                     | 1              | 2000/16                                                                                               | 2   | 28                                      |
| Standard_DC2s_v2 | 2    | 8           | 100                    | 2              | 4000/32                                                                                               | 2  | 56                                          |
| Standard_DC4s_v2 | 4    | 16          | 200                    | 4              | 8000/64                                                                                               | 2  | 112                                          |
| Standard_DC8_v2  | 8   | 32          | 400                    | 8              | 16000/128                                                                                         | 2   | 168                                         |

- Виртуальные машины серии DCsv2 [являются ВМ 2-го поколения](./generation-2.md#creating-a-generation-2-vm) и поддерживают только образы `Gen2`.
- В настоящее время доступно в регионах, перечисленных [здесь](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines&regions=all).
- Предыдущее поколение виртуальных машин для конфиденциальных вычислений: [Серия DC](sizes-previous-gen.md#preview-dc-series)
- Создание виртуальных машин DCsv2 с помощью [портала Azure](./linux/quick-create-portal.md) или [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-compute.acc-virtual-machine-v2?tab=overview)



## <a name="other-sizes-and-information"></a>Другие размеры и сведения

- [Универсальные](sizes-general.md)
- [Оптимизированные для памяти](sizes-memory.md)
- [Оптимизированные для хранилища](sizes-storage.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Предыдущие поколения](sizes-previous-gen.md)

Калькулятор цен: [Калькулятор цен](https://azure.microsoft.com/pricing/calculator/)

Дополнительные сведения о типах дисков: [типы дисков](./disks-types.md#ultra-disk)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
