---
title: Обнаружение свойств ресурсов
description: Описание процесса поиска свойств ресурсов.
ms.topic: conceptual
ms.date: 06/10/2020
ms.openlocfilehash: c8bbe0dcb1bc9dc9751a1dc0d0b98a6368473546
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91327754"
---
# <a name="discover-resource-properties"></a>Обнаружение свойств ресурсов

Перед созданием шаблонов диспетчер ресурсов необходимо понять, какие типы ресурсов доступны и какие значения использовать в шаблоне. В этой статье описаны некоторые способы поиска свойств, включаемых в шаблон.

## <a name="find-resource-provider-namespaces"></a>Поиск пространств имен поставщиков ресурсов

Ресурсы в шаблоне ARM определяются с помощью пространства имен поставщика ресурсов и типа ресурса. Например, Microsoft. Storage/storageAccounts — это полное имя типа ресурса учетной записи хранения. Microsoft. Storage — это пространство имен. Если вы еще не знакомы с пространствами имен для типов ресурсов, которые вы хотите использовать, см. статью [поставщики ресурсов для служб Azure](../management/azure-services-resource-providers.md).

![Сопоставление пространства имен поставщика ресурсов диспетчер ресурсов](./media/view-resources/resource-provider-namespace-and-azure-service-mapping.png)

## <a name="export-templates"></a>Экспорт шаблонов

Самый простой способ получить свойства шаблона для существующих ресурсов — это экспортировать шаблон. Дополнительные сведения см. в статье [экспорт одного и нескольких ресурсов в шаблон в портал Azure](./export-template-portal.md).

## <a name="use-resource-manager-tools-extension"></a>Использование расширения средств диспетчер ресурсов

