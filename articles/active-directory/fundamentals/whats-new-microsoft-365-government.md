---
title: Новые возможности Azure AD в Microsoft 365 правительственных учреждениях | Документы Майкрософт
description: Узнайте об изменениях в Azure Active Directory (Azure AD) в облачном экземпляре Microsoft 365, что может повлиять на вас.
services: active-directory
author: eross-msft
manager: daveba
ms.author: lizross
ms.reviewer: sumitp
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 05/07/2019
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: a0efc4bc8f89b0fbefbba171d80a3f8a1ed5e7f6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89318936"
---
# <a name="whats-new-for-azure-active-directory-in-microsoft-365-government"></a>Новые возможности Azure Active Directory в Microsoft 365 государственных организаций

Мы внесли некоторые изменения в Azure Active Directory (Azure AD) в облачном экземпляре Microsoft 365 для государственных организаций, который применим к клиентам, использующим следующие службы:

- Azure для государственных организаций

- Microsoft 365 правительственные учреждения — GCC High

- Microsoft 365 правительственные учреждения — DoD

Эта статья не относится к Microsoft 365 правительственным учреждениям — клиентам GCC.

## <a name="changes-to-the-initial-domain-name"></a>Изменение исходного доменного имени

Во время начальной регистрации вашей организации для службы Microsoft 365 государственных организаций вам было предложено выбрать доменное имя организации `<your-domain-name>.onmicrosoft.com` . Если у вас уже есть имя домена с суффиксом. com, ничего не изменится.

Однако если вы регистрируетесь для использования новой службы Microsoft 365 правительственных учреждений, вам будет предложено выбрать доменное имя с помощью `.us` суффикса. Итак, это будет `<your-domain-name>.onmicrosoft.us` .

>[!Note]
>Это изменение не относится к клиентам, которые управляются поставщиками облачных служб (CSP).

## <a name="changes-to-portal-access"></a>Изменения в доступе к порталу

Мы обновили конечные точки портала для Microsoft Azure для государственных организаций, Microsoft 365 правительственные учреждения — GCC High и Microsoft 365 правительства — DoD, как показано в [таблице сопоставления конечных точек](#endpoint-mapping).

Ранее клиенты могли входить с помощью порталов Azure (portal.azure.com) и Office 365 (portal.office.com) на разных языках. После этого обновления клиенты должны войти в систему, используя конкретные Microsoft Azure для государственных организаций, Microsoft 365 правительственные учреждения и Microsoft 365ные порталы для государственных организаций.

## <a name="endpoint-mapping"></a>Сопоставление конечных точек

В следующей таблице показаны конечные точки для всех клиентов.

| Имя | Сведения о конечной точке |
|------|------------------|
| Порталы |Microsoft Azure для государственных организаций: https://portal.azure.us<p>Microsoft 365 правительственные учреждения — GCC High: https://portal.office365.us<p>Microsoft 365 правительственные учреждения — DoD: https://portal.apps.mil |
| Конечная точка центра Azure Active Directory | https://login.microsoftonline.us |
| Microsoft Graph API для Microsoft 365 государственных организаций — GCC High | https://graph.microsoft.us |
| Microsoft Graph API для Microsoft 365 государственных организаций — DoD | https://dod-graph.microsoft.us |
| Конечные точки служб Azure для государственных организаций | Дополнительные сведения см. в разделе [руководством для разработчиков Azure для государственных организаций](../../azure-government/documentation-government-developer-guide.md) . |
| Microsoft 365 государственные учреждения — старшие конечные точки GCC | Дополнительные сведения см. в статье [Office 365 U.S. правительства GCC High Endpoints](/office365/enterprise/office-365-u-s-government-gcc-high-endpoints) |
| Microsoft 365 для государственных организаций — DoD | Дополнительные сведения см. в разделе [конечные точки Microsoft Office 365 США для государственных организаций](/office365/enterprise/office-365-u-s-government-dod-endpoints) |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения вы найдете в следующих статьях:

- [Что такое Azure для государственных организаций?](../../azure-government/documentation-government-welcome.md)

- [Обновление конечной точки центра сертификации AAD в Azure для государственных организаций](https://devblogs.microsoft.com/azuregov/azure-government-aad-authority-endpoint-update/)

- [Microsoft Graph конечных точек в облаке правительства США](https://developer.microsoft.com/graph/blogs/new-microsoft-graph-endpoints-in-us-government-cloud/)

- [Office 365 США, американская для государственных организаций, GCC High и DoD](/office365/servicedescriptions/office-365-platform-service-description/office-365-us-government/gcc-high-and-dod)