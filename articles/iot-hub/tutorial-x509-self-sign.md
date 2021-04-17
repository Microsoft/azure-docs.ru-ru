---
title: Руководство. Использование OpenSSL для создания самозаверяющих сертификатов для Центра Интернета вещей Azure | Документация Майкрософт
description: Руководство. Использование OpenSSL для создания самозаверяющих сертификатов X.509 для Центра Интернета вещей Azure
author: v-gpettibone
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 02/26/2021
ms.author: robinsh
ms.custom:
- mvc
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: 982e402946cbd71d946bc1e316cef99621c536a3
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378199"
---
# <a name="tutorial-using-openssl-to-create-self-signed-certificates"></a>Руководство. Использование OpenSSL для создания самозаверяющих тестовых сертификатов

Вы можете выполнить аутентификацию устройства в Центре Интернета вещей с помощью двух самозаверяющих сертификатов устройств. Иногда это называется аутентификацией с использованием отпечатков, так как сертификаты содержат отпечатки (хэш-значения), которые вы отправляете в Центр Интернета вещей. Ниже показано, как создать два самозаверяющих сертификата.

## <a name="step-1---create-a-key-for-the-first-certificate"></a>Шаг 1. Создание ключа для первого сертификата

```bash
openssl genpkey -out device1.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048
```

## <a name="step-2---create-a-csr-for-the-first-certificate"></a>Шаг 2. Создание CSR для первого сертификата

При появлении запроса обязательно укажите код устройства.

```bash
openssl req -new -key device1.key -out device1.csr

Country Name (2 letter code) [XX]:.
State or Province Name (full name) []:.
Locality Name (eg, city) [Default City]:.
Organization Name (eg, company) [Default Company Ltd]:.
Organizational Unit Name (eg, section) []:.
Common Name (eg, your name or your server hostname) []:{your-device-id}
Email Address []:

```

## <a name="step-3---check-the-csr"></a>Шаг 3. Проверка CSR

```bash
openssl req -text -in device1.csr -noout
```

## <a name="step-4---self-sign-certificate-1"></a>Шаг 4. Самостоятельное заверение сертификата 1

```bash
openssl x509 -req -days 365 -in device1.csr -signkey device1.key -out device.crt
```

## <a name="step-5---create-a-key-for-certificate-2"></a>Шаг 5. Создание ключа для сертификата 2

При появлении запроса укажите тот же код устройства, который вы использовали для сертификата 1.

```bash
openssl req -new -key device2.key -out device2.csr

Country Name (2 letter code) [XX]:.
State or Province Name (full name) []:.
Locality Name (eg, city) [Default City]:.
Organization Name (eg, company) [Default Company Ltd]:.
Organizational Unit Name (eg, section) []:.
Common Name (eg, your name or your server hostname) []:{your-device-id}
Email Address []:

```

## <a name="step-6---self-sign-certificate-2"></a>Шаг 6. Самостоятельное заверение сертификата 2

```bash
openssl x509 -req -days 365 -in device2.csr -signkey device2.key -out device2.crt
```

## <a name="step-7---retrieve-the-thumbprint-for-certificate-1"></a>Шаг 7. Получение отпечатка для сертификата 1

```bash
openssl x509 -in device.crt -text -fingerprint
```

## <a name="step-8---retrieve-the-thumbprint-for-certificate-2"></a>Шаг 8. Получение отпечатка для сертификата 2

```bash
openssl x509 -in device2.crt -text -fingerprint
```

## <a name="step-9---create-a-new-iot-device"></a>Шаг 9. Создание устройства Интернета вещей

Перейдите в Центр Интернета вещей на портале Azure и создайте новое удостоверение устройства Интернета вещей со следующими характеристиками:

* Укажите **идентификатор устройства**, соответствующий имени субъекта двух ваших сертификатов.
* Выберите тип проверки подлинности **Самозаверяющий сертификат X.509**.
* Вставьте отпечатки шестнадцатеричных строк, скопированные в основном и дополнительном сертификатах устройства. Убедитесь, что в шестнадцатеричных строках нет разделителей в виде двоеточия.

## <a name="next-steps"></a>Next Steps

Руководство [по тестированию проверки подлинности на основе сертификатов](tutorial-x509-test-certificate.md) поможет вам убедиться, что ваш сертификат позволяет устройству выполнять вход в Центр Интернета вещей.
