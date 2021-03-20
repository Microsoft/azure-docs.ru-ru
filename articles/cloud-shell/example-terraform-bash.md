---
title: Развертывание с помощью terraform из Azure Cloud Shell | Документация Майкрософт
description: Развертывание с помощью terraform из Azure Cloud Shell
services: Azure
documentationcenter: ''
author: tomarchermsft
manager: routlaw
tags: azure-cloud-shell
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 11/15/2017
ms.author: tarcher
ms.custom: devx-track-terraform
ms.openlocfilehash: e13e6d9ac2f4600e41f221efd23997f712dffc54
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92745443"
---
# <a name="deploy-with-terraform-from-bash-in-azure-cloud-shell"></a>Развертывание с помощью Terraform и Bash в Azure Cloud Shell
В этой статье описывается создание группы ресурсов с помощью [поставщика Terraform AzureRM](https://www.terraform.io/docs/providers/azurerm/index.html).

[Hashicorp Terraform](https://www.terraform.io/) — это средство с открытым кодом, которое кодифицирует API в декларативные файлы конфигурации, которые участники команды могут совместно использовать для редактирования, просмотра и управления версиями. Поставщик Microsoft AzureRM позволяет взаимодействовать с ресурсами, поддерживаемыми Azure Resource Manager через AzureRM API.

## <a name="automatic-authentication"></a>Автоматическая проверка подлинности
Средство Terraform установлено в Bash в Cloud Shell по умолчанию. Кроме того, Cloud Shell автоматически выполняет проверку подлинности вашей стандартной подписки Azure CLI для развертывания ресурсов с помощью модулей Terraform Azure.

Terraform использует стандартную настроенную подписку Azure CLI. Чтобы обновить стандартную подписку, выполните следующую команду:

```azurecli-interactive
az account set --subscription mySubscriptionName
```

## <a name="walkthrough"></a>Пошаговое руководство
### <a name="launch-bash-in-cloud-shell"></a>Запуск Bash в Cloud Shell.
1. Запуск Cloud Shell из предпочтительного расположения.
2. Проверка настройки предпочтительной подписки.

```azurecli-interactive
az account show
```

### <a name="create-a-terraform-template"></a>Создание шаблона Terraform
Создайте шаблон Terraform с именем main.tf с помощью предпочтительного текстового редактора.

```
vim main.tf
```

Скопируйте и вставьте указанный ниже код в Cloud Shell.

```
resource "azurerm_resource_group" "myterraformgroup" {
    name = "myRgName"
    location = "West US"
}
```

Сохраните файл и закройте текстовый редактор.

### <a name="terraform-init"></a>Terraform Init
Начните с выполнения команды `terraform init`.

```
justin@Azure:~$ terraform init

Initializing provider plugins...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.azurerm: version = "~> 0.2"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

[Команду terraform init](https://www.terraform.io/docs/commands/init.html) используют для инициализации рабочей папки, содержащей файл конфигурации Terraform. Команду `terraform init` следует выполнить первой после создания конфигурации Terraform или клонирования имеющейся конфигурации из системы управления версиями. Выполнять эту команду несколько раз не опасно.

### <a name="terraform-plan"></a>План Terraform
Предварительно просмотрите ресурсы, которые будут созданы шаблоном Terraform с помощью `terraform plan`.

```
justin@Azure:~$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + azurerm_resource_group.demo
      id:       <computed>
      location: "westus"
      name:     "myRGName"
      tags.%:   <computed>


Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

Команда [terraform plan](https://www.terraform.io/docs/commands/plan.html) создает план выполнения. Terraform выполняет обновление, если это не отключено явным образом, а затем определяет, какие действия необходимы для достижения желаемого состояния, указанного в файлах конфигурации. План можно сохранить с помощью команды -out. Затем он предоставляется для применения Terraform, чтобы убедиться, что выполняются только заранее запланированные действия.

### <a name="terraform-apply"></a>Применение Terraform
Подготовьте ресурсы Azure с помощью `terraform apply`.

```
justin@Azure:~$ terraform apply
azurerm_resource_group.demo: Creating...
  location: "" => "westus"
  name:     "" => "myRGName"
  tags.%:   "" => "<computed>"
azurerm_resource_group.demo: Creation complete after 0s (ID: /subscriptions/mySubIDmysub/resourceGroups/myRGName)

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

[Команда terraform apply](https://www.terraform.io/docs/commands/apply.html) используется для применения изменений, необходимых для достижения требуемого состояния конфигурации.

### <a name="verify-deployment-with-azure-cli"></a>Проверка развертывания с помощью Azure CLI
Выполните `az group show -n myRgName`, чтобы проверить, что ресурс был успешно подготовлен.

```azurecli-interactive
az group show -n myRgName
```

### <a name="clean-up-with-terraform-destroy"></a>Очистка с помощью команды terraform destroy
Очистите группу ресурсов, созданную с помощью команды [Terraform destroy](https://www.terraform.io/docs/commands/destroy.html), чтобы очистить инфраструктуру, созданную Terraform.

```
justin@Azure:~$ terraform destroy
azurerm_resource_group.demo: Refreshing state... (ID: /subscriptions/mySubID/resourceGroups/myRGName)

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  - azurerm_resource_group.demo


Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

azurerm_resource_group.demo: Destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 10s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 20s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 30s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 40s elapsed)
azurerm_resource_group.demo: Destruction complete after 45s

Destroy complete! Resources: 1 destroyed.
```

Ресурс Azure успешно создан с помощью Terraform. Ознакомьтесь со следующими действиями, чтобы продолжить изучение Cloud Shell.

## <a name="next-steps"></a>Дальнейшие действия
[Microsoft AzureRM Provider](https://www.terraform.io/docs/providers/azurerm/#) (Поставщик Microsoft AzureRM)<br>
[Краткое руководство по использованию Bash в Azure Cloud Shell](quickstart.md)
