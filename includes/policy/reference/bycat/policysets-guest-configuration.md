---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/24/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 0cb4e9bb5c65dd32b20c8fc0fc8a45db8856371a
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105032717"
---
|Имя |Описание |Политики |Версия |
|---|---|---|---|
|[Аудит компьютеров с ненадежными параметрами безопасности паролей](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordSettingsAINE.json) |Эта инициатива развертывает требуемые компоненты политики и осуществляет аудит компьютеров с ненадежными параметрами безопасности пароля. Дополнительные сведения о политиках гостевой конфигурации: [https://aka.ms/gcpol](https://aka.ms/gcpol) |9 |1.0.0 |
|[\[Предварительная версия\]. Развернуть необходимые компоненты, чтобы включить политики гостевой конфигурации на виртуальных машинах](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_Prerequisites.json) |Эта инициатива добавляет управляемое удостоверение, назначаемое системой, и развертывает соответствующее платформе расширение гостевой конфигурации на виртуальных машинах, которые могут отслеживаться с помощью политик гостевой конфигурации. Это обязательный компонент для всех политик гостевой конфигурации, который необходимо назначить области назначения политики перед использованием любой политики гостевой конфигурации. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |4 |1.0.0 (предварительная версия) |
|[\[Предварительная версия\]. Компьютеры под управлением Windows должны соответствовать требованиям для базовой конфигурации безопасности Azure](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_AzureBaseline.json) |Эта инициатива осуществляет аудит компьютеров под управлением Windows с параметрами, не соответствующими базовой конфигурации безопасности Azure. Дополнительные сведения см. по адресу [https://aka.ms/gcpol](https://aka.ms/gcpol). |29 |2.0.0-preview |
