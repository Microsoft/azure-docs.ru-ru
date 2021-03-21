---
title: Часто задаваемые вопросы о Azure Веснного облака | Документация Майкрософт
description: В этой статье содержатся ответы на часто задаваемые вопросы о Microsoft Azure Веснного облака.
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: brendm
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 50c8a36b943abb24f2c31bfc2bc33e646a74524a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100596612"
---
# <a name="azure-spring-cloud-faq"></a>Вопросы и ответы по Azure весн Cloud

В этой статье содержатся ответы на часто задаваемые вопросы о Microsoft Azure Веснного облака.

## <a name="general"></a>Общие сведения

### <a name="why-azure-spring-cloud"></a>Вопросы и ответы по Azure Spring Cloud.

Azure Веснного облака предоставляет платформу как услугу (PaaS) для разработчиков весны облака. Azure Веснного облака управляет инфраструктурой приложений, чтобы вы могли сосредоточиться на коде приложения и бизнес-логике. К основным функциям, встроенным в Azure "Весна Cloud", относятся Еурека, сервер конфигурации, сервер реестра служб, Служба построения сводных служб, сине-зеленое развертывание и многое другое. Эта служба также позволяет разработчикам привязывать свои приложения к другим службам Azure, таким как Azure Cosmos DB, база данных Azure для MySQL и кэш Azure для Redis.

Azure Веснного облака расширяет возможности диагностики приложений для разработчиков и операторов, интегрируя Azure Monitor, Application Insights и Log Analytics.

### <a name="how-secure-is-azure-spring-cloud"></a>Насколько безопасным является облако Azure Spring Cloud?

Безопасность и конфиденциальность являются важнейшими приоритетами для клиентов Azure и весны в облаке Azure. Azure гарантирует, что только у клиентов есть доступ к данным приложения, журналам или конфигурациям, безопасно шифруя все эти данные. 

* Экземпляры службы в Azure Веснного облака изолированы друг от друга.
* Azure Веснного облака предоставляет полный TLS/SSL и управление сертификатами.
* Критические исправления безопасности для сред выполнения Azure Spring Cloud и OpenJDK применяются как можно скорее в Azure Spring Cloud.

### <a name="in-which-regions-is-azure-spring-cloud-available"></a>В каких регионах доступно облако Azure весны?

