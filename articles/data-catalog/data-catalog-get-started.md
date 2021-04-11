---
title: Краткое руководство. Создание Каталога данных Azure
description: В этом кратком руководстве описано, как создать Каталог данных Azure с помощью портала Azure.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: quickstart
ms.date: 05/26/2020
ms.openlocfilehash: a56e5a4cae7c5a8e931b074f08a7152e53a8eb31
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "104674739"
---
# <a name="quickstart-create-an-azure-data-catalog"></a>Краткое руководство. Создание Каталога данных Azure

[!INCLUDE [Azure Purview redirect](../../includes/data-catalog-use-purview.md)]

Каталог данных Azure — это полностью управляемая облачная служба, выполняющая функции систем регистрации и обнаружения корпоративных ресурсов данных. Подробный обзор см. в статье [Что такое каталог данных Azure?](overview.md)

Это краткое руководство поможет вам приступить к созданию Каталога данных Azure.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

> [!Note]
> Согласно требованиям безопасности в Каталоге данных Azure применяется протокол TLS 1.2. Протоколы TLS 1.0 и TLS 1.1 отключены. Если на компьютере не выполнено обновление до TLS 1.2, при запуске средства регистрации могут возникать ошибки. Сведения о том, как выполнить это обновление, см. в статье [Как включить TLS 1.2](/mem/configmgr/core/plan-design/security/enable-tls-1-2).

Для начала работы необходимы перечисленные ниже компоненты и данные.

* [Подписка Microsoft Azure](https://azure.microsoft.com/).
* Вам потребуется собственный [арендатор Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

Чтобы настроить каталог данных, необходимо быть владельцем или совладельцем подписки Azure.

## <a name="create-a-data-catalog"></a>Создание каталога данных

В рамках организации можно подготовить только один каталог данных (в домене Azure Active Directory). Таким образом, если владелец или совладелец подписки Azure, принадлежащий домену Azure Active Directory, уже создал каталог, создать еще один каталог невозможно, даже если в организации есть несколько подписок Azure. Чтобы проверить наличие каталога данных в домене Azure Active Directory, перейдите на [домашнюю страницу каталога данных Azure](http://azuredatacatalog.com). Если он создан, вы увидите его на этой странице. Если каталог уже создан, пропустите описанную ниже процедуру и перейдите к следующему разделу.

1. Перейдите на [портал Azure](https://portal.azure.com) > , выберите **Создать ресурс** и **Каталог данных**.

    ![Кнопка создания Каталога данных Azure](media/data-catalog-get-started/data-catalog-create.png)

2. Укажите **имя** каталога данных, **подписку**, которую необходимо использовать, **расположение**, в котором будет находиться каталог, и **ценовую категорию**. Щелкните **Создать**.

3. Перейдите на [домашнюю страницу каталога данных Azure](http://azuredatacatalog.com) и нажмите кнопку **Опубликовать данные**.

   ![Каталог данных Azure — кнопка публикации данных](media/data-catalog-get-started/data-catalog-publish-data.png)

   Доступ к домашней странице Каталога данных также можно получить со [страницы службы Каталога данных](https://azure.microsoft.com/services/data-catalog), выбрав **Начало работы**.

   ![Каталог данных Azure — маркетинговая целевая страница](media/data-catalog-get-started/data-catalog-marketing-landing-page.png)

4. Перейдите на страницу **Параметры**.

    ![Каталог данных Azure — подготовка каталога данных к работе](media/data-catalog-get-started/data-catalog-create-azure-data-catalog.png)

5. Разверните раздел **Цены** и проверьте тип **выпуска** ("Стандартный&quot; или &quot;Бесплатный") для Каталога данных Azure.

    ![Каталог данных Azure — выбор выпуска](media/data-catalog-get-started/data-catalog-create-catalog-select-edition.png)

6. При выборе ценовой категории *Стандартный* можно развернуть **Группы безопасности** и включить авторизацию групп безопасности Active Directory для доступа к Каталогу данных и включения автоматической настройки выставления счетов.

    ![Группы безопасности Каталога данных Azure](media/data-catalog-get-started/data-catalog-standard-security-groups.png)

7. Разверните раздел **Пользователи каталога** и нажмите кнопку **Добавить**, чтобы добавить пользователей каталога данных. Вы автоматически добавлены в эту группу.

    ![Каталог данных Azure — пользователи](media/data-catalog-get-started/data-catalog-add-catalog-user.png)

8. При выборе ценовой категории *Стандартный* можно развернуть **Администраторы глоссария** и нажать кнопку **Добавить** для добавления пользователей-администраторов глоссария. Вы автоматически добавлены в эту группу.

    ![Администраторы глоссария Каталога данных Azure](media/data-catalog-get-started/data-catalog-standard-glossary-admin.png)

9. Разверните раздел **Администраторы каталога** и нажмите кнопку **Добавить**, чтобы добавить администраторов каталога данных. Вы автоматически добавлены в эту группу.

    ![Каталог данных Azure — администраторы](media/data-catalog-get-started/data-catalog-add-catalog-admins.png)

10. Разверните **Заголовок портала** и добавьте дополнительный текст, который будет отображаться в заголовке на портале.

    ![Каталог данных Azure — заголовок портала](media/data-catalog-get-started/data-catalog-portal-title.png)

11. Заполнив страницу **Параметры**, перейдите к странице **Опубликовать**.

    ![Каталог данных Azure — создан](media/data-catalog-get-started/data-catalog-created.png)

## <a name="find-a-data-catalog-in-the-azure-portal"></a>Поиск каталога данных на портале Azure

1. На отдельной вкладке в браузере или в отдельном окне браузера перейдите [на портал Azure](https://portal.azure.com) и войдите в систему с помощью той же учетной записи, используемой для создания каталога данных на предыдущем этапе.

2. Щелкните **Все службы**, а затем — **Каталог данных**.

    ![Каталог данных Azure — поиск по Azure](media/data-catalog-get-started/data-catalog-browse-azure-portal.png)

    Отобразится созданный каталог данных.

    ![Каталог данных Azure — просмотр каталога в списке](media/data-catalog-get-started/data-catalog-azure-portal-show-catalog.png)

3. Щелкните созданный вами каталог. На портале откроется колонка **Каталог данных** .

   ![Каталог данных Azure — колонка на портале](media/data-catalog-get-started/data-catalog-blade-azure-portal.png)

4. Вы можете просматривать и обновлять свойства каталога данных. Например, щелкните **Ценовая категория** , чтобы изменить выпуск.

    ![Каталог данных Azure — ценовая категория](media/data-catalog-get-started/data-catalog-change-pricing-tier.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать Каталог данных Azure для своей организации. Теперь вы можете зарегистрировать источники данных в каталоге данных.

> [!div class="nextstepaction"]
> [Регистрация источников данных в каталоге данных Azure](data-catalog-how-to-register.md)