---
title: Как запланировать Azure-SSIS Integration Runtime
description: В этой статье описан процесс планирования запуска и остановки Azure-SSIS Integration Runtime с использованием фабрики данных.
ms.service: data-factory
ms.devlang: powershell
ms.topic: conceptual
ms.date: 07/09/2020
author: swinarko
ms.author: sawinark
ms.openlocfilehash: 40c3b8ab228c93fd4c681281d89d16f88ddf30f1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100384373"
---
# <a name="how-to-start-and-stop-azure-ssis-integration-runtime-on-a-schedule"></a>Запуск и остановка Azure-SSIS Integration Runtime по расписанию

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этой статье описан процесс планирования запуска и остановки Azure-SSIS Integration Runtime (IR) с использованием Фабрики данных Azure (ADF). Azure-SSIS IR представляет собой вычислительный ресурс ADF, предназначенный для выполнения пакетов SQL Server Integration Services (SSIS). Выполнение Azure-SSIS IR сопряжено с определенными затратами. Поэтому среду выполнения интеграции обычно запускают только на тот период, когда требуется выполнение пакетов SSIS в Azure, а затем останавливают. С помощью пользовательского интерфейса (приложения) ADF или Azure PowerShell вы можете [запустить или остановить среду выполнения интеграции вручную](manage-azure-ssis-integration-runtime.md)).

