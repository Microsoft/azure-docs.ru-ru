---
title: Краткое руководство по развертыванию платформы корпоративного приложения JBoss (EAP) с помощью Red Hat Enterprise Linux (RHEL) на виртуальных машинах Azure и масштабируемых наборах виртуальных машин
description: Развертывание корпоративных приложений Java с помощью Red Hat JBoss EAP на виртуальных машинах RHEL Azure и масштабируемых наборах виртуальных машин.
author: theresa-nguyen
ms.author: bicnguy
ms.date: 10/30/2020
ms.assetid: 8a4df7bf-be49-4198-800e-db381cda98f5
ms.topic: quickstart
ms.service: virtual-machines
ms.subservice: redhat
ms.custom:
- mode-api
ms.collection: linux
ms.openlocfilehash: 18cf093adad858f50b2b1fa1c97e38821bd1c949
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538680"
---
# <a name="deploy-enterprise-java-applications-to-azure-with-jboss-eap-on-red-hat-enterprise-linux"></a>Развертывание корпоративных приложений Java в Azure с помощью JBoss EAP на Red Hat Enterprise Linux

В этой статье приведены шаблоны быстрого запуска Azure, в которых показано, как развертывать [платформу корпоративного приложения JBoss (EAP)](https://www.redhat.com/en/technologies/jboss-middleware/application-platform) с помощью [Red Hat Enterprise Linux (RHEL)](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) на виртуальных машинах Azure и масштабируемых наборах виртуальных машин. Вы будете использовать пример приложения Java для проверки развертывания. 

JBoss EAP — это платформа сервера приложений с открытым кодом. Она обеспечивает безопасность, масштабируемость и производительность приложений Java на уровне предприятия. RHEL — это платформа операционной системы (ОС) с открытым кодом. Она позволяет масштабировать существующие приложения и развертывать новейшие технологии во всех средах. 

JBoss EAP и RHEL содержат все компоненты, которые необходимы для создания, запуска, развертывания корпоративных приложений Java и управления ими в любой среде. Эта комбинация представляет собой решение с открытым кодом для локальных, виртуальных сред, которое также можно использовать в частных, общедоступных или гибридных облаках.

## <a name="prerequisites"></a>Предварительные требования 

* Учетная запись Azure с активной подпиской. Чтобы получить подписку Azure, активируйте [деньги на счете в Azure для подписчиков Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) или [бесплатно создайте учетную запись](https://azure.microsoft.com/pricing/free-trial).

* Установленное JBoss EAP. У вас должна быть учетная запись Red Hat с правом на управление подписками Red Hat (RHSM) для JBoss EAP. Это право позволит вам скачать версию JBoss EAP, протестированную и сертифицированную для Red Hat.  

  Если у вас нет права на EAP, перед началом работы необходимо получить [подписку на ознакомительную версию JBoss EAP](https://access.redhat.com/products/red-hat-jboss-enterprise-application-platform/evaluation). Чтобы создать новую подписку Red Hat, перейдите на [Клиентский портал Red Hat](https://access.redhat.com/) и настройте учетную запись.

* [Интерфейс командной строки Azure](/cli/azure/overview).

* Параметры RHEL. Выберите между Оплатой по мере использования (PAYG) и использованием собственной подписки (BYOS). При использовании BYOS необходимо активировать образы RHEL Gold [Red Hat Cloud Access](https://access.redhat.com/), прежде чем развертывать шаблон быстрого запуска.

## <a name="java-ee-and-jakarta-ee-application-migration"></a>Миграция приложений Java EE и Jakarata EE

### <a name="migrate-to-jboss-eap"></a>Переход на JBoss EAP
JBoss EAP 7.2 и 7.3 — это сертифицированные реализации спецификаций Java Enterprise Edition (Java EE) 8 и Jakarata EE 8. JBoss EAP предоставляет предварительно настроенные параметры для таких функций, как кластеризация, обмен сообщениями и распределенное кэширование с высоким уровнем доступности (HA). Эта платформа также позволяет пользователям записывать, развертывать и запускать приложения с помощью различных интерфейсов API и служб, которые предоставляет JBoss EAP.  

Дополнительные сведения о JBoss EAP см. во [введении к JBoss EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html-single/introduction_to_jboss_eap/index) или во [введении к JBoss EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/introduction_to_jboss_eap/index).

 #### <a name="applications-on-jboss-eap"></a>Приложения на JBoss EAP

* **Приложения веб-служб.** Веб-службы предоставляют стандартный способ взаимодействия между программными приложениями. Каждое приложение можно запускать на различных платформах и средах. Эти веб-службы упрощают внутреннюю и разнородную связь между подсистемами. 

  Дополнительные сведения см. в статье [Разработка приложений веб-служб в EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/developing_web_services_applications/index) или [Разработка приложений веб-служб в EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/developing_web_services_applications/index).

* **Корпоративные приложения Java Beans (EJB).** EJB 3.2 — это программный интерфейс для разработки распределенных, транзакционных, безопасных и переносимых приложений Java EE и Jakarta EE. EJB использует компоненты на стороне сервера, именуемые Enterprise Beans, чтобы реализовать бизнес-логику приложения как отдельное решение, которое создает возможности для повторного использования. 

  Дополнительные сведения см. в статье [Разработка приложений EJB в EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/developing_ejb_applications/index) или [Разработка приложений EJB в EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/developing_ejb_applications/index).

* **Приложения Hibernate.** Разработчики и администраторы могут разрабатывать и развертывать Java Persistence API (JPA) и приложения Hibernate с помощью JBoss EAP. Hibernate Core — это объектно-реляционная платформа сопоставления для языка Java. Она предоставляет платформу для сопоставления объектно-ориентированной модели предметной области с реляционной базой данных, поэтому приложения могут избежать непосредственного взаимодействия с базой данных. 

  Hibernate Entity Manager реализует программные интерфейсы и правила жизненного цикла, определенные [спецификацией JPA 2.1](https://www.jcp.org/en/jsr/overview). Вместе с Hibernate Annotations эта программа-оболочка реализует полноценное (и изолированное) решение JPA наряду с устоявшейся платформой Hibernate Core. 
  
  Чтобы узнать больше о Hibernate, перейдите на страницу [JPA в EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/development_guide/java_persistence_api) или [Jakarta Persistence в EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/development_guide/java_persistence_api).

#### <a name="red-hat-migration-toolkit-for-applications"></a>Red Hat Migration Toolkit for Applications
[Red Hat Migration Toolkit for Applications (MTA)](https://developers.redhat.com/products/mta/overview) — это средство миграции для серверов приложений Java. Используйте это средство для перехода с сервера приложений на JBoss EAP. Оно работает с подключаемыми модулями IDE для [Eclipse IDE](https://www.eclipse.org/ide/), [Red Hat CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview), а также [Visual Studio Code](https://code.visualstudio.com/docs/languages/java) для Java. 

MTA — это бесплатное средство с открытым кодом, которое:
* Автоматизирует анализ приложений.
* Поддерживает оценку затрат.
* Ускоряет перенос кода.
* Поддерживает контейнеризацию.
* Интегрируется с Построителем рабочих нагрузок Azure.

### <a name="migrate-jboss-eap-from-on-premises-to-azure"></a>Перенос JBoss EAP из локальной среды в Azure
Предложение Azure Marketplace для JBoss EAP на RHEL будет установлено и подготовлено к роботе на виртуальных машинах Azure менее чем за 20 минут. Доступ к этим предложениям можно получить из [Azure Marketplace](https://azuremarketplace.microsoft.com/).

Это предложение Azure Marketplace использует различные сочетания версий EAP и RHEL для поддержки ваших требований. Для JBoss EAP всегда используется BYOS, однако для ОС RHEL возможен выбор между использованием собственной подписки (BYOS) и Оплатой по мере использования (PAYG). Предложение Azure Marketplace содержит варианты плана для JBoss EAP на RHEL в качестве автономных или кластеризованных виртуальных машин:

* JBoss EAP 7.2 на виртуальной машине RHEL 7.7 (PAYG)
* JBoss EAP 7.2 на виртуальной машине RHEL 8.0 (PAYG)
* JBoss EAP 7.3 на виртуальной машине RHEL 8.0 (PAYG)
* JBoss EAP 7.2 на виртуальной машине RHEL 7.7 (BYOS)
* JBoss EAP 7.2 на виртуальной машине RHEL 8.0 (BYOS)
* JBoss EAP 7.3 на виртуальной машине RHEL 8.0 (BYOS)

Наряду с предложениями Azure Marketplace можно использовать шаблоны быстрого запуска, которые помогут приступить к миграции в Azure. Эти краткие руководства содержат встроенные шаблоны Azure Resource Manager (ARM) и скрипты для развертывания JBoss EAP на RHEL в различных конфигурациях и комбинациях версии. Вы получите:

* Подсистему балансировки нагрузки.
* Частный IP-адрес для балансировки нагрузки и виртуальных машин.
* Виртуальную сеть с одной подсетью.
* Конфигурацию виртуальной машины (кластерная или автономная).
* Пример приложения Java.

Архитектура решения для этих шаблонов содержит:

* JBoss EAP на автономной виртуальной машине RHEL.
* JBoss EAP, кластеризованное на нескольких виртуальных машинах RHEL.
* JBoss EAP, кластеризованное с помощью масштабируемого набора виртуальных машин Azure.

#### <a name="linux-workload-migration-for-jboss-eap"></a>Миграция рабочей нагрузки Linux для JBoss EAP
Построитель рабочих нагрузок Azure упрощает процесс подтверждения концепции, вычисления и миграции для локальных приложений Java в Azure. Построитель рабочих нагрузок интегрируется со средством службы "Миграция Azure" для обнаружения серверов JBoss EAP. Затем он динамически создает сборник схем Ansible для развертывания сервера JBoss EAP. Он использует средство Red Hat MTA для переноса серверов с других серверов приложений в JBoss EAP. 

Ниже приведены шаги для упрощения миграции:
1. **Вычисление.** Вычисление кластеров JBoss EAP с использованием виртуальной машины Azure или масштабируемого набора виртуальных машин.
1. **Оценка.** Проверка приложений и инфраструктуры.
1. **Конфигурация инфраструктуры.** Создание профиля рабочей нагрузки.
1. **Развертывание и тестирование.** Развертывание, миграция и тестирование рабочей нагрузки.
1. **Конфигурация после развертывания.** Интеграция с данными, мониторингом, безопасностью, резервным копированием и т. д.

## <a name="server-configuration-choice"></a>Выбор конфигурации сервера

Для развертывания виртуальной машины RHEL можно выбрать PAYG или BYOS. Образы из [Azure Marketplace](https://azuremarketplace.microsoft.com) по умолчанию используют PAYG. Разверните виртуальную машину RHEL типа BYOS, если у вас есть собственный образ ОС RHEL. Перед развертыванием виртуальных машин или масштабируемого набора виртуальных машин убедитесь, что ваша учетная запись RHSM имеет право на BYOS через Cloud Access.

JBoss EAP обладает мощными возможностями управления вместе с предоставлением функциональных средств и программных интерфейсов в своих приложениях. Эти возможности управления зависят от того, какой режим работы вы используете для запуска JBoss EAP. Они поддерживаются в RHEL и Windows Server. JBoss EAP предлагает работу в режиме изолированного сервера для управления дискретными экземплярами, а также режим управляемого домена для управления группами экземпляров из отдельной контрольной точки. 

> [!NOTE]
> Управляемые домены JBoss EAP не поддерживаются в Microsoft Azure, так как службы инфраструктуры Azure управляют функцией высокого уровня доступности (HA). 

Переменная среды `EAP_HOME` используется для обозначения пути к установке JBoss EAP. Выполните следующую команду для запуска службы JBoss EAP в автономном режиме:

```
$EAP_HOME/bin/standalone.sh
```
    
В этом скрипте запуска для задания некоторых настроек по умолчанию, таких как параметры виртуальной машины Java, используется файл EAP_HOME/bin/standalone.conf. Параметры можно настроить в этом файле. JBoss EAP использует файл конфигурации standalone.xml для запуска в автономном режиме по умолчанию, однако его можно запускать и в другом режиме. 

Дополнительные сведения о доступных автономных файлах конфигурации и способах их использования см. в разделе [Файлы конфигурации изолированного сервера для EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/configuration_guide/jboss_eap_management#standalone_server_configuration_files) или [Файлы конфигурации изолированного сервера для EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/configuration_guide/jboss_eap_management#standalone_server_configuration_files). 

Чтобы запустить JBoss EAP с помощью другой конфигурации, используйте аргумент `--server-config`. Пример:
    
 ```
 $EAP_HOME/bin/standalone.sh --server-config=standalone-full.xml
 ```
    
Чтобы ознакомиться с полным списком всех доступных аргументов скрипта запуска и их целями, используйте аргумент `--help`. Дополнительные сведения см. в разделе [Аргументы среды выполнения сервера в EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/configuration_guide/reference_material#reference_of_switches_and_arguments_to_pass_at_server_runtime) или [Аргументы среды выполнения сервера в EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/configuration_guide/reference_material#reference_of_switches_and_arguments_to_pass_at_server_runtime).
    
JBoss EAP также может работать в режиме кластера. Обмен сообщениями в кластере JBoss EAP позволяет группировать серверы обмена сообщениями JBoss EAP для распределения нагрузки на обработку сообщений. Каждый активный узел в кластере является активным сервером обмена сообщениями JBoss EAP, который управляет собственными сообщениями и обрабатывает свои собственные подключения. Дополнительные сведения см. в статье [Общие сведения о кластерах в EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/configuring_messaging/clusters_overview) или [Общие сведения о кластерах в EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/configuring_messaging/clusters_overview). 

## <a name="support-and-subscription-notes"></a>Рекомендации по поддержке и подпискам
Эти шаблоны быстрого запуска предоставляются следующим образом: 

- ОС RHEL предоставляется как PAYG или BYOS через модель образов Red Hat Gold.
- JBoss EAP предоставляется только в виде BYOS.

#### <a name="using-rhel-os-with-the-payg-model"></a>Использование ОС RHEL с моделью PAYG

По умолчанию эти шаблоны быстрого запуска используют образ RHEL 7.7 по запросу или 8.0 PAYG из Azure Marketplace. За образы PAYG взимается дополнительная почасовая плата за подписку RHEL, помимо обычной стоимости вычислений, сети и хранения. В то же время экземпляр регистрируется в подписке Red Hat. Это означает, что вы будете использовать одно из ваших прав. 

Этот образ PAYG приводит к "двойному выставлению счетов". Этой ошибки можно избежать, создав собственный образ RHEL. Дополнительные сведения см. в статье базы знаний Red Hat [Подготовка виртуальной машины RHEL для Microsoft Azure](https://access.redhat.com/articles/uploading-rhel-image-to-azure). Или активируйте свой образ RHEL Gold для [Red Hat Cloud Access](https://access.redhat.com/).

Дополнительные сведения о ценах на виртуальные машины PAYG см. в статье [Цены на виртуальные машины Red Hat](https://azure.microsoft.com/pricing/details/virtual-machines/red-hat/). Чтобы использовать RHEL в модели PAYG, вам потребуется подписка Azure с указанным методом оплаты для [RHEL 7.7 в Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/RedHat.RedHatEnterpriseLinux77-ARM) или [RHEL 8.0 в Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/RedHat.RedHatEnterpriseLinux80-ARM). Для этих предложений требуется указать метод оплаты в подписке Azure.

#### <a name="using-rhel-os-with-the-byos-model"></a>Использование ОС RHEL с моделью BYOS

Чтобы использовать BYOS для ОС RHEL, необходимо иметь действующую подписку Red Hat с правами для использования ОС RHEL в Azure. Перед развертыванием этого ОС RHEL с моделью BYOS выполните следующие предварительные требования:

1. Убедитесь, что у вас есть права на ОС RHEL и JBoss EAP, подключенные к подписке Red Hat.
2. Авторизуйте идентификатор подписки Azure, чтобы использовать образы RHEL BYOS. Следуйте инструкциям [документации по управлению подписками Red Hat](https://access.redhat.com/documentation/red_hat_subscription_management/1/), чтобы завершить процесс, который включает следующие действия:

   1. Включите Microsoft Azure в качестве поставщика на панели мониторинга Red Hat Cloud Access.

   1. Добавьте идентификаторы подписок Azure.

   1. Включите новые продукты для Cloud Access в Microsoft Azure.
    
   1. Активируйте образы Red Hat Gold для подписки Azure. Дополнительную информацию см. в разделе [Образы Red Hat Gold в Microsoft Azure](https://access.redhat.com/documentation/en-us/red_hat_subscription_management/1/html/red_hat_cloud_access_reference_guide/cloud-access-gold-images_cloud-access#proc_using-gold-images-azure_cloud-access).

   1. Подождите, пока в подписке Azure будут доступны образы Red Hat Gold. Эти образы обычно стают доступными в течение трех часов после отправки.
    
3. Примите условия Azure Marketplace для образов RHEL BYOS. Этот процесс можно выполнить, запустив следующие команды Azure CLI. Дополнительные сведения см. в документации по [образам RHEL Gold BYOS в Azure](./byos.md). Важно, чтобы вы запустили последнюю версию Azure CLI.

   1. Откройте сеанс Azure CLI и пройдите проверку подлинности с помощью учетной записи Azure. Дополнительные сведения см. в статье о [входе с помощью Azure CLI](/cli/azure/authenticate-azure-cli).

   1. Убедитесь, что в подписке доступны образы RHEL BYOS, выполнив следующую команду CLI. Если вы не получите результатов, убедитесь, что подписка Azure активирована для образов RHEL BYOS.
   
      ```
      az vm image list --offer rhel-byos --all
      ```

   1. Выполните следующую команду, чтобы принять условия Azure Marketplace для RHEL 7.7 BYOS и RHEL 8.0 BYOS соответственно:
      ```
      az vm image terms accept --publisher redhat --offer rhel-byos --plan rhel-lvm77
      ``` 

      ```
      az vm image terms accept --publisher redhat --offer rhel-byos --plan rhel-lvm8
      ``` 
   
Теперь ваша подписка готова к развертыванию RHEL 7.7 или 8.0 BYOS на виртуальных машинах Azure.

#### <a name="using-jboss-eap-with-the-byos-model"></a>Использование JBoss EAP с моделью BYOS

Платформа JBoss EAP доступна в Azure только в модели BYOS. При развертывании этого шаблона необходимо предоставить учетные данные RHSM, а также идентификатор пула RHSM с допустимыми правами EAP. Если у вас нет прав на EAP, перед началом работы необходимо получить [подписку на ознакомительную версию JBoss EAP](https://access.redhat.com/products/red-hat-jboss-enterprise-application-platform/evaluation).

## <a name="deployment-options"></a>Параметры развертывания

Вы можете развернуть шаблон следующими способами:

- **PowerShell.** Чтобы развернуть этот шаблон, выполните следующие команды: 

  ```
  New-AzResourceGroup -Name <resource-group-name> -Location <resource-group-location> #use this command when you need to create a new resource group for your deployment
  ```

  ```
  New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateUri <raw link to the template which can be obtained from github>
  ```
 
  Сведения об установке и настройке Azure PowerShell см. в [соответствующей документации](/powershell/azure/).  

- **Azure CLI**. Чтобы развернуть этот шаблон, выполните следующие команды:

  ```
  az group create --name <resource-group-name> --location <resource-group-location> #use this command when you need to create a new resource group for your deployment
  ```

  ```
  az deployment group create --resource-group <my-resource-group> --template-uri <raw link to the template which can be obtained from github>
  ```

  Сведения об установке и настройке Azure CLI см. в [этой статье](/cli/azure/install-azure-cli).

- **Портал Azure**. Вы можете выполнить развертывание на портале Azure, перейдя к шаблонам быстрого запуска Azure, как указано в следующем разделе. После завершения работы с кратким руководством нажмите кнопку **Развертывание в Azure** или **Обзор в GitHub**.

## <a name="azure-quickstart-templates"></a>Шаблоны быстрого запуска Azure

Вы можете начать, используя один из следующих шаблонов быстрого запуска для JBoss EAP на RHEL, который соответствует вашей цели развертывания:

* <a href="https://azure.microsoft.com/resources/templates/jboss-eap-standalone-rhel/"> JBoss EAP на RHEL (автономная виртуальная машина)</a>. Развертывание веб-приложения с именем JBoss-EAP в Azure в JBoss EAP 7.2 или 7.3, выполняющемся на виртуальной машине RHEL 7.7 или 8.0.

* <a href="https://azure.microsoft.com/resources/templates/jboss-eap-clustered-multivm-rhel/"> JBoss EAP на RHEL (кластеризованный, с несколькими виртуальными машинами)</a>. Развертывание веб-приложения с именем EAP-Session-Replication в JBoss EAP 7.2 или 7.3, выполняющемся на виртуальных машинах с номером *n* в RHEL 7.7 или 8.0. Пользователь выбирает значение *n*. Все виртуальные машины добавляются во внутренний пул подсистемы балансировки нагрузки.

* <a href="https://azure.microsoft.com/en-us/resources/templates/jboss-eap-clustered-vmss-rhel/"> JBoss EAP на RHEL (кластеризованный, масштабируемый набор виртуальных машин)</a>. Развертывание веб-приложения с именем EAP-Session-Replication в JBoss EAP 7.2 или 7.3, выполняющемся в масштабируемых наборах виртуальных машин RHEL 7.7 или 8.0.

## <a name="resource-links"></a>Ссылки на ресурсы

* [Преимущество гибридного использования Azure](../../windows/hybrid-use-benefit-licensing.md)
* [Настройка приложения Java для Службы приложений Azure](../../../app-service/configure-language-java.md)
* [Azure Red Hat OpenShift](https://azure.microsoft.com/services/openshift/)
* [Создание приложения Java в Службе приложений Azure](../../../app-service/quickstart-java.md)
* [Развертывание JBoss EAP в Службе приложений Azure](https://github.com/JasonFreeberg/jboss-on-app-service)

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [JBoss EAP 7.2](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/).
* Дополнительные сведения о [JBoss EAP 7.3](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.3/).
* Дополнительные сведения об [управлении подписками Red Hat](https://access.redhat.com/products/red-hat-subscription-management).
* Дополнительные сведения о [рабочей нагрузке Red Hat в Azure](./overview.md).
* Развертывание [JBoss EAP на виртуальной машине RHEL или масштабируемом наборе виртуальных машин из Azure Marketplace](https://aka.ms/AMP-JBoss-EAP).
* Развертывание [JBoss EAP на виртуальной машине RHEL или масштабируемом наборе виртуальных машин с помощью шаблонов быстрого запуска Azure](https://aka.ms/Quickstart-JBoss-EAP).
