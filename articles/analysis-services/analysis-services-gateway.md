---
title: Локальный шлюз данных для Azure Analysis Services | Документация Майкрософт
description: Локальный шлюз данных необходим тогда, когда сервер служб Analysis Services в Azure будет подключен к локальным источникам данных.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 6ca96f76287482a445d8a9a1cdc441333b36efbd
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97739609"
---
# <a name="connecting-to-on-premises-data-sources-with-on-premises-data-gateway"></a>Подключение к локальным источникам данных с помощью локального шлюза данных

Локальный шлюз данных обеспечивает защищенную передачу данных между локальными источниками данных и серверами служб Azure Analysis Services в облаке. Помимо работы с несколькими серверами Azure Analysis Services в одном регионе, последняя версия шлюза также работает с Azure Logic Apps, Power BI, Power Apps и Power автоматизируется. Хотя устанавливаемый шлюз одинаков во всех этих службах, Azure Analysis Services и Logic Apps имеют некоторые дополнительные действия.

Приведенные здесь сведения относятся к работе Azure Analysis Services с локальным шлюзом данных. Дополнительные сведения о шлюзе в целом и о том, как он работает с другими службами, см. в статье [что такое локальный шлюз данных?](/data-integration/gateway/service-gateway-onprem).

Для Azure Analysis Services установка с помощью шлюза в первый раз состоит из четырех этапов:

- **Скачивание и запуск программы установки.** На этом шаге устанавливается служба шлюза на компьютер организации. Войдите в Azure с помощью учетной записи своего [клиента](/previous-versions/azure/azure-services/jj573650(v=azure.100)#what-is-an-azure-ad-tenant) Azure AD. Учетные записи Azure B2B (гостевые) не поддерживаются.

- **Регистрация шлюза.** На этом шаге следует указать имя и ключ восстановления для шлюза и выбрать регион. Таким образом шлюз регистрируется в облачной службе шлюза. Ресурс шлюза можно зарегистрировать в любом регионе, но рекомендуется находиться в том же регионе, что и серверы Analysis Services. 

- **Создание ресурса шлюза в Azure** . на этом шаге вы создадите ресурс шлюза в Azure.

- **Подключите ресурс шлюза к серверам** . После создания ресурса шлюза можно приступить к подключению к нему серверов. Можно подключить несколько серверов и другие ресурсы, если они находятся в одном регионе.

## <a name="installing"></a>Установка

При установке для Azure Analysis Servicesной среды важно выполнить действия, описанные в [статье Установка и настройка локального шлюза данных для Azure Analysis Services](analysis-services-gateway-install.md). Эта статья относится к Azure Analysis Services. Он включает дополнительные шаги, необходимые для настройки локального ресурса шлюза данных в Azure и подключения сервера Azure Analysis Services к ресурсу.

## <a name="connecting-to-a-gateway-resource-in-a-different-subscription"></a>Подключение к ресурсу шлюза в другой подписке

Рекомендуется создать ресурс шлюза Azure в той же подписке, что и сервер. Однако можно настроить серверы для подключения к ресурсу шлюза в другой подписке. Подключение к ресурсу шлюза в другой подписке не поддерживается при настройке существующих параметров сервера или при создании нового сервера на портале, но может быть настроено с помощью PowerShell. Дополнительные сведения см. в статье [подключение ресурса шлюза к серверу](analysis-services-gateway-install.md#connect-gateway-resource-to-server).

## <a name="ports-and-communication-settings"></a>Порты и параметры связи

Шлюз создает исходящее подключение к служебной шине Azure. Он обменивается данными через исходящие порты: TCP 443 (по умолчанию), а также 5671, 5672 и 9350–9354.  Шлюзу не требуются входящие порты.

Может потребоваться включить IP-адреса для вашей области данных в брандмауэре. Вы можете скачать [список IP-адресов центра обработки данных Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=56519). Список обновляется раз в неделю. IP-адреса, перечисленные в списке центра обработки данных Azure, находятся в нотации CIDR. Дополнительные сведения о CIDR см. [здесь](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Ниже приведены полные доменные имена, используемые шлюзом.

| Доменные имена | Исходящие порты | Описание |
| --- | --- | --- |
| *.powerbi.com |80 |HTTP для скачивания установщика. |
| *.powerbi.com |443 |HTTPS |
| *.analysis.windows.net |443 |HTTPS |
| *. login.windows.net, login.live.com, aadcdn.msauth.net |443 |HTTPS |
| *.servicebus.windows.net |5671–5672 |Протокол AMQP |
| *.servicebus.windows.net |443, 9350–9354 |Прослушиватели ретрансляции служебной шины по протоколу TCP (порт 443 требуется для получения маркера контроля доступа). |
| *.frontend.clouddatahub.net |443 |HTTPS |
| *.core.windows.net |443 |HTTPS |
| login.microsoftonline.com |443 |HTTPS |
| *.msftncsi.com |80 |Используется для проверки подключения к Интернету, если шлюз недоступен для службы Power BI. |
| *.microsoftonline-p.com |443 |Используется для проверки подлинности в зависимости от конфигурации. |
| dc.services.visualstudio.com    |443 |Используется AppInsights для сбора данных телеметрии. |

## <a name="next-steps"></a>Дальнейшие действия 

Следующие статьи включены в общее содержимое локального шлюза данных, которое применяется ко всем службам, которые поддерживает шлюз:

* [Вопросы и ответы о локальном шлюзе данных](/data-integration/gateway/service-gateway-onprem-faq)   
* [Использование приложения локального шлюза данных](/data-integration/gateway/service-gateway-app)   
* [Администрирование на уровне клиента](/data-integration/gateway/service-gateway-tenant-level-admin)
* [Настройка параметров прокси](/data-integration/gateway/service-gateway-proxy)   
* [Настройка параметров связи](/data-integration/gateway/service-gateway-communication)   
* [Настройка файлов журналов](/data-integration/gateway/service-gateway-log-files)   
* [Устранение проблем](/data-integration/gateway/service-gateway-tshoot)
* [Мониторинг и оптимизация производительности шлюза](/data-integration/gateway/service-gateway-performance)
