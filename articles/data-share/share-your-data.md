---
title: Руководство по Общий доступ за пределами организации в Azure Data Share
description: Руководство по совместному использованию данных клиентами и партнерами с помощью Azure Data Share
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: tutorial
ms.date: 03/24/2021
ms.openlocfilehash: 8e149270d8f98cbf72d3864d238a3d8ddfd61c67
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105639538"
---
# <a name="tutorial-share-data-using-azure-data-share"></a>Руководство по Совместное использование данных с помощью Azure Data Share  

Из этого руководства вы узнаете, как настроить новый Azure Data Share и начать предоставлять свои данные клиентам и партнерам за пределами вашей организации Azure. 

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * создать общий доступ к данным;
> * добавить наборы данных в общий доступ к данным;
> * включить расписание моментальных снимков для Data Share; 
> * добавить получателей для общего доступа к данным. 

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
* Адрес электронной почты для входа получателей в Azure (псевдонимы электронной почты не поддерживаются).
* Если исходное хранилище данных Azure находится в подписке Azure, отличной от той, которая будет использоваться для создания ресурса Data Share, зарегистрируйте [поставщик ресурсов Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) в подписке, в которой находится хранилище данных Azure. 

### <a name="share-from-a-storage-account"></a>Предоставление общего доступа к данным из учетной записи хранения

* Учетная запись хранения Azure. Если у вас ее еще нет, вы можете создать [Учетную запись службі хранилища Azure](../storage/common/storage-account-create.md)
* Разрешение на запись в учетную запись хранения, которая имеется в *Microsoft.Storage/storageAccounts/write*. Это разрешение назначено роли **участника**.
* Разрешение на добавление назначения ролей в учетную запись хранения, имеющуюся в *Microsoft.Authorization/role assignments/write*. Это разрешение назначено роли **владельца**. 


### <a name="share-from-a-sql-based-source"></a>Предоставление общего доступа к данным из источника на основе SQL
Ниже приведен список предварительных условий для предоставления общего доступа к данным из источника SQL. 

#### <a name="prerequisites-for-sharing-from-azure-sql-database-or-azure-synapse-analytics-formerly-azure-sql-dw"></a>Предварительные требования для предоставления общего доступа к данным Базы данных SQL Azure или Azure Synapse Analytics (ранее — хранилище данных SQL Azure)

* База данных SQL Azure или Azure Synapse Analytics (ранее — хранилище данных SQL Azure) с таблицами и представлениями, к которым вы хотите предоставить общий доступ.
* Разрешение на запись в базы данных на сервере SQL, которое присутствует в *Microsoft.Sql/servers/databases/write*. Это разрешение назначено роли **участника**.
* **Администратор Azure Active Directory** для сервера SQL.
* Доступ к брандмауэру SQL Server. Это можно обеспечить следующим образом. 
    1. На портале Azure перейдите к SQL Server. Выберите *Брандмауэры и виртуальные сети* в левой области навигации.
    1. Выберите значение **Да** для параметра *Разрешить доступ к серверу службам и ресурсам Azure*.
    1. Щелкните **+Добавить IP-адрес клиента**. IP-адрес клиента можно изменить. При следующем совместном использовании данных SQL из портала Azure может потребоваться повторение этого процесса. Вы можете также добавить диапазон IP-адресов.
    1. Выберите команду **Сохранить**. 

#### <a name="prerequisites-for-sharing-from-azure-synapse-analytics-workspace-sql-pool"></a>Необходимые условия для предоставления общего доступа к данным из пула SQL Azure Synapse Analytics (рабочая область)

* * Выделенный пул SQL Azure Synapse Analytics (рабочая область) с таблицами, к которым вы хотите предоставить общий доступ. Предоставление общего доступа к данным представления в настоящее время не поддерживается. Предоставление общего доступа к данным из бессерверного пула SQL в настоящее время не поддерживается.
* Разрешение на запись в пул SQL в рабочей области Synapse, которое предоставляется через *Microsoft.Synapse/workspaces/sqlPools/write*. Это разрешение назначено роли **участника**.
* Разрешение на доступ управляемого удостоверения Data Share к пулу SQL рабочей области Synapse. Это можно обеспечить следующим образом. 
    1. На портале Azure перейдите в рабочую область Synapse. Выберите на панели навигации слева "Администратор Active Directory для SQL" и назначьте себе роль **администратора Azure Active Directory**.
    1. Откройте Synapse Studio, затем выберите *Управление* в области навигации слева. Выберите *Управление доступом* в разделе "Безопасность". Назначьте себе роль **Администратор SQL** или **Администратор рабочей области**.
    1. В Synapse Studio в области навигации слева выберите *Разработка*. Выполните указанный ниже сценарий в пуле SQL, чтобы добавить управляемое удостоверение для ресурса Data Share в качестве db_datareader. 
    
        ```sql
        create user "<share_acct_name>" from external provider;     
        exec sp_addrolemember db_datareader, "<share_acct_name>"; 
        ```                   
       Обратите внимание на то, что *<share_acc_name>* — это имя вашего ресурса Data Share. Если вы еще не создали ресурс Data Share, вы можете вернуться к этому требованию позже.  

