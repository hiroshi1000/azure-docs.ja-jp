---
title: Azure Digital Twins で UDF をデバッグする方法 | Microsoft Docs
description: Azure Digital Twins で UDF をデバッグする方法に関するガイドライン
author: stefanmsft
manager: deshner
ms.service: digital-twins
services: digital-twins
ms.topic: conceptual
ms.date: 10/22/2018
ms.author: stefanmsft
ms.openlocfilehash: 852b2d35ae605f5529d162d52655fd258ca07c5a
ms.sourcegitcommit: 9e179a577533ab3b2c0c7a4899ae13a7a0d5252b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/23/2018
ms.locfileid: "49946098"
---
# <a name="how-to-debug-issues-with-user-defined-functions-in-azure-digital-twins"></a>Azure Digital Twins のユーザー定義関数の問題をデバッグする方法

この記事では、ユーザー定義関数を診断する方法について説明します。 次に、それらを操作するときに直面する最も一般的なシナリオをいくつか紹介します。

## <a name="debug-issues"></a>問題をデバッグする

Azure Digital Twins 内で発生するあらゆる問題を診断する方法について理解することで、問題、原因、解決策を効率的に特定するのに役立ちます。

### <a name="enable-log-analytics-for-your-instance"></a>インスタンスのログ分析を有効にする

Azure Digital Twins インスタンスのログとメトリックは Azure Monitor を通じて公開されます。 次のドキュメントでは、[Azure portal](../log-analytics/log-analytics-quick-create-workspace.md)、[Azure CLI](../log-analytics/log-analytics-quick-create-workspace-cli.md)、または [PowerShell](../log-analytics/log-analytics-quick-create-workspace-posh.md) を通じて [Azure Log Analytics](../log-analytics/log-analytics-queries.md) ワークスペースを作成していると仮定しています。

> [!NOTE]
> 初めて **Log Analytics** にイベントを送信するときに 5 分の遅延が発生することがあります。

「[Azure リソースからのログ データの収集と使用](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)」の記事を参考に、Azure portal、Azure CLI、または PowerShell を使用して Azure Digital Twins インスタンスの診断設定を有効にします。 すべてのログ カテゴリ、メトリック、および Azure Log Analytics ワークスペースを選択してください。

### <a name="trace-sensor-telemetry"></a>センサー テレメトリのトレース

Azure Digital Twins インスタンスの診断設定が有効になっていること、すべてのカテゴリが選択されていること、および Azure Log Analytics が選択されていることを確認します。

センサー テレメトリのメッセージが対応するログと一致するように、送信されるイベント データに相関 ID を指定できます。 これを行うには、`x-ms-client-request-id` プロパティを GUID に設定します。

テレメトリを送信した後は、Azure Log Analytics を開き、設定された相関 ID を使用してログに対してクエリを実行します。

```Kusto
AzureDiagnostics
| where CorrelationId = 'yourCorrelationIdentifier'
```

| カスタム属性の名前 | 置換後の文字列 |
| --- | --- |
| *yourCorrelationIdentifier* | イベント データで指定した相関 ID |

ユーザー定義関数をログに記録した場合、それらのログがカテゴリ `UserDefinedFunction` で Azure Log Analytics インスタンスに表示されます。 それらを取得するには、Azure Log Analytics で、次のクエリ条件を入力します。

```Kusto
AzureDiagnostics
| where Category = 'UserDefinedFunction'
```

