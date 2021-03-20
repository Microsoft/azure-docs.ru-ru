---
title: Удаление узлов для решения VMware с помощью Клаудсимпле — Azure
description: Узнайте, как удалить узлы из VMWare с помощью развертывания Клаудсимпле. Для узлов Клаудсимпле выполняется измерение. Удалите узлы, которые не используются из портал Azure.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/05/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 569bc6350b1bfa01228d49d28a1d12e2ab62f6f0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88142270"
---
# <a name="delete-nodes-from-azure-vmware-solution-by-cloudsimple"></a>Удаление узлов из решения VMware Azure с помощью Клаудсимпле

Узлы Клаудсимпле измеряются после их создания.  Чтобы отключить отслеживание узлов, необходимо удалить узлы.  Вы удаляете узлы, которые не используются из портал Azure.

## <a name="before-you-begin"></a>Перед началом

Узел можно удалить только при следующих условиях:

* Частное облако, созданное с помощью узлов, удаляется.  Сведения об удалении частного облака см. в статье [Удаление решения VMware для Azure с помощью клаудсимпле](delete-private-cloud.md).
* Узел был удален из частного облака путем сжатия частного облака.  Сведения о сжатии частного облака см. в статье [Сжатие решения VMware для Azure с помощью Клаудсимпле Private Cloud](shrink-private-cloud.md).

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу [https://portal.azure.com](https://portal.azure.com).

## <a name="delete-cloudsimple-node"></a>Удалить узел Клаудсимпле

1. Выбор пункта **Все службы**.

2. Выполните поиск **узлов клаудсимпле**.

   ![Поиск узлов Клаудсимпле](media/create-cloudsimple-node-search.png)

3. Выберите **узлы клаудсимпле**.

4. Выберите узлы, которые не принадлежат частному облаку для удаления.  В столбце **имя частного облака** отображается имя частного облака, к которому принадлежит узел.  Если узел не используется частным облаком, значение будет пустым. 

    ![Выбор узлов Клаудсимпле](media/select-delete-cloudsimple-node.png)

> [!NOTE]
> Удалить можно только узлы, которые не являются частью частного облака.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о [частном облаке](cloudsimple-private-cloud.md)
