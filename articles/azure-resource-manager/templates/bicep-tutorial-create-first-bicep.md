---
title: Руководство. Создание и развертывание файлов Bicep Azure Resource Manager
description: Создайте первый файл Bicep для развертывания ресурсов Azure. В этом руководстве описан синтаксис файла Bicep и показано, как развернуть учетную запись хранения.
author: mumian
ms.date: 03/03/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 6a335b554fa0cfc2e12c8ddbe3e24a50fdedec0f
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102036355"
---
# <a name="tutorial-create-and-deploy-first-azure-resource-manager-bicep-file"></a>Руководство. Создание и развертывание первого файла Bicep Azure Resource Manager

В этом руководстве показано, что такое [Bicep](./bicep-overview.md). Вы узнаете, как создать изначальный файл Bicep и развернуть его в Azure. Вы также узнаете о структуре файла Bicep и средствах, необходимых для работы с файлами Bicep. Для работы с этим учебником требуется около **12 минут**, но фактическое время зависит от того, сколько средств необходимо установить.

Этот учебник первый в серии. Работая с этой серией, вы будете пошагово изменять исходный файл Bicep, пока не изучите все его основные компоненты. Эти элементы являются стандартными блоками для более сложных файлов Bicep. Мы надеемся, что в конце серии вы сможете уверенно создавать собственные файлы Bicep и будете готовы к автоматизации развертываний с их помощью.

См. дополнительные сведения о преимуществах использования файлов [Bicep](./bicep-overview.md) и о том, почему следует автоматизировать развертывание с их помощью.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="get-tools"></a>Получение средств

Убедитесь, что у вас есть средства, необходимые для создания и развертывания файлов Bicep. Установите средства на локальном компьютере.

### <a name="editor"></a>Редактор

Для создания файлов Bicep нужен хороший редактор. Рекомендуется использовать Visual Studio Code с расширением Bicep. Если необходимо установить эти средства, см. статью [Краткое руководство. Создание файлов Bicep с помощью Visual Studio Code](quickstart-create-bicep-use-visual-studio-code.md).

### <a name="command-line-deployment"></a>Развертывание из командной строки

Для развертывания файла Bicep также требуется последняя версия Azure PowerShell или Azure CLI. Ознакомьтесь с инструкциями по установке:

- [Установка Azure PowerShell](/powershell/azure/install-az-ps)
- [Установка Azure CLI в Windows](/cli/azure/install-azure-cli-windows)
- [Установка Azure CLI в Linux](/cli/azure/install-azure-cli-linux)
- [Установка Azure CLI в macOS](/cli/azure/install-azure-cli-macos)

