---
title: Условный доступ. Запрос на MFA для управления Azure. Azure Active Directory
description: Создайте политику условного доступа для включения многофакторной проверки подлинности для задач управления Azure
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2e2b6b3e9a6bdead4e4da7f1a829698d86cfbf52
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92366179"
---
# <a name="conditional-access-require-mfa-for-azure-management"></a>Условный доступ: Запрос на MFA для управления Azure

Организации используют различные службы Azure, управляемые через инструменты на базе диспетчера ресурсов Azure, в том числе следующие.

* Портал Azure
* Azure PowerShell
* Azure CLI

Эти инструменты могут предоставлять высокопривилегированный доступ к ресурсам, которые поддерживают изменение конфигурации на уровне подписки, параметры службы и выставление счетов за подписку. Для защиты этих привилегированных ресурсов корпорация Майкрософт рекомендует включить запрос на многофакторную проверку подлинности для любого пользователя, который обращается к этим ресурсам.

## <a name="user-exclusions"></a>Пользовательские исключения

Политики условного доступа представляют собой довольно мощные инструменты, поэтому рекомендуется исключить из политики следующие учетные записи.

* Учетные записи для **аварийного доступа** и учетные записи для **обхода стандартной системы контроля доступа**, чтобы предотвратить блокировку учетной записи на уровне арендатора. Если все администраторы заблокированы для вашего арендатора (хотя это маловероятная ситуация), можно использовать администраторскую учетную запись аварийного доступа для входа в систему арендатора, чтобы выполнить действия по восстановлению доступа.
   * Дополнительные сведения можно найти в статье [Управление учетными записями для аварийного доступа в Azure AD](../roles/security-emergency-access.md).
* **Учетные записи служб** и **участники-службы**, например учетная запись синхронизации Azure AD Connect. Учетные записи служб представляют собой автономные учетные записи, которые не привязаны к какому-либо конкретному пользователю. Они обычно используются службами сервера для предоставления программного доступа к приложениям, но также могут применяться для входа в системы в целях администрирования. Такие учетные записи служб должны быть исключены, так как MFA невозможно выполнить программным способом. Вызовы, выполняемые участниками-службами, не блокируются условным доступом.
   * Если в вашей организации эти учетные записи используются в сценариях или в коде, попробуйте заменить их [управляемыми удостоверениями](../managed-identities-azure-resources/overview.md). В качестве временного решения проблемы можно исключить эти конкретные учетные записи пользователей из базовой политики.

## <a name="create-a-conditional-access-policy"></a>Создание политики условного доступа

Следующие действия используются для создания политики условного доступа, в рамках которой пользователи, имеющие доступ к приложению для [управления Microsoft Azure](concept-conditional-access-cloud-apps.md#microsoft-azure-management), должны пройти многофакторную проверку подлинности.

1. Войдите на **портал Azure** с учетными данными глобального администратора, администратора безопасности или администратора условного доступа.
1. Выберите **Azure Active Directory** > **Безопасность** > **Условный доступ**.
1. Выберите **Новая политика**.
1. Присвойте политике имя. Мы рекомендуем организациям присваивать политикам понятные имена.
1. В разделе **Назначения** выберите **Пользователи и группы**.
   1. В разделе **Включить** выберите **Все пользователи**.
   1. В разделе **Исключить** выберите **Пользователи и группы**, а затем выберите учетные записи для аварийного доступа или для обхода стандартной системы контроля доступа в вашей организации. 
   1. Нажмите кнопку **Готово**.
1. В разделе **Облачные приложения или действия** > **Включить** выберите **Выбрать приложения**, **Управление Microsoft Azure** и **Выбрать**, а затем нажмите **Готово**.
1. В разделе **условия**  >  **клиентские приложения (Предварительная версия)** в разделе **выберите клиентские приложения эта политика будет применяться, чтобы** оставить все значения по умолчанию и выбрать **Готово**.
1. В разделе **Управление доступом** > **Предоставление разрешения** выберите **Предоставить доступ**, **Запрос на многофакторную проверку подлинности**, а затем нажмите **Выбрать**.
1. Подтвердите параметры и задайте для параметра **Включить политику** значение **Включить**.
1. Нажмите **Создать**, чтобы создать и включить политику.

## <a name="next-steps"></a>Дальнейшие действия

[Распространенные политики условного доступа](concept-conditional-access-policy-common.md)

[Определение влияния условного доступа в режиме "только отчет"](howto-conditional-access-insights-reporting.md)

[Моделирование поведения входа с помощью средства What If условного доступа](troubleshoot-conditional-access-what-if.md)
