---
title: Azure API Management で発行された API を監視する | Microsoft Docs
description: このチュートリアルの手順に従って、Azure API Management で API を監視する方法を学びます。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom: mvc
ms.topic: tutorial
ms.date: 06/15/2018
ms.author: apimpm
ms.openlocfilehash: 683b77e47857e4efbe2c24425953e9a3d83177f9
ms.sourcegitcommit: 6135cd9a0dae9755c5ec33b8201ba3e0d5f7b5a1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/31/2018
ms.locfileid: "50418848"
---
# <a name="monitor-published-apis"></a>発行された API を監視する

Azure Monitor を使用すると、Azure リソースのメトリックまたはログを視覚化、クエリ、ルーティング、アーカイブし、そのメトリックとログに対してアクションを実行できます。

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * アクティビティ ログを表示する
> * 診断ログを表示する
> * API のメトリックを表示する 
> * API が許可されていない呼び出しを受けたときのアラート ルールをセットアップする

次のビデオは、Azure Monitor を使用して API Management を監視する方法を示しています。 

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Monitor-API-Management-with-Azure-Monitor/player]

## <a name="prerequisites"></a>前提条件

+ [Azure API Management の用語](api-management-terminology.md)について学習します。
+ [Azure API Management インスタンスの作成](get-started-create-service-instance.md)に関するクイック スタートを完了します。
+ また、「[Import and publish your first API (最初の API をインポートして発行する)](import-and-publish.md)」のチュートリアルも完了します。

## <a name="view-metrics-of-your-apis"></a>API のメトリックを表示する

API Management はメトリックを 1 分間隔で出力するので、API の状態と正常性をほぼリアルタイムで把握できます。 使用できるメトリックの一部の概要を次に示します。

* 容量 (プレビュー): APIM サービスのアップグレード/ダウングレードに関する判断に役立ちます。 このメトリックは 1 分ごとに出力され、報告時のゲートウェイの容量を反映しています。 メトリックの範囲は 0 から 100 で、CPU やメモリの使用率などのゲートウェイ リソースに基づいて計算されます。
* ゲートウェイ要求の合計: 期間中の API 要求の数。 
* 成功したゲートウェイ要求: 304、307 および 301 よりも小さな値 (200 など) を含む成功した HTTP 応答コードを受信した API 要求の数。
* 失敗したゲートウェイ要求: 400 および 500 よりも大きな値を含む失敗した HTTP 応答コードを受信した API 要求の数。
* 未承認ゲートウェイ要求: 401、403、および 429 を含む HTTP 応答コードを受信した API 要求の数。
* その他のゲートウェイ要求: 上記のカテゴリのいずれにも属していない HTTP 応答コード (たとえば 418) を受信した API 要求の数。

![メトリックのグラフ](./media/api-management-azure-monitor/apim-monitor-metrics.png)

メトリックにアクセスするには、次の手順に従います。

1. ページの下部にあるメニューの **[メトリック]** を選択します。

    ![metrics](./media/api-management-azure-monitor/api-management-metrics-blade.png)

2. ドロップダウン リストで、関心のあるメトリックを選択します。 たとえば、**[Successful Gateway Requests]\(成功したゲートウェイ要求\)** を選択します。 さらにメトリックをグラフに追加することもできます。
3. グラフには、成功した API 呼び出しの合計数が表示されます。

## <a name="set-up-an-alert-rule-for-unauthorized-request"></a>許可されていない要求に対するアラート ルールをセットアップする

メトリックとアクティビティ ログに基づいてアラートを受け取るように設定できます。 Azure Monitor では、アラートがトリガーされたときに次の処理を実行するように構成することができます。

* 電子メール通知を送信する
* Webhook を呼び出す
* Azure Logic App を呼び出す

アラートを構成するには、以下の手順に従います。

1. ページの下部にあるメニュー バーの **[アラート (クラシック)]** を選択します。

    ![alerts](./media/api-management-azure-monitor/api-management-alert-rules-blade.png)

