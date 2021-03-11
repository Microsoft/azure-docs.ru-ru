---
title: Развертывание первого приложения в Cloud Foundry в Microsoft Azure
description: Сведения о развертывании приложения в Cloud Foundry в Azure
author: seanmck
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 06/14/2017
ms.author: seanmck
ms.openlocfilehash: 037ac972dca49484f7b8c0ad8eed6942c901b997
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102562934"
---
# <a name="deploy-your-first-app-to-cloud-foundry-on-microsoft-azure"></a>Развертывание первого приложения в Cloud Foundry в Microsoft Azure

[Cloud Foundry](https://cloudfoundry.org) — это популярная платформа приложений с открытым кодом в Microsoft Azure. В этой статье рассматривается развертывание приложения в Cloud Foundry в среде Azure и управление им.

## <a name="create-a-cloud-foundry-environment"></a>Создание среды Cloud Foundry

Существует несколько способов создания среды Cloud Foundry в Azure.

- Используйте [предложение Pivotal Cloud Foundry][pcf-azuremarketplace] в Azure Marketplace для создания стандартной среды, содержащей диспетчер операций PCF и компонент Azure Service Broker. [Полные инструкции][pcf-azuremarketplace-pivotaldocs] по развертыванию предложения Marketplace представлены в документации по Pivotal.
- Создайте пользовательскую среду, [развернув Pivotal Cloud Foundry вручную][pcf-custom].
- [Разверните пакеты Cloud Foundry напрямую][oss-cf-bosh], настроив директор [BOSH](https://bosh.io), виртуальную машину, координирующую развертывание среды Cloud Foundry.

> [!IMPORTANT] 
> Если вы развертываете PCF из Azure Marketplace, запишите SYSTEMDOMAINURL и учетные данные администратора, необходимые для доступа к Pivotal Apps Manager (описано в руководстве по развертыванию Marketplace). Эти значения необходимы для работы с этим руководством. Для развертываний Marketplace SYSTEMDOMAINURL имеет форму `https://system.*ip-address*.cf.pcfazure.com` .

## <a name="connect-to-the-cloud-controller"></a>Подключение к Cloud Controller

Cloud Controller — основная точка входа в среду Cloud Foundry для развертывания приложений и управления ими. Основной API Cloud Controller — это REST API. Доступ к нему можно получить с помощью различных средств. В этом случае взаимодействие с ним осуществляется с помощью [интерфейса командной строки Cloud Foundry][cf-cli]. Вы можете установить интерфейс командной строки в Linux, macOS или Windows, но если вы предпочитаете не устанавливать его вообще, то он будет доступен предварительно установленным в [Azure Cloud Shell][cloudshell-docs].

Для входа добавьте `api` перед SYSTEMDOMAINURL, полученным из развертывания Marketplace. Так как развертывание по умолчанию использует самозаверяющий сертификат, следует также добавить параметр `skip-ssl-validation`.

```bash
cf login -a https://api.SYSTEMDOMAINURL --skip-ssl-validation
```

Вам будет предложено войти в Cloud Controller. Используйте учетные данные учетной записи администратора, полученные при развертывании из Marketplace.

Для изоляции команд и сред в общей среде Cloud Foundry предоставляет *организации* и *пространства*, используемые в качестве пространств имен. Развертывание PCF Marketplace содержит *системную* организацию по умолчанию и несколько пространств, созданных для хранения основных компонентов, таких как служба автомасштабирования и Azure Service Broker. Сейчас выберите *системное* пространство.


## <a name="create-an-org-and-space"></a>Создание организации и пространства

Если ввести `cf apps`, отобразится набор системных приложений, развернутых в системном пространстве в системной организации. 

*Системную* организацию необходимо зарезервировать для системных приложений. Создайте организацию и пространство для размещения примера приложения.

```bash
cf create-org myorg
cf create-space dev -o myorg
```

Используйте команду target для перехода к новой организации и пространству:

```bash
cf target -o testorg -s dev
```

Теперь при развертывании приложения оно автоматически создается в новой организации и пространстве. Чтобы убедиться, что в новой организации или пространстве нет приложений, введите `cf apps` еще раз.

> [!NOTE] 
> Дополнительные сведения о организации и пробелах, а также об их использовании для Cloud Foundry управления доступом на основе ролей (Cloud Foundry RBAC) см. в [документации по Cloud Foundry][cf-orgs-spaces-docs].

## <a name="deploy-an-application"></a>Развертывание приложения

Воспользуемся примером приложения Cloud Foundry (Hello Spring Cloud), написанным на языке Java и созданным на основе [Spring Framework](https://spring.io) и [Spring Boot](https://projects.spring.io/spring-boot/).

### <a name="clone-the-hello-spring-cloud-repository"></a>Клонирование репозитория Hello Spring Cloud

Пример приложения Hello Spring Cloud доступен на GitHub. Клонируйте его в свою среду и измените расположение на новый каталог:

```bash
git clone https://github.com/cloudfoundry-samples/hello-spring-cloud
cd hello-spring-cloud
```

### <a name="build-the-application"></a>Построение приложения

Создайте приложение с помощью [Apache Maven](https://maven.apache.org).

```bash
mvn clean package
```

### <a name="deploy-the-application-with-cf-push"></a>Развертывание приложения с помощью команды cf push

Большинство приложений можно развернуть в Cloud Foundry с помощью команды `push`:

```bash
cf push
```

При *принудительной передаче* приложения Cloud Foundry определяет его тип (в данном случае — приложение Java) и зависимости (в данном случае — Spring Framework). Затем это решение упаковывает все необходимые для выполнения кода компоненты в автономный образ контейнера, известный как *дроплет*. Наконец, Cloud Foundry планирует приложение на одном из компьютеров, доступных в среде, и создает URL-адрес, по которому к нему можно получить доступ. Этот URL-адрес доступен в выходных данных команды.

![Выходные данные команды cf push][cf-push-output]

Чтобы просмотреть приложение Hello Spring Cloud, откройте указанный URL-адрес в браузере:

![Пользовательский интерфейс по умолчанию для приложения Hello Spring Cloud][hello-spring-cloud-basic]

> [!NOTE] 
> Дополнительные сведения о том, что происходит при выполнении команды `cf push`, см. в разделе [How Applications Are Staged][cf-push-docs] (Поэтапное развертывание приложений) в документации по Cloud Foundry.

## <a name="view-application-logs"></a>Просмотр журналов приложения

С помощью интерфейса командной строки Cloud Foundry можно просмотреть журналы приложения по имени:

```bash
cf logs hello-spring-cloud
```

По умолчанию команда logs использует *заключительный фрагмент*, который позволяет просмотреть новые записываемые журналы. Чтобы просмотреть новые отображаемые журналы, обновите приложение Hello Spring Cloud в браузере.

Чтобы просмотреть записанные журналы, добавьте параметр `recent`:

```bash
cf logs --recent hello-spring-cloud
```

## <a name="scale-the-application"></a>Масштабирование приложения

По умолчанию команда `cf push` создает только один экземпляр приложения. Чтобы обеспечить высокий уровень доступности и развертывания для поддержки более высокой пропускной способности, обычно требуется запустить несколько экземпляров приложения. Развернутые приложения можно легко масштабировать с помощью команды `scale`:

```bash
cf scale -i 2 hello-spring-cloud
```

Если выполнить команду `cf app` в приложении, мы увидим, что Cloud Foundry создает другой экземпляр приложения. После запуска приложения Cloud Foundry автоматически запускает балансировку нагрузки по трафику к нему.


## <a name="next-steps"></a>Дальнейшие действия

- [Ознакомьтесь с документацией по Cloud Foundry][cloudfoundry-docs]
- [Настройте подключаемый модуль Azure DevOps Services для Cloud Foundry][vsts-plugin]
- [Настройте Microsoft Log Analytics Nozzle для Cloud Foundry][loganalytics-nozzle]

<!-- LINKS -->

[pcf-azuremarketplace]: https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry
[pcf-custom]: https://docs.pivotal.io/pivotalcf/1-10/customizing/azure.html
[oss-cf-bosh]: https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs
[pcf-azuremarketplace-pivotaldocs]: https://docs.pivotal.io/pivotalcf/customizing/pcf_azure.html
[cf-cli]: https://github.com/cloudfoundry/cli
[cloudshell-docs]: ../cloud-shell/overview.md
[cf-orgs-spaces-docs]: https://docs.cloudfoundry.org/concepts/roles.html
[spring-boot]: https://projects.spring.io/spring-boot/
[spring-framework]: https://spring.io
[cf-push-docs]: https://docs.cloudfoundry.org/concepts/how-applications-are-staged.html
[cloudfoundry-docs]: https://docs.cloudfoundry.org
[vsts-plugin]: https://github.com/Microsoft/vsts-cloudfoundry
[loganalytics-nozzle]: https://github.com/Azure/oms-log-analytics-firehose-nozzle

<!-- IMAGES -->
[cf-push-output]: ./media/cloudfoundry-deploy-your-first-app/cf-push-output.png
[hello-spring-cloud-basic]: ./media/cloudfoundry-deploy-your-first-app/hello-spring-cloud-basic.png