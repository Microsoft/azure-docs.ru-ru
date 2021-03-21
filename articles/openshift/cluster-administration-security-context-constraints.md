---
title: Управление ограничениями контекста безопасности в Azure Red Hat OpenShift | Документация Майкрософт
description: Ограничения контекста безопасности для администраторов кластера Azure Red Hat OpenShift
services: container-service
author: troy0820
ms.author: b-trconn
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 09/25/2019
ms.openlocfilehash: 977504c1faec9bd8134646a8cbe31f9eea665edd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100636209"
---
# <a name="manage-security-context-constraints-in-azure-red-hat-openshift"></a>Управление ограничениями контекста безопасности в Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3,11 будет прекращена 30 июня 2022. Поддержка для создания новых кластеров Azure Red Hat OpenShift 3,11 продолжится до 30 ноября 2020. После выхода из эксплуатации оставшиеся кластеры Azure Red Hat OpenShift 3,11 будут закрыты для предотвращения уязвимости системы безопасности.
> 
> Следуйте указаниям этого руководством, чтобы [создать кластер Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Если у вас есть определенные вопросы, [свяжитесь с нами](mailto:arofeedback@microsoft.com).

Ограничения контекста безопасности (SCC) позволяют администраторам кластера управлять разрешениями для модулей Pod. Дополнительные сведения об этом типе API см. в [документации по архитектуре для SCC](https://docs.openshift.com/container-platform/3.11/architecture/additional_concepts/authorization.html). Вы можете управлять SCC в экземпляре как обычные объекты API с помощью интерфейса командной строки.

## <a name="list-security-context-constraints"></a>Перечисление ограничений контекста безопасности

Чтобы получить текущий список SCC, используйте следующую команду: 

```bash
$ oc get scc

NAME               PRIV      CAPS      SELINUX     RUNASUSER          FSGROUP     SUPGROUP    PRIORITY   READONLYROOTFS   VOLUMES
anyuid             false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    10         false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
hostaccess         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath persistentVolumeClaim secret]
hostmount-anyuid   false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath nfs persistentVolumeClaim secret]
hostnetwork        false     []        MustRunAs   MustRunAsRange     MustRunAs   MustRunAs   <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
nonroot            false     []        MustRunAs   MustRunAsNonRoot   RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
privileged         true      [*]       RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
restricted         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
```

## <a name="examine-an-object-for-security-context-constraints"></a>Проверка объекта на наличие ограничений контекста безопасности

Чтобы проверить определенную SCC, используйте `oc get` , `oc describe` или `oc edit` .  Например, чтобы проверить **ограниченную** SCC, используйте следующую команду:
```bash
$ oc describe scc restricted
Name:                    restricted
Priority:                <none>
Access:
  Users:                <none>
  Groups:                system:authenticated
Settings:
  Allow Privileged:            false
  Default Add Capabilities:        <none>
  Required Drop Capabilities:        KILL,MKNOD,SYS_CHROOT,SETUID,SETGID
  Allowed Capabilities:            <none>
  Allowed Seccomp Profiles:        <none>
  Allowed Volume Types:            configMap,downwardAPI,emptyDir,persistentVolumeClaim,projected,secret
  Allow Host Network:            false
  Allow Host Ports:            false
  Allow Host PID:            false
  Allow Host IPC:            false
  Read Only Root Filesystem:        false
  Run As User Strategy: MustRunAsRange
    UID:                <none>
    UID Range Min:            <none>
    UID Range Max:            <none>
  SELinux Context Strategy: MustRunAs
    User:                <none>
    Role:                <none>
    Type:                <none>
    Level:                <none>
  FSGroup Strategy: MustRunAs
    Ranges:                <none>
  Supplemental Groups Strategy: RunAsAny
    Ranges:                <none>
```
## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Создание кластера Azure Red Hat OpenShift](tutorial-create-cluster.md) 
