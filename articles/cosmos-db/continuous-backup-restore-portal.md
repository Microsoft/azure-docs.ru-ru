---
title: Настройка непрерывного резервного копирования и восстановление на момент времени с помощью портал Azure в Azure Cosmos DB.
description: Узнайте, как определить точку восстановления и настроить непрерывную архивацию с помощью портал Azure. В нем показано, как восстановить активную и удаленную учетную запись.
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 02/01/2021
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: ee6eedbc078e1b9c07ed00922ce1c37b38410128
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100381874"
---
# <a name="configure-and-manage-continuous-backup-and-point-in-time-restore-preview---using-azure-portal"></a>Настройка непрерывного резервного копирования и восстановление на момент времени (Предварительная версия) и управление ими с помощью портал Azure
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

> [!IMPORTANT]
> Функция восстановления на момент времени (режим непрерывного резервного копирования) для Azure Cosmos DB в настоящее время находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Функция восстановления на момент времени (Предварительная версия) Azure Cosmos DB позволяет восстановиться после случайного изменения в контейнере, восстановления удаленной учетной записи, базы данных или контейнера или восстановления в любом регионе (где существовали резервные копии). Режим непрерывного резервного копирования позволяет выполнять восстановление на любой момент времени в течение последних 30 дней.

В этой статье описывается, как определить точку восстановления и настроить непрерывное резервное копирование с помощью портал Azure.

## <a name="provision-an-account-with-continuous-backup"></a><a id="provision"></a>Предоставление учетной записи с непрерывным резервным копированием

При создании новой учетной записи Azure Cosmos DB для параметра **политика резервного копирования** выберите **непрерывный** режим, чтобы включить функцию восстановления на момент времени для новой учетной записи. После включения этой функции для учетной записи все базы данных и контейнеры будут доступны для непрерывного резервного копирования. При восстановлении до точки во времени данные всегда восстанавливаются в новую учетную запись. в настоящее время восстановление в существующую учетную запись невозможно.

:::image type="content" source="./media/continuous-backup-restore-portal/configure-continuous-backup-portal.png" alt-text="Подготавливает учетную запись Azure Cosmos DB с конфигурацией непрерывного резервного копирования." border="true":::

## <a name="restore-a-live-account-from-accidental-modification"></a><a id="restore-live-account"></a>Восстановление действующей учетной записи из случайного изменения

С помощью портал Azure можно восстановить действующую учетную запись или выбранные базы данных и контейнеры. Чтобы восстановить данные, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Перейдите к учетной записи Azure Cosmos DB и откройте панель восстановления на момент **времени** .

   > [!NOTE]
   > Панель восстановление в портал Azure заполняется только при наличии `Microsoft.DocumentDB/locations/restorableDatabaseAccounts/*/read` разрешения. Дополнительные сведения о настройке этого разрешения см. в статье разрешения на [резервное копирование и восстановление](continuous-backup-restore-permissions.md) .

