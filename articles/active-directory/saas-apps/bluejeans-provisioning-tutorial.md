---
title: Руководство по настройке BlueJeans для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в BlueJeans.
services: active-directory
author: zhchia
writer: zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 58cd69ebe97d9d0965d7e648b0ded012ac71cd0d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101646135"
---
# <a name="tutorial-configure-bluejeans-for-automatic-user-provisioning"></a>Руководство по настройке BlueJeans для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в BlueJeans и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей для [BlueJeans](https://www.bluejeans.com/pricing) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 

## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в BlueJeans.
> * Удаление пользователей в BlueJeans, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей в Azure AD и BlueJeans.
> * [Единый вход](./bluejeans-tutorial.md) в BlueJeans (рекомендуется).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md)
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) настраивать подготовку (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* Включенный арендатор BlueJeans с планом [My Company](https://www.bluejeans.com/pricing) или более высокого уровня.
* Учетная запись пользователя в BlueJeans с разрешениями администратора.
* Включенная подготовка SCIM в BlueJeans Enterprise.

> [!NOTE]
> Интеграция подготовки в Azure AD зависит от [API BlueJeans](https://BlueJeans.github.io/developer), доступного для команд BlueJeans, использующих план "Стандартный" или более высокого уровня.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и BlueJeans](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-bluejeans-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка BlueJeans для поддержки подготовки с помощью Azure AD

1. Войдите в консоль администрирования BlueJeans. Перейдите к разделу Group Settings > Security (Параметры группы > Безопасность).
2. Выберите **Single Sign On** (Единый вход) и **Configure Now** (Настроить сейчас).

    ![now](./media/bluejeans-provisioning-tutorial/configure.png)

3. В окне **Provision & Integration** (Подготовка и интеграция), выберите **Create and manage user accounts through IDP** (Создание учетных записей пользователей и управление ими с помощью поставщика удостоверений) и щелкните **GENERATE TOKEN** (Создать маркер).

    ![generate](./media/bluejeans-provisioning-tutorial/token.png)
    
4. Скопируйте и сохраните маркер. 
5. URL-адрес арендатора BlueJeans — `https://api.bluejeans.com/v2/scim`. Значения **URL-адрес клиента** и **Маркер секрета**, полученные на предыдущем этапе, нужно ввести на вкладке "Подготовка" приложения BlueJeans на портале Azure.

## <a name="step-3-add-bluejeans-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление BlueJeans из коллекции приложений Azure AD

Добавьте BlueJeans из коллекции приложений Azure AD, чтобы начать управление подготовкой в BlueJeans. Если вы ранее настроили BlueJeans для единого входа, можете использовать то же приложение. Но мы рекомендуем создать отдельное приложение для исходной проверки интеграции. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя. Если вы решили определить пользователей на основе назначения, выполните указанные ниже [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей приложению. Если вы решили определить пользователей только на основе атрибутов пользователя, примените фильтр области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей для BlueJeans необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с такой ролью исключаются из подготовки и будут помечены в журналах подготовки как не имеющие разрешений. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи, проверьте этот механизм, назначив приложению одного или двух пользователей. Если в область включены все пользователи, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

## <a name="step-5-configure-automatic-user-provisioning-to-bluejeans"></a>Шаг 5. Настройка автоматической подготовки пользователей в BlueJeans

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей в TestApp на основании назначения пользователей в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-bluejeans-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в BlueJeans:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **BlueJeans**.

    ![Ссылка на BlueJeans в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Вкладка "Подготовка" с выделенным значением "Авто"](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** укажите URL-адрес арендатора и маркер секрета BlueJeans, полученные на шаге 2. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к BlueJeans. Если установить подключение не удалось, убедитесь, что учетной записи BlueJeans назначены разрешения администратора, и повторите попытку.

    ![Токен](common/provisioning-testconnection-tenanturltoken.png)


6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя, который будет получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to BlueJeans** (Синхронизировать пользователей Azure Active Directory с BlueJeans).

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в BlueJeans. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в BlueJeans для операций обновления. Если вы решили изменить [соответствующий целевой атрибут](../app-provisioning/customize-application-attributes.md), нужно убедиться, что API BlueJeans поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

|attribute|Тип|Поддерживается для фильтрации|
|---|---|---|
|userName|Строка|&check;|
|active|Логическое|
|title|Строка|
|emails[type eq "work"].value|Строка|
|name.givenName|Строка|
|name.familyName|Строка|
|phoneNumbers[type eq "work"].value|Строка|
|urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Строка|

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для BlueJeans, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей для подготовки в BlueJeans, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="connector-limitations"></a>Ограничения соединителя

* В Bluejeans нельзя использовать имена пользователей длиннее 30 знаков.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)