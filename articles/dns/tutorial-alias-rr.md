---
title: Руководство по созданию записи псевдонима для ссылки на запись ресурса в зоне
titleSuffix: Azure DNS
description: В этом руководстве описывается процесс создания записи псевдонима Azure DNS для ссылки на запись ресурса в зоне.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 04/19/2021
ms.author: rohink
ms.openlocfilehash: c20f079fdbccdf003d96a1b2683060c2cd6d0e5a
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107737376"
---
# <a name="tutorial-create-an-alias-record-to-refer-to-a-zone-resource-record"></a>Руководство по Создание записи псевдонима для ссылки на запись ресурса в зоне

Записи псевдонимов могут ссылаться на другие наборы записей одного типа. Например, вы можете иметь набор записей DNS CNAME, как псевдоним для другого набора записей CNAME того же типа. Эта возможность полезна, если вы хотите иметь некоторые наборы записей в качестве псевдонимов, а другие не как псевдонимы по поведению.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание записи псевдонима для записи ресурса в зоне.
> * Тестирование записи псевдонима.


Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования
Необходимо иметь доступное имя домена, который можно будет разместить в Azure DNS и использовать для тестирования. У вас должен быть полный контроль над этим доменом, включая возможность определять записи сервера имен (NS) для домена.

Инструкции по размещению домена в Azure DNS см. в [руководстве по размещению домена в Azure DNS](dns-delegate-domain-azure-dns.md).


## <a name="create-an-alias-record"></a>Создание записи псевдонима

Создайте запись псевдонима, которая указывает на запись ресурса в зоне.

### <a name="create-the-target-resource-record"></a>Создайте целевую запись ресурса
1. Выберите свою зону Azure DNS, чтобы открыть ее.
2. Выберите **Набор записей**.
3. В текстовом поле **Имя** введите **server**.
4. Для **типа** выберите **A**.
5. В текстовое поле **IP-адрес** введите **10.10.10.10**.
6. Щелкните **ОК**.

### <a name="create-the-alias-record"></a>Создание записи псевдонима
1. Выберите свою зону Azure DNS, чтобы открыть ее.
2. Выберите **Набор записей**.
3. В текстовом поле **Имя** введите **test**.
4. Для **типа** выберите **A**.
5. Выберите **Да** для параметра **Набор записей псевдонимов**. Затем выберите параметр **Zone record set** (Набор записей зоны).
6. Для **зоны набора записей** выберите запись **сервер**.
7. Щелкните **ОК**.

## <a name="test-the-alias-record"></a>Тестирование записи псевдонима

1. Запустите предпочитаемый инструмент nslookup. Один из вариантов — перейдите к [https://network-tools.com/nslook](https://network-tools.com/nslook).
2. Задайте тип запроса для записей A и найдите **test.\<your domain name\>** . Ответ: **10.10.10.10**.
3. На портале Azure измените запись для **server** A на **10.11.11.11**.
4. Подождите несколько минут, а затем снова используйте nslookup для записи **test**. Ответ: **10.11.11.11**.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам больше не нужны ресурсы, созданные в этом руководстве, удалите записи ресурсов **server** и **test** в своей зоне.


## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы создали запись псевдонима, чтобы ссылаться на запись ресурса в пределах зоны. Чтобы узнать об Azure DNS и веб-приложениях, перейдите к руководству для веб-приложений.

> [!div class="nextstepaction"]
> [Создание записей DNS для веб-приложения в пользовательском домене](./dns-web-sites-custom-domain.md)
