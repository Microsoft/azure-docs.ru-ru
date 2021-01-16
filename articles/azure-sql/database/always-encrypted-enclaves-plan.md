---
title: Планирование енклавес и аттестации Intel SGX в базе данных SQL Azure
description: Спланируйте развертывание Always Encrypted с помощью Secure енклавес в базе данных SQL Azure.
keywords: Шифрование данных, шифрование SQL, шифрование базы данных, конфиденциальные данные, Always Encrypted, безопасный енклавес, SGX, аттестация
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviwer: vanto
ms.date: 01/15/2021
ms.openlocfilehash: c41856c394166f2e3b8fd3bde794f0f294ef6af9
ms.sourcegitcommit: 25d1d5eb0329c14367621924e1da19af0a99acf1
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/16/2021
ms.locfileid: "98253565"
---
# <a name="plan-for-intel-sgx-enclaves-and-attestation-in-azure-sql-database"></a>Планирование енклавес и аттестации Intel SGX в базе данных SQL Azure

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> Always Encrypted с безопасным енклавес для базы данных SQL Azure в настоящее время находится в **общедоступной предварительной версии**.

[Always encrypted с безопасным енклавес](https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-enclaves) в базе данных SQL Azure использует [расширения Intel Software Guard (Intel SGX)](https://itpeernetwork.intel.com/microsoft-azure-confidential-computing/) енклавес и требует [аттестации Microsoft Azure](https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-enclaves#secure-enclave-attestation).

## <a name="plan-for-intel-sgx-in-azure-sql-database"></a>Планирование для Intel SGX в базе данных SQL Azure

Intel SGX — это надежная технология среды выполнения на основе оборудования. Intel SGX доступен для баз данных, использующих [модель Виртуальное ядро](service-tiers-vcore.md) и поколение оборудования для [серий контроллеров домена](service-tiers-vcore.md?#dc-series) . Таким образом, чтобы вы могли использовать Always Encrypted с безопасными енклавес в базе данных, необходимо либо выбрать создание оборудования серий контроллеров домена при создании базы данных, либо обновить существующую базу данных для использования создания оборудования серий контроллеров домена.

> [!NOTE]
> Intel SGX недоступен в аппаратных поколениях, отличных от серии DC. Например, Intel SGX недоступен для оборудования го поколения и недоступна для баз данных, использующих [модель DTU](service-tiers-dtu.md).

> [!IMPORTANT]
> Перед настройкой создания оборудования серий контроллеров домена для базы данных проверьте региональный уровень доступности рядов DC и убедитесь, что вы понимаете его ограничения производительности. Дополнительные сведения см. в разделе [DC-Series](service-tiers-vcore.md#dc-series).

## <a name="plan-for-attestation-in-azure-sql-database"></a>Планирование аттестации в базе данных SQL Azure

[Аттестация Microsoft Azure](../../attestation/overview.md) (Предварительная версия) — это решение для подтверждения доверенных сред выполнения (TEEs), включая Intel SGX енклавес в базах данных SQL Azure с использованием создания оборудования с помощью серии контроллеров домена.

Чтобы использовать аттестацию Azure для аттестации Intel SGX енклавес в базе данных SQL Azure, необходимо:

1. Создайте [поставщик аттестации](../../attestation/basic-concepts.md#attestation-provider) и настройте его с помощью политики аттестации. 

2. Предоставьте логическому серверу SQL Azure доступ к созданному поставщику аттестации.

## <a name="roles-and-responsibilities-when-configuring-sgx-enclaves-and-attestation"></a>Роли и обязанности при настройке енклавес и аттестации SGX

Настройка среды для поддержки Intel SGX енклавес и аттестации для Always Encrypted в базе данных SQL Azure включает в себя настройку компонентов различных типов: Microsoft Azure аттестация, база данных SQL Azure и приложения, которые инициируют анклава аттестацию. Настройка компонентов каждого типа выполняется пользователями при условии одной из следующих ролей:

- Администратор аттестации. создает поставщик аттестации в Microsoft Azure аттестации, авторы политики аттестации, предоставляют логическому серверу Azure SQL доступ к поставщику аттестации и совместно используют URL-адрес аттестации, указывающий на политику для администраторов приложений.
- Администратор базы данных SQL Azure — включает SGX енклавес в базах данных, выбирая создание оборудования из серии контроллеров домена и предоставляющая администратору аттестации удостоверение логического сервера Azure SQL, которому требуется доступ к поставщику аттестации.
- Администратор приложений — настраивает приложения с URL-адресом аттестации, полученным от администратора аттестации.

В рабочих средах (обработка реальных конфиденциальных данных) важно, чтобы Организация придерживается разделения ролей при настройке аттестации, где каждая отдельная роль принимается разными людьми. В частности, если целью развертывания Always Encrypted в Организации является сокращение контактной зоны для атаки путем обеспечения администраторам баз данных SQL Azure доступа к конфиденциальным данным, администраторы базы данных SQL Azure не должны управлять политиками аттестации.

## <a name="next-steps"></a>Дальнейшие действия

- [Включение Intel SGX для базы данных SQL Azure](always-encrypted-enclaves-enable-sgx.md)

## <a name="see-also"></a>См. также

- [Руководство. Приступая к работе с Always Encrypted Secure енклавес в базе данных SQL Azure](always-encrypted-enclaves-getting-started.md)