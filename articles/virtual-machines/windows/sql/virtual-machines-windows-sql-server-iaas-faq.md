---
title: "Windows Azure Virtual Machines における SQL Server に関する FAQ | Microsoft Docs"
description: "この記事では、Azure VM で SQL Server を実行することに関してよく寄せられる質問に対する回答を示します。"
services: virtual-machines-windows
documentationcenter: 
author: v-shysun
manager: felixwu
editor: 
tags: azure-service-management
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 10/04/2017
ms.author: v-shysun
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8d692d806bab072fc2c4b734677223bad4142561
ms.sourcegitcommit: 0930aabc3ede63240f60c2c61baa88ac6576c508
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/07/2017
---
# <a name="frequently-asked-questions-for-sql-server-on-windows-azure-virtual-machines"></a>Windows Azure Virtual Machines における SQL Server についてよく寄せられる質問

> [!div class="op_single_selector"]
> * [Windows](virtual-machines-windows-sql-server-iaas-faq.md)
> * [Linux](../../linux/sql/sql-server-linux-faq.md)

このトピックでは、[Windows Azure Virtual Machines における SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/) に関して特に多く寄せられる質問への回答を示します。

> [!NOTE]
> このトピックでは、Windows VM 上の SQL Server に固有の問題について説明します。 Linux VM で SQL Server を実行している場合は、[Linux での FAQ](../../linux/sql/sql-server-linux-faq.md) に関する記事をご覧ください。

[!INCLUDE [support-disclaimer](../../../../includes/support-disclaimer.md)]

## <a name="frequently-asked-questions"></a>よく寄せられる質問

