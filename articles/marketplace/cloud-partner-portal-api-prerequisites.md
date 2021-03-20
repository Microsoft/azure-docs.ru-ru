---
title: Необходимые условия для API — Azure Marketplace
description: Необходимые условия для использования API-интерфейсов Портал Cloud Partner.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 09/23/2020
ms.openlocfilehash: 374cc3398d1037b19ceddcbdaafd333f643e44ff
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91261164"
---
# <a name="api-prerequisites"></a>Предварительные требования для API

> [!NOTE]
> Портал Cloud Partner API интегрированы с и будут продолжать работать в центре партнеров. Переход содержит небольшие изменения. Ознакомьтесь с изменениями, приведенными в [справочнике по портал Cloud Partner API](cloud-partner-portal-api-overview.md) , чтобы убедиться, что код продолжит работать после перехода в центр партнеров. Используйте только API-интерфейсы CPP для существующих продуктов, которые уже были интегрированы до перехода в центр партнеров. новые продукты должны использовать API-интерфейсы отправки центра партнеров.

Для использования Портал Cloud Partner API требуются два необходимых программных ресурса: субъект-служба и маркер доступа Azure Active Directory (Azure AD).

## <a name="create-service-principal-in-azure-active-directory-tenant"></a>Создание субъекта-службы в Azure Active Directory клиенте

Сначала необходимо создать субъект-службу в клиенте Azure Active Directory. Этому клиенту будет назначен собственный набор разрешений на Портале Cloud Partner. Код будет вызывать API, используя этот клиент, а не личные учетные данные. Полное описание создания субъекта-службы см. в статье [как использовать портал для создания приложения Azure AD и субъекта-службы, которые имеют доступ к ресурсам](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="add-service-principal-to-your-account"></a>Добавление субъекта-службы в учетную запись

Теперь, когда вы создали субъект-службу в клиенте, вы можете добавить его в качестве пользователя в учетную запись портала центра партнеров. Как и пользователь, субъект-служба может быть владельцем или участником на портале. Дополнительные сведения см. в разделе **дальнейшие действия** ниже.

## <a name="next-steps"></a>Дальнейшие действия

См. раздел [Управление приложениями Azure AD](partner-center-portal/manage-account.md#manage-azure-ad-applications).
