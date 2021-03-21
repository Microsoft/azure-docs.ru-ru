---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/13/2020
ms.author: alkohli
ms.openlocfilehash: d12a5042d0c2d79989d82e86e7265d030912f5ee
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98901161"
---
В этом разделе описываются ограничения для службы хранилища Azure и необходимые соглашения об именовании для файлов Azure, блочных BLOB-объектов Azure и страничных больших двоичных объектов Azure, которые применимы к службе Azure Stack ребра. Тщательно изучите ограничения хранилища и следуйте всем рекомендациям.

Актуальные сведения об ограничениях службы хранилища Azure и рекомендациях по именованию общих папок, контейнеров и файлов см. в следующих статьях:

- [Именование контейнеров и ссылка на них](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)
- [Именование общих папок и ссылка на них](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata)
- [Блочные BLOB-объекты и соглашения о страничных объектах](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)

> [!IMPORTANT]
> Если имеются файлы или каталоги, превышающие ограничения службы хранилища Azure, или не соответствуют соглашениям об именовании файлов и BLOB-объектов Azure, эти файлы или каталоги не будут приняты в службу хранилища Azure через службы Azure Stack ребра.