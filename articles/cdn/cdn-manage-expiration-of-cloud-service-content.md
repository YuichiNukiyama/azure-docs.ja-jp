---
title: "Azure Content Delivery Network で Web コンテンツの有効期限を管理する | Microsoft Docs"
description: "Azure CDN で Azure Web Apps/Cloud Services、ASP.NET、または IIS コンテンツの有効期限を管理する方法について説明します。"
services: cdn
documentationcenter: .NET
author: zhangmanling
manager: erikre
editor: 
ms.assetid: bef53fcc-bb13-4002-9324-9edee9da8288
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 11/10/2017
ms.author: mazha
ms.openlocfilehash: dca6ca5f21f4a4f1701af57eb40d92094b6a4754
ms.sourcegitcommit: 5d3e99478a5f26e92d1e7f3cec6b0ff5fbd7cedf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/06/2017
---
# <a name="manage-expiration-of-web-content-in-azure-content-delivery-network"></a>Azure Content Delivery Network で Web コンテンツの有効期限を管理する
> [!div class="op_single_selector"]
> * [Azure Web コンテンツ](cdn-manage-expiration-of-cloud-service-content.md)
> * [Azure BLOB Storage](cdn-manage-expiration-of-blob-content.md)
> 

パブリックにアクセス可能な任意の配信元 Web サーバーのファイルは、それらの有効期間 (TTL) が経過するまで、Azure Content Delivery Network (CDN) でキャッシュできます。 TTL は、配信元サーバーからの HTTP 応答の `Cache-Control` ヘッダーによって決まります。 この記事では、Microsoft Azure App Service、Azure Cloud Services、ASP.NET アプリケーション、およびインターネット インフォメーション サービス (IIS) サイトでの Web Apps 機能の `Cache-Control` ヘッダーを設定する方法について説明します。これらはすべて同様に構成されます。 `Cache-Control` ヘッダーを設定するには、構成ファイルを使用するか、プログラムを使用します。 

[CDN キャッシュ規則](cdn-caching-rules.md)を設定して、Azure Portal からキャッシュの設定を制御することもです。 1 つ以上のキャッシュ規則を設定し、キャッシュの動作を **[上書き]** または**[キャッシュのバイパス]** に設定すると、この記事で説明する最初に提供されるキャッシュ設定は無視されます。 全般的なキャッシュの概念については、「[How caching works (キャッシュのしくみ)](cdn-how-caching-works.md)」を参照してください。

> [!TIP]
> ファイルに TTL を設定しないことも選択できます。 この場合は、Azure Portal でキャッシュ規則を設定していない限り、Azure CDN によって既定の 7 日間の TTL が自動的に適用されます。 この既定の TTL は、一般的な Web 配信の最適化に対してのみ適用されます。 大きなファイルの最適化に対する既定の TTL は 1 日、メディア ストリーミングの最適化に対する既定の TTL は 1 年です。
> 
> ファイルとその他のリソースへのアクセスを高速化する Azure CDN のしくみについて詳しくは、「[Azure Content Delivery Network (CDN) の概要](cdn-overview.md)」をご覧ください。
> 

## <a name="setting-cache-control-headers-by-using-configuration-files"></a>構成ファイルを使用した Cache-Control ヘッダーの設定
画像やスタイル シートなどの静的コンテンツの場合は、Web アプリケーションの **applicationHost.config** ファイルか **Web.config** 構成ファイルを変更することで、更新頻度を制御できます。 いずれかのファイルの `<system.webServer>/<staticContent>/<clientCache>` 要素により、コンテンツの `Cache-Control` ヘッダーが設定されます。

### <a name="using-applicationhostconfig-files"></a>ApplicationHost.config ファイルの使用
**ApplicationHost.config** ファイルは、IIS 構成システムのルート ファイルです。 **ApplicationHost.config** ファイルの構成設定は、サイト上のすべてのアプリケーションに影響を与えますが、Web アプリケーション用の既存の **Web.config** ファイルの設定によってオーバーライドされます。