1. **SQL Server で Azure 仮想マシンを作成するにはどうするのですか?**

   最も簡単な解決方法は、SQL Server を含む仮想マシンを作成することです。 Azure にサインアップして、ポータルから SQL VM を作成するチュートリアルについては、「[Azure Portal での SQL Server 仮想マシンのプロビジョニング](virtual-machines-windows-portal-sql-server-provision.md)」をご覧ください。 分単位の SQL Server のライセンスを使用する仮想マシン イメージを選択するか、現在所有している SQL Server ライセンスを使用するイメージを使用できます。 無料ライセンス版 (Developer または Express) を利用するかオンプレミスのライセンスを再利用して、SQL Server を VM に手動でインストールすることもできます。 現在所有しているライセンスを使用する場合は、[Azure でのソフトウェア アシュアランスによるライセンス モビリティ](https://azure.microsoft.com/pricing/license-mobility/)が必要です。 詳細については、「[Pricing guidance for SQL Server Azure VMs](virtual-machines-windows-sql-server-pricing-guidance.md)」(SQL Server Azure VM の料金ガイダンス) を参照してください。

1. **SQL VM と SQL Database サービスの違いは何ですか?**

   概念上、Azure 仮想マシンで SQL Server を実行することは、リモート データ センターで SQL Server を実行することと違いません。 これに対し、 [SQL Database](../../../sql-database/sql-database-technical-overview.md) はサービスとしてデータベースを提供します。 SQL Database では、データベースをホストするマシンにアクセスできません。 完全な比較については、「 [クラウド SQL Server オプションの選択: Azure SQL (PaaS) Database または Azure VM (IaaS) の SQL Server](../../../sql-database/sql-database-paas-vs-sql-server-iaas.md)」をご覧ください。

1. **オンプレミス SQL Server データベースをクラウドに移行するにはどうすればよいですか?**

   まず SQL Server インスタンスで、Azure 仮想マシンを作成します。 次に、オンプレミス データベースをそのインスタンスに移行します。 データ移行の方法については、「[Azure VM の SQL Server への SQL Server データベースの移行](virtual-machines-windows-migrate-sql.md)」をご覧ください。

1. **同じ VM に SQL Server の 2 つ目のインスタンスをインストールできますか? 既定のインスタンスのインストール済みの機能を変更することはできますか?**

   はい。 SQL Server インストール メディアは、 **C** ドライブ上のフォルダーにあります。 その場所から、 **Setup.exe** を実行して、新しい SQL Server インスタンスを追加するか、またはコンピューター上の SQL Server の他のインストールされている機能を変更します。 一部の機能 (自動バックアップ、自動修正、Azure Key Vault の統合など) は、既定のインスタンスでしか動作しないので注意が必要です。

1. **SQL Server の既定のインスタンスをアンインストールできますか?**

   はい。 ただし、いくつか注意点があります。 直前の回答で述べたように、[SQL Server IaaS Agent 拡張機能](virtual-machines-windows-sql-server-agent-extension.md)に依存する機能は既定のインスタンスでしか動作しません。 この拡張機能は、アンインストールされた既定のインスタンスを探し続けるため、イベント ログ エラーが生成される可能性があります。 これらのエラーの発生源となるのは、**Microsoft SQL Server Credential Management** と **Microsoft SQL Server IaaS Agent** の 2 つです。 エラーの例を次に示します。

      SQL Server への接続を確立しているときにネットワーク関連またはインスタンス固有のエラーが発生しました。 サーバーが見つからないかアクセスできません。

   あえて既定のインスタンスをアンインストールする場合は、[SQL Server IaaS Agent 拡張機能](virtual-machines-windows-sql-server-agent-extension.md)もアンインストールしてください。
   
   >[!NOTE]
   >SQL Server Azure 仮想マシンには、「[SQL Server Azure VM の料金ガイダンス](virtual-machines-windows-sql-server-pricing-guidance.md)」に説明されているように料金が発生します。 SQL Server を削除した場合、利用料金が発生し続けます。 SQL Server が不要になった場合は、新しい仮想マシンをデプロイし、データとアプリケーションを新しい仮想マシンに移行してから、 SQL Server 仮想マシンを削除してください。

1. **Azure VM で SQL Server の新しいバージョン/エディションにアップグレードするにはどうすればよいですか?**

   現在、Azure VM で実行している SQL Server のインプレース アップグレードはありません。 目的の SQL Server バージョン/エディションで新しい Azure 仮想マシンを作成し、標準の[データ移行方法](virtual-machines-windows-migrate-sql.md)を使用して、データベースを新しいサーバーに移行します。

1. **Azure VM に SQL Server のライセンス版をインストールするにはどうすればよいですか?**

   2 つの方法があります。 [ライセンスをサポートする仮想マシン イメージ](virtual-machines-windows-sql-server-iaas-overview.md#BYOL)のいずれかをプロビジョニングできます。 SQL Server インストール メディアを Windows Server VM にコピーしてから、SQL Server を VM にインストールするオプションもあります。 ライセンス上の理由で、 [Azure でのソフトウェア アシュアランスによるライセンス モビリティ](https://azure.microsoft.com/pricing/license-mobility/)が必要です。 詳細については、「[Pricing guidance for SQL Server Azure VMs](virtual-machines-windows-sql-server-pricing-guidance.md)」(SQL Server Azure VM の料金ガイダンス) を参照してください。

1. **従量課金のギャラリー イメージから作成した VM を、現在所有している SQL Server ライセンスを使用するように変更できますか。**

   いいえ。 分単位のライセンスから、所有しているライセンスへの使用に切り替えることはできません。 いずれかの [BYOL イメージ](virtual-machines-windows-sql-server-iaas-overview.md#BYOL)を使用して新しい Azure 仮想マシンを作成し、標準の[データ移行方法](virtual-machines-windows-migrate-sql.md)を使用して、データベースを新しいサーバーに移行します。

1. **SQL Server フェールオーバー クラスター インスタンス (FCI) は Azure VM でサポートされますか?**

   はい。 [Windows フェールオーバー クラスターを Windows Server 2016 上で作成](virtual-machines-windows-portal-sql-create-failover-cluster.md)し、クラスター ストレージとして Storage Spaces Direct (S2D) を使用できます。 または、[Azure 仮想マシンでの SQL Server の高可用性と障害復旧](virtual-machines-windows-sql-high-availability-dr.md#azure-only-high-availability-solutions)に関する記事の説明に従って、サードパーティのクラスタリング ソリューションまたは記憶域ソリューションを使用できます。

1. **スタンバイ/フェールオーバーのみ使用するために作成した Azure VM 上の SQL Server のライセンスに料金を支払う必要がありますか。**

   「[Virtual Machines のライセンス FAQ](http://azure.microsoft.com/pricing/licensing-faq/)」に説明されているように、ソフトウェア アシュアランスのライセンス モビリティを使用している場合は、HA デプロイにパッシブなセカンダリ レプリカとして参加している SQL Server のライセンスに料金を支払う必要はありません。 それ以外の場合は、ライセンスの料金を支払う必要があります。

1. **SQL Server VM に更新プログラムと Service Pack を適用するにはどうすればよいですか?**

   仮想マシンでは、更新プログラムを適用するタイミングと方法を含めて、ホスト マシンを制御できます。 オペレーティング システムの場合、Windows 更新プログラムを手動で適用するか、または[自動修正](virtual-machines-windows-sql-automated-patching.md)と呼ばれるスケジュール サービスを有効にすることができます。 自動修正では、そのカテゴリ内の SQL Server の更新を含む、重要とマークされている更新プログラムがインストールされます。 SQL Server へのその他のオプションの更新プログラムは、手動でインストールする必要があります。

1. **仮想マシン ギャラリーに表示されていない構成 (Windows 2008 R2  + SQL Server 2012 など) をセットアップすることはできますか?**

   いいえ。 SQL Server を含む仮想マシン ギャラリー イメージでは、提供されているイメージのいずれかを選択する必要があります。

1. **Azure VM に SQL データ ツールをインストールするにはどうすればよいですか?**

    「[Microsoft SQL Server Data Tools - Business Intelligence for Visual Studio 2013](https://www.microsoft.com/en-us/download/details.aspx?id=42313)」から、SQL データ ツールをダウンロードしてインストールします。

## <a name="resources"></a>リソース

**Windows VM**:

* [Windows VM における SQL Server の概要](virtual-machines-windows-sql-server-iaas-overview.md)
* [SQL Server Windows VM のプロビジョニング](virtual-machines-windows-portal-sql-server-provision.md)
* [Azure VM の SQL Server へのデータベースの移行](virtual-machines-windows-migrate-sql.md)
* [Azure 仮想マシンにおける SQL Server の高可用性と障害復旧](virtual-machines-windows-sql-high-availability-dr.md)
* [Azure Virtual Machines における SQL Server のパフォーマンスに関するベスト プラクティス](virtual-machines-windows-sql-performance.md)
* [Azure Virtual Machines における SQL Server のアプリケーション パターンと開発計画](virtual-machines-windows-sql-server-app-patterns-dev-strategies.md)

**Linux VM**:

* [Linux VM における SQL Server の概要](../../linux/sql/sql-server-linux-virtual-machines-overview.md)
* [SQL Server Linux VM のプロビジョニング](../../linux/sql/provision-sql-server-linux-virtual-machine.md)
* [よく寄せられる質問 (Linux)](../../linux/sql/sql-server-linux-faq.md)
* [SQL Server on Linux のドキュメント](https://docs.microsoft.com/sql/linux/sql-server-linux-overview)