強力なクエリ操作について詳しくは、[クエリの概要](https://docs.microsoft.com/azure/log-analytics/query-language/get-started-queries)に関するページをご覧ください。

## <a name="identify-common-issues"></a>一般的な問題の識別

ソリューションに対してトラブルシューティングを実行するときは、一般的な問題の診断と識別の両方が重要です。 ユーザー定義関数を開発するときに直面するいくつかの一般的な問題を以下にまとめています。

### <a name="ensure-a-role-assignment-was-created"></a>ロールの割り当てが作成されていることを確認する

Management API 内にロールの割り当てが作成されないと、ユーザー定義関数に通知の送信、メタデータの取得、トポロジ内での計算された値の設定などのアクションを実行するアクセス権限が付与されません。

次の Management API を使用してユーザー定義関数にロールの割り当てが存在することを確認してください。

```plaintext
GET https://yourManagementApiUrl/api/v1.0/roleassignments?path=/&traverse=Down&objectId=yourUserDefinedFunctionId
```

| カスタム属性の名前 | 置換後の文字列 |
| --- | --- |
| *yourManagementApiUrl* | Management API の完全な URL パス  |
| *yourUserDefinedFunctionId* | ロールの割り当てを取得するユーザー定義関数の ID|

ロールの割り当てを取得できない場合は、[ユーザー定義関数にロールの割り当てを作成する方法](./how-to-user-defined-functions.md)に関する記事をご覧ください。

### <a name="check-if-the-matcher-will-work-for-a-sensors-telemetry"></a>センサー テレメトリでマッチャーが機能することを確認する

Azure Digital Twins インスタンスの Management API に対して次の呼び出しを実行すると、指定のマッチャーが指定のセンサーに適用されるかどうかを判断できます。

```plaintext
GET https://yourManagementApiUrl/api/v1.0/matchers/yourMatcherIdentifier/evaluate/yourSensorIdentifier?enableLogging=true
```

| カスタム属性の名前 | 置換後の文字列 |
| --- | --- |
| *yourManagementApiUrl* | Management API の完全な URL パス  |
| *yourMatcherIdentifier* | 評価するマッチャーの ID |
| *yourSensorIdentifier* | 評価するセンサーの ID |

応答:

```JavaScript
{
    "success": true,
    "logs": [
        "$.dataType: \"Motion\" Equals \"Motion\" => True"
    ]
}
```

### <a name="check-what-a-sensor-will-trigger"></a>トリガーされるセンサーを確認する

Azure Digital Twins インスタンスの Management API に対して次の呼び出しを実行すると、指定のセンサーの受信テレメトリによってトリガーされるユーザー定義関数の識別子を判別できます。

```plaintext
GET https://yourManagementApiUrl/api/v1.0/sensors/yourSensorIdentifier/matchers?includes=UserDefinedFunctions
```

| カスタム属性の名前 | 置換後の文字列 |
| --- | --- |
| *yourManagementApiUrl* | Management API の完全な URL パス  |
| *yourSensorIdentifier* | テレメトリを送信するセンサーの ID |

応答:

```JavaScript
[
    {
        "id": "48a64768-797e-4832-86dd-de625f5f3fd9",
        "name": "MotionMatcher",
        "spaceId": "2117b3e1-b6ce-42c1-9b97-0158bef59eb7",
        "conditions": [
            {
                "id": "024a067a-414f-415b-8424-7df61392541e",
                "target": "Sensor",
                "path": "$.dataType",
                "value": "\"Motion\"",
                "comparison": "Equals"
            }
        ],
        "userDefinedFunctions": [
            {
                "id": "373d03c5-d567-4e24-a7dc-f993460120fc",
                "spaceId": "2117b3e1-b6ce-42c1-9b97-0158bef59eb7",
                "name": "Motion User-Defined Function",
                "disabled": false
            }
        ]
    }
]
```

### <a name="issue-with-receiving-notifications"></a>通知の受信に関する問題

トリガーされたユーザー定義関数内から通知を受信していない場合は、トポロジのオブジェクト型パラメーターが使用されている識別子の型と一致していることを確認します。

**正しくない**例:

```JavaScript
var customNotification = {
    Message: 'Custom notification that will not work'
};

sendNotification(telemetry.SensorId, "Space", JSON.stringify(customNotification));
```

このシナリオは、使用された識別子がセンサーを参照する一方で、指定されたトポロジのオブジェクト型が "Space" であるため発生します。

**正しい**例:

```JavaScript
var customNotification = {
    Message: 'Custom notification that will work'
};

sendNotification(telemetry.SensorId, "Sensor", JSON.stringify(customNotification));
```

この問題に直面しないための最も簡単な方法は、メタデータ オブジェクトに `Notify` メソッドを使用する方法です。

例:

```JavaScript
function process(telemetry, executionContext) {
    var sensorMetadata = getSensorMetadata(telemetry.SensorId);

    var customNotification = {
        Message: 'Custom notification'
    };

    // Short-hand for above methods where object type is known from metadata.
    sensorMetadata.Notify(JSON.stringify(customNotification));
}
```

## <a name="common-diagnostic-exceptions"></a>一般的な診断の例外

診断設定を有効にした場合は、これらの一般的な例外が発生する可能性があります。

1. **調整**: ユーザー定義関数が[サービスの制限](./concepts-service-limits.md)の記事で説明されている実行レート制限を超える場合、調整が行われます。 その制限が解除されるまで、調整は追加の操作が正常に実行されることを必要としません。

1. **データが見つからない**: ユーザー定義関数が存在しないメタデータにアクセスを試行すると、操作は失敗します。

1. **承認されていない**: ユーザー定義関数にロールの割り当てが設定されていない、またはトポロジから特定のメタデータにアクセスするための十分なアクセス許可がない場合、操作は失敗します。

## <a name="next-steps"></a>次の手順

Azure Digital Twins で[監視とログ](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)を有効にする方法について確認する。
