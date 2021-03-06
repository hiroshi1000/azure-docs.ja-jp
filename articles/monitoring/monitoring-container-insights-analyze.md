---
title: コンテナーの Azure Monitor を使用して AKS クラスターのパフォーマンスを監視する (プレビュー) | Microsoft Docs
description: この記事では、コンテナーの Azure Monitor を使用してパフォーマンスとログ データを表示および分析する方法について説明します。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/19/2018
ms.author: magoedte
ms.openlocfilehash: daec3d6e6cd8e4df3fdfe45fbb8ee98966c8a38e
ms.sourcegitcommit: 6e09760197a91be564ad60ffd3d6f48a241e083b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2018
ms.locfileid: "50214156"
---
# <a name="understand-aks-cluster-performance-with-azure-monitor-for-containers-preview"></a>コンテナーの Azure Monitor を使用して AKS クラスターのパフォーマンスを把握する (プレビュー)
Azure Kubernetes Service (AKS) クラスターのパフォーマンスは、コンテナーの Azure Monitor を使用して、AKS クラスターから直接観察するか、またはサブスクリプションのすべての AKS クラスターをまとめて観察することができます。 

この記事を読むと、2 つの観点のエクスペリエンスと、検出された問題をすばやく評価、調査、解決する方法を理解できます。

コンテナーの Azure Monitor を有効にする方法については、「[コンテナー用 Azure Monitor の利用を開始する方法](monitoring-container-insights-onboard.md)」をご覧ください。

Azure Monitor では、サブスクリプション内のリソース グループに展開されているすべての監視対象 AKS クラスターと、ソリューションによって監視されていない AKS クラスターの正常性状態を表示する、複数クラスター ビューを提供します。  すぐにクラスターの正常性を理解することができ、ここからは、ノードとコントローラーのパフォーマンス ページにドリルダウンしたり、クラスターのパフォーマンス グラフを表示したりできます。  監視対象外として検出および識別された AKS クラスターについては、いつでも監視対象にできます。  

