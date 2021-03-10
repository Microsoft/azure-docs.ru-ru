---
title: Аттестация енклавес в Azure
description: Узнайте, как можно использовать аттестацию для проверки безопасности доверенной среды конфиденциальных вычислений.
services: virtual-machines
author: JBCook
ms.service: virtual-machines
ms.subservice: confidential-computing
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 9/22/2020
ms.author: JenCook
ms.openlocfilehash: a7b0ca65329016b0a73f612115d8caba43dfbe2a
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102551357"
---
# <a name="attesting-sgx-enclaves"></a>Аттестация Енклавес SGX

Конфиденциальные вычисления в Azure предлагают виртуальные машины на основе Intel SGX, которые могут изолировать часть кода или данных. При работе с этими [енклавесми](confidential-computing-enclaves.md)необходимо проверить и проверить надежность доверенной среды. Такая проверка выполняется в ходе аттестации. 

## <a name="overview"></a>Обзор 

Аттестация позволяет проверяющей стороне получить дополнительные гарантии того, что программное обеспечение: (1) выполняется только в анклаве и (2) этот анклав содержит актуальное и защищенное содержимое. Например, анклав требует, чтобы аппаратное обеспечение создавало учетные данные, которые подтверждают наличие анклава на используемой платформе. Такой отчет можно передать в другой анклав, который подтвердит, что отчет был создан на той же платформе.

![код аттестации в анклава](media/attestation/attestation.png)



Аттестация должна быть реализована с использованием защищенной службы аттестации, которая совместима с программным и аппаратными обеспечением системы. Ниже приведены некоторые примеры служб, которые можно использовать.

- [Аттестация Microsoft Azure (Предварительная версия)](../attestation/overview.md) или
- [Службы аттестации и подготовки Intel](https://software.intel.com/sgx/attestation-services)


они совместимы с конфиденциальными вычислениями в Azure с использованием инфраструктуры Intel SGX. 

## <a name="next-steps"></a>Дальнейшие действия
Попробуйте [Microsoft Azure примеры аттестации для приложений, поддерживающих анклава](/samples/azure-samples/microsoft-azure-attestation/sample-code-for-intel-sgx-attestation-using-microsoft-azure-attestation/).