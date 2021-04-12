---
title: Создание и развертывание веб-приложения на базе нескольких служб в Сетке Service Fabric
description: В рамках этого руководства вы создадите приложение Сетки Azure Service Fabric на базе нескольких служб, состоящее из веб-сайта ASP.NET Core, который взаимодействует с серверной веб-службой, выполните отладку приложения локально и опубликуете его в Azure.
author: georgewallace
ms.topic: tutorial
ms.date: 09/18/2018
ms.author: gwallace
ms.custom: mvc, devcenter, devx-track-csharp
ms.openlocfilehash: b0bdb3c09aead812e1c16f4d0d17aae58e141809
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99626798"
---
# <a name="tutorial-create-debug-deploy-and-upgrade-a-multi-service-service-fabric-mesh-app"></a>Руководство по Создание, отладка, развертывание и обновление приложения на базе нескольких служб в Сетке Service Fabric

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Это руководство представляет первую часть цикла. Здесь описывается, как с помощью Visual Studio создать приложение Сетки Azure Service Fabric с веб-интерфейсом ASP.NET и серверной службой веб-API ASP.NET Core. Затем вы выполните отладку приложения в локальном кластере разработки. Вы опубликуете приложение в Azure, а затем внесете изменения в конфигурацию и код и обновите приложение. Наконец, вы удалите неиспользуемые ресурсы Azure, чтобы за них не взималась плата.

Закончив работу с этим руководством, вы изучите большинство этапов управления жизненным циклом приложения и создадите приложение, демонстрирующее вызовы между службами в приложении Сетки Service Fabric.

