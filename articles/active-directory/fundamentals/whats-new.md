---
title: Новые возможности Заметки о выпуске Azure Active Directory | Документация Майкрософт
description: Узнайте о новых возможностях Azure Active Directory; Например, последние заметки о выпуске, известные проблемы, исправления ошибок, устаревшие функции и предстоящие изменения.
services: active-directory
author: ajburnle
manager: daveba
featureFlags:
- clicktale
ms.assetid: 06a149f7-4aa1-4fb9-a8ec-ac2633b031fb
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 3/4/2021
ms.author: ajburnle
ms.reviewer: dhanyahk
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0d9769a2cfdbd5f552e97a6cd665263cbd488325
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104592971"
---
# <a name="whats-new-in-azure-active-directory"></a>Новые возможности Azure Active Directory

>Получайте уведомления о том, когда следует повторно посетить эту страницу для обновлений, скопировав и вставив URL-адрес `https://docs.microsoft.com/api/search/rss?search=%22Release+notes+-+Azure+Active+Directory%22&locale=en-us` в средство чтения RSS-канала ![значок средства чтения RSS](./media/whats-new/feed-icon-16x16.png).

Azure AD усовершенствуется на постоянной основе. Чтобы вы оставались в курсе последних разработок, в этой статье предоставлены такие сведения:

- Последние выпуски.
- Известные проблемы
- Исправления ошибок
- Нерекомендуемые функции.
- Планы по изменениям.

Эта страница обновляется ежемесячно, поэтому регулярно пересматривайте ее. Если вы ищете элементы старше шести месяцев, их можно найти в [архиве для новых возможностей Azure Active Directory](whats-new-archive.md).

---
## <a name="february-2021"></a>Февраль 2021 года

### <a name="email-one-time-passcode-authentication-on-by-default-starting-october-2021"></a>Проверка подлинности по одноразовому секретному код по умолчанию начинается с 2021 октября

**Тип.** Планирование изменений.  
**Категория службы.** B2B.  
**Возможности продукта.** B2B и B2C.
 

Начиная с 31 октября 2021 Microsoft Azure Active Directory [Проверка подлинности по электронной почте](../external-identities/one-time-passcode.md) становится методом по умолчанию для приглашения учетных записей и клиентов для сценариев совместной работы B2B. В настоящее время корпорация Майкрософт больше не будет разрешать погашение приглашений с использованием неуправляемых учетных записей Azure Active Directory. 

---

### <a name="unrequested-but-consented-permissions-will-no-longer-be-added-to-tokens-if-they-would-trigger-conditional-access"></a>Незапрошенные разрешения больше не будут добавляться в токены, если они активируют условный доступ.

**Тип.** Планирование изменений.  
**Категория службы.** Проверки подлинности (имена входа).  
**Возможности продукта.** Платформа.
 
