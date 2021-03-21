---
title: Просмотр заданий Azure Stream Analytics в Visual Studio
description: Узнайте, как просматривать, запускать и останавливаться, тестировать подключения, проверять результаты и экспортировать задания Azure Stream Analytics с помощью Visual Studio.
author: su-jie
ms.author: sujie
ms.service: stream-analytics
ms.topic: how-to
ms.date: 07/10/2018
ms.openlocfilehash: 4b568fa8d9e2fb8bd00acb27b0dc3f7c45c09c57
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98019131"
---
# <a name="use-visual-studio-to-view-azure-stream-analytics-jobs"></a>Просмотр заданий Azure Stream Analytics с помощью Visual Studio

Инструменты Azure Stream Analytics для Visual Studio позволяют разработчикам легко управлять своими заданиями Stream Analytics непосредственно из IDE. С помощью инструментов Azure Stream Analytics вы можете выполнять такие задачи:
- [Создание задания](stream-analytics-quick-create-vs.md).
- Запуск, остановка и [мониторинг заданий](stream-analytics-monitor-jobs-use-vs.md).
- Проверка результатов задания
- Экспорт имеющихся заданий в проект.
- Проверка входных и выходных подключений.
- [Локальное выполнение запросов](stream-analytics-vs-tools-local-run.md).

Узнайте, как [установить инструменты Azure Stream Analytics для Visual Studio](stream-analytics-tools-for-visual-studio-install.md).

## <a name="explore-the-job-view"></a>Изучение представления задания

С помощью представления задания можно взаимодействовать с заданиями Azure Stream Analytics из Visual Studio.

### <a name="open-the-job-view"></a>Открытие представления задания

1. В **обозревателе сервера** щелкните **Задания Stream Analytics** и нажмите кнопку **Обновить**. Ваше задание должно отобразиться в разделе **Задания Stream Analytics**.

    ![Список обозревателя серверов Stream Analytics](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-list-jobs-01.png)

2. Разверните узел задания и дважды щелкните узел **Представление заданий**, чтобы открыть представление заданий.
    
   ![Развернутый узел задания](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-job-view-01.png)

### <a name="start-and-stop-jobs"></a>Запуск и остановка заданий

Заданиями Azure Stream Analytics можно полностью управлять в представлении заданий в Visual Studio. Используйте элементы управления для запуска, остановки или удаления задания.
    
   ![Элементы управления задания Stream Analytics](./media/stream-analytics-vs-tools/azure-stream-analytics-job-view-controls.png)

## <a name="check-job-results"></a>Проверка результатов задания

Инструменты Stream Analytics для Visual Studio в настоящее время поддерживают просмотр выходных данных для хранилища Azure Data Lake Storage и хранилища BLOB-объектов. Чтобы просмотреть результат, просто дважды щелкните выходной узел схемы задания в разделе **Представление задания** и введите соответствующие учетные данные.

   ![Выходные данные большого двоичного объекта задания Stream Analytics](./media/stream-analytics-vs-tools/stream-analytics-blob-preview.png)

## <a name="export-jobs-to-a-project"></a>Экспорт заданий в проект

Существуют два способа экспортировать существующее задание в проект.

1. Щелкните правой кнопкой мыши узел заданий Stream Analytics в **обозревателе сервера**. Выберите **Экспортировать в новый проект Stream Analytics**.
    
   ![Экспорт задания в проект](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-export-job-01.png)
    
    Созданный проект появится в **обозревателе решений**.
    
   ![Обозреватель решений](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-export-job-02.png)

2. В представлении заданий выберите **Создать проект**.
    
   ![Создание проекта из представления заданий](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-export-job-03.png)

## <a name="test-connections"></a>Проверка подключения

Входные и выходные подключения можно протестировать в **представлении заданий**, выбрав параметр из раскрывающегося меню **Проверить подключение**.

   ![Раскрывающееся меню "Проверить подключение"](./media/stream-analytics-vs-tools/stream-analytics-test-connection-dropdown.png)

Результаты **проверки подключения** отображаются в окне **Выход**.

   ![Результаты проверки подключения](./media/stream-analytics-vs-tools/stream-analytics-test-connection-results.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Мониторинг заданий Stream Analytics и управление ими с помощью Visual Studio](stream-analytics-monitor-jobs-use-vs.md)
* [Краткое руководство. Создание задания Stream Analytics с использованием инструментов Azure Stream Analytics для Visual Studio](stream-analytics-quick-create-vs.md)
* [Непрерывная интеграция и разработка с помощью инструментов Stream Analytics](stream-analytics-tools-for-visual-studio-cicd.md)
