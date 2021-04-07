---
title: Общие сведения о Хранилище таблиц. Хранилище объектов в Azure | Документация Майкрософт
description: Хранение структурированных данных в облаке в хранилище таблиц Azure (хранилище данных NoSQL).
services: storage
ms.service: storage
author: tamram
ms.author: tamram
ms.devlang: dotnet
ms.topic: overview
ms.date: 01/07/2021
ms.subservice: tables
ms.openlocfilehash: 292104651c6bb9b63e9d8cbe26fea2bf8c6acbf1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98027746"
---
# <a name="what-is-azure-table-storage-"></a>Что собой представляет табличное хранилище Azure? 

[!INCLUDE [storage-table-cosmos-db-tip-include](../../../includes/storage-table-cosmos-db-tip-include.md)]

Хранилище таблиц Azure — это служба, которая хранит нереляционные структурированные данные (также называются структурированными данными NoSQL) в облаке, предоставляя хранилище ключей и атрибутов с бессхемной структурой. Такая конструкция хранилища таблиц позволяет легко адаптировать данные по мере расширения приложения. Разным типам приложений может быть предоставлен быстрый и экономичный доступ к хранилищу таблиц. Такое хранилище обычно дешевле, чем традиционные хранилища SQL для похожих объемов данных.

Хранилище таблиц можно использовать для хранения гибких наборов данных, например пользовательских данных для веб-приложений, адресных книг, сведений об устройстве или метаданных любого другого типа, которые требуются вашей службе. В таблице можно хранить любое количество сущностей, а учетная запись хранения может содержать любое количество таблиц в пределах емкости учетной записи.

[!INCLUDE [storage-table-concepts-include](../../../includes/storage-table-concepts-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

* [Обозреватель хранилищ Microsoft Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md) — это бесплатное автономное приложение от корпорации Майкрософт, позволяющее визуализировать данные из службы хранилища Azure на платформе Windows, macOS и Linux.

* [Getting Started with Azure Table Storage in .NET](../../cosmos-db/tutorial-develop-table-dotnet.md) (Приступая к работе с хранилищем таблиц Azure в .NET)

* Дополнительные сведения о доступных API-интерфейсах см. в справочной документации по службе таблиц:

    * [Справочник по клиентской библиотеке хранилища для .NET](/dotnet/api/overview/azure/storage)

    * [Справочник по REST API](/rest/api/storageservices/)