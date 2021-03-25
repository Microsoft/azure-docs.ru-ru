---
title: Учебник. Создание шаблона и его развертывание
description: Создайте свой первый шаблон Azure Resource Manager (шаблон ARM). В этом учебнике вы узнаете о синтаксисе файла шаблона и о том, как развернуть учетную запись хранения.
author: mumian
ms.date: 12/17/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 001a20c38a82d053ae22a2614a32f0be9d8aabce
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97656994"
---
# <a name="tutorial-create-and-deploy-your-first-arm-template"></a>Руководство по Создание и развертывание шаблона ARM

В этом руководстве рассматриваются шаблоны Azure Resource Manager (ARM). В нем показано, как создать изначальный шаблон и развернуть его в Azure. Вы узнаете о структуре шаблона и средствах, необходимых для работы с шаблонами. Для работы с этим учебником требуется около **12 минут**, но фактическое время зависит от того, сколько средств необходимо установить.

Этот учебник первый в серии. Работая с этой серией, вы будете пошагово изменять исходный шаблон, пока не изучите основные компоненты шаблона Resource Manager. Эти элементы являются стандартными блоками для более сложных шаблонов. Мы надеемся, что в конце серии вы сможете уверенно создавать собственные шаблоны и будете готовы к автоматизации развертываний с помощью шаблонов.

Если вы хотите узнать о преимуществах использования шаблонов и о том, почему следует автоматизировать развертывание с их помощью, ознакомьтесь с [общими сведениями о шаблонах ARM](overview.md). Чтобы узнать больше о шаблонах Resource Manager из модулей Microsoft Learn, см. статью [Развертывание ресурсов и управление ими в Azure с помощью шаблонов Resoure Manager](/learn/paths/deploy-manage-resource-manager-templates/).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="get-tools"></a>Получение средств

Убедитесь, что у вас есть средства, необходимые для создания и развертывания шаблонов. Установите средства на локальном компьютере.

### <a name="editor"></a>Редактор

