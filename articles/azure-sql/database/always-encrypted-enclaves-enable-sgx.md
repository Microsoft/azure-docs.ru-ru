---
title: Включение Intel SGX для Базы данных SQL Azure
description: Узнайте, как включить Intel SGX для Always Encrypted с защищенным енклавес в базе данных SQL Azure, выбрав создание оборудования с поддержкой SGX.
keywords: шифрование данных, шифрование SQL, шифрование базы данных, конфиденциальные данные, Always Encrypted, безопасные анклавы, SGX, аттестация
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviwer: vanto
ms.date: 01/15/2021
ms.openlocfilehash: ded1406c47bb3f00c366da7a5b28319f3712f8a7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98733762"
---
# <a name="enable-intel-sgx-for-your-azure-sql-database"></a>Включение Intel SGX для Базы данных SQL Azure 

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> Always Encrypted с безопасными анклавами для Базы данных SQL Azure в настоящее время предоставляется в **общедоступной предварительной версии**.

[Always encrypted с безопасным енклавес](/sql/relational-databases/security/encryption/always-encrypted-enclaves) в базе данных SQL Azure использует [расширения Intel Software Guard (Intel SGX)](https://itpeernetwork.intel.com/microsoft-azure-confidential-computing/) енклавес. Чтобы обеспечить доступность Intel SGX, база данных должна использовать [модель Виртуальное ядро](service-tiers-vcore.md) и создание оборудования для [серий контроллеров домена](service-tiers-vcore.md#dc-series) .

Настройка создания оборудования серии контроллеров домена для поддержки Intel SGX енклавес несет администратор базы данных SQL Azure. Ознакомьтесь с [ролями и обязанностями при настройке енклавес и аттестации SGX](always-encrypted-enclaves-plan.md#roles-and-responsibilities-when-configuring-sgx-enclaves-and-attestation).

> [!NOTE]
> Intel SGX недоступен в аппаратных поколениях, отличных от серии DC. Например, Intel SGX недоступен для оборудования го поколения и недоступна для баз данных, использующих [модель DTU](service-tiers-dtu.md).

> [!IMPORTANT]
> Перед настройкой создания оборудования серий контроллеров домена для базы данных проверьте региональный уровень доступности рядов DC и убедитесь, что вы понимаете его ограничения производительности. Подробные сведения см. в разделе о [серии DC](service-tiers-vcore.md#dc-series).

Подробные инструкции по настройке новой или существующей базы данных для использования определенного поколения оборудования см. в разделе [Выбор создания оборудования](service-tiers-vcore.md#selecting-a-hardware-generation).
   
## <a name="next-steps"></a>Дальнейшие действия

- [Настройка аттестации Azure для сервера базы данных SQL Azure](always-encrypted-enclaves-configure-attestation.md)

## <a name="see-also"></a>См. также раздел

- [Учебник. Начало работы с Always Encrypted и безопасными анклавами в Базе данных SQL Azure](always-encrypted-enclaves-getting-started.md)