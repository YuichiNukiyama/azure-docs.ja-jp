---
title: "Azure Database for PostgreSQL でのインポートとエクスポートを使用したデータベースの移行 | Microsoft Docs"
description: "PostgreSQL データベースをスクリプト ファイルに抽出し、そのファイルから対象のデータベースにデータをインポートする方法について説明します。"
services: postgresql
author: SaloniSonpal
ms.author: salonis
manager: jhubbard
editor: jasonwhowell
ms.service: postgresql
ms.topic: article
ms.date: 11/03/2017
ms.openlocfilehash: 937423d0b844f6ce30e532bbae12b85071a6f8d4
ms.sourcegitcommit: 38c9176c0c967dd641d3a87d1f9ae53636cf8260
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/06/2017
---
# <a name="migrate-your-postgresql-database-using-export-and-import"></a>エクスポートとインポートを使用した PostgreSQL データベースの移行
[pg_dump](https://www.postgresql.org/docs/9.3/static/app-pgdump.html) を使用することで、PostgreSQL データベースをスクリプト ファイルに抽出できます。また、[psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) を使用することで、そのファイルから対象のデータベースにデータをインポートできます。

## <a name="prerequisites"></a>前提条件
このハウツー ガイドの手順を実行するには、以下が必要です。
- アクセスとその下のデータベースを許可するファイアウォール規則のある [Azure Database for PostgreSQL サーバー](quickstart-create-server-database-portal.md)
- インストールされた [pg_dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html) コマンド ライン ユーティリティ
- インストールされた [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) コマンド ライン ユーティリティ

次の手順に従って PostgreSQL データベースのエクスポートとインポートを行います。

## <a name="create-a-script-file-using-pgdump-that-contains-the-data-to-be-loaded"></a>pg_dump を使用して読み込むデータが含まれたスクリプト ファイルを作成する
オンプレミスまたは VM 内にある既存の PostgreSQL データベースを SQL スクリプト ファイルにエクスポートするには、既存の環境で次のコマンドを実行します。
```bash
pg_dump –-host=<host> --username=<name> --dbname=<database name> --file=<database>.sql
```
たとえば、ローカル サーバーとその中に **testdb** というデータベースがあるとします。
```bash
pg_dump --host=localhost --username=masterlogin --dbname=testdb --file=testdb.sql
```

## <a name="import-the-data-on-target-azure-database-for-postrgesql"></a>対象の Azure Database for PostrgeSQL にデータをインポートする
psql コマンド ラインと --dbname パラメーター (-d) を使用して、Azure Database for PostgreSQL サーバーにデータをインポートし、SQL ファイルからデータを読み込みます。
```bash
psql --file=<database>.sql --host=<server name> --port=5432 --username=<user@servername> --dbname=<target database name>
```
この例は、psql ユーティリティと前の手順で作成した **testdb.sql** という名前のスクリプト ファイルを使って、対象サーバー **mypgserver-20170401.postgres.database.azure.com** のデータベース **mypgsqldb** にデータをインポートします。
```bash
psql --file=testdb.sql --host=mypgserver-20170401.database.windows.net --port=5432 --username=mylogin@mypgserver-20170401 --dbname=mypgsqldb
```

## <a name="next-steps"></a>次のステップ
- ダンプと復元を使用して PostgreSQL データベースを移行するには、「[ダンプと復元を使用した PostgreSQL データベースの移行](howto-migrate-using-dump-and-restore.md)」をご覧ください。
