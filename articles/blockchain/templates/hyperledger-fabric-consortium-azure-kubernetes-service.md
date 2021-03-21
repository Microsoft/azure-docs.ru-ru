---
title: Развертывание в службе Azure Kubernetes Service Fabric Consortium
description: Развертывание и настройка сети консорциума Kubernetes для структуры в службе Azure
ms.date: 03/01/2021
ms.topic: how-to
ms.reviewer: ravastra
ms.custom: contperf-fy21q3
ms.openlocfilehash: 42d16adbc5e6396c8d5d38176ac7681c712f4555
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102101109"
---
# <a name="deploy-hyperledger-fabric-consortium-on-azure-kubernetes-service"></a>Развертывание в службе Azure Kubernetes Service Fabric Consortium

Для развертывания и настройки сети "консорциум по структуре" в Azure можно использовать шаблон "структура" в Azure Kubernetes Service (AKS).

После прочтения этой статьи вы узнаете:

- Получите опыт работы с этой структурой и компонентами, которые формируют стандартные блоки блокчейн сети.
- Узнайте, как развернуть и настроить сеть консорциума Kubernetes в службе Azure для рабочих сценариев.

[!INCLUDE [Preview note](./includes/preview.md)]

## <a name="choose-an-azure-blockchain-solution"></a>Выберите решение Azure Блокчейн

Прежде чем использовать шаблон решения, Сравните свой сценарий с общими вариантами использования доступных параметров Azure Блокчейн:

Параметр | Модель службы | Типичный сценарий использования
-------|---------------|-----------------
Шаблоны решений | IaaS | Шаблоны решений — это Azure Resource Manager шаблоны, которые можно использовать для предоставления полностью настроенной топологии сети блокчейн. Шаблоны развертывают и настраивают Microsoft Azure службы вычислений, сети и хранения для типа сети блокчейн. Шаблоны решений предоставляются без соглашения об уровне обслуживания. Для поддержки используйте [Microsoft Q&страницу](/answers/topics/azure-blockchain-workbench.html) .
[Служба "Блокчейн Azure"](../service/overview.md) | PaaS | Предварительная версия Azure Блокчейн Service позволяет упростить образование, управление и руководство по сетям блокчейн в консорциуме W3C. Используйте службу Блокчейн Azure для решений, которым требуется служба PaaS, управления консорциумом, а также конфиденциальность контрактов и транзакций.
[Azure Blockchain Workbench](../workbench/overview.md) | IaaS и PaaS | Azure Блокчейн Workbench Preview — это набор служб и возможностей Azure, которые помогают создавать и развертывать приложения блокчейн для совместного использования бизнес-процессов и данных с другими организациями. Используйте Azure Блокчейн Workbench для создания прототипа решения блокчейн или подтверждения концепции для приложения блокчейн. Azure Блокчейн Workbench предоставляется без соглашения об уровне обслуживания. Для поддержки используйте [Microsoft Q&страницу](/answers/topics/azure-blockchain-workbench.html) .

## <a name="deploy-the-orderer-and-peer-organization"></a>Развертывание заказа и одноранговой Организации

