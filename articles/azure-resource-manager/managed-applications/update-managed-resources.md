---
title: Обновление управляемых ресурсов
description: В этой статье объясняется процесс работы с ресурсами в управляемой группе ресурсов для управляемого приложения Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 10/26/2017
ms.author: tomfitz
ms.openlocfilehash: 3dbb88d53a961e277e102d7eec580309951c73b6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75651219"
---
# <a name="work-with-resources-in-the-managed-resource-group-for-azure-managed-application"></a>Работа с ресурсами в управляемой группе ресурсов для управляемого приложения Azure

В этой статье объясняется, как обновлять ресурсы, развернутые в рамках управляемого приложения. Издатель управляемого приложения имеет доступ к ресурсам в управляемой группе ресурсов. Чтобы обновить эти ресурсы, найдите управляемую группу ресурсов, которая связана с управляемым приложением, и перейдите к ресурсу в этой группе.

В этой статье предполагается, что вы уже развернули управляемое приложение в примере проекта [Managed Web Application (IaaS) with Azure management services](https://github.com/Azure/azure-managedapp-samples/tree/master/Managed%20Application%20Sample%20Packages/201-managed-web-app) (Управляемое веб-приложение (IaaS) и службы управления Azure). Это управляемое приложение развертывается с виртуальной машиной **Standard_D1_v2**. Если вы не развернули это управляемое приложение, можете продолжить ознакомление с этой статьей, чтобы понять, как обновляется управляемая группа ресурсов.

На рисунке ниже показано развернутое управляемое приложение.

![Развернутое управляемое приложение](./media/update-managed-resources/deployed.png)

В этой статье Azure CLI используется для следующих задач:

* определение управляемого приложения;
* определение управляемой группы ресурсов;
* определение ресурсов виртуальной машины в управляемой группе ресурсов;
* изменение размера виртуальной машины (на меньший размер, если она не используется, или на больший размер для поддержки большей нагрузки);
* назначение управляемой группе ресурсов политики, которая определяет допустимые расположения.

## <a name="get-managed-application-and-managed-resource-group"></a>Получение данных об управляемом приложении и управляемой группе ресурсов

Чтобы получить данные об управляемых приложениях в группе ресурсов, используйте следующий код:

```azurecli-interactive
az managedapp list --query "[?contains(resourceGroup,'DemoApp')]"
```

Чтобы получить идентификатор управляемой группы ресурсов, используйте следующий код:

```azurecli-interactive
az managedapp list --query "[?contains(resourceGroup,'DemoApp')].{ managedResourceGroup:managedResourceGroupId }"
```

## <a name="resize-vms-in-managed-resource-group"></a>Изменение размера виртуальных машин в управляемой группе ресурсов

Чтобы просмотреть виртуальные машины в управляемой группе ресурсов, укажите имя этой группы.

```azurecli-interactive
az vm list -g DemoApp6zkevchqk7sfq --query "[].{VMName:name,OSType:storageProfile.osDisk.osType,VMSize:hardwareProfile.vmSize}"
```

Чтобы обновить размер виртуальной машины, используйте следующий код:

```azurecli-interactive
az vm resize --size Standard_D2_v2 --ids $(az vm list -g DemoApp6zkevchqk7sfq --query "[].id" -o tsv)
```

По завершении операции проверьте, выполняется ли приложение на виртуальной машине D2 v2 категории "Стандартный".

![Управляемое приложение, использующее виртуальную машину D2 v2 категории "Стандартный"](./media/update-managed-resources/upgraded.png)

## <a name="apply-policy-to-managed-resource-group"></a>Применение политики к управляемой группе ресурсов

Получите данные об управляемой группе ресурсов и назначьте ей политику в той же области. **e56962a6-4747-49cd-b67b-bf8b01975c4c** — это встроенная политика для указания допустимых расположений.

```azurecli-interactive
managedGroup=$(az managedapp show --name <app-name> --resource-group DemoApp --query managedResourceGroupId --output tsv)

az policy assignment create --name locationAssignment --policy e56962a6-4747-49cd-b67b-bf8b01975c4c --scope $managedGroup --params '{
                            "listofallowedLocations": {
                                "value": [
                                    "northeurope",
                                    "westeurope"
                                ]
                            }
                        }'
```

Чтобы просмотреть допустимые расположения, используйте следующий код:

```azurecli-interactive
az policy assignment show --name locationAssignment --scope $managedGroup --query parameters.listofallowedLocations.value
```

Сведения о назначении политики отобразятся на портале.

![Просмотр назначения политики](./media/update-managed-resources/assignment.png)

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения об управляемых приложениях Azure см. в разделе [Обзор управляемых приложений Azure](overview.md).
* Примеры проектов см. в статье [Примеры проектов для управляемых приложений Azure](sample-projects.md).
