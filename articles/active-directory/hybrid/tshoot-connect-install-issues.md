---
title: Устранение неполадок при установке Azure AD Connect | Документация Майкрософт
description: В этой статье приводятся пошаговые инструкции для устранения неполадок при установке Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 01/31/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 25314b4a306678dc877a95194907b3d73979e4f8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89275867"
---
# <a name="troubleshoot-azure-ad-connect-install-issues"></a>Устранение неполадок: Azure AD Connect проблемы установки

## <a name="recommended-steps"></a>**Рекомендуемые действия**
Убедитесь, что [тип установки Azure AD Connect](./how-to-connect-install-select-installation.md) подходит для вас. Если возникло условие экспресс-установки, то мы настоятельно рекомендуем перейти к экспресс-установке. Экспресс-установка предоставляет минимальные параметры, необходимые для завершения установки, поэтому существует меньше вероятности возникновения проблем. 

Тем не менее, если условия экспресс-установки не возникают и вам необходимо выполнить выборочную установку, ознакомьтесь с приведенными ниже рекомендациями, которые помогут вам избежать распространенных проблем. Для простоты здесь упомянуты только выборочные параметры.

* Убедитесь, что вы являетесь администратором на компьютере, где устанавливается AAD Connect. Войдите на компьютер под этими же учетными данными администратора.

* Оставьте значения по умолчанию для всех параметров на следующей странице, за исключением параметра "Использовать существующий SQL Server", если вы хотите использовать существующий SQL Server. Дополнительные сведения об использовании параметров выборочной установки см. [здесь](./how-to-connect-install-custom.md). 

    ![Параметр "Использовать существующий SQL Server"](media/tshoot-connect-install-issues/tshoot-connect-install-issues/useexistingsqlserver.png)

* На следующей странице выберите вариант "Создайте учетную запись AD", чтобы избежать любых проблем с разрешениями, связанными с существующей учетной записью.

    ![Учетная запись леса AD](media/tshoot-connect-install-issues/tshoot-connect-install-issues/createnewaccount.png)

### <a name="common-issues"></a>**Распространенные проблемы**

* [Проблемы с подключением к локальной Active Directory](./reference-connect-adconnectivitytools.md).

* [Проблемы с подключением к веб-службе Azure Active Directory](./tshoot-connect-connectivity.md).

* [Проблемы с разрешениями локального Active Directory](./how-to-connect-configure-ad-ds-connector-account.md).

## <a name="recommended-documents"></a>**Рекомендуемые документы**
* [Необходимые условия для Azure AD Connect](./how-to-connect-install-prerequisites.md)
* [Выбор типа установки для Azure AD Connect](./how-to-connect-install-select-installation.md)
* [Приступая к работе с Azure AD Connect с использованием стандартных параметров](./how-to-connect-install-express.md)
* [Выборочная установка Azure AD Connect](./how-to-connect-install-custom.md)
* [Azure AD Connect: обновление до последней версии](./how-to-upgrade-previous-version.md)
* [Azure AD Connect: что такое промежуточный сервер?](./plan-connect-topologies.md#staging-server)
* [Что такое модуль PowerShell ADConnectivityTools?](./how-to-connect-adconnectivitytools.md)

## <a name="next-steps"></a>Дальнейшие действия
- [Синхронизация Azure AD Connect](how-to-connect-sync-whatis.md).
- [Что собой представляет гибридная идентификация](whatis-hybrid-identity.md)