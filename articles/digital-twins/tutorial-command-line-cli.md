---
title: Учебник. Создание графа в Azure Digital Twins (CLI)
titleSuffix: Azure Digital Twins
description: Учебник по созданию сценария Azure Digital Twins с помощью Azure CLI
author: baanders
ms.author: baanders
ms.date: 2/26/2021
ms.topic: tutorial
ms.service: digital-twins
ms.openlocfilehash: 578befe3e26ebb42fa2172976e07d0a5836e3743
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107165"
---
# <a name="tutorial-create-an-azure-digital-twins-graph-using-the-azure-cli"></a>Учебник. Создание графа Azure Digital Twins с помощью Azure CLI

[!INCLUDE [digital-twins-tutorial-selector.md](../../includes/digital-twins-tutorial-selector.md)]

В этом учебнике вы создадите граф Azure Digital Twins с помощью моделей, двойников и связей. Средством для работы с этим учебником является [набор команд Azure Digital Twins для **Azure CLI**](how-to-use-cli.md). 

С помощью этих команд CLI можно выполнять базовые действия Azure Digital Twins, такие как отправка моделей, создание и изменение двойников, а также создание связей. Вы также можете ознакомиться со [справочной документацией по набору команд *az dt*](/cli/azure/dt), чтобы увидеть полный набор команд CLI.

В этом учебнике вам предстоит выполнить следующее.
> [!div class="checklist"]
> * Моделирование среды
> * Создание цифровых двойников
> * Добавление связей для формирования графа
> * Запрос графа для получения ответов

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником требуется следующее.

Если у вас еще нет подписки Azure, **создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)** , прежде чем начинать работу.

### <a name="download-the-sample-models"></a>Скачивание примеров моделей

