---
title: DTU から仮想コアに移行する
description: DTU モデルから仮想コア モデルに移行します。 仮想コアへの移行は、Standard レベルと Premium レベルの間でのアップグレードまたはダウングレードに似ています。
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: sashan, moslake, carlrab
ms.date: 03/09/2020
ms.openlocfilehash: 693065046f92e0e9eade14c43e9942772440937d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "78945396"
---
# <a name="migrate-from-the-dtu-based-model-to-the-vcore-based-model"></a>DTU ベースのモデルから仮想コア ベースのモデルに移行する

## <a name="migrate-a-database"></a>データベースの移行

DTU ベースの購入モデルから仮想コアベースの購入モデルへのデータベースの移行は、DTU ベースの購入モデルでの Standard および Premium サービス レベル間のアップグレードまたはダウングレードに似ています。

## <a name="migrate-databases-that-use-geo-replication"></a>geo レプリケーションを使用するデータベースの移行

DTU ベースのモデルから仮想コアベースの購入モデルへの移行は、Standard および Premium サービス レベルでのデータベース間の geo レプリケーションのリレーションシップのアップグレードまたはダウングレードに似ています。 移行中に geo レプリケーションを停止する必要はありませんが、次のシーケンス処理のルールに従う必要があります。

- アップグレードの場合は、最初にセカンダリ データベースをアップグレードしてから、プライマリをアップグレードする必要があります。
- ダウングレードは逆の順序で行います。つまり、最初にプライマリ データベースをダウングレードしてから、セカンダリをダウングレードします。

2 つのエラスティック プール間で geo レプリケーションを使用している場合は、1 つのプールをプライマリとして、もう一方をセカンダリとして指定することをお勧めします。 その場合、エラスティック プールを移行するときは、同じシーケンス処理のガイダンスを使用する必要があります。 ただし、プライマリ データベースとセカンダリ データベースの両方を含むエラスティック プールがある場合は、使用率が高い方のプールをプライマリとして扱い、それに応じてシーケンス処理のルールに従ってください。  

次の表は、特定の移行シナリオのためのガイダンスを示しています。

|現在のサービス レベル|移行先のサービス レベル|移行の種類|ユーザー操作|
|---|---|---|---|
|Standard|汎用|ラテラル|任意の順序で移行できますが、適切な仮想コア サイズを確保する必要があります*|
|Premium|Business Critical|ラテラル|任意の順序で移行できますが、適切な仮想コア サイズを確保する必要があります*|
|Standard|Business Critical|アップグレード|セカンダリを最初に移行する必要があります|
|Business Critical|Standard|ダウングレード|プライマリを最初に移行する必要があります|
|Premium|汎用|ダウングレード|プライマリを最初に移行する必要があります|
|汎用|Premium|アップグレード|セカンダリを最初に移行する必要があります|
|Business Critical|汎用|ダウングレード|プライマリを最初に移行する必要があります|
|汎用|Business Critical|アップグレード|セカンダリを最初に移行する必要があります|
||||

\* 原則として、Standard レベルでは 100 DTU ごとに少なくとも 1 つの仮想コアが必要であり、Premium レベルでは 125 DTU ごとに少なくとも 1 つの仮想コアが必要です。 詳細については、「[仮想コアベースの購入モデル](https://docs.microsoft.com/azure/sql-database/sql-database-purchase-models#vcore-based-purchasing-model)」をご覧ください。

## <a name="migrate-failover-groups"></a>フェールオーバー グループを移行する

複数のデータベースが含まれるフェールオーバー グループの移行では、プライマリ データベースとセカンダリ データベースを個別に移行する必要があります。 その処理中は、同じ考慮事項とシーケンス処理ルールが適用されます。 データベースが仮想コアベースの購入モデルに変換された後も、フェールオーバー グループでは同じポリシー設定が引き続き有効です。

### <a name="create-a-geo-replication-secondary-database"></a>geo レプリケーションのセカンダリ データベースを作成する

geo レプリケーションのセカンダリ データベース (geo セカンダリ) は、プライマリ データベースに対して使用したのと同じサービス レベルを使用してのみ作成できます。 ログ生成速度が速いデータベースの場合は、プライマリと同じコンピューティング サイズを持つ geo セカンダリを作成することをお勧めします。

geo セカンダリを単一プライマリ データベースのエラスティック プール内に作成している場合は、そのプールの `maxVCore` 設定がプライマリ データベースのコンピューティング サイズに一致していることを確認してください。 プライマリの geo セカンダリを別のエラスティック プール内に作成している場合は、そのプールに同じ `maxVCore` 設定を割り当てることをお勧めします。

## <a name="use-database-copy-to-convert-a-dtu-based-database-to-a-vcore-based-database"></a>データベースのコピーを使用して DTU ベースのデータベースを仮想コア ベースのデータベースに変換する

DTU ベース コンピューティング サイズのデータベースを、仮想コアベース コンピューティング サイズのデータベースにコピーする場合、コピー先のコンピューティング サイズが、コピー元データベースの最大データベース サイズをサポートしている限り、制限や特別なシーケンス処理は伴いません。 データベースのコピーでは、コピー操作が開始された時点のデータのスナップショットが作成され、ソースとターゲットの間でデータは同期されません。

## <a name="next-steps"></a>次のステップ

- 単一データベースに対して使用できる特定のコンピューティング サイズとストレージ サイズの選択については、[単一データベースに対する SQL Database の仮想コア ベースのリソース制限](sql-database-vcore-resource-limits-single-databases.md)に関するページを参照してください。
- エラスティック プールに対して使用できる特定のコンピューティング サイズとストレージ サイズの選択については、[エラスティック プールに対する SQL Database の仮想コア ベースのリソース制限](sql-database-vcore-resource-limits-elastic-pools.md)に関するページを参照してください。
