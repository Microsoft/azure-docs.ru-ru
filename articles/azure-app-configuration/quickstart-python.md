---
title: Краткое руководство по использованию конфигурации приложений Azure в приложениях Python | Документация Майкрософт
description: В этом кратком руководстве описано, как создать приложение Python с помощью службы "Конфигурация приложений Azure", чтобы обеспечить централизованное хранение параметров приложения и управление ими отдельно от кода.
services: azure-app-configuration
author: drewbatgit
ms.service: azure-app-configuration
ms.topic: quickstart
ms.custom: devx-track-python
ms.date: 9/17/2020
ms.author: drewbat
ms.openlocfilehash: 1be74ceeb1f4808a8df8f56b71b66aa870cdd91d
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104800879"
---
# <a name="quickstart-create-a-python-app-with-azure-app-configuration"></a>Краткое руководство. Создание приложения Python с помощью конфигурации приложения Azure

В этом кратком руководстве будет использоваться конфигурация приложений Azure для централизации хранения параметров приложения и управления ими с помощью [клиентской библиотеки конфигурации приложений Azure для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/appconfiguration/azure-appconfiguration).

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- Python 2.7, 3.6 или более поздней версии: сведения о настройке Python в Windows см. в документации по [Python для Windows]( https://docs.microsoft.com/windows/python/).

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Выберите **Обозреватель конфигураций** > **Создать** > **Ключ-значение**, чтобы добавить следующие пары "ключ-значение":

    | Клавиши | Значение |
    |---|---|
    | TestApp:Settings:FontSize | Данные из конфигурации приложения Azure |

    Поля **Метка** и **Тип контента** пока заполнять не нужно.

8. Нажмите кнопку **Применить**.

## <a name="setting-up-the-python-app"></a>Настройка приложения Python

1. В этом руководстве вы создадите новый каталог для проекта с именем *app-configuration-quickstart*.

    ```console
    mkdir app-configuration-quickstart
    ```

1. Перейдите в созданный каталог *app-configuration-quickstart*.

    ```console
    cd app-configuration-quickstart
    ```

1. Установите клиентскую библиотеку конфигурации приложений Azure с помощью команды `pip install`.

    ```console
    pip install azure-appconfiguration
    ```

1. Создайте новый файл с именем *app-configuration-quickstart.py* в каталоге *app-configuration-quickstart* и добавьте следующий код:

    ```python
    import os
    from azure.appconfiguration import AzureAppConfigurationClient, ConfigurationSetting
    
    try:
        print("Azure App Configuration - Python Quickstart")
        # Quickstart code goes here
    except Exception as ex:
        print('Exception:')
        print(ex)
    ```

> [!NOTE]
> Фрагменты кода, приведенные в этом кратком руководстве, помогут приступить к работе с клиентской библиотекой конфигурации приложений для Python. Для приложения также рекомендуется предусмотреть обработку исключений в соответствии с вашими потребностями. Дополнительные сведения об обработке исключений см. в [документации по пакету SDK Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/appconfiguration/azure-appconfiguration).

## <a name="configure-your-app-configuration-connection-string"></a>Настройка строки подключения в конфигурации приложения

1. Задайте переменную среды с именем **AZURE_APP_CONFIG_CONNECTION_STRING** и присвойте ей значение ключа доступа к хранилищу службы "Конфигурация приложений". В командной строке выполните следующую команду:

    ```cmd
    setx AZURE_APP_CONFIG_CONNECTION_STRING "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете Windows PowerShell, выполните следующую команду:

    ```azurepowershell
    $Env:AZURE_APP_CONFIG_CONNECTION_STRING = "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете macOS или Linux, выполните следующую команду:

    ```console
    export AZURE_APP_CONFIG_CONNECTION_STRING='connection-string-of-your-app-configuration-store'
    ```

2. Перезапустите командную строку, чтобы изменение вступило в силу. Выведите значение переменной среды, чтобы проверить правильность ее настройки.

## <a name="code-samples"></a>Примеры кода

В примерах фрагментов кода в этом разделе показано, как выполнять общие операции с клиентской библиотекой конфигурации приложений для Python. Добавьте эти фрагменты кода в блок `try` в файле *app-configuration-quickstart.py*, созданном ранее.

> [!NOTE]
> Клиентская библиотека конфигурации приложений ссылается на объект типа "ключ-значение" как на `ConfigurationSetting`. Поэтому в этой статье пары **ключ-значение** в хранилище конфигураций приложений будут называться **параметрами конфигурации**.

* [Подключение к хранилищу Конфигурации приложений](#connect-to-an-app-configuration-store)
* [Получить параметр конфигурации](#get-a-configuration-setting)
* [Добавить параметр конфигурации](#add-a-configuration-setting)
* [Получение списка параметров конфигурации](#get-a-list-of-configuration-settings)
* [Заблокировать параметр конфигурации](#lock-a-configuration-setting)
* [Разблокировать параметр конфигурации](#unlock-a-configuration-setting)
* [Обновить параметр конфигурации](#update-a-configuration-setting)
* [Удалить параметр конфигурации](#delete-a-configuration-setting)

### <a name="connect-to-an-app-configuration-store"></a>Подключение к хранилищу Конфигурации приложений

В следующем фрагменте кода с использованием строки подключения, хранящейся в переменных среды, создается экземпляр **AzureAppConfigurationClient**.

```python
    connection_string = os.getenv('AZURE_APP_CONFIG_CONNECTION_STRING')
    app_config_client = AzureAppConfigurationClient.from_connection_string(connection_string)
```

### <a name="get-a-configuration-setting"></a>Получить параметр конфигурации

Следующий фрагмент кода извлекает параметр конфигурации по имени `key`.

```python
    retrieved_config_setting = app_config_client.get_configuration_setting(key='TestApp:Settings:Message')
    print("\nRetrieved configuration setting:")
    print("Key: " + retrieved_config_setting.key + ", Value: " + retrieved_config_setting.value)
```

### <a name="add-a-configuration-setting"></a>Добавить параметр конфигурации

В следующем фрагменте кода создается объект `ConfigurationSetting` с полями `key` и `value` и вызывается метод `add_configuration_setting`. Этот метод вызовет исключение при попытке добавить параметр конфигурации, который уже существует в хранилище. Если вы хотите избежать этого исключения, вместо этого можно использовать метод [set_configuration_setting](#update-a-configuration-setting).

```python
    config_setting = ConfigurationSetting(
        key='TestApp:Settings:NewSetting',
        value='New setting value'
    )
    added_config_setting = app_config_client.add_configuration_setting(config_setting)
    print("\nAdded configuration setting:")
    print("Key: " + added_config_setting.key + ", Value: " + added_config_setting.value)
```

### <a name="get-a-list-of-configuration-settings"></a>Получение списка параметров конфигурации

Следующий фрагмент кода извлекает список параметров конфигурации. Аргументы `key_filter` и `label_filter` могут быть предоставлены для фильтрации пар "ключ-значение" на основе `key` и `label`, соответственно. Дополнительные сведения о фильтрации см. в разделе [Параметры конфигурации запросов](./concept-key-value.md#query-key-values).

```python
    filtered_settings_list = app_config_client.list_configuration_settings(key_filter="TestApp*")
    print("\nRetrieved list of configuration settings:")
    for item in filtered_settings_list:
        print("Key: " + item.key + ", Value: " + item.value)
```

### <a name="lock-a-configuration-setting"></a>Заблокировать параметр конфигурации

Состояние блокировки пары "ключ-значение" в конфигурации приложения обозначается атрибутом `read_only` объекта `ConfigurationSetting`. Если `read_only` равен `True`, параметр блокируется. Метод `set_read_only` можно вызвать с аргументом `read_only=True`, чтобы заблокировать параметр конфигурации.

```python
    locked_config_setting = app_config_client.set_read_only(added_config_setting, read_only=True)
    print("\nRead-only status for " + locked_config_setting.key + ": " + str(locked_config_setting.read_only))
```

### <a name="unlock-a-configuration-setting"></a>Разблокировать параметр конфигурации

Если атрибут `read_only` `ConfigurationSetting` равен `False`, этот параметр разблокируется. Метод `set_read_only` можно вызвать с аргументом `read_only=False`, чтобы разблокировать параметр конфигурации.

```python
    unlocked_config_setting = app_config_client.set_read_only(locked_config_setting, read_only=False)
    print("\nRead-only status for " + unlocked_config_setting.key + ": " + str(unlocked_config_setting.read_only))
```

### <a name="update-a-configuration-setting"></a>Обновить параметр конфигурации

Метод `set_configuration_setting` можно использовать для обновления существующего параметра или создания нового параметра. Следующий фрагмент кода изменяет значение существующего параметра конфигурации.

```python
    added_config_setting.value = "Value has been updated!"
    updated_config_setting = app_config_client.set_configuration_setting(added_config_setting)
    print("\nUpdated configuration setting:")
    print("Key: " + updated_config_setting.key + ", Value: " + updated_config_setting.value)
```

### <a name="delete-a-configuration-setting"></a>Удалить параметр конфигурации

Следующий фрагмент кода удаляет параметр конфигурации по имени `key`.

```python
    deleted_config_setting = app_config_client.delete_configuration_setting(key="TestApp:Settings:NewSetting")
    print("\nDeleted configuration setting:")
    print("Key: " + deleted_config_setting.key + ", Value: " + deleted_config_setting.value)
```

## <a name="run-the-app"></a>Запуск приложения

В этом кратком руководстве вы создали приложение Python, которое использует клиентскую библиотеку конфигурации приложений Azure для получения параметра конфигурации, созданного с помощью портала Azure, добавили новый параметр, получили список существующих параметров, заблокировали и разблокировали параметр, обновили параметр и, наконец, удалили параметр.

На этом этапе файл *app-configuration-quickstart.py* должен содержать следующий код:

```python
import os
from azure.appconfiguration import AzureAppConfigurationClient, ConfigurationSetting

try:
    print("Azure App Configuration - Python Quickstart")
    # Quickstart code goes here

    connection_string = os.getenv('AZURE_APP_CONFIG_CONNECTION_STRING')
    app_config_client = AzureAppConfigurationClient.from_connection_string(connection_string)

    retrieved_config_setting = app_config_client.get_configuration_setting(key='TestApp:Settings:Message')
    print("\nRetrieved configuration setting:")
    print("Key: " + retrieved_config_setting.key + ", Value: " + retrieved_config_setting.value)

    config_setting = ConfigurationSetting(
        key='TestApp:Settings:NewSetting',
        value='New setting value'
    )
    added_config_setting = app_config_client.add_configuration_setting(config_setting)
    print("\nAdded configuration setting:")
    print("Key: " + added_config_setting.key + ", Value: " + added_config_setting.value)

    filtered_settings_list = app_config_client.list_configuration_settings(key_filter="TestApp*")
    print("\nRetrieved list of configuration settings:")
    for item in filtered_settings_list:
        print("Key: " + item.key + ", Value: " + item.value)

    locked_config_setting = app_config_client.set_read_only(added_config_setting, read_only=True)
    print("\nRead-only status for " + locked_config_setting.key + ": " + str(locked_config_setting.read_only))

    unlocked_config_setting = app_config_client.set_read_only(locked_config_setting, read_only=False)
    print("\nRead-only status for " + unlocked_config_setting.key + ": " + str(unlocked_config_setting.read_only))

    added_config_setting.value = "Value has been updated!"
    updated_config_setting = app_config_client.set_configuration_setting(added_config_setting)
    print("\nUpdated configuration setting:")
    print("Key: " + updated_config_setting.key + ", Value: " + updated_config_setting.value)

    deleted_config_setting = app_config_client.delete_configuration_setting(key="TestApp:Settings:NewSetting")
    print("\nDeleted configuration setting:")
    print("Key: " + deleted_config_setting.key + ", Value: " + deleted_config_setting.value)

except Exception as ex:
    print('Exception:')
    print(ex)
```

В окне консоли перейдите в каталог, содержащий файл *app-configuration-quickstart.py*, и выполните следующую команду Python, чтобы запустить приложение:

```console
python app-configuration-quickstart.py
```

Вы должны увидеть следующий результат:

```output
Azure App Configuration - Python Quickstart

Retrieved configuration setting:
Key: TestApp:Settings:Message, Value: Data from Azure App Configuration

Added configuration setting:
Key: TestApp:Settings:NewSetting, Value: New setting value

Retrieved list of configuration settings:
Key: TestApp:Settings:Message, Value: Data from Azure App Configuration
Key: TestApp:Settings:NewSetting, Value: New setting value

Read-only status for TestApp:Settings:NewSetting: True

Read-only status for TestApp:Settings:NewSetting: False

Updated configuration setting:
Key: TestApp:Settings:NewSetting, Value: Value has been updated!

Deleted configuration setting:
Key: TestApp:Settings:NewSetting, Value: Value has been updated!
```

## <a name="clean-up-resources"></a>Очистка ресурсов


[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали новое хранилище конфигурации приложений и узнали, как получить доступ к парам "ключ-значение" из приложения Python.

Дополнительные примеры кода см. здесь:

> [!div class="nextstepaction"]
> [Примеры клиентской библиотеки конфигурации приложений Azure](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/appconfiguration/azure-appconfiguration/samples)
