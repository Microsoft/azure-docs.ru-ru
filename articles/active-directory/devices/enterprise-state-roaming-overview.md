---
title: Сведения о службе Enterprise State Roaming в Azure Active Directory
description: Служба Enterprise State Roaming представляет собой единое решение для всех устройств Windows.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: overview
ms.date: 02/12/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: na
ms.collection: M365-identity-device-management
ms.openlocfilehash: c22baf0a08718883f0c0c9844cc395f607b5b20d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "77194285"
---
# <a name="what-is-enterprise-state-roaming"></a>Служба Enterprise State Roaming

Используя Windows 10, пользователи [Azure Active Directory (Azure AD)](../fundamentals/active-directory-whatis.md) получают возможность безопасно синхронизировать свои параметры пользователя и данные параметров приложения в облаке. Служба Enterprise State Roaming представляет собой единое решение для всех устройств Windows и сокращает процесс настройки нового устройства. Enterprise State Roaming работает так же, как и стандартная [синхронизация параметров потребителя](https://go.microsoft.com/fwlink/?linkid=2015135) , которая впервые представлена в Windows 8. Кроме того, Enterprise State Roaming обеспечивает следующее:

* **Разделение корпоративных и потребительских данных**. Организации управляют своими данными, поэтому корпоративные данные не попадают в облачную учетную запись потребителя, а данные потребителя — в корпоративную облачную учетную запись.
* **Повышенная безопасность**. Перед отправкой с устройства Windows 10 данные шифруются с помощью службы управления правами Azure (Azure RMS) и остаются зашифрованными в облаке. Все содержимое, хранящееся в облаке, остается зашифрованным, кроме пространств имен, таких как имена параметров и приложений Windows.  
* **Улучшенное управление и наблюдение**. Благодаря интеграции с порталом Azure AD обеспечиваются контроль и видимость, позволяющие определить, кто синхронизирует параметры в организации и на каких устройствах. 

Служба Enterprise State Roaming доступна в различных регионах Azure. Обновленный список доступных регионов см. на странице [Регионы Azure — Службы по региону](https://azure.microsoft.com/regions/#services) в разделе Azure Active Directory.

| Статья | Описание |
| --- | --- |
| [Включение службы Enterprise State Roaming в Azure Active Directory](enterprise-state-roaming-enable.md) |Служба Enterprise State Roaming доступна для любой компании с подпиской Azure Active Directory Premium (Azure AD). Дополнительные сведения о том, как получить подписку Azure AD, см. на [странице продукта Azure AD](https://azure.microsoft.com/services/active-directory). |
| [Часто задаваемые вопросы о перемещении параметров и данных](enterprise-state-roaming-faqs.md) |В этой статье содержится информация о синхронизации параметров и данных приложений, которая может быть полезной для ИТ-администраторов. |
| [Group policy and MDM settings for settings sync (Параметры групповой политики и управления мобильными устройствами)](enterprise-state-roaming-group-policy-settings.md) |Windows 10 предоставляет параметры групповой политики и управления мобильными устройствами (MDM) для ограничения синхронизации параметров. |
| [Справочник по перемещаемым параметрам в Windows 10](enterprise-state-roaming-windows-settings-reference.md) |Список параметров, для которых в Windows 10 будет выполнено перемещение или резервное копирование. |
| [Устранение неполадок](enterprise-state-roaming-troubleshooting.md) |В этой статье представлены основные инструкции по устранению неполадок, а также список известных проблем. |

## <a name="next-steps"></a>Дальнейшие действия

Сведения об активации службы Enterprise State Roaming см. в [соответствующей статье](enterprise-state-roaming-enable.md).