В настоящее время приложениям, использующим [динамические разрешения](../develop/v2-permissions-and-consent.md#requesting-individual-user-consent) , предоставляются все разрешения, к которым они имеют доступ. Сюда входят незапрошенные приложения, даже если они активируют условный доступ. Например, это может привести к тому, что приложение, запрашивающее только запрос, `user.read` также будет иметь согласие на `files.read` , чтобы принудительно передать условный доступ, назначенный данному `files.read` разрешению. 

Чтобы сократить количество ненужных запросов условного доступа, Azure AD изменяет способ, которым в приложениях предоставляются незапрошенные области. Приложения активируют условный доступ только для тех разрешений, которые им явно запрашивают. Дополнительные сведения см. [в статье новые возможности проверки подлинности](../develop/reference-breaking-changes.md#conditional-access-will-only-trigger-for-explicitly-requested-scopes).
 
---
 
### <a name="public-preview---use-a-temporary-access-pass-to-register-passwordless-credentials"></a>Общедоступная Предварительная версия — используйте временный проход доступа для регистрации учетных данных, не имеющих пароля.

**Тип.** Новая функция.  
**Категория службы:** MFA  
**Возможности продукта.** Безопасность и защита удостоверений.

Временный проход доступа — это секретный код с ограниченным временем, который служит надежными учетными данными и позволяет подключать учетные данные без пароля и восстановления, когда пользователь теряет или забыл свой надежный фактор проверки подлинности (например, FIDO2 ключ безопасности или Microsoft Authenticator) и должен войти для регистрации новых методов проверки подлинности. [Подробнее](../authentication/howto-authentication-temporary-access-pass.md).

---

### <a name="public-preview---keep-me-signed-in-kmsi-in-next-generation-of-user-flows"></a>Общедоступная Предварительная версия — оставаться в системе (функции "оставаться) в следующем поколении потоков пользователей

**Тип.** Новая функция.  
**Категория службы.** B2C — управление удостоверениями клиентов.  
**Возможности продукта.** B2B и B2C.

Новое поколение B2C User Flows теперь поддерживает функцию " [оставаться в системе" (функции "оставаться)](../../active-directory-b2c/session-behavior.md?pivots=b2c-custom-policy#enable-keep-me-signed-in-kmsi) , которая позволяет клиентам продлить время существования сеанса для пользователей своих веб-приложений и собственного приложения с помощью постоянного файла cookie.  функция сохраняет активный сеанс даже в том случае, когда пользователь закрывает и снова открывает браузер и отменяется, когда пользователь выходит из него.

---

### <a name="public-preview---external-identities-self-service-sign-up-in-aad-using-msa-accounts"></a>Общедоступная Предварительная версия — внешние удостоверения Self-Service зарегистрироваться в AAD с помощью учетных записей MSA

**Тип.** Новая функция.  
**Категория службы.** B2B.  
**Возможности продукта.** B2B и B2C.
 
Внешние пользователи теперь смогут использовать учетные записи Майкрософт для входа в первую сторону Azure AD и бизнес-приложения. [Подробнее](../external-identities/self-service-sign-up-overview.md).

---

### <a name="public-preview---reset-redemption-status-for-a-guest-user"></a>Общедоступная Предварительная версия — сброс состояния активации для гостевого пользователя

**Тип.** Новая функция.  
**Категория службы.** B2B.  
**Возможности продукта.** B2B и B2C.
 
Теперь клиенты могут повторно приглашать существующих внешних гостевых пользователей, чтобы сбросить их состояние активации, что позволит сохранить учетную запись гостевого пользователя без потери доступа. [Подробнее](../external-identities/reset-redemption-status.md).
 
---

### <a name="public-preview---synchronization-provisioning-apis-now-support-application-permissions"></a>Общедоступная Предварительная версия — API-интерфейсы/синчронизатион (подготовка) теперь поддерживают разрешения приложения.

**Тип.** Новая функция.  
**Категория службы.** Подготовка приложений.  
**Возможности продукта:** Управление жизненным циклом удостоверений
 
Теперь клиенты могут использовать Application. ReadWrite. овнедби в качестве разрешения приложения для вызова API-интерфейсов синхронизации. Примечание. это поддерживается только при подготовке из Azure AD в сторонние приложения (например, AWS, модули данных и т. д.). Сейчас она не поддерживается для подготовки кадров (Workday/Successfactors) или облачной синхронизации (AD в Azure AD). [Подробнее](/graph/api/resources/provisioningobjectsummary?view=graph-rest-beta).
 
---

### <a name="general-availability---authentication-policy-administrator-built-in-role"></a>Общая доступность — встроенная роль администратора политики проверки подлинности

**Тип.** Новая функция.  
**Категория службы:** RBAC  
**Возможность продукта.** Контроль доступа.
 
Пользователи с этой ролью могут настраивать политики методов проверки подлинности, параметры MFA для всего клиента и политику защиты паролем. Эта роль предоставляет разрешение на управление параметрами защиты паролем: Конфигурация смарт-блокировки и обновление настраиваемого списка запрещенных паролей. [Подробнее](../roles/permissions-reference.md#authentication-policy-administrator).

---

### <a name="general-availability---user-collections-on-my-apps-are-available-now"></a>Общедоступная доступность — коллекции пользователей в моих приложениях теперь доступны сейчас!

**Тип.** Новая функция.  
**Категория службы.** "Мои приложения".  
**Возможности продукта:** Взаимодействие с конечным пользователем
 
Теперь пользователи могут создавать собственные группы приложений в средстве запуска приложения "Мои приложения". Они также могут переупорядочивать и скрывать коллекции, совместно используемые их администратором. [Подробнее](../user-help/my-apps-portal-user-collections.md).

---

### <a name="general-availability---autofill-in-authenticator"></a>Общая доступность — автозаполнение в средствах проверки подлинности

**Тип.** Новая функция.  
**Категория службы:** Приложение Microsoft Authenticator  
**Возможности продукта.** Безопасность и защита удостоверений.
 
Microsoft Authenticator предоставляет возможности многофакторной проверки подлинности (MFA) и управления учетными записями, и теперь также будет выполнять автозаполнение паролей на сайтах и в приложениях, которые пользователи посещают на мобильных устройствах (iOS и Android). 

Чтобы использовать автозаполнение для проверки подлинности, пользователям необходимо добавить свои персональные учетная запись Майкрософт в средство проверки подлинности и использовать его для синхронизации паролей. Рабочие или учебные учетные записи нельзя использовать для синхронизации паролей в настоящее время. [Подробнее](../user-help/user-help-auth-app-faq.md#autofill-for-it-admins).

---

### <a name="general-availability---invite-internal-users-to-b2b-collaboration"></a>Общая доступность. Приглашаем внутренних пользователей в службу совместной работы B2B

**Тип.** Новая функция.  
**Категория службы.** B2B.  
**Возможности продукта.** B2B и B2C.
 
Теперь клиенты могут приглашать внутренних гостей для использования службы совместной работы B2B вместо отправки приглашения в существующую внутреннюю учетную запись. Это позволяет клиентам защитить идентификатор объекта пользователя, имя участника-пользователя, членство в группах и назначения приложений. [Подробнее](../external-identities/invite-internal-users.md).

---

### <a name="general-availability---domain-name-administrator-built-in-role"></a>Общая доступность — встроенная роль администратора доменного имени

**Тип.** Новая функция.  
**Категория службы:** RBAC  
**Возможность продукта.** Контроль доступа.
 
Пользователи с этой ролью могут управлять доменными именами (чтение, добавление, проверка, обновление и удаление). Они также могут считывать данные каталога о пользователях, группах и приложениях, так как эти объекты имеют зависимости домена. 

Для локальных сред пользователи с этой ролью могут настроить доменные имена для Федерации, чтобы связанные пользователи всегда выполняли проверку подлинности в локальной среде. Затем эти пользователи могут войти в службы на основе Azure AD со своими локальными паролями через единый вход. Параметры Федерации необходимо синхронизировать через Azure AD Connect, поэтому у пользователей также есть разрешения на управление Azure AD Connect. [Подробнее](../roles/permissions-reference.md#domain-name-administrator).
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---february-2021"></a>Новые Федеративные приложения, доступные в коллекции приложений Azure AD — Февраль 2021

**Тип.** Новая функция.  
**Категория службы:** Корпоративные приложения  
**Возможности продукта.** Интеграция с решениями сторонних производителей.
 
В феврале 2021 мы добавили в коллекцию приложений следующие 37 новых приложений с поддержкой федерации:

[Добавочное расширение Messenger](https://loopworks.com/loop-flow-messenger/), [адаптер силверфорт Azure AD](http://www.silverfort.com/), [взаимозависимостьное обучение](https://skilledtrades.interplaylearning.com/#login), [Нура пространство](https://dashboard.nuraspace.com/login)Йооз, [ЕС](https://eu1.getyooz.com/?kc_idp_hint=microsoft), [укспрессиа](https://uxpressia.com/users/sign-in), интродус, [пред-и интегрированная платформа](http://app.introdus.dk/login), [Хаппибот](https://login.microsoftonline.com/organizations/oauth2/v2.0/authorize?client_id=34353e1e-dfe5-4d2f-bb09-2a5e376270c8&response_type=code&redirect_uri=https://api.happyteams.io/microsoft/integrate&response_mode=query&scope=offline_access%20User.Read%20User.Read.All), [леаксид](https://app.leaksid.com/), [ШИФТВИЗАРД](http://www.shiftwizard.com/), [ПИНГФЛОВ SSO](https://app.pingview.io/), [свифтлане](https://admin.swiftlane.com/login), [Quasydoc SSO](https://www.quasydoc.eu/login), [Fenwick Gold account](https://businesscentral.dynamics.com/), [SeamlessDesk](https://www.seamlessdesk.com/login), [Learnsoft LMS & TMS](http://www.learnsoft.com/), [P-TH +](https://p-th.jp/), [MyViewBoard](https://api.myviewboard.com/auth/microsoft/), [Tartabit IOT мост](https://bridge-us.tartabit.com/), [AKASHI](../saas-apps/akashi-tutorial.md), [](../saas-apps/enablon-tutorial.md) [пересмотреть](../saas-apps/rewatch-tutorial.md), [Zuddl](../saas-apps/zuddl-tutorial.md), [Parkalot — управление приостановкой автомобиля](../saas-apps/parkalot-car-park-management-tutorial.md), [HSB ThoughtSpot](../saas-apps/hsb-thoughtspot-tutorial.md), [IBMid](../saas-apps/ibmid-tutorial.md), [SharingCloud](../saas-apps/sharingcloud-tutorial.md), [PoolParty](../saas-apps/poolparty-semantic-suite-tutorial.md), [GlobeSmart,](../saas-apps/globesmart-tutorial.md) [Samsung Knox и Business Services](../saas-apps/samsung-knox-and-business-services-tutorial.md) [,](../saas-apps/klaxoon-saml-tutorial.md) [Penji](../saas-apps/penji-tutorial.md), [, платформа](../saas-apps/sigma-computing-tutorial.md) [Agile](../saas-apps/kendis-scaling-agile-platform-tutorial.md), [Kendis, Maptician](../saas-apps/maptician-tutorial.md) [, Olfeo](../saas-apps/olfeo-saas-tutorial.md) [](../saas-apps/cloudknox-permissions-management-platform-tutorial.md)

Документацию по всем приложениям можно также найти здесь: https://aka.ms/AppsTutorial

Чтобы получить список приложений в коллекции приложений Azure AD, ознакомьтесь со сведениями в этой статье: https://aka.ms/AzureADAppRequest

--- 

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---february-2021"></a>Новые соединители подготовки в коллекции приложений Azure AD — Февраль 2021

**Тип.** Новая функция.  
**Категория службы.** Подготовка приложений.  
**Возможности продукта.** Интеграция с решениями сторонних производителей.
 

Теперь вы можете автоматизировать процессы создания, обновление и удаление учетных записей пользователей для следующих новых интегрированных приложений:

- [Atea](../saas-apps/atea-provisioning-tutorial.md)
- [GetAbstract](../saas-apps/getabstract-provisioning-tutorial.md)
- [HelloID](../saas-apps/helloid-provisioning-tutorial.md)
- [Hoxhunt](../saas-apps/hoxhunt-provisioning-tutorial.md)
- [Iris Intranet](../saas-apps/iris-intranet-provisioning-tutorial.md)
- [Preciate](../saas-apps/preciate-provisioning-tutorial.md)

Дополнительные сведения см. в статье [Автоматизация подготовки пользователей в приложениях SaaS с помощью Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="general-availability---10-azure-active-directory-roles-now-renamed"></a>Общая доступность-10 Azure Active Directory ролей теперь переименованы

**Тип.** Измененная функция.  
**Категория службы:** RBAC  
**Возможность продукта.** Контроль доступа.
 
10 встроенных ролей Azure AD были переименованы, поэтому они будут выровнены по [центру администрирования Microsoft 365](/microsoft-365/admin/microsoft-365-admin-center-preview), на [портале Azure AD](https://portal.azure.com/)и [Microsoft Graph](https://developer.microsoft.com/graph/). Дополнительные сведения о новых ролях см. в разделе [Администратор роли администратора в Azure Active Directory](../roles/permissions-reference.md#all-roles).

![Таблица, в которой показаны имена ролей в MS API Graph и портал Azure, а также предложенное окончательное имя для API, портал Azure и Mac.](media/whats-new/roles-table-rbac.png)

---

### <a name="new-company-branding-in-mfasspr-combined-registration"></a>Добавление фирменной символики компании в Объединенную регистрацию MFA/SSPR

**Тип.** Измененная функция.  
**Категория службы:** Взаимодействие с пользователем и управление  
**Возможности продукта:** Взаимодействие с конечным пользователем
 
В прошлом логотипы компании не использовались на страницах входа Azure Active Directory. Фирменная символика компании теперь расположена в верхней левой части многофакторной регистрации MFA/SSPR. Фирменная символика компании также включена на странице "Мои Sign-Ins" и "сведения о безопасности". [Подробнее](../fundamentals/customize-branding.md).

---

### <a name="general-availability---second-level-manager-can-be-set-as-alternate-approver"></a>Общая доступность — диспетчер второго уровня можно задать в качестве альтернативного утверждающего.

**Тип.** Измененная функция.  
**Категория службы:** Управление доступом пользователей  
**Возможности продукта:** Управление назначениями
 
Дополнительный параметр при выборе утверждающих теперь доступен в управлении назначением. Если выбрать "руководитель как утверждающий" для первого утверждающего лица, у вас будет еще один параметр "второй уровень менеджера в качестве альтернативного утверждающего", доступный для выбора в поле альтернативное утверждение. Если выбран этот параметр, необходимо добавить утверждающего утверждающий, чтобы перенаправить запрос в, если система не может найти диспетчер второго уровня. [Подробнее](../governance/entitlement-management-access-package-approval-policy.md#alternate-approvers).
 
---

### <a name="authentication-methods-activity-dashboard"></a>Панель мониторинга действий методов проверки подлинности

**Тип.** Измененная функция.  
**Категория службы.** Отчеты.  
**Возможности продукта.** Мониторинг и отчетность.
 

Обновленные методы проверки подлинности на панели мониторинга предоставляют администраторам общие сведения о действиях по регистрации и использованию методов проверки подлинности в клиенте. В отчете суммируется количество пользователей, зарегистрированных для каждого метода, а также методы, используемые при входе и сбросе пароля. [Подробнее](../authentication/howto-authentication-methods-activity.md).
 
---

### <a name="refresh-and-session-token-lifetimes-configurability-in-configurable-token-lifetime-ctl-are-retired"></a>Возможность настройки времени существования маркера обновления и сеанса в настраиваемом сроке действия токена (CTL) прекращена

**Тип.** Прекращение поддержки.  
**Категория службы.** Другая.  
**Возможности продукта:** Проверка подлинности пользователя
 
Возможность настройки времени существования маркера обновления и сеанса в списке CTL прекращена. Azure Active Directory больше не учитывает конфигурацию маркера обновления и сеанса в существующих политиках. [Подробнее](../develop/active-directory-configurable-token-lifetimes.md#token-lifetime-policies-for-refresh-tokens-and-session-tokens).
 
---
 
## <a name="january-2021"></a>Январь 2021 г.

### <a name="secret-token-will-be-a-mandatory-field-when-configuring-provisioning"></a>Токен секрета будет обязательным полем при настройке подготовки

**Тип.** Планирование изменений.  
**Категория службы.** Подготовка приложений.  
**Возможности продукта:** Управление жизненным циклом удостоверений

В прошлом поле секретного токена можно было оставить пустым при настройке подготовки в пользовательском приложении/БЙОА. Эта функция предназначена только для тестирования. Мы будем обновлять пользовательский интерфейс, чтобы сделать поле обязательным. 

Клиенты могут обойти это требование для целей тестирования, используя флаг функции в URL-адресе браузера. [Подробнее](../app-provisioning/use-scim-to-provision-users-and-groups.md#authorization-to-provisioning-connectors-in-the-application-gallery).
 
---

### <a name="public-preview---customize-and-configure-android-shared-devices-for-frontline-workers-at-scale"></a>Общедоступная Предварительная версия. Настройка и настройка общих устройств Android для рабочих ролей оказался в масштабе

**Тип.** Новая функция.  
**Категория службы:** Регистрация устройств и управление ими  
**Возможности продукта.** Безопасность и защита удостоверений.
 
Группы Azure AD и Microsoft Endpoint Manager сочетаются с возможностью настройки, масштабирования и защиты рабочих устройств оказался.

Следующие возможности предварительной версии позволяют:
- Подготавливает общие устройства Android в масштабе с помощью Microsoft Endpoint Manager
- Защита доступа для сменных рабочих ролей с помощью условного доступа на основе устройств
- Настройка интерфейса входа для сменных рабочих ролей с помощью управляемого главного экрана

Дополнительные сведения см. в статье [Настройка и настройка общих устройств для рабочих ролей оказался в масштабе](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/customize-and-configure-shared-devices-for-firstline-workers-at/ba-p/1751708).

---

### <a name="public-preview---provisioning-logs-can-now-be-downloaded-as-a-csv-or-json"></a>Общедоступная Предварительная версия. журналы подготовки теперь можно скачать в виде CSV-файла или JSON.

**Тип.** Новая функция.  
**Категория службы.** Подготовка приложений.  
**Возможности продукта:** Управление жизненным циклом удостоверений

Клиенты могут скачивать журналы подготовки в виде CSV-файла или JSON с помощью пользовательского интерфейса и API Graph. Дополнительные сведения см. в разделе [Подготовка отчетов на портале Azure Active Directory](../reports-monitoring/concept-provisioning-logs.md).

---

### <a name="public-preview---assign-cloud-groups-to-azure-ad-custom-roles-and-admin-unit-scoped-roles"></a>Общедоступная Предварительная версия. Назначьте облачные группы настраиваемым ролям Azure AD и ролям с областью администратора.

**Тип.** Новая функция.  
**Категория службы:** RBAC  
**Возможность продукта.** Контроль доступа.
 
Клиенты могут назначить облачную группу настраиваемым ролям Azure AD или роли с областью администратора. Сведения об использовании этой функции см. в разделе [Использование облачных групп для управления назначениями ролей в Azure Active Directory](../roles/groups-concept.md).

---

### <a name="general-availability---azure-ad-connect-cloud-sync-previously-known-as-cloud-provisioning"></a>Общая доступность — Azure AD Connect облачная синхронизация (ранее известная как подготовка облака)

**Тип.** Новая функция.  
**Категория службы:** Azure AD Connect облачной синхронизации  
**Возможности продукта:** Управление жизненным циклом удостоверений
 
Azure AD Connect облачная синхронизация теперь доступна для всех клиентов.

Azure AD Connectное облако переносит тяжелую часть логики преобразования в облако, уменьшая объем локальных ресурсов. Кроме того, для более высокой доступности синхронизации доступны несколько развертываний агентов с небольшими плотностьми. [Подробнее](https://aka.ms/cloudsyncGA).
 
---
### <a name="general-availability---attack-simulation-administrator-and-attack-payload-author-built-in-roles"></a>Общая доступность — администратор имитации атак и автор полезных данных атаки создание встроенных ролей

**Тип.** Новая функция.  
**Категория службы:** RBAC  
**Возможность продукта.** Контроль доступа.
 
Для назначения пользователям доступны две новые роли в Role-Based управления доступом, администратор имитации атак и автор полезных данных атаки. 

Пользователи с ролью [администратора имитации атак](../roles/permissions-reference.md#attack-simulation-administrator) имеют доступ ко всем имитациям в клиенте и могут:
- Создание и управление всеми аспектами создания симуляции атак
- Запуск и планирование моделирования
-  Проверьте результаты имитации. 

Пользователи в роли " [Автор полезных данных атаки](../roles/permissions-reference.md#attack-payload-author) " могут создавать полезные данные для атак, но не запускаются и не планируются. Полезные данные для атак становятся доступными для всех администраторов в клиенте, которые могут использовать их для создания симуляции.

---

### <a name="general-availability---usage-summary-reports-reader-built-in-role"></a>Общая доступность — встроенная роль читателя сводных отчетов об использовании

**Тип.** Новая функция.  
**Категория службы:** RBAC  
**Возможность продукта.** Контроль доступа.
 
Пользователи с ролью читателя сводные отчеты об использовании могут получать доступ к агрегированным данным уровня клиента и связанным аналитикам в Microsoft 365 центре администрирования для оценки эффективности использования и производительности. Однако они не могут получить доступ к сведениям о уровне пользователя или аналитике. 

В центре администрирования Microsoft 365 для двух отчетов мы будем отличать Объединенные данные уровня клиента и сведения о уровне пользователя. Эта роль добавляет дополнительный уровень защиты для отдельных данных, идентифицируемых пользователем. [Подробнее](../roles/permissions-reference.md#usage-summary-reports-reader).

---

### <a name="general-availability---require-app-protection-policy-grant-in-azure-ad-conditional-access"></a>Общая доступность — требуется предоставление политики защиты приложений в условном доступе Azure AD

**Тип:** Новая функция  
**Категория службы:** Условный доступ  
**Возможности продукта.** Безопасность и защита удостоверений.
 
Предоставление условного доступа Azure AD для "требуется политика защиты приложений" теперь является общедоступным. 

Политика предоставляет следующие возможности.
- Разрешает доступ только при использовании мобильного приложения, поддерживающего защиту приложений Intune
- Разрешает доступ, только если пользователь имеет политику защиты приложений Intune, доставляемую в мобильное приложение

Дополнительные сведения о настройке политики условного доступа для защиты приложений см. [здесь](../conditional-access/app-protection-based-conditional-access.md).
 
---

### <a name="general-availability---email-one-time-passcode"></a>Общая доступность — One-Time секретный код электронной почты

**Тип.** Новая функция.  
**Категория службы.** B2B.  
**Возможности продукта.** B2B и B2C.
 
OTP позволяет организациям по всему миру сотрудничать с любым пользователем, отправляя ссылку или приглашение по электронной почте. Приглашенные пользователи могут проверить свою личность с помощью одноразового секретного кода, отправляемого в сообщение электронной почты для доступа к ресурсам партнера. [Подробнее](../external-identities/one-time-passcode.md). 
 
---

 ### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---january-2021"></a>Новые соединители подготовки в коллекции приложений Azure AD — Январь 2021

**Тип.** Новая функция.  
**Категория службы.** Подготовка приложений.  
**Возможности продукта.** Интеграция с решениями сторонних производителей.
 
Теперь вы можете автоматизировать процессы создания, обновление и удаление учетных записей пользователей для следующих новых интегрированных приложений:
- [Fortes Change Cloud](../saas-apps/fortes-change-cloud-provisioning-tutorial.md)
- [Gtmhub](../saas-apps/gtmhub-provisioning-tutorial.md)
- [monday.com](../saas-apps/mondaycom-provisioning-tutorial.md)
- [Splashtop](../saas-apps/splashtop-provisioning-tutorial.md)
- [Templafy OpenID Connect](../saas-apps/templafy-openid-connect-provisioning-tutorial.md)
- [WEDO](../saas-apps/wedo-provisioning-tutorial.md)

Дополнительные сведения см [. в статье что такое автоматизированная подготовка пользователей приложения SaaS в Azure AD?](../app-provisioning/user-provisioning.md)

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---january-2021"></a>Новые Федеративные приложения, доступные в коллекции приложений Azure AD — Январь 2021

**Тип.** Новая функция.  
**Категория службы:** Корпоративные приложения  
**Возможности продукта.** Интеграция с решениями сторонних производителей.

В январе 2021 мы добавили в коллекцию приложений следующие 29 новых приложений с поддержкой федерации:

[мисквиев](https://dev.myscview.com/), [талентеч](https://talentech.com/contact/), [бипсинк](https://www.bipsync.com/), [оротимешит](https://app.orotimesheet.com/login.php), [МиО](https://app.m.io/auth/install/microsoft?scopetype=hub), [совелто Easy](https://login.soveltoeasy.fi/), [СУППОРТБЕНЧ](https://account.supportbench.net/agent/login/),[биенвенуеие](https://formation.bienvenue.pro/login), [аидае единый вход](https://aidaforparents.com/login/organizations), [международные продукты поддержки SOS](../saas-apps/international-sos-assistance-products-tutorial.md), [навекс один](../saas-apps/navex-one-tutorial.md), [LabLog](../saas-apps/lablog-tutorial.md), [Oktopost SAML](../saas-apps/oktopost-saml-tutorial.md), [EPHOTO Dam](../saas-apps/ephoto-dam-tutorial.md), концепция, [Syndio](../saas-apps/syndio-tutorial.md), [Yello Enterprise](../saas-apps/yello-enterprise-tutorial.md), [TimeClock 365 SAML](../saas-apps/timeclock-365-saml-tutorial.md) [,](../saas-apps/notion-tutorial.md) [Nalco E-Data](https://www.ecolab.com/), [дежурный](https://app.vacancy-filler.co.uk/VFMVC/Account/Login)процесс, [Synerise экосистема роста искусственного интеллекта](../saas-apps/synerise-ai-growth-ecosystem-tutorial.md), Imperva Data [Security,](../saas-apps/imperva-data-security-tutorial.md) [Illusive](../saas-apps/contentsquare-sso-tutorial.md) [Networks](../saas-apps/illusive-networks-tutorial.md) [, PROWARE,-](../saas-apps/proware-tutorial.md) [Splan 81](../saas-apps/aruba-user-experience-insight-tutorial.md) [](../saas-apps/splan-visitor-tutorial.md) [](../saas-apps/perimeter-81-tutorial.md) [](../saas-apps/burp-suite-enterprise-edition-tutorial.md)

Документацию по всем приложениям можно также найти здесь. https://aka.ms/AppsTutorial

Чтобы получить список приложений в коллекции приложений Azure AD, ознакомьтесь со сведениями в этой статье. https://aka.ms/AzureADAppRequest 

---

### <a name="public-preview---second-level-manager-can-be-set-as-alternate-approver"></a>Общедоступная Предварительная версия — диспетчер второго уровня можно задать в качестве альтернативного утверждающего.

**Тип.** Измененная функция.  
**Категория службы:** Управление доступом пользователей  
**Возможности продукта:** Управление назначениями
 
Дополнительный параметр при выборе утверждающих теперь доступен в управлении назначением. Если выбрать "руководитель как утверждающий" для первого утверждающего лица, у вас будет еще один параметр "второй уровень менеджера в качестве альтернативного утверждающего", доступный для выбора в поле альтернативное утверждение. Если выбран этот параметр, необходимо добавить утверждающего утверждающий, чтобы перенаправить запрос в, если система не может найти диспетчер второго уровня. [Дополнительные сведения](../governance/entitlement-management-access-package-approval-policy.md#alternate-approvers)
 
---

### <a name="general-availability---navigate-to-teams-directly-from-my-access-portal"></a>Общие сведения о доступности — переход к командам непосредственно с моего портала доступа

**Тип.** Измененная функция.  
**Категория службы:** Управление доступом пользователей  
**Возможности продукта:** Управление назначениями
 
Теперь команды можно запускать непосредственно на портале My Access. 

Для этого войдите в мой доступ ( https://myaccess.microsoft.com/) , перейдите к разделу "доступ к пакетам", перейдите на вкладку "Active", чтобы просмотреть все пакеты доступа, к которым у вас уже есть доступ. При развертывании выбранного пакета Access и наведении указателя мыши на команды можно запустить его, нажав кнопку "Открыть". [Подробнее](../governance/entitlement-management-request-access.md).
 
---

### <a name="improved-logging--end-user-prompts-for-risky-guest-users"></a>Улучшено ведение журнала & End-User запрашивает рискованных гостевых пользователей

**Тип.** Измененная функция.  
**Категория службы:** Защита идентификации  
**Возможности продукта.** Безопасность и защита удостоверений.
 

В журнале и End-User запрашиваются рискованные гостевые пользователи. Дополнительные сведения см. в статье [Защита идентификации и пользователи B2B](../identity-protection/concept-identity-protection-b2b.md).
 
---
 
## <a name="december-2020"></a>Декабрь 2020 г.

### <a name="public-preview---azure-ad-b2c-phone-sign-up-and-sign-in-using-built-in-policy"></a>Общедоступная Предварительная версия — Azure AD B2C телефон регистрации и входа с помощью встроенной политики

**Тип.** Новая функция.  
**Категория службы.** B2C — управление удостоверениями клиентов.  
**Возможности продукта.** B2B и B2C.
 
Регистрация и вход в B2C с помощью встроенной политики позволяют ИТ-администраторам и разработчикам организаций разрешать пользователям входить в систему и подписываться с помощью номера телефона в потоках пользователей. Дополнительные сведения см. в статье [Настройка телефонного входа и вход в систему для потоков пользователей (Предварительная версия)](../../active-directory-b2c/phone-authentication-user-flows.md) .

---

### <a name="general-availability---security-defaults-now-enabled-for-all-new-tenants-by-default"></a>Общая доступность — параметры безопасности по умолчанию теперь включены для всех новых клиентов по умолчанию.

**Тип.** Новая функция.  
**Категория службы.** Другая.  
**Возможности продукта.** Безопасность и защита удостоверений.
 
Чтобы защитить учетные записи пользователей, все новые клиенты, созданные после 12 ноября 2020, будут включены в систему по умолчанию. Параметры безопасности по умолчанию применяют несколько политик, в том числе:
- Требует, чтобы все пользователи и администраторы зарегистрировались для использования MFA с помощью приложения Microsoft Authenticator
- Требуются критические роли администратора для использования MFA каждый раз, когда они входят в систему. Все остальные пользователи будут получать запрос на MFA при необходимости. 
- Устаревшая проверка подлинности будет заблокирована для всего клиента. 

Дополнительные сведения см. в статье [что такое параметры безопасности по умолчанию?](../fundamentals/concept-fundamentals-security-defaults.md)

---

### <a name="general-availability---support-for-groups-with-up-to-250k-members-in-aadconnect"></a>Общая доступность — поддержка групп с членами до 250 000 в AADConnect

**Тип.** Измененная функция.  
**Категория службы.** AD Connect.  
**Возможности продукта:** Управление жизненным циклом удостоверений
 
Корпорация Майкрософт развернула новую конечную точку (API) для Azure AD Connect, которая повышает производительность операций службы синхронизации с Azure Active Directory. При использовании новой [конечной точки версии 2](../hybrid/how-to-connect-sync-endpoint-api-v2.md)вы получаете заметное увеличение производительности при экспорте и импорте в Azure AD. Эта новая конечная точка поддерживает следующие сценарии.

- Синхронизация групп с членами до 250 000
- Выигрыш в производительности при экспорте и импорте в Azure AD

---

### <a name="general-availability---entitlement-management-available-for-tenants-in-azure-china-cloud"></a>Общедоступная доступность. Управление обслуживанием доступно для клиентов в облаке Azure для Китая

**Тип.** Новая функция.  
**Категория службы:** Управление доступом пользователей  
**Возможности продукта:** Управление назначениями
 

Возможности управления назначением теперь доступны для всех клиентов в облаке Azure для Китая. Дополнительные сведения см. на нашем сайте [документации по управлению удостоверениями](https://docs.azure.cn/zh-cn/active-directory/governance/) .

---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---december-2020"></a>Новые соединители подготовки в коллекции приложений Azure AD — Декабрь 2020

**Тип.** Новая функция.  
**Категория службы.** Подготовка приложений.  
**Возможности продукта.** Интеграция с решениями сторонних производителей.

Теперь вы можете автоматизировать процессы создания, обновление и удаление учетных записей пользователей для следующих новых интегрированных приложений:

- [Использование Bizagi Studio для автоматизации цифровых процессов](../saas-apps/bizagi-studio-for-digital-process-automation-provisioning-tutorial.md)
- [CybSafe](../saas-apps/cybsafe-provisioning-tutorial.md)
- [GroupTalk](../saas-apps/grouptalk-provisioning-tutorial.md)
- [PaperCut Cloud Print Management](../saas-apps/papercut-cloud-print-management-provisioning-tutorial.md)
- [Parsable](../saas-apps/parsable-provisioning-tutorial.md)
- [Shopify Plus](../saas-apps/shopify-plus-provisioning-tutorial.md)

Дополнительные сведения об усилении защиты организации с помощью автоматизированной подготовки учетных записей пользователей см. в статье [Автоматическая подготовка пользователей для приложений SaaS в Azure AD](../app-provisioning/user-provisioning.md).
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---december-2020"></a>Новые Федеративные приложения, доступные в коллекции приложений Azure AD — Декабрь 2020

**Тип.** Новая функция.  
**Категория службы:** Корпоративные приложения  
**Возможности продукта.** Интеграция с решениями сторонних производителей.
 
В декабре 2020 мы добавили следующие 18 новых приложений в коллекцию приложений с поддержкой федерации:

[Аварего](../saas-apps/awarego-tutorial.md), [ховнов SSO](https://gethownow.com/), [зилаб одно юридическое хранение](https://www.zylab.com/en/product/legal-hold), [руководство](http://www.guider-ai.com/), [софткрисис](https://www.softcrisis.se/sv/), [ПИМС 365](http://www.omega365.com/pims), [информакаст](../saas-apps/informacast-tutorial.md), [Ретриевермедиадатабасе](../saas-apps/retrievermediadatabase-tutorial.md), [вонаже](../saas-apps/vonage-tutorial.md), [количество меня на панели мониторинга операций](../saas-apps/count-me-in-operations-dashboard-tutorial.md), [пропрофс база знаний](../saas-apps/proprofs-knowledge-base-tutorial.md), [Управление ресурсами RightCrowd](../saas-apps/rightcrowd-workforce-management-tutorial.md), [JLL TRIRIGA](../saas-apps/jll-tririga-tutorial.md), [Shutterstock](../saas-apps/shutterstock-tutorial.md), [брандмауэр веб-приложения FortiWeb](../saas-apps/linkedin-talent-solutions-tutorial.md), [решения LinkedIn](../saas-apps/linkedin-talent-solutions-tutorial.md)для [Equinix Федерации](../saas-apps/equinix-federation-app-tutorial.md), [KFAdvance](../saas-apps/kfadvance-tutorial.md)

Документацию по всем приложениям можно также найти здесь. https://aka.ms/AppsTutorial

Чтобы получить список приложений в коллекции приложений Azure AD, ознакомьтесь со сведениями в этой статье. https://aka.ms/AzureADAppRequest

---

### <a name="navigate-to-teams-directly-from-my-access-portal"></a>Переход к командам непосредственно из портала доступа

**Тип.** Измененная функция.  
**Категория службы:** **Возможности продукта** для управления доступом пользователей: Управление назначениями

Теперь команды можно запускать непосредственно на портале доступа. Для этого войдите в [мой доступ](https://myaccess.microsoft.com/), перейдите к разделу **доступ к пакетам**, а затем перейдите на вкладку **Активная** , чтобы просмотреть все пакеты Access, к которым у вас уже есть доступ. При развертывании пакета Access и наведении указателя мыши на команды можно запустить его, нажав кнопку **Открыть** . 

Дополнительные сведения об использовании портала "мой доступ" см. в статье [запрос доступа к пакету Access в](../governance/entitlement-management-request-access.md#sign-in-to-the-my-access-portal)средстве управления назначением Azure AD.

---

### <a name="public-preview---second-level-manager-can-be-set-as-alternate-approver"></a>Общедоступная Предварительная версия — диспетчер второго уровня можно задать в качестве альтернативного утверждающего.

**Тип.** Измененная функция.  
**Категория службы:** Управление доступом пользователей  
**Возможности продукта:** Управление назначениями

В процессе утверждения в управлении назначением теперь доступен дополнительный параметр. Если выбрать менеджер в качестве утверждающего для первого утверждающего, у вас будет еще один вариант, а второй — менеджер второго уровня как альтернативный утверждающий, доступный для выбора в поле альтернативное утверждение. При выборе этого параметра необходимо добавить утверждающего утверждающий, чтобы перенаправить запрос в, если система не может найти диспетчер второго уровня.

Дополнительные сведения см. в подразделах [изменение параметров утверждения для пакета Access в управлении](../governance/entitlement-management-access-package-approval-policy.md#alternate-approvers)назначением Azure AD.

--- 

## <a name="november-2020"></a>Ноябрь 2020 г.

### <a name="azure-active-directory-tls-10-tls-11-and-3des-deprecation"></a>Azure Active Directory TLS 1,0, TLS 1,1 и нерекомендуемое использование 3DES

**Тип.** Планирование изменений.  
**Категория службы:** Все приложения Azure AD  
**Возможности продукта:** Стандартами

Azure Active Directory будет использовать следующие протоколы в Azure Active Directory регионах по всему миру до 30 июня 2021:

- TLS 1.0
- TLS 1.1
- набор шифров 3DES (TLS_RSA_WITH_3DES_EDE_CBC_SHA)

Затронутые среды:
- Коммерческое облако Azure
- Office 365 GCC и WW

Связанные объявления все сочетания клиент-сервер и браузер-сервер должны использовать TLS 1,2 и современные комплекты шифров для обеспечения безопасного подключения к Azure Active Directory для Azure, Office 365 и служб Microsoft 365. Это изменение связано с [Azure Active Directory TLS 1,0 & 1,1 и прекращением использования алгоритма 3DES в US gov Cloud](whats-new.md#azure-active-directory-tls-10-tls-11-and-3des-deprecation-in-us-gov-cloud).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---november-2020"></a>Новые Федеративные приложения, доступные в коллекции приложений Azure AD — Ноябрь 2020

**Тип.** Новая функция.  
**Категория службы:** Корпоративные приложения  
**Возможности продукта.** Интеграция с решениями сторонних производителей.

В ноябре 2020 мы добавили в коллекцию приложений следующие 52 новых приложений с поддержкой федерации:

[Выезд & управление расходами](https://app.expenseonce.com/Account/Login), [Трибелу](../saas-apps/tribeloo-tutorial.md), [средство выбора файлов Itslearning](https://pmteam.itslearning.com/), [элемент управления пожаров](../saas-apps/crises-control-tutorial.md), [каурталерт](https://www.courtalert.com/), [стеалсмаил](https://stealthmail.com/), [Едментум-учение](https://app.studyisland.com/cfw/login/), [Виртуальные риски](../saas-apps/virtual-risk-manager-tutorial.md), [Тиму](../saas-apps/timu-tutorial.md), [платформа аналитики для анализа](../saas-apps/looker-analytics-platform-tutorial.md), [талвиев-](https://recruit.talview.com/login)набор для работы, переводчик в реальном времени, [клаксун](https://access.klaxoon.com/login), [подбеан](../saas-apps/podbean-tutorial.md), [Зкал](https://zcal.co/signup), [експенсеманажер](https://api.expense-manager.com/), [Netsparker Enterprise](../saas-apps/netsparker-enterprise-tutorial.md), [EN-Trak. Платформа взаимодействия с клиентами](https://portal.en-trak.app/), [Appian](../saas-apps/appian-tutorial.md), [Panorays](../saas-apps/panorays-tutorial.md), [Builterra](https://portal.builterra.com/) [,](../saas-apps/coupa-risk-assess-tutorial.md) [Ирина](https://my.evacheckin.com/organization) [, HowNow](../saas-apps/hownow-webapp-sso-tutorial.md) [(все продукты)](../saas-apps/lucid-tutorial.md), [webapp](https://portal.brightbooking.eu/), [COUPA вразумительное](../saas-apps/sailpoint-identitynow-tutorial.md),[центральный ресурс](../saas-apps/resource-central-tutorial.md), [UiPathStudioO365App](https://www.uipath.com/product/platform), [Jedox](../saas-apps/jedox-tutorial.md), [Cequence Application Security](../saas-apps/cequence-application-security-tutorial.md), [PerimeterX](../saas-apps/perimeterx-tutorial.md), [TrendMiner](../saas-apps/trendminer-tutorial.md), [Lexion](../saas-apps/lexion-tutorial.md), [WorkWare](../saas-apps/workware-tutorial.md), [ProdPad](../saas-apps/prodpad-tutorial.md), [AWS ClientVPN](../saas-apps/aws-clientvpn-tutorial.md), [AppSec единый вход](../saas-apps/appsec-flow-sso-tutorial.md), [Luum](../saas-apps/luum-tutorial.md), [мера доставки](https://www.gpcsl.com/freight.html), [terraform Cloud](../saas-apps/terraform-cloud-tutorial.md), [исследование природы](../saas-apps/nature-research-tutorial.md), [воспроизведение цифрового входа](https://login.playsignage.com/login), [RemotePC](../saas-apps/remotepc-tutorial.md), [Prolorus](../saas-apps/prolorus-tutorial.md), [Hirebridge ATS](../saas-apps/hirebridge-ats-tutorial.md), [Teamgage](https://www.teamgage.com/Account/ExternalLoginAzure), [Roadmunk](../saas-apps/roadmunk-tutorial.md), [восход солнца Software отношения CRM](https://cloud.relations-crm.com/) [, Procaire](../saas-apps/procaire-tutorial.md), [инструктор® по eDriving: Business](https://www.edriving.com/), [Gradle предприятие](https://gradle.com/)

Документацию по всем приложениям можно также найти здесь. https://aka.ms/AppsTutorial

Чтобы получить список приложений в коллекции приложений Azure AD, ознакомьтесь со сведениями в этой статье. https://aka.ms/AzureADAppRequest

---

### <a name="public-preview---custom-roles-for-enterprise-apps"></a>Общедоступная Предварительная версия — пользовательские роли для корпоративных приложений

**Тип.** Новая функция.  
**Категория службы:** RBAC  
**Возможность продукта.** Контроль доступа.
 
 [Пользовательские роли RBAC для делегированного управления корпоративными приложениями](../roles/custom-available-permissions.md) теперь доступны в общедоступной предварительной версии. Эти новые разрешения создаются на основе пользовательских ролей для управления регистрацией приложений, что позволяет точно контролировать доступ администраторов. Со временем дополнительные разрешения на делегирование управления Azure AD будут выпущены.

Некоторые распространенные сценарии делегирования:
- Назначение пользователей и групп, которые могут получать доступ к приложениям единого входа на основе SAML
- Создание приложений из коллекции Azure AD
- обновление и чтение базовых конфигураций SAML для приложений единого входа на основе SAML
- Управление сертификатами подписи для приложений единого входа на основе SAML
- Обновление сертификатов с истекшим сроком действия. адреса электронной почты уведомлений для приложений единого входа на основе SAML
- обновление алгоритма сигнатуры SAML Token и входа для приложений единого входа на основе SAML
- Создание, удаление и обновление атрибутов и утверждений пользователей для приложений единого входа на основе SAML
- возможность включения, отключения и перезапуска заданий подготовки
- обновления сопоставления атрибутов
- возможность чтения параметров подготовки, связанных с объектом
- возможность чтения параметров подготовки, связанных с субъектом-службой
- возможность авторизации доступа к приложениям для подготовки

---

### <a name="public-preview---azure-ad-application-proxy-natively-supports-single-sign-on-access-to-applications-that-use-headers-for-authentication"></a>Общедоступная Предварительная версия Azure AD Application Proxy изначально поддерживает единый вход для приложений, использующих заголовки для проверки подлинности.

**Тип.** Новая функция.  
**Категория службы.** Прокси-сервер приложения.  
**Возможность продукта.** Контроль доступа.
 
Azure Active Directory прокси приложения (Azure AD) изначально поддерживает единый вход для приложений, использующих заголовки для проверки подлинности. Вы можете настроить значения заголовков, необходимые для приложения в Azure AD. Значения заголовков будут отправляться в приложение через прокси приложения. Дополнительные сведения см. в статье [единый вход на основе заголовков для локальных приложений с Azure AD App прокси-сервером](../manage-apps/application-proxy-configure-single-sign-on-with-headers.md) .
 
---

### <a name="general-availability---azure-ad-b2c-phone-sign-up-and-sign-in-using-custom-policy"></a>Общедоступная доступность — Azure AD B2C телефонной регистрации и входа с помощью настраиваемой политики

**Тип.** Новая функция.  
**Категория службы.** B2C — управление удостоверениями клиентов.  
**Возможности продукта.** B2B и B2C.

При регистрации и входе в систему разработчики и предприятия могут позволить своим клиентам регистрироваться и входить с помощью одноразового пароля, отправленного на телефонный номер пользователя через SMS. Эта функция также позволяет клиенту изменить свой номер телефона в случае утери доступа к телефону. Благодаря возможности пользовательских политик разработчики и предприятия могут обмениваться своими торговыми марками через настройку страниц. Узнайте, как [настроить регистрацию и вход в систему с помощью пользовательских политик в Azure AD B2C](../../active-directory-b2c/phone-authentication.md).
 
---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---november-2020"></a>Новые соединители подготовки в коллекции приложений Azure AD — Ноябрь 2020

**Тип.** Новая функция.  
**Категория службы.** Подготовка приложений.  
**Возможности продукта.** Интеграция с решениями сторонних производителей.
 
Теперь вы можете автоматизировать процессы создания, обновление и удаление учетных записей пользователей для следующих новых интегрированных приложений:

- [Adobe Identity Management](../saas-apps/adobe-identity-management-provisioning-tutorial.md)
- [Блог](../saas-apps/blogin-provisioning-tutorial.md)
- [Clarizen One](../saas-apps/clarizen-one-provisioning-tutorial.md)
- [Contentful](../saas-apps/contentful-provisioning-tutorial.md)
- [GitHub AE](../saas-apps/github-ae-provisioning-tutorial.md)
- [Playvox](../saas-apps/playvox-provisioning-tutorial.md)
- [PrinterLogic SaaS](../saas-apps/printer-logic-saas-provisioning-tutorial.md)
- [Крестики Mobile](../saas-apps/tic-tac-mobile-provisioning-tutorial.md)
- [Visibly](../saas-apps/visibly-provisioning-tutorial.md)

Дополнительные сведения см. в статье [Автоматизация подготовки пользователей в приложениях SaaS с помощью Azure AD](../app-provisioning/user-provisioning.md).
 
---

### <a name="public-preview---email-sign-in-with-proxyaddresses-now-deployable-via-staged-rollout"></a>Общедоступная Предварительная версия — электронная почта Sign-In с развертыванием, которое теперь можно развернуть через промежуточное развертывание

**Тип.** Новая функция.  
**Категория службы.** Проверки подлинности (имена входа).  
**Возможности продукта:** Проверка подлинности пользователя
 
Теперь администраторы клиента могут использовать промежуточное развертывание для развертывания электронной почты Sign-In с ProxyAddresses для конкретных групп Azure AD. Это может помочь при попытке выполнить эту функцию до развертывания ее во всем клиенте с помощью политики обнаружения домашней области. Инструкции по развертыванию Sign-In электронной почты с помощью ProxyAddresses через промежуточное развертывание см. в [документации](../authentication/howto-authentication-use-email-signin.md).
 
---

### <a name="limited-preview---sign-in-diagnostic"></a>Ограниченная Предварительная версия — диагностика для входа

**Тип.** Новая функция.  
**Категория службы.** Отчеты.  
**Возможности продукта.** Мониторинг и отчетность.
 
После первоначального выпуска диагностики для входа в систему администраторы могут просматривать вход пользователей. Администраторы могут получить контекстные, конкретные и соответствующие сведения и рекомендации о том, что произошло во время входа и как устранить проблемы. Диагностика доступна как на уровне Azure AD, так и в колонках диагностики и разрешения условного доступа. Сценарии диагностики, рассмотренные в этом выпуске, являются условным доступом, многофакторной проверкой подлинности и успешным входом.

Дополнительные сведения см. [в статье что такое диагностика входа в Azure AD?](../reports-monitoring/overview-sign-in-diagnostics.md).
 
---

### <a name="improved-unfamiliar-sign-in-properties"></a>Улучшенные незнакомые свойства входа

**Тип.** Измененная функция.  
**Категория службы:** Защита идентификации  
**Возможности продукта.** Безопасность и защита удостоверений.

  Были обновлены незнакомые обнаружения свойств входа. Клиенты могут заметить более высокий риск незнакомых обнаружений свойств входа. Дополнительные сведения см. в разделе [что такое риск?](../identity-protection/concept-identity-protection-risks.md)
 
---

### <a name="public-preview-refresh-of-cloud-provisioning-agent-now-available-version-112810"></a>Общедоступное предварительное обновление агента подготовки облака сейчас доступно (версия: 1.1.281.0)

**Тип.** Измененная функция.  
**Категория службы:** Подготовка облака Azure AD  
**Возможности продукта:** Управление жизненным циклом удостоверений
 
Агент подготовки облака выпущен в общедоступной предварительной версии и теперь доступен на портале. Этот выпуск содержит несколько усовершенствований, включая поддержку GMSA для доменов, обеспечивающих лучшую безопасность, улучшенные циклы начальной синхронизации и поддержку больших групп. Дополнительные сведения см. в [журнале](../app-provisioning/provisioning-agent-release-version-history.md) версий выпуска. 
 
---

### <a name="bitlocker-recovery-key-api-endpoint-now-under-informationprotection"></a>Конечная точка API ключа восстановления BitLocker сейчас находится в/Информатионпротектион

**Тип.** Измененная функция.  
**Категория службы:** Управление доступом к устройствам  
**Возможности продукта:** Управление жизненным циклом устройств
 
Ранее вы могли восстановить ключи BitLocker с помощью конечной точки/битлоккер. В итоге мы будем использовать эту конечную точку, и клиенты должны приступить к использованию API, который теперь находится в разделе/Информатионпротектион. 

В разделе [API восстановления BitLocker](/graph/api/resources/bitlockerrecoverykey?view=graph-rest-beta) приведены обновления документации, в которых отражены эти изменения.

---

### <a name="general-availability-of-application-proxy-support-for-remote-desktop-services-html5-web-client"></a>Общая доступность поддержки прокси приложения для веб-клиента службы удаленных рабочих столов HTML5

**Тип.** Измененная функция.  
**Категория службы.** Прокси-сервер приложения.  
**Возможность продукта.** Контроль доступа.
 
Поддержка Azure AD Application Proxy для веб-клиента службы удаленных рабочих столов (RDS) теперь доступна в общедоступной системе. Веб-клиент RDS позволяет пользователям получать доступ к инфраструктуре удаленный рабочий стол через любой браузер, поддерживающий HTLM5, такой как Microsoft ребр, Internet Explorer 11, Google Chrome и т. д. Пользователи могут взаимодействовать с удаленными приложениями или настольными системами, как и с локальными устройствами из любого места. 

С помощью AD Application Proxy Azure можно повысить безопасность развертывания RDS, установив политики предварительной проверки подлинности и условного доступа для всех типов клиентских приложений с богатыми возможностями. Дополнительные сведения см. в статье [публикация удаленный рабочий стол с помощью Azure AD application proxy](../manage-apps/application-proxy-integrate-with-remote-desktop-services.md)
 
---

### <a name="new-enhanced-dynamic-group-service-is-in-public-preview"></a>Новая улучшенная динамическая группа в общедоступной предварительной версии

**Тип.** Измененная функция.  
**Категория службы.** Управление группами.  
**Возможности продукта.** Совместная работа.
 
Улучшенная динамическая групповая служба теперь доступна в общедоступной предварительной версии. Новые клиенты, создающие динамические группы в своих клиентах, будут использовать новую службу. Время, необходимое для создания динамической группы, будет пропорционально размеру создаваемой группы, а не размеру клиента. Это обновление значительно повысит производительность для больших клиентов, когда клиенты создают группы меньшего размера. 

Новая служба также нацелена на завершение добавления и удаления членов из-за изменений атрибутов в течение нескольких минут. Кроме того, ошибки одной обработки не блокируют обработку клиента. Дополнительные сведения о создании динамических групп см. в нашей [документации](../enterprise-users/groups-create-rule.md).
 
---
## <a name="october-2020"></a>Октябрь 2020 г.

### <a name="azure-ad-on-premises-hybrid-agents-impacted-by-azure-tls-certificate-changes"></a>Локальные агенты гибридного развертывания Azure AD, затронутые изменениями сертификатов Azure TLS

**Тип.** Планирование изменений.  
**Категория службы:** недоступно  
**Возможности продукта.** Платформа.

Корпорация Майкрософт обновляет службы Azure для использования TLS-сертификатов от других корневых центров сертификации (ЦС). Это обновление связано с тем, что существующие сертификаты ЦС не удовлетворяют базовым требованиям форума центра сертификации и браузера. Это изменение повлияет на гибридные локальные агенты Azure AD, которые имеют защищенные среды с фиксированным списком корневых сертификатов и должны быть обновлены для доверия новым издателям сертификатов.

Это изменение приведет к нарушению работы службы, если вы не выполните действие немедленно. Эти агенты включают [соединители прокси приложения](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AppProxy) для удаленного доступа к локальным агентам [проверки подлинности](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) , которые позволяют пользователям входить в приложения, используя те же пароли, и агенты [предварительной подготовки облака](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) , которые выполняют AD в Azure AD Sync. 

Если у вас есть среда, для которой настроены правила брандмауэра, разрешающие исходящие вызовы только для конкретного скачанного списка отзыва сертификатов (CRL), необходимо разрешить следующие URL-адреса CRL и OCSP. Полные сведения об изменениях и URL-адресах CRL и OCSP для обеспечения доступа к см. в статье  [изменения сертификатов Azure TLS](../../security/fundamentals/tls-certificate-changes.md).

---

### <a name="provisioning-events-will-be-removed-from-audit-logs-and-published-solely-to-provisioning-logs"></a>События подготовки будут удалены из журналов аудита и опубликованы только в журналах подготовки.

**Тип.** Планирование изменений.  
**Категория службы.** Отчеты.  
**Возможности продукта.** Мониторинг и отчетность.
 
Действия, выполняемые [службой подготовки](../app-provisioning/user-provisioning.md) scim, регистрируются как в журналах аудита, так и в журналах подготовки. Сюда входят такие действия, как создание пользователя в ServiceNow, группирование в Гсуите или импорт роли из AWS. В будущем эти события будут опубликованы только в журналах подготовки. Это изменение реализуется во избежание дублирования событий в журналах, а также дополнительных затрат клиентов, использующих журналы в log Analytics. 

Мы предоставим обновление после завершения даты. Это прекращение не планируется в течение календарного года 2020. 

> [!NOTE]
> Это не влияет на события в журналах аудита за пределами событий синхронизации, генерируемых службой подготовки. Такие события, как создание приложения, политика условного доступа, пользователь в каталоге и т. п., будут по-прежнему выдаваться в журналах аудита. [Подробнее](../reports-monitoring/concept-provisioning-logs.md?context=azure%2factive-directory%2fapp-provisioning%2fcontext%2fapp-provisioning-context).
 

---

### <a name="azure-ad-on-premises-hybrid-agents-impacted-by-azure-transport-layer-security-tls-certificate-changes"></a>Локальные гибридные агенты Azure AD, на которые влияют изменения сертификата безопасности уровня транспорта (TLS) Azure

**Тип.** Планирование изменений.  
**Категория службы:** недоступно  
**Возможности продукта.** Платформа.
 
Корпорация Майкрософт обновляет службы Azure для использования TLS-сертификатов от других корневых центров сертификации (ЦС). Обновление будет выполняться из-за того, что текущие сертификаты ЦС не появятся в базовых требованиях центра сертификации и браузера. Это изменение повлияет на гибридные агенты Azure AD, установленные локально, с защищенными средами с фиксированным списком корневых сертификатов. Эти агенты необходимо будет обновить, чтобы доверять новым издателям сертификатов.

Это изменение приведет к нарушению работы службы, если вы не выполните действие немедленно. К этим агентам относятся: 
- [Соединители прокси приложения](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AppProxy) для удаленного доступа к локальной среде 
- Агенты [сквозной проверки подлинности](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) , позволяющие пользователям входить в приложения с использованием одних и тех же паролей
- Агенты [предварительной подготовки облачных](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) служб, которые выполняют синхронизацию Azure AD. 

Если у вас есть среда с правилами брандмауэра, разрешающими исходящие вызовы только для конкретного скачанного списка отзыва сертификатов (CRL), необходимо разрешить URL-адреса CRL и OCSP. Полные сведения об изменениях и URL-адресах CRL и OCSP для обеспечения доступа к см. в статье  [изменения сертификатов Azure TLS](../../security/fundamentals/tls-certificate-changes.md).
 
---

### <a name="azure-active-directory-tls-10-tls-11-and-3des-deprecation-in-us-gov-cloud"></a>Azure Active Directory TLS 1,0, TLS 1,1 и нерекомендуемое использование 3DES в US Gov облаке

**Тип.** Планирование изменений.  
**Категория службы:** Все приложения Azure AD  
**Возможности продукта:** Стандартами
 
Azure Active Directory будет использовать устаревшие следующие протоколы до 31 марта 2021:
- TLS 1.0
- TLS 1.1
- набор шифров 3DES (TLS_RSA_WITH_3DES_EDE_CBC_SHA)

Все комбинации "клиент-сервер" и "браузер-сервер" должны использовать TLS 1,2 и современные комплекты шифров для обеспечения безопасного подключения к Azure Active Directory для Azure, Office 365 и служб Microsoft 365.

Затронутые среды:
- US Gov Azure
- [Office 365 GCC High & DoD](/microsoft-365/compliance/tls-1-2-in-office-365-gcc)
 
---

### <a name="assign-applications-to-roles-on-au-and-object-scope"></a>Назначение приложений ролям в AU и области объектов

**Тип.** Новая функция.  
**Категория службы:** RBAC  
**Возможность продукта.** Контроль доступа.
 
Эта функция позволяет назначать приложению (SPN) роль администратора в области административной единицы. Дополнительные сведения см. в разделе [назначение ограниченных ролей административной единице](../roles/admin-units-assign-roles.md).

---

### <a name="now-you-can-disable-and-delete-guest-users-when-theyre-denied-access-to-a-resource"></a>Теперь вы можете отключить и удалить гостевых пользователей, если им отказано в доступе к ресурсу.

**Тип.** Новая функция.  
**Категория службы:** Проверки доступа  
**Возможности продукта:** Управление удостоверениями
 
Отключение и удаление — это расширенный элемент управления в проверках доступа Azure AD, позволяющий организациям лучше управлять внешними гостями в группах и приложениях. Если в ходе проверки доступа запрещены гости, то **Отключение и удаление** автоматически заблокирут их вход в течение 30 дней. Через 30 дней они будут полностью удалены из клиента.

Дополнительные сведения об этой функции см. [в разделе Отключение и удаление внешних удостоверений с помощью проверок доступа Azure AD](../governance/access-reviews-external-users.md#disable-and-delete-external-identities-with-azure-ad-access-reviews).
 
---

### <a name="access-review-creators-can-add-custom-messages-in-emails-to-reviewers"></a>Авторы проверки доступа могут добавлять пользовательские сообщения в сообщениях электронной почты рецензентам

**Тип.** Новая функция.  
**Категория службы:** Проверки доступа  
**Возможности продукта:** Управление удостоверениями
 
В проверках доступа Azure AD администраторы, создающие рецензию, теперь могут записывать пользовательское сообщение в рецензенты. Рецензенты увидят сообщение в сообщении электронной почты с просьбой выполнить проверку. Дополнительные сведения об использовании этой функции см. в разделе Шаг 14 раздела [Создание проверок доступа](../governance/create-access-review.md#create-one-or-more-access-reviews) .

---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---october-2020"></a>Новые соединители подготовки в коллекции приложений Azure AD — Октябрь 2020

**Тип.** Новая функция.  
**Категория службы.** Подготовка приложений.  
**Возможности продукта.** Интеграция с решениями сторонних производителей.
 
Теперь вы можете автоматизировать процессы создания, обновление и удаление учетных записей пользователей для следующих новых интегрированных приложений:

- [Apple Business Manager;](../saas-apps/apple-business-manager-provision-tutorial.md)
- [Apple School Manager](../saas-apps/apple-school-manager-provision-tutorial.md)
- [Code42](../saas-apps/code42-provisioning-tutorial.md)
- [AlertMedia](../saas-apps/alertmedia-provisioning-tutorial.md)
- [Службы каталога OpenText](../saas-apps/open-text-directory-services-provisioning-tutorial.md)
- [Cinode](../saas-apps/cinode-provisioning-tutorial.md)
- [Синхронизация удостоверений глобальной ретрансляции](../saas-apps/global-relay-identity-sync-provisioning-tutorial.md)

Дополнительные сведения об усилении защиты организации с помощью автоматизированной подготовки учетных записей пользователей см. в статье [Автоматическая подготовка пользователей для приложений SaaS в Azure AD](../app-provisioning/user-provisioning.md).
 
---

### <a name="integration-assistant-for-azure-ad-b2c"></a>Помощник по интеграции для Azure AD B2C

**Тип.** Новая функция.  
**Категория службы.** B2C — управление удостоверениями клиентов.  
**Возможности продукта.** B2B и B2C.
 
Теперь интерфейс помощника по интеграции (Предварительная версия) доступен для Azure AD B2C Регистрация приложений. Этот интерфейс поможет вам в настройке приложения для распространенных сценариев. Ознакомьтесь с рекомендациями и рекомендациями по [платформе Microsoft Identity](../develop/identity-platform-integration-checklist.md).
 
---

### <a name="view-role-template-id-in-azure-portal-ui"></a>Просмотр идентификатора шаблона роли в пользовательском интерфейсе портал Azure

**Тип.** Новая функция.  
**Категория службы:** Роли Azure  
**Возможность продукта.** Контроль доступа.
 

Теперь можно просмотреть идентификатор шаблона каждой роли Azure AD в портал Azure. В Azure AD выберите  **Описание** выбранной роли. 

Пользователям рекомендуется использовать идентификаторы шаблонов ролей в сценариях и коде PowerShell вместо отображаемого имени. Идентификатор шаблона роли поддерживается для объектов [directoryRoles](/graph/api/resources/directoryrole) и [определения роли](/graph/api/resources/unifiedroledefinition?view=graph-rest-beta) . Дополнительные сведения об идентификаторах шаблонов ролей см. [в статье встроенные роли Azure AD](../roles/permissions-reference.md).

---

### <a name="api-connectors-for-azure-ad-b2c-sign-up-user-flows-is-now-in-public-preview"></a>Соединители API для регистрации пользователей в Azure AD B2C теперь доступны в общедоступной предварительной версии

**Тип.** Новая функция.  
**Категория службы.** B2C — управление удостоверениями клиентов.  
**Возможности продукта.** B2B и B2C.
 

Соединители API теперь доступны для использования с Azure Active Directory B2C. Соединители API позволяют использовать веб-API для настройки потоков пользователей для регистрации и интеграции с внешними облачными системами. Соединители API можно использовать для:

- Интеграция с настраиваемыми рабочими процессами утверждения
- Проверка входных данных пользователя
- Перезаписать атрибуты пользователя 
- Выполнение настраиваемой бизнес-логики 

 Дополнительные сведения см. в статье [Использование соединителей API для настройки и расширения](../../active-directory-b2c/api-connectors-overview.md) документации по регистрации.

---

### <a name="state-property-for-connected-organizations-in-entitlement-management"></a>Свойство State для подключенных организаций в управлении назначением

**Тип.** Новая функция.  
**Категория службы:** **Возможности продукта** управления каталогом: Управление назначениями
 

 Все подключенные Организации теперь будут иметь дополнительное свойство с именем State. Состояние управляет тем, как подключенная Организация будет использоваться в политиках, ссылающихся на "все настроенные подключенные организации". Значение будет "настроено" (это означает, что Организация находится в области политик, использующих предложение "все") или "предложено" (это означает, что Организация не находится в области).  

У созданных вручную подключенных организаций будет значение по умолчанию "настроено". В то же время, автоматически созданные (созданные с помощью политик, которые позволяют любому пользователю в Интернете запрашивать доступ) будут по умолчанию предлагаться.  Все подключенные Организации, созданные до сентября 9 2020, будут иметь значение "настроено". При необходимости администраторы могут обновлять это свойство. [Подробнее](../governance/entitlement-management-organization.md#managing-a-connected-organization-programmatically).
 

---

### <a name="azure-active-directory-external-identities-now-has-premium-advanced-security-settings-for-b2c"></a>Azure Active Directory внешние удостоверения теперь имеют дополнительные параметры безопасности Premium для B2C

**Тип.** Новая функция.  
**Категория службы.** B2C — управление удостоверениями клиентов.  
**Возможности продукта.** B2B и B2C.
 
Функции условного доступа и обнаружения рисков на основе рисков теперь доступны в [Azure AD B2C](../..//active-directory-b2c/conditional-access-identity-protection-overview.md). Благодаря этим расширенным функциям безопасности клиенты теперь могут:
- Используйте Intelligent Insights для оценки рисков с помощью B2C приложений и учетных записей конечных пользователей. В число обнаружений входят нетипичные пути, анонимные IP-адреса, связанные с вредоносными программами IP-адреса и аналитика угроз Azure AD. Также доступны портал и отчеты на основе API.
- Автоматическое устранение рисков путем настройки политик адаптивной проверки подлинности для пользователей B2C. Разработчики и администраторы приложений могут уменьшить риск в реальном времени, запросив многофакторную проверку подлинности (MFA) или блокируя доступ в зависимости от обнаруженного уровня риска пользователя, при этом дополнительные элементы управления доступны на основе расположения, группы и приложения.
- Интеграция с Azure AD B2C пользовательскими потоками и пользовательскими политиками. Условия могут быть активированы из встроенных последовательностей пользователей в Azure AD B2C или могут быть включены в пользовательские политики B2C. Как и в других аспектах потока пользователей B2C, можно настроить обмен сообщениями с пользователем. Настройка зависит от альтернативного голоса, марки и вариантов защиты Организации.
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---october-2020"></a>Новые Федеративные приложения, доступные в коллекции приложений Azure AD — Октябрь 2020

**Тип.** Новая функция.  
**Категория службы:** Корпоративные приложения  
**Возможности продукта.** Интеграция с решениями сторонних производителей.
 
В октябре 2020 мы добавили в галерею приложений следующие 27 новых приложений с поддержкой федерации:

[Sentry](../saas-apps/sentry-tutorial.md), [бумблеби-Суперапп](https://app.yellowmessenger.com/user/login), [ABBYY FlexiCapture Cloud](../saas-apps/abbyy-flexicapture-cloud-tutorial.md), [Еакомпосер](../saas-apps/eacomposer-tutorial.md), [женесис облачная интеграция для Azure](https://apps.mypurecloud.com/msteams-integration/), [портал технологий для зон](https://portail.zonetechnologie.com/signin), [Beautiful.AI](../saas-apps/beautiful.ai-tutorial.md), [датавиза Access Broker](https://console.datawiza.com/), [зокри](https://app.zokri.com/), [чеккпруф](../saas-apps/checkproof-tutorial.md), Ecochallenge.org [, атспоке,](http://atspoke.com/login) [напоминание о встрече](https://app.appointmentreminder.co.nz/account/login), [Cloud.](https://cloud.market/) [Marketplace, травелперк, гритли](../saas-apps/travelperk-tutorial.md) [, OrgVitality](https://app.greetly.com/) [SSO](../saas-apps/orgvitality-sso-tutorial.md), [веб-](../saas-apps/web-cargo-air-tutorial.md)канал, загруженный в сеть, [CRM](../saas-apps/loop-flow-crm-tutorial.md), [Starmind](../saas-apps/starmind-tutorial.md) [, Workstem,](https://hrm.workstem.com/login) [Retail zipline](../saas-apps/retail-zipline-tutorial.md) [,](../saas-apps/pulse-secure-virtual-traffic-manager-tutorial.md) [Hoxhunt](../saas-apps/hoxhunt-tutorial.md), [MEVISIO](../saas-apps/mevisio-tutorial.md) [, Samsara,](../saas-apps/samsara-tutorial.md) [Nimbus](../saas-apps/nimbus-tutorial.md). [](https://events.ecochallenge.org/users/login)

Документацию по всем приложениям можно также найти здесь. https://aka.ms/AppsTutorial

Чтобы получить список приложений в коллекции приложений Azure AD, ознакомьтесь со сведениями в этой статье. https://aka.ms/AzureADAppRequest

---

### <a name="provisioning-logs-can-now-be-streamed-to-log-analytics"></a>Журналы подготовки теперь можно передавать в log Analytics.

**Тип.** Новая функция.  
**Категория службы.** Отчеты.  
**Возможности продукта.** Мониторинг и отчетность.
 

Опубликуйте журналы подготовки в log Analytics, чтобы:
- Храните журналы подготовки в течение более чем 30 дней
- Определение пользовательских оповещений и уведомлений
- Создание панелей мониторинга для визуализации журналов
- Выполнение сложных запросов для анализа журналов 

Сведения о том, как использовать эту функцию, см. в статье [сведения о том, как подготовка интегрируется с журналами Azure Monitor](../app-provisioning/application-provisioning-log-analytics.md).
 
---

### <a name="provisioning-logs-can-now-be-viewed-by-application-owners"></a>Теперь владельцы приложений могут просматривать журналы подготовки.

**Тип.** Измененная функция.  
**Категория службы.** Отчеты.  
**Возможности продукта.** Мониторинг и отчетность.
 
Теперь вы можете разрешить владельцам приложений отслеживать действия службы подготовки и устранять проблемы, не предоставляя привилегированной роли и не делая их узким местом. [Подробнее](../reports-monitoring/concept-provisioning-logs.md).
 
---

### <a name="renaming-10-azure-active-directory-roles"></a>Переименование 10 Azure Active Directory ролей

**Тип.** Измененная функция.  
**Категория службы:** Роли Azure  
**Возможность продукта.** Контроль доступа.
 
Некоторые встроенные роли Azure Active Directory (AD) имеют имена, отличающиеся от имен, отображаемых в центре администрирования Microsoft 365, на портале Azure AD и Microsoft Graph. Такая несогласованность может вызвать проблемы в автоматизированных процессах. В этом обновлении мы переименование 10 имен ролей, чтобы сделать их единообразными. В следующей таблице присвоены новые имена ролей:

![Таблица, в которой показаны имена ролей в MS API Graph и портал Azure, а также предложенное новое имя роли в центре администрирования M365, портал Azure и API.](media/whats-new/azure-role.png)

---

### <a name="azure-ad-b2c-support-for-auth-code-flow-for-spas-using-msal-js-2x"></a>Azure AD B2C поддержка потока кода проверки подлинности для одностраничные приложения с помощью MSAL JS 2. x

**Тип.** Измененная функция.  
**Категория службы.** B2C — управление удостоверениями клиентов.  
**Возможности продукта.** B2B и B2C.
 
MSAL.js версии 2. x теперь включает поддержку потока кода авторизации для одностраничных веб-приложений (одностраничные приложения). Azure AD B2C теперь поддерживает использование типа приложения SPA на портал Azure и использование потока кода авторизации MSAL.js с PKCE для одностраничных приложений. Это позволит одностраничные приложения с помощью Azure AD B2C поддерживать единый вход с более новыми браузерами и соблюдать новые рекомендации по протоколам проверки подлинности. Приступая к работе с [регистрацией одностраничного приложения (SPA) в Azure Active Directory B2C](../../active-directory-b2c/tutorial-register-spa.md) руководстве.

---

### <a name="updates-to-remember-multi-factor-authentication-mfa-on-a-trusted-device-setting"></a>Обновления для запоминания многофакторной проверки подлинности (MFA) в параметре доверенного устройства

**Тип.** Измененная функция.  
**Категория службы:** MFA  
**Возможности продукта.** Безопасность и защита удостоверений.
 

Мы недавно обновили функцию " [Запомнить многофакторную проверку подлинности" (MFA)](../authentication/howto-mfa-mfasettings.md#remember-multi-factor-authentication) на доверенном устройстве, чтобы продлить аутентификацию до 365 дней. Azure Active Directory (Azure AD) Premium Licenses также может использовать [политику условного доступа — частоту входа](../conditional-access/howto-conditional-access-session-lifetime.md#user-sign-in-frequency) , которая обеспечивает большую гибкость параметров повторной проверки подлинности.

Для оптимального взаимодействия с пользователем рекомендуется использовать частоту входа условного доступа для расширения времени существования сеанса на доверенных устройствах, расположениях или сеансах с низким риском в качестве альтернативы параметру запомнить MFA на доверенном устройстве. Чтобы приступить к работе, ознакомьтесь с нашим [последним руководством по оптимизации процесса повторной проверки подлинности](../authentication/concepts-azure-multi-factor-authentication-prompts-session-lifetime.md).

---

## <a name="september-2020"></a>Сентябрь 2020 г.

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---september-2020"></a>Новые соединители подготовки в коллекции приложений Azure AD — Сентябрь 2020

**Тип.** Новая функция.  
**Категория службы.** Подготовка приложений.  
**Возможности продукта.** Интеграция с решениями сторонних производителей.
 
Теперь вы можете автоматизировать процессы создания, обновление и удаление учетных записей пользователей для следующих новых интегрированных приложений:

- [Coda](../saas-apps/coda-provisioning-tutorial.md)
- [Cofense Recipient Sync](../saas-apps/cofense-provision-tutorial.md)
- [InVision](../saas-apps/invision-provisioning-tutorial.md)
- [myday](../saas-apps/myday-provision-tutorial.md)
- [SAP Analytics Cloud](../saas-apps/sap-analytics-cloud-provisioning-tutorial.md)
- [Осведомленность о безопасности корневого каталога](../saas-apps/webroot-security-awareness-training-provisioning-tutorial.md)

Дополнительные сведения об усилении защиты организации с помощью автоматизированной подготовки учетных записей пользователей см. в статье [Автоматическая подготовка пользователей для приложений SaaS в Azure AD](../app-provisioning/user-provisioning.md).
 
---
### <a name="cloud-provisioning-public-preview-refresh"></a>Общедоступное предварительное обновление подготовки облака

**Тип.** Новая функция.  
**Категория службы:** **Возможности продукта** подготовки облака Azure AD: Управление жизненным циклом удостоверений
 
В Azure AD Connect общедоступной предварительной версии обновления доступны два основных улучшения, разработанных на основе отзывов клиентов: 

- Возможности сопоставления атрибутов с помощью портал Azure

    С помощью этой функции ИТ администраторы могут сопоставлять атрибуты пользователя, группы или контакта из AD с Azure AD, используя различные типы сопоставления. Сопоставление атрибутов — это функция, используемая для стандартизации значений атрибутов, переходящих из Active Directory в Azure Active Directory. Можно определить, следует ли напрямую сопоставлять значение атрибута, как это было из AD в Azure AD, или использовать выражения для преобразования значений атрибутов при подготовке пользователей. [Дополнительные сведения](../cloud-sync/how-to-attribute-mapping.md)

- Подготовка или тестирование взаимодействия с пользователем по требованию

    После настройки конфигурации может потребоваться проверить, правильно ли работает преобразование пользователя, прежде чем применить его ко всем пользователям в области. При подготовке по требованию ИТ-администраторы могут ввести различающееся имя (DN) пользователя AD и проверить, выполняется ли синхронизация как ожидается. Подготовка по требованию обеспечивает отличный способ проверки соответствия атрибутов, которые вы ранее работали. [Подробнее](../cloud-sync/how-to-on-demand-provision.md)
 
---

### <a name="audited-bitlocker-recovery-in-azure-ad---public-preview"></a>Аудит восстановления BitLocker в Azure AD — общедоступная Предварительная версия

**Тип.** Новая функция.  
**Категория службы:** Управление доступом к устройствам  
**Возможности продукта:** Управление жизненным циклом устройств
 
Когда ИТ-администраторы или конечные пользователи читают ключи восстановления BitLocker, к которым у них есть доступ, Azure Active Directory теперь создает журнал аудита, который фиксирует доступ к ключу восстановления. Один и тот же Аудит содержит сведения об устройстве, с которым связан ключ BitLocker.

Конечные пользователи могут [получить доступ к своим ключам восстановления через мою учетную запись](../user-help/my-account-portal-devices-page.md#view-a-bitlocker-key). ИТ-администраторы могут получить доступ к ключам восстановления через [API ключа восстановления BitLocker в бета](/graph/api/resources/bitlockerrecoverykey?view=graph-rest-beta) или через портал Azure AD. Дополнительные сведения см. в статье [Просмотр или копирование ключей BitLocker на портале Azure AD](../devices/device-management-azure-portal.md#view-or-copy-bitlocker-keys).

---

### <a name="teams-devices-administrator-built-in-role"></a>Встроенная роль администратора устройств групп

**Тип.** Новая функция.  
**Категория службы:** RBAC  
**Возможность продукта.** Контроль доступа.
 
Пользователи с ролью [администратора устройств групп](../roles/permissions-reference.md#teams-devices-administrator) могут управлять [сертифицированными для групп устройствами](https://www.microsoft.com/microsoft-365/microsoft-teams/across-devices/devices) из центра администрирования команд. 

Эта роль позволяет пользователю просматривать все устройства на одном взгляде с возможностью поиска и фильтрации устройств. Пользователь также может проверить сведения о каждом устройстве, включая учетную запись входа, а также марка и модель устройства. Пользователь может изменить параметры на устройстве и обновить версии программного обеспечения. Эта роль не предоставляет разрешения на проверку активности команд и качество вызова для устройства.
 
---

### <a name="advanced-query-capabilities-for-directory-objects"></a>Расширенные возможности запросов для объектов каталога

**Тип.** Новая функция.  
**Категория службы:** MS Graph  
**Возможности продукта.** Возможности для разработчиков
 
Все новые возможности запросов, появившиеся для объектов каталога в API-интерфейсах Azure AD, теперь доступны в конечной точке версии 1.0 и готовы для рабочей среды. Разработчики могут подсчитывать, искать, фильтровать и сортировать объекты каталога и связанные ссылки с помощью стандартных операторов OData.

Дополнительные [сведения см. в документации,](https://aka.ms/BlogPostMezzoGA)а также в этом [кратком опросе](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR_yN8EPoGo5OpR1hgmCp1XxUMENJRkNQTk5RQkpWTE44NEk2U0RIV0VZRy4u).
 
---

### <a name="public-preview-continuous-access-evaluation-for-tenants-who-configured-conditional-access-policies"></a>Общедоступная Предварительная версия: Оценка непрерывного доступа для клиентов, которые настроили политики условного доступа

**Тип.** Новая функция.  
**Категория службы.** Проверки подлинности (имена входа).  
**Возможности продукта.** Безопасность и защита удостоверений.
 
Ознакомительная версия с непрерывным доступом (автоматизированного конструирования) теперь доступна в общедоступной предварительной версии для клиентов Azure AD с политиками условного доступа. При использовании автоматизированного конструирования критические события и политики безопасности оцениваются в режиме реального времени. К ним относятся отключение учетной записи, сброс пароля и изменение расположения. Дополнительные сведения см. в разделе [Оценка непрерывного доступа](../conditional-access/concept-continuous-access-evaluation.md).

---

### <a name="public-preview-ask-users-requesting-an-access-package-additional-questions-to-improve-approval-decisions"></a>Общедоступная Предварительная версия: Попросите пользователей запросить у пакета доступа дополнительные вопросы для улучшения принятия решений об утверждении

**Тип.** Новая функция.  
**Категория службы:** Управление доступом пользователей  
**Возможности продукта:** Управление назначениями
 
Теперь администраторы могут требовать, чтобы пользователи, запрашивающие пакет доступа, ответили на дополнительные вопросы, кроме простоев бизнес-обоснованием на портале управления доступом Azure AD. Затем ответы пользователей будут отображаться для утверждающих, чтобы помочь им принять более точное решение об утверждении доступа. Дополнительные сведения см. в разделе [Получение дополнительных сведений о запрашивающей стороны для утверждения (Предварительная версия)](../governance/entitlement-management-access-package-approval-policy.md#collect-additional-requestor-information-for-approval-preview).
 
---

### <a name="public-preview-enhanced-user-management"></a>Общедоступная Предварительная версия: Расширенное управление пользователями

**Тип.** Новая функция.  
**Категория службы:** Управление пользователями  
**Возможности продукта:** Управление пользователями
 

Портал Azure AD был обновлен, чтобы упростить поиск пользователей на страницах «все пользователи» и «удаленные пользователи». В предварительной версии могут быть внесены следующие изменения: 
- Более видимые пользовательские свойства, включая идентификатор объекта, состояние синхронизации каталогов, тип создания и издателя удостоверений.
- Теперь поиск позволяет сочетать Поиск по именам, сообщениям электронной почты и идентификаторам объектов.
- Расширенная фильтрация по типу пользователя (член, гость и нет), состояние синхронизации каталогов, тип создания, название компании и доменное имя.
- Новые возможности сортировки для таких свойств, как имя, имя участника-пользователя и Дата удаления.
- Новое число пользователей, которые обновляются при любых операциях поиска или фильтрации.

Дополнительные сведения см. в разделе [улучшения управления пользователями (Предварительная версия) в Azure Active Directory](../enterprise-users/users-search-enhanced.md).

---

### <a name="new-notes-field-for-enterprise-applications"></a>Новое поле "Примечания" для корпоративных приложений

**Тип.** Новая функция.  
**Категория службы:** **Возможности продукта** для корпоративных приложений: единый вход

В корпоративные приложения можно добавлять текстовые заметки с произвольным текстом. Вы можете добавить любую важную информацию, которая поможет вам руководителю приложений в рамках корпоративных приложений. Дополнительные сведения см. [в разделе Краткое руководство. Настройка свойств приложения в клиенте Azure Active Directory (Azure AD)](../manage-apps/add-application-portal-configure.md). 

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---september-2020"></a>Новые Федеративные приложения, доступные в коллекции приложений Azure AD — Сентябрь 2020

**Тип.** Новая функция.  
**Категория службы:** Корпоративные приложения  
**Возможности продукта.** Интеграция с решениями сторонних производителей.

В сентябре 2020 мы добавили следующие 34 новых приложений в галерее приложений с поддержкой федерации:

[VMware горизонт — унифицированный шлюз доступа](), [импульсные безопасные ПК](../saas-apps/vmware-horizon-unified-access-gateway-tutorial.md), [Inventory360](../saas-apps/pulse-secure-pcs-tutorial.md), [фронтитуде](https://services.enteksystems.de/sso/microsoft/signup), [буквиджетс](https://www.bookwidgets.com/sso/office365), [ZVD_SERVER](https://zaas.zenmutech.com/user/signin), [хашдата для бизнеса](https://hashdata.app/login.xhtml), [секурелогин](https://securelogin.securelogin.nu/sso/azure/login), [циберсолутионс маилбасеσ/контентом](../saas-apps/cybersolutions-mailbase-tutorial.md), [циберсолутионс CYBERMAILΣ](../saas-apps/cybersolutions-cybermail-tutorial.md), [LimbleCMMS](https://auth.limblecmms.com/), [Glint Inc](../saas-apps/glint-inc-tutorial.md), [zeroheight](../saas-apps/zeroheight-tutorial.md), [фитнес по полу](https://app.genderfitness.com/), [портал Coeo](https://my.coeo.com/), [грамматика](../saas-apps/grammarly-tutorial.md), [Fivetran](../saas-apps/fivetran-tutorial.md), [Kumolus](../saas-apps/kumolus-tutorial.md), [RSA лучнику Suite](../saas-apps/rsa-archer-suite-tutorial.md), [TeamzSkill](../saas-apps/teamzskill-tutorial.md), [raumfürraum](../saas-apps/raumfurraum-tutorial.md), [Saviynt](../saas-apps/saviynt-tutorial.md), [BIZMERLINHR](https://marketplace.bizmerlin.net/bmone/signup), [мобильный замок](../saas-apps/mobile-locker-tutorial.md), [Zengine](../saas-apps/zengine-tutorial.md), [CloudCADI](https://app.cloudcadi.com/login), Simfoni [Analytics](https://simfonianalytics.com/accounts/microsoft/login/), [частн Identity & Access Management](https://my.priva.com/), [Nitro Pro](https://www.gonitro.com/nps/product-details/downloads), [Eventfinity](../saas-apps/eventfinity-tutorial.md), [Fexa](../saas-apps/fexa-tutorial.md), [безопасное подписывание Enterprise Portal](https://www.securedsigning.com/aad/Auth/ExternalLogin/AdminPortal), защищенная [Подписывание Enterprise Portal AAD](https://www.securedsigning.com/aad/Auth/ExternalLogin/AdminPortal), [Wistec Online](https://wisteconline.com/auth/oidc), [Oracle PeopleSoft — protected-IP APM](../saas-apps/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial.md)

Документацию по всем приложениям можно также найти здесь: https://aka.ms/AppsTutorial .

Чтобы получить список приложений в коллекции приложений Azure AD, ознакомьтесь со сведениями в этой статье: https://aka.ms/AzureADAppRequest .

---

### <a name="new-delegation-role-in-azure-ad-entitlement-management-access-package-assignment-manager"></a>Новая роль делегирования в управлении назначением Azure AD: доступ к диспетчеру назначения пакетов

**Тип.** Новая функция.  
**Категория службы:** Управление доступом пользователей  
**Возможности продукта:** Управление назначениями
 
В управление назначением Azure AD добавлена новая роль Диспетчер назначения пакетов Access, которая предоставляет детализированные разрешения для управления назначениями. Теперь вы можете делегировать задачи пользователю в этой роли, который может делегировать управление назначениями пакета доступа владельцу компании. Однако диспетчер назначения пакетов доступа не может изменять политики пакета доступа или другие свойства, заданные администраторами. 

Благодаря этой новой роли вы получите преимущество от минимальных привилегий, необходимых для делегирования управления назначениями и поддержания административного управления во всех остальных конфигурациях пакетов Access. Дополнительные сведения см. в разделе [роли управления](../governance/entitlement-management-delegate.md#entitlement-management-roles)назначением.
 
---

### <a name="changes-to-privileged-identity-managements-onboarding-flow"></a>Изменения в потоке адаптации управление привилегированными пользователями

**Тип.** Измененная функция.  
**Категория службы.** Управление привилегированными пользователями.  
**Возможности продукта.** Управление привилегированными пользователями.
 
Ранее подключение к управление привилегированными пользователями (PIM) требовало согласия пользователя и потока адаптации в колонке PIM, которая включала регистрацию в Azure AD MFA. Благодаря недавней интеграции интерфейса PIM в колонке роли и администраторы Azure AD мы удалим этот интерфейс. Любой клиент с действующей лицензией P2 будет автоматическым образом подключен к PIM.

Подключение к PIM не оказывает никакого прямого воздействия на клиент. Вы можете рассчитывать на следующие изменения:
- Дополнительные параметры назначения, такие как активные или допустимые с временем начала и окончания действия при назначении в колонке "PIM", "роли" и "Администраторы" Azure AD. 
- Дополнительные механизмы определения области, такие как административные единицы и пользовательские роли, вводятся непосредственно в процесс назначения. 
- Если вы являетесь глобальным администратором или администратором привилегированных ролей, вы можете получить несколько дополнительных сообщений электронной почты, таких как Еженедельный дайджест PIM. 
- Кроме того, в журнале аудита, связанном с назначением ролей, может отображаться субъект-служба MS PIM. Это ожидаемое изменение не должно влиять на обычный рабочий процесс.

 Дополнительные сведения см. в разделе [Начало работы с управление привилегированными пользователями](../privileged-identity-management/pim-getting-started.md).

---

### <a name="azure-ad-entitlement-management-the-select-pane-of-access-package-resources-now-shows-by-default-the-resources-currently-in-the-selected-catalog"></a>Управление назначением Azure AD: панель выбора ресурсов пакета Access теперь отображается по умолчанию для ресурсов, находящихся в выбранном каталоге.

**Тип.** Измененная функция.  
**Категория службы:** Управление доступом пользователей  
**Возможности продукта:** Управление назначениями
 

В последовательности создания пакета Access на вкладке роли ресурсов изменяется поведение панели выбор. В настоящее время по умолчанию отображаются все ресурсы, принадлежащие пользователю, и ресурсы, добавленные в выбранный каталог. 

Этот интерфейс будет изменен для просмотра ресурсов, которые в настоящее время добавлены в каталог по умолчанию, чтобы пользователи могли легко выбирать ресурсы из каталога. Обновление поможет обнаружить ресурсы, которые необходимо добавить к пакетам доступа, и снизить риск непреднамеренного добавления ресурсов, принадлежащих пользователю, не входящему в каталог. Дополнительные сведения см. в статье [Создание пакета Access с помощью управления назначением Azure AD](../governance/entitlement-management-access-package-create.md#resource-roles).
 
---