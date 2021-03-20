---
title: Очистка задания Azure Stream Analytics
description: В этой статье показаны различные методы удаления заданий Azure Stream Analytics.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/21/2019
ms.custom: seodec18
ms.openlocfilehash: 31812ac805bd3465b1b735842b45adb372286d66
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98016071"
---
# <a name="stop-or-delete-your-azure-stream-analytics-job"></a>Завершение или удаление задания Azure Stream Analytics

Azure Stream Analytics задания можно легко остановить или удалить с помощью портал Azure, Azure PowerShell, пакета Azure SDK для .NET или REST API. Задание Stream Analytics нельзя восстановить после его удаления.

>[!NOTE] 
>При остановке задания Stream Analytics данные сохраняются в хранилище входных и выходных данных, например Центрах событий или Базе данных SQL Azure. Если необходимо удалить данные из Azure, обязательно следуйте процедуре удаления входящих и исходящих ресурсов задания Stream Analytics.

## <a name="stop-a-job-in-azure-portal"></a>Остановка задания на портале Azure

При остановке задания ресурсы отменяются и прекращается обработка событий. Расходы, связанные с этим заданием, также останавливаются. Однако все ваши настройки сохраняются, и вы можете перезапустить задание позже. 

1. Войдите на [портал Azure](https://portal.azure.com). 

2. Найдите выполняющееся задание Stream Analytics и выберите его.

3. На странице задания Stream Analytics щелкните **Остановить**, чтобы остановить задание. 

   ![Остановка задания Azure Stream Analytics](./media/stream-analytics-clean-up-your-job/stop-stream-analytics-job.png)


## <a name="delete-a-job-in-azure-portal"></a>Удаление задания на портале Azure

>[!WARNING] 
>Задание Stream Analytics нельзя восстановить после его удаления.

1. Войдите на портал Azure. 

2. Найдите имеющееся задание Stream Analytics и выберите его.

3. На странице задания Stream Analytics щелкните **Удалить**, чтобы удалить задание. 

   ![Удаление задания Azure Stream Analytics](./media/stream-analytics-clean-up-your-job/delete-stream-analytics-job.png)


## <a name="stop-or-delete-a-job-using-powershell"></a>Остановка или удаление задания с помощью PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Чтобы прерывать задание с помощью PowerShell, используйте командлет [азстреаманалитиксжоб](/powershell/module/az.streamanalytics/stop-azstreamanalyticsjob) . Чтобы удалить задание с помощью PowerShell, используйте командлет [Remove-азстреаманалитиксжоб](/powershell/module/az.streamanalytics/Remove-azStreamAnalyticsJob) .

## <a name="stop-or-delete-a-job-using-azure-sdk-for-net"></a>Остановка или удаление задания с помощью пакета Azure SDK для .NET

Чтобы остановить задание с помощью пакета Azure SDK для .NET, используйте метод [StreamingJobsOperationsExtensions.BeginStop](/dotnet/api/microsoft.azure.management.streamanalytics.streamingjobsoperationsextensions.beginstop). Чтобы удалить задание с помощью пакета Azure SDK для .NET, используйте метод [StreamingJobsOperationsExtensions.BeginDelete](/dotnet/api/microsoft.azure.management.streamanalytics.streamingjobsoperationsextensions.begindelete).

## <a name="stop-or-delete-a-job-using-rest-api"></a>Остановка или удаление задания с помощью REST API

Чтобы остановить задание с помощью REST API, используйте метод [Stop](/rest/api/streamanalytics/2016-03-01/streamingjobs/stop). Чтобы удалить задание с помощью REST API, используйте метод [Delete](/rest/api/streamanalytics/2016-03-01/streamingjobs/delete).