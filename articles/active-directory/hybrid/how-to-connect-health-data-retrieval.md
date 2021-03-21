---
title: Получение сведений о Azure AD Connect Healthных инструкциях | Документация Майкрософт
description: На этой странице описано, как получить данные из Azure AD Connect Health.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/02/2020
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8d147d2c094923e971e52e1dbfe3f7a19776d38c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89463657"
---
# <a name="azure-ad-connect-health-instructions-for-data-retrieval"></a>Azure AD Connect Health инструкции по извлечению данных

В этом документе описано, как использовать Azure AD Connect для получения данных из Azure AD Connect Health.

[!INCLUDE [active-directory-app-provisioning.md](../../../includes/gdpr-intro-sentence.md)]

## <a name="retrieve-all-email-addresses-for-users-configured-for-health-alerts"></a>Получение всех адресов электронной почты для пользователей, для которых настроены оповещения о работоспособности.

Чтобы получить адреса электронной почты всех пользователей, настроенных в Azure AD Connect Health для получения оповещений, выполните следующие действия.

1.  Откройте колонку работоспособности Azure Active Directory Connect и выберите **службы синхронизации** на панели навигации слева.
 ![Службы синхронизации](./media/how-to-connect-health-data-retrieval/retrieve1.png)

2.  Щелкните плитку **оповещения** .</br>
 ![Предупреждение](./media/how-to-connect-health-data-retrieval/retrieve3.png)

3.  Щелкните **Параметры уведомлений**.
 ![Уведомление](./media/how-to-connect-health-data-retrieval/retrieve4.png)

4.  В колонке " **параметр уведомления** " вы найдете список адресов электронной почты, которые были включены в качестве получателей уведомлений о работоспособности.
 ![Сообщений электронной почты](./media/how-to-connect-health-data-retrieval/retrieve5a.png)
 
## <a name="retrieve-accounts-that-were-flagged-with-ad-fs-bad-password-attempts"></a>Получение учетных записей, помеченных AD FS неверных попыток ввода пароля

Чтобы получить учетные записи, помеченные AD FS неверных попыток ввода пароля, выполните следующие действия.

1.  В колонке работоспособности Azure Active Directory выберите **ошибки синхронизации**.
 ![Ошибки синхронизации](./media/how-to-connect-health-data-retrieval/retrieve6.png)

2.  В колонке **ошибки синхронизации** щелкните **Экспорт**. Будет экспортирован список записанных ошибок синхронизации.
 ![Экспорт](./media/how-to-connect-health-data-retrieval/retrieve7.png)

## <a name="next-steps"></a>Дальнейшие действия
* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Установка агента Azure AD Connect Health](how-to-connect-health-agent-install.md)
* [Операции Azure AD Connect Health](how-to-connect-health-operations.md)
* [Часто задаваемые вопросы об Azure AD Connect Health](reference-connect-health-faq.md)
* [Azure AD Connect Health: история версий](reference-connect-health-version-history.md)