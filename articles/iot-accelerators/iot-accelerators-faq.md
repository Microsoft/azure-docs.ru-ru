---
title: Часто задаваемые вопросы об акселераторах решений для Интернета вещей Azure | Документация Майкрософт
description: В этой статье содержатся ответы на часто задаваемые вопросы по акселераторам решений IoT. Он содержит ссылки на репозитории GitHub.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 02/15/2018
ms.author: dobett
ms.openlocfilehash: 1fd2b8461bd66c826dc4890c331b740c4703f896
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96903995"
---
# <a name="frequently-asked-questions-for-iot-solution-accelerators"></a>Часто задаваемые вопросы об акселераторах решений для Интернета вещей

См. также [часто задаваемые вопросы о подключенной фабрике](iot-accelerators-faq-cf.md).

### <a name="where-can-i-find-the-source-code-for-the-solution-accelerators"></a>Где можно найти исходный код акселераторов решений?

Исходный код хранится в следующих репозиториях GitHub:

* [акселератор решения "Подключенная фабрика"](https://github.com/Azure/azure-iot-connected-factory).
* [Акселератор решений для моделирования устройств](https://github.com/Azure/device-simulation-dotnet)

### <a name="where-can-i-find-the-remote-monitoring-and-predictive-maintenance-solution-accelerators"></a>Где найти Акселераторы решений для удаленного мониторинга и прогнозного обслуживания?

По состоянию на 10 декабря 2020 г. ускорители удаленного мониторинга и прогнозного обслуживания были удалены с сайта [акселераторов решений Azure IOT](https://www.azureiotsolutions.com/Accelerators) и больше не доступны для новых развертываний. Репозитории GitHub для обоих сочетаний клавиш были архивированы. Код по-прежнему доступен всем, но репозитории не занимают новых материалов.

### <a name="what-happens-to-my-existing-remote-monitoring-and-predictive-maintenance-deployments"></a>Что происходит с существующими развертываниями для удаленного мониторинга и прогнозного обслуживания?

На существующие развертывания не влияет удаление ускорителей решений для удаленного мониторинга и прогнозного обслуживания. они будут продолжать работать. Разветвленные репозитории также не затрагиваются. Главные репозитории на GitHub были архивированы.

### <a name="how-do-i-deploy-device-simulation-solution-accelerator"></a>Разделы справки развернуть акселератор решений для моделирования устройств?

Сведения о развертывании акселератора решений для моделирования устройств см. в репозитории [симуляторов устройств](https://github.com/Azure/device-simulation-dotnet/blob/master/README.md) .

### <a name="where-can-i-find-information-about-the-removed-solution-accelerators"></a>Где можно найти сведения об удаленных ускорителях решений?

См. следующие страницы на сайте предыдущих версий:

* [удаленный мониторинг;](/previous-versions/azure/iot-accelerators/about-iot-accelerators)
* [Прогнозное обслуживание](/previous-versions/azure/iot-accelerators/about-iot-accelerators)

### <a name="what-sdks-can-i-use-to-develop-device-clients-for-the-solution-accelerators"></a>Какие пакеты SDK можно использовать для разработки клиентов устройств для акселераторов решений?

Ссылки на пакеты SDK для устройств Интернета вещей для других языков (C, .NET, Java, Node.js, Python) можно найти в репозиториях GitHub для [пакетов SDK Microsoft Azure для Интернета вещей](https://github.com/Azure/azure-iot-sdks).

Если вы используете устройство DevKit, ресурсы и примеры можно найти в репозитории GitHub для [пакета SDK для DevKit IoT](https://github.com/Microsoft/devkit-sdk).

### <a name="im-a-service-administrator-and-id-like-to-change-the-directory-mapping-between-my-subscription-and-a-specific-azure-ad-tenant-how-do-i-complete-this-task"></a>Я являюсь администратором службы и хочу изменить сопоставление каталогов подписки и конкретного клиента Azure AD. Как выполнить эту задачу?

Ознакомьтесь с разделом [Добавление существующей подписки в каталог Azure AD](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md#to-associate-an-existing-subscription-to-your-azure-ad-directory)

### <a name="i-want-to-change-a-service-administrator-or-co-administrator-when-logged-in-with-an-organizational-account"></a>Я хочу сменить администратора службы и соадминистратора при входе в систему с использованием рабочей учетной записи.

См. справочную статью [Смена администратора службы и соадминистратора при входе в систему с использованием учетной записи организации](https://azure.microsoft.com/support/changing-service-admin-and-co-admin).

### <a name="why-am-i-seeing-this-error-your-account-does-not-have-the-proper-permissions-to-create-a-solution-please-check-with-your-account-administrator-or-try-with-a-different-account"></a>Почему появляется ошибка «У вашей учетной записи нет необходимых разрешений для создания решения. Обратитесь к администратору учетной записи или попробуйте использовать другую учетную запись»

Руководствуйтесь следующей схемой:

![Блок-схема разрешений](media/iot-accelerators-faq/flowchart.png)

> [!NOTE]
> Вы выполнили проверку, которая подтверждает ваши права глобального администратора на клиенте Azure AD и соадминистратора подписки, но ошибка не устранена. В таком случае администратор учетной записи должен удалить этого пользователя и повторно назначить необходимые разрешения в указанном порядке. Сначала добавить пользователя в качестве глобального администратора, а затем — пользователя в качестве соадминистратора подписки Azure. Если проблемы не удается устранить, обратитесь в службу [справки и поддержки](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="why-am-i-seeing-this-error-when-i-have-an-azure-subscription-an-azure-subscription-is-required-to-create-pre-configured-solutions-you-can-create-a-free-trial-account-in-just-a-couple-of-minutes"></a>Почему эта ошибка появляется при наличии подписки Azure? "Для создания предварительно настроенных решений требуется подписка Azure. Вы можете создать бесплатную пробную учетную запись всего за несколько минут".

Если точно известно, что у вас есть подписка Azure, проверьте сопоставление клиентов для своей подписки и убедитесь, что в раскрывающемся списке выбран правильный клиент. Если клиент выбран правильно, следуйте приведенной выше схеме и проверьте сопоставление подписки и этого клиента Azure AD.

### <a name="whats-the-difference-between-deleting-a-resource-group-in-the-azure-portal-and-clicking-delete-on-a-solution-accelerator-in-azureiotsolutionscom"></a>В чем разница между удалением группы ресурсов в портал Azure и нажатием кнопки Удалить в Solution Accelerator в azureiotsolutions.com?

* Если удалить Solution Accelerator в [azureiotsolutions.com](https://www.azureiotsolutions.com/), удаляются все ресурсы, развернутые при создании акселератора решений. Если вы добавляли в группу ресурсов дополнительные ресурсы, они также будут удалены.
* Если удалить группу ресурсов на [портале Azure](https://portal.azure.com), будут удалены ресурсы в этой группе. Вам также придется удалить приложение Azure Active Directory, связанное с акселератором решений.

### <a name="can-i-continue-to-leverage-my-existing-investments-in-azure-iot-solution-accelerators"></a>Можно ли и дальше использовать существующие системы на базе акселераторов решений для Интернета вещей Azure?

Да. Любое имеющееся решение продолжает работать в подписке Azure, а исходный код остается доступным на GitHub.

### <a name="how-many-iot-hub-instances-can-i-provision-in-a-subscription"></a>Сколько экземпляров центров Интернета вещей можно подготовить в рамках одной подписки?

По умолчанию [в рамках одной подписки можно подготовить 10 экземпляров Центра Интернета вещей](../azure-resource-manager/management/azure-subscription-service-limits.md#iot-hub-limits). Вы можете отправить [запрос в службу поддержки Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade), чтобы увеличить этот лимит. Так как каждый акселератор решений подготавливает новый Центр Интернета вещей, в рамках одной подписки можно подготовить не более 10 акселераторов решений.

### <a name="how-many-azure-cosmos-db-instances-can-i-provision-in-a-subscription"></a>Сколько экземпляров Azure Cosmos DB можно подготовить в рамках одной подписки?

Пятьдесят. Вы можете отправить [запрос в службу поддержки Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade), чтобы увеличить это количество, но по умолчанию в рамках одной подписки можно подготовить только 50 экземпляров Cosmos DB.

### <a name="can-i-create-a-solution-accelerator-if-i-have-microsoft-azure-for-dreamspark"></a>Можно ли создать акселератор решений, если я использую Microsoft Azure для DreamSpark?

> [!NOTE]
> Microsoft Azure для DreamSpark теперь называется Microsoft Imagine для учащихся.

Сейчас вы не можете создать акселератор решений с использованием учетной записи [Microsoft Azure для DreamSpark](https://azure.microsoft.com/pricing/member-offers/imagine/). Но вы можете всего за пару минут создать [бесплатную пробную учетную запись Azure](https://azure.microsoft.com/free/), которая позволит создать акселератор решений.

### <a name="how-do-i-delete-an-azure-ad-tenant"></a>Как удалить клиент Azure AD?

Ознакомьтесь с записью блога Эрика Голпа (Eric Golpe) [Walkthrough of Deleting an Azure AD Tenant](/archive/blogs/ericgolpe/walkthrough-of-deleting-an-azure-ad-tenant) (Пошаговое руководство по удалению клиента Azure AD).

### <a name="next-steps"></a>Дальнейшие действия

Вы также можете ознакомиться с другими функциями и возможностями акселераторов решений для Интернета вещей:

* [Развертывание акселератора решения "Подключенная фабрика"](quickstart-connected-factory-deploy.md)
* [Все аспекты безопасности Интернета вещей](../iot-fundamentals/iot-security-ground-up.md)