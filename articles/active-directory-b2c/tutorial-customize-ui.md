---
title: チュートリアル - Azure Active Directory B2C でアプリケーションのユーザー インターフェイスをカスタマイズする | Microsoft Docs
description: Azure portal を使用し、Azure Active Directory B2C でアプリケーションのユーザー インターフェイスをカスタマイズする方法について説明します。
services: B2C
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 10/30/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 9c206ac7a13ea222a01cac78c447c0764f753517
ms.sourcegitcommit: 6135cd9a0dae9755c5ec33b8201ba3e0d5f7b5a1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/31/2018
ms.locfileid: "50669349"
---
# <a name="tutorial-customize-the-user-interface-of-your-applications-in-azure-active-directory-b2c"></a>チュートリアル - Azure Active Directory B2C でアプリケーションのユーザー インターフェイスをカスタマイズする

サインアップ、サインイン、プロファイル編集など、より一般的なユーザー エクスペリエンスについては、Azure Active Directory (Azure AD) B2C の[組み込みポリシー](active-directory-b2c-reference-policies.md)を使用できます。 このチュートリアルの情報は、独自の HTML ファイルや CSS ファイルを使用してそのようなエクスペリエンスの[ユーザー インターフェイス (UI) をカスタマイズする](customize-ui-overview.md)方法を理解する上で役立ちます。

この記事では、次のことについて説明します:

> [!div class="checklist"]
> * UI カスタマイズ ファイルを作成する
> * ファイルを使用するサインアップ/サインイン ポリシーを作成する
> * カスタマイズした UI をテストする

Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

## <a name="prerequisites"></a>前提条件

独自の [Azure AD B2C テナント](tutorial-create-tenant.md)をまだ作成していない場合、すぐに作成してください。 前のチュートリアルで作成した場合、既存のテナントを使用できます。

## <a name="create-customization-files"></a>カスタマイズ ファイルを作成する

Azure のストレージ アカウントとコンテナーを作成し、基本の HTML ファイルと CSS ファイルをコンテナーに置きます。

### <a name="create-a-storage-account"></a>ストレージ アカウントの作成

ファイルはさまざまな方法で保存できますが、このチュートリアルでは、[Azure BLOB ストレージ](../storage/blobs/storage-blobs-introduction.md)に保存します。

1. Azure サブスクリプションが含まれるディレクトリを必ず使用してください。 上部メニューで **[ディレクトリとサブスクリプション]** フィルターを選択し、ご利用のサブスクリプションが含まれるディレクトリを選択します。 このディレクトリは、Azure B2C テナントが含まれるディレクトリとは異なります。

    ![サブスクリプション ディレクトリに切り替える](./media/tutorial-customize-ui/switch-directories.png)

2. Azure portal の左上隅の [すべてのサービス] を選択し、**[ストレージ アカウント]** を検索して選択します。 
3. **[追加]** を選択します。
4. **[リソース グループ]** の下で **[新規作成]** を選択し、新しいリソース グループの名前を入力し、**[OK]** をクリックします。
5. ストレージ アカウントの名前を入力します。 選択する名前は、Azure 全体で一意であり、長さは 3 ～ 24 文字でなければならず、数字と小文字のみを使用できます。
6. ストレージ アカウントの場所を選択するか、既定の場所をそのまま使用します。 
7. その他の既定値をそのまま使用し、**[確認および作成]** を選択し、**[作成]** をクリックします。
8. ストレージ アカウントの作成後、**[リソースに移動]** を選択します。

### <a name="create-a-container"></a>コンテナーを作成する

1. ストレージ アカウントの [概要] ページで **[BLOB]** を選択します。
2. **[コンテナー]** を選択し、コンテナーの名前を入力し、**[BLOB (BLOB 専用の匿名読み取りアクセス)]** を選択し、**[OK]** をクリックします。

