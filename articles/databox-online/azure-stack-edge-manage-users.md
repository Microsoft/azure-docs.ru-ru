---
title: Azure Stack пограничной Pro FPGA Управление пользователями | Документация Майкрософт
description: Описывает, как использовать портал Azure для управления пользователями на Azure Stack крае Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 01/05/2021
ms.author: alkohli
ms.openlocfilehash: 27ca190f3bad7f75175e5206d48e13dae1f5687e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97913354"
---
# <a name="use-the-azure-portal-to-manage-users-on-your-azure-stack-edge-pro-fpga"></a>Использование портал Azure для управления пользователями на Azure Stackном пограничном мобильном FPGA

В этой статье описывается, как управлять пользователями на устройстве Azure Stack ребра Pro FPGA. Вы можете управлять Azure Stack пограничным Pro через портал Azure или через локальный веб-интерфейс. Используйте портал Azure, чтобы добавлять, изменять или удалять пользователей.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Добавление пользователей
> * Изменение пользователя
> * Удаление пользователя

## <a name="about-users"></a>Сведения о пользователях

У пользователей может быть доступ только для чтения или полный доступ. Как указано в определении, пользователи с доступом только для чтения могут только просматривать данные в общем файловом ресурсе. Пользователи с полным доступом могут просматривать данные в общем файловом ресурсе, выполнять запись в общие папки, а также изменять или удалять данные из общей папки.

 - **Пользователь с правами полного доступа** — это локальный пользователь, у которого есть полный доступ.
 - **Пользователь с правами только для чтения** — это локальный пользователь, у которого есть доступ только для чтения. Эти пользователи связаны с общими папками, которые позволяют выполнять операции только для чтения.

Сначала разрешения пользователя определяются при его создании во время создания общей папки. Изменение разрешений на уровне общего ресурса сейчас не поддерживается.

## <a name="add-a-user"></a>Добавление пользователей

Выполните на портале Azure шаги ниже, чтобы добавить пользователя.

1. В портал Azure перейдите к ресурсу Azure Stackного периметра и перейдите к разделу **Пользователи**. На панели команд выберите **+ Добавить пользователя** .

    ![Добавление пользователя](media/azure-stack-edge-manage-users/add-user-1.png)

2. Укажите имя и пароль для пользователя, которого вы хотите добавить. Подтвердите пароль и щелкните **Добавить**.

    ![Указание имени пользователя и пароля](media/azure-stack-edge-manage-users/add-user-2.png)

    > [!IMPORTANT] 
    > Эти пользователи зарезервированы системой и не должны использоваться: Administrator, EdgeUser, EdgeSupport, HcsSetupUser, WDAGUtilityAccount, CLIUSR, DefaultAccount, Guest.  

3. Вы получите уведомление о начале и завершении создания пользователя. После создания пользователя выберите **Обновить** на панели команд, чтобы просмотреть обновленный список пользователей.


## <a name="modify-user"></a>Изменение пользователя

Вы можете изменить пароль, связанный с пользователем, после создания пользователя. Выберите пользователя из списка. Введите и подтвердите новый пароль. Сохраните изменения.
 
![Изменение пользователя](media/azure-stack-edge-manage-users/modify-user-1.png)


## <a name="delete-a-user"></a>Удаление пользователя

Чтобы удалить пользователя, выполните следующие действия на портале Azure.


1. В портал Azure перейдите к ресурсу Azure Stackного периметра и перейдите к разделу **Пользователи**.

    ![Выбор пользователя, которого нужно удалить](media/azure-stack-edge-manage-users/delete-user-1.png)

2. Выберите пользователя из списка, а затем щелкните **Удалить**.  

   ![Выбор "Удалить"](media/azure-stack-edge-manage-users/delete-user-2.png)

3. При появлении запроса подтвердите удаление. 

   ![Подтверждение удаления](media/azure-stack-edge-manage-users/delete-user-3.png)

Список пользователей обновляется с учетом удаления.

![Обновленный список пользователей](media/azure-stack-edge-manage-users/delete-user-4.png)


## <a name="next-steps"></a>Дальнейшие действия

- Узнайте об [управлении пропускной способностью](azure-stack-edge-manage-bandwidth-schedules.md).
