---
title: Обновление кластера Azure Red Hat OpenShift
description: Узнайте, как обновить кластер Azure Red Hat OpenShift, работающий под OpenShift 4
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 1/10/2021
author: sakthi-vetrivel
ms.author: suvetriv
keywords: aro, openshift, az aro, red hat, cli
ms.openlocfilehash: 742da12bd3a10cd1f541e9c43f654cfe7df04340
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101720891"
---
# <a name="upgrade-an-azure-red-hat-openshift-aro-cluster"></a>Обновление кластера Azure Red Hat OpenShift (АТО)

В рамках жизненного цикла кластера АТО выполняется периодическое обновление до последней версии OpenShift. Важно установить последние версии системы безопасности или обновить, чтобы получить новейшие функции. В этой статье показано, как обновить все компоненты в кластере OpenShift с помощью веб-консоли OpenShift.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что вы используете Azure CLI версии 2.0.65 более поздней. Чтобы узнать, какая версия используется сейчас, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, ознакомьтесь со статьей [Установка Azure CLI](/cli/azure/install-azure-cli).

В этой статье предполагается, что у вас есть доступ к существующему кластеру Azure Red Hat OpenShift в качестве пользователя с `admin` привилегиями.

## <a name="check-for-available-aro-cluster-upgrades"></a>Проверка наличия доступных обновлений кластера АТО

В веб-консоли OpenShift выберите **Администрирование**  >  **Параметры кластера** и откройте вкладку **сведения** .

Если **состояние обновления** для кластера отражает **Доступные обновления**, можно обновить кластер.

## <a name="upgrade-your-aro-cluster"></a>Обновление кластера АТО

В веб-консоли на предыдущем шаге задайте **каналу** правильный канал для версии, которую требуется обновить, например `stable-4.5` .

Выберите версию для обновления и щелкните **Обновить**. Вы увидите, что состояние обновления изменится на: `Update to <product-version> in progress` . Ход обновления кластера можно просмотреть, просмотрев индикаторы выполнения для операторов и узлов.

## <a name="next-steps"></a>Дальнейшие действия
- [Сведения об обновлении кластера АТО с помощью интерфейса командной строки OC](https://docs.openshift.com/container-platform/4.6/updating/updating-cluster-between-minor.html)
- Сведения о доступных рекомендациях и обновлениях для платформы контейнеров OpenShift см. в [разделе ошибок](https://access.redhat.com/downloads/content/290/ver=4.6/rhel---8/4.6.0/x86_64/product-errata) на портале клиента.
