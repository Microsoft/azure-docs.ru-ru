---
title: Создание приложения Службы приложений с помощью шаблона Azure Resource Manager
description: Создайте свое первое приложение в Службе приложений Azure за считаные секунды с помощью шаблона Azure Resource Manager (ARM), который является одним из множества способов развертывания в Службе приложений.
author: msangapu-msft
ms.author: msangapu
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
ms.date: 10/16/2020
ms.custom: subject-armqs, devx-track-azurecli
zone_pivot_groups: app-service-platform-windows-linux
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: bce6bfb61eb59d1fa66c550a133ac8b6f8d7f2c5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107769012"
---
# <a name="quickstart-create-app-service-app-using-an-arm-template"></a>Краткое руководство. Создание приложения Службы приложений с помощью шаблона Resource Manager

Начните работу со [Службы приложений Azure](overview.md), а именно с развертывания приложения в облаке с помощью <abbr title="JSON-файла, который декларативно определяет один или несколько ресурсов Azure и зависимостей между развернутыми ресурсами. Шаблон можно использовать для согласованного и многократного развертывания ресурсов.">Шаблон ARM</abbr> и [Azure CLI](/cli/azure/get-started-with-azure-cli) в Cloud Shell. Так как вы используете бесплатный уровень Службы приложений, для прохождения этого краткого руководства никакие затраты не требуются. <abbr title="В декларативном синтаксисе вы можете описать предполагаемое развертывание без написания последовательности команд программирования для создания развертывания.">В шаблоне используется декларативный синтаксис.</abbr>

 Если окружение соответствует предварительным требованиям и вы знакомы с использованием [шаблонов ARM](../azure-resource-manager/templates/overview.md), нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

::: zone pivot="platform-windows"
[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-service-docs-windows%2Fazuredeploy.json)
::: zone-end

::: zone pivot="platform-linux"
[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-service-docs-linux%2Fazuredeploy.json)
::: zone-end

<hr/>

## <a name="1-prepare-your-environment"></a>1. Подготовка среды

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

<hr/>

## <a name="2-review-the-template"></a>2. Изучение шаблона

::: zone pivot="platform-windows"
Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-app-service-docs-windows). Он развертывает план и приложение Службы приложений в Windows.

:::code language="json" source="~/quickstart-templates/101-app-service-docs-windows/azuredeploy.json":::

<details>
<summary>Какие ресурсы и параметры определены в шаблоне?</summary>

В шаблоне определено два ресурса Azure:

* [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms): создание плана Службы приложений.
* [**Microsoft.Web/Sites**](/azure/templates/microsoft.web/sites): создание приложения Службы приложений.

В следующей таблице приведены параметры по умолчанию и их описания.

| Параметры | Тип    | Значение по умолчанию                | Описание |
|------------|---------|------------------------------|-------------|
| webAppName | строка  | webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** | Имя приложения. |
| location   | строка  | [[resourceGroup().location]](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup) | Регион приложения |
| sku        | строка  | F1                         | Размер экземпляра (F1 = уровень "Бесплатный") |
| Язык   | строка  | .NET                       | Стек языков программирования (.NET, PHP, Node или HTML) |
| helloWorld | Логическое | False                        | True = развертывание приложения Hello World |
| repoUrl    | строка  | " "                          | Внешний репозиторий Git (необязательно) |

---

</details>
::: zone-end
::: zone pivot="platform-linux"
Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-app-service-docs-linux). Он развертывает план и приложение Службы приложений в Windows.

:::code language="json" source="~/quickstart-templates/101-app-service-docs-linux/azuredeploy.json":::

Этот шаблон включает в себя ресурсы и параметры Azure, определенные для вашего удобства.

<details>
<summary>Какие ресурсы и параметры определены в шаблоне?</summary>

В шаблоне определено два ресурса Azure:

* [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms): создание плана Службы приложений.
* [**Microsoft.Web/Sites**](/azure/templates/microsoft.web/sites): создание приложения Службы приложений.

В следующей таблице приведены параметры по умолчанию и их описания.

| Параметры | Тип    | Значение по умолчанию                | Описание |
|------------|---------|------------------------------|-------------|
| webAppName | строка  | webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** | Имя приложения. |
| location   | строка  | [[resourceGroup().location]](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup) | Регион приложения |
| sku        | строка  | F1                         | Размер экземпляра (F1 = уровень "Бесплатный") |
| linuxFxVersion   | строка  | DOTNETCORE&#124;3.0        | Версия стека языка программирования &#124; |
| repoUrl    | строка  | " "                          | Внешний репозиторий Git (необязательно) |

---

</details>

::: zone-end

<hr/>

## <a name="3-deploy-the-template"></a>3. Развертывание шаблона

