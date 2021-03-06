---
title: Подключение к базе данных Oracle
description: Вставляйте записи и управляйте ими с помощью интерфейсов REST API Oracle Database и Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 05/20/2020
tags: connectors
ms.openlocfilehash: 91873a2d6a498712773bfe721653e64c3364666f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92674809"
---
# <a name="get-started-with-the-oracle-database-connector"></a>Начало работы с соединителем базы данных Oracle

Используя соединитель базы данных Oracle, вы можете создавать рабочие процессы, использующие данные из существующей базы данных. Соединитель позволяет подключаться к базе данных Oracle, размещенной локально или на виртуальной машине Azure. С помощью соединителя вы можете:

* создавать рабочие процессы путем добавления нового клиента в базу данных клиентов или обновления заказа в базе данных заказов;
* использовать действия для получения строки данных, вставки новой строки и даже ее удаления. Например, создание записи в Dynamics CRM Online (триггер) может приводить к добавлению строки в базу данных Oracle (действие). 

Этот соединитель не поддерживает следующие элементы:

* Представления 
* таблицы с составными ключами;
* типы вложенных объектов в таблицах.
* Функции базы данных с нескалярными значениями

В этой статье показано, как использовать соединитель базы данных Oracle в приложении логики.

## <a name="prerequisites"></a>Предварительные требования

* Поддерживаемые версии Oracle: 
    * Oracle 9 и более поздних версий;
    * клиентское программное обеспечение Oracle 8.1.7 и более поздних версий.

* Установите локальный шлюз данных. Этот процесс описан в статье [Подключение к локальным данным из приложений логики](../logic-apps/logic-apps-gateway-connection.md). Шлюз является обязательным компонентом для подключения к локальной базе данных Oracle или виртуальной машине Azure, на которой установлена база данных Oracle. 

    > [!NOTE]
    > Локальный шлюз данных используется как мост для передачи данных между приложением логики и локальными (не расположенными в облаке) источниками. Один шлюз может использоваться с несколькими службами и источниками данных.  Поэтому, как правило, шлюз устанавливается один раз.

