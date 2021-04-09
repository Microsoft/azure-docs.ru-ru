---
title: Примеры для Azure PowerShell. Сброс Облачных служб Azure (расширенная поддержка)
description: Примеры скриптов для сброса развертывания Облачной службы Azure (расширенная поддержка)
ms.topic: sample
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: 5c43d61b1e7cd98674eab4c6d857cc1114a06013
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99475325"
---
# <a name="reset-an-azure-cloud-service-extended-support"></a>Сброс Облачной службы Azure (расширенная поддержка) 
В этих примерах рассматриваются различные способы для сброса существующего развертывания Облачной службы Azure (расширенная поддержка).

## <a name="reimage-role-instances-of-cloud-service"></a>Пересоздание образов для экземпляров ролей облачной службы
```powershell
$roleInstances = @("ContosoFrontEnd_IN_0", "ContosoBackEnd_IN_1")
Invoke-AzCloudServiceReimage -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance $roleInstances
```
Эта команда пересоздает образы для 2 экземпляров ролей ContosoFrontEndIN_0 и ContosoBackEndIN_1 облачной службы с именем ContosoCS, которая принадлежит группе ресурсов с именем ContosOrg.

## <a name="reimage-all-roles-of-cloud-service"></a>Пересоздание образа для всех ролей Облачной службы
```powershell
Invoke-AzCloudServiceReimage -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance "*"
```
Эта команда пересоздает образы для всех экземпляров ролей облачной службы с именем ContosoCS, которая принадлежит группе ресурсов с именем ContosOrg.

## <a name="reimage-a-single-role-instance-of-a-cloud-service"></a>Пересоздание образа для отдельного экземпляра ролей Облачной службы
```powershell
Invoke-AzCloudServiceRoleInstanceReimage -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstanceName "ContosoFrontEnd_IN_0"
```
Эта команда пересоздает образы для экземпляра роли ContosoFrontEnd_IN_0 облачной службы с именем ContosoCS, которая принадлежит группе ресурсов с именем ContosOrg.

## <a name="rebuild-role-instances-of-cloud-service"></a>Перестройка для экземпляров ролей облачной службы
```powershell
$roleInstances = @("ContosoFrontEnd_IN_0", "ContosoBackEnd_IN_1")
Invoke-AzCloudServiceRebuild -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance $roleInstances
```
Эта команда перестраивает 2 экземпляра ролей ContosoFrontEndIN_0 и ContosoBackEndIN_1 облачной службы с именем ContosoCS, которая принадлежит группе ресурсов с именем ContosOrg.

## <a name="rebuild-all-roles-of-cloud-service"></a>Перестройка всех ролей облачной службы
```powershell
Invoke-AzCloudServiceRebuild -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance "*"
```
Эта команда перестраивает все экземпляры ролей облачной службы с именем ContosoCS, которая принадлежит группе ресурсов с именем ContosOrg.

## <a name="restart-role-instances-of-cloud-service"></a>Перезапуск экземпляров ролей облачной службы
```powershell
$roleInstances = @("ContosoFrontEnd_IN_0", "ContosoBackEnd_IN_1")
Restart-AzCloudService -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance $roleInstances
```
Эта команда перезапускает 2 экземпляра ролей ContosoFrontEndIN_0 и ContosoBackEndIN_1 облачной службы с именем ContosoCS, которая принадлежит группе ресурсов с именем ContosOrg.

## <a name="restart-all-roles-of-cloud-service"></a>Перезапуск всех ролей облачной службы
```powershell
Restart-AzCloudService -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance "*"
```
Эта команда перезапускает всех экземпляры ролей облачной службы с именем ContosoCS, которая принадлежит группе ресурсов с именем ContosOrg.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об Облачных службах Azure (расширенная поддержка) см. в статье [Общие сведения об Облачных службах Azure (расширенная поддержка)](overview.md).
- Перейдите в [репозиторий примеров для Облачных служб (расширенная поддержка) ](https://github.com/Azure-Samples/cloud-services-extended-support)
