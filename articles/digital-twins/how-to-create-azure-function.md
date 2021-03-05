---
title: Настройка функции в Azure для обработки данных
titleSuffix: Azure Digital Twins
description: Узнайте, как создать функцию в Azure, которая может получать доступ и вызываться цифровым двойников.
author: baanders
ms.author: baanders
ms.date: 8/27/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 8ed4e550ea441d5d99a3debb6bf37eb7db2a4a20
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102180179"
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

Чтобы использовать пакет SDK, необходимо включить в проект следующие пакеты. Можно либо установить пакеты с помощью диспетчера пакетов NuGet Visual Studio, либо добавить пакеты с помощью `dotnet` в программе командной строки.

* [Azure. Дигиталтвинс. Core](https://www.nuget.org/packages/Azure.DigitalTwins.Core/)
* [Azure. Identity](https://www.nuget.org/packages/Azure.Identity/)
* [System.Net.Http](https://www.nuget.org/packages/System.Net.Http/)
* [Azure. Core](https://www.nuget.org/packages/Azure.Core/)

Затем в Visual Studio обозреватель решений откройте файл _function1.CS_ , в котором имеется образец кода, и добавьте следующие `using` инструкции для этих пакетов в функцию. 

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

# <a name="cli"></a>[CLI](#tab/cli)

Эти команды можно выполнить в [Azure Cloud Shell](https://shell.azure.com) или [локальной Azure CLI установки](/cli/azure/install-azure-cli).

### <a name="assign-access-role"></a>Назначение роли доступа

В схеме функций из предыдущих примеров требуется передать в него токен носителя, чтобы иметь возможность проходить проверку подлинности в Azure Digital двойников. Чтобы убедиться, что этот токен носителя передан, необходимо настроить разрешения [управляемое удостоверение службы (MSI)](../active-directory/managed-identities-azure-resources/overview.md) для приложения-функции, чтобы получить доступ к Azure Digital двойников. Это необходимо сделать только один раз для каждого приложения функции.

Вы можете использовать управляемое системой удостоверение приложения-функции, чтобы предоставить ему роль _**владельца данных Digital двойников Azure**_ для своего экземпляра Azure Digital двойников. Это предоставит приложению функции в экземпляре разрешение на выполнение действий плоскости данных. Затем сделайте URL-адрес экземпляра Azure Digital двойников доступным для вашей функции, задав переменную среды.

1. Используйте следующую команду, чтобы просмотреть сведения об управляемом системой удостоверении для функции. Запишите значение поля _principalId_ в выходных данных команды.

    ```azurecli-interactive 
    az functionapp identity show -g <your-resource-group> -n <your-App-Service-(function-app)-name> 
    ```

    >[!NOTE]
    > Если результат пуст, вместо отображения сведений об удостоверении создайте новое управляемое системой удостоверение для функции с помощью следующей команды:
    > 
    >```azurecli-interactive    
    >az functionapp identity assign -g <your-resource-group> -n <your-App-Service-(function-app)-name>  
    >```
    >
    > В выходных данных отобразятся сведения об удостоверении, включая значение _principalId_ , необходимое для следующего шага. 

1. Используйте значение _principalId_ в следующей команде, чтобы назначить удостоверение приложения-функции роли _владельца Azure Digital Twins_ для вашего экземпляра Azure Digital Twins.

    ```azurecli-interactive 
    az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<principal-ID>" --role "Azure Digital Twins Data Owner"
    ```

### <a name="configure-application-settings"></a>Настройка параметров приложения

Наконец, предоставьте доступ к URL-адресу вашего экземпляра Azure Digital двойников вашей функции, задав для него **переменную среды** . Дополнительные сведения о переменных среды см. в разделе [*Управление приложением функции*](../azure-functions/functions-how-to-use-azure-function-app-settings.md?tabs=portal). 

> [!TIP]
> URL-адрес экземпляра Azure Digital двойников создается путем добавления *https://* в начало *имени узла* цифрового двойников Azure. Чтобы просмотреть имя узла, а также все свойства экземпляра, можно запустить `az dt show --dt-name <your-Azure-Digital-Twins-instance>` .

```azurecli-interactive 
az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "ADT_SERVICE_URL=https://<your-Azure-Digital-Twins-instance-host-name>"
```

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Выполните следующие действия в [портал Azure](https://portal.azure.com/).

### <a name="assign-access-role"></a>Назначение роли доступа

Управляемое удостоверение, назначенное системой, позволяет ресурсам Azure проходить проверку подлинности в облачных службах (например, Azure Key Vault) без сохранения учетных данных в коде. После включения все необходимые разрешения можно будет предоставить через Управление доступом на основе ролей в Azure. Жизненный цикл этого типа управляемого удостоверения связан с жизненным циклом этого ресурса. Кроме того, каждый ресурс может иметь только одно назначенное системой управляемое удостоверение.

1. В [портал Azure](https://portal.azure.com/)найдите приложение функции, введя его имя в строку поиска. Выберите приложение из результатов. 

    :::image type="content" source="media/how-to-create-azure-function/portal-search-for-function-app.png" alt-text="Снимок экрана портал Azure: имя приложения-функции ищется на панели поиска портала, а результат поиска выделяется.":::

1. На странице приложение-функция на панели навигации слева выберите _удостоверение_ , чтобы работать с управляемым удостоверением для функции. На странице _назначенные системы_ убедитесь, что для параметра _состояние_ установлено значение **вкл** . (если это не так, установите его сейчас и *Сохраните* изменения).

    :::image type="content" source="media/how-to-create-azure-function/verify-system-managed-identity.png" alt-text="Снимок экрана портал Azure: на странице удостоверение для приложения-функции параметр status имеет значение ON." lightbox="media/how-to-create-azure-function/verify-system-managed-identity.png":::

1. Нажмите кнопку _назначения ролей Azure_ , после чего откроется страница *назначения ролей Azure* .

    :::image type="content" source="media/how-to-create-azure-function/add-role-assignment-1.png" alt-text="Снимок экрана портал Azure. в разделе &quot;разрешения&quot; на странице удостоверений функции Azure выделит кнопку назначения ролей Azure." lightbox="media/how-to-create-azure-function/add-role-assignment-1.png":::

    Выберите _+ добавить назначение ролей (Предварительная версия)_.

    :::image type="content" source="media/how-to-create-azure-function/add-role-assignment-2.png" alt-text="Снимок экрана портал Azure: выделение + добавить назначение ролей (Предварительная версия) на странице назначения ролей Azure." lightbox="media/how-to-create-azure-function/add-role-assignment-2.png":::

1. На открывшейся странице _Добавление назначения ролей (Предварительная версия)_ выберите следующие значения:

    * **Область** — группа ресурсов.
    * **Подписка**: выберите подписку Azure.
    * **Группа ресурсов**. Выберите группу ресурсов из раскрывающегося списка.
    * **Роль**: выберите _Azure Digital двойников Data Owner_ в раскрывающемся списке.

    Затем сохраните сведения, нажав кнопку Save ( _сохранить_ ).

    :::image type="content" source="media/how-to-create-azure-function/add-role-assignment-3.png" alt-text="Снимок экрана портал Azure: диалоговое окно для добавления нового назначения роли (Предварительная версия). Существуют поля для области, подписки, группы ресурсов и роли.":::

### <a name="configure-application-settings"></a>Настройка параметров приложения

Чтобы сделать URL-адрес своего экземпляра Digital двойников доступным для вашей функции, можно задать для него **переменную среды** . Дополнительные сведения о переменных среды см. в разделе [*Управление приложением функции*](../azure-functions/functions-how-to-use-azure-function-app-settings.md?tabs=portal). Параметры приложения предоставляются как переменные среды для доступа к экземпляру Digital двойников Azure. 

Чтобы задать переменную среды с URL-адресом своего экземпляра, сначала получите URL-адрес, находя имя узла своего экземпляра Azure Digital двойников. Найдите свой экземпляр на панели поиска [портал Azure](https://portal.azure.com) . Затем на левой панели навигации выберите _Обзор_ , чтобы просмотреть _имя узла_. Скопируйте это значение.

:::image type="content" source="media/how-to-create-azure-function/instance-host-name.png" alt-text="Снимок экрана портал Azure: на странице &quot;Обзор&quot; для экземпляра Azure Digital двойников выделяется значение имени узла.":::

Теперь можно создать параметр приложения, выполнив следующие действия.

1. Найдите приложение функции на панели поиска портала и выберите его из результатов.

    :::image type="content" source="media/how-to-create-azure-function/portal-search-for-function-app.png" alt-text="Снимок экрана портал Azure: имя приложения-функции ищется на панели поиска портала, а результат поиска выделяется.":::

1. На панели навигации слева выберите _Конфигурация_ . На вкладке _Параметры приложения_ выберите _+ создать параметр приложения_.

    :::image type="content" source="media/how-to-create-azure-function/application-setting.png" alt-text="Снимок экрана портал Azure. на странице конфигурации для приложения-функции будет выделена кнопка для создания нового параметра приложения.":::

1. В открывшемся окне используйте скопированное выше значение имя узла, чтобы создать параметр приложения.
    * **Имя**: ADT_SERVICE_URL
    * **Значение**: https://{ваш-Azure-Digital-двойников-Host-Name}
    
    Нажмите кнопку _ОК_ , чтобы создать параметр приложения.
    
    :::image type="content" source="media/how-to-create-azure-function/add-application-setting.png" alt-text="Снимок экрана портал Azure. Кнопка &quot;ОК&quot; выделяется после заполнения полей &quot;имя&quot; и &quot;значение&quot; на странице &quot;Добавление и изменение параметров приложения&quot;.":::

1. После создания параметра вы увидите, что он отображается на вкладке _Параметры приложения_ . Убедитесь, *ADT_SERVICE_URL* появится в списке, а затем сохраните новый параметр приложения, нажав кнопку _сохранить_ .

    :::image type="content" source="media/how-to-create-azure-function/application-setting-save-details.png" alt-text="Снимок экрана портал Azure: страница &quot;Параметры приложения&quot; с выделенным параметром &quot;новый ADT_SERVICE_URL&quot;. Кнопка сохранить также выделяется.":::

1. Чтобы изменения параметров приложения вступили в силу, необходимо перезапустить приложение, поэтому нажмите кнопку _продолжить_ , чтобы перезапустить приложение при появлении соответствующего запроса.

    :::image type="content" source="media/how-to-create-azure-function/save-application-setting.png" alt-text="Снимок экрана портал Azure. Обратите внимание, что все изменения параметров приложения с перезапуском приложения. Кнопка продолжить выделена.":::

---

## <a name="next-steps"></a>Дальнейшие действия

В этой статье описано, как настроить приложение-функцию в Azure для использования с цифровым двойников Azure.

Далее Узнайте, как создать базовую функцию для приема данных центра Интернета вещей в Azure Digital двойников:
* [*Пошаговое руководство. прием данных телеметрии из центра Интернета вещей*](how-to-ingest-iot-hub-data.md)
