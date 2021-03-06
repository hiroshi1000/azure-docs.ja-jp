---
title: Azure Monitor でのログのメトリック アラートの作成
description: 一般的な Log Analytics データでほぼリアルタイムのメトリック アラートを作成する方法に関するチュートリアル。
author: msvijayn
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/17/2018
ms.author: vinagara
ms.component: alerts
ms.openlocfilehash: 55b86e0d23936e47a82aedf213e20eb6dec3d56d
ms.sourcegitcommit: 1f9e1c563245f2a6dcc40ff398d20510dd88fd92
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2018
ms.locfileid: "51621821"
---
# <a name="create-metric-alerts-for-logs-in-azure-monitor"></a>Azure Monitor でのログのメトリック アラートの作成  

## <a name="overview"></a>概要
Azure Monitor では、[クラシック アラート](alert-metric-classic.md)よりも多くの利点を備えた[メトリック アラート](monitoring-near-real-time-metric-alerts.md)をサポートしています。 メトリックは[さまざまな Azure サービスで](monitoring-supported-metrics.md)利用できます。 この記事では、リソース (`Microsoft.OperationalInsights/workspaces`) のサブセットの使用方法について説明します。 

メトリック アラートは、Azure またはオンプレミスのリソースを含むログからのメトリックの一部として、メトリックとして抽出された一般的な Log Analytics ログで使用できます。 サポートされている Log Analytics ソリューションは次のとおりです。
- Windows および Linux マシンの[パフォーマンス カウンター](../log-analytics/log-analytics-data-sources-performance-counters.md)
- [Agent Health のためのハートビート レコード](../monitoring/monitoring-solution-agenthealth.md)
- [更新管理](../automation/automation-update-management.md)レコード
- [イベント データ](../log-analytics/log-analytics-data-sources-windows-events.md) ログ
 
**ログのメトリック アラート**には、Azure のクエリ ベースの[ログ アラート](alert-log.md)よりも多くの利点があります。その一部を次に示します。
- メトリック アラートでは、ほぼリアルタイムの監視機能が提供されます。ログのメトリック アラートは、同じものを確保するためにログ ソースからデータをフォークします。
- メトリック アラートはステートフルです。アラートが発生したときとアラートが解決されたときに、それぞれ一度だけ通知します。アラートの条件が満たされると、間隔ごとに発生し続けるステートレスなログ アラートとは異なります。
- ログのメトリック アラートでは複数のディメンションが提供されるので、コンピューターや OS の種類などの特定の値に簡単にフィルター処理できます。分析でクエリを作成する必要はありません。

> [!NOTE]
> 特定のメトリックやディメンションは、選択された期間内にそのためのデータが存在する場合にのみ表示されます。 Azure Log Analytics ワークスペースを使用するお客様は、これらのメトリックを使用できます。

