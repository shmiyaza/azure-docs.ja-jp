---
title: チュートリアル - アクセス パッケージを作成する - Azure AD エンタイトルメント管理
description: Azure Active Directory エンタイトルメント管理で最初のアクセス パッケージを作成する方法に関するステップ バイ ステップのチュートリアル。
services: active-directory
documentationCenter: ''
author: msaburnley
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.subservice: compliance
ms.date: 10/22/2019
ms.author: ajburnle
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: cd4feeb83acc3842874e7a2e4bbd32dacabcc00d
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/24/2020
ms.locfileid: "75422652"
---
# <a name="tutorial-create-your-first-access-package-in-azure-ad-entitlement-management"></a>チュートリアル:Azure AD エンタイトルメント管理で最初のアクセス パッケージを作成する

グループ、アプリケーション、サイトなど、従業員が必要とするすべてのリソースに対するアクセスを管理することは、組織にとって重要な機能の 1 つです。 生産性を維持するために必要とされる適切なレベルのアクセスを従業員に付与し、不要になったアクセスは削除する必要があります。

このチュートリアルでは、あなたは Woodgrove Bank の IT 管理者として勤務しています。 内部ユーザーがセルフサービスの要求を実行できる、マーケティング キャンペーン用のリソースのパッケージを作成するよう求められました。 要求は承認を必要とせず、ユーザーのアクセスは 30 日後に有効期限が切れます。 このチュートリアルでは、マーケティング キャンペーンのリソースは 1 つのグループ内のメンバーシップにすぎませんが、グループ、アプリケーション、または SharePoint Online サイトのコレクションである場合もあります。

![シナリオの概要](./media/entitlement-management-access-package-first/elm-scenario-overview.png)

このチュートリアルでは、以下の内容を学習します。

> [!div class="checklist"]
> * グループをリソースとしてアクセス パッケージを作成する
> * ディレクトリ内のユーザーがアクセスを要求することを許可する
> * 内部ユーザーがどのようにしてアクセス パッケージを要求できるかを示す

最初のアクセス パッケージの作成など、Azure Active Directory のエンタイトルメント管理をデプロイするプロセスのステップバイステップのデモンストレーションについては、次のビデオを参照してください。