2. **[メトリック アラートの追加]** を選択します。
3. このアラートの**名前**を入力します。
4. 監視するメトリックとして、**[Unauthorized Gateway Requests]\(未承認ゲートウェイ要求\)** を選択します。
5. **[所有者、共同作成者、閲覧者に電子メールを送信]** を選択します。
6. **[OK]** をクリックします。
7. API キーなしで、Conference API を呼び出してみます。 この API Management サービスの所有者として、電子メール アラートを受信します。 

    > [!TIP]
    > アラート ルールは、トリガーされたときに webhook または Azure Logic App を呼び出すこともできます。

    ![アラートのセットアップ](./media/api-management-azure-monitor/set-up-alert.png)

## <a name="activity-logs"></a>アクティビティ ログ

アクティビティ ログは、API Management サービスで実行された操作に関する情報を提供します。 アクティビティ ログを使用すると、API Management サービスで発生した書き込み操作 (PUT、POST、DELETE) について、"いつ誰が何を" 行ったのかを確認できます。

> [!NOTE]
> アクティビティ ログには、読み取り (GET) 操作、Azure Portal で実行された操作、または元の管理 API の使用に関する情報は含まれません。

アクティビティ ログには API Management サービスでアクセスするか、Azure Monitor ですべての Azure リソースのログにアクセスできます。 

![アクティビティ ログ](./media/api-management-azure-monitor/apim-monitor-activity-logs.png)

アクティビティ ログを表示するには、次の手順に従います。

1. APIM サービス インスタンスを選びます。
2. **[アクティビティ ログ]** をクリックします。

    ![アクティビティ ログ](./media/api-management-azure-monitor/api-management-activity-logs-blade.png)

3. 目的のフィルター処理の範囲を選択し、**[適用]** をクリックします。

## <a name="diagnostic-logs"></a>診断ログ

診断ログは、監査とトラブルシューティングを行うために重要な、操作とエラーについての豊富な情報を提供します。 診断ログは、アクティビティ ログとは異なります。 アクティビティ ログは、API リソースで実行された操作に関する情報を提供します。 診断ログでは、リソースが実行した操作を調査できます。

診断ログを構成するには、次の手順に従います。

1. APIM サービス インスタンスを選びます。
2. **[診断設定]** をクリックします。

    ![診断ログ](./media/api-management-azure-monitor/api-management-diagnostic-logs-blade.png)

3. **[診断を有効にする]** をクリックします。 診断ログをメトリックと共にストレージ アカウントにアーカイブし、それらをイベント ハブにストリーム配信したり、Log Analytics に送信したりすることができます。 

現時点では、API Management は、個々の API 要求についての診断ログ (1 時間ごとにバッチ処理) を、次のスキーマを持つエントリで提供します。

```json
{  
    "isRequestSuccess" : "",
    "time": "",
    "operationName": "",
    "category": "",
    "durationMs": ,
    "callerIpAddress": "",
    "correlationId": "",
    "location": "",
    "httpStatusCodeCategory": "",
    "resourceId": "",
    "properties": {   
        "method": "", 
        "url": "", 
        "clientProtocol": "", 
        "responseCode": , 
        "backendMethod": "", 
        "backendUrl": "", 
        "backendResponseCode": ,
        "backendProtocol": "",  
        "requestSize": , 
        "responseSize": , 
        "cache": "", 
        "cacheTime": "", 
        "backendTime": , 
        "clientTime": , 
        "apiId": "",
        "operationId": "", 
        "productId": "", 
        "userId": "", 
        "apimSubscriptionId": "", 
        "backendId": "",
        "lastError": { 
            "elapsed" : "", 
            "source" : "", 
            "scope" : "", 
            "section" : "" ,
            "reason" : "", 
            "message" : ""
        } 
    }      
}  
```