1. Укажите следующие сведения для восстановления:

   * **Точка восстановления (UTC)** — метка времени за последние 30 дней. Учетная запись должна существовать по этой метке времени. Точку восстановления можно указать в формате UTC. Оно может быть близко к второму, когда вы хотите восстановить его. Выберите ссылку **щелкните здесь** , чтобы получить справку по [идентификации точки восстановления](#event-feed).

   * **Location** — регион назначения, в котором была восстановлена учетная запись. Учетная запись должна существовать в этом регионе в заданной метке времени (например, Западная часть США или Восточная часть США). Учетную запись можно восстановить только в тех регионах, в которых исходная учетная запись существовала.

   * **RESTORE Resource** — можно выбрать **всю учетную запись** или **выбранную базу данных или контейнер** для восстановления. Базы данных и контейнеры должны существовать по заданной метке времени. В зависимости от выбранной точки восстановления и расположения, Restore Resources заполняется, что позволяет пользователю выбрать определенные базы данных или контейнеры, которые необходимо восстановить.

   * **Группа ресурсов** — группа ресурсов, в которой будет создана и восстановлена Целевая учетная запись. Группа ресурсов уже должна существовать.

   * **Восстановить целевую учетную запись** — имя целевой учетной записи. Имя целевой учетной записи должно соответствовать тем же рекомендациям, что и при создании новой учетной записи. Эта учетная запись будет создана процессом восстановления в том же регионе, в котором находится исходная учетная запись.
 
   :::image type="content" source="./media/continuous-backup-restore-portal/restore-live-account-portal.png" alt-text="Восстановите действующую учетную запись от случайного изменения портал Azure." border="true":::

1. Выбрав указанные выше параметры, нажмите кнопку Submit ( **Отправить** ), чтобы начать восстановление. Стоимость восстановления — это одноразовая плата, основанная на объеме данных и расходах на хранилище в данном регионе. Дополнительные сведения см. в разделе о [ценах](continuous-backup-restore-introduction.md#continuous-backup-pricing) .

## <a name="use-event-feed-to-identify-the-restore-time"></a><a id="event-feed"></a>Использование канала событий для задания времени восстановления

Если при заполнении точки восстановления в портал Azure вам нужна помощь с определением точки восстановления, выберите ссылку **щелкните здесь** , чтобы перейти в колонку "веб-канал событий". Веб-канал событий предоставляет полный список точности событий Create, Replace, DELETE в базах данных и контейнерах исходной учетной записи. 

Например, если требуется восстановить до точки перед удалением или обновлением определенного контейнера, проверьте этот канал событий. События отображаются в хронологическом порядке по убыванию времени, когда они произошли, с последними событиями в верхней части. Вы можете просмотреть результаты и выбрать время до или после события, чтобы еще больше сузить время.

:::image type="content" source="./media/continuous-backup-restore-portal/event-feed-portal.png" alt-text="Используйте канал событий для указания времени точки восстановления." border="true":::

> [!NOTE]
> Веб-канал событий не отображает изменения в ресурсах элементов. Вы всегда можете вручную указать любую метку времени в течение последних 30 дней (при условии, что учетная запись существует в это время) для восстановления.

## <a name="restore-a-deleted-account"></a><a id="restore-deleted-account"></a>Восстановление удаленной учетной записи

Портал Azure можно использовать для полного восстановления удаленной учетной записи в течение 30 дней после ее удаления. Чтобы восстановить удаленную учетную запись, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Поиск *Azure Cosmos DB* ресурсов в глобальной строке поиска. В ней перечислены все существующие учетные записи.
1. Затем нажмите кнопку **восстановить** . На панели восстановление отображается список удаленных учетных записей, которые могут быть восстановлены в течение срока хранения, что составляет 30 дней со времени удаления.
1. Выберите учетную запись, которую требуется восстановить.

   :::image type="content" source="./media/continuous-backup-restore-portal/restore-deleted-account-portal.png" alt-text="Восстановление удаленной учетной записи из портал Azure." border="true":::

   > [!NOTE]
   > Примечание. Панель восстановления в портал Azure заполняется только при наличии `Microsoft.DocumentDB/locations/restorableDatabaseAccounts/*/read` разрешения. Дополнительные сведения о настройке этого разрешения см. в статье разрешения на [резервное копирование и восстановление](continuous-backup-restore-permissions.md) .

1. Выберите учетную запись для восстановления и введите следующие сведения для восстановления удаленной учетной записи:

   * **Точка восстановления (UTC)** — метка времени за последние 30 дней. Учетная запись должна существовать в этой метке времени. Укажите точку восстановления в формате UTC. Оно может быть близко к второму, когда вы хотите восстановить его.

   * **Location** — регион назначения, в котором требуется восстановить учетную запись. Исходная учетная запись должна существовать в этом регионе в заданной метке времени. Например, Западная часть США или Восточная часть США.  

   * **Группа ресурсов** — группа ресурсов, в которой будет создана и восстановлена Целевая учетная запись. Группа ресурсов уже должна существовать.

   * **Восстановить целевую учетную запись** — имя целевой учетной записи должно соответствовать тем же рекомендациям, что и при создании новой учетной записи. Эта учетная запись будет создана процессом восстановления в том же регионе, в котором находится исходная учетная запись.

## <a name="track-the-status-of-restore-operation"></a><a id="track-restore-status"></a>Отслеживание состояния операции восстановления

После инициализации операции восстановления щелкните значок колокольчика **уведомления** в правом верхнем углу портала. Она предоставляет ссылку, отображающую состояние восстанавливаемой учетной записи. Пока выполняется восстановление, состояние учетной записи будет *создаваться* после завершения операции восстановления состояние учетной записи изменится на "в *сети*".

:::image type="content" source="./media/continuous-backup-restore-portal/track-restore-operation-status.png" alt-text="Состояние восстановленной учетной записи изменяется с создания на в сети после завершения операции." border="true":::

## <a name="next-steps"></a>Дальнейшие действия

* Настройка непрерывного резервного копирования и управление им с помощью [Azure CLI](continuous-backup-restore-command-line.md), [PowerShell](continuous-backup-restore-powershell.md)или [Azure Resource Manager](continuous-backup-restore-template.md).
* [Модель ресурсов режима непрерывного резервного копирования](continuous-backup-restore-resource-model.md)
* [Управление разрешениями](continuous-backup-restore-permissions.md) , необходимыми для восстановления данных в режиме непрерывной архивации.
