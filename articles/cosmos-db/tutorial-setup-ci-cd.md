---
title: Настройка конвейера CI/CD с помощью задачи сборки Azure Cosmos DB Emulator
description: Руководство по настройке рабочего процесса сборки и выпуска в Azure DevOps с использованием задачи сборки эмулятора Cosmos DB.
author: deborahc
ms.service: cosmos-db
ms.topic: how-to
ms.date: 01/28/2020
ms.author: dech
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: c7246511a88e2d2756a8ef56c5adf51ddbfd3e58
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102560537"
---
# <a name="set-up-a-cicd-pipeline-with-the-azure-cosmos-db-emulator-build-task-in-azure-devops"></a>Настройка конвейера CI/CD с помощью задачи сборки Azure Cosmos DB Emulator в Azure DevOps
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Эмулятор Azure Cosmos DB предоставляет локальную среду для разработки, которая эмулирует службу Azure Cosmos DB. Этот эмулятор позволяет разрабатывать и тестировать приложения локально без создания подписки Azure и каких-либо затрат. 

Задача сборки эмулятора Azure Cosmos DB для Azure DevOps позволяет выполнять те же действия в среде CI. Задачу сборки можно использовать для выполнения в эмуляторе тестов в рамках рабочих процессов сборки и выпуска. Эта задача запускает контейнер Docker с уже работающим эмулятором и предоставляет конечную точку, которую можно использовать для других частей определения сборки. Вы можете создать и запустить любое число экземпляров эмулятора, каждый в отдельном контейнере. 

В этой статье показано, как настроить в Azure DevOps конвейер непрерывной интеграции для приложения ASP.NET, которое использует задачу сборки эмулятора Cosmos DB для выполнения тестов. Аналогичный подход можно использовать для настройки конвейера непрерывной интеграции для приложения Node.js или Python. 

## <a name="install-the-emulator-build-task"></a>Установка задачи сборки эмулятора

