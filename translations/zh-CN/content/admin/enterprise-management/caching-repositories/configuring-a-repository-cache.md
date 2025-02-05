---
title: 配置存储库缓存
intro: '你可以通过创建新实例、将存储库缓存连接到主实例，以及配置存储库网络到存储库缓存的副本来配置 {% data variables.product.product_name %} 的存储库缓存。'
versions:
  ghes: '*'
type: how_to
topics:
  - Enterprise
ms.openlocfilehash: 682e169c55ef7ded453934720bf47f8843bc4acc
ms.sourcegitcommit: 1d757a4f3e1947fdd3868208b63041de30c9f60c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/03/2022
ms.locfileid: '148132377'
---
{% data reusables.enterprise.repository-caching-release-phase %}

## 关于存储库缓存的配置

{% data reusables.enterprise.repository-caching-config-summary %} 然后，您可以设置数据位置策略来控制将哪些存储库网络复制到存储库缓存。

群集不支持存储库缓存。

## 存储库缓存的 DNS

主实例和存储库缓存应具有不同的 DNS 名称。 例如，如果主实例位于 `github.example.com`，则可以决定将缓存命名为 `europe-ci.github.example.com` 或 `github.asia.example.com`。

要让 CI 计算机从存储库缓存，而不是主实例中进行提取，可以使用 Git 的 `url.<base>.insteadOf` 配置设置。 有关详细信息，请参阅 Git 文档中的“[`git-config`](https://git-scm.com/docs/git-config#Documentation/git-config.txt-urlltbasegtinsteadOf)”。 

例如，CI 计算机的全局 `.gitconfig` 包含这些行。

```
[url "https://europe-ci.github.example.com/"]
    insteadOf = https://github.example.com/
```

然后，当被告知提取 `https://github.example.com/myorg/myrepo` 时，Git 会改从 `https://europe-ci.github.example.com/myorg/myrepo` 中提取。

## 配置存储库缓存

{% ifversion ghes = 3.3 %} {% data reusables.enterprise_installation.ssh-into-instance %}
1. 若要启用存储库缓存，请运行以下命令。
   
   ```
   $ ghe-config cluster.cache-enabled true
   ```
{%- endif %}
1. 在所需的平台上设置新的 {% data variables.product.prodname_ghe_server %} 实例。 此实例将是你的存储库缓存。 有关详细信息，请参阅“[设置 {% data variables.product.prodname_ghe_server %} 实例](/admin/guides/installation/setting-up-a-github-enterprise-server-instance)”。
{% data reusables.enterprise_installation.replica-steps %}
1. 使用 SSH 连接到存储库缓存的 IP 地址。

   ```shell
   $ ssh -p 122 admin@REPLICA-IP
   ```
{%- ifversion ghes = 3.3 %}
1. 在缓存副本上，为存储库缓存启用功能标志。
   
   ```
   $ ghe-config cluster.cache-enabled true
   ```
{%- endif %} {% data reusables.enterprise_installation.generate-replication-key-pair %} {% data reusables.enterprise_installation.add-ssh-key-to-primary %}
1. 要验证与主实例的连接并为存储库缓存启用副本模式，请再次运行 `ghe-repl-setup`。

   ```shell
   $ ghe-repl-setup PRIMARY-IP
   ```

{% ifversion ghes < 3.6 %}
1. 为存储库缓存设置一个 `cache-location`，将 CACHE-LOCATION 替换为字母数字标识符，例如部署缓存的区域。 还要为此缓存设置数据中心名称；新缓存将尝试从同一数据中心的另一个缓存中播种。

   ```shell
   $ ghe-repl-node --cache CACHE-LOCATION --datacenter REPLICA-DC-NAME
   ```
{% else %}
1. 若要配置存储库缓存，请使用 `ghe-repl-node` 命令并包含必要的参数。
    - 为存储库缓存设置一个 `cache-location`，将 CACHE-LOCATION 替换为字母数字标识符，例如部署缓存的区域。  CACHE-LOCATION 值不能是保留用于子域隔离的任何子域，例如 `assets` 或 `media`。  有关已保留名称的列表，请参阅“[启用子域隔离](/enterprise/admin/guides/installation/enabling-subdomain-isolation#about-subdomain-isolation)”。
    - 为存储库缓存设置 `cache-domain`，同时将 EXTERNAL-CACHE-DOMAIN 替换为客户端将用于访问存储库缓存的主机名 Git。 如果没有指定 `cache-domain`，{% data variables.product.product_name %} 会将 CACHE-LOCATION 值作为子域追加到为实例配置的主机名前面。 有关详细信息，请参阅“[配置主机名](/admin/configuration/configuring-network-settings/configuring-a-hostname)”。
    - 新缓存将尝试从同一数据中心的另一个缓存中播种。 为存储库缓存设置 `datacenter`，将 REPLICA-DC-NAME 替换为要部署节点的数据中心的名称。

    ```shell
    $ ghe-repl-node --cache CACHE-LOCATION --cache-domain EXTERNAL-CACHE-DOMAIN --datacenter REPLICA-DC-NAME
    ```
{% endif %}

{% data reusables.enterprise_installation.replication-command %} {% data reusables.enterprise_installation.verify-replication-channel %}
1. 要启用存储库网络到存储库缓存的复制，请设置数据位置策略。 有关详细信息，请参阅“[数据位置策略](#data-location-policies)”。

## 数据位置策略

可以通过用 `spokesctl cache-policy` 命令为存储库配置数据位置策略来控制数据局部性。 数据位置策略确定在哪些存储库缓存上复制哪些存储库网络。 默认情况下，在配置数据位置策略之前，不会在任何存储库缓存上复制任何存储库网络。

数据位置策略仅影响 Git 内容。 无论策略如何，数据库中的内容（如问题和拉取请求注释）都将复制到所有节点。

{% note %}

注意：数据位置策略与访问控制不同。 必须使用存储库角色来控制哪些用户可以访问存储库。 有关存储库角色的详细信息，请参阅“[组织的存储库角色](/organizations/managing-access-to-your-organizations-repositories/repository-roles-for-an-organization)”。

{% endnote %} 

可以配置一个策略来复制带有 `--default` 标志的所有网络。 例如，此命令将创建一个策略，以将每个存储库网络的单个副本复制到 `cache_location` 为“kansas”的存储库缓存集。

 ```
 $ ghe-spokesctl cache-policy set --default 1 kansas
 ```

要为存储库网络配置复制，请指定作为网络根目录的存储库。 存储库网络包括一个存储库和存储库的所有分支。 如果不复制整个网络，则无法复制网络的一部分。

```
$ ghe-spokesctl cache-policy set <owner/repository> 1 kansas
```

您可以通过为网络指定副本计数为零来覆盖复制所有网络并排除特定网络的策略。 例如，此命令指定位置“kansas”中的任何存储库缓存都不能包含该网络的任何副本。

```
$ ghe-spokesctl cache-policy set <owner/repository> 0 kansas
```

不支持给定缓存位置中大于 1 的副本计数。
