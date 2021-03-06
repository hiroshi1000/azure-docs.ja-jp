---
title: Azure Monitor での統合アラート
description: Azure サービスにまたがるアラートおよびアラート ルールを管理できる、Azure での統合アラートの説明。
author: manishsm-msft
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 06/07/2018
ms.author: mamit
ms.component: alerts
ms.openlocfilehash: 30b2d60868702c6113612668b8e4cf9975aa2c40
ms.sourcegitcommit: ada7419db9d03de550fbadf2f2bb2670c95cdb21
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/02/2018
ms.locfileid: "50962199"
---
# <a name="unified-alerts-in-azure-monitor"></a>Azure Monitor での統合アラート

## <a name="overview"></a>概要

> [!NOTE]
>  複数のサブスクリプションからアラートを管理でき、さらにアラートの状態やスマート グループを導入する新しい統合アラート エクスペリエンスが現在、パブリック プレビューで使用できます。 この拡張されたエクスペリエンスと、それを有効にするためのプロセスについては、この記事の最後のセクションを参照してください。


この記事では、Azure Monitor での統合アラート エクスペリエンスについて説明します。 [以前のアラート エクスペリエンス](monitoring-overview-alerts.md)は、Azure Monitor メニューの **[アラート (クラシック)]** オプションから使用できます。 

## <a name="features-of-the-unified-alert-experience"></a>統合アラート エクスペリエンスの機能

統合されたエクスペリエンスには、クラシック エクスペリエンスに比べて次の利点があります。

-   **より優れた通知システム**: [アクション グループ](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-action-groups)は、複数のアラートで再利用できる通知とアクションの名前付きグループです。 
- **統合された作成エクスペリエンス**: Azure Monitor、Log Analytics、および Application Insights にわたるメトリック、ログ、アクティビティ ログのアラートおよびアラート ルールを 1 か所で管理できます。 
- **発生した Log Analytics のアラートを Azure portal で表示する** - Log Analytics のアラートを、Azure portal で他のソースからのアラートと共に表示できるようになりました。 以前は、他のソースからのアラートは別のポータルに表示されました。
- **発生したアラートおよびアラート ルールの分離**: アラート ルールは現在、アラートから区別されます。 アラート ルールは、アラートをトリガーする条件の定義です。 アラートは、発生したアラート ルールのインスタンスです。
- **より優れたワークフロー**: 統合アラートの作成エクスペリエンスは、アラート ルールを構成するプロセスを案内します。
 
メトリック アラートは、クラシック メトリック アラートに比べて次の点が機能強化されています。

-   **待ち時間の短縮**: メトリック アラートを 1 分ごとの頻度で実行できます。 クラシック メトリック アラートは常に、5 分ごとに 1 回の頻度で実行されました。 ログ アラートは、ログの取り込みに時間がかかるため、まだ 1 分を超える遅延が発生します。 
-   **多次元メトリックのサポート**: 各次元のメトリックに関するアラートを生成して、メトリックの特定のインスタンスを監視できます。
-   **メトリックの条件に対する制御の向上**: メトリックの最大、最小、平均、および合計値の監視をサポートするより高機能なアラート ルールを定義できます。
-   **複数のメトリックの監視の組み合わせ**: 最大 2 つのメトリックを 1 つのルールで監視できます。 両方のメトリックが、指定された期間にわたってしきい値を超えた場合、アラートがトリガーされます。
-   **ログからのメトリック** (限定パブリック プレビュー): Log Analytics に移動するログ データの一部を抽出して、Azure Monitor メトリックに変換し、他のメトリックと同様にアラートの対象にできるようになりました。 


## <a name="alert-rules"></a>アラート ルール
統合アラート エクスペリエンスでは、アラート ルールとアラートを区別しながら、さまざまなアラートの種類にわたる作成エクスペリエンスを統合するために次の概念を使用します。

