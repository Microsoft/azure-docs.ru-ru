---
title: Создание приложений логики и управление ими с помощью Azure CLI
description: Используйте Azure CLI, чтобы создать приложение логики и управлять им, выполняя операции перечисления, отображения (получение), обновления и удаления.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: quickstart
ms.custom: mvc, devx-track-azurecli, contperf-fy21q2
ms.date: 11/23/2020
ms.openlocfilehash: bc172fd1702addf8f4e34094452a779b09320a4d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97033380"
---
# <a name="quickstart-create-and-manage-logic-apps-using-the-azure-cli"></a>Краткое руководство. Создание приложений логики и управление ими с помощью Azure CLI

В этом кратком руководстве показано, как создавать приложения логики и управлять ими с помощью [расширений Azure CLI Logic Apps](/cli/azure/ext/logic/logic) (`az logic`). В командной строке можно создать приложение логики, определив его рабочий процесс с помощью JSON-файла. Затем вы можете управлять приложением логики, выполняя из командной строки такие операции, как `list`, `show` (`get`), `update` и `delete`.

> [!WARNING]
> Расширение Azure CLI Logic Apps в настоящее время *экспериментальное*, на него не предоставляется *клиентская поддержка*. Используйте это расширение CLI с осторожностью, особенно в рабочих средах.

