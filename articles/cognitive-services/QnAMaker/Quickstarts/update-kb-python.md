---
title: 'クイック スタート: ナレッジ ベースを更新する - REST (Python) - QnA Maker'
description: この REST ベースのクイック スタートでは、QnA Maker ナレッジ ベース (KB) のサンプルをプログラムから更新する手順を紹介しています。 質問と回答のペアは、KB の更新に使用する JSON 定義によって追加、変更、削除することができます。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: quickstart
ms.date: 10/19/2018
ms.author: diberry
ms.openlocfilehash: 35ef0222de6093e7bae3b41377db2ea3fd3c3d75
ms.sourcegitcommit: ccdea744097d1ad196b605ffae2d09141d9c0bd9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/23/2018
ms.locfileid: "49645916"
---
# <a name="quickstart-update-a-knowledge-base-in-qna-maker-using-python"></a>クイック スタート: Python を使用して QnA Maker のナレッジ ベースを更新する

このクイック スタートでは、既存の QnA Maker ナレッジ ベース (KB) をプログラムから更新する手順を紹介しています。  この JSON を使用して新しいデータ ソースを追加したり、データ ソースを変更したり、データ ソースを削除したりすることによって、KB を更新することができます。

この API には、QnA Maker ポータルでの編集後、**[Save and train]\(保存してトレーニング\)** ボタンを使用することと同等の働きがあります。

このクイック スタートで呼び出す QnA Maker API は次のとおりです。
* [更新](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600) - ナレッジ ベースのモデルは、API 要求の本文で送信される JSON で定義されます。 
* [取得操作の詳細](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/operations_getoperationdetails)

[!INCLUDE [Code is available in Azure-Samples Github repo](../../../../includes/cognitive-services-qnamaker-python-repo-note.md)]

まだナレッジ ベースがない場合は、[新しいナレッジ ベースの作成](create-new-kb-python.md)に関するページを参照し、このクイック スタート用のサンプルを作成してください。

