---
title: Использование свойства JSON content-type для пар "ключ-значение"
titleSuffix: Azure App Configuration
description: Сведения об использовании типа содержимого JSON для значений ключа
services: azure-app-configuration
author: avanigupta
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: azurecli
ms.topic: how-to
ms.date: 08/03/2020
ms.author: avgupta
ms.openlocfilehash: 19de46bc87b72ada221c63e36e87d0545304d344
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102122159"
---
# <a name="leverage-content-type-to-store-json-key-values-in-app-configuration"></a>Использование Content-Type для хранения значений ключа JSON в конфигурации приложения

Данные хранятся в конфигурации приложения в виде значений ключа, где значения по умолчанию обрабатываются как строковые типы. Однако можно указать пользовательский тип, используя свойство Content-Type, связанное с каждым значением ключа, чтобы сохранить исходный тип данных или поведение приложения по-разному в зависимости от типа содержимого.


## <a name="overview"></a>Обзор

В конфигурации приложения тип мультимедиа JSON можно использовать в качестве типа содержимого ключа, чтобы получить такие преимущества, как:
- **Упрощенное управление данными**. Управление значениями ключа, как и массивы, значительно упрощает портал Azure.
- **Расширенный экспорт данных**: типы-примитивы, массивы и объекты JSON будут сохранены во время экспорта данных.
- **Встроенная поддержка поставщика конфигурации приложения**: значения ключа с типом содержимого JSON будут работать нормально при использовании библиотек поставщика конфигурации приложений в приложениях.

#### <a name="valid-json-content-type"></a>Допустимый тип содержимого JSON

Типы носителей, как определено [здесь](https://www.iana.org/assignments/media-types/media-types.xhtml), могут быть назначены типу содержимого, связанному с каждым ключевым значением.
Тип мультимедиа состоит из типа и подтипа. Если тип является `application` и подтипом (или суффиксом) является `json` , тип мультимедиа будет считаться допустимым типом содержимого JSON.
Ниже приведены некоторые примеры допустимых типов содержимого JSON.

- приложение/json
- приложение/действие + JSON
- application/vnd. foobar + JSON; charset = UTF-8

#### <a name="valid-json-values"></a>Допустимые значения JSON

Если значение ключа имеет тип содержимого JSON, его значение должно быть в допустимом формате JSON, чтобы клиенты правильно обрабатывали его. В противном случае клиенты могут завершаться сбоем или возвращаться и обрабатывать их в виде строкового формата.
Ниже приведены некоторые примеры допустимых значений JSON.

- "John Doe"
- 723
- false
- null
- "2020-01-01T12:34:56.789 Z"
- [1, 2, 3, 4]
- {"Обжектсеттинг": {"нацеливание": {"default": true, "уровень": "Information"}}}

> [!NOTE]
> В оставшейся части этой статьи любое значение ключа в конфигурации приложения с допустимым типом содержимого JSON и допустимым значением JSON будет называться как **значение ключа JSON**. 

В этом руководстве вы узнаете, как:
> [!div class="checklist"]
> * Создание значений ключа JSON в конфигурации приложения.
> * Импорт значений ключа JSON из JSON-файла.
> * Экспорт значений ключа JSON в JSON-файл.
> * Использование значений ключа JSON в приложениях.


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим руководством требуется версия 2.10.0 или более поздняя Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]


## <a name="create-json-key-values-in-app-configuration"></a>Создание значений ключа JSON в конфигурации приложения

Значения ключа JSON можно создавать с помощью портал Azure, Azure CLI или путем импорта из JSON-файла. В этом разделе вы найдете инструкции по созданию одних и тех же значений ключа JSON с помощью всех трех методов.

### <a name="create-json-key-values-using-azure-portal"></a>Создание значений ключа JSON с помощью портал Azure

Перейдите к хранилищу конфигураций приложений и выберите **Обозреватель конфигураций**  >  **создать**  >  **ключ — значение** , чтобы добавить следующие пары "ключ-значение":

| Ключ | Значение | Тип содержимого |
|---|---|---|
| Параметры: BackgroundColor | Зеленого | приложение/json |
| Параметры: размер шрифта | 24 | приложение/json |
| Параметры: Уседефаултраутинг | false | приложение/json |
| Параметры: Блоккедусерс | null | приложение/json |
| Параметры: ReleaseDate | "2020-08-04T12:34:56.789 Z" | приложение/json |
| Параметры: Роллаутперцентаже | [25, 50, 75100] | приложение/json |
| Параметры: ведение журнала | {"Test": {"уровень": "Debug"}, "произв": {"Level": "warning"}} | приложение/json |

Оставьте поле **Метка** пустым и нажмите кнопку **Применить**.

### <a name="create-json-key-values-using-azure-cli"></a>Создание значений ключа JSON с помощью Azure CLI

Следующие команды будут создавать значения ключа JSON в хранилище конфигурации приложения. Замените `<appconfig_name>` именем хранилища конфигураций приложения.

```azurecli-interactive
appConfigName=<appconfig_name>
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:BackgroundColor --value \"Green\"
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:FontSize --value 24
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:UseDefaultRouting --value false
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:BlockedUsers --value null
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:ReleaseDate --value \"2020-08-04T12:34:56.789Z\"
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:RolloutPercentage --value [25,50,75,100]
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:Logging --value {\"Test\":{\"Level\":\"Debug\"},\"Prod\":{\"Level\":\"Warning\"}}
```