## <a name="metrics-and-dimensions-supported-for-logs"></a>ログでサポートされるメトリックとディメンション
 メトリック アラートでは、ディメンションを使用するメトリックのアラートがサポートされています。 ディメンションを使用すると、メトリックを適切なレベルにフィルター処理できます。 サポートされているソリューションでの、Log Analytics ワークスペースのログでサポートされているメトリックの一覧については、[こちら](monitoring-supported-metrics.md#microsoftoperationalinsightsworkspaces)をご覧ください。

> [!NOTE]
> [Azure Monitor のメトリック](monitoring-metric-charts.md)を使用して Log Analytics ワークスペースから抽出するためにサポートされているメトリックを表示するには、そのメトリックに対してログのメトリック アラートを作成する必要があります。 ログのメトリック アラートで選択されたディメンションは、Azure Monitor のメトリックを使用して探索する場合にのみ表示されます。

# <a name="creating-metric-alert-for-log-analytics"></a>Log Analytics のメトリック アラートの作成
一般的なログのメトリック データは、Log Analytics で処理される前に Azure Monitor のメトリックにパイプ処理されます。 これにより、ユーザーはメトリック プラットフォームとメトリック アラートの機能 (1 分という低頻度のアラートの使用など) を活用できます。 ログのメトリック アラートを作成する方法を以下に示します。

## <a name="prerequisites-for-metric-alert-for-logs"></a>ログのメトリック アラートの前提条件
Log Analytics データで収集されたログのメトリックを機能させるには、以下を設定し、使用できるようにしておく必要があります。
1. **アクティブな Log Analytics ワークスペース**: 有効かつアクティブな Log Analytics ワークスペースが存在する必要があります。 詳細については、[Azure portal での Log Analytics ワークスペースの作成](../log-analytics/log-analytics-quick-create-workspace.md)に関するページをご覧ください。
2. **Log Analytics ワークスペースのエージェントを構成する**: 前の手順で使用した Log Analytics ワークスペースにデータを送信するために、Azure VM およびオンプレミスの VM でエージェントを構成する必要があります。 詳細については、[Log Analytics エージェントの概要](../monitoring/monitoring-overview-azure-agents.md)に関する記事をご覧ください。
3. **サポートされている Log Analytics ソリューションをインストールする**: Log Analytics ソリューションを構成し、Log Analytics ワークスペースにデータを送信する必要があります。サポートされているソリューションは、[Windows および Linux のパフォーマンス カウンター](../log-analytics/log-analytics-data-sources-performance-counters.md)、[Agent Health のハートビート レコード](../monitoring/monitoring-solution-agenthealth.md)、Update Management、[イベント データ](../log-analytics/log-analytics-data-sources-windows-events.md)です。
4. **ログを送信するように構成された Log Analytics ソリューション**: Log Analytis ソリューションでは、[Log Analytics ワークスペースでサポートされるメトリック](monitoring-supported-metrics.md#microsoftoperationalinsightsworkspaces)に対応するログ/データが必要です。 たとえば、*% Available Memory* の場合、[パフォーマンス カウンター](../log-analytics/log-analytics-data-sources-performance-counters.md) ソリューションでこのメトリックのカウンターを構成しておく必要があります。

## <a name="configuring-metric-alert-for-logs"></a>ログのメトリック アラートの構成
 メトリック アラートは、Azure portal、Resource Manager テンプレート、REST API、PowerShell、Azure CLI を使用して作成および管理できます。 ログのメトリック アラートはメトリック アラートの一種であるため、前提条件が満たされたら、指定した Log Analytics ワークスペースに対してログのメトリック アラートを作成できます。 ペイロード スキーマ、適用されるクォータ制限、課金価格など、[メトリック アラート](monitoring-near-real-time-metric-alerts.md)の特性と機能はすべて、ログのメトリック アラートにも適用されます。

手順の詳細とサンプルについては、[メトリック アラートの作成と管理](https://aka.ms/createmetricalert)に関するページをご覧ください。 具体的には、ログのメトリック アラートでは、メトリック アラートを管理する手順に従います。次の点に注意してください。
- メトリック アラートのターゲットが有効な "*Log Analytics ワークスペース*" であることを確認します。
- 選択した "*Log Analytics ワークスペース*" のメトリック アラート用に選択したシグナルの種類が**メトリック**であることを確認します。
- ディメンション フィルターを使用して、特定の条件またはリソースをフィルター処理します。ログのメトリックは多次元です。
- "*シグナル ロジック*" を構成すると、ディメンション (コンピューターなど) の複数の値にまたがる単一のアラートを作成できます。
- 選択した "*Log Analytics ワークスペース*" のメトリック アラートを作成する際に Azure portal を使用**しない**場合は、まず、[Azure Monitor のスケジュールされたクエリ ルール](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules
)を使用してログ データをメトリックに変換するための明示的なルールを手動で作成する必要があります。

> [!NOTE]
> Azure portal を使用して Log Analytics ワークスペースのメトリック アラートを作成すると、[Azure Monitor のスケジュールされたクエリ ルール](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules)を使用してログ データをメトリックに変換するための対応するルールがバックグラウンドで自動的に作成されます。*ユーザーの介入や操作は不要です*。 Azure portal 以外の方法を使用してログのメトリック アラートを作成する場合は、「[ログのメトリック アラートのリソース テンプレート](#resource-template-for-metric-alerts-for-logs)」をご覧ください。このセクションには、メトリック アラートを作成する前に、ScheduledQueryRule ベースのログからメトリックへの変換ルールを作成する方法のサンプルが示されています。このルールがないと、ログのメトリック アラートを作成するためのデータが存在しなくなります。

## <a name="resource-template-for-metric-alerts-for-logs"></a>ログのメトリック アラートのリソース テンプレート
前述のように、ログからのメトリック アラートの作成は次の 2 段階のプロセスです。
1. scheduledQueryRule API を使用してサポートされているログからメトリックを抽出するためのルールを作成する
2. (手順 1. で) ログから抽出したメトリックのメトリック アラートを作成し、ターゲット リソースとして Log Analytics ワークスペースを作成する

同じことを行うために、次の Azure Resource Manager テンプレートのサンプルを使用できます。メトリック アラートの作成は、scheduledQueryRule を使用してログからメトリックを抽出するためのルールが正常に作成されているかどうかに左右されます。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the rule to convert log to metric"
            }
        },
        "convertRuleDescription": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Description for log converted to metric"
            }
        },
        "convertRuleRegion": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the region used by workspace"
            }
        },
        "convertRuleStatus": {
            "type": "string",
            "defaultValue": "true",
            "metadata": {
                "description": "Specifies whether the log convertion rule is enabled"
            }
        },
        "convertRuleMetric": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric once extraction done from logs."
            }
        },
        "alertName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the alert"
            }
        },
        "alertDescription": {
            "type": "string",
            "defaultValue": "This is a metric alert",
            "metadata": {
                "description": "Description of alert"
            }
        },
        "alertSeverity": {
            "type": "int",
            "defaultValue": 3,
            "allowedValues": [
                0,
                1,
                2,
                3,
                4
            ],
            "metadata": {
                "description": "Severity of alert {0,1,2,3,4}"
            }
        },
        "isEnabled": {
            "type": "bool",
            "defaultValue": true,
            "metadata": {
                "description": "Specifies whether the alert is enabled"
            }
        },
        "resourceId": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Full Resource ID of the resource emitting the metric that will be used for the comparison. For example /subscriptions/00000000-0000-0000-0000-0000-00000000/resourceGroups/ResourceGroupName/providers/Microsoft.compute/virtualMachines/VM_xyz"
            }
        },
        "metricName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric used in the comparison to activate the alert."
            }
        },
        "operator": {
            "type": "string",
            "defaultValue": "GreaterThan",
            "allowedValues": [
                "Equals",
                "NotEquals",
                "GreaterThan",
                "GreaterThanOrEqual",
                "LessThan",
                "LessThanOrEqual"
            ],
            "metadata": {
                "description": "Operator comparing the current value with the threshold value."
            }
        },
        "threshold": {
            "type": "string",
            "defaultValue": "0",
            "metadata": {
                "description": "The threshold value at which the alert is activated."
            }
        },
        "timeAggregation": {
            "type": "string",
            "defaultValue": "Average",
            "allowedValues": [
                "Average",
                "Minimum",
                "Maximum",
                "Total"
            ],
            "metadata": {
                "description": "How the data that is collected should be combined over time."
            }
        },
        "windowSize": {
            "type": "string",
            "defaultValue": "PT5M",
            "metadata": {
                "description": "Period of time used to monitor alert activity based on the threshold. Must be between five minutes and one day. ISO 8601 duration format."
            }
        },
        "evaluationFrequency": {
            "type": "string",
            "defaultValue": "PT1M",
            "metadata": {
                "description": "how often the metric alert is evaluated represented in ISO 8601 duration format"
            }
        },
        "actionGroupId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The ID of the action group that is triggered when the alert is activated or deactivated"
            }
        }
    },
    "variables": {
        "convertRuleTag": "hidden-link:/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName",
        "convertRuleSourceWorkspace": {
            "SourceId": "/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        }
    },
    "resources": [
        {
            "name": "[parameters('convertRuleName')]",
            "type": "Microsoft.Insights/scheduledQueryRules",
            "apiVersion": "2018-04-16",
            "location": "[parameters('convertRuleRegion')]",
            "tags": {
                "[variables('convertRuleTag')]": "Resource"
            },
            "properties": {
                "description": "[parameters('convertRuleDescription')]",
                "enabled": "[parameters('convertRuleStatus')]",
                "source": {
                    "dataSourceId": "[variables('convertRuleSourceWorkspace').SourceId]"
                },
                "action": {
                    "odata.type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.Microsoft.AppInsights.Nexus.DataContracts.Resources.ScheduledQueryRules.LogToMetricAction",
                    "criteria": [{
                            "metricName": "[parameters('convertRuleMetric')]",
                            "dimensions": []
                        }
                    ]
                }
            }
        },
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "dependsOn":["[resourceId('Microsoft.Insights/scheduledQueryRules',parameters('convertRuleName'))]"],
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": ["[parameters('resourceId')]"],
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria",
                    "allOf": [
                        {
                            "name" : "1st criterion",
                            "metricName": "[parameters('metricName')]",
                            "dimensions":[],   
                            "operator": "[parameters('operator')]",
                            "threshold" : "[parameters('threshold')]",
                            "timeAggregation": "[parameters('timeAggregation')]"
                        }
                    ]
                },
                "actions": [
                    {
                        "actionGroupId": "[parameters('actionGroupId')]"                
                    }
                ]
            }
        }
    ]
}

