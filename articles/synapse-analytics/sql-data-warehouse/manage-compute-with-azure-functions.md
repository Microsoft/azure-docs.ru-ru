---
title: Руководство. Управление средами вычислений с помощью функций Azure
description: Использование функций Azure для управления вычислением выделенного пула SQL (ранее — хранилища данных SQL) в Azure синапсе Analytics.
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/27/2018
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: f0731f0deaf46ec419cfe43037804e10f2b73fd4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96448387"
---
# <a name="use-azure-functions-to-manage-compute-resources-for-your-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Использование функций Azure для управления ресурсами вычислений для выделенного пула SQL (ранее — хранилища данных SQL) в Azure синапсе Analytics

В этом руководстве используются функции Azure для управления ресурсами вычислений для выделенного пула SQL (ранее — хранилища данных SQL) в Azure синапсе Analytics.

Чтобы использовать приложение-функция Azure с выделенным пулом SQL (ранее — ХРАНИЛИЩЕм данных SQL), необходимо создать [учетную запись субъекта-службы](../../active-directory/develop/howto-create-service-principal-portal.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Учетной записи субъекта-службы требуется доступ участника в той же подписке, что и выделенный экземпляр пула SQL (ранее SQL DW).

## <a name="deploy-timer-based-scaling-with-an-azure-resource-manager-template"></a>Развертывание масштабирования по таймеру с помощью шаблона Azure Resource Manager

Чтобы развернуть шаблон, понадобится следующая информация:

- Имя группы ресурсов, в которой находится выделенный экземпляр пула SQL (прежнее название — SQL DW)
- Имя сервера, в котором находится выделенный экземпляр пула SQL (прежнее название — SQL DW)
- Имя выделенного экземпляра пула SQL (прежнее название — SQL DW)
- идентификатор клиента (идентификатор каталога) Azure Active Directory;
- Идентификатор подписки
- идентификатор приложения субъекта-службы;
- секретный ключ субъекта-службы.

После получения предыдущей информации разверните этот шаблон:

[![Изображение с кнопкой "Развернуть в Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Fsql-data-warehouse-samples%2Fmaster%2Farm-templates%2FsqlDwTimerScaler%2Fazuredeploy.json)

После развертывания шаблона, необходимо найти три новых ресурса: бесплатный план обслуживания приложений Azure, план приложения-функции на основе потребления и учетную запись хранения, которая управляет журналом и очередью операций. Продолжайте изучать следующие разделы, чтобы узнать, как изменить развернутые функции в соответствии с потребностями.

## <a name="change-the-compute-level"></a>Изменение уровня вычислений

1. Перейдите к службе приложения-функции. Если вы развернули шаблон со значениями по умолчанию, эта служба должна называться *DWOperations*. Когда приложение-функция откроется, вы увидите пять развернутых функций в службе приложения-функции.

   ![Функции, развернутые с помощью шаблона](./media/manage-compute-with-azure-functions/five-functions.png)

2. Выберите *DWScaleDownTrigger* или *DWScaleUpTrigger* , чтобы увеличить или уменьшить масштаб. В раскрывающемся меню выберите пункт "Интегрировать".

   ![Выбор пункта "Интегрировать" для функции](./media/manage-compute-with-azure-functions/select-integrate.png)

3. В текущий момент должно отображаться значение *%ScaleDownTime%* или *%ScaleUpTime%*. Эти значения указывают на то, что расписание основано на значениях, определенных в [параметрах приложения](../../azure-functions/functions-how-to-use-azure-function-app-settings.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Пока что это значение можно проигнорировать и изменить время в расписании по своему усмотрению, основываясь на следующих шагах.

4. В области расписание добавьте выражение CRON, которое будет показывать, как часто требуется масштабировать службу аналитики Azure синапсе Analytics.

   ![Изменение расписания функции](./media/manage-compute-with-azure-functions/change-schedule.png)

   Значение `schedule` представляет собой [выражение CRON](https://en.wikipedia.org/wiki/Cron#CRON_expression) с шестью полями:

   ```json
   {second} {minute} {hour} {day} {month} {day-of-week}
   ```

   Например, *«0 30 9 * * 1-5»* будет отражать триггер, активирующийся каждый будний день в 9:30. Дополнительные сведения см. в [примерах расписаний](../../azure-functions/functions-bindings-timer.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#example) службы "Функции Azure".

## <a name="change-the-time-of-the-scale-operation"></a>Изменение времени операции масштабирования

1. Перейдите к службе приложения-функции. Если вы развернули шаблон со значениями по умолчанию, эта служба должна называться *DWOperations*. Когда приложение-функция откроется, вы увидите пять развернутых функций в службе приложения-функции.

2. Выберите *DWScaleDownTrigger* или *DWScaleUpTrigger* , чтобы увеличить или уменьшить масштаб значения вычислений. После выбора функций в области должен отобразиться файл *index.js*.

   ![Изменение уровня вычислений триггера функции](././media/manage-compute-with-azure-functions/index-js.png)

3. Измените значение *ServiceLevelObjective* на нужный уровень и нажмите кнопку Сохранить. *ServiceLevelObjective* — это уровень вычислений, по которому будет масштабироваться экземпляр хранилища данных в соответствии с расписанием, определенным в разделе Интеграция.

## <a name="use-pause-or-resume-instead-of-scale"></a>Использование приостановки или возобновления работы вместо масштабирования

Сейчас по умолчанию включены функции *DWScaleDownTrigger* и *DWScaleUpTrigger*. Если вместо функции масштабирования вам нужно использовать приостановку или возобновление работы, можете включить *DWPauseTrigger* или *DWResumeTrigger*.

1. Перейдите на панель "Функции".

   ![Панель "Функции"](./media/manage-compute-with-azure-functions/functions-pane.png)

2. Выберите переключатель с скользящим для соответствующих триггеров, которые вы хотите включить.

3. Перейдите на вкладки *интеграции* для соответствующих триггеров, чтобы изменить их расписание.

   > [!NOTE]
   > Функциональная разница между триггерами масштабирования и триггерами паузы или возобновления — это сообщение, отправленное в очередь. Дополнительные сведения см. в разделе [Добавление новой функции триггера](manage-compute-with-azure-functions.md#add-a-new-trigger-function).

## <a name="add-a-new-trigger-function"></a>Добавление новой функции триггера

Сейчас в шаблоне содержатся только две функции масштабирования. С помощью этих функций в течение дня можно только один раз уменьшить масштаб и один раз увеличить его. Для более детального контроля, например для уменьшения масштаба несколько раз в день или настройки другого режима масштабирования на выходные дни, нужно добавить еще один триггер.

1. Создайте пустую функцию. Нажмите *+* кнопку рядом с расположением функций, чтобы отобразить панель шаблон функции.

   ![Снимок экрана, показывающий меню "приложения функций" с значком "плюс" рядом с выбранными функциями "функции".](./media/manage-compute-with-azure-functions/create-new-function.png)

2. На языке выберите *JavaScript*, а затем выберите *TimerTrigger*.

   ![Создание функции](./media/manage-compute-with-azure-functions/timertrigger-js.png)

3. Присвойте функции имя и настройте шаблон. На рисунке показано, как активировать функцию каждую субботу в полночь (поздним вечером в пятницу).

   ![Уменьшение масштаба в субботу](./media/manage-compute-with-azure-functions/scale-down-saturday.png)

4. Скопируйте содержимое файла *index.js* из других функций триггера.

   ![Копирование содержимого файла index.js](././media/manage-compute-with-azure-functions/index-js.png)

5. Установите для переменной операции требуемое поведение следующим образом:

   ```JavaScript
   // Resume the dedicated SQL pool (formerly SQL DW) instance
   var operation = {
       "operationType": "ResumeDw"
   }

   // Pause the dedicated SQL pool (formerly SQL DW) instance
   var operation = {
       "operationType": "PauseDw"
   }

   // Scale the dedicated SQL pool (formerly SQL DW)l instance to DW600c
   var operation = {
       "operationType": "ScaleDw",
       "ServiceLevelObjective": "DW600c"
   }
   ```

## <a name="complex-scheduling"></a>Создание сложного расписания

В этом разделе кратко показано, что необходимо для более комплексного планирования возможностей паузы, возобновления и масштабирования.

### <a name="example-1"></a>Пример 1

Ежедневное увеличение масштаба в 8:00 до DW600c и уменьшение масштаба в 20:00 до DW200c.

| Компонент  | Расписание     | Операция                                |
| :-------- | :----------- | :--------------------------------------- |
| Функция 1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW600c"}` |
| Функция 2 | 0 0 20 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200c"}` |

### <a name="example-2"></a>Пример 2

Ежедневное увеличение масштаба в 8:00 до DW1000c, уменьшение масштаба до DW600 в 16:00 и масштабирование в 10pm до DW200c.

| Компонент  | Расписание     | Операция                                |
| :-------- | :----------- | :--------------------------------------- |
| Функция 1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW1000c"}` |
| Функция 2 | 0 0 16 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600c"}` |
| Функция 3 | 0 0 22 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200c"}` |

### <a name="example-3"></a>Пример 3

Увеличьте масштаб в 8:00 до DW1000c, прокрутите один раз на DW600c в 16:00 по рабочим дням. Приостановка работы в пятницу в 23:00 и возобновление работы в понедельник в 7:00.

| Компонент  | Расписание       | Операция                                |
| :-------- | :------------- | :--------------------------------------- |
| Функция 1 | 0 0 8 * * 1-5  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW1000c"}` |
| Функция 2 | 0 0 16 * * 1-5 | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600c"}` |
| Функция 3 | 0 0 23 * * 5   | `var operation = {"operationType": "PauseDw"}` |
| Функция 4 | 0 0 7 * * 1    | `var operation = {"operationType": "ResumeDw"}` |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о функциях [таймера](../../azure-functions/functions-create-scheduled-function.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) Azure.

См. раздел [хранилище примеров](https://github.com/Microsoft/sql-data-warehouse-samples)выделенного пула SQL (ранее — SQL DW).
