---
title: Руководство по принятию и получению данных в Azure Data Share
description: Руководство по принятию и получению данных с помощью службы Azure Data Share.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: tutorial
ms.date: 03/24/2021
ms.openlocfilehash: ccfda4975b6453ed67edc2640520bc0a76df5709
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105644878"
---
# <a name="tutorial-accept-and-receive-data-using-azure-data-share"></a>Руководство по Прием и получение данных с помощью Azure Data Share  

В этом руководстве вы узнаете, как принять приглашение в общий ресурс данных с помощью службы Azure Data Share, а также как получить данные, к которым вам предоставили доступ и включить интервал регулярного обновления, чтобы всегда иметь последний моментальный снимок этих данных. 

> [!div class="checklist"]
> * Как принять приглашение Azure Data Share
> * Создание учетной записи Azure Data Share
> * Указание места назначения данных.
> * Создание подписки на общий ресурс с данными для получения запланированного обновления.

## <a name="prerequisites"></a>Предварительные требования
Чтобы принять приглашение в общий ресурс данных, вам необходимо подготовить к работе ряд ресурсов Azure, которые перечислены ниже. 

Прежде чем принять приглашение в общий ресурс данных, выполните все предварительные требования. 

* Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
* Приглашение Data Share. Приглашение от Microsoft Azure с темой "Приглашение Azure Data Share от **<yourdataprovider@domain.com>** ".
* Зарегистрируйте [поставщик ресурсов Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) в подписке Azure, где вы создадите ресурс Data Share и подписку Azure, в которой находятся целевые хранилища данных Azure.

### <a name="receive-data-into-a-storage-account"></a>Получение данных в учетную запись хранения

* Учетная запись хранения Azure. Если у вас еще нет учетной записи хранения Azure, вы можете [создать ее](../storage/common/storage-account-create.md). 
* Разрешение на запись в учетную запись хранения, которая имеется в *Microsoft.Storage/storageAccounts/write*. Это разрешение назначено роли участника. 
* Разрешение на добавление назначения ролей в учетную запись хранения, имеющуюся в *Microsoft.Authorization/role assignments/write*. Это разрешение назначено роли владельца.  

### <a name="receive-data-into-a-sql-based-target"></a>Поступление данных в целевое расположение на основе SQL
Если вы решили принимать данные в Базу данных SQL Azure или Azure Synapse Analytics, изучите следующий список предварительных условий. 

#### <a name="prerequisites-for-receiving-data-into-azure-sql-database-or-azure-synapse-analytics-formerly-azure-sql-dw"></a>Необходимые условия для получения данных в Базу данных SQL Azure или Azure Synapse Analytics (прежнее название — Хранилище данных SQL Azure)

* База данных SQL Azure или Azure Synapse Analytics (прежнее название — Хранилище данных SQL).
* Разрешение на запись в базы данных на сервере SQL, которое присутствует в *Microsoft.Sql/servers/databases/write*. Это разрешение назначено роли **участника**. 
* **Администратор Azure Active Directory** для сервера SQL.
* Доступ к брандмауэру SQL Server. Это можно обеспечить следующим образом. 
    1. В SQL Server на портале Azure перейдите к разделу *Брандмауэры и виртуальные сети*
    1. Выберите значение **Да** для параметра *Разрешить доступ к серверу службам и ресурсам Azure*.
    1. Щелкните **+Добавить IP-адрес клиента**. IP-адрес клиента можно изменить. При следующем совместном использовании данных SQL из портала Azure может потребоваться повторение этого процесса. Вы можете также добавить диапазон IP-адресов.
    1. Выберите команду **Сохранить**. 
 
#### <a name="prerequisites-for-receiving-data-into-azure-synapse-analytics-workspace-sql-pool"></a>Необходимые условия для получения данных в пул SQL в Azure Synapse Analytics (рабочая область)

