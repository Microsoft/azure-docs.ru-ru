---
title: Создание сред выполнения интеграции и управление ими
description: В этой статье описаны действия по созданию и управлению средой выполнения интеграции в Azure зрения.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 02/03/2021
ms.openlocfilehash: 9276f845c95b5e736180159b282ddedc33523c17
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99980751"
---
# <a name="create-and-manage-a-self-hosted-integration-runtime"></a>Создание локальной среды выполнения интеграции и управление ей

В этой статье описывается, как создать локальную среду выполнения интеграции (шир) и управлять ею, что позволяет проверять источники данных в Azure зрения.

> [!NOTE]
> Integration Runtime зрения нельзя использовать совместно с Integration Runtime Azure синапсе Analytics или фабрики данных Azure на одном компьютере. Его необходимо установить на отдельном компьютере.

## <a name="create-a-self-hosted-integration-runtime"></a>Создание локальной среды выполнения интеграции

1. На домашней странице зрения Studio в области навигации слева выберите **центр управления** .

2. В разделе **источники и сканирование** в левой области выберите **Integration Runtimes (среды выполнения интеграции**), а затем щелкните **+ создать**.

   :::image type="content" source="media/manage-integration-runtimes/select-integration-runtimes.png" alt-text="Щелкните IR.":::

3. На странице **Настройка среды выполнения интеграции** выберите локальное **Размещение** , чтобы создать Self-Hosted IR, а затем нажмите кнопку **продолжить**.

   :::image type="content" source="media/manage-integration-runtimes/select-self-hosted-ir.png" alt-text="Создайте новый шир.":::

4. Введите имя для своего типа IR и нажмите кнопку Создать.

5. На странице **параметры Integration Runtime** выполните действия, описанные в разделе **Настройка вручную** . Необходимо загрузить среду выполнения интеграции с сайта загрузки на ВИРТУАЛЬную машину или компьютер, где планируется ее запустить.

   :::image type="content" source="media/manage-integration-runtimes/integration-runtime-settings.png" alt-text="получить ключ":::

   - Скопируйте и вставьте ключ проверки подлинности.

   - Скачайте локальную среду выполнения интеграции из [Microsoft Integration Runtime](https://www.microsoft.com/download/details.aspx?id=39717) на локальном компьютере Windows. Запустите установщик.

   - На странице **регистрация Integration Runtime (Self-Hosted)** вставьте один из двух сохраненных ранее ключей и нажмите кнопку **зарегистрировать**.

     :::image type="content" source="media/manage-integration-runtimes/register-integration-runtime.png" alt-text="ключ ввода.":::

   - На странице **Новый узел среды выполнения интеграции (с локальным размещением)** нажмите кнопку **Finish** (Завершить)

6. После успешной регистрации локальной среды выполнения интеграции вы увидите следующее окно:

   :::image type="content" source="media/manage-integration-runtimes/successfully-registered.png" alt-text="Регистрация успешно выполнена.":::

## <a name="manage-a-self-hosted-integration-runtime"></a>Управление локальной средой выполнения интеграции

Локальную среду выполнения интеграции можно изменить, перейдя **в** **центр управления**, выбрав IR, а затем выбрав пункт изменить. Теперь можно обновить описание, скопировать ключ или повторно создать новые ключи.

:::image type="content" source="media/manage-integration-runtimes/edit-integration-runtime.png" alt-text="изменить IR.":::

:::image type="content" source="media/manage-integration-runtimes/edit-integration-runtime-settings.png" alt-text="изменение сведений о IR.":::

Локальную среду выполнения интеграции можно удалить, перейдя **в центр** управления, выбрав ее, а затем щелкнув **Удалить**. После удаления инфракрасной связи все текущие проверки, использующие ее, завершатся ошибкой.

## <a name="next-steps"></a>Дальнейшие действия

[Обнаружение удаленных ресурсов при проверке](concept-detect-deleted-assets.md)
