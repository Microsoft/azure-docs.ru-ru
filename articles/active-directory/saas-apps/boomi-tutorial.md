---
title: Руководство по интеграции единого входа Azure Active Directory с Boomi | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Boomi.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/25/2021
ms.author: jeedes
ms.openlocfilehash: c58566c628eedd1dbc3d86ae6a142156cbf31211
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104585236"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-boomi"></a>Руководство по интеграции единого входа Azure Active Directory с Boomi

В этом руководстве вы узнаете, как интегрировать Boomi с Azure Active Directory (Azure AD). Интеграция Boomi с Azure AD обеспечивает следующие возможности.

* Контроль доступа к Boomi с помощью Azure AD.
* Включение автоматического входа пользователей в Boomi с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Boomi с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Boomi поддерживает единый вход, инициированный **поставщиком удостоверений**.

## <a name="add-boomi-from-the-gallery"></a>Добавление Boomi из коллекции

Чтобы настроить интеграцию Boomi с Azure AD, необходимо добавить Boomi из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Boomi**.
1. Выберите **Boomi** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-boomi"></a>Настройка и проверка единого входа Azure AD для Boomi

Настройте и проверьте единый вход Azure AD в Boomi с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Boomi.

Чтобы настроить и проверить единый вход Azure AD в Boomi, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    * **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    * **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Boomi](#configure-boomi-sso)** необходима чтобы настроить параметры единого входа на стороне приложения.
    * **[Создание тестового пользователя Boomi](#create-boomi-test-user)** требуется для создания в Boomi пользователя B.Simon, связанного с представлением этого же пользователя в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Boomi** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вам нужно настроить единый вход, инициируемый **поставщиком удостоверений**, и у вас есть **файл метаданных поставщика услуг**, выполните следующие действия в разделе **Базовая конфигурация SAML**:

    а. Щелкните **Отправить файл метаданных**.

    ![Передача файла метаданных](common/upload-metadata.png)

    b. Щелкните **значок папки**, выберите файл метаданных и нажмите кнопку **Отправить**.

    ![Выбор файла метаданных](common/browse-upload-metadata.png)

    c. После успешной передачи файла метаданных значения **Идентификатор** и **URL-адрес ответа** в разделе "Базовая конфигурация SAML" заполнятся автоматически.

    d. Введите **URL-адрес для входа**, например `https://platform.boomi.com/AtomSphere.html#build;accountId={your-accountId}`.

    > [!Note]
    > Вы получите **файл метаданных поставщика службы** из раздела, посвященного **настройке единого входа в Boomi**, которая будет рассматриваться далее в этом руководстве. Если поля **Идентификатор** и **URL-адрес ответа** автоматически не заполняются, введите эти значения вручную в соответствии с поставленной задачей.

1. Приложение Boomi ожидает проверочные утверждения SAML в определенном формате, который требует добавить настраиваемые сопоставления атрибутов в конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию.

    ![На снимке экрана показаны раздел "Атрибуты и утверждения пользователя" со значениями по умолчанию, такими как "Givenname: user.givenname" и "Emailaddress: user.mail"](common/default-attributes.png)

1. В дополнение к описанному выше приложение Boomi ожидает несколько дополнительных атрибутов в ответе SAML, как показано ниже. Эти атрибуты также заранее заполнены, но вы можете изменить их в соответствии со своими требованиями.

    | Имя |  Исходный атрибут|
    | ---------------|  --------- |
    | FEDERATION_ID | user.mail |

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите пункт **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка Boomi**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как на портале Azure создать тестового пользователя с именем B.Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `B.Simon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к Boomi.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Boomi**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-boomi-sso"></a>Настройка единого входа в Boomi

1. В другом окне браузера войдите на свой корпоративный сайт Boomi с правами администратора.

1. Перейдите к пункту **Company Name** (Название компании) и выберите **Set up** (Настройка).

1. Щелкните вкладку **SSO Options** (Параметры единого входа) и выполните следующие действия:

    ![Настройка единого входа на стороне приложения](./media/boomi-tutorial/import.png)

    а. Установите флажок **Enable SAML Single Sign-On** (Разрешить единый вход SAML).

    b. Щелкните **Import** (Импорт), чтобы передать сертификат, загруженный из Azure AD, в качестве **сертификата поставщика удостоверений**.

    c. В текстовое поле **URL-адрес для входа поставщика удостоверений** вставьте значение **URL-адрес входа** из окна конфигурации приложения Azure AD.

    d. Для параметра **Federation Id Location** (Расположение идентификатора федерации) выберите переключатель **Federation Id is in FEDERATION_ID Attribute element** (Идентификатор федерации передается в элементе атрибута FEDERATION_ID).

    д) Скопируйте **URL-адрес метаданных AtomSphere**, перейдите по **URL-адресу метаданных** с помощью браузера по своему усмотрению и сохраните выходные данные в файл. Отправьте **URL-адрес метаданных** в разделе **базовой конфигурации SAML** на портале Azure.

    е) Нажмите кнопку **Сохранить** .

### <a name="create-boomi-test-user"></a>Создание тестового пользователя Boomi

Чтобы пользователи Azure AD могли выполнять вход в Boomi, они должны быть подготовлены для него. В случае с Boomi подготовка выполняется вручную.

### <a name="to-provision-a-user-account-perform-the-following-steps"></a>Чтобы подготовить учетную запись пользователя, выполните следующие действия.

1. Войдите на корпоративный сайт Boomi в качестве администратора.

1. После входа в систему, перейдите к разделу **User Management** (Управление пользователями) и найдите элемент **Users** (Пользователи).

    ![Снимок экрана, на котором показана страница "User Management" (Управление пользователями) с выбранным элементом "Users" (Пользователи)](./media/boomi-tutorial/user.png "Пользователи")

1. Щелкните значок **+** . Откроется диалоговое окно **Add/Maintain User Roles** (Добавить или настроить роли пользователей).

    ![Снимок экрана, на котором показан выбранный значок "+"](./media/boomi-tutorial/add.png "Пользователи")

    ![Снимок экрана, на котором показано диалоговое окно "Add/Maintain User Roles" (Добавить или настроить роли пользователей), где можно настроить параметры пользователя](./media/boomi-tutorial/roles.png "Пользователи")

    а. В текстовом поле **User e-mail address** (Адрес электронной почты пользователя) введите электронный адрес пользователя, например B.Simon@contoso.com.

    b. В текстовом поле **First name** (Имя) введите имя пользователя, например B.

    c. В текстовом поле **Last name** (Фамилия) введите фамилию, предположим, Simon.

    d. Введите **идентификатор федерации** пользователя. Каждый пользователь должен иметь уникальный идентификатор федерации в пределах учетной записи.

    д) Назначьте для пользователя роль **Standard User** (Обычный пользователь). Не назначайте роль администратора, так как она не только обеспечивает возможность единого входа, но и предоставляет и обычный доступ к AtomSphere.

    е) Нажмите кнопку **ОК**.

    > [!NOTE]
    > Пользователь не получит по электронной почте сообщение с паролем для входа с учетной записью AtomSphere. Вместо этого управление паролем осуществляется через поставщик удостоверений. Вы можете использовать для создания учетной записи Boomi любые другие средства или интерфейсы API, которые Boomi предоставляет для подготовки учетных записей AAD.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* Выберите "Тестировать приложение" на портале Azure, и вы автоматически войдете в приложение Boomi, для которого настроен единый вход.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Boomi на портале "Мои приложения", вы автоматически войдете в приложение Boomi, для которого настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки Boomi вы можете применить функцию управления сеансами, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).