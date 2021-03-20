---
title: Управление доступом к функциям управления Azure с помощью условного доступа в Azure AD
description: Сведения об использовании условного доступа в Azure AD для управления доступом к функциям управления Azure.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: skwan
ms.assetid: 0adc8b11-884e-476c-8c43-84f9bf12a34b
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/15/2019
ms.author: rolyon
ms.reviewer: skwan
ms.openlocfilehash: 2a52635dbaa7a76034f3a535b099320a901e8c07
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83758781"
---
# <a name="manage-access-to-azure-management-with-conditional-access"></a>Управление доступом к функциям управления Azure с помощью условного доступа

> [!CAUTION]
> Перед настройкой политики для управления доступом к функциям управления Azure тщательно изучите принципы работы условного доступа. Убедитесь в отсутствии условий, которые могут заблокировать вам доступ к порталу.

Компонент условного доступа в Azure Active Directory (Azure AD) управляет доступом к облачным приложениям на основе заданных вами условий. Чтобы разрешить доступ, создайте политики условного доступа, разрешающие или запрещающие доступ в зависимости от того, соблюдены ли требования политики. 

Обычно условный доступ используется для управления доступом к облачным приложениям. Вы также можете настроить политики для управления доступом к функциям управления Azure на основе определенных условий (таких как риск при входе, расположение или устройство) и для обеспечения соблюдения требований, таких как многофакторная проверка подлинности.

Чтобы создать политику для управления Azure, выберите **Microsoft Azure Management** (Управление Microsoft Azure) в области **Облачные приложения** при выборе приложения, к которому применяется политика.

![Условный доступ для управления Azure](./media/conditional-access-azure-management/conditional-access-azure-mgmt.png)

Создаваемая политика применяется ко всем конечным точкам управления Azure, включая следующие.

- Портал Azure
- Поставщик диспетчера ресурсов Azure
- Классические API управления службами
- Azure PowerShell
- Портал администрирования для подписок Visual Studio
- Azure DevOps
- Портал Фабрики данных Azure

Обратите внимание на то, что политика применяется к среде Azure PowerShell, которая вызывает API Azure Resource Manager. Она не применяется к [Azure AD PowerShell](/powershell/azure/active-directory/install-adv2), которая вызывает Microsoft Graph.

Информацию о том, как настроить пример политики, чтобы включить условный доступ для управления Microsoft Azure, см. в разделе [Условный доступ: запрос на MFA для управления Azure](../active-directory/conditional-access/howto-conditional-access-policy-azure-management.md).
