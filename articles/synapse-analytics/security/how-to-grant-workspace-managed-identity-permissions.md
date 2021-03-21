---
title: Предоставление разрешений управляемому удостоверению в рабочей области синапсе
description: Статья, в которой объясняется, как настроить разрешения для управляемого удостоверения в рабочей области Azure синапсе.
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 04/15/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: 46fe27205a2c30d6cb2319bf620c6fe1ee34c31e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96459012"
---
# <a name="grant-permissions-to-workspace-managed-identity"></a>Предоставление разрешений управляемому удостоверению рабочей области

В этой статье описывается, как предоставить разрешения управляемому удостоверению в рабочей области Azure синапсе. Разрешения, в свою очередь, разрешают доступ к выделенным пулам SQL в рабочей области и ADLS 2-го поколения учетной записи хранения с помощью портал Azure.

>[!NOTE]
>Это управляемое удостоверение рабочей области будет называться управляемым удостоверением далее в этом документе.

## <a name="grant-managed-identity-permissions-to-the-dedicated-sql-pool"></a>Предоставление разрешений управляемого удостоверения выделенному пулу SQL

Управляемое удостоверение предоставляет разрешения для выделенных пулов SQL в рабочей области. С предоставленными разрешениями можно управлять конвейерами, выполняющими выделенные действия, связанные с пулом SQL. При создании рабочей области синапсе для Azure с помощью портал Azure можно предоставить разрешения управляемого управления удостоверениями для выделенных пулов SQL.

При создании рабочей области Azure синапсе выберите **Безопасность** . Затем выберите **Разрешить конвейеры (под учетной записью назначения в рабочей области) для доступа к пулам SQL.**

![Разрешение CONTROL на выделенные пулы SQL](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-16.png)

## <a name="grant-the-managed-identity-permissions-to-adls-gen2-storage-account"></a>Предоставление разрешений управляемого удостоверения ADLS 2-го поколения учетной записи хранения

Для создания рабочей области синапсе Azure требуется учетная запись хранения ADLS 2-го поколения. Чтобы успешно запустить пулы Spark в рабочей области Azure синапсе, управляемому удостоверению Azure синапсе требуется роль *участника данных BLOB-объекта хранилища* в этой учетной записи хранения. Механизм оркестрации в Azure синапсе также дает преимущества от этой роли.

### <a name="grant-permissions-to-managed-identity-during-workspace-creation"></a>Предоставление разрешений управляемому удостоверению во время создания рабочей области

Azure синапсе будет пытаться предоставить управляемому удостоверению роль участника данных BLOB-объекта хранилища после создания рабочей области Azure синапсе с помощью портал Azure. Сведения об учетной записи хранения ADLS 2-го поколения можно указать на вкладке " **основы** ".

![Вкладка "Основные сведения" в последовательности создания рабочей области](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-1.png)

Выберите учетную запись хранения ADLS 2-го поколения и файловую систему в поле **имя учетной записи** и **имя файловой системы**.

![Предоставление сведений об учетной записи хранения ADLS 2-го поколения](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-2.png)

Если создатель рабочей области также является **владельцем** учетной записи хранения ADLS 2-го поколения, Azure синапсе присвоит управляемому удостоверению роль *участника данных BLOB-объекта хранилища* . После введенных данных учетной записи хранения вы увидите следующее сообщение.

![Успешное назначение участника данных BLOB-объекта хранилища](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-3.png)

Если создатель рабочей области не является владельцем учетной записи хранения ADLS 2-го поколения, Azure синапсе не назначает управляемому удостоверению роль *участника данных BLOB-объекта хранилища* . Сообщение, которое отображается под данными учетной записи хранения, уведомляет создателя рабочей области о том, что у него нет достаточных разрешений для предоставления роли *участника данных BLOB-объекта хранилища* управляемому удостоверению.

![Неудачное назначение участника данных BLOB-объекта хранилища](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-4.png)

Как указано в сообщении, нельзя создавать пулы Spark, если только *участник данных BLOB-объектов хранилища* не назначен управляемому удостоверению.

