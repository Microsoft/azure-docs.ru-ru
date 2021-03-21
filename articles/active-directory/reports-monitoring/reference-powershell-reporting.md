---
title: Командлеты Azure AD PowerShell для создания отчетов | Документация Майкрософт
description: Справочник по командлетам Azure AD PowerShell для создания отчетов.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: a1f93126-77d1-4345-ab7d-561066041161
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 08/07/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 329036f3ed815eaaba94f441e372f4be86edd629
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102558089"
---
# <a name="azure-ad-powershell-cmdlets-for-reporting"></a>Командлеты Azure AD PowerShell для создания отчетов

> [!NOTE] 
> Сейчас эти командлеты PowerShell работают только с модулем [предварительной версии Azure AD](/powershell/module/azuread/?view=azureadps-2.0-preview&preserve-view=true#directory_auditing) . Обратите внимание, что модуль предварительной версии не предлагается для использования в рабочей среде. 

Чтобы установить общедоступный предварительный выпуск, используйте следующую версию. 

```powershell
Install-module AzureADPreview
```

Дополнительные сведения о подключении к Azure AD с помощью PowerShell см. в статье [Azure AD PowerShell для Graph](/powershell/azure/active-directory/install-adv2).  

С помощью отчетов Azure Active Directory (Azure AD) можно получить сведения о действиях, связанных со всеми операциями записи в вашем направлении (журналы аудита) и данными проверки подлинности (журналы входа). Хотя сведения доступны с помощью MS API Graph, теперь вы можете получить те же данные с помощью командлетов Azure AD PowerShell для создания отчетов.

В этой статье приводятся общие сведения о командлетах PowerShell, используемых для журналов аудита и журналов входа.

## <a name="audit-logs"></a>Журналы аудита

[Журналы аудита](concept-audit-logs.md) обеспечивают трассировку с помощью журналов для всех изменений, выполненных различными функциями в Azure AD. Примеры журналов аудита включают изменения, внесенные в такие ресурсы в Azure AD, как добавление или удаление пользователей, приложений, групп, ролей и политик.

Получить доступ к журналам аудита можно с помощью командлета Get-Азуреадаудитдиректорилогс.


| Сценарий                      | Команда PowerShell |
| :--                           | :--                |
| Отображаемое имя приложения      | Get-AzureADAuditDirectoryLogs-Filter "Инитиатедби/App/displayName EQ" облачная синхронизация Azure AD "" |
| Category                      | Get-AzureADAuditDirectoryLogs-фильтровать "Category EQ" Аппликатионманажемент "" |
| Дата и время действия            | Get-AzureADAuditDirectoryLogs-Filter "Активитидатетиме gt 2019-04-18" |
| Все вышеперечисленное.              | Get-AzureADAuditDirectoryLogs-Filter "Инитиатедби/App/displayName EQ" облачная синхронизация Azure AD "и Категория EQ" Аппликатионманажемент "и Активитидатетиме gt 2019-04-18"|


На следующем рисунке показан пример для этой команды. 

![На снимке экрана показан результат выполнения команды Get-Azure D Audit Directory Logs.](./media/reference-powershell-reporting/get-azureadauditdirectorylogs.png)



## <a name="sign-in-logs"></a>Журналы входа

Журналы [входа](concept-sign-ins.md) предоставляют сведения об использовании управляемых приложений и действий входа пользователя.

Получить доступ к журналам входа можно с помощью командлета Get-Азуреадаудитсигнинлогс.


| Сценарий                      | Команда PowerShell |
| :--                           | :--                |
| Отображаемое имя пользователя             | Get-AzureADAuditSignInLogs-Filter "userDisplayName EQ" Тимоти Перкинс "" |
| Создать дату и время              | Get-AzureADAuditSignInLogs-Filter "Креатеддатетиме gt 2019-04-18T17:30:00.0 Z" (все, начиная с 5:30 PM на 4/18) |
| Состояние                        | Get-AzureADAuditSignInLogs-Filter "Status/errorCode EQ 50105" |
| Отображаемое имя приложения      | Get-AzureADAuditSignInLogs-Filter "appDisplayName EQ" Сторефронтстудио [WSFED Enabled] " |
| Все вышеперечисленное.              | Get-AzureADAuditSignInLogs-Filter "userDisplayName EQ" Тимоти Перкинс "and Status/errorCode Ne 0 и appDisplayName EQ" Сторефронтстудио [WSFED Enabled] "" |


На следующем рисунке показан пример для этой команды. 

![На снимке экрана показан результат выполнения команды Get-Azure журнал аудита входа в систему.](./media/reference-powershell-reporting/get-azureadauditsigninlogs.png)



## <a name="next-steps"></a>Следующие шаги

- [Что такое отчеты в Azure Active Directory](overview-reports.md).
- [Отчет о журналах аудита](concept-audit-logs.md). 
- [Программный доступ к отчетам Azure Active Directory](concept-reporting-api.md).