| Item | 定義 |
|:---|:---|
| アラート ルール | アラートを作成するための条件の定義。 アラート ルールは、"_ターゲット リソース_"、"_シグナル_"、"_条件_"、および "_ロジック_" で構成されます。 アラート ルールは、_有効な_状態にある場合にのみアクティブになります。
| ターゲット リソース | アラートに使用できる特定のリソースとシグナルを定義します。 ターゲットには、任意の Azure リソースを指定できます。<br><br>例: 仮想マシン、ストレージ アカウント、仮想マシン スケール セット、Log Analytics ワークスペース、Application Insights リソース |
| シグナル | ターゲット リソースによって出力されるデータのソース。 サポートされるシグナルの種類は、*メトリック*、*アクティビティ ログ*、*Application Insights*、および*ログ*です。 |
| 条件 | ターゲット リソースに適用される_シグナル_と_ロジック_の組み合わせ。<br><br>(例: CPU 使用率が 70% を超える、サーバー応答時間が 4 ミリ秒を超える、ログ クエリの結果数が 100 を超える)。 |
| ロジック | シグナルが予測された範囲/値内にあることを確認するためのユーザー定義ロジック。 |
| Action | アラートが発生したときに実行するアクション。 アラートが発生したときに、複数のアクションが実行される可能性があります。 これらのアラートは、アクション グループをサポートします。<br><br>例: メール アドレスへのメール送信、webhook URL の呼び出し。 |
| 監視条件 | メトリック アラートを作成した条件が解決されたかどうかを示します。 メトリック アラート ルールは、特定のメトリックを定期的にサンプリングします。 そのアラート ルールの条件が満たされた場合は、新しいアラートが "発生済み" の条件で作成されます。  そのメトリックが再びサンプリングされたとき、条件がまだ満たされている場合は何も発生しません。  条件が満たされていない場合は、そのアラートの条件が "解決済み" に変更されます。 後でその条件が満たされた場合は、別のアラートが "発生済み" の条件で作成されます。 |


## <a name="alert-pages"></a>アラート ページ
統合アラートには、すべての Azure アラートを表示および管理するための 1 つの場所が用意されています。 以降のセクションでは、統合されたエクスペリエンスの各ページの機能について説明します。

### <a name="alerts-overview-page"></a>アラートの概要ページ
**[アラート]** 概要ページには、発生したすべてのアラートの集計された概要と、有効なアラート ルールの合計数が表示されます。 サブスクリプションまたはフィルター パラメーターを変更すると、集計と発生したアラートの一覧が更新されます。

 ![alerts-overview](./media/monitoring-overview-unified-alerts/alerts-preview-overview2.png) 

### <a name="alert-rules-management"></a>アラート ルールの管理
**[ルール]** は、Azure サブスクリプションにまたがるすべてのアラート ルールを管理するための単一のページです。 ここにはすべてのアラート ルールが一覧表示され、それをターゲット リソース、リソース グループ、ルール名、または状態に基づいて並べ替えることができます。 このページからアラート ルールを編集したり、有効または無効にしたりすることもできます。

 ![alerts-rules](./media/monitoring-overview-unified-alerts/alerts-preview-rules.png)


## <a name="create-an-alert-rule"></a>アラート ルールを作成する
アラートは、監視サービスやシグナルの種類には関係なく、一貫した方法で作成できます。 発生したすべてのアラートや関連する詳細を単一のページで確認できます。
 
新しいアラート ルールは、次の 3 つの手順で作成します。
1. アラートの_ターゲット_を選択します。
1. そのターゲットに使用できるシグナルから_シグナル_を選択します。
1. そのシグナルからのデータに適用される_ロジック_を指定します。
 
この簡素化された作成プロセスでは、ユーザーは Azure リソースを選択する前に、サポートされている監視ソースやシグナルを把握しておく必要はなくなります。 使用可能なシグナルの一覧は、ユーザーが選択したターゲット リソースに基づいて自動的にフィルター処理され、アラート ルールのロジックの定義を案内します。