Если вы не работали с Logic Apps, вы также можете узнать, как создать первые приложения логики с помощью [портала Azure](quickstart-create-first-logic-app-workflow.md), [Visual Studio](quickstart-create-logic-apps-with-visual-studio.md) и [Visual Studio Code](quickstart-create-logic-apps-visual-studio-code.md).

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Azure CLI](/cli/azure/install-azure-cli), установленный на локальном компьютере.
* Расширение [Logic Apps Azure CLI](/cli/azure/azure-cli-extensions-list), установленное на компьютере. Чтобы установить это расширение, используйте следующую команду: `az extension add --name logic`
* [Группа ресурсов Azure](#example---create-resource-group), в которой создается приложение логики.

### <a name="prerequisite-check"></a>Проверка предварительных условий

Перед началом работы проверьте среду, выполнив следующие действия.

* Войдите на портал Azure и убедитесь, что ваша подписка активна, выполнив команду `az login`.
* Проверьте версию Azure CLI в терминале или в командном окне, выполнив команду `az --version`. Сведения о последней версии см. в [заметках о выпуске](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Если у вас нет последней версии, обновите установку, следуя указаниям в [руководстве по установке операционной системы или платформы](/cli/azure/install-azure-cli).

### <a name="example---create-resource-group"></a>Пример. Создание группы ресурсов

Если у вас еще нет группы ресурсов для приложения логики, создайте ее с помощью команды `az group create`. Например, следующая команда создает группу ресурсов `testResourceGroup` в расположении `westus`.

```azurecli-interactive

az group create --name testResourceGroup --location westus

```

При успешном создании группы ресурсов в выходных данных `provisioningState` будет отображаться значение `Succeeded`:

```output

<...>
  "name": "testResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
<...>

```

## <a name="workflow-definition"></a>Определение рабочего процесса

Прежде чем [создать приложение логики](#create-logic-apps-from-cli) или [обновить существующее](#update-logic-apps-from-cli) с помощью Azure CLI, вам потребуется определение рабочего процесса для приложения логики. На портале Azure можно просмотреть базовое определение рабочего процесса приложения логики в формате JSON, переключив представление **Конструктор** на **Представление кода**.

При выполнении команд для создания или обновления приложения логики определение рабочего процесса отправляется в качестве обязательного параметра (`--definition`). Необходимо создать определение рабочего процесса в виде JSON-файла, который следует за [схемой языка определения рабочего процесса](./logic-apps-workflow-definition-language.md).

## <a name="create-logic-apps-from-cli"></a>Создание приложений логики из CLI

Рабочий процесс приложения логики можно создать из Azure CLI с помощью команды [`az logic workflow create`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create), используя JSON-файл для определения.

```azurecli

az logic workflow create --definition
                         --location
                         --name
                         --resource-group
                         [--access-control]
                         [--endpoints-configuration]
                         [--integration-account]
                         [--integration-service-environment]
                         [--state {Completed, Deleted, Disabled, Enabled, NotSpecified, Suspended}]
                         [--tags]

```

Команда должна включать следующие [обязательные параметры](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create-required-parameters):

| Параметр | Значение | Описание |
| --------- | ----- | ----------- |
| Определение рабочего процесса | `--definition` | JSON-файл с определением рабочего процесса [для приложения логики](#workflow-definition). |
| Расположение | `--location -l` | Регион Azure, в котором находится ваше приложение логики. |
| Имя | `--name -n` | Имя приложения логики. Имя может содержать только буквы, цифры, дефисы (`-`), символы подчеркивания (`_`), круглые скобки (`()`) и точки (`.`). Это имя должно быть уникальным в пределах регионов. |
| Имя группы ресурсов | `--resource-group -g` | [Группа ресурсов Azure](../azure-resource-manager/management/overview.md), в которой создается приложение логики. Если для приложения логики еще не создана [группа ресурсов](#example---create-resource-group), создайте ее прежде чем приступить к работе |

Кроме того, можно включить дополнительные [необязательные параметры](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create-optional-parameters) для настройки элементов управления доступом вашего приложения логики, конечных точек, учетной записи интеграции, среды службы интеграции, состояния и тегов ресурсов.

### <a name="example---create-logic-app"></a>Пример. Создание приложения логики

В этом примере в группе ресурсов `testResourceGroup` в расположении `westus` создается рабочий процесс `testLogicApp`. JSON-файл `testDefinition.json` содержит определение рабочего процесса.

```azurecli-interactive

az logic workflow create --resource-group "testResourceGroup" --location "westus" --name "testLogicApp" --definition "testDefinition.json"

```

После успешного создания рабочего процесса интерфейс командной строки отображает код JSON определения нового рабочего процесса. Если создание рабочего процесса завершается неудачей, ознакомьтесь со [списком возможных ошибок](#errors).

## <a name="update-logic-apps-from-cli"></a>Обновление приложений логики из CLI

Вы также можете обновить рабочий процесс приложения логики из Azure CLI с помощью команды [`az logic workflow create`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create).

Команда должна включать те же [обязательные параметры](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create-required-parameters), что и при [создании](#create-logic-apps-from-cli) приложения логики. Кроме того, можно добавить те же [необязательные параметры](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create-optional-parameters) как при создании приложения логики.

```azurecli

az logic workflow create --definition
                         --location
                         --name
                         --resource-group
                         [--access-control]
                         [--endpoints-configuration]
                         [--integration-account]
                         [--integration-service-environment]
                         [--state {Completed, Deleted, Disabled, Enabled, NotSpecified, Suspended}]
                         [--tags]

```

### <a name="example---update-logic-app"></a>Пример. Обновление приложения логики

В этом примере [пример рабочего процесса, созданный в предыдущем разделе,](#example---create-logic-app) обновляется для использования другого файла определения JSON (`newTestDefinition.json`) и добавления двух тегов ресурсов: `testTag1` и `testTag2` с описанием значений.

```azurecli-interactive

az logic workflow create --resource-group "testResourceGroup" --location "westus" --name "testLogicApp" --definition "newTestDefinition.json" --tags "testTag1=testTagValue1" "testTag2=testTagValue"

```

После успешного обновления рабочего процесса интерфейс командной строки отображает определения нового рабочего процесса приложения логики. Если обновление завершается неудачей, ознакомьтесь со [списком возможных ошибок](#errors).

## <a name="delete-logic-apps-from-cli"></a>Удаление приложений логики из CLI

Вы можете также удалить рабочий процесс приложения логики из Azure CLI с помощью команды [`az logic workflow delete`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-delete).

Команда должна включать следующие [обязательные параметры](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-delete-required-parameters):

| Параметр | Значение | Описание |
| --------- | ----- | ----------- |
| Имя | `--name -n` | Имя приложения логики. |
| Имя группы ресурсов | `-resource-group -g` | Группа ресурсов, в которой находится ваше приложение логики. |

Также можно включить [необязательный параметр](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-delete-optional-parameters), чтобы пропустить запросы на подтверждение `--yes -y`.

```azurecli

az logic workflow delete --name
                         --resource-group
                         [--yes]

```

Затем в интерфейсе командной строки отображается запрос на подтверждение удаления приложения логики. Запрос на подтверждение можно пропустить, используя в команде необязательный параметр `--yes -y`.

```azurecli

Are you sure you want to perform this operation? (y/n):

```

Подтвердить удаление приложения логики можно, [добавив приложения логики в интерфейс командной строки](#list-logic-apps-in-cli) или просмотрев их на портале Azure.

### <a name="example---delete-logic-app"></a>Пример. Удаление приложения логики

В этом примере удаляется [пример рабочего процесса, созданный в предыдущем разделе](#example---create-logic-app).

```azurecli-interactive

az logic workflow delete --resource-group "testResourceGroup" --name "testLogicApp"

```

После ответа на запрос подтверждения с `y` приложение логики будет удалено.

## <a name="show-logic-apps-in-cli"></a>Отображение приложений логики в интерфейсе командной строки

Конкретный рабочий процесс приложения логики можно получить с помощью команды [`az logic workflow show`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-show).

```azurecli

az logic workflow show --name
                       --resource-group

```

Команда должна включать следующие [обязательные параметры](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-show-required-parameters):

| Параметр | Значение | Описание |
| --------- | ----- | ----------- |
| Имя | `--name -n` | Имя приложения логики. |
| Имя группы ресурсов | `--resource-group -g` | Имя группы ресурсов, в которой находится ваше приложение логики. |

### <a name="example---get-logic-app"></a>Пример. Получение приложения логики

В этом примере приложение логики `testLogicApp` в группе ресурсов `testResourceGroup` возвращается с полными журналами для отладки.

```azurecli-interactive

az logic workflow show --resource-group "testResourceGroup" --name "testLogicApp" --debug

```

## <a name="list-logic-apps-in-cli"></a>Перечисление приложений логики в интерфейсе командной строки

Вы можете перечислить приложения логики по подписке с помощью команды [`az logic workflow list`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-list). Эта команда возвращает код JSON для рабочих процессов приложений логики.

Отфильтровать результаты можно по следующим [необязательным параметрам](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-list-optional-parameters):

| Параметр | Значение | Описание |
| --------- | ----- | ----------- |
| Имя группы ресурсов | `--resource-group -g` | Имя группы ресурсов, по которой нужно отфильтровать результаты. |
| Количество элементов | `--top` | Количество элементов, которые включены в результаты. |
| Filter | `--filter` | Тип фильтра, который используется в списке. Вы можете выполнить фильтрацию по состоянию (`State`), триггеру (`Trigger`) и идентификатору ресурса, на который указывает ссылка (`ReferencedResourceId`). |

```azurecli

az logic workflow list [--filter]
                       [--resource-group]
                       [--top]

```

### <a name="example---list-logic-apps"></a>Пример. Перечисление приложений логики

В этом примере все включенные рабочие процессы в группе ресурсов `testResourceGroup` возвращаются в формате таблицы ASCII.

```azurecli-interactive

az logic workflow list --resource-group "testResourceGroup" --filter "(State eq 'Enabled')" --output "table"

```

## <a name="errors"></a>ошибки

Указанная ниже ошибка означает, что расширение Azure Logic Apps CLI не установлено. Выполните действия, описанные в разделе с предварительными требованиями, чтобы [установить расширение Logic Apps](#prerequisites) на компьютере.

```output

az: 'logic' is not in the 'az' command group. See 'az --help'. If the command is from an extension, please make sure the corresponding extension is installed. To learn more about extensions, please visit https://docs.microsoft.com/cli/azure/azure-cli-extensions-overview

```

Следующая ошибка может означать, что путь к файлу для отправки определения рабочего процесса неверен.

```output

Expecting value: line 1 column 1 (char 0)

```

## <a name="global-parameters"></a>Глобальные параметры

В командах `az logic` можно использовать следующие необязательные глобальные параметры Azure CLI.

| Параметр | Значение | Описание |
| --------- | ----- | ----------- |
| Формат вывода | `--output -o` | Изменение [формата выходных данных](/cli/azure/format-output-azure-cli) файла JSON по умолчанию. |
| Отображать только ошибки | `--only-show-errors` | Отключение предупреждений и отображение только ошибок. |
| Подробный | `--verbose` | Отображение подробных журналов. |
| Отладка | `--debug` | Отображение всех журналов отладки. |
| Справочное сообщение | `--help -h` | Отображение диалогового окна справки. |
| Запрос | `--query` | Задание строки запроса JMESPath для выходных данных JSON. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure/).

Дополнительные примеры скриптов CLI для Logic Apps можно найти в [браузере примеров кода Майкрософт](/samples/browse/?products=azure-logic-apps).

Затем можно создать пример логики приложения с помощью Azure CLI, используя пример скрипта и определение рабочего процесса.

> [!div class="nextstepaction"]
> [Создание приложения логики с помощью примера скрипта](sample-logic-apps-cli-script.md).
