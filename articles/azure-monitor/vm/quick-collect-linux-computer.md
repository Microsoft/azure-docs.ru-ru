---
title: Краткое руководство. Сбор данных с гибридного компьютера Linux с помощью Azure Monitor
description: Из этого краткого руководства вы узнаете, как развернуть агент Log Analytics для компьютеров Linux, которые работают вне среды Azure, и включить сбор данных с помощью журналов Azure Monitor.
services: azure-monitor
documentationcenter: azure-monitor
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: quickstart
ms.date: 12/24/2019
ms.author: bwren
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019
ms.openlocfilehash: dbe51930ec92ec4f89738dc5d543003f45acebf9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101733828"
---
# <a name="quickstart-collect-data-from-a-linux-computer-in-a-hybrid-environment-with-azure-monitor"></a>Краткое руководство. Сбор данных с компьютера Linux в гибридной среде с помощью Azure Monitor

[Azure Monitor](../overview.md) может собирать данные напрямую c физических компьютеров или виртуальных машин Linux в вашей среде в рабочую область Log Analytics для подробного анализа и корреляции. [Агент Log Analytics](../agents/log-analytics-agent.md) позволяет Azure Monitor собирать данные из центра обработки данных или другой облачной среды. В этом кратком руководстве показано, как настроить и собирать данные c сервера Linux с помощью нескольких простых действий. Сведения о виртуальных машинах Linux в Azure см. в статье [Сбор данных о виртуальных машинах Azure](./quick-collect-azurevm.md).  