1. 適切な IDE で新しい Python プロジェクトを作成します。
1. 次に示すコードを追加します。
1. `subscriptionKey` 値を、有効なサブスクリプション キーに置き換えます。
1. `kb` 値を、有効なナレッジ ベース ID に置き換えます。 この値は、ご利用の [QnA Maker ナレッジ ベース](https://www.qnamaker.ai/Home/MyServices)のいずれかにアクセスして調べることができます。 更新するナレッジ ベースを選択します。 そのページにアクセスしたら、URL 中の "kdid=" を探してください (下図参照)。 その値をコード サンプルで使用します。

    ![QnA Maker ナレッジ ベース ID](../media/qnamaker-quickstart-kb/qna-maker-id.png)

1. プログラムを実行します。

```python
# -*- coding: utf-8 -*-

import http.client, urllib.parse, json, time

# **********************************************
# *** Update or verify the following values. ***
# **********************************************

# Replace this with a valid subscription key.
subscriptionKey = 'ADD KEY HERE'

# Replace this with a valid knowledge base ID.
kb = 'ADD ID HERE';

# Represents the various elements used to create HTTP request path
# for QnA Maker operations.
host = 'westus.api.cognitive.microsoft.com'
service = '/qnamaker/v4.0'
method = '/knowledgebases/'

'''
Formats and indents JSON for display.
:param content: The JSON to format and indent.
:type: string
:return: Formatted and indented JSON.
:rtype: string
'''
def pretty_print(content):
    # Note: We convert content to and from an object so we can pretty-print it.
    return json.dumps(json.loads(content), indent=4)

'''
Sends a PATCH HTTP request.
:param path: The URL path of your request.
:param content: The contents of your PATCH request.
:type: string
:return: URL with status of PATCH request in updating the kb, actual response.
:rtype: string, string
'''
def update_kb(path, content):
    print('Calling ' + host + path + '.')
    headers = {
        'Ocp-Apim-Subscription-Key': subscriptionKey,
        'Content-Type': 'application/json',
        'Content-Length': len (content)
    }
    conn = http.client.HTTPSConnection(host)
    conn.request ("PATCH", path, content, headers)
    response = conn.getresponse ()
    return response.getheader('Location'), response.read ()

'''
Gets the status of the specified QnA Maker operation.
:param path: The URL of the request.
:type: string
:return: Header from retrying of the request (if retry is needed), response of the retry.
:rtype: string, string
'''
def check_status (path):
    print('Calling ' + host + path + '.')
    headers = {'Ocp-Apim-Subscription-Key': subscriptionKey}
    conn = http.client.HTTPSConnection(host)
    conn.request("GET", path, None, headers)
    response = conn.getresponse ()
    # If the operation is not finished, /operations returns an HTTP header named
    # 'Retry-After' with the number of seconds to wait before querying the operation again.
    return response.getheader('Retry-After'), response.read ()

'''
Dictionary that holds the knowledge base.
Modifications to the knowledge base are made here, using 'update', 'delete' and so on.
'''
req = {
    'add': {
        'qnaList': [
            {
              'id': 1,
              'answer': 'You can change the default message if you use the QnAMakerDialog. '
                      + 'See this for details:  https://docs.botframework.com/en-us'
                      + '/azure-bot-service/templates/qnamaker/#navtitle',
              'source': 'Custom Editorial',
              'questions': [
                  'How can I change the default message from QnA Maker?'
              ],
              'metadata': []
            }
        ],
        'urls': []
    },
    'update' : {
        'name' : 'New KB Name'
    },
    'delete': {
        'ids': [
            0
        ]
    }
}

# Builds the path URL.
path = service + method + kb
# Convert the request to a string.
content = json.dumps(req)
# Retrieve the operation ID to check status, and JSON result.
operation, result = update_kb(path, content)
# Print request response in JSON with presentable formatting.
print(pretty_print(result))

'''
Iteratively gets the operation state, updating the knowledge base.
Once state is no longer "Running" or "NotStarted", the loop ends.
'''
done = False
while False == done:
    path = service + operation
    # Gets the status of the operation.
    wait, status = check_status(path)
    # Print status checks in JSON with presentable formatting
    print(pretty_print(status))

    # Convert the JSON response into an object and get the value of the operationState field.
    state = json.loads(status)['operationState']
    # If the operation isn't finished, wait and query again.
    if state == 'Running' or state == 'NotStarted':
        print('Waiting ' + wait + ' seconds...')
        time.sleep (int(wait))
    else:
        done = True # request has been processed, if successful, knowledge base is updated
```

## <a name="understand-what-qna-maker-returns"></a>QnA Maker から返される応答

成功した応答は、次の例に示すように JSON で返されます。 実際に返される結果は若干異なる場合があります。 最後の呼び出しで "Succeeded" 状態が返された場合、ナレッジ ベースは正常に更新されています。 トラブルシューティングについては、QnA Maker API の「[Update Knowledgebase (ナレッジ ベースの更新)](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600)」の応答コードをご覧ください。

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-04-13T01:49:48Z",
  "lastActionTimestamp": "2018-04-13T01:49:48Z",
  "userId": "2280ef5917rt4ebfa1aae41fb1cebb4a",
  "operationId": "5156f64e-e31d-4638-ad7c-a2bdd7f41658"
}
...
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-04-13T01:49:48Z",
  "lastActionTimestamp": "2018-04-13T01:49:50Z",
  "resourceLocation": "/knowledgebases/140a46f3-b248-4f1b-9349-614bfd6e5563",
  "userId": "2280ef5917rt4ebfa1aae41fb1cebb4a",
  "operationId": "5156f64e-e31d-4638-ad7c-a2bdd7f41658"
}
Press any key to continue.
```

更新されたナレッジ ベースは、QnA Maker ポータルの [[My knowledge bases]\(マイ ナレッジ ベース\)](https://www.qnamaker.ai/Home/MyServices) ページで確認できます。 新しい QnA ペアが追加され、データベースの名前が "New KB Name" に変わっていることに注目してください。

ナレッジ ベースに関して、その他の要素に変更を加える場合は、QnA Maker の [JSON スキーマ](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600)を参照し、`req` 文字列を編集してください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [QnA Maker (V4) REST API リファレンス](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff)
