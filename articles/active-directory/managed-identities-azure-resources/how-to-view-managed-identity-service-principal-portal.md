---
title: Просмотр субъекта-службы управляемого удостоверения в портал Azure Azure AD
description: Пошаговое руководство по просмотру субъекта-службы управляемого удостоверения на портале Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/30/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9b25f2691646192faf4d4c133835b456038abd94
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96003741"
---
# <a name="view-the-service-principal-of-a-managed-identity-in-the-azure-portal"></a>Просмотр субъекта-службы управляемого удостоверения на портале Azure

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые удостоверения в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде. 

В этой статье описано, как просмотреть субъект-службу управляемого удостоверения на портале Azure.

 > [!NOTE] 
 > Субъекты-службы — это корпоративные приложения. 

## <a name="prerequisites"></a>Предварительные требования

- Если вы не работали с управляемыми удостоверениями для ресурсов Azure, изучите [общие сведения](overview.md).
- Если у вас нет учетной записи Azure, зарегистрируйтесь для получения [бесплатной пробной версии](https://azure.microsoft.com/free/).
- [Включите назначенное системой удостоверение в виртуальной машине](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity) или [приложении](../../app-service/overview-managed-identity.md#add-a-system-assigned-identity).

## <a name="view-the-service-principal"></a>Просмотр субъекта-службы

Вы можете просмотреть субъект-службу виртуальной машины с включенным управляемым удостоверением, назначаемым системой (те же действия применяются для приложения).

1. Щелкните **Azure Active Directory** и выберите **Корпоративные приложения**.
2. В разделе **Тип приложения** выберите **все приложения** , а затем нажмите кнопку **Применить**.
3. В поле Фильтр поиска введите имя ресурса Azure с включенным управляемым удостоверением или выберите его из представленного списка.

   ![Просмотр субъектов-служб для управляемых удостоверений на портале](./media/how-to-view-managed-identity-service-principal-portal/view-managed-identity-service-principal-portal.png)

## <a name="next-steps"></a>Дальнейшие действия

[Управляемые удостоверения для ресурсов Azure](./overview.md)