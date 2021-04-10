---
title: Учебник. Создание графа в Azure Digital Twins (клиентское приложение)
titleSuffix: Azure Digital Twins
description: Учебник по созданию сценария Azure Digital Twins с использованием примера приложения командной строки
author: baanders
ms.author: baanders
ms.date: 5/8/2020
ms.topic: tutorial
ms.service: digital-twins
ms.openlocfilehash: c18366fd4bc510f32ac0ef255b27709797a3b626
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103493716"
---
# <a name="tutorial-create-an-azure-digital-twins-graph-using-a-sample-client-app"></a>Учебник. Создание графа Azure Digital Twins с помощью примера клиентского приложения

[!INCLUDE [digital-twins-tutorial-selector.md](../../includes/digital-twins-tutorial-selector.md)]

В этом учебнике вы создадите граф Azure Digital Twins с помощью моделей, двойников и связей. В качестве инструмента будет использоваться **пример клиентского приложения командной строки** для взаимодействия с экземпляром Azure Digital Twins. Это клиентское приложение аналогично тому, которое создается в разделе [*Учебник. Написание кода для клиентского приложения*](tutorial-code.md).

Этот пример можно использовать для выполнения базовых действий Azure Digital Twins, таких как отправка моделей, создание и изменение двойников и создание связей. Вы также можете изучить [код этого примера](https://github.com/Azure-Samples/digital-twins-samples/tree/master/), чтобы познакомиться с API-интерфейсами Azure Digital Twins и попробовать реализовать собственные команды, изменяя пример проекта под ваши нужды.

В этом учебнике вам предстоит выполнить следующее.
> [!div class="checklist"]
> * Моделирование среды
> * Создание цифровых двойников
> * Добавление связей для формирования графа
> * Запрос графа для получения ответов

[!INCLUDE [Azure Digital Twins tutorial: sample prerequisites](../../includes/digital-twins-tutorial-sample-prereqs.md)]

[!INCLUDE [Azure Digital Twins tutorial: configure the sample project](../../includes/digital-twins-tutorial-sample-configure.md)]

### <a name="run-the-sample-project"></a>Запуск примера проекта

Настроив приложение и проверку подлинности, запустите проект, нажав кнопку на панели инструментов:

:::image type="content" source="media/tutorial-command-line/app/start-button-sample.png" alt-text="Снимок экрана с кнопкой запуска Visual Studio (проект SampleClientApp)." lightbox="media/tutorial-command-line/app/start-button-sample.png":::

Откроется окно консоли. Выполните проверку подлинности и дождитесь выполнения команды. 
* Проверка подлинности осуществляется через браузер: ваш веб-браузер по умолчанию открывается с запросом проверки подлинности. Используйте этот запрос для входа со своими учетными данными Azure. Затем можно закрыть эту вкладку или окно браузера.

Ниже приведен снимок экрана, показывающий, как выглядит консоль проекта.

:::image type="content" source="media/tutorial-command-line/app/command-line-app.png" alt-text="Снимок экрана приветственного сообщения из приложения командной строки." lightbox="media/tutorial-command-line/app/command-line-app.png":::

> [!TIP]
> Чтобы получить список команд, которые вы можете использовать с этим проектом, введите `help` в консоли проекта и нажмите клавишу ВВОД.

Оставьте консоль проекта в рабочем состоянии для выполнения остальных шагов в этом учебнике.

## <a name="model-a-physical-environment-with-dtdl"></a>Моделирование физической среды с помощью DTDL

Подготовив экземпляр Azure Digital Twins и пример приложения, приступим к созданию графа сценария. 

При создании решения Azure Digital Twins в первую очередь необходимо определить [**модели**](concepts-models.md) двойников для вашей среды. 

Модели похожи на классы в объектно-ориентированных языках программирования. Они предоставляют определяемые пользователем шаблоны для [цифровых двойников](concepts-twins-graph.md), экземпляры которых будут затем создаваться на основе этих шаблонов. Для их создания используется похожий на JSON язык, который называется **языком определения цифровых двойников или DTDL (Digital Twins Definition Language)** . На этом языке можно определять *свойства*, *телеметрию*, *связи* и *компоненты* двойника.

> [!NOTE]
> DTDL также позволяет определять *команды* в цифровых двойниках. Однако в настоящее время в службе Azure Digital Twins команды не поддерживаются.

В окне Visual Studio, где открыт проект _**AdtE2ESample**_, из панели *Обозревателя решений* перейдите в папку *AdtSampleApp\SampleClientApp\Models*. В этой папке содержатся примеры моделей.

Выберите *Room.json*, чтобы открыть его в окне редактирования, и внесите следующие изменения.

[!INCLUDE [digital-twins-tutorial-model-create.md](../../includes/digital-twins-tutorial-model-create.md)]

### <a name="upload-models-to-azure-digital-twins"></a>Отправка моделей в Azure Digital Twins

После построения моделей вам необходимо отправить их в свой экземпляр Azure Digital Twins. Таким образом ваш экземпляр службы Azure Digital Twins настраивается с вашим собственным словарем личного домена. После отправки моделей можно создавать экземпляры двойников, которые их используют.

1. В окне консоли проекта выполните следующую команду, чтобы отправить ваши обновленные модели *Room* и *Floor*, которые вы будете использовать в следующем разделе для создания различных типов двойников.

    ```cmd/sh
    CreateModels Room Floor
    ```
    
    В выходных данных этой команды должно быть указано, что модели успешно созданы.

1. Убедитесь, что модели успешно созданы, выполнив команду `GetModels true`. Эта команда запрашивает в экземпляре Azure Digital Twins все отправленные модели и возвращает полные сведения о них. Найдите отредактированную модель *Room* в результатах команды:

    :::image type="content" source="media/tutorial-command-line/app/output-get-models.png" alt-text="Снимок экрана результатов GetModels, демонстрирующих обновленную модель Room." lightbox="media/tutorial-command-line/app/output-get-models.png":::

### <a name="errors"></a>ошибки

Пример приложения также обрабатывает ошибки службы. 

Повторите команду `CreateModels`, чтобы попытаться второй раз отправить одну из тех моделей, которые только что были отправлены:

```cmd/sh
CreateModels Room
```

Так как модели не перезаписываются, появится ошибка службы.
Дополнительные сведения о том, как удалить существующие модели, см. в разделе [*Практическое руководство. Управление моделями Digital Twins*](how-to-manage-model.md).
```cmd/sh
Response 409: Service request failed.
Status: 409 (Conflict)

Content:
{"error":{"code":"ModelAlreadyExists","message":"Could not add model dtmi:example:Room;2 as it already exists. Use Model_List API to view models that already exist. See the Swagger example.(http://aka.ms/ModelListSwSmpl)"}}

Headers:
Strict-Transport-Security: REDACTED
Date: Wed, 20 May 2020 00:53:49 GMT
Content-Length: 223
Content-Type: application/json; charset=utf-8
```

## <a name="create-digital-twins"></a>Создание цифровых двойников

Теперь, когда несколько моделей отправлено в ваш экземпляр Azure Digital Twins, можно создавать [**цифровые двойники**](concepts-twins-graph.md) на основе определений этих моделей. Цифровые двойники представляют сущности в вашей бизнес-среде — такие как датчики на ферме, комнаты в здании или фары автомобиля. 

Для создания цифрового двойника используется команда `CreateDigitalTwin`. Вы должны сослаться на модель, на основе которой создается двойник, и при необходимости можете задать начальные значения каких-либо свойств в модели. На этом этапе вам не нужно передавать какие-либо сведения о связях.

1. Выполните следующий код в консоли запущенного проекта, чтобы создать несколько двойников на основе модели *Room*, которую вы обновили ранее, и другой модели, *Floor*. Напомним, что модель *Room* имеет три свойства, поэтому можно включить в команду аргументы с начальными значениями для этих свойств. (Как правило, инициализация значений свойств является необязательной, но они необходимы для этого учебника.)

    ```cmd/sh
    CreateDigitalTwin dtmi:example:Room;2 room0 RoomName string Room0 Temperature double 70 HumidityLevel double 30
    CreateDigitalTwin dtmi:example:Room;2 room1 RoomName string Room1 Temperature double 80 HumidityLevel double 60
    CreateDigitalTwin dtmi:example:Floor;1 floor0
    CreateDigitalTwin dtmi:example:Floor;1 floor1
    ```

    В выходных данных этих команд должно быть указано, что двойники успешно созданы. 
    
    :::image type="content" source="media/tutorial-command-line/app/output-create-digital-twin.png" alt-text="Снимок экрана с фрагментом результатов команд CreateDigitalTwin, включающим floor0, floor1, room0 и room1." lightbox="media/tutorial-command-line/app/output-create-digital-twin.png":::

1. Вы можете проверить создание двойников, выполнив команду `Query`. Эта команда запрашивает в вашем экземпляре Azure Digital Twins все содержащиеся в нем цифровые близнецы. Проверьте наличие в результатах двойников *room0*, *room1*, *floor0* и *floor1*.

### <a name="modify-a-digital-twin"></a>Изменение цифровых двойников

Вы также можете изменить свойства созданного вами двойника. 

> [!NOTE]
> Для определения вносимых в двойник изменений в используемом REST API применяется формат [JSON Patch](http://jsonpatch.com/). Этот же формат используется в приложении командной строки для более точного соответствия принципам работы текущего API.

1. Выполните следующую команду, чтобы изменить RoomName двойника *room0* с *Room0* на *PresidentialSuite*:
    
    ```cmd/sh
    UpdateDigitalTwin room0 add /RoomName string PresidentialSuite
    ```
    
    В выходных данных должно быть указано, что двойник успешно обновлен.

1. Для проверки изменения можно выполнить следующую команду, отображающую сведения о *room0*:

    ```cmd/sh
    GetDigitalTwin room0
    ```
    
    В выходных данных должно быть указано обновленное имя.


## <a name="create-a-graph-by-adding-relationships"></a>Создание графа путем добавления связей

Далее вы можете создать некоторые **связи** между этими двойниками, чтобы подключить их к [**графу двойников**](concepts-twins-graph.md). Графы двойников используются для представления всей среды. 

Типы создаваемых между двойниками связей определены в [моделях](#model-a-physical-environment-with-dtdl), которые вы загрузили ранее. [Определение модели для *Floor*](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json) указывает, что этажи могут иметь связь типа *contains* ("содержит"). Таким образом, можно создать связи типа *contains*, идущие от каждого двойника *Floor* к соответствующим номерам room, содержащимся на том или ином этаже.

Чтобы добавить связь, используйте команду `CreateRelationship`. Укажите двойник, от которого исходит связь, тип этой связи и двойник, к которому она присоединяется. Наконец, присвойте связи уникальный идентификатор.

1. Выполните следующий код, чтобы добавить связь "contains" от каждого из созданных ранее двойников *Floor* к соответствующему двойнику *Room*. Связи называются *relationship0* и *relationship1*.

    ```cmd/sh
    CreateRelationship floor0 contains room0 relationship0
    CreateRelationship floor1 contains room1 relationship1
    ```

    >[!TIP]
    >Связь *contains* в модели [*Floor*](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json) также определялась двумя строковыми свойствами `ownershipUser` и `ownershipDepartment`, так что вы также можете указать для них начальные значения аргументов при создании связей.
    > Вот другая версия вышеуказанной команды для создания связи *relationship0*, которая также указывает начальные значения этих свойств:
    > ```cmd/sh
    > CreateRelationship floor0 contains room0 relationship0 ownershipUser string MyUser ownershipDepartment string myDepartment
    > ``` 
    
    Выходные данные этих команд подтверждают, что связи были успешно созданы:
    
    :::image type="content" source="media/tutorial-command-line/app/output-create-relationship.png" alt-text="Снимок экрана с фрагментом результатов команд CreateRelationship, включающим relationship0 и relationship1." lightbox="media/tutorial-command-line/app/output-create-relationship.png":::

1. Для проверки связей используйте любые из приведенных ниже команд, которые запрашивают связи в вашем экземпляре Azure Digital Twins.
    * Чтобы просмотреть все связи, исходящие от каждого этажа (просмотреть связи с одной стороны):
        ```cmd/sh
        GetRelationships floor0
        GetRelationships floor1
        ```
    * Чтобы просмотреть все связи, идущие к каждому номеру room (просмотреть связи с другой стороны):
        ```cmd/sh
        GetIncomingRelationships room0
        GetIncomingRelationships room1
        ```
    * Чтобы запросить любую из этих связей отдельно по ее идентификатору:
        ```cmd/sh
        GetRelationship floor0 relationship0
        GetRelationship floor1 relationship1
        ```

Двойники и связи, которые вы настроили в этом учебнике, образуют следующий концептуальный граф:

:::image type="content" source="media/tutorial-command-line/app/sample-graph.png" alt-text="Диаграмма концептуального графа, где floor0 соединяется связью relationship0 с room0, а floor1 — связью relationship1 с room1." border="false" lightbox="media/tutorial-command-line/app/sample-graph.png":::

## <a name="query-the-twin-graph-to-answer-environment-questions"></a>Запрос графа двойников для получения ответов по среде

Главной особенностью Azure Digital Twins является возможность просто и эффективно [запрашивать](concepts-query-language.md) граф двойников, чтобы получать ответы о вашей среде. 

Следующие команды в консоли запущенного проекта позволяют ответить на некоторые вопросы о примере среды.

1. **Каковы все сущности из моей среды, представленные в Azure Digital Twins?** (запрос всех)

    ```cmd/sh
    Query
    ```

    Это позволяет быстро оценить состояние среды и убедиться, что в Azure Digital Twins все представлено именно так, как вы хотите. В результате будут выведены все цифровые двойники с подробными сведениями. Ниже приведен фрагмент:

    :::image type="content" source="media/tutorial-command-line/app/output-query-all.png" alt-text="Снимок экрана с частью результатов запроса к двойнику, включающей room0 и floor1.":::

    >[!NOTE]
    >В примере проекта команда `Query` без дополнительных аргументов является эквивалентом запроса `Query SELECT * FROM DIGITALTWINS`. Чтобы запросить данные о всех двойниках в экземпляре с помощью [интерфейсов API для запросов](/rest/api/digital-twins/dataplane/query) или [команд CLI](how-to-use-cli.md), используйте более длинный (полный) запрос.

1. **Какие комнаты существуют в моей среде?** (запрос по модели)

    ```cmd/sh
    Query SELECT * FROM DIGITALTWINS T WHERE IS_OF_MODEL(T, 'dtmi:example:Room;2')
    ```

    Вы можете ограничить свой запрос двойниками определенного типа, чтобы получить более конкретную информацию о том, что есть в вашей среде. В результате будут показаны *room0* и *room1*, но **не** показаны *floor0* и *room1* (так как это этажи, а не комнаты).
    
    :::image type="content" source="media/tutorial-command-line/app/output-query-model.png" alt-text="Снимок экрана результатов запроса к модели, где показаны только room0 и room1.":::

1. **Какие комнаты есть на этаже *floor0*?** (запрос по связи)

    ```cmd/sh
    Query SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.$dtId = 'floor0'
    ```

    Вы можете выполнять запрос в свой граф на основе связей, чтобы получить сведения о том, как связаны двойники, или чтобы ограничить запрос определенной областью. На этаже *floor0* есть только комната *room0*, так что в результате будет только одна комната.

    :::image type="content" source="media/tutorial-command-line/app/output-query-relationship.png" alt-text="Снимок экрана результатов запроса связи, где показано room0.":::

1. **Какие двойники имеют температуру выше 75?** (запрос по свойству)

    ```cmd/sh
    Query SELECT * FROM DigitalTwins T WHERE T.Temperature > 75
    ```

    Вы можете запрашивать граф на основе свойств, чтобы получать ответы на множество вопросов, например о наличии в вашей среде выбросов, которые требуют внимания. Поддерживаются также и другие операторы сравнения ( *<* , *>* , *=* и *!=* ). В результатах отображается комната *room1*, так как она имеет температуру 80.

    :::image type="content" source="media/tutorial-command-line/app/output-query-property.png" alt-text="Снимок экрана результатов запроса свойства, где показано только room1.":::

1. **Какие комнаты на этаже *floor0* имеют температуру выше 75?** (составной запрос)

    ```cmd/sh
    Query SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.$dtId = 'floor0' AND IS_OF_MODEL(room, 'dtmi:example:Room;2') AND room.Temperature > 75
    ```

    Вы можете сочетать предыдущие запросы, как это делается в SQL, с помощью логических операторов, таких как `AND`, `OR`, `NOT`. В этом запросе используется `AND`, чтобы уточнить предыдущий запрос о температурах двойников. Теперь результат должен включать только комнаты с температурой выше 75, которые находятся на этаже *floor0* — в данном случае таких комнат нет. В результате мы получим пустой набор.

    :::image type="content" source="media/tutorial-command-line/app/output-query-compound.png" alt-text="Снимок экрана результатов составного запроса, где никаких результатов не показано." lightbox="media/tutorial-command-line/app/output-query-compound.png":::

## <a name="clean-up-resources"></a>Очистка ресурсов

По завершении работы с этим руководством вы можете выбрать ресурсы, которые нужно удалить, в зависимости от планируемых действий.

* **Если вы собираетесь перейти к следующему руководству**, то можете сохранить настроенные здесь ресурсы, чтобы продолжить работу с этим экземпляром Azure Digital Twins и настроенным примером приложения для работы со следующим руководством.

* **Если вы хотите и дальше использовать экземпляр Azure Digital Twins, но вам нужно очистить все его модели, двойники и связи**, можно использовать команды `DeleteAllTwins` и `DeleteAllModels` примера приложения, чтобы очистить двойники и модели в своем экземпляре соответственно.

[!INCLUDE [digital-twins-cleanup-basic.md](../../includes/digital-twins-cleanup-basic.md)]

Также, возможно, потребуется удалить папку проекта с вашего локального компьютера.

## <a name="next-steps"></a>Дальнейшие действия 

В этом учебнике вы приступили к работе с Azure Digital Twins, создав в своем экземпляре граф, используя пример клиентского приложения. Для формирования графа вы создали модели, цифровые двойники и связи. Вы также выполнили к графу ряд запросов, чтобы понять, на какие вопросы о среде может ответить Azure Digital Twins.

Переходите к следующему учебнику, в котором вы реализуете комплексный сценарий на основе данных, объединив Azure Digital Twins с другими службами Azure:
> [!div class="nextstepaction"]
> [*Руководство. Подключение комплексного решения*](tutorial-end-to-end.md)