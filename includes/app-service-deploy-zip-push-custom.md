---
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 11/03/2016
ms.author: cephalin
ms.openlocfilehash: 0019e50615f3e66778709ad8cb28f92967c66e2e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96018456"
---
## <a name="deployment-customization"></a>Настройка развертывания

При развертывании предполагается, что отправляемый ZIP-файл содержит готовое к запуску приложение. По умолчанию настройки не задаются. Чтобы включить те же процессы компиляции, что и при непрерывной интеграции, добавьте в параметры приложения следующее:

`SCM_DO_BUILD_DURING_DEPLOYMENT=true`

При развертывании из отправленного ZIP-файла для этого параметра по умолчанию установлено значение **false**. Для развертываний с непрерывной интеграцией значение по умолчанию — **true**. Если установлено значение **true**, при развертывании используются связанные с ним параметры, которые вы задали. Эти параметры можно настроить как параметры приложения, или указать в файле конфигурации .deployment, который находится в корневом каталоге ZIP-файла. Дополнительные сведения см. в разделе о [параметрах, связанных с репозиторием и развертыванием,](https://github.com/projectkudu/kudu/wiki/Configurable-settings#repository-and-deployment-related-settings) справочника по развертыванию.