>[!VIDEO https://www.youtube.com/embed/zaaKvaaYwI4]

## <a name="prerequisites"></a>前提条件

Azure AD エンタイトルメント管理を使用するには、次のいずれかのライセンスが必要です。

- Azure AD Premium P2
- Enterprise Mobility + Security (EMS) E5 ライセンス

詳細については、「[License requirements ライセンスの要件](entitlement-management-overview.md#license-requirements)」を参照してください。

## <a name="step-1-set-up-users-and-group"></a>手順 1:ユーザーとグループを設定する

リソース ディレクトリには、共有する 1 つ以上のリソースがあります。 この手順では、Woodgrove Bank ディレクトリ内に、エンタイトルメント管理のターゲット リソースである **Marketing resources** という名前のグループを作成します。 また、内部要求者も設定します。

**事前に必要なロール:** グローバル管理者またはユーザー管理者

![ユーザーとグループを作成する](./media/entitlement-management-access-package-first/elm-users-groups.png)

1. [Azure portal](https://portal.azure.com) にグローバル管理者またはユーザー管理者としてサインインします。  

1. 左側のナビゲーションで、 **[Azure Active Directory]** をクリックします。

1. 次の 2 人のユーザーを作成または構成します。 以下の名前を使用することも、別の名前を使用することもできます。 **Admin1** は、現在サインイン済みのユーザーで構いません。

    | 名前 | ディレクトリ ロール |
    | --- | --- |
    | **Admin1** | 全体管理者<br/>または<br/>ユーザー管理者 |
    | **Requestor1** | User |

1. メンバーシップの種類が **[割り当て済み]** である **Marketing resources** という名前の Azure AD セキュリティ グループを作成します。

    このグループは、エンタイトルメント管理のターゲット リソースになります。 開始するグループのメンバーは空である必要があります。

## <a name="step-2-create-an-access-package"></a>手順 2:アクセス パッケージを作成します。

*アクセス パッケージ*は、チームまたはプロジェクトが必要とし、ポリシーで管理されるリソースのバンドルです。 アクセス パッケージは、*カタログ*と呼ばれるコンテナーに定義されます。 この手順では、 **[標準]** カタログ内に **Marketing Campaign** アクセス パッケージを作成します。

**事前に必要なロール:** グローバル管理者、ユーザー管理者、カタログ所有者、またはアクセス パッケージ マネージャー

![アクセス パッケージを作成します。](./media/entitlement-management-access-package-first/elm-access-package.png)

1. Azure portal の左側のナビゲーションで、 **[Azure Active Directory]** をクリックします。

1. 左側のメニューで、 **[Identity Governance]** をクリックします

1. 左側のメニューで **[アクセス パッケージ]** をクリックします。  **[アクセスが拒否されました]** と表示される場合は、Azure AD Premium P2 ライセンスがそのディレクトリに存在することを確認してください。

1. **[New access package]\(新しいアクセス パッケージ\)** をクリックします。

    ![Azure portal でのエンタイトルメント管理](./media/entitlement-management-shared/access-packages-list.png)

1. **[基本]** タブで、「**Marketing Campaign**」というアクセス パッケージの名前と「**キャンペーン用のリソースへのアクセス**」という説明を入力します。

1. **[カタログ]** ドロップダウン リストは **[標準]** に設定したままにします。

    ![[New access package]\(新しいアクセス パッケージ\) - [基本] タブ](./media/entitlement-management-access-package-first/basics.png)

1. **[次へ]** をクリックして、 **[Resource roles]\(リソース ロール\)** タブを開きます。

    このタブでは、アクセス パッケージに含めるリソースとリソース ロールを選択します。

1. **[Groups and Teams] (グループとチーム)** をクリックします。

1. [グループの選択] ウィンドウで、前に作成した **Marketing resources** グループを見つけて選択します。

    既定では、 **[標準]** カタログ内外のグループが表示されます。 **[標準]** カタログ外のグループを選択すると、 **[標準]** カタログに追加されます。

    ![[New access package]\(新しいアクセス パッケージ\) - [Resource roles]\(リソース ロール\) タブ](./media/entitlement-management-access-package-first/resource-roles-select-groups.png)

1. **[選択]** をクリックして、そのグループを一覧に追加します。

1. **[ロール]** ドロップダウン リストで **[メンバー]** を選択します。

    ![[New access package]\(新しいアクセス パッケージ\) - [Resource roles]\(リソース ロール\) タブ](./media/entitlement-management-access-package-first/resource-roles.png)

1. **[次へ]** をクリックして、 **[要求]** タブを開きます。

    このタブでは、要求ポリシーを作成します。 *ポリシー*により、アクセス パッケージにアクセスするための規則またはガードレールを定義します。 リソース ディレクトリ内の特定のユーザーがこのアクセス パッケージを要求することを許可するポリシーを作成します。

1. **[アクセス権を要求できるユーザー]** セクションで、 **[ディレクトリ内のユーザーの場合]** 、 **[特定のユーザーとグループ]** の順にクリックします。

    ![[New access package] (新しいアクセス パッケージ) - [要求] タブ](./media/entitlement-management-access-package-first/requests.png)

1. **[ユーザーとグループを追加する]** をクリックします。

1. [ユーザーとグループを選択] ウィンドウで、前に作成した **Requestor1** ユーザーを選択します。

    ![[New access package] (新しいアクセス パッケージ) - [要求] タブ - [ユーザーとグループを選択]](./media/entitlement-management-access-package-first/requests-select-users-groups.png)

1. **[選択]** をクリックします。

1. **[承認]** と **[要求の有効化]** のセクションまで下へスクロールします。

1. **[承認を要求する]** を **[いいえ]** に設定されたのままにします。

1. **[要求の有効化]** については、 **[はい]** をクリックして、このアクセス パッケージを作成されたらすぐに要求できるようにします。

    ![[New access package] (新しいアクセス パッケージ) - [要求] タブ - [承認] と [要求の有効化]](./media/entitlement-management-access-package-first/requests-approval-enable.png)

1. **[次へ]** をクリックして **[ライフサイクル]** タブを開きます。

1. **[有効期限]** セクションで、 **[Access package assignments expire] (アクセス パッケージの割り当ての有効期限)** を **[日数]** を設定します。

1. **[Assignments expire after] (割り当ての有効期限)** を **[30]** 日に設定します。

    ![[New access package] (新しいアクセス パッケージ) - [ライフサイクル] タブ](./media/entitlement-management-access-package-first/lifecycle.png)

1. **[次へ]** をクリックして、 **[Review + Create]\(確認と作成\)** タブを開きます。

    ![[New access package]\(新しいアクセス パッケージ\) - [Review + Create]\(確認と作成\) タブ](./media/entitlement-management-access-package-first/review-create.png)

    しばらくすると、アクセス パッケージが正常に作成されたという通知が表示されます。

1. Marketing Campaign アクセス パッケージの左側のメニューで、 **[概要]** をクリックします。

1. **[My Access portal link]\(マイ アクセス ポータルのリンク\)** をコピーします。

    このリンクは次の手順で使用します。

    ![アクセス パッケージの概要 - マイ アクセス ポータルのリンク](./media/entitlement-management-shared/my-access-portal-link.png)

## <a name="step-3-request-access"></a>手順 3:アクセスの要求

この手順では、**内部要求者**として手順を実行し、アクセス パッケージへのアクセスを要求します。 要求者は、マイ アクセス ポータルと呼ばれるサイトを使用して要求を送信します。 マイ アクセス ポータルを使用すると、要求者は、アクセス パッケージの要求の送信、既にアクセス権を持つアクセス パッケージの参照、要求の履歴の表示ができます。

**事前に必要なロール:** 内部要求者

1. Azure portal からサインアウトします。

1. 新しいブラウザー ウィンドウで、前の手順でコピーしたマイ アクセス ポータルのリンクに移動します。

1. マイ アクセス ポータルに **Requestor1** としてサインインします。

    **Marketing Campaign** アクセス パッケージが表示されます。

1. 必要に応じて **[説明]** 列で矢印をクリックして、アクセス パッケージに関する詳細を表示します。

    ![マイ アクセス ポータル - [アクセス パッケージ]](./media/entitlement-management-shared/my-access-access-packages.png)

1. チェックマークをクリックして、このパッケージを選択します。

1. **[アクセスの要求]** をクリックして、[アクセスの要求] ウィンドウを開きます。

    ![マイ アクセス ポータル - [アクセスの要求] ボタン](./media/entitlement-management-access-package-first/my-access-request-access-button.png)

1. **[業務上の正当な理由]** ボックスに、正当な理由として「**新しいマーケティング キャンペーンで作業している**」と入力します。

    ![マイ アクセス ポータル - [アクセスの要求]](./media/entitlement-management-shared/my-access-request-access.png)

1. **[送信]** をクリックします。

1. 左側のメニューで **[Request history]\(要求の履歴\)** をクリックして、要求が送信されたことを確認します。

## <a name="step-4-validate-that-access-has-been-assigned"></a>手順 4:アクセス権が割り当てられていることを検証する

この手順では、**内部要求者**にアクセス パッケージが割り当てられ、そのユーザーが **Marketing resources** グループのメンバーになっていることを確認します。

**事前に必要なロール:** グローバル管理者、ユーザー管理者、カタログ所有者、またはアクセス パッケージ マネージャー

1. マイ アクセス ポータルからサインアウトします。

1. [Azure portal](https://portal.azure.com) に **Admin1** としてサインインします。

1. **[Azure Active Directory]** をクリックしてから **[Identity Governance]** をクリックします。

1. 左側のメニューで **[アクセス パッケージ]** をクリックします。

1. **Marketing Campaign** アクセス パッケージを見つけてクリックします。

1. 左側のメニューで **[要求]** をクリックします。

    Requestor1 と初期のポリシーが **[配信済み]** の状態で表示されます。

1. この要求をクリックして、要求の詳細を表示します。

    ![アクセス パッケージ - 要求の詳細](./media/entitlement-management-access-package-first/request-details.png)

1. 左側のナビゲーションで、 **[Azure Active Directory]** をクリックします。

1. **[グループ]** をクリックし、**Marketing resources** グループを開きます。

1. **[メンバー]** をクリックします。

    **Requestor1** がメンバーとして一覧表示されます。

    ![Marketing resources のメンバー](./media/entitlement-management-access-package-first/group-members.png)

## <a name="step-5-clean-up-resources"></a>手順 5:リソースをクリーンアップする

この手順では、行った変更を削除し、**Marketing Campaign** アクセス パッケージを削除します。

**事前に必要なロール:** グローバル管理者またはユーザー管理者

1. Azure portal で **[Azure Active Directory]** をクリックし、 **[Identity Governance]** をクリックします。

1. **Marketing Campaign** アクセス パッケージを開きます。

1. **[割り当て]** をクリックします。

1. **Requestor1** について、省略記号 ( **...** ) をクリックし、 **[Remove access]\(アクセスの削除\)** をクリックします。 表示されるメッセージで、 **[はい]** をクリックします。

    しばらくすると、状態が [配信済み] から [有効期限切れ] に変化します。

1. **[Resource roles]\(リソース ロール\)** をクリックします。

1. **Marketing resources** については、省略記号 ( **...** ) をクリックし、 **[Remove resource role] (リソース ロールの削除)** をクリックします。 表示されるメッセージで、 **[はい]** をクリックします。

1. アクセス パッケージの一覧を開きます。

1. **Marketing Campaign** については、省略記号 ( **...** ) をクリックし、 **[削除]** をクリックします。 表示されるメッセージで、 **[はい]** をクリックします。

1. Azure Active Directory で、**Requestor1**、**Admin1** など、作成したユーザーをすべて削除します。

1. **Marketing resources** グループを削除します。

## <a name="next-steps"></a>次のステップ

次の記事に進み、エンタイトルメント管理の一般的なシナリオの手順を確認してください。
> [!div class="nextstepaction"]
> [一般的なシナリオ](entitlement-management-scenarios.md)
