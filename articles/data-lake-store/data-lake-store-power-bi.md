---
title: Анализ данных в Azure Data Lake Storage 1-го поколения — Power BI
description: Узнайте, как использовать Power BI Desktop для анализа и визуализации данных, хранящихся в Azure Data Lake Storage 1-го поколения.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: bcc561cd5eea4372d798fff4580362ba0879c3a4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91574200"
---
# <a name="analyze-data-in-azure-data-lake-storage-gen1-by-using-power-bi"></a>Анализ данных в Azure Data Lake Storage 1-го поколения с помощью Power BI
В этой статье вы узнаете, как использовать Power BI Desktop для анализа и визуализации данных в Azure Data Lake Storage 1-го поколения.

## <a name="prerequisites"></a>Предварительные требования
Перед началом работы с этим учебником необходимо иметь следующее:

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Учетная запись Data Lake Storage 1-го поколения**. Следуйте инструкциям из статьи [Начало работы с Azure Data Lake Storage Gen1 с помощью портала Azure](data-lake-store-get-started-portal.md). В этой статье предполагается, что вы уже создали учетную запись Data Lake Storage 1-го поколения с именем **myadlsg1** и добавили в нее образец файла данных (**Drivers.txt**). Этот образец файла можно скачать в [репозитории Git для озера данных Azure](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData/Drivers.txt).
* **Power BI Desktop**. Это средство можно скачать в [Центре загрузки Майкрософт](https://www.microsoft.com/en-us/download/details.aspx?id=45331). 

## <a name="create-a-report-in-power-bi-desktop"></a>Создание отчета в Power BI Desktop
1. Запустите Power BI Desktop на своем компьютере.
2. На ленте **Главная** нажмите кнопку **Получить данные**, а затем щелкните "Дополнительно". В диалоговом окне **Получение данных** выберите пункт **Azure**, а затем выберите пункт **Azure Data Lake Store** и нажмите кнопку **Подключиться**.
   
    ![Снимок экрана: диалоговое окно "получение данных" с выделенным параметром Azure Data Lake Store и вызываемым параметром Connect.](./media/data-lake-store-power-bi/get-data-lake-store-account.png "Подключение к Data Lake Storage 1-го поколения")
3. Если появится диалоговое окно с сообщением о том, что соединитель находится в стадии разработки, выберите вариант "Продолжить".
4. В диалоговом окне **Azure Data Lake Store** укажите URL-адрес для своей учетной записи Data Lake Storage 1-го поколения и нажмите кнопку **ОК**.
   
    ![URL-адрес Data Lake Storage 1-го поколения](./media/data-lake-store-power-bi/get-data-lake-store-account-url.png "URL-адрес Data Lake Storage 1-го поколения")
5. В следующем диалоговом окне нажмите кнопку **Вход**, чтобы войти в учетную запись Data Lake Storage 1-го поколения. Вы будете перенаправлены на страницу входа своей организации. Следуйте инструкциям для входа в учетную запись.
   
    ![Вход в Data Lake Storage 1-го поколения](./media/data-lake-store-power-bi/get-data-lake-store-account-signin.png "Вход в Data Lake Storage 1-го поколения")
6. Успешно выполнив вход, нажмите кнопку **Подключиться**.
   
    ![Снимок экрана диалогового окна Azure Data Lake Store с вызываемым параметром Connect.](./media/data-lake-store-power-bi/get-data-lake-store-account-connect.png "Подключение к Data Lake Storage 1-го поколения")
7. В следующем диалоговом окне будет указан файл, который вы добавили в учетную запись Data Lake Storage 1-го поколения. Проверьте информацию и нажмите кнопку **Загрузить**.
   
    ![Загрузка данных из Data Lake Storage 1-го поколения](./media/data-lake-store-power-bi/get-data-lake-store-account-load.png "Загрузка данных из Data Lake Storage 1-го поколения")
8. После успешной загрузки данных в Power BI вы увидите приведенные ниже поля на вкладке **Поля** .
   
    ![Импортированные поля](./media/data-lake-store-power-bi/imported-fields.png "Импортированные поля")
   
    Однако для визуализации и анализа данных мы рекомендуем использовать показанные ниже поля.
   
    ![Нужные поля](./media/data-lake-store-power-bi/desired-fields.png "Нужные поля")
   
    В следующих шагах мы обновим запрос, чтобы преобразовать импортированные данные в нужный формат.
9. На вкладке **Главная** ленты нажмите кнопку **Изменить запросы**.
   
    ![Снимок экрана ленты "Главная" с вызываемым параметром "изменить запросы".](./media/data-lake-store-power-bi/edit-queries.png "Изменить запросы")
10. В редакторе запросов в столбце **Содержимое** выберите вариант **Двоичные данные**.
    
    ![Снимок экрана редактора запросов с вызываемым столбцом содержимого.](./media/data-lake-store-power-bi/convert-query1.png "Изменить запросы")
11. Вы увидите значок, представляющий добавленный вами файл **Drivers.txt** . Щелкните файл правой кнопкой мыши и выберите пункт **CSV**.    
    
    ![Снимок экрана редактора запросов с вызываемым параметром CSV.](./media/data-lake-store-power-bi/convert-query2.png "Изменить запросы")
12. В результате вы увидите выходные данные, приведенные ниже. Данные теперь имеют формат, который можно использовать для создания визуализаций.
    
    ![Снимок экрана редактора запросов, в котором отображаются ожидаемые выходные данные.](./media/data-lake-store-power-bi/convert-query3.png "Изменить запросы")
13. На вкладке **Главная** ленты нажмите кнопку **Закрыть и применить** и выберите пункт **Закрыть и применить**.
    
    ![Снимок экрана домашней ленты с вызываемым параметром закрытия и применения.](./media/data-lake-store-power-bi/load-edited-query.png "Изменить запросы")
14. После обновления запроса на вкладке **Поля** появятся новые поля, доступные для визуализации.
    
    ![Обновленные поля](./media/data-lake-store-power-bi/updated-query-fields.png "Обновленные поля")
15. Создадим круговую диаграмму, на которой будет представлено количество водителей в каждом городе данной страны или региона. Для этого выполним указанные ниже действия.
    
    1. На вкладке "Визуализации" щелкните символ круговой диаграммы.
       
        ![Создание круговой диаграммы](./media/data-lake-store-power-bi/create-pie-chart.png "Создание круговой диаграммы")
    2. Мы собираемся использовать следующие столбцы: **Столбец 4** (название города) и **Столбец 7** (название страны или региона). Перетащите их с вкладки **Поля** на вкладку **Визуализации**, как показано ниже.
       
        ![Создание визуализаций](./media/data-lake-store-power-bi/create-visualizations.png "Создание визуализации")
    3. Круговая диаграмма теперь должна выглядеть так, как показано ниже.
       
        ![Круговая диаграмма](./media/data-lake-store-power-bi/pie-chart.png "Создание визуализации")
16. Выбрав определенную страну или регион в фильтрах уровня страницы, вы можете увидеть количество водителей в каждом городе этой страны или региона. Например, на вкладке **Визуализации** в списке **Фильтры** уровня страницы выберите **Бразилия**.
    
    ![Выбор страны или региона](./media/data-lake-store-power-bi/select-country.png "Выбор страны или региона")
17. Круговая диаграмма автоматически обновится, и вы увидите количество водителей в городах Бразилии.
    
    ![Количество водителей в стране или регионе](./media/data-lake-store-power-bi/driver-per-country.png "Количество водителей в стране или регионе")
18. В меню **Файл** выберите пункт **Сохранить**, чтобы сохранить визуализацию как файл Power BI Desktop.

## <a name="publish-report-to-power-bi-service"></a>Публикация отчета в службе Power BI
Создав визуализации в Power BI Desktop, вы можете предоставить общий доступ к ним другим пользователям, опубликовав их в службе Power BI. Инструкции см. в разделе [Публикация из Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-upload-desktop-files/).

## <a name="see-also"></a>См. также раздел
* [Анализ данных в Data Lake Storage 1-го поколения с помощью Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)

