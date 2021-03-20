---
title: Управление учетными записями разработчиков с помощью групп в службе управления API Azure
titleSuffix: Azure API Management
description: Узнайте, как управлять учетными записями разработчика с помощью групп в службе управления API Azure. Создайте группы, а затем свяжите их с продуктами или разработчиками.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 02/13/2018
ms.author: apimpm
ms.openlocfilehash: ea674981036b4be292329a4b30b43180ed26d642
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92092789"
---
# <a name="how-to-create-and-use-groups-to-manage-developer-accounts-in-azure-api-management"></a>Как создавать и использовать группы для управления учетными записями разработчика в службе управления Azure API

В службе управления API группы используются для управления видимостью продуктов для разработчиков. Продукты сначала делаются видимыми для групп, а затем разработчики в таких группах могут просматривать и подписываться на продукты, которые связаны с группами. 

Служба управления API включает несколько неизменяемых системных групп.

* **Администраторы** — членами этой группы являются администраторы подписок Azure. Администраторы управляют экземплярами службы управления API, созданием API, операциями и продуктами, которые используются разработчиками.
* **Разработчики** — в эту группу входят пользователи, прошедшие проверку подлинности на портале разработчиков. Разработчики — это клиенты, которые создают приложения с использованием API. Разработчикам предоставляется доступ к порталу разработчика, и они могут создавать приложения, вызывающие операции этого API.
* **Гости** — пользователи портала разработчиков, не прошедшие проверку подлинности; например, в эту группу попадают возможные клиенты, посетившие портал разработчиков экземпляра управления API. Им может быть предоставлен доступ только для чтения, позволяющий просматривать API, но не вызывать их.

Помимо системных групп администраторы могут создавать пользовательские группы или [использовать внешние группы в связанных с ними клиентах Azure Active Directory][leverage external groups in associated Azure Active Directory tenants]. Чтобы обеспечить разработчикам видимость и предоставить доступ к продуктам API, пользовательские и внешние группы можно использовать вместе с системными группами. Например, можно создать одну пользовательскую группу для разработчиков, связанных с конкретной партнерской организацией, и предоставить им доступ к интерфейсам API в продукте, содержащем только соответствующие интерфейсы API. Пользователь может входить в несколько групп.

В этом руководстве показано, как администраторы экземпляра службы управления API могут добавлять новые группы и связывать их с продуктами и разработчиками.

Помимо создания групп и управления ими на портале издателя, это можно делать с помощью сущности [Group](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity) REST API интерфейса API управления.

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="prerequisites"></a>Предварительные условия

Выполните задачи из статьи [Создание экземпляра службы управления API Azure](get-started-create-service-instance.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-a-group"></a><a name="create-group"> </a>Создание группы

В этом разделе показано, как добавить новую группу к учетной записи службы управления API.

1. В левой части экрана выберите вкладку **Группы**.
2. Щелкните **+ Добавить**.
3. Введите уникальное имя для группы и описание (необязательно).
4. Нажмите кнопку **Создать**.

    ![Добавление новой группы](./media/api-management-howto-create-groups/groups001.png)

После создания группа будет добавлена в список **Группы**. <br/>Чтобы изменить **имя** или **описание** группы, щелкните имя группы, а затем — **Параметры**.<br/>Чтобы удалить группу, щелкните имя группы и нажмите кнопку **Удалить**.

Теперь, когда группа создана, ее можно связать с продуктами и разработчиками.

## <a name="associate-a-group-with-a-product"></a><a name="associate-group-product"> </a>Связывание группы с продуктом

1. Слева выберите вкладку **Продукты**.
2. Щелкните имя нужного продукта.
3. Щелкните **Контроль доступа**.
4. Щелкните **+ Добавить группу**.

    ![Снимок экрана, на котором выделяется кнопка "добавить группу".](./media/api-management-howto-create-groups/groups002.png)
5. Выберите группу, которую нужно добавить.

    ![Снимок экрана, на котором показана выбранная группа, и выделяется кнопка выбора.](./media/api-management-howto-create-groups/groups003.png)

    Чтобы удалить группу из продукта, нажмите кнопку **Удалить**.

    ![Удаление группы](./media/api-management-howto-create-groups/groups004.png)

После установления связи между продуктом и группой разработчики в данной группе могут просматривать продукт и подписаться на него.

> [!NOTE]
> Дополнительную информацию о добавлении групп Azure Active Directory см. в статье [Авторизация учетных записей разработчиков с помощью Azure Active Directory в управлении API Azure](api-management-howto-aad.md).

## <a name="associate-groups-with-developers"></a><a name="associate-group-developer"> </a>Связывание групп с разработчиками

В этом разделе показано, как связать группы с их элементами.

1. В левой части экрана выберите вкладку **Группы**.
2. Выберите пункт **Участники**.

    ![Добавление элементов](./media/api-management-howto-create-groups/groups005.png)
3. Нажмите кнопку **+ Добавить** и выберите элемент.

    ![Снимок экрана, посвященный кнопке "Добавить", выбранному пользователю и кнопке "выбрать".](./media/api-management-howto-create-groups/groups006.png)
4. Нажмите кнопку **Выбрать**.

После добавления связи между разработчиком и группой ее можно просмотреть на вкладке **Пользователи** .

## <a name="next-steps"></a><a name="next-steps"> </a>Дальнейшие действия

* После добавления  в группу разработчик может просматривать связанные с данной группой продукты и подписываться на них. Дополнительные сведения см. в статье [Создание и публикация продукта в службе управления API Azure][How create and publish a product in Azure API Management].
* Помимо создания групп и управления ими на портале издателя, это можно делать с помощью сущности [Group](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity) REST API интерфейса API управления.

[Create a group]: #create-group
[Associate a group with a product]: #associate-group-product
[Associate groups with developers]: #associate-group-developer
[Next steps]: #next-steps

[How create and publish a product in Azure API Management]: api-management-howto-add-products.md

[Get started with Azure API Management]: get-started-create-service-instance.md
[Create an API Management service instance]: get-started-create-service-instance.md
[leverage external groups in associated Azure Active Directory tenants]: api-management-howto-aad.md
