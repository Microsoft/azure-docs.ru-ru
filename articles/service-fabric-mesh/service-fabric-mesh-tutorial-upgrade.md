---
title: Руководство по обновлению приложения сетки Service Fabric Azure
description: Это руководство является четвертой частью цикла, в котором показано, как обновить приложение Сетки Azure Service Fabric непосредственно из Visual Studio.
author: georgewallace
ms.topic: conceptual
ms.date: 11/29/2018
ms.author: gwallace
ms.custom: mvc, devcenter, devx-track-csharp
ms.openlocfilehash: 1020613eb43177ba159601f253848f8d03f385a8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99625438"
---
# <a name="tutorial-learn-how-to-upgrade-a-service-fabric-application-using-visual-studio"></a>Руководство. Узнайте, как обновить приложение Service Fabric с помощью Visual Studio

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Это руководство является четвертой частью цикла, в котором показано, как обновить приложение Сетки Azure Service Fabric непосредственно из Visual Studio. Обновления затронут как код, так и конфигурацию. Вы увидите, что действия по обновлению и публикации в Visual Studio одинаковы.

Из этого руководства вы узнаете, как выполнить следующие задачи:
> [!div class="checklist"]
> * Обновление службы сетки Service Fabric с помощью Visual Studio

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * [Создание приложения Сетки Service Fabric в Visual Studio](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [Отладка приложения Сетки Service Fabric, выполняющегося в локальном кластере разработки](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [Развертывание приложения Сетки Service Fabric](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * Обновление приложения Сетки Service Fabric
> * [Удаление ресурсов Сетки Service Fabric](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если вы еще не развернули приложение списка дел, следуйте инструкциям в статье [Руководство. Развертывание приложения Сетки Service Fabric](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md).

## <a name="upgrade-a-service-fabric-mesh-service-by-using-visual-studio"></a>Обновление службы сетки Service Fabric с помощью Visual Studio

В этой статье показано, как обновить микрослужбу в приложении. В этом примере мы изменим службу, `WebFrontEnd` чтобы она отображала категорию задач, и увеличит объем ЦП, который он задает. Затем мы выполним обновление развернутой службы.

## <a name="modify-the-config"></a>Изменение конфигурации

При создании Service Fabric приложения для сетки Visual Studio добавляет файл **Parameters. YAML** для каждой среды развертывания (облачной и локальной). В этих файлах можно определить параметры и их значения, на которые затем можно будет ссылаться из файлов сетки *. YAML, таких как Service. YAML или Network. YAML.  Visual Studio предоставляет несколько переменных, например, сколько ресурсов ЦП может использовать служба.

Мы будем обновлять `WebFrontEnd_cpu` параметр, чтобы обновить ресурсы ЦП до `1.5` , в результате чего служба WebService  будет более интенсивно использоваться.

1. В проекте **тодолистапп** в разделе **среды**  >  **облака** откройте файл **Parameters. YAML** . Измените `WebFrontEnd_cpu` значение, равное `1.5` . Перед именем параметра рекомендуется использовать имя службы, `WebFrontEnd_` чтобы отличить его от параметров с тем же именем, которые применяются к разным службам.

    ```xml
    WebFrontEnd_cpu: 1.5
    ```

2. Откройте файл **Service. YAML проекта WebService** **в разделе**   >  **ресурсы службы WebService**.

    Обратите внимание, что в `resources:` разделе in `cpu:` задано значение `"[parameters('WebFrontEnd_cpu')]"` . Если проект строится для облака, значение для `'WebFrontEnd_cpu` будет взято из файла Environments   >  **облака**  >  **Parameters. YAML** и будет `1.5` . Если проект строится для запуска локально, значение будет взято из   >  файла **Local**  >  **Parameters. YAML** окружений и будет равно "0,5".

> [!Tip]
> По умолчанию файл параметров, являющийся одноранговым узлом файла Profile. YAML, будет использоваться для предоставления значений для этого файла Profile. YAML.
> Например, среды > облачные > параметры. YAML предоставляет значения параметров для сред > Cloud > Profile. YAML.
>
> Это можно переопределить, добавив следующий элемент в файл Profile. YAML: например `parametersFilePath=”relative or full path to the parameters file”` , или. `parametersFilePath=”C:\MeshParms\CustomParameters.yaml”``parametersFilePath=”..\CommonParameters.yaml”`

## <a name="modify-the-model"></a>Изменение модели

Чтобы внести изменения кода, добавьте свойство `Category` в класс `ToDoItem` в файле `ToDoItem.cs`.

```csharp
public class ToDoItem
{
    public string Category { get; set; }
    ...
}
```

Затем обновите метод `Load()` в том же файле, чтобы присвоить категорию строке по умолчанию:

```csharp
public static ToDoItem Load(string description, int index, bool completed)
{
    ToDoItem newItem = new ToDoItem(description)
    {
        Index = index,
        Completed = completed, 
        Category = "none"    // <-- add this line
    };

    return newItem;
}
```

## <a name="modify-the-service"></a>Изменение службы

Проект `WebFrontEnd` — это приложение ASP.NET Core с веб-страницей, которая отображает список дел. В проекте `WebFrontEnd` откройте `Index.cshtml` и добавьте две строки, указанные ниже, чтобы отобразить категорию задачи:

```HTML
<div>
    <table class="table-bordered">
        <thead>
            <tr>
                <th>Description</th>
                <th>Category</th>           @*add this line*@
                <th>Done?</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model.Items)
            {
                <tr>
                    <td>@item.Description</td>
                    <td>@item.Category</td>           @*add this line*@
                    <td>@item.Completed</td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

Выполните сборку и запуск приложения, чтобы убедиться, что новый столбец категории отображается на веб-странице, в которой перечислены задачи.

## <a name="upgrade-the-app-from-visual-studio"></a>Обновление приложения из Visual Studio

Как при обновлении кода, так и при обновлении конфигурации (в данном случае мы делаем оба варианта) обновите приложение Service Fabricной сети в Azure, щелкнув правой кнопкой мыши **тодолистапп** в Visual Studio и выбрав **Опубликовать...**

Дальше появится диалоговое окно **Публикация приложения Service Fabric**.

Используйте раскрывающийся список **целевой профиль** , чтобы выбрать файл Profile. YAML, который будет использоваться для этого развертывания. Мы обновляем приложение в облаке, чтобы мы выбрали **Cloud. YAML** в раскрывающемся списке, который будет использовать `WebFrontEnd_cpu` значение 1,0, определенное в этом файле.

![Диалоговое окно публикации службы "Сетка Service Fabric" в Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-dialog.png)

Выберите учетную запись Azure и подписку. Установите **Расположение** в расположение, которое использовалось при первоначальной публикации этого приложения в Azure. В этой статье используется **восточная часть США**.

Укажите **группу ресурсов,** которая использовалась при первоначальной публикации этого приложения в Azure.

Укажите в **реестре контейнеров** Azure имя реестра контейнеров Azure, которое вы создали при первоначальной публикации этого приложения в Azure.

В диалоговом окне публикации нажмите кнопку **Опубликовать**, чтобы обновить приложение списка дел в Azure.

Отслеживайте ход обновления, выбрав область **средства Service Fabric** в окне **вывода** Visual Studio. 

После того как образ создан и отправлен в реестр контейнеров Azure, в выходных данных появится ссылка **для состояния** , которую можно щелкнуть для мониторинга развертывания в портал Azure.

После завершения обновления выходные данные в области **Средства Service Fabric** будут отображать IP-адрес и порт приложения в виде URL-адреса.

```json
The application was deployed successfully and it can be accessed at http://10.000.38.000:20000.
```

Откройте веб-браузер и перейдите к URL-адресу, чтобы увидеть работу сайта в Azure. Теперь вы видите веб-страницу, содержащую столбец категории.

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства было показано следующее.
> [!div class="checklist"]
> * Как обновить приложение Сетки Service Fabric с помощью Visual Studio

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Удаление ресурсов Сетки Service Fabric](service-fabric-mesh-tutorial-cleanup-resources.md)