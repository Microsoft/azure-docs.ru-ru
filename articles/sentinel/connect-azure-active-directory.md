---
title: Подключение Azure Active Directory данных к Azure Sentinel | Документация Майкрософт
description: Узнайте, как получать данные из Azure Active Directory и выполнять потоковую передачу журналов входа в Azure AD, аудита и подготовки в Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 0a8f4a58-e96a-4883-adf3-6b8b49208e6a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/20/2021
ms.author: yelevin
ms.openlocfilehash: f8931fedb380cf81d72b7b5280a5795498daaa57
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99251987"
---
# <a name="connect-azure-active-directory-azure-ad-data-to-azure-sentinel"></a>Подключение данных Azure Active Directory (Azure AD) к Azure Sentinel

С помощью встроенного соединителя Azure Sentinel можно собирать данные из [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) и выполнять их потоковую передачу в Azure Sentinel. Соединитель позволяет выполнять потоковую передачу следующих типов журналов:

- [**Журналы входа**](../active-directory/reports-monitoring/concept-all-sign-ins.md), содержащие сведения о [входах интерактивных пользователей](../active-directory/reports-monitoring/concept-all-sign-ins.md#user-sign-ins) , где пользователь предоставляет фактор проверки подлинности.

    Соединитель Azure AD теперь включает в себя следующие три дополнительные категории журналов входа, которые сейчас доступны в **предварительной версии**:
    
    - [**Неинтерактивные журналы входа пользователя**](../active-directory/reports-monitoring/concept-all-sign-ins.md#non-interactive-user-sign-ins), содержащие сведения о входах, выполняемых клиентом от имени пользователя без какого-либо взаимодействия или фактора проверки подлинности от пользователя.
    
    - [**Журналы входа субъекта-службы**](../active-directory/reports-monitoring/concept-all-sign-ins.md#service-principal-sign-ins), содержащие сведения о входах в приложения и субъекты-службы, не использующие каких-либо пользователей. В этих операциях входа приложение или служба предоставляют учетные данные от своего имени для проверки подлинности или доступа к ресурсам.
    
    - [**Журналы входа управляемых удостоверений**](../active-directory/reports-monitoring/concept-all-sign-ins.md#managed-identity-for-azure-resources-sign-ins), которые содержат сведения об операциях входа в ресурсы Azure с секретами, управляемыми Azure. Дополнительные сведения см. в статье [что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md)

- [**Журналы аудита**](../active-directory/reports-monitoring/concept-audit-logs.md), содержащие сведения о системных действиях, связанных с управлением пользователями и группами, управляемыми приложениями и действиями с каталогами.

- [**Журналы подготовки**](../active-directory/reports-monitoring/concept-provisioning-logs.md) (также в **предварительной версии**), содержащие сведения о действиях системы для пользователей, групп и ролей, подготовленных службой подготовки Azure AD. 

> [!IMPORTANT]
> Как указано выше, некоторые из доступных типов журналов сейчас доступны в **предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.
## <a name="prerequisites"></a>Предварительные требования

- Любой лицензии Azure AD (бесплатная/O365/P1/P2) достаточно для приема журналов событий входа в Azure Sentinel. Для Azure Monitor (анализ журналов) и Azure Sentinel может взиматься дополнительная плата за гигабайт.

- Пользователю необходимо назначить роль участника Azure Sentinel в рабочей области.

- Пользователю необходимо назначить роли глобального администратора или администратора безопасности на клиенте, с которого требуется выполнять потоковую передачу журналов.

- Чтобы увидеть состояние подключения, пользователь должен иметь разрешения на чтение и запись для параметров диагностики Azure AD. 

## <a name="connect-to-azure-active-directory"></a>Подключение к Azure Active Directory

1. В пункте Azure Sentinel выберите **соединители данных** в меню навигации.

1. В коллекции соединителей данных выберите **Azure Active Directory** а затем щелкните **открыть страницу соединителя**.

1. Установите флажки рядом с типами журналов, которые вы хотите передавать в Azure Sentinel (см. выше), и нажмите кнопку **подключить**.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе **логманажемент** в следующих таблицах:

- `SigninLogs`
- `AuditLogs`
- `AADNonInteractiveUserSignInLogs`
- `AADServicePrincipalSignInLogs`
- `AADManagedIdentitySignInLogs`
- `AADProvisioningLogs`

Чтобы запросить журналы Azure AD, введите соответствующее имя таблицы в верхней части окна запроса.

## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как подключить Azure Active Directory к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