В этом учебнике используются две предварительно написанные модели, которые являются частью [полного примера проекта](/samples/azure-samples/digital-twins-samples/digital-twins-samples/) на C# для Azure Digital Twins. Файлы модели находятся здесь: 
* [*Room.json*](https://github.com/Azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Room.json)
* [*Floor.json*](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json)

Чтобы скачать файлы на компьютер, используйте ссылки навигации выше и скопируйте содержимое файлов в локальные файлы на компьютере с теми же именами (*Room.json* and *Floor.json*).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="set-up-cloud-shell-session"></a>Создание сеанса Cloud Shell
[!INCLUDE [Cloud Shell for Azure Digital Twins](../../includes/digital-twins-cloud-shell.md)]

### <a name="prepare-an-azure-digital-twins-instance"></a>Подготовка экземпляра Azure Digital Twins

Для работы с Azure Digital Twins, как показано в этой статье, сначала нужно настроить **экземпляр Azure Digital Twins** и необходимые разрешения для его использования. Если у вас уже есть настроенный экземпляр Azure Digital Twins после предыдущих действий, можно использовать его.

В противном случае следуйте инструкциям в статье [*Практическое руководство. Настройка экземпляра Azure Digital двойников и проверки подлинности (с помощью сценария)*](how-to-set-up-instance-cli.md). Кроме того, в этих инструкциях указано, как проверить, успешно ли завершен каждый шаг и готов ли новый экземпляр к работе.

После настройки экземпляра Azure Digital Twins запишите следующие значения, которые потребуются вам позднее при подключении к экземпляру:
* **_Имя узла_** экземпляра.
* **Подписка Azure**, которую вы использовали для создания экземпляра. 

Вы можете получить оба этих значения для своего экземпляра в выходных данных с помощью следующей команды Azure CLI: 

```azurecli-interactive
az dt show -n <ADT_instance_name>
```

:::image type="content" source="media/tutorial-command-line/cli/instance-details.png" alt-text="Снимок экрана: окно браузера Cloud Shell с выходными данными команды az dt. Выделенные поле hostName и идентификатор подписки (часть поля идентификатора).":::

## <a name="model-a-physical-environment-with-dtdl"></a>Моделирование физической среды с помощью DTDL

После подготовки экземпляра Azure Digital Twins и CLI приступим к созданию графа сценария. 

При создании решения Azure Digital Twins в первую очередь необходимо определить [**модели**](concepts-models.md) двойников для вашей среды. 

Модели похожи на классы в объектно-ориентированных языках программирования. Они предоставляют определяемые пользователем шаблоны для [цифровых двойников](concepts-twins-graph.md), экземпляры которых будут затем создаваться на основе этих шаблонов. Для их создания используется похожий на JSON язык, который называется **языком определения цифровых двойников или DTDL (Digital Twins Definition Language)** . На этом языке можно определять *свойства*, *телеметрию*, *связи* и *компоненты* двойника.

> [!NOTE]
> DTDL также позволяет определять *команды* в цифровых двойниках. Однако в настоящее время в службе Azure Digital Twins команды не поддерживаются.

На компьютере перейдите к файлу *Room.json*, созданному в разделе [Предварительные требования](#prerequisites). Откройте его в редакторе кода и измените следующим образом:

[!INCLUDE [digital-twins-tutorial-model-create.md](../../includes/digital-twins-tutorial-model-create.md)]

### <a name="upload-models-to-azure-digital-twins"></a>Отправка моделей в Azure Digital Twins

После построения моделей вам необходимо отправить их в свой экземпляр Azure Digital Twins. Таким образом ваш экземпляр службы Azure Digital Twins настраивается с вашим собственным словарем личного домена. После отправки моделей можно создавать экземпляры двойников, которые их используют.

1. Для добавления моделей с помощью Cloud Shell необходимо отправить файлы модели в хранилище Cloud Shell, чтобы они были доступны при выполнении команды Cloud Shell, которая их использует. Для этого выберите значок отправки и скачивания файлов, а затем — пункт "Отправить".

    :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-upload.png" alt-text="Снимок экрана: окно браузера Cloud Shell с выбором значка отправки.":::
    
    На компьютере перейдите к файлу *Room.json* и откройте его. Затем повторите этот шаг с файлом *Floor.json*.

1. Далее используйте команду [**az dt model create**](/cli/azure/dt/model#az_dt_model_create) (как показано ниже), чтобы отправить обновленную модель *Room* в свой экземпляр Azure Digital Twins. Вторая команда отправляет другую модель (*Floor*), которую также можно использовать в следующем разделе для создания различных типов двойников.

    ```azurecli-interactive
    az dt model create -n <ADT_instance_name> --models Room.json
    az dt model create -n <ADT_instance_name> --models Floor.json
    ```
    
    Выходные данные каждой команды будут отображать сведения об успешно отправленной модели.

    >[!TIP]
    >Кроме того, можно передать все модели в каталоге одновременно, используя параметр `--from-directory` для команды создания модели. Дополнительные сведения см. в разделе [Необязательные параметры для *az dt model create*](/cli/azure/dt/model#az_dt_model_create-optional-parameters).

1. Убедитесь, что модели были созданы с помощью команды [**az dt model list**](/cli/azure/dt/model#az_dt_model_list), как показано ниже. Эта команда выведет список с полными сведениями всех моделей, которые были отправлены в экземпляр Azure Digital Twins. 

    ```azurecli-interactive
    az dt model list -n <ADT_instance_name> --definition
    ```
    
    Найдите отредактированную модель *Room* в результатах команды:
    
    :::image type="content" source="media/tutorial-command-line/cli/output-get-models.png" alt-text="Снимок экрана: окно Cloud Shell с результатом команды списка моделей, включающим обновленную модель Room." lightbox="media/tutorial-command-line/cli/output-get-models.png":::

### <a name="errors"></a>ошибки

CLI также обрабатывает ошибки службы. 

Повторите команду `az dt model create`, чтобы попытаться второй раз отправить одну из тех моделей, которые только что были отправлены:

```azurecli-interactive
az dt model create -n <ADT_instance_name> --models Room.json
```

Так как модели не перезаписываются, появится код ошибки `ModelIdAlreadyExists`.

## <a name="create-digital-twins"></a>Создание цифровых двойников

Теперь, когда несколько моделей отправлено в ваш экземпляр Azure Digital Twins, можно создавать [**цифровые двойники**](concepts-twins-graph.md) на основе определений этих моделей. Цифровые двойники представляют сущности в вашей бизнес-среде — такие как датчики на ферме, комнаты в здании или фары автомобиля. 

Для создания цифрового двойника используйте команду [**az dt twin create**](/cli/azure/dt/twin#az_dt_twin_create). Вы должны сослаться на модель, на основе которой создается двойник, и при необходимости можете задать начальные значения каких-либо свойств в модели. На этом этапе вам не нужно передавать какие-либо сведения о связях.

1. Выполните следующий код в Cloud Shell, чтобы создать несколько двойников на основе модели *Room*, обновленной ранее, и другой модели (*Floor*). Напомним, что модель *Room* имеет три свойства, поэтому можно включить в команду аргументы с начальными значениями для этих свойств. (Как правило, инициализация значений свойств является необязательной, но они необходимы для этого учебника.)

    ```azurecli-interactive
    az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Room;2" --twin-id room0 --properties '{"RoomName":"Room0", "Temperature":70, "HumidityLevel":30}'
    az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Room;2" --twin-id room1 --properties '{"RoomName":"Room1", "Temperature":"80", "HumidityLevel":"60"}'
    az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Floor;1" --twin-id floor0
    az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Floor;1" --twin-id floor1
    ```

    >[!NOTE]
    > Если вы используете Cloud Shell в окружении PowerShell, может потребоваться экранировать символы кавычек, чтобы значение JSON `--properties` было проанализировано правильно. После этого изменения команды для создания двойников room будут выглядеть следующим образом:
    >
    > ```azurecli-interactive
    > az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Room;2" --twin-id room0 --properties '{\"RoomName\":\"Room0\", \"Temperature\":70, \"HumidityLevel\":30}'
    > az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Room;2" --twin-id room1 --properties '{\"RoomName\":\"Room1\", \"Temperature\":80, \"HumidityLevel\":60}'
    > ```
    > Это видно на следующем снимке экрана.
    
    Выходные данные каждой команды будут отображать сведения об успешно созданном двойнике (включая свойства двойников room, инициализированные с ними).

1. Вы можете убедиться, что двойники созданы с помощью команды [**az dt twin query**](/cli/azure/dt/twin#az_dt_twin_query), как показано ниже. Показанный запрос находит все цифровые двойники в вашем экземпляре Azure Digital Twins.
    
    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT * FROM DIGITALTWINS"
    ```
    
    Проверьте наличие в результатах двойников *room0*, *room1*, *floor0* и *floor1*. Ниже приведен фрагмент с частью результата этого запроса.
    
    :::image type="content" source="media/tutorial-command-line/cli/output-query-all.png" alt-text="Снимок экрана: окно Cloud Shell с частичным результатом запроса к двойнику, включая room0 и room1." lightbox="media/tutorial-command-line/cli/output-query-all.png":::

### <a name="modify-a-digital-twin"></a>Изменение цифровых двойников

Вы также можете изменить свойства созданного вами двойника. 

1. Выполните команду [**az dt twin update**](/cli/azure/dt/twin#az_dt_twin_update), чтобы изменить значение RoomName для *room0* с *Room0* на *PresidentialSuite*:

    ```azurecli-interactive
    az dt twin update -n <ADT_instance_name> --twin-id room0 --json-patch '{"op":"add", "path":"/RoomName", "value": "PresidentialSuite"}'
    ```
    
    >[!NOTE]
    > Если вы используете Cloud Shell в окружении PowerShell, может потребоваться экранировать символы кавычек, чтобы значение JSON `--json-patch` было проанализировано правильно. После этого изменения команда для обновления двойника будет выглядеть следующим образом:
    >
    > ```azurecli-interactive
    > az dt twin update -n <ADT_instance_name> --twin-id room0 --json-patch '{\"op\":\"add\", \"path\":\"/RoomName\", \"value\": \"PresidentialSuite\"}'
    > ```
    > Это видно на следующем снимке экрана.
    
    В выходных данных этой команды будут отображены текущие сведения о двойнике, а в результатах вы увидите новое значение для `RoomName`.

    :::image type="content" source="media/tutorial-command-line/cli/output-update-twin.png" alt-text="Снимок экрана: окно Cloud Shell с результатом команды обновления, который содержит значение PresidentialSuite для RoomName." lightbox="media/tutorial-command-line/cli/output-update-twin.png":::

1. Для проверки успешности выполнения обновления можно запустить команду [**az dt twin show**](/cli/azure/dt/twin#az_dt_twin_show), чтобы увидеть сведения о *room0*:

    ```azurecli-interactive
    az dt twin show -n <ADT_instance_name> --twin-id room0
    ```
    
    В выходных данных должно быть указано обновленное имя.

## <a name="create-a-graph-by-adding-relationships"></a>Создание графа путем добавления связей

Далее вы можете создать некоторые **связи** между этими двойниками, чтобы подключить их к [**графу двойников**](concepts-twins-graph.md). Графы двойников используются для представления всей среды. 

Типы создаваемых между двойниками связей определены в [моделях](#model-a-physical-environment-with-dtdl), которые вы загрузили ранее. [Определение модели для *Floor*](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json) указывает, что этажи могут иметь связь типа *contains* ("содержит"). Таким образом, можно создать связи типа *contains*, идущие от каждого двойника *Floor* к соответствующим номерам room, содержащимся на том или ином этаже.

Чтобы добавить связь, используйте команду [**az dt twin relationship create**](/cli/azure/dt/twin/relationship#az_dt_twin_relationship_create). Укажите двойник, от которого исходит связь, тип этой связи и двойник, к которому она присоединяется. Наконец, присвойте связи уникальный идентификатор. Если связь определена как имеющая свойства, можно также инициализировать свойства связи в этой команде.

1. Выполните следующий код, чтобы добавить связи типа *contains*, идущие от каждого созданного ранее двойника *Floor* к соответствующему двойнику *Room*. Связи называются *relationship0* и *relationship1*.

    ```azurecli-interactive
    az dt twin relationship create -n <ADT_instance_name> --relationship-id relationship0 --relationship contains --twin-id floor0 --target room0
    az dt twin relationship create -n <ADT_instance_name> --relationship-id relationship1 --relationship contains --twin-id floor1 --target room1
    ```
    
    >[!TIP]
    >Связь *contains* в модели [*Floor*](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json) также определялась двумя свойствами `ownershipUser` и `ownershipDepartment`, так что вы опять же можете указать для них начальные значения аргументов при создании связей.
    > Чтобы создать связь с этими инициализированными свойствами, добавьте параметр `--properties` в одну из приведенных выше команд следующим образом:
    > ```azurecli-interactive
    > ... --properties '{"ownershipUser":"MyUser", "ownershipDepartment":"MyDepartment"}'
    > ``` 
    > 
    > Если вы используете Cloud Shell в окружении PowerShell, может потребоваться экранировать символы кавычек, чтобы значение JSON `--properties` было проанализировано правильно.
    
    В выходных данных каждой команды будут отображаться сведения об успешно созданной связи.

1. Для проверки связей используйте любые из приведенных ниже команд, которые запрашивают связи в вашем экземпляре Azure Digital Twins.
    * Чтобы просмотреть все связи, исходящие от каждого этажа (просмотреть связи с одной стороны):
        ```azurecli-interactive
        az dt twin relationship list -n <ADT_instance_name> --twin-id floor0
        az dt twin relationship list -n <ADT_instance_name> --twin-id floor1
        ```
    * Чтобы просмотреть все связи, идущие к каждому номеру room (просмотреть связи с другой стороны):
        ```azurecli-interactive
        az dt twin relationship list -n <ADT_instance_name> --twin-id room0 --incoming
        az dt twin relationship list -n <ADT_instance_name> --twin-id room1 --incoming
        ```
    * Чтобы запросить любую из этих связей отдельно по ее идентификатору:
        ```azurecli-interactive
        az dt twin relationship show -n <ADT_instance_name> --twin-id floor0 --relationship-id relationship0
        az dt twin relationship show -n <ADT_instance_name> --twin-id floor1 --relationship-id relationship1
        ```

Двойники и связи, которые вы настроили в этом учебнике, образуют следующий концептуальный граф:

:::image type="content" source="media/tutorial-command-line/app/sample-graph.png" alt-text="Диаграмма концептуального графа, где floor0 соединяется связью relationship0 с room0, а floor1 — связью relationship1 с room1." border="false" lightbox="media/tutorial-command-line/app/sample-graph.png":::

## <a name="query-the-twin-graph-to-answer-environment-questions"></a>Запрос графа двойников для получения ответов по среде

Главной особенностью Azure Digital Twins является возможность просто и эффективно [запрашивать](concepts-query-language.md) граф двойников, чтобы получать ответы о вашей среде. В Azure CLI это делается с помощью команды [**az dt twin query**](/cli/azure/dt/twin#az_dt_twin_query).

Следующие запросы в Cloud Shell позволяют ответить на некоторые вопросы о примере окружения.

1. **Каковы все сущности из моей среды, представленные в Azure Digital Twins?** (запрос всех)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT * FROM DIGITALTWINS"
    ```

    Это позволяет быстро оценить состояние среды и убедиться, что в Azure Digital Twins все представлено именно так, как вы хотите. В результате будут выведены все цифровые двойники с подробными сведениями. Ниже приведен фрагмент:

    :::image type="content" source="media/tutorial-command-line/cli/output-query-all.png" alt-text="Снимок экрана: окно Cloud Shell с частичным результатом запроса к двойнику, включая room0 и room1." lightbox="media/tutorial-command-line/cli/output-query-all.png":::

    >[!TIP]
    >Вы можете увидеть, что это та же команда, которую вы использовали ранее в разделе [*Создание цифровых двойников*](#create-digital-twins), чтобы найти все Azure Digital Twins в экземпляре.

1. **Какие комнаты существуют в моей среде?** (запрос по модели)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT * FROM DIGITALTWINS T WHERE IS_OF_MODEL(T, 'dtmi:example:Room;2')"
    ```

    Вы можете ограничить свой запрос двойниками определенного типа, чтобы получить более конкретную информацию о том, что есть в вашей среде. В результате будут показаны *room0* и *room1*, но **не** показаны *floor0* и *room1* (так как это этажи, а не комнаты).
    
    :::image type="content" source="media/tutorial-command-line/cli/output-query-model.png" alt-text="Снимок экрана: окно Cloud Shell с результатом запроса модели, который включает только room0 и room1." lightbox="media/tutorial-command-line/cli/output-query-model.png":::

1. **Какие комнаты есть на этаже *floor0*?** (запрос по связи)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.`$dtId = 'floor0'"
    ```

    Вы можете выполнять запрос в свой граф на основе связей, чтобы получить сведения о том, как связаны двойники, или чтобы ограничить запрос определенной областью. На этаже *floor0* есть только комната *room0*, так что в результате будет только одна комната.

    :::image type="content" source="media/tutorial-command-line/cli/output-query-relationship.png" alt-text="Снимок экрана: окно Cloud Shell с результатом запроса связи, включающего room0." lightbox="media/tutorial-command-line/cli/output-query-relationship.png":::

    > [!NOTE]
    > Обратите внимание, что идентификатор двойника (например, *floor0* в приведенном выше запросе) запрашивается с помощью поля метаданных `$dtId`. 
    >
    >При использовании Cloud Shell для выполнения запроса с такими полями метаданных, которые начинаются с `$`, следует экранировать символ `$` с помощью обратного апострофа, чтобы указать Cloud Shell, что это не переменная, а литерал в тексте запроса. Это видно на снимке экрана выше.

1. **Какие двойники имеют температуру выше 75?** (запрос по свойству)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT * FROM DigitalTwins T WHERE T.Temperature > 75"
    ```

    Вы можете запрашивать граф на основе свойств, чтобы получать ответы на множество вопросов, например о наличии в вашей среде выбросов, которые требуют внимания. Поддерживаются также и другие операторы сравнения ( *<* , *>* , *=* и *!=* ). В результатах отображается комната *room1*, так как она имеет температуру 80.

    :::image type="content" source="media/tutorial-command-line/cli/output-query-property.png" alt-text="Снимок экрана: окно Cloud Shell с результатом запроса свойства, который включает только room1." lightbox="media/tutorial-command-line/cli/output-query-property.png":::

1. **Какие комнаты на этаже *floor0* имеют температуру выше 75?** (составной запрос)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.`$dtId = 'floor0' AND IS_OF_MODEL(room, 'dtmi:example:Room;2') AND room.Temperature > 75"
    ```

    Вы можете сочетать предыдущие запросы, как это делается в SQL, с помощью логических операторов, таких как `AND`, `OR`, `NOT`. В этом запросе используется `AND`, чтобы уточнить предыдущий запрос о температурах двойников. Теперь результат должен включать только комнаты с температурой выше 75, которые находятся на этаже *floor0* — в данном случае таких комнат нет. В результате мы получим пустой набор.

    :::image type="content" source="media/tutorial-command-line/cli/output-query-compound.png" alt-text="Снимок экрана: окно Cloud Shell с результатом составного запроса без элементов." lightbox="media/tutorial-command-line/cli/output-query-compound.png":::

## <a name="clean-up-resources"></a>Очистка ресурсов

По завершении работы с этим руководством вы можете выбрать ресурсы, которые нужно удалить, в зависимости от планируемых действий.

* **Если вы планируете перейти к следующему учебнику**, можете сохранить настроенные здесь ресурсы и повторно использовать экземпляр Azure Digital Twins без очистки.

* **Если вы хотите и дальше использовать экземпляр Azure Digital Twins, но вам нужно очистить все его модели, двойники и связи**, можно использовать команды [**az dt twin relationship delete**](/cli/azure/dt/twin/relationship#az_dt_twin_relationship_delete), [**az dt twin delete**](/cli/azure/dt/twin#az_dt_twin_delete) и [**az dt model delete**](/cli/azure/dt/model#az_dt_model_delete), чтобы очистить связи, двойники и модели в своем экземпляре соответственно.

[!INCLUDE [digital-twins-cleanup-basic.md](../../includes/digital-twins-cleanup-basic.md)]

Также может потребоваться удалить файлы модели, созданные на локальном компьютере.

## <a name="next-steps"></a>Дальнейшие действия 

В этом учебнике вы приступили к работе с Azure Digital Twins путем создания в своем экземпляре графа с использованием Azure CLI. Для формирования графа вы создали модели, цифровые двойники и связи. Вы также выполнили к графу ряд запросов, чтобы понять, на какие вопросы о среде может ответить Azure Digital Twins.

Переходите к следующему учебнику, в котором вы реализуете комплексный сценарий на основе данных, объединив Azure Digital Twins с другими службами Azure:
> [!div class="nextstepaction"]
> [*Руководство. Подключение комплексного решения*](tutorial-end-to-end.md)