### <a name="enable-cors"></a>CORS を有効にする

 ブラウザーの Azure AD B2C コードでは、最新かつ標準の手法を利用し、ポリシーに指定された URL からカスタム コンテンツを読み込みます。 クロスオリジン リソース共有 (CORS) によって、Web ページ上の制限付きリソースを他のドメインから要求することが許可されます。

1. メニューで **[CORS]** を選択します。
2. **[許可されたオリジン]**、**[許可されたヘッダー]**、**[公開されるヘッダー]** については、「`your-tenant-name.b2clogin.com`」と入力します。 `your-tenant-name`を Azure AD B2C テナントの名前に置き換えます。 たとえば、「 `fabrikam.b2clogin.com` 」のように入力します。
3. **[使用できる動詞]** には、`GET` と `OPTIONS` を両方選択します。
4. **[最長有効期間]** には「200」と入力します。

    ![CORS を有効にする](./media/tutorial-customize-ui/enable-cors.png)

5. **[Save]** をクリックします。

### <a name="create-the-customization-files"></a>カスタマイズ ファイルを作成する

サインアップ エクスペリエンスの UI をカスタマイズするには、まず簡単な HTML ファイルと CSS ファイルを作成することから始めます。 HTML は自由に構成できますが、識別子が `api` の **div** 要素を含める必要があります。 たとえば、「 `<div id="api"></div>` 」のように入力します。 ページが表示されるとき、Azure AD B2C によって `api` コンテナーに要素が挿入されます。

1. ローカル フォルダーで次のファイルを作成したら、必ずストレージ アカウントの名前を `your-storage-account` に変更し、作成したコンテナーの名前を `your-container` に変更します。 たとえば、「 `https://store1.blob.core.windows.net/b2c/style.css` 」のように入力します。

    ```html
    <!DOCTYPE html>
    <html>
      <head>
        <title>My B2C Application</title>
        <link rel="stylesheet" href="https://your-storage-account.blob.core.windows.net/your-container/style.css">
      </head>
      <body>  
        <h1>My B2C Application</h1>
        <div id="api"></div>
      </body>
    </html>
    ```

    ページは自由にデザインできますが、作成した HTML カスタマイズ ファイルには **api** div 要素が必須です。 

3. *custom-ui.html* としてファイルを保存します。
4. Azure AD B2C によって挿入される要素など、サインアップまたはサインイン ページのすべての要素を中心とする次のような簡単な CSS を作成します。

    ```css
    h1 {
      color: blue;
      text-align: center;
    }
    .intro h2 {
      text-align: center; 
    }
    .entry {
      width: 300px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    .divider h2 {
      text-align: center; 
    }
    .create {
      width: 300px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    ```

5. *style.css* としてファイルを保存します。

### <a name="upload-the-customization-files"></a>カスタマイズ ファイルをアップロードする

このチュートリアルでは、Azure AD B2C でアクセスできるように、ストレージ アカウントで作成したファイルを保存します。

1. Azure portal の左上隅の **[すべてのサービス]** を選択し、**[ストレージ アカウント]** を検索して選択します。
2. 作成したストレージ アカウントを選択し、**[BLOB]** を選択し、作成したコンテナーを選択します。
3. **[アップロード]** を選択し、*custom-ui.html* ファイルに移動してそれを選択し、**[アップロード]** をクリックします。

    ![カスタマイズ ファイルをアップロードする](./media/tutorial-customize-ui/upload-blob.png)

4. チュートリアルの後半で使用するため、アップロードしたファイルの URL をコピーしておきます。
5. *style.css* ファイルに手順 3 と 4 を繰り返します。

## <a name="create-a-sign-up-and-sign-in-policy"></a>サインアップおよびサインイン ポリシーの作成

このチュートリアルの手順を完了するには、Azure AD B2C でテスト アプリケーションとサインアップまたはサインイン ポリシーを作成する必要があります。 このチュートリアルで説明する原則は、プロファイル編集など、他のユーザー エクスペリエンスに応用できます。

