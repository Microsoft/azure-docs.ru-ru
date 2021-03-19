---
title: Настройка уведомлений и шаблонов сообщений электронной почты
titleSuffix: Azure API Management
description: Сведения о настройке уведомлений и шаблонов писем в службе управления API Azure
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/10/2020
ms.author: apimpm
ms.openlocfilehash: 786a9e26003a7afb98307e0bd7fae94c42a2f00d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84690308"
---
# <a name="how-to-configure-notifications-and-email-templates-in-azure-api-management"></a>Как настраивать уведомления и почтовые шаблоны в Azure API Management

Служба управления API позволяет настраивать уведомления для определенных событий, а также почтовые шаблоны, которые используются для связи с администраторами и разработчиками экземпляра службы управления API. В этой статье показано, как настраивать уведомления и почтовые шаблоны для доступных событий.

## <a name="prerequisites"></a>Предварительные требования

Если у вас нет экземпляра службы управления API, выполните инструкции из краткого руководства [Создание экземпляра службы управления API Azure](get-started-create-service-instance.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="configure-notifications"></a><a name="publisher-notifications"> </a>Настройка уведомлений

1.  Выберите экземпляр **управления API** .
2.  Нажмите кнопку **Уведомления**, чтобы просмотреть доступные уведомления.

    ![Уведомления издателя][api-management-publisher-notifications]

    Следующий список событий можно настроить для уведомлений.

    -   **Запросы на подписку (требуется утверждение)** - Указанные получатели электронной почты и пользователи будут получать почтовые уведомления о запросах подписки на продукты API, для которых требуются утверждение.
    -   **Новые подписки** - Указанные получатели электронной почты и пользователи будут получать почтовые уведомления о новых подписках на продукты API.
    -   **Запросы коллекции приложений** - Указанные получатели электронной почты и пользователи будут получать почтовые уведомления при поступлении новых заявлений в коллекцию приложений.
    -   **BCC** - Указанные получатели электронной почты и пользователи будут получать слепые копии всех сообщений электронной почты, отправленных разработчикам.
    -   **Новая проблема или комментарий** - Указанные получатели электронной почты и пользователи будут получать почтовые уведомления при поступлении сведений о новой проблеме или комментария на портал разработчика.
    -   **Сообщение о закрытии учетной записи** - Указанные получатели электронной почты и пользователи будут получать почтовые уведомления при закрытии учетной записи.
    -   **Приближается предельная квота подписок** - Указанные получатели электронной почты и пользователи будут получать почтовые уведомления, когда использование подписок приблизится к квоте использования.

        > [!NOTE]
        > Уведомления активируются только политикой [подписки](api-management-access-restriction-policies.md#SetUsageQuota) . Политика [квоты по ключу](api-management-access-restriction-policies.md#SetUsageQuotaByKey) не создает уведомления.

    Для каждого события можно указать получателей электронной почты с помощью текстового поля адреса электронной почты или можно выбрать пользователей в списке.

3.  Для указания адресов электронной почты, которые должны будут получать уведомления, введите их в текстовом поле адресов электронной почты. При наличии нескольких адресов электронной почты разделяйте их с помощью запятых.

    ![Получатели уведомлений][api-management-email-addresses]

4.  Нажмите кнопку **Добавить**.

## <a name="configure-notification-templates"></a><a name="email-templates"> </a>Настройка шаблонов уведомлений

Служба управления API предоставляет шаблоны уведомлений для сообщений электронной почты, которые отправляются во время администрирования и использования службы. Существуют следующие почтовые шаблоны.

-   Отправка в коллекцию приложений утверждена
-   Прощальное письмо разработчика
-   Уведомление о приближении предельной квоты разработчика
-   Приглашение пользователя
-   Добавлен новый комментарий по проблеме
-   Получена новая проблема
-   Активирована новая подписка
-   Подтверждение обновления подписки
-   Отклонение запроса подписки
-   Получен запрос подписки

При необходимости, эти шаблоны можно изменять.

Чтобы просмотреть и настроить шаблоны сообщений электронной почты для вашего экземпляра службы управления API, щелкните **Шаблоны уведомлений**.

![Шаблоны для электронной почты][api-management-email-templates]

Каждый почтовый шаблон имеет тему в формате обычного текста и определение тела в формате HTML. При необходимости, можно настраивать каждый элемент.

![Редактор почтовых шаблонов][api-management-email-template]

Список **Параметры** содержит список параметров, который в случае вставки в тему или тело будет заменен на заданное значение при отправке сообщения электронной почты. Для вставки параметра поместите курсор в то место, куда необходимо вставить параметр, и щелкните стрелку слева от имени параметра.

> [!NOTE]
> При предварительном просмотре или отправке тестового сообщения параметры не заменяются фактическими значениями.

Чтобы сохранить изменения в почтовом шаблоне, щелкните **Сохранить**. Для отмены изменений щелкните **Отклонить**.

[api-management-management-console]: ./media/api-management-howto-configure-notifications/api-management-management-console.png
[api-management-publisher-notifications]: ./media/api-management-howto-configure-notifications/api-management-publisher-notifications.png
[api-management-email-addresses]: ./media/api-management-howto-configure-notifications/api-management-email-addresses.png
[api-management-email-templates]: ./media/api-management-howto-configure-notifications/api-management-email-templates.png
[api-management-email-templates-list]: ./media/api-management-howto-configure-notifications/api-management-email-templates-list.png
[api-management-email-template]: ./media/api-management-howto-configure-notifications/api-management-email-template.png
[configure publisher notifications]: #publisher-notifications
[configure email templates]: #email-templates
[how to create and use groups]: api-management-howto-create-groups.md
[how to associate groups with developers]: api-management-howto-create-groups.md#associate-group-developer
[get started with azure api management]: get-started-create-service-instance.md
[create an api management service instance]: get-started-create-service-instance.md
