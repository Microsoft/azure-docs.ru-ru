---
title: Руководство по Подготовка пользователей для Asana — Azure AD
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отмены подготовки учетных записей пользователей в Asana.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: arvinh
ms.reviewer: celested
ms.openlocfilehash: 4abc117ae0e983cf684f0e70a363758f9be196aa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94359432"
---
# <a name="tutorial-configure-asana-for-automatic-user-provisioning"></a>Руководство по настройке Asana для автоматической подготовки пользователей

Цель этого руководства — показать, как в Asana и Azure Active Directory (Azure AD) настроить автоматическую подготовку и отзыв учетных записей пользователей из Azure AD в Asana.

## <a name="prerequisites"></a>Предварительные требования

Сценарий, описанный в этом учебнике, предполагает, что у вас уже имеется:

* клиент Azure AD;
* клиент Asana с планом [Корпоративный](https://www.asana.com/pricing) или выше;
* учетная запись пользователя в Asana с разрешениями администратора.

> [!NOTE]
> Интеграция подготовки Azure AD зависит от [API Asana](https://asana.com/developers/api-reference/users), доступного для Asana.

## <a name="assign-users-to-asana"></a>Назначение пользователей в Asana

В Azure AD для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется понятие *назначение*. В контексте автоматической подготовки учетной записи синхронизированы будут только те пользователи, которые назначены приложению в Azure Active Directory.

Перед настройкой и включением службы подготовки необходимо решить, каким пользователям в Azure AD требуется доступ к приложению Asana. После этого можно назначить таких пользователей приложению Asana, сделав следующее:

[Назначение пользователя корпоративному приложению](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-asana"></a>Важные рекомендации по назначению пользователей в Asana

Мы рекомендуем назначить одного пользователя Azure AD в Asana для тестирования конфигурации подготовки. Дополнительных пользователей можно назначить позже.

## <a name="configure-user-provisioning-to-asana"></a>Настройка подготовки учетных записей пользователей в Asana

В этом разделе описывается, как подключить Azure AD к API подготовки учетных записей пользователей Asana. Вы также настроите службу подготовки, чтобы создавать, обновлять или отключать учетные записи назначенных пользователей в Asana на основе назначений пользователя в Azure AD.

> [!TIP]
> Чтобы включить единый вход на основе SAML для Asana, выполните инструкции на [портале Azure](https://portal.azure.com). Единый вход можно настроить независимо от автоматической подготовки, хотя две эти функции дополняют друг друга.

### <a name="to-configure-automatic-user-account-provisioning-to-asana-in-azure-ad"></a>Настройка автоматической подготовки учетных записей пользователей Azure AD в Asana

1. На [портале Azure](https://portal.azure.com) перейдите в раздел **Azure Active Directory** > **Корпоративные приложения** > **Все приложения**.

1. Если вы уже настроили единый вход для Asana, найдите свой экземпляр Asana, используя поле поиска. В противном случае щелкните **Добавить** и найдите **Asana** в коллекции приложений. Выберите **Asana** в результатах поиска и добавьте его в список приложений.

1. Выберите экземпляр Asana и откройте вкладку **Подготовка**.

1. Для параметра **Режим подготовки к работе** выберите значение **Автоматически**.

    ![Подготовка Asana](./media/asana-provisioning-tutorial/asanaazureprovisioning.png)

1. В разделе **Учетные данные администратора** выполните следующие инструкции по созданию токена и введите его в текстовое поле **Секретный токен**.

    a. Войдите в [Asana](https://app.asana.com) с помощью учетной записи администратора.

    b. Щелкните фотографию профиля на верхней панели и выберите текущие параметры для названия организации.

    c. Перейдите на вкладку **Service Accounts** (Учетные записи служб).

    d. Выберите **Add Service Account** (Добавить учетную запись службы).

    д) При необходимости обновите значения в полях **Name** (Имя) и **About** (О себе), а также фотографию профиля. Скопируйте токен в поле **Token** (Токен) и нажмите кнопку **Save Changes** (Сохранить изменения).

1. На портале Azure выберите **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к приложению Asana. Если подключение отсутствует, убедитесь, что учетной записи Asana назначены разрешения администратора, и повторите шаг **Проверить подключение**.

1. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки. Установите флажок под полем.

1. Нажмите кнопку **Сохранить**.

1. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Asana**.

1. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые будут синхронизированы из Azure AD в Asana. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Asana для операций обновления. Чтобы зафиксировать изменения, щелкните **Сохранить**. Дополнительные сведения см. в статье [Настройка сопоставлений атрибутов для подготовки пользователей для приложений SaaS в Azure Active Directory](../app-provisioning/customize-application-attributes.md).

1. Чтобы включить службу подготовки Azure AD для Asana, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

1. Нажмите кнопку **Сохранить**.

После этого запустится начальная синхронизация всех пользователей, назначенных для Asana в разделе **Пользователи**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. Чтобы отслеживать ход выполнения и переходить по ссылкам для просмотра журналов действий по подготовке, используйте раздел **Сведения о синхронизации**. В журналах аудита зафиксированы все действия, выполняемые службой подготовки в приложении Asana.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [созданию отчетов об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Managing user account provisioning for enterprise apps in the Azure portal](../app-provisioning/configure-automatic-user-provisioning-portal.md) (Управление подготовкой учетных записей пользователей для корпоративных приложений на портале Azure)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Настройка единого входа](asana-tutorial.md)
