---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 06/08/2020
ms.author: alkohli
ms.openlocfilehash: 0e1b70ba3f1359c7cf9c6633ab5e0e05c7b6e946
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96027974"
---
Этот раздел описывает ограничения для службы хранилища Azure и требуемые соглашения об именовании для файлов Azure, блочных BLOB-объектов Azure и страничных BLOB-объектов Azure применительно к службе Data Box. Тщательно изучите ограничения хранилища и следуйте всем рекомендациям.

Актуальные сведения об ограничениях службы хранилища Azure и рекомендациях по именованию общих папок, контейнеров и файлов см. в следующих статьях:

- [Именование контейнеров и ссылка на них](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)
- [Именование общих папок и ссылка на них](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata)
- [Блочные BLOB-объекты и соглашения о страничных объектах](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)

> [!IMPORTANT]
> Если имеются файлы или каталоги, превышающие ограничения службы хранилища Azure, или не соответствуют соглашениям об именовании файлов и BLOB-объектов Azure, эти файлы или каталоги не будут поступать в службу хранилища Azure с помощью службы Data Box.