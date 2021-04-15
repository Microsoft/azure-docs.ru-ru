---
title: Включить файл
description: включить файл
services: active-directory
author: daveba
ms.service: active-directory
ms.subservice: msi
ms.topic: include
ms.date: 05/31/2018
ms.author: daveba
ms.custom: include file
ms.openlocfilehash: 6e5885e076222cd23ba127f3be41c1218f327ca0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92379860"
---
- Каждое управляемое удостоверение учитывается в квоте объектов в арендаторе Azure AD, как описано в статье [Ограничения службы Azure AD](../articles/active-directory/enterprise-users/directory-service-limits-restrictions.md).
-   К частоте создания управляемых удостоверений применяются следующие ограничения:

    1. На каждого арендатора Azure AD в каждом регионе Azure: 200 операций создания в 20 с.
    2. На каждую подписку Azure в каждом регионе Azure: 40 операций создания в 20 с.

- При создании назначенных пользователем управляемых удостоверений можно использовать только буквенно-цифровые символы (0–9, a–z, A–Z) и дефис (-). Для корректной работы назначения для виртуальной машины или масштабируемого набора виртуальных машин имя должно содержать не более 24 символов.