### <a name="grant-permissions-to-managed-identity-after-workspace-creation"></a>Предоставление разрешений управляемому удостоверению после создания рабочей области

Если при создании рабочей области *участник данных BLOB-объекта хранилища* не назначается управляемому удостоверению, **владелец** ADLS 2-го поколения учетной записи хранения вручную назначает эту роль удостоверению. Следующие шаги помогут выполнить назначение вручную.

#### <a name="step-1-navigate-to-the-adls-gen2-storage-account-in-azure-portal"></a>Шаг 1. Перейдите к учетной записи хранения ADLS 2-го поколения в портал Azure

В портал Azure откройте учетную запись хранения ADLS 2-го поколения и выберите **Обзор** в левой области навигации. Вам потребуется назначить роль *участника данных большого двоичного объекта хранилища* на уровне контейнера или файловой системы. Выберите **Контейнеры**.  
![Общие сведения об учетной записи хранения ADLS 2-го поколения](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-5.png)

#### <a name="step-2-select-the-container"></a>Шаг 2. Выбор контейнера

Управляемое удостоверение должно иметь доступ к данным контейнера (файловой системы), который был предоставлен при создании рабочей области. Этот контейнер или файловую систему можно найти в портал Azure. Откройте рабочую область Azure синапсе в портал Azure и выберите вкладку **Обзор** в левой области навигации.
![Контейнер учетной записи хранения ADLS 2-го поколения](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-7.png)


Выберите тот же контейнер или файловую систему, чтобы предоставить управляемому удостоверению роль *участника данных BLOB-объекта хранилища* .
![Снимок экрана, показывающий контейнер или файловую систему, которые следует выбрать.](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-6.png)

#### <a name="step-3-navigate-to-access-control"></a>Шаг 3. Переход к контролю доступа

Выберите **Управление доступом (IAM)** .

![Управление доступом (IAM)](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-8.png)

#### <a name="step-4-add-a-new-role-assignment"></a>Шаг 4. Добавление нового назначения роли

Щелкните **+ Добавить**.

![Добавить новое назначение роли](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-9.png)

#### <a name="step-5-select-the-azure-role"></a>Шаг 5. Выбор роли Azure

Выберите роль **участника данных BLOB-объекта хранилища** .

![Выберите роль Azure](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-10.png)

#### <a name="step-6-select-the-azure-ad-security-principal"></a>Шаг 6. Выбор субъекта безопасности Azure AD

Выберите **пользователя Azure AD, группу или субъект-службу** в раскрывающемся списке **назначить доступ** .

![Выбор субъекта безопасности AAD](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-11.png)

#### <a name="step-7-search-for-the-managed-identity"></a>Шаг 7. Поиск управляемого удостоверения

Имя управляемого удостоверения также является именем рабочей области. Найдите управляемое удостоверение, введя имя рабочей области Azure синапсе в окне **выбора**. В списке должно отобразиться управляемое удостоверение.

![Поиск управляемого удостоверения](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-12.png)

#### <a name="step-8-select-the-managed-identity"></a>Шаг 8. Выбор управляемого удостоверения

Выберите управляемое удостоверение для **выбранных элементов**. Нажмите кнопку **сохранить** , чтобы добавить назначение ролей.

![Выбор управляемого удостоверения](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-13.png)

#### <a name="step-9-verify-that-the-storage-blob-data-contributor-role-is-assigned-to-the-managed-identity"></a>Шаг 9. Проверка назначения роли участника данных BLOB-объекта хранилища управляемому удостоверению

Выберите **Управление доступом (IAM)** , а затем выберите **назначения ролей**.

![Проверка назначения роли](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-14.png)

Управляемое удостоверение должно отображаться в разделе **участник данных BLOB-объекта хранилища** с назначенной ему ролью *участника данных BLOB-объекта хранилища* . 
![ADLS 2-го поколения выбор контейнера учетной записи хранения](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-15.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об [управляемом удостоверении рабочей области](./synapse-workspace-managed-identity.md)