Восточная часть США, Восточная часть США 2, Центральная часть США, Юго-Центральный регион США, северо-центральная часть US, Западная часть США, Запад 2, Западная Европа, Северная Европа, южная часть Соединенного Королевства, Юго-Восточная Азия, Восточная Австралия, Центральная Канада, Северная ОАЭ, Центральная Индия, Центральная Корея, Восточная Азия и Восточный Китай 2 (Mooncake). [Подробнее](https://azure.microsoft.com/global-infrastructure/services/?products=spring-cloud)

### <a name="is-any-customer-data-stored-outside-of-the-specified-region"></a>Хранятся ли данные клиента за пределами указанного региона?

Azure Веснного облака — это региональная служба. Все клиентские данные в Azure Веснного облака хранятся в нескольких регионах в пределах одного географического региона для обеспечения избыточности. Дополнительные сведения об Geo и регионе см. [в статье местонахождение данных в Azure](https://azure.microsoft.com/global-infrastructure/data-residency/).

### <a name="what-are-the-known-limitations-of-azure-spring-cloud"></a>Каковы известные ограничения Azure Веснного облака?

Azure Веснного облака имеет следующие известные ограничения.
    
* `spring.application.name` будет переопределено именем приложения, которое используется для создания каждого приложения.
* `server.port` по умолчанию используется порт 1025. Если применяется любое другое значение, оно будет переопределено. Также следует учитывать этот параметр и не указывать порт сервера в коде.
* Шаблоны портал Azure и Azure Resource Manager не поддерживают отправку пакетов приложений. Пакеты приложений можно отправлять только путем развертывания приложения с помощью Azure CLI.

### <a name="what-pricing-tiers-are-available"></a>Какие ценовые категории доступны? 
Какой из них следует использовать и каковы ограничения в пределах каждого уровня?
* Azure Веснного облака предлагает две ценовые категории: "базовый" и "Стандартный". Уровень "базовый" предназначен для разработки и тестирования, а также для пробного облака Azure. Уровень "Стандартный" оптимизирован для выполнения рабочего трафика общего назначения. Ограничения и сравнение на уровне функций см. в статье [сведения о ценах на облачную службу Azure](https://azure.microsoft.com/pricing/details/spring-cloud/) .

### <a name="how-can-i-provide-feedback-and-report-issues"></a>Как можно отправить отзыв и сообщить о проблемах?

Если вы столкнулись с проблемами с облаком Azure весны, создайте [запрос в службу поддержки Azure](../azure-portal/supportability/how-to-create-azure-support-request.md). Чтобы отправить запрос на функцию или оставить отзыв, перейдите в службу [отзывов Azure](https://feedback.azure.com/forums/34192--general-feedback).

## <a name="development"></a>Разработка

### <a name="i-am-a-spring-cloud-developer-but-new-to-azure-what-is-the-quickest-way-for-me-to-learn-how-to-develop-an-azure-spring-cloud-application"></a>Я являюсь пружинным разработчиком облачных решений, но не впервые работаю в Azure. Каков самый быстрый способ научиться разрабатывать облачные приложения Azure для весны?

Чтобы быстро приступить к работе с Azure "пружинное облако", следуйте инструкциям в [кратком руководстве по запуску облачного приложения Azure весны с помощью портал Azure](spring-cloud-quickstart.md).

::: zone pivot="programming-language-java"
### <a name="what-java-runtime-does-azure-spring-cloud-support"></a>Какие среды выполнения Java поддерживает Azure Spring Cloud?

Azure Spring Cloud поддерживает Java 8 и 11. См. раздел [Среда выполнения Java и версии ОС](#java-runtime-and-os-versions) .

### <a name="is-spring-boot-24x-supported"></a>Поддерживается ли пружинная Загрузка 2.4. x?
Мы обнаружили проблему с пружинной загрузкой 2,4 и сейчас работаем над ее устранением. В то же время включите эти две зависимости, чтобы включить проверку подлинности TLS между приложениями и Еурека.

```xml
<dependency> 
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-client</artifactId>
    <version>1.19.4</version>
</dependency>
<dependency>
    <groupId>com.sun.jersey.contribs</groupId>
    <artifactId>jersey-apache-client4</artifactId>
    <version>1.19.4</version>
</dependency>
```

::: zone-end

### <a name="where-can-i-view-my-spring-cloud-application-logs-and-metrics"></a>Где можно просматривать журналы и метрики приложений весны облака?

Метрики находятся на вкладке "Обзор приложения" и [Azure Monitor](../azure-monitor/essentials/data-platform-metrics.md#metrics-explorer).

Azure Веснного облака поддерживает экспорт журналов облачных приложений и метрик в службу хранилища Azure, EventHub и [log Analytics](../azure-monitor/logs/data-platform-logs.md). Имя таблицы в Log Analytics — *аппплатформлогсфорспринг*. Сведения о том, как ее включить, см. в разделе [службы диагностики](diagnostic-services.md).

### <a name="does-azure-spring-cloud-support-distributed-tracing"></a>Поддерживает ли Azure Spring Cloud распределенную трассировку?

Да. Дополнительные сведения см. в разделе [учебник. Использование распределенной трассировки с помощью Azure веснного облака](spring-cloud-tutorial-distributed-tracing.md).

::: zone pivot="programming-language-java"
### <a name="what-resource-types-does-service-binding-support"></a>Какие типы ресурсов поддерживает привязка к службам?

В настоящее время поддерживаются три службы:
* Azure Cosmos DB
* База данных Azure для MySQL
* Кэш Redis для Azure.
::: zone-end

### <a name="can-i-view-add-or-move-persistent-volumes-from-inside-my-applications"></a>Можно ли просматривать, добавлять или перемещать постоянные тома в приложениях?

Да.

### <a name="how-many-outbound-public-ip-addresses-does-an-azure-spring-cloud-instance-have"></a>Сколько исходящих общедоступных IP-адресов использует экземпляр облака Azure весны?

Количество исходящих общедоступных IP-адресов может отличаться в зависимости от уровней и других факторов. 

| Тип экземпляра Azure весны облака | Число исходящих общедоступных IP-адресов по умолчанию |
| -------------------------------- | ---------------------------------------------- |
| Экземпляры уровня Basic             | 1                                              |
| Экземпляры уровня "Стандартный"          | 2                                              |
| Экземпляры внедрения виртуальной сети         | 1                                              |


### <a name="can-i-increase-the-number-of-outbound-public-ip-addresses"></a>Можно ли увеличить число исходящих общедоступных IP-адресов?

Да, вы можете отправить запрос в [службу поддержки](https://azure.microsoft.com/support/faq/)  , чтобы запросить дополнительные исходящие общедоступные IP-адреса.

### <a name="when-i-deletemove-an-azure-spring-cloud-service-instance-will-its-extension-resources-be-deletedmoved-as-well"></a>При удалении или перемещении экземпляра облачной службы Azure весны ресурсы расширения будут удалены и перемещены?

Это зависит от логики поставщиков ресурсов, владеющих ресурсами расширения. Ресурсы расширения `Microsoft.AppPlatform` экземпляра не принадлежат одному пространству имен, поэтому поведение зависит от поставщика ресурсов. Например, операция удаления или перемещения не будет каскадной для ресурсов **параметров диагностики** . Если новый экземпляр облачной службы Azure весны подготавливается с тем же ИДЕНТИФИКАТОРом ресурса, что и у удаленного, или если предыдущий экземпляр облачного облака Azure перемещается обратно, предыдущие ресурсы **параметров диагностики** продолжат его расширение.

Вы можете удалить параметры диагностики пружины облака с помощью Azure CLI:

```azurecli
 az monitor diagnostic-settings delete --name $diagnosticSettingName --resource $azureSpringCloudResourceId
```

::: zone pivot="programming-language-java"
## <a name="java-runtime-and-os-versions"></a>Среда выполнения Java и версии ОС

### <a name="which-versions-of-java-runtime-are-supported-in-azure-spring-cloud"></a>Какие версии среды выполнения Java поддерживаются в Azure Веснного облака?

Azure Веснное облако поддерживает версии Java LTS с самыми последними сборками, в настоящее время 2020 июня, Java 8 и Java 11. См. статью [Установка JDK для Azure и Azure Stack](/azure/developer/java/fundamentals/java-jdk-install)

### <a name="who-built-these-java-runtimes"></a>Кто создал эти среды выполнения Java?

Azul системы. Сборки JDK корпоративного выпуска Azul Zulu для Azure — это бесплатный мультиплатформенный готовый к использованию дистрибутив OpenJDK для Azure и Azure Stack, который поддерживается корпорацией Майкрософт и Azul Systems. Они содержат все компоненты для сборки и запуска приложений Java SE.

### <a name="how-often-will-java-runtimes-get-updated"></a>Как часто будут обновляться среды выполнения Java?

Для LTS- и MTS-выпусков JDK выходят ежеквартальные обновления системы безопасности и исправления ошибок, а также критические внеплановые обновления и исправления по мере необходимости. Эта поддержка предусматривает обратный перенос в Java 7 и 8 обновлений безопасности и исправления ошибок, обнаруженных в более новых версиях Java, как, например, Java 11.

### <a name="how-long-will-java-8-and-java-11-lts-versions-be-supported"></a>Как долго будут поддерживаться версии Java 8 и Java 11 LTS?

См. [Дополнительные сведения о долгосрочной поддержке Java для Azure и Azure Stack](/azure/developer/java/fundamentals/java-jdk-long-term-support).

* Поддержка Java 8 LTS будет поддерживаться до декабря 2030.
* Поддержка Java 11 LTS будет поддерживаться до 2027 сентября.

### <a name="how-can-i-download-a-supported-java-runtime-for-local-development"></a>Как загрузить поддерживаемую среду выполнения Java для локальной разработки?

См. статью [Установка JDK для Azure и Azure Stack](/azure/developer/java/fundamentals/java-jdk-install).

### <a name="what-is-the-retire-policy-for-older-java-runtimes"></a>Какова политика снятия с учета для старых сред выполнения Java?

Общедоступное уведомление будет отправлено через 12 месяцев до прекращения использования любой старой версии среды выполнения. Для перехода на более позднюю версию вы получите 12 месяцев.

* Администраторы подписки получат уведомление по электронной почте о том, что версия Java будет прекращена.
* Сведения об отмене будут опубликованы в документации.

### <a name="how-can-i-get-support-for-issues-at-the-java-runtime-level"></a>Как можно получить поддержку проблем на уровне среды выполнения Java?

Вы можете открыть запрос в службу поддержки Azure.  См. статью [о создании запроса на поддержку Azure](../azure-portal/supportability/how-to-create-azure-support-request.md).

### <a name="what-is-the-operation-system-to-run-my-apps"></a>Что такое система эксплуатации для запуска приложений?

Используется самая последняя версия Ubuntu LTS, в настоящее время [Ubuntu 20,04 LTS (фокус Фосса)](https://releases.ubuntu.com/focal/) — это ОС по умолчанию.

### <a name="how-often-are-os-security-patches-applied"></a>Как часто применяются исправления безопасности ОС?

Исправления безопасности, применимые к облаку Azure для весны, выводятся на условиях ежемесячного развертывания.
Критические исправления безопасности (CVE Score >= 9), применимые к облаку Azure весны, будут как можно быстрее.
::: zone-end

## <a name="deployment"></a>Deployment (Развертывание)

### <a name="does-azure-spring-cloud-support-blue-green-deployment"></a>Поддерживает ли Azure Веснное облако развертывание со синим зеленым цветом?
Да. Дополнительные сведения см. в разделе [Настройка промежуточной среды](spring-cloud-howto-staging-environment.md).

### <a name="can-i-access-kubernetes-to-manipulate-my-application-containers"></a>Можно ли получить доступ к Kubernetes для управления контейнерами приложений?

Нет.  Azure Spring Cloud абстрагирует разработчика от базовой архитектуры, позволяя сосредоточиться на коде приложения и бизнес-логике.

### <a name="does-azure-spring-cloud-support-building-containers-from-source"></a>Поддерживает ли Azure Spring Cloud создание контейнеров из источника?

Да. Дополнительные сведения см. [в статье запуск приложения "Весна Cloud" из исходного кода](spring-cloud-quickstart.md).

### <a name="does-azure-spring-cloud-support-autoscaling-in-app-instances"></a>Поддерживает ли Azure Spring Cloud автоматическое масштабирование в экземплярах приложений?

Да.  Дополнительные сведения см. в разделе [Setup Автомасштабирование](spring-cloud-tutorial-setup-autoscale.md).

::: zone pivot="programming-language-java"
### <a name="what-are-the-best-practices-for-migrating-existing-spring-cloud-microservices-to-azure-spring-cloud"></a>Каковы рекомендации по переносу существующих микрослужб с высокой задолженностью в облако Azure весны?

При переносе существующих микрослужб облачного облака в Azure Веснного облака рекомендуется следовать приведенным ниже рекомендациям.
* нужно разрешить все зависимости приложений.
* Подготовьте записи конфигурации, переменные среды и параметры ВИРТУАЛЬНОЙ машины Java, чтобы их можно было сравнить с развертыванием в Azure Веснного облака.
* Если вы хотите использовать привязку службы, перейдите к службам Azure и убедитесь, что установлены соответствующие права доступа.
* Рекомендуется удалить или отключить все внедренные службы, которые могут конфликтовать со службами, управляемыми Azure Веснным облаком, такими как служба обнаружения служб, сервер конфигурации и т. д.
* Рекомендуется использовать официальные и стабильные сводные библиотеки. Неофициальная, бета-версия или разветвленная версия низкоуровневых библиотек не поддерживает соглашение об уровне обслуживания (SLA).

После миграции Отслеживайте метрики ЦП и ОЗУ и сетевой трафик, чтобы убедиться, что экземпляры приложения масштабируются соответствующим образом.
::: zone-end

::: zone pivot="programming-language-csharp"
## <a name="net-core-versions"></a>Версии .NET Core

### <a name="which-net-core-versions-are-supported"></a>Какие версии .NET Core поддерживаются?

.NET Core 3,1 и более поздние версии.

### <a name="how-long-will-net-core-31-be-supported"></a>Как долго будет поддерживаться .NET Core 3,1?

До 3 декабря 2022 г. См. раздел [Политика поддержки .NET Core](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).
::: zone-end


## <a name="troubleshooting"></a>Устранение неполадок

### <a name="what-are-the-impacts-of-service-registry-rarely-unavailable"></a>Каковы последствия редкого недоступности реестра службы?

В нередком случае могут возникнуть некоторые ошибки, например 
```
RetryableEurekaHttpClient: Request execution failure with status code 401; retrying on another server if available
```
из журналов приложений. Эта проблема появилась в пружинной инфраструктуре с очень низкой скоростью из-за нестабильной работы сети или других сетевых проблем. 

Это не должно влиять на взаимодействие с пользователем. у клиента Еурека есть как пульс, так и политика повтора, чтобы позаботиться об этом. Его можно рассматривать как одну временную ошибку и спокойно пропускать.

Мы улучшаем эту часть и не будем использовать эту ошибку в приложениях пользователей в течение короткого будущего времени.


## <a name="next-steps"></a>Дальнейшие действия

Если у вас возникнут другие вопросы, см. [руководство по устранению неполадок в облаке Azure весны](spring-cloud-troubleshoot.md).
