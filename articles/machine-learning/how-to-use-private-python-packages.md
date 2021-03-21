---
title: Использование частных пакетов Python
titleSuffix: Azure Machine Learning
description: Узнайте, как безопасно работать с частными пакетами Python из Машинное обучение Azureных сред.
services: machine-learning
author: rastala
ms.author: roastala
ms.reviewer: laobri
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 07/10/2020
ms.openlocfilehash: 5cc1025528959f8c81aa80703e3fc26d59c5266f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102618106"
---
# <a name="use-private-python-packages-with-azure-machine-learning"></a>Использование частных пакетов Python с Машинное обучение Azure


Из этой статьи вы узнаете, как безопасно использовать закрытые пакеты Python в Машинное обучение Azure. Варианты использования частных пакетов Python включают:

 * Вы разработали частный пакет, который не хочет поделиться общедоступным.
 * Вы хотите использовать проверенный репозиторий пакетов, хранящихся в корпоративном брандмауэре.

Рекомендуемый подход зависит от наличия нескольких пакетов для одной Машинное обучение Azure рабочей области или всего репозитория пакетов для всех рабочих областей в Организации.

Частные пакеты используются через класс [среды](/python/api/azureml-core/azureml.core.environment.environment) . В среде вы объявляете, какие пакеты Python следует использовать, включая частные. Дополнительные сведения о среде в Машинное обучение Azure см. в разделе [использование сред](how-to-use-environments.md). 

## <a name="prerequisites"></a>Предварительные условия

 * [Пакет SDK для машинное обучение Azure для Python](/python/api/overview/azure/ml/install)
 * [Рабочая область машинное обучение Azure](how-to-manage-workspace.md)

## <a name="use-small-number-of-packages-for-development-and-testing"></a>Использование небольшого числа пакетов для разработки и тестирования

Для небольшого числа частных пакетов в одной рабочей области используйте статический [`Environment.add_private_pip_wheel()`](/python/api/azureml-core/azureml.core.environment.environment#add-private-pip-wheel-workspace--file-path--exist-ok-false-) метод. Такой подход позволяет быстро добавить в рабочую область частный пакет, который хорошо подходит для целей разработки и тестирования.

Укажите аргумент пути к файлу в локальном файле колеса и выполните ```add_private_pip_wheel``` команду. Команда возвращает URL-адрес, используемый для наблюдения за расположением пакета в рабочей области. Запишите URL-адрес хранилища и передайте ему `add_pip_package()` метод.

```python
whl_url = Environment.add_private_pip_wheel(workspace=ws,file_path = "my-custom.whl")
myenv = Environment(name="myenv")
conda_dep = CondaDependencies()
conda_dep.add_pip_package(whl_url)
myenv.python.conda_dependencies=conda_dep
```

На внутреннем уровне Машинное обучение Azure служба заменяет URL-адрес защищенным URL-адресом SAS, поэтому файл колеса остается закрытым и безопасным.

## <a name="use-a-repository-of-packages-from-azure-devops-feed"></a>Использование репозитория пакетов из веб-канала Azure DevOps

Если вы активно разрабатываете пакеты Python для приложения машинного обучения, их можно разместить в репозитории Azure DevOps как артефакты и опубликовать в виде веб-канала. Такой подход позволяет интегрировать рабочий процесс DevOps для создания пакетов с помощью рабочая область машинного обучения Azure. Чтобы узнать, как настроить веб-каналы Python с помощью Azure DevOps, прочитайте статью [Начало работы с пакетами Python в Azure Artifacts](/azure/devops/artifacts/quickstarts/python-packages)

Этот подход использует личный маркер доступа для проверки подлинности в репозитории. Такой же подход применим к другим репозиториям с проверкой подлинности на основе маркеров, например закрытых репозиториях GitHub. 

 1. [Создайте личный маркер доступа (PAT)](/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?tabs=preview-page#create-a-pat) для своего экземпляра Azure DevOps. Задайте область токена для __упаковки > чтения__. 

 2. Добавьте URL-адрес Azure DevOps и PAT в качестве свойств рабочей области с помощью метода [Workspace.set_connection](/python/api/azureml-core/azureml.core.workspace.workspace#set-connection-name--category--target--authtype--value-) .

     ```python
    from azureml.core import Workspace
    
    pat_token = input("Enter secret token")
    ws = Workspace.from_config()
    ws.set_connection(name="connection-1", 
        category = "PythonFeed",
        target = "https://<my-org>.pkgs.visualstudio.com", 
        authType = "PAT", 
        value = pat_token) 
     ```

 3. Создайте среду Машинное обучение Azure и добавьте пакеты Python из веб-канала.
    
    ```python
    from azureml.core import Environment
    from azureml.core.conda_dependencies import CondaDependencies
    
    env = Environment(name="my-env")
    cd = CondaDependencies()
    cd.add_pip_package("<my-package>")
    cd.set_pip_option("--extra-index-url https://<my-org>.pkgs.visualstudio.com/<my-project>/_packaging/<my-feed>/pypi/simple")
    env.python.conda_dependencies=cd
    ```

Теперь среда готова к использованию при обучении или развертывании конечной точки веб-службы. При создании среды служба Машинное обучение Azure использует службу PAT для проверки подлинности в канале с совпадающим базовым URL-адресом.

## <a name="use-a-repository-of-packages-from-private-storage"></a>Использование репозитория пакетов из частного хранилища

Вы можете использовать пакеты из учетной записи хранения Azure в брандмауэре вашей организации. Учетная запись хранения может содержать проверенный набор пакетов или внутреннее зеркало общедоступных пакетов.

Сведения о настройке этого частного хранилища см. в статье [защита машинное обучение Azure рабочей области и связанных ресурсов](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-service-endpoints). Также необходимо [разместить реестр контейнеров Azure (запись контроля доступа) за виртуальной](how-to-secure-workspace-vnet.md#enable-azure-container-registry-acr)сетью.

> [!IMPORTANT]
> Необходимо выполнить этот шаг, чтобы иметь возможность обучать или развертывать модели с помощью частного репозитория пакетов.

После завершения этих конфигураций можно сослаться на пакеты в определении среды Машинное обучение Azure по полному URL-адресу в хранилище больших двоичных объектов Azure.

## <a name="next-steps"></a>Дальнейшие действия

 * Дополнительные сведения о [корпоративной безопасности в машинное обучение Azure](concept-enterprise-security.md)