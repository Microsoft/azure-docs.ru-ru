---
title: Проверка подлинности Azure Spring Cloud с использованием Key Vault в GitHub Actions
description: Как использовать хранилище ключей с рабочим процессом CI/CD для Azure Веснного облака с действиями GitHub
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: how-to
ms.date: 09/08/2020
ms.custom: devx-track-java
ms.openlocfilehash: 0ea0db1faf8c452958b8d95c193d45506057777c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98673338"
---
# <a name="authenticate-azure-spring-cloud-with-key-vault-in-github-actions"></a>Проверка подлинности Azure Spring Cloud с использованием Key Vault в GitHub Actions

**Эта статья применима к:** ✔️ Java ✔️ C#

Хранилище ключей — это безопасное место для хранения ключей. Корпоративным пользователям необходимо хранить учетные данные для сред CI/CD в области, в которой они контролируются. Ключ для получения учетных данных в хранилище ключей должен быть ограничен областью действия ресурса.  Он имеет доступ только к области хранилища ключей, а не ко всей области Azure. Он похож на ключ, который может открыть только надежное поле, а не главный ключ, который может открыть все двери в здании. Это способ получить ключ с помощью другого ключа, который полезен в рабочем процессе CICD. 

## <a name="generate-credential"></a>Создание учетных данных
Чтобы создать ключ для доступа к хранилищу ключей, выполните следующую команду на локальном компьютере:

```azurecli
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.KeyVault/vaults/<KEY_VAULT> --sdk-auth
```
Область, заданная `--scopes` параметром, ограничивает доступ ключа к ресурсу.  Он может получить доступ только к надежному полю.

С результатами:
```output
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
}
```
Затем сохраните результаты в **секреты** GitHub, как описано в разделе [Настройка репозитория GitHub и аутентификация в Azure](./spring-cloud-howto-github-actions.md#set-up-github-repository-and-authenticate).

## <a name="add-access-policies-for-the-credential"></a>Добавление политик доступа для учетных данных
Созданные выше учетные данные могут получать только общие сведения о Key Vault, а не содержимое, которое она хранит.  Чтобы получить секреты, хранящиеся в Key Vault, необходимо задать политики доступа для учетных данных.

Перейдите к панели мониторинга **Key Vault** в портал Azure, щелкните меню **управления доступом** , а затем откройте вкладку **назначения ролей** . Выберите **приложения** для **типа** и `This resource` **область**.  Вы должны увидеть учетные данные, созданные на предыдущем шаге:

 ![Настройка политики доступа](./media/github-actions/key-vault1.png)

Скопируйте имя учетных данных, например `azure-cli-2020-01-19-04-39-02` . Откройте меню **политики доступа** , щелкните ссылку **+ Добавить политику доступа** .  Выберите `Secret Management` для **шаблона**, а затем выберите **субъект**. Вставьте имя учетных данных в поле **основной** / **Выбор** входа.

 ![Выберите пункт](./media/github-actions/key-vault2.png)

 Нажмите кнопку **Добавить** в диалоговом окне **Добавление политики доступа** , а затем нажмите кнопку **сохранить**.

## <a name="generate-full-scope-azure-credential"></a>Создание учетных данных Azure с полной областью действия
Это главный ключ для открытия всех дверей в здании. Процедура аналогична предыдущему шагу, но здесь мы изменим область для создания главного ключа:

```azurecli
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID> --sdk-auth
```

Опять же, результаты:
```output
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
}
```
Скопируйте целую строку JSON.  Вернуться к панели мониторинга **Key Vault** . Откройте меню **секреты** и нажмите кнопку **создать/импортировать** . Введите имя секрета, например `AZURE-CREDENTIALS-FOR-SPRING` . Вставьте строку учетных данных JSON в поле ввода **значения** . Можно заметить, что поле ввода значения является однострочным текстовым полем, а не многострочной текстовой областью.  Здесь можно вставить полную строку JSON.

 ![Учетные данные полной области действия](./media/github-actions/key-vault3.png)

## <a name="combine-credentials-in-github-actions"></a>Объединение учетных данных в действиях GitHub
Задайте учетные данные, используемые при выполнении конвейера CICD:

```console
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}           # Strong box key you generated in the first step
    - uses: Azure/get-keyvault-secrets@v1.0
      with:
        keyvault: "<Your Key Vault Name>"
        secrets: "AZURE-CREDENTIALS-FOR-SPRING"           # Master key to open all doors in the building
      id: keyvaultaction
    - uses: azure/login@v1
      with:
        creds: ${{ steps.keyvaultaction.outputs.AZURE-CREDENTIALS-FOR-SPRING }}
    - name: Azure CLI script
      uses: azure/CLI@v1
      with:
        azcliversion: 2.0.75
        inlineScript: |
          az extension add --name spring-cloud             # Spring CLI commands from here
          az spring-cloud list

```

## <a name="next-steps"></a>Дальнейшие действия
* [Действия весны GitHub Cloud](./spring-cloud-howto-github-actions.md)
