---
title: Руководство по началу работы с Azure Synapse Analytics. Визуализация данных рабочей области с помощью Power BI
description: В этом учебнике вы узнаете, как создать рабочую область Power BI, привязать к ней рабочую область Azure Synapse и создать набор данных Power BI, использующий данные из рабочей области Azure Synapse.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: business-intelligence
ms.topic: tutorial
ms.date: 03/25/2021
ms.openlocfilehash: 829daaa9c739657528a9085201c61c88635931d0
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105608448"
---
# <a name="visualize-data-with-power-bi"></a>Визуализация данных с помощью Power BI

В этом учебнике вы узнаете, как создать рабочую область Power BI, привязать к ней рабочую область Azure Synapse и создать набор данных Power BI, использующий данные из вашей рабочей области Azure Synapse. 

## <a name="prerequisites"></a>Предварительные требования
Для работы с этим руководством [установите приложение Power BI Desktop](https://aka.ms/pbidesktopstore).

## <a name="overview"></a>Обзор

Из данных нью-йоркского такси мы создали агрегированные наборы данных в двух таблицах:
- **nyctaxi.passengercountstats**
- **SQLDB1.dbo.PassengerCountStats**

Рабочую область Power BI можно связать с рабочей областью Azure Synapse. Это позволяет легко передавать данные в рабочую область Power BI. Вы можете изменить отчеты Power BI непосредственно в рабочей области Azure Synapse. 

### <a name="create-a-power-bi-workspace"></a>Создание рабочей области Power BI

1. Войдите на портал [powerbi.microsoft.com](https://powerbi.microsoft.com/).
1. Щелкните **Рабочие области** и выберите **Создать рабочую область**. Создайте рабочую область Power BI с именем **NYCTaxiWorkspace1** или похожим уникальным именем.

### <a name="link-your-azure-synapse-workspace-to-your-new-power-bi-workspace"></a>Связывание рабочей области Azure Synapse с новой рабочей областью Power BI

1. В Synapse Studio последовательно выберите команду **Управление** > **Cвязанные службы**.
1. Выберите **Создать** > **Подключение к Power BI**.
1. В поле **Имя** укажите **NYCTaxiWorkspace1**.
1. В поле **Имя рабочей области** укажите имя созданной ранее рабочей области Power BI, аналогично **NYCTaxiWorkspace1**.
1. Нажмите кнопку **создания**.

### <a name="create-a-power-bi-dataset-that-uses-data-in-your-azure-synapse-workspace"></a>Создание набора данных Power BI, использующего данные в рабочей области Azure Synapse

1. В Synapse Studio перейдите в **Разработка** > **Power BI**.
1. Перейдите в раздел **NYCTaxiWorkspace1** > **Наборы данных Power BI** и выберите **Новый набор данных Power BI**. Щелкните **Запуск**.
1. Выберите источник данных **SQLPOOL1** и нажмите **Продолжить**.
1. Щелкните **Скачать** , чтобы скачать файл PBIDS для файла **NYCTaxiWorkspace1SQLPOOL1.pbids**. Нажмите кнопку **Continue**(Продолжить).
1. Откройте загруженный файл **.pbids**. При этом запустится приложение Power BI Desktop, которое автоматически подключится к **SQLDB1** в рабочей области Azure Synapse.
1. Если откроется диалоговое окно **База данных SQL Server**:
    1. Выберите **Учетная запись Майкрософт**.
    1. Выберите **Войти** и войдите в свою учетную запись.
    1. Выберите **Подключиться**.
1. После того как откроется диалоговое окно **Навигатор**, проверьте таблицу **PassengerCountStats** и выберите команду **Загрузить**.
1. После того как откроется диалоговое окно **Параметры подключения**, выберите команду **DirectQuery** > **ОК**.
1. Нажмите кнопку **Отчет** слева.
1. В разделе **Визуализации** щелкните значок графика, чтобы добавить **график** в отчет.
    1. В разделе **Поля** перетащите столбец **PassengerCount** в **Визуализации** > **Ось**.
    1. Перетащите столбцы **SumTripDistance** и **AvgTripDistance** в **Визуализации** > **Значения**.
1. На вкладке **Главная** нажмите кнопку **Опубликовать**.
1. Выберите **Сохранить**, чтобы сохранить изменения.
1. Выберите имя файла **PassengerAnalysis.pbix**, а затем выберите команду **Сохранить**.
1. В окне **Публикация в Power BI** в разделе **Выбор конечной папки** выберите **NYCTaxiWorkspace1**, а затем щелкните **Выбрать**.
1. Дождитесь завершения публикации. 

### <a name="configure-authentication-for-your-dataset"></a>Настройка проверки подлинности для набора данных

1. Откройте [powerbi.microsoft.com](https://powerbi.microsoft.com/) и выполните **Вход**.
1. В левой части в разделе **Рабочие области** выберите рабочую область **NYCTaxiWorkspace1**.
1. В этой рабочей области выберите набор данных с именем **Анализ пассажиров** и отчет с именем **Анализ пассажиров**.
1. Наведите указатель мыши на набор данных **PassengerAnalysis**, нажмите кнопку с многоточием (...), а затем выберите команду **Параметры**.
1. В разделе **Учетные данные источника данных** выберите **Правка**, задайте для параметра **Метод аутентификации** значение **OAuth2** и щелкните **Вход**.

### <a name="edit-a-report-in-synapse-studio"></a>Изменение отчета в Synapse Studio

1. Вернитесь в Synapse Studio и выберите **Закрыть и обновить**.
1. Перейдите в центр **Разработка**.
1. Справа от слоя **Power BI** щелкните многоточие (...) и **Обновить**, чтобы обновить узел **Отчеты Power BI**.
1. Теперь в **Power BI** должны отобразиться:
    * в разделе **NYCTaxiWorkspace1** > **Наборы данных Power BI** новый набор с именем **PassengerAnalysis**.
    * В разделе **NYCTaxiWorkspace1** > **Отчеты Power BI**, новый отчет с именем **PassengerAnalysis**.
1. Выберите отчет **PassengerAnalysis**. Отчет откроется, и его можно изменить непосредственно в Synapse Studio.



## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Мониторинг](get-started-monitor.md)
                                 