* Доступ к брандмауэру рабочей области Synapse. Это можно обеспечить следующим образом. 
    1. На портале Azure перейдите в рабочую область Synapse. В области навигации слева выберите *Брандмауэры*.
    1. Выберите значение **Вкл.** для параметра *Разрешить службам и ресурсам Azure доступ к этой рабочей области*.
    1. Щелкните **+Добавить IP-адрес клиента**. IP-адрес клиента можно изменить. При следующем совместном использовании данных SQL из портала Azure может потребоваться повторение этого процесса. Вы можете также добавить диапазон IP-адресов.
    1. Выберите команду **Сохранить**. 


### <a name="share-from-azure-data-explorer"></a>Предоставление общего доступа к данным из Azure Data Explorer
* Кластер Azure Data Explorer с базами данных, к которым вы хотите предоставить общий доступ.
* Разрешение на запись в кластер Azure Data Explorer, имеющийся в *Microsoft.Kusto/clusters/write*. Это разрешение назначено роли **участника**.

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="create-a-data-share-account"></a>Создание учетной записи для общего доступа к данным

### <a name="portal"></a>[Портал](#tab/azure-portal)

Создайте ресурс Azure Data Share в группе ресурсов Azure.

1. Нажмите кнопку меню в левом верхнем углу окна портала и выберите элемент **Создать ресурс** (+).

1. Найдите *Data Share*.

1. Выберите Data Share и команду **Создать**.

1. Укажите основные сведения о ресурсе Azure Data Share, указав следующую информацию. 

     **Параметр** | **Рекомендуемое значение** | **Описание поля**
    |---|---|---|
    | Подписка | Ваша подписка | Выберите подписку Azure, которую нужно использовать для своей учетной записи Data Share.|
    | Группа ресурсов | *testresourcegroup* | Воспользуйтесь существующей группой ресурсов или создайте новую. |
    | Расположение | *Восточная часть США 2* | Выберите регион для учетной записи Data Share.
    | Имя | *datashareaccount* | Укажите имя вашей учетной записи Data Share. |
    | | |

1. Выберите **Проверить и создать**, а затем **Создать**, чтобы подготовить учетную запись для общего ресурса данных. Подготовка новой учетной записи для общего доступа к данным обычно занимает около 2 минут или меньше. 

1. Когда развертывание будет завершено, выберите **Перейти ресурсу**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Создайте ресурс Azure Data Share в группе ресурсов Azure.

Чтобы подготовить среду для Azure CLI, выполните указанные ниже действия.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Чтобы создать ресурс, выполните следующие команды:

