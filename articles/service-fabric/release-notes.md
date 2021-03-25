---
title: Выпуски Service Fabric Azure
description: Заметки о выпуске Azure Service Fabric. Содержит сведения о последних функциях и улучшениях в Service Fabric.
ms.date: 06/10/2019
ms.topic: conceptual
hide_comments: true
hideEdit: true
ms.openlocfilehash: 7f99c37cd9f029c7478d4102bd17c952ffd52f8c
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105044283"
---
# <a name="service-fabric-releases"></a>Service Fabric выпуски

- <a href="https://github.com/Azure/Service-Fabric-Troubleshooting-Guides" target="blank">Руководства по устранению неполадок</a> 
- <a href="https://github.com/Azure/service-fabric-issues" target="blank">Отслеживание проблем</a> 
- <a href="/azure/service-fabric/service-fabric-support" target="blank">Варианты поддержки</a> 
- <a href="/azure/service-fabric/service-fabric-versions" target="blank">Поддерживаемые версии</a> 
- <a href="https://azure.microsoft.com/resources/samples/?service=service-fabric&sort=0" target="blank">Примеры кода</a>

В этой статье содержатся дополнительные сведения о последних выпусках и обновлениях для среды выполнения Service Fabric и пакетов SDK.

## <a name="service-fabric-72"></a>Service Fabric 7,2

Мы рады сообщить, что выпуск 7,2 среды выполнения Service Fabric был запущен в различных регионах Azure вместе с инструментами и обновлениями пакета SDK. Обновления для пакета SDK для .NET, пакета SDK для Java и Service Fabric среды выполнения доступны через установщик веб-платформы, пакеты NuGet и репозитории Maven.

### <a name="key-announcements"></a>Основные объявления

- **Предварительная версия**: [**Service Fabric управляемые кластеры**](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-managed-clusters-are-now-in-public-preview/ba-p/1721572) теперь доступны в общедоступной предварительной версии. Service Fabric управляемые кластеры предназначены для упрощения развертывания и управления кластером путем инкапсуляции базовых ресурсов, образующих кластер Service Fabric, в один ресурс ARM. Дополнительные сведения см. в разделе [Общие сведения об управляемом кластере Service Fabric](./overview-managed-cluster.md).
- **Предварительная версия**. [**Поддержка служб без отслеживания состояния с количеством экземпляров больше, чем число узлов**](./service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md) , теперь находится в общедоступной предварительной версии. Политика размещения позволяет создавать несколько экземпляров секции без отслеживания состояния на узле.
- [**Фабрикобсервер (FO) 3,0**](https://aka.ms/sf/fabricobserver) теперь доступен.
    - Теперь вы можете запускать Фабрикобсервер в кластерах Linux и Windows.
    - Теперь можно создавать пользовательские подключаемые модули наблюдателя. Дополнительные сведения и код см. в [файле сведений о подключаемых](https://github.com/microsoft/service-fabric-observer/blob/master/Documentation/Plugins.md) [модулях и образце проекта надстройки](https://github.com/microsoft/service-fabric-observer/tree/master/SampleObserverPlugin) .
    - Теперь можно изменить любой параметр наблюдателя через обновление параметров приложения. Это означает, что больше не требуется повторное развертывание для изменения конкретных параметров наблюдателя. Ознакомьтесь с [примером](https://github.com/microsoft/service-fabric-observer/blob/master/Documentation/Using.md#parameterUpdates).
- [**Поддержка образов контейнеров Ubuntu 18,04 OneBox**](https://hub.docker.com/_/microsoft-service-fabric-onebox).
- **Предварительная версия**: [ **Справочник по KeyVault для приложений Service Fabric поддерживает **только секреты с версиями**. Секреты без версий не поддерживаются.**](./service-fabric-keyvault-references.md)
- Пакету SDK для SF требуется последняя версия VS 2019 с обновлением 16.7.6 или 16,8 (Предварительная версия 4) для создания новых платформа .NET Framework проектов без отслеживания состояния и с отслеживанием состояния или субъектов. Если у вас нет последнего обновления VS, после создания проекта службы используйте диспетчер пакетов, чтобы установить Microsoft. ServiceFabric. Services (версии 4.2. x) для проектов с отслеживанием состояния и без отслеживания состояния, а также Microsoft. ServiceFabric. Actors (версия 4.2. x) для проектов субъектов из nuget.org.
- **Рунтокомплетион**: Service Fabric поддерживает концепцию выполнения до завершения для гостевых исполняемых файлов. После выполнения этого обновления после завершения репликации ресурсы кластера, выделенные этой реплике, будут освобождены.
- [**Улучшена поддержка управления ресурсами**](./service-fabric-resource-governance.md): разрешение и ограничение спецификаций для ресурсов ЦП и памяти.

### <a name="service-fabric-72-releases"></a>Выпуски Service Fabric 7,2
| Дата выпуска | Выпуск | Дополнительные сведения |
|---|---|---|
| 21 октября 2020 г. | [Azure Service Fabric 7,2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-2-release/ba-p/1805653)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72-releasenotes.md)|
| 9 ноября 2020 г. | [Обновление второго выпуска Azure Service Fabric 7,2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-2-second-refresh-release/ba-p/1874738) | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU2-releasenotes.md) |
| 10 ноября 2020 г.  | Обновление третьего выпуска Azure Service Fabric 7,2 | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU3-releasenotes.md) |
| 2 декабря 2020 г. | [Обновление четвертого выпуска Azure Service Fabric 7,2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-2-fourth-refresh-release/ba-p/1950584) | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU4.md)
| 25 января 2021 г. | [Azure Service Fabric 7,2 пятое обновление](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-2-fifth-refresh-release/ba-p/2096575) | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU5-ReleaseNotes.md)
| 17 февраля 2021 г. | [Обновление шестого обновления Azure Service Fabric 7,2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-sixth-refresh-release/ba-p/2144685) | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU6-ReleaseNotes.md)
| 10 марта 2021 г. | [Версия обновления 7,2 для Azure Service Fabric седьмая](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-seventh-refresh-release/ba-p/2201100) | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU7-releasenotes.md)

