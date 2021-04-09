---
title: Краткое руководство. Создание группы управления с помощью Python
description: В этом кратком руководстве показано, как с помощью Python создать группу управления для организации ресурсов в иерархии ресурсов.
ms.date: 01/29/2021
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: e3c55cc14a8ac980318fd0de9485a3e0ca31b582
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100101663"
---
# <a name="quickstart-create-a-management-group-with-python"></a>Краткое руководство. Создание группы управления с помощью Python

Группы управления — это контейнеры, которые помогают управлять доступом, политикой и соответствием требованиям в нескольких подписках. Создание таких контейнеров позволяет построить эффективную и экономную иерархию, которую можно использовать с [политикой Azure](../policy/overview.md) и [элементами управления доступом на основе ролей Azure](../../role-based-access-control/overview.md). Дополнительные сведения о группах управления см. в статье [Упорядочение ресурсов с помощью групп управления Azure](overview.md).

Создание первой группы управления в каталоге может занять до 15 минут. Существуют процессы, выполняемые в первый раз при настройке службы групп управления в Azure для вашего каталога. По завершении процесса вы получите уведомление. См. сведения о [начальной настройке групп управления](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Любой пользователь Azure AD в арендаторе может создать группу управления без разрешения на запись для группы управления, назначенного этому пользователю, если [защита иерархии](./how-to/protect-resource-hierarchy.md#setting---require-authorization) не включена. Эта новая группа управления становится дочерней по отношению к корневой группе управления или [группе управления по умолчанию](./how-to/protect-resource-hierarchy.md#setting---default-management-group), а ее создателю назначается роль "Владелец". Служба группы управления обеспечивает эту возможность, чтобы назначения ролей не требовались на корневом уровне. У пользователей нет доступа к корневой группе управления при ее создании. Чтобы избежать трудностей при поиске глобальных администраторов Azure AD для настройки работы с группами управления, мы разрешаем создавать исходные группы управления на корневом уровне.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-library"></a>Добавление библиотеки Resource Graph

Чтобы можно было управлять группами управления с помощью Python, необходимо добавить библиотеку. Эта библиотека работает во всех средах, где можно использовать Python, в том числе в [оболочке Bash для Windows 10](/windows/wsl/install-win10). Также ее можно установить локально.

1. Убедитесь, что установлена последняя версия Python (не ниже **3.8**). Если он еще не установлен, скачайте его на сайте [Python.org](https://www.python.org/downloads/).

1. Убедитесь, что установлена последняя версия Azure CLI (не ниже **2.5.1**). Если Azure CLI еще не установлен, см. [эту статью](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Azure CLI требуется для того, чтобы с помощью Python можно было использовать **проверки подлинности на основе CLI** в следующих примерах. Дополнительные сведения о других параметрах см. в статье о [проверке подлинности с помощью библиотек управления Azure для Python](/azure/developer/python/azure-sdk-authenticate).

1. Проверка подлинности с помощью Azure CLI.

   ```azurecli
   az login
   ```

1. В выбранной вами среде Python установите необходимые библиотеки для групп управления:

   ```bash
   # Add the management groups library for Python
   pip install azure-mgmt-managementgroups

   # Add the Resources library for Python
   pip install azure-mgmt-resource

   # Add the CLI Core library for Python for authentication (development only!)
   pip install azure-cli-core
   ```

   > [!NOTE]
   > Если Python установлен для всех пользователей, эти команды необходимо запустить из консоли с повышенными привилегиями.

1. Убедитесь, что библиотеки установлены. `azure-mgmt-managementgroups` должна быть версии **0.2.0** или более поздней, `azure-mgmt-resource` ⁠— **9.0.0** или более поздней, а `azure-cli-core`  — **2.5.0** или более поздней.

   ```bash
   # Check each installed library
   pip show azure-mgmt-managementgroups azure-mgmt-resource azure-cli-core
   ```

## <a name="create-the-management-group"></a>Создание группы управления

1. Создайте скрипт Python и сохраните следующий исходный код как `mgCreate.py`:

   ```python
   # Import management group classes
   from azure.mgmt.managementgroups import ManagementGroupsAPI
   
   # Import specific methods and models from other libraries
   from azure.common.credentials import get_azure_cli_credentials
   from azure.common.client_factory import get_client_from_cli_profile
   from azure.mgmt.resource import ResourceManagementClient, SubscriptionClient
   
   # Wrap all the work in a function
   def createmanagementgroup( strName ):
       # Get your credentials from Azure CLI (development only!) and get your subscription list
       subsClient = get_client_from_cli_profile(SubscriptionClient)
       subsRaw = []
       for sub in subsClient.subscriptions.list():
           subsRaw.append(sub.as_dict())
       subsList = []
       for sub in subsRaw:
           subsList.append(sub.get('subscription_id'))
       
       # Create management group client and set options
       mgClient = get_client_from_cli_profile(ManagementGroupsAPI)
       mg_request = {'name': strName, 'display_name': strName}
       
       # Create management group
       mg = mgClient.management_groups.create_or_update(group_id=strName,create_management_group_request=mg_request)
       
       # Show results
       print(mg)
   
   createmanagementgroup("MyNewMG")
   ```

1. Выполните аутентификацию в Azure CLI с помощью команды `az login`.

1. Введите следующую команду в окне терминала.

   ```bash
   py mgCreate.py
   ```

Результат создания группы управления выводится в консоль как объект `LROPoller`.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить установленные библиотеки из среды Python, это можно сделать с помощью следующей команды:

```bash
# Remove the installed libraries from the Python environment
pip uninstall azure-mgmt-managementgroups azure-mgmt-resource azure-cli-core
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать группу управления для организации иерархии ресурсов. Группа управления может содержать подписки и другие группы управления.

Чтобы узнать больше о группах управления и управлении иерархией ресурсов, см. следующее руководство:

> [!div class="nextstepaction"]
> [Управление ресурсами с помощью групп управления](./manage.md)