### <a name="using-webconfig-files"></a>Web.config ファイルの使用
**Web.config** ファイルの使用では、Web アプリケーション全体、または Web アプリケーション上の特定のディレクトリの動作をカスタマイズできます。 通常、Web アプリケーションのルート フォルダーには、少なくとも 1 つの **Web.config** ファイルがあります。 特定のフォルダーの各 **Web.config** ファイルについて、構成設定は、他の **Web.config** ファイルによってサブフォルダー レベルでオーバーライドされない限り、そのフォルダーとそのすべてのサブフォルダー内にあるすべてのものに影響を与えます。 たとえば、Web アプリケーションのルート フォルダー内の **Web.config** ファイルに `<clientCache>` 要素を設定して、Web アプリケーション上のすべての静的なコンテンツを 3 日間キャッシュできます。 また、可変コンテンツ (`\frequent` など) が含まれるサブフォルダーに **Web.config** ファイルを追加して、サブフォルダーのコンテンツを 6 時間キャッシュするようにその `<clientCache>` 要素を設定することもできます。 最終的に、Web サイト全体のコンテンツが 3 日間キャッシュされます (ただし、`\frequent` ディレクトリのコンテンツを除く。このコンテンツは 6 時間のみキャッシュされます)。  

次の XML の例は、3 日間の最大有効期間を指定するように構成ファイルに `<clientCache>` 要素を設定する方法を示しています。  

```xml
<configuration>
    <system.webServer>
        <staticContent>
            <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="3.00:00:00" />
        </staticContent>
    </system.webServer>
</configuration>
```

**cacheControlMaxAge** 属性を使用するには、**cacheControlMode** 属性の値を `UseMaxAge` に設定する必要があります。 この設定により、HTTP ヘッダーとディレクティブ、`Cache-Control: max-age=<nnn>` が応答に追加されます。 **cacheControlMaxAge** 属性の期間の値の形式は `<days>.<hours>:<min>:<sec>` です。 その値は秒に変換され、`Cache-Control` `max-age` ディレクティブの値として使用されます。 `<clientCache>` 要素の詳細については、[クライアント キャッシュ<clientCache>](http://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache)に関するページをご覧ください。  

## <a name="setting-cache-control-headers-programmatically"></a>プログラムによる Cache-Control ヘッダーの設定
ASP.NET アプリケーションでは、.NET API の **HttpResponse.Cache** プロパティを設定することによって、CDN のキャッシュ動作をプログラムで制御します。 **HttpResponse.Cache** プロパティの詳細については、[HttpResponse.Cache プロパティ](http://msdn.microsoft.com/library/system.web.httpresponse.cache.aspx)に関するページと [HttpCachePolicy クラス](http://msdn.microsoft.com/library/system.web.httpcachepolicy.aspx)に関するページをご覧ください。  

アプリケーションのコンテンツを ASP.NET でプログラムでキャッシュするには、次の手順に従います。
   1. `HttpCacheability` を `Public` に設定することで、コンテンツをキャッシュ可能とマークします。 
   2. 次のいずれかの `HttpCachePolicy` メソッドを呼び出して、キャッシュ検証を設定します。
      - `SetLastModified` を呼び出して、`Last-Modified` ヘッダーのタイムスタンプの値を設定します。
      - `SetETag` を呼び出して、`ETag` ヘッダーの値を設定します。
   3. 必要に応じて、`SetExpires` を呼び出して `Expires` ヘッダーの値を設定して、キャッシュの有効期間を指定します。 それ以外の場合は、このドキュメントで既に説明した既定のキャッシュのヒューリスティックが適用されます。

たとえば、コンテンツを 1 時間キャッシュするには、次の C# コードを追加します。  

```csharp
// Set the caching parameters.
Response.Cache.SetExpires(DateTime.Now.AddHours(1));
Response.Cache.SetCacheability(HttpCacheability.Public);
Response.Cache.SetLastModified(DateTime.Now);
```

## <a name="testing-the-cache-control-header"></a>Cache-Control ヘッダーのテスト
Web コンテンツの TTL 設定を簡単に確認できます。 ブラウザーの[開発者ツール](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/)を使って、Web コンテンツに `Cache-Control` 応答ヘッダーが含まれているかどうかをテストします。 **wget**、[Postman](https://www.getpostman.com/)、[Fiddler](http://www.telerik.com/fiddler) などのツールを使って応答ヘッダーを確認することもできます。

## <a name="next-steps"></a>次のステップ
* [**clientCache** 要素の詳細を確認する](http://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache)
* [**HttpResponse.Cache** プロパティのドキュメントを参照する](http://msdn.microsoft.com/library/system.web.httpresponse.cache.aspx) 
* [**HttpCachePolicy クラス**のドキュメントを参照する](http://msdn.microsoft.com/library/system.web.httpcachepolicy.aspx)  
* [キャッシュの概念を学習する](cdn-how-caching-works.md)
