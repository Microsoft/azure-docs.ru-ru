---
title: Управление базой данных Azure для PostgreSQL — портал Azure
description: Узнайте, как управлять сервером базы данных Azure для PostgreSQL из портал Azure.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: how-to
ms.date: 11/20/2019
ms.openlocfilehash: 4b581ac137a4172ab70e4fb3fb2a75e268115c06
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105604130"
---
# <a name="manage-an-azure-database-for-postgresql-server-using-the-azure-portal"></a>Управление базой данных Azure для сервера PostgreSQL с помощью портал Azure

В этой статье показано, как управлять базой данных Azure для серверов PostgreSQL. Задачи управления включают в себя вычисление и масштабирование хранилища, сброс пароля администратора и Просмотр сведений о сервере.

## <a name="sign-in"></a>Вход

Войдите на [портал Azure](https://portal.azure.com).

## <a name="create-a-server"></a>Создание сервера

Чтобы узнать, как создать и начать работу с сервером базы данных Azure для PostgreSQL, ознакомьтесь с [кратким](quickstart-create-server-database-portal.md) руководством.

## <a name="scale-compute-and-storage"></a>Масштабирование вычислений и хранилища

После создания сервера можно выполнить масштабирование общего назначения и оптимизированных для памяти уровней по мере изменения потребностей. Кроме того, можно масштабировать вычисление и память, увеличивая или уменьшая виртуальных ядер. Возможно масштабирование хранилища (Однако нельзя масштабировать хранилище).

### <a name="scale-between-general-purpose-and-memory-optimized-tiers"></a>Масштабирование между уровнями общего назначения и оптимизированной для памяти

Можно масштабировать из общего назначения в оптимизированную для памяти и наоборот. Переход на базовый уровень и с него после создания сервера не поддерживается.

1. Выберите сервер в портал Azure. Выберите **ценовую категорию**, расположенную в разделе **Параметры** .

2. Выберите **общего назначения** или **оптимизировать память** в зависимости от того, что вы масштабируете.

   :::image type="content" source="./media/howto-create-manage-server-portal/change-pricing-tier.png" alt-text="Снимок экрана портал Azure для выбора уровня &quot;базовый&quot;, &quot;общего назначения&quot; или &quot;оптимизировано для памяти&quot; в базе данных Azure для PostgreSQL":::

   > [!NOTE]
   > Изменение уровней приводит к перезапуску сервера.

3. Нажмите кнопку **ОК** , чтобы сохранить изменения.

### <a name="scale-vcores-up-or-down"></a>Масштабирование виртуальных ядер вверх или вниз

1. Выберите сервер в портал Azure. Выберите **ценовую категорию**, расположенную в разделе **Параметры** .

2. Измените параметр **Виртуальное ядро**, переместив ползунок на нужное значение.

   :::image type="content" source="./media/howto-create-manage-server-portal/scaling-compute.png" alt-text="Снимок экрана портал Azure выбор параметра Виртуальное ядро в базе данных Azure для PostgreSQL":::

   > [!NOTE]
   > Масштабирование виртуальных ядер приводит к перезапуску сервера.

3. Нажмите кнопку **ОК** , чтобы сохранить изменения.

### <a name="scale-storage-up"></a>Масштабирование хранилища

1. Выберите сервер в портал Azure. Выберите **ценовую категорию**, расположенную в разделе **Параметры** .

2. Измените параметр **хранилища** , переместив ползунок вверх до нужного значения.

   :::image type="content" source="./media/howto-create-manage-server-portal/scaling-storage.png" alt-text="Снимок экрана портал Azure выбор масштаба хранилища в базе данных Azure для PostgreSQL":::

   > [!NOTE]
   > Невозможно масштабировать хранилище.

3. Нажмите кнопку **ОК** , чтобы сохранить изменения.

## <a name="update-admin-password"></a>Обновление пароля администратора

Пароль роли администратора можно изменить с помощью портал Azure.

1. Выберите сервер в портал Azure. В окне **обзора** выберите **Сброс пароля**.

   :::image type="content" source="./media/howto-create-manage-server-portal/overview-reset-password.png" alt-text="Снимок экрана портал Azure сброс пароля в базе данных Azure для PostgreSQL":::

2. Введите новый пароль и подтвердите его. В текстовом поле будет предложено ввести требования к сложности пароля.

   :::image type="content" source="./media/howto-create-manage-server-portal/reset-password.png" alt-text="Снимок экрана портал Azure сброс пароля и сохранение в базе данных Azure для PostgreSQL":::

3. Нажмите кнопку **ОК** , чтобы сохранить новый пароль.

## <a name="delete-a-server"></a>Удаление сервера

Вы можете удалить сервер, если он больше не нужен. 

1. Выберите сервер в портал Azure. В окне **обзора** выберите **Удалить**.

   :::image type="content" source="./media/howto-create-manage-server-portal/overview-delete.png" alt-text="Снимок экрана портал Azure для удаления сервера в базе данных Azure для PostgreSQL":::

2. Введите имя сервера в поле ввода, чтобы убедиться, что это сервер, который требуется удалить.

   :::image type="content" source="./media/howto-create-manage-server-portal/confirm-delete.png" alt-text="Снимок экрана портал Azure для подтверждения удаления сервера в базе данных Azure для PostgreSQL":::

   > [!NOTE]
   > Удаление сервера необратимо.

3. Выберите команду **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о [резервном копировании и восстановлении сервера](howto-restore-server-portal.md)
- Сведения о [параметрах настройки и мониторинга в базе данных Azure для PostgreSQL](concepts-monitoring.md)