Шаблоны — это файлы JSON. Для создания шаблонов нужен редактор JSON. Советуем использовать Visual Studio Code с расширением средств Resource Manager. Если необходимо установить эти средства, см. статью [Краткое руководство. Создание шаблонов ARM с помощью Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

### <a name="command-line-deployment"></a>Развертывание из командной строки

Для развертывания шаблона используйте либо Azure CLI, либо Azure PowerShell. При использовании Azure CLI необходимо иметь последнюю версию. Ознакомьтесь с инструкциями по установке:

- [Установка Azure PowerShell](/powershell/azure/install-az-ps)
- [Установка Azure CLI в Windows](/cli/azure/install-azure-cli-windows)
- [Установка Azure CLI в Linux](/cli/azure/install-azure-cli-linux)
- [Установка Azure CLI в macOS](/cli/azure/install-azure-cli-macos)

После установки Azure PowerShell или Azure CLI убедитесь, что вы вошли в систему в первый раз. Дополнительные сведения см. в разделах для [PowerShell](/powershell/azure/install-az-ps#sign-in) или [Azure CLI](/cli/azure/get-started-with-azure-cli#sign-in).

> [!IMPORTANT]
> Если вы используете Azure CLI, убедитесь, что у вас установлена версия 2.6 или более поздняя. Команды, приведенные в этом учебнике, не будут работать, если вы используете более ранние версии. Выполните команду `az --version`, чтобы узнать установленную версию.

Итак, все готово для начала изучения шаблонов.

## <a name="create-your-first-template"></a>Создание первого шаблона

1. Откройте Visual Studio Code с расширением средств Resource Manager.
1. В меню **Файл** щелкните **Новый файл**, чтобы создать файл.
1. В меню **Файл** щелкните **Сохранить как**.
1. Присвойте файлу имя _azuredeploy_ и выберите расширение файла _JSON_. Полное имя файла — _azuredeploy.json_.
1. Сохраните этот файл на рабочую станцию. Выберите путь, который легко запомнить, так как он будет указан позже при развертывании шаблона.
1. Скопируйте следующий код JSON и вставьте его в файл.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": []
    }
    ```

    Вот как должна выглядеть ваша среда Visual Studio Code:

    ![Первый шаблон ARM в Visual Studio Code](./media/template-tutorial-create-first-template/resource-manager-visual-studio-code-first-template.png)

    Этот шаблон не развертывает ресурсы. Начнем с пустого шаблона, чтобы вы могли ознакомиться с инструкциями по развертыванию шаблона и свести к минимуму вероятность ошибки.

    Файл JSON содержит следующие элементы:

    - `$schema`: расположение файла схемы JSON. В файле схемы описываются свойства, доступные в шаблоне. Например, схема определяет `resources` как одно из допустимых свойств шаблона. Не беспокойтесь, что для схемы указана дата 2019-04-01. Эта версия схемы обновлена и включает все новейшие функции. Дата схемы не изменилась, так как в ней отсутствуют критические изменения.
    - `contentVersion`: версия шаблона (например, 1.0.0.0). Для этого элемента можно предоставить любое значение. Это значение позволяет задокументировать важные изменения в шаблоне. При развертывании ресурсов с помощью шаблона это значение позволяет убедиться в том, что используется нужный шаблон.
    - `resources` Содержит ресурсы, которые требуется развернуть или обновить. В настоящее время элемент пуст, но вы добавите ресурсы позже.

1. Сохраните файл.

Поздравляем, вы создали свой первый шаблон.

## <a name="sign-in-to-azure"></a>Вход в Azure

Чтобы начать работу с Azure PowerShell или Azure CLI, выполните вход, используя данные своей учетной записи в Azure.

Выберите вкладки в следующих разделах кода, чтобы выбрать между Azure PowerShell и Azure CLI. Примеры интерфейса командной строки в этой статье написаны для оболочки bash.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Connect-AzAccount
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az login
```

---

Если у вас несколько подписок Azure, выберите ту, которую хотите использовать. Замените `[SubscriptionID/SubscriptionName]` и квадратные скобки `[]` сведениями о подписке:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Set-AzContext [SubscriptionID/SubscriptionName]
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az account set --subscription [SubscriptionID/SubscriptionName]
```

---

## <a name="create-resource-group"></a>Создать группу ресурсов

При развертывании шаблона необходимо указать группу ресурсов, которая будет содержать развернутые ресурсы. Перед выполнением команды развертывания создайте группу ресурсов с помощью Azure CLI или Azure PowerShell.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name myResourceGroup `
  -Location "Central US"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az group create \
  --name myResourceGroup \
  --location "Central US"
```

---

## <a name="deploy-template"></a>Развертывание шаблона

Для развертывания шаблона используйте либо Azure CLI, либо Azure PowerShell. Используйте группу ресурсов, созданную ранее. Присвойте имя развертыванию, чтобы его можно было легко найти в журнале развертываний. Для удобства также создайте переменную, в которой хранится путь к файлу шаблона. Эта переменная упрощает выполнение команд развертывания, так как вам не нужно повторно вводить путь при каждом развертывании. Замените `{provide-the-path-to-the-template-file}` и фигурные скобки `{}` путем к файлу шаблона.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$templateFile = "{provide-the-path-to-the-template-file}"
New-AzResourceGroupDeployment `
  -Name blanktemplate `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
templateFile="{provide-the-path-to-the-template-file}"
az deployment group create \
  --name blanktemplate \
  --resource-group myResourceGroup \
  --template-file $templateFile
```

---

Результаты выполнения команды развертывания выглядят так: Найдите `ProvisioningState`, чтобы определить успешность развертывания.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

![Состояние подготовки развертывания PowerShell](./media/template-tutorial-create-first-template/resource-manager-deployment-provisioningstate.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

![Состояние подготовки развертывания Azure CLI](./media/template-tutorial-create-first-template/azure-cli-provisioning-state.png)

---

> [!NOTE]
> Если развертывание завершилось сбоем, используйте параметр `verbose`, чтобы получить сведения о создаваемых ресурсах. Используйте параметр `debug`, чтобы получить дополнительные сведения для отладки.

## <a name="verify-deployment"></a>Проверка развертывания

Чтобы проверить развертывание, просмотрите группу ресурсов на портале Azure.

1. Войдите на [портал Azure](https://portal.azure.com).

1. В меню слева выберите **Группы ресурсов**.

1. Выберите группу ресурсов, развернутую в предыдущей процедуре. Имя по умолчанию — **myResourceGroup**. В группе ресурсов не должно быть развернуто ни одного ресурса.

1. Обратите внимание, что в правом верхнем углу обзора отображается состояние развертывания. Щелкните **1 Succeeded** (1: Успешно).

   ![Просмотр состояния развертывания](./media/template-tutorial-create-first-template/deployment-status.png)

1. Отобразится журнал развертываний для группы ресурсов. Выберите **blanktemplate**.

   ![Выбор развертывания](./media/template-tutorial-create-first-template/select-from-deployment-history.png)

1. В колонке отобразится сводка по развертыванию. В этом случае ничего не отображается, так как ни один ресурс не был развернут. Далее в этой серии может оказаться полезным просмотреть сводку в журнале развертывания. Обратите внимание, что в левой части можно просматривать входные и выходные данные, а также шаблон, используемый во время развертывания.

   ![Просмотр сводки по развертыванию](./media/template-tutorial-create-first-template/view-deployment-summary.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы переходите к следующему учебнику, вам не нужно удалять группу ресурсов.

Если вы прекращаете работу, то можете удалить группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

Вы создали простой шаблон для развертывания в Azure. В следующем руководстве вы добавите учетную запись хранения в шаблон и развернете ее в группе ресурсов.

> [!div class="nextstepaction"]
> [Добавление ресурса](template-tutorial-add-resource.md)
