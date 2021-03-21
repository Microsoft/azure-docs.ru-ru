---
title: Управление репликами чтения — портал Azure — гибкий сервер в базе данных Azure для MySQL
description: Узнайте, как настроить и управлять репликами чтения в базе данных Azure для гибкого сервера MySQL с помощью портал Azure.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 10/26/2020
ms.openlocfilehash: d33734dc7404e49aed94dffae8644b2bc4386925
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96492834"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mysql-flexible-server-using-the-azure-portal"></a>Как создавать реплики чтения и управлять ими в базе данных Azure для гибкого сервера MySQL с помощью портал Azure

> [!IMPORTANT]
> Чтение реплик в базе данных Azure для MySQL — гибкий сервер находится на этапе предварительной версии.

В этой статье вы узнаете, как создавать реплики чтения и управлять ими в базе данных Azure для гибкого сервера MySQL с помощью портал Azure.

> [!Note]
> Реплика не поддерживается на сервере с включенной высокой доступностью. 

## <a name="prerequisites"></a>Предварительные условия

- [Гибкий сервер базы данных Azure для MySQL](quickstart-create-server-portal.md) , который будет использоваться в качестве исходного сервера.

## <a name="create-a-read-replica"></a>Создание реплики чтения

> [!IMPORTANT]
> При создании реплики для источника, не имеющего существующих реплик, источник сначала перезапускается для подготовки к репликации. Это необходимо учесть, то есть выполнять такие операции в период низкой нагрузки.

Чтобы создать сервер-реплику чтения, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Выберите имеющийся гибкий сервер базы данных Azure для MySQL, который вы хотите использовать в качестве источника. Откроется страница **Обзор**.

3. В меню в разделе **Параметры** выберите **Репликация**.

4. Выберите **Добавить реплику**.

   :::image type="content" source="./media/how-to-read-replica-portal/add-replica.png" alt-text="База данных Azure для MySQL — репликация":::

5. Введите имя сервера реплики.

    :::image type="content" source="./media/how-to-read-replica-portal/replica-name.png" alt-text="База данных Azure для MySQL — имя реплики":::

6. Нажмите кнопку **ОК**, чтобы подтвердить создание реплики.

> [!NOTE]
> Реплики чтения создаются с той же конфигурацией сервера, что и источник. Вы можете изменить созданную конфигурацию сервера-реплики. Сервер реплики всегда создается в той же группе ресурсов, что и исходный сервер, и ту же самую подписку. Если вы хотите создать сервер реплики в другой группе ресурсов или другой подписке, можно [переместить сервер реплики](../../azure-resource-manager/management/move-resource-group-and-subscription.md) после его создания. Рекомендуется, чтобы конфигурация сервера реплики хранилась в значениях, равных или превышающих источник, чтобы гарантировать, что реплика сможет справиться с источником.

