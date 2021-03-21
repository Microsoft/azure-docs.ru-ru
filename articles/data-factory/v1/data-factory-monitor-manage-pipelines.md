---
title: Мониторинг конвейеров и управление ими с помощью портал Azure и PowerShell
description: Сведения о том, как с помощью портала Azure и Azure PowerShell отслеживать состояние созданных конвейеров и фабрик данных Azure и управлять ими.
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 04/30/2018
ms.openlocfilehash: c728654e868bcb8213e6a4039fa1e2e169b0078c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100576382"
---
# <a name="monitor-and-manage-azure-data-factory-pipelines-by-using-the-azure-portal-and-powershell"></a>Мониторинг конвейеров фабрики данных Azure и управление ими с помощью портала Azure и PowerShell
> [!div class="op_single_selector"]
> * [Использование портала Azure или Azure PowerShell](data-factory-monitor-manage-pipelines.md)
> * [Использование приложения для мониторинга и управления](data-factory-monitor-manage-app.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [мониторинге и администрировании конвейеров в службе "Фабрика данных Azure"](../monitor-visually.md).

В этой статье описываются мониторинг и отладка конвейеров, а также управление ими с помощью портала Azure и PowerShell.

> [!IMPORTANT]
> Приложение для мониторинга и управления предоставляет улучшенную поддержку мониторинга и управления конвейерами данных и усовершенствованное устранение проблем. Чтобы узнать больше об этом приложении, ознакомьтесь со статьей [Мониторинг конвейеров фабрики данных Azure и управление ими с помощью приложения для мониторинга и управления](data-factory-monitor-manage-app.md). 

> [!IMPORTANT]
> Фабрика данных Azure версии 1 теперь использует новую [инфраструктуру оповещения Azure Monitor](../../azure-monitor/alerts/alerts-metric.md). Старая инфраструктура оповещения не рекомендуется к использованию. В результате существующие оповещения, настроенные для фабрик данных версии 1, больше не будут работать. Существующие оповещения для фабрик данных версии 1 не переносятся автоматически. Вы должны воссоздать эти оповещения в новой инфраструктуре оповещений. Войдите на портал Azure и выберите **Монитор**, чтобы создать оповещения о метриках (например, неудачные или успешные выполнения) для ваших фабрик данных версии 1.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="understand-pipelines-and-activity-states"></a>Состояния конвейеров и действий
Используя портал Azure, можно делать следующее:

* представлять фабрику данных в виде схемы;
* просматривать действия в конвейере;
* просматривать входные и выходные наборы данных.

В этом разделе также содержатся сведения о переходе среза набора данных из одного состояния в другое.   

### <a name="navigate-to-your-data-factory"></a>Переход к фабрике данных
1. Войдите на [портал Azure](https://portal.azure.com).
2. Щелкните колонку **Фабрики данных** в меню слева. Если вы ее не видите, выберите **Больше служб** и щелкните **Фабрики данных** в категории **Аналитика**.

   !["Просмотреть все" -> "Фабрики данных"](./media/data-factory-monitor-manage-pipelines/browseall-data-factories.png)
3. В колонке **Фабрики данных** выберите фабрику данных, которая вас интересует.

    ![Выбор фабрики данных](./media/data-factory-monitor-manage-pipelines/select-data-factory.png)

   После этого откроется домашняя страница фабрики данных.

   ![Колонка "Фабрика данных"](./media/data-factory-monitor-manage-pipelines/data-factory-blade.png)

#### <a name="diagram-view-of-your-data-factory"></a>Представление фабрики данных в виде схемы
Представление **схемы** позволяет отслеживать состояние фабрики данных и всех ее ресурсов, а также управлять ими. Щелкните **Схема** на домашней странице фабрики данных, чтобы представить ее в виде **схемы**.

![Представление схемы](./media/data-factory-monitor-manage-pipelines/diagram-view.png)

Вы можете увеличивать и уменьшать масштаб, выбирать 100%-ный масштаб или масштаб по размеру, блокировать структуру схемы и автоматически размещать конвейеры и наборы данных. Кроме того, можно отображать сведения из журнала преобразований данных (выделение восходящих и нисходящих элементов).

### <a name="activities-inside-a-pipeline"></a>Действия в конвейере
1. Щелкните конвейер правой кнопкой мыши и выберите **Открыть конвейер**. Отобразятся все действия, а также входные и выходные наборы данных. Эта функция удобна, когда конвейер состоит из нескольких действий, а вы хотите понять структуру взаимосвязей во всем конвейере.

    ![Откройте меню конвейера](./media/data-factory-monitor-manage-pipelines/open-pipeline-menu.png)     
2. В приведенном ниже примере используется действие копирования в конвейере с входными и выходными данными. 

    ![Действия в конвейере](./media/data-factory-monitor-manage-pipelines/activities-inside-pipeline.png)
3. Чтобы вернуться на домашнюю страницу фабрики данных, щелкните ссылку **Фабрика данных** в строке навигации в левом верхнем углу.

    ![Возврат к фабрике данных](./media/data-factory-monitor-manage-pipelines/navigate-back-to-data-factory.png)

### <a name="view-the-state-of-each-activity-inside-a-pipeline"></a>Просмотр состояния действий в конвейере
Вы можете просмотреть текущее состояние отдельного действия, отобразив состояние любого созданного им набора данных.

Чтобы отобразить все срезы, созданные в рамках отдельных циклов выполнения действия в конвейере, дважды щелкните **OutputBlobTable** в представлении **Схема**. Как видите, действие копирования успешно выполнялось последние восемь часов и создало срезы в состоянии **Готово**.  

![Состояние конвейера](./media/data-factory-monitor-manage-pipelines/state-of-pipeline.png)

Срезы наборов данных в фабрике данных могут находиться в одном из следующих состояний:

<table>
<tr>
    <th align="left">Состояние</th><th align="left">Подсостояние</th><th align="left">Описание</th>
</tr>
<tr>
    <td rowspan="8">Ожидание</td><td>ScheduleTime</td><td>Время для выполнения среза еще не пришло.</td>
</tr>
<tr>
<td>DatasetDependencies</td><td>Восходящие зависимости не готовы.</td>
</tr>
<tr>
<td>ComputeResources</td><td>Вычислительные ресурсы недоступны.</td>
</tr>
<tr>
<td>ConcurrencyLimit</td> <td>Все экземпляры действия заняты выполнением других срезов.</td>
</tr>
<tr>
<td>ActivityResume</td><td>Действие приостановлено, и до его возобновления выполнять срезы нельзя.</td>
</tr>
<tr>
<td>Повторить попытку</td><td>Действие выполняется повторно.</td>
</tr>
<tr>
<td>Проверка</td><td>Проверка еще не начата.</td>
</tr>
<tr>
<td>ValidationRetry</td><td>Ожидание повторения проверки.</td>
</tr>
<tr>
<tr>
<td rowspan="2">InProgress</td><td>Validating</td><td>Проверка выполняется.</td>
</tr>
<td>-</td>
<td>Срез обрабатывается.</td>
</tr>
<tr>
<td rowspan="4">Сбой</td><td>TimedOut</td><td>Выполнение действия заняло больше времени, чем разрешено для данного действия.</td>
</tr>
<tr>
<td>Отменено</td><td>Срез был отменен пользователем.</td>
</tr>
<tr>
<td>Проверка</td><td>Сбой проверки.</td>
</tr>
<tr>
<td>-</td><td>Не удалось создать и/или проверить срез.</td>
</tr>
<td>Ready</td><td>-</td><td>Срез готов к использованию.</td>
</tr>
<tr>
<td>Пропущено</td><td>Нет</td><td>Срез не обрабатывается.</td>
</tr>
<tr>
<td>Нет</td><td>-</td><td>Срез, который ранее существовал с другим состоянием, но был сброшен.</td>
</tr>
</table>



Чтобы просмотреть сведения о срезе, щелкните запись среза в колонке **Последние обновленные срезы**.

![Сведения о срезе](./media/data-factory-monitor-manage-pipelines/slice-details.png)

Если срез выполнялся несколько раз, вы увидите несколько строк в списке **Выполнения действий** . Чтобы просмотреть сведения о выполнении действия, щелкните запись цикла в списке **Циклы выполнения действия** . Отобразятся все файлы журналов и сообщения об ошибках, если таковые были. Эта функция удобна для просмотра и обработки файлов журналов непосредственно из фабрики данных.

![СВЕДЕНИЯ О ВЫПОЛНЕННОМ ДЕЙСТВИИ](./media/data-factory-monitor-manage-pipelines/activity-run-details.png)

Если срез не находится в состоянии **Готово**, вы можете увидеть восходящие срезы, которые не находятся в состоянии готовности и блокируют выполнение текущего среза в списке **Неготовые восходящие срезы**. Эта функция удобна для просмотра восходящих зависимостей, если срез находится в состоянии **Ожидание**.

![Неготовые восходящие срезы](./media/data-factory-monitor-manage-pipelines/upstream-slices-not-ready.png)

### <a name="dataset-state-diagram"></a>Схема состояний наборов данных
После развертывания фабрики данных и определения периода активности конвейера срезы наборов данных переходят из одного состояния в другое. В настоящее время переходы между состояниями выполняются в соответствии со следующей схемой:

![Схема состояний](./media/data-factory-monitor-manage-pipelines/state-diagram.png)

Поток переходов между состояниями выглядит так: "Ожидание" -> "Выполняется" или "Выполняется (проверка)" -> "Готово" или "Сбой".

Изначально срезы находятся в состоянии **Ожидание**, ожидая, что предварительные условия будут соблюдены до выполнения действий. Затем начинается выполнение действия, и срез переходит в состояние **Выполняется**. Выполнение действия может завершиться успешно или с ошибкой. В зависимости от того, как завершится действие, срез перейдет в состояние **Готово** или **Сбой**.

Вы можете сбросить состояние среза **Готово** или **Сбой** обратно в состояние **Ожидание**. Вы также можете установить для среза флажок **Пропустить** — действие со срезом не выполняется, и срез не обрабатывается.

## <a name="pause-and-resume-pipelines"></a>Приостановка и возобновление работы конвейеров
Управлять конвейерами можно с помощью Azure Powershell. Например, вы можете приостановить и возобновить работу конвейеров, используя командлеты Azure PowerShell. 

> [!NOTE] 
> Представление схемы не поддерживает приостановку и возобновление конвейеров. Если вы хотите использовать пользовательский интерфейс, используйте приложение для мониторинга и управления. Чтобы узнать больше об этом приложении, ознакомьтесь со статьей [Мониторинг конвейеров фабрики данных Azure и управление ими с помощью приложения для мониторинга и управления](data-factory-monitor-manage-app.md). 

Приостанавливать и приостанавливать конвейеры можно с помощью командлета PowerShell **Suspend-аздатафакторипипелине** . Этот командлет полезен, если не требуется запускать конвейеры до устранения проблемы. 

```powershell
Suspend-AzDataFactoryPipeline [-ResourceGroupName] <String> [-DataFactoryName] <String> [-Name] <String>
```
Пример:

```powershell
Suspend-AzDataFactoryPipeline -ResourceGroupName ADF -DataFactoryName productrecgamalbox1dev -Name PartitionProductsUsagePipeline
```

Устранив проблему в конвейере, возобновите его работу. Для этого выполните следующую команду PowerShell:

```powershell
Resume-AzDataFactoryPipeline [-ResourceGroupName] <String> [-DataFactoryName] <String> [-Name] <String>
```
Пример:

```powershell
Resume-AzDataFactoryPipeline -ResourceGroupName ADF -DataFactoryName productrecgamalbox1dev -Name PartitionProductsUsagePipeline
```

## <a name="debug-pipelines"></a>Отладка конвейеров
Фабрика данных Azure предоставляет широкие возможности отладки и устранения неполадок с конвейерами с помощью портала Azure и Azure PowerShell.

> [!NOTE] 
> Гораздо проще устранять ошибки и проблемы с помощью приложения для мониторинга и управления. Чтобы узнать больше об этом приложении, ознакомьтесь со статьей [Мониторинг конвейеров фабрики данных Azure и управление ими с помощью приложения для мониторинга и управления](data-factory-monitor-manage-app.md). 

### <a name="find-errors-in-a-pipeline"></a>Поиск ошибок в конвейере
Если при выполнении действия в конвейере происходит сбой, созданный конвейером набор данных будет находиться в состоянии "Ошибка". Вы можете устранить неполадки или выполнить отладку в фабрике данных Azure, используя следующие методы.

#### <a name="use-the-azure-portal-to-debug-an-error"></a>Отладка ошибок с помощью портала Azure
1. В колонке **Таблица** щелкните проблемный срез с **состоянием****Сбой**.

   ![Колонка "Таблица" с проблемным срезом](./media/data-factory-monitor-manage-pipelines/table-blade-with-error.png)
2. В колонке **Срез данных** щелкните цикл выполнения действия, который завершился ошибкой.

   ![Срез данных с ошибкой](./media/data-factory-monitor-manage-pipelines/dataslice-with-error.png)
3. В колонке **Подробности о выполнении операции** вы можете скачать файлы, связанные с обработкой HDInsight. Чтобы скачать файл журнала ошибок c подробными сведениями об ошибке, щелкните **Скачать** в строке журнала Status/stderr.

   ![Колонка "Сведения о циклах выполнения действия" с ошибкой](./media/data-factory-monitor-manage-pipelines/activity-run-details-with-error.png)     

#### <a name="use-powershell-to-debug-an-error"></a>Отладка ошибок с помощью PowerShell
1. Запустите **PowerShell**.
2. Выполните команду **Get-аздатафакторислице** , чтобы просмотреть срезы и их состояния. Вы должны увидеть срез с состоянием **Сбой**.        

    ```powershell   
    Get-AzDataFactorySlice [-ResourceGroupName] <String> [-DataFactoryName] <String> [-DatasetName] <String> [-StartDateTime] <DateTime> [[-EndDateTime] <DateTime> ] [-Profile <AzureProfile> ] [ <CommonParameters>]
    ```   
   Пример:

    ```powershell   
    Get-AzDataFactorySlice -ResourceGroupName ADF -DataFactoryName LogProcessingFactory -DatasetName EnrichedGameEventsTable -StartDateTime 2014-05-04 20:00:00
    ```

   Замените **StartDateTime** на время запуска конвейера. 
3. Теперь выполните командлет **Get-аздатафакторирун** , чтобы получить сведения о выполнении действия для среза.

    ```powershell   
    Get-AzDataFactoryRun [-ResourceGroupName] <String> [-DataFactoryName] <String> [-DatasetName] <String> [-StartDateTime]
    <DateTime> [-Profile <AzureProfile> ] [ <CommonParameters>]
    ```

    Пример:

    ```powershell   
    Get-AzDataFactoryRun -ResourceGroupName ADF -DataFactoryName LogProcessingFactory -DatasetName EnrichedGameEventsTable -StartDateTime "5/5/2014 12:00:00 AM"
    ```

    В качестве значения StartDateTime укажите время начала для проблемного (содержащего ошибку) среза, которое вы отметили на предыдущем шаге. Значение даты-времени необходимо заключить в двойные кавычки.
4. Вы должны увидеть выходные данные с подробными сведениями об ошибке, аналогичные следующим:

    ```   
    Id                      : 841b77c9-d56c-48d1-99a3-8c16c3e77d39
    ResourceGroupName       : ADF
    DataFactoryName         : LogProcessingFactory3
    DatasetName               : EnrichedGameEventsTable
    ProcessingStartTime     : 10/10/2014 3:04:52 AM
    ProcessingEndTime       : 10/10/2014 3:06:49 AM
    PercentComplete         : 0
    DataSliceStart          : 5/5/2014 12:00:00 AM
    DataSliceEnd            : 5/6/2014 12:00:00 AM
    Status                  : FailedExecution
    Timestamp               : 10/10/2014 3:04:52 AM
    RetryAttempt            : 0
    Properties              : {}
    ErrorMessage            : Pig script failed with exit code '5'. See wasb://        adfjobs@spestore.blob.core.windows.net/PigQuery
                                    Jobs/841b77c9-d56c-48d1-99a3-
                8c16c3e77d39/10_10_2014_03_04_53_277/Status/stderr' for
                more details.
    ActivityName            : PigEnrichLogs
    PipelineName            : EnrichGameLogsPipeline
    Type                    :
    ```
5. Вы можете запустить командлет **Save-аздатафакторилог** со значением идентификатора, отображаемым на выходе, и скачать файлы журнала с помощью командлета **-DownloadLogsoption** .

    ```powershell
    Save-AzDataFactoryLog -ResourceGroupName "ADF" -DataFactoryName "LogProcessingFactory" -Id "841b77c9-d56c-48d1-99a3-8c16c3e77d39" -DownloadLogs -Output "C:\Test"
    ```

## <a name="rerun-failures-in-a-pipeline"></a>Повторная обработка проблемных срезов в конвейере

> [!IMPORTANT]
> Использование приложения для мониторинга и управления облегчает устранение ошибок и перезапуск неудачных срезов. Чтобы узнать больше об этом приложении, ознакомьтесь со статьей [Мониторинг конвейеров фабрики данных Azure и управление ими с помощью приложения для мониторинга и управления](data-factory-monitor-manage-app.md). 

### <a name="use-the-azure-portal"></a>Использование портала Azure
После устранения неполадок и отладки ошибок в конвейере можно повторно обработать срез с ошибками. Для этого выберите срез и на панели команд нажмите кнопку **Запустить**.

![Повторная обработка среза](./media/data-factory-monitor-manage-pipelines/rerun-slice.png)

Если срез не прошел проверку из-за ошибки политики (например, данные недоступны), вы можете исправить ошибку и повторно выполнить проверку, нажав на панели команд кнопку **Проверить**.

![Исправление ошибок и проверка](./media/data-factory-monitor-manage-pipelines/fix-error-and-validate.png)

### <a name="use-azure-powershell"></a>Использование Azure PowerShell
Ошибки можно повторно запустить с помощью командлета **Set-аздатафакторислицестатус** . Синтаксис и другие сведения о командлете см. в разделе [Set-аздатафакторислицестатус](/powershell/module/az.datafactory/set-azdatafactoryslicestatus) .

**Пример**.

В следующем примере состояние всех срезов в таблице DAWikiAggregatedData в фабрике данных WikiADF меняется на Waiting.

Для UpdateType задано значение UpstreamInPipeline. Это означает, что состояние каждого среза в таблице и всех зависимых (восходящих) таблиц переходит в состояние Waiting. Этот параметр может иметь еще одно значение — Individual.

```powershell
Set-AzDataFactorySliceStatus -ResourceGroupName ADF -DataFactoryName WikiADF -DatasetName DAWikiAggregatedData -Status Waiting -UpdateType UpstreamInPipeline -StartDateTime 2014-05-21T16:00:00 -EndDateTime 2014-05-21T20:00:00
```
## <a name="create-alerts-in-the-azure-portal"></a>Создание оповещений на портале Azure

1.  Войдите на портал Azure и выберите **Монитор -> Оповещения**, чтобы открыть страницу оповещений.

    ![Откройте страницу "Оповещения".](media/data-factory-monitor-manage-pipelines/v1alerts-image1.png)

2.  Нажмите кнопку **+ Новое правило генерации оповещений**, чтобы создать оповещение.

    ![Создание оповещения](media/data-factory-monitor-manage-pipelines/v1alerts-image2.png)

3.  Определите **условия оповещения**. (Не забудьте выбрать **фабрики данных** в поле **Фильтр по типу ресурса** .) Можно также указать значения для **измерений**.

    ![Определение условия оповещения — выбор цели](media/data-factory-monitor-manage-pipelines/v1alerts-image3.png)

    ![Определение условия оповещения — добавление критерия оповещения](media/data-factory-monitor-manage-pipelines/v1alerts-image4.png)

    ![Определение условия оповещения — добавление логики оповещения](media/data-factory-monitor-manage-pipelines/v1alerts-image5.png)

4.  Укажите **сведения о предупреждении**.

    ![Определение сведений об оповещении](media/data-factory-monitor-manage-pipelines/v1alerts-image6.png)

5.  Определите **группу действий**.

    ![Определение группы действий — создание группы действий](media/data-factory-monitor-manage-pipelines/v1alerts-image7.png)

    ![Определение группы действий — настройка свойств](media/data-factory-monitor-manage-pipelines/v1alerts-image8.png)

    ![Определение группы действий — создана группа действий](media/data-factory-monitor-manage-pipelines/v1alerts-image9.png)

## <a name="move-a-data-factory-to-a-different-resource-group-or-subscription"></a>Перемещение фабрики данных в другую группу ресурсов или подписку
Фабрику данных можно переместить в другую группу ресурсов или подписку, воспользовавшись кнопкой **Переместить** в командной строке на домашней странице фабрики данных.

![Перемещение фабрики данных](./media/data-factory-monitor-manage-pipelines/MoveDataFactory.png)

Вместе с фабрикой данных можно переместить все связанные ресурсы (например, связанные с ней оповещения).

![Диалоговое окно "Перемещение ресурсов"](./media/data-factory-monitor-manage-pipelines/MoveResources.png)