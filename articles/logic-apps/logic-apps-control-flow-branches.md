---
title: Создание или присоединение параллельных ветвей для действий в рабочих процессах
description: Узнайте, как создавать или объединять параллельно выполняющиеся ветви для независимых действий рабочих процессов в Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 10/10/2018
ms.openlocfilehash: 3514ce966e1de9af1741de6b966964aca2599610
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91269239"
---
# <a name="create-or-join-parallel-branches-for-workflow-actions-in-azure-logic-apps"></a>Создание или присоединение параллельных ветвей для действий рабочих процессов в Azure Logic Apps

По умолчанию действия в рабочих процессах приложений логики выполняются последовательно. Для выполнения независимых действий одновременно можно создать [параллельные ветви](#parallel-branches), а затем [соединить эти ветви](#join-branches) в потоке позднее. 

> [!TIP] 
> Если есть триггер, который получает массив, и нужно запустить рабочий процесс для каждого элемента массива, вы можете выполнить *индивидуальную обработку* этого массива с помощью [свойства **SplitOn** триггера](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch).

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас нет ее, вы можете [зарегистрироваться для получения бесплатной учетной записи Azure](https://azure.microsoft.com/free/). 

* Базовые знания [создания приложений логики](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="parallel-branches"></a>

## <a name="add-parallel-branch"></a>Добавление параллельной ветви

Для одновременного выполнения независимых действий можно добавить параллельные ветви рядом с существующим шагом. 

![Выполнение шагов в параллельном режиме](media/logic-apps-control-flow-branches/parallel.png)

Перед продолжением рабочего процесса приложение логики ожидает завершения всех ветвей. Параллельные ветви выполняются, только если их значения свойств `runAfter` соответствуют состоянию завершенного родительского шага. Например, оба действия `branchAction1` и `branchAction2` настроены для выполнения только тогда, когда действие `parentAction` завершается с состоянием `Succeeded`.

> [!NOTE]
> Прежде чем приступать, в приложении логики уже должен присутствовать шаг, на котором можно добавить параллельные ветви.

1. На <a href="https://portal.azure.com" target="_blank">портале Azure</a> откройте приложение логики в конструкторе приложений логики.

1. Наведите указатель мыши на стрелку над шагом, для которого необходимо добавить параллельные ветви. Щелкните появившийся знак **плюс** ( **+** ), а затем выберите **Добавить параллельную ветвь**. 

   ![Добавление параллельной ветви](media/logic-apps-control-flow-branches/add-parallel-branch.png)

1. В поле поиска найдите и выберите необходимое действие.

   ![Снимок экрана, показывающий окно "Выбор действия" в конструкторе приложений логики.](media/logic-apps-control-flow-branches/find-select-parallel-action.png)

   Теперь выбранное действие отображается в параллельной ветви, как показано ниже:

   ![Поиск и выбор нужных действий](media/logic-apps-control-flow-branches/added-parallel-branch.png)

1. Теперь добавьте нужные шаги для каждой параллельной ветви. Чтобы добавить последующее действие в ветвь, необходимо навести указатель мыши над действием, в которое его необходимо добавить. Щелкните появившийся знак **плюс** ( **+** ), затем выберите **Добавить действие**.

   ![Добавление последующего действия в параллельную ветвь](media/logic-apps-control-flow-branches/add-sequential-action.png)

1. В поле поиска найдите и выберите необходимое действие.

   ![Снимок экрана, показывающий окно "Выбор действия" и поле поиска в конструкторе приложений логики.](media/logic-apps-control-flow-branches/find-select-sequential-action.png)

   Теперь выбранное действие отображается в текущей ветви, как показано ниже:

   ![Поиск и выбор последующего действия](media/logic-apps-control-flow-branches/added-sequential-action.png)

Для выполнения обратного слияния ветвей [соедините параллельные ветви](#join-branches). 

<a name="parallel-json"></a>

## <a name="parallel-branch-definition-json"></a>Определение параллельной ветви (JSON)

При работе в представлении кода вместо этого можно определить параллельную структуру в определении JSON приложения логики, например:

``` json
{
  "triggers": {
    "myTrigger": {}
  },
  "actions": {
    "parentAction": {
      "type": "<action-type>",
      "inputs": {},
      "runAfter": {}
    },
    "branchAction1": {
      "type": "<action-type>",
      "inputs": {},
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "branchAction2": {
      "type": "<action-type>",
      "inputs": {},
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

<a name="join-branches"></a>

## <a name="join-parallel-branches"></a>Соединение параллельных ветвей

Для слияния параллельных ветвей внизу каждой ветви необходимо добавить шаг. Этот шаг будет выполнен, когда все параллельные ветви завершат работу.

![Соединение параллельных ветвей](media/logic-apps-control-flow-branches/join.png)

1. На [портале Azure](https://portal.azure.com) найдите приложение логики и откройте его в конструкторе приложений логики. 

1. Под параллельными ветвями, которые нужно соединить, выберите **Новый шаг**. 

   ![Добавление шага для объединения](media/logic-apps-control-flow-branches/add-join-step.png)

1. В поле поиска найдите и выберите необходимое действие в качестве шага, который соединяет ветви.

   ![Поиск и выбор действия, которое соединяет параллельные ветви](media/logic-apps-control-flow-branches/join-steps.png)

   Теперь ваши параллельные ветви объединены.

   ![Объединенные ветви](media/logic-apps-control-flow-branches/joined-branches.png)

<a name="join-json"></a>

## <a name="join-definition-json"></a>Определение соединения (JSON)

При работе в представлении кода вместо этого можно определить структуру соединения в определении JSON приложения логики, например:

``` json
{
  "triggers": {
    "myTrigger": { }
  },
  "actions": {
    "parentAction": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {}
    },
    "branchAction1": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "branchAction2": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "joinAction": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "branchAction1": [
          "Succeeded"
        ],
        "branchAction2": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

## <a name="get-support"></a>Получение поддержки

* Если у вас есть вопросы, посетите [страницу вопросов и ответов по Azure Logic Apps на сайте Майкрософт](/answers/topics/azure-logic-apps.html).
* Оставить предложения по функциям или проголосовать за них вы можете на [сайте отзывов пользователей Azure Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Дальнейшие действия

* [Conditional statements: Run steps based on a condition in logic apps](../logic-apps/logic-apps-control-flow-conditional-statement.md) (Условные операторы. Выполнение шагов на основе условия в приложениях логики)
* [Операторы switch. Выполнение различных шагов на основе различных значений в приложениях логики](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Loops: Process arrays or repeat actions until a condition is met](../logic-apps/logic-apps-control-flow-loops.md) (Циклы. Обработка массивов или повторение действий до выполнения условия)
* [Области. Выполнение шагов на основе состояния группы в приложениях логики](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
