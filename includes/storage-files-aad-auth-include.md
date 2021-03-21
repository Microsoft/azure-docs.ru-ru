---
title: включить файл
description: включить файл
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/30/2019
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 84fa97ec964d490eb9571c7e030704562a4a81d0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95563670"
---
Служба [файлов Azure](../articles/storage/files/storage-files-introduction.md) поддерживает проверку подлинности на основе удостоверений через протокол SMB с помощью [локальных служб домен Active Directory Services (AD DS)](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) и [Azure Active Directory доменных служб (Azure AD DS)](../articles/active-directory-domain-services/overview.md). Эта статья посвящена тому, как файловые ресурсы Azure могут использовать службы домена локально или в Azure для поддержки доступа на основе удостоверений к файловым ресурсам Azure через SMB. Включение доступа на основе удостоверений для файловых ресурсов Azure позволяет заменить существующие файловые серверы на файловые ресурсы Azure, не заменяя существующую службу каталогов, сохраняя при этом простой доступ пользователей к общим ресурсам. 

Служба "файлы Azure" обеспечивает авторизацию доступа пользователей к общим ресурсам и уровням каталогов и файлов. Назначение разрешений на уровне общего ресурса можно выполнять для пользователей Azure Active Directory (Azure AD) или групп, управляемых с помощью модели [управления доступом на основе ролей Azure (Azure RBAC)](../articles/role-based-access-control/overview.md) . При использовании RBAC учетные данные, используемые для доступа к файлам, должны быть доступны или синхронизированы с Azure AD. Вы можете назначить встроенные роли Azure, такие как хранилище файлов хранилища SMB, для пользователей или групп в Azure AD, чтобы предоставить доступ на чтение к общему файловому ресурсу Azure.

На уровне каталога или файла служба файлов Azure поддерживает сохранение, наследование и принудительное применение списков управления [доступом Windows](/windows/win32/secauthz/access-control-lists) , как и для любых файловых серверов Windows. Вы можете использовать списки DACL Windows при копировании данных по протоколу SMB между существующей общей папкой и файловыми ресурсами Azure. Если вы планируете применять авторизацию, вы можете использовать файловые ресурсы Azure для резервного копирования списков ACL вместе с данными.