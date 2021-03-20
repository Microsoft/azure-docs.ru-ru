---
title: Настройка параметров сервера — портал Azure — гибкий сервер в базе данных Azure для PostgreSQL
description: В этой статье описывается, как настроить параметры postgres в базе данных Azure для гибкого сервера PostgreSQL с помощью портал Azure.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 4f945c2c7fffb143bdb8324a330775fb072b25c1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90938828"
---
# <a name="configure-server-parameters-in-azure-database-for-postgresql---flexible-server-via-the-azure-portal"></a>Настройка параметров сервера в базе данных Azure для PostgreSQL-гибкого сервера с помощью портал Azure 

Вы можете вывести список, отобразить и обновить параметры конфигурации для гибкого сервера базы данных Azure для PostgreSQL с помощью портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к выполнению этого руководства, вам потребуются следующие компоненты:
- [Гибкий сервер базы данных Azure для PostgreSQL](quickstart-create-server-portal.md)

## <a name="viewing-and-editing-parameters"></a>Просмотр и изменение параметров

1. Откройте [портал Azure](https://portal.azure.com).

2. Выберите гибкий сервер.

3. В разделе **Параметры** выберите **Параметры сервера**. На странице отобразится список параметров, их значения и описания.
![Страница общих сведений для параметров](./media/howto-configure-server-parameters-in-portal/3-overview-of-parameters.png)

4. Нажмите кнопку **раскрывающегося списка**, чтобы просмотреть возможные значения параметров перечисляемого типа, например client_min_messages.
![Раскрывающийся список для перечисляемого типа](./media/howto-configure-server-parameters-in-portal/4-enum-drop-down.png)

5. Выберите кнопку с буквой **i** (информация) или наведите на нее указатель мыши, чтобы просмотреть диапазон возможных значений числовых параметров, например cpu_index_tuple_cost.
![Кнопка информации](./media/howto-configure-server-parameters-in-portal/4-information-button.png)

6. При необходимости воспользуйтесь **полем поиска**, чтобы сузить область поиска до определенного параметра. Поиск выполняется по имени и описанию параметров.
![Результаты поиска](./media/howto-configure-server-parameters-in-portal/5-search.png)

7. Измените значения требуемых параметров. Все изменения, вносимые в этом сеансе, выделены фиолетовым цветом. Изменив значения, можно нажать кнопку **Сохранить**. Также вы можете нажать кнопку **Отменить**, чтобы отменить изменения.
![Сохранение или отмена изменений](./media/howto-configure-server-parameters-in-portal/6-save-and-discard-buttons.png)

8. Если вы сохранили новые значения параметров, всегда можно восстановить значения по умолчанию, выбрав **Сбросить все к значениям по умолчанию**.
![Сбросить все к значениям по умолчанию](./media/howto-configure-server-parameters-in-portal/7-reset-to-default-button.png)

## <a name="next-steps"></a>Дальнейшие действия

Вам необходимы дополнительные сведения о следующих аспектах:

- [Серверы базы данных Azure для PostgreSQL](concepts-servers.md)
- [Настройка параметров конфигурации сервера с помощью Azure CLI](howto-configure-server-parameters-using-cli.md)
