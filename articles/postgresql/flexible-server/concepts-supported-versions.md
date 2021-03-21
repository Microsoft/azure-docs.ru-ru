---
title: Поддерживаемые версии — база данных Azure для PostgreSQL — гибкий сервер
description: Описание поддерживаемых основных и вспомогательных версий PostgreSQL в базе данных Azure для PostgreSQL-гибкого сервера.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 03/03/2021
ms.openlocfilehash: 474545b7c5eca64b558e5fe26e23e6800b0cab96
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102037172"
---
# <a name="supported-postgresql-major-versions-in-azure-database-for-postgresql---flexible-server"></a>Поддерживаемые основные версии PostgreSQL в базе данных Azure для PostgreSQL-гибкого сервера

> [!IMPORTANT]
> Гибкий сервер Базы данных Azure для PostgreSQL предоставляется в режиме предварительной версии

База данных Azure для PostgreSQL — гибкий сервер в настоящее время поддерживает следующие основные версии:

## <a name="postgresql-version-12"></a>PostgreSQL версии 12

Текущий дополнительный выпуск — 12,5. Дополнительные сведения об улучшениях и исправлениях в этом дополнительном выпуске см. в [документации по PostgreSQL](https://www.postgresql.org/docs/12/static/release-12-4.html) .

## <a name="postgresql-version-11"></a>PostgreSQL версии 11

Текущий дополнительный выпуск — 11,10. Дополнительные сведения об улучшениях и исправлениях в этом дополнительном выпуске см. в [документации по PostgreSQL](https://www.postgresql.org/docs/11/static/release-11-9.html) .

## <a name="postgresql-version-10-and-older"></a>PostgreSQL версии 10 и более ранних версий

Мы не поддерживаем PostgreSQL версии 10 и более ранних версий для базы данных Azure для PostgreSQL-гибкого сервера. Если требуются более старые версии, используйте вариант развертывания на [одном сервере](../concepts-supported-versions.md) .

## <a name="managing-upgrades"></a>Управление обновлениями

Проект PostgreSQL регулярно выдает дополнительные выпуски для исправления ошибок, о которых сообщили. База данных Azure для PostgreSQL автоматически закрывает серверы с дополнительными выпусками во время ежемесячных развертываний службы.

Автоматизация обновления основной версии пока не поддерживается. Например, в настоящее время автоматическое обновление с PostgreSQL 11 до PostgreSQL 12 отсутствует.<!-- To upgrade to the next major version, create a [database dump and restore](howto-migrate-using-dump-and-restore.md) to a server that was created with the new engine version.-->

<!--
## Next steps

For information on supported PostgreSQL extensions, see [the extensions document](concepts-extensions.md).
-->