* Выделенный пул SQL в Azure Synapse Analytics (рабочая область). Получение данных в бессерверный пул SQL в настоящее время не поддерживается.
* Разрешение на запись в пул SQL в рабочей области Synapse, которое предоставляется через *Microsoft.Synapse/workspaces/sqlPools/write*. Это разрешение назначено роли **участника**.
* Разрешение на доступ управляемого удостоверения Data Share к пулу SQL рабочей области Synapse. Это можно обеспечить следующим образом. 
    1. На портале Azure перейдите в рабочую область Synapse. Выберите на панели навигации слева "Администратор Active Directory для SQL" и назначьте себе роль **администратора Azure Active Directory**.
    1. Откройте Synapse Studio, затем выберите *Управление* в области навигации слева. Выберите *Управление доступом* в разделе "Безопасность". Назначьте себе роль **Администратор SQL** или **Администратор рабочей области**.
    1. В Synapse Studio в области навигации слева выберите *Разработка*. Выполните в пуле SQL следующий скрипт, чтобы добавить ресурс управляемого удостоверения Data Share с ролями db_datareader, db_datawriter и db_ddladmin. 
    
        ```sql
        create user "<share_acc_name>" from external provider; 
        exec sp_addrolemember db_datareader, "<share_acc_name>"; 
        exec sp_addrolemember db_datawriter, "<share_acc_name>"; 
        exec sp_addrolemember db_ddladmin, "<share_acc_name>";
        ```                   
       Обратите внимание на то, что *<share_acc_name>* — это имя вашего ресурса Data Share. Если вы еще не создали ресурс Data Share, вы можете вернуться к этому требованию позже.  

* Доступ к брандмауэру рабочей области Synapse. Это можно обеспечить следующим образом. 
    1. На портале Azure перейдите в рабочую область Synapse. В области навигации слева выберите *Брандмауэры*.
    1. Выберите значение **Вкл.** для параметра *Разрешить службам и ресурсам Azure доступ к этой рабочей области*.
    1. Щелкните **+Добавить IP-адрес клиента**. IP-адрес клиента можно изменить. При следующем совместном использовании данных SQL из портала Azure может потребоваться повторение этого процесса. Вы можете также добавить диапазон IP-адресов.
    1. Выберите команду **Сохранить**. 

### <a name="receive-data-into-an-azure-data-explorer-cluster"></a>Получение данных в кластере Azure Data Explorer: 

* Кластер Azure Data Explorer в том же центре обработки данных Azure, что и кластер Azure Data Explorer поставщика данных: Если у вас еще нет кластера, вы можете создать [кластер Azure Data Explorer](/azure/data-explorer/create-cluster-database-portal). Если вы не знакомы с центром обработки данных Azure в кластере поставщика данных, вы можете создать кластер позже.
* Разрешение на запись в кластер Azure Data Explorer, имеющийся в *Microsoft.Kusto/clusters/write*. Это разрешение назначено роли участника. 

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="open-invitation"></a>Открытие приглашения

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Вы можете открыть приглашение из электронной почты или непосредственно из портала Azure. 

   Чтобы открыть приглашение из электронной почты, проверьте папку "Входящие" на наличие приглашения от поставщика данных. Это должно быть приглашение от Microsoft Azure с названием **Приглашение Azure Data Share от <yourdataprovider@domain.com>** . Выберите **Просмотреть приглашение**, чтобы увидеть ваше приглашение в Azure. 

   Чтобы открыть приглашение непосредственно из портала Azure, найдите на портале параметр **Приглашения Data Share**. После этого вы перейдете к списку приглашений Data Share.

   ![Список приглашений](./media/invitations.png "Список приглашений") 

1. Выберите общий ресурс для просмотра. 

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Подготовьте среду Azure CLI и просмотрите свои приглашения.

Чтобы подготовить среду для Azure CLI, выполните указанные ниже действия.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

