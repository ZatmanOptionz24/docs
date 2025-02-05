---
title: Безопасность в GitHub Codespaces
intro: 'Обзор архитектуры безопасности {% data variables.product.prodname_github_codespaces %} с рекомендациями по обеспечению безопасности и минимизации рисков атак.'
miniTocMaxHeadingLevel: 3
versions:
  fpt: '*'
  ghec: '*'
topics:
  - Codespaces
  - Security
type: reference
shortTitle: Security in Codespaces
redirect_from:
  - /codespaces/codespaces-reference/security-in-codespaces
ms.openlocfilehash: 0e7fe9a7644f78fc0dfa6d5bb624c5d74f3d8713
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/05/2022
ms.locfileid: '147111691'
---
## Общие сведения о безопасности пространств кода

В {% data variables.product.prodname_github_codespaces %} по умолчанию используется улучшенная защита. Следовательно, необходимо убедиться в том, что ваши методы разработки программного обеспечения не приводят к риску снижения уровня безопасности вашего пространства кода. 

В этом руководстве описывается, как Codespaces обеспечивает безопасность среды разработки и предоставляет ряд рекомендаций, которые помогут обеспечить безопасность по мере работы. Как и для любых других средств разработки, помните, что вы должны работать только с теми репозиториями, которые вы знаете и которым доверяете.

### Изоляция среды

{% data variables.product.prodname_github_codespaces %} отделяет пространства кода друг от друга, и каждое пространство кода использует собственную виртуальную машину и сеть.

#### Изолированные виртуальные машины

Каждое пространство кода размещается на собственной специально созданной виртуальной машине. Два пространства кода никогда не размещаются на одной виртуальной машине. 

Каждый раз при перезапуске пространства кода оно развертывается на новой виртуальной машине с последними доступными обновлениями для системы безопасности.

#### Изолированная сеть

Каждое пространство кода имеет собственную изолированную виртуальную сеть. Используются брандмауэры для блокировки входящих подключений из Интернета и предотвращения обмена данными между пространствами кода во внутренних сетях. По умолчанию пространства кода могут устанавливать исходящие подключения к Интернету.

### Аутентификация

Можно подключиться к codespace с помощью веб-браузера или из {% data variables.product.prodname_vscode %}. При подключении из {% data variables.product.prodname_vscode_shortname %} вам будет предложено пройти проверку подлинности с помощью {% data variables.product.product_name %}. 

Каждый раз при создании или перезапуске пространства кода ему назначается новый маркер {% data variables.product.company_short %} с автоматическим сроком действия. Срок действия позволяет работать в пространстве кода без повторной проверки подлинности в течение обычного рабочего дня, а также снижает вероятность того, что подключение останется открытым после того, как вы перестанете использовать пространство кода.

Область действия маркера зависит от имеющегося уровня доступа к репозиторию, в котором было создано пространство кода:

- **Если у вас есть доступ к репозиторию на запись**: область действия маркера будет ограничена доступом к репозиторию на чтение и запись.
- **Если у вас есть только доступ на чтение к репозиторию**: маркер будет разрешать только клонирование кода из исходного репозитория. Если вы попытаетесь отправить код в частный репозиторий, для которого у вас есть только доступ на чтение, {% data variables.product.prodname_codespaces %} предложит создать личную вилку репозитория. Затем маркер будет обновлен с предоставлением доступа на чтение и запись к новой личной вилке. 
- **Если вы разрешили доступ к другим репозиториям для пространства кода**: если пространству кода предоставлен [доступ к другим репозиториям](/codespaces/managing-codespaces-for-your-organization/managing-access-and-security-for-your-organizations-codespaces), все пространства кода, созданные из этого репозитория, будут иметь маркеры чтения и записи в исходном репозитории. Кроме того, маркеры также получат доступ на чтение к другим репозиториям, указанным пользователем или организацией.

Администраторы организации указывают, какие репозитории следует считать доверенными. Администратор может [настроить доверие](/codespaces/managing-codespaces-for-your-organization/managing-access-and-security-for-your-organizations-codespaces) для всех или некоторых репозиториев организации или отключить доверие для всех репозиториев. Пространство кода не может иметь разрешения на доступ к ресурсам, превышающие разрешения пользователя, создавшего пространство кода, даже если администратор организации предоставил доступ всем пользователям и всем репозиториям.

### Подключения к пространству кода

Вы можете подключиться к пространству кода с помощью зашифрованного туннеля TLS, предоставляемого службой {% data variables.product.prodname_github_codespaces %}. Подключаться к пространству кода может только создатель пространства кода. Подключения проходят проверку подлинности в {% data variables.product.product_name %}.

Если необходимо разрешить внешний доступ к службам, работающим в пространстве кода, можно включить перенаправление портов для частного или общего доступа.

### перенаправление портов;

Если необходимо подключиться к службе (например, к веб-серверу разработки), работающей в пространстве кода, можно настроить перенаправление портов, чтобы сделать службу доступной через Интернет. 

Владельцы организации могут сделать перенаправляемые порты общедоступными или доступными только внутри организации. Дополнительные сведения см. в разделе [Ограничение видимости перенаправляемых портов](/codespaces/managing-codespaces-for-your-organization/restricting-the-visibility-of-forwarded-ports).

