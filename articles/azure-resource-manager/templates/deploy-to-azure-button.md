---
title: Кнопка "Развертывание в Azure"
description: Используйте кнопку для развертывания шаблонов Azure Resource Manager из репозитория GitHub.
ms.topic: conceptual
ms.date: 11/10/2020
ms.openlocfilehash: abe59f377474540e9209691df8b1d1a7b806c26d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98028749"
---
# <a name="use-a-deployment-button-to-deploy-templates-from-github-repository"></a>Использование кнопки развертывания для развертывания шаблонов из репозитория GitHub

В этой статье описывается, как использовать кнопку **развертывание в Azure** для развертывания шаблонов из репозитория GitHub. Вы можете добавить кнопку непосредственно в файл _readme.md_ в репозитории GitHub. Или можно добавить кнопку на веб-страницу, которая ссылается на репозиторий.

Область развертывания определяется схемой шаблона. Дополнительные сведения см. в разделе:

- [группы ресурсов](deploy-to-resource-group.md)
- [оформления](deploy-to-subscription.md)
- [группы управления](deploy-to-management-group.md)
- [клиентов](deploy-to-tenant.md)

## <a name="use-common-image"></a>Использовать общий образ

Чтобы добавить кнопку на веб-страницу или в репозиторий, используйте следующий рисунок:

```markdown
![Deploy to Azure](https://aka.ms/deploytoazurebutton)
```

```html
<img src="https://aka.ms/deploytoazurebutton"/>
```

Изображение выглядит следующим образом:

![Кнопка "Развертывание в Azure"](https://aka.ms/deploytoazurebutton)

## <a name="create-url-for-deploying-template"></a>Создание URL-адреса для развертывания шаблона

Чтобы создать URL-адрес для шаблона, начните с необработанного URL-адреса шаблона в репозитории. Чтобы просмотреть необработанный URL-адрес, выберите **необработанный**.

:::image type="content" source="./media/deploy-to-azure-button/select-raw.png" alt-text="выбрать необработанный":::

URL-адрес имеет формат:

```html
https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
```

Затем преобразуйте URL-адрес в значение, закодированное URL-адресом. Можно использовать интерактивный кодировщик или выполнить команду. В следующем примере PowerShell показано, как кодировать значение в URL-адрес.

```powershell
$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
[uri]::EscapeDataString($url)
```

URL-адрес в примере имеет следующее значение при кодировании URL-адреса.

```html
https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-storage-account-create%2Fazuredeploy.json
```

Каждая ссылка начинается с того же базового URL-адреса:

```html
https://portal.azure.com/#create/Microsoft.Template/uri/
```

Добавьте ссылку на шаблон в кодировке URL-адреса в конец базового URL-адреса.

```html
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-storage-account-create%2Fazuredeploy.json
```

У вас есть полный URL-адрес ссылки.

Как правило, шаблон размещается в общедоступном репозитории. При использовании частного репозитория необходимо включить маркер для доступа к необработанному содержимому шаблона. Токен, созданный GitHub, действителен в течение короткого времени. Часто приходится обновлять ссылку.

Если вы используете [Git с Azure Repos](/azure/devops/repos/git/) вместо репозитория GitHub, вы по-прежнему можете использовать кнопку **развертывание в Azure** . Убедитесь, что репозиторий является общедоступным. Используйте [операцию Items](/rest/api/azure/devops/git/items/get) для получения шаблона. Ваш запрос должен иметь следующий формат:

```http
https://dev.azure.com/{organization-name}/{project-name}/_apis/git/repositories/{repository-name}/items?scopePath={url-encoded-path}&api-version=6.0
```

Закодировать этот URL-адрес запроса.

## <a name="create-deploy-to-azure-button"></a>Кнопка "создать развертывание в Azure"

Наконец, разместите ссылку и изображение вместе.

Чтобы добавить кнопку с Markdown в файл _readme.md_ в репозитории GitHub или на веб-странице, используйте:

```markdown
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-storage-account-create%2Fazuredeploy.json)
```

Для HTML используйте:

```html
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-storage-account-create%2Fazuredeploy.json" target="_blank">
  <img src="https://aka.ms/deploytoazurebutton"/>
</a>
```

Для Git с репозиторием Azure кнопка имеет формат:

```markdown
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fdev.azure.com%2Forgname%2Fprojectname%2F_apis%2Fgit%2Frepositories%2Freponame%2Fitems%3FscopePath%3D%2freponame%2fazuredeploy.json%26api-version%3D6.0)
```

## <a name="deploy-the-template"></a>Развертывание шаблона

Чтобы протестировать полное решение, нажмите следующую кнопку:

[![Развертывание в Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-storage-account-create%2Fazuredeploy.json)

На портале отображается панель, с помощью которой можно легко указать значения параметров. Параметры предварительно заполняются значениями по умолчанию из шаблона.

![Развертывание с помощью портала](./media/deploy-to-azure-button/portal.png)

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о шаблонах см. в статье [Общие сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
