---
title: включить файл
description: включить файл
services: virtual-machines-windows, virtual-machines-linux
author: cynthn
ms.service: multiple
ms.topic: include
ms.date: 10/09/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 98d765e2f6909f00f8dfe76d06aef017aad67adf
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "71174987"
---
## <a name="terminology"></a>Терминология

Атрибуты образа Marketplace в Azure:

* **Издатель.** Организация, создавшая образ. Примеры: Canonical, MicrosoftWindowsServer.
* **Предложение.** Имя группы связанных образов, созданных издателем. Примеры: UbuntuServer, WindowsServer
* **Номер SKU.** Экземпляр предложения, например основной выпуск дистрибутива. Примеры: 18,04-LTS, 2019-Datacenter
* **Версия.** Номер версии SKU образа. 

Чтобы определить образ Marketplace при программном развертывании виртуальной машины, укажите эти значения как отдельные параметры. Некоторые средства принимают *URN* образа, который объединяет эти значения, разделенные двоеточием (:): *Издатель*:*Предложение*:*Номер SKU*:*Версия*. В URN номер версии можно заменить ключевым словом "latest". В этом случае будет выбрана последняя версия образа. 

Если у издателя образа есть дополнительные условия лицензии и приобретения, необходимо принять их, чтобы включить программное развертывание. Также при программном развертывании виртуальной машины нужно ввести параметры *плана приобретения*. См. дополнительные сведения о [развертывании образа на условиях Marketplace](#deploy-an-image-with-marketplace-terms).
