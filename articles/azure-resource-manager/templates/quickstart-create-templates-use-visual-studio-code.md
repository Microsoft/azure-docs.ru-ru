---
title: Создание шаблона — Visual Studio Code
description: Для работы с шаблонами Azure Resource Manager (шаблоны ARM) используйте Visual Studio Code и расширение средств Azure Resource Manager.
author: neilpeterson
ms.date: 08/09/2020
ms.topic: quickstart
ms.author: nepeters
ms.openlocfilehash: 3e41d4f6f8b8aeb2be376ae1385189fe02e9b897
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102504663"
---
# <a name="quickstart-create-arm-templates-with-visual-studio-code"></a>Краткое руководство. Создание шаблонов ARM с помощью Visual Studio Code

Средства Azure Resource Manager для Visual Studio Code предоставляют языковую поддержку, фрагменты ресурсов и функцию автозавершения ресурсов. Эти средства помогают создавать и проверять шаблоны Azure Resource Manager (шаблоны ARM). В этом кратком руководстве показано, как с помощью расширения создать шаблон ARM с нуля. При этом вы воспользуетесь возможностями расширений, например фрагментами кода шаблона Resource Manager, функциями проверки и завершения, а также поддержкой файла параметров.

Для работы с этим кратким руководством вам необходимо установить [Visual Studio Code](https://code.visualstudio.com/) с [расширением средств Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools). Вам также потребуется установить и аутентифицировать [Azure CLI](/cli/azure/) или [модуль Azure PowerShell](/powershell/azure/new-azureps-module-az).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="create-an-arm-template"></a>Создание шаблона Resource Manager

Создайте и откройте файл *azuredeploy.json* с помощью Visual Studio Code. Введите `arm` в редактор кода, который инициирует фрагмент кода Azure Resource Manager для формирования шаблона Resource Manager.

Выберите `arm!`, чтобы создать шаблон для развертывания группы ресурсов Azure.

![Изображение с демонстрацией фрагментов для формирования шаблонов Azure Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/1.png)

В этом фрагменте кода создаются стандартные блоки для шаблона Resource Manager.

![Изображение, отображающее полностью сформированный шаблон Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/2.png)

Обратите внимание, что языковой режим Visual Studio Code изменился с *JSON* на *шаблон Azure Resource Manager*. Расширение включает в себя языковой сервер, который предоставляет возможности проверки и завершения, и другие языковые службы, предназначенные для шаблонов Resource Manager.

![Изображение, отображающее Azure Resource Manager в языковом режиме Visual Studio Code](./media/quickstart-create-templates-use-visual-studio-code/3.png)

## <a name="add-an-azure-resource"></a>Добавление ресурса Azure

Расширение содержит фрагменты кода для многих ресурсов Azure. С помощью этих фрагментов кода можно легко добавить ресурсы к развертыванию шаблона.

Поместите курсор в блок **ресурсов** шаблона, введите `storage` и выберите фрагмент кода *arm-storage*.

![Изображение, отображающее ресурс, добавляемый в шаблон Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/4.png)

В результате этого действия в шаблон будет добавлен ресурс хранилища.

![Изображение, отображающее ресурс службы хранилища Azure в шаблоне Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/5.png)

Вы можете переключаться между настраиваемыми свойствами в учетной записи хранения с помощью клавиши **TAB**.

![Изображение, показывающее, как можно использовать клавишу TAB, чтобы перемещаться в конфигурации ресурсов](./media/quickstart-create-templates-use-visual-studio-code/6.png)

## <a name="completion-and-validation"></a>Завершение и проверка

Одной из наиболее мощных возможностей расширения является интеграция со схемами Azure. Схемы Azure предоставляют расширения с возможностями проверки и завершения ресурсов. Давайте изменим учетную запись хранения, чтобы увидеть возможности проверки и завершения в действии.

Сначала задайте для типа учетной записи хранения недопустимое значение, например `megaStorage`. Обратите внимание, что в результате этого действия появится предупреждение, уведомляющее, что `megaStorage` не является допустимым значением.

![Изображение, отображающее недопустимую конфигурацию хранилища](./media/quickstart-create-templates-use-visual-studio-code/7.png)

Чтобы использовать возможности завершения, удалите `megaStorage`, поместите курсор внутри двойных кавычек и нажмите `ctrl` + `space`. Отобразится список завершения допустимых значений.

![Изображение, отображающее автозавершение расширения](./media/quickstart-create-templates-use-visual-studio-code/8.png)

## <a name="add-template-parameters"></a>Добавление параметров шаблона

Теперь создайте параметр и воспользуйтесь им, чтобы указать имя учетной записи хранения.

Поместите курсор в блок параметров, добавьте символ возврата каретки, введите `"`, а затем выберите фрагмент кода `new-parameter`. В шаблон будет добавлен универсальный параметр.

![Изображение, отображающее параметр, добавляемый в шаблон Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/9.png)

Измените имя параметра на `storageAccountName`, а описание на `Storage Account Name`.

![Изображение, отображающее готовый параметр в шаблоне Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/10.png)

Минимальная длина имен учетных записей хранения Azure — 3 символа, а максимальная — 24. Добавьте в параметр `minLength` и `maxLength`, а затем укажите соответствующие значения.

![Изображение, отображающее, что значения minLength и maxLength добавляются в параметр шаблона Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/11.png)

Теперь обновите свойство Name в ресурсе хранилища, чтобы использовать параметр. Для этого удалите текущее имя. Введите двойные кавычки и открывающуюся квадратную скобку `[`, которая вызывает список функций шаблона Resource Manager. В списке выберите *parameters*.

![Изображение, отображающее автоматическое завершение при использовании параметров в ресурсах шаблона Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/12.png)

Если ввести одинарную кавычку `'` внутри круглых скобок, откроется список всех параметров, определенных в шаблоне, в данном случае — *storageAccountName*. Выберите параметр.

![Изображение, отображающее готовый параметр в ресурсе шаблона Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/13.png)

## <a name="create-a-parameter-file"></a>Создание файла параметров

Файл параметров шаблона Resource Manager позволяет сохранять значения параметров среды и передавать их в виде группы во время развертывания. Например, у вас может быть один файл параметров со значениями для конкретной тестовой среды, а второй — для рабочей среды.

Расширение позволяет легко создавать файл параметров на основе имеющихся шаблонов. Для этого щелкните правой кнопкой мыши шаблон в редакторе кода и выберите `Select/Create Parameter File`.

![Изображение, отображающее процесс выбора правой кнопкой шаблона Resource Manager для создания файла параметров](./media/quickstart-create-templates-use-visual-studio-code/14.png)

Щелкните `New` > `All Parameters`, а затем выберите имя и расположение файла параметров.

![Изображение, отображающее диалоговое окно для сохранения и указания имени файла при создании файла параметров на основе шаблона Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/15.png)

В результате этого действия будет создан файл параметров, который затем сопоставляется с шаблоном, на основе которого он был создан. Вы можете выбрать параметр, чтобы просмотреть и изменить текущий шаблон и сопоставление файла параметров в строке состояния Visual Studio Code.

![Изображение, показывающее сопоставление шаблона или файла параметров в строке состояния Visual Studio Code.](./media/quickstart-create-templates-use-visual-studio-code/16.png)

Теперь, когда файл параметров сопоставлен с шаблоном, расширение проверит шаблон и файл параметров вместе. Чтобы увидеть эту проверку на практике, добавьте двухсимвольное значение в параметр `storageAccountName` в файле параметров и сохраните файл.

![Изображение, отображающее недопустимый шаблон из-за проблемы файла параметров](./media/quickstart-create-templates-use-visual-studio-code/17.png)

Вернитесь к шаблону Resource Manager и обратите внимание, что возникла ошибка, указывающая, что значение не соответствует критериям параметра.

![Изображение, отображающее допустимый шаблон Resource Manager](./media/quickstart-create-templates-use-visual-studio-code/18.png)

Установите соответствующее значение, сохраните файл и вернитесь к шаблону. Обратите внимание, что ошибка в параметре устранена.

## <a name="deploy-the-template"></a>Развертывание шаблона

Откройте встроенный терминал Visual Studio Code, используя сочетание клавиш `ctrl` + ```` ` ````, и разверните шаблон с помощью Azure CLI или модуля Azure PowerShell.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az group create --name arm-vscode --location eastus

az deployment group create --resource-group arm-vscode --template-file azuredeploy.json --parameters azuredeploy.parameters.json
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell
New-AzResourceGroup -Name arm-vscode -Location eastus

New-AzResourceGroupDeployment -ResourceGroupName arm-vscode -TemplateFile ./azuredeploy.json -TemplateParameterFile ./azuredeploy.parameters.json
```
---

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы Azure больше не нужны, используйте Azure CLI или модуль Azure PowerShell, чтобы удалить группу ресурсов, созданную для краткого руководства.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az group delete --name arm-vscode
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell
Remove-AzResourceGroup -Name arm-vscode
```
---

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [руководств для начинающих](./template-tutorial-create-first-template.md).
