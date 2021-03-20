---
title: Изменение хэш-алгоритма подписи для отношения доверия с проверяющей стороной Microsoft 365 — Azure
description: На этой странице приводятся рекомендации по изменению алгоритма SHA для доверия федерации с Microsoft 365.
keywords: SHA1, SHA256, M365, Федерация, aadconnect, ADFS, AD FS, изменение SHA, доверие федерации, отношение доверия с проверяющей стороной
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: cf6880e2-af78-4cc9-91bc-b64de4428bbd
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/26/2018
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6bf9347d4d14e6583febd4ffaf0447e912133b80
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89660919"
---
# <a name="change-signature-hash-algorithm-for-microsoft-365-relying-party-trust"></a>Изменение хэш-алгоритма подписи для отношения доверия с проверяющей стороной Microsoft 365
## <a name="overview"></a>Обзор
Службы федерации Active Directory (AD FS) подписывают маркеры для Microsoft Azure Active Directory, чтобы защитить их от незаконного изменения. Подпись основывается на алгоритме SHA1 или SHA256. Теперь Azure Active Directory поддерживает маркеры, подписанные с помощью алгоритма SHA256. Мы рекомендуем выбирать этот алгоритм подписи маркера для обеспечения максимальной защиты. В этой статье описывается процедура замены алгоритма подписи маркера на алгоритм с более высоким уровнем защиты — SHA256.

>[!NOTE]
>Корпорация Майкрософт рекомендует использовать алгоритм SHA256 для подписания маркеров, так как он защищеннее, чем SHA1, но SHA1 по-прежнему поддерживается.

## <a name="change-the-token-signing-algorithm"></a>Изменение алгоритма подписи маркера
После настройки алгоритма подписи одним из двух приведенных ниже процессов AD FS подписывает маркеры для Microsoft 365 отношения доверия с проверяющей стороной с помощью SHA256. Вам не нужно вносить дополнительные изменения в конфигурацию, и это изменение не повлияет на возможность доступа к Microsoft 365 или другим приложениям Azure AD.

### <a name="ad-fs-management-console"></a>Консоль управления AD FS
1. Откройте консоль управления AD FS на сервере-источнике AD FS.
2. Разверните узел AD FS и выберите **Relying party trusts**(Отношения доверия с проверяющей стороной).
3. Щелкните правой кнопкой мыши Microsoft 365 отношения доверия с проверяющей стороной Azure и выберите пункт **Свойства**.
4. Перейдите на вкладку **Дополнительно** и в поле "Secure hash algorithm" (Алгоритм SHA) выберите значение SHA256.
5. Нажмите кнопку **OK**.

![Алгоритм подписи SHA256 — MMC](./media/how-to-connect-fed-sha256-guidance/mmc.png)

### <a name="ad-fs-powershell-cmdlets"></a>Командлеты AD FS PowerShell
1. На любом сервере AD FS откройте PowerShell с правами администратора.
2. Задайте алгоритм SHA с помощью командлета **Set-AdfsRelyingPartyTrust** .
   
   <code>Set-AdfsRelyingPartyTrust -TargetName 'Microsoft Office 365 Identity Platform' -SignatureAlgorithm 'https://www.w3.org/2001/04/xmldsig-more#rsa-sha256'</code>

## <a name="also-read"></a>Также ознакомьтесь
* [Восстановление Microsoft 365 доверия с Azure AD Connect](how-to-connect-fed-management.md#repairthetrust)