アラート ルールを作成する方法の詳細については、「[Azure Monitor を使用してアラートを作成、表示、管理する](alert-metric.md)」を参照してください。

アラートは複数の Azure 監視サービス全体で使用できます。 これらの各サービスを使用する方法およびタイミングについては、「[Azure のアプリケーションおよびリソースの監視](../azure-monitor/overview.md)」を参照してください。 次の表では、Azure 全体で使用できるアラートのルールの種類の一覧を示します。 また、統合アラート エクスペリエンスで現在サポートされているものの一覧も示します。

| **モニター ソース** | **シグナルの種類**  | **説明** | 
|-------------|----------------|-------------|
| Azure Monitor | メトリック  | [ほぼリアルタイムのメトリック アラート](monitoring-near-real-time-metric-alerts.md)とも呼ばれます。これらは、1 分おきの頻度でのメトリック条件の評価をサポートし、マルチメトリックおよび多次元メトリックのルールを使用できます。 サポートされるリソースの種類の一覧は、「[Azure Portal で使用できる Azure サービスの新しいメトリック アラート](monitoring-near-real-time-metric-alerts.md#metrics-and-dimensions-supported)」で確認できます。<br>[クラシック メトリック アラート](monitoring-overview-alerts.md)は、新しいアラート エクスペリエンスではサポートされていません。 これらは、Azure Portal の [アラート (クラシック)] の下にあります。 クラシック アラートは、新しいアラートにまだ移動されていない一部のメトリックの種類をサポートします。 完全な一覧については、[サポートされるメトリック](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-supported-metrics)に関するページを参照してください。 |
| Log Analytics | ログ  | ログ検索クエリが特定の条件を満たしたときに、通知を受信するか、または自動化されたアクションを実行します。 Log Analytics のアラートは、[新しいエクスペリエンスにコピーされます](monitoring-alerts-extend.md)。 [*メトリックとしての Log Analytics ログのプレビュー*](monitoring-alerts-extend-tool.md)を使用できます。 このプレビュー機能では、一部のログの種類を取得してメトリックに変換し、新しいアラート エクスペリエンスを使ってそれらについてのアラートを行うことができます。 このプレビュー機能は、ネイティブの Azure Monitor メトリックと共に Azure 以外のログを取得したい場合に便利です。 |
| アクティビティ ログ | アクティビティ ログ | 選択したターゲットによって作成されたすべての作成、更新、削除アクションのレコードが含まれます。 |
| サービス正常性 | アクティビティ ログ  | 統合アラートではサポートされていません。 「[サービス通知のアクティビティ ログ アラートを作成する](monitoring-activity-log-alerts-on-service-notifications.md)」をご覧ください。  |
| Application Insights | ログ  | アプリケーションのパフォーマンスの詳細が記録されたログが含まれます。 分析クエリを使用すると、アプリケーション データに基づいて実行されるアクションの条件を定義できます。 |
| Application Insights | メトリック | 統合アラートではサポートされていません。 [メトリック アラート](../application-insights/app-insights-alerts.md)に関するページをご覧ください。 |
| Application Insights | Web 可用性テスト | 統合アラートではサポートされていません。  [Web テスト アラート](../application-insights/app-insights-monitor-web-app-availability.md)に関するページをご覧ください。 Application Insights にデータを送信するようにインストルメント化されたすべての Web サイトで使用できます。 Web サイトの可用性または応答性が期待を下回る場合に通知を受け取ります。 |

## <a name="enhanced-unified-alerts-public-preview"></a>拡張された統合アラート (パブリック プレビュー)

拡張された統合アラート エクスペリエンスは、2018 年 6 月 1 日に Azure Monitor のパブリック プレビューでリリースされました。 このエクスペリエンスは、2018 年 3 月にリリースされた[統合アラート](#overview)の利点に基づいて構築されており、個々のアラートを管理および集計し、アラートの状態を変更する機能を提供します。 このセクションでは、新機能と、Azure Portal で新しいアラート ページに移動する方法について説明します。

### <a name="enhanced-unified-alerts"></a>拡張された統合アラート

新しいエクスペリエンスは、従来の統合されたエクスペリエンスでは使用できない次の機能を提供します。

- **サブスクリプションにまたがるアラートを表示する**: 複数のサブスクリプションにまたがるアラートの個々のインスタンスを 1 つのビューで表示および管理できるようになりました。
- **アラートの状態を管理する**: アラートには、それが終了済みとして確認されたかどうかを示す状態が割り当てられています。
- **スマート グループを使用してアラートを整理する**: スマート グループは、関連するアラートを個別にではなく、セットとして管理できるように自動的にグループ化します。

### <a name="enable-enhanced-unified-alerts"></a>拡張された統合アラートを有効にする
新しい統合アラート エクスペリエンスを有効にするには、[アラート] ページの上部にあるバナーを選択します。 このプロセスにより、サポートされるサービスにまたがる過去 30 日間の発生したアラートを含むアラート ストアが作成されます。 新しいエクスペリエンスが有効になった後、このバナーを選択することによって、新しいエクスペリエンスと古いエクスペリエンスを切り替えることができます。

> [!NOTE]
>  新しいエクスペリエンスが最初に有効になるには数分かかることがあります。

![バナー](media/monitoring-overview-unified-alerts/opt-in-banner.png)

新しいエクスペリエンスを有効にすると、アクセスできるすべてのサブスクリプションが登録されます。 サブスクリプション全体が有効になりますが、新しいエクスペリエンスを表示できるのは、それを選択したユーザーだけです。 そのサブスクリプションにアクセスできる他のユーザーは、このエクスペリエンスを個別に有効にする必要があります。

新しいアラート エクスペリエンスを有効にしても、アクション グループの構成やアラート ルールでの通知には影響を与えません。 発生したアラートのインスタンスを Azure Portal で表示および管理する方法が変更されるだけです。

### <a name="smart-groups"></a>スマート グループ
スマート グループは、個々のアラートとしてではなく、関連するアラートを 1 つの単位として管理できるようにすることによってノイズを削減します。 スマート グループの詳細を表示したり、アラートと同様に状態を設定したりできます。 各アラートは、1 つだけのスマート グループのメンバーです。

スマート グループは、1 つの問題を表す関連するアラートを結合するために、機械学習を使用して自動的に作成されます。 アラートが作成されると、このアルゴリズムは履歴パターン、似たプロパティ、似た構造などの情報に基づいて、そのアラートを新しいスマート グループまたは既存のスマート グループに追加します。 

現在、このアルゴリズムでは、サブスクリプション内の同じ監視サービスからのアラートだけが考慮されます。 スマート グループは、この統合によってアラートのノイズの最大 99% を削減できます。 アラートがグループに含まれた理由は、スマート グループの詳細ページで表示できます。

スマート グループの名前は、その最初のアラートの名前です。 スマート グループを作成したり、その名前を変更したりすることはできません。


### <a name="alert-states"></a>アラートの状態
拡張された統合アラートでは、アラートの状態の概念が導入されています。 アラートの状態を設定して、それが解決プロセス内のどこにあるかを指定できます。 アラートが作成されたとき、その状態は *[新規]* になります。 アラートを確認した場合や、アラートを終了した場合は、その状態を変更できます。 状態の変更はすべて、アラートの履歴に格納されます。

次のアラートの状態がサポートされています。

| 州 | 説明 |
|:---|:---|
| 新規 | 問題が検出されたばかりであり、まだレビューされていません。 |
| [Acknowledged] (確認済み) | 管理者がアラートをレビューし、それに対する作業を開始しました。 |
| クローズ | 問題が解決されました。 アラートが終了されされた後、それを再び開いて別の状態に変更できます。 |

アラートの状態は、監視条件とは異なります。 メトリック アラート ルールは、エラーの条件が満たされなくなったら、アラートを_解決済み_の条件に設定できます。 アラートの状態はユーザーによって設定され、監視条件には依存しません。 システムは監視条件を "解決済み" に設定できますが、アラートの状態はユーザーがそれを変更するまで変更されません。

#### <a name="change-the-state-of-an-alert-or-smart-group"></a>アラートまたはスマート グループの状態の変更
個々のアラートの状態を変更するか、またはスマート グループの状態を設定することによって複数のアラートをまとめて管理できます。

アラートの状態を変更するには、アラートの詳細表示で **[アラートの状態の変更]** を選択します。 または、スマート グループの詳細表示で **[スマート グループの状態の変更]** を選択してスマート グループの状態を変更します。 複数の項目の状態を一度に変更するには、最初に一覧ビューで項目を選択した後、ページの上部にある **[状態の変更]** を選択します。 

どちらの場合も、ドロップダウン メニューから新しい状態を選択します。 その後、必要に応じてコメントを入力します。 1 つの項目を変更する場合は、スマート グループ内のすべてのアラートに同じ変更を適用するオプションもあります。

![状態の変更](media/monitoring-overview-unified-alerts/change-tate.png)

### <a name="alerts-page"></a>Alerts page
既定の [アラート] ページには、特定の時間枠内に作成されたアラートの概要が表示されます。 ここには、重大度ごとのアラートの合計が、重大度ごとの各状態にあるアラートの総数を識別する列と共に表示されます。 任意の重大度を選択すると、その重大度でフィルター処理された [[すべてのアラート]](#all-alerts-page) ページが開きます。

![Alerts page](media/monitoring-overview-unified-alerts/alerts-page.png)

ページの上部にあるドロップダウン メニューで値を選択することにより、このビューをフィルター処理できます。

| 列 | 説明 |
|:---|:---|
| サブスクリプション | 最大 5 つの Azure サブスクリプションを選択します。 このビューには、選択されたサブスクリプション内のアラートのみが含まれます。 |
| リソース グループ | 1 つのリソース グループを選択します。 このビューには、選択されたリソース グループ内のターゲットを含むアラートのみが含まれます。 |
| 時間範囲 | このビューには、選択された時間枠内に発生したアラートのみが含まれます。 サポートされる値は、過去 1 時間、過去 24 時間、過去 7 日間、および過去 30 日間です。 |

[アラート] ページの上部にある次の値を選択すると、別のページが開きます。

| 値 | 説明 |
|:---|:---|
| アラート合計数 | 選択された条件に一致するアラートの総数。 この値を選択すると、フィルター処理されていない [すべてのアラート] ビューが開きます。 |
| スマート グループ | 選択された条件に一致する、アラートから作成されたスマート グループの総数。 この値を選択すると、[すべてのアラート] ビューにスマート グループの一覧が開きます。
| アラート ルールの合計 | 選択されたサブスクリプションおよびリソース グループ内のアラート ルールの総数。 この値を選択すると、選択されたサブスクリプションおよびリソース グループに対してフィルター処理された [ルール] ビューが開きます。


### <a name="all-alerts-page"></a>[すべてのアラート] ページ 
[すべてのアラート] ページを使用すると、選択された時間枠内に作成されたアラートの一覧を表示できます。 個々のアラートの一覧、またはそれらのアラートを含むスマート グループの一覧のどちらかを表示できます。 ページの上部にあるバナーを選択すると、ビューが切り替わります。

![[すべてのアラート] ページ](media/monitoring-overview-unified-alerts/all-alerts-page.png)

ページの上部にあるドロップダウン メニュー内の次の値を選択することによって、ビューをフィルター処理できます。

| 列 | 説明 |
|:---|:---|
| サブスクリプション | 最大 5 つの Azure サブスクリプションを選択します。 このビューには、選択されたサブスクリプション内のアラートのみが含まれます。 |
| リソース グループ | 1 つのリソース グループを選択します。 このビューには、選択されたリソース グループ内のターゲットを含むアラートのみが含まれます。 |
| リソースの種類 | 1 つ以上のリソースの種類を選択します。 このビューには、選択された種類のターゲットを含むアラートのみが含まれます。 この列は、リソース グループを指定した後でのみ使用できます。 |
| リソース | リソースを選択します。 このビューには、そのリソースをターゲットとして含むアラートのみが含まれます。 この列は、リソースの種類を指定した後でのみ使用できます。 |
| severity | アラートの重大度を選択するか、または *[すべて]* を選択してすべての重大度のアラートを含めます。 |
| 監視条件 | 監視条件を選択するか、または *[すべて]* を選択してすべての条件のアラートを含めます。 |
| アラートの状態 | アラートの状態を選択するか、または *[すべて]* を選択してすべての状態のアラートを含めます。 |
| サービスの監視 | サービスを選択するか、または *[すべて]* を選択してすべてのサービスを含めます。 そのサービスをターゲットとして使用してルールによって作成されたアラートのみが含まれます。 |
| 時間範囲 | このビューには、選択された時間枠内に発生したアラートのみが含まれます。 サポートされる値は、過去 1 時間、過去 24 時間、過去 7 日間、および過去 30 日間です。 |

表示する列を選択するには、ページの上部にある **[列]** を選択します。 

### <a name="alert-detail-page"></a>アラートの詳細ページ
アラートを選択すると、[アラートの詳細] ページが表示されます。 ここにはアラートの詳細が表示され、その状態を変更できます。

![アラートの詳細](media/monitoring-overview-unified-alerts/alert-detail.png)

[アラートの詳細] ページには、次のセクションが含まれています。

| セクション | 説明 |
|:---|:---|
| 要点 | アラートに関するプロパティやその他の重大な情報を表示します。 |
| 履歴 | アラートによって実行された各アクションと、アラートに加えられたすべての変更を一覧表示します。 これは現在、状態の変更に制限されています。 |
| スマート グループ | そのアラートが含まれているスマート グループに関する情報。 "*アラートの数*" は、そのスマート グループに含まれているアラートの数を示します。 これには、過去 30 日間に作成された、同じスマート グループ内の他のアラートが含まれます。  これは、アラートの一覧ページ内の時間フィルターには関係ありません。 アラートを選択すると、その詳細が表示されます。 |
| 詳細 | 通常はそのアラートを作成したソースの種類に固有である、アラートの詳細なコンテキスト情報を表示します。 |


### <a name="smart-group-detail-page"></a>[スマート グループ 詳細] ページ
スマート グループを選択すると、[スマート グループ 詳細] ページが表示されます。 ここにはスマート グループの詳細 (そのグループを作成するために使用された理由など) が表示され、その状態を変更できます。
 
![スマート グループ詳細](media/monitoring-overview-unified-alerts/smart-group-detail.png)


[スマート グループ 詳細] ページには、次のセクションが含まれています。

| セクション | 説明 |
|:---|:---|
| アラート | そのスマート グループに含まれている個々のアラートを一覧表示します。 アラートを選択すると、[アラートの詳細] ページが開きます。 |
| 履歴 | スマート グループによって実行された各アクションと、それに加えられたすべての変更を一覧表示します。 これは現在、状態の変更とアラート メンバーシップの変更に制限されています。 |

## <a name="next-steps"></a>次の手順
- [新しいアラート エクスペリエンスを使用してアラートを作成、表示、管理する方法を確認する](alert-metric.md)
- [アラート エクスペリエンスのログ アラートについて確認する](monitor-alerts-unified-log.md)
- [アラート エクスペリエンスのメトリック アラートについて確認する](monitoring-near-real-time-metric-alerts.md)
- [アラート エクスペリエンスのアクティビティ ログ アラートについて確認する](alert-activity-log.md)
