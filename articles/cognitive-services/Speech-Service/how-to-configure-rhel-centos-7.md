---
title: Как настроить RHEL/CentOS 7-Speech Service
titleSuffix: Azure Cognitive Services
description: Узнайте, как настроить RHEL/CentOS 7, чтобы можно было использовать речевой пакет SDK.
services: cognitive-services
author: pankopon
manager: jhakulin
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/02/2020
ms.author: pankopon
ms.openlocfilehash: ba531164e024f96d3bdd23912f3f6e90275edda4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83589743"
---
# <a name="configure-rhelcentos-7-for-speech-sdk"></a>Настройка RHEL/CentOS 7 для Speech SDK

Red Hat Enterprise Linux (RHEL) 8 x64 и CentOS 8 x64 официально поддерживаются пакетом SDK для распознавания речи версии 1.10.0 и более поздних версий. Также можно использовать речевой пакет SDK на RHEL/CentOS 7 x64, но для этого требуется обновление компилятора C++ (для разработки C++) и общей библиотеки времени выполнения C++ в системе.

Чтобы проверить версию компилятора C++, выполните команду:

```bash
g++ --version
```

Если компилятор установлен, выходные данные должны выглядеть следующим образом:

```bash
g++ (GCC) 4.8.5 20150623 (Red Hat 4.8.5-39)
```

Это сообщение позволяет убедиться, что установлена версия GCC Major 4. Эта версия не поддерживает полную поддержку стандарта C++ 11, который использует пакет SDK для распознавания речи. Попытка компиляции программы C++ с этой версией GCC и заголовками речевого пакета SDK приведет к ошибкам компиляции.

Также важно проверить версию общей библиотеки времени выполнения C++ (libstdc + +). Большая часть речевого пакета SDK реализована в виде собственных библиотек C++, что означает, что она зависит от libstdc + + независимо от языка, используемого для разработки приложений.

Чтобы найти расположение libstdc + + в вашей системе, выполните:

```bash
ldconfig -p | grep libstdc++
```

Выходные данные в обычный RHEL/CentOS 7 (x64):

```bash
libstdc++.so.6 (libc6,x86-64) => /lib64/libstdc++.so.6
```

На основе этого сообщения необходимо проверить определения версий с помощью следующей команды:

```bash
strings /lib64/libstdc++.so.6 | egrep "GLIBCXX_|CXXABI_"
```

Результат должен выглядеть примерно так:

```bash
...
GLIBCXX_3.4.19
...
CXXABI_1.3.7
...
```

Для работы с речевыми ПАКЕТами требуется **CXXABI_1.3,9** и **GLIBCXX_3.4.21**. Эти сведения можно найти, запустив `ldd libMicrosoft.CognitiveServices.Speech.core.so` в библиотеках SDK для распознавания речи из пакета Linux.

> [!NOTE]
> Рекомендуется, чтобы версия GCC, установленная в системе, была по крайней мере **5.4.0**, с соответствующими библиотеками среды выполнения.

## <a name="example"></a>Пример

Это пример набора команд, демонстрирующий настройку RHEL/CentOS 7 x64 для разработки (C++, C#, Java, Python) с помощью пакета SDK для Speech 1.10.0 или более поздней версии:

### <a name="1-general-setup"></a>1. Общая Настройка

Сначала установите все общие зависимости:

```bash
# Only run ONE of the following two commands
# - for CentOS 7:
sudo rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm
# - for RHEL 7:
sudo rpm -Uvh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm

# Install development tools and libraries
sudo yum update -y
sudo yum groupinstall -y "Development tools"
sudo yum install -y alsa-lib dotnet-sdk-2.1 java-1.8.0-openjdk-devel openssl python3
sudo yum install -y gstreamer1 gstreamer1-plugins-base gstreamer1-plugins-good gstreamer1-plugins-bad-free gstreamer1-plugins-ugly-free
```

### <a name="2-cc-compiler-and-runtime-libraries"></a>2. компилятор C/C++ и библиотеки времени выполнения

Установите необходимые пакеты с помощью следующей команды:

```bash
sudo yum install -y gmp-devel mpfr-devel libmpc-devel
```

> [!NOTE]
> Пакет либмпк-которые устарел в обновлении RHEL 7,8. Если выходные данные предыдущей команды содержат сообщение
>
> ```bash
> No package libmpc-devel available.
> ```
>
> Затем необходимо установить необходимые файлы из исходных источников. Выполните следующие команды:
>
> ```bash
> curl https://ftp.gnu.org/gnu/mpc/mpc-1.1.0.tar.gz -O
> tar zxf mpc-1.1.0.tar.gz
> mkdir mpc-1.1.0-build && cd mpc-1.1.0-build
> ../mpc-1.1.0/configure --prefix=/usr/local --libdir=/usr/local/lib64
> make -j$(nproc)
> sudo make install-strip
> ```

Затем обновите библиотеки компилятора и среды выполнения:

```bash
# Build GCC 5.4.0 and runtimes and install them under /usr/local
curl https://ftp.gnu.org/gnu/gcc/gcc-5.4.0/gcc-5.4.0.tar.bz2 -O
tar jxf gcc-5.4.0.tar.bz2
mkdir gcc-5.4.0-build && cd gcc-5.4.0-build
../gcc-5.4.0/configure --enable-languages=c,c++ --disable-bootstrap --disable-multilib --prefix=/usr/local
make -j$(nproc)
sudo make install-strip
```

Если обновленный компилятор и библиотеки необходимо развернуть на нескольких компьютерах, можно просто скопировать их из раздела в `/usr/local` другие компьютеры. Если требуются только библиотеки времени выполнения, файлы в `/usr/local/lib64` будут достаточно.

### <a name="3-environment-settings"></a>3. параметры среды

Выполните следующие команды, чтобы завершить настройку:

```bash
# Set SSL cert file location
# (this is required for any development/testing with Speech SDK)
export SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt

# Add updated C/C++ runtimes to the library path
# (this is required for any development/testing with Speech SDK)
export LD_LIBRARY_PATH=/usr/local/lib64:$LD_LIBRARY_PATH

# For C++ development only:
# - add the updated compiler to PATH
#   (note, /usr/local/bin should be already first in PATH on vanilla systems)
# - add Speech SDK libraries from the Linux tar package to LD_LIBRARY_PATH
#   (note, use the actual path to extracted files!)
export PATH=/usr/local/bin:$PATH
hash -r # reset cached paths in the current shell session just in case
export LD_LIBRARY_PATH=/path/to/extracted/SpeechSDK-Linux-1.10.0/lib/x64:$LD_LIBRARY_PATH

# For Python: install the Speech SDK module
python3 -m pip install azure-cognitiveservices-speech --user
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о пакете SDK службы "Речь"](speech-sdk.md)
