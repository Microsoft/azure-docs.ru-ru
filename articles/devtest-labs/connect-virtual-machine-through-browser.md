---
title: Подключение к виртуальным машинам через браузер — Azure | Документация Майкрософт
description: Узнайте, как подключиться к виртуальным машинам через браузер.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 352fc5fd3ff53a00d9f62966ecf21417ad898706
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87288062"
---
# <a name="connect-to-your-virtual-machines-through-a-browser"></a>Подключение к виртуальным машинам через браузер 

DevTest Labs интегрируется с [Azure бастиона](../bastion/index.yml), что позволяет подключаться к виртуальным машинам через браузер. Сведения о том, как включить эту функцию в DevTest Labs, см. в статье [Включение подключения браузера на виртуальных машинах лаборатории](enable-browser-connection-lab-virtual-machines.md).

После включения *подключения к браузеру* пользователи лаборатории смогут получать доступ к виртуальным машинам через браузер.  

## <a name="create-a-lab-virtual-machine"></a>Создание виртуальной машины лаборатории

Сначала необходимо создать виртуальную машину лаборатории в виртуальной сети, на которой настроен бастиона. Выберите вторую созданную **подсеть** , а не азуребастионсубнет. Вы можете выбрать виртуальную сеть во время создания виртуальной машины, перейдя на вкладку **Дополнительные параметры** .

![Создание виртуальной машины](./media/connect-virtual-machine-through-browser/create-virtual-machine.png)

## <a name="launch-virtual-machine-in-a-browser"></a>Запуск виртуальной машины в браузере

После создания виртуальной машины ее можно запустить в браузере, нажав кнопку *подключить к браузеру* и введя имя пользователя и пароль для компьютера.  

![Запуск в браузере](./media/connect-virtual-machine-through-browser/browser-connect.png)

## <a name="next-steps"></a>Дальнейшие действия

[Добавление виртуальной машины в лабораторию в Azure DevTest Labs](devtest-lab-add-vm.md)
