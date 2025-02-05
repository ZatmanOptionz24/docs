---
title: 在 CI 系统中配置 CodeQL 运行器
shortTitle: Configure CodeQL runner
intro: '可以配置 {% data variables.code-scanning.codeql_runner %} 如何扫描项目中的代码并将结果上传到 {% data variables.product.prodname_dotcom %}。'
product: '{% data reusables.gated-features.code-scanning %}'
miniTocMaxHeadingLevel: 3
redirect_from:
  - /github/finding-security-vulnerabilities-and-errors-in-your-code/configuring-code-scanning-in-your-ci-system
  - /github/finding-security-vulnerabilities-and-errors-in-your-code/configuring-codeql-code-scanning-in-your-ci-system
  - /code-security/secure-coding/configuring-codeql-code-scanning-in-your-ci-system
  - /code-security/secure-coding/configuring-codeql-runner-in-your-ci-system
  - /code-security/secure-coding/using-codeql-code-scanning-with-your-existing-ci-system/configuring-codeql-runner-in-your-ci-system
versions:
  feature: codeql-runner-supported
type: how_to
topics:
  - Advanced Security
  - Code scanning
  - CodeQL
  - Integration
  - CI
  - Repositories
  - Pull requests
  - C/C++
  - C#
  - Java
ms.openlocfilehash: 64245dd9f320947510db3e108b30c886c95b89d1
ms.sourcegitcommit: b617c4a7a1e4bf2de3987a86e0eb217d7031490f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/11/2022
ms.locfileid: '148161068'
---
{% data reusables.code-scanning.deprecation-codeql-runner %} {% data reusables.code-scanning.beta %} {% data reusables.code-scanning.enterprise-enable-code-scanning %}

## 关于在 CI 系统中配置 {% data variables.product.prodname_codeql %}{% data variables.product.prodname_code_scanning %}

要将 {% data variables.product.prodname_code_scanning %} 集成到 CI 系统中，可以使用 {% data variables.code-scanning.codeql_runner %}。 有关详细信息，请参阅“[在 CI 系统中运行 {% data variables.code-scanning.codeql_runner %}](/code-security/secure-coding/running-codeql-runner-in-your-ci-system)”。

通常，调用 {% data variables.code-scanning.codeql_runner %} 如下所示。

```shell
$ /path/to-runner/codeql-runner-OS <COMMAND> <FLAGS>
```

`/path/to-runner/` 取决于你在 CI 系统中下载 {% data variables.code-scanning.codeql_runner %} 的位置。 `codeql-runner-OS` 取决于所使用的操作系统。
有三个版本的 {% data variables.code-scanning.codeql_runner %}：`codeql-runner-linux`、`codeql-runner-macos` 和 `codeql-runner-win`，分别适用于 Linux、macOS 和 Windows 系统。 

要自定义 {% data variables.code-scanning.codeql_runner %} 扫描代码的方式，可以使用 `--languages` 和 `--queries` 等标志，也可以在单独的配置文件中指定自定义设置。

## 扫描拉取请求

每当创建拉取请求时扫描代码可防止开发者在代码中引入新的漏洞和错误。

若要扫描拉取请求，请运行 `analyze` 命令并使用 `--ref` 标志来指定拉取请求。 引用为 `refs/pull/<PR-number>/head` 或 `refs/pull/<PR-number>/merge`，具体取决于你签出的是拉取请求分支的 HEAD 提交还是基础分支的合并提交。

```shell
$ /path/to-runner/codeql-runner-linux analyze --ref refs/pull/42/merge
```

{% note %}

注意：如果你使用第三方工具分析代码，并希望结果显示为拉取请求检查，则必须运行 `upload` 命令并使用 `--ref` 标志来指定拉取请求而不是分支。 引用为 `refs/pull/<PR-number>/head` 或 `refs/pull/<PR-number>/merge`。

{% endnote %}

## 覆盖自动语言检测

{% data variables.code-scanning.codeql_runner %} 自动检测并扫描用支持的语言编写的代码。

{% data reusables.code-scanning.codeql-languages-bullets %}