Visual Studio Code и расширение Azure Resource Manager Tools помогают увидеть, какие именно свойства необходимы для каждого типа ресурса. Они предоставляют IntelliSense и фрагменты кода, упрощающие определение ресурсов в шаблоне. Дополнительные сведения см. в разделе [Краткое руководство. Создание шаблонов Azure Resource Manager с помощью Visual Studio Code](./quickstart-create-templates-use-visual-studio-code.md#add-an-azure-resource).

На следующем снимке экрана показан ресурс учетной записи хранения, добавленный в шаблон:

![Фрагменты расширения средств диспетчер ресурсов](./media/view-resources/resource-manager-tools-extension-snippets.png)

Расширение также предоставляет список параметров для свойств конфигурации.

![Настраиваемые значения расширения диспетчер ресурсов Tools](./media/view-resources/resource-manager-tools-extension-configurable-properties.png)

## <a name="use-template-reference"></a>Использование справочника по шаблонам

Справочник по шаблону Azure Resource Manager является наиболее полным ресурсом для схемы шаблона. Можно найти версии API, формат шаблона и сведения о свойствах.

1. Перейдите по [ссылке на шаблон Azure Resource Manager](/azure/templates/).
1. В левой области навигации выберите **хранилище**, а затем выберите **все ресурсы**. На странице все ресурсы перечислены типы ресурсов и версии.

    ![версии ресурсов справочника по шаблонам](./media/view-resources/resource-manager-template-reference-resource-versions.png)

    Если вы знакомы с типом ресурса, можно перейти непосредственно на эту страницу со следующим форматом URL-адреса: `https://docs.microsoft.com/azure/templates/{provider-namespace}/{resource-type}` .

1. Выберите последнюю версию. Рекомендуется использовать последнюю версию API.

    В разделе **Формат шаблона** перечислены все свойства учетной записи хранения. **номер SKU** указан в списке:

    ![формат учетной записи хранения справочника по шаблону](./media/view-resources/resource-manager-template-reference-storage-account-sku.png)

    Прокрутите вниз, чтобы просмотреть **объект SKU** в разделе **значения свойств** . В статье показаны допустимые значения для имени SKU:

    ![значения SKU учетной записи хранения справочника по шаблону](./media/view-resources/resource-manager-template-reference-storage-account-sku-values.png)

    В конце страницы в разделе **шаблоны** быстрого запуска перечислены некоторые шаблоны быстрого запуска Azure, которые содержат тип ресурса.

    ![шаблоны быстрого запуска для учетной записи хранения справочника по шаблонам](./media/view-resources/resource-manager-template-reference-quickstart-templates.png)

Ссылка на шаблон связана с каждым из сайтов документации по службе Azure.  Например, [Key Vault сайт документации](../../key-vault/general/overview.md):

![Справочник по шаблону диспетчер ресурсов Key Vault](./media/view-resources/resource-manager-template-reference-key-vault.png)

## <a name="use-resource-explorer"></a>Использование обозреватель ресурсов

Обозреватель ресурсов внедряется в портал Azure. Перед использованием этого метода вам потребуется учетная запись хранения. Если у вас ее нет, нажмите следующую кнопку, чтобы создать ее.

[![Развертывание в Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-storage-account-create%2fazuredeploy.json)

1. Войдите на [портал Azure](https://portal.azure.com).
1. В поле поиска введите **Обозреватель ресурсов**, а затем выберите **Обозреватель ресурсов**.

    ![На снимке экрана показан поиск обозреватель ресурсов в портал Azure.](./media/view-resources/azure-portal-resource-explorer.png)

1. В левой части разверните узел **подписки**, а затем разверните подписку Azure. Вы можете найти учетную запись хранения с помощью **поставщиков** или **ResourceGroups**.

    ![портал Azure обозреватель ресурсов](./media/view-resources/azure-portal-resource-explorer-home.png)

    - **Поставщики**: разверните узел **поставщики**  ->  **Microsoft. Storage**  ->  **storageAccounts**, а затем выберите свою учетную запись хранения.
    - **ResourceGroups**: выберите группу ресурсов, которая содержит учетную запись хранения, выберите **ресурсы**, а затем выберите учетную запись хранения.

    Справа вы увидите конфигурацию SKU для имеющейся учетной записи хранения, как показано ниже.

    ![Номер SKU учетной записи хранения портал Azure обозреватель ресурсов](./media/view-resources/azure-portal-resource-explorer-sku.png)

## <a name="use-resourcesazurecom"></a>Использование Resources.azure.com

Resources.azure.com — общедоступный веб-сайт, доступ к которому может получить любой пользователь с подпиской Azure. Он находится на этапе предварительной версии.  Вместо этого рекомендуется использовать [Обозреватель ресурсов](#use-resource-explorer) . Это средство предоставляет следующие возможности:

- Узнайте о API-интерфейсах управления ресурсами Azure.
- Получение документации и сведений о схеме API.
- Сделайте вызовы API непосредственно в собственных подписках.

Чтобы продемонстрировать, как получить сведения о схеме с помощью этого средства, необходима учетная запись хранения. Если у вас ее нет, нажмите следующую кнопку, чтобы создать ее.

[![Развертывание в Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-storage-account-create%2fazuredeploy.json)

1. Перейдите по [Resources.Azure.com](https://resources.azure.com/). Для широкой левой панели это средство занимает несколько секунд.
1. Выберите **подписки**.

    ![сопоставление API resource.azure.com](./media/view-resources/resources-azure-com-api-mapping.png)

    Узел слева соответствует вызову API справа. Вызов API можно выполнить, нажав кнопку **Get** .
1. В левой части разверните узел **подписки**, а затем разверните подписку Azure. Вы можете найти учетную запись хранения с помощью **поставщиков** или **ResourceGroups**.

    - **Поставщики**: разверните узел **поставщики**  ->  **Microsoft. Storage**  ->  **storageAccounts**, а затем перейдите к учетной записи хранения.
    - **ResourceGroups**: выберите группу ресурсов, которая содержит учетную запись хранения, а затем выберите **ресурсы**.

    Справа вы увидите конфигурацию SKU для имеющейся учетной записи хранения, как показано ниже.

    ![Номер SKU учетной записи хранения портал Azure обозреватель ресурсов](./media/view-resources/azure-portal-resource-explorer-sku.png)

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как найти сведения о схеме шаблона. Дополнительные сведения о создании шаблонов диспетчер ресурсов см. в разделе Общие сведения о [структуре и синтаксисе шаблонов ARM](./template-syntax.md).