После установки Azure PowerShell или Azure CLI убедитесь, что вы вошли в систему в первый раз. Дополнительные сведения см. в разделах для [PowerShell](/powershell/azure/install-az-ps#sign-in) или [Azure CLI](/cli/azure/get-started-with-azure-cli#sign-in).

Итак, вы готовы начать работу с файлами Bicep.

## <a name="create-your-first-bicep-file"></a>Создание первого файла Bicep

1. Откройте Visual Studio Code с установленным расширением Bicep.
1. В меню **Файл** щелкните **Новый файл**, чтобы создать файл.
1. В меню **Файл** щелкните **Сохранить как**.
1. Присвойте файлу имя _azuredeploy_ и выберите расширение файла _bicep_. Полное имя файла — _azuredeploy.bicep_.
1. Сохраните этот файл на рабочую станцию. Выберите путь, который легко запомнить, так как он будет указан позже при развертывании файла Bicep.
1. Скопируйте и вставьте в файл следующее содержимое:

    ```bicep
    resource stg 'Microsoft.Storage/storageAccounts@2019-06-01' = {
      name: '{provide-unique-name}'
      location: 'eastus'
      sku: {
        name: 'Standard_LRS'
      }
      kind: 'StorageV2'
      properties: {
        supportsHttpsTrafficOnly: true
      }
    }
    ```

    Вот как должна выглядеть ваша среда Visual Studio Code:

    ![Visual Studio Code: первый файл Bicep](./media/Bicep-tutorial-create-first-bicep/resource-manager-visual-studio-code-first-bicep.png)

    Объявление ресурса состоит из четырех компонентов:

    - **resource**: ключевое слово.
    - **symbolic name** (stg): идентификатор для ссылки на ресурс в файле Bicep. Это не имя ресурса после развертывания. Имя ресурса определяет свойство **name**.  Ознакомьтесь с четвертым компонентом в списке. Чтобы упростить работу с руководствами, **stg** используется как символьное имя для ресурса учетной записи хранения в этой серии руководств.
    - **Тип ресурса** (Microsoft.Storage/storageAccounts@2019-06-01): состоит из поставщика ресурсов (Microsoft.Storage), типа ресурса (StorageAccounts) и версии API (2019-06-01). Каждый поставщик ресурсов опубликовал собственные версии API, поэтому это значение зависит от типа. Дополнительные типы и версии API для различных ресурсов Azure можно найти в [справочнике по шаблонам ARM](/azure/templates/).
    - **properties** (все внутри = {...}): это конкретные свойства, которые нужно указать для указанного типа ресурса. Это те же свойства, которые доступны в шаблоне ARM. У каждого ресурса есть свойство `name`. Большинство ресурсов также имеют свойство `location`, которое задает регион, где развернут ресурс. Другие свойства зависят от типа ресурса и версии API. Важно понимать связь между версией API и доступными свойствами, поэтому давайте перейдем к более подробным сведениям.

        Для этой учетной записи хранения эта версия API будет отображаться здесь: [storageAccounts 2019-06-01](/azure/templates/microsoft.storage/2019-06-01/storageaccounts). Обратите внимание, что вы не добавили все свойства в свой файл Bicep. Многие из свойств являются необязательными. Поставщик ресурсов `Microsoft.Storage` может выпустить новую версию API, но развертываемая версия не должна изменяться. Вы можете продолжать использовать эту версию и знать, что результаты развертывания будут согласованы.

        Если вы просмотрите более старую версию API, такую как [storageAccounts 2016-05-01](/azure/templates/microsoft.storage/2016-05-01/storageaccounts), вы увидите, что доступен меньший набор свойств.

        Если вы решите изменить версию API для ресурса, убедитесь, что вы оценили свойства этой версии и соответствующим образом изменили свой файл Bicep.

1. Замените `{provide-unique-name}` и фигурные скобки `{}` уникальным именем учетной записи хранения.

    > [!IMPORTANT]
    > Имя учетной записи хранения должно быть уникальным в среде Azure. Имя должно содержать только строчные буквы или цифры. Оно должно содержать не больше 24 знаков. Вы можете попробовать использовать шаблон именования (например, **store1**) в качестве префикса, а затем добавить свои инициалы и сегодняшнюю дату. Например, имя, которое вы используете, может выглядеть таким образом: **store1abc09092019**.

    Подобрать уникальное имя для учетной записи хранения не так просто и не подходит для автоматизации больших развертываний. Далее в этой серии руководств вы будете использовать компоненты файлов Bicep, которые облегчат создание уникального имени.

1. Сохраните файл.

Поздравляем, вы создали свой первый файл Bicep!

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

При развертывании файла Bicep необходимо указать группу ресурсов, которая будет содержать ресурсы. Перед выполнением команды развертывания создайте группу ресурсов с помощью Azure CLI или Azure PowerShell.

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

## <a name="deploy-bicep-file"></a>Развертывание файла Bicep

Bicep — это прозрачная абстракция шаблонов Azure Resource Manager (шаблонов ARM). Каждый файл Bicep компилируется в стандартный шаблон ARM перед развертыванием. Вы можете скомпилировать файл Bicep в шаблон ARM перед развертыванием или же напрямую развернуть его. Для развертывания файла Bicep используйте Azure CLI или Azure PowerShell. Используйте группу ресурсов, созданную ранее. Присвойте имя развертыванию, чтобы его можно было легко найти в журнале развертываний. Для удобства также создайте переменную, в которой хранится путь к файлу Bicep. Эта переменная упрощает запуск команд развертывания, так как вам не нужно будет повторно вводить путь при каждом развертывании. Измените `{provide-the-path-to-the-bicep-file}`, включая фигурные скобки (`{}`), на путь к файлу Bicep с именем расширения файла _.bicep_.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы выполнить этот командлет развертывания, необходимо иметь [последнюю версию](/powershell/azure/install-az-ps) Azure PowerShell.

```azurepowershell
$bicepFile = "{provide-the-path-to-the-bicep-file}"
New-AzResourceGroupDeployment `
  -Name firstbicep `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.

```azurecli
bicepFile="{provide-the-path-to-the-bicep-file}"
az deployment group create \
  --name firstbicep \
  --resource-group myResourceGroup \
  --template-file $bicepFile
```

---

Результаты выполнения команды развертывания выглядят так: Найдите `ProvisioningState`, чтобы определить успешность развертывания.

> [!NOTE]
> Если развертывание завершилось сбоем, используйте параметр `verbose`, чтобы получить сведения о создаваемых ресурсах. Используйте параметр `debug`, чтобы получить дополнительные сведения для отладки.

## <a name="verify-deployment"></a>Проверка развертывания

Чтобы проверить развертывание, просмотрите группу ресурсов на портале Azure.

1. Войдите на [портал Azure](https://portal.azure.com).

1. В меню слева выберите **Группы ресурсов**.

1. Выберите группу ресурсов, развернутую в предыдущей процедуре. Имя по умолчанию — **myResourceGroup**. В группе ресурсов не должно быть развернуто ни одного ресурса.

1. Обратите внимание, что в правом верхнем углу обзора отображается состояние развертывания. Щелкните **1 Succeeded** (1: Успешно).

   ![Просмотр состояния развертывания](./media/bicep-tutorial-create-first-bicep/deployment-status.png)

1. Отобразится журнал развертываний для группы ресурсов. Выберите **firstbicep**.

   ![Выбор развертывания](./media/bicep-tutorial-create-first-bicep/select-from-deployment-history.png)

1. В колонке отобразится сводка по развертыванию. Развернута одна учетная запись хранения. Обратите внимание, что в левой части можно просматривать входные и выходные данные, а также шаблон, используемый во время развертывания.

   ![Просмотр сводки по развертыванию](./media/bicep-tutorial-create-first-bicep/view-deployment-summary.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы переходите к следующему учебнику, вам не нужно удалять группу ресурсов.

Если вы прекращаете работу, то можете удалить группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

Вы создали простой файл Bicep для развертывания в Azure.  В последующих руководствах показано, как добавлять параметры, переменные, выходные данные и модули в файл Bicep. Эти компоненты являются строительными блоками для более сложных файлов Bicep.

> [!div class="nextstepaction"]
> [Учебник по добавлению параметров в шаблон Resource Manager](bicep-tutorial-add-parameters.md)
