---
title: Облачные приложения или действия в политике условного доступа (Azure Active Directory)
description: Сведения об облачных приложениях или действиях в политике условного доступа Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 10/16/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2895588a5a82ec2b6c69d33ff6cea39bbe3a0372
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103492002"
---
# <a name="conditional-access-cloud-apps-or-actions"></a>Условный доступ. Облачные приложения или действия

Облачные приложения или действия являются ключевым сигналом в политике условного доступа. Политики условного доступа позволяют администраторам назначать элементы управления конкретным приложениям или действиям.

- Администраторы могут выбрать из списка приложений те, которые содержат встроенные приложения Майкрософт или [интегрированные приложения Azure AD](../manage-apps/what-is-application-management.md), в том числе приложения из коллекции, не из коллекции или опубликованные через [прокси приложения](../manage-apps/what-is-application-proxy.md).
- Администраторы могут определить политику, основанную не на облачном приложении, а на действии пользователя. Сейчас поддерживается только действие "Регистрация сведений о безопасности (предварительная версия)", которое позволяет системе условного доступа применять элементы управления к [функции объединенной регистрации сведений о безопасности](../authentication/howto-registration-mfa-sspr-combined.md).

![Определение политики условного доступа и выбор облачных приложений](./media/concept-conditional-access-cloud-apps/conditional-access-cloud-apps-or-actions.png)

## <a name="microsoft-cloud-applications"></a>Облачные приложения Майкрософт

В список приложений, из которого вы можете выбрать, входят многие из существующих облачных приложений корпорации Майкрософт. 

Политики условного доступа администраторы могут назначить для следующих облачных приложений от корпорации Майкрософт. Некоторые приложения, такие как Office 365 и управление Microsoft Azure, включают несколько связанных дочерних приложений или служб. Следующий список не является исчерпывающим и может измениться.