**Частные перенаправляемые порты**: доступны через Интернет, но доступ к ним может получить только создатель пространства кода после прохождения проверки подлинности в {% data variables.product.product_name %}.

**Общедоступные перенаправляемые порты в вашей организации**: доступны через Интернет, но доступ к ним могут получить только участники той же организации, в которой находится пространство кода, после прохождения проверки подлинности в {% data variables.product.product_name %}.

**Общедоступные перенаправляемые порты**: доступны через Интернет, и доступ к ним может получить любой пользователь в Интернете. Для доступа к общедоступным перенаправляемым портам проверка подлинности не требуется.

Все перенаправляемые порты по умолчанию являются частными, что означает, что перед доступом к ним необходимо пройти проверку подлинности. Доступ к частным перенаправляемым портам пространства кода контролируется файлами cookie проверки подлинности со сроком действия 3 часа. По истечении срока действия файла cookie необходимо повторно пройти проверку подлинности.

При удалении и повторном добавлении порта или при перезапуске пространства кода общедоступный перенаправляемый порт по умолчанию снова станет частным.

На панели "Порты" можно настроить общий или частный доступ к портам, а также отключить перенаправление портов, если оно больше не требуется. Дополнительные сведения см. в разделе [Перенаправление портов в пространстве кода](/codespaces/developing-in-codespaces/forwarding-ports-in-your-codespace).

## Рекомендации по обеспечению безопасности для пространств кода

Пространства кода по умолчанию имеют улучшенную защиту. Для поддержания этого уровня безопасности соблюдайте рекомендации по безопасности в ходе разработки: 

- Как и для любых других средств разработки, помните, что вы должны работать только с теми репозиториями, которые вы знаете и которым доверяете.
- Перед добавлением новых зависимостей в пространство кода проверьте, насколько хорошо они обслуживаются, и выпускаются ли для них обновления системы безопасности, устраняющие все уязвимости системы безопасности, обнаруженные в коде.

### Использование секретов для доступа к конфиденциальной информации 

Всегда используйте зашифрованные секреты при доступе к конфиденциальной информации (например, к маркерам доступа) в пространстве кода. К секретам можно обращаться как к переменным среды в пространстве кода, в том числе из терминала. Например, можно запустить терминал в пространстве кода и использовать команду `echo $SECRET_NAME ` для просмотра значения секрета.

Значения секретов копируются в переменные среды всякий раз, когда пространство возобновляется или создается, а также синхронизируется при их изменении.

Если у вас нет доступа на запись к репозиторию codespace, секреты не копируются в среду.

Дополнительные сведения о секретах см. в следующих разделах:
- [Управление зашифрованными секретами для пространств кода](/codespaces/managing-your-codespaces/managing-encrypted-secrets-for-your-codespaces)
- [Управление зашифрованными секретами репозитория и организации для {% data variables.product.prodname_github_codespaces %}](/codespaces/managing-codespaces-for-your-organization/managing-encrypted-secrets-for-your-repository-and-organization-for-github-codespaces)

### Работа с вкладом и репозиториями других пользователей

При создании пространства кода из ветви запроса на вытягивание в вилке маркер в пространстве кода будет отличаться в зависимости от того, является ли репозиторий общедоступным или частным:
- Для частного репозитория пространство кода получает доступ как к вилке, так и к родительскому репозиторию.
- Для общедоступного репозитория пространство кода будет иметь доступ только к вилке и открытым запросам на вытягивание в родительском репозитории.

Кроме того, мы также защищаем вас в этих сценариях, поскольку не внедряем ни один из [секретов codespace](/codespaces/managing-your-codespaces/managing-encrypted-secrets-for-your-codespaces) в среду.

### Дополнительные рекомендации

Существуют некоторые дополнительные рекомендации и риски, которые следует учитывать при использовании {% data variables.product.prodname_codespaces %}. 

#### Общие сведения о файле devcontainer.json репозитория

При создании codespace, если файл `devcontainer.json` найден для репозитория, он анализируется и используется для настройки codespace. Файл `devcontainer.json` может содержать мощные функции, такие как установка сторонних расширений и выполнение произвольного кода, предоставленного в `postCreateCommand`.

Дополнительные сведения см. в статье [Общие сведения о контейнерах разработки](/codespaces/setting-up-your-project-for-codespaces/introduction-to-dev-containers).

#### Предоставление доступа с помощью функций

Некоторые функции разработки могут привести к повышению риска в вашей среде. Например, к потенциальным проблемам при обеспечении безопасности могут привести подписывание фиксаций, секреты в переменных среды, доступ к репозиторию с проверкой подлинности и пакеты. Мы рекомендуем предоставлять доступ только тем пользователям, которым он необходим, и принять как можно более строгую политику доступа. 

#### Применение расширений

Любые дополнительные установленные расширения {% data variables.product.prodname_vscode_shortname %} также могут привести к увеличению риска. Чтобы устранить этот риск, устанавливайте только те расширения, которым вы доверяете, и регулярно обновляйте их.
