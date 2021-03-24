---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/17/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 654a7f18d7b06756a6073b4203ea7b6042d1c747
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104604945"
---
|Имя |Описание |Политики |Версия |
|---|---|---|---|
|[Аудит компьютеров с ненадежными параметрами безопасности паролей](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordSettingsAINE.json) |Эта инициатива развертывает требуемые компоненты политики и осуществляет аудит компьютеров с ненадежными параметрами безопасности пароля. Дополнительные сведения о политиках гостевой конфигурации: [https://aka.ms/gcpol](https://aka.ms/gcpol) |9 |1.0.0 |
|[\[Предварительная версия\]. Развернуть необходимые компоненты, чтобы включить политики гостевой конфигурации на виртуальных машинах](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_Prerequisites.json) |Эта инициатива добавляет управляемое удостоверение, назначаемое системой, и развертывает соответствующее платформе расширение гостевой конфигурации на виртуальных машинах, которые могут отслеживаться с помощью политик гостевой конфигурации. Это обязательный компонент для всех политик гостевой конфигурации, который необходимо назначить области назначения политики перед использованием любой политики гостевой конфигурации. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |4 |1.0.0 (предварительная версия) |
|[\[Предварительная версия\]. Компьютеры под управлением Windows должны соответствовать требованиям для базовой конфигурации безопасности Azure](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_AzureBaseline.json) |Эта инициатива осуществляет аудит компьютеров под управлением Windows с параметрами, не соответствующими базовой конфигурации безопасности Azure. Дополнительные сведения см. по адресу [https://aka.ms/gcpol](https://aka.ms/gcpol). |29 |2.0.0-preview |