{% data reusables.code-scanning.specify-language-to-analyze %}

要替代自动语言检测，请运行带有 `--languages` 标志的 `init` 命令，后跟以逗号分隔的语言关键字列表。 支持语言的关键词是 {% data reusables.code-scanning.codeql-languages-keywords %}。

```shell
$ /path/to-runner/codeql-runner-linux init --languages cpp,java
```

## 运行额外查询

{% data reusables.code-scanning.run-additional-queries %}

{% data reusables.code-scanning.codeql-query-suites-explanation %}

要添加一个或多个查询，请将以逗号分隔的路径列表传递到 `init` 命令的 `--queries` 标志。 您也可以在配置文件中指定额外查询。

如果你还要将配置文件用于自定义设置，并使用 `--queries` 标志指定额外查询，则 {% data variables.code-scanning.codeql_runner %} 会使用通过 <nobr>`--queries`</nobr> 指定的额外查询而不是配置文件中的任何查询。
如果要运行使用该标志指定的额外查询与配置文件中指定的查询的组合集，请在传递给 <nobr>`--queries`</nobr> 的值前面加上 `+` 符号。
有关详细信息，请参阅“[使用自定义配置文件](#using-a-custom-configuration-file)”。

在下面的示例中，`+` 符号可确保 {% data variables.code-scanning.codeql_runner %} 结合使用额外查询与所引用配置文件中指定的任何查询。

```shell
$ /path/to-runner/codeql-runner-linux init --config-file .github/codeql/codeql-config.yml 
    --queries +security-and-quality,octo-org/python-qlpack/show_ifs.ql@main
```

## 使用自定义配置文件

可以在单独的配置文件中指定自定义设置，而不是将其他信息传递给 {% data variables.code-scanning.codeql_runner %} 命令。

配置文件为 YAML 文件。 它使用的语法类似于 {% data variables.product.prodname_actions %} 的工作流程语法，如下例所示。 有关详细信息，请参阅“[{% data variables.product.prodname_actions %} 的工作流语法](/actions/reference/workflow-syntax-for-github-actions)”。 

使用 `init` 命令的 `--config-file` 标志指定配置文件。 <nobr>`--config-file`</nobr> 的值是要使用的配置文件的路径。 此示例加载配置文件 .github/codeql/codeql-config.yml。

```shell
$ /path/to-runner/codeql-runner-linux init --config-file .github/codeql/codeql-config.yml
```

{% data reusables.code-scanning.custom-configuration-file %}

### 示例配置文件

{% data reusables.code-scanning.example-configuration-files %}

## 为编译语言配置 {% data variables.product.prodname_code_scanning %}

对于编译语言 C/C++、C#、{% ifversion codeql-go-autobuild %} Go {% endif %}和 Java，{% data variables.product.prodname_codeql %} 在分析之前构建代码。 {% data reusables.code-scanning.analyze-go %}

对于许多常见的生成系统，{% data variables.code-scanning.codeql_runner %} 可以自动生成代码。 要尝试自动生成代码，请在步骤 `init` 和 `analyze` 之间运行 `autobuild`。 请注意，如果您的仓库需要特定版本的构建工具，您可能需要先手动安装该构建工具。 

`autobuild` 过程仅尝试为存储库生成一种编译语言。 自动选择用于分析的语言是涵盖文件最多的语言。 如果要明确选择一种语言，请使用 `autobuild` 命令的 `--language` 标志。

```shell
$ /path/to-runner/codeql-runner-linux autobuild --language csharp
```

如果 `autobuild` 命令无法生成代码，可以在步骤 `init` 和 `analyze` 之间自行运行生成步骤。 有关详细信息，请参阅“[在 CI 系统中运行 {% data variables.code-scanning.codeql_runner %}](/code-security/secure-coding/running-codeql-runner-in-your-ci-system#compiled-language-example)”。

## 将 {% data variables.product.prodname_code_scanning %} 数据上传到 {% data variables.product.prodname_dotcom %}

默认情况下，当你运行 `analyze` 命令时，{% data variables.code-scanning.codeql_runner %} 会上传来自 {% data variables.product.prodname_code_scanning %} 的结果。 还可以使用 `upload` 命令单独上传 SARIF 文件。

上传数据后，{% data variables.product.prodname_dotcom %} 将在您的仓库中显示警报。 
- 如果已上传到拉取请求，例如 `--ref refs/pull/42/merge` 或 `--ref refs/pull/42/head`，则结果会在拉取请求检查中显示为警报。 有关详细信息，请参阅[在拉取请求中对代码扫描警报进行会审](/code-security/secure-coding/triaging-code-scanning-alerts-in-pull-requests)。
- 如果已上传到分支，例如 `--ref refs/heads/my-branch`，则结果会显示在存储库的“安全”选项卡中。 有关详细信息，请参阅“[管理存储库的代码扫描警报](/code-security/secure-coding/managing-code-scanning-alerts-for-your-repository#viewing-the-alerts-for-a-repository)”。

## {% data variables.code-scanning.codeql_runner %} 命令引用

{% data variables.code-scanning.codeql_runner %} 支持以下命令和标志。

### `init`

为每种要分析的语言初始化 {% data variables.code-scanning.codeql_runner %} 并创建 {% data variables.product.prodname_codeql %} 数据库。

| 标志 | 必需 | 输入值 |
| ---- |:--------:| ----------- |
| `--repository` | ✓ | 要初始化的仓库名称。 |
| `--github-url` | ✓ | 托管仓库的 {% data variables.product.prodname_dotcom %} 实例的 URL。 |
| <nobr>`--github-auth-stdin`</nobr> | ✓ | 从标准输入读取 {% data variables.product.prodname_github_apps %} 令牌或 {% data variables.product.pat_generic %}。 |
| `--languages` | | 要分析的语言列表，以逗号分隔。 默认情况下，{% data variables.code-scanning.codeql_runner %} 会检测并分析存储库中所有受支持的语言。 |
| `--queries` | | 除了默认的安全查询套件之外，要运行的额外查询列表，以逗号分隔。 这会覆盖自定义配置文件中的 `queries` 设置。 |
| `--config-file` | | 自定义配置文件的路径。 |
| `--codeql-path` | | 要使用的 {% data variables.product.prodname_codeql %} CLI 可执行文件副本的路径。 默认情况下，{% data variables.code-scanning.codeql_runner %} 会下载一个副本。 |
| `--temp-dir` | | 存储临时文件的目录。 默认为 `./codeql-runner`。 |
| `--tools-dir` | | 在运行之间存储 {% data variables.product.prodname_codeql %} 工具和其他文件的目录。 默认值为主目录的子目录。 |
| <nobr>`--checkout-path`</nobr> | | 检出仓库的路径。 默认值为当前工作目录。 | 
| `--debug` | | 无。 打印更详细的输出。 |
| <nobr>`--trace-process-name`</nobr> | | Advanced, Windows only. 注入此进程的 Windows 追踪器的过程名称。 |
| <nobr>`--trace-process-level`</nobr> | | Advanced, Windows only. 注入此进程的 Windows 追踪器的父进程级别数。 |
| `-h`, `--help` | | 无。 显示命令的帮助。 |

### `autobuild`

尝试为编译语言 C/C++、C# 和 Java 构建代码。 对于这些语言，{% data variables.product.prodname_codeql %} 在分析之前构建代码。 在步骤 `init` 和 `analyze` 之间运行 `autobuild`。

| 标志 | 必需 | 输入值 |
| ---- |:--------:| ----------- |
| `--language` | | 要构建的语言。 默认情况下，{% data variables.code-scanning.codeql_runner %} 会生成涵盖最多文件的编译语言。 |
| <nobr>`--temp-dir`</nobr> | | 存储临时文件的目录。 默认为 `./codeql-runner`。 |
| `--debug` | | 无。 打印更详细的输出。 |
| <nobr> `-h`, `--help`</nobr> | | 无。 显示命令的帮助。 |

### `analyze`

分析 {% data variables.product.prodname_codeql %} 数据库中的代码并将结果上传到 {% data variables.product.product_name %}。

| 标志 | 必需 | 输入值 |
| ---- |:--------:| ----------- |
| `--repository` | ✓ | 要分析的仓库名称。 |
| `--commit` | ✓ | 要分析的提交的 SHA。 在 Git 和 Azure DevOps 中，这对应于 `git rev-parse HEAD` 的值。 在 Jenkins 中，这对应于 `$GIT_COMMIT`。 |
| `--ref` | ✓ | 要分析的引用的名称，例如 `refs/heads/main` 或 `refs/pull/42/merge`。 在 Git 或 Jenkins 中，这对应于 `git symbolic-ref HEAD` 的值。 在 Azure DevOps 中，这对应于 `$(Build.SourceBranch)`。 |
| `--github-url` | ✓ | 托管仓库的 {% data variables.product.prodname_dotcom %} 实例的 URL。 |
| <nobr>`--github-auth-stdin`</nobr> | ✓ | 从标准输入读取 {% data variables.product.prodname_github_apps %} 令牌或 {% data variables.product.pat_generic %}。 |
| <nobr>`--checkout-path`</nobr> | | 检出仓库的路径。 默认值为当前工作目录。  |
| `--no-upload` | | 无。 阻止 {% data variables.code-scanning.codeql_runner %} 将结果上传到 {% data variables.product.product_name %}。 |
| `--output-dir` | | 存储输出 SARIF 文件的目录。 默认在临时文件目录中。 |
| `--ram` | | 运行查询时要使用的内存量。 默认使用所有可用的内存。 |
| <nobr>`--no-add-snippets`</nobr> | | 无。 从 SARIF 输出排除代码片段。 |
| <nobr>`--category`<nobr> | | 用于此分析的 SARIF 结果文件中要包含的类别。 类别可用于区分同一工具和提交的多次分析，但是在不同语言或代码的不同部分进行。 该值将显示在 SARIF v2.1.0 中的 `<run>.automationDetails.id` 属性中。 |
| `--threads` | | 运行查询时要使用的线程数。 默认使用所有可用的核心。 |
| `--temp-dir` | | 存储临时文件的目录。 默认为 `./codeql-runner`。 |
| `--debug` | | 无。 打印更详细的输出。 |
| `-h`, `--help` | | 无。 显示命令的帮助。 |

### `upload`

将 SARIF 文件上传到 {% data variables.product.product_name %}。

{% note %}

注意：如果使用 CodeQL 运行器分析代码，则 `analyze` 命令默认上传 SARIF 结果。 可以使用 `upload` 命令上传由其他工具生成的 SARIF 结果。

{% endnote %}

| 标志 | 必需 | 输入值 |
| ---- |:--------:| ----------- |
| `--sarif-file` | ✓ | 要上传的 SARIF 文件，或包含多个 SARIF 文件的目录。 |
| `--repository` | ✓ | 已分析的仓库名称。 |
| `--commit` | ✓ | 已分析的提交的 SHA。 在 Git 和 Azure DevOps 中，这对应于 `git rev-parse HEAD` 的值。 在 Jenkins 中，这对应于 `$GIT_COMMIT`。 |
| `--ref` | ✓ | 已分析的引用的名称，例如 `refs/heads/main` 或 `refs/pull/42/merge`。 在 Git 或 Jenkins 中，这对应于 `git symbolic-ref HEAD` 的值。 在 Azure DevOps 中，这对应于 `$(Build.SourceBranch)`。 |
| `--github-url` | ✓ | 托管仓库的 {% data variables.product.prodname_dotcom %} 实例的 URL。 |
| <nobr>`--github-auth-stdin`</nobr> | ✓ | 从标准输入读取 {% data variables.product.prodname_github_apps %} 令牌或 {% data variables.product.pat_generic %}。 |
| <nobr>`--checkout-path`</nobr> | | 检出仓库的路径。 默认值为当前工作目录。  |
| `--debug` | | 无。 打印更详细的输出。 |
| `-h`, `--help` | | 无。 显示命令的帮助。 |