> [!IMPORTANT]
> Если для создания значений ключа JSON используется Azure CLI или Azure Cloud Shell, то указанное значение должно быть экранированной строкой JSON.

### <a name="import-json-key-values-from-a-file"></a>Импорт значений ключа JSON из файла

Создайте JSON `Import.json` -файл с помощью следующего содержимого и импортируйте его как значения ключа в конфигурацию приложения:

```json
{
  "Settings": {
    "BackgroundColor": "Green",
    "BlockedUsers": null,
    "FontSize": 24,
    "Logging": {"Test":{"Level":"Debug"},"Prod":{"Level":"Warning"}},
    "ReleaseDate": "2020-08-04T12:34:56.789Z",
    "RolloutPercentage": [25,50,75,100],
    "UseDefaultRouting": false
  }
}
```

```azurecli-interactive
az appconfig kv import -s file --format json --path "~/Import.json" --content-type "application/json" --separator : --depth 2
```

> [!Note]
> `--depth`Аргумент используется для выравнивания иерархических данных из файла в пары "ключ-значение". В этом учебнике глубина указана для демонстрации того, что можно также хранить объекты JSON в качестве значений в конфигурации приложения. Если параметр depth не указан, объекты JSON по умолчанию будут сведены к более глубокому уровню.

Создаваемые значения ключа JSON должны выглядеть следующим образом в конфигурации приложения:

![Хранилище конфигураций, содержащее значения ключа JSON](./media/create-json-settings.png)


## <a name="export-json-key-values-to-a-file"></a>Экспорт значений ключа JSON в файл

Одним из основных преимуществ использования значений ключа JSON является возможность сохранения исходных типов данных значений при экспорте. Если значение ключа в конфигурации приложения не имеет типа содержимого JSON, его значение будет рассматриваться как строка. 

Рассмотрите следующие значения ключей без содержимого JSON-Type:

| Ключ | Значение | Тип содержимого |
|---|---|---|
| Параметры: размер шрифта | 24 | |
| Параметры: Уседефаултраутинг | false | |

При экспорте этих значений ключа в JSON-файл значения будут экспортированы в виде строк:
```json
{
  "Settings": {
    "FontSize": "24",
    "UseDefaultRouting": "false"
  }
}
```

Однако при экспорте значений ключа JSON в файл все значения сохраняют свой исходный тип данных. Чтобы проверить это, экспортируйте значения ключа из конфигурации приложения в JSON-файл. Вы увидите, что экспортированный файл имеет то же содержимое, что и `Import.json` импортируемый ранее файл.

```azurecli-interactive
az appconfig kv export -d file --format json --path "~/Export.json" --separator :
```

> [!NOTE]
> Если хранилище конфигурации приложения имеет некоторые значения ключа без содержимого JSON-Type, они также будут экспортированы в один и тот же файл в строковом формате. Если вы хотите экспортировать только значения ключа JSON, назначьте уникальную метку или префикс для значений ключа JSON и используйте фильтрацию по меткам или префиксу во время экспорта.


## <a name="consuming-json-key-values-in-applications"></a>Использование значений ключа JSON в приложениях

Самым простым способом использования значений ключа JSON в приложении являются библиотеки поставщиков конфигурации приложений. При использовании библиотек поставщиков не требуется реализовывать специальную обработку значений ключа JSON в приложении. Они будут проанализированы и преобразованы в соответствии с собственной конфигурацией приложения.

Например, если в конфигурации приложения имеется следующее ключевое значение:

| Ключ | Значение | Тип содержимого |
|---|---|---|
| Параметры | {"FontSize": 24, "Уседефаултраутинг": false} | приложение/json |

Конфигурация приложения .NET будет иметь следующие значения ключа:

| Ключ | Значение |
|---|---|
| Параметры: размер шрифта | 24 |
| Параметры: Уседефаултраутинг | false |

Вы можете получить доступ к новым ключам напрямую или [привязать значения конфигурации к экземплярам объектов .NET](/aspnet/core/fundamentals/configuration/#bind-hierarchical-configuration-data-using-the-options-pattern).


> [!Important]
> Собственная поддержка для значений ключа JSON доступна в поставщике конфигурации .NET версии 4.0.0 (или более поздней). Дополнительные сведения см. в разделе [*дальнейшие действия*](#next-steps) .

Если вы используете пакет SDK или REST API для считывания значений ключа из конфигурации приложения на основе типа содержимого, приложение отвечает за анализ значения ключа JSON.


## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы умеете работать с значениями ключа JSON в хранилище конфигураций приложения, создайте приложение для использования этих значений ключа:

* [Краткое руководство по ASP.NET Core](./quickstart-aspnet-core-app.md)
    * Предварительные требования: [Microsoft. Azure. аппконфигуратион. AspNetCore](https://www.nuget.org/packages/Microsoft.Azure.AppConfiguration.AspNetCore) Package v 4.0.0 или более поздней версии.

* [Краткое руководство по .NET](./quickstart-dotnet-core-app.md)
    * Необходимое условие: [Microsoft.Extensions.Configфигурации. Азуреаппконфигуратион](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureAppConfiguration) Package v 4.0.0 или более поздней версии.