С помощью команды [az datashare consumer invitation list](/cli/azure/ext/datashare/datashare/consumer/invitation#ext_datashare_az_datashare_consumer_invitation_list) получите список актуальных приглашений.

```azurecli
az datashare consumer invitation list --subscription 11111111-1111-1111-1111-111111111111
```

Скопируйте идентификатор приглашения, который вам потребуется в следующем разделе.

---

## <a name="accept-invitation"></a>Принятие приглашения

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Просмотрите все поля, в том числе **Условия использования**. Если вы согласны с условиями использования, установите флажок для подтверждения своего согласия. 

   ![Условия использования](./media/terms-of-use.png "Условия использования") 

1. В разделе *Target Data Share Account* (Целевая учетная запись Data Share) выберите подписку и группу ресурсов, куда будете развертывать общий ресурс данных. 

   В поле **учетной записи Data Share** выберите **Создать**, если у вас нет существующей учетной записи Data Share. В противном случае выберите учетную запись Data Share, где бы вы хотели принимать данные. 

   В поле **Received Share Name** (Имя полученного общего ресурса) можно оставить значение по умолчанию, заданное поставщиком данных, или указать для полученного общего ресурса новое имя. 

   Согласившись с условиями использования и указав учетную запись Data Share для управления полученным общим ресурсом, выберите **Accept and Configure** (Принять и настроить). Будет создана подписка на общий ресурс. 

   ![Принятие параметров](./media/accept-options.png "Принятие параметров") 

   После этого вы перейдете к полученному общему ресурсу в учетной записи Data Share. 

   Если вы не хотите принимать приглашение, выберите *Отклонить*. 

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Для создания общего ресурса данных используйте команду [az datashare consumer share-subscription create](/cli/azure/ext/datashare/datashare/consumer/share-subscription#ext_datashare_az_datashare_consumer_share_subscription_create).

```azurecli
az datashare consumer share-subscription create --resource-group share-rg \
  --name "Fabrikam Solutions" --account-name FabrikamDataShareAccount \
  --invitation-id 89abcdef-0123-4567-89ab-cdef01234567 \
  --source-share-location "East US 2" --subscription 11111111-1111-1111-1111-111111111111
```

---

## <a name="configure-received-share"></a>Настройка полученного общего ресурса

### <a name="portal"></a>[Портал](#tab/azure-portal)

Выполните следующие действия, чтобы настроить место для получения данных.

1. Перейдите на вкладку **Наборы данных**. Установите флажок рядом с набором данных, которому следует назначить место назначение. Нажмите **+ Сопоставить с целевым объектом**, чтобы выбрать целевое хранилище данных. 

   ![Кнопка "Сопоставить с целевым объектом"](./media/dataset-map-target.png "Сопоставление с целевым объектом") 

1. Выберите тип целевого хранилища данных, в котором должны находиться данные. Все файлы и таблицы данных в целевом хранилище данных с тем же путем и именем будут перезаписаны. Если вы получаете данные в базу данных SQL Azure или Azure Synapse Analytics (прежнее название — Azure SQL DW), установите флажок **Разрешить Data Share выполнять указанный выше скрипт create user от моего имени**.

   Для общего доступа на месте выберите хранилище данных в указанном расположении. Расположение — это центр обработки данных Azure, в котором находится исходное хранилище поставщика данных. После сопоставления набора данных можно перейти по ссылке в столбце целевого пути, чтобы получить доступ к данным.

   ![Целевая учетная запись хранения](./media/dataset-map-target-sql.png "Целевое хранилище") 

1. Для совместного использования на основе моментальных снимков можно также включить на вкладе **Расписание моментальных снимков** функцию расписания моментальных снимков при условии, что поставщик данных создал такое расписание для регулярного обновления данных. Установите флажок рядом с расписанием моментальных снимков и выберите **+ Включить**. Обратите внимание, что первый запланированный моментальный снимок будет запускаться в течение одной минуты в назначенное время, а последующие моментальные снимки — в течение нескольких секунд.

   ![Включение расписания моментальных снимков](./media/enable-snapshot-schedule.png "Включение расписания моментальных снимков")

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Следующие команды позволяют настроить место для получения данных.

1. Команда [az datashare consumer share-subscription list-source-dataset](/cli/azure/ext/datashare/datashare/consumer/share-subscription#ext_datashare_az_datashare_consumer_share_subscription_list_source_dataset) возвращает идентификатор набора данных:

   ```azurecli
   az datashare consumer share-subscription list-source-dataset \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" \
     --subscription 11111111-1111-1111-1111-111111111111 --query "[0].dataSetId"
   ```

1. Команда [az storage account create](/cli/azure/storage/account#az_storage_account_create) создает учетную запись хранения для указанного общего ресурса данных:

   ```azurecli
   az storage account create --resource-group "share-rg" --name "FabrikamDataShareAccount" \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

1. Команда [az storage account create](/cli/azure/storage/account#az_storage_account_show) возвращает идентификатор учетной записи хранения:

   ```azurecli
   az storage account show --resource-group "share-rg" --name "FabrikamDataShareAccount" \
     --subscription 11111111-1111-1111-1111-111111111111 --query "id"
   ```

1. Следующая команда возвращает идентификатор субъекта для учетной записи:

   ```azurecli
   az datashare account show --resource-group "share-rg" --name "cli_test_consumer_account" \
     --subscription 11111111-1111-1111-1111-111111111111 --query "identity.principalId"
   ```

1. Команда [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) создает назначение роли для субъекта учетной записи.

   ```azurecli
   az role assignment create --role "01234567-89ab-cdef-0123-456789abcdef" \
     --assignee-object-id 6789abcd-ef01-2345-6789-abcdef012345 
     --assignee-principal-type ServicePrincipal --scope 456789ab-cdef-0123-4567-89abcdef0123 \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

1. Следующая команда создает переменную для сопоставления на основе идентификатора набора данных:

   ```azurecli
   $mapping='{\"data_set_id\":\"' + $dataset_id + '\",\"container_name\":\"newcontainer\",
     \"storage_account_name\":\"datashareconsumersa\",\"kind\":\"BlobFolder\",\"prefix\":\"consumer\"}'
   ```

1. Команда [az datashare consumer dataset-mapping create](/cli/azure/ext/datashare/datashare/consumer/dataset-mapping#ext_datashare_az_datashare_consumer_dataset_mapping_create) создает сопоставление набора данных:

   ```azurecli
   az datashare consumer dataset-mapping create --resource-group "share-rg" \
     --name "consumer-data-set-mapping" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" --mapping $mapping \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

1. Команда [az datashare consumer share-subscription synchronization start](/cli/azure/ext/datashare/datashare/consumer/share-subscription/synchronization#ext_datashare_az_datashare_consumer_share_subscription_synchronization_start) запускает синхронизацию набора данных.

   ```azurecli
   az datashare consumer share-subscription synchronization start \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount"  \
     --share-subscription-name "Fabrikam Solutions" --synchronization-mode "Incremental" \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

   Команда [az datashare consumer share-subscription synchronization list](/cli/azure/ext/datashare/datashare/consumer/share-subscription/synchronization#ext_datashare_az_datashare_consumer_share_subscription_synchronization_list) возвращает список синхронизаций:

   ```azurecli
   az datashare consumer share-subscription synchronization list \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

   Команда [az datashare consumer share-subscription list-source-share-synchronization-setting](/cli/azure/ext/datashare/datashare/consumer/share-subscription#ext_datashare_az_datashare_consumer_share_subscription_list_source_share_synchronization_setting) возвращает параметры синхронизации, настроенные для указанного общего ресурса.

   ```azurecli
   az datashare consumer share-subscription list-source-share-synchronization-setting \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" --subscription 11111111-1111-1111-1111-111111111111
   ```

---

## <a name="trigger-a-snapshot"></a>Активация создания моментальных снимков

### <a name="portal"></a>[Портал](#tab/azure-portal)

Эти действия применимы только к общему доступу на основе моментальных снимков.

1. Вы можете активировать создание моментальных снимков, нажав на вкладке **Подробности** кнопку **Активировать моментальный снимок**. Здесь вы сможете активировать полные или добавочные моментальные снимки ваших данных. Если вы осуществляете прием данных от поставщика в первый раз, выберите полное копирование. 

   ![Активация создания моментального снимка](./media/trigger-snapshot.png "Активация создания моментального снимка") 

1. Если состояние последнего запуска является *успешным*, перейдите к целевому хранилищу данных для просмотра полученных данных. Выберите **Наборы данных** и щелкните ссылку в целевом пути. 

   ![Наборы данных объекта-получателя](./media/consumer-datasets.png "Сопоставление набора данных объекта-получателя") 

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Команда [az datashare consumer trigger create](/cli/azure/ext/datashare/datashare/consumer/trigger#ext_datashare_az_datashare_consumer_trigger_create) запускает создание моментального снимка:

```azurecli
az datashare consumer trigger create --resource-group "share-rg" \
  --name "share_test_trigger" --account-name "FabrikamDataShareAccount" \
  --share-subscription-name "Fabrikam Solutions" --recurrence-interval "Day" \
  --synchronization-time "2020-04-23 18:00:00 +00:00" --kind ScheduleBased \
  --subscription 11111111-1111-1111-1111-111111111111
```

> [!NOTE]
> Используйте эту команду только для того, чтобы предоставлять доступ к данным через моментальные снимки.

---

## <a name="view-history"></a>Просмотр истории
Этот шаг применим только для общего доступа на основе моментальных снимков. Чтобы просмотреть журнал моментальных снимков, выберите вкладку **Журнал**. Здесь вы найдете журнал всех моментальных снимков за последние 30 дней.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурс больше не нужен, перейдите на страницу **Общие сведения об общем ресурсе данных** и нажмите **Удалить**, чтобы удалить его.

## <a name="next-steps"></a>Дальнейшие действия
В этом учебнике вы узнали, как принять приглашение и получить данные Azure Data Share. Чтобы узнать больше о понятиях Azure Data Share, ознакомьтесь с терминологией Azure Data Share.

> [!div class="nextstepaction"]
> [Понятия Azure Data Share](terminology.md)