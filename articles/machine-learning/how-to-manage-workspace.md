---
title: Создание рабочих областей на портале
titleSuffix: Azure Machine Learning
description: Узнайте, как создавать, просматривать и удалять рабочие области Машинное обучение Azure в портал Azure или с помощью пакета SDK для Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sgilley
author: sdgilley
ms.date: 09/30/2020
ms.topic: conceptual
ms.custom: how-to, fasttrack-edit
ms.openlocfilehash: 472bc66c75881d622e8ecfe23031f58db773a919
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102518931"
---
# <a name="create-and-manage-azure-machine-learning-workspaces"></a>Создание рабочих областей Машинное обучение Azure и управление ими 

В этой статье вы создадите, просмотрите и удалите [**машинное обучение Azure рабочие области**](concept-workspace.md) для [Машинное обучение Azure](overview-what-is-azure-ml.md), используя портал Azure или [пакет SDK для Python](/python/api/overview/azure/ml/) .

По мере необходимости изменения или требования к автоматизации можно также создавать и удалять рабочие области [с помощью интерфейса командной строки](reference-azure-machine-learning-cli.md)или с помощью [расширения VS Code](tutorial-setup-vscode-extension.md).

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Опробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree) уже сегодня.
* При использовании пакета SDK для Python [установите пакет SDK](/python/api/overview/azure/ml/install).

## <a name="limitations"></a>Ограничения

[!INCLUDE [register-namespace](../../includes/machine-learning-register-namespace.md)]

По умолчанию при создании рабочей области также создается реестр контейнеров Azure (запись контроля доступа).  Так как запись контроля доступа в настоящее время не поддерживает символы Юникода в именах групп ресурсов, используйте группу ресурсов, которая не содержит этих символов.

## <a name="create-a-workspace"></a>Создание рабочей области

# <a name="python"></a>[Python](#tab/python)

* **Спецификация по умолчанию.** По умолчанию зависимые ресурсы, а также группа ресурсов будут созданы автоматически. Этот код создает рабочую область с именем `myworkspace` и группу ресурсов `myresourcegroup` в `eastus2` .
    
    ```python
    from azureml.core import Workspace
    
    ws = Workspace.create(name='myworkspace',
                   subscription_id='<azure-subscription-id>',
                   resource_group='myresourcegroup',
                   create_resource_group=True,
                   location='eastus2'
                   )
    ```
    Задайте значение `create_resource_group` false, если у вас есть группа ресурсов Azure, которую вы хотите использовать для рабочей области.

