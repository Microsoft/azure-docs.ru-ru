---
title: Руководство по настройке Jive для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как с помощью Jive и Azure AD выполнять автоматическую подготовку и отмену подготовки учетных записей пользователей из Azure AD в Jive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: ebee5d986007e07d497056620f0cfc437b2da4d1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94356405"
---
# <a name="tutorial-configure-jive-for-automatic-user-provisioning"></a>Руководство по настройке Jive для автоматической подготовки пользователей

Цель этого руководства — показать, как в Jive и Azure AD настроить автоматическую подготовку и отзыв учетных записей пользователей из Azure AD в Jive.

## <a name="prerequisites"></a>Предварительные требования

Сценарий, описанный в этом учебнике, предполагает, что у вас уже имеется:

*   клиент Azure Active Directory;
*   подписка Jive с поддержкой единого входа;
*   учетная запись пользователя Jive с разрешениями администратора команды.

## <a name="assigning-users-to-jive"></a>Назначение пользователей в Jive

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая "назначение". В контексте автоматической подготовки учетной записи будут синхронизированы только те записи и группы, которые были "назначены" приложению в Azure AD.

Перед настройкой и включением службы подготовки необходимо решить, какие пользователи или группы в Azure AD представляют пользователей, которым требуется доступ к приложению Jive. Когда данное решение будет принято, можно будет назначить этих пользователей приложению Jive, следуя указаниям ниже.

[Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-jive"></a>Важные рекомендации по назначению пользователей в Jive

*   Рекомендуется назначить одного пользователя Azure AD в Jive для тестирования конфигурации подготовки. Дополнительные пользователи и/или группы можно назначить позднее.

*   При назначении пользователя Jive необходимо выбрать допустимую роль пользователя. Роль "Доступ по умолчанию" не работает для подготовки.

## <a name="enable-user-provisioning"></a>Включение подготовки пользователей

В этом разделе описывается подключение вашего каталога Azure AD к API подготовки учетных записей пользователей в Jive и настройка службы подготовки для создания, обновления и отключения назначенных учетных записей пользователей в Jive на основе назначения пользователей и групп в Azure AD.

> [!TIP]
> Для Jive можно также включить единый вход на основе SAML. Для этого следуйте инструкциям на [портале Azure](https://portal.azure.com). Единый вход можно настроить независимо от автоматической подготовки, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-user-account-provisioning"></a>Чтобы настроить подготовку учетных записей пользователей:

В этом разделе показано, как включить подготовку учетных записей пользователей Active Directory для Jive.
В рамках этой процедуры потребуется предоставить маркер безопасности пользователя, который необходимо запросить на веб-сайте Jive.com.

1. На [портале Azure](https://portal.azure.com) перейдите в раздел **Azure Active Directory > Корпоративные приложения > Все приложения**.

1. Если в Jive уже настроен единый вход, найдите свой экземпляр Jive с помощью поля поиска. В противном случае щелкните **Добавить** и выполните поиск **Jive** в коллекции приложений. Выберите Jive в результатах поиска и добавьте в список приложений.

1. Выберите экземпляр Jive, а затем перейдите на вкладку **Подготовка**.

1. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически). 

    ![Снимок экрана: страницы "Jive — Подготовка" со значением "Авто" для параметра "Режим подготовки" и другими значениями, которые можно задать.](./media/jive-provisioning-tutorial/provisioning.png)

1. В разделе **Учетные данные администратора** укажите следующие параметры конфигурации.
   
    a. В текстовом поле **Jive Admin User Name** (Имя пользователя администратора Jive) укажите имя учетной записи Jive, которой назначен профиль **System Administrator** (Системный администратор) на сайте Jive.com.
   
    b. В текстовом поле **Jive Admin Password** (Пароль администратора Jive) введите пароль для этой учетной записи.
   
    c. В текстовом поле **Jive Tenant URL** (URL-адрес клиента Jive) введите URL-адрес клиента Jive.
      
      > [!NOTE]
      > URL-адрес клиента Jive — это URL-адрес, используемый вашей организацией для входа в Jive.  
      > Как правило, этот URL-адрес имеет следующий формат: **www.\<organization\>.jive.com**.          

1. На портале Azure щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к приложению Jive.

1. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок ниже.

1. Нажмите кнопку **Сохранить**.

1. В разделе "Сопоставления" выберите **Synchronize Azure Active Directory Users to Jive** (Синхронизировать пользователей Azure Active Directory с Jive).

1. В разделе **Сопоставления атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Jive. Атрибуты, выбранные как свойства **Matching**, используются для сопоставления учетных записей пользователей в Jive для операций обновления. Нажмите кнопку "Сохранить", чтобы подтвердить все изменения.

1. Чтобы включить службу подготовки Azure AD для Jive, в разделе "Параметры" измените значение параметра **Состояние подготовки** на **Включено**.

1. Нажмите кнопку **Сохранить**.

После этого будет запущена начальная синхронизация всех пользователей и групп, назначенных для Jive в разделе "Пользователи и группы". Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход синхронизации и с помощью ссылок просматривать журналы действий по подготовке, в которых описаны все действия, выполняемые службой подготовки для приложения Jive.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](tutorial-list.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Настройка единого входа](jive-tutorial.md)