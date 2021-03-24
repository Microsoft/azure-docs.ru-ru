---
title: 'Azure Active Directory: лицензирование самостоятельного сброса пароля'
description: Сведения о разнице Azure Active Directory требования к лицензированию самостоятельного сброса пароля
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 03/08/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5d332c831cc764c61a4672ea5ad1db231b68e106
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104952377"
---
# <a name="licensing-requirements-for-azure-active-directory-self-service-password-reset"></a>Требования к лицензированию для самостоятельного сброса пароля Azure Active Directory

Чтобы снизить количество обращений в службу поддержки и потери производительности, когда пользователь не может войти на устройство или в приложение, учетные записи пользователей в Azure Active Directory (Azure AD) можно включить для самостоятельного сброса пароля (SSPR). Функции, составляющие SSPR, включают изменение пароля, сброс, разблокировку и обратную запись в локальный каталог. Базовые функции SSPR доступны в Microsoft 365 бизнес Standard или выше и все Azure AD Premium SKU без каких бы то ни было затрат.

В этой статье описаны различные способы лицензирования и использования самостоятельного сброса пароля. Конкретные сведения о ценах и выставлении счетов см. на [странице цен на Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="compare-editions-and-features"></a>Сравнение выпусков и функций

Для SSPR требуется лицензия только для клиента. 

В следующей таблице описаны различные сценарии SSPR для изменения, сброса или обратной записи паролей, а также номера SKU, которые предоставляют эту функцию.

| Функция | Azure AD уровня "Бесплатный" | Microsoft 365 бизнес стандартный; | Microsoft 365 бизнес премиум. | Azure AD Premium (P1 или P2) |
| --- |:---:|:---:|:---:|:---:|
| **Изменение пароля пользователя только в облаке**<br />Когда пользователь в Azure AD знает свой пароль и хочет изменить его на что-то новое. | ● | ● | ● | ● |
| **Сброс пароля пользователя только в облаке**<br />Когда пользователь в Azure AD забыл свой пароль и его необходимо сбросить. | | ● | ● | ● |
| **Изменение или сброс пароля гибридного пользователя с помощью локальной обратной записи**<br />Когда пользователь в Azure AD, синхронизированный из локального каталога, использует Azure AD Connect хочет изменить или сбросить пароль, а также записать новый пароль обратно в локальную среду. | | | ● | ● |

> [!WARNING]
> Автономные Microsoft 365 планы лицензирования "базовый" и "Стандартный" не поддерживают SSPR с обратной записью в локальной среде. Для функции обратной записи в локальной среде требуется Azure AD Premium P1, Premium P2 или Microsoft 365 бизнес Premium.

Дополнительные сведения о лицензировании, включая затраты, см. на следующих страницах:

* [Цены на Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/)
* [Функции и возможности Azure Active Directory](https://www.microsoft.com/cloud-platform/azure-active-directory-features)
* [Enterprise Mobility + Security](https://www.microsoft.com/cloud-platform/enterprise-mobility-security)
* [Microsoft 365 Enterprise](https://www.microsoft.com/microsoft-365/enterprise)
* [Microsoft 365 для бизнеса](/office365/servicedescriptions/microsoft-365-service-descriptions/microsoft-365-business-service-description)

## <a name="next-steps"></a>Дальнейшие действия

Чтобы приступить к работе с SSPR, выполните инструкции из следующего руководства.

> [!div class="nextstepaction"]
> [Учебник. Включение самостоятельного сброса пароля (SSPR)](tutorial-enable-sspr.md)