Чтобы приступить к работе, вам понадобится подписка Azure, которая может поддерживать развертывание нескольких виртуальных машин и стандартные учетные записи хранения. Если у вас нет подписки Azure, вы можете [создать бесплатную учетную запись Azure](https://azure.microsoft.com/free/).

Чтобы приступить к развертыванию сетевых компонентов структуры, перейдите на [портал Azure](https://portal.azure.com).

1. Выберите **создать ресурс**  >  **блокчейн**, а затем выполните поиск по **структуре "книга" в службе Kubernetes Azure (Предварительная версия)**.

2. Введите сведения о проекте на вкладке " **основы** ".

    ![Снимок экрана, на котором показана вкладка "основные".](./media/hyperledger-fabric-consortium-azure-kubernetes-service/create-for-hyperledger-fabric-basics.png)

3. Введите следующие сведения:
    - **Подписка**: выберите имя подписки, в которой необходимо развернуть сетевые компоненты структуры «книга».
    - **Группа ресурсов**. Создайте новую группу ресурсов или выберите существующую пустую группу ресурсов. Группа ресурсов будет содержать все ресурсы, развернутые в составе шаблона.
    - **Регион**: Выберите регион Azure, в котором вы хотите развернуть кластер службы Azure Kubernetes для компонентов структуры компонента "структура". Шаблон доступен во всех регионах, где доступна AKS. Выберите регион, в котором ваша подписка не достигает предельной квоты виртуальной машины (ВМ).
    - **Префикс ресурса**: введите префикс для именования развертываемых ресурсов. Его длина должна быть меньше шести символов, а сочетание символов должно содержать как цифры, так и буквы.
4. Перейдите на вкладку **Параметры структуры** , чтобы определить сетевые компоненты структуры «книга», которые будут развернуты.

    ![Снимок экрана, на котором показана вкладка "Параметры структуры".](./media/hyperledger-fabric-consortium-azure-kubernetes-service/create-for-hyperledger-fabric-settings.png)

5. Введите следующие сведения:
    - **Название организации**: введите имя Организации, которая необходима для различных операций с плоскостью данных. Имя Организации должно быть уникальным для каждого развертывания.
    - **Компонент Fabric Network**: выберите **службу упорядочения** или **одноранговые узлы** на основе сетевого компонента блокчейн, который требуется настроить.
    - **Число узлов**. ниже приведены два типа узлов.
        - **Служба упорядочения**: узлы, ответственные за упорядочение транзакций в главной книге. Выберите число узлов, чтобы обеспечить отказоустойчивость сети. Поддерживаемое количество узлов заказа — 3, 5 и 7.
        - **Одноранговые узлы**: узлы, на которых размещаются книги учета и интеллектуальные контракты. В зависимости от требований можно выбрать от 1 до 10 узлов.
    - **База данных состояния мира однорангового узла**: базы данных состояния мира для одноранговых узлов. Левелдб — это база данных состояния по умолчанию, внедренная в одноранговый узел. Данные чаинкоде хранятся в виде простых пар "ключ-значение" и поддерживают только запросы ключей, диапазонов ключей и составных ключевых запросов. CouchDB — это необязательная альтернативная база данных состояний, которая поддерживает расширенные запросы, когда чаинкоде значения данных моделируются как JSON. Это поле отображается при выборе **одноранговых узлов** в раскрывающемся списке **сетевой компонент структуры** .
    - **Имя пользователя ЦС структуры**. центр сертификатов структуры позволяет инициализировать и запустить серверный процесс, на котором размещается центр сертификации. Она позволяет управлять удостоверениями и сертификатами. Каждый кластер AKS, развернутый как часть шаблона, по умолчанию будет иметь модуль ЦС Fabric. Введите имя пользователя, используемое для проверки подлинности ЦС структуры.
    - **Пароль ЦС структуры**. Введите пароль для проверки подлинности ЦС структуры.
    - **Подтверждение пароля**: Подтвердите пароль ЦС структуры.
    - **Сертификаты**: Если вы хотите использовать собственные корневые сертификаты для инициализации центра сертификации Fabric, выберите параметр **Отправить корневой сертификат для центра сертификации** . В противном случае ЦС структуры по умолчанию создает самозаверяющие сертификаты.
    - **Корневой сертификат**: Отправьте корневой сертификат (открытый ключ), с помощью которого необходимо инициализировать ЦС Fabric. Поддерживаются сертификаты формата PEM. Сертификаты должны быть допустимыми и находиться в часовом поясе UTC.
    - **Закрытый ключ корневого сертификата**. Отправьте закрытый ключ корневого сертификата. Если у вас есть PEM-сертификат с Объединенным открытым и закрытым ключом, отправьте его здесь.


6. Перейдите на вкладку **Параметры кластера AKS** , чтобы определить конфигурацию кластера службы Kubernetes Azure. В кластере AKS есть различные модули, настроенные для работы сетевых компонентов структуры «книга». Развернутые ресурсы Azure:

    - **Инструменты структуры**: средства, отвечающие за настройку компонентов структуры в книге.
    - Модули **(одноранговые** модули): узлы структуры «сеть» в среде «Главная книга».
    - **Прокси**— модуль-посредник нгникс, через который клиентские приложения могут обмениваться данными с кластером AKS.
    - **CA Fabric**: модуль, на котором работает ЦС структуры.
    - **PostgreSQL**: экземпляр базы данных, который поддерживает удостоверения ЦС структуры.
    - **Хранилище ключей**. экземпляр службы Azure Key Vault, которая развернута для сохранения учетных данных ЦС структуры и корневых сертификатов, предоставленных клиентом. Хранилище используется в случае повторной попытки развертывания шаблона для управления механикой шаблона.
    - **Управляемый диск**: экземпляр службы управляемых дисков Azure, предоставляющий постоянное хранилище для главной книги и для базы данных состояния мирового узла.
    - **Общедоступный IP-адрес**: конечная точка кластера AKS, развернутая для взаимодействия с кластером.

    Введите следующие сведения: 

    ![Снимок экрана, на котором показана вкладка параметров кластера K S.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/create-for-hyperledger-fabric-aks-cluster-settings-1.png)

    - **Имя кластера Kubernetes**. при необходимости измените имя кластера AKS. Это поле заполняется в соответствии с предоставленным префиксом ресурса.
    - **Версия Kubernetes**: выберите версию Kubernetes, которая будет развернута в кластере. В зависимости от региона, выбранного на вкладке « **основы** », доступные Поддерживаемые версии могут измениться.
    - **Префикс DNS**. Введите префикс имени системы доменных имен (DNS) для кластера AKS. Вы будете использовать DNS для подключения к API Kubernetes при управлении контейнерами после создания кластера.
    - **Размер узла**. размер узла Kubernetes можно выбрать в списке единиц хранения для виртуальной машины (SKU), доступных в Azure. Для оптимальной производительности рекомендуется использовать стандартные DS3 версии 2.
    - **Число узлов**: введите число узлов Kubernetes, которые будут развернуты в кластере. Рекомендуется хранить это число узлов, равное количеству узлов структуры, указанных на вкладке **Параметры структуры** , или больше.
    - **Идентификатор клиента субъекта-службы**. Введите идентификатор клиента существующего субъекта-службы или создайте новый. Для проверки подлинности AKS требуется субъект-служба. См. [инструкции по созданию субъекта-службы](/powershell/azure/create-azure-service-principal-azureps#create-a-service-principal).
    - **Секрет клиента субъекта-службы**. Введите секрет клиента субъекта-службы, указанный в идентификаторе клиента для субъекта-службы.
    - **Подтверждение секрета клиента**: подтвердите секрет клиента для субъекта-службы.
    - **Включить мониторинг контейнеров**: выберите Включение мониторинга AKS, которое позволяет журналам AKS отправлять данные в указанную рабочую область log Analytics.
    - **Рабочая область log Analytics**. рабочая область log Analytics будет заполнена рабочей областью по умолчанию, созданной при включенном мониторинге.

8. Перейдите на вкладку **Проверка и создание** . Этот шаг активирует проверку указанных значений.
9. После завершения проверки щелкните **Создать**.

    Развертывание обычно занимает от 10 до 12 минут. Время может изменяться в зависимости от размера и количества указанных узлов AKS.
10. После успешного развертывания вы получите уведомление по уведомлениям Azure в правом верхнем углу.
11. Выберите **Переход к группе ресурсов** , чтобы проверить все ресурсы, созданные в ходе развертывания шаблона. Все имена ресурсов будут начинаться с префикса, указанного на вкладке « **основы** ».

## <a name="build-the-consortium"></a>Создание консорциума

Чтобы создать блокчейн Consortium после развертывания службы упорядочения и одноранговых узлов, выполните последовательно следующие действия. Скрипт структуры Azure азлф позволяет настроить консорциум, создать канал и выполнить операции чаинкоде.

> [!NOTE]
> Сценарий Azure азлф Fabric был обновлен для предоставления дополнительных функциональных возможностей. Если вы хотите обратиться к старому сценарию, см. [файл сведений на сайте GitHub](https://github.com/Azure/Hyperledger-Fabric-on-Azure-Kubernetes-Service/blob/master/consortiumScripts/README.md). Этот сценарий совместим со структурой "Kubernetes" в шаблоне службы Azure, 2.0.0 и более поздних версий. Чтобы проверить версию развертывания, выполните действия, описанные в разделе [Устранение неполадок](#troubleshoot).

> [!NOTE]
> Сценарий предоставляется только в демонстрационных сценариях, разработках и тестах. Канал и консорциум, создаваемый этим сценарием, имеет базовые политики структуры «Главная книга», позволяющие упростить сценарии демонстрации, разработки и тестирования. Для настройки рабочей среды мы рекомендуем обновить политики структуры Windows Channel/Consortium в соответствии с потребностями вашей организации с помощью собственных интерфейсов API структуры веб-журнала.


Все команды для запуска сценария структуры Azure "Главная книга" можно выполнять с помощью интерфейса командной строки Azure bash (CLI). Вы можете войти в Azure Cloud Shell с помощью ![ команды «структура» в шаблоне службы Azure Kubernetes Service в ](./media/hyperledger-fabric-consortium-azure-kubernetes-service/arrow.png) правом верхнем углу портал Azure. В командной строке введите `bash` и выберите клавишу ВВОД, чтобы переключиться на интерфейс командной строки bash, или выберите **Bash** на панели инструментов Cloud Shell.

Дополнительные сведения см. в разделе [Azure Cloud Shell](../../cloud-shell/overview.md) .

![Снимок экрана, на котором показаны команды в Azure Cloud Shell.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/hyperledger-powershell.png)


На следующем рисунке показан пошаговый процесс создания консорциума между Организацией-последовательностью и одноранговой Организацией. В следующих разделах приведены подробные команды для выполнения этих действий.

![Схема процесса создания консорциума.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/process-to-build-consortium-flow-chart.png)

После завершения начальной настройки используйте клиентское приложение для выполнения следующих операций:  

- Управление каналами
- Управление консорциумом
- Управление чаинкоде

### <a name="download-client-application-files"></a>Скачать файлы клиентских приложений

Сначала необходимо загрузить файлы клиентского приложения. Выполните следующие команды, чтобы скачать все необходимые файлы и пакеты:

```bash-interactive
curl https://raw.githubusercontent.com/Azure/Hyperledger-Fabric-on-Azure-Kubernetes-Service/master/azhlfToolSetup.sh | bash
cd azhlfTool
npm install
npm run setup
```

Эти команды будут клонировать код клиентского приложения структуры Azure "Главная книга" из общедоступного репозитория GitHub, а затем загрузить все зависимые пакеты NPM. После успешного выполнения команды вы увидите папку node_modules в текущем каталоге. Все необходимые пакеты загружаются в папку node_modules.

### <a name="set-up-environment-variables"></a>Настройка переменных среды

Все переменные среды соответствуют соглашению об именовании ресурсов Azure.

#### <a name="set-environment-variables-for-the-orderer-organizations-client"></a>Настройка переменных среды для клиента организации заказа

```bash
ORDERER_ORG_SUBSCRIPTION=<ordererOrgSubscription>
ORDERER_ORG_RESOURCE_GROUP=<ordererOrgResourceGroup>
ORDERER_ORG_NAME=<ordererOrgName>
ORDERER_ADMIN_IDENTITY="admin.$ORDERER_ORG_NAME"
CHANNEL_NAME=<channelName>
```

#### <a name="set-environment-variables-for-the-peer-organizations-client"></a>Настройка переменных среды для клиента одноранговой Организации

```bash
PEER_ORG_SUBSCRIPTION=<peerOrgSubscritpion>
PEER_ORG_RESOURCE_GROUP=<peerOrgResourceGroup>
PEER_ORG_NAME=<peerOrgName>
PEER_ADMIN_IDENTITY="admin.$PEER_ORG_NAME"
CHANNEL_NAME=<channelName>
```

В зависимости от числа одноранговых организаций в консорциуме, возможно, потребуется повторить одноранговые команды и задать соответствующим образом переменную среды.

#### <a name="set-environment-variables-for-an-azure-storage-account"></a>Настройка переменных среды для учетной записи хранения Azure

```bash
STORAGE_SUBSCRIPTION=<subscriptionId>
STORAGE_RESOURCE_GROUP=<azureFileShareResourceGroup>
STORAGE_ACCOUNT=<azureStorageAccountName>
STORAGE_LOCATION=<azureStorageAccountLocation>
STORAGE_FILE_SHARE=<azureFileShareName>
```

Используйте следующие команды для создания учетной записи хранения Azure. Если у вас уже есть учетная запись хранения Azure, пропустите этот шаг.

```bash
az account set --subscription $STORAGE_SUBSCRIPTION
az group create -l $STORAGE_LOCATION -n $STORAGE_RESOURCE_GROUP
az storage account create -n $STORAGE_ACCOUNT -g  $STORAGE_RESOURCE_GROUP -l $STORAGE_LOCATION --sku Standard_LRS
```

Используйте следующие команды, чтобы создать общую папку в учетной записи хранения Azure. Если у вас уже есть файловый ресурс, пропустите этот шаг.

```bash
STORAGE_KEY=$(az storage account keys list --resource-group $STORAGE_RESOURCE_GROUP  --account-name $STORAGE_ACCOUNT --query "[0].value" | tr -d '"')
az storage share create  --account-name $STORAGE_ACCOUNT  --account-key $STORAGE_KEY  --name $STORAGE_FILE_SHARE
```

Используйте следующие команды, чтобы создать строку подключения для файлового ресурса Azure.

```bash
STORAGE_KEY=$(az storage account keys list --resource-group $STORAGE_RESOURCE_GROUP  --account-name $STORAGE_ACCOUNT --query "[0].value" | tr -d '"')
SAS_TOKEN=$(az storage account generate-sas --account-key $STORAGE_KEY --account-name $STORAGE_ACCOUNT --expiry `date -u -d "1 day" '+%Y-%m-%dT%H:%MZ'` --https-only --permissions lruwd --resource-types sco --services f | tr -d '"')
AZURE_FILE_CONNECTION_STRING=https://$STORAGE_ACCOUNT.file.core.windows.net/$STORAGE_FILE_SHARE?$SAS_TOKEN

```

### <a name="import-an-organization-connection-profile-admin-user-identity-and-msp"></a>Импорт профиля подключения Организации, удостоверения пользователя администратора и MSP

Используйте следующие команды, чтобы получить профиль подключения Организации, удостоверение пользователя администратора и поставщика управляемых служб (MSP) из кластера Azure Kubernetes Service и сохранить эти удостоверения в локальном хранилище клиентского приложения. Примером локального хранилища является каталог *азлфтул/Stores* .

Для организации заказа:

```bash
./azhlf adminProfile import fromAzure -o $ORDERER_ORG_NAME -g $ORDERER_ORG_RESOURCE_GROUP -s $ORDERER_ORG_SUBSCRIPTION
./azhlf connectionProfile import fromAzure -g $ORDERER_ORG_RESOURCE_GROUP -s $ORDERER_ORG_SUBSCRIPTION -o $ORDERER_ORG_NAME   
./azhlf msp import fromAzure -g $ORDERER_ORG_RESOURCE_GROUP -s $ORDERER_ORG_SUBSCRIPTION -o $ORDERER_ORG_NAME
```

Для одноранговой Организации:

```bash
./azhlf adminProfile import fromAzure -g $PEER_ORG_RESOURCE_GROUP -s $PEER_ORG_SUBSCRIPTION -o $PEER_ORG_NAME
./azhlf connectionProfile import fromAzure -g $PEER_ORG_RESOURCE_GROUP -s $PEER_ORG_SUBSCRIPTION -o $PEER_ORG_NAME
./azhlf msp import fromAzure -g $PEER_ORG_RESOURCE_GROUP -s $PEER_ORG_SUBSCRIPTION -o $PEER_ORG_NAME
```

### <a name="create-a-channel"></a>Создание канала

В клиенте организации заказа используйте следующую команду, чтобы создать канал, содержащий только организацию заказа.  

```bash
./azhlf channel create -c $CHANNEL_NAME -u $ORDERER_ADMIN_IDENTITY -o $ORDERER_ORG_NAME
```

### <a name="add-a-peer-organization-for-consortium-management"></a>Добавление одноранговой Организации для управления консорциумом

>[!NOTE]
> Прежде чем начать работу с любыми операциями консорциума, убедитесь, что вы завершили начальную настройку клиентского приложения.  

Выполните следующие команды в указанном порядке, чтобы добавить одноранговую организацию в канал и в консорциуме: 

1.  В клиенте одноранговой Организации отправьте MSP в службе хранилища Azure.

      ```bash
      ./azhlf msp export toAzureStorage -f  $AZURE_FILE_CONNECTION_STRING -o $PEER_ORG_NAME
      ```
2.  В клиенте организации заказа скачайте MSP в службе "хранилище Azure" для одноранговой Организации. Затем выполните команду, чтобы добавить одноранговую организацию в канал и консорциум.

      ```bash
      ./azhlf msp import fromAzureStorage -o $PEER_ORG_NAME -f $AZURE_FILE_CONNECTION_STRING
      ./azhlf channel join -c  $CHANNEL_NAME -o $ORDERER_ORG_NAME  -u $ORDERER_ADMIN_IDENTITY -p $PEER_ORG_NAME
      ./azhlf consortium join -o $ORDERER_ORG_NAME  -u $ORDERER_ADMIN_IDENTITY -p $PEER_ORG_NAME
      ```

3.  В клиенте организации заказа отправьте профиль подключения заказа в службе хранилища Azure, чтобы одноранговая Организация могла подключаться к узлам заказов с помощью этого профиля подключения.

      ```bash
      ./azhlf connectionProfile  export toAzureStorage -o $ORDERER_ORG_NAME -f $AZURE_FILE_CONNECTION_STRING
      ```

4.  Из клиента одноранговой Организации Скачайте профиль подключения заказа из службы хранилища Azure. Затем выполните команду, чтобы добавить одноранговые узлы в канал.

      ```bash
      ./azhlf connectionProfile  import fromAzureStorage -o $ORDERER_ORG_NAME -f $AZURE_FILE_CONNECTION_STRING
      ./azhlf channel joinPeerNodes -o $PEER_ORG_NAME  -u $PEER_ADMIN_IDENTITY -c $CHANNEL_NAME --ordererOrg $ORDERER_ORG_NAME
      ```

Аналогичным образом, чтобы добавить в канал другие одноранговые Организации, обновите переменные среды одноранговой сети в соответствии с требуемой одноранговой Организацией и повторите шаги 1 – 4.

### <a name="set-anchor-peers"></a>Установка одноранговых узлов

В клиенте одноранговой организации выполните команду для установки одноранговых узлов для одноранговой Организации в указанном канале.

>[!NOTE]
> Перед выполнением этой команды убедитесь, что в канале добавлена одноранговая организация с помощью команд управления консорциумом.

```bash
./azhlf channel setAnchorPeers -c $CHANNEL_NAME -p <anchorPeersList> -o $PEER_ORG_NAME -u $PEER_ADMIN_IDENTITY --ordererOrg $ORDERER_ORG_NAME
```

`<anchorPeersList>` Разделенный пробелами список одноранговых узлов, которые должны быть установлены в качестве однорангового узла. Пример:

  - Задайте значение `<anchorPeersList>` , `"peer1"` Если вы хотите задать только узел Peer1 в качестве однорангового узла.
  - Задайте значение `<anchorPeersList>` , `"peer1" "peer3"` Если вы хотите задать узлы Peer1 и peer3 в качестве узлов привязки.

## <a name="chaincode-management-commands"></a>Команды управления чаинкоде

>[!NOTE]
> Перед началом работы с чаинкоде убедитесь, что вы выполнили начальную настройку клиентского приложения.  

### <a name="set-the-chaincode-specific-environment-variables"></a>Задание переменных среды, относящихся к чаинкоде

```bash
# Peer organization name where the chaincode operation will be performed
ORGNAME=<PeerOrgName>
USER_IDENTITY="admin.$ORGNAME"  
# If you are using chaincode_example02 then set CC_NAME=“chaincode_example02”
CC_NAME=<chaincodeName>  
# If you are using chaincode_example02 then set CC_VERSION=“1” for validation
CC_VERSION=<chaincodeVersion>
# Language in which chaincode is written. Supported languages are 'node', 'golang', and 'java'  
# Default value is 'golang'  
CC_LANG=<chaincodeLanguage>  
# CC_PATH contains the path where your chaincode is placed. This is the absolute path to the chaincode project root directory.
# If you are using chaincode_example02 to validate then CC_PATH=“/home/<username>/azhlfTool/samples/chaincode/src/chaincode_example02/go”
CC_PATH=<chaincodePath>  
# Channel on which chaincode will be instantiated/invoked/queried  
CHANNEL_NAME=<channelName>  
```

### <a name="install-chaincode"></a>Установка чаинкоде  

Выполните следующую команду, чтобы установить чаинкоде в одноранговой Организации.  

```bash
./azhlf chaincode install -o $ORGNAME -u $USER_IDENTITY -n $CC_NAME -p $CC_PATH -l $CC_LANG -v $CC_VERSION  

```
Команда установит чаинкоде на всех одноранговых узлах одноранговой Организации, установленной в `ORGNAME` переменной среды. Если в канале находятся две или более одноранговые Организации и вы хотите установить чаинкоде на всех этих узлах, выполните эту команду отдельно для каждой одноранговой Организации.  

Выполните следующие действия.  

1.  Задайте `ORGNAME` и `USER_IDENTITY` в соответствии с `peerOrg1` и выполните `./azhlf chaincode install` команду.  
2.  Задайте `ORGNAME` и `USER_IDENTITY` в соответствии с `peerOrg2` и выполните `./azhlf chaincode install` команду.  

### <a name="instantiate-chaincode"></a>Создание экземпляра чаинкоде  

В приложении однорангового клиента выполните следующую команду, чтобы создать экземпляр чаинкоде в канале.  

```bash
./azhlf chaincode instantiate -o $ORGNAME -u $USER_IDENTITY -n $CC_NAME -v $CC_VERSION -c $CHANNEL_NAME -f <instantiateFunc> --args <instantiateFuncArgs>
```

Передайте имя функции создания экземпляра и список аргументов с разделителями-пробелами в `<instantiateFunc>` и `<instantiateFuncArgs>` соответственно. Например, чтобы создать экземпляр chaincode_example02. Go чаинкоде, задайте `<instantiateFunc>` для значение `init` и `<instantiateFuncArgs>` равным `"a" "2000" "b" "1000"` .

Можно также передать JSON файла конфигурации коллекции с помощью `--collections-config` флага. Или задайте временные аргументы с помощью `-t` флага при создании экземпляра чаинкоде, используемого для частных транзакций.

Пример:

```bash
./azhlf chaincode instantiate -c $CHANNEL_NAME -n $CC_NAME -v $CC_VERSION -o $ORGNAME -u $USER_IDENTITY --collections-config <collectionsConfigJSONFilePath>
./azhlf chaincode instantiate -c $CHANNEL_NAME -n $CC_NAME -v $CC_VERSION -o $ORGNAME -u $USER_IDENTITY --collections-config <collectionsConfigJSONFilePath> -t <transientArgs>
```

`<collectionConfigJSONFilePath>`Часть — это путь к JSON-файлу, который содержит коллекции, определенные для создания экземпляра закрытых чаинкоде данных. JSON-файл конфигурации образца коллекции можно найти относительно каталога *азлфтул* по следующему пути: `./samples/chaincode/src/private_marbles/collections_config.json` .
Передача `<transientArgs>` в виде допустимого JSON в строковом формате. Экранирование любых специальных символов. Пример: `'{\\\"asset\":{\\\"name\\\":\\\"asset1\\\",\\\"price\\\":99}}'`

> [!NOTE]
> Выполните команду один раз из одной одноранговой Организации в канале. После успешной отправки транзакции в заказ, заказ распределяет эту транзакцию всем одноранговым организациям в канале. Затем создается экземпляр чаинкоде на всех одноранговых узлах во всех одноранговых организациях в канале.  

### <a name="invoke-chaincode"></a>Вызвать чаинкоде  

В клиенте одноранговой организации выполните следующую команду, чтобы вызвать функцию чаинкоде:  

```bash
./azhlf chaincode invoke -o $ORGNAME -u $USER_IDENTITY -n $CC_NAME -c $CHANNEL_NAME -f <invokeFunc> -a <invokeFuncArgs>  
```

Передайте имя функции Invoke и список аргументов, разделенных пробелами, в  `<invokeFunction>`   и  `<invokeFuncArgs>`   соответственно. Продолжение работы с примером chaincode_example02. Go чаинкоде для выполнения операции Invoke, для которой задано значение  `<invokeFunction>`    `invoke`   и  `<invokeFuncArgs>`   `"a" "b" "10"` .  

>[!NOTE]
> Выполните команду один раз из одной одноранговой Организации в канале. После успешной отправки транзакции в заказ, заказ распределяет эту транзакцию всем одноранговым организациям в канале. Затем состояние мира обновляется на всех одноранговых узлах всех одноранговых организаций в канале.  


### <a name="query-chaincode"></a>Чаинкоде запросов  

Выполните следующую команду, чтобы запросить чаинкоде:  

```bash
./azhlf chaincode query -o $ORGNAME -p <endorsingPeers> -u $USER_IDENTITY -n $CC_NAME -c $CHANNEL_NAME -f <queryFunction> -a <queryFuncArgs> 
```
Одноранговые узлы подтверждающий — это равноправные узлы, где чаинкоде установлен и вызывается для выполнения транзакций. Необходимо задать параметр, `<endorsingPeers>` содержащий имена одноранговых узлов из текущей одноранговой Организации. Перечислите одноранговые узлы подтверждающий для заданного чаинкоде и сочетания каналов, разделенные пробелами. Например, `-p "peer1" "peer3"`.

Если вы используете *азлфтул* для установки чаинкоде, передайте имя однорангового узла в качестве значения аргумента Peer подтверждающий. Чаинкоде устанавливается на каждом одноранговом узле этой Организации. 

Передайте имя функции запроса и список аргументов с разделителями-пробелами в  `<queryFunction>`   и  `<queryFuncArgs>`   соответственно. Снова chaincode_example02. Go чаинкоде в качестве ссылки, чтобы запросить значение "a" в мировом состоянии, задайте  `<queryFunction>`   для значения и значение  `query`  `<queryArgs>` `"a"` .  

## <a name="troubleshoot"></a>Диагностика

### <a name="find-deployed-version"></a>Найти развернутую версию

Выполните следующие команды, чтобы найти версию развертывания шаблона. Задайте переменные среды в соответствии с группой ресурсов, в которой развернут шаблон.

```bash
SWITCH_TO_AKS_CLUSTER $AKS_CLUSTER_RESOURCE_GROUP $AKS_CLUSTER_NAME $AKS_CLUSTER_SUBSCRIPTION
kubectl describe pod fabric-tools -n tools | grep "Image:" | cut -d ":" -f 3
```

### <a name="patch-previous-version"></a>Исправление предыдущей версии

Если у вас возникают проблемы с запуском чаинкоде при развертывании версии шаблона ниже v 3.0.0, выполните следующие действия, чтобы выполнить исправление для одноранговых узлов.

Скачайте скрипт развертывания однорангового узла.

```bash
curl https://raw.githubusercontent.com/Azure/Hyperledger-Fabric-on-Azure-Kubernetes-Service/master/scripts/patchPeerDeployment.sh -o patchPeerDeployment.sh; chmod 777 patchPeerDeployment.sh
```

Запустите сценарий, используя следующую команду, заменив параметры для своего однорангового узла.

```bash
source patchPeerDeployment.sh <peerOrgSubscription> <peerOrgResourceGroup> <peerOrgAKSClusterName>
```

Дождитесь исправления всех одноранговых узлов. Вы всегда можете проверить состояние одноранговых узлов в другом экземпляре оболочки с помощью следующей команды.

```bash
kubectl get pods -n hlf
```

## <a name="support-and-feedback"></a>Поддержка и обратная связь

Чтобы оставаться в курсе предложений службы блокчейн и информации из группы разработчиков Блокчейн Azure, посетите [блог Блокчейн Azure](https://azure.microsoft.com/blog/topics/blockchain/).

Чтобы отправить отзыв о продукте или запросить новые функции, опубликуйте обращение на [форуме обратной связи по блокчейну Azure](https://aka.ms/blockchainuservoice).

### <a name="community-support"></a>Поддержка сообщества

Общайтесь с инженерами Майкрософт и экспертами сообщества Azure Блокчейн:

- [Microsoft Q&странице](/answers/topics/azure-blockchain-workbench.html) 
   
  Поддержка инженеров шаблонов блокчейн ограничена проблемами развертывания.
- [Сообщество Майкрософт для технического сообщества](https://techcommunity.microsoft.com/t5/Blockchain/bd-p/AzureBlockchain)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-blockchain-workbench)
