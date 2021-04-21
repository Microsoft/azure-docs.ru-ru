---
title: Автоматизация задач и рабочих процессов с помощью Visual Studio
description: Создание, планирование и запуск автоматизированных рабочих процессов для сценариев корпоративной интеграции с помощью Azure Logic Apps и Visual Studio.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: quickstart
ms.custom: mvc
ms.date: 03/24/2021
ms.openlocfilehash: 5ae67e5708a7298385a4e27d612566008884b972
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790065"
---
# <a name="quickstart-create-automated-tasks-processes-and-workflows-with-azure-logic-apps---visual-studio"></a>Краткое руководство. Создание автоматизированных задач, операций и рабочих процессов с помощью Azure Logic Apps в Visual Studio

С помощью [Azure Logic Apps](../logic-apps/logic-apps-overview.md) и Visual Studio можно создавать рабочие процессы, которые автоматизируют задачи и процессы для интеграции приложений, данных, систем и служб в предприятиях и организациях. В этом кратком руководстве показано, как разрабатывать и создавать такие рабочие процессы, создавая приложения логики в Visual Studio и развертывая их в Azure. Хотя эти задачи можно выполнять на портале Azure, Visual Studio позволяет добавлять приложения логики в систему управления версиями, публиковать различные версии и создавать шаблоны Azure Resource Manager для различных сред развертывания.

Если вы не работали с Azure Logic Apps и хотите ознакомиться с основными понятиями, см. [руководство по созданию приложения логики на портале Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md). Конструктор приложений логики на портале Azure и в Visual Studio работает аналогичным образом.

В этом кратком руководстве с помощью Visual Studio создается то же приложение логики, что и с помощью портала Azure. Вы также можете узнать, как [создать пример приложения в Visual Studio Code](quickstart-create-logic-apps-visual-studio-code.md) и как [создавать приложения логики и управлять ими с помощью интерфейс командной строки Azure (Azure CLI)](quickstart-logic-apps-azure-cli.md). Это приложение логики отслеживает канал RSS веб-сайта и отправляет сообщение электронной почты для каждого нового элемента в этом канале. По завершении приложение логики будет выглядеть как этот высокоуровневый рабочий процесс:

![Снимок экрана: высокоуровневый рабочий процесс готового приложения логики](./media/quickstart-create-logic-apps-with-visual-studio/high-level-workflow-overview.png)