## <a name="previous-versions"></a>предыдущих версий

### <a name="service-fabric-71"></a>Service Fabric 7,1

Из-за текущего сбоя КОВИД-19 и принятия решения о проблемах, с которыми сталкиваются наши клиенты, мы делаем 7,1, но не будут автоматически обновлять кластеры, настроенные для получения автоматических обновлений. Мы приостанавливаете автоматические обновления, пока не заметите, что клиенты могут применять обновления, если они наиболее подходят для них, чтобы избежать непредвиденных нарушений.

Вы сможете выполнить обновление до 7,1 через [портал Azure](./service-fabric-cluster-upgrade-version-azure.md#upgrading-to-a-new-version-on-a-cluster-that-is-set-to-manual-mode-via-portal) или с помощью [Azure Resource Managerного развертывания](./service-fabric-cluster-upgrade-version-azure.md#set-the-upgrade-mode-using-a-resource-manager-template).

Service Fabric кластеры с включенным автоматическим обновлением будут автоматически принимать обновление 7,1 после возобновления стандартной процедуры развертывания. Мы предложим другое объявление до начала выпуска Standard на [веб-сайте технического сообщества Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric).
Мы также опубликовали обновления для основных выпусков, начиная с 6,5 вплоть [до 7,1.](./service-fabric-versions.md#supported-versions) 

#### <a name="key-announcements"></a>Основные объявления

- **Общая доступность** [ **Service Fabric управляемых удостоверений для Service Fabric приложений**](./concepts-managed-identity.md)
- [**Поддержка Ubuntu 18,04**](./service-fabric-tutorial-create-vnet-and-linux-cluster.md)
 - [**Предварительная версия: Поддержка диска временного набора виртуальных машин**](./service-fabric-cluster-azure-deployment-preparation.md#use-ephemeral-os-disks-for-virtual-machine-scale-sets). * *: временные диски ОС — это хранилище, созданное на локальной виртуальной машине и не сохраненное в удаленном хранилище Azure. Они рекомендуются для всех типов узлов Service Fabric (основной и дополнительный), так как по сравнению с традиционными дисками с постоянными ОС, временными дисками ОС:
      -  Уменьшение задержки чтения и записи на диск ОС
      -  Включение более быстрых операций по управлению узлами сброса и повторного создания образа
      -  Сократите общие затраты (диски бесплатно и не требуют дополнительных затрат на хранение).
- Поддержка объявления [**сертификатов конечной точки службы Service Fabric приложений по общему имени субъекта**](./service-fabric-service-manifest-resources.md).
- [**Поддержка проверок работоспособности для контейнерных служб**](./probes-codepackage.md): поддержка механизма проверки жизни для контейнерных приложений. Справка по проверке актуальности сообщает о реальной жизни контейнерного приложения и о том, что они не отвечают вовремя, что приведет к перезапуску. 
- [**Поддержка пакетов кода инициализатора**](./initializer-codepackages.md) для [контейнеров](./service-fabric-containers-overview.md) и [гостевых исполняемых](./service-fabric-guest-executables-introduction.md) приложений. Это позволяет выполнять пакеты кода (например, контейнеры) в указанном порядке для инициализации пакета служб.
- **Фабрикобсервер и клустеробсервер** — это приложения без отслеживания состояния, которые фиксируют Service Fabric телеметрии, связанные с различными аспектами кластера SF. Оба эти приложения готовы к развертыванию в рабочих кластерах Windows для записи многофункциональной телеметрии с реализованной поддержкой ApplicationInsights, EventSource и LogAnalytics.
    - [**Фабрикобсервер (FO) 2,0**](https://github.com/microsoft/service-fabric-observer)— выполняется на всех узлах, создает события работоспособности, передает данные телеметрии при достижении пороговых значений использования ресурсов, настроенных пользователем. Этот выпуск содержит несколько улучшений для мониторинга, управления данными, сведений о событиях работоспособности, структурированных данных телеметрии.
     - [**Клустеробсервер (CO) 1,1**](https://github.com/microsoft/service-fabric-observer/tree/master/ClusterObserver) — выполняется на одном узле, захватывается данные телеметрии работоспособности на уровне кластера. В этом выпуске Клустеробсервер также отслеживает состояние узла и выдает данные телеметрии при отключении или отключении или выключении узла дольше указанного пользователем периода времени.

#### <a name="improve-application-life-cycle-experience"></a>Улучшение взаимодействия с жизненным циклом приложения

- **[Предварительный просмотр: сток запросов](./service-fabric-application-upgrade-advanced.md#avoid-connection-drops-during-stateless-service-planned-downtime)**. во время запланированного обслуживания службы, например обновления службы или деактивации узла, вы хотите разрешить службам корректное завершение подключений. Эта функция добавляет длительность задержки закрытия экземпляра в конфигурации службы. Во время плановых операций SF удалит адрес службы из обнаружения, а затем дождитесь его завершения перед завершением работы службы.
- **[Автоматическое обнаружение и балансировка подкластеров](./cluster-resource-manager-subclustering.md)**. Если службы с разными ограничениями на размещение имеют общую [метрику нагрузки](./service-fabric-cluster-resource-manager-metrics.md), происходит подкластеризация. Если нагрузка на разные наборы узлов существенно различается, кластер Service Fabric диспетчер ресурсов считает, что кластер несбалансирован, даже если он имеет лучшее возможное сальдо из-за ограничений на размещение. В результате он пытается перераспределить кластер, что может привести к ненужным перемещениям служб (поскольку "дисбаланс" не может быть значительно улучшен). Начиная с этого выпуска, диспетчер ресурсов кластера будет пытаться автоматически обнаруживать эти виды конфигураций и понять, когда дисбаланс можно исправить с помощью перемещения, а когда это не так, следует оставить только те вещи, которые не могут быть сделаны значительным улучшением.  
- [**Различные затраты на перемещение вторичных реплик**](./service-fabric-cluster-resource-manager-movement-cost.md): мы добавили новое значение стоимости перемещения верихигх, которое обеспечивает дополнительную гибкость в некоторых сценариях, чтобы определить, следует ли использовать отдельные затраты на перемещение для вторичных реплик.
- Включен механизм [**проверки жизни**](./probes-codepackage.md) для контейнерных приложений. Справка по проверке актуальности сообщает о реальной жизни контейнерного приложения и о том, что они не отвечают вовремя, что приведет к перезапуску.
- [**Выполнение до завершения или один раз для служб**](./run-to-completion.md)**

#### <a name="image-store-improvements"></a>Улучшения Хранилище образов
 - Service Fabric 7,1 использует **Настраиваемый транспорт для защиты передачи файлов между узлами по умолчанию**. Зависимость от файлового ресурса SMB удаляется из версии 7,1. Защищенные файловые ресурсы SMB по-прежнему существуют на узлах, содержащих реплику службы Хранилище образов, чтобы отказаться от установки по умолчанию, а также для обновления и перехода на более старую версию.
       
 #### <a name="reliable-collections-improvements"></a>Усовершенствования надежных коллекций

- [**В памяти поддержка служб с отслеживанием состояния, использующих надежные коллекции**](./service-fabric-work-with-reliable-collections.md#volatile-reliable-collections). постоянные надежные коллекции позволяют сохранять данные на диск для обеспечения устойчивости в случае крупномасштабных простоев, а также могут использоваться для рабочих нагрузок, таких как реплицируемый кэш, например, где возможна некоторая вероятность потери данных. На основе [ограничений и ограничений для временных надежных коллекций](./service-fabric-reliable-services-reliable-collections-guidelines.md#volatile-reliable-collections)рекомендуется использовать этот параметр для рабочих нагрузок, которые не нуждаются в постоянном хранении, для служб, которые обрабатывали редкие случаи потери кворума.
- [**Предварительная версия: Service Fabric обозреватель резервного копирования**](https://github.com/microsoft/service-fabric-backup-explorer). для упрощения управления резервными копиями надежных коллекций для Service Fabric приложений с отслеживанием состояния Service Fabric обозреватель резервного копирования позволяет пользователям
    - Аудит и проверка содержимого надежных коллекций
    - Обновить текущее состояние до постоянного представления
    - Создание резервной копии текущего моментального снимка надежных коллекций
    - Исправление поврежденных данных
                 
#### <a name="service-fabric-71-releases"></a>Выпуски Service Fabric 7,1
| Дата выпуска | Выпуск | Дополнительные сведения |
|---|---|---|
| 20 апреля 2020 г. | [Azure Service Fabric 7,1](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-release/ba-p/1311373)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/tree/master/release_notes/Service-Fabric-71-releasenotes.md)|
| 16 июня 2020 г. | [Microsoft Azure Service Fabric 7,1 Первое обновление](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-first-refresh-release/ba-p/1466517) | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU1-releasenotes.md)
| 20 июля 2020 г. | [Microsoft Azure Service Fabric 7,1, второе обновление](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-second-refresh-release/ba-p/1534246) | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU2-releasenotes.md)
| 12 августа 2020 г. | [Третье обновление Microsoft Azure Service Fabric 7,1](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-third-refresh-release/ba-p/1587586) | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU3-releasenotes.md)
| 10 сентября 2020 г. | [Четвертое обновление Microsoft Azure Service Fabric 7,1](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-fourth-refresh-release/ba-p/1653859)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU5-releasenotes.md)|
| 7 октября 2020 г. | Шестое обновление Microsoft Azure Service Fabric 7,1 | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU6-releasenotes.md)|
| 23 ноября 2020 г. | Microsoft Azure Service Fabric 7,1, восьмое обновление | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU8-releasenotes.md)|


### <a name="service-fabric-70"></a>Service Fabric 7,0

Azure Service Fabric 7,0 теперь доступна! Вы сможете выполнить обновление до 7,0 с помощью портал Azure или с помощью Azure Resource Managerного развертывания. Из-за отзывов клиентов о выпусках по истечении праздничного периода мы не будем начинать автоматическое обновление кластеров для получения автоматических обновлений до января.
В январе мы возбудем возобновить стандартную процедуру развертывания, и кластеры с включенным автоматическим обновлением начнут автоматически принимать обновление 7,0. Перед началом развертывания мы предложим другое объявление.
Мы также будем обновлять наши запланированные даты выпуска, чтобы указать, что мы рассматриваем эту политику. Здесь можно найти обновления для будущих [расписаний выпуска](https://github.com/Microsoft/service-fabric/#service-fabric-release-schedule).

#### <a name="key-announcements"></a>Основные объявления
 - [**Поддержка кэйваултреференце для секретов приложений (Предварительная версия)**](./service-fabric-keyvault-references.md): Service Fabric приложения с включенными [управляемыми удостоверениями](./concepts-managed-identity.md) теперь могут напрямую ссылаться на Key Vault секретный URL-адрес в качестве переменной среды, параметра приложения или учетных данных репозитория контейнеров. Service Fabric будет автоматически разрешать секрет с помощью управляемого удостоверения приложения. 
     
- **Улучшенная безопасность обновления для служб без отслеживания состояния**. чтобы обеспечить доступность во время обновления приложения, мы предоставили новые конфигурации для определения [минимального количества экземпляров для служб без отслеживания состояния](/dotnet/api/system.fabric.description.statelessservicedescription) , которые будут считаться доступными. Ранее это значение было равно 1 для всех служб и не было изменено. Благодаря этой новой проверке безопасности для каждой службы можно гарантировать, что службы будут хранить минимальное количество экземпляров во время обновления приложения, обновления кластера и прочее обслуживание, основанное на проверках работоспособности Service Fabric и безопасности.
  
- [**Ограничения ресурсов для пользовательских служб**](./service-fabric-resource-governance.md#enforcing-the-resource-limits-for-user-services). пользователи могут настроить ограничения ресурсов для пользовательских служб на узле, чтобы предотвратить такие сценарии, как исчерпание ресурсов Service Fabric системных служб. 
  
- [**Очень высокая стоимость перемещения служб**](./service-fabric-cluster-resource-manager-movement-cost.md) для типа реплики. Реплики с очень высокими затратами на перемещение будут перемещены только в случае нарушения ограничения в кластере, которое не может быть исправлено каким-либо другим способом. Дополнительные сведения об использовании "очень высоких" затрат на перемещение см. в связанном документе.
  
-  **Дополнительные проверки безопасности кластера**. в этом выпуске мы предоставили настраиваемую проверку безопасности кворума узла начального значения. Это позволяет настроить количество узлов начальных значений, которые должны быть доступны во время жизненного цикла кластера и сценариев управления. Операции, которые принимают кластер под заданным значением, блокируются. Сегодня значение по умолчанию всегда является кворумом узлов начальных значений. Например, если у вас 7 начальных узлов, то операция, которая помогла бы перейти на 5 начальных узлов, будет заблокирована по умолчанию. С этим изменением можно было бы сделать минимальное опасное значение 6, которое позволяло только одному узлу начальных значений одновременно.
   
- Добавлена поддержка [**управления службой резервного копирования и восстановления в Service Fabric Explorer**](./service-fabric-backuprestoreservice-quickstart-azurecluster.md). Это делает возможными следующие действия непосредственно в SFX: обнаружение службы резервного копирования и восстановления, создание политики резервного копирования, включение автоматического резервного копирования, получение нерегламентированных резервных копий, активация операций восстановления и Просмотр существующих резервных копий.

- Объявление о доступности [**релиаблеколлектионсмиссингтипестул**](https://github.com/hiadusum/ReliableCollectionsMissingTypesTool): это средство помогает проверить, что типы, используемые в надежных коллекциях, поддерживают прямую и обратную совместимость во время последовательного обновления приложения. Это помогает предотвратить сбои обновления, потерю данных и повреждение данных из-за отсутствия или несовместимости типов.

- [**Включить стабильные операции чтения на вторичных репликах. при**](./service-fabric-reliable-services-configuration.md#configuration-names-1)стабильном чтении вторичные реплики будут ограничены возвратом значений, которые были подтверждено) кворума.

Кроме того, этот выпуск содержит другие новые функции, исправления ошибок, а также улучшения поддержки, надежности и производительности. Полный список изменений см. в [заметках о выпуске](https://github.com/Azure/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_70.md).

#### <a name="service-fabric-70-releases"></a>Выпуски Service Fabric 7,0

| Дата выпуска | Выпуск | Дополнительные сведения |
|---|---|---|
| 18 ноября 2019 г. | [Azure Service Fabric 7,0](https://techcommunity.microsoft.com/t5/Azure-Service-Fabric/Service-Fabric-7-0-Release/ba-p/1015482)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_70.md)|
| 30 января 2020 г. | [Обновление выпуска Azure Service Fabric 7,0](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-0-second-refresh-release/ba-p/1137690)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU2-releasenotes.md)|
| 6 февраля 2020 г. | [Обновление выпуска Azure Service Fabric 7,0](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-0-third-refresh-release/ba-p/1156508)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU3-releasenotes.md)|
| 2 марта 2020 г. | [Обновление выпуска Azure Service Fabric 7,0](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-0-fourth-refresh-release/ba-p/1205414)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU4-releasenotes.md)
| 6 мая, 2020 | [Обновление шестого обновления Azure Service Fabric 7,0](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-0-sixth-refresh-release/ba-p/1365709) | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU6-releasenotes.md)|
| 9 октября 2020 г. | Обновление выпуска Azure Service Fabric 7,0 девятый | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU9-releasenotes.md)|

### <a name="service-fabric-65"></a>Service Fabric 6,5

Этот выпуск включает в себя поддержку, надежность и улучшения производительности, новые функции, исправления ошибок и усовершенствования для упрощения управления жизненным циклом кластера и приложений.

> [!IMPORTANT]
> Service Fabric 6,5 — это окончательный выпуск с поддержкой средств Service Fabric в Visual Studio 2015. Пользователям рекомендуется перейти на [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) в будущем.

Новые возможности Service Fabric 6,5:

- Service Fabric Explorer включает [средство просмотра хранилище образов](service-fabric-visualizing-your-cluster.md#image-store-viewer) для проверки приложений, отправленных в хранилище образов.

- [Приложение для управления исправлениями (вех)](service-fabric-patch-orchestration-application.md) версии [1.4.0](https://github.com/microsoft/Service-Fabric-POA/releases/tag/v1.4.0) включает в себя множество улучшений самодиагностики. Клиентам ВЕХ рекомендуется перейти на эту версию.

- [Служба евентсторе по умолчанию включена](service-fabric-visualizing-your-cluster.md#event-store) для кластеров Service Fabric 6,5, если вы не выбрали.

- Добавлены [события жизненного цикла реплики](service-fabric-diagnostics-event-generation-operational.md#replica-events) для служб с отслеживанием состояния.

- [Улучшенная видимость состояния узла начальных значений](service-fabric-understand-and-troubleshoot-with-system-health-reports.md#seed-node-status), включая предупреждения на уровне кластера, если начальный узел неработоспособен(отключен *или* *неизвестен*).

- [Service Fabric средство аварийного восстановления приложений](https://github.com/Microsoft/Service-Fabric-AppDRTool) позволяет Service Fabric службам с отслеживанием состояния быстро восстанавливаться при аварии основного кластера. Данные из основного кластера непрерывно синхронизируются в резервном приложении-получателе с помощью периодического резервного копирования и восстановления.

- Поддержка [публикации приложений .NET Core в кластерах под управлением Linux в](service-fabric-how-to-publish-linux-app-vs.md)Visual Studio.

- [Azure Service Fabric CLI (SFCTL)](./service-fabric-cli.md) будет автоматически устанавливаться для Service Fabric 6,5 (и более поздних версий) при обновлении или создании нового кластера Linux в Azure.

- [SFCTL](./service-fabric-cli.md) устанавливается по умолчанию в кластерах MacOS/Linux OneBox.

Дополнительные сведения см. в [заметках о Выпуске Service Fabric 6,5](https://github.com/Azure/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65.pdf).

#### <a name="service-fabric-65-releases"></a>Выпуски Service Fabric 6,5

| Дата выпуска | Выпуск | Дополнительные сведения |
|---|---|---|
| 11 июня 2019 г. | [Azure Service Fabric 6,5](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65.pdf)|
| 2 июля 2019 г. | [Обновление выпуска Azure Service Fabric 6,5](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65CU1.pdf)  |
| 29 июля 2019 г. | [Обновление выпуска Azure Service Fabric 6,5](https://techcommunity.microsoft.com/t5/Azure-Service-Fabric/Azure-Service-Fabric-6-5-Second-Refresh-Release/ba-p/800523)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65CU2.pdf)  |
| Авг 23, 2019 | [Обновление выпуска Azure Service Fabric 6,5](https://techcommunity.microsoft.com/t5/Azure-Service-Fabric/Azure-Service-Fabric-6-5-Third-Refresh-Release/ba-p/818599)  | [Заметки о выпуске](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65CU3.pdf)  |
| 14 октября 2019 г. | [Обновление выпуска Azure Service Fabric 6,5](https://techcommunity.microsoft.com/t5/Azure-Service-Fabric/Azure-Service-Fabric-6-5-Fifth-Refresh-Release/ba-p/913296)  | [Заметки о выпуске] (https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65CU5.md  |


### <a name="service-fabric-64-releases"></a>Выпуски Service Fabric 6,4

| Дата выпуска | Выпуск | Дополнительные сведения |
|---|---|---|
| 30 ноября 2018 г. | [Azure Service Fabric 6,4](https://blogs.msdn.microsoft.com/azureservicefabric/2018/11/30/azure-service-fabric-6-4-release/)  | [Заметки о выпуске](https://msdnshared.blob.core.windows.net/media/2018/12/Service-Fabric-6.4-Release.pdf)|
| 12 декабря 2018 г. | [Обновление выпуска Azure Service Fabric 6,4 для кластеров Windows](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric)  | [Заметки о выпуске](https://msdnshared.blob.core.windows.net/media/2018/12/Links.pdf)  |
| 4 февраля 2019 г. | [Обновление выпуска Azure Service Fabric 6,4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) | [Заметки о выпуске](https://msdnshared.blob.core.windows.net/media/2019/02/Service-Fabric-6.4CU3-Release-Notes.pdf) |
| 4 марта 2019 г. | [Обновление выпуска Azure Service Fabric 6,4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) | [Заметки о выпуске](https://msdnshared.blob.core.windows.net/media/2019/03/Service-Fabric-6.4CU4-Release-Notes.pdf)
| 8 апреля 2019 г. | [Обновление выпуска Azure Service Fabric 6,4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) | [Заметки о выпуске](https://msdnshared.blob.core.windows.net/media/2019/04/Service-Fabric-6.4CU5-ReleaseNotes3.pdf)
| 2 мая 2019 г. | [Обновление выпуска Azure Service Fabric 6,4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) | [Заметки о выпуске](https://msdnshared.blob.core.windows.net/media/2019/05/Service-Fabric-64CU6-Release-Notes-V2.pdf)
| 28 мая 2019 г. | [Обновление выпуска Azure Service Fabric 6,4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) | [Заметки о выпуске](https://msdnshared.blob.core.windows.net/media/2019/05/Service_Fabric_64CU7_Release_Notes1.pdf)