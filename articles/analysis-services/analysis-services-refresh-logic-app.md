---
title: Обновление с помощью Logic Apps для моделей Azure Analysis Services | Документация Майкрософт
description: В этой статье описывается, как выполнить код асинхронного обновления для Azure Analysis Services с помощью Azure Logic Apps.
author: chrislound
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 10/30/2019
ms.author: chlound
ms.custom: references_regions
ms.openlocfilehash: 8a8d434fca7cab4432f38fc64093cf1fe060bd5f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92019092"
---
# <a name="refresh-with-logic-apps"></a>Обновление с помощью Logic Apps

Используя Logic Apps и вызовы RESTFUL, можно выполнять автоматические операции обновления данных в табличных моделях анализа Azure, включая синхронизацию реплик только для чтения для горизонтального масштабирования запросов.

Дополнительные сведения об использовании API-интерфейсов RESTFUL с Azure Analysis Services см. в разделе [асинхронное обновление с помощью REST API](analysis-services-async-refresh.md).

## <a name="authentication"></a>Аутентификация

Все вызовы должны пройти проверку подлинности с помощью допустимого маркера Azure Active Directory (OAuth 2).  В примерах, приведенных в этой статье, для проверки подлинности в Azure Analysis Services используется субъект-служба (SPN). Дополнительные сведения см. в статье [Создание субъекта-службы с помощью портал Azure](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="design-the-logic-app"></a>Проектирование приложения логики

> [!IMPORTANT]
> В следующих примерах предполагается, что брандмауэр Azure Analysis Services отключен. Если брандмауэр включен, общедоступный IP-адрес инициатора запроса должен быть добавлен в список утверждений в брандмауэре Azure Analysis Services. Дополнительные сведения о Azure Logic Apps диапазонах IP-адресов для региона см. в разделе [ограничения и сведения о конфигурации для Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md#configuration).

### <a name="prerequisites"></a>Предварительные условия

#### <a name="create-a-service-principal-spn"></a>Создание субъекта-службы (SPN)

Дополнительные сведения о создании субъекта-службы см. в разделе [Создание субъекта-службы с помощью портал Azure](../active-directory/develop/howto-create-service-principal-portal.md).

#### <a name="configure-permissions-in-azure-analysis-services"></a>Настройка разрешений в Azure Analysis Services
 
Создаваемый субъект-служба должен иметь разрешения администратора сервера на сервере. Дополнительные сведения см. в статье [Добавление субъекта-службы к роли администратора сервера](analysis-services-addservprinc-admins.md).

### <a name="configure-the-logic-app"></a>Настройка приложения логики

В этом примере приложение логики предназначено для активации при получении HTTP-запроса. Это позволит использовать средство оркестрации, такое как фабрика данных Azure, для активации Azure Analysis Services обновления модели.

После создания приложения логики выполните следующие действия.

1. В конструкторе приложений логики выберите первое действие, как **при получении HTTP-запроса**.

   ![Добавить действие "получено HTTP"](./media/analysis-services-async-refresh-logic-app/1.png)

Этот шаг будет заполнен URL-адресом HTTP POST после сохранения приложения логики.

2. Добавьте новый шаг и выполните поиск по запросу **http**.  

   ![Снимок экрана: раздел "Выбор действия" с выбранной плиткой "HTTP".](./media/analysis-services-async-refresh-logic-app/9.png)

   ![Снимок экрана окна "HTTP" с выбранной плиткой "HTTP-HTTP".](./media/analysis-services-async-refresh-logic-app/10.png)

3. Выберите **http** , чтобы добавить это действие.

   ![Добавление действия HTTP](./media/analysis-services-async-refresh-logic-app/2.png)

Настройте действие HTTP следующим образом:

|Свойство.  |Значение  |
|---------|---------|
|**Метод**     |POST         |
|**URI**     | https://*/серверс/* имя *сервера консультантов*/Моделс/*имя базы данных*/рефрешес <br /> <br /> Пример: https: \/ /westus.asazure.Windows.NET/Servers/MyServer/Models/AdventureWorks/refreshes|
|**Заголовки**     |   Content-Type, Application/JSON <br /> <br />  ![Заголовки](./media/analysis-services-async-refresh-logic-app/6.png)    |
|**Текст**     |   Дополнительные сведения о создании текста запроса см. в разделе [асинхронное обновление с помощью REST API-POST/рефрешес](analysis-services-async-refresh.md#post-refreshes). |
|**Аутентификация**     |Active Directory OAuth         |
|**Клиент**     |Заполните Azure Active Directory TenantId         |
|**Аудитория**     |HTTPS://*. asazure. Windows. NET         |
|**Идентификатор клиента**     |Введите имя участника-службы ClientID         |
|**Тип учетных данных**     |Секрет         |
|**Секрет**     |Введите секретное имя субъекта-службы         |

Пример

![Завершенное действие HTTP](./media/analysis-services-async-refresh-logic-app/7.png)

Теперь протестируйте приложение логики.  В конструкторе приложений логики нажмите кнопку **выполнить**.

![Тестирование приложения логики](./media/analysis-services-async-refresh-logic-app/8.png)

## <a name="consume-the-logic-app-with-azure-data-factory"></a>Использование приложения логики с фабрикой данных Azure

После сохранения приложения логики просмотрите, **когда ПОЛУЧЕН HTTP-запрос** , а затем скопируйте созданный **URL HTTP POST** .  Это URL-адрес, который может использоваться фабрикой данных Azure для выполнения асинхронного вызова для активации приложения логики.

Ниже приведен пример веб-действия фабрики данных Azure, которое выполняет это действие.

![Веб-действие фабрики данных](./media/analysis-services-async-refresh-logic-app/11.png)

## <a name="use-a-self-contained-logic-app"></a>Использование автономного приложения логики

Если вы не планируете использовать средство оркестрации, такое как фабрика данных для активации обновления модели, можно настроить приложение логики для запуска обновления по расписанию.

Используя приведенный выше пример, удалите первое действие и замените его действием **расписания** .

![Снимок экрана, на котором показана страница "Logic Apps" с выбранной плиткой "Расписание".](./media/analysis-services-async-refresh-logic-app/12.png)

![Снимок экрана, на котором показана страница "триггеры".](./media/analysis-services-async-refresh-logic-app/13.png)

В этом примере будет использоваться **повторение**.

После добавления действия настройте интервал и частоту, а затем добавьте новый параметр и выберите **эти часы**.

![Снимок экрана, показывающий раздел "повторение" с выбранным параметром "в этом часовом".](./media/analysis-services-async-refresh-logic-app/16.png)

Выберите желаемые часы.

![Действие по расписанию](./media/analysis-services-async-refresh-logic-app/15.png)

Сохраните приложение логики.

## <a name="next-steps"></a>Дальнейшие действия

[Примеры](analysis-services-samples.md)  
[REST API](/rest/api/analysisservices/servers)