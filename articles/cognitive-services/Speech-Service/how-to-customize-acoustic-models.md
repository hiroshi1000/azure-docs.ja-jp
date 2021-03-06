---
title: 'チュートリアル: Speech Service で音響モデルを作成する'
titlesuffix: Azure Cognitive Services
description: Azure Cognitive Services の Speech Service を使用して音響モデルを作成する方法について説明します。
services: cognitive-services
author: PanosPeriorellis
manager: cgronlun
ms.service: cognitive-services
ms.component: speech-service
ms.topic: tutorial
ms.date: 06/25/2018
ms.author: panosper
ms.openlocfilehash: 70fc9c34599f27eb5d67b79ef823f8037ae55ba9
ms.sourcegitcommit: 6e09760197a91be564ad60ffd3d6f48a241e083b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2018
ms.locfileid: "50215244"
---
# <a name="tutorial-create-a-custom-acoustic-model"></a>チュートリアル: カスタム音響モデルを作成する

カスタム音響モデルの作成が役立つのは、アプリケーションが、使われる録音デバイスや使用状況がはっきりしている特定の環境 (自動車など) で使用されることや、特定のユーザーによって使用されることを想定した設計になっている場合です。 たとえば、アクセント記号付きの音声、特定の背景ノイズ、録音に特定のマイクを使用する場合などです。

この記事では、次のことについて説明します:
> [!div class="checklist"]
> * データを準備する
> * 音響データセットをインポートする
> * カスタム音響モデルを作成する

Azure Cognitive Services アカウントをお持ちでない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/try/cognitive-services)を作成してください。

## <a name="prerequisites"></a>前提条件

