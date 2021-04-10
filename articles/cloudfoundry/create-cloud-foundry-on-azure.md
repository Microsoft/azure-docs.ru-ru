---
title: Создание кластера Pivotal Cloud Foundry в Azure
description: Узнайте, как настроить параметры, необходимые для подготовки кластера Pivotal Cloud Foundry (PCF) к работе в Azure
services: Cloud Foundry
documentationcenter: CloudFoundry
author: ruyakubu
manager: brunoborges
editor: ruyakubu
ms.assetid: ''
ms.author: ruyakubu
ms.date: 09/13/2018
ms.devlang: ''
ms.service: azure
ms.tgt_pltfrm: multiple
ms.topic: tutorial
ms.workload: web
ms.openlocfilehash: 3444d6c62b5e36b7e68cfaf6da1ec534e2ea4ec6
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102551459"
---
# <a name="create-a-pivotal-cloud-foundry-cluster-on-azure"></a>Создание кластера Pivotal Cloud Foundry в Azure

Это руководство предоставляет быстрые шаги для создания параметров, необходимых для подготовки кластера Pivotal Cloud Foundry (PCF) к работе в Azure. Чтобы найти решение Pivotal Cloud Foundry, выполните поиск в Azure [Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry).

![Поиск Pivotal Cloud Foundry в Azure](media/deploy/pcf-marketplace.png)


## <a name="generate-an-ssh-public-key"></a>Создание открытого ключа SSH

Существует несколько способов создать открытый ключ Secure Shell (SSH) с помощью Windows, Mac или Linux.

```Bash
ssh-keygen -t rsa -b 2048
```

Дополнительные сведения см. в статье [Как использовать ключи SSH с Windows в Azure](../virtual-machines/linux/ssh-from-windows.md).

## <a name="create-a-service-principal"></a>Создание субъекта-службы

> [!NOTE]
>
> Для создания субъекта-службы требуется разрешение владельца учетной записи. Можно также написать сценарий для автоматизации создания субъекта-службы. Например, можно использовать команду [az ad sp create-for-rbac](/cli/azure/ad/sp) Azure CLI.

1. Войдите в учетную запись Azure.

    ```azurecli
    az login
    ```

    ![Вход в Azure CLI](media/deploy/az-login-output.png )
 
    Скопируйте значение id как **идентификатор подписки** и скопируйте значение tenantId для дальнейшего использования.

2. Определите подписку по умолчанию для этой конфигурации.

    ```azurecli
    az account set -s {id}
    ```

3. Создайте приложение Azure Active Directory для вашего PCF. Укажите уникальный буквенно-цифровой пароль. Сохраните пароль как **clientSecret** для дальнейшего использования.

    ```azurecli
    az ad app create --display-name "Svc Principal for OpsManager" --password {enter-your-password} --homepage "{enter-your-homepage}" --identifier-uris {enter-your-homepage}
    ```

    Скопируйте значение appId в выходных данных как **clientID** для дальнейшего использования.

    > [!NOTE]
    >
    > Выберите домашнюю страницу приложения и URI идентификатора, например http\://www\.contoso.com.

4. Создайте субъект-службу с новым идентификатором приложения.

    ```azurecli
    az ad sp create --id {appId}
    ```

5. Задайте для субъекта-службы роль разрешения "Участник".

    ```azurecli
    az role assignment create --assignee "{enter-your-homepage}" --role "Contributor"
    ```

    Можно также использовать:

    ```azurecli
    az role assignment create --assignee {service-principal-name} --role "Contributor"
    ```

    ![Назначение роли субъекту-службе](media/deploy/svc-princ.png )

6. Убедитесь, что вы можете успешно войти в субъект-службу с помощью идентификатора приложения, пароля и идентификатора клиента.

    ```azurecli
    az login --service-principal -u {appId} -p {your-password}  --tenant {tenantId}
    ```

7. Создайте JSON-файл в следующем формате. Используйте значения **subscriptionID**, **tenantID**, **clientID** и **clientSecret**, скопированные ранее. Сохраните файл.

    ```json
    {
        "subscriptionID": "{enter-your-subscription-Id-here}",
        "tenantID": "{enter-your-tenant-Id-here}",
        "clientID": "{enter-your-app-Id-here}",
        "clientSecret": "{enter-your-key-here}"
    }
    ```

## <a name="get-the-pivotal-network-token"></a>Получение токена Pivotal Network

1. Зарегистрируйтесь или войдите в учетную запись [Pivotal Network](https://network.pivotal.io).
2. Выберите имя профиля в правом верхнем углу страницы. Выберите команду **Edit Profile** (Изменить профиль).
3. Прокрутите до нижней части страницы и скопируйте значение **Legacy API token** (Устаревший токен API). Это ваше значение **токена Pivotal Network**, которое будет использоваться позже.

## <a name="provision-your-cloud-foundry-cluster-on-azure"></a>Подготовка кластера Cloud Foundry к работе в Azure

Теперь у вас есть все параметры, необходимые для подготовки [кластера Pivotal Cloud Foundry к работе в Azure](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry).
Введите параметры и создайте кластер PCF.

## <a name="verify-the-deployment-and-sign-in-to-the-pivotal-ops-manager"></a>Проверка развертывания и вход в Pivotal Ops Manager

1. Кластер PCF отображает состояние развертывания.

    ![Состояние развертывания в Azure](media/deploy/deployment.png )

2. Выберите ссылку **Развертывания** в области навигации слева, чтобы получить учетные данные для вашего диспетчера PCF Ops Manager. Выберите **Имя развертывания** на следующей странице.
3. На панели навигации слева выберите ссылку **Выходные данные**, чтобы отобразить URL-адрес, имя пользователя и пароль для PCF Ops Manager. Значение OPSMAN-FQDN является URL-адресом.
 
    ![Выходные данные развертывания Cloud Foundry](media/deploy/deploy-outputs.png )
 
4. В веб-браузере перейдите по URL-адресу. Введите учетные данные с предыдущего шага, чтобы выполнить вход.

    ![Страница входа Pivotal](media/deploy/pivotal-login.png )
         
    > [!NOTE]
    >
    > Если произошел сбой браузера Internet Explorer из-за предупреждающего сообщения о небезопасности веб-сайта, выберите **Подробнее** и перейдите на веб-страницу. Для Firefox выберите **Дополнительно** и добавьте сертификат, чтобы продолжить.

5. Ваш PCF Ops Manager отображает развернутые экземпляры Azure. Теперь здесь можно создавать и администрировать приложения.
               
    ![Развернутый экземпляр Azure в Pivotal](media/deploy/ops-mgr.png )