```
上記の JSON が metricfromLogsAlert.json として保存されていれば、リソース テンプレート ベースの作成用のパラメーター JSON ファイルと組み合わせることができます。 サンプルのパラメーター JSON ファイルを次に示します。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "value": "TestLogtoMetricRule" 
        },
        "convertRuleDescription": {
            "value": "Test rule to extract metrics from logs via template"
        },
        "convertRuleRegion": {
            "value": "West Central US"
        },
        "convertRuleStatus": {
            "value": "true"
        },
        "convertRuleMetric": {
            "value": "Average_% Idle Time"
        },
        "alertName": {
            "value": "TestMetricAlertonLog"
        },
        "alertDescription": {
            "value": "New multi-dimensional metric alert created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "resourceId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        },
        "metricName":{
            "value": "Average_% Idle Time"
        },
        "operator": {
            "value": "GreaterThan" 
        },
        "threshold":{
            "value": "1"
        },
        "timeAggregation":{
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/microsoft.insights/actionGroups/actionGroupName"
        }
    }    
}
```
上記のパラメーター ファイルが metricfromLogsAlert.parameters.json として保存されている場合、[Azure portal での作成用のリソース テンプレート](../azure-resource-manager/resource-group-template-deploy-portal.md)を使用してログのメトリック アラートを作成できます。 

次の Azure PowerShell コマンドを使用することもできます。
```PowerShell
New-AzureRmResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile metricfromLogsAlert.json TemplateParameterFile metricfromLogsAlert.parameters.json
```

または、Azure CLI を使用してリソース テンプレートをデプロイします。
```CLI
az group deployment create --resource-group myRG --template-file metricfromLogsAlert.json --parameters @metricfromLogsAlert.parameters.json
```

## <a name="next-steps"></a>次の手順

* [メトリック アラート](https://aka.ms/createmetricalert)の詳細を確認します。
* [Azure でのログ アラート](monitor-alerts-unified-log.md)について学習します。
* [Azure のアラート](monitoring-overview-alerts.md)について確認します。