1. Выполните команду [az account set](/cli/azure/account#az_account_set), чтобы установить свою подписку в качестве текущей подписки по умолчанию:

   ```azurecli
   az account set --subscription 00000000-0000-0000-0000-000000000000
   ```

1. Выполните команду [az provider register](/cli/azure/provider#az_provider_register), чтобы зарегистрировать поставщик ресурсов:

   ```azurecli
   az provider register --name "Microsoft.DataShare"
   ```

1. Выполните команду [az group create](/cli/azure/group#az_group_create), чтобы создать группу ресурсов или использовать существующую группу ресурсов:

   ```azurecli
   az group create --name testresourcegroup --location "East US 2"
   ```

1. Выполните команду [az datashare account create](/cli/azure/ext/datashare/datashare/account#ext_datashare_az_datashare_account_create), чтобы создать учетную запись Data Share:

   ```azurecli
   az datashare account create --resource-group testresourcegroup --name datashareaccount --location "East US 2" 
   ```

   Выполните команду [az datashare account list](/cli/azure/ext/datashare/datashare/account#ext_datashare_az_datashare_account_list), чтобы просмотреть учетные записи Data Share:

   ```azurecli
   az datashare account list --resource-group testresourcegroup
   ```

---

## <a name="create-a-share"></a>Создание общей папки

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Перейдите на страницу "Обзор Data Share".

    ![Общий доступ к данным](./media/share-receive-data.png "Общий доступ к данным") 

1. Выберите команду **Начать совместное использование данных**.

1. Нажмите кнопку **создания**.   

1. Введите сведения о совместном использовании. Укажите имя, тип совместного использования, описание содержимого и условия использования (необязательно). 

    ![EnterShareDetails](./media/enter-share-details.png "Введите сведения об общем ресурсе") 

1. Выберите **Continue** (Продолжить).

1. Чтобы добавить наборы данных в общий ресурс, выберите **Добавить наборы данных**. 

    ![Добавление наборов данных в общий ресурс](./media/datasets.png "Наборы данных")

1. Выберите тип набора данных, который хотите добавить. Вы увидите другой список типов наборов данных в зависимости от типа ресурса (моментальный снимок или на странице), который вы выбрали на предыдущем этапе. При предоставлении общего доступа к данным из Базы данных SQL Azure или Azure Synapse Analytics (ранее — Хранилище SQL Azure) вам будет предложено указать метод аутентификации, чтобы отобразить таблицы. Выберите значение "Аутентификация AAD" и установите флажок **Разрешить Data Share выполнять указанный выше скрипт create user от моего имени**. 

    ![AddDatasets](./media/add-datasets.png "Добавление наборов данных")    

1. Перейдите к объекту, доступ к которому нужно предоставить, и выберите "Добавить наборы данных". 

    ![SelectDatasets](./media/select-datasets.png "Выбор наборов данных")    

1. На вкладке "Получатели" введите адреса электронной почты получателей данных, выбрав команду "+Добавить получателя". 

    ![AddRecipients](./media/add-recipient.png "Добавление получателей") 

1. Выберите **Continue** (Продолжить).

1. Если выбран тип общего доступа к моментальным снимкам, вы можете настроить расписание моментальных снимков для предоставления обновлений данных потребителям. 

    ![EnableSnapshots](./media/enable-snapshots.png "Включение моментальных снимков") 

1. Выберите время начала и интервал повторения. 

1. Выберите **Continue** (Продолжить).

1. На вкладке "Просмотр и создание" просмотрите содержимое пакета, параметры, получателей и параметры синхронизации. Нажмите кнопку **Создать**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Выполните команду [az storage account create](/cli/azure/storage/account#az_storage_account_create), чтобы создать Data Share:

   ```azurecli
   az storage account create --resource-group testresourcegroup --name ContosoMarketplaceAccount
   ```

1. Выполните команду [az storage container create](/cli/azure/storage/container#az_storage_container_create), чтобы создать контейнер для общей папки из предыдущей команды:

   ```azurecli
   az storage container create --name ContosoMarketplaceContainer --account-name ContosoMarketplaceAccount
   ```

1. Выполните команду [az datashare create](/cli/azure/ext/datashare/datashare#ext_datashare_az_datashare_create), чтобы создать Data Share:

   ```azurecli
   az datashare create --resource-group testresourcegroup \
     --name ContosoMarketplaceDataShare --account-name ContosoMarketplaceAccount \
     --description "Data Share" --share-kind "CopyBased" --terms "Confidential"
   ```

1. Выполните команду [az datashare invitation create](/cli/azure/ext/datashare/datashare/invitation#ext_datashare_az_datashare_invitation_create), чтобы создать приглашение для указанного адреса:

   ```azurecli
   az datashare invitation create --resource-group testresourcegroup \
     --name DataShareInvite --share-name ContosoMarketplaceDataShare \
     --account-name ContosoMarketplaceAccount --target-email "jacob@fabrikam"
   ```

---

Azure Data Share создан, и получатель Data Share готов принять ваше приглашение.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурс больше не нужен, перейдите на страницу **Общие сведения об общем ресурсе данных** и нажмите **Удалить**, чтобы удалить его.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как создать Azure Data Share и пригласить получателей. Чтобы узнать больше о том, как потребитель данных может получить и принять Data Share, см. пособие о том, как принять и получить данные.

> [!div class="nextstepaction"]
> [Руководство по Прием и получение данных с помощью Azure Data Share](subscribe-to-data-share.md)
