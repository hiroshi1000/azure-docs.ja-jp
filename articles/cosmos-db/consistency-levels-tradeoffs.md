---
title: Azure Cosmos DB のさまざまな整合性レベルでの可用性およびパフォーマンスのトレードオフ | Microsoft Docs
description: Azure Cosmos DB のさまざまな整合性レベルでの可用性およびパフォーマンスのトレードオフ。
keywords: 整合性, パフォーマンス, azure cosmos db, azure, Microsoft azure
services: cosmos-db
author: markjbrown
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 10/20/2018
ms.author: mjbrown
ms.openlocfilehash: 0e4105d6f56a8eb45a83e970c85319cf25041781
ms.sourcegitcommit: 5a1d601f01444be7d9f405df18c57be0316a1c79
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/10/2018
ms.locfileid: "51514776"
---
# <a name="availability-and-performance-tradeoffs-for-various-consistency-levels-in-azure-cosmos-db"></a>Azure Cosmos DB のさまざまな整合性レベルでの可用性およびパフォーマンスのトレードオフ

分散型データベースでは高可用性もしくは低遅延、またはその両方がレプリケーションに依存するため、読み取りの整合性と、可用性、待機時間、およびスループットとの間に基本的なトレードオフが生じます。 Azure Cosmos DB では、"厳密" と "最終的" な整合性という両極端ではなく、幅広い選択肢を利用できるデータ整合性に対応しています。 Cosmos DB により開発者は、整合性の程度 (強い順) に応じて明確に定義された 5 種類の整合性モデルの中から選択することができます。具体的には、**厳密**、**有界整合性制約**、**セッション**、**一貫性のあるプレフィックス**、**最終的**の 5 つが用意されています。 この 5 つの整合性モデルはそれぞれ、可用性およびパフォーマンスのトレードオフを提供し、包括的な SLA によってサポートされます。

## <a name="consistency-levels-and-latency"></a>整合性レベルと待機時間

- すべての整合性レベルの**読み取り待機時間**は 99 パーセンタイルで 10 ミリ秒未満となるように常に保証され、SLA でサポートされます。 読み取り待機時間の平均 (50 パーセンタイルでの) は、通常 2 ミリ秒未満です。

- いくつかのリージョンにまたがっていて厳密整合性で構成されている Cosmos アカウントの場合を除き、他の整合性レベルにおける**書き込み待機時間**は 99 パーセンタイルで 10 ミリ秒未満となるように常に保証されます。 この書き込み待機時間は、SLA によってサポートされます。 書き込み待機時間の平均 (50 パーセンタイルでの) は、通常 5 ミリ秒未満です。

- 厳密整合性で構成されたリージョンがいくつか存在する Cosmos アカウント (現在、プレビューの段階) の場合、**書き込み待機時間**は 99 パーセンタイルで (2 * RTT) + 10 ミリ秒未満となるように保証されています (RTT はラウンドトリップ時間)。 最も遠く離れた任意の 2 つのリージョン間の RTT が Cosmos アカウントに関連付けられます。 正確な RTT 待機時間は、光の速さで通信する距離と正確な Azure ネットワーク トポロジによって決まります。 Azure ネットワークでは、任意の 2 つの Azure リージョン間の RTT に対して待機時間の SLA は提供されません。 Cosmos DB レプリケーションの待機時間は、ご利用の Cosmos アカウントの Azure portal に表示されます。これにより、その Cosmos アカウントに関連付けられたさまざまなリージョン間でのレプリケーション待機時間を監視することができます。

## <a name="consistency-levels-and-throughput"></a>整合性レベルとスループット

- 要求ユニットの数が同じである場合、"セッション"、"一貫性のあるプレフィックス"、"最終的" の整合性レベルでは、"厳密" および "有界整合性制約" の場合と比べてほぼ 2 倍の読み取りスループットを発揮します。

- 書き込み操作の種類 (挿入、置換、アップサート、削除など) が指定された場合、要求ユニットに対するスループットはすべての整合性レベルで同じです。

## <a name="next-steps"></a>次の手順

次に、以下の記事を使用して分散システムでのグローバル分散および一般的な整合性のトレードオフについて学習できます。

* [最新の分散データベース システム設計における整合性のトレードオフ](https://www.computer.org/web/csdl/index/-/csdl/mags/co/2012/02/mco2012020037-abs.html)
* [高可用性](high-availability.md)
* [Cosmos DB SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_2/)
