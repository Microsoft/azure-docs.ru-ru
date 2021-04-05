---
title: Руководство по настройке TeamViewer для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить автоматическую подготовку и учетных записей пользователей и ее отмену из Azure AD в TeamViewer.
services: active-directory
author: Zhchia
writer: Zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/27/2020
ms.author: Zhchia
ms.openlocfilehash: 0362f0d69581c252170b4f37f8564caa7acd3d29
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96182246"
---
# <a name="tutorial-configure-teamviewer-for-automatic-user-provisioning"></a>Руководство по настройке TeamViewer для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в TeamViewer и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей и групп для [TeamViewer](https://www.teamviewer.com/buy-now/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в TeamViewer.
> * Удаление пользователей в TeamViewer, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей между Azure AD и TeamViewer.
> * [Единый вход](./teamviewer-tutorial.md) в TeamViewer (рекомендуется).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* Действительная [лицензия на Tensor](https://www.teamviewer.com/de/teamviewer-tensor/) для TeamViewer.
* Действительный настраиваемый идентификатор из доступной конфигурации [единого входа](https://community.teamviewer.com/t5/Knowledge-Base/Single-Sign-On-with-Azure-Active-Directory/ta-p/60209#toc-hId--473669723).

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и TeamViewer](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-teamviewer-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка TeamViewer для поддержки подготовки с помощью Azure AD

1. Войдите в [консоль управления TeamViewer](https://login.teamviewer.com). Выберите **Edit Profile** (Изменить профиль).

    ![Консоль администрирования TeamViewer](./media/teamviewer-provisioning-tutorial/admin.png)

2.  Перейдите к разделу **Apps** (Приложения). Щелкните **Create Script Token** (Создать токен скрипта).

    ![Создание токена в TeamViewer](./media/teamviewer-provisioning-tutorial/createtoken.png)

3.  Введите имя токена скрипта. Нажмите кнопку **Save** (Сохранить).

    ![Имя токена TeamViewer](./media/teamviewer-provisioning-tutorial/tokenname.png)

4. Скопируйте **токен** и нажмите кнопку **OK** (ОК). Его нужно будет ввести на вкладке в поле **Секретный токен** для приложения TeamViewer на портале Azure.

    ![Токен TeamViewer](./media/teamviewer-provisioning-tutorial/token.png)

## <a name="step-3-add-teamviewer-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление TeamViewer из коллекции приложений Azure AD

Добавьте TeamViewer из коллекции приложений Azure AD, чтобы начать управление подготовкой в TeamViewer. Если вы ранее настроили TeamViewer для единого входа, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для TeamViewer необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-teamviewer"></a>Шаг 5. Настройка автоматической подготовки пользователей в TeamViewer 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-teamviewer-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей TeamViewer в Azure AD, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **TeamViewer**.

    ![Ссылка на TeamViewer в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите `ttps://webapi.teamviewer.com/scim/v2` в поле **URL-адрес клиента**, а затем введите созданный ранее токен скрипта в поле **Секретный токен**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к TeamViewer. Если установить подключение не удалось, убедитесь, что у учетной записи TeamViewer есть разрешения администратора, и повторите попытку.

    ![Снимок экрана: диалоговое окно учетных данных администратора, в котором можно ввести URL-адрес клиента и секретный токен.](./media/teamViewer-provisioning-tutorial/provisioning.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с TeamViewer**.

9. В разделе **Сопоставления атрибутов** просмотрите пользовательские атрибуты, которые синхронизируются из Azure AD в TeamViewer. Атрибуты, выбранные как свойства в разделе **Сопоставления**, используются для сопоставления учетных записей пользователей в TeamViewer для операций обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), сначала убедитесь, что API TeamViewer поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |displayName|Строка|
   |active|Логическое|

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для TeamViewer, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей и (или) группы для подготовки в TeamViewer, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

* Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
* Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
* Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)