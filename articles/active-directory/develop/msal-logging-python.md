---
title: Ведение журнала ошибок и исключений в MSAL для Python
titleSuffix: Microsoft identity platform
description: Сведения о записи ошибок и исключений в MSAL для Python
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 01/25/2021
ms.author: marsma
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ms.openlocfilehash: 1d52b017f94785f5fb25a25f127ae52d96e97d8b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578759"
---
# <a name="logging-in-msal-for-python"></a>Ведение журналов в MSAL для Python

[!INCLUDE [MSAL logging introduction](../../../includes/active-directory-develop-error-logging-introduction.md)]

## <a name="msal-for-python-logging"></a>MSAL для ведения журнала Python

Ведение журнала в MSAL для Python использует [модуль ведения журнала в стандартной библиотеке Python](https://docs.python.org/3/library/logging.html). Вы можете настроить ведение журнала MSAL следующим образом (и увидеть его в действии в [username_password_sample](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.0.0/sample/username_password_sample.py#L31L32)):

### <a name="enable-debug-logging-for-all-modules"></a>Включить ведение журнала отладки для всех модулей

По умолчанию ведение журнала в любом сценарии Python отключено. Если вы хотите включить подробное ведение журнала для **всех** модулей Python в скрипте, используйте `logging.basicConfig` с уровнем `logging.DEBUG` :

```python
import logging

logging.basicConfig(level=logging.DEBUG)
```

При этом все сообщения журнала, предоставленные модулю ведения журнала, будут напечатаны на стандартном выходе.

### <a name="configure-msal-logging-level"></a>Настройка уровня ведения журнала MSAL

Уровень ведения журнала MSAL для регистратора Python можно настроить с помощью `logging.getLogger()` метода с именем средства ведения журнала `"msal"` :

```python
import logging

logging.getLogger("msal").setLevel(logging.WARN)
```

### <a name="configure-msal-logging-with-azure-app-insights"></a>Настройка ведения журнала MSAL с помощью Azure App Insights

Журналы Python предоставляются обработчику журнала, который по умолчанию имеет значение `StreamHandler` . Чтобы отправить журналы MSAL в Application Insights с помощью ключа инструментирования, используйте `AzureLogHandler` предоставленную `opencensus-ext-azure` библиотекой.

Чтобы установить, `opencensus-ext-azure` добавьте `opencensus-ext-azure` пакет из PyPI в зависимости или установку PIP:

```console
pip install opencensus-ext-azure
```

Затем измените обработчик по умолчанию `"msal"` регистратора на экземпляр `AzureLogHandler` с помощью ключа инструментирования, установленного в `APP_INSIGHTS_KEY` переменной среды:

```python
import logging
import os

from opencensus.ext.azure.log_exporter import AzureLogHandler

APP_INSIGHTS_KEY = os.getenv('APP_INSIGHTS_KEY')

logging.getLogger("msal").addHandler(AzureLogHandler(connection_string='InstrumentationKey={0}'.format(APP_INSIGHTS_KEY))
```

### <a name="personal-and-organizational-data-in-python"></a>Личные и организационные данные в Python

MSAL для Python не регистрирует персональные данные или данные Организации. Отсутствует свойство для включения или отключения ведения журнала личных данных или организации.

Вы можете использовать стандартное ведение журнала Python, чтобы заносить в журнал любые нужные данные, но вы несете ответственность за безопасную обработку конфиденциальных данных и соблюдение нормативных требований.

Дополнительные сведения о ведении журнала в Python см. в статье  [ведение журнала Python: инструкции](https://docs.python.org/3/howto/logging.html#logging-basic-tutorial).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода см. в статье [примеры кода платформы идентификации Майкрософт](sample-v2-code.md).
