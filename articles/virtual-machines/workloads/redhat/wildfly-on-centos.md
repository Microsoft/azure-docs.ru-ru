---
title: Краткое руководство по использованию WildFly в CentOS
description: Развертывание приложений Java в WildFly на виртуальной машине CentOS
author: Theresa-Nguyen
ms.author: bicnguy
ms.date: 10/23/2020
ms.assetid: 7aa21ef8-9cfb-43e0-bfda-3f10a2a2f3ef
ms.topic: quickstart
ms.service: virtual-machines
ms.subservice: redhat
ms.custom:
- mode-api
ms.collection: linux
ms.openlocfilehash: bad9331a6d9a0c0936b1458e3aa4a84559e4a107
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533563"
---
# <a name="quickstart-wildfly-on-centos-8"></a>Краткое руководство. Использование WildFly в CentOS 8

В этом кратком руководстве показано, как развернуть автономный узел WildFly на виртуальной машине CentOS 8. Это идеальный вариант для разработки и тестирования корпоративных приложений Java в Azure. Для развертывания в этом кратком руководстве подписка сервера приложений не требуется.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) или [создать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial).

## <a name="use-case"></a>Вариант использования

WildFly — это идеальный вариант для разработки и тестирования корпоративных приложений Java в Azure. Списки технологий, доступных в профилях конфигурации сервера WildFly 18, доступны в документации по [началу работы с WildFly](https://docs.wildfly.org/18/Getting_Started_Guide.html#getting-started-with-wildfly).

WildFly может запускаться как в автономном, так и в кластерном режиме в зависимости от вашего варианта использования. Вы можете обеспечить высокий уровень доступности критических приложений Jakarta EE, используя WildFly в кластере узлов. Внесите небольшое количество изменений в конфигурацию приложения, а затем разверните приложение в кластере. Чтобы узнать больше о высоком уровне доступности, ознакомьтесь с [этой документацией](https://docs.wildfly.org/18/High_Availability_Guide.html).

## <a name="configuration-choice"></a>Вариант конфигурации

WildFly можно загрузить в режиме **отдельного сервера**. Экземпляр отдельного сервера — это независимый процесс, очень похожий на экземпляр сервера приложений JBoss 3, 4, 5 или 6. Автономные экземпляры можно запустить с помощью сценариев запуска standalone.sh или standalone.bat. Для более чем одного автономного экземпляра ответственность за координацию управления несколькими серверами на серверах несет пользователь.

Кроме того, можно запустить экземпляр WildFly с альтернативной конфигурацией, используя файлы конфигурации, доступные в папке с настройками.

Ниже приведены файлы конфигурации отдельного сервера.

- standalone.xml (по умолчанию). Эта конфигурация является файлом по умолчанию, используемым для запуска экземпляра WildFly. Он содержит сертифицированную конфигурацию Jakarta Web Profile с необходимыми функциями.
   
- standalone-ha.xml. Сертифицированная конфигурация Jakarta EE Web Profile 8 с высоким уровнем доступности (ориентированная на веб-приложения).
   
- standalone-full.xml. Сертифицированная конфигурация Jakarta EE Platform 8, включая все необходимые функции для размещения приложений Jakarta EE.

- standalone-full-ha.xml. Сертифицированная конфигурация Jakarta EE Platform 8 с высоким уровнем доступности для размещения приложений Jakarta EE.

Чтобы запустить автономный сервер WildFly с другой предоставленной конфигурацией, используйте аргумент --server-config в файле конфигурации сервера.

Например, чтобы использовать Jakarta EE Platform 8 с возможностями кластеризации, используйте следующую команду:

```
./standalone.sh --server-config=standalone-full-ha.xml
```

Чтобы узнать больше о конфигурациях, ознакомьтесь с документацией по [началу работы с WildFly](https://docs.wildfly.org/18/Getting_Started_Guide.html#wildfly-10-configurations).

## <a name="licensing-support-and-subscription-notes"></a>Рекомендации по лицензированию, поддержке и подпискам

Образ Azure CentOS 8 — это образ виртуальной машины с оплатой по мере использования (PAYG), который не требует от пользователя получения лицензии. При первом запуске виртуальной машины лицензированная ОС виртуальной машины будет автоматически активирована и начнет взиматься почасовая оплата. Это в дополнение к почасовой оплате за виртуальные машины Майкрософт на платформе Linux. Чтобы получить дополнительные сведения, перейдите на страницу [Цены на виртуальные машины Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#linux). WildFly можно скачать и использовать без подписки Red Hat или лицензии.

## <a name="how-to-consume"></a>Как использовать

Вы можете развернуть шаблон следующими тремя способами.

- Чтобы развернуть этот шаблон с помощью PowerShell, выполните следующие команды. (Сведения об установке и настройке Azure PowerShell см. в [соответствующей документации](/powershell/azure/).)

    ```
    New-AzResourceGroup -Name <resource-group-name> -Location <resource-group-location> #use this command when you need to create a new Resource Group for your deployment
    ```

    ```
    New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/wildfly-standalone-centos8/azuredeploy.json
    ```
    
- Чтобы развернуть этот шаблон с помощью Azure CLI, выполните следующие команды. (Дополнительные сведения об установке и настройке межплатформенного интерфейса командной строки Azure см. в [этой статье](/cli/azure/install-azure-cli).)

    ```
    az group create --name <resource-group-name> --location <resource-group-location> #use this command when you need to create a new Resource Group for your deployment
    ```

    ```
    az deployment group create --resource-group <my-resource-group> --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/wildfly-standalone-centos8/azuredeploy.json
    ```

- Чтобы развернуть этот шаблон с помощью портала Azure, щелкните <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fwildfly-standalone-centos8%2Fazuredeploy.json" target="_blank">здесь</a>, а затем войдите на портал Azure.

## <a name="arm-template"></a>Шаблон ARM

<a href="https://github.com/Azure/azure-quickstart-templates/tree/master/wildfly-standalone-centos8" target="_blank">WildFly 18 в CentOS 8 (автономная виртуальная машина)</a> — это шаблон быстрого запуска, который создает автономный узел WildFly 18.0.1.Final на виртуальной машине CentOS 8 в группе ресурсов, которая включает в себя частный IP-адрес для виртуальной машины, виртуальную сеть и учетную запись хранения для диагностики. Он также развертывает пример приложения Java с именем JBoss-EAP в Azure для WildFly.

## <a name="resource-links"></a>Ссылки на ресурсы

* [WildFly 18](https://docs.wildfly.org/18/)
* [Рекомендуемые дистрибутивы Linux в Azure](../../linux/endorsed-distros.md)
* [Документация для разработчика Java в Azure](https://github.com/JasonFreeberg/jboss-on-app-service)

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать подробные сведения о рабочей среде, ознакомьтесь с шаблонами быстрого запуска Azure для JBoss EAP в Red Hat:

Автономная виртуальная машина RHEL с примером приложения:

*  <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/jboss-eap-standalone-rhel" target="_blank"> JBoss EAP в RHEL (автономная виртуальная машина)</a>

Кластеризованные виртуальные машины RHEL с примером приложения:

* <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/jboss-eap-clustered-multivm-rhel" target="_blank"> JBoss EAP в RHEL (кластеризованные виртуальные машины)</a>

Масштабируемый набор кластеризованных виртуальных машин RHEL с примером приложения:

* <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/jboss-eap-clustered-vmss-rhel" target="_blank"> JBoss EAP в RHEL (масштабируемый набор кластеризованных виртуальных машин)</a>
