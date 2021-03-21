---
title: Руководство по настройке Apple School Manager для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения об автоматической подготовке и отзыве учетных записей пользователей из Azure AD в Apple School Manager.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: f006c177-7b35-4af1-84f2-db4a4e2bf96a
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/08/2020
ms.author: Zhchia
ms.openlocfilehash: be21151be719098dd6ec65f369ea6fc81ff45a98
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97591843"
---
# <a name="tutorial-configure-apple-school-manager-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Apple School Manager



В этом учебнике описаны действия, которые необходимо выполнить в Apple School Manager и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически подготавливает и отзывает пользователей в [Apple School Manager](https://school.apple.com/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 

## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Apple School Manager.
> * Удаление из Apple School Manager пользователей, которым больше не нужен доступ.
> * Синхронизация определенных атрибутов пользователей между Azure AD и Apple School Manager.

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) настраивать подготовку (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* Учетная запись Apple School Manager с ролью администратора, менеджера сайта или менеджера пользователей.

> [!NOTE]
> Перемещение токена в Azure AD и установку успешного подключения необходимо выполнить в течение 4 календарных дней. В противном случае процесс нужно будет запустить заново.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и Apple School Manager](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-apple-school-manager-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Apple School Manager для поддержки подготовки с помощью Azure AD

1. Войдите в систему Apple School Manager с помощью учетной записи с ролью администратора, менеджера сайта или менеджера пользователей.
2. Щелкните "Параметры" в нижней части боковой панели, выберите "Источник данных" под параметрами организации, а затем — "Подключиться к источнику данных".
3. Щелкните "Подключиться" рядом с SCIM. Внимательно прочтите предупреждение, щелкните "Копировать", а затем — "Закрыть".
[Окно подключения к SCIM с токеном и кнопкой "Копировать".] Оставьте это окно открытым, чтобы скопировать URL-адрес клиента из Apple Business Manager в Azure AD. Он выглядит так: https://federation.apple.com/feeds/school/scim

    ![Apple School Manager](media/appleschoolmanager-provisioning-tutorial/scim-token.png)

> [!NOTE]
> Секретный токен можно предоставлять лишь администратору Azure AD.

## <a name="step-3-add-apple-school-manager-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Apple School Manager из коллекции приложений Azure AD

Добавьте Apple School Manager из коллекции приложений Azure AD, чтобы начать управление подготовкой в Apple School Manager. Если вы ранее настроили единый вход в Apple School Manager, можно использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей для Apple School Manager необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

## <a name="step-5-configure-automatic-user-provisioning-to-apple-school-manager"></a>Шаг 5. Настройка автоматической подготовки пользователей в Apple School Manager

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Apple School Manager**.

    ![Apple School Manager в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Вкладка "Подготовка"](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Вкладка "Подготовка" с выделенным значением "Авто"](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите полученные из Apple School Manager значения **базового URL-адреса SCIM 2.0 и маркера доступа** в поля **URL-адрес клиента** и **Секретный токен** соответственно. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Apple School Manager. Если установить подключение не удалось, убедитесь, что у учетной записи Apple School Manager есть разрешения администратора, и повторите попытку.

    ![Токен](common/provisioning-testconnection-tenanturltoken.png)

> [!NOTE]
>Если подключение установлено успешно, Apple School Manager отобразит подключение SCIM как активное. Процесс отображения последнего состояния подключения в Apple School Manager может занять до 60 секунд.

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Apple School Manager**.

9. В разделе **Сопоставление атрибутов** просмотрите атрибуты пользователей, которые синхронизированы из Azure AD в Apple School Manager. Атрибуты, которые выбраны в качестве свойств **сопоставления**, будут использоваться для сопоставления учетных записей пользователей в Apple School Manager для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |active|Логическое|
   |userName|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |name.givenName|Строка|
   |externalId|Строка|
   |локаль|Строка|
   |timezone|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для Apple School Manager, в разделе "Параметры" переведите переключатель **Состояние подготовки** в положение **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей и (или) группы для подготовки в Apple School Manager, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут.

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания

После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Проверка требований SCIM для Apple School Manager](https://support.apple.com/guide/apple-school-manager/apdd88331cd6)
* [Использование кода личности в Apple School Manager](https://support.apple.com/guide/apple-school-manager/apd69e1e48e9)
* [Импорт пользователей в Apple School Manager с помощью SCIM](https://support.apple.com/guide/apple-school-manager/apd3ec7b95ad)
* [Разрешение конфликтов учетных записей пользователей SCIM в Apple School Manager](https://support.apple.com/guide/apple-school-manager/apd313013d12)
* [Удаление учетных записей Azure AD, которые отображаются в Apple School Manager](https://support.apple.com/guide/apple-school-manager/apdaa5798fbe)
* [Просмотр действий SCIM в Apple School Manager](https://support.apple.com/guide/apple-school-manager/apd1bfd8dfde)
* [Управление имеющимся токеном и подключениями SCIM в Apple School Manager](https://support.apple.com/guide/apple-school-manager/apdc9a823611)
* [Отключение SCIM в Apple School Manager](https://support.apple.com/guide/apple-school-manager/apd609be3a61)
* [Устранение неполадок с SCIM в Apple School Manager](https://support.apple.com/guide/apple-school-manager/apd403a0f3bd)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)