| プロパティ  | type | 説明 |
| ------------- | ------------- | ------------- |
| isRequestSuccess | ブール値 | 応答の状態コードが 2xx または 3xx の範囲内で HTTP 要求が完了した場合は True |
| time | date-time | ゲートウェイによる HTTP 要求の受信タイムスタンプ |
| operationName | string | "Microsoft.ApiManagement/GatewayLogs" (定数値) |
| category | string | "GatewayLogs" (定数値) |
| durationMs | integer | ゲートウェイが要求を受信した時点から、応答全体が送信された時点までのミリ秒数 |
| callerIpAddress | string | 直接 (中間の場合もあります) のゲートウェイ呼び出し元の IP アドレス |
| correlationId | string | API Management によって割り当てられる一意の http 要求識別子 |
| location | string | 要求を処理したゲートウェイが存在する Azure リージョンの名前 |
| httpStatusCodeCategory | string | http 応答状態コードのカテゴリ: 成功 (301 以下または 304 または 307)、未承認 (401、403、429)、エラー (400、500 から 600)、その他 |
| resourceId | string | API Management リソース /SUBSCRIPTIONS/<subscription>/RESOURCEGROUPS/<resource-group>/PROVIDERS/MICROSOFT.APIMANAGEMENT/SERVICE/<name> の ID |
| properties | オブジェクト | 現在の要求のプロパティ |
| method | string | 受信要求の HTTP メソッド |
| url | string | 受信要求の URL |
| clientProtocol | string | 受信要求の HTTP プロトコル バージョン |
| responseCode | integer | クライアントに送信された HTTP 応答の状態コード |
| backendMethod | string | バックエンドに送信された要求の HTTP メソッド |
| backendUrl | string | バックエンドに送信された要求の URL |
| backendResponseCode | integer | バックエンドから受信した HTTP 応答のコード |
| backendProtocol | string | バックエンドに送信された要求の HTTP プロトコル バージョン | 
| requestSize | integer | 要求の処理中にクライアントから受信したバイト数 | 
| responseSize | integer | 要求の処理中にクライアントに送信されたバイト数 | 
| cache | string | 要求の処理における API Management キャッシュ使用の状態 (ヒット、ミス、なし) | 
| cacheTime | integer | API Management キャッシュの IO 全体 (接続バイト、送信バイト、受信バイト) に費やされたミリ秒数 | 
| backendTime | integer | バックエンドの IO 全体 (接続バイト、送信バイト、受信バイト) に費やされたミリ秒数 | 
| clientTime | integer | クライアントの IO 全体 (接続バイト、送信バイト、受信バイト) に費やされたミリ秒数 | 
| apiId | string | 現在の要求の API エンティティ識別子 | 
| operationId | string | 現在の要求の操作エンティティ識別子 | 
| productId | string | 現在の要求の製品エンティティ識別子 | 
| userId | string | 現在の要求のユーザー エンティティ識別子 | 
| apimSubscriptionId | string | 現在の要求のサブスクリプション エンティティ識別子 | 
| backendId | string | 現在の要求のバックエンド エンティティ識別子 | 
| lastError | オブジェクト | 最後の要求処理エラー | 
| elapsed | integer | ゲートウェイが要求を受信してからエラーが発生した時点までの経過ミリ秒数 | 
| source | string | エラーの原因となったポリシー (または処理中の内部ハンドラー) の名前 | 
| scope | string | エラーの原因となったポリシーを含んでいるポリシー ドキュメントのスコープ | 
| section | string | エラーの原因となったポリシーを含んでいるポリシー ドキュメントのセクション | 
| reason | string | エラーの理由 | 
| message | string | エラー メッセージ | 

## <a name="next-steps"></a>次の手順

このチュートリアルでは、以下の内容を学習しました。

> [!div class="checklist"]
> * アクティビティ ログを表示する
> * 診断ログを表示する
> * API のメトリックを表示する
> * API が許可されていない呼び出しを受けたときのアラート ルールをセットアップする

次のチュートリアルに進みます。

> [!div class="nextstepaction"]
> [呼び出しをトレースする](api-management-howto-api-inspector.md)
