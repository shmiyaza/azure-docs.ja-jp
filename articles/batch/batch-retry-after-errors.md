---
title: エラー後のタスクの再試行
description: エラーを確認し、それらを再試行します。
services: batch
documentationcenter: .net
author: LauraBrenner
manager: evansma
editor: ''
tags: azure-batch
ms.assetid: 16279b23-60ff-4b16-b308-5de000e4c028
ms.service: batch
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 02/15/2020
ms.author: labrenne
ms.custom: seodec18
ms.openlocfilehash: 94ed936e619461a2dbf7ec837c2d80e21c01c88e
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "77919993"
---
# <a name="detecting-and-handling-batch-service-errors"></a>Batch サービス エラーの検出と処理

REST サービス API を使用する場合は、忘れずにエラーを確認することが重要です。 Batch ジョブの実行では、よくエラーが発生します。

## <a name="common-errors"></a>一般的なエラー 

- ネットワーク エラー: これらは、Batch に到達しなかった要求であるか、または Batch 応答が時間内にクライアントに到達しなかった要求です。
- 内部サーバー エラー: これらは、標準の 5xx の状態コードの HTTP 応答です。
- 調整によって、Retry-after ヘッダー付きの 429 や 503 の状態コードの HTTP 応答などのエラーが発生する可能性があります。
- AlreadyExists と InvalidOperation などのエラーを含む 4xx エラーです。 これは、状態遷移でリソースが正しい状態にないことを意味します。

さまざまな種類のエラー コードと特定のエラー コードの詳細については、[バッチの状態とエラー コード](https://docs.microsoft.com/rest/api/batchservice/batch-status-and-error-codes)に関するページを参照してください。

## <a name="when-to-retry"></a>再試行のタイミング

Batch API では、発生したエラーが通知されます。 これらはすべて再試行でき、これらにはすべてその目的用のグローバル再試行ハンドラーが含まれています。 この組み込みのメカニズムを使用することをお勧めします。

エラーが発生した場合、少し待ってから再試行してください (次の再試行まで数秒待機します)。 再試行の回数が多すぎる場合、または再試行が早すぎる場合は、再試行ハンドラーは調整されます。

### <a name="for-more-information"></a>詳細情報  

[Batch API とツール](batch-apis-tools.md)に関するページには、API リファレンス情報へのリンクが掲載されています。 たとえば、.NET API には、必要な再試行ポリシーを指定する必要がある [RetryPolicyProvider クラス]( https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.retrypolicyprovider?view=azure-dotnet)があります。 

各 API とその既定の再試行ポリシーの詳細については、[バッチの状態とエラー コード](https://docs.microsoft.com/rest/api/batchservice/batch-status-and-error-codes)に関するページを参照してください。
