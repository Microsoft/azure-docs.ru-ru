---
title: Настройка функции в Azure для обработки данных
titleSuffix: Azure Digital Twins
description: Узнайте, как создать функцию в Azure, которая может получать доступ и вызываться цифровым двойников.
author: baanders
ms.author: baanders
ms.date: 8/27/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 2419761c195258c60561e284abf0227b915ed4f6
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102123638"
---
# <a name="connect-function-apps-in-azure-for-processing-data"></a>Подключение приложений функций в Azure для обработки данных

Обновление цифровых двойников на основе данных осуществляется с помощью [**маршрутов событий**](concepts-route-events.md) через ресурсы вычислений, таких как функция, созданная с помощью [функций Azure](../azure-functions/functions-overview.md). Функции можно использовать для обновления цифрового двойника в ответ на следующее:
* данные телеметрии устройства, поступающие из центра Интернета вещей
* изменение свойства или другие данные, поступающие из другого цифрового двойника в графе двойника

В этой статье описывается создание функции в Azure для использования с цифровым двойников Azure. 

Ниже приведен обзор шагов, которые он содержит.

1. Создание проекта функций Azure в Visual Studio
2. Написание функции с помощью триггера [сетки событий](../event-grid/overview.md)
3. Добавление кода проверки подлинности в функцию (для доступа к Azure Digital двойников)
4. Публикация приложения-функции в Azure
5. Настройка [безопасного](concepts-security.md) доступа для приложения функции

## <a name="prerequisite-set-up-azure-digital-twins-instance"></a>Предварительные требования: Настройка экземпляра Digital двойников для Azure

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

## <a name="create-a-function-app-in-visual-studio"></a>Создание приложения-функции в Visual Studio

В Visual Studio 2019 выберите _файл > создать > проект_ и выполните поиск по шаблону _функции Azure_ . Выберите _Далее_.

:::image type="content" source="media/how-to-create-azure-function/create-azure-function-project.png" alt-text="Снимок экрана Visual Studio, показывающий диалоговое окно &quot;новый проект&quot;. Шаблон проекта функций Azure выделяется.":::

Укажите имя для приложения-функции и нажмите кнопку _создать_.

:::image type="content" source="media/how-to-create-azure-function/configure-new-project.png" alt-text="Снимок экрана Visual Studio, показывающий диалоговое окно для настройки нового проекта, включая имя проекта, расположение для сохранения, возможность создания нового решения и имени решения.":::

Выберите тип приложения функции *триггера сетки событий* и нажмите кнопку _создать_.

:::image type="content" source="media/how-to-create-azure-function/event-grid-trigger-function.png" alt-text="Снимок экрана Visual Studio с отображением диалогового окна для создания нового приложения функций Azure. Параметр триггера сетки событий выделен.":::

После создания приложения-функции Visual Studio создаст пример кода в файле **function1.CS** в папке проекта. Эта небольшая функция используется для записи событий в журнал.

:::image type="content" source="media/how-to-create-azure-function/visual-studio-sample-code.png" alt-text="Снимок экрана Visual Studio в окне проекта для нового проекта, который был создан. Существует код для примера функции с именем функция1." lightbox="media/how-to-create-azure-function/visual-studio-sample-code.png":::

## <a name="write-a-function-with-an-event-grid-trigger"></a>Написать функцию с помощью триггера сетки событий

Вы можете написать функцию, добавив пакет SDK в приложение-функцию. Приложение-функция взаимодействует с Azure Digital двойников с помощью [пакета SDK Azure Digital двойников для .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true). 

Чтобы использовать пакет SDK, необходимо включить в проект следующие пакеты. Можно либо установить пакеты с помощью диспетчера пакетов NuGet Visual Studio, либо добавить пакеты с помощью `dotnet` в программе командной строки. Выполните приведенные ниже действия для выбранного метода.

**Вариант 1. Добавление пакетов с помощью диспетчера пакетов Visual Studio:**
    
Щелкните проект правой кнопкой мыши и выберите в списке пункт _Управление пакетами NuGet_ . Затем в открывшемся окне выберите вкладку _Обзор_ и выполните поиск следующих пакетов. Выберите _установить_ и _принять_ лицензионное соглашение для установки пакетов.

* `Azure.DigitalTwins.Core`
* `Azure.Identity`
* `System.Net.Http`
* `Azure.Core.Pipeline`