* <a name="create-multi-tenant"></a>**Несколько клиентов.**  Если у вас несколько учетных записей, добавьте идентификатор клиента Azure Active Directory, который вы хотите использовать.  Найдите идентификатор клиента из [портал Azure](https://portal.azure.com) в разделе **Azure Active Directory внешние удостоверения**.

    ```python
    from azureml.core.authentication import InteractiveLoginAuthentication
    from azureml.core import Workspace
    
    interactive_auth = InteractiveLoginAuthentication(tenant_id="my-tenant-id")
    ws = Workspace.create(name='myworkspace',
                subscription_id='<azure-subscription-id>',
                resource_group='myresourcegroup',
                create_resource_group=True,
                location='eastus2',
                auth=interactive_auth
                )
    ```

* **[Независимых Cloud](reference-machine-learning-cloud-parity.md)**. Вам потребуется дополнительный код для проверки подлинности в Azure, если вы работаете в независимых облаке.

    ```python
    from azureml.core.authentication import InteractiveLoginAuthentication
    from azureml.core import Workspace
    
    interactive_auth = InteractiveLoginAuthentication(cloud="<cloud name>") # for example, cloud="AzureUSGovernment"
    ws = Workspace.create(name='myworkspace',
                subscription_id='<azure-subscription-id>',
                resource_group='myresourcegroup',
                create_resource_group=True,
                location='eastus2',
                auth=interactive_auth
                )
    ```

* **Используйте существующие ресурсы Azure**.  Вы также можете создать рабочую область, которая использует существующие ресурсы Azure с форматом идентификатора ресурса Azure. Поиск конкретных идентификаторов ресурсов Azure в портал Azure или с помощью пакета SDK. В этом примере предполагается, что группа ресурсов, учетная запись хранения, хранилище ключей, App Insights и реестр контейнеров уже существуют.

   ```python
   import os
   from azureml.core import Workspace
   from azureml.core.authentication import ServicePrincipalAuthentication

   service_principal_password = os.environ.get("AZUREML_PASSWORD")

   service_principal_auth = ServicePrincipalAuthentication(
      tenant_id="<tenant-id>",
      username="<application-id>",
      password=service_principal_password)

                        auth=service_principal_auth,
                             subscription_id='<azure-subscription-id>',
                             resource_group='myresourcegroup',
                             create_resource_group=False,
                             location='eastus2',
                             friendly_name='My workspace',
                             storage_account='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.storage/storageaccounts/mystorageaccount',
                             key_vault='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.keyvault/vaults/mykeyvault',
                             app_insights='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.insights/components/myappinsights',
                             container_registry='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.containerregistry/registries/mycontainerregistry',
                             exist_ok=False)
   ```

Дополнительные сведения см. в статье [Справочник по пакету SDK для рабочей области](/python/api/azureml-core/azureml.core.workspace.workspace).

Если у вас возникли проблемы при доступе к подписке, см. статью [Настройка проверки подлинности для машинное обучение Azure ресурсов и рабочих процессов](how-to-setup-authentication.md), а также [Проверка подлинности в машинное обучение Azure](https://aka.ms/aml-notebook-auth) записной книжке.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Войдите на [портал Azure](https://portal.azure.com/) с помощью учетных данных вашей подписки Azure. 

1. На портале Azure вверху слева щелкните **+ Создать ресурс**.

      ![Создать новый ресурс](./media/how-to-manage-workspace/create-workspace.gif)

1. С помощью строки поиска выполните поиск по запросу **Машинное обучение**.

1. Выберите **Машинное обучение**.

1. В области **Машинное обучение** выберите **Создать**.

1. Укажите следующие сведения для настройки новой рабочей области:

   Поле|Описание 
   ---|---
   имя рабочей области. |Введите уникальное имя для идентификации рабочей области. В этом примере мы используем **docs-ws**. Имена должны быть уникальными в группе ресурсов. Используйте имя, которое позволит легко запомнить рабочую область и отличить ее от областей, созданных другими пользователями. В имени рабочей области не учитывается регистр.
   Подписка |Выберите подписку Azure, которую нужно использовать.
   Группа ресурсов | Используйте группу ресурсов, которая уже есть в подписке, или введите имя, чтобы создать группу ресурсов. Группа ресурсов содержит связанные ресурсы для решения Azure. В этом примере мы используем **docs-aml**. Для использования существующей группы ресурсов требуется роль *участника* или *владельца* .  Дополнительные сведения о доступе см. [в статье Управление доступом к рабочей области машинное обучение Azure](how-to-assign-roles.md).
   Регион | Выберите ближайший к вашим пользователям регион Azure и ресурсы данных, чтобы создать рабочую область.
   | Учетная запись хранения | Учетная запись хранения по умолчанию для рабочей области. По умолчанию создается новый. |
   | Key Vault | Azure Key Vault, используемый рабочей областью. По умолчанию создается новый. |
   | Application Insights | Экземпляр Application Insights для рабочей области. По умолчанию создается новый. |
   | Реестр контейнеров | Реестр контейнеров Azure для рабочей области. По умолчанию для рабочей области изначально _не_ создается новый. Вместо этого он создается один раз при создании образа DOCKER во время обучения или развертывания. |

   :::image type="content" source="media/how-to-manage-workspace/create-workspace-form.png" alt-text="Настройте рабочую область.":::

1. Завершив настройку рабочей области, выберите **проверить и создать**. При необходимости используйте разделы " [Сетевые](#networking) и [Дополнительные](#advanced) параметры" для настройки дополнительных параметров рабочей области.

1. Проверьте параметры и внесите дополнительные изменения или исправления. Когда вы будете удовлетворены параметрами, нажмите кнопку **создать**.

   > [!Warning] 
   > Создание рабочей области в облаке может занять несколько минут.

   По завершении процесса появится сообщение об успешном развертывании. 
 
 1. Чтобы просмотреть новую рабочую область, выберите **Перейти к ресурсу**.
 
---



### <a name="networking"></a>Сеть  

> [!IMPORTANT]  
> Дополнительные сведения об использовании частной конечной точки и виртуальной сети с рабочей областью см. в разделе [Сетевая изоляция и конфиденциальность](how-to-network-security-overview.md).


# <a name="python"></a>[Python](#tab/python)

Пакет SDK для Машинное обучение Azure Python предоставляет класс [приватиндпоинтконфиг](/python/api/azureml-core/azureml.core.privateendpointconfig) , который можно использовать с [рабочей областью. Create ()](/python/api/azureml-core/azureml.core.workspace.workspace#create-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---tags-none--friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--adb-workspace-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--private-endpoint-config-none--private-endpoint-auto-approval-true--exist-ok-false--show-output-true-) для создания рабочей области с закрытой конечной точкой. Для этого класса требуется существующая виртуальная сеть.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Конфигурация сети по умолчанию — использовать __общедоступную конечную точку__, доступную в общедоступном Интернете. Чтобы ограничить доступ к рабочей области до созданной виртуальной сети Azure, можно выбрать в качестве __метода подключения__ __закрытую конечную точку__ (Предварительная версия), а затем использовать __+ добавить__ для настройки конечной точки.   

   :::image type="content" source="media/how-to-manage-workspace/select-private-endpoint.png" alt-text="Выбор закрытой конечной точки":::  

1. В форме __создание закрытой конечной точки__ задайте расположение, имя и виртуальную сеть для использования. Если вы хотите использовать конечную точку с зоной Частная зона DNS, выберите __интегрировать с частной зоной DNS__ и выберите зону, используя поле __зона частная зона DNS__ . Нажмите кнопку __ОК__ , чтобы создать конечную точку.   

   :::image type="content" source="media/how-to-manage-workspace/create-private-endpoint.png" alt-text="Создание закрытой конечной точки":::   

1. Завершив настройку сети, можно выбрать пункт проверить и __создать__ или перейти к дополнительной __расширенной__ конфигурации.

---

> [!IMPORTANT]  
> В настоящее время в общедоступной предварительной версии используется частная конечная точка с Машинное обучение Azure рабочей областью. Предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендуется для использования в рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.     
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="multiple-workspaces-with-private-endpoint"></a>Несколько рабочих областей с частной конечной точкой

При создании частной конечной точки создается новая зона Частная зона DNS с именем __privatelink.API.azureml.MS__ . Содержит ссылку на виртуальную сеть. При создании нескольких рабочих областей с частными конечными точками в одной группе ресурсов в зону DNS может быть добавлена только виртуальная сеть для первой частной конечной точки. Чтобы добавить записи для виртуальных сетей, используемых дополнительными рабочими областями или частными конечными точками, выполните следующие действия.

1. В [портал Azure](https://portal.azure.com)выберите группу ресурсов, содержащую рабочую область. Затем выберите ресурс Частная зона DNS зоны с именем __privatelink.API.azureml.MS__ .
2. В окне __Параметры__ выберите __ссылки на виртуальную сеть__.
3. Выберите __Добавить__. На странице __Добавление ссылки на виртуальную сеть__ укажите уникальное __имя ссылки__, а затем выберите добавляемую __виртуальную сеть__ . Нажмите кнопку __ОК__ , чтобы добавить сетевое соединение.

Дополнительные сведения см. в разделе [Конфигурация DNS для частной конечной точки Azure](../private-link/private-endpoint-dns.md).

### <a name="vulnerability-scanning"></a>Сканирование уязвимостей

Центр безопасности Azure обеспечивает унифицированное управление безопасностью и расширенную защиту от угроз для гибридных облачных рабочих нагрузок. Вы должны разрешить центру безопасности Azure проверять ресурсы и следовать рекомендациям. Дополнительные сведения см. в статье  [сканирование образа реестра контейнеров Azure с помощью центра безопасности](../security-center/defender-for-container-registries-introduction.md) и [интеграции Azure Kubernetes Services с центром безопасности](../security-center/defender-for-kubernetes-introduction.md).

### <a name="advanced"></a>Продвинутый уровень

По умолчанию метаданные для рабочей области хранятся в экземпляре Azure Cosmos DB, который обслуживает Корпорация Майкрософт. Эти данные шифруются с помощью ключей, управляемых корпорацией Майкрософт.

Чтобы ограничить данные, собираемые корпорацией Майкрософт в рабочей области, выберите __рабочую область "бизнес-воздействие__ " на портале или задайте `hbi_workspace=true ` в Python. Дополнительные сведения об этом параметре см. в разделе [Шифрование неактивных](concept-data-encryption.md#encryption-at-rest)данных.

> [!IMPORTANT]  
> Выбор высокого воздействия на бизнес может выполняться только при создании рабочей области. Этот параметр нельзя изменить после создания рабочей области.   

#### <a name="use-your-own-key"></a>Использование собственного ключа

Вы можете указать собственный ключ для шифрования данных. При этом создается экземпляр Azure Cosmos DB, в котором хранятся метаданные в подписке Azure.

[!INCLUDE [machine-learning-customer-managed-keys.md](../../includes/machine-learning-customer-managed-keys.md)]

Чтобы указать собственный ключ, выполните следующие действия.

> [!IMPORTANT]  
> Перед выполнением этих действий необходимо сначала выполнить следующие действия.   
>
> 1. Авторизовать __приложение машинное обучение__ (в службе управления удостоверениями и доступом) с разрешениями участника в вашей подписке.  
> 1. Выполните действия, описанные в разделе [Настройка ключей, управляемых клиентом](../cosmos-db/how-to-setup-cmk.md) :
>     * Регистрация поставщика Azure Cosmos DB
>     * Создание и настройка Azure Key Vault
>     * Создание ключа
>   
>     Не нужно вручную создавать экземпляр Azure Cosmos DB, он будет создан автоматически во время создания рабочей области. Этот экземпляр Azure Cosmos DB будет создан в отдельной группе ресурсов, используя имя на основе этого шаблона: `<your-workspace-resource-name>_<GUID>` .   
>   
> Этот параметр нельзя изменить после создания рабочей области. При удалении Azure Cosmos DB, используемого рабочей областью, необходимо также удалить рабочую область, которая ее использует.

# <a name="python"></a>[Python](#tab/python)

Используйте `cmk_keyvault` и `resource_cmk_uri` для указания ключа, управляемого клиентом.

```python
from azureml.core import Workspace
   ws = Workspace.create(name='myworkspace',
               subscription_id='<azure-subscription-id>',
               resource_group='myresourcegroup',
               create_resource_group=True,
               location='eastus2'
               cmk_keyvault='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.keyvault/vaults/<keyvault-name>', 
               resource_cmk_uri='<key-identifier>'
               )

```

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Выберите __ключи, управляемые клиентом__, а затем __щелкните, чтобы выбрать ключ__.

    :::image type="content" source="media/how-to-manage-workspace/advanced-workspace.png" alt-text="Ключи, управляемые клиентом":::

1. На __Azure Key Vault форме Выбор ключа__ выберите существующую Azure Key Vault, содержащийся в ней ключ и версию ключа. Этот ключ используется для шифрования данных, хранящихся в Azure Cosmos DB. Наконец, используйте кнопку __выбрать__ , чтобы использовать этот ключ.

   :::image type="content" source="media/how-to-manage-workspace/select-key-vault.png" alt-text="Выберите ключ":::

---

### <a name="download-a-configuration-file"></a>Скачивание файла конфигурации.

Если вы будете создавать [вычислительный экземпляр](tutorial-1st-experiment-sdk-setup.md#azure), пропустите этот шаг.  Экземпляр COMPUTE уже создал копию этого файла.

# <a name="python"></a>[Python](#tab/python)

Если вы планируете использовать код в локальной среде, которая ссылается на эту рабочую область ( `ws` ), напишите файл конфигурации:

```python
ws.write_config()
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

Если вы планируете использовать код в локальной среде, которая ссылается на эту рабочую область, выберите **Скачать config.json** в разделе **Обзор** рабочей области.  

   ![Скачивание config.json](./media/how-to-manage-workspace/configure.png)

---

Поместите файл в структуру каталогов со скриптами Python или приложениями Jupyter Notebook. Он может находиться в том же каталоге, подкаталоге с именем *.azureml* или родительском каталоге. При создании вычислительного экземпляра этот файл добавляется в правильный каталог на виртуальной машине.

## <a name="connect-to-a-workspace"></a>Подключение к рабочей области

В коде Python создайте объект рабочей области для подключения к рабочей области.  Этот код будет считывать содержимое файла конфигурации для поиска рабочей области.  Вы получите запрос на вход, если вы еще не прошли проверку подлинности.

```python
from azureml.core import Workspace

ws = Workspace.from_config()
```

* <a name="connect-multi-tenant"></a>**Несколько клиентов.**  Если у вас несколько учетных записей, добавьте идентификатор клиента Azure Active Directory, который вы хотите использовать.  Найдите идентификатор клиента из [портал Azure](https://portal.azure.com) в разделе **Azure Active Directory внешние удостоверения**.

    ```python
    from azureml.core.authentication import InteractiveLoginAuthentication
    from azureml.core import Workspace
    
    interactive_auth = InteractiveLoginAuthentication(tenant_id="my-tenant-id")
    ws = Workspace.from_config(auth=interactive_auth)
    ```

* **[Независимых Cloud](reference-machine-learning-cloud-parity.md)**. Вам потребуется дополнительный код для проверки подлинности в Azure, если вы работаете в независимых облаке.

    ```python
    from azureml.core.authentication import InteractiveLoginAuthentication
    from azureml.core import Workspace
    
    interactive_auth = InteractiveLoginAuthentication(cloud="<cloud name>") # for example, cloud="AzureUSGovernment"
    ws = Workspace.from_config(auth=interactive_auth)
    ```
    
Если у вас возникли проблемы при доступе к подписке, см. статью [Настройка проверки подлинности для машинное обучение Azure ресурсов и рабочих процессов](how-to-setup-authentication.md), а также [Проверка подлинности в машинное обучение Azure](https://aka.ms/aml-notebook-auth) записной книжке.

## <a name="find-a-workspace"></a><a name="view"></a>Найти рабочую область

Просмотрите список всех рабочих областей, которые можно использовать.

# <a name="python"></a>[Python](#tab/python)

Найдите свои подписки на [странице "подписки" в портал Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).  Скопируйте идентификатор и используйте его в приведенном ниже коде, чтобы просмотреть все рабочие области, доступные для этой подписки.

```python
from azureml.core import Workspace

Workspace.list('<subscription-id>')
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Войдите на [портал Azure](https://portal.azure.com/).

1. В верхнем поле поиска введите **машинное обучение**.  

1. Выберите **Машинное обучение**.

   ![Поиск Машинное обучение Azure рабочей области](./media/how-to-manage-workspace/find-workspaces.png)

1. Просмотрите список найденных рабочих областей. Можно выполнять фильтрацию на основе подписки, групп ресурсов и расположений.  

1. Выберите рабочую область, чтобы отобразить ее свойства.

---


## <a name="delete-a-workspace"></a>Удаление рабочей области

Если Рабочая область больше не нужна, удалите ее.  

# <a name="python"></a>[Python](#tab/python)

Удалите рабочую область `ws` :

```python
ws.delete(delete_dependent_resources=False, no_wait=False)
```

Действие по умолчанию — не удалять ресурсы, связанные с рабочей областью, т. е. Реестр контейнеров, учетную запись хранения, хранилище ключей и Application Insights.  Задайте значение `delete_dependent_resources` true, чтобы удалить эти ресурсы.

# <a name="portal"></a>[Портал](#tab/azure-portal)

В [портал Azure](https://portal.azure.com/)выберите **Удалить**  в верхней части рабочей области, которую вы хотите удалить.

:::image type="content" source="./media/how-to-manage-workspace/delete-workspace.png" alt-text="Удалить рабочую область":::

---

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

## <a name="troubleshooting"></a>Диагностика

* **Поддерживаемые браузеры в машинное обучение Azure Studio**. рекомендуется использовать наиболее актуальный браузер, совместимый с вашей операционной системой. Поддерживаются следующие браузеры:
  * Microsoft Edge (новый Microsoft Edge последней версии, а не устаревшая версия Microsoft Edge);
  * Safari (последняя версия, только для Mac);
  * Chrome (последняя версия);
  * Firefox (последняя версия).

* **Портал Azure**: 
  * Если перейти непосредственно к рабочей области из ссылки на общий ресурс из пакета SDK или портал Azure, вы не сможете просмотреть стандартную страницу **обзора** со сведениями о подписке в расширении. В этом случае вы также не сможете переключиться на другую рабочую область. Чтобы просмотреть другую рабочую область, перейдите непосредственно в [машинное обучение Azure Studio](https://ml.azure.com) и найдите имя рабочей области.
  * Все активы (наборы данных, эксперименты, расчеты и т. д.) доступны только в [машинное обучение Azure Studio](https://ml.azure.com). Они недоступны *из* портал Azure.

### <a name="resource-provider-errors"></a>Ошибки поставщика ресурсов

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="moving-the-workspace"></a>Перемещение рабочей области

> [!WARNING]
> Перемещение рабочей области Машинного обучения Azure в другую подписку или перемещение главной подписки на новый клиент не поддерживается. Это может привести к ошибкам.

### <a name="deleting-the-azure-container-registry"></a>Удаление Реестра контейнеров Azure

Для некоторых операций в рабочей области Машинного обучения Azure используется реестр контейнеров Azure (ACR) Он автоматически создает экземпляр ACR при необходимости.

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

## <a name="examples"></a>Примеры

Примеры создания рабочей области.
* Использование портал Azure для [создания рабочей области и вычислительного экземпляра](tutorial-1st-experiment-sdk-setup.md)
* Использование пакета SDK для Python для [создания рабочей области в собственной среде](tutorial-1st-experiment-sdk-setup-local.md)

## <a name="next-steps"></a>Дальнейшие действия

После создания рабочей области Узнайте, как [обучать и развертывать модель](tutorial-train-models-with-aml.md).