Дополнительные сведения о поддерживаемой конфигурации см. в разделах о [поддерживаемых операционных системах](../agents/agents-overview.md#supported-operating-systems) и [конфигурации сетевых брандмауэров](../agents/log-analytics-agent.md#network-requirements).
 
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на портал Azure по адресу [https://portal.azure.com](https://portal.azure.com). 

## <a name="create-a-workspace"></a>Создание рабочей области

1. На портале Azure щелкните **Все службы**. В списке ресурсов введите **Log Analytics**. Как только вы начнете вводить символы, список отфильтруется соответствующим образом. Выберите **Рабочие области Log Analytics**.

    ![Поиск рабочей области Log Analytics на портале Azure](media/quick-collect-azurevm/azure-portal-log-analytics-workspaces.png)<br>  

2. Выберите **Создать** и задайте следующие параметры:

   * Введите имя для новой **Рабочей области Log Analytics**, например *DefaultLAWorkspace*.  
   * Выберите в раскрывающемся списке **Подписку**, с которой нужно связать рабочую область, если выбранная по умолчанию не подходит.
   * В разделе **Группа ресурсов** выберите имеющуюся группу ресурсов, в которой содержится одна или несколько виртуальных машин Azure.  
   * Выберите **Расположение**, в котором развернуты виртуальные машины.  Дополнительные сведения о доступности службы Log Analytics в регионах см. в [этой статье](https://azure.microsoft.com/regions/services/).
   * При создании рабочей области в новой подписке, созданной после 2 апреля 2018 г., будет автоматически использоваться тарифный план *За ГБ*, и выбор ценовой категории будет недоступен.  При создании рабочей области в существующей подписке, созданной до 2 апреля, или в подписке, которая была привязана к существующей регистрации EA, выберите нужную ценовую категорию.  Дополнительные сведения о конкретной ценовой категории см. в статье [Цены на Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/).
  
        ![Создание рабочей области Log Analytics на портале Azure](media/quick-collect-azurevm/create-log-analytics-workspace-azure-portal.png) 

3. Завершив ввод обязательных сведений на панели **Рабочая область Log Analytics**, щелкните **OK**.  

Пока проверяются данные, ход создания рабочей области можно проверить в разделе **Уведомления** в меню. 

## <a name="obtain-workspace-id-and-key"></a>Получение идентификатора и ключа рабочей области

Перед установкой агента Log Analytics для Linux требуется получить идентификатор и ключ для рабочей области Log Analytics. Эта информация необходима скрипту оболочки агента для правильной настройки агента и обеспечения его взаимодействия с Azure Monitor.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]  

1. Щелкните **Все службы** в левом верхнем углу на портале Azure. В поле поиска введите **Log Analytics**. По мере ввода символов список отфильтруется соответствующим образом. Выберите **Рабочие области Log Analytics**.

2. В списке рабочих областей Log Analytics выберите созданную ранее рабочую область. (Возможно, она называется **DefaultLAWorkspace**.)

3. Выберите **Agents management** (Управление агентами):
 
4. Нажмите **Серверы Linux**.

5. Необходимые значения указаны справа от полей **Идентификатор рабочей области** и **Первичный ключ**. Скопируйте их и вставьте в любой удобный для вас редактор.

## <a name="install-the-agent-for-linux"></a>Установка агента для Linux

Ниже приведены инструкции по настройке установки агента для Log Analytics в Azure и облаке Azure для государственных организаций.  

>[!NOTE]
>Агент Log Analytics для Linux невозможно настроить для отправки отчетов в несколько рабочих областей Log Analytics.  

Если требуется подключение компьютера Linux через прокси-сервер к Log Analytics, конфигурацию прокси-сервера можно указать в командной строке, включив `-p [protocol://][user:password@]proxyhost[:port]`.  Свойство *proxyhost* принимает полное доменное имя или IP-адрес прокси-сервера. 

Например: `https://user01:password@proxy01.contoso.com:30443`

1. Чтобы настроить компьютер Linux для подключения к рабочей области Log Analytics, выполните следующую команду, указав идентификатор рабочей области и первичный ключ, скопированные ранее. Следующая команда скачивает агент, проверяет его контрольную сумму и устанавливает его. 
    
    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

    Следующая команда включает параметр `-p` прокси-сервера и пример синтаксиса, если для прокси-сервера требуется проверка подлинности.

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://][user:password@]proxyhost[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

2. Чтобы настроить компьютер Linux для подключения к рабочей области Log Analytics в облаке Azure для государственных организаций, выполните следующую команду, указав идентификатор рабочей области и первичный ключ, скопированные ранее. Следующая команда скачивает агент, проверяет его контрольную сумму и устанавливает его. 

    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ``` 

    Следующая команда включает параметр `-p` прокси-сервера и пример синтаксиса, если для прокси-сервера требуется проверка подлинности.

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://][user:password@]proxyhost[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ```

3. Перезапустите агент, выполнив следующую команду. 

    ```
    sudo /opt/microsoft/omsagent/bin/service_control restart [<workspace id>]
    ``` 

## <a name="collect-event-and-performance-data"></a>Сбор данных событий и производительности

Azure Monitor может собирать события из системного журнала и счетчиков производительности Linux, указанных для долгосрочного анализа и формирования отчетов. Служба также может выполнять определенные действия при обнаружении указанного условия. Сначала выполните приведенные ниже действия для настройки сбора событий из системного журнала Linux, а также нескольких стандартных счетчиков производительности.  

1. На портале Azure щелкните **Все службы**. В списке ресурсов введите Log Analytics. По мере ввода символов список отфильтруется соответствующим образом. Выберите **Рабочие области Log Analytics** и выберите в списке рабочих областей Log Analytics нужную рабочую область, затем перейдите к разделу **Дополнительные параметры** для этой рабочей области **Log Analytics**.

2. Выберите **Данные** и **Системный журнал**.  

3. Чтобы добавить системный журнал, введите имя журнала. Введите **Системный журнал** и щелкните знак "плюс" **+** .  

4. В таблице снимите флажок для степеней серьезности **Информация**, **Уведомление** и **Отладка**. 

5. Вверху щелкните **Сохранить**, чтобы сохранить конфигурацию.

6. Выберите **Данные производительности Linux**, чтобы включить сбор данных счетчиков производительности на компьютере Linux. 

7. При первой настройке счетчиков производительности Linux для новой рабочей области Log Analytics вы можете быстро создать несколько распространенных счетчиков. Рядом с каждым счетчиком в списке есть флажок.

    ![Счетчики производительности Linux по умолчанию, выбранные в Azure Monitor](media/quick-collect-azurevm/linux-perfcounters-azure-monitor.png)

    Выберите **Применение конфигурации, указанной ниже, к моим машинам** и щелкните **Добавить выбранные счетчики производительности**. Они добавляются и устанавливаются с десятисекундным интервалом сбора.  

8. Вверху щелкните **Сохранить**, чтобы сохранить конфигурацию.

## <a name="view-data-collected"></a>Просмотр собранных данных

Теперь, когда сбор данных включен, можно запустить простой пример поиска по журналам, чтобы просмотреть некоторые данные с целевого компьютера.  

1. В выбранной рабочей области на панели слева выберите **Журналы**.

2. На странице запросов к журналам в редакторе запросов введите `Perf` и щелкните **Выполнить**.
 
    ![Поиск по журналам Log Analytics](media/quick-collect-linux-computer/log-analytics-portal-queryexample.png)

    Например, запрос на следующем рисунке вернул 10 000 записей о производительности. Ваши результаты будут значительно меньше.

    ![Результат поиска по журналам Log Analytics](media/quick-collect-linux-computer/log-analytics-search-perf.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда агент больше не нужен, его можно удалить с компьютера Linux, а также можно удалить рабочую область Log Analytics.  

Чтобы удалить агент, выполните следующую команду на компьютере Linux. Аргумент *--purge* полностью удаляет агент и его конфигурацию.

   `wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh --purge`

Чтобы удалить рабочую область Log Analytics, созданную ранее, выберите ее и на странице ресурсов щелкните **Удалить**.

![Удаление ресурса Log Analytics](media/quick-collect-linux-computer/log-analytics-portal-delete-resource.png)

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы собираете данные о работе и производительности со своих локальных компьютеров Linux, можно легко начать изучение и анализ собранных данных, а также производить с ними различные операции *бесплатно*.  

Чтобы узнать, как просматривать и анализировать данные, перейдите к следующему руководству.

> [!div class="nextstepaction"]
> [Просмотр и анализ данных, собранных с помощью поиска по журналам Log Analytics](../logs/log-analytics-tutorial.md)