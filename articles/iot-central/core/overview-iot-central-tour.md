---
title: Общие сведения о пользовательском интерфейсе Azure IoT Central | Документация Майкрософт
description: Ознакомьтесь с ключевыми областями пользовательского интерфейса Azure IoT Central, с помощью которого можно создавать решения Интернета вещей, использовать их и управлять ими.
author: TheJasonAndrew
ms.author: v-anjaso
ms.date: 02/09/2021
ms.topic: overview
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: corywink
ms.openlocfilehash: 569a1365e73acbc2fdaf351f2e2cff21181241e1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100523487"
---
# <a name="take-a-tour-of-the-azure-iot-central-ui"></a>Общие сведения о пользовательском интерфейсе Azure IoT Central

В этой статье содержатся сведения о пользовательском интерфейсе Microsoft Azure IoT Central. С помощью пользовательского интерфейса можно создавать и использовать решение Azure IoT Central и их подключенные устройства, а также управлять ими.

С помощью пользовательского интерфейса Azure IoT Central _разработчики_ могут определить свое решение Azure IoT Central. Этот интерфейс позволяет выполнять следующие задачи:

* определение типов устройств, подключающихся к решению.
* настройка правил и действий устройств; 
* настройка пользовательского интерфейса для _операторов_, использующих решение.

С помощью пользовательского интерфейса Azure IoT Central _оператор_ может управлять решением Azure IoT Central. Этот интерфейс позволяет выполнять следующие задачи:

* мониторинг устройств;
* настройка устройств;
* устранение проблем и неполадок с устройствами;
* подготавливать новые устройства;

## <a name="iot-central-homepage"></a>Домашняя страница IoT Central

