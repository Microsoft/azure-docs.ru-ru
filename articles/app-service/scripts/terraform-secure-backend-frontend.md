---
title: Terraform. Развертывание внешнего и внутреннего веб-приложения, безопасно подключенных с помощью интеграции с виртуальной сетью и частной конечной точки
description: Узнайте, как использовать поставщик Terraform для Службы приложений, чтобы развертывать два веб-приложения, безопасно подключенных с помощью частной конечной точки и интеграции с виртуальной сетью
author: ericgre
ms.assetid: 3e5d1bbd-5581-40cc-8f65-bc74f1802156
ms.topic: sample
ms.date: 08/10/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.openlocfilehash: c1de8ebbd9ad381628cfeb19413baa295b42b3db
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91739839"
---
# <a name="create-two-web-apps-connected-securely-with-private-endpoint-and-vnet-integration"></a>Создание безопасного подключения двух веб-приложений с помощью частной конечной точки и интеграции с виртуальной сетью

В этой статье показано, как использовать [частную конечную точку](../networking/private-endpoint.md) и [интеграцию с локальной виртуальной сетью](../web-sites-integrate-with-vnet.md) для безопасного подключения двух веб-приложений (внешнего и внутреннего) с помощью следующих действий:
- Развертывание виртуальной сети
- Создание первой подсети для интеграции
- Создание второй подсети для частной конечной точки и задание конкретного параметра для отключения сетевых политик
- Развертывание одного плана Службы приложений категории "Премиум 2" или "Премиум 3", необходимого для функции "Частная конечная точка"
- Создание внешнего веб-приложения с конкретными параметрами приложения для использования частной зоны DNS ([больше](../web-sites-integrate-with-vnet.md#azure-dns-private-zones))
- Подключение внешнего веб-приложения к подсети интеграции
- Создание внутреннего веб-приложения
- Создание частной зоны DNS с именем зоны приватного канала для веб-приложения privatelink.azurewebsites.net
- Связывание этой зоны с виртуальной сетью
- Создание частной конечной точки для внутреннего веб-приложения в подсети конечной точки и регистрация имен DNS (веб-сайт и SCM) в ранее созданной частной зоне DNS

## <a name="how-to-use-terraform-in-azure"></a>Как использовать Terraform в Azure

Перейдите к [документации по Azure](/azure/developer/terraform/), чтобы получить сведения об использовании Terraform в Azure.

## <a name="the-complete-terraform-file"></a>Полный файл Terraform

Чтобы использовать этот файл, необходимо изменить свойство name для ресурсов frontwebapp и backwebapp (имя webapp должно быть глобально уникальным DNS-именем). 

```hcl
provider "azurerm" {
  version = "~>2.0"
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "appservice-rg"
  location = "francecentral"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "vnet"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  address_space       = ["10.0.0.0/16"]
}

resource "azurerm_subnet" "integrationsubnet" {
  name                 = "integrationsubnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
  delegation {
    name = "delegation"
    service_delegation {
      name = "Microsoft.Web/serverFarms"
    }
  }
}

resource "azurerm_subnet" "endpointsubnet" {
  name                 = "endpointsubnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.2.0/24"]
  enforce_private_link_endpoint_network_policies = true
}

resource "azurerm_app_service_plan" "appserviceplan" {
  name                = "appserviceplan"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  sku {
    tier = "Premiumv2"
    size = "P1v2"
  }
}

resource "azurerm_app_service" "frontwebapp" {
  name                = "frontwebapp20200810"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  app_service_plan_id = azurerm_app_service_plan.appserviceplan.id

  app_settings = {
    "WEBSITE_DNS_SERVER": "168.63.129.16",
    "WEBSITE_VNET_ROUTE_ALL": "1"
  }
}

resource "azurerm_app_service_virtual_network_swift_connection" "vnetintegrationconnection" {
  app_service_id  = azurerm_app_service.frontwebapp.id
  subnet_id       = azurerm_subnet.integrationsubnet.id
}

resource "azurerm_app_service" "backwebapp" {
  name                = "backwebapp20200810"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  app_service_plan_id = azurerm_app_service_plan.appserviceplan.id
}

resource "azurerm_private_dns_zone" "dnsprivatezone" {
  name                = "privatelink.azurewebsites.net"
  resource_group_name = azurerm_resource_group.rg.name
}

resource "azurerm_private_dns_zone_virtual_network_link" "dnszonelink" {
  name = "dnszonelink"
  resource_group_name = azurerm_resource_group.rg.name
  private_dns_zone_name = azurerm_private_dns_zone.dnsprivatezone.name
  virtual_network_id = azurerm_virtual_network.vnet.id
}

resource "azurerm_private_endpoint" "privateendpoint" {
  name                = "backwebappprivateendpoint"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  subnet_id           = azurerm_subnet.endpointsubnet.id

  private_dns_zone_group {
    name = "privatednszonegroup"
    private_dns_zone_ids = [azurerm_private_dns_zone.dnsprivatezone.id]
  }

  private_service_connection {
    name = "privateendpointconnection"
    private_connection_resource_id = azurerm_app_service.backwebapp.id
    subresource_names = ["sites"]
    is_manual_connection = false
  }
}
```




## <a name="next-steps"></a>Дальнейшие действия


> [Документация по Terraform в Azure](/azure/developer/terraform/)