<a name="prerequisites"></a>

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись и подписка Azure. Если у вас нет ее, вы можете [зарегистрироваться для получения бесплатной учетной записи Azure](https://azure.microsoft.com/free/). Если у вас есть подписка Azure для государственных организаций, выполните следующие дополнительные шаги, чтобы [настроить Visual Studio для облака Azure для государственных организаций](#azure-government).

* Скачайте и установите эти средства, если вы еще этого не сделали:

  * [Visual Studio 2019, 2017 или 2015 — выпуски Community или выше](https://aka.ms/download-visual-studio). В этом кратком руководстве используется Visual Studio Community 2017.

    > [!IMPORTANT]
    > При установке Visual Studio 2019 или 2017 обязательно выберите рабочую нагрузку **разработки Azure**.

  * [Пакет Microsoft Azure SDK для .NET (2.9.1 или более поздней версии)](https://azure.microsoft.com/downloads/). Дополнительные сведения о [пакете Azure SDK для .NET](/dotnet/azure/intro).

  * [Azure PowerShell](https://github.com/Azure/azure-powershell#installation)

  * Новейшие средства Azure Logic Apps для расширения Visual Studio нужной версии:

    * [Visual Studio 2019](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2019)

    * [Visual Studio 2017](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2017)

    * [Visual Studio 2015](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2015)
  
    Вы можете скачать и установить средства Azure Logic Apps напрямую из Visual Studio Marketplace или узнать, [как установить это расширение из Visual Studio](/visualstudio/ide/finding-and-using-visual-studio-extensions). После завершения установки перезагрузите Visual Studio.

* Доступ к Интернету при использовании встроенного конструктора приложений логики

  Конструктору требуется подключение к Интернету, чтобы создать ресурсы в Azure и считать свойства и данные из соединителей в приложении логики.

* Учетная запись, поддерживаемая Logic Apps, например Outlook для Microsoft 365, Outlook.com или Gmail. Сведения о дополнительных поставщиках см. в [списке соединителей](/connectors/). В этом примере используется Office 365 Outlook. Если вы используете другой поставщик, общие шаги те же, но пользовательский интерфейс может немного отличаться.

  > [!IMPORTANT]
  > Только учетные записи для бизнеса G-Suite могут использовать соединитель Gmail без ограничений в приложениях логики. Если у вас есть учетная запись потребителя Gmail, вы можете использовать этот соединитель только с определенными утвержденными Google службами. Кроме того, вы можете [создать клиентское приложение Google, которое будет использоваться для проверки подлинности в соединителе Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Дополнительные сведения см. в статье [Политики безопасности и конфиденциальности данных для соединителей Google в Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Если для приложения логики нужно реализовать обмен данными через брандмауэр, который допускает трафик только на определенные IP-адреса, этот брандмауэр должен разрешать доступ [входящим](logic-apps-limits-and-config.md#inbound) *и* [исходящим](logic-apps-limits-and-config.md#outbound) IP-адресам, используемым службой Logic Apps или средой выполнения в регионе Azure, где существует ваше приложение логики. Если приложение логики также использует [управляемые соединители](../connectors/managed.md) (например, соединитель Office 365 Outlook или соединитель SQL) либо [настраиваемые соединители](/connectors/custom-connectors/), брандмауэр также должен разрешать доступ *всем* [исходящим IP-адресам управляемого соединителя](logic-apps-limits-and-config.md#outbound) в регионе Azure приложения логики.

<a name="azure-government"></a>

## <a name="set-up-visual-studio-for-azure-government"></a>Настройка Visual Studio для Azure для государственных организаций

### <a name="visual-studio-2017"></a>Visual Studio 2017

Вы можете использовать [расширение селектора среды Azure для Visual Studio](https://devblogs.microsoft.com/azuregov/introducing-the-azure-environment-selector-visual-studio-extension/), которое можно скачать и установить из [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=SteveMichelotti.AzureEnvironmentSelector).

### <a name="visual-studio-2019"></a>Visual Studio 2019

Для работы с подписками Azure для государственных организаций в Azure Logic Apps необходимо [добавить в Visual Studio конечную точку обнаружения для облака Azure для государственных организаций](../azure-government/documentation-government-connect-vs.md). Но *перед входом в Visual Studio с учетной записью Azure для государственных организаций* вам необходимо переименовать файл JSON, созданный после добавления конечной точки обнаружения, выполнив следующие действия:

1. Закройте Visual Studio.

1. Найдите созданный файл JSON с именем `Azure U.S. Government-A3EC617673C6C70CC6B9472656832A26.Configuration` в этом расположении:

   `%localappdata%\.IdentityService\AadConfigurations`
 
1. Переименуйте файл JSON на `AadProvider.Configuration.json`.

1. Перезапустите Visual Studio.

1. Выполните шаги для входа с помощью учетной записи Azure для государственных организаций.

Чтобы отменить эту настройку, удалите файл JSON в следующем расположении и перезапустите Visual Studio:

`%localappdata%\.IdentityService\AadConfigurations\AadProvider.Configuration.json`

<a name="create-resource-group-project"></a>

## <a name="create-azure-resource-group-project"></a>Создание проекта группы ресурсов Azure

Чтобы начать работу, создайте [проект группа ресурсов Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md). Узнайте больше о [группах ресурсов Azure и ресурсах](../azure-resource-manager/management/overview.md).

1. Запустите среду Visual Studio. Войдите в систему с использованием учетной записи Azure.

1. В меню **Файл** выберите **Создать** > **Проект**. (Или нажмите клавиши CTRL+SHIFT+N)

   ![В меню "Файл" выберите "Создать > Проект".](./media/quickstart-create-logic-apps-with-visual-studio/create-new-visual-studio-project.png)

1. В разделе **Установленные** выберите **Visual C#** или **Visual Basic**. Выберите **Облако** > **Группа ресурсов Azure**. Назовите свой проект, например:

   ![Создание проекта группы ресурсов Azure](./media/quickstart-create-logic-apps-with-visual-studio/create-azure-cloud-service-project.png)

   > [!NOTE]
   > Имена групп ресурсов могут содержать только буквы, цифры, точки (`.`), символы подчеркивания (`_`), дефисы (`-`) и круглые скобки (`(`, `)`), но не могут *оканчиваться* точками (`.`).
   >
   > Если категория **Cloud** или проект **Группа ресурсов Azure** не отображаются, убедитесь, что у вас установлен пакет Azure SDK для Visual Studio.

   Если вы используете Visual Studio 2019, сделайте следующее:

   1. В поле **Создать проект** выберите проект **Группа ресурсов Azure** для языка C# или Visual Basic. Выберите **Далее**.

   1. Укажите имя группы ресурсов Azure, которую вы хотите использовать, и другие сведения о проекте. Нажмите кнопку **создания**.

1. В списке шаблонов выберите **Приложение логики**. Щелкните **ОК**.

   ![Выбор шаблона Logic App](./media/quickstart-create-logic-apps-with-visual-studio/select-logic-app-template.png)

   После того, как в Visual Studio будет создан проект, в обозревателе решений откроется ваше решение. В решении файл **LogicApp.json** не только хранит определение приложения логики, но также является шаблоном Azure Resource Manager, который можно использовать для развертывания.

   ![Обозреватель решений с новым решением приложения логики и файлом развертывания](./media/quickstart-create-logic-apps-with-visual-studio/logic-app-solution-created.png)

## <a name="create-blank-logic-app"></a>Создание пустого приложения логики

Создав проект группы ресурсов Azure, создайте приложение логики на основе шаблона **Пустое приложение логики**.

1. В обозревателе решений откройте контекстное меню для файла **LogicApp.json**. Выберите действие **Открыть в конструкторе приложений логики**. (Или нажмите клавиши CTRL+L)

   ![Открытие файла JSON приложения логики с помощью конструктора приложений логики](./media/quickstart-create-logic-apps-with-visual-studio/open-logic-app-designer.png)

   > [!TIP]
   > Если у вас нет этой команды в Visual Studio 2019, убедитесь, что установлены последние обновления для Visual Studio.

   Visual Studio требуется подписка Azure и группа ресурсов для создания и развертывания ресурсов, связанных с приложением логики и подключениями.

1. **Подписка** — выберите подписку Azure. **Группа ресурсов** — выберите **Создать новую**, чтобы создать другую группу ресурсов Azure.

   ![Выберите подписку, группу ресурсов и расположение.](./media/quickstart-create-logic-apps-with-visual-studio/select-azure-subscription-resource-group-location.png)

   | Параметр | Пример значения | Описание |
   | ------- | ------------- | ----------- |
   | Учетная запись пользователя | Fabrikam <br> sophia-owen@fabrikam.com | Учетная запись, которая использовалась при входе в Visual Studio |
   | **Подписка** | Оплата по мере использования <br> (sophia-owen@fabrikam.com) | Имя подписки Azure и связанной учетной записи |
   | **Группа ресурсов** | MyLogicApp-RG <br> (Западная часть США) | Группа ресурсов Azure и расположение для хранения и развертывания ресурсов приложения логики |
   | **Расположение** | **То же, что и для группы ресурсов** | Тип расположения и конкретное расположение для развертывания приложения логики. Тип расположения — это либо регион Azure, либо имеющаяся [среда службы интеграции (ISE)](connect-virtual-network-vnet-isolated-environment.md). <p>В этом кратком руководстве предполагается, что для типа расположения оставлено значение **Регион**, а для расположения задано значение **Same as Resource Group** (То же, что и для группы ресурсов). <p>**Примечание.** После создания проекта группы ресурсов можно [изменить расположение и его тип](manage-logic-apps-with-visual-studio.md#change-location), однако следует учитывать, что разные типы расположения по-разному влияют на приложение логики. |
   ||||

1. Откроется конструктор Logic Apps и отобразится страница с вводным видео и часто используемыми триггерами. Прокрутите вниз раздел с видео и триггерами до раздела **Шаблоны** и выберите **Пустое приложение логики**.

   ![Выбор пункта "Пустое приложение логики"](./media/quickstart-create-logic-apps-with-visual-studio/choose-blank-logic-app-template.png)

## <a name="build-logic-app-workflow"></a>Рабочий процесс сборки приложения логики

Затем добавьте [триггер](../logic-apps/logic-apps-overview.md#logic-app-concepts) RSS, который срабатывает при появлении нового элемента веб-канала. Каждое приложение логики начинается с триггера, который срабатывает при удовлетворении определенных критериев. При каждой активации триггера обработчик Logic Apps создает экземпляр приложения логики для выполнения рабочего процесса.

1. В конструкторе приложений логики под полем поиска выберите **Все**. В поле поиска введите rss. В списке триггеров выберите триггер: **При публикации элемента веб-канала**.

   ![Создание приложений логики путем добавления триггера и действий](./media/quickstart-create-logic-apps-with-visual-studio/add-trigger-logic-app.png)

1. Когда триггер появится в конструкторе, завершите создание приложения логики, следуя инструкциям из [краткого руководства по порталу Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md#add-rss-trigger), а затем вернитесь к этой статье. По завершении приложение логики может выглядеть следующим образом:

   ![Готовое приложение логики](./media/quickstart-create-logic-apps-with-visual-studio/finished-logic-app-workflow.png)

1. Сохраните решение Visual Studio. (Или нажмите клавиши Ctrl + S.)

<a name="deploy-to-Azure"></a>

## <a name="deploy-logic-app-to-azure"></a>Развертывание приложения логики в Azure

Перед запуском и тестированием приложения логики разверните приложение из Visual Studio в Azure.

1. В обозревателе решений в контекстном меню проекта выберите **Развернуть** > **Создать**. Если отобразится запрос на вход в учетную запись Azure, выполните его.

   ![Создайте развертывание приложения логики](./media/quickstart-create-logic-apps-with-visual-studio/create-logic-app-deployment.png)

1. Для этого развертывания сохраните подписку, группу ресурсов и другие настройки Azure по умолчанию. Выберите **Развернуть**.

   ![Развертывание приложения логики в группу ресурсов Azure](./media/quickstart-create-logic-apps-with-visual-studio/select-azure-subscription-resource-group-deployment.png)

1. Если появится поле **Изменить параметры**, введите имя ресурса для приложения логики. Сохраните параметры.

   ![Указание имени развертывания для приложения логики](./media/quickstart-create-logic-apps-with-visual-studio/edit-parameters-deployment.png)

   При запуске развертывания его состояние отобразится в окне **Выходные данные** Visual Studio. Если состояние не отображается, откройте список **Показать вывод из** и выберите свою группу ресурсов Azure.

   ![Вывод информации о состоянии развертывания](./media/quickstart-create-logic-apps-with-visual-studio/logic-app-output-window.png)

   Если подключенные соединители требуют ввода данных, может открыться окно PowerShell в фоновом режиме с запросом на ввод необходимых паролей или секретных ключей. После ввода этих сведений развертывание продолжится.

   ![Окно PowerShell](./media/quickstart-create-logic-apps-with-visual-studio/logic-apps-powershell-window.png)

   После развертывания приложение логики будет работать в реальном времени на портале Azure согласно заданному расписанию (каждую минуту). Триггер срабатывает при обнаружении новых элементов веб-канала, создавая экземпляр рабочего процесса, который выполняет действия приложения логики. Приложение логики отправляет сообщение электронной почты при появлении каждого нового элемента. Если же триггер не обнаруживает новые элементы, он не срабатывает и не создает экземпляр рабочего процесса. В свою очередь, приложение логики ожидает наступления следующего интервала перед повторной проверкой.

   Ниже приведены примеры электронных писем, которые отправляет это приложение логики. Если сообщения электронной почты не приходят, проверьте папку нежелательной почты.

   ![Outlook отправляет сообщение электронной почты для каждого нового элемента RSS](./media/quickstart-create-logic-apps-with-visual-studio/outlook-email.png)

Итак, вы успешно создали и развернули приложение логики с помощью Visual Studio. Чтобы управлять приложением логики и просматривать его журнал выполнения, ознакомьтесь со статьей [Manage logic apps with Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md) (Управление приложениями логики в Visual Studio).

## <a name="add-new-logic-app"></a>Добавление нового приложения логики

Если у вас уже есть проект группы ресурсов Azure, вы можете добавить в него новое пустое приложение логики в окне "Структура JSON".

1. В обозревателе решений откройте файл `<logic-app-name>.json`.

1. В меню **Вид** выберите **Другие окна** > **Структура JSON**.

1. Чтобы добавить ресурс в файл шаблона, нажмите **Добавить ресурс** в верхней части окна "Структура JSON". Также можно открыть контекстное меню **ресурсы** в окне "Структура JSON" и выбрать пункт **Добавить новый ресурс**.

   ![Окно "Структура JSON"](./media/quickstart-create-logic-apps-with-visual-studio/json-outline-window-add-resource.png)

1. В диалоговом окне **Добавление ресурса** найдите `logic app` с помощью поля поиска и выберите **Приложение логики**. Присвойте имя приложению логики и выберите **Добавить**.

   ![Добавление ресурса](./media/quickstart-create-logic-apps-with-visual-studio/add-logic-app-resource.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Завершив работу с приложением логики, удалите группу ресурсов, содержащую приложение логики и связанные ресурсы.

1. Войдите на [портал Azure](https://portal.azure.com) с учетной записью, использованной для создания приложения логики.

1. В меню портала Azure выберите **Группы ресурсов** или выполните поиск по запросу **Группы ресурсов** на любой странице и выберите этот пункт. Выберите группу ресурсов приложения логики.

1. На вкладке **Обзор** выберите **Удалить группу ресурсов**. Введите имя группы ресурсов для подтверждения и нажмите кнопку **Удалить**.

   !["Группы ресурсов > Обзор > Удалить группу ресурсов"](./media/quickstart-create-logic-apps-with-visual-studio/clean-up-resources.png)

1. Удалите решение Visual Studio со своего локального компьютера.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы создавали, развертывали и выполняли свое приложение логики с помощью Visual Studio. См. дополнительные сведения о выполнении расширенного развертывания для приложений логики и управлении им c помощью Visual Studio:

> [!div class="nextstepaction"]
> [Управление приложениями логики в Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md)
