---
title: Настройка параметров экспорта в API Azure для FHIR
description: В этой статье описывается, как настроить параметры экспорта в API Azure для FHIR.
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 3/5/2020
ms.author: matjazl
ms.openlocfilehash: bb728b81d4dad5d880c27d1ebe8f85f1508f8231
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103019755"
---
# <a name="configure-export-setting-and-set-up-the-storage-account"></a>Настройка параметров экспорта и настройка учетной записи хранения

API Azure для FHIR поддерживает $exportную команду, которая позволяет экспортировать данные из Azure API для учетной записи FHIR в учетную запись хранения.

Настройка экспорта в API Azure для FHIR состоит из трех этапов.

1. Включение управляемого удостоверения в Azure API для службы FHIR
2. Создание учетной записи хранения Azure (если это еще не сделано) и назначение разрешения API Azure для FHIR учетной записи хранения
3. Выбор учетной записи хранения в API Azure для FHIR в качестве учетной записи экспорта хранения

## <a name="enabling-managed-identity-on-azure-api-for-fhir"></a>Включение управляемого удостоверения в API Azure для FHIR

Первым шагом в настройке API Azure для FHIR для экспорта является включение управляемого на уровне системы удостоверения в службе. Сведения об управляемых удостоверениях в Azure можно прочитать [здесь](../../active-directory/managed-identities-azure-resources/overview.md).

Для этого перейдите в Azure API for FHIR Service и выберите колонку удостоверений. Изменение состояния на вкл. включит управляемое удостоверение в Azure API для службы FHIR.

![Включение управляемого удостоверения](media/export-data/fhir-mi-enabled.png)

Теперь можно перейти к следующему шагу и создать учетную запись хранения и назначить ей разрешение.

## <a name="adding-permission-to-storage-account"></a>Добавление разрешения в учетную запись хранения

Следующим шагом экспорта является назначение разрешения на запись в учетную запись хранения в службе API Azure для FHIR.

После создания учетной записи хранения перейдите в колонку управление доступом (IAM) в учетной записи хранения и выберите добавить назначения ролей.

![Экспортировать назначение ролей](media/export-data/fhir-export-role-assignment.png)

Затем мы добавим участника данных роли большого двоичного объекта хранилища ролей в имя службы.

![Добавление роли](media/export-data/fhir-export-role-add.png)

Теперь мы готовы к следующему шагу, где мы можем выбрать учетную запись хранения в Azure API для FHIR в качестве учетной записи хранения по умолчанию для $export.

## <a name="selecting-the-storage-account-for-export"></a>Выбор учетной записи хранения для $export

Последний шаг — Назначение учетной записи хранения Azure, которую Azure API для FHIR будет использовать для экспорта данных в. Для этого перейдите в колонку интеграция в API Azure для службы FHIR в портал Azure и выберите учетную запись хранения.

![FHIR экспорт хранилища](media/export-data/fhir-export-storage.png)

После этого все готово для экспорта данных с помощью команды $export.

>[!div class="nextstepaction"]
>[Дополнительные параметры](azure-api-for-fhir-additional-settings.md)
