---
title: Вопросы управления интересами и устранение неполадок — центр партнеров Майкрософт
description: Общие сведения об общих ошибках и вопросах при настройке потенциальных клиентов Marketplace в центре партнеров
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 10/01/2020
ms.openlocfilehash: 681498073a5f111e3babd50443623589ffeec02f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92330926"
---
# <a name="common-questions-and-troubleshooting-for-lead-configuration"></a>Часто задаваемые вопросы и устранение неполадок в конфигурации интересов

В этой статье содержатся ответы на некоторые распространенные вопросы об управлении интересами коммерческих решений. Кроме того, в нем рассматриваются ошибки, которые могут возникнуть при настройке системы управления отношениями с клиентами (CRM) в центре партнеров.

## <a name="common-questions-about-lead-management"></a>Часто задаваемые вопросы об управлении потенциальными клиентами

#### <a name="where-can-i-get-help-in-setting-up-my-lead-destination"></a>Где я могу получить помощь в настройке назначения потенциального клиента?

Общие сведения о подключении системы CRM к предложениям коммерческого рынка см. в статье [клиентские интересы из коммерческого рынка](partner-center-portal/commercial-marketplace-get-customer-leads.md) . Если возникла ошибка, ознакомьтесь с инструкциями по устранению неполадок ниже. Чтобы получить дополнительную поддержку, отправьте запрос в службу поддержки через [Центр партнеров и службу поддержки](https://aka.ms/marketplacepublishersupport). Потом выберите **Offer creation** > **Your type of offer** > **Lead management configuration** (Создание предложения > Тип предложения > Конфигурация управления потенциальным клиентом).

#### <a name="am-i-required-to-configure-a-lead-destination-in-order-to-publish-an-offer-in-the-commercial-marketplace"></a>Нужно ли мне настроить назначение потенциального клиента, чтобы опубликовать предложение на коммерческой платформе?

Ответ зависит от типа публикуемого предложения. Программное обеспечение как услуга (SaaS) и Dynamics 365 Customer Engagement используют форму **Связаться со мной**, чтобы регистрировать все предложения Dynamics 365 for Finance and Operations, все предложения Dynamics 365 Business Central и все предложения Consulting Service. В итоге, им требуется связь с назначением потенциального клиента. Если тип вашего предложения не был обозначен, связь с назначением потенциального клиента не требуется. Мы рекомендуем сформировать назначение потенциального клиента, чтобы вы не упустили свои бизнес-возможности.

#### <a name="how-can-i-find-the-test-lead"></a>Как мне найти тестового потенциального клиента?

Найдите `"MSFT_TEST"` в назначении потенциального клиента. Ниже приведен пример специалиста по тестированию от Майкрософт. Обратите внимание, что формат руководителя теста зависит от назначения интереса.

```
{
    "userDetails": {
      "FirstName": "MSFT_TEST_636573304831318844",
      "LastName": "MSFT_TEST_636573304831318844",
      "Email": "MSFT_TEST_636573304831318844@test.com",
      "Phone": "1234567890",
      "Country": "US",
      "Company": "MSFT_TEST_636573304831318844",
      "Title": "MSFT_TEST_636573304831318844"
    },
    "LeadSource": "AzureMarketplace",
    "ActionCode": "INS",
    "OfferTitle": "Contoso Test"
    "Description": "MSFT_TEST_636573304831318844"
}
```

#### <a name="i-have-a-live-offer-but-why-am-i-not-seeing-any-leads"></a>У меня есть действующее предложение, но почему я не вижу потенциальных клиентов?

Убедитесь, что ваша связь с назначением потенциального клиента действительна. Мы отправим тестового потенциального клиента после того как вы нажмете кнопку **Опубликовать** на вашем предложении в Центре партнеров. Если вы видите потенциального клиента — связь действительна. Вы можете также протестировать эту связь с потенциальным клиентом пытаясь получить предварительный просмотр предложения на этапе предварительной версии. Выберите **Get It Now** (Получить сейчас), **Contact Me** (Связаться со мной) или **Free Trial** (Бесплатная пробная версия) в списке на странице коммерческой платформы.

Также, убедитесь, что вы ищете правильные данные. Сведения о ведущих данных, отправляемых в цель интереса, см. в разделе Общие сведения о [ведущих данных](partner-center-portal/commercial-marketplace-get-customer-leads.md) .

#### <a name="i-configured-azure-blob-storage-as-my-lead-destination-but-why-dont-i-see-the-lead"></a>У меня настроено хранилище больших двоичных объектов Azure в качестве назначения потенциального клиента, но почему потенциальный клиент не виден?

Хранилище больших двоичных объектов больше не поддерживается в качестве назначения потенциального клиента, поэтому вы теряете всех потенциальных клиентов, сгенерированных вашим предложением. Выберите любой другой [вариант назначения потенциального клиента](partner-center-portal/commercial-marketplace-get-customer-leads.md). 

#### <a name="i-received-an-email-from-the-commercial-marketplace-but-why-cant-i-find-the-lead-in-my-crm"></a>Мне пришло электронное письмо от коммерческой платформы, но почему мне не удается найти потенциального клиента в моей CRM-системе?

Вполне возможно, что домен электронной почты пользователя — из .edu. По соображениям конфиденциальности мы не передаем личную информацию из домена .edu. Подайте запрос в службу поддержки на странице [Справка и поддержка](https://aka.ms/marketplacepublishersupport) Центра партнеров.

#### <a name="i-configured-an-azure-table-as-my-lead-destination-how-can-i-view-the-leads"></a>Таблица Azure настроена в качестве пункта назначения потенциального клиента. Как я могу посмотреть потенциальных клиентов?

Вы можете получить доступ к данным потенциальных клиентов, которые хранятся в таблице Azure на портале Azure. Вы можете бесплатно загрузить и установить [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/) для просмотра таблицы вашей учетной записи службы хранилища Azure.

#### <a name="i-configured-an-azure-table-as-my-lead-destination-can-i-get-notified-whenever-a-new-commercial-marketplace-lead-is-sent"></a>Таблица Azure настроена в качестве пункта назначения потенциального клиента. Могу ли я получать уведомления, когда коммерческой платформой генерируется новый потенциальный клиент?

Да. Следуйте инструкции в статье [Настройка управления потенциальными клиентами с помощью таблицы Azure](partner-center-portal/commercial-marketplace-lead-management-instructions-azure-table.md), чтобы настроить поток Microsoft, который отправляет письмо на электронную почту, если потенциальный клиент был добавлен в таблицу Azure.

#### <a name="i-configured-salesforce-as-my-lead-destination-but-why-cant-i-find-the-leads"></a>Если у меня в качестве назначения потенциальных клиентов настроен Salesforce, то почему я не могу найти потенциальных клиентов?

Проверьте, является ли веб-форма потенциальных клиентов обязательным полем в списке выбора. Если это так, то измените свойства этого поля на необязательное текстовое поле.

#### <a name="there-was-an-issue-with-my-lead-destination-and-i-missed-some-leads-can-i-have-them-sent-to-me-in-an-email"></a>У меня проблема с назначением потенциального клиента, и уже потеряны несколько потенциальных клиентов. Можно получить их по электронной почте?

Ввиду ограничений политики для персональных данных, мы не можем делиться сведениями о потенциальных клиентах, используя незащищенные сообщения электронной почты.

#### <a name="i-configured-an-azure-table-as-my-lead-destination-how-much-will-it-cost"></a>Таблица Azure настроена в качестве пункта назначения потенциального клиента. Сколько это будет стоить?

Генерация потенциальных клиентов не создает много данных. Эта величина меньше 1 ГБ почти для всех издателей. Цена зависит от количества полученных потенциальных клиентов. Например, если за месяц было получено 1000 потенциальных клиентов, цена будет около 50 центов. Дополнительные сведения о ценах см. на странице [Обзор цен на хранилище Azure](https://azure.microsoft.com/pricing/details/storage/).

Если вы не нашли ответа на вопрос, обратитесь в службу поддержки Майкрософт на странице [Справка и поддержка](https://aka.ms/marketplacepublishersupport) Центра партнеров. Потом выберите **Offer creation** > **Your type of offer** > **Lead management configuration** (Создание предложения > Тип предложения > Конфигурация управления потенциальным клиентом).

#### <a name="im-receiving-email-notifications-when-new-customer-leads-are-received-how-can-i-configure-someone-else-to-receive-these-emails"></a>Мне приходят уведомления по электронной почте когда получаю новых потенциальных клиентов. Как можно изменить настройки, чтобы кто-то еще мог получать эти электронные сообщения?

Перейдите к вашему предложению в Центре партнеров и откройте страницу **Настройка предложения** > **Управление интересами** > **Изменить**. Обновите адреса электронной почты под полем **Контактный адрес электронной почты**.

## <a name="troubleshooting-lead-configuration-errors"></a><a id="publishing-config-errors"></a> Устранение ошибок конфигурации интересов

**Не удалось сохранить интерес в Dynamics CRM. Проверьте параметры учетной записи Dynamics CRM. LastCRMError: Unable to Login to Dynamics CRM, LastCRMException:** (Не удалось выполнить вход в Dynamics CRM, LastCRMException:) 

> Если выбрана Microsoft 365 проверка подлинности, проверьте правильность учетной записи пользователя и пароля. Если выбран параметр Azure Active Directory, проверьте, соответствует ли идентификатор клиента, идентификатор приложения и секретный ключ приложения тем, что было настроено на Azure Active Directory. Инструкции см. [здесь](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md). Если имя пользователя/пароль учетной записи является допустимым, убедитесь, что она имеет доступ к Dynamics 365, и ему присвоена лицензия (шаги 11–15, если при использовании пользователя Office используются Azure Active Directory или параметры безопасности). 

**Не удалось сохранить интерес в Dynamics CRM. У пользователя нет разрешений на создание атрибута leadourcecode в объекте интереса** 

> В приложения/пользователя нет роли (ролей) безопасности для автора интересов Microsoft Marketplace. Выполните шаги 11-15, если используется Azure Active Directory, или параметры безопасности при [использовании пользователя Office](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md).

**Не удалось сохранить интерес в Dynamics CRM, используя AAD. Exception:: Tenant not found. Этот экземпляр может произойти, если у клиента отсутствуют активные подписки.**  

> Идентификатор каталога, приведенный в разделе управления интересами, является недопустимым каталогом. Получите идентификатор каталога на основе инструкций на шаге 2 (в разделе Azure Active Directory) [здесь](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md).

**Не удалось сохранить интерес в Dynamics CRM. LastCRMError: SecLib::RetrievePrivilegeForUser failed - no roles are assigned to user.**  

> Способы устранения: назначьте роль безопасности автору интересов Microsoft Marketplace. Следуйте [этим](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md) инструкциям в параметрах безопасности.

**Не удалось сохранить интерес в Dynamics CRM, используя AAD. Exception:: Application with identifier was not found in the directory** 

> Идентификатор приложения, приведенный в разделе управления интересами, является недопустимым каталогом. Получите идентификатор каталога в соответствии с инструкциями в шаге 8 (в Azure Active Directory, [отсюда](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)). 

**Не удалось сохранить интерес в Dynamics CRM, используя AAD. Exception:: Requested tenant identifier is not valid and not valid external domain format** 

> Идентификатор каталога, приведенный в разделе управления интересами, является недопустимым каталогом. Получите идентификатор каталога в соответствии с инструкциями в шаге 2 (в Azure Active Directory, [отсюда](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)). 

**Не удалось сохранить интерес в Dynamics CRM, используя AAD. Exception:: Error validating credentials.: Invalid client secret is provided.** 

> Способы устранения: Войдите на портал Azure и проверьте, соответствует ли ключ приложения значениям в Центре партнеров. Получите сгенерированный пароль в соответствии с инструкциями в шаге 10 (в Azure Active Directory, [отсюда](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)). 

**Не удалось сохранить интерес в Dynamics CRM. LastCRMError: The request channel timed out while waiting for a reply after 00:02:00. Увеличьте значение времени ожидания, которое передается во время вызова запроса, или значение SendTimeout в привязке. Время, назначенное для выполнения этой операции, может быть составной частью более длинного времени ожидания.**  

> Решение. Войдите в центр партнеров, установите флажок Настройка предложения >> Customer Leads >> URL-адрес, проверьте, является ли он допустимым динамическим экземпляром CRM.

