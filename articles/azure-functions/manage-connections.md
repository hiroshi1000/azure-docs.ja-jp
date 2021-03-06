---
title: Azure Functions で接続を管理する方法
description: 静的接続クライアントを使用して、Azure Functions のパフォーマンスの問題を回避する方法について説明します。
services: functions
author: ggailey777
manager: jeconnoc
ms.service: azure-functions
ms.topic: conceptual
ms.date: 11/02/2018
ms.author: glenga
ms.openlocfilehash: eb5c302c807f85f24f53fa1ba32ef4cd7b52274a
ms.sourcegitcommit: f0c2758fb8ccfaba76ce0b17833ca019a8a09d46
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/06/2018
ms.locfileid: "51036463"
---
# <a name="how-to-manage-connections-in-azure-functions"></a>Azure Functions で接続を管理する方法

関数アプリ内の Functions はリソースを共有します。それらの共有リソースの中には、HTTP 接続、データベース接続、ストレージなどの Azure サービスへの接続があります。 多くの関数が同時に実行されている場合、利用可能な接続がなくなる可能性があります。 この記事では、実際に必要な接続よりも多くの接続を使用しないように関数をコーディングする方法について説明します。

## <a name="connections-limit"></a>接続の制限

利用できる接続の数が制限される理由の一部は、関数アプリが [Azure App Service サンドボックス](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)内で実行されるためです。 サンドボックスがコードに課す制限の 1 つは、[接続数の上限 (現在は 300)](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#numerical-sandbox-limits) です。 この制限に達すると、関数ランタイムは `Host thresholds exceeded: Connections` というメッセージでログを作成します。

より多くの要求を処理するために[スケール コントローラーが関数アプリのインスタンスを追加する](functions-scale.md#how-the-consumption-plan-works)と、制限を超える可能性が高くなります。 各関数アプリ インスタンスは一度に多くの関数を実行でき、そのすべてが 300 の制限に加算される接続を使用します。

## <a name="use-static-clients"></a>静的クライアントを使用する

必要以上に多くの接続を保持しないようにするには、関数の呼び出しごとに新しいインスタンスを作成するのではなく、クライアント インスタンスを再利用します。 [HttpClient](https://msdn.microsoft.com/library/system.net.http.httpclient(v=vs.110).aspx)、[DocumentClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.documentclient
)、Azure Storage クライアントのような .NET クライアントは、単一の静的クライアントを使用する場合に接続を管理できます。

Azure Functions アプリケーションでサービス固有のクライアントを使用する場合のガイドラインを次に示します。

- 関数の呼び出しごとに新しいクライアントを**作成しない**。
- 関数の呼び出しごとに使用できる単一の静的クライアントを**作成する**。
- 異なる関数が同じサービスを使用している場合、共有ヘルパー クラスで単一の静的クライアントを**作成することを検討する**。

## <a name="client-code-examples"></a>クライアント コードの例

このセクションでは、関数のコードからクライアントを作成および使用するためのベスト プラクティスを示します。

### <a name="httpclient-example-c"></a>HttpClient の例 (C#)

静的 [HttpClient](https://msdn.microsoft.com/library/system.net.http.httpclient(v=vs.110).aspx) を作成する C# 関数コードの例を次に示します。

```cs
// Create a single, static HttpClient
private static HttpClient httpClient = new HttpClient();

public static async Task Run(string input)
{
    var response = await httpClient.GetAsync("http://example.com");
    // Rest of function
}
```

.NET の [HttpClient](https://msdn.microsoft.com/library/system.net.http.httpclient(v=vs.110).aspx) については、"クライアントを破棄する方がよいですか" という質問がよく寄せられます。 一般的に、`IDisposable` を実装したオブジェクトは、使用の終了後に破棄します。 ただし、関数の終了時に静的クライアントの使用は終了しないため、静的クライアントは破棄しません。 アプリケーションの起動中は、静的クライアントを存続することができます。

### <a name="http-agent-examples-nodejs"></a>HTTP エージェントの例 (Node.js)

優れた接続管理オプションが提供されることから、`node-fetch` モジュールなどの非ネイティブ メソッドではなくネイティブの [`http.agent`](https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_class_http_agent) クラスを使用する必要があります。 接続パラメーターは `http.agent` クラスのオプションを使用して構成されます。 HTTP エージェントで利用できるオプションの詳細については、 [new Agent(\[options\])](https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_new_agent_options) を参照してください。

`http.request()` で使用されるグローバル `http.globalAgent` では、これらのすべての値がそれぞれの既定値に設定されます。 関数の接続制限を構成するための推奨される方法は、グローバルに最大数を設定することです。 次の例は、関数アプリのソケットの最大数を設定します。

```js
http.globalAgent.maxSockets = 200;
```

 次の例は、HTTP 要求に対してのみ使用されるカスタム HTTP エージェントで新しい HTTP 要求を作成します。

```js
var http = require('http');
var httpAgent = new http.Agent();
httpAgent.maxSockets = 200;
options.agent = httpAgent;
http.request(options, onResponseCallback);
```

### <a name="documentclient-code-example-c"></a>DocumentClient のコード例 (C#)

[DocumentClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.documentclient
) は、Azure Cosmos DB のインスタンスに接続します。 Azure Cosmos DB のドキュメントでは、[アプリケーションの有効期間中はシングルトン Azure Cosmos DB クライアントを使用する](https://docs.microsoft.com/azure/cosmos-db/performance-tips#sdk-usage)ことが推奨されています。 次の例では、関数内でそれを行うパターンの 1 つを示します。

```cs
#r "Microsoft.Azure.Documents.Client"
using Microsoft.Azure.Documents.Client;

private static Lazy<DocumentClient> lazyClient = new Lazy<DocumentClient>(InitializeDocumentClient);
private static DocumentClient documentClient => lazyClient.Value;

private static DocumentClient InitializeDocumentClient()
{
    // Perform any initialization here
    var uri = new Uri("example");
    var authKey = "authKey";
    
    return new DocumentClient(uri, authKey);
}

public static async Task Run(string input)
{
    Uri collectionUri = UriFactory.CreateDocumentCollectionUri("database", "collection");
    object document = new { Data = "example" };
    await documentClient.UpsertDocumentAsync(collectionUri, document);
    
    // Rest of function
}
```

## <a name="sqlclient-connections"></a>SqlClient の接続

関数のコードでは、SQL リレーショナル データベースに接続するために、SQL Server に対する .NET Framework データ プロバイダー ([SqlClient](https://msdn.microsoft.com/library/system.data.sqlclient(v=vs.110).aspx)) を使うことがあります。 これは、Entity Framework のような ADO.NET に依存するデータ フレームワークの基になるプロバイダーでもあります。 [HttpClient](https://msdn.microsoft.com/library/system.net.http.httpclient(v=vs.110).aspx) や [DocumentClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.documentclient
) の接続とは異なり、ADO.NET は接続プールを既定で実装します。 ただし、それでも接続を使い果たす可能性があるため、データベースへの接続を最適化する必要があります。 詳細については、「[SQL Server Connection Pooling (ADO.NET)](https://docs.microsoft.com/dotnet/framework/data/adonet/sql-server-connection-pooling)」(SQL Server の接続プーリング (ADO.NET)) をご覧ください。

> [!TIP]
> [Entity Framework](https://msdn.microsoft.com/library/aa937723(v=vs.113).aspx) などの一部のデータ フレームワークは、通常、構成ファイルの **ConnectionStrings** セクションから接続文字列を取得します。 その場合は、関数アプリの設定およびローカル プロジェクトの [local.settings.json ファイル](functions-run-local.md#local-settings-file)の**接続文字列**コレクションに、SQL データベースの接続文字列を明示的に追加する必要があります。 関数のコードで [SqlConnection](https://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection(v=vs.110).aspx) を作成する場合は、接続文字列の値を他の接続と共に**アプリケーションの設定**に格納する必要があります。

## <a name="next-steps"></a>次の手順

静的クライアントが推奨される理由の詳細については、「[不適切なインスタンス化のアンチパターン](https://docs.microsoft.com/azure/architecture/antipatterns/improper-instantiation/)」を参照してください。

Azure Functions のパフォーマンスに関するその他のヒントについては、「[Azure Functions のパフォーマンスと信頼性を最適化する](functions-best-practices.md)」を参照してください。
