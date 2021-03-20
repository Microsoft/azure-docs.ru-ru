---
title: Начало работы с API отчетов Azure AD | Документация Майкрософт
description: Как начать работу с API отчетов Azure Active Directory
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 8813b911-a4ec-4234-8474-2eef9afea11e
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 01/21/2021
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: b6ef122cd36c3d4e9135876647075f22158df0d1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101653241"
---
# <a name="get-started-with-the-azure-active-directory-reporting-api"></a>Начало работы с API отчетов Azure Active Directory

Azure Active Directory предоставляет разнообразные [отчеты](overview-reports.md), содержащий полезные сведения для приложений, таких как системы SIEM, средства аудита и бизнес-аналитики. 

API Microsoft Graph для отчетов Azure AD обеспечивает программный доступ к данным с помощью набора интерфейсов API на базе REST. Эти интерфейсы API можно вызвать, используя различные языки и инструменты программирования.

В этой статье представлены общие сведения об API отчетов, включая информацию о способах доступа к нему.

При возникновении проблем ознакомьтесь со статьей [Как получить поддержку для Azure Active Directory](../fundamentals/active-directory-troubleshooting-support-howto.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы получить доступ к API отчетов с вмешательством пользователя или без него, потребуется:

1. Назначение ролей (читателя сведений о безопасности, администратора безопасности, глобального администратора).
2. Регистрация приложения
3. Предоставить разрешения
4. Сбор параметров конфигурации

Подробные инструкции см. в [предварительных требованиях для доступа к API отчетов Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md). 

## <a name="api-endpoints"></a>Конечные точки API 

`https://graph.microsoft.com/v1.0/auditLogs/directoryAudits` — это конечная точка API Microsoft Graph для журналов аудита, а `https://graph.microsoft.com/v1.0/auditLogs/signIns` — конечная точка API Microsoft Graph для входа в систему. Дополнительные сведения см. в справке по [API аудита](/graph/api/resources/directoryaudit) и [API входа](/graph/api/resources/signIn).

[API обнаружения угроз для защиты идентификации](/graph/api/resources/identityriskevent?view=graph-rest-beta) можно использовать для получения программного доступа к средствам обнаружения безопасности с помощью Microsoft Graph. Дополнительные сведения см. в статье [Начало работы с защитой идентификации Azure Active Directory и Microsoft Graph](../identity-protection/howto-identity-protection-graph-api.md). 
  
Для получения программного доступа к событиям подготовки в клиенте также можно использовать [API журналов подготовки](/graph/api/resources/provisioningobjectsummary?view=graph-rest-beta) . 

## <a name="apis-with-microsoft-graph-explorer"></a>API с песочницей Microsoft Graph

Вы можете использовать [песочницу Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer) для проверки входа и аудита данных API. Войдите в учетную запись, используя одну из кнопок входа в пользовательском интерфейсе песочницы Graph и задав для клиента разрешения **AuditLog.Read.All** и **Directory.ReadAll**, как показано ниже.   

![Песочница Graph](./media/concept-reporting-api/graph-explorer.png)

![Пользовательский интерфейс для изменения разрешений](./media/concept-reporting-api/modify-permissions.png)

## <a name="use-certificates-to-access-the-azure-ad-reporting-api"></a>Использование сертификатов для доступа к API отчетов Azure AD 

Если вы планируете получать данные отчетов без вмешательства пользователя, используйте API отчетов Azure AD с сертификатами.

Подробные инструкции см. в статье о [получении данных с помощью API отчетов Azure AD с сертификатами](tutorial-access-api-with-certificates.md).

## <a name="next-steps"></a>Дальнейшие действия

 * [Предварительные требования для доступа к API отчетов](howto-configure-prerequisites-for-reporting-api.md) 
 * [Получение данных, используя API отчетов Azure AD с сертификатами](tutorial-access-api-with-certificates.md)
 * [Устранение ошибок в API отчетов Azure Active Directory](troubleshoot-graph-api.md)