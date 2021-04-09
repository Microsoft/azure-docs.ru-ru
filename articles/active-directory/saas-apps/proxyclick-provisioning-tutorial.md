---
title: Руководство по настройке Proxyclick для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Proxyclick.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/3/2019
ms.author: jeedes
ms.openlocfilehash: f7d2a6f01e891a7fb1c14cde552d66679e474139
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94359176"
---
# <a name="tutorial-configure-proxyclick-for-automatic-user-provisioning"></a>Руководство по настройке Proxyclick для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые нужно выполнить в Proxyclick и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и (или) групп в Proxyclick.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Proxyclick](https://www.proxyclick.com/pricing);
* учетная запись пользователя в Proxyclick с разрешениями администратора.

## <a name="add-proxyclick-from-the-gallery"></a>Добавление Proxyclick из коллекции

Перед настройкой Proxyclick для автоматической подготовки пользователей в Azure AD необходимо добавить Proxyclick из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Proxyclick из коллекции приложений Azure AD, сделайте следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Proxyclick**, выберите **Proxyclick** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Proxyclick в списке результатов](common/search-new-app.png)

## <a name="assigning-users-to-proxyclick"></a>Назначение пользователей в Proxyclick

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Proxyclick. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Proxyclick, следуя приведенным ниже указаниям.

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-proxyclick"></a>Важные советы по назначению пользователей в Proxyclick

* Рекомендуется назначить одного пользователя Azure AD в Proxyclick для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Proxyclick в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configuring-automatic-user-provisioning-to-proxyclick"></a>Настройка автоматической подготовки пользователей в Proxyclick 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Proxyclick на основе их назначений в Azure AD.

> [!TIP]
> Для Proxyclick также можно включить единый вход на основе SAML. Для этого следуйте инструкциям, приведенным в [учебнике по единому входу в Proxyclick](proxyclick-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-proxyclick-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей Proxyclick в Azure AD, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Proxyclick**.

    ![Ссылка на Proxyclick в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. Чтобы получить **URL-адрес клиента** и **секретный токен** учетной записи Proxyclick, выполните пошаговые инструкции для шага 6.

6. Войдите в [консоль администрирования Proxyclick](https://app.proxyclick.com/login//?destination=%2Fdefault). Последовательно выберите **Settings** > **Integrations** > **Browse Marketplace** (Параметры > Интеграции > Обзор Marketplace).

    ![Параметры Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick09.png)

    ![Интеграции Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick01.png)

    ![Proxyclick Marketplace](media/proxyclick-provisioning-tutorial/proxyclick02.png)

    Выберите **Azure AD**. Нажмите кнопку **Установить**.

    ![Azure AD в Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick03.png)

    ![Установка Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick04.png)

    Выберите **User Provisioning** (Подготовка пользователей) и щелкните **Start integration** (Начать интеграцию). 

    ![Подготовка пользователей Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick05.png)

    Теперь в разделе **Settings** > **Integrations** (Параметры > Интеграции) должен отображаться соответствующий пользовательский интерфейс настройки параметров. В разделе **Azure AD (User Provisioning)** (Azure AD (подготовка пользователей)) выберите **Settings** (Параметры).

    ![Создание в Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick06.png)

    Здесь можно найти **URL-адрес клиента** и **секретный токен**.

    ![Создание токена в Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick07.png)

7. После заполнения полей, указанных на шаге 5, щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Proxyclick. Если установить подключение не удалось, убедитесь, что у учетной записи Proxyclick есть разрешения администратора, и повторите попытку.

    ![Токен](common/provisioning-testconnection-tenanturltoken.png)

8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

9. Выберите команду **Сохранить**.

10. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Proxyclick**.

    ![Сопоставления пользователей Proxyclick](media/proxyclick-provisioning-tutorial/Proxyclick-user-mappings.png)

11. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Proxyclick. Атрибуты, выбранные как свойства в разделе **Сопоставление**, используются для сопоставления учетных записей пользователей Proxyclick для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователя Proxyclick](media/proxyclick-provisioning-tutorial/Proxyclick-user-attribute.png)

13. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

14. Чтобы включить службу подготовки Azure AD для Proxyclick, измените значение параметра **Provisioning Status** (Состояние подготовки) на **On** (Включено) в разделе **Настройки**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

15. Определите пользователей и (или) группы для подготовки в Proxyclick, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

16. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Proxyclick.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

* Proxyclick требует, чтобы параметры **emails** и **userName** имели одинаковое исходное значение. При изменении одного из атрибутов изменится также значение второго.
* Proxyclick не поддерживает подготовку для групп.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