[[Cognitive Services サブスクリプション]](https://cris.ai/Subscriptions) ページを開いて、Cognitive Services アカウントに接続されることを確認します。

**[Connect existing subscription]\(既存のサブスクリプションに接続する\)** を選択すると、Azure portal で作成された Speech Service サブスクリプションに接続できます。

Azure portal での Speech Service サブスクリプションの作成については、「[Speech Service を無料で試す](get-started.md)」を参照してください。

## <a name="prepare-the-data"></a>データを準備する

特定のドメインの音響モデルをカスタマイズするには、音声データのコレクションが必要です。 二言三言の発話から数百時間の音声まで、さまざまなコレクションを使用できます。 このコレクションは、一連の音声データのオーディオ ファイルと、各オーディオ ファイルの文字起こしのテキスト ファイルで構成されます。 オーディオ データは、認識エンジンを使用するシナリオで典型的なものである必要があります。

例: 

* 騒音の多い工場の環境で音声認識の精度を高めるには、オーディオ ファイルが騒音の多い工場における人の会話で構成されている必要があります。
* 1 人の話者に対するパフォーマンスを最適化する場合は (&mdash;たとえば、FDR の炉辺談話をすべて文字起こしする場合など&mdash;)、オーディオ ファイルはその話者だけの多くのサンプルで構成される必要があります。

音響モデルをカスタマイズするための音響データセットは、2 つの部分で構成されます。(1) 音声データを含む一連のオーディオ ファイルと、(2) すべてのオーディオ ファイルの文字起こしを含むファイルです。

### <a name="audio-data-recommendations"></a>オーディオ データの推奨事項

* データセット内のすべてのオーディオ ファイルは、WAV (RIFF) オーディオ形式で格納されている必要があります。
* オーディオのサンプリング レートは 8 kHz (キロヘルツ) または 16 kHz であること、サンプル値は圧縮されていないパルス符号変調 (PCM) 16 ビット符号付き整数 (short) として格納されていることが必要です。
* 単一チャンネル (モノラル) のオーディオ ファイルのみがサポートされます。
* オーディオ ファイルの長さは 100 マイクロ秒から 1 分までにすることができますが、約 10 秒から 12 秒が理想的です。 各オーディオ ファイルの先頭と末尾に 100 マイクロ秒以上の無音部分が存在することが理想ですが、一般的には 500 マイクロ秒から 1 秒の範囲です。
* データに背景ノイズがある場合は、音声コンテンツの前と後の一方または両方にさらに長い無音セグメント (数秒など) が含まれるサンプルをいくつか用意することもお勧めします。
* 各オーディオ ファイルは、単一の発話で構成される必要があります (単一のディクテーションの文、単一のクエリ、対話システムの単一のターンなど)。
* データセット内の各オーディオ ファイルには、一意のファイル名と .wav 拡張子が必要です。
* オーディオ ファイルのセットはサブディレクトリのない単一のフォルダーに置く必要があります。また、オーディオ ファイルのセット全体を単一の .zip ファイル アーカイブとしてパッケージ化する必要があります。

> [!NOTE]
> 現在、Web ポータルを使用したデータのインポートは 2 GB に制限されているので、これが音響データセットの最大サイズとなります。 このサイズは、16 kHz で録音されたオーディオの約 17 時間分、または 8 kHz で録音されたオーディオの約 34 時間分に相当します。 オーディオ データの主な要件を次の表にまとめます。
>

| プロパティ | 値 |
|---------- |----------|
| ファイル形式 | RIFF (WAV) |
| サンプル速度 | 8,000 Hz (ヘルツ) または 16,000 Hz |
| チャンネル | 1 (モノラル) |
| サンプル形式 | PCM、16 ビット整数 |
| ファイルの時間 | 0.1 秒 < 時間 < 12 秒 | 
| サイレンス カラー | > 0.1 秒 |
| アーカイブ形式 | .zip |
| 最大アーカイブ サイズ | 2 GB |

> [!NOTE]
> ファイル名は、ラテン文字のみで構成され、"<ファイル名>.<拡張子>" という形式になっている必要があります。

## <a name="language-support"></a>言語のサポート

カスタムの**音声テキスト変換**言語モデルでサポートされる言語の全一覧については、[Speech Service でサポートされる言語](language-support.md#speech-to-text)に関するページを参照してください。

### <a name="transcriptions-for-the-audio-dataset"></a>オーディオのデータセットの文字起こし

すべての WAV ファイルの文字起こしは、1 つのプレーン テキスト ファイルに格納されている必要があります。 文字起こしファイルの各行には、いずれかのオーディオ ファイルの名前に続けて、対応する文字起こしが含まれている必要があります。 ファイル名と文字起こしは、タブ (\t) で区切る必要があります。

  例: 
```
  speech01.wav  speech recognition is awesome
  speech02.wav  the quick brown fox jumped all over the place
  speech03.wav  the lazy dog was not amused
```
> [!NOTE]
> 文字起こしは、UTF-8 バイト オーダー マーク (BOM) としてエンコードされている必要があります。

文字起こしに対しては、システムによって処理できるように、テキストの正規化が行われます。 ただし、Custom Speech Service にデータをアップロードする "_前_" に、ユーザーが行う必要があるいくつかの重要な正規化があります。 文字起こしを準備する際に使用する適切な言語については、「[Speech Service を使用するための文字起こしガイドライン](prepare-transcription.md)」を参照してください。

次の各セクションの手順は、[Speech Service ポータル](https://cris.ai)を使用して実行してください。

## <a name="import-the-acoustic-dataset"></a>音響データセットをインポートする

オーディオ ファイルと文字起こしの準備ができたら、それらをサービス Web ポータルにインポートできます。

インポートするためには、まず [Speech Service ポータル](https://cris.ai)にサインインします。 次に、リボンの **[Custom Speech]** ボックスの一覧から **[Adaptation Data]\(適応データ\)** を選択します。 Custom Speech Service へのデータのアップロードを初めて行う場合は、**[データセット]** という名前の空のテーブルが表示されます。 

**[音響データセット]** 行の **[インポート]** ボタンを選択します。すると、新しいデータセットをアップロードするためのページがサイトに表示されます。

![[Import Acoustic Data]\(音響データのインポート\) ページ](media/stt/speech-acoustic-datasets-import.png)

**[名前]** ボックスと **[説明]** ボックスに、適切な情報を入力します。 アップロードしたさまざまなデータセットを追跡するのに役立つわかりやすい説明にします。 

**[Transcriptions file (.txt)]\(文字起こしファイル (.txt)\)** ボックスと **[Audio files (.zip)]\(オーディオ ファイル (.zip)\)** ボックスの **[参照]** を選択し、プレーン テキストの文字起こしファイルと WAV ファイルの zip アーカイブを選択します。 準備が完了したら、**[インポート]** を選択してデータをアップロードします。 データがアップロードされます。 比較的大きなデータセットでは、インポート プロセスに数分かかることがあります。

アップロードが完了したら、**[音響データセット]** テーブルに戻ります。 音響データセットに対応するエントリが表示されます。 一意の ID (GUID) が割り当てられていることに注意してください。 データには、その最新の状態が表示されます。処理を待っている間は *[NotStarted]\(未開始\)*、検証中は *[Running]\(実行中\)*、データが使用可能な状態になると *[Complete]\(完了\)* になります。

データの入力規則には、オーディオ ファイルに関するファイル形式、長さ、サンプル速度の確認、および文字起こしファイルに関するファイル形式の確認とテキスト正規化の実行のための、一連のチェックが含まれます。

状態が *[Succeeded]\(成功\)* になったら、**[詳細]** を選択して音響データ検証レポートを表示できます。 失敗した発言の詳細と共に、検証に成功した発言の数と失敗した発言の数が表示されます。 以下に示した画像の例では、オーディオ形式が不適切であるために検証に失敗した WAV ファイルが 2 つあります。 データセット中、1 つのファイルはサンプリング レートが不適切で、もう 1 つはファイル形式が正しくありません。

![データの適応の詳細ページ](media/stt/speech-acoustic-datasets-report.png)

データセットの名前または説明を変更したい場合は、**[編集]** リンクを選択してそのエントリを変更できます。 文字起こしファイルやオーディオ ファイルのエントリを変更することはできません。

## <a name="create-a-custom-acoustic-model"></a>カスタム音響モデルの作成

音響データセットの状態が *[Complete]\(完了\)* になったら、そのデータセットを使用してカスタム音響モデルを作成できます。 そのためには、**[Custom Speech]** ボックスの一覧の **[Acoustic Models]\(音響モデル\)** を選択します。 **[Your models]\(自分のモデル\)** という名前のテーブルに、自分のカスタム音響モデルがすべて一覧表示されます。 初めて使用するときはこのテーブルは空です。 テーブルのタイトルには現在のロケールが表示されます。 現在、音響モデルを作成できるのは英語 (米国) のみです。

新しいモデルを作成するには、テーブル タイトルの下の **[Create New]\(新規作成\)** を選択します。 前と同じように、このモデルを識別しやすい名前と説明を入力します。 たとえば、**[説明]** フィールドを使って、モデルの作成に使用した開始モデルと音響データセットを記録できます。 

次に、**[Base Acoustic Model]\(基本音響モデル\)** ボックスの一覧から基本モデルを選択します。 この基本モデルがカスタマイズの開始点です。 次の 2 つの基本音響モデルから選択できます。
* **Microsoft Search and Dictation AM** モデルは、コマンド、検索クエリ、ディクテーションなどのアプリケーションに向けられた音声に適しています。 
* **Microsoft Conversational モデル**は、会話形式の音声の認識に適しています。 この種類の音声は、通常は他の人に向けて発せられるものであり、コール センターや会議で発生します。 

Conversational モデルでの部分結果の待機時間は、Search and Dictation モデルより長くなります。

> [!NOTE]
> 現在、すべてのシナリオに対応することを目的とする新しい **Universal** モデルを展開しています。 前述のモデルも引き続きパブリックに利用できます。

次に、**[Acoustic Data]\(音響データ\)** ボックスの一覧で、カスタマイズの実行に使用する音響データを選択します。

![[Create Acoustic Model]\(音響モデルの作成\) ページ](media/stt/speech-acoustic-models-create2.png)

処理が完了したら、必要に応じて、新しいモデルの正確性テストを選択して実行できます。 このテストでは、カスタマイズされた音響モデルを使用して、指定された音響データセットを対象に音声テキスト変換の評価を実行し、結果を報告します。 このテストを実行するには、**[Accuracy Testing]\(正確性テスト\)** チェック ボックスをオンにします。 そのうえで、ドロップダウン リストから言語モデルを選択します。 カスタム言語モデルを作成していない場合は、基本言語モデルのみがドロップダウン リストに表示されます。 最適な言語モデルの選択については、「[チュートリアル: カスタム言語モデルを作成する](how-to-customize-language-model.md)」を参照してください。

最後に、カスタム モデルの評価に使用する音響データセットを選択します。 正確性テストを実行する場合は、現実的なモデルのパフォーマンスを把握できるように、モデルの作成に使用したものとは異なる音響データセットを選択することが重要です。 トレーニング データで正確性をテストすると、適応したモデルが実際の条件下でどのように動作するかを評価することができません。 結果は、過度に楽観的になります。 また、正確性テストは 1,000 個の発話に制限されることにも注意してください。 テスト用音響データセットが大きい場合は、最初の 1,000 個の発話のみが評価されます。

カスタマイズ プロセスの実行を開始する準備ができたら、**[作成]** を選択します。

この新しいモデルに対応する新しいエントリが音響モデル テーブルに表示されます。 このテーブルには、プロセスの状態 (*[Waiting]\(待機中\)*、*[Processing]\(処理中\)*、または *[Complete]\(完了\)*) も表示されます。

![[Acoustic Models]\(音響モデル\) ページ](media/stt/speech-acoustic-models-creating.png)

## <a name="next-steps"></a>次の手順

- [Speech Service 試用版サブスクリプションを取得する](https://azure.microsoft.com/try/cognitive-services/)
- [C# で音声を認識する](quickstart-csharp-dotnet-windows.md)
- [Git サンプル データ](https://github.com/Microsoft/Cognitive-Custom-Speech-Service)
