---
title: Руководство по Настройка автоматической подготовки пользователей в Federated Directory с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure Active Directory автоматическую подготовку и отзыв учетных записей пользователей для Federated Directory.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/12/2019
ms.author: zhchia
ms.openlocfilehash: 8ca7654d930247f70d85cbc20fbbeb961223f05f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95998368"
---
# <a name="tutorial-configure-federated-directory-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Federated Directory

В этом руководстве описаны действия, которые нужно выполнить в Federated Directory и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку и отзыв пользователей и групп в AAD для Federated Directory.

> [!NOTE]
>  В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* Клиент Azure AD.
* [федеративный каталог Federated Directory](https://www.federated.directory/pricing);
* учетная запись пользователя Federated Directory с разрешениями администратора.

## <a name="assign-users-to-federated-directory"></a>Назначение пользователей в Federated Directory
В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется понятие "назначение". В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Federated Directory. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Federated Directory по приведенным ниже инструкциям.

 * [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md) 
 
 ## <a name="important-tips-for-assigning-users-to-federated-directory"></a>Важные замечания о назначении пользователей для Federated Directory
 * Рекомендуется назначить одного пользователя Azure AD для Federated Directory, чтобы проверить конфигурацию автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя для Federated Directory в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если это доступно). Пользователи с ролью Доступ по умолчанию исключаются из подготовки.
    
 ## <a name="set-up-federated-directory-for-provisioning"></a>Настройка подготовки в Federated Directory

Прежде чем настраивать автоматическую подготовку пользователей в AAD для Federated Directory, необходимо включить подготовку SCIM для Federated Directory.

1. Войдите в [консоль администрирования Federated Directory](https://federated.directory/of).

    :::image type="content" source="media/federated-directory-provisioning-tutorial/companyname.png" alt-text="Снимок экрана для консоли администрирования Federated Directory, где отображается поле для ввода имени компании. Также здесь видны кнопки для входа." border="false":::

2. Перейдите к разделу **Directories (Каталоги) > User directories (Каталоги пользователей)** и выберите нужный клиент. 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/ad-user-directories.png" alt-text="Снимок экрана для консоли администрирования Federated Directory, где выделены элементы Directories (Каталоги) и Federated Directory Azure AD Test (Тестирование Azure AD для Federated Directory)." border="false":::

3.  Чтобы создать токен носителя с неограниченным сроком действия, щелкните элементы **Directory Keys (Ключи каталога) > Create New Key (Создать ключ)** . 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/federated01.png" alt-text="Снимок экрана для страницы Directory keys (Ключи каталога) в консоли администрирования Federated Directory. Выделена кнопка Create new key (Создать ключ)." border="false":::

4. Создайте ключ каталога. 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/federated02.png" alt-text="Снимок экрана для страницы Create a directory key (Создание ключа каталога) на консоли администрирования Federated Directory, где выделены поля Name (Имя) и Description (Описание), а также кнопка Create key (Создать ключ)." border="false":::
    

5. Скопируйте значение параметра **Маркер доступа**. Его нужно будет ввести в поле **Секретный токен** на вкладке "Подготовка" для приложения Federated Directory на портале Azure. 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/federated03.png" alt-text="Снимок экрана для страницы в консоли администрирования Federated Directory. На ней отображаются заполнитель для токена доступа, а также значения имени, описания и издателя для ключа." border="false":::
    
## <a name="add-federated-directory-from-the-gallery"></a>Добавление Federated Directory из коллекции

Чтобы настроить в Federated Directory автоматическую подготовку пользователей в Azure AD, нужно добавить Federated Directory из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Federated Directory из коллекции приложений AAD, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Federated Directory** и выберите элемент **Federated Directory** на панели результатов.

    ![Federated Directory в списке результатов](common/search-new-app.png)

5. Перейдите по выделенному ниже **URL-адресу** в другом браузере. 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/loginpage1.png" alt-text="Снимок экрана для портала Azure, где отображается информация о Federated Directory. Выделено значение URL-адреса." border="false":::

6. Щелкните **LOG IN** (Войти в систему).

    :::image type="content" source="media/federated-directory-provisioning-tutorial/federated04.png" alt-text="Снимок экрана с главным меню на сайте Federated Directory. Выделена кнопка Log in (Войти в систему)." border="false":::

7.  Так как Federated Directory — это приложение OpenIDConnect, для входа в Federated Directory необходимо выбрать рабочую учетную запись Майкрософт.
    
    :::image type="content" source="media/federated-directory-provisioning-tutorial/loginpage3.png" alt-text="Снимок экрана для страницы тестирования SCIM для AD на сайте Federated Directory. Выделен элемент Log in with your Microsoft account (Вход с учетной записью Майкрософт)." border="false":::
 
8. После успешной проверки подлинности дайте согласие на странице согласия. После этого приложение автоматически добавится к вашему клиенту и вы будете перенаправлены в свою учетную запись Federated Directory.

    ![Добавление SCIM в Federated Directory](media/federated-directory-provisioning-tutorial/premission.png)



## <a name="configuring-automatic-user-provisioning-to-federated-directory"></a>Настройка автоматической подготовки пользователей в Federated Directory 

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Federated Directory на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-federated-directory-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в AAD для Federated Directory сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите элемент **Federated Directory**.

    ![Ссылка на Federated Directory в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://api.federated.directory/v2/` в поле "URL-адрес клиента". Введите в поле **Секретный токен** полученное и сохраненное ранее значение из Federated Directory. Щелкните элемент **Проверить подключение** и убедитесь, что Azure AD может подключиться к Federated Directory. Если установить подключение не удалось, убедитесь, что у учетной записи Federated Directory есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

9. Выберите команду **Сохранить**.

10. В разделе **Сопоставления** выберите элемент **Синхронизировать пользователей Azure Active Directory с Federated Directory**.

    :::image type="content" source="media/federated-directory-provisioning-tutorial/user-mappings.png" alt-text="Снимок экрана: раздел &quot;Сопоставления&quot;. Под элементом &quot;Имя&quot; выделен элемент &quot;Синхронизировать пользователей Azure Active Directory с Federated Directory&quot;" border="false":::.
    
    
11. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Federated Directory. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления учетных записей пользователей в Federated Directory при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    :::image type="content" source="media/federated-directory-provisioning-tutorial/user-attributes.png" alt-text="Снимок экрана: страница &quot;Сопоставление атрибутов&quot;. В таблице перечислены все атрибуты Azure Active Directory и Federated Directory с указанием состояния сопоставления." border="false":::
    

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Federated Directory, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Укажите пользователей или группы для подготовки в Federated Directory, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Текущее состояние** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Federated Directory.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в учебнике по [отчетам об автоматической подготовке учетных записей пользователей](../app-provisioning/check-status-user-account-provisioning.md).
## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