**Вариант 2. Добавление пакетов с помощью `dotnet` программы командной строки:**

Кроме того, можно использовать следующие `dotnet add` команды в программе командной строки:

```cmd/sh
dotnet add package Azure.DigitalTwins.Core
dotnet add package Azure.Identity
dotnet add package System.Net.Http
dotnet add package Azure.Core.Pipeline
```

Затем в Visual Studio обозреватель решений откройте файл _function1.CS_ , в котором имеется образец кода, и добавьте в `using` функцию следующие инструкции. 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="Function_dependencies":::

## <a name="add-authentication-code-to-the-function"></a>Добавление кода проверки подлинности в функцию

Теперь вы можете объявить переменные уровня класса и добавить код проверки подлинности, который позволит функции получить доступ к Azure Digital двойников. Вы добавите следующий элемент в функцию в файл _function1.CS_ .

* Код для чтения URL-адреса службы цифровых двойников Azure в качестве **переменной среды**. Рекомендуется считать URL-адрес службы из переменной среды, а не жестко кодировать его в функции. Значение этой переменной среды задается [Далее в этой статье](#set-up-security-access-for-the-function-app). Дополнительные сведения о переменных среды см. в разделе [*Управление приложением функции*](../azure-functions/functions-how-to-use-azure-function-app-settings.md?tabs=portal).

    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="ADT_service_URL":::

* Статическая переменная для хранения экземпляра HttpClient. HttpClient довольно дорого создавать, и мы хотим избежать этого при каждом вызове функции.

    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="HTTP_client":::

* Учетные данные управляемого удостоверения можно использовать в функциях Azure.
    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="ManagedIdentityCredential":::

* Добавьте локальную переменную _дигиталтвинсклиент_ в функцию для хранения своего экземпляра клиента Azure Digital двойников. *Не* делайте эту переменную статической внутри класса.
    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="DigitalTwinsClient":::

* Добавьте проверку значения NULL для _адтинстанцеурл_ и заключите логику функции в блок try/catch, чтобы перехватить все исключения.

После этих изменений код функции будет выглядеть следующим образом:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs":::

Теперь, когда приложение написано, его можно опубликовать в Azure, выполнив действия, описанные в следующем разделе.

## <a name="publish-the-function-app-to-azure"></a>Публикация приложения-функции в Azure

[!INCLUDE [digital-twins-publish-azure-function.md](../../includes/digital-twins-publish-azure-function.md)]

## <a name="set-up-security-access-for-the-function-app"></a>Настройка безопасного доступа для приложения функции

Вы можете настроить доступ к безопасности для приложения функции, используя либо Azure CLI, либо портал Azure. Выполните приведенные ниже действия для выбранного варианта.

### <a name="option-1-set-up-security-access-for-the-function-app-using-cli"></a>Вариант 1. Настройка безопасного доступа для приложения функции с помощью интерфейса командной строки

В схеме функций из предыдущих примеров требуется передать в него токен носителя, чтобы иметь возможность проходить проверку подлинности в Azure Digital двойников. Чтобы убедиться, что этот токен носителя передан, необходимо настроить [управляемое удостоверение службы (MSI)](../active-directory/managed-identities-azure-resources/overview.md) для приложения-функции. Это необходимо сделать только один раз для каждого приложения функции.

Вы можете создать управляемое системой удостоверение и назначить удостоверение приложения-функции для роли _**владельца данных Azure Digital двойников**_ для своего экземпляра Azure Digital двойников. Это предоставит приложению функции в экземпляре разрешение на выполнение действий плоскости данных. Затем сделайте URL-адрес экземпляра Azure Digital двойников доступным для вашей функции, задав переменную среды.

Используйте [Azure Cloud Shell](https://shell.azure.com) для выполнения команд.

С помощью следующей команды создайте управляемое системой удостоверение. Запишите значение поля _principalId_ в выходных данных команды.

```azurecli-interactive 
az functionapp identity assign -g <your-resource-group> -n <your-App-Service-(function-app)-name>   
```
Используйте значение _principalId_ в следующей команде, чтобы назначить удостоверение приложения-функции роли _владельца Azure Digital Twins_ для вашего экземпляра Azure Digital Twins.

```azurecli-interactive 
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<principal-ID>" --role "Azure Digital Twins Data Owner"
```
Наконец, предоставьте доступ к URL-адресу вашего экземпляра Azure Digital двойников вашей функции, задав для него **переменную среды** . Дополнительные сведения о переменных среды см. в разделе [*Управление приложением функции*](../azure-functions/functions-how-to-use-azure-function-app-settings.md?tabs=portal). 

> [!TIP]
> URL-адрес экземпляра Azure Digital двойников создается путем добавления *https://* в начало *имени узла* для цифрового двойникова Azure. Чтобы просмотреть имя узла, а также все свойства экземпляра, можно запустить `az dt show --dt-name <your-Azure-Digital-Twins-instance>` .

```azurecli-interactive 
az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "ADT_SERVICE_URL=https://<your-Azure-Digital-Twins-instance-hostname>"
```
### <a name="option-2-set-up-security-access-for-the-function-app-using-azure-portal"></a>Вариант 2. Настройка безопасного доступа для приложения функции с помощью портал Azure

Управляемое удостоверение, назначенное системой, позволяет ресурсам Azure проходить проверку подлинности в облачных службах (например, Azure Key Vault) без сохранения учетных данных в коде. После включения все необходимые разрешения можно предоставить через Управление доступом на основе ролей в Azure. Жизненный цикл этого типа управляемого удостоверения связан с жизненным циклом этого ресурса. Кроме того, каждый ресурс (например, виртуальная машина) может иметь только одно назначенное системой управляемое удостоверение.

В [портал Azure](https://portal.azure.com/)найдите _приложение функции_ в строке поиска с именем приложения функции, созданным ранее. Выберите *приложение-функция* из списка. 

:::image type="content" source="media/how-to-create-azure-function/portal-search-for-function-app.png" alt-text="Снимок экрана портал Azure: имя приложения-функции ищется на панели поиска портала, а результат поиска выделяется.":::

В окне приложение функции выберите _удостоверение_ на панели навигации слева, чтобы включить управляемое удостоверение.
На вкладке _назначенная система_ переключите _состояние_ на вкл. и _Сохраните_ его. Вы увидите всплывающее окно для _включения управляемого удостоверения, назначенного системой_.
Нажмите кнопку _Да_ . 

:::image type="content" source="media/how-to-create-azure-function/enable-system-managed-identity.png" alt-text="Снимок экрана портал Azure. на странице удостоверение для приложения-функции параметр Включить управляемое удостоверение, назначенный системе, имеет значение Да. Параметр Status имеет значение ON.":::

Вы можете проверить в уведомлениях о том, что функция успешно зарегистрирована в Azure Active Directory.

:::image type="content" source="media/how-to-create-azure-function/notifications-enable-managed-identity.png" alt-text="Снимок экрана портал Azure. в списке уведомлений выберите значок колокольчика на верхней панели портала. Имеется уведомление о том, что пользователь включил управляемое удостоверение, назначенное системой.":::

Также обратите внимание на **идентификатор объекта** , показанный на странице _удостоверений_ , как он будет использоваться в следующем разделе.

:::image type="content" source="media/how-to-create-azure-function/object-id.png" alt-text="Снимок экрана портал Azure: выделение поля &quot;идентификатор объекта&quot; на странице удостоверений функции Azure.":::

### <a name="assign-access-roles-using-azure-portal"></a>Назначение ролей доступа с помощью портал Azure

Нажмите кнопку _назначения ролей Azure_ , после чего откроется страница *назначения ролей Azure* . Затем выберите _+ добавить назначение ролей (Предварительная версия)_.

:::image type="content" source="media/how-to-create-azure-function/add-role-assignments.png" alt-text="Снимок экрана портал Azure. в разделе &quot;разрешения&quot; на странице удостоверений функции Azure выделит кнопку назначения ролей Azure.":::

На открывшейся странице _Добавление назначения ролей (Предварительная версия)_ выберите:

* _Область_ — группа ресурсов.
* _Подписка_: выберите подписку Azure.
* _Группа ресурсов_. Выберите группу ресурсов из раскрывающегося списка.
* _Роль_: выберите _Azure Digital двойников Data Owner_ в раскрывающемся списке.

Затем сохраните сведения, нажав кнопку Save ( _сохранить_ ).

:::image type="content" source="media/how-to-create-azure-function/add-role-assignment.png" alt-text="Снимок экрана портал Azure: диалоговое окно для добавления нового назначения роли (Предварительная версия). Существуют поля для области, подписки, группы ресурсов и роли.":::

### <a name="configure-application-settings-using-azure-portal"></a>Настройка параметров приложения с помощью портал Azure

Чтобы сделать URL-адрес своего экземпляра Digital двойников доступным для вашей функции, можно задать для него **переменную среды** . Дополнительные сведения о переменных среды см. в разделе [*Управление приложением функции*](../azure-functions/functions-how-to-use-azure-function-app-settings.md?tabs=portal). Параметры приложения предоставляются как переменные среды для доступа к экземпляру Digital двойников Azure. 

Чтобы задать переменную среды с URL-адресом своего экземпляра, сначала получите URL-адрес, находя имя узла своего экземпляра Azure Digital двойников. Найдите свой экземпляр на панели поиска [портал Azure](https://portal.azure.com) . Затем на левой панели навигации выберите _Обзор_ , чтобы просмотреть _имя узла_. Скопируйте это значение.

:::image type="content" source="media/how-to-create-azure-function/adt-hostname.png" alt-text="Снимок экрана портал Azure: на странице &quot;Обзор&quot; для экземпляра Azure Digital двойников выделяется значение имени узла.":::

Теперь можно создать параметр приложения, выполнив следующие действия:

1. Найдите приложение функции на панели поиска портала и выберите его из результатов.
1. Выберите пункт _Конфигурация_ на панели навигации слева, чтобы создать новый параметр приложения.
1. На вкладке _Параметры приложения_ выберите _+ создать параметр приложения_ .

:::image type="content" source="media/how-to-create-azure-function/portal-search-for-function-app.png" alt-text="Снимок экрана портал Azure: имя приложения-функции ищется на панели поиска портала, а результат поиска выделяется.":::

:::image type="content" source="media/how-to-create-azure-function/application-setting.png" alt-text="Снимок экрана портал Azure. на странице конфигурации для приложения-функции будет выделена кнопка для создания нового параметра приложения.":::

В открывшемся окне используйте скопированное выше значение имя узла, чтобы создать параметр приложения.
* **Имя**: ADT_SERVICE_URL
* **Значение**: https://{ваш-Azure-Digital-двойников-Host-Name}

Нажмите кнопку _ОК_ , чтобы создать параметр приложения.

:::image type="content" source="media/how-to-create-azure-function/add-application-setting.png" alt-text="Снимок экрана портал Azure. Кнопка &quot;ОК&quot; выделяется после заполнения полей &quot;имя&quot; и &quot;значение&quot; на странице &quot;Добавление и изменение параметров приложения&quot;.":::

Параметры приложения можно просмотреть с помощью имени приложения в поле _имя_ . Затем сохраните параметры приложения, нажав кнопку _сохранить_ .

:::image type="content" source="media/how-to-create-azure-function/application-setting-save-details.png" alt-text="Снимок экрана портал Azure: страница &quot;Параметры приложения&quot; с выделенным параметром &quot;новый ADT_SERVICE_URL&quot;. Кнопка сохранить также выделяется.":::

Чтобы изменения параметров приложения вступили в силу, потребуется перезапуск приложения. Нажмите кнопку _продолжить_ , чтобы перезапустить приложение.

:::image type="content" source="media/how-to-create-azure-function/save-application-setting.png" alt-text="Снимок экрана портал Azure. Обратите внимание, что все изменения параметров приложения с перезапуском приложения. Кнопка продолжить выделена.":::

Вы можете просмотреть параметры приложения, щелкнув значок _уведомления_ . Если параметр приложения не создан, можно повторно добавить параметр приложения, выполнив описанный выше процесс.

:::image type="content" source="media/how-to-create-azure-function/notifications-update-web-app-settings.png" alt-text="Снимок экрана портал Azure. в списке уведомлений выберите значок колокольчика на верхней панели портала. Существует уведомление о том, что параметры веб-приложения успешно обновлены.":::

## <a name="next-steps"></a>Дальнейшие действия

В этой статье описано, как настроить приложение-функцию в Azure для использования с цифровым двойников Azure.

Далее Узнайте, как создать базовую функцию для приема данных центра Интернета вещей в Azure Digital двойников:
* [*Пошаговое руководство. прием данных телеметрии из центра Интернета вещей*](how-to-ingest-iot-hub-data.md)
