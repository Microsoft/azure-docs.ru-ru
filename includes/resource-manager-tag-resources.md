---
title: Включить файл
description: включить файл
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 03/19/2020
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: a00291182059506aeab9cde965fa4cbd5177ecf7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "80132147"
---
Если у пользователя нет необходимых прав доступа для применения тегов, можно назначить ему роль **Участник по тегам**. Дополнительные сведения см. в статье [Учебник. Предоставление доступа пользователю с помощью RBAC и портала Azure](../articles/role-based-access-control/quickstart-assign-role-user-portal.md).

1. Чтобы просмотреть теги для ресурса или группы ресурсов, ищите существующие теги в обзоре. Если теги ранее не применялись, то список будет пустым.

   ![Просмотр тегов для ресурса или группы ресурсов](./media/resource-manager-tag-resources/view-tags.png)

1. Чтобы добавить тег, выберите **Click here to add tags** (Щелкните здесь, чтобы добавить теги).

1. Укажите имя и значение.

   ![Добавление тега](./media/resource-manager-tag-resources/add-tag.png)

1. При необходимости продолжайте добавлять теги. Затем нажмите кнопку **Сохранить**.

   ![Сохранение тегов](./media/resource-manager-tag-resources/save-tags.png)

1. Теперь теги отображаются в обзоре.

   ![Отображение тегов](./media/resource-manager-tag-resources/view-new-tags.png)

1. Чтобы добавить или удалить тег, выберите **Изменить**.

1. Чтобы удалить тег, щелкните значок корзины. Затем нажмите кнопку **Сохранить**.

   ![Удаление тега](./media/resource-manager-tag-resources/delete-tag.png)

Вот как можно выполнить пакетное назначение тегов нескольким ресурсам.

1. В любом списке ресурсов установите флажки для ресурсов, которым требуется назначить тег. Затем выберите команду **Назначить теги**.

   ![Выбор нескольких ресурсов](./media/resource-manager-tag-resources/select-multiple-resources.png)

1. Добавьте имена и значения. Затем нажмите кнопку **Сохранить**.

   ![Выбор элемента "Назначить"](./media/resource-manager-tag-resources/select-assign.png)

Чтобы просмотреть все ресурсы с тегом, сделайте следующее.

1. В меню портала Azure выполните поиск по слову **теги**. Выберите подходящий вариант из предложенных.

   ![Поиск по тегу](./media/resource-manager-tag-resources/find-tags-general.png)

1. Выберите тег для просмотра ресурсов.

   ![Выбор тега](./media/resource-manager-tag-resources/select-tag.png)

1. Будут отображены все ресурсы с этим тегом.

   ![Просмотр ресурсов по тегу](./media/resource-manager-tag-resources/view-resources-by-tag.png)
