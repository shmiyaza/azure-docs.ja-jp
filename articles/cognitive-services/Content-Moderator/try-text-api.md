---
title: テキスト モデレート API を使用してテキストをモデレートする - Content Moderator
titleSuffix: Azure Cognitive Services
description: オンライン コンソールでテキスト モデレート API を使用して、テキストのモデレートを試します。
services: cognitive-services
author: PatrickFarley
ms.author: pafarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 05/29/2019
ms.openlocfilehash: e0930558f31b27a77fa2cd6b44fcea2fe9091086
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "74538820"
---
# <a name="moderate-text-from-the-api-console"></a>API コンソールからテキストをモデレートする

Azure Content Moderator の [Text Moderation API](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f) を使用して、テキスト コンテンツの内容をスキャンし、カスタム一覧と共有一覧と比較します。

## <a name="get-your-api-key"></a>API キーを取得する

オンライン コンソールで API を試すには、サブスクリプション キーが必要です。 これは、 **[設定]** タブの **[Ocp-Apim-Subscription-Key]** ボックス内にあります。 詳細については、[概要](overview.md)に関するページを参照してください。

## <a name="navigate-to-the-api-reference"></a>API リファレンスに移動する

[テキスト モデレート API のリファレンス](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f)に移動します。 

  **[Text - Screen]** (テキスト - 選別) ページが開きます。

## <a name="open-the-api-console"></a>API コンソールを開く

**API テスト コンソールを開く場合**、実際の場所を最もよく表しているリージョンを選択します。 

  ![(テキスト - 選別) ページでのリージョンの選択](images/test-drive-region.png)

  **[Text - Screen]** \(テキスト - 選別) API コンソールが開きます。

## <a name="select-the-inputs"></a>入力を選択する

### <a name="parameters"></a>パラメーター

テキスト画面で使用するクエリ パラメーターを選択します。 この例の場合、**language** には既定値を使用します。 この操作では実行の一環として、可能性の高い言語が自動的に検出されるため、これは空白のままにしておくこともできます。

> [!NOTE]
> **language** パラメーターについては、`eng` を割り当てるか、パラメーターを空のままにして、コンピューター支援による**分類**応答を表示します (プレビュー機能)。 **この機能でサポートされているのは英語のみです**。
>
> **不適切用語**の検出については、この記事の一覧に示したサポート対象言語の [ISO 639-3 コード](http://www-01.sil.org/iso639-3/codes.asp)を使用するか、空のままにします。

**[autocorrect]** 、 **[PII]** 、および **[classify]** (プレビュー) については、 **[true]** を選択します。 **[ListId]** フィールドは空白のままにしておきます。

  ![[Text - Screen]\(テキスト - 選別) コンソールのクエリ パラメーター](images/text-api-console-inputs.PNG)

### <a name="content-type"></a>Content type

**[Content-type]** については、選別するコンテンツの種類を選択します。 この例では、コンテンツの種類として既定値の **[text/plain]** を使用します。 **[Ocp-Apim-Subscription-Key]** ボックスにサブスクリプション キーを入力します。

### <a name="sample-text-to-scan"></a>スキャンするサンプル テキスト

**[Request body]** (要求本文) ボックスに、いくらかのテキストを入力します。 次の例は、テキスト内の意図的なタイプミスを示しています。

```
Is this a grabage or crap email abcdef@abcd.com, phone: 4255550111, IP: 255.255.255.255, 1234 Main Boulevard, Panapolis WA 96555. These are all UK phone numbers, the last two being Microsoft UK support numbers: +44 870 608 4000 or 0344 800 2400 or 0800 820 3300. Also, 999-99-9999 looks like a social security number (SSN).
```

## <a name="analyze-the-response"></a>応答を分析する

次の応答は、API から返されたさまざまな分析情報を示しています。 不適切である可能性のある表現、個人データ、分類 (プレビュー)、および自動修正済みのバージョンが含まれています。

> [!NOTE]
> コンピューター支援型の "分類" 機能はプレビュー段階にあり、英語のみをサポートしています。

```json
{"OriginalText":"Is this a grabage or crap email abcdef@abcd.com, phone: 4255550111, IP: 255.255.255.255, 1234 Main Boulevard, Panapolis WA 96555.\r\nThese are all UK phone numbers: +44 123 456 7890 or 0234 567 8901 or 0456 789 0123.\r\nAlso, 999-99-9999 looks like a social security number (SSN).",
"NormalizedText":"Is this a grabage or crap email abcdef@ abcd. com, phone: 4255550111, IP: 255. 255. 255. 255, 1234 Main Boulevard, Panapolis WA 96555. \r\nThese are all UK phone numbers: +44 123 456 7890 or 0234 567 8901 or 0456 789 0123. \r\nAlso, 999- 99- 9999 looks like a social security number ( SSN) .",
"Misrepresentation":null,
"PII":{  
  "Email":[  
    {  
      "Detected":"abcdef@abcd.com",
      "SubType":"Regular",
      "Text":"abcdef@abcd.com",
      "Index":32
    }
  ],
  "IPA":[  
    {  
      "SubType":"IPV4",
      "Text":"255.255.255.255",
      "Index":72
    }
  ],
  "Phone":[  
    {  
      "CountryCode":"US",
      "Text":"4255550111",
      "Index":56
    },
    {  
      "CountryCode":"US",
      "Text":"425 555 0111",
      "Index":211
    },
    {  
      "CountryCode":"UK",
      "Text":"+44 123 456 7890",
      "Index":207
    },
    {  
      "CountryCode":"UK",
      "Text":"0234 567 8901",
      "Index":227
    },
    {  
      "CountryCode":"UK",
      "Text":"0456 789 0123",
      "Index":244
    }
  ],
  "Address":[  
    {  
      "Text":"1234 Main Boulevard, Panapolis WA 96555",
      "Index":89
    }
  ],
  "SSN":[  
    {  
      "Text":"999999999",
      "Index":56
    },
    {  
      "Text":"999-99-9999",
      "Index":266
    }
  ]
},
"Classification":{  
  "ReviewRecommended":true,
  "Category1":{  
    "Score":1.5113095059859916E-06
  },
  "Category2":{  
    "Score":0.12747249007225037
  },
  "Category3":{  
    "Score":0.98799997568130493
  }
},
"Language":"eng",
"Terms":[  
  {  
    "Index":21,
    "OriginalIndex":21,
    "ListId":0,
    "Term":"crap"
  }
],
"Status":{  
  "Code":3000,
  "Description":"OK",
  "Exception":null
},
"TrackingId":"2eaa012f-1604-4e36-a8d7-cc34b14ebcb4"
}
```

JSON 応答に含まれるすべてのセクションの詳細説明については、[テキスト モデレート](text-moderation-api.md)に関する概念ガイドを参照してください。

## <a name="next-steps"></a>次のステップ

コードで REST API を使用するか、[.NET SDK のクイックスタート](dotnet-sdk-quickstart.md)に従って、アプリケーションと統合します。
