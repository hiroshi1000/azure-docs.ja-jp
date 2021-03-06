---
title: Avere vFXT システムの計画 - Azure
description: Avere vFXT for Azure をデプロイする前に行う計画について説明します
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 10/31/2018
ms.author: v-erkell
ms.openlocfilehash: f0e5523565dc561ed457dbc340835ad1889cb876
ms.sourcegitcommit: 6135cd9a0dae9755c5ec33b8201ba3e0d5f7b5a1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/31/2018
ms.locfileid: "50669869"
---
# <a name="plan-your-avere-vfxt-system"></a>Avere vFXT システムの計画

この記事では、確実に作成したクラスターが配置され、ニーズに適したサイズになるように、新しい Avere vFXT for Azure クラスターを計画する方法について説明します。 

Azure Marketplace への移動、または任意の VM の作成を行う前に、クラスターでどのように Azure 内の他の要素を操作するかを検討してください。 クラスター リソースがご利用のプライベート ネットワークとサブネットに配置される場所を計画し、ご利用のバックエンド ストレージを配置する場所を決定します。 作成したクラスター ノードが、自分のワークフローをサポートするために、十分に強力であることを確認します。 

詳細については、後の説明を参照してください。

## <a name="resource-group-and-network-infrastructure"></a>リソース グループとネットワーク インフラストラクチャ

ご利用の Avere vFXT for Azure のデプロイの要素を配置する場所を検討してください。 以下の図は、Avere vFXT for Azure コンポーネントに対して可能な配置を示しています。

![1 つのサブネット内のクラスター コントローラーとクラスター VM を示す図。 サブネット境界の周囲は、VNet の境界です。 VNet の内側にはストレージ サービス エンドポイントを表す六角形があり、VNet の外側の BLOB ストレージに破線の矢印で接続されています。](media/avere-vfxt-components-option.png)

Avere vFXT システムのネットワーク インフラストラクチャを計画する場合、次のガイドラインに従います。

* 要素はすべて、Avere vFXT のデプロイ向けに作成された新しいサブスクリプションで管理される必要があります。 この戦略によって、コスト管理とクリーンアップが簡略化され、パーティションのリソース クォータにも役立ちます。 Avere vFXT は多数のクライアントで使用されるため、1 つのサブスクリプション内のクライアントとクラスターを分離させて、その他の重要なワークフローをクライアントのプロビジョニング中の可能性があるリソースの調整から保護します。

* ご利用のクライアント コンピューティング システムを vFXT クラスターの近くに配置します。 バックエンド ストレージは、さらにリモートである可能性があります。  

* 簡単にするために、vFXT クラスターとクラスター コントローラー VM を、同じ仮想ネットワーク (VNet) と同じリソース グループに配置します。 また、これらは同じストレージ アカウントを使用する必要もあります。 

* このクラスターは、IP アドレスがクライアントやコンピューティング リソースと競合することを避けるために、独自のサブネットに配置する必要があります。 

## <a name="ip-address-requirements"></a>IP アドレスの要件 

ご利用のクラスターのサブネットに、クラスターをサポートするために十分な IP アドレスの範囲があることを確認してください。 

Avere vFXT クラスターでは、次の IP アドレスを使用します。

* 1 つのクラスターの管理 IP アドレス。 このアドレスはクラスター内のノードからノードへ移動できますが、Avere Control Panel の構成ツールに接続できるように、常に利用可能です。
* クラスター ノードごと
  * 少なくとも 1 つのクライアント側の IP アドレス。 (クライアント側のアドレスはすべて、クラスターの *vserver* によって管理されます。これは、必要に応じて、ノード間を移動できます。)
  * クラスター通信用に 1 つの IP アドレス
  * (VM に割り当てられている) 1 つのインスタンスの IP アドレス

Azure BLOB ストレージを使用する場合、ご利用のクラスターの VNet からの IP アドレスを必要とする可能性もあります。  

* Azure BLOB ストレージ アカウントには、少なくとも 5 つの IP アドレスが必要です。 BLOB ストレージをご利用のクラスターと同じ VNet に配置する場合、この要件に留意してください。
* ご利用のクラスターの仮想ネットワークの外側にある Azure BLOB ストレージを使用する場合、VNet の内側にストレージ サービス エンドポイントを作成する必要があります。 このエンドポイントでは、IP アドレスを使用しません。

