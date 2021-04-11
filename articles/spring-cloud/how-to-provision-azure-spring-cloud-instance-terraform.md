---
title: Подготовка экземпляра Azure Spring Cloud с помощью Terraform
description: Подготовка экземпляра Azure Spring Cloud с помощью Terraform.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 06/26/2020
ms.custom: devx-track-java
ms.openlocfilehash: 060ef2d08b849706b47b24748142c608292971b5
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104878456"
---
# <a name="provision-an-azure-spring-cloud-instance-with-terraform"></a>Подготовка экземпляра Azure Spring Cloud с помощью Terraform

**Эта статья применима к:** ✔️ Java ✔️ C#

В этом примере с помощью Terraform создается экземпляр Azure Spring Cloud. Приведенные здесь процедуры помогут вам создать следующие ресурсы:

> [!div class="checklist"]
> * Группа ресурсов
> * экземпляр Azure Spring Cloud;
> * служба хранилища Azure для Log Analytics.

> [!NOTE]
> Чтобы получить поддержку по Terraform, используйте один из каналов службы поддержки Terraform в сообществе HashiCorp.
>
> * Вопросы, варианты использования и полезные шаблоны: [раздел Terraform на портале сообщества HashiCorp](https://discuss.hashicorp.com/c/terraform-core).
> * Вопросы о поставщиках: [раздел с информацией о поставщиках Terraform на портале сообщества HashiCorp](https://discuss.hashicorp.com/c/terraform-providers).

## <a name="prerequisites"></a>Предварительные требования

- **Подписка Azure**: Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начинать работу.

## <a name="create-configuration-file"></a>Создание файла конфигурации

1. Войдите на [портал Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Откройте [Azure Cloud Shell](../app-service/quickstart-java.md#use-azure-cloud-shell).

1. Запустите редактор Cloud Shell:

    ```bash
    code main.tf
    ```

1. Конфигурация на этом шаге создает модель ресурсов Azure, в том числе группы ресурсов Azure и экземпляра Azure Spring Cloud.

    ```hcl
    provider "azurerm" {
        features {}
    }

    resource "azurerm_resource_group" "example" {
      name     = "username"
      location = "eastus"
    }

    resource "azurerm_spring_cloud_service" "example" {
      name                = "usernamesp"
      resource_group_name = azurerm_resource_group.example.name
      location            = azurerm_resource_group.example.location

      config_server_git_setting {
        uri          = "https://github.com/Azure-Samples/piggymetrics-config"
        label        = "master"
        search_paths = ["."]
      }
    }
    ```

1. Сохраните файл ( **&lt;Ctrl>S**) и закройте редактор ( **&lt;Ctrl>Q**).

## <a name="apply-the-configuration"></a>Применение конфигурации

В этом разделе вы используете несколько команд Terraform для запуска конфигурации.

1. Команда [terraform init](https://www.terraform.io/docs/commands/init.html) инициализирует рабочий каталог. Выполните следующую команду в Cloud Shell:

    ```bash
    terraform init
    ```

1. Команда [terraform plan](https://www.terraform.io/docs/commands/plan.html) используется для проверки синтаксиса конфигурации. Параметр `-out` направляет результаты в файл. Выходной файл можно использовать позже, чтобы применить конфигурацию. Выполните следующую команду в Cloud Shell:

    ```bash
    terraform plan --out plan.out
    ```

1. Команда [terraform apply](https://www.terraform.io/docs/commands/apply.html) используется для применения конфигурации. Эта команда указывает выходной файл из предыдущего шага. Эта команда создает ресурсы Azure. Выполните следующую команду в Cloud Shell:

    ```bash
    terraform apply plan.out
    ```

1. Перейдите к новой группе ресурсов, чтобы проверить результаты на портале Azure. Новый экземпляр **Azure Spring Cloud** появится в новой группе ресурсов.

## <a name="update-configuration-to-config-logs-and-metrics"></a>Обновление конфигурации для включения журналов и метрик

В этом разделе описано, как обновить конфигурацию для включения журналов и метрик для Azure Spring Cloud с помощью учетной записи службы хранилища Azure.

1. Запустите редактор Cloud Shell:

    ```bash
    code main.tf
    ```

1. Добавьте следующую конфигурацию в конец файла:

    ```hcl
    resource "azurerm_storage_account" "example" {
      name                     = "usernamest"
      resource_group_name      = azurerm_resource_group.example.name
      location                 = azurerm_resource_group.example.location
      account_tier             = "Standard"
      account_replication_type = "GRS"
    }

    resource "azurerm_monitor_diagnostic_setting" "example" {
      name               = "example"
      target_resource_id = "${azurerm_spring_cloud_service.example.id}"
      storage_account_id = "${azurerm_storage_account.example.id}"

      log {
        category = "SystemLogs"
        enabled  = true

        retention_policy {
          enabled = false
        }
      }

      metric {
        category = "AllMetrics"

        retention_policy {
          enabled = false
        }
      }
    }
    ```

1. Сохраните файл ( **&lt;Ctrl>S**) и закройте редактор ( **&lt;Ctrl>Q**).

1. Как и в предыдущем разделе, выполните следующую команду, чтобы внести изменения:

    ```bash
    terraform plan --out plan.out
    ```

1. Наконец, запустите команду `terraform apply`, чтобы применить конфигурацию.

    ```bash
    terraform apply plan.out
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите ресурсы Azure, созданные в рамках этой статьи, если они вам больше не нужны.

Чтобы удалить ресурсы Azure, созданные в рамках этого упражнения, выполните команду [terraform destroy](https://www.terraform.io/docs/commands/destroy.html):

```bash
terraform destroy -auto-approve
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Install and configure Terraform to provision Azure resources](/azure/developer/terraform/getting-started-cloud-shell) (Установка и настройка Terraform для подготовки ресурсов Azure)