---
title: Руководство по Подача запросов из Power Apps
titleSuffix: Azure Cognitive Search
description: Пошаговое руководство. Создание приложения Power Apps, которое подключается к индексу Когнитивного поиска Azure, отправляет запросы и отображает результаты.
author: luiscabrer
manager: eladz
ms.author: luisca
ms.service: cognitive-search
ms.devlang: rest-api
ms.topic: tutorial
ms.date: 11/17/2020
ms.openlocfilehash: e8c16f02cf6b77fa54d2a19abac48e9914aa99bd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96008571"
---
# <a name="tutorial-query-a-cognitive-search-index-from-power-apps"></a>Руководство по Создание запроса к индексу Когнитивного поиска из Power Apps

Используйте среду быстрой разработки приложений Power Apps, чтобы создать пользовательское приложение для содержимого в Когнитивном поиске Azure, доступного для поиска.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Подключение к Когнитивному поиску Azure
> * Настройка запроса
> * Визуализация результатов в приложении холста

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* [Учетная запись Power Apps](https://make.powerapps.com)

* [Индекс hotels-sample](search-get-started-portal.md), размещенный в службе поиска

* [Ключ API запроса](search-security-api-keys.md#find-existing-keys)

## <a name="1---create-a-custom-connector"></a>1\. Создание настраиваемого соединителя

Соединитель в Power Apps определяет подключение к источнику данных. На этом шаге вы создадите настраиваемый соединитель для подключения к индексу поиска в облаке.

1. [Войдите](https://make.powerapps.com) в Power Apps.

1. В области слева щелкните **Данные** > **Настраиваемые соединители**.
 
    :::image type="content" source="./media/search-howto-powerapps/1-2-custom-connector.png" alt-text="Меню настраиваемых соединителей" border="true":::

1. Щелкните **+ Создать настраиваемый соединитель** и выберите **Создать с нуля**.

    :::image type="content" source="./media/search-howto-powerapps/1-3-create-blank.png" alt-text="Пункт меню для создания с нуля" border="true":::

1. Присвойте имя настраиваемому соединителю (например, *AzureSearchQuery*) и щелкните **Продолжить**.

1. Введите сведения на странице "Общие".

   * Цвет фона для значка (например, #007ee5).
   * Описание (например, "Соединитель для Когнитивного поиска Azure")
   * В качестве узла укажите URL-адрес службы поиска (например, `<yourservicename>.search.windows.net`).
   * В качестве базового URL-адреса просто укажите /.

    :::image type="content" source="./media/search-howto-powerapps/1-5-general-info.png" alt-text="Диалоговое окно для ввода общей информации" border="true":::

1. На странице "Безопасность" укажите *Ключ API* для параметра **Тип проверки подлинности**, а затем введите *api-key* в качестве метки и имени параметра. В поле **Расположение параметра** выберите *Заголовок*, как показано ниже.

    :::image type="content" source="./media/search-howto-powerapps/1-6-authentication-type.png" alt-text="Параметр Тип проверки подлинности" border="true":::

1. На странице "Определения" выберите **+ Новое действие**, чтобы создать действие для обращения к индексу. Введите значение "Запрос" в качестве описания и имени для идентификатора операции. Введите описание, например *Запросы к индексу поиска*.

    :::image type="content" source="./media/search-howto-powerapps/1-7-new-action.png" alt-text="Параметры для нового действия" border="true":::

1. Прокрутите страницу вниз. В окне "Запросы" нажмите кнопку **+ Импорт из примера**, чтобы настроить запрос к службе поиска.

   * Выберите команду `GET`.

   * В качестве URL-адреса введите пример запроса к индексу поиска (например, `search=*` возвращает все документы, а `$select=` позволяет выбрать поля). Версию API нужно обязательно указать. Полный URL-адрес может выглядеть так: `https://mydemo.search.windows.net/indexes/hotels-sample-index/docs?search=*&$select=HotelName,Description,Address/City&api-version=2020-06-30`.

   * В разделе "Заголовки" введите `Content-Type`. Значение `application/json` будет задано на следующем шаге.

     **Power Apps** использует синтаксис в URL-адресе для извлечения параметров из запроса: параметры поиска, выбора и версии API можно настроить с помощью мастера.

       :::image type="content" source="./media/search-howto-powerapps/1-8-1-import-from-sample.png" alt-text="Импорт из примера" border="true":::

1. Щелкните **Импорт**, чтобы автоматически заполнить страницу "Запрос". Завершите настройку метаданных параметров, щелкнув символ **...** рядом с каждым параметром. После обновления каждого параметра щелкайте **Назад**, чтобы вернуться на страницу "Запрос".

   :::image type="content" source="./media/search-howto-powerapps/1-8-2-import-from-sample.png" alt-text="Диалоговое окно импорта из примера" border="true":::

1. Для параметра *search* сделайте следующее. Задайте `*` в качестве **значения по умолчанию**, для параметра **Обязательно** укажите *false* и для параметра **Видимость** — *Нет*. 

    :::image type="content" source="./media/search-howto-powerapps/1-10-1-parameter-metadata-search.png" alt-text="Метаданные параметра search" border="true":::

1. Для параметра *select* сделайте следующее. Задайте `HotelName,Description,Address/City` в качестве **значения по умолчанию**, для параметра **Обязательно** укажите *false* и для параметра **Видимость** — *Нет*.  

    :::image type="content" source="./media/search-howto-powerapps/1-10-4-parameter-metadata-select.png" alt-text="Метаданные параметра select" border="true":::

1. В разделе *api-version* выполните следующее. Задайте `2020-06-30` в качестве **значения по умолчанию**, для параметра **Обязательно** укажите *true* и для параметра **Видимость** — *Внутренняя*.  

    :::image type="content" source="./media/search-howto-powerapps/1-10-2-parameter-metadata-version.png" alt-text="Метаданные параметра version" border="true":::

1. Для параметра *Content-Type* сделайте следующее. Задайте значение `application/json`.

1. После внесения этих изменений переключитесь в представление **редактора Swagger**. В разделе параметров вы увидите следующую конфигурацию.

    ```JSON
    parameters:
      - {name: search, in: query, required: false, type: string, default: '*'}
      - {name: $select, in: query, required: false, type: string, default: 'HotelName,Description,Address/City'}
      - {name: api-version, in: query, required: true, type: string, default: '2020-06-30',
        x-ms-visibility: internal}
      - {name: Content-Type, in: header, required: false, type: string}
    ```

1. Вернитесь к мастеру и перейдите на шаг **3. Запрос**. Прокрутите вниз до раздела с ответом. Щелкните **Добавить ответ по умолчанию**. Это важно, так как Power Apps нужно распознать схему ответа. 

1. Вставьте пример ответа. Пример ответа проще всего записать с помощью проводника поиска на портале Azure. Введите в проводнике поиска те же сведения, что и для основного запроса, но добавьте параметр **$top=2**, чтобы ограничить результаты двумя документами: `search=*&$select=HotelName,Description,Address/City&$top=2`. 

   Для обнаружения схемы Power Apps требуется несколько результатов. Вы можете скопировать следующий ответ в мастер, если вы используете индекс hotels-sample.

    ```JSON
    {
        "@odata.context": "https://mydemo.search.windows.net/indexes('hotels-sample-index')/$metadata#docs(*)",
        "value": [
            {
                "@search.score": 1,
                "HotelName": "Arcadia Resort & Restaurant",
                "Description": "The largest year-round resort in the area offering more of everything for your vacation – at the best value!  What can you enjoy while at the resort, aside from the mile-long sandy beaches of the lake? Check out our activities sure to excite both young and young-at-heart guests. We have it all, including being named “Property of the Year” and a “Top Ten Resort” by top publications.",
                "Address": {
                    "City": "Seattle"
                }
            },
            {
                "@search.score": 1,
                "HotelName": "Travel Resort",
                "Description": "The Best Gaming Resort in the area.  With elegant rooms & suites, pool, cabanas, spa, brewery & world-class gaming.  This is the best place to play, stay & dine.",
                "Address": {
                    "City": "Albuquerque"
                }
            }
        ]
    }
    ```

    > [!TIP] 
    > При этом есть ограничение на количество символов в ответе JSON, поэтому перед вставкой JSON иногда нужно сокращать. Схема и формат ответа важнее, чем сами значения. Например, в поле Description можно оставить только первое предложение.

1. Щелкните **Импорт**, чтобы добавить ответ по умолчанию.

1. Щелкните **Создать соединитель** вверху справа, чтобы сохранить определение.

1. Щелкните **Закрыть**, чтобы закрыть соединитель.

## <a name="2---test-the-connection"></a>2\. Проверка подключения

Созданный соединитель нужно повторно открыть в списке "Настраиваемые соединители", чтобы протестировать. Позже при внесении обновлений можно выполнить тестирование в мастере.

Для выполнения этой задачи требуется [ключ API запросов](search-security-api-keys.md#find-existing-keys). При каждом создании для запуска теста или включения в приложение соединителю требуется ключ API запроса, который использовался для подключения к Когнитивному поиску Azure.

1. В области слева щелкните **Настраиваемые соединители**.

1. Найдите соединитель в списке (в нашем примере — AzureSearchQuery).

1. Выберите нужный соединитель, разверните список действий и выберите **Просмотреть параметры**.

    :::image type="content" source="./media/search-howto-powerapps/1-11-1-test-connector.png" alt-text="Просмотр свойств" border="true":::

1. Вверху справа щелкните **Изменить**.

1. Теперь выберите **4. Тест**, чтобы открыть страницу тестирования.

1. В разделе "Операция тестирования" щелкните **+ Новое подключение**.

1. Введите ключ API запроса. Это запрос Когнитивного поиска Azure, который обращается к индексу только для чтения. Нужный ключ можно [найти на портале Azure](search-security-api-keys.md#find-existing-keys). 

1. В разделе "Операции" нажмите кнопку **Проверить операцию**. Если проверка пройдет успешно, вы увидите состояние ответа 200, а в самом тексте ответа будет содержаться код JSON с результатами поиска.

    :::image type="content" source="./media/search-howto-powerapps/1-11-2-test-connector.png" alt-text="Ответ в формате JSON" border="true":::

## <a name="3---visualize-results"></a>3\. Визуализация результатов

На этом шаге вы создадите приложение Power Apps с полем поиска, кнопкой поиска и областью для отображения результатов. Это приложение Power Apps будет подключаться к созданному настраиваемому соединителю для получения данных из Поиска Azure.

1. В области слева щелкните **Приложения** >  **+ Создать приложение** > **Холст**.

    :::image type="content" source="./media/search-howto-powerapps/2-1-create-canvas.png" alt-text="Создание приложения на основе холста" border="true":::

1. Выберите тип приложения. Для работы с этим руководством выберите **Пустое приложение** с макетом **Макет для телефона**. Откроется окно **Power Apps Studio**.

1. В этом окне откройте вкладку **Источники данных** и щелкните соединитель, который вы только что создали. В нашем примере это соединитель с именем *AzureSearchQuery*. Щелкните **Добавить подключение**.

   Введите ключ API запроса.

    :::image type="content" source="./media/search-howto-powerapps/2-3-connect-connector.png" alt-text="Подключение соединителя" border="true":::

    Теперь *AzureSearchQuery* будет считаться источником данных, доступным для использования из приложения.

1. На вкладке **Вставка** добавьте на холст несколько элементов управления.

    :::image type="content" source="./media/search-howto-powerapps/2-4-add-controls.png" alt-text="Добавление элементов управления" border="true":::

1. Добавьте следующие элементы.

   * Текстовая подпись со значением "Запрос".
   * Поле для ввода текста (присвойте ему имя *txtQuery* и значение по умолчанию *).
   * Кнопка с текстом "Поиск". 
   * Вертикальная коллекция с именем *galleryResults*.

    Теперь холст будет выглядеть так:

    :::image type="content" source="./media/search-howto-powerapps/2-5-controls-layout.png" alt-text="Макет с элементами управления" border="true":::

1. Чтобы кнопка **Поиск** отправляла запрос, вставьте следующее действие в поле **OnSelect**.

    ```
    If(!IsBlank(txtQuery.Text),
        ClearCollect(azResult, AzureSearchQuery.Query({search: txtQuery.Text}).value))
    ```

   На следующем снимке экрана показана панель ввода формулы для действия **OnSelect**.

    :::image type="content" source="./media/search-howto-powerapps/2-6-search-button-event.png" alt-text="Действие OnSelect" border="true":::

   Это действие будет по нажатию кнопки обновлять новую коллекцию *azResult*, записывая в нее результат выполнения поискового запроса, используя текст в поле *txtQuery* в качестве условия поиска.

   > [!NOTE]
   > Если формула возвращает ошибку с сообщением The function 'ClearCollect' has some invalid functions (Функция ClearCollect содержит несколько недопустимых функций), попробуйте сделать следующее.
   > 
   > * Для начала проверьте ссылку на соединитель. Удалите имя соединителя и начните вводить его заново. Intellisense предложит правильные имена соединителя и команды.
   > 
   > * Если ошибка не будет устранена, удалите этот соединитель и создайте его заново. Если существует несколько экземпляров соединителя, приложение может выбрать недопустимый.
   > 

1. Подключите элемент управления вертикальной коллекции к коллекции *azResult*, которую вы создали на предыдущем шаге. 

   Выберите элемент управления коллекцией и выполните следующие действия в области свойств.

   * Для параметра **DataSource** укажите значение *azResult*.
   * Выберите **макет**, который соответствует типу данных в вашем индексе. В нашем примере выбран макет *Заголовок, подзаголовок и текст*.
   * **Измените поля**, чтобы выбрать поля для отображения.

    Так как мы уже предоставили пример результата при определении соединителя, приложение учитывает поля в вашем индексе.
    
    :::image type="content" source="./media/search-howto-powerapps/2-7-gallery-select-fields.png" alt-text="Поля коллекции" border="true":::   
 
1. Нажмите клавишу **F5** для предварительного просмотра приложения.  

    :::image type="content" source="./media/search-howto-powerapps/2-8-3-final.png" alt-text="Окончательная версия приложения" border="true":::    

<!--     Remember that the fields can be set to calculated values.

    For the example, setting using the *"Image, Title and Subtitle"* layout and specifying the *Image* function as the concatenation of the root path for the data and the file name (for instance, `"https://mystore.blob.core.windows.net/multilang/" & ThisItem.metadata_storage_name`) will produce the result below.

    :::image type="content" source="./media/search-howto-powerapps/2-8-2-final.png" alt-text="Final app" border="true":::         -->

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы работаете в своей подписке, по окончании проекта рекомендуем решить, нужны ли вам созданные ресурсы. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок **Все ресурсы** или **Группы ресурсов** на панели навигации слева.

При работе с бесплатной версией службы помните о том, что вам доступно максимум три индекса, индексатора и источника данных. Вы можете удалить отдельные элементы на портале, чтобы не превысить лимит.

## <a name="next-steps"></a>Дальнейшие действия

Power Apps позволяет очень быстро разрабатывать настраиваемые приложения. Теперь, когда вы знаете, как подключиться к индексу поиска, вы можете переходить к созданию дополнительных возможностей визуализации в настраиваемом приложении Power Apps.

> [!div class="nextstepaction"]
> [Каталог материалов для обучения в Power Apps ](/powerapps/learning-catalog/get-started)