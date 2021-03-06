---
title: Управляемые удостоверения для ресурсов Azure
description: Общие сведения об управляемых удостоверениях для ресурсов Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.assetid: 0232041d-b8f5-4bd2-8d11-27999ad69370
ms.service: active-directory
ms.subservice: msi
ms.devlang: ''
ms.topic: overview
ms.custom: mvc
ms.date: 10/06/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5390811c8da4a8cace32e0e7ba4524e8c537a26a
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106055592"
---
# <a name="what-are-managed-identities-for-azure-resources"></a>Что такое управляемые удостоверения для ресурсов Azure?

Распространенной проблемой для разработчиков является управление секретами и учетными данными для защиты обмена данными между различными службами. В Azure управляемые удостоверения устраняют необходимость в управлении учетными данными для разработчиков, предоставляя идентификатор для ресурса Azure в Azure AD и используя его для получения маркеров Azure Active Directory (Azure AD). Это также помогает получить доступ к [Azure Key Vault](../../key-vault/general/overview.md), где разработчики могут безопасно хранить учетные данные. Управляемые удостоверения для ресурсов Azure решают эту проблему, предоставляя службы Azure с автоматически управляемым удостоверением в Azure AD.

Для чего можно использовать управляемое удостоверение?

   > [!VIDEO https://www.youtube.com/embed/5lqayO_oeEo]

Ниже приведены некоторые преимущества использования управляемых удостоверений.

- Управление учетными данными не требуется. Учетные данные даже недоступны.
- Управляемые удостоверения можно использовать для проверки подлинности в любой службе Azure, которая поддерживает проверку подлинности Azure AD, включая Azure Key Vault.
- Управляемые удостоверения можно использовать без дополнительных затрат.

> [!NOTE]
> Управляемые удостоверения для ресурсов Azure — это новое название службы "Управляемое удостоверение службы" (MSI).

## <a name="managed-identity-types"></a>Типы управляемых удостоверений

Существует два типа управляемых удостоверений.

- **Назначенные системой**. Некоторые службы Azure позволяют включить управляемое удостоверение непосредственно в экземпляре службы. При включении управляемого удостоверения, назначенного системой, в Azure AD создается удостоверение, связанное с жизненным циклом этого экземпляра службы. Поэтому при удалении ресурса Azure автоматически удаляет удостоверение. Только ресурс Azure может использовать это удостоверение для запроса маркеров из Azure AD.
- **Назначенные пользователем**. Вы также можете сами создать управляемое удостоверение в качестве автономного ресурса Azure. Вы можете [создать назначаемое пользователем управляемое удостоверение](how-to-manage-ua-identity-portal.md) и назначить его одному или нескольким экземплярам службы Azure. В случае назначенных пользователем управляемых удостоверений управление таким удостоверением осуществляется отдельно от ресурсов, которые его используют. </br></br>

  > [!VIDEO https://www.youtube.com/embed/OzqpxeD3fG0]

В таблице ниже показаны различия между двумя типами управляемых удостоверений.

|  Свойство    | Управляемое удостоверение, назначаемое системой | Управляемое удостоверение, назначаемое пользователем |
|------|----------------------------------|--------------------------------|
| Создание |  Создано как часть ресурса Azure (например, виртуальная машина Azure или Служба приложений Azure). | Создано в качестве автономного ресурса Azure. |
| Жизненный цикл | Жизненный цикл в общем доступе с ресурсом Azure, указанным при создании управляемого удостоверения. <br/> При удалении родительского ресурса управляемое удостоверение также удаляется. | Независимый жизненный цикл. <br/> Должен быть явно удален. |
| Совместное использование ресурсов Azure | Невозможно настроить общий доступ. <br/> Может быть связано только с одним ресурсом Azure. | Может быть в общем доступе. <br/> Одно и то же назначаемое пользователем управляемое удостоверение может быть связано с несколькими ресурсами Azure. |
| Распространенные варианты использования | Рабочие нагрузки, которые содержатся в одном ресурсе Azure. <br/> Рабочие нагрузки, для которых требуются независимые удостоверения. <br/> Например, приложение, выполняющееся на одной виртуальной машине. | Рабочие нагрузки, которые выполняются на нескольких ресурсах и могут совместно использовать одно удостоверение. <br/> Рабочие нагрузки, требующие предварительную авторизацию к защищенному ресурсу, как часть потока подготовки. <br/> Рабочие нагрузки, где ресурсы перезапускаются часто, но разрешения должны быть согласованными. <br/> Например, рабочая нагрузка, где нескольким виртуальным машинам требуется доступ к одному и тому же ресурсу. |

>[!IMPORTANT]
>Независимо от выбранного типа удостоверения управляемое удостоверение является субъектом-службой специального типа, который может использоваться только с ресурсами Azure. При удалении управляемого удостоверения соответствующий субъект-служба автоматически удаляется.

## <a name="how-can-i-use-managed-identities-for-azure-resources"></a>Как применять управляемые удостоверения для ресурсов Azure?

![Некоторые примеры того, как разработчик может использовать управляемые удостоверения для получения доступа к ресурсам из своего кода без управления данными аутентификации.](media/overview/azure-managed-identities-examples.png)

## <a name="what-azure-services-support-the-feature"></a>Службы Azure, в которых поддерживается данная функция.<a name="which-azure-services-support-managed-identity"></a>

Управляемые удостоверения для ресурсов Azure можно использовать для аутентификации служб с поддержкой аутентификации Azure AD. См. дополнительные сведения о [службах, которые поддерживают управляемые удостоверения для ресурсов Azure](./services-support-managed-identities.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Использование назначаемого системой управляемого удостоверения на виртуальной машине Windows для доступа к Resource Manager](tutorial-windows-vm-access-arm.md)
* [Использование назначаемого системой управляемого удостоверения на виртуальной машине Linux для доступа к Resource Manager](tutorial-linux-vm-access-arm.md)
* [Использование управляемых удостоверений в Службе приложений и Функциях Azure](../../app-service/overview-managed-identity.md)
* [Использование управляемых удостоверений для службы "Экземпляры контейнеров Azure"](../../container-instances/container-instances-managed-identity.md)
* [Реализация управляемых удостоверений для ресурсов Microsoft Azure](https://www.pluralsight.com/courses/microsoft-azure-resources-managed-identities-implementing).