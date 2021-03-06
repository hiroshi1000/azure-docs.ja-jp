---
title: Azure portal を使用して Azure Database for MySQL VNet のサービス エンドポイントとルールを作成および管理する | Microsoft Docs
description: Azure portal を使用して、Azure Database for MySQL VNet のサービス エンドポイントとルールを作成および管理します
services: mysql
author: mbolz
ms.author: mbolz
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 10/22/2018
ms.openlocfilehash: 73655cf55a93a758ecb29b3408cdd0d803060397
ms.sourcegitcommit: 5c00e98c0d825f7005cb0f07d62052aff0bc0ca8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49956644"
---
# <a name="create-and-manage-azure-database-for-mysql-vnet-service-endpoints-and-vnet-rules-by-using-the-azure-portal"></a>Azure portal を使用して、Azure Database for MySQL VNet のサービス エンドポイントと VNet ルールを作成および管理する
Virtual Network (VNet) サービス エンドポイントおよびルールは、仮想ネットワークのプライベート アドレス空間を Azure Database for MySQL サーバーに拡張します。 制限事項も含め、Azure Database for MySQL VNet サービス エンドポイントの概要については、[Azure Database for MySQL サーバー VNet サービス エンドポイント ](concepts-data-access-and-security-vnet.md) に関する記事を参照してください。 VNet サービス エンドポイントは、Azure Database for MySQL でサポートされるすべてのリージョンで利用できます。

> [!NOTE]
> VNet サービス エンドポイントは、汎用サーバーとメモリ最適化サーバーでのみサポートされています。

## <a name="create-a-vnet-rule-and-enable-service-endpoints-in-the-azure-portal"></a>Azure portal で VNet ルールを作成してサービス エンドポイントを有効にする

1. MySQL サーバー ページの [設定] で、**[接続のセキュリティ]** をクリックして Azure Database for MySQL の [接続のセキュリティ] ページを開きます。 次に、**[+ 既存の仮想ネットワークを追加]** をクリックします。 既存の VNet がない場合は、**[+ 新しい仮想ネットワークの作成]** をクリックして作成できます。 「[クイック スタート: Azure Portal を使用した仮想ネットワークの作成](../virtual-network/quick-create-portal.md)」をご覧ください

   ![Azure Portal - [接続のセキュリティ] のクリック](./media/howto-manage-vnet-using-portal/1-connection-security.png)

2. VNet ルール名を入力し、サブスクリプション、仮想ネットワーク、サブネット名を選択して、**[有効にする]** をクリックします。 これにより、**Microsoft.SQL** サービス タグを使用してサブネット上で VNet サービス エンドポイントが自動的に有効になります。

   ![Azure portal - VNet の構成](./media/howto-manage-vnet-using-portal/2-configure-vnet.png)

   アカウントには、仮想ネットワークとサービス エンドポイントを作成するためのアクセス許可が必要です。

   サービス エンドポイントは、仮想ネットワークへの書き込みアクセス権を持つユーザーが仮想ネットワーク上で個別に構成できます。
    
   Azure サービス リソースへのアクセスを VNet に限定するには、ユーザーが、追加されるサブネットの "Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/" へのアクセス許可を持っている必要があります。 このアクセス許可は、既定では組み込みのサービス管理者のロールに含まれ、カスタム ロールを作成することで変更できます。
    
   [組み込みロール](https://docs.microsoft.com/azure/active-directory/role-based-access-built-in-roles)と、特定のアクセス許可を[カスタム ロール](https://docs.microsoft.com/azure/active-directory/role-based-access-control-custom-roles)に割り当てる方法の詳細をご覧ください。
    
   Vnet と Azure サービス リソースのサブスクリプションは、同じでも異なっていてもかまいません。 VNet と Azure サービス リソースのサブスクリプションが異なる場合、リソースは同じ Active Directory (AD) テナントの下に置かれている必要があります。

   > [!IMPORTANT]
   > サービス エンドポイントを構成する前に、サービス エンドポイントの構成と考慮事項について、この記事を読むことを強くお勧めします。 **仮想ネットワーク サービス エンドポイント:** [仮想ネットワーク サービス エンドポイント](../virtual-network/virtual-network-service-endpoints-overview.md)は、プロパティ値に 1 つ以上の正式な Azure サービスの種類名が含まれるサブネットです。 VNet サービス エンドポイントでは、SQL Database という名前の Azure サービスを参照する **Microsoft.Sql** というサービス種類名を使用します。 このサービス タグは、Azure SQL Database、Azure Database for PostgreSQL および MySQL サービスにも適用されます。 VNet サービス エンドポイントに **Microsoft.Sql** サービス タグを適用すると、サブネット上の Azure SQL Database、Azure Database for PostgreSQL、Azure Database for MySQL サーバーを含むすべての Azure Database サービスに対してサービス エンドポイント トラフィックが構成されることに注意することが重要です。 
   > 

3. 有効になったら、**[OK]** をクリックすると、VNet ルールと共に VNet サービス エンドポイントが有効になっていることが表示されます。

   ![VNet サービス エンドポイントが有効で、VNet ルールが作成されている](./media/howto-manage-vnet-using-portal/3-vnet-service-endpoints-enabled-vnet-rule-created.png)

## <a name="next-steps"></a>次の手順
- 同様に、スクリプトを作成して、[Azure CLI を使用して、VNet サービス エンドポイントを有効にし、Azure Database for MySQL の VNET ルールを作成](howto-manage-vnet-using-cli.md)できます。
- Azure Database for MySQL サーバーに接続する方法のヘルプについては、「[Azure Database for MySQL の接続ライブラリ](./concepts-connection-libraries.md)」をご覧ください。
