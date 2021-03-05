---
title: Создание восстановления приложения кластера Azure Red Hat OpenShift 4 с помощью Велеро
description: Узнайте, как создать восстановление кластерных приложений Azure Red Hat OpenShift с помощью Велеро.
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 06/22/2020
author: troy0820
ms.author: b-trconn
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc
ms.openlocfilehash: 8a9bb191390056caf7d5ba3b42c278205177d5a2
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102213019"
---
# <a name="create-an-azure-red-hat-openshift-4-cluster-application-restore"></a>Создание восстановления приложения кластера Azure Red Hat OpenShift 4

В этой статье вы подготовите среду для создания восстановления приложения кластера Azure Red Hat OpenShift 4. Вы узнаете, как:

> [!div class="checklist"]
> * Настройка необходимых компонентов и установка необходимых средств
> * Создание приложения Azure Red Hat OpenShift 4 для восстановления

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.6.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Перед началом

### <a name="create-an-azure-red-hat-openshift-4-application-backup"></a>Создание резервной копии приложений Azure Red Hat OpenShift 4

Сведения о создании резервной копии приложений Azure Red Hat OpenShift 4 см. в статье [Создание резервной копии Azure Red Hat OpenShift 4](./howto-create-a-backup.md) .

## <a name="restore-an-azure-red-hat-openshift-4-application"></a>Восстановление приложения Azure Red Hat OpenShift 4

Эти действия позволяют восстановить приложение, для которого ранее была создана резервная копия с помощью Велеро.
Чтобы узнать, какие резервные копии доступны для восстановления, можно проверить список резервных копий, распознаваемых кластером в настоящее время.  Для выполнения этого шага необходимо выполнить следующую команду:

_(Для этого шага предполагается, что вы установили Велеро в проекте с именем "велеро")._

```bash
oc get backups -n velero
```

После создания резервной копии, которую вы хотите восстановить, необходимо выполнить восстановление с помощью следующей команды:

```bash
velero restore create <name of restore> --from-backup <name of backup from above output list>
```

На этом шаге будут созданы объекты Kubernetes, которые были созданы из предыдущего шага при создании резервной копии.

Чтобы просмотреть состояние восстановления, выполните следующий шаг:

```bash
oc get restore -n velero <name of restore created previously> -o yaml
```
На этом этапе `Completed` необходимо восстановить приложение Azure Red Hat 4.

## <a name="restore-an-azure-red-hat-openshift-4-application-with-included-snapshots"></a>Восстановление приложения Azure Red Hat OpenShift 4 с добавленными моментальными снимками


Чтобы создать восстановление приложения Azure Red Hat OpenShift 4 с постоянными томами с помощью Велеро, необходимо выполнить восстановление с помощью следующей команды:

```bash
velero restore create <name of the restore> --from-backup <name of backup from above output list> --exclude-resources="nodes,events,events.events.k8s.io,backups.ark.heptio.com,backups.velero.io,restores.ark.heptio.com,restores.velero.io"
```
На этом шаге будут созданы объекты Kubernetes, которые были созданы из предыдущего шага при создании резервной копии.

Чтобы просмотреть состояние восстановления, выполните следующий шаг:

```bash
oc get restore -n velero <name of restore created previously> -o yaml
```
На этом этапе `Completed` необходимо восстановить приложение Azure Red Hat 4.

Дополнительные сведения о создании резервных копий и восстановлении с помощью Велеро см. [в статье резервное копирование ресурсов OpenShift в собственном виде](https://www.openshift.com/blog/backup-openshift-resources-the-native-way) .

## <a name="next-steps"></a>Дальнейшие действия

В этой статье было восстановлено приложение кластера Azure Red Hat OpenShift 4. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Создание восстановления приложения кластера OpenShift v4 с помощью Велеро
> * Создание восстановления приложения кластера OpenShift v4 с помощью моментальных снимков с использованием Велеро


Перейдите к следующей статье, чтобы узнать о поддерживаемых ресурсах Azure Red Hat OpenShift 4.

* [Поддерживаемые ресурсы Azure Red Hat OpenShift v4](supported-resources.md)