Созданный сервер-реплику можно просмотреть в колонке **Репликация**.

   [:::image type="content" source="./media/how-to-read-replica-portal/list-replica.png" alt-text="База данных Azure для MySQL — список реплик":::](./media/how-to-read-replica-portal/list-replica.png#lightbox)

## <a name="stop-replication-to-a-replica-server"></a>Остановка репликации на сервер-реплику

> [!IMPORTANT]
> Остановка репликации на сервер является необратимой операцией. После остановки репликации между источником и репликой отменить ее нельзя. Сервер-реплика становится автономным и начинает поддерживает операции чтения и записи. Это сервер нельзя снова преобразовать в реплику.

Чтобы предотвратить репликацию между источником и сервером-репликой из портал Azure, выполните следующие действия.

1. В портал Azure выберите исходную базу данных Azure для гибкого сервера MySQL. 

2. В меню в разделе **Параметры** выберите **Репликация**.

3. Выберите сервер-реплику, для которого нужно остановить репликацию.

   [:::image type="content" source="./media/how-to-read-replica-portal/stop-replication-select.png" alt-text="База данных Azure для MySQL — выбор сервера, для которого нужно остановить репликацию":::](./media/how-to-read-replica-portal/stop-replication-select.png#lightbox)

4. Щелкните **Остановить репликацию**.

   [:::image type="content" source="./media/how-to-read-replica-portal/stop-replication.png" alt-text="База данных Azure для MySQL — остановка репликации":::](./media/how-to-read-replica-portal/stop-replication.png#lightbox)

5. Подтвердите остановку репликации, нажав кнопку **ОК**.

   [:::image type="content" source="./media/how-to-read-replica-portal/stop-replication-confirm.png" alt-text="База данных Azure для MySQL — подтверждение остановки репликации":::](./media/how-to-read-replica-portal/stop-replication-confirm.png#lightbox)

## <a name="delete-a-replica-server"></a>Удаление сервера-реплики

Чтобы удалить сервер-реплику чтения на портале Azure, следуйте инструкциям ниже.

1. В портал Azure выберите исходную базу данных Azure для гибкого сервера MySQL.

2. В меню в разделе **Параметры** выберите **Репликация**.

3. Выберите сервер-реплику, который нужно удалить.

   [:::image type="content" source="./media/how-to-read-replica-portal/delete-replica-select.png" alt-text="База данных Azure для MySQL — выбор удаляемого сервера-реплики":::](./media/how-to-read-replica-portal/delete-replica-select.png#lightbox)

4. Щелкните **Удалить реплику**.

   :::image type="content" source="./media/how-to-read-replica-portal/delete-replica.png" alt-text="База данных Azure для MySQL — удаление реплики":::

5. Введите имя реплики и нажмите кнопку **Удалить**, чтобы подтвердить удаление реплики.  

   :::image type="content" source="./media/how-to-read-replica-portal/delete-replica-confirm.png" alt-text="База данных Azure для MySQL — подтверждение удаления реплики":::

## <a name="delete-a-source-server"></a>Удаление исходного сервера

> [!IMPORTANT]
> Удаление исходного сервера приводит к остановке репликации на все серверы-реплики и удалению самого исходного сервера. Серверы-реплики становятся автономными серверами, которые начинают поддерживать операции чтения и записи.

Чтобы удалить исходный сервер из портал Azure, выполните следующие действия.

1. В портал Azure выберите исходную базу данных Azure для гибкого сервера MySQL.

2. На странице **Обзор** выберите **Удалить**.

   [:::image type="content" source="./media/how-to-read-replica-portal/delete-master-overview.png" alt-text="База данных Azure для MySQL — удаление источника":::](./media/how-to-read-replica-portal/delete-master-overview.png#lightbox)

3. Введите имя исходного сервера и нажмите кнопку **Удалить** , чтобы подтвердить удаление исходного сервера.  

   :::image type="content" source="./media/how-to-read-replica-portal/delete-master-confirm.png" alt-text="База данных Azure для MySQL — подтверждение удаления источника":::

## <a name="monitor-replication"></a>Мониторинг репликации

1. В [портал Azure](https://portal.azure.com/)выберите реплику базы данных Azure для гибкого сервера MySQL, которую требуется отслеживать.

2. В разделе боковой панели **Мониторинг** выберите **Метрики**.

3. В раскрывающемся списке доступных метрик выберите **Replication lag in seconds** (Задержка репликации в секундах).

   [:::image type="content" source="./media/how-to-read-replica-portal/monitor-select-replication-lag.png" alt-text="Выбор задержки репликации":::](./media/how-to-read-replica-portal/monitor-select-replication-lag.png#lightbox)

4. Выберите нужный диапазон времени. На рисунке ниже выбран диапазон в 30 минут.

   [:::image type="content" source="./media/how-to-read-replica-portal/monitor-replication-lag-time-range.png" alt-text="Выбор диапазона времени":::](./media/how-to-read-replica-portal/monitor-replication-lag-time-range.png#lightbox)

5. Просмотрите задержку репликации для выбранного диапазона времени. На рисунке ниже отображаются последние 30 минут.

   [:::image type="content" source="./media/how-to-read-replica-portal/monitor-replication-lag-time-range-thirty-mins.png" alt-text="Выберите диапазон времени 30 минут":::](./media/how-to-read-replica-portal/monitor-replication-lag-time-range-thirty-mins.png#lightbox)

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [репликах чтения](concepts-read-replicas.md)