### <a name="create-an-azure-ad-b2c-application"></a>Azure AD B2C アプリケーションを作成する

Azure AD B2C との通信は、テナントで作成したアプリケーション経由で行われます。 次の手順では、[https://jwt.ms](https://jwt.ms) に返される認証トークンをリダイレクトするアプリケーションが作成されます。

1. [Azure Portal](https://portal.azure.com) にサインインします。
2. お使いの Azure AD B2C テナントを含むディレクトリを使用していることを確認してください。確認のために、トップ メニューにある **[ディレクトリとサブスクリプション フィルター]** をクリックして、お使いのテナントを含むディレクトリを選択します。
3. Azure portal の左上隅にある **[すべてのサービス]** を選択してから、**[Azure AD B2C]** を検索して選択します。
4. **[アプリケーション]** を選択し、**[追加]** を選択します。
5. アプリケーションの名前を入力します (*testapp1* など)。
6. **[Web アプリ / Web API]** には `Yes` を選択し、**[応答 URL]** に `https://jwt.ms` を入力します。
7. **Create** をクリックしてください。

### <a name="create-the-policy"></a>ポリシーの作成

カスタマイズ ファイルをテストするには、前に作成したアプリケーションを使用する組み込みのサインアップまたはサインイン ポリシーを作成します。

1. Azure AD B2C テナントで、**[サインアップまたはサインイン ポリシー]** を選択し、**[追加]** をクリックします。
2. ポリシーの名前を入力します。 たとえば、「*signup_signin*」にします。 ポリシーが作成されるとき、プレフィックス *B2C_1* is が自動的に名前に追加されます。
3. **[ID プロバイダー]** を選択し、ローカル アカウントの **[Email sign-up]\(電子メール サインアップ\)** を設定し、**[OK]** をクリックします。
4. **[サインアップ属性]** を選択し、サインアップ中にカスタマーから収集する属性を選択します。 たとえば、**[国/リージョン]**、**[表示名]**、**[郵便番号]** を設定し、**[OK]** をクリックします。
5. **[アプリケーション要求]** を選択し、サインアップまたはサインイン エクスペリエンスの成功後にアプリケーションに戻される承認トークンで返される要求を選択します。 たとえば、**[表示名]**、**[ID プロバイダー]**、**[郵便番号]**、**[新しいユーザー]**、**[ユーザーのオブジェクト ID]** を選択し、**[OK]** をクリックします。
6. **[ページ UI のカスタマイズ]** を選択し、**[統合されたサインアップまたはサインイン ページ]** を選択し、**[カスタム ページの使用]** に対して **[はい]** をクリックします。
7. **[カスタム ページ URI]** に、先に記録しておいた *custom-ui.html* ファイルの URL を入力し、**[OK]** をクリックします。
8. **Create** をクリックしてください。

## <a name="test-the-policy"></a>ポリシーをテストする

1. Azure AD B2C テナントで、**[サインアップまたはサインイン ポリシー]** を選択し、作成したポリシーを選択します。 たとえば、*B2C_1_signup_signin* を選択します。
2. 作成したアプリケーションが **[アプリケーションの選択]** で選択されていることを確認し、**[今すぐ実行]** をクリックします。

    ![サインアップまたはサインイン ポリシーの実行](./media/tutorial-customize-ui/signup-signin.png)

    次の例のような、作成した CSS ファイルに基づいて要素が配置されているページが表示されるはずです。

    ![ポリシーの結果](./media/tutorial-customize-ui/run-now.png) 

## <a name="next-steps"></a>次の手順

この記事で学習した内容は次のとおりです。

> [!div class="checklist"]
> * UI カスタマイズ ファイルを作成する
> * ファイルを使用するサインアップ/サインイン ポリシーを作成する
> * カスタマイズした UI をテストする

> [!div class="nextstepaction"]
> [Azure Active Directory B2C での言語のカスタマイズ](active-directory-b2c-reference-language-customization.md)