## <a name="sign-in-to-the-azure-portal"></a>Azure ポータルにサインインします。
[Azure Portal](https://portal.azure.com) にサインインします。 

## <a name="multi-cluster-view-from-azure-monitor"></a>Azure Monitor の複数クラスター ビュー 
展開されているすべての AKS クラスターの正常性状態を表示するには、Azure portal で左側のウィンドウから **[モニター]** を選択します。  **[分析情報]** セクションで **[コンテナー (プレビュー)]** を選択します。  

![Azure Monitor の複数クラスター ダッシュボードの例](./media/monitoring-container-insights-analyze/azmon-containers-multiview.png)

**[Monitored clusters]\(監視対象クラスター\)** タブでは次のことがわかります。

1. 重大または異常な状態のクラスターの数と、正常なクラスターまたは報告していないクラスター (不明状態と呼ばれます) の数。
2. クラスターごとに展開されているノード、ユーザー ポッド、システム ポッドの数。  

次の正常性状態が定義されています。 

1. **正常** – VM では問題は検出されておらず、要求されたとおりに機能しています。 
2. **重大** – 1 つまたは複数の重大な問題が検出されており、意図される正常な動作状態に戻すには対処する必要があります。
3. **警告** – 1 つまたは複数の問題が検出されており、対処する必要があります。対処しないと、正常性状態が重大になる可能性があります。
4. **不明** – サービスがノードまたはポッドに接続できなかった場合、状態は不明に変わります。

クラスター全体的の状態は、3 つの状態のうち "*最悪のもの*" として計算されます。ただし、いずれかの状態が "*不明*" の場合は、クラスター全体の状態も**不明**と表示されます。  

次の表では、複数クラスター ビューでの監視対象クラスターの正常性状態を制御する計算の詳細を示します。

| |Status |可用性 |  
|-------|-------|-----------------|  
|**ユーザー ポッド**| | |  
| |Healthy |100% |  
| |警告 |90 ～ 99% |  
| |重大 |< 90% |  
| |Unknown |過去 30 分以内に報告していない場合 |  
|**システム ポッド**| | |  
| |Healthy |100% |
| |警告 |該当なし |
| |重大 |< 100% |
| |Unknown |過去 30 分以内に報告していない場合 |
|**Node** | | |
| |Healthy |> 85% |
| |警告 |60 ～ 84% |
| |重大 |< 60% |
| |Unknown |過去 30 分以内に報告していない場合 |

クラスターの一覧からは、クラスターの名前をクリックして **[クラスター]** ページに、特定のクラスターの **[Nodes]\(ノード\)** 列のノードのロールアップをクリックして **[ノード]** パフォーマンス ページに、あるいは **[User pods]\(ユーザー ポッド\)** 列または **[System pods]\(システム ポッド\)** 列のロールアップをクリックして **[コントローラー]** パフォーマンス ページに、ドリルダウンできます。   

## <a name="view-performance-directly-from-an-aks-cluster"></a>AKS クラスターから直接、パフォーマンスを確認する
コンテナーの Azure Monitor へは、左側のパネルの **[Insights (プレビュー)]** を選択することにより、AKS クラスターから直接アクセスできます。 AKS クラスターに関する情報は 4 つの観点から確認できます。

- クラスター
- Nodes 
- コントローラー  
- Containers

**[Insights (プレビュー)]** をクリックすると開く既定のページは **[クラスター]** であり、クラスターの主要なパフォーマンス メトリックを表示する 4 つのパフォーマンス折れ線グラフが含まれます。 

![[クラスター] タブのパフォーマンス グラフの例](./media/monitoring-container-insights-analyze/containers-cluster-perfview.png)

パフォーマンス グラフには、次の 4 つのパフォーマンス メトリックが表示されます。

- **ノードの CPU 使用率&nbsp;%**: クラスター全体の CPU 使用率の集計されたパースペクティブ。 時間範囲ごとに結果をフィルター処理するには、グラフの上のパーセンタイル セレクターで "**Avg**"、"**Min**"、"**Max**"、"**50th**"、"**90th**"、"**95th**" を個別にまたは組み合わせて選択します。 
- **ノードのメモリ使用率&nbsp;%**: クラスター全体のメモリ使用率の集計されたパースペクティブ。 時間範囲ごとに結果をフィルター処理するには、グラフの上のパーセンタイル セレクターで "**Avg**"、"**Min**"、"**Max**"、"**50th**"、"**90th**"、"**95th**" を個別にまたは組み合わせて選択します。 
- **ノード数**: Kubernetes からのノードの数と状態。 表されるクラスター ノードの状態には、*[すべて]*、*[準備完了]*、*[準備未完了]* があり、グラフの上のセレクターで個別にまたは組み合わせてフィルター処理できます。 
- **アクティビティ ポッド数**: Kubernetes からのポッドの数と状態。 表されるポッドの状態には、*[すべて]*、*[保留中]*、*[実行中]*、*[不明]* があり、グラフの上のセレクターで個別にまたは組み合わせてフィルター処理できます。 

**[ノード]**、**[コントローラー]**、および **[コンテナー]** タブに切り替えると、ページの右側に自動的にプロパティ ウィンドウが表示されます。  ここには選択された項目のプロパティが示され、Kubernetes オブジェクトを整理するために定義するラベルが含まれます。 ウィンドウの表示と非表示を切り替えるには、ウィンドウの **>>** リンクをクリックします。  

![Kubernetes パースペクティブ プロパティ ウィンドウの例](./media/monitoring-container-insights-analyze/perspectives-preview-pane-01.png)

階層内のオブジェクトを展開すると、選択されたオブジェクトに基づいて、プロパティ ウィンドウが更新されます。 ウィンドウから、定義済みのログ検索を使用して Kubernetes イベントを表示することもできます。その場合は、ウィンドウの上部にある **[View Kubernetes event logs]\(Kubernetes イベント ログの表示\)** リンクをクリックします。 Kubernetes ログ データの表示の詳細については、「[データを分析するためのログの検索](#search-logs-to-analyze-data)」を参照してください。

ページ上部の **[+ フィルターの追加]** オプションを使用して、**[サービス]**、**[ノード]**、または **[名前空間]** でビューの結果をフィルター処理できます。また、フィルター スコープを選択した後は、**[値の選択]** フィールドに表示される値のいずれかを選択できます。  構成したフィルターは、AKS クラスターのいずれかの観点を表示するときにグローバルに適用されます。  数式は、等号のみがサポートされています。  最初のフィルターの上に新しいフィルターを追加して、結果をさらに絞り込むことができます。  たとえば、**ノード**によるフィルターを指定した場合、2 番目のフィルターとしては**サービス**または**名前空間**だけを選択できます。  

![フィルターを使用した結果の絞り込みの例](./media/monitoring-container-insights-analyze/add-filter-option-01.png)

あるタブで指定したフィルターは別のタブを選択しても引き続き適用され、指定されているフィルターの横にある **[x]** 記号をクリックすると削除されます。   

**[ノード]** タブに切り替えます。行階層では、クラスター内のノードから始まり、Kubernetes オブジェクト モデルに従います。 ノードを展開し、ノードで実行されている 1 つまたは複数のポッドを表示することができます。 複数のコンテナーがポッドにグループ化されている場合、階層内の最後の行として表示されます。 ホストでプロセッサまたはメモリが不足している場合、ホスト上でどのくらいの数のポッドに関連しないワークロードが実行されているかを確認することもできます。

![パフォーマンス ビューの Kubernetes ノード階層の例](./media/monitoring-container-insights-analyze/containers-nodes-view.png)

展開されたノードでは、ノード上で実行されているポッドまたはコンテナーからコントローラーにドリルダウンして、そのコントローラーでフィルター処理されたパフォーマンス データを見ることができます。 特定のノードの **[コントローラー]** 列の値をクリックします。   

![パフォーマンス ビューでのノードからコントローラーへのドリルダウンの例](./media/monitoring-container-insights-analyze/drill-down-node-controller.png)

ページ上部のコントローラーまたはコンテナーを選択し、それらのオブジェクトの状態やリソース使用率を確認できます。  また、**[メトリック]** ドロップダウン リストで **[Memory RSS]\(使用メモリ (RSS)\)** または **[メモリ ワーキング セット]** を選択すると、メモリ使用率を確認することもできます。 **[Memory RSS]\(使用メモリ (RSS)\)** は、Kubernetes 1.8 以降でのみサポートされています。 それ以外のバージョンでは、**Min&nbsp;%** の値が、未定義または表示できない値を示す数値データ型である、*NaN&nbsp;%* として示されます。 

![コンテナー ノード パフォーマンス ビュー](./media/monitoring-container-insights-analyze/containers-node-metric-dropdown.png)

既定では、パフォーマンス データは、過去 6 時間のものですが、左上にある **[時間範囲]** オプションを使用して時間枠を変更できます。 パーセンタイル セレクターで "**Avg**"、"**Min**"、"**Max**"、"**50th**"、"**90th**"、"**95th**" を選択して、時間範囲内の結果をフィルター処理することもできます。 

![パーセンタイル選択によるデータのフィルター処理](./media/monitoring-container-insights-analyze/containers-metric-percentile-filter.png)

次の例では、*aks-nodepool1-* ノードの **[コンテナー]** の値は 14 で、これは展開されているコンテナーの合計数のロールアップです。

![ノードあたりのコンテナーのロールアップの例](./media/monitoring-container-insights-analyze/containers-nodes-containerstotal.png)

これにより、クラスター内のノード間のコンテナーのバランスが適切かどうかをすばやく識別できます。 

次の表では、ノードを表示した場合に示される情報について説明します。

| 分割 | 説明 | 
|--------|-------------|
| Name | ホストの名前。 |
| Status | ノードの状態の Kubernetes ビュー。 |
| Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 選択した期間中の、パーセンタイルに基づいたノードの平均率。 |
| Avg、Min、Max、50th、90th | 選択した期間中の、パーセンタイルに基づいたノードの実際の平均値。 平均値は、ノードに設定されている CPU/メモリの制限から測定されます。ポッドおよびコンテナーについては、ホストによって報告された平均値です。 |
| Containers | コンテナーの数。 |
| Uptime | ノードが起動または再起動されてから経過した時間を示します。 |
| コントローラー | コンテナーとポッド限定。 どのコントローラーに存在しているかが示されます。 コントローラーにないポッドもあるため、一部は **N/A** と表示される可能性があります。 | 
| Trend Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | コントローラーのパーセンタイル メトリック率を表す棒グラフ トレンド。 |

セレクターで、**[コントローラー]** を選択します。

![コントローラーの選択ビュー](./media/monitoring-container-insights-analyze/containers-controllers-tab.png)

ここで、コントローラーのパフォーマンスの正常性を表示できます。

![<名前> コントローラーのパフォーマンス ビュー](./media/monitoring-container-insights-analyze/containers-controllers-view.png)

行階層はコントローラーから開始し、コントローラーを展開すると、1 つまたは複数のポッドが表示されます。  ポッドを展開すると、ポッドにグループ化されているコンテナーが最後の行に表示されます。 展開されたコントローラーからは、それが実行されているノードにドリルダウンして、そのノードでフィルター処理されたパフォーマンス データを見ることができます。 特定のコントローラーの **[ノード]** 列の値をクリックします。   

![パフォーマンス ビューでのノードからコントローラーへのドリルダウンの例](./media/monitoring-container-insights-analyze/drill-down-controller-node.png)

次の表では、コントローラーを表示した場合に示される情報について説明します。

| 分割 | 説明 | 
|--------|-------------|
| Name | コントローラーの名前。|
| Status | *[OK]*、*[終了]*、*[失敗]*、*[停止]*、*[一時停止]* などの、実行完了時のコンテナーのロールアップ状態です。 コンテナーが実行されているのに状態が正しく表示されない、またはエージェントが状態を認識せず、コンテナーが 30 分を超えても応答しない場合は、*[不明]* 状態になります。 状態アイコンのその他の詳細については、以下の表を参照してください。|
| Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 各エンティティの選択されたメトリックとパーセンタイルの平均率のロールアップ平均。 |
| Avg、Min、Max、50th、90th  | 選択されたパーセンタイルのコンテナーの平均 CPU ミリコアまたはメモリ パフォーマンスのロールアップ。 平均値は、ポッドに設定されている CPU/メモリ制限から測定されます。 |
| Containers | コントローラーまたはポッドのコンテナーの合計数。 |
| Restarts | コンテナーの再起動数のロールアップ。 |
| Uptime | コンテナーが起動されてからの経過時間を表します。 |
| ノード | コンテナーとポッド限定。 これには存在しているコントローラーが表示されます。 | 
| Trend Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;%| コントローラーのパーセンタイル メトリックを表す棒グラフ トレンドです。 |

状態フィールドのアイコンは、コンテナーのオンライン状態を示します。
 
| アイコン | Status | 
|--------|-------------|
| ![実行中 (準備完了) 状態アイコン](./media/monitoring-container-insights-analyze/containers-ready-icon.png) | 実行中 (準備完了)|
| ![待機中または一時停止状態アイコン](./media/monitoring-container-insights-analyze/containers-waiting-icon.png) | 待機中または一時停止|
| ![最後に報告された実行中の状態アイコン](./media/monitoring-container-insights-analyze/containers-grey-icon.png) | 最後に報告された実行中。ただし、30 分以上に応答していない|
| ![正常な状態アイコン](./media/monitoring-container-insights-analyze/containers-green-icon.png) | 正常に停止したか、停止に失敗した|

状態アイコンには、ポッドでの指定内容に基づいた数が示されます。 最も良くない 2 つの状態を示し、状態の上にマウス ポインターを移動すると、コンテナー内のすべてのポッドのロールアップ状態を示します。 準備完了の状態がない場合、状態の値は **(0)** と表示されます。 

セレクターで、**[コンテナー]** を選択します。

![コンテナーの選択ビュー](./media/monitoring-container-insights-analyze/containers-containers-tab.png)

ここで、Azure Kubernetes コンテナーのパフォーマンスの正常性を表示できます。  

![<名前> コントローラーのパフォーマンス ビュー](./media/monitoring-container-insights-analyze/containers-containers-view.png)

コンテナーからは、ポッドまたはノードにドリルダウンして、そのオブジェクトでフィルター処理されたパフォーマンス データを見ることができます。 特定のコンテナーの **[ポッド]** または **[ノード]** 列の値をクリックします。   

![パフォーマンス ビューでのノードからコントローラーへのドリルダウンの例](./media/monitoring-container-insights-analyze/drill-down-controller-node.png)

次の表では、コンテナーを表示した場合に示される情報について説明します。

| 分割 | 説明 | 
|--------|-------------|
| Name | コントローラーの名前。|
| Status | 存在する場合、コンテナーの状態です。 状態アイコンのその他の詳細については、次の表を参照してください。|
| Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 各エンティティの選択されたメトリックとパーセンタイルの平均率のロールアップ。 |
| Avg、Min、Max、50th、90th  | 選択されたパーセンタイルのコンテナーの平均 CPU ミリコアまたはメモリ パフォーマンスのロールアップ。 平均値は、ポッドに設定されている CPU/メモリ制限から測定されます。 |
| Pod | ポッドが存在するコンテナーです。| 
| ノード |  コンテナーが存在するノードです。 | 
| Restarts | コンテナーが起動されてからの経過時間を表します。 |
| Uptime | コンテナーが起動または再起動されてからの経過時間を表します。 |
| Trend Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | コンテナーのメトリックの平均率を表す棒グラフ トレンド。 |

状態フィールドのアイコンは、次の表の説明とのとおり、ポッドのオンライン状態を示します。
 
| アイコン | Status |  
|--------|-------------|  
| ![実行中 (準備完了) 状態アイコン](./media/monitoring-container-insights-analyze/containers-ready-icon.png) | 実行中 (準備完了)|  
| ![待機中または一時停止状態アイコン](./media/monitoring-container-insights-analyze/containers-waiting-icon.png) | 待機中または一時停止|  
| ![最後に報告された実行中の状態アイコン](./media/monitoring-container-insights-analyze/containers-grey-icon.png) | 最後に報告された実行中。ただし、30 分を超えても応答しない|  
| ![終了状態アイコン](./media/monitoring-container-insights-analyze/containers-terminated-icon.png) | 正常に停止したか、停止に失敗した|  
| ![失敗状態アイコン](./media/monitoring-container-insights-analyze/containers-failed-icon.png) | 失敗の状態 |  

## <a name="container-data-collection-details"></a>コンテナーのデータ収集の詳細
コンテナーの洞察により、コンテナー ホストとコンテナーからさまざまなパフォーマンス メトリックとログ データが収集されます。 データは、3 分ごとに収集されます。

### <a name="container-records"></a>コンテナー レコード

次の表には、コンテナーの Azure Monitor によって収集されるレコードと、ログ検索結果に表示されるデータ型の例を示します。

| データ型 | ログ検索のデータ型 | フィールド |
| --- | --- | --- |
| ホストとコンテナーのパフォーマンス | `Perf` | Computer、ObjectName、CounterName &#40;%Processor Time、Disk Reads MB、Disk Writes MB、Memory Usage MB、Network Receive Bytes、Network Send Bytes、Processor Usage sec、Network&#41;、CounterValue、TimeGenerated、CounterPath、SourceSystem |
| コンテナー インベントリ | `ContainerInventory` | TimeGenerated、Computer、container name、ContainerHostname、Image、ImageTag、ContainerState、ExitCode、EnvironmentVar、Command、CreatedTime、StartedTime、FinishedTime、SourceSystem、ContainerID、ImageID |
| コンテナー イメージ インベントリ | `ContainerImageInventory` | TimeGenerated、Computer、Image、ImageTag、ImageSize、VirtualSize、Running、Paused、Stopped、Failed、SourceSystem、ImageID、TotalContainer |
| コンテナー ログ | `ContainerLog` | TimeGenerated、Computer、image ID、container name、LogEntrySource、LogEntry、SourceSystem、ContainerID |
| コンテナー サービス ログ | `ContainerServiceLog`  | TimeGenerated、Computer、TimeOfCommand、Image、Command、SourceSystem、ContainerID |
| コンテナー ノード インベントリ | `ContainerNodeInventory_CL`| TimeGenerated、Computer、ClassName_s、DockerVersion_s、OperatingSystem_s、Volume_s、Network_s、NodeRole_s、OrchestratorType_s、InstanceID_g、SourceSystem|
| コンテナー プロセス | `ContainerProcess_CL` | TimeGenerated、Computer、Pod_s、Namespace_s、ClassName_s、InstanceID_s、Uid_s、PID_s、PPID_s、C_s、STIME_s、Tty_s、TIME_s、Cmd_s、Id_s、Name_s、SourceSystem |
| Kubernetes クラスター内のポッドのインベントリ | `KubePodInventory` | TimeGenerated、Computer、ClusterId、ContainerCreationTimeStamp、PodUid、PodCreationTimeStamp、ContainerRestartCount、PodRestartCount、PodStartTime、ContainerStartTime、ServiceName、ControllerKind、ControllerName、ContainerStatus、ContainerID、ContainerName、Name、PodLabel、Namespace、PodStatus、ClusterName、PodIp、SourceSystem |
| Kubernetes クラスター内のノード部分のインベントリ | `KubeNodeInventory` | TimeGenerated, Computer, ClusterName, ClusterId, LastTransitionTimeReady, Labels, Status, KubeletVersion, KubeProxyVersion, CreationTimeStamp, SourceSystem | 
| Kubernetes イベント | `KubeEvents_CL` | TimeGenerated, Computer, ClusterId_s, FirstSeen_t, LastSeen_t, Count_d, ObjectKind_s, Namespace_s, Name_s, Reason_s, Type_s, TimeGenerated_s, SourceComponent_s, ClusterName_s, Message,  SourceSystem | 
| Kubernetes クラスター内のサービス | `KubeServices_CL` | TimeGenerated, ServiceName_s, Namespace_s, SelectorLabels_s, ClusterId_s, ClusterName_s, ClusterIP_s, ServiceType_s, SourceSystem | 
| Kubernetes クラスターのノード部分のパフォーマンス メトリック | Perf &#124; where ObjectName == “K8SNode” | Computer、ObjectName、CounterName &#40;cpuUsageNanoCores、memoryWorkingSetBytes、memoryRssBytes、networkRxBytes、networkTxBytes、restartTimeEpoch、networkRxBytesPerSec、networkTxBytesPerSec、cpuAllocatableNanoCores、memoryAllocatableBytes、cpuCapacityNanoCores、memoryCapacityBytes&#41;、CounterValue、TimeGenerated、CounterPath、SourceSystem | 
| Kubernetes クラスターのコンテナー部分のパフォーマンス メトリック | Perf &#124; where ObjectName == “K8SContainer” | CounterName &#40;cpuUsageNanoCores、memoryWorkingSetBytes、memoryRssBytes、restartTimeEpoch、cpuRequestNanoCores、memoryRequestBytes、cpuLimitNanoCores、memoryLimitBytes&#41;、CounterValue、TimeGenerated、CounterPath、SourceSystem | 

## <a name="search-logs-to-analyze-data"></a>データを分析するためのログの検索
Log Analytics を使用することにより、傾向を特定し、ボトルネックを診断、予想したり、データを関連付けて現在のクラスター構成のパフォーマンスが最適化されているかを判断したりできます。 すぐに使用できる事前定義のログ検索が提供されています。また、検索結果として返される情報の表示方法をカスタマイズすることもできます。 

プレビュー ウィンドウで **[View Kubernetes event logs]\(Kubernetes イベント ログの表示\)** または **[コンテナー ログの表示]** オプションを選択することにより、ワークスペース内のデータを対話式に分析できます。 **[ログ検索]** ページは、元の Azure portal ページの右側に表示されます。

![Log Analytics でデータを解析する](./media/monitoring-container-insights-analyze/container-health-log-search-example.png)   

Log Analytics に転送されるコンテナーのログ出力は STDOUT および STDERR です。 Azure Monitor では、Azure マネージド Kubernetes (AKS) が監視され、大量のデータが生成されるため、現時点では Kube システムのデータは収集されません。 

### <a name="example-log-search-queries"></a>検索クエリの例
多くの場合、1、2 個の例を使ってクエリを作成し、その後、要件に合わせて変更するとうまくいきます。 より高度なクエリを作成できるように、次のサンプル クエリを試すことができます。

| Query | 説明 | 
|-------|-------------|
| ContainerInventory<br> &#124; project Computer, Name, Image, ImageTag, ContainerState, CreatedTime, StartedTime, FinishedTime<br> &#124; render table | コンテナーのライフ サイクル情報をすべて一覧表示します| 
| KubeEvents_CL<br> &#124; where not(isempty(Namespace_s))<br> &#124; sort by TimeGenerated desc<br> &#124; render table | Kubernetes イベント|
| ContainerImageInventory<br> &#124; summarize AggregatedValue = count() by Image, ImageTag, Running | イメージ インベントリ | 
| **高度な分析では、折れ線グラフを選択します**:<br> Perf<br> &#124; where ObjectName == "Container" and CounterName == "% Processor Time"<br> &#124; summarize AvgCPUPercent = avg(CounterValue) by bin(TimeGenerated, 30m), InstanceName | コンテナー CPU | 
| **高度な分析では、折れ線グラフを選択します**:<br> Perf &#124; where ObjectName == "Container" and CounterName == "Memory Usage MB"<br> &#124; summarize AvgUsedMemory = avg(CounterValue) by bin(TimeGenerated, 30m), InstanceName | コンテナー メモリ |
