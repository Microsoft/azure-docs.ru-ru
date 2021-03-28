---
title: Изменение и Управление приложениями логики с помощью Visual Studio с Cloud Explorer
description: Редактирование, обновление, управление, добавление в систему управления версиями и развертывание приложений логики с помощью Visual Studio с Cloud Explorer
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: article
ms.custom: mvc
ms.date: 04/29/2020
ms.openlocfilehash: 56b74e440fcb09ab206bbb069517dd756221f809
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105639557"
---
# <a name="manage-logic-apps-with-visual-studio"></a>Управление приложениями логики в Visual Studio

Несмотря на то, что вы можете создавать, изменять, администрировать и развертывать приложения логики в [портал Azure](https://portal.azure.com), можно также использовать Visual Studio, если вы хотите добавить приложения логики в систему управления версиями, опубликовать разные версии и создать шаблоны [Azure Resource Manager](../azure-resource-manager/management/overview.md) для различных сред развертывания. С помощью Visual Studio Cloud Explorer вы можете найти приложения логики и другие ресурсы Azure и управлять ими. Например, вы можете открывать, загружать, редактировать, запускать, просматривать журнал запусков, а также отключать и включать приложения логики, которые уже развернуты на портале Azure. Если вы ранее не работали с Azure Logic Apps в Visual Studio, [узнайте о создании приложений логики с помощью Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

Вы также можете [управлять приложениями логики в портал Azure](manage-logic-apps-with-azure-portal.md).

> [!IMPORTANT]
> Развертывание или публикация приложения логики из Visual Studio перезаписывает версию этого приложения на портале Azure. Поэтому, если вы вносите изменения на портале Azure, которые нужно сохранить, [обновите приложение логики в Visual Studio](#refresh) с портала Azure перед следующим развертыванием или публикацией из Visual Studio.

<a name="requirements"></a>

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас еще нет подписки Azure, [зарегистрируйтесь для получения бесплатной учетной записи Azure](https://azure.microsoft.com/free/).

* Скачайте и установите эти средства, если вы еще этого не сделали:

  * [Visual Studio 2019, 2017 или 2015 — выпуски Community или выше](https://aka.ms/download-visual-studio). В этом кратком руководстве используется бесплатная версия Visual Studio Community 2017.

    > [!IMPORTANT]
    > При установке Visual Studio 2019 или 2017 обязательно выберите рабочую нагрузку **разработки Azure**.
    > Дополнительные сведения см. [в разделе Управление ресурсами, связанными с учетными записями Azure в Visual Studio Cloud Explorer](/visualstudio/azure/vs-azure-tools-resources-managing-with-cloud-explorer).

    Чтобы установить Cloud Explorer для Visual Studio 2015, [скачайте Cloud Explorer из Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=MicrosoftCloudExplorer.CloudExplorerforVisualStudio2015). Дополнительные сведения см. [в статье Управление ресурсами, связанными с учетными записями Azure в Visual Studio Cloud Explorer (2015)](/visualstudio/azure/vs-azure-tools-resources-managing-with-cloud-explorer?view=vs-2015&preserve-view=true).

  * [Пакет SDK для Azure (2.9.1 или более поздней версии)](https://azure.microsoft.com/downloads/)

  * [Azure PowerShell](https://github.com/Azure/azure-powershell#installation)

  * Новейшие средства Azure Logic Apps для расширения Visual Studio нужной версии:

    * [Visual Studio 2019](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2019)

    * [Visual Studio 2017](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2017)

    * [Visual Studio 2015](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2015)

    Вы можете скачать и установить средства Azure Logic Apps напрямую из Visual Studio Marketplace или узнать, [как установить это расширение из Visual Studio](/visualstudio/ide/finding-and-using-visual-studio-extensions). После завершения установки перезагрузите Visual Studio.

  * Чтобы использовать подписки Azure для государственных организаций в Visual Studio, ознакомьтесь с приведенными ниже разделами. В них описывается дополнительная настройка.

    * Visual Studio 2019: [Краткое руководство. Подключение к Azure для государственных организаций с помощью Visual Studio](../azure-government/documentation-government-connect-vs.md)

    * Visual Studio 2017: [Introducing the Azure Environment Selector Visual Studio extension](https://devblogs.microsoft.com/azuregov/introducing-the-azure-environment-selector-visual-studio-extension/) (Знакомство с расширением селектора среды Azure для Visual Studio). Описываемое расширение можно скачать и установить из [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=SteveMichelotti.AzureEnvironmentSelector).

* Доступ к Интернету при использовании встроенного конструктора Logic Apps

  Конструктору требуется подключение к Интернету, чтобы создать ресурсы в Azure и считать свойства и данные из соединителей в приложении логики.

<a name="find-logic-apps-vs"></a>

## <a name="find-logic-apps"></a>Поиск приложений логики

В Visual Studio с помощью Cloud Explorer можно найти все приложения логики, которые связаны с подпиской Azure и развернуты на портале Azure.

1. Запустите Visual Studio. В меню **Представление** выберите **Cloud Explorer**.

1. В Cloud Explorer выберите значок **Управление учетной записью** . Выберите подписку Azure, связанную с приложениями логики, и нажмите кнопку **Применить**. Пример:

   ![Выберите "Управление учетными записями".](./media/manage-logic-apps-with-visual-studio/account-management-select-Azure-subscription.png)

1. Рядом со значком **Управление учетной записью** выберите **типы ресурсов**. В подписке Azure разверните узел **Logic Apps** , чтобы просмотреть все развернутые приложения логики, связанные с вашей подпиской.

Затем откройте приложение логики в редакторе приложений логики.

<a name="open-designer"></a>

## <a name="open-logic-apps-in-visual-studio"></a>Открытие приложений логики в Visual Studio

В Visual Studio можно открыть ранее созданные и развернутые приложения логики непосредственно через портал Azure или как проекты группы ресурсов Azure с помощью Visual Studio.

1. [Откройте Cloud Explorer и найдите свое приложение логики](#find-logic-apps-vs).

1. В контекстном меню приложения логики выберите **Открыть в редакторе приложений логики**.

   > [!TIP]
   > Если у вас нет этой команды в Visual Studio 2019, убедитесь, что установлены последние обновления для Visual Studio.

   ![Открытие развернутого приложения логики с портала Azure](./media/manage-logic-apps-with-visual-studio/open-logic-app-in-editor.png)

   После открытия приложения логики в конструкторе Logic Apps в нижней части конструктора можно выбрать **представление кода** , чтобы можно было просмотреть структуру определения базового приложения логики. Если вы хотите создать шаблон развертывания для приложения логики, узнайте о [загрузке шаблона Azure Resource Manager](#download-logic-app) для этого приложения. Дополнительные сведения о [шаблонах диспетчер ресурсов](../azure-resource-manager/templates/overview.md).

<a name="download-logic-app"></a>

## <a name="download-from-azure"></a>Скачивание из Azure

Вы можете [скачать](../azure-resource-manager/templates/export-template-portal.md#export-template-from-a-resource) приложения логики из [портал Azure](https://portal.azure.com) и сохранить их как шаблоны [Azure Resource Manager](../azure-resource-manager/management/overview.md) . Затем вы можете локально изменить шаблоны в Visual Studio и настроить приложения логики для разных сред развертывания.  При загрузке приложений логики автоматически *параметризуются* их определения внутри [шаблонов Resource Manager](../azure-resource-manager/templates/overview.md), которые также используют формат JSON.

1. В Visual Studio с помощью Cloud Explorer [откройте приложение логики, которое необходимо скачать из Azure](#open-designer).

1. В контекстном меню приложения логики выберите **Открыть в редакторе приложений логики**.

   > [!TIP]
   > Если у вас нет этой команды в Visual Studio 2019, убедитесь, что установлены последние обновления для Visual Studio.

   Приложение логики откроется в конструкторе приложений логики.

1. На панели инструментов конструктора нажмите кнопку **загрузить**.

   ![Скачать приложение логики из портал Azure](./media/manage-logic-apps-with-visual-studio/download-logic-app-from-portal.png)

1. Когда появится запрос указать расположение, перейдите к нему и сохраните шаблон Resource Manager для определения этого приложения логики в виде JSON-файла.

   Определение вашего приложения логики появится в подразделе `resources` внутри шаблона Resource Manager. Теперь вы можете изменить определение приложения логики и шаблон Resource Manager с помощью Visual Studio. Вы также можете добавить шаблон в качестве [проекта группы ресурсов Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md) в решение Visual Studio. Сведения о [проектах групп ресурсов Azure для приложений логики в Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

<a name="link-integration-account"></a>

## <a name="link-to-integration-account"></a>Ссылка на учетную запись интеграции

Чтобы создать приложения логики для сценариев интеграции "бизнес — бизнес" (B2B) Enterprise, можно связать приложение логики с ранее созданной [учетной записью интеграции](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) , которая находится в том же регионе, что и приложение логики. Учетная запись интеграции содержит артефакты B2B, такие как торговые партнеры, соглашения, схемы и карты, и позволяет приложению логики использовать соединители B2B для проверки XML и кодирования неструктурированных файлов или декодирования. Несмотря на то, что [эту ссылку можно создать с помощью портал Azure](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account), можно также использовать Visual Studio после выполнения [предварительных требований](#requirements), и приложение логики существует в виде JSON-файла (JSON) в [проекте группы ресурсов Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md). Сведения о [проектах групп ресурсов Azure для приложений логики в Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#create-resource-group-project).

1. В Visual Studio откройте проект группы ресурсов Azure, содержащий приложение логики.

1. В обозреватель решений откройте контекстное меню **<Logic-App-Name # C1.js** и выберите **Открыть в конструкторе приложений логики**. (Или нажмите клавиши CTRL+L)

   ![Открытие JSON файла приложения логики с помощью конструктора приложений логики](./media/manage-logic-apps-with-visual-studio/open-logic-app-designer.png)

   > [!TIP]
   > Если у вас нет этой команды в Visual Studio 2019, убедитесь, что у вас есть последние обновления Visual Studio и расширения Azure Logic Apps Tools.

1. Убедитесь, что конструктор приложений логики имеет фокус, выбрав вкладку или область конструктора, чтобы окно свойств Показать свойство **учетной записи интеграции** для приложения логики.

   ![Окно свойств — свойство "учетная запись интеграции"](./media/manage-logic-apps-with-visual-studio/open-logic-app-properties-integration-account.png)

   > [!TIP]
   > Если окно свойств еще не открыт, в меню **вид** выберите пункт **окно свойств**. (Клавиатура: нажмите клавишу F4)

1. Откройте список свойств **учетной записи интеграции** и выберите учетную запись интеграции, которую нужно связать с приложением логики, например:

   ![Открыть список свойств "учетная запись интеграции"](./media/manage-logic-apps-with-visual-studio/select-integration-account.png)

1. Когда все будет готово, не забудьте сохранить решение Visual Studio.

Если задать свойство **учетной записи интеграции** в Visual Studio и сохранить приложение логики как шаблон Azure Resource Manager, этот шаблон также будет содержать объявление параметра для выбранной учетной записи интеграции. Дополнительные сведения о параметрах шаблонов и приложениях логики см. в разделе [Обзор: Автоматизация развертывания приложений логики](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#template-parameters).

<a name="change-location"></a>

## <a name="change-deployment-location"></a>Изменить расположение развертывания

В Visual Studio, если приложение логики существует как файл JSON (JSON) в [проекте группы ресурсов Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md) , который используется для автоматизации развертывания, для этого приложения логики задается тип расположения и определенное расположение. Это расположение является либо регионом Azure, либо существующей [средой службы интеграции (ISE)](connect-virtual-network-vnet-isolated-environment.md).

Чтобы изменить тип расположения или расположение приложения логики, необходимо открыть файл определения рабочего процесса (JSON) приложения логики из обозреватель решений с помощью конструктора приложений логики. Эти свойства нельзя изменить с помощью Cloud Explorer.

> [!IMPORTANT]
> Изменение типа расположения с **региона** на [**Среда службы интеграции**](connect-virtual-network-vnet-isolated-environment-overview.md) влияет на [модель ценообразования](logic-apps-pricing.md#fixed-pricing) приложения логики, используемую для выставления счетов, [ограничений](logic-apps-limits-and-config.md#integration-account-limits), [поддержки учетных записей интеграции](connect-virtual-network-vnet-isolated-environment-overview.md#ise-skus)и т. д. Прежде чем выбрать другой тип расположения, убедитесь, что вы понимаете влияние на приложение логики.

1. В Visual Studio откройте проект группы ресурсов Azure, содержащий приложение логики.

1. В обозреватель решений откройте `<logic-app-name>.json` контекстное меню файла и выберите **Открыть в конструкторе приложений логики**. (Или нажмите клавиши CTRL+L)

   ![Открытие JSON файла приложения логики с помощью конструктора приложений логики](./media/manage-logic-apps-with-visual-studio/open-logic-app-designer.png)

   > [!TIP]
   > Если у вас нет этой команды в Visual Studio 2019, убедитесь, что у вас есть последние обновления Visual Studio и расширения Azure Logic Apps Tools.

1. Убедитесь, что конструктор приложений логики имеет фокус, выбрав вкладку или область конструктора, чтобы окно свойств Показать свойства **выбора типа расположения** и **расположения** для приложения логики. Для типа расположения проекта задано значение " **регион** " или " **Среда службы интеграции**".

   ![Окно свойств-"Выбор типа расположения" & "расположение"](./media/manage-logic-apps-with-visual-studio/open-logic-app-properties-location.png)

   > [!TIP]
   > Если окно свойств еще не открыт, в меню **вид** выберите пункт **окно свойств**. (Клавиатура: нажмите клавишу F4)

1. Чтобы изменить тип расположения, откройте список свойств **Выбор типа расположения** и выберите нужный тип расположения.

   Например, если тип расположения — **Среда службы интеграции**, можно выбрать **регион**.

   ![Свойство "Выбор типа расположения" — изменение типа расположения](./media/manage-logic-apps-with-visual-studio/change-location-type.png)

1. Чтобы изменить конкретное расположение, откройте список свойств **Расположение** . В зависимости от типа расположения выберите нужное расположение, например:

   * Выберите другой регион Azure:

     ![Откройте список свойств "расположение", выберите другой регион Azure.](./media/manage-logic-apps-with-visual-studio/change-azure-resource-group-region.png)

   * Выберите другую интегрированную среду сценариев:

     ![Откройте список свойств "расположение", выберите другую интегрированную среду сценариев](./media/manage-logic-apps-with-visual-studio/change-integration-service-environment.png)

1. Когда все будет готово, не забудьте сохранить решение Visual Studio.

При изменении типа расположения или расположения в Visual Studio и сохранении приложения логики в качестве шаблона Azure Resource Manager этот шаблон также включает объявления параметров для этого типа расположения и расположения. Дополнительные сведения о параметрах шаблонов и приложениях логики см. в разделе [Обзор: Автоматизация развертывания приложений логики](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#template-parameters).

<a name="refresh"></a>

## <a name="refresh-from-azure"></a>Обновление из Azure

Если вы измените приложение логики на портале Azure, чтобы сохранить эти изменения, необходимо обновить версию этого приложения с изменениями в Visual Studio.

* В Visual Studio на панели инструментов конструктора приложений логики выберите **Обновить**.

  -или-

* В Cloud Explorer Visual Studio откройте контекстное меню приложения логики и выберите **Обновить**.

![Обновление приложения логики с изменениями](./media/manage-logic-apps-with-visual-studio/refresh-logic-app-with-updates-from-portal.png)

## <a name="publish-logic-app-updates"></a>Публикация обновлений приложения логики

Когда вы будете готовы к развертыванию обновлений приложения логики из Visual Studio в Azure, на панели инструментов конструктора приложений логики выберите **опубликовать**.

![Публикация обновленного приложения логики в портал Azure](./media/manage-logic-apps-with-visual-studio/publish-logic-app-to-azure-portal.png)

## <a name="manually-run-your-logic-app"></a>Запуск приложения логики вручную

В Visual Studio вы можете вручную запустить приложение логики, развернутое в Azure. На панели инструментов конструктора приложений логики выберите **запустить триггер**.

![Запуск триггера для приложения логики вручную](./media/manage-logic-apps-with-visual-studio/manually-run-logic-app.png)

## <a name="review-run-history"></a>Просмотр журнала выполнения

Чтобы проверить состояние приложения логики и диагностировать проблемы, связанные с запуском этого приложения, можно просмотреть подробные сведения для запусков в Visual Studio, например входные и выходные данные.

1. В Cloud Explorer откройте контекстное меню приложения логики и выберите **Открыть журнал запусков**.

   ![Открытие журнала выполнения для приложения логики](./media/manage-logic-apps-with-visual-studio/open-run-history-for-logic-app.png)

1. Чтобы просмотреть сведения об определенном запуске, дважды щелкните его. Пример:

   ![Просмотр сведений о конкретном запуске](./media/manage-logic-apps-with-visual-studio/view-run-history-details.png)
  
   > [!TIP]
   > Чтобы отсортировать таблицу по свойству, выберите заголовок столбца для этого свойства.

1. Разверните шаги, входные и выходные данные которых необходимо проверить, например:

   ![Просмотр входных и выходных данных для каждого шага](./media/manage-logic-apps-with-visual-studio/view-run-history-inputs-outputs.png)

## <a name="disable-or-enable-logic-app"></a>Отключение и включение приложения логики

Остановить запуск триггера в следующий раз, когда будет выполнено его условие, можно без удаления приложения логики. Отключение приложения логики предотвращает создание и запуск обработчиком Logic Apps будущих экземпляров рабочего процесса для этого приложения. В Cloud Explorer откройте контекстное меню приложения логики и выберите **Отключить**.

![Отключение приложения логики в Cloud Explorer](./media/manage-logic-apps-with-visual-studio/disable-logic-app-cloud-explorer.png)

> [!NOTE]
> При отключении приложения логики новые экземпляры запуска не создаются. Работа всех выполняющихся и ожидающих выполнения запусков будет продолжаться до завершения, что может занять некоторое время.

Чтобы повторно активировать приложение логики, в Cloud Explorer откройте контекстное меню приложения логики и выберите **включить**.

![Включение приложения логики в Cloud Explorer](./media/manage-logic-apps-with-visual-studio/enable-logic-app-cloud-explorer.png)

## <a name="delete-your-logic-app"></a>Удаление приложения логики

Чтобы удалить приложение логики с портала Azure, в Cloud Explorer откройте его контекстное меню и выберите **Удалить**.

![Удаление приложения логики из портал Azure](./media/manage-logic-apps-with-visual-studio/delete-logic-app-from-azure-portal.png)

> [!NOTE]
> При удалении приложения логики новые экземпляры запуска не создаются. Всех выполняющиеся и ожидающие выполнения запуски будут отменены. При наличии тысяч операций запуска отмена может занять значительное время.

> [!NOTE]
> При удалении и повторном создании дочернего приложения логики необходимо повторно сохранить родительское приложение логики. Повторно созданное дочернее приложение будет иметь разные метаданные.
> Если вы не пересохраняете родительское приложение логики после повторного создания своего дочернего приложения, вызов дочернего приложения логики завершится ошибкой "несанкционированный". Это поведение относится к приложениям логики родительского и дочернего приложения, например к тем, которые используют артефакты в учетных записях интеграции или вызывают функции Azure.

## <a name="troubleshooting"></a>Устранение неполадок

При открытии проекта приложения логики в конструкторе Logic Apps может не отображаться параметр для выбора подписки Azure. Вместо этого приложение логики откроется не с той подпиской Azure, которую вы хотите использовать. Это происходит, потому что после открытия JSON-файла для приложения логики Visual Studio кэширует первую выбранную подписку для использования в будущем. Чтобы устранить эту проблему, попробуйте одно из этих действий:

* Переименуйте JSON-файл приложения логики. Кэш подписки зависит от имени файла.

* Чтобы удалить ранее выбранные подписки для *всех* приложений логики в решении, удалите скрытую папку параметров Visual Studio (VS) в каталоге решения. Это расположение хранит сведения о подписке.

## <a name="next-steps"></a>Дальнейшие шаги

Из этой статьи вы узнали, как управлять развернутыми приложениями логики в Visual Studio. Теперь ознакомьтесь с настройкой определений приложения логики для развертывания:

> [!div class="nextstepaction"]
> [Создание определений для приложений логики с помощью JSON](../logic-apps/logic-apps-author-definitions.md)
