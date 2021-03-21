---
title: GitHub Actions для CI/CD
titleSuffix: Azure Machine Learning
description: Узнайте, как создать рабочий процесс действий GitHub для обучения модели на Машинное обучение Azure
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: juliakm
ms.author: jukullam
ms.date: 10/19/2020
ms.topic: conceptual
ms.custom: github-actions-azure
ms.openlocfilehash: b21f53f8ec76257fc19e0e30cd025ecc46ad2188
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102218287"
---
# <a name="use-github-actions-with-azure-machine-learning"></a>Использование GitHub Actions с Машинным обучением Azure

Начните работу с [действиями GitHub](https://docs.github.com/en/actions) , чтобы обучить модель на машинное обучение Azure. 

> [!NOTE]
> Действия GitHub для Машинное обучение Azure предоставляются "как есть" и не полностью поддерживаются корпорацией Майкрософт. Если возникли проблемы с конкретным действием, откройте проблему в репозитории для действия. Например, если возникла проблема с действием AML-deploy, сообщите о проблеме в [https://github.com/Azure/aml-deploy]( https://github.com/Azure/aml-deploy) репозитории.

## <a name="prerequisites"></a>Предварительные требования 

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Учетная запись GitHub. Если у вас ее нет, зарегистрируйтесь [бесплатно](https://github.com/join).  

## <a name="workflow-file-overview"></a>Общие сведения о файле рабочего процесса

Рабочий процесс определяется файлом YAML (.yml) по пути `/.github/workflows/` в вашем репозитории. Это определение содержит разные шаги и параметры рабочего процесса.

Файл состоит из четырех разделов:

|Section  |Задания  |
|---------|---------|
|**Аутентификация** | 1. Определение субъекта-службы. <br /> 2. Создание секрета GitHub. |
|**Подключить** | 1. Подключитесь к рабочей области машинного обучения. <br /> 2. Подключитесь к целевому объекту вычислений. |
|**Выполнить** | 1. Отправьте обучающий запуск. |
|**Развертывание** | 1. Зарегистрируйте модель в реестре Машинное обучение Azure. 1. Развертывание модели. |

## <a name="create-repository"></a>Создать репозиторий

Создайте репозиторий с помощью операций [ml с действиями GitHub и шаблоном машинное обучение Azure](https://github.com/machine-learning-apps/ml-template-azure). 

1. Откройте [шаблон](https://github.com/machine-learning-apps/ml-template-azure) на сайте GitHub. 
2. Щелкните **Использовать этот шаблон**. 

    :::image type="content" source="media/how-to-github-actions-machine-learning/gh-actions-use-template.png" alt-text="Выберите использовать этот шаблон":::
3. Создайте новый репозиторий из шаблона. В качестве имени репозитория укажите `ml-learning` или выберите нужное имя. 


## <a name="generate-deployment-credentials"></a>Создание учетных данных для развертывания.

Вы можете создать [субъект-службу](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) с помощью команды [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) в [Azure CLI](/cli/azure/). Чтобы выполнить эту команду, откройте [Azure Cloud Shell](https://shell.azure.com/) на портале Azure или нажмите кнопку **Попробовать**.

```azurecli-interactive
az ad sp create-for-rbac --name "myML" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name> \
                            --sdk-auth
```

В приведенном выше примере Замените заполнители ИДЕНТИФИКАТОРом подписки, именем группы ресурсов и именем приложения. Выходные данные содержат объект JSON с учетными данными назначения роли, которые предоставляют доступ к приложению Службы приложений, как показано ниже. Скопируйте этот объект JSON для последующего использования.

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

## <a name="configure-the-github-secret"></a>Настройка секрета GitHub

1. В [GitHub](https://github.com/)найдите репозиторий, выберите **параметры > секреты > добавить новый секрет**.

2. Вставьте все выходные данные JSON, полученные из команды Azure CLI, в поле значения секрета. Присвойте секрету имя `AZURE_CREDENTIALS`.

## <a name="connect-to-the-workspace"></a>Подключение к рабочей области

Используйте [действие Рабочая область машинного обучения Azure](https://github.com/marketplace/actions/azure-machine-learning-workspace) для подключения к рабочей области машинное обучение Azure. 

```yaml
    - name: Connect/Create Azure Machine Learning Workspace
      id: aml_workspace
      uses: Azure/aml-workspace@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```

По умолчанию действие принимает `workspace.json` файл. Если файл JSON имеет другое имя, его можно указать с помощью `parameters_file` параметра input. Если файл не существует, новый объект будет создан с именем репозитория.


```yaml
    - name: Connect/Create Azure Machine Learning Workspace
      id: aml_workspace
      uses: Azure/aml-workspace@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          parameters_file: "alternate_workspace.json"
```
Действие записывает свойства рабочей области Azure Resource Manager (ARM) в файл конфигурации, который будет выбран всеми дальнейшими действиями Машинное обучение Azure GitHub. Файл сохраняется в `GITHUB_WORKSPACE/aml_arm_config.json` . 

## <a name="connect-to-a-compute-target-in-azure-machine-learning"></a>Подключение к целевому объекту вычислений в Машинное обучение Azure

Используйте [действие машинное обучение Azure вычисление](https://github.com/Azure/aml-compute) для подключения к целевому объекту вычислений в машинное обучение Azure.  Если целевой объект вычислений существует, действие будет подключаться к нему. В противном случае действие создаст новый целевой объект вычислений. [Действие AML COMPUTE](https://github.com/Azure/aml-compute) поддерживает только расчетный кластер Azure ML и службу Kubernetes Azure (AKS). 

```yaml
    - name: Connect/Create Azure Machine Learning Compute Target
      id: aml_compute_training
      uses: Azure/aml-compute@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```
## <a name="submit-training-run"></a>Выполнение обучающего прогона

Используйте [машинное обучение Azureное действие обучения](https://github.com/Azure/aml-run) для отправки скриптрун, средства оценки или конвейера для машинное обучение Azure. 

```yaml
    - name: Submit training run
      id: aml_run
      uses: Azure/aml-run@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```

## <a name="register-model-in-registry"></a>Регистрация модели в реестре

Используйте [действие машинное обучение Azure зарегистрировать модель](https://github.com/Azure/aml-registermodel) , чтобы зарегистрировать модель для машинное обучение Azure.

```yaml
    - name: Register model
      id: aml_registermodel
      uses: Azure/aml-registermodel@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          run_id:  ${{ steps.aml_run.outputs.run_id }}
          experiment_name: ${{ steps.aml_run.outputs.experiment_name }}
```

## <a name="deploy-model-to-azure-machine-learning-to-aci"></a>Развертывание модели для Машинное обучение Azure в ACI

Используйте [действие развертывание машинное обучение Azure](https://github.com/Azure/aml-deploy) , чтобы развернуть модель и создать конечную точку для модели. Можно также использовать Машинное обучение Azure развертывание для развертывания в службе Azure Kubernetes. См. [Этот пример рабочего процесса](https://github.com/Azure-Samples/mlops-enterprise-template) для модели, которая развертывается в службе Azure Kubernetes.

```yaml
    - name: Deploy model
      id: aml_deploy
      uses: Azure/aml-deploy@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          model_name:  ${{ steps.aml_registermodel.outputs.model_name }}
          model_version: ${{ steps.aml_registermodel.outputs.model_version }}

```

## <a name="complete-example"></a>Полный пример

Обучение модели и развертывание в Машинное обучение Azure. 

```yaml
# Actions train a model on Azure Machine Learning
name: Azure Machine Learning training and deployment
on:
  push:
    branches:
      - master
    # paths:
    #   - 'code/*'
jobs:
  train:
    runs-on: ubuntu-latest
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - name: Check Out Repository
      id: checkout_repository
      uses: actions/checkout@v2
        
    # Connect or Create the Azure Machine Learning Workspace
    - name: Connect/Create Azure Machine Learning Workspace
      id: aml_workspace
      uses: Azure/aml-workspace@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
    
    # Connect or Create a Compute Target in Azure Machine Learning
    - name: Connect/Create Azure Machine Learning Compute Target
      id: aml_compute_training
      uses: Azure/aml-compute@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
    
    # Submit a training run to the Azure Machine Learning
    - name: Submit training run
      id: aml_run
      uses: Azure/aml-run@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}

    # Register model in Azure Machine Learning model registry
    - name: Register model
      id: aml_registermodel
      uses: Azure/aml-registermodel@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          run_id:  ${{ steps.aml_run.outputs.run_id }}
          experiment_name: ${{ steps.aml_run.outputs.experiment_name }}

    # Deploy model in Azure Machine Learning to ACI
    - name: Deploy model
      id: aml_deploy
      uses: Azure/aml-deploy@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          model_name:  ${{ steps.aml_registermodel.outputs.model_name }}
          model_version: ${{ steps.aml_registermodel.outputs.model_version }}

```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если группа ресурсов и репозиторий больше не требуются, очистите развернутые ресурсы, удалив группу ресурсов и репозиторий GitHub. 

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание и запуск конвейеров машинного обучения с помощью пакета SDK для Машинное обучение Azure](./how-to-create-machine-learning-pipelines.md)