---
title: Создание и удаление заданий в кластере Azure Stream Analytics
description: Сведения о том, как управлять заданиями Stream Analytics в кластере Azure Stream Analytics
author: sidramadoss
ms.author: sidram
ms.service: stream-analytics
ms.topic: overview
ms.custom: mvc
ms.date: 04/16/2021
ms.openlocfilehash: 4a6334d33dea959bdd3704f848e7bc8250b6e7c6
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/18/2021
ms.locfileid: "107600729"
---
# <a name="add-and-remove-jobs-in-an-azure-stream-analytics-cluster"></a>Добавление и удаление заданий в кластере Azure Stream Analytics

В кластере Azure Stream Analytics можно запустить сразу несколько заданий Stream Analytics. Для выполнения заданий в кластере используется простой двухэтапный процесс: добавьте задание в кластер и запустите задание. В этой статье показано, как добавлять задания в существующий кластер и удалять их из него. Выполните инструкции из краткого руководства [по созданию кластера Stream Analytics](create-cluster.md), если у вас его еще нет.

## <a name="add-a-stream-analytics-job-to-a-cluster"></a>Добавление задания Stream Analytics в кластер

В кластеры можно добавлять только существующие задания Stream Analytics. Сведения о создании заданий с помощью портала Azure см. в [этом кратком руководстве](stream-analytics-quick-create-portal.md). Создав задание для добавления в кластер, переходите к следующей процедуре, чтобы добавить это задание в кластер.

1. Найдите и выберите нужный кластер Stream Analytics на портале Azure.

1. В разделе **Параметры** выберите **Задания Stream Analytics**. Выберите действие **Добавление существующего задания**.

1. Выберите нужную подписку и задание Stream Analytics, которое вы хотите добавить в кластер. В кластер можно добавлять задания Stream Analytics только из того же региона, где расположен кластер.

   ![Добавление задания в кластер](./media/manage-jobs-cluster/add-job.png)

1. Добавив задание в кластер, перейдите к ресурсу задания и [запустите это задание](start-job.md#azure-portal). Задание немедленно начнет выполняться в кластере.

Все остальные операции, в том числе с мониторингом, оповещениями и журналами диагностики, выполняются на странице ресурса задания Stream Analytics.

## <a name="remove-a-stream-analytics-job-from-a-cluster"></a>Удаление задания Stream Analytics из кластера

Чтобы удалить задание Stream Analytics из кластера, оно должно находиться в остановленном состоянии. Если задание еще выполняется, завершите его, прежде чем переходить к следующей процедуре.

1. Найдите и выберите нужный кластер Stream Analytics.

1. В разделе **Параметры** выберите **Задания Stream Analytics**.

1. Выберите задания, которые нужно удалить из кластера, а затем щелкните **Удалить**.

   ![Удаление задания из кластера](./media/manage-jobs-cluster/remove-job.png)

   Удаленное из кластера Stream Analytics задание возвращается в стандартную среду с несколькими клиентами.

## <a name="next-steps"></a>Следующие шаги

Теперь вы знаете, как добавлять и удалять задания в кластере Azure Stream Analytics. Далее вы можете узнать, как управлять частными конечными точками и масштабировать кластеры:

* [Масштабирование кластера Azure Stream Analytics](scale-cluster.md)
* [Управление частными конечными точками в кластере Azure Stream Analytics](private-endpoints.md)
