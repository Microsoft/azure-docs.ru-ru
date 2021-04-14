---
title: Учебник. Приступая к работе. Добавление администратора
description: Из этого учебника вы узнаете, как добавить в рабочую область еще одного пользователя с правами администратора.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: workspace
ms.topic: tutorial
ms.date: 04/02/2021
ms.openlocfilehash: 8b854dfcff7dfb4d03038b542308b6f3ebb6d491
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106553504"
---
# <a name="add-an-administrator-to-your-synapse-workspace"></a>Добавление администратора в рабочую область Synapse

Из этого учебника вы узнаете, как добавить администратора в рабочую область Synapse. Этот пользователь будет иметь полный контроль над рабочей областью.

## <a name="overview"></a>Обзор

До этого момента в руководстве по началу работы мы говорили о действиях в рабочей области, которые выполняются *вами*. Так как вы создали рабочую область на шаге 1, вы являетесь администратором рабочей области Synapse. Теперь мы создадим другого пользователя (Райана, `ryan@contoso.com`) с правами администратора. Когда все будет готово, Райан сможет выполнять в рабочей области все те же действия, что и вы.

## <a name="azure-rbac-owner-role-for-the-workspace"></a>Azure RBAC: роль владельца для рабочей области

Назначьте `ryan@contoso.com` роль **Владелец** Azure RBAC в рабочей области.

1. Откройте портал Azure и рабочую область Synapse.
1. Слева выберите **Управление доступом (IAM)** .
1. Добавьте роль **Владелец** для `ryan@contoso.com`. 
1. Выберите команду **Сохранить**.
 
 
## <a name="synapse-rbac-synapse-administrator-role-for-the-workspace"></a>Synapse RBAC: роль "Администратор Synapse" для рабочей области

Назначьте `ryan@contoso.com` роль Synapse RBAC **Администратор Synapse** для рабочей области.

1. Откройте рабочую область в Synapse Studio.
1. Слева щелкните **Управление**, чтобы открыть центр управления.
1. В разделе **Безопасность** щелкните **Управление доступом**.
1. Нажмите кнопку **Добавить**.
1. Для параметра **Область** оставьте значение **Рабочая область**.
1. Добавьте роль **Администратор Synapse** для `ryan@contoso.com`. 
1. Нажмите кнопку **Применить**:
 
## <a name="azure-rbac-role-assignments-on-the-primary-storage-account"></a>Azure RBAC: назначения ролей в основной учетной записи хранения

Назначьте `ryan@contoso.com` роль **Владелец** в основной учетной записи хранения рабочей области.
Назначьте `ryan@contoso.com` роль **Участник для данных BLOB-объектов хранилища Azure** в основной учетной записи хранения рабочей области.

1. Откройте основную учетную запись хранения рабочей области на портале Azure.
1. Слева щелкните **Управление доступом (IAM)** .
1. Добавьте роль **Владелец** для `ryan@contoso.com`. 
1. Добавьте роль **Участник для данных BLOB-объектов хранилища Azure** для `ryan@contoso.com`

## <a name="dedicated-sql-pools-db_owner-role"></a>Выделенные пулы SQL: роль db_owner

Назначьте `ryan@contoso.com` роль **db_owner** для каждого выделенного пула SQL в рабочей области.

```
CREATE USER [ryan@contoso.com] FROM EXTERNAL PROVIDER; 
EXEC sp_addrolemember 'db_owner', 'ryan@contoso.com'
```

## <a name="next-steps"></a>Дальнейшие действия

* [Начало работы с Azure Synapse Analytics](get-started.md)
* [Создание рабочей области](quickstart-create-workspace.md)
* [Использование бессерверного пула SQL](quickstart-sql-on-demand.md)
