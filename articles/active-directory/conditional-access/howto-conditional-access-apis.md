---
title: Интерфейсы API условного доступа и PowerShell — Azure Active Directory
description: Использование API условного доступа Azure AD и PowerShell для управления политиками, такими как код
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 09/10/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: videor, jeevanb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6d4dd6e64053af41d66e3accbb7fc9b21d951fc6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96860219"
---
# <a name="conditional-access-programmatic-access"></a>Условный доступ: программный доступ

Многие организации выражают необходимость в управлении как можно большей частью своих сред, например кода. С помощью Microsoft Graph можно рассматривать политики условного доступа как любые другие фрагменты кода в вашей среде.

Microsoft Graph предоставляет унифицированную модель программирования, которую организации могут использовать для взаимодействия с данными в Microsoft 365, Windows 10 и Enterprise Mobility + Security. Дополнительные сведения о Microsoft Graph см. в статье [Общие сведения о Microsoft Graph](/graph/overview).

![Изображение, показывающее основные ресурсы и связи, которые являются частью графа](./media/howto-conditional-access-apis/microsoft-graph.png)

Следующие примеры представлены в виде без поддержки. Эти примеры можно использовать в качестве базиса для инструментов в Организации. 

Во многих из следующих примеров используются такие средства, как [управляемые удостоверения](../managed-identities-azure-resources/overview.md), [Logic Apps](../../logic-apps/logic-apps-overview.md), [OneDrive](https://www.microsoft.com/microsoft-365/onedrive/online-cloud-storage), [команды](https://www.microsoft.com/microsoft-365/microsoft-teams/group-chat-software/)и [Azure Key Vault](../../key-vault/general/overview.md).

## <a name="configure"></a>Configure

### <a name="powershell"></a>PowerShell

Для многих администраторов PowerShell уже является понятным средством создания сценариев. В следующем примере показано, как использовать [модуль Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureAD) для управления политиками условного доступа.

- [Настройка политик условного доступа с помощью команд Azure AD PowerShell](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/01-configure/powershell)

### <a name="graph-api"></a>API Graph

В этом примере показаны основные параметры создания, чтения, обновления и удаления (CRUD), доступные в API графа условного доступа. Пример также включает некоторые шаблоны JSON, которые можно использовать для создания некоторых примеров политик.

- [Настройка политик условного доступа с помощью вызовов API Microsoft Graph](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/01-configure/graphapi)

### <a name="configure-using-templates"></a>Настройка с помощью шаблонов

Используйте API условного доступа для развертывания политик условного доступа в подготовительной среде с помощью шаблона.

- [Настройка политик условного доступа с помощью шаблонов API Microsoft Graph](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/01-configure/templates)

## <a name="test"></a>Проверить

Этот пример моделирует более безопасные методики развертывания с помощью рабочих процессов утверждения, которые могут копировать политики условного доступа из одной среды (например, предварительной) в другую, например в рабочую среду.

- [Повышение уровня политик условного доступа из тестовых сред](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/02-test)

## <a name="deploy"></a>Развернуть

Этот пример предоставляет механизм для постепенного выполнения промежуточного развертывания политик условного доступа к заполнению пользователей, что позволяет управлять влиянием и проблемами поддержки на ранних этапах.

- [Развертывание политик условного доступа в рабочих средах с помощью рабочих процессов утверждения](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/03-deploy)

## <a name="monitor"></a>Монитор

Этот пример предоставляет механизм отслеживания изменений политики условного доступа с течением времени и может активировать предупреждения при изменении ключевых политик.

- [Мониторинг развернутых политик условного доступа для изменений и триггеров оповещений](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/04-monitor)

## <a name="manage"></a>Управление

### <a name="backup-and-restore"></a>Резервное копирование и восстановление

Автоматизируйте резервное копирование и восстановление политик условного доступа с утверждениями в командах, в которых используется этот пример.

- [Управление процессом резервного копирования и восстановления политик условного доступа с помощью Microsoft Graph вызовов API](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/05-manage/01-backup-restore)

### <a name="emergency-access-accounts"></a>Учетные записи для аварийного доступа

Несколько администраторов могут создавать политики условного доступа и могут забыть добавить [учетные записи аварийного доступа](../roles/security-emergency-access.md) в качестве исключения для этих политик. Этот пример гарантирует, что все политики будут обновлены для включения назначенных учетных записей аварийного доступа.

- [Управление назначением учетных записей аварийного доступа в политиках условных запросов с помощью Microsoft Graph вызовов API](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/05-manage/02-emergency-access)

### <a name="contingency-planning"></a>Планирование действий на непредвиденные случаи

Они не всегда работают так, как вам нужно. в этом случае вам нужен способ вернуться к состоянию, в котором можно продолжить работу. В следующем примере показано, как восстановить политики до известного хорошего плана на непредвиденные случаи и отключить другие политики условного доступа.

- [Управление активацией политик на непредвиденные случаи условного доступа с помощью Microsoft Graph вызовов API](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/05-manage/03-contingency)

## <a name="community-contribution"></a>Публикация сообщества

Эти примеры доступны в нашем [репозитории GitHub](https://github.com/Azure-Samples/azure-ad-conditional-access-apis). Мы рады поддерживать вклады сообщества, которые являются исчерпывающими проблемами GitHub и запросами на вытягивание.

## <a name="next-steps"></a>Дальнейшие действия

- [Overview of Microsoft Graph (Обзор Microsoft Graph)](/graph/overview)

- [API условного доступа](/graph/api/resources/conditionalaccesspolicy)

- [API именованного расположения](/graph/api/resources/namedlocation)
