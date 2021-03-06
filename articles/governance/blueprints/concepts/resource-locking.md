---
title: Azure Blueprint でのリソース ロックについて
description: ブループリントを割り当てるときにリソースを保護するロック オプションについて説明します。
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 10/25/2018
ms.topic: conceptual
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: 4e71797837927fe5f5233bcf88d35fef98f504e9
ms.sourcegitcommit: 0f54b9dbcf82346417ad69cbef266bc7804a5f0e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/26/2018
ms.locfileid: "50139444"
---
# <a name="understand-resource-locking-in-azure-blueprints"></a>Azure Blueprint でのリソース ロックについて

一貫性のある環境を大規模に作成することが本当に価値があるのは、一貫性を維持するためのメカニズムがある場合だけです。 この記事では、Azure Blueprint でのリソース ロックのしくみについて説明します。

## <a name="locking-modes-and-states"></a>ロック モードと状態

ブループリント割り当てに適用されるロック モードには、**なし**と**すべてのリソース**の 2 つのオプションしかありません。 ロック モードはブループリント割り当て時に構成され、割り当てがサブスクリプションに正常に適用された後は変更できません。

ブループリント割り当て内のアーティファクトによって作成されたリソースは、**ロックなし**、**読み取り専用**、または**編集/削除不可**の 3 つのいずれかの状態になります。 **ロックなし**の状態は、各アーティファクトに共通するものです。 しかし、**読み取り専用**状態になるのはリソース グループ以外のアーティファクトで、**編集/削除不可**状態となるのはリソース グループです。 これは、これらのリソースの管理方法において重要な違いです。

**読み取り専用**状態は、定義されているとおりで、いかなる方法でもリソースを変更することはできません。変更することも削除することもできません。 **編集/削除不可**は、リソース グループの "コンテナー" の性質により、少し異なります。 リソース グループ オブジェクトは読み取り専用ですが、リソース グループ内のロックされていないリソースを変更することは可能です。

## <a name="overriding-locking-states"></a>ロック状態をオーバーライドする

通常、サブスクリプションに対する適切な[ロールベースのアクセス制御](../../../role-based-access-control/overview.md) (RBAC) を持つユーザー (所有者ロールのユーザーなど) が、リソースの変更や削除を許可されることはありえます。 ただし、デプロイされた割り当ての一部として Blueprint がロックを適用している場合には、このアクセスは許可されません。 **ロック** オプションを使用して割り当てが設定された場合、サブスクリプション所有者であっても含まれているリソースを変更することはできません。

このセキュリティ対策により、定義済みのブループリントの一貫性と作成目的で設計された環境が、偶発的またはプログラムによる削除や変更から保護されます。

## <a name="removing-locking-states"></a>ロック状態を削除する

割り当てによって作成したリソースを削除する必要が生じた場合、リソースを削除するには、まず割り当てを削除する必要があります。 割り当てを削除すると、Blueprint によって作成されたロックが削除されます。 ただし、リソース自体は維持されるので、通常の方法を使用して削除する必要があります。

## <a name="how-blueprint-locks-work"></a>ブループリントのロックのしくみ

割り当てで**ロック** オプションを選択した場合、ブループリントの割り当て時に RBAC ロール `denyAssignments` がアーティファクト リソースに適用されます。 このロールは、ブループリント割り当てのマネージド ID によって追加され、アーティファクト リソースから削除するには、同じマネージド ID を使用する必要があります。 このセキュリティ対策により、ロック メカニズムを強制して、Blueprint 外からブループリントのロックを解除できないようにします。 ロールおよびロックを削除するには、ブループリント割り当てを削除する必要があり、これは適切な権限を持つ人だけが実行できます。

> [!IMPORTANT]
> Azure Resource Manager では、ロール割り当ての詳細が最大 30 分間キャッシュされます。 そのため、ブループリント リソースに対する `denyAssignments` は、すぐには完全に反映されない場合があります。 その間は、ブルー プリントのロックで保護しようとしたリソースが削除される可能性もあります。

## <a name="next-steps"></a>次の手順

- [ブループリントのライフサイクル](lifecycle.md)を参照する
- [静的および動的パラメーター](parameters.md)の使用方法を理解する
- [ブループリントの優先順位](sequencing-order.md)のカスタマイズを参照する
- [既存の割り当ての更新](../how-to/update-existing-assignments.md)方法を参照する
- ブループリントの割り当て時の問題を[一般的なトラブルシューティング](../troubleshoot/general.md)で解決する