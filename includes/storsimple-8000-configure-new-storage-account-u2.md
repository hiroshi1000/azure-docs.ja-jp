---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 2f847cfe4e0097cdb6f70e8831e15e602b5e9ac6
ms.sourcegitcommit: 48592dd2827c6f6f05455c56e8f600882adb80dc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/26/2018
ms.locfileid: "50165055"
---
<!--author=alkohli last changed: 01/20/17-->


#### <a name="to-add-a-storage-account-credential-in-the-same-azure-subscription-as-the-storsimple-device-manager-service"></a>StorSimple デバイス マネージャー サービスと同じ Azure サブスクリプションにストレージ アカウントの資格情報を追加するには、次の手順を実行します。

1. StorSimple デバイス マネージャー サービスに移動します。 **[構成]** セクションで **[ストレージ アカウントの資格情報]** をクリックします。

    ![ストレージ アカウントの資格情報](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct1.png)

2. **[ストレージ アカウントの資格情報]** ブレードで、**[+ 追加]** をクリックします。

    ![ストレージ アカウントの資格情報を追加する](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct2.png)

3. **[ストレージ アカウント資格情報の追加]** ブレードで、次の手順を実行します。

    1. お使いのサービスと同じ Azure サブスクリプションにストレージ アカウントの資格情報を追加するので、**[現在]** が選択されていることを確認します。

    2. **[ストレージ アカウント]** ボックスの一覧で、既存のストレージ アカウントを選択します。

    3. 選択したストレージ アカウントに基づいて、**[場所]** が表示されます (灰色表示になっており、変更することはできません)。

    4. **[SSL モードを有効にする]** を選択して、デバイスとクラウド間のネットワーク通信用のセキュリティで保護されたチャネルを作成します。 プライベート クラウド内で動作している場合にのみ、**[SSL を有効にする]** を無効にします。

        ![[ストレージ アカウントの資格情報] ブレードの追加](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct3.png)

    5. **[追加]** をクリックすると、ストレージ アカウントの資格情報のジョブの作成が開始されます。 ストレージ アカウントの資格情報が正常に作成されると、その旨が通知が表示されます。

        ![ストレージ アカウントの資格情報が正常に作成された旨の通知](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct5.png)

新しく作成されたストレージ アカウントの資格情報が、**[ストレージ アカウントの資格情報]** の一覧に表示されます。

![ストレージ アカウントの資格情報の一覧](./media/storsimple-8000-configure-new-storage-account-u2/createnewstorageacct6.png)

