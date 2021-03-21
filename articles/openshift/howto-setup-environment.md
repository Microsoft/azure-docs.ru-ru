---
title: Настройка среды разработки Azure Red Hat OpenShift
description: Ниже приведены необходимые условия для работы с Microsoft Azure Red Hat OpenShift.
keywords: Настройка установки Red Hat openshift
author: jimzim
ms.author: jzim
ms.date: 11/04/2019
ms.topic: conceptual
ms.service: azure-redhat-openshift
ms.custom: devx-track-azurecli
ms.openlocfilehash: c253c6bf81305b9b336525c20980cf9599463648
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102209872"
---
# <a name="set-up-your-azure-red-hat-openshift-dev-environment"></a>Настройка среды разработки Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3,11 будет прекращена 30 июня 2022. Поддержка для создания новых кластеров Azure Red Hat OpenShift 3,11 продолжится до 30 ноября 2020. После выхода из эксплуатации оставшиеся кластеры Azure Red Hat OpenShift 3,11 будут закрыты для предотвращения уязвимости системы безопасности.
> 
> Следуйте указаниям этого руководством, чтобы [создать кластер Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Если у вас есть определенные вопросы, [свяжитесь с нами](mailto:arofeedback@microsoft.com).

Для сборки и запуска Microsoft Azure приложений OpenShift Red Hat необходимо:

* Установите версию 2.0.65 (или более позднюю) Azure CLI (или используйте Azure Cloud Shell).
* Зарегистрируйтесь для `AROGA` доступа к функции и связанным поставщикам ресурсов.
* Создайте клиент Azure Active Directory (Azure AD).
* Создайте объект приложения Azure AD.
* Создайте пользователя Azure AD.

Приведенные ниже инструкции покажут все необходимые условия.

## <a name="install-the-azure-cli"></a>Установка Azure CLI

Для Azure Red Hat OpenShift требуется версия 2.0.65 или более поздняя Azure CLI. Если вы уже установили Azure CLI, можно проверить, какая версия установлена, выполнив команду:

```azurecli
az --version
```

Первая строка выходных данных будет иметь версию CLI, например `azure-cli (2.0.65)` .

Ниже приведены инструкции по [установке Azure CLI](/cli/azure/install-azure-cli) , если требуется новая установка или обновление.

Кроме того, можно использовать [Azure Cloud Shell](../cloud-shell/overview.md). При использовании Azure Cloud Shell не забудьте выбрать среду **Bash** , если вы планируете следовать инструкциям из руководства по [созданию и управлению кластером Azure Red Hat OpenShift](tutorial-create-cluster.md) .

## <a name="register-providers-and-features"></a>Регистрация поставщиков и компонентов

Компонент,,, `Microsoft.ContainerService AROGA` `Microsoft.Solutions` `Microsoft.Compute` `Microsoft.Storage` `Microsoft.KeyVault` и `Microsoft.Network` поставщики должны быть зарегистрированы в своей подписке вручную перед развертыванием первого кластера Azure Red Hat OpenShift.

Чтобы зарегистрировать эти поставщики и компоненты вручную, используйте следующие инструкции оболочки bash, если вы установили интерфейс командной строки или в сеансе Azure Cloud Shell (bash) в портал Azure:

1. Если у вас несколько подписок Azure, выберите нужный идентификатор подписки:

    ```azurecli
    az account set --subscription <SUBSCRIPTION ID>
    ```

1. Зарегистрируйте компонент Microsoft. ContainerService АРОГА:

    ```azurecli
    az feature register --namespace Microsoft.ContainerService -n AROGA
    ```

1. Зарегистрируйте поставщик Microsoft. Storage:

    ```azurecli
    az provider register -n Microsoft.Storage --wait
    ```
    
1. Зарегистрируйте поставщик Microsoft. COMPUTE:

    ```azurecli
    az provider register -n Microsoft.Compute --wait
    ```

1. Зарегистрируйте поставщика Microsoft. Solutions:

    ```azurecli
    az provider register -n Microsoft.Solutions --wait
    ```

1. Зарегистрируйте поставщик Microsoft. Network:

    ```azurecli
    az provider register -n Microsoft.Network --wait
    ```

1. Зарегистрируйте поставщик Microsoft. KeyVault:

    ```azurecli
    az provider register -n Microsoft.KeyVault --wait
    ```

1. Обновите регистрацию поставщика ресурсов Microsoft. ContainerService:

    ```azurecli
    az provider register -n Microsoft.ContainerService --wait
    ```

## <a name="create-an-azure-active-directory-azure-ad-tenant"></a>Создание клиента Azure Active Directory (Azure AD)

Службе Azure Red Hat OpenShift требуется связанный клиент Azure Active Directory (Azure AD), представляющий вашу организацию и ее связь с Майкрософт. Клиент Azure AD позволяет регистрировать, создавать приложения и управлять ими, а также использовать другие службы Azure.

Если у вас нет Azure AD для использования в качестве клиента для кластера Azure Red Hat OpenShift или вы хотите создать клиент для тестирования, следуйте инструкциям в статье [Создание клиента Azure AD для кластера OpenShift для Azure Red Hat](howto-create-tenant.md) , прежде чем продолжить работу с этим руководством.

## <a name="create-an-azure-ad-user-security-group-and-application-object"></a>Создание пользователя Azure AD, группы безопасности и объекта приложения

Azure Red Hat OpenShift требуются разрешения для выполнения задач в кластере, например для настройки хранилища. Эти разрешения представлены с помощью [субъекта-службы](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object). Вам также потребуется создать нового пользователя Active Directory для тестирования приложений, выполняющихся в кластере Azure Red Hat OpenShift.

Следуйте инструкциям в разделе [Создание объекта приложения Azure AD и пользователя](howto-aad-app-configuration.md) для создания субъекта-службы, создание секрета клиента и URL обратного вызова проверки подлинности для вашего приложения, а также создание новой группы безопасности Azure AD и пользователя для доступа к кластеру.

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы готовы использовать Azure Red Hat OpenShift!

Воспользуйтесь руководством.
> [!div class="nextstepaction"]
> [Создание кластера Azure Red Hat OpenShift](tutorial-create-cluster.md)

[azure-cli-install]: /cli/azure/install-azure-cli