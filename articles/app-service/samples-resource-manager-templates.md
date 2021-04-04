---
title: Примеры шаблонов Azure Resource Manager
description: В этой статье приведены примеры шаблонов Azure Resource Manager для некоторых распространенных сценариев Службы приложений. Узнайте, как автоматизировать задачи развертывания и управления в Службе приложений.
author: tfitzmac
tags: azure-service-management
ms.topic: sample
ms.date: 08/26/2020
ms.author: tomfitz
ms.custom: mvc, fasttrack-edit
ms.openlocfilehash: 5129fccceb633991767cdd3b52bbb5b6af067270
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88891051"
---
# <a name="azure-resource-manager-templates-for-app-service"></a>Шаблоны Azure Resource Manager для Службы приложений

В следующей таблице представлены ссылки на шаблоны Azure Resource Manager для Службы приложений Azure. Рекомендации по предотвращению распространенных ошибок при создании шаблонов приложений см. в статье [Руководство по развертыванию веб-приложений с помощью шаблонов Azure Resource Manager](deploy-resource-manager-template.md).

Синтаксис и свойства JSON для ресурсов Службы приложений см. в статье о [типах ресурсов Microsoft.Web](/azure/templates/microsoft.web/allversions).

| Развертывание приложения | Описание |
|-|-|
| [План службы приложений и базовое приложение Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-basic-linux) | Развертывает приложение Службы приложений, настроенное для Linux. |
| [План службы приложений и базовое приложение Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-basic-windows) | Развертывает приложение Службы приложений, настроенное для Windows. |
| [Приложение, связанное с репозиторием GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-github-deploy)| Развертывает приложение Службы приложений Azure, которое вытягивает код из GitHub. |
| [Приложение с пользовательскими слотами развертывания](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-custom-deployment-slots)| Развертывает приложение Службы приложений с пользовательскими слотами развертывания или средами. |
| [Приложение с частной конечной точкой](https://github.com/Azure/azure-quickstart-templates/tree/master/101-private-endpoint-webapp)| Развертывает приложение в Службе приложений с частной конечной точкой. |
|**Настройка приложения**| **Описание** |
| [Сертификат приложения из Key Vault](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-certificate-from-key-vault)| Позволяет развернуть сертификат приложения Службы приложений из секрета Azure Key Vault и использует его для привязки TLS/SSL. |
| [Приложение с личным доменом и SSL](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-custom-domain-and-ssl)| Позволяет развернуть приложение Службы приложений с пользовательским именем узла и получает сертификат приложения из Key Vault для привязки TLS/SSL. |
| [Приложение с расширением GoLang](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-with-golang)| Развертывает приложение Службы приложений с расширением сайта Golang. Вы сможете запускать веб-приложения, разработанные на Golang в Azure. |
| [Приложения с Java 8 и Tomcat 8](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-java-tomcat)| Развертывает приложение Службы приложений с поддержкой Java 8 и Tomcat 8. Вы сможете запускать приложения Java в Azure. |
| [Приложение с поддержкой интеграции с региональной виртуальной сетью](https://github.com/Azure/azure-quickstart-templates/tree/master/101-app-service-regional-vnet-integration)| Позволяет развернуть приложение Службы приложений с поддержкой интеграции с региональной виртуальной сетью. |
|**Защита приложения**| **Описание** |
| [Приложение, интегрированное с Шлюзом приложений Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-app-gateway-v2)| Развертывает приложение Службы приложений и Шлюз приложений, а также изолирует трафик, используя конечную точку службы и ограничения доступа. |
|**Приложение с подключенными ресурсами**| **Описание** |
| [Приложение в Linux с MySQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-mysql) | Развертывает приложение Службы приложений в Linux со службой "База данных Azure для MySQL". |
| [Приложения в Linux с PostgreSQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-postgresql) | Развертывает приложение Службы приложений в Linux со службой "База данных Azure для PostgreSQL". |
|**Приложение с подключенными ресурсами**| **Описание** |
| [Приложение с MySQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-mysql)| Развертывает приложение Службы приложений в Windows со службой "База данных Azure для MySQL". |
| [Приложение с PostgreSQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-postgresql)| Развертывает приложение Службы приложений в Windows со службой "База данных Azure для PostgreSQL". |
| [Приложение с базой данных в службе "База данных SQL Azure"](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-sql-database)| Развертывает приложение Службы приложений и базу данных в службе "База данных SQL Azure" с уровнем обслуживания "Базовый". |
| [Приложение с подключением к хранилищу BLOB-объектов](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-blob-connection)| Развертывает приложение Службы приложений со строкой подключения к хранилищу BLOB-объектов Azure. Вы сможете использовать хранилище BLOB-объектов из приложения. |
| [Приложение с кэшем Azure для Redis](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-redis-cache)| Развертывает приложение Службы приложений с кэшем Azure для Redis. |
| [Приложение, подключенное к внутреннему веб-приложению](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-privateendpoint-vnet-injection)| Развертывает два веб-приложения (внешнее и внутреннее), безопасно подключенных друг к другу с помощью внедрения виртуальной сети и частной конечной точки. |
|**Среда службы приложений**| **Описание** |
| [Создание среды службы приложений версии 2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-create) | Позволяет создать среду службы приложений версии 2 в виртуальной сети. |
| [Создание среды службы приложений версии 2 с адресом ILB](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-ilb-create/) | Позволяет создать среду службы приложений версии 2 в виртуальной сети с адресом частной внутренней подсистемы балансировки нагрузки. |
| [Настройка SSL-сертификата по умолчанию для среды службы приложений ILB версии 1 или 2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-ase-ilb-configure-default-ssl) | Позволяет настроить TLS/SSL-сертификат по умолчанию для среды службы приложений ILB версии 1 или 2. |
| | |