Чтобы использовать задачу сборки, нам нужно установить ее в организации Azure DevOps. Найдите расширение **Эмулятор Azure Cosmos DB** в [Marketplace](https://marketplace.visualstudio.com/items?itemName=azure-cosmosdb.emulator-public-preview) и щелкните **Получить бесплатно**.

:::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_1.png" alt-text="Поиск и установка задачи сборки эмулятора Azure Cosmos DB из Marketplace в Azure DevOps":::

Теперь выберите организацию, в которой нужно установить это расширение. 

> [!NOTE]
> Чтобы установить расширение в организации Azure DevOps, нужно быть владельцем учетной записи или администратором коллекции проектов. Если у вас нет нужных разрешений, но вы являетесь участником учетной записи, попробуйте запросить расширения. [Подробнее.](/azure/devops/marketplace/faq-extensions)

:::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_2.png" alt-text="Выбор организации Azure DevOps, в которой нужно установить расширение":::

## <a name="create-a-build-definition"></a>Создание определения сборки

Теперь, когда расширение установлено, войдите в организацию Azure DevOps и найдите проект на панели мониторинга проектов. Вы можете добавить [конвейер сборки](/azure/devops/pipelines/get-started-designer?preserve-view=true&tabs=new-nav) в проект или изменить имеющийся. Если у вас уже есть конвейер сборки, переходите к разделу, посвященному [добавлению задачи конвейера сборки эмулятора в определение сборки](#addEmulatorBuildTaskToBuildDefinition).

1. Чтобы создать определение сборки, откройте в Azure DevOps вкладку **Сборки**. Щелкните **+Создать.** \> **Новый конвейер сборки**

   :::image type="content" source="./media/tutorial-setup-ci-cd/CreateNewBuildDef_1.png" alt-text="Создание конвейера сборки":::

2. Выберите нужный **источник**, **командный проект**, **репозиторий** и **ветвь по умолчанию для сборок вручную и по расписанию**. Выбрав необходимые параметры, щелкните **Продолжить**.

   :::image type="content" source="./media/tutorial-setup-ci-cd/CreateNewBuildDef_2.png" alt-text="Выбор командного проекта, репозитория и ветви для конвейера сборки":::

3. И наконец, выберите нужный шаблон для конвейера сборки. В нашем примере для этого руководства выбран шаблон **ASP.NET**. Теперь у вас есть конвейер сборки, который можно настроить для использования задачи сборки Azure Cosmos DB Emulator. 

> [!NOTE]
> Если в предыдущей задаче в рамках непрерывной интеграции установка выполняется вручную, то у пула агентов, выбираемого для непрерывной интеграции, должен быть установлен Docker для Windows. Чтобы выбрать пул агентов, см. описание [агентов, размещенных корпорацией Майкрософт](/azure/devops/pipelines/agents/hosted?tabs=yaml) (мы рекомендуем начать с `Hosted VS2017`).

Эмулятор Azure Cosmos DB в настоящее время не поддерживает размещенный Пул агентов VS2019. Однако эмулятор уже поставляется с VS2019 и используется для запуска эмулятора с помощью следующих командлетов PowerShell. Если при использовании VS2019 возникли проблемы, обратитесь к команде [Azure DevOps](https://developercommunity.visualstudio.com/spaces/21/index.html) для получения справки.

```powershell
Import-Module "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules\Microsoft.Azure.CosmosDB.Emulator"
Start-CosmosDbEmulator
```

## <a name="add-the-task-to-a-build-pipeline"></a><a name="addEmulatorBuildTaskToBuildDefinition"></a>Добавление задачи в конвейер сборки

1. Прежде чем добавлять задачи в конвейер сборки, необходимо добавить задание агента. Перейдите в конвейер сборки, выберите **...** , а затем — **Добавить задание агента**.

1. Затем выберите **+** рядом с заданием агента, чтобы добавить задачу сборки эмулятора. Выполните поиск по слову **cosmos** в поле поиска, затем выберите **Azure Cosmos DB Emulator** (Эмулятор Azure Cosmos DB) и добавьте его в задание агента. Задача сборки запустит контейнер с уже выполняющимся экземпляром эмулятора Cosmos DB, поэтому задачи эмулятора должны выполняться перед выполнением других задач, для которых требуется эмулятор в состоянии выполнения.

   :::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_3.png" alt-text="Добавление задачи сборки эмулятора в определение сборки":::

В этом руководстве вы добавите задачи в начало, чтобы обеспечить доступность эмулятора перед выполнением наших тестов.

### <a name="add-the-task-using-yaml"></a>Добавление задачи с помощью YAML

Этот шаг является необязательным и необходим только при настройке конвейера CI/CD с помощью задачи YAML. В таких случаях можно определить задачу YAML, как показано в следующем коде:

```yml
- task: azure-cosmosdb.emulator-public-preview.run-cosmosdbemulatorcontainer.CosmosDbEmulator@2
  displayName: 'Run Azure Cosmos DB Emulator'

- script: yarn test
  displayName: 'Run API tests (Cosmos DB)'
  env:
    HOST: $(CosmosDbEmulator.Endpoint)
    # Hardcoded key for emulator, not a secret
    AUTH_KEY: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
    # The emulator uses a self-signed cert, disable TLS auth errors
    NODE_TLS_REJECT_UNAUTHORIZED: '0'
```

## <a name="configure-tests-to-use-the-emulator"></a>Настройка использования эмулятора в тестах

Теперь давайте настроим тесты, чтобы они использовали эмулятор. Задача сборки эмулятора экспортирует переменную среды с именем CosmosDbEmulator.Endpoint, значение которой может использовать любая задача, расположенная дальше в конвейере сборки. 

В этом руководстве описано, как использовать [задачу Visual Studio Test](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md) для запуска модульных тестов, которые настраиваются в файле **.runsettings**. См. дополнительные сведения о [настройке модульных тестов](/visualstudio/test/configure-unit-tests-by-using-a-dot-runsettings-file?preserve-view=true&view=vs-2017). Полный пример кода приложения со списком задач, которое используется при работе с этим документом, можно найти на сайте [GitHub](https://github.com/Azure-Samples/documentdb-dotnet-todo-app).

Ниже приведен пример файла **.runsettings**, в котором определены параметры для передачи в модульные тесты приложения. Обратите внимание, что используемая переменная `authKey` является [известным ключом](./local-emulator.md#authenticate-requests) эмулятора. Задача сборки эмулятора ожидает именно этот ключ `authKey`, который нужно определить в файле **.runsettings**.

```csharp
<RunSettings>
    <TestRunParameters>
    <Parameter name="endpoint" value="https://localhost:8081" />
    <Parameter name="authKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==" />
    <Parameter name="database" value="ToDoListTest" />
    <Parameter name="collection" value="ItemsTest" />
  </TestRunParameters>
</RunSettings>
```

Если вы настраиваете конвейер CI/CD для приложения, использующего API Azure Cosmos DB для MongoDB, по умолчанию строка подключения содержит такой номер порта: 10255. Но сейчас этот порт не открыт. Вместо него можете использовать порт 10250 для подключения. В строке подключения API Azure Cosmos DB для MongoDB нужно изменить только номер порта 10250 на 10255.

Эти параметры `TestRunParameters` указываются с помощью свойства `TestContext` в тестовом проекте приложения. Ниже приведен пример теста, который выполняется для Cosmos DB.

```csharp
namespace todo.Tests
{
    [TestClass]
    public class TodoUnitTests
    {
        public TestContext TestContext { get; set; }

        [TestInitialize()]
        public void Initialize()
        {
            string endpoint = TestContext.Properties["endpoint"].ToString();
            string authKey = TestContext.Properties["authKey"].ToString();
            Console.WriteLine("Using endpoint: ", endpoint);
            DocumentDBRepository<Item>.Initialize(endpoint, authKey);
        }
        [TestMethod]
        public async Task TestCreateItemsAsync()
        {
            var item = new Item
            {
                Id = "1",
                Name = "testName",
                Description = "testDescription",
                Completed = false,
                Category = "testCategory"
            };

            // Create the item
            await DocumentDBRepository<Item>.CreateItemAsync(item);
            // Query for the item
            var returnedItem = await DocumentDBRepository<Item>.GetItemAsync(item.Id, item.Category);
            // Verify the item returned is correct.
            Assert.AreEqual(item.Id, returnedItem.Id);
            Assert.AreEqual(item.Category, returnedItem.Category);
        }

        [TestCleanup()]
        public void Cleanup()
        {
            DocumentDBRepository<Item>.Teardown();
        }
    }
}
```

Перейдите к разделу с параметрами выполнения в задаче Visual Studio Test. В параметре **Файл настроек** укажите, что тесты настраиваются с помощью файла **.runsettings**. В параметре **Переопределить параметры тестового запуска** добавьте `-endpoint $(CosmosDbEmulator.Endpoint)`. Это действие настраивает для тестовой задачи ссылку на другую конечную точку задачи сборки эмулятора вместо той, которая определена в файле **.runsettings**.  

:::image type="content" source="./media/tutorial-setup-ci-cd/addExtension_5.png" alt-text="Переопределение переменной конечной точки для задачи сборки эмулятора":::

## <a name="run-the-build"></a>Запустите сборку.

Теперь щелкните **Save and queue** (Сохранить и поместить в очередь), чтобы сохранить сборку и поместить ее в очередь. 

:::image type="content" source="./media/tutorial-setup-ci-cd/runBuild_1.png" alt-text="На снимке экрана показана сборка с выбранной очередью Save &.":::

После запуска сборки вы увидите, что задача эмулятора Cosmos DB запрашивает и получает образ Docker, в котором установлен эмулятор. 

:::image type="content" source="./media/tutorial-setup-ci-cd/runBuild_4.png" alt-text="На снимке экрана показана задача эмулятора Cosmos D B, которая будет извлечена.":::

После завершения сборки вы увидите, что все тесты из задачи сборки успешно выполнены для эмулятора Cosmos DB.

:::image type="content" source="./media/tutorial-setup-ci-cd/buildComplete_1.png" alt-text="Снимок экрана показывает значение прогрессии на вкладке Сводка.":::

## <a name="next-steps"></a>Дальнейшие действия

См. дополнительные сведения об [использовании эмулятора Azure Cosmos DB для разработки и тестирования в локальной среде](./local-emulator.md).

Инструкции см. в статье об [экспорте TLS/SSL-сертификатов эмулятора Azure Cosmos DB для использования с Java, Python и Node.js](./local-emulator-export-ssl-certificates.md).
