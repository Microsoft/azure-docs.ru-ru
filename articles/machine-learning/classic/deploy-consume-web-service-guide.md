---
title: 'ML Studio (классическая модель): развертывание и использование — Azure'
description: Вы можете использовать Машинное обучение Azure Studio (классическая модель) для развертывания рабочих процессов и моделей машинного обучения в качестве веб-служб. Затем эти веб-службы можно использовать для вызова моделей машинного обучения из приложений в Интернете, чтобы делать прогнозы в режиме реального времени или в пакетном режиме.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 04/19/2017
ms.openlocfilehash: 57c5462cd6eccab1d9c42e674e10ea34db3612b7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100520363"
---
# <a name="azure-machine-learning-studio-classic-web-services-deployment-and-consumption"></a>Веб-службы Машинное обучение Azure Studio (классическая модель): развертывание и использование

**ПРИМЕНИМО К:**  ![Применимо к.](../../../includes/media/aml-applies-to-skus/yes.png)Студия машинного обучения (классическая)   ![Неприменимо к.](../../../includes/media/aml-applies-to-skus/no.png)[Машинное обучение Azure](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Вы можете использовать Машинное обучение Azure Studio (классическая модель) для развертывания рабочих процессов и моделей машинного обучения в качестве веб-служб. Затем эти веб-службы можно использовать для вызова моделей машинного обучения из приложений в Интернете, чтобы делать прогнозы в режиме реального времени или в пакетном режиме. Так как это веб-службы RESTful, их можно вызывать, используя различные языки программирования и платформы, например .NET и Java, а также приложения, например Microsoft Excel.

В следующих разделах представлены ссылки на пошаговые инструкции, код и документацию, которые помогут вам приступить к работе.

## <a name="deploy-a-web-service"></a>Развертывание веб-службы

### <a name="with-azure-machine-learning-studio-classic"></a>С помощью Машинное обучение Azure Studio (классическая модель)

Портал Studio (классическая модель) и портал веб-служб Машинное обучение Microsoft Azure помогают развертывать веб-службу и управлять ею без написания кода.

В статьях по следующим ссылкам содержатся общие сведения о процессе развертывания новой веб-службы:

* Общие сведения о развертывании новой веб-службы на основе Azure Resource Manager см. в статье [Развертывание новой веб-службы](deploy-a-machine-learning-web-service.md).
* Пошаговое руководство по развертыванию веб-службы см. в статье [Развертывание веб-службы машинного обучения Azure](deploy-a-machine-learning-web-service.md).
* Полное пошаговое руководство по созданию и развертыванию веб-службы см. в [руководстве 1. прогнозирование кредитного риска](tutorial-part1-credit-risk.md).
* Конкретные примеры развертывания веб-службы см. в следующих статьях:

  * [Учебник 3. Развертывание модели кредитных рисков](tutorial-part3-credit-risk-deploy.md)
  * [Развертывание веб-службы в нескольких регионах](deploy-a-machine-learning-web-service.md#multi-region)

### <a name="with-web-services-resource-provider-apis-azure-resource-manager-apis"></a>С помощью интерфейсов API поставщика ресурсов веб-служб (интерфейсов API Azure Resource Manager)

Поставщик ресурсов Машинное обучение Azure Studio (классическая модель) для веб-служб позволяет развертывать веб-службы и управлять ими с помощью вызовов REST API. Дополнительные сведения см. в статье [Azure Machine Learning Studio Management REST APIs](/rest/api/machinelearning/index) (REST API для управления Студией машинного обучения Azure).

<!-- [Machine Learning Web Service (REST)](/rest/api/machinelearning/webservices) reference. -->

### <a name="with-powershell-cmdlets"></a>С помощью командлетов PowerShell

Поставщик ресурсов Машинное обучение Azure Studio (классическая модель) для веб-служб позволяет развертывать веб-службы и управлять ими с помощью командлетов PowerShell.

Чтобы использовать командлеты, необходимо сначала войти в учетную запись Azure из среды PowerShell с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) . Если вы не знакомы с вызовом команд PowerShell на основе Resource Manager, см. статью [Использование Azure PowerShell с Azure Resource Manager](../../azure-resource-manager/management/manage-resources-powershell.md).

Чтобы экспортировать прогнозный эксперимент, используйте [этот пример кода](https://github.com/ritwik20/AzureML-WebServices). После создания EXE-файла из кода можно ввести следующую команду:

```azurepowershell
C:\<folder>\GetWSD <experiment-url> <workspace-auth-token>
```

При запуске приложения создается шаблон JSON веб-службы. Чтобы использовать шаблон для развертывания веб-службы, необходимо добавить следующие сведения:

* Имя и ключ учетной записи хранения.

    Их можно получить на [портале Azure](https://portal.azure.com/).
* Идентификатор плана предложения.

    Идентификатор плана можно получить на портале [веб-служб машинного обучения Azure](https://services.azureml.net). Для этого необходимо войти на портал и щелкнуть имя плана.

Добавьте их в шаблон JSON в качестве дочерних элементов узла *Properties* на том же уровне, где находится узел *MachineLearningWorkspace*.

Приведем пример:

```json
"StorageAccount": {
        "name": "YourStorageAccountName",
        "key": "YourStorageAccountKey"
},
"CommitmentPlan": {
    "id": "subscriptions/YouSubscriptionID/resourceGroups/YourResourceGroupID/providers/Microsoft.MachineLearning/commitmentPlans/YourPlanName"
}
```

Дополнительные сведения см. в следующих статьях и примерах кода:

* Справочник по [командлетам машинное обучение Azure Studio (классическая модель)](/powershell/module/az.machinelearning) на сайте MSDN

## <a name="consume-the-web-services"></a>Использование веб-служб

### <a name="from-the-azure-machine-learning-web-services-ui-testing"></a>Через пользовательский интерфейс веб-служб Машинного обучения Azure (тестирование)

Веб-службу можно проверить на портале веб-служб Машинного обучения Azure. Эта проверка включает в себя тестирование интерфейсов службы запрос-ответ (RRS) и службы пакетного выполнения (BES).

* [Развертывание новой веб-службы](deploy-a-machine-learning-web-service.md)
* [Развертывание веб-службы машинного обучения Azure](deploy-a-machine-learning-web-service.md)
* [Учебник 3. Развертывание модели кредитных рисков](tutorial-part3-credit-risk-deploy.md)

### <a name="from-excel"></a>Из Excel

Вы можете скачать шаблон Excel, который использует веб-службу:

* [Использование веб-службы Машинного обучения Azure в Excel](consuming-from-excel.md)
* [Надстройка Excel для веб-служб Машинное обучение Azure](excel-add-in-for-web-services.md)

### <a name="from-a-rest-based-client"></a>Из клиента на основе REST

Веб-службы Машинного обучения Azure представляют собой интерфейсы API RESTful. Эти API можно использовать на различных платформах, таких как .NET, Python, R, Java и т. д. На странице **Использование** веб-службы на [портале веб-служб машинное обучение Microsoft Azure](https://services.azureml.net) есть пример кода, который поможет приступить к работе. Дополнительные сведения см. в статье [Как использовать веб-службу машинного обучения Azure, развернутую из эксперимента машинного обучения](consume-web-services.md).