* Установите клиент Oracle на компьютере, где установлен локальный шлюз данных.  Не забудьте установить 64-разрядный поставщик данных Oracle для .NET, который предоставляется Oracle:  

  [ODAC 12c, 64-разрядная версия 4 (12.1.0.2.4) для Windows x64](https://www.oracle.com/technetwork/database/windows/downloads/index-090165.html)

    > [!TIP]
    > Если клиент Oracle не установлен, при попытке создать или использовать соединение будет возникать ошибка. Описание ошибок приводится в соответствующем разделе этой статьи.


## <a name="add-the-connector"></a>Добавление соединителя

> [!IMPORTANT]
> Этот соединитель не содержит триггеров. В нем есть только действия. Поэтому при создании приложения логики следует добавить отдельный триггер для запуска приложения логики, например **Расписание — Периодичность** или **Запрос/ответ — Ответ**. 

1. Создайте пустое приложение логики на [портале Azure](https://portal.azure.com).

2. Для запуска приложения логики выберите триггер **Запрос/ответ — Запрос**: 

    ![Диалоговое окно содержит поле для поиска всех триггеров. Кроме того, отображается один триггер "запрос/ответ-запрос" с кнопкой выбора.](./media/connectors-create-api-oracledatabase/request-trigger.png)

3. Щелкните **Сохранить**. При сохранении данных автоматически создается URL-адрес запроса. 

4. Выберите **Новый шаг**, а затем — **Добавить действие**. Введите `oracle`, чтобы увидеть список доступных действий: 

    ![Поле поиска содержит "Oracle". При поиске будет нажата одна метка "Oracle Database". Имеется страница с вкладками, одна вкладка, в которой отображается "TRIGGERs (0)", другая — "действия (6)". Перечислены шесть действий. Первый из них — "получить предварительную версию строки".](./media/connectors-create-api-oracledatabase/oracledb-actions.png)

    > [!TIP]
    > Это самый быстрый способ просмотреть триггеры и действия, доступные для определенного соединителя. Введите часть имени соединителя, например `oracle`. Конструктор выведет полный список триггеров и действий. 

5. Выберите одно из этих действий, например **База данных Oracle — Получение строки**. Установите флажок **Connect via on-premises data gateway** (Подключение через локальный шлюз данных). Введите имя сервера Oracle, метод аутентификации, имя пользователя и пароль, а также выберите шлюз:

    ![Диалоговое окно называется "Oracle Database" получить строку ". Имеется флажок, помеченный как "подключение через локальный шлюз данных". Ниже представлены пять других текстовых полей.](./media/connectors-create-api-oracledatabase/create-oracle-connection.png)

6. Подключившись, выберите таблицу из списка и введите идентификатор нужной таблицы. Этот идентификатор таблицы у вас уже должен быть. Если его нет, обратитесь к администратору базы данных Oracle и получите выходные данные инструкции `select * from yourTableName`. В них содержится информация, которую нужно использовать на этом шаге.

    Следующий пример возвращает данные о задании из базы данных "Human Resources": 

    ![Диалоговое окно "получить строку (Предварительная версия)" имеет два текстовых поля: "имя таблицы", которое содержит "H R", и содержит раскрывающийся список, а "строка i d", которая содержит "S A _ представитель".](./media/connectors-create-api-oracledatabase/table-rowid.png)

7. На этом шаге в создаваемый рабочий процесс можно добавить любой другой соединитель. Если вы хотите проверить получение данных из Oracle, отправьте себе сообщение электронной почты с данными Oracle с помощью одного из соединителей отправки электронной почты, например Office 365 Outlook. Используйте динамические маркеры из таблицы Oracle для формирования полей `Subject` и `Body` в сообщении электронной почты:

    ![Существует два диалоговых окна. Поле "отправить сообщение электронной почты" содержит поля для указания адреса электронной почты "текст", "Тема" и "Кому". Диалоговое окно "Добавление динамического содержимого" обеспечивает поиск динамического содержимого из приложений и служб потока.](./media/connectors-create-api-oracledatabase/oracle-send-email.png)

8. **Сохраните** приложение логики, а затем выберите **Выполнить**. Закройте конструктор и проверьте состояние приложения в журнале запусков. Если вы увидите сообщение об ошибке, выберите эту строку. Откроется конструктор, в котором вы увидите, какой шаг привел к сбою, а также сведения об ошибке. Если действие будет выполнено успешно, вы получите сообщение электронной почты со сведениями, которые только что добавили.


### <a name="workflow-ideas"></a>Идеи для рабочих процессов

* Вы можете отслеживать хэштег #oracle и переносить найденные твиты в базу данных, чтобы позднее использовать их в других приложениях. Добавьте в приложение логики триггер `Twitter - When a new tweet is posted` и введите хэштег **#oracle**. Затем добавьте действие `Oracle Database - Insert row` и выберите нужную таблицу:

    ![В диалоговом окне «при публикации нового твита» в качестве искомого текста отображается «хэш-тег Oracle», который позволяет указать частоту проверки. Это диалоговое окно ведет к диалоговому окну "Oracle Database", которое позволяет выбрать действие.](./media/connectors-create-api-oracledatabase/twitter-oracledb.png)

* Сообщения отправляются в очередь служебной шины. Теперь вам нужно получить эти сообщения и поместить их в базу данных. Добавьте в приложение логики триггер `Service Bus - when a message is received in a queue` и выберите используемую очередь. Затем добавьте действие `Oracle Database - Insert row` и выберите нужную таблицу:

    ![Параметр "при получении сообщения..." в диалоговом окне отображаются "заказы" как "имя очереди", и вы можете указать частоту проверки. В этом поле появляется диалоговое окно "Вставка строки (Предварительная версия)", которое позволяет выбрать "имя таблицы".](./media/connectors-create-api-oracledatabase/sbqueue-oracledb.png)

## <a name="common-errors"></a>Распространенные ошибки

#### <a name="error-cannot-reach-the-gateway"></a>**Ошибка**. Шлюз недоступен

**Причина.** Локальный шлюз данных не может подключиться к облаку. 

**Устранение.** Убедитесь, что шлюз работает на локальной машине, где он установлен, и что он может подключаться к Интернету.    Мы рекомендуем не устанавливать шлюза на компьютере, который может быть выключен или переведен в спящий режим.  Можно также попытаться перезапустить локальную службу шлюза данных (PBIEgwService).

#### <a name="error-the-provider-being-used-is-deprecated-systemdataoracleclient-requires-oracle-client-software-version-817-or-greater-see-httpsgomicrosoftcomfwlinkplinkid272376-to-install-the-official-provider"></a>**Ошибка**. Используемый поставщик является устаревшим. Для System.Data.OracleClient требуется клиентское программное обеспечение Oracle версии 8.1.7 или более поздней. Перейдите по ссылке [https://go.microsoft.com/fwlink/p/?LinkID=272376](/power-bi/connect-data/desktop-connect-oracle-database), чтобы установить официальный поставщик.

**Причина.** На компьютере, где выполняется локальный шлюз данных, не установлен пакет SDK для клиента Oracle.  

**Решение**. Скачайте и установите пакет SDK для клиента Oracle на компьютере, где установлен локальный шлюз данных.

#### <a name="error-table-tablename-does-not-define-any-key-columns"></a>**Ошибка**. В таблице [Имя_таблицы] не определены ключевые столбцы.

**Причина.** В таблице отсутствует первичный ключ.  

**Решение**. Соединитель базы данных Oracle можно использовать только с таблицей, в которой есть столбец первичного ключа.
 
## <a name="connector-specific-details"></a>Сведения о соединителях

Информацию о существующих ограничениях, а также о триггерах и действиях, определенных в Swagger, см. в статье со [сведениями о соединителях](/connectors/oracle/). 

## <a name="get-some-help"></a>Справочные сведения

Мы рекомендуем посетить [страницу Майкрософт с вопросами и ответами по Azure Logic Apps](/answers/topics/azure-logic-apps.html), где вы сможете задать вопросы, помочь другим пользователям и узнать, что они делают. 

Чтобы улучшить Logic Apps и соединители, внесите свои предложения или проголосуйте за уже внесенные на сайте [https://aka.ms/logicapps-wish](https://aka.ms/logicapps-wish). 


## <a name="next-steps"></a>Дальнейшие действия
[Создайте приложение логики](../logic-apps/quickstart-create-first-logic-app-workflow.md) и просмотрите в [списке интерфейсов API](apis-list.md) другие доступные соединители в Logic Apps.