クラスターとは別のリソース グループにネットワーク リソースと BLOB ストレージ (使用する場合) を配置するオプションがあります。

## <a name="vfxt-node-sizes"></a>vFXT ノードのサイズ 

クラスター ノートとして機能する VM によって、要求のスループットと自分のキャッシュのストレージ容量が決定されます。 異なるメモリ、プロセッサ、ローカル ストレージの特性と共に、2 つのインスタンスの種類から選ぶことができます。 

それぞれの vFXT ノードは同一になります。 つまり、3 ノード クラスターを作成する場合、同じ種類とサイズの VM が 3 つ用意されます。 

| インスタンスの種類 | vCPU 数 | メモリ  | ローカル SSD ストレージ  | 最大データ ディスク数 | キャッシュされていないディスク スループット | NIC (数) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D16s_v3 | 16  | 64 GiB  | 128 GiB  | 32 | 25,600 IOPS <br/> 384 Mbps | 8,000 Mbps (8) |
| Standard_E32s_v3 | 32  | 256 GiB | 512 GiB  | 32 | 51,200 IOPS <br/> 768 Mbps | 16,000 Mbps (8)  |

ノードごとのディスク キャッシュを構成することができ、1,000 GB から 8,000 GB の範囲にすることができます。 Standard_D16s_v3 ノードには、ノードごとに 1 TB のキャッシュ サイズをお勧めします。また、Standard_E32s_v3 ノードには、ノードごとに 4 TB をお勧めします。

3 つの VM に関するその他の情報については、次の Microsoft Azure ドキュメントを参照してください。

* [汎用仮想マシンのサイズ](https://docs.microsoft.com/azure/virtual-machines/windows/sizes-general)
* [メモリ最適化済み仮想マシンのサイズ](https://docs.microsoft.com/azure/virtual-machines/windows/sizes-memory)

## <a name="account-quota"></a>アカウント クォータ

ご利用のサブスクリプションに、Avere vFXT クラスターおよび使用されている任意のコンピューティング システムまたはクライアント システムを実行する容量があることを確認してください。 詳細については、「[vFXT クラスターのクォータ](avere-vfxt-prereqs.md#quota-for-the-vfxt-cluster)」を参照してください。

## <a name="back-end-data-storage"></a>バックエンドのデータ ストレージ

キャッシュにない場合、ワーキング セットは新しい BLOB コンテナーまたは既存のクラウドやハードウェア ストレージ システムに保存されますか?

バック エンドで Azure BLOB ストレージを使用する必要がある場合、vFXT クラスターを作成する一環として、新しいコンテナーを作成する必要があります。 ``create-cloud-backed-container`` デプロイ スクリプトを使用して、新しい BLOB コンテナーにストレージ アカウントを指定します。 このオプションでは、クラスターが準備できたらすぐに使用できるように、新しいコンテナーを作成および構成します。 詳細については、「[ノードを作成してクラスターを構成する](avere-vfxt-deploy.md#create-nodes-and-configure-the-cluster)」を参照してください。

> [!NOTE]
> 空の BLOB ストレージ コンテナーのみを、Avere vFXT システムのコア フィルターとして使用できます。 vFXT は、既存のデータを保持する必要なく、そのオブジェクト ストアを管理できる必要があります。 
>
> クライアント マシンと Avere vFXT のキャッシュを使用することで、データをクラスターの新しいコンテナーに効率的にコピーする方法については、「[vFXT クラスターへのデータの移動](avere-vfxt-data-ingest.md)」を参照してください。

既存のオンプレミスのストレージ システムを使用する場合は、作成された後に、vFXT クラスターに追加する必要があります。 ``create-minimal-cluster`` デプロイ スクリプトでは、バックエンド ストレージのない vFXT クラスターが作成されます。 既存のストレージ システムを Avere vFXT クラスターに追加する方法の詳細な手順については、「[ストレージの構成](avere-vfxt-add-storage.md)」を参照してください。 

## <a name="next-step-understand-the-deployment-process"></a>次の手順: デプロイ プロセスを理解する

[デプロイの概要](avere-vfxt-deploy-overview.md)に関するページでは、Avere vFXT for Azure システムを作成し、データを使用する準備をするために必要なすべての手順の全体像を示します。  