---
title: Azure AD Connect поддерживаемые топологии и сценарии облачной синхронизации
description: Узнайте о различных локальных и Azure Active Directory (Azure AD) топологиях, использующих облачную синхронизацию Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 02/26/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2fd14ed8d149cdc5296229c52ceb74afb2ce7b23
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98614031"
---
# <a name="azure-ad-connect-cloud-sync-supported-topologies-and-scenarios"></a>Azure AD Connect поддерживаемые топологии и сценарии облачной синхронизации
В этой статье описываются различные локальные и Azure Active Directory (Azure AD) топологии, в которых используется облачная синхронизация Azure AD Connect. Эта статья содержит только поддерживаемые конфигурации и сценарии.

> [!IMPORTANT]
> Корпорация Майкрософт не поддерживает изменение или эксплуатацию Azure AD Connect облачной синхронизации за пределами конфигураций или действий, которые формально документированы. Любая из этих конфигураций или действий может привести к несогласованному или неподдерживаемому состоянию Azure AD Connect облачной синхронизации. В результате корпорация Майкрософт не может предоставить техническую поддержку для таких развертываний.

## <a name="things-to-remember-about-all-scenarios-and-topologies"></a>Вопросы, которые следует помнить о всех сценариях и топологиях
Ниже приведен список сведений, которые следует учитывать при выборе решения.

- Пользователи и группы должны быть однозначно идентифицированы во всех лесах
- Сопоставление между лесами не выполняется при синхронизации с облаком
- Пользователь или группа должны быть представлены только один раз во всех лесах.
- Привязка источника для объектов выбирается автоматически.  Он использует MS-DS-ConsistencyGuid, если он есть, в противном случае используется ObjectGUID.
- Нельзя изменить атрибут, используемый для привязки к источнику.

## <a name="single-forest-single-azure-ad-tenant"></a>Один лес, один клиент Azure AD
![Схема, на которой показана топология для одного леса и одного клиента.](media/tutorial-single-forest/diagram-2.png)

Простейшая топология — это один локальный лес с одним или несколькими доменами и одним клиентом Azure AD.  Пример этого сценария см [. в разделе Учебник. один лес с одним клиентом Azure AD.](tutorial-single-forest.md)


## <a name="multi-forest-single-azure-ad-tenant"></a>Один клиент Azure AD с несколькими лесами
![Топология для нескольких лесов и одного клиента](media/plan-cloud-provisioning-topologies/multi-forest-2.png)

Общая топология — это несколько лесов AD с одним или несколькими доменами и одним клиентом Azure AD.  

## <a name="existing-forest-with-azure-ad-connect-new-forest-with-cloud-provisioning"></a>Существующий лес с Azure AD Connect, новый лес с подготовкой облачных сред
![Схема, на которой показана топология для существующего леса и нового леса.](media/tutorial-existing-forest/existing-forest-new-forest-2.png)

Этот сценарий аналогичен сценарию с несколькими лесами, однако в нем используется существующая Azure AD Connect среда, а затем в новом лесу выполняется синхронизация Azure AD Connect Cloud.  Пример этого сценария см. в разделе [учебник. существующий лес с одним клиентом Azure AD.](tutorial-existing-forest.md)

## <a name="piloting-azure-ad-connect-cloud-sync-in-an-existing-hybrid-ad-forest"></a>Пилотное развертывание Azure AD Connect облачной синхронизации в существующем гибридном лесу AD
![Топология для одного леса и одного клиента. ](media/tutorial-migrate-aadc-aadccp/diagram-2.png) сценарий пилотного развертывания включает в себя как Azure AD Connect, так Azure AD Connect синхронизацию облаков в одном лесу, а также соответствующим образом расставить пользователей и группы. Примечание. объект должен находиться в области только одного из средств. 

Пример этого сценария см [. в разделе Tutorial: пилотная Azure AD Connect облачная синхронизация в существующем лесу AD.](tutorial-pilot-aadc-aadccp.md)



## <a name="next-steps"></a>Дальнейшие действия 

- [Что собой представляет подготовка?](what-is-provisioning.md)
- [Что представляет собой облачная синхронизация Azure AD Connect?](what-is-cloud-sync.md)