- [Office 365.](#office-365)
- Службы Azure Analysis Services
- Azure DevOps
- [База данных SQL Azure и Azure синапсе Analytics](../../azure-sql/database/conditional-access-configure.md)
- Dynamics CRM Online
- Аналитика Microsoft Application Insights
- [Microsoft Azure Information Protection](/azure/information-protection/faqs#i-see-azure-information-protection-is-listed-as-an-available-cloud-app-for-conditional-accesshow-does-this-work)
- [Управление Microsoft Azure](#microsoft-azure-management)
- Управление подписками Microsoft Azure
- Microsoft Cloud App Security
- Портал управления доступом к средствам Microsoft Commerce
- Служба аутентификации для средств Microsoft Commerce
- Microsoft Flow
- Microsoft Forms
- Microsoft Intune
- [Регистрация в Microsoft Intune](/intune/enrollment/multi-factor-authentication)
- Планировщик (Майкрософт);
- Microsoft PowerApps
- Microsoft Search для Bing
- Microsoft StaffHub
- Microsoft Stream;
- Microsoft Teams
- Exchange Online
- SharePoint
- Yammer
- Office Delve
- Office Sway
- Outlook Groups
- Служба Power BI
- Project Online
- Skype для бизнеса Online
- Виртуальная частная сеть (VPN)
- ATP в Защитнике Windows

### <a name="office-365"></a>Office 365

Microsoft 365 предоставляет облачные службы для повышения производительности и совместной работы, такие как Exchange, SharePoint и Microsoft Teams. Microsoft 365 облачные службы тесно интегрированы для обеспечения беспрепятственного и совместного взаимодействия. Такая интеграция может приводить к путанице при создании политик, так как Microsoft Teams и некоторые другие приложения имеют зависимости от SharePoint, Exchange и т. д.

Приложение Office 365 позволяет одновременно ориентироваться на эти службы. Мы рекомендуем использовать новое приложение Office 365, а не выбирать отдельные облачные приложения, чтобы избежать проблем с [зависимостями служб](service-dependencies.md). Использование этой группы приложений избавляет от потенциальных проблем с несогласованностью политик и зависимостей.

Администраторы могут исключить определенные приложения из политики, если они хотят включить приложение Office 365 и исключить определенные приложения по своему усмотрению в политике.

Ключевые приложения, которые входят в клиентское приложение Office 365:

   - Microsoft Flow
   - Microsoft Forms
   - Microsoft Stream;
   - Microsoft To-Do
   - Microsoft Teams
   - Exchange Online
   - SharePoint Online
   - Microsoft 365 Служба поиска
   - Yammer
   - Office Delve
   - Office Online
   - Office.com
   - OneDrive
   - PowerApps.
   - Skype для бизнеса Online
   - Sway

### <a name="microsoft-azure-management"></a>Управление Microsoft Azure

Приложение "Управление Microsoft Azure" включает несколько базовых служб. 

   - Портал Azure
   - Поставщик диспетчера ресурсов Azure
   - API классической модели развертывания
   - Azure PowerShell
   - Azure CLI
   - Портал администрирования для подписок Visual Studio
   - Azure DevOps
   - Портал Фабрики данных Azure

> [!NOTE]
> Приложение "Управление Microsoft Azure" применяется к среде Azure PowerShell, которая использует API Azure Resource Manager. Но оно не применяется к среде Azure AD PowerShell, которая вызывает Microsoft Graph.

## <a name="other-applications"></a>Другие приложения

Помимо приложений корпорации Майкрософт, администраторы могут добавить в политики условного доступа любое зарегистрированное приложение Azure AD. Сюда можно включить, среди прочего, следующее: 

- Приложения, опубликованные через [прокси приложения Azure AD](../manage-apps/what-is-application-proxy.md).
- [Приложения, добавленные из коллекции.](../manage-apps/add-application-portal.md)
- [Пользовательские приложения, не включенные в коллекцию.](../manage-apps/view-applications-portal.md)
- [Устаревшие приложения, опубликованные через контроллеры и сети доставки приложений.](../manage-apps/secure-hybrid-access.md)
- Приложения, использующие [единый вход на основе пароля](../manage-apps/configure-password-single-sign-on-non-gallery-applications.md)

> [!NOTE]
> Так как политика условного доступа устанавливает требования к доступу к службе, вы не можете применить ее к клиентскому приложению (открытому или собственному). Иными словами, политика задается не напрямую для клиентского приложения (открытого или собственного), а только для вызова служб из клиента. Например, политика для службы SharePoint применяется ко всем клиентам, которые вызывают SharePoint. Политика, заданная для Exchange, применяется к любой попытке доступа к электронной почте из клиента Outlook. По этой причине клиентские приложения (открытые и собственные) не предлагаются для выбора в окне выбора облачных приложений, а настройка условного доступа отсутствует в параметрах клиентского приложения (открытого или собственного), которое регистрируется в арендаторе. 

## <a name="user-actions"></a>Действия пользователя

Действиями пользователя называются задачи, которые может выполнять пользователь. В настоящее время условный доступ поддерживает два действия пользователя: 

- **Регистрация сведений о безопасности**. это действие пользователя позволяет применять политику условного доступа, когда пользователи, которым разрешена Объединенная регистрация, могут зарегистрировать свои сведения о безопасности. Дополнительные сведения см. в статье об [объединенной регистрации сведений о безопасности](../authentication/concept-registration-mfa-sspr-combined.md).

- **Регистрация или присоединение устройств (Предварительная версия)**. это действие пользователя позволяет администраторам применять политику условного доступа, когда пользователи [регистрируют](../devices/concept-azure-ad-register.md) устройства в Azure AD или [присоединяются](../devices/concept-azure-ad-join.md) к ним. Существует два основных аспекта этого действия пользователя: 
   - `Require multi-factor authentication` доступен только для контроля доступа с этим действием пользователя, и все остальные отключены. Это ограничение предотвращает конфликты с элементами управления доступом, которые либо зависят от регистрации устройств Azure AD, либо неприменимы к регистрации устройств Azure AD. 
   - При включении политики условного доступа с этим действием пользователя необходимо установить для   >  **устройств Azure Active Directory устройствам**  >  **значение**  -  `Devices to be Azure AD joined or Azure AD registered require Multi-Factor Authentication` **нет**. В противном случае политика условного доступа с этим действием пользователя не применяется надлежащим образом. Дополнительные сведения об этом параметре устройства можно найти в окне " [Настройка параметров устройства](../device-management-azure-portal.md##configure-device-settings)". Это действие пользователя обеспечивает гибкость, необходимую для регистрации или присоединения устройств для конкретных пользователей и групп или условий вместо политики клиента в параметрах устройства. 
   
## <a name="next-steps"></a>Дальнейшие действия

- [Условный доступ. Условия](concept-conditional-access-conditions.md)

- [Распространенные политики условного доступа](concept-conditional-access-policy-common.md)
- [Зависимости клиентского приложения](service-dependencies.md)
