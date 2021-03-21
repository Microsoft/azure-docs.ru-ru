---
title: Развертывание сопел Log Analytics Azure для мониторинга Cloud Foundry
description: Пошаговые рекомендации по развертыванию компонента Nozzle системы Cloud Foundry Loggregator для Azure Log Analytics. Используйте Nozzle для мониторинга метрик работоспособности и производительности системы Cloud Foundry.
services: virtual-machines
author: ningk
tags: Cloud-Foundry
ms.assetid: 00c76c49-3738-494b-b70d-344d8efc0853
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/22/2017
ms.author: ningk
ms.openlocfilehash: 54001c47d03b686a8e7c1f59f1e53d405e3bc506
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102557392"
---
# <a name="deploy-azure-log-analytics-nozzle-for-cloud-foundry-system-monitoring"></a>Развертывание компонента Azure Log Analytics Nozzle для мониторинга системы Cloud Foundry

[Azure Monitor](https://azure.microsoft.com/services/log-analytics/) — это служба в Azure. Она помогает собирать и анализировать данные, генерируемые в облачных и локальных средах.

Log Analyticsная сопла (сопла) — это компонент Cloud Foundry (CF), который перенаправляет метрики из [Cloud Foundry loggregator](https://docs.cloudfoundry.org/loggregator/architecture.html) firehose в журналы Azure Monitor. С помощью Nozzle можно собирать, просматривать и анализировать метрики производительности и работоспособности системы CF в нескольких развертываниях.

В этом документе вы узнаете, как развернуть сопло в среде CF, а затем получить доступ к данным из консоли Azure Monitor журналов.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Предварительные условия

Для развертывания Nozzle необходимо выполнить следующие шаги.

### <a name="1-deploy-a-cf-or-pivotal-cloud-foundry-environment-in-azure"></a>1. развертывание среды CF или сводной Cloud Foundry в Azure

Компонент Nozzle можно использовать для развертывания CF с открытым кодом или для развертывания Pivotal Cloud Foundry (PCF).

* [Развертывание Cloud Foundry в Azure](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/blob/master/docs/guidance.md)

* [Развертывание Pivotal Cloud Foundry в Azure](https://docs.pivotal.io/pivotalcf/1-11/customizing/azure.html)

### <a name="2-install-the-cf-command-line-tools-for-deploying-the-nozzle"></a>2. Установка программ командной строки CF для развертывания сопел

Nozzle запускается как приложение в среде CF. Для развертывания приложения необходим интерфейс командной строки CF.

В Nozzle также требуется разрешение на доступ к Loggregator Firehose и Cloud Controller. Для создания и настройки пользователя необходимо иметь службу учетной записи пользователя и проверки подлинности (UAA).

* [Установка интерфейса командной строки Cloud Foundry](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)

* [Установка клиента командной строки UAA для Cloud Foundry](https://github.com/cloudfoundry/cf-uaac/blob/master/README.md)

Перед настройкой клиента командной строки UAA убедитесь, что RubyGems установлен.

### <a name="3-create-a-log-analytics-workspace-in-azure"></a>3. Создание рабочей области Log Analytics в Azure

Рабочую область Log Analytics можно создать вручную или с помощью шаблона. Шаблон будет развертывать настройку предварительно настроенных представлений и оповещений ключевых показателей эффективности для консоли Azure Monitor журналов. 

#### <a name="to-create-the-workspace-manually"></a>Чтобы создать рабочую область OMS вручную, сделайте следующее:

1. В портал Azure выполните поиск в списке служб в Azure Marketplace, а затем выберите Log Analytics рабочие области.
2. Выберите **Создать** и задайте следующие параметры:

   * **Рабочая область Log Analytics**. Введите имя рабочей области.
   * **Подписка**: если у вас несколько подписок, выберите ту же подписку, в которой содержится развертывание CF.
   * **Группа ресурсов**: можно создать группу ресурсов или использовать группу ресурсов с развертыванием CF.
   * **Расположение**: введите расположение.
   * **Ценовая категория**: нажмите кнопку **ОК** для завершения.

Дополнительные сведения см. в статье [Начало работы с журналами Azure Monitor](../azure-monitor/overview.md).

#### <a name="to-create-the-log-analytics-workspace-through-the-monitoring-template-from-azure-market-place"></a>Чтобы создать рабочую область Log Analytics с использованием шаблона мониторинга из Azure Marketplace, сделайте следующее:

1. Откройте портал Azure.
1. Щелкните знак "+" или "Создать ресурс" в левом верхнем углу.
1. Введите Cloud Foundry в окне поиска и выберите "Решение для мониторинга Cloud Foundry".
1. Загрузится начальная страница шаблона решения для мониторинга Cloud Foundry. Щелкните "Создать", чтобы открыть колонку шаблона.
1. Введите обязательные параметры.
    * **Подписка.** Выберите подписку Azure для рабочей области Log Analytics (обычно это та же подписка, что и в развертывании Cloud Foundry).
    * **Группа ресурсов.** Выберите имеющуюся группу ресурсов или создайте ее для рабочей области Log Analytics.
    * **Расположение группы ресурсов.** Выберите расположение группы ресурсов.
    * **Имя рабочей области OMS**. Введите название рабочей области (если рабочей области нет, шаблон создаст ее).
    * **Регион рабочей области OMS**. Выберите расположение рабочей области.
    * **Ценовая категория рабочей области OMS.** Выберите номер SKU рабочей области Log Analytics. Дополнительные сведения см. в разделе, посвященном [ценовым рекомендациям](https://azure.microsoft.com/pricing/details/log-analytics/).
    * **Условия использования.** Выберите "Условия использования" и щелкните "Создать", чтобы принять условия.
1. Указав все параметры, щелкните "Создать", чтобы развернуть шаблон. После завершения развертывания на вкладке уведомлений появится состояние.


## <a name="deploy-the-nozzle"></a>Развертывание Nozzle

Существует несколько различных способов развертывания Nozzle: как элемент PCF или как приложение CF.

### <a name="deploy-the-nozzle-as-a-pcf-ops-manager-tile"></a>Развертывание Nozzle в качестве элемента PCF Operations Manager

Выполните шаги по [установке и настройке Azure Log Analytics Nozzle для PCF](https://docs.pivotal.io/partners/azure-log-analytics-nozzle/installing.html). Это упрощенный подход, элемент PCF Operations Manager автоматически настроит и отправит Nozzle. 

### <a name="deploy-the-nozzle-manually-as-a-cf-application"></a>Развертывание Nozzle вручную в качестве приложения CF

Если PCF Operations Manager не используется, необходимо развернуть Nozzle как приложение. В следующих разделах описан этот процесс.

#### <a name="sign-in-to-your-cf-deployment-as-an-admin-through-cf-cli"></a>Вход в развертывание CF с правами администратора с помощью интерфейса командной строки CF

Выполните следующую команду.
```
cf login -a https://api.${SYSTEM_DOMAIN} -u ${CF_USER} --skip-ssl-validation
```

SYSTEM_DOMAIN — это доменное имя CF. Его можно получить, выполнив поиск "SYSTEM_DOMAIN" в файле манифеста развертывания CF. 

CF_User — это имя администратора CF. Чтобы получить имя и пароль, можно в разделе scim файла манифеста развертывания CF выполнить поиск имени администратора и значения cf_admin_password.

#### <a name="create-a-cf-user-and-grant-required-privileges"></a>Создание пользователя CF и предоставление необходимых привилегий

Выполните следующие команды:
```
uaac target https://uaa.${SYSTEM_DOMAIN} --skip-ssl-validation
uaac token client get admin
cf create-user ${FIREHOSE_USER} ${FIREHOSE_USER_PASSWORD}
uaac member add cloud_controller.admin ${FIREHOSE_USER}
uaac member add doppler.firehose ${FIREHOSE_USER}
```

SYSTEM_DOMAIN — это доменное имя CF. Его можно получить, выполнив поиск "SYSTEM_DOMAIN" в файле манифеста развертывания CF.

#### <a name="download-the-latest-log-analytics-nozzle-release"></a>Скачивание последнего выпуска Log Analytics Nozzle

Выполните следующую команду.
```
git clone https://github.com/Azure/oms-log-analytics-firehose-nozzle.git
cd oms-log-analytics-firehose-nozzle
```

#### <a name="set-environment-variables"></a>Настройка переменных среды

Теперь в текущем каталоге можно задать переменные среды в файле manifest.yml. Ниже показан манифест приложения для Nozzle. Замените значения своими данными рабочей области Log Analytics.

```
OMS_WORKSPACE             : Log Analytics workspace ID: Open your Log Analytics workspace in the Azure portal, select **Advanced settings**, select **Connected Sources**, and select **Windows Servers**.
OMS_KEY                   : OMS key: Open your Log Analytics workspace in the Azure portal, select **Advanced settings**, select **Connected Sources**, and select **Windows Servers**.
OMS_POST_TIMEOUT          : HTTP post timeout for sending events to Azure Monitor logs. The default is 10 seconds.
OMS_BATCH_TIME            : Interval for posting a batch to Azure Monitor logs. The default is 10 seconds.
OMS_MAX_MSG_NUM_PER_BATCH : The maximum number of messages in a batch to Azure Monitor logs. The default is 1000.
API_ADDR                  : The API URL of the CF environment. For more information, see the preceding section, "Sign in to your CF deployment as an admin through CF CLI."
DOPPLER_ADDR              : Loggregator's traffic controller URL. For more information, see the preceding section, "Sign in to your CF deployment as an admin through CF CLI."
FIREHOSE_USER             : CF user you created in the preceding section, "Create a CF user and grant required privileges." This user has firehose and Cloud Controller admin access.
FIREHOSE_USER_PASSWORD    : Password of the CF user above.
EVENT_FILTER              : Event types to be filtered out. The format is a comma-separated list. Valid event types are METRIC, LOG, and HTTP.
SKIP_SSL_VALIDATION       : If true, allows insecure connections to the UAA and the traffic controller.
CF_ENVIRONMENT            : Enter any string value for identifying logs and metrics from different CF environments.
IDLE_TIMEOUT              : The Keep Alive duration for the firehose consumer. The default is 60 seconds.
LOG_LEVEL                 : The logging level of the Nozzle. Valid levels are DEBUG, INFO, and ERROR.
LOG_EVENT_COUNT           : If true, the total count of events that the Nozzle has received and sent are logged to Azure Monitor logs as CounterEvents.
LOG_EVENT_COUNT_INTERVAL  : The time interval of the logging event count to Azure Monitor logs. The default is 60 seconds.
```

### <a name="push-the-application-from-your-development-computer"></a>Отправка приложения из компьютера разработчика

Убедитесь, что вы находитесь в папке oms-log-analytics-firehose-nozzle. Выполните следующую команду.
```
cf push
```

## <a name="validate-the-nozzle-installation"></a>Проверка установки Nozzle

### <a name="from-apps-manager-for-pcf"></a>Из Apps Manager (для PCF)

1. Войдите в Operations Manager и убедитесь, что соответствующий элемент отображается на панели мониторинга установки.
2. Войдите в Apps Manager и убедитесь, что пространство, созданное для Nozzle, указано в отчете об использовании и находится в нормальном состоянии.

### <a name="from-your-development-computer"></a>Из компьютера разработчика

В окне интерфейса командной строки CF введите:
```
cf apps
```
Убедитесь, что приложение Nozzle OMS выполняется.

## <a name="view-the-data-in-the-azure-portal"></a>Просмотр данных на портале Azure

Если вы развернули решение для мониторинга с помощью шаблона Marketplace, перейдите на портал Azure и найдите решение. Решение можно найти в группе ресурсов, указанной в шаблоне. Щелкните решение, перейдите к консоли log Analytics, просмотрите предварительно настроенные представления, используя основные ключевые показатели эффективности Cloud Foundry системы, данные приложений, оповещения и метрики работоспособности виртуальной машины. 

Если вы создали рабочую область Log Analytics вручную, выполните следующие действия, чтобы создать представления и оповещения.

### <a name="1-import-the-oms-view"></a>1. Импорт представления OMS

На портале OMS перейдите к **просмотру конструктора**  >  **Импорт**  >  **Обзор** и выберите один из файлов omsview. например *Cloud Foundry.omsview*, и сохраните представление. Теперь на главной странице **Обзор** отображается элемент. Выберите его, чтобы просмотреть визуализированные метрики.

Можно настроить эти представления или создать другие в **конструкторе представлений**.

Файл *Cloud Foundry.omsview* является предварительной версией шаблона представления OMS для Cloud Foundry. Это полностью настроенный шаблон по умолчанию. Вы можете отправить свои предложения и отзывы в [разделе проблем](https://github.com/Azure/oms-log-analytics-firehose-nozzle/issues).

### <a name="2-create-alert-rules"></a>2. Создание правил генерации оповещений

В случае необходимости можно [создавать оповещения](../azure-monitor/alerts/alerts-overview.md), настраивать запросы и пороговые значения. Ниже приведены рекомендуемые оповещения.

| Поисковый запрос                                                                  | Создать оповещение на основе | Описание                                                                       |
| ----------------------------------------------------------------------------- | ----------------------- | --------------------------------------------------------------------------------- |
| Type=CF_ValueMetric_CL Origin_s=bbs Name_s="Domain.cf-apps"                   | Число результатов < 1   | **bbs.Domain.cf-apps** указывает, актуальны ли данные домена cf-apps, то есть синхронизированы ли выполняемые запросы приложения CF из Cloud Controller с bbs.LRPsDesired (ИИ Diego-desired). Если данные не получены, это означает, что данные домена cf-apps не являются актуальными в указанный период времени. |
| Type=CF_ValueMetric_CL Origin_s=rep Name_s=UnhealthyCell Value_d>1            | Число результатов > 0   | Для ячеек Diego 0 означает работоспособное состояние, а 1 — неработоспособное. Установите оповещение, срабатывающее при обнаружении нескольких неработоспособных ячеек Diego в указанный период времени. |
| Type=CF_ValueMetric_CL Origin_s="bosh-hm-forwarder" Name_s="system.healthy" Value_d=0 | Число результатов > 0 | 1 означает, что система находится в работоспособном состоянии, а 0 — что она находится в неработоспособном состоянии. |
| Type=CF_ValueMetric_CL Origin_s=route_emitter Name_s=ConsulDownMode Value_d>0 | Число результатов > 0   | Consul периодически выдает состояние работоспособности. 0 означает, что система находится в работоспособном состоянии, а 1 означает, что передатчик маршрута обнаружил, что Consul не работает. |
| Type=CF_CounterEvent_CL Origin_s=DopplerServer (Name_s="TruncatingBuffer.DroppedMessages" or Name_s="doppler.shedEnvelopes") Delta_d>0 | Число результатов > 0 | Количество намеренно удаленных сообщений в Doppler из-за замедленной обратной реакции. |
| Type=CF_LogMessage_CL SourceType_s=LGR MessageType_s=ERR                      | Число результатов > 0   | Loggregator генерирует сообщение **LGR**, чтобы сообщить о проблемах процесса ведения журнала, например, если вывод сообщений журнала слишком интенсивен. |
| Type=CF_ValueMetric_CL Name_s=slowConsumerAlert                               | Число результатов > 0   | Когда сопла получает оповещение о медленных потребителях от loggregator, оно отправляет **оповещение slowconsumeralert** о valuemetric в журналы Azure Monitor. |
| Type=CF_CounterEvent_CL Job_s=nozzle Name_s=eventsLost Delta_d>0              | Число результатов > 0   | Если количество потерянных событий достигает порогового значения, это может указывать на проблемы в работе Nozzle. |

## <a name="scale"></a>Масштабирование

Nozzle и Loggregator можно масштабировать.

### <a name="scale-the-nozzle"></a>Масштабирование Nozzle

Нужно начать как минимум с двух экземпляров Nozzle. Firehose распределяет рабочую нагрузку между всеми экземплярами Nozzle.
Чтобы убедиться, что Nozzle справляется с трафиком данных из Firehose, настройте оповещение **slowConsumerAlert** (указанное в предыдущем разделе "Создание правил генерации оповещений"). После получения оповещения следует выполнить [инструкции на случай низкой производительности Nozzle](https://docs.pivotal.io/pivotalcf/1-11/loggregator/log-ops-guide.html#slow-noz), чтобы определить, не требуется ли изменить масштаб.
Чтобы увеличить масштаб Nozzle, используйте Apps Manager или интерфейс командной строки CF для увеличения числа экземпляров либо ресурсов памяти или диска для Nozzle.

### <a name="scale-the-loggregator"></a>Масштабирование Loggregator

Loggregator отправляет сообщение журнала **LGR**, чтобы сообщить о проблемах процесса ведения журнала. Вы можете отслеживать это оповещение, чтобы определить, нужно ли увеличить масштаб Loggregator.
Чтобы увеличить масштаб Loggregator, увеличьте размер буфера Doppler или добавьте дополнительные сущности сервера Doppler в манифест CF. Дополнительные сведения см. в статье [Configuring System Logging](https://docs.cloudfoundry.org/running/managing-cf/logging-config.html#scaling) (Настройка ведения системного журнала).

## <a name="update"></a>Update

Чтобы установить более новую версию Nozzle, скачайте новый выпуск Nozzle, выполните указания в разделе "Развертывание Nozzle" и отправьте приложение еще раз.

### <a name="remove-the-nozzle-from-ops-manager"></a>Удаление Nozzle из Operations Manager

1. Войдите в Operations Manager.
2. Откройте плитку **Microsoft Azure log Analytics сопла для PCF** .
3. Выберите значок корзины и подтвердите удаление.

### <a name="remove-the-nozzle-from-your-development-computer"></a>Удаление Nozzle из компьютера разработчика

В окне интерфейса командной строки CF введите:
```
cf delete <App Name> -r
```

При удалении Nozzle данные на портале OMS не удаляются автоматически. Срок ее действия истекает в соответствии с параметром хранения журналов Azure Monitor.

## <a name="support-and-feedback"></a>Поддержка и обратная связь

Azure Log Analytics Nozzle является компонентом с открытым кодом. Отправляйте свои вопросы и отзывы в [разделе GitHub](https://github.com/Azure/oms-log-analytics-firehose-nozzle/issues). Чтобы отправить запрос в службу поддержки Azure, выберите категорию службы "Виртуальная машина со службой Cloud Foundry". 

## <a name="next-step"></a>Следующий шаг

В PCF 2.0 метрики производительности виртуальной машины передаются в службу "сопла" Azure Log Analytics с помощью сервера пересылки метрик системы и интегрируются в рабочую область Log Analytics. Вам больше не нужно использовать агент Log Analytics для получения метрик производительности виртуальной машины. Однако агент Log Analytics по-прежнему можно применять для сбора данных системного журнала. Он устанавливается на виртуальные машины CF в виде надстройки Bosh. 

Дополнительные сведения см. в разделе [Deploy Log Analytics agent to your Cloud Foundry deployment](https://github.com/Azure/oms-agent-for-linux-boshrelease) (Развертывание агента Log Analytics в развертывании Cloud Foundry).