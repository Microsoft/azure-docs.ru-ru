---
title: Руководство по интеграции Azure Active Directory с E Sales Manager Remix | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и E Sales Manager Remix.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/12/2018
ms.author: jeedes
ms.openlocfilehash: c06595b683092abf52300481068daab26394c4cb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95995647"
---
# <a name="integrate-azure-active-directory-with-e-sales-manager-remix"></a>Интеграция Azure Active Directory с E Sales Manager Remix

В этом руководстве описано, как интегрировать Azure Active Directory (Azure AD) с E Sales Manager Remix.

Интеграция Azure AD с приложением E Sales Manager Remix обеспечивает следующие преимущества:

- С помощью Azure AD можно контролировать доступ к E Sales Manager Remix.
- Вы можете включить для пользователей автоматический вход в E Sales Manager Remix (единый вход) с использованием учетных записей Azure AD.
- Вы можете управлять учетными записями централизованно, через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с E Sales Manager Remix, вам потребуется следующее.

- подписка Azure AD;
- подписка E Sales Manager Remix с поддержкой единого входа.

> [!NOTE]
> Мы *не рекомендуем* использовать рабочую среду для тестирования действий, описанных в этом руководстве.

При проверке действий в этом руководстве соблюдайте следующие рекомендации:

- Не используйте рабочую среду без необходимости.
- Если у вас нет пробной среды Azure AD, вы можете [получить пробную версию на один месяц](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Описание сценария
В рамках этого руководства проводится проверка единого входа Azure AD в тестовой среде. 

Сценарий, описанный в этом учебнике, состоит из двух стандартных блоков.

* Добавление E Sales Manager Remix из коллекции
* настройка и проверка единого входа в Azure AD.

## <a name="add-e-sales-manager-remix-from-the-gallery"></a>Добавление E Sales Manager Remix из коллекции
Чтобы настроить интеграцию Azure AD с E Sales Manager Remix, необходимо добавить E Sales Manager Remix из коллекции в список управляемых приложений SaaS, выполнив следующие действия:

1. На [портале Azure](https://portal.azure.com) в области слева щелкните **Azure Active Directory**. 

    ![Кнопка Azure Active Directory][1]

1. Щелкните **Корпоративные приложения** > **Все приложения**.

    ![Окно "Корпоративные приложения"][2]
    
1. Чтобы добавить новое приложение, выберите **Новое приложение** в верхней части окна.

    ![Кнопка "Создать приложение"][3]

1. В поле поиска введите **E Sales Manager Remix**, а затем выберите **E Sales Manager Remix** в списке результатов и щелкните **Добавить**.

    ![E Sales Manager Remix в списке результатов](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в E Sales Manager Remix с использованием тестового пользователя Britta Simon.

Чтобы настроить единый вход в Azure AD, необходимо определить, какой пользователь в E Sales Manager Remix соответствует пользователю в Azure Active Directory. Иными словами, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в E Sales Manager Remix.

Чтобы настроить и проверить единый вход Azure AD в E Sales Manager Remix, выполните действия в следующих пяти разделах.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure и настроить его в приложении E Sales Manager Remix:

1. На портале Azure на странице интеграции с приложением **E Sales Manager Remix** выберите **Единый вход**.

    ![Ссылка "Единый вход"][4]

1. В окне **Единый вход** в поле **Режим единого входа** выберите **Вход на основе SAML**.
 
    ![Окно "Единый вход"](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_samlbase.png)

1. В разделе **URL-адреса и домены E Sales Manager Remix** выполните следующие действия:

    ![Сведения о домене и URL-адресах единого входа для E Sales Manager Remix](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_url.png)

    a. В поле **URL-адрес входа** введите URL-адрес в следующем формате: *https://\<Server-Based-URL>/\<sub-domain>/esales-pc*.

    b. В поле **Идентификатор** введите URL-адрес в следующем формате: *https://\<Server-Based-URL>/\<sub-domain>/* .

    c. Запишите значение **идентификатора**, оно вам понадобится далее в этом руководстве.
    
    > [!NOTE] 
    > Приведенные выше значения используются только для примера. Замените их фактическими значениями URL-адреса входа и идентификатора. Обратитесь к [группе поддержки клиентов E Sales Manager Remix](mailto:esupport@softbrain.co.jp), чтобы получить эти значения.

1. В разделе **Сертификат подписи SAML** щелкните **Сертификат (Base64)** и сохраните файл сертификата на компьютере.

    ![Ссылки для скачивания сертификата в кодировке Base64](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_certificate.png) 

1. Установите флажок **Просмотреть и изменить все другие атрибуты пользователей** и выберите атрибут **emailaddress**.
    
    ![Окно "Атрибуты пользователя"](./media/esalesmanagerremix-tutorial/configure1.png)

    Откроется окно **Изменение атрибута**.

1. Скопируйте из него значения параметров **Пространство имен** и **Имя**. Создайте значение в формате *\<Namespace>/\<Name>* и сохраните его, чтобы использовать дальше при работе с этим руководством.

    ![Окно "Изменение атрибута"](./media/esalesmanagerremix-tutorial/configure2.png)

1. В разделе **Конфигурация E Sales Manager Remix** выберите **Настроить E Sales Manager Remix**.

    ![Снимок экрана: раздел "Конфигурация E Sales Manager Remix" с выбранным параметром "Настроить E Sales Manager Remix".](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_configure.png) 

    Откроется окно **Настройка единого входа**

1. Из раздела **Краткий справочник** скопируйте URL-адрес выхода и URL-адрес службы единого входа SAML.

1. Нажмите кнопку **Сохранить**.

    ![Кнопка "Сохранить"](./media/esalesmanagerremix-tutorial/tutorial_general_400.png)

1. Войдите в приложение E Sales Manager Remix в качестве администратора.

1. Выберите действие **To Administrator Menu** (К меню администратора) в правом верхнем углу.

    ![Команда "To Administrator Menu" (К меню администратора)](./media/esalesmanagerremix-tutorial/configure4.png)

1. В левой области выберите **System settings** > **Cooperation with external system** (Параметры системы > Взаимодействие с внешней системой).

    ![Ссылки "Параметры системы" и "Взаимодействие с внешней системой"](./media/esalesmanagerremix-tutorial/configure5.png)
    
1. В окне **Cooperation with external system** (Взаимодействие с внешней системой) выберите **SAML**.

    ![Окно "Взаимодействие с внешней системой"](./media/esalesmanagerremix-tutorial/configure6.png)

1. В разделе **SAML authentication setting** (Настройка аутентификации SAML) выполните следующие действия:

    ![Раздел "SAML authentication setting" (Настройка аутентификации SAML)](./media/esalesmanagerremix-tutorial/configure3.png)
    
    a. Установите флажок **PC version** (Версия для компьютера).
    
    b. Из раскрывающегося списка в разделе **Collaboration item** (Элемент совместной работы) выберите значение **email** (Электронная почта).

    c. В поле **Collaboration item** (Элемент совместной работы) вставьте значение утверждения, которое вы ранее скопировали на портале Azure (**`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`**).

    d. В текстовое поле **Issuer (entity ID)** (Издатель или идентификатор сущности) вставьте значение идентификатора, скопированное ранее из раздела **Домены и URL-адреса приложения E Sales Manager Remix** на портале Azure.

    д) Чтобы передать скачанный с портала Azure сертификат, щелкните **File selection** (Выбор файла).

    е) В поле **ID provider login URL** (URL-адрес входа поставщика удостоверений) вставьте URL-адрес службы единого входа SAML, скопированный на портале Azure.

    ж. В поле **Identity Provider Logout URL** (URL-адрес выхода поставщика удостоверений) вставьте URL-адрес выхода, скопированный на портале Azure.

    h. Выберите **Setting complete** (Завершить настройку).

> [!TIP]
> Настроив приложение, вы можете прочитать краткую версию предыдущих инструкций на [портале Azure](https://portal.azure.com). Добавив приложение из раздела **Active Directory** > **Корпоративные приложения**, выберите вкладку **Единый вход** и откройте встроенную документацию через раздел **Настройка** внизу. На [этой странице]( https://go.microsoft.com/fwlink/?linkid=845985) вы можете получить дополнительную информацию о встроенной документации Azure AD.
> 

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе вы создадите на портале Azure тестового пользователя Britta Simon, выполнив следующие действия.

![Создание тестового пользователя Azure AD][100]

1. На портале Azure в области слева щелкните **Azure Active Directory**.

    ![Ссылка на Azure Active Directory](./media/paloaltoadmin-tutorial/create_aaduser_01.png)

1. Чтобы отобразить список активных пользователей, выберите **Пользователи и группы** > **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](./media/paloaltoadmin-tutorial/create_aaduser_02.png)

1. В верхней части окна **Все пользователи** выберите **Добавить**.

    ![Кнопка "Добавить"](./media/paloaltoadmin-tutorial/create_aaduser_03.png)
    
    Откроется окно **Пользователь**.

1. В окне **Пользователь** сделайте следующее.

    ![Окно пользователя](./media/paloaltoadmin-tutorial/create_aaduser_04.png)

    a. В поле **Имя** введите **BrittaSimon**.

    b. В поле **Имя пользователя** введите адрес электронной почты для пользователя Britta Simon.

    c. Установите флажок **Показать пароль** и сохраните значение, которое отображается в поле **Пароль**.

    d. Нажмите кнопку **создания**.
 
### <a name="create-an-e-sales-manager-remix-test-user"></a>Создание тестового пользователя E Sales Manager Remix

1. Войдите в приложение E Sales Manager Remix в качестве администратора.

1. Выберите **To Administrator Menu** (К меню администратора) в меню в правом верхнем углу.

    ![Конфигурация E Sales Manager Remix](./media/esalesmanagerremix-tutorial/configure4.png)

1. Выберите **Your company's settings** > **Maintenance of departments and employees** (Параметры вашей компании > Обслуживание отделов и сотрудников), а затем щелкните **Employees registered** (Зарегистрированные сотрудники).

    ![Вкладка "Employees registered" (Зарегистрированные сотрудники)](./media/esalesmanagerremix-tutorial/user1.png)

1. В разделе **New employee registration** (Регистрация нового сотрудника) сделайте следующее:
    
    ![Раздел "New employee registration" (Регистрация нового сотрудника)](./media/esalesmanagerremix-tutorial/user2.png)

    a. В поле **Employee Name** (Имя сотрудника) введите имя пользователя (например, **Britta**).

    b. Заполните остальные обязательные поля.
    
    c. Если включен SAML, администратор не сможет войти через страницу входа. Предоставьте этому пользователю права администратора, установив флажок **Admin Login** (Вход администратора).

    d. Выберите **Регистрация**.

1. Чтобы позднее выполнить вход от имени администратора, войдите с учетными данными пользователя с правами администратора, и в правом верхнем углу выберите действие **To Administrator Menu** (К меню администратора).

    ![Команда "To Administrator Menu" (К меню администратора)](./media/esalesmanagerremix-tutorial/configure4.png)

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к E Sales Manager Remix. Для этого выполните следующее. 

![Назначение роли пользователя][200] 

1. На портале Azure откройте представление **Приложения**, перейдите к представлению **Каталог** и выберите **Корпоративные приложения** > **Все приложения**.

    ![Ссылки "Корпоративные приложения" и "Все приложения"][201] 

1. Из списка **Приложения** выберите **E Sales Manager Remix**.

    ![Ссылка на приложение E Sales Manager Remix](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_app.png)  

1. В области слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"][202]

1. Нажмите кнопку **Добавить**, затем в области **Добавление назначения** щелкните **Пользователи и группы**.

    ![Область "Добавление назначения"][203]

1. В окне **Пользователи и группы** в списке **Пользователи** выберите **Britta Simon**.

1. Нажмите кнопку **Выбрать**.

1. В окне **Добавление назначения** выберите **Назначить**.
    
### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Выбрав элемент "E Sales Manager Remix" на панели доступа, вы автоматически войдете в приложение E Sales Manager Remix.

См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md) 

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Список руководств, посвященных интеграции приложений SaaS с Azure Active Directory](tutorial-list.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/esalesmanagerremix-tutorial/tutorial_general_01.png
[2]: ./media/esalesmanagerremix-tutorial/tutorial_general_02.png
[3]: ./media/esalesmanagerremix-tutorial/tutorial_general_03.png
[4]: ./media/esalesmanagerremix-tutorial/tutorial_general_04.png

[100]: ./media/esalesmanagerremix-tutorial/tutorial_general_100.png

[200]: ./media/esalesmanagerremix-tutorial/tutorial_general_200.png
[201]: ./media/esalesmanagerremix-tutorial/tutorial_general_201.png
[202]: ./media/esalesmanagerremix-tutorial/tutorial_general_202.png
[203]: ./media/esalesmanagerremix-tutorial/tutorial_general_203.png