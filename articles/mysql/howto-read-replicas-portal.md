---
title: Управление репликами чтения — портал Azure — база данных Azure для MySQL
description: Узнайте, как настраивать реплики чтения и управлять ими в базе данных Azure для MySQL с помощью портала Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 6/10/2020
ms.openlocfilehash: 26b503e7d55ed3d2f9bd06837551655e7af05a17
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94541946"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mysql-using-the-azure-portal"></a>Создание реплик чтения и управление ими в базе данных Azure для MySQL с помощью портала Azure

В этой статье описано, как создавать реплики чтения и управлять ими в службе "База данных Azure для MySQL" с помощью портала Azure.

## <a name="prerequisites"></a>Предварительные требования

- [Сервер базы данных Azure для MySQL](quickstart-create-mysql-server-database-using-azure-portal.md) , который будет использоваться в качестве исходного сервера.

> [!IMPORTANT]
> Функция создания реплики чтения доступна только для серверов базы данных Azure для MySQL в ценовой категории "Общее назначение" или "Оптимизированная для операций в памяти". Убедитесь, что исходный сервер находится в одной из этих ценовых категорий.

## <a name="create-a-read-replica"></a>Создание реплики чтения

> [!IMPORTANT]
> При создании реплики для источника, не имеющего существующих реплик, источник сначала перезапускается для подготовки к репликации. Это необходимо учесть, то есть выполнять такие операции в период низкой нагрузки.

Чтобы создать сервер-реплику чтения, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Выберите имеющийся сервер базы данных Azure для MySQL, который будет главным сервером. Откроется страница **Обзор**.

3. В меню в разделе **Параметры** выберите **Репликация**.

4. Выберите **Добавить реплику**.

   :::image type="content" source="./media/howto-read-replica-portal/add-replica.png" alt-text="База данных Azure для MySQL — репликация":::

5. Введите имя сервера реплики.

    :::image type="content" source="./media/howto-read-replica-portal/replica-name.png" alt-text="База данных Azure для MySQL — имя реплики":::

6. Укажите расположение сервера реплики. Расположение по умолчанию совпадает с местоположением исходного сервера.

    :::image type="content" source="./media/howto-read-replica-portal/replica-location.png" alt-text="База данных Azure для MySQL — расположение реплики":::

   > [!NOTE]
   > Дополнительные сведения о том, в каких регионах можно создать реплику, см. в статье [об основных понятиях реплики чтения](concepts-read-replicas.md). 

7. Нажмите кнопку **ОК**, чтобы подтвердить создание реплики.

> [!NOTE]
> Реплики чтения создаются с той же конфигурацией сервера, что и у главного сервера. Вы можете изменить созданную конфигурацию сервера-реплики. Сервер реплики всегда создается в той же группе ресурсов и в той же подписке, что и исходный сервер. Если вы хотите создать сервер реплики в другой группе ресурсов или другой подписке, можно [переместить сервер реплики](../azure-resource-manager/management/move-resource-group-and-subscription.md) после его создания. Рекомендуется, чтобы конфигурация сервера реплики хранилась в значении, превышающем значение источника, чтобы реплика могла поддерживать базу данных master.

Созданный сервер-реплику можно просмотреть в колонке **Репликация**.

   :::image type="content" source="./media/howto-read-replica-portal/list-replica.png" alt-text="База данных Azure для MySQL — список реплик":::

## <a name="stop-replication-to-a-replica-server"></a>Остановка репликации на сервер-реплику

> [!IMPORTANT]
> Остановка репликации на сервер является необратимой операцией. После остановки репликации между источником и репликой отменить ее нельзя. Сервер-реплика становится автономным и начинает поддерживает операции чтения и записи. Это сервер нельзя снова преобразовать в реплику.

Чтобы предотвратить репликацию между источником и сервером-репликой из портал Azure, выполните следующие действия.

1. В портал Azure выберите исходную базу данных Azure для сервера MySQL. 