Кроме того, можно создать в конвейерах ADF веб-действия, которые будут запускать и останавливать IR по расписанию. Например, вы можете запускать среду утром перед выполнением регулярных рабочих нагрузок ETL и останавливать ее вечером, когда выполнение рабочих нагрузок завершится.  Вы можете добавить действие выполнения пакета SSIS в цепочку между двумя веб-действиями, которые запускают и останавливают IR, чтобы эта среда выполнения интеграции запускалась и останавливалась по требованию ровно тогда, когда она потребуется для выполнения пакета. Дополнительные сведения о действии "Выполнить пакет SSIS" см. в статье [Запуск пакета Integration Services с помощью действия "Выполнить пакет SSIS" в фабрике данных Azure](how-to-invoke-ssis-package-ssis-activity.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия
Если среда Azure-SSIS IR еще не подготовлена, выполните подготовку по указаниям из [этого руководства](./tutorial-deploy-ssis-packages-azure.md). 

## <a name="create-and-schedule-adf-pipelines-that-start-and-or-stop-azure-ssis-ir"></a>Создание и планирование конвейеров ADF для запуска и остановки Azure-SSIS IR
Из этого раздела вы узнаете, как использовать веб-действия в конвейерах ADF для запуска и остановки Azure-SSIS IR по расписанию или в любой момент по требованию. Мы поможем вам создать три конвейера. 

1. Первый конвейер содержит веб-действие запуска Azure-SSIS IR. 
2. Второй конвейер содержит веб-действие остановки Azure-SSIS IR.
3. Третий конвейер содержит действие "Выполнить пакет SSIS", размещенное в цепочке между двумя веб-действиями для запуска и остановки Azure-SSIS IR. 

Завершив создание и тестирование конвейеров, вы сможете создать триггер расписания и связать его с любым из конвейеров. Триггер расписания определяет расписание для выполнения соответствующего конвейера. 

Например, можно создать два триггера, первый из которых запускается ежедневно в 06:00 и связан с первым конвейером, а второй запланирован на выполнение каждый день в 18:00 и связан со вторым конвейером.  В таком сценарии ежедневно в период между 6:00 и 18:00 будет работать среда выполнения интеграции, готовая к выполнению ежедневных рабочих нагрузок ETL.  

Также вы можете создать третий триггер, запланировать его на ежедневный запуск в полночь и связать его с третьим конвейером. Тогда этот конвейер будет выполняться ежедневно в полночь, запуская среду выполнения интеграции перед выполнением пакета и останавливая ее работу сразу же после выполнения пакета, чтобы среда выполнения интеграции не работала, когда это не требуется.

### <a name="create-your-adf"></a>Создание ADF

1. Войдите на [портал Azure](https://portal.azure.com/).    
2. В меню слева щелкните **Создать**, выберите **Данные+аналитика** и щелкните **Фабрика данных**. 
   
   ![Создать -> Фабрика данных](./media/tutorial-create-azure-ssis-runtime-portal/new-data-factory-menu.png)
   
3. На странице **Новая фабрика данных** введите **MyAzureSsisDataFactory** в поле **Имя**. 
      
   ![Страница "Новая фабрика данных"](./media/tutorial-create-azure-ssis-runtime-portal/new-azure-data-factory.png)
 
   Имя ADF должно быть глобально уникальным. Если вы получите указанное ниже сообщение об ошибке, введите другое имя ADF (например, [ваше_имя]MyAzureSsisDataFactory) и попробуйте создать фабрику данных снова. Ознакомьтесь со статьей [Фабрика данных Azure — правила именования](naming-rules.md), чтобы узнать правила именования для артефактов ADF.
  
   `Data factory name MyAzureSsisDataFactory is not available`
      
4. Выберите **подписку** Azure, в которой нужно создать ADF. 
5. Для **группы ресурсов** выполните одно из следующих действий.
     
   - Выберите **Использовать существующую** и укажите существующую группу ресурсов в раскрывающемся списке. 
   - Выберите **Создать новую** и укажите имя новой группы ресурсов.   
         
   Дополнительные сведения см. в статье [об использовании групп ресурсов для управления ресурсами Azure](../azure-resource-manager/management/overview.md).
   
6. Для параметра **Версия** укажите значение **V2**.
7. Для параметра **Расположение** выберите из раскрывающегося списка одно из расположений, в которых поддерживается создание ADF.
8. Кроме того, установите флажок **Закрепить на панели мониторинга**.     
9. Нажмите кнопку **Создать**.
10. На панели мониторинга Azure вы увидите следующую плитку с состоянием " **развертывание фабрики данных**". 

    ![Элемент Deploying data factory (Развертывание фабрики данных)](media/tutorial-create-azure-ssis-runtime-portal/deploying-data-factory.png)
   
11. Когда создание завершится, вы увидите страницу ADF, которая представлена ниже.
   
    ![Домашняя страница фабрики данных](./media/tutorial-create-azure-ssis-runtime-portal/data-factory-home-page.png)
   
12. Щелкните **Создание и мониторинг**, чтобы запустить в отдельной вкладке пользовательский интерфейс (приложение) ADF.

### <a name="create-your-pipelines"></a>Создание конвейеров

1. На странице **Начало работы** выберите **Создать конвейер**. 

   ![Страница "Начало работы"](./media/how-to-schedule-azure-ssis-integration-runtime/get-started-page.png)
   
2. На панели элементов **Действия** разверните меню **Общие** и перетащите **веб-действие** в область конструктора конвейера. На вкладке **Общие** в окне свойств действия измените имя действия на **startMyIR**. Перейдите на вкладку **Настройки** и выполните здесь следующие действия:

    1. В поле **URL-адрес** введите следующий URL-адрес для REST API, который начинается Azure-SSIS IR, заменяя `{subscriptionId}` ,, `{resourceGroupName}` `{factoryName}` и `{integrationRuntimeName}` фактические значения для вашего IR: `https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories/{factoryName}/integrationRuntimes/{integrationRuntimeName}/start?api-version=2018-06-01` Кроме того, можно также скопировать & вставьте идентификатор ресурса IR со страницы мониторинга в пользовательском интерфейсе или приложении ADF, чтобы заменить следующую часть приведенного выше URL-адреса: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories/{factoryName}/integrationRuntimes/{integrationRuntimeName}`
    
       ![Идентификатор ресурса Azure-SSIS IR](./media/how-to-schedule-azure-ssis-integration-runtime/adf-ssis-ir-resource-id.png)
  
    2. Для поля **Метод** выберите **POST**. 
    3. В поле **Текст** введите `{"message":"Start my IR"}`. 
    4. Для **проверки подлинности** выберите **MSI** для использования управляемого удостоверения для ADF. Дополнительные сведения см. в статье [управляемое удостоверение для фабрики данных](./data-factory-service-identity.md) .
    5. В поле **Ресурс** введите `https://management.azure.com/`.
    
       ![Расписание веб-действия ADF для Azure-SSIS IR](./media/how-to-schedule-azure-ssis-integration-runtime/adf-web-activity-schedule-ssis-ir.png)
  
3. Клонируйте первый конвейер, чтобы создать второй, присвоив ему имя действия **stopMyIR** и изменив свойства, как описано ниже.

    1. В поле **URL-адрес** введите следующий URL-адрес для REST API, который останавливает Azure-SSIS IR, заменяя `{subscriptionId}` ,, `{resourceGroupName}` `{factoryName}` и `{integrationRuntimeName}` фактические значения для своего IR: `https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories/{factoryName}/integrationRuntimes/{integrationRuntimeName}/stop?api-version=2018-06-01`
    
    2. В поле **Текст** введите `{"message":"Stop my IR"}`. 

4. Создайте третий конвейер, перетащите в конструктор конвейера действие **Выполнение пакета SSIS** из панели инструментов **Действия**, а затем настройте его по инструкциям из статьи [Запуск пакета Integration Services с помощью действия "Выполнить пакет SSIS" в фабрике данных Azure](how-to-invoke-ssis-package-ssis-activity.md).  Вместо этого вы можете применить действие **Хранимая процедура** и настроить его по инструкциям из статьи [Запуск в Фабрике данных Azure пакета SQL Server Integration Services с помощью действия хранимой процедуры](how-to-invoke-ssis-package-stored-procedure-activity.md).  Теперь создайте вокруг действия "Выполнение пакета SSIS" или "Хранимая процедура" цепочку с двумя веб-действиями для запуска и остановки IR, (так же, как веб-действия в первом и втором конвейерах).

   ![Веб-действие ADF для запуска Azure-SSIS IR по запросу](./media/how-to-schedule-azure-ssis-integration-runtime/adf-web-activity-on-demand-ssis-ir.png)

5. Назначьте управляемому удостоверению ADF роль **участника** для этой же фабрики данных, чтобы веб-действия в ее конвейерах могли вызывать REST API для запуска и остановки подготовленной в ней Azure-SSIS IR.  На странице ADF на портале Azure щелкните **Управление доступом (IAM)**, затем **+ Добавить назначение ролей** и выполните описанные ниже действия в колонке **Добавить назначение ролей**.

    1. В поле **Роль** выберите **Участник**. 
    2. В поле **Назначение доступа к** выберите **Пользователь, группа или субъект-служба Azure AD**. 
    3. В поле **Выберите** найдите имя нужной ADF и выберите его. 
    4. Выберите команду **Сохранить**.
    
   ![Назначение роли управляемому удостоверению ADF](./media/how-to-schedule-azure-ssis-integration-runtime/adf-managed-identity-role-assignment.png)

6. Проверьте параметры ADF и всех конвейеров, щелкнув действие **Проверить все** (или "Проверить") на панели инструментов фабрики или конвейера. Закройте **выходные данные проверки** для фабрики или конвейера, нажав кнопку **>>**.  

   ![Проверка конвейера](./media/how-to-schedule-azure-ssis-integration-runtime/validate-pipeline.png)

### <a name="test-run-your-pipelines"></a>Тестовый запуск конвейеров

1. Выберите **Тестовый запуск** на панели инструментов для каждого конвейера и проверьте данные в **окне вывода** на панели снизу. 

   ![Тестовый запуск](./media/how-to-schedule-azure-ssis-integration-runtime/test-run-output.png)
    
2. Чтобы протестировать третий конвейер, запустите SQL Server Management Studio (SSMS). В окне **Подключение к серверу** сделайте следующее: 

    1. В качестве **имени сервера** введите **&lt; имя сервера &gt; . Database.Windows.NET**.
    2. Выберите **Параметры >>**.
    3. В поле **Подключение к базе данных** выберите **SSISDB**.
    4. Выберите **Подключиться**. 
    5. Разверните папку **Integration Services каталоги**  ->  **SSISDB** -> > **проекты** , > свои **пакеты**> проекта SSIS. 
    6. Щелкните правой кнопкой мыши указанный пакет служб SSIS для запуска и выберите пункт **отчеты**  ->  **Стандартные отчеты**  ->  **все выполнения**. 
    7. Убедитесь, что конвейер выполнен. 

   ![Проверка запуска пакета SSIS](./media/how-to-schedule-azure-ssis-integration-runtime/verify-ssis-package-run.png)

### <a name="schedule-your-pipelines"></a>Расписание конвейеров

Теперь, когда конвейеры работают так, как нам нужно, можно создать триггеры для их запуска с нужной частотой. Дополнительные сведения о связывании триггеров с конвейером см. в статье [Запуск конвейера по расписанию](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule).

1. На панели инструментов для конвейера выберите **Триггер** и **New/Edit** (Создать или изменить). 

   ![Снимок экрана, посвященный пункту меню триггера-> создать/изменить.](./media/how-to-schedule-azure-ssis-integration-runtime/trigger-new-menu.png)

2. На панели **Add Triggers** (Добавить триггеры) выберите **+ Создать**.

   !["Добавление триггеров" — "Создать"](./media/how-to-schedule-azure-ssis-integration-runtime/add-triggers-new.png)

3. На панели **Новый триггер** сделайте следующее: 

    1. В поле **Имя** введите имя триггера. В приведенном ниже примере триггеру присвоено имя **Запускать ежедневно**. 
    2. В поле **Тип** выберите **Расписание**. 
    3. В поле **Дата начала (UTC)** выберите дату и время начала в формате UTC. 
    4. В поле **Повторение** укажите периодичность активации триггера. В следующем примере выбран вариант **Ежедневно**. 
    5. В поле **Конец** выберите вариант **Нет конца** или введите дату окончания и время, выбрав вариант **Дата**. 
    6. Выберите **Активировано**, чтобы запустить триггер сразу после публикации этих параметров ADF. 
    7. Выберите **Далее**.

   !["Триггер" > "Создать/изменить"](./media/how-to-schedule-azure-ssis-integration-runtime/new-trigger-window.png)
    
4. Изучите предупреждение на странице **Trigger Run Parameters** (Параметры выполнения триггера) и выберите **Готово**. 
5. Опубликуйте параметры ADF, выбрав действие **Опубликовать все** на панели инструментов фабрики. 

   ![Опубликовать все](./media/how-to-schedule-azure-ssis-integration-runtime/publish-all.png)

### <a name="monitor-your-pipelines-and-triggers-in-azure-portal"></a>Мониторинг конвейеров и триггеров на портале Azure

1. Чтобы отслеживать запуски триггеров и конвейеров, используйте вкладку **Мониторинг** в левой части пользовательского интерфейса (приложения) ADF. Подробные инструкции см. в разделе [о мониторинге конвейера](quickstart-create-data-factory-portal.md#monitor-the-pipeline).

   ![Запуски конвейера](./media/how-to-schedule-azure-ssis-integration-runtime/pipeline-runs.png)

2. Чтобы просмотреть сведения о выполнении действий, связанных с запуском конвейера, щелкните первую ссылку (**View Activity Runs** (Просмотр запусков действий)) в столбце **Действия**. Для третьего конвейера отобразятся сведения о трех выполнениях действий: по одному для каждого действия в цепочке конвейера (веб-действие для запуска среды выполнения интеграции, действие "Хранимая процедура" для выполнения пакета и веб-действие для остановки IR). Чтобы вернуться к просмотру выполнений конвейера, выберите ссылку **Конвейеры** в верхней части окна.

   ![Выполнение действия](./media/how-to-schedule-azure-ssis-integration-runtime/activity-runs.png)

3. Чтобы просмотреть сведения о запусках триггера, выберите **Trigger Runs** (Запуски триггера) в раскрывающемся списке под разделом **Pipeline Runs** (Запуски конвейера) в верхней части экрана. 

   ![Выполнения триггеров](./media/how-to-schedule-azure-ssis-integration-runtime/trigger-runs.png)

### <a name="monitor-your-pipelines-and-triggers-with-powershell"></a>Мониторинг конвейеров и триггеров с помощью PowerShell

Для мониторинга конвейеров и триггеров можно применить сценарии, аналогичные приведенному ниже примеру.

1. Получить сведения о состоянии работы конвейера.

   ```powershell
   Get-AzDataFactoryV2PipelineRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $myPipelineRun
   ```

2. Получение сведений о триггере.

   ```powershell
   Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name  "myTrigger"
   ```

3. Получить сведения о состоянии работы триггера.

   ```powershell
   Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "myTrigger" -TriggerRunStartedAfter "2018-07-15" -TriggerRunStartedBefore "2018-07-16"
   ```

## <a name="create-and-schedule-azure-automation-runbook-that-startsstops-azure-ssis-ir"></a>Создание и планирование runbook службы автоматизации Azure, который запускает или останавливает Azure-SSIS IR

В рамках этого раздела вы научитесь создавать runbook службы автоматизации Azure, который по расписанию выполняет скрипт PowerShell для запуска и (или) остановки Azure-SSIS IR.  Это полезно в тех случаях, когда нужна предварительная обработка или постобработка, то есть, если требуется выполнить дополнительные скрипты до и (или) после запуска и (или) остановки IR.

### <a name="create-your-azure-automation-account"></a>Создание учетной записи службы автоматизации Azure

Если у вас еще нет учетной записи службы автоматизации Azure, создайте ее по инструкциям в этом шаге. Подробные инструкции см. в статье [Создание учетной записи службы автоматизации Azure](../automation/automation-quickstart-create-account.md). На этом шаге вы создадите учетную запись **запуска от имени** в Azure (субъект-службу Azure Active Directory) и добавите ее в роль **участника** подписки Azure. Это должна быть та же подписка, которая содержит ADF с Azure-SSIS IR. Служба автоматизации Azure применит эту учетную запись для аутентификации в Azure Resource Manager и управления ресурсами. 

1. Запустите веб-браузер **Microsoft Edge** или **Google Chrome**. Сейчас только браузеры Microsoft Edge и Google Chrome поддерживают пользовательский интерфейс (приложение) ADF.
2. Войдите на [портал Azure](https://portal.azure.com/).    
3. В меню слева выберите **Создать**, **Мониторинг и управление** и **Служба автоматизации**. 

   ![Снимок экрана, посвященный параметру автоматизации Мониторинг и управление >.](./media/how-to-schedule-azure-ssis-integration-runtime/new-automation.png)
    
2. На панели **Добавление учетной записи службы автоматизации** выполните следующие действия:

    1. В поле **Имя** введите имя учетной записи службы автоматизации Azure. 
    2. В поле **Подписка** выберите подписку, в которой размещена ADF с Azure-SSIS IR. 
    3. В поле **Группа ресурсов** выберите **Создать новую**, чтобы создать группу ресурсов, или **Использовать существующую**, чтобы выбрать уже существующую. 
    4. В поле **Расположение** выберите расположение для учетной записи службы автоматизации Azure. 
    5. В поле **Создать учетную запись запуска от имени Azure** подтвердите создание, выбрав **Да**. Это действие позволяет создать в Azure Active Directory субъект-службу и назначить ей роль **Участник** в подписке Azure.
    6. Выберите **Закрепить на панели мониторинга**, чтобы этот субъект постоянно отображался на панели мониторинга Azure. 
    7. Нажмите кнопку **создания**. 

   ![Создать -> Мониторинг и управление -> Служба автоматизации](./media/how-to-schedule-azure-ssis-integration-runtime/add-automation-account-window.png)
   
3. Состояние развертывания учетной записи службы автоматизации Azure можно контролировать по панели мониторинга Azure и по уведомлениям. 
    
   ![Развертывание службы автоматизации](./media/how-to-schedule-azure-ssis-integration-runtime/deploying-automation.png) 
    
4. После успешного создания учетной записи службы автоматизации Azure откроется ее домашняя страница. 

   ![Домашняя страница службы автоматизации](./media/how-to-schedule-azure-ssis-integration-runtime/automation-home-page.png)

### <a name="import-adf-modules"></a>Импорт модулей ADF

1. Выберите **модули** в разделе **Общие ресурсы** в меню слева и убедитесь, что у вас есть команда **AZ.** объект  +  **AZ. Profile** в списке модулей.

   ![Проверка необходимых модулей](media/how-to-schedule-azure-ssis-integration-runtime/automation-fix-image1.png)

2.  Если у вас нет **AZ. объект фактов**, перейдите к коллекция PowerShell для [AZ. объект фактов](https://www.powershellgallery.com/packages/Az.DataFactory/), выберите пункт **развернуть в службе автоматизации Azure**, выберите учетную запись службы автоматизации Azure и нажмите кнопку **ОК**. Вернитесь к разделу Просмотр **модулей** в разделе **Общие ресурсы** в меню слева и подождите, пока не отобразится **состояние** **AZ. модуль фактов** изменился на **доступный**.

    ![Проверка модуля фабрики данных](media/how-to-schedule-azure-ssis-integration-runtime/automation-fix-image2.png)

3.  Если у вас нет **AZ. Profile**, перейдите к модулю коллекция PowerShell для [AZ. Profile](https://www.powershellgallery.com/packages/Az.profile/), выберите пункт **развернуть в службе автоматизации Azure**, выберите учетную запись службы автоматизации Azure и нажмите кнопку **ОК**. Вернитесь к разделу Просмотр **модулей** в разделе **Общие ресурсы** в меню слева и подождите, пока **состояние** модуля **AZ. Profile** изменится на **доступно**.

    ![Проверка модуля профиля](media/how-to-schedule-azure-ssis-integration-runtime/automation-fix-image3.png)

### <a name="create-your-powershell-runbook"></a>Создание runbook PowerShell

В следующем разделе приведены инструкции по созданию runbook PowerShell. Скрипт, связанный с этим модулем runbook, позволяет запустить или остановить Azure-SSIS IR в зависимости от команды, указанной в параметре **OPERATION**. В этом разделе предоставлены не все сведения по созданию runbook. Дополнительные сведения см. в статье [Создание runbook службы автоматизации Azure](../automation/automation-quickstart-create-runbook.md).

1. Перейдите на вкладку **Модули Runbook** и выберите **+ Добавить Runbook** на панели инструментов. 

   ![Снимок экрана, на котором выделяется кнопка + Добавить Runbook.](./media/how-to-schedule-azure-ssis-integration-runtime/runbooks-window.png)
   
2. Выберите **Создать новый Runbook** и сделайте следующее: 

    1. В поле **Имя** введите **StartStopAzureSsisRuntime**.
    2. Для поля **Тип Runbook** выберите значение **PowerShell**.
    3. Нажмите кнопку **создания**.
    
   ![Кнопка добавления модуля runbook](./media/how-to-schedule-azure-ssis-integration-runtime/add-runbook-window.png)
   
3. Скопируйте и вставьте приведенный ниже скрипт PowerShell в окно сценария runbook. Сохраните и опубликуйте runbook, нажав кнопки **Сохранить** и **Опубликовать** на панели инструментов. 

    ```powershell
    Param
    (
          [Parameter (Mandatory= $true)]
          [String] $ResourceGroupName,
    
          [Parameter (Mandatory= $true)]
          [String] $DataFactoryName,
    
          [Parameter (Mandatory= $true)]
          [String] $AzureSSISName,
    
          [Parameter (Mandatory= $true)]
          [String] $Operation
    )
    
    $connectionName = "AzureRunAsConnection"
    try
    {
        # Get the connection "AzureRunAsConnection "
        $servicePrincipalConnection=Get-AutomationConnection -Name $connectionName         
    
        "Logging in to Azure..."
        Connect-AzAccount `
            -ServicePrincipal `
            -TenantId $servicePrincipalConnection.TenantId `
            -ApplicationId $servicePrincipalConnection.ApplicationId `
            -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
    }
    catch {
        if (!$servicePrincipalConnection)
        {
            $ErrorMessage = "Connection $connectionName not found."
            throw $ErrorMessage
        } else{
            Write-Error -Message $_.Exception
            throw $_.Exception
        }
    }
    
    if($Operation -eq "START" -or $operation -eq "start")
    {
        "##### Starting #####"
        Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $AzureSSISName -Force
    }
    elseif($Operation -eq "STOP" -or $operation -eq "stop")
    {
        "##### Stopping #####"
        Stop-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Force
    }  
    "##### Completed #####"    
    ```

   ![Редактирование модуля runbook PowerShell](./media/how-to-schedule-azure-ssis-integration-runtime/edit-powershell-runbook.png)
    
4. Протестируйте runbook, нажав кнопку **Запуск** на панели инструментов. 

   ![Кнопка запуска модуля runbook](./media/how-to-schedule-azure-ssis-integration-runtime/start-runbook-button.png)
    
5. В области **Запуск Runbook** выполните следующие действия. 

    1. В поле **RESOURCE GROUP NAME** (Имя группы ресурсов) введите имя группы ресурсов, в которой размещается ADF c Azure-SSIS IR. 
    2. В поле **DATA FACTORY NAME** (Имя Фабрики данных) введите имя ADF, где расположена Azure-SSIS IR. 
    3. В поле **AZURESSISNAME** (Имя Azure-SSIS) введите имя Azure-SSIS IR. 
    4. Для параметра **OPERATION** введите **START**. 
    5. Щелкните **ОК**.  

   ![Окно "Запуск Runbook"](./media/how-to-schedule-azure-ssis-integration-runtime/start-runbook-window.png)
   
6. В окне "Задание" выберите плитку **Вывод**. Дождитесь, пока в окне вывода появится сообщение **### Completed ###** (Завершено) после сообщения **### Starting ###** (Запуск). Запуск Azure-SSIS IR занимает приблизительно 20 минут. Закройте окно **Задание** и вернитесь к окну **Runbook**.

   ![Снимок экрана, на котором выделена плитка вывода.](./media/how-to-schedule-azure-ssis-integration-runtime/start-completed.png)
    
7. Повторите предыдущие два шага с действием **STOP** в качестве значения параметра **OPERATION**. Еще раз протестируйте runbook, нажав кнопку **Запуск** на панели инструментов. Введите имена группы ресурсов,ADF и Azure-SSIS IR. Для параметра **OPERATION** введите **STOP**. Дождитесь, пока в окне вывода появится сообщение **### Completed ###** (Завершено) после сообщения **### Stopping ###** (Остановка). Остановка Azure-SSIS IR занимает меньше времени, чем ее запуск. Закройте окно **Задание** и вернитесь к окну **Runbook**.

8. Вы также можете активировать модуль Runbook через веб-перехватчик, который можно создать, выбрав пункт меню **веб-перехватчиков** или указав расписание, которое можно создать, выбрав пункт меню **расписания** , как указано ниже.  

## <a name="create-schedules-for-your-runbook-to-startstop-azure-ssis-ir"></a>Создание расписаний для запуска и остановки Azure-SSIS IR с помощью runbook

В рамках предыдущего раздела вы создали runbook службы автоматизации Azure, который может запустить или остановить Azure-SSIS IR. В рамках этого раздела вы создадите для этого runbook два расписания. При настройке первого расписания для параметра **OPERATION** укажите значение **START**. Этот же параметр **OPERATION** для второго расписания должен иметь значение **STOP**. Подробные инструкции по созданию расписаний см. в [этой статье](../automation/shared-resources/schedules.md#create-a-schedule).

1. В окне **Runbook** выберите **Расписания** и щелкните **+ Добавить расписание** на панели инструментов. 

   ![IR Azure SSIS запущена](./media/how-to-schedule-azure-ssis-integration-runtime/add-schedules-button.png)
   
2. На панели **Включить модуль Runbook в расписание** выполните следующие действия: 

    1. Выберите **Связать расписание с модулем Runbook**. 
    2. Выберите **Создать новое расписание**.
    3. На панели **Новое расписание** введите **Start IR daily** (Запускать IR ежедневно) в поле **Имя**. 
    4. В поле **Запускается** введите время на несколько минут позже текущего времени. 
    5. Для параметра **Периодичность** выберите значение **Повторяющееся**. 
    6. В поле **Повторять каждые** введите значение **1** и выберите **День**. 
    7. Нажмите кнопку **создания**. 

   ![Расписание для запуска IR Azure SSIS](./media/how-to-schedule-azure-ssis-integration-runtime/new-schedule-start.png)
    
3. Переключитесь на вкладку Параметры **и параметры запуска** . Укажите имя группы ресурсов, ADF и Azure-SSIS IR. Для параметра **OPERATION** введите **START**, затем щелкните **OK**. Снова щелкните **ОК**, чтобы просмотреть расписание на странице **Расписания** для runbook. 

   ![Снимок экрана, на котором выделено поле "операция".](./media/how-to-schedule-azure-ssis-integration-runtime/start-schedule.png)
    
4. Повторите предыдущие два шага, чтобы создать расписание с именем **Останавливать IR ежедневно**. Укажите время как минимум на 30 минут позже времени, указанного для расписания **Start IR daily** (Запускать IR ежедневно). Для параметра **OPERATION** введите значение **STOP**, затем щелкните **OK**. Снова щелкните **ОК**, чтобы просмотреть расписание на странице **Расписания** для runbook. 

5. В окне **Runbook** выберите **Задания** в меню слева. Вы увидите задания, созданные расписаниями в указанное время, и их состояния. Отобразятся сведения о задании, например его выходные данные. Они будут похожи на результаты, полученные при тестировании runbook. 

   ![Расписание для запуска IR Azure SSIS](./media/how-to-schedule-azure-ssis-integration-runtime/schedule-jobs.png)
    
6. Завершив тестирование, измените параметры расписаний, чтобы отключить их. Выберите **Расписания** в меню слева, выберите **Start IR daily/Stop IR daily** ("Запускать IR ежедневно" или "Останавливать IR ежедневно") и укажите значение **Нет** для параметра **Включено**. 

## <a name="next-steps"></a>Дальнейшие действия
См. в следующей записи блога:
-   [Модернизация и расширение рабочих процессов ETL/ELT с помощью операций MSSQL Integration Services в конвейерах ADF](https://techcommunity.microsoft.com/t5/SQL-Server-Integration-Services/Modernize-and-Extend-Your-ETL-ELT-Workflows-with-SSIS-Activities/ba-p/388370)

См. документацию SSIS в следующих статьях: 

- [Развертывание, запуск и отслеживание пакета служб SSIS в Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial)   
- [Connect to the SSISDB Catalog database on Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database) (Подключение к базе данных каталога SSISDB в Azure)
- [Schedule the execution of an SSIS package on Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages) (Планирование выполнения пакета SSIS в Azure)
- [Connect to on-premises data sources with Windows Authentication](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth) (Подключение к локальным источникам данных с помощью аутентификации Windows)