::: zone pivot="platform-windows"
Выполните приведенный ниже код, чтобы развернуть приложение .NET Framework в Windows с помощью Azure CLI. 

Заменить <abbr title="Допустимые символы: `a-z`, `0-9`, а также `-`.">`<app-name>`</abbr> глобально уникальным именем приложения. Чтобы узнать о других <abbr title="Вы можете также использовать Azure PowerShell, портал Azure и REST API.">методах развертывания</abbr>, см. раздел [Развертывание шаблонов](../azure-resource-manager/templates/deploy-powershell.md). Дополнительные примеры шаблонов Службы приложений Azure можно найти [здесь](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sites).

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup \
--parameters language=".net" helloWorld="true" webAppName="<app-name>" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-app-service-docs-windows/azuredeploy.json"
```
::: zone-end
::: zone pivot="platform-linux"
Выполните приведенный ниже код, чтобы создать приложение Python в Linux. 

Заменить <abbr title="Допустимые символы: `a-z`, `0-9`, а также `-`.">`<app-name>`</abbr> глобально уникальным именем приложения.

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup --parameters webAppName="<app-name>" linuxFxVersion="PYTHON|3.7" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-app-service-docs-linux/azuredeploy.json"
```
::: zone-end

<details>
<summary>Что делает код?</summary>
<p>Команды выполняют следующие действия:</p>
<ul>
<li>Создание стандартной <abbr title="Логический контейнер для связанных ресурсов Azure, которыми можно управлять как единым целым.">resource group</abbr>.</li>
<li>Создание стандартного <abbr title="План, который позволяет определить расположение, размер и функции фермы веб-серверов для размещения приложения.">План службы приложений</abbr>.</li>
<li><a href="/cli/azure/webapp#az_webapp_create">Создайте <abbr title="Представление веб-приложения, которое содержит код приложения, имена узлов DNS, сертификаты и связанные ресурсы. ">приложение Службы приложений Azure</abbr></a> с указанным именем.</li>
</ul>
</details>

::: zone pivot="platform-windows"
<details>
<summary>Как развернуть другой стек языка?</summary>
Чтобы развернуть другой стек языка, обновите <abbr title="Этот шаблон совместим с .NET Core, .NET Framework, PHP, Node.js и статическими HTML-приложениями. ">параметр языка</abbr> с соответствующими значениями. Дополнительные сведения о создании приложения Java см. в <a href="/azure/app-service/quickstart-java-uiex">этой статье</a>.

| Параметры | Тип    | Значение по умолчанию                | Описание |
|------------|---------|------------------------------|-------------|
| Язык   | строка  | .NET                       | Стек языков программирования (.NET, PHP, Node или HTML) |

---

</details>
::: zone-end
::: zone pivot="platform-linux"
<details>
<summary>Как развернуть другой стек языка?</summary>
Чтобы развернуть другой стек языка, обновите соответствующие значения `linuxFxVersion`. Примеры приведены ниже. Чтобы отобразить сведения о поддерживаемой версии, выполните следующую команду в Cloud Shell: `az webapp config show --resource-group myResourceGroup --name <app-name> --query linuxFxVersion`

| Язык    | Например, .                                              |
|-------------|------------------------------------------------------|
| **.NET**    | linuxFxVersion="DOTNETCORE&#124;3.0"                 |
| **PHP**     | linuxFxVersion="PHP&#124;7.4"                        |
| **Node.js** | linuxFxVersion="NODE&#124;10.15"                     |
| **Java**    | linuxFxVersion="JAVA&#124;1.8 &#124;TOMCAT&#124;9.0" |
| **Python**  | linuxFxVersion="PYTHON&#124;3.7"                     |
| **Ruby**    | linuxFxVersion="RUBY&#124;2.6"                       |

---

</details>
::: zone-end

<hr/>

## <a name="4-validate-the-deployment"></a>4. Проверка развертывания

Перейдите к `http://<app_name>.azurewebsites.net/` и убедитесь, что оно создано.

<hr/>

## <a name="5-clean-up-resources"></a>5. Очистка ресурсов

[Удалите группу ресурсов](../azure-resource-manager/management/delete-resource-group.md?tabs=azure-portal#delete-resource-group), если она больше не нужна.

<hr/>

## <a name="next-steps"></a>Дальнейшие действия

- Развертывание из локального репозитория Git
- [Использование ASP.NET Core с базой данных SQL](tutorial-dotnetcore-sqldb-app.md)
- Приложение Python с PostgreSQL
- [Использование PHP и MySQL](tutorial-php-mysql-app.md)
- [Подключение к Базе данных SQL Azure на Java](../azure-sql/database/connect-query-java.md?toc=%2fazure%2fjava%2ftoc.json)
- [Сопоставление пользовательского домена](app-service-web-tutorial-custom-domain-uiex.md)