2. В меню в разделе **Параметры** выберите **Репликация**.

3. Выберите сервер-реплику, для которого нужно остановить репликацию.

   :::image type="content" source="./media/howto-read-replica-portal/stop-replication-select.png" alt-text="База данных Azure для MySQL — выбор сервера, для которого нужно остановить репликацию":::

4. Щелкните **Остановить репликацию**.

   :::image type="content" source="./media/howto-read-replica-portal/stop-replication.png" alt-text="База данных Azure для MySQL — остановка репликации":::

5. Подтвердите остановку репликации, нажав кнопку **ОК**.

   :::image type="content" source="./media/howto-read-replica-portal/stop-replication-confirm.png" alt-text="База данных Azure для MySQL — подтверждение остановки репликации":::

## <a name="delete-a-replica-server"></a>Удаление сервера-реплики

Чтобы удалить сервер-реплику чтения на портале Azure, следуйте инструкциям ниже.

1. В портал Azure выберите исходную базу данных Azure для сервера MySQL.

2. В меню в разделе **Параметры** выберите **Репликация**.

3. Выберите сервер-реплику, который нужно удалить.

   :::image type="content" source="./media/howto-read-replica-portal/delete-replica-select.png" alt-text="База данных Azure для MySQL — выбор удаляемого сервера-реплики":::

4. Щелкните **Удалить реплику**.

   :::image type="content" source="./media/howto-read-replica-portal/delete-replica.png" alt-text="База данных Azure для MySQL — удаление реплики":::

5. Введите имя реплики и нажмите кнопку **Удалить**, чтобы подтвердить удаление реплики.  

   :::image type="content" source="./media/howto-read-replica-portal/delete-replica-confirm.png" alt-text="База данных Azure для MySQL — подтверждение удаления реплики":::

## <a name="delete-a-source-server"></a>Удаление исходного сервера

> [!IMPORTANT]
> Удаление исходного сервера приводит к остановке репликации на все серверы-реплики и удалению самого исходного сервера. Серверы-реплики становятся автономными серверами, которые начинают поддерживать операции чтения и записи.

Чтобы удалить исходный сервер из портал Azure, выполните следующие действия.

1. В портал Azure выберите исходную базу данных Azure для сервера MySQL.

2. На странице **Обзор** выберите **Удалить**.

   :::image type="content" source="./media/howto-read-replica-portal/delete-master-overview.png" alt-text="База данных Azure для MySQL — удаление главного сервера":::

3. Введите имя исходного сервера и нажмите кнопку **Удалить** , чтобы подтвердить удаление исходного сервера.  

   :::image type="content" source="./media/howto-read-replica-portal/delete-master-confirm.png" alt-text="База данных Azure для MySQL — подтверждение удаления главного сервера":::

## <a name="monitor-replication"></a>Мониторинг репликации

1. На [портале Azure](https://portal.azure.com/) выберите сервер-реплику базы данных Azure для MySQL, который нужно отследить.

2. В разделе боковой панели **Мониторинг** выберите **Метрики**.

3. В раскрывающемся списке доступных метрик выберите **Replication lag in seconds** (Задержка репликации в секундах).

   :::image type="content" source="./media/howto-read-replica-portal/monitor-select-replication-lag.png" alt-text="Выбор задержки репликации":::

4. Выберите нужный диапазон времени. На рисунке ниже выбран диапазон в 30 минут.

   :::image type="content" source="./media/howto-read-replica-portal/monitor-replication-lag-time-range.png" alt-text="Выбор диапазона времени":::

5. Просмотрите задержку репликации для выбранного диапазона времени. На рисунке ниже отображаются последние 30 минут.

   :::image type="content" source="./media/howto-read-replica-portal/monitor-replication-lag-time-range-thirty-mins.png" alt-text="Выберите диапазон времени 30 минут":::

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [репликах чтения](concepts-read-replicas.md)