Если вы не хотите вручную создавать приложение со списком задач, вы можете [скачать исходный код](https://github.com/azure-samples/service-fabric-mesh) для завершенного приложения и сразу перейти к [отладке приложения локально](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md).

В первой части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание приложения Сетки Service Fabric, состоящего из веб-интерфейса ASP.NET, с помощью Visual Studio.
> * Создание модели для представления задач.
> * Создание серверной службы и извлечение из нее данных.
> * Добавление контроллера и DataContext как части шаблона контроллера представления модели для серверной службы.
> * Создание веб-страницы для отображения задач.
> * Создание переменных среды, которые идентифицируют серверную службу.

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * Создание приложения Сетки Service Fabric в Visual Studio
> * [Отладка приложения Сетки Service Fabric, выполняющегося в локальном кластере разработки](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [Развертывание приложения Сетки Service Fabric](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * [Обновление приложения Сетки Service Fabric](service-fabric-mesh-tutorial-upgrade.md)
> * [Удаление ресурсов Сетки Service Fabric](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет Azure подписки до начала работы, можно [создать бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Убедитесь, что имеется [настроенная среда разработки](service-fabric-mesh-howto-setup-developer-environment-sdk.md), которая включает установку среды выполнения Service Fabric, пакет SDK, Docker и Visual Studio 2017.

## <a name="create-a-service-fabric-mesh-project-in-visual-studio"></a>Создание проекта Сетки Service Fabric в Visual Studio

Запустите Visual Studio и выберите **Файл** > **Создать** > **Проект...** .

В диалоговом окне **Новый проект** вверху в поле **Поиск** введите `mesh`. Выберите шаблон **Service Fabric Mesh Application** (Приложение службы "Сетка Service Fabric"). Если шаблон не отображается, убедитесь, что вы установили пакет SDK для Сетки Service Fabric и предварительную версию средств VS, как описано в статье о [настройке среды разработки](service-fabric-mesh-howto-setup-developer-environment-sdk.md).  

В поле **Имя** введите `todolistapp`, а в поле **Расположение** укажите путь к папке, где вы хотите хранить файлы для проекта.

Установите флажок **Create directory for solution** (Создать каталог для решения) и нажмите кнопку **ОК**, чтобы создать проект службы "Сетка Service Fabric".

![Снимок экрана: создание проекта Сетки Service Fabric.](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-project.png)

Затем откроется диалоговое окно **Служба структуры для новой службы**.

### <a name="create-the-web-front-end-service"></a>Создание службы веб-интерфейса

В диалоговом окне **Служба структуры для новой службы** выберите тип проекта **ASP.NET Core**. В поле **Container OS** (ОС контейнера) выберите **Windows**.

В поле **Имя службы** укажите **WebFrontEnd**. Нажмите кнопку **ОК**, чтобы создать службу ASP.NET Core.

![Диалоговое окно нового проекта службы "Сетка Service Fabric" в Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-service-fabric-service.png)

После этого откроется диалоговое окно Веб-приложение ASP.NET Core. Выберите **Веб-приложение** и нажмите кнопку **ОК**.

![Снимок экрана: выделенный шаблон веб-приложения.](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-aspnetcore-app.png)

Теперь у вас есть приложение Сетки Service Fabric. Создайте модель для сведений о задачах.

## <a name="create-the-to-do-items-model"></a>Создание модели задач

Для простоты задачи хранятся в списке в памяти. Создайте библиотеку классов для задач и список для их хранения. В Visual Studio, где в настоящее время загружено решение приложения **todolistapp**, выберите **Файл** > **Добавить** > **Новый проект**.

В диалоговом окне **Добавление нового проекта** вверху в поле **Поиск** введите `C# .net core class`. Выберите шаблон **Class Library (.NET Core)** (Библиотека классов (.NET Core)).

В поле **Имя файла** введите `Model`. Нажмите кнопку **ОК**, чтобы создать библиотеку классов.

В обозревателе решений в разделе **Модель** щелкните правой кнопкой мыши **Class1.cs** и выберите **Переименовать**. Переименуйте класс на **ToDoItem.cs**. Если появится запрос на переименование всех ссылок, выберите **Да**.

Замените содержимое пустой строки `class ToDoItem` на:

```csharp
public class ToDoItem
{
    public string Description { get; set; }
    public int Index { get; set; }
    public bool Completed { get; set; }

    public ToDoItem(string description)
    {
        Description = description;
        Index = 0;
    }

    public static ToDoItem Load(string description, int index, bool completed)
    {
        ToDoItem newItem = new ToDoItem(description)
        {
            Index = index,
            Completed = completed
        };

        return newItem;
    }
}
```

Этот класс представляет задачи.

В Visual Studio щелкните правой кнопкой мыши библиотеку классов **Модель** и выберите **Добавить** > **Класс...**, чтобы создать список для хранения задач. Откроется диалоговое окно **Добавление нового элемента**. В поле **Имя** укажите `ToDoList.cs` и нажмите кнопку **Добавить**.

В **ToDoList.cs** замените пустую строку `class ToDoList` на:

```csharp
public class ToDoList
{
    private List<ToDoItem> _items;

    public string Name { get; set; }
    public IEnumerable<ToDoItem> Items { get => _items; }

    public ToDoList(string name)
    {
        Name = name;
        _items = new List<ToDoItem>();
    }

    public ToDoItem Add(string description)
    {
        var item = new ToDoItem(description);
        _items.Add(item);
        item.Index = _items.IndexOf(item);
        return item;
    }
    public void Add(ToDoItem item)
    {
        _items.Add(item);
        item.Index = _items.Count - 1;
    }

    public ToDoItem RemoveAt(int index)
    {
        if (index >= 0 && index < _items.Count)
        {
            var result = _items[index];
            _items.RemoveAt(index);

            // Reorder items
            for (int i = index; i < _items.Count; i++)
            {
                _items[i].Index = i;
            }

            return result;
        }
        else
        {
            throw new IndexOutOfRangeException();
        }
    }
}
```

Затем создайте службу Service Fabric, которая будет отслеживать задачи.

## <a name="create-the-back-end-service"></a>Создание серверной службы

В Visual Studio в окне **Обозреватель решений** щелкните правой кнопкой мыши **todolistapp** и выберите **Добавить** > **Служба структуры для новой службы...**

Откроется диалоговое окно **Служба структуры для новой службы**. Выберите тип проекта **ASP.NET Core**, а в поле **Container OS** (ОС контейнера) выберите **Windows**. В поле **Имя службы** укажите **ToDoService**. Щелкните **ОК**, чтобы создать службу ASP.NET Core.

Откроется диалоговое окно **Создать веб-приложение ASP.NET Core**. В этом диалоговом окне выберите **API**, а затем щелкните **ОК**. Проект для службы будет добавлен в решение.

![Новое приложение ASP.NET Core в Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-webapi.png)

Так как серверная служба не предоставляет никакого пользовательского интерфейса, отключите запуск браузера при запуске службы. В **обозревателе решений** щелкните правой кнопкой мыши **ToDoService** и выберите **Свойства**. В появившемся окне свойств выберите слева вкладку **Отладка** и снимите флажок **Запуск браузера**. Нажмите клавиши **CTRL+S**, чтобы сохранить изменение.

Так как эта служба хранит сведения о задачах, добавьте ссылку на библиотеку классов модели. В обозревателе решений щелкните правой кнопкой мыши **ToDoService**, а затем выберите **Добавить** > **Ссылка...** Откроется диалоговое окно **Диспетчер ссылок**.

В **диспетчере ссылок** установите флажок **Модель** и нажмите кнопку **ОК**.

### <a name="add-a-data-context&quot;></a>Добавление контекста данных

Затем создайте контекст данных, который координирует работу с данными из модели данных.

Чтобы добавить класс контекста данных, в обозревателе решений щелкните правой кнопкой мыши **ToDoService**, а затем **Добавить** > **Класс**.
В открывшимся диалоговом окне **Добавление нового элемента** выберите **Класс**, а в поле **Имя** укажите `DataContext.cs` и нажмите кнопку **Добавить**.

В **DataContext.cs** замените содержимое пустой строки `class DataContext` на:

```csharp
public static class DataContext
{
    public static Model.ToDoList ToDoList { get; } = new Model.ToDoList(&quot;Azure learning List");

    static DataContext()
    {
        // Seed to-do list
        ToDoList.Add(Model.ToDoItem.Load("Learn about microservices", 0, true));
        ToDoList.Add(Model.ToDoItem.Load("Learn about Service Fabric", 1, true));
        ToDoList.Add(Model.ToDoItem.Load("Learn about Service Fabric Mesh", 2, false));
    }
}
```

Этот минимальный контекст данных заполняет некоторые примеры задач и предоставляет к ним доступ.

### <a name="add-a-controller"></a>Добавление контроллера

Контроллер по умолчанию, который обрабатывает запросы HTPP и создает ответ HTTP, был предоставлен шаблоном при создании проекта **ToDoService**. В **обозревателе решений** в разделе **ToDoService** откройте папку **Контроллеры**, чтобы увидеть файл **ValuesController.cs**. 

Щелкните правой кнопкой мыши файл **ValuesController.cs**, а затем — **Переименовать**. Переименуйте файл на `ToDoController.cs`. Если появится запрос на переименование всех ссылок, нажмите кнопку **Да**.

Откройте файл **ToDoController.cs** и замените содержимое `class ToDoController` на:

```csharp
[Route("api/[controller]")]
public class ToDoController : Controller
{
    // GET api/todo
    [HttpGet]
    public IEnumerable<Model.ToDoItem> Get()
    {
        return DataContext.ToDoList.Items;
    }

    // GET api/todo/5
    [HttpGet("{index}")]
    public Model.ToDoItem Get(int index)
    {
        return DataContext.ToDoList.Items.ElementAt(index);
    }

    //// POST api/values
    //[HttpPost]
    //public void Post([FromBody]string value)
    //{
    //}

    //// PUT api/values/5
    //[HttpPut("{id}")]
    //public void Put(int id, [FromBody]string value)
    //{
    //}

    // DELETE api/values/5
    [HttpDelete("{index}")]
    public void Delete(int index)
    {
    }
}
```

Чтобы сосредоточить все внимание на связи с другой службой, в этом руководстве не описывается добавление, удаление и т. д.

## <a name="create-the-web-page-that-displays-to-do-items"></a>Создание веб-страницы, отображающей задачи

Внедрив серверную службу, создайте веб-сайт, на котором будут отображаться представляемые им задачи. Следующие шаги выполняются в рамках проекта **WebFrontEnd**.

Веб-странице, на которой отображается список задач, нужен доступ к классу и списку **ToDoItem**.
В **обозревателе решений** добавьте ссылку на проект модели, щелкнув правой кнопкой мыши **WebFrontEnd** и выбрав **Добавить** > **Ссылка...** Откроется диалоговое окно **Диспетчер ссылок**.

В **диспетчере ссылок** установите флажок **Модель** и нажмите кнопку **ОК**.

В **обозревателе решений** откройте страницу Index.cshtml, выбрав **WebFrontEnd** > **Страницы** > **Index.cshtml**. Откройте **Index.cshtml**.

Замените содержимое всего файла на следующий HTML, который определяет простую таблицу для отображения задач:

```HTML
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<div>
    <table class="table-bordered">
        <thead>
            <tr>
                <th>Description</th>
                <th>Done?</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model.Items)
            {
                <tr>
                    <td>@item.Description</td>
                    <td>@item.Completed</td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

Щелкните значок раскрывающегося списка файла **Index.cshtml** в **Обозревателе решений**, а затем откройте **Index.cshtml.cs**.

В верхней части файла **Index.cshtml.cs** добавьте `using System.Net.Http;`.

Замените содержимое `public class IndexModel` на:

```csharp
public class IndexModel : PageModel
{
    public Model.ToDoItem[] Items = new Model.ToDoItem[] { };

    public void OnGet()
    {
        HttpClient client = new HttpClient();

        using (HttpResponseMessage response = client.GetAsync(backendUrl).GetAwaiter().GetResult())
        {
            if (response.StatusCode == System.Net.HttpStatusCode.OK)
            {
                Items = Newtonsoft.Json.JsonConvert.DeserializeObject<Model.ToDoItem[]>(response.Content.ReadAsStringAsync().Result);
            }
        }
    }

    private static string backendDNSName = $"{Environment.GetEnvironmentVariable("ToDoServiceName")}";
    private static Uri backendUrl = new Uri($"http://{backendDNSName}:{Environment.GetEnvironmentVariable("ApiHostPort")}/api/todo");
}
```

### <a name="create-environment-variables"></a>Создание переменных среды

Для взаимодействия с серверной службой требуется ее URL-адрес. В целях этого руководства следующий фрагмент кода (который определен выше как часть IndexModel) считывает переменные среды для составления URL-адреса:

```csharp
private static string backendDNSName = $"{Environment.GetEnvironmentVariable("ToDoServiceName")}";
private static Uri backendUrl = new Uri($"http://{backendDNSName}:{Environment.GetEnvironmentVariable("ApiHostPort")}/api/todo");
```

URL-адрес состоит из имени службы и порта. Вся эта информация содержится в файле service.yaml в проекте **ToDoService**.

> [!IMPORTANT]
> В следующих шагах файлы YAML будут изменены.
> Для отступов переменных в файле service.yaml должны использоваться пробелы, а не символы табуляции. В противном случае они не будут компилироваться. Служба Visual Studio может вставлять символы табуляции при создании переменных среды. Замените все символы табуляции пробелами. Несмотря на то, что будут отображаться ошибки в выходных данных отладки **построения**, приложение по-прежнему будет запускаться, но при условиях, что вы преобразуете вкладки в пробелы и перестроите их. Чтобы гарантировать отсутствие символов табуляции в файле service.yaml, вы можете сделать пробелы видимыми в редакторе Visual Studio, выбрав **Правка**  > **Дополнительно**  > **Показать пустое пространство**.
> Обратите внимание, что файлы service.yaml обрабатываются с использованием английского языкового стандарта. Например, если требуется использовать десятичный разделитель, используйте точку, а не запятую.

Перейдите в **обозревателе решений** к проекту **ToDoService** и откройте **Service Resources** (Ресурсы службы)  > **service.yaml**.

![Рис. 1. Файл service.yaml в проекте ToDoService](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-serviceyaml-port.png)

 Имя службы `ToDoService` находится под строкой `services:` (см. (1) на рисунке выше).

* Порт `80` находится в разделе `endpoints:` (см. (2) на рисунке выше). Номер порта вашего проекта может быть другим.

Далее необходимо определить переменные среды, представляющие имя службы и номер порта в проекте WebFrontEnd, чтобы он мог вызвать серверную службу.

В **обозревателе решений** выберите **WebFrontEnd** > **Service Resources** (Ресурсы службы)  > **service.yaml**, чтобы определить переменные, указывающие адрес серверной службы.

В файле service.yaml добавьте следующие переменные в `environmentVariables:` (сначала нужно удалить `#`, чтобы раскомментировать `environmentVariables:`). Интервал важен, поэтому выровняйте переменные, которые вы добавляете, с другими переменными в `environmentVariables:`. Очень важно, чтобы значение для ApiHostPort соответствовало значению порта для ToDoServiceListener, которое ранее было замечено в файле service.yaml ToDoService.

```yaml
- name: ApiHostPort
  value: 
- name: ToDoServiceName
  value: ToDoService
```

> [!Tip]
> Задать значение для `ToDoServiceName` можно двумя способами: 
> - Имя службы, которое будет разрешаться как в сценариях отладки в Windows 10, так и при развертывании службы Сетки Azure Service Fabric.
> - Полное имя servicename.appname. Оно будет действительно только для отладки в Windows 10.
> Рекомендуется использовать имя службы только для ее разрешения.

Ваш файл **service.yaml** в проекте **WebFrontEnd** должен выглядеть примерно так (хотя значение `ApiHostPort`, вероятно, будет другим):

![Файл service.yaml в проекте WebFrontEnd](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-serviceyaml-envvars.png)


Теперь вы готовы к созданию и развертыванию образа приложения Сетки Service Fabric вместе с серверной веб-службой в локальном кластере.

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * Создание приложения Сетки Service Fabric, состоящего из веб-интерфейса ASP.NET.
> * Создание модели для представления задач.
> * Создание серверной службы и извлечение из нее данных.
> * Добавление контроллера и DataContext как части шаблона контроллера представления модели для серверной службы.
> * Создание веб-страницы для отображения задач.
> * Создание переменных среды, которые идентифицируют серверную службу.

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Отладка приложения Сетки Service Fabric, выполняющегося в локальном кластере разработки](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)