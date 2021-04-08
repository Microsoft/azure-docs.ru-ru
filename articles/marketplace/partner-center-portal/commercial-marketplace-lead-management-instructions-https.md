---
title: Управление потенциальными клиентами с помощью конечной точки HTTPS — Microsoft Marketplace для коммерческих клиентов
description: Узнайте, как использовать Power Automate и конечную точку HTTPS для управления потенциальными клиентами из Microsoft AppSource и Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 03/30/2020
ms.openlocfilehash: 5bea2cf256e30bd896957bbee0e0ad824057a569
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98247188"
---
# <a name="use-an-https-endpoint-to-manage-commercial-marketplace-leads"></a>Использование конечной точки HTTPS для управления потенциальными клиентами на Marketplace для коммерческих клиентов

Если система управления отношениями с клиентами (CRM) не поддерживает получение сведений о потенциальных клиентах Microsoft AppSource и Azure Marketplace из Центра партнеров, для обработки этих потенциальных клиентов можно использовать конечную точку HTTPS в [Power Automate](https://powerapps.microsoft.com/automate-processes/). Используя конечную точку HTTPS, сведения о потенциальных клиентах с Marketplace для коммерческих клиентов можно отправлять в качестве уведомления по электронной почте. Кроме того, их можно записать в CRM-систему, поддерживаемую Power Automate.

В этой статье объясняется, как создать новый поток в Power Automate, чтобы генерировать URL-адреса HTTP POST, которые будут использоваться для настройки потенциальных клиентов в Центре партнеров. Здесь же есть пошаговое руководство по тестированию потока с помощью программы [Postman](https://www.getpostman.com/downloads/).

>[!NOTE]
>Для работы с соединителем Power Automate, используемом в этих инструкциях, требуется платная подписка на Power Automate. Обязательно учитывайте это, прежде чем приступать к настройке потока.

## <a name="create-a-flow-by-using-power-automate"></a>Создание потока с использованием Power Automate

1. Откройте веб-страницу [Power Automate](https://flow.microsoft.com/). Выберите **Войти**. Если у вас еще нет учетной записи, выберите **Зарегистрироваться бесплатно**, чтобы создать бесплатную учетную запись Power Automate.

1. Войдите и выберите **Мои потоки** в меню.

    ![Вход в раздел "Мои потоки"](./media/commercial-marketplace-lead-management-instructions-https/my-flows-automated.png)

1. В разделе **+ Создать** выберите пункт **+ Мгновенно — с "нуля"** .

    ![Мои потоки + Automated — с "нуля"](./media/commercial-marketplace-lead-management-instructions-https/https-myflows-create-fromblank.png)

1. Назовите поток, а затем в разделе **Выбор способа активации этого потока** выберите **При получении HTTP-запроса**.

    ![Создание кнопки "Пропустить" в окне автоматизированного потока](./media/commercial-marketplace-lead-management-instructions-https/https-myflows-pick-request-trigger.png)

1. Выберите шаг потока, чтобы развернуть сведения о нем.

    ![Разворачивание шага потока](./media/commercial-marketplace-lead-management-instructions-https/expand-flow-step.png)

1. Настройте **схему текста запроса JSON** одним из следующих способов.

    - Скопируйте схему JSON в текстовое поле **Схема текста запроса JSON**.
    - Выберите **Использовать пример полезной нагрузки, чтобы создать схему**. В текстовое поле **Введите или вставьте пример полезных данных JSON** вставьте пример JSON. Выберите **Готово**, чтобы создать схему.

    **Схема JSON**

    ```JSON
    {
      "$schema": "https://json-schema.org/draft-04/schema#",
      "definitions": {},
      "id": "http://example.com/example.json",
      "properties": {
        "ActionCode": {
          "id": "/properties/ActionCode",
          "type": "string"
        },
        "OfferTitle": {
          "id": "/properties/OfferTitle",
          "type": "string"
        },
        "LeadSource": {
          "id": "/properties/LeadSource",
          "type": "string"
        },
        "Description": {
          "id": "/properties/Description",
          "type": "string"
        },
        "UserDetails": {
          "id": "/properties/UserDetails",
          "properties": {
            "Company": {
              "id": "/properties/UserDetails/properties/Company",
              "type": "string"
            },
            "Country": {
              "id": "/properties/UserDetails/properties/Country",
              "type": "string"
            },
            "Email": {
              "id": "/properties/UserDetails/properties/Email",
              "type": "string"
            },
            "FirstName": {
              "id": "/properties/UserDetails/properties/FirstName",
              "type": "string"
            },
            "LastName": {
              "id": "/properties/UserDetails/properties/LastName",
              "type": "string"
            },
            "Phone": {
              "id": "/properties/UserDetails/properties/Phone",
              "type": "string"
            },
            "Title": {
              "id": "/properties/UserDetails/properties/Title",
              "type": "string"
            }
          },
          "type": "object"
        }
      },
      "type": "object"
    }
    ```

    **Пример JSON**
    
    ```json
    {
      "UserDetails": {
        "FirstName": "Some",
        "LastName": "One",
        "Email": "someone@contoso.com",
        "Phone": "16175555555",
        "Country": "USA",
        "Company": "Contoso",
        "Title": "Esquire"
     },
      "LeadSource": "AzureMarketplace",
      "ActionCode": "INS",
      "OfferTitle": "Test Microsoft",
      "Description": "Test run through Power Automate"
    }
    ```

>[!NOTE]
>На этом этапе конфигурации можно подключиться к системе CRM или настроить уведомление по электронной почте. Следуйте оставшимся инструкциям по своему усмотрению.

### <a name="connect-to-a-crm-system"></a>Подключение к системе CRM

1. Выберите **+ Новый шаг**.
1. Выберите систему CRM по своему усмотрению, выполнив поиск в разделе **Поиск соединителей и действий**. Выберите ее на вкладке **Действия** с действием "Создать новую запись". На следующем снимке экрана в качестве примера показано **Создание новой записи (Dynamics 365)** .

    ![Создание записи](./media/commercial-marketplace-lead-management-instructions-https/create-new-record.png)

1. Укажите **Название организации**, связанное с системой CRM. Выберите **Потенциальные клиенты** в раскрывающемся списке **Имя сущности**.

    ![Выбор потенциальных клиентов](./media/commercial-marketplace-lead-management-instructions-https/select-leads.png)

1. В Power Automate отобразится форма для предоставления информации о потенциальном клиенте. Вы можете сопоставить элементы из запроса ввода, выбрав динамическое содержимое. На приведенном ниже снимке экрана показан пример **OfferTitle**.

    ![Добавление динамического содержимого](./media/commercial-marketplace-lead-management-instructions-https/add-dynamic-content.png)

1. Сопоставьте нужные поля и сохраните поток, нажав **Сохранить**. URL-адрес HTTP POST создается и доступен в окне **При получении HTTP-запроса**. Скопируйте этот URL-адрес с помощью инструмента копирования, расположенного справа от URL-адреса HTTP POST. Важно воспользоваться инструментом копирования, чтобы не пропустить никакую часть URL-адреса. Сохраните этот URL-адрес, так как он понадобится при настройке управления потенциальными клиентами на портале публикации.

    ![при получении HTTP-запроса;](./media/commercial-marketplace-lead-management-instructions-https/when-http-request-received.png)

### <a name="set-up-email-notification"></a>Настройка уведомлений по электронной почте

1. Теперь, когда работа со схемой JSON завершена, выберите **+ Новый шаг**.
1. В разделе **Выберите действие** выберите **Действия**.
1. На вкладке **Действия** выберите **Отправить электронное письмо (Office 365 Outlook)** .

    >[!NOTE]
    >Если вы хотите использовать другого поставщика услуг электронной почты, выполните поиск и выберите **Отправить уведомление по электронной почте (почта)** в качестве действия.

    ![Добавление действия электронной почты](./media/commercial-marketplace-lead-management-instructions-https/https-request-received-send-email.png)

1. В окне **Отправить электронное письмо** настройте следующие обязательные поля:

   - **Кому**. Введите по крайней мере один допустимый адрес электронной почты, по которому будут отправляться данные о потенциальных клиентах.
   - **Субъект**: Power Automate дает возможность добавить динамическое содержимое, например **LeadSource**, как показано на экране ниже. Начните с ввода имени поля. Затем выберите список выбора динамического содержимого во всплывающем окне. 

        >[!NOTE] 
        > При добавлении имен полей можно ставить двоеточие (:) после каждого из них, а затем нажать клавишу **Ввод**, чтобы создать новую строку. После добавления имен полей можно добавить каждый связанный параметр из списка динамического выбора.

        ![Добавление действия электронной почты с помощью динамического содержимого](./media/commercial-marketplace-lead-management-instructions-https/add-email-using-dynamic-content.png)

   - **Текст**: добавьте нужные сведения в текст электронного письма из списка динамического содержимого. Например, фамилия, имя, адрес электронной почты и название компании. После настройки уведомление по электронной почте будет выглядеть как в примере на приведенном ниже экране.


       ![Пример уведомления по электронной почте](./media/commercial-marketplace-lead-management-instructions-https/send-an-email.png)

1. Нажмите кнопку **Сохранить**, чтобы завершить поток. URL-адрес HTTP POST создается и доступен в окне **При получении HTTP-запроса**. Скопируйте этот URL-адрес с помощью инструмента копирования, расположенного справа от URL-адреса HTTP POST. Важно воспользоваться этим инструментом, чтобы не пропустить никакую часть URL-адреса. Сохраните этот URL-адрес, так как он понадобится при настройке управления потенциальными клиентами на портале публикации.

   ![URL-адрес HTTP POST](./media/commercial-marketplace-lead-management-instructions-https/http-post-url.png)

### <a name="testing"></a>Тестирование

Вы можете проверить конфигурацию с помощью программы [Postman](https://app.getpostman.com/app/download/win64). Для Windows доступна загрузка Postman из Интернета. 

1. Запустите Postman и выберите **Создать** > **запрос**, чтобы настроить инструмент тестирования. 

   ![Запрос настройки инструмента тестирования](./media/commercial-marketplace-lead-management-instructions-https/postman-request.png)

1. Заполните форму **Сохранение запроса**, а затем сохраните его в созданную папку.

   ![Форма сохранения запроса](./media/commercial-marketplace-lead-management-instructions-https/postman-save-to-test.png)

1. Выберите **POST** в раскрывающемся списке. 

   ![Тестирование потока](./media/commercial-marketplace-lead-management-instructions-https/test-my-flow.png)

1. Вставьте URL-адрес HTTP POST из потока, созданного в Power Automate, в поле **Введите URL-адрес запроса**.

   ![Вставьте URL-адрес HTTP POST](./media/commercial-marketplace-lead-management-instructions-https/paste-http-post-url.png)

1. Вернитесь в [Power Automate](https://flow.microsoft.com/). Найдите созданный поток для отправки потенциальных клиентов, перейдя в раздел **Мои потоки** из строки меню Power Automate. Нажмите кнопку с многоточием рядом с именем потока, чтобы просмотреть дополнительные параметры, и выберите **Изменить**.


1. Выберите **Тест** в правом верхнем углу, щелкните **Я выполню действие-триггер** и нажмите **Тест**. В верхней части экрана появится индикатор, указывающий, что тест начался.

   ![Параметр "Я выполню действие-триггер"](./media/commercial-marketplace-lead-management-instructions-https/test-flow-trigger-action.png)

1. Вернитесь в приложение Postman и выберите **Отправить**.

   ![Кнопка "Отправить"](./media/commercial-marketplace-lead-management-instructions-https/postman-send.png)

1. Вернитесь в поток и проверьте результат. Если все работает правильно, вы увидите сообщение о том, что поток выполнен успешно.

   ![Проверка результатов](./media/commercial-marketplace-lead-management-instructions-https/my-flow-check-results.png)

1. Кроме того, вы должны были получить сообщение электронной почты. Проверьте почту. 

    >[!NOTE] 
    >Если электронное письмо с тестом отсутствует, проверьте папки со спамом и нежелательной почтой. На следующем экране вы заметите только метки полей, добавленные при настройке уведомления по электронной почте. Если бы речь шла о фактическом потенциальном клиенте, появившемся в ответ на ваше предложение, вы бы увидели реальную информацию от контактного лица потенциального клиента в основной части письма и в строке "Тема".

   ![Электронное сообщение получено.](./media/commercial-marketplace-lead-management-instructions-https/email-received.png)

## <a name="configure-your-offer-to-send-leads-to-the-https-endpoint"></a>Настройка предложения для отправки списка потенциальных клиентов в конечную точку HTTPS

После того как все будет готово к настройке сведений об управлении потенциальными клиентами для вашего предложения на портале публикации, выполните следующие действия.

1. Войдите в [Центр партнеров](https://partner.microsoft.com/dashboard/home).

1. Выберите свое предложение и перейдите на вкладку **Настройка предложения**.

1. В разделе **Потенциальные клиенты** выберите **Подключить**.

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-https/customer-leads.png" alt-text="Потенциальные клиенты":::

1. Во всплывающем окне со **сведениями о подключении** в качестве **назначения потенциального клиента** выберите **Конечная точка HTTPS**. Вставьте URL-адрес HTTP POST из потока, созданного в пошаговом руководстве ранее, в поле **URL-адрес конечной точки HTTPS**.
    ![Сведения о подключении. Контактный адрес электронной почты](./media/commercial-marketplace-lead-management-instructions-https/https-connection-details.png)

1. В разделе **Контактный адрес электронной почты** введите адреса электронной почты сотрудников организации, которые должны получать уведомления по электронной почте при обнаружении нового потенциального клиента. Можно указать несколько адресов электронной почты, разделяя их точкой с запятой.

1. Щелкните **ОК**.

Чтобы убедиться в том, что вы успешно подключились к месту назначения потенциальных клиентов, нажмите кнопку **Проверить**. При успешном подключении вы получите тестового потенциального клиента в месте назначения потенциальных клиентов.

>[!NOTE] 
>Прежде чем вы сможете получать сведения о потенциальных клиентах для своего предложения необходимо завершить настройку его остальной части и опубликовать его.

При создании потенциальных клиентов Microsoft отправляет их в поток. Потенциальные клиенты направляются в CRM-систему, либо уведомление о них отправляется по указанному адресу электронной почты.