На [домашней странице IoT Central](https://aka.ms/iotcentral-get-started) вы можете просмотреть последние новости и узнать о новых функциях, доступных в IoT Central. Кроме того, здесь можно создавать приложения, а также просматривать и запускать имеющиеся приложения.

:::image type="content" source="media/overview-iot-central-tour/iot-central-homepage.png" alt-text="Домашняя страница IoT Central":::

### <a name="create-an-application"></a>Создание приложения

В разделе "Сборка" приведен список шаблонов IoT Central, соответствующих отраслям, которые позволяют быстро приступить к работе. Вы также можете начать с нуля с помощью шаблона пользовательского приложения.  

:::image type="content" source="media/overview-iot-central-tour/iot-central-build.png" alt-text="Страница &quot;Сборка&quot; в IoT Central":::

Дополнительные сведения см. в кратком руководстве [Создание приложения Azure IoT Central (предварительная версия функции)](quick-deploy-iot-central.md).

### <a name="launch-your-application"></a>Запуск приложения

Чтобы запустить приложение IoT Central, перейдите по URL-адресу, который вы или разработчик решений выбрали при создании приложения. Список всех приложений, к которым вы имеете доступ, также можно просмотреть в [диспетчере приложений IoT Central](https://aka.ms/iotcentral-apps).

:::image type="content" source="media/overview-iot-central-tour/app-manager.png" alt-text="Диспетчер приложений IoT Central":::

## <a name="navigate-your-application"></a>Навигация по приложению

В области слева можно получить доступ к разным областям приложения Интернета вещей. Вы можете развернуть или свернуть панель слева, щелкнув значок с тремя линиями в верхней части панели:

> [!NOTE]
> Элементы, отображаемые на панели слева, зависят от роли пользователя. Дополнительные сведения об управлении учетными записями пользователей и ролями см. [здесь](howto-manage-users-roles.md). 

:::row:::
  :::column span="":::
      :::image type="content" source="media/overview-iot-central-tour/navigation-bar.png" alt-text="Область слева":::

  :::column-end:::
  :::column span="2":::
     **Панель мониторинга** — отображается панель мониторинга приложения. Как *разработчик решений* вы можете настроить глобальную панель мониторинга для операторов. В зависимости от их роли операторы также могут создавать персональные панели мониторинга.
     
     На странице **Устройства** вы можете управлять подключенными устройствами (реальными и имитированными).

     На странице **Группы устройств** вы можете просматривать и создавать логические коллекции устройств, указанных в запросе. Вы можете сохранить этот запрос и использовать группы устройств в приложении для выполнения массовых операций.

     На странице **Правила** вы можете создавать и изменять правила мониторинга устройств. Правила оцениваются на основе данных телеметрии устройства и вызывают настраиваемые действия.

     На странице **Аналитика** вы можете создавать настраиваемые представления на основе данных устройства для получения аналитических сведений из приложения.

     На странице **Задания** вы можете управлять устройствами в нужном масштабе с помощью массовых операций.

     На странице **Шаблоны устройств** вы можете создавать характеристики устройств, которые можно подключить к приложению, и управлять ими.

     На странице **Экспорт данных** вы можете настроить непрерывный экспорт во внешние службы, такие как хранилище и очереди.

     На странице **Администрирование** вы можете управлять параметрами, настройками, выставлением счетов, пользователями и ролями приложения.

     С помощью кнопки **IoT Central** *администраторы* могут вернуться к диспетчеру приложений IoT Central.
     
   :::column-end:::
:::row-end:::

### <a name="search-help-theme-and-support"></a>Поиск, справка, тема и поддержка

На каждой странице сверху отображается меню.

:::image type="content" source="media/overview-iot-central-tour/toolbar.png" alt-text="Панель инструментов IoT Central":::

* Чтобы найти устройства и шаблоны устройств, введите значение в **Поиск**.
* Чтобы изменить язык пользовательского интерфейса или тему, щелкните значок **Параметры**. Дополнительные сведения об управлении параметрами приложения см. [здесь](howto-manage-preferences.md).
* Чтобы получить справку и поддержку, выберите раскрывающийся список **Справка** с перечнем ресурсов. Вы можете [получить сведения о приложении](./howto-get-app-info.md), перейдя по ссылке **О приложении**. В приложении с бесплатным тарифным планом ресурсы поддержки включают в себя доступ к [чату в реальном времени](howto-show-hide-chat.md).
* Чтобы выйти из приложения, щелкните значок **Учетная запись**.

Вы можете выбирать между светлой или темной темой пользовательского интерфейса.

> [!NOTE]
> Возможность выбора между светлой и темной темами будет недоступен, если администратор настроил пользовательскую тему для приложения.

:::image type="content" source="media/overview-iot-central-tour/themes.png" alt-text="Снимок экрана: выбор темы в IoT Central.":::

### <a name="dashboard"></a>Панель мониторинга

:::image type="content" source="Media/overview-iot-central-tour/dashboard.png" alt-text="Снимок экрана: панель мониторинга в IoT Central.":::

* Панель мониторинга — это первая страница, отображаемая при входе в приложение Azure IoT Central. Как *разработчик решений* вы можете создавать и настраивать несколько глобальных панелей мониторинга приложений для других пользователей. Дополнительные сведения о добавлении плиток на панель мониторинга см. [здесь](howto-add-tiles-to-your-dashboard.md).

* *Оператор* с нужной ролью может создавать персональные панели мониторинга для отслеживания необходимых ресурсов. Дополнительные сведения см. в статье о [создании персональных панелей мониторинга Azure IoT Central и управлении ими](howto-create-personal-dashboards.md).

### <a name="devices"></a>Устройства

:::image type="content" source="Media/overview-iot-central-tour/devices.png" alt-text="Снимок экрана: страница устройств.":::

На странице обозревателя отображаются _устройства_ в приложении Azure IoT Central, сгруппированные по _шаблону устройства_. 

* Шаблон определяет тип устройства, которое можно подключить к приложению.
* В устройстве могут использоваться два типа приложений: реальные и смоделированные.

Дополнительные сведения см. в кратком руководстве по [мониторингу устройств](./quick-monitor-devices.md). 

### <a name="device-groups"></a>Группы устройств

:::image type="content" source="Media/overview-iot-central-tour/device-groups.png" alt-text="Страницы группы устройств":::

Группа устройств — это коллекция связанных устройств.Группа устройств — это коллекция связанных устройств. *Разработчик решений* определяет запрос для идентификации устройств, входящих в группу. С помощью групп устройств можно выполнять массовые операции в приложении. Дополнительные сведения см. в статье [Use device groups in your Azure IoT Central application (preview features)](tutorial-use-device-groups.md) (Использование групп устройств в приложении Azure IoT Central (предварительные версии функций)).

### <a name="rules"></a>Правила
:::image type="content" source="Media/overview-iot-central-tour/rules.png" alt-text="Снимок экрана: страница правил.":::

На станице "Правила" можно определить правила на основе телеметрии, состояния или событий устройства. При срабатывании правила может активироваться одно или несколько действий, например отправка сообщения электронной почты, отправка оповещений веб-перехватчика внешней системе и т. д. Дополнительные сведения см. в руководстве по [настройке правил](tutorial-create-telemetry-rules.md). 

### <a name="analytics"></a>Analytics

:::image type="content" source="Media/overview-iot-central-tour/analytics.png" alt-text="Снимок экрана: страница аналитики":::

На странице "Аналитика" вы можете создавать настраиваемые представления на основе данных устройства для получения аналитических сведений из приложения. Дополнительные сведения см. в статье [How to use analytics to analyze device data](howto-create-analytics.md) (Анализ данных устройства с помощью аналитики).

### <a name="jobs"></a>Задания

:::image type="content" source="Media/overview-iot-central-tour/jobs.png" alt-text="Страница заданий":::

На странице "Задания" можно выполнять массовые операции управления устройствами. Вы можете обновлять свойства и параметры, а также выполнить команды для групп устройств. Чтобы узнать больше, ознакомьтесь со статьей [Создание и запуск заданий в приложении Azure IoT Central](howto-run-a-job.md).

### <a name="device-templates"></a>Шаблоны устройств

:::image type="content" source="Media/overview-iot-central-tour/templates.png" alt-text="Снимок экрана: шаблоны устройств":::

На странице шаблонов устройств построитель создает шаблоны устройств в приложении и управляет ими. Шаблоны устройств определяют характеристики устройства, в том числе:

* данные телеметрии, состояние и измерения событий;
* Свойства
* Команды
* Представления

*Разработчик решений* также может создавать формы и панели мониторинга, которые операторы могут использовать для управления устройствами.

Дополнительные сведения см. в статье [Define a new device type in your Azure IoT Central application](howto-set-up-template.md) (Определение типа нового устройства в приложении Azure IoT Central). 

### <a name="data-export"></a>Экспорт данных

:::image type="content" source="Media/overview-iot-central-tour/export.png" alt-text="Экспорт данных":::

На странице "Экспорт данных" вы можете настраивать потоки данных, например данных телеметрии, из приложения во внешние системы. Дополнительные сведения см. в статье [Экспорт данных в Azure IoT Central](./howto-export-data.md).

### <a name="administration"></a>Администрирование

:::image type="content" source="media/overview-iot-central-tour/administration.png" alt-text="Снимок экрана: администрирование Интернета вещей.":::

На странице "Администрирование" вы можете настраивать приложение IoT Central. Здесь можно изменить имя приложения, URL-адрес, тему, управлять пользователями и ролями, создавать маркеры API и экспортировать приложение. Дополнительные сведения см. в статье [How to administer your application](howto-administer.md) (Как управлять приложением).

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы ознакомились с общими сведениями об Azure IoT Central и макетом пользовательского интерфейса, рекомендуем приступить к изучению краткого руководства [Create an Azure IoT Central application](quick-deploy-iot-central.md) (Создание приложения Azure IoT Central).
