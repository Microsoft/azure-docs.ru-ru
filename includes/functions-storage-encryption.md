---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/06/2020
ms.author: glenga
ms.openlocfilehash: 0bc5977a581006dc760c0435f9d68371ced7e4cd
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83648772"
---
Служба хранилища Azure шифрует все данные в неактивных учетных записях хранения. Дополнительные сведения см. в статье [Шифрование службы хранилища Azure для неактивных данных](../articles/storage/common/storage-service-encryption.md).

По умолчанию данные шифруются с помощью ключей, управляемых корпорацией Майкрософт. Для дополнительного управления ключами шифрования можно предоставить ключи, управляемые клиентом, чтобы зашифровать большие двоичные объекты и данные файлов. Эти ключи должны присутствовать в Azure Key Vault, чтобы Функции Azure могли получить доступ к учетной записи хранения. Дополнительные сведения см. в разделе [Шифрование неактивных данных с помощью управляемых клиентом ключей](../articles/azure-functions/configure-encrypt-at-rest-using-cmk.md).  