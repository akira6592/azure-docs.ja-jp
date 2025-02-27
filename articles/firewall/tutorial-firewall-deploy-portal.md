---
title: Azure portal を使用して Azure Firewall のデプロイと構成を行う
description: この記事では、Azure portal を使用して Azure Firewall をデプロイおよび構成する方法について説明します。
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 11/10/2021
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 99b7d27ef16414df161b7d6e120084f2b8220f46
ms.sourcegitcommit: 677e8acc9a2e8b842e4aef4472599f9264e989e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/11/2021
ms.locfileid: "132297527"
---
# <a name="deploy-and-configure-azure-firewall-using-the-azure-portal"></a>Azure portal を使用して Azure Firewall をデプロイして構成する

アウトバウンド ネットワーク アクセスを制御することは、ネットワーク セキュリティ プラン全体の重要な要素です。 たとえば、Web サイトへのアクセスを制限することができます。 また、アクセスできるアウトバウンドの IP アドレスとポートを制限することもできます。

Azure サブネットから外に向かうアウトバウンド ネットワーク アクセスを制御する方法の 1 つとして、Azure Firewall の使用が挙げられます。 Azure Firewall を使用して、次のルールを構成できます。

* アプリケーション ルール: サブネットからアクセスできる完全修飾ドメイン名 (FQDN) を定義します。
* ネットワーク ルール: 送信元アドレス、プロトコル、宛先ポート、送信先アドレスを定義します。

ネットワーク トラフィックは、サブネットの既定ゲートウェイとしてのファイアウォールにルーティングしたときに、構成されているファイアウォール ルールに制約されます。

この記事では、簡単にデプロイできるように、2 つのサブネットを含む単純な単一の VNet を作成します。

運用環境のデプロイでは、[ハブとスポーク モデル](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)を採用して、独自の VNet にファイアウォールを配置することをお勧めします。 ワークロード サーバーは、1 つ以上のサブネットを含む同じリージョンのピアリングされた VNet に配置されます。

* **AzureFirewallSubnet** - このサブネットにファイアウォールが存在します。
* **Workload-SN** - このサブネットにはワークロード サーバーがあります。 このサブネットのネットワーク トラフィックは、ファイアウォールを通過します。

![ネットワーク インフラストラクチャ](media/tutorial-firewall-deploy-portal/tutorial-network.png)

この記事では、次の方法について説明します。

> [!div class="checklist"]
> * テスト ネットワーク環境を設定する
> * ファイアウォールをデプロイする
> * 既定のルートを作成する
> * www.google.com へのアクセスを許可するようにアプリケーションを構成する
> * 外部 DNS サーバーへのアクセスを許可するようにネットワーク ルールを構成する
> * テスト サーバーへのリモート デスクトップ接続を許可するように NAT 規則を構成する
> * ファイアウォールをテストする

> [!NOTE]
> この記事では、従来のファイアウォール規則を使用してファイアウォールを管理します。 推奨される方法は、[ファイアウォール ポリシー](../firewall-manager/policy-overview.md)を使用することです。 ファイアウォール ポリシーを使用してこの手順を実行するには、「[チュートリアル: Azure portal を使用して Azure Firewall とポリシーをデプロイおよび構成する](tutorial-firewall-deploy-portal-policy.md)」をご覧ください。

好みに応じて、[Azure PowerShell](deploy-ps.md) を使ってこの手順を実行することもできます。

## <a name="prerequisites"></a>前提条件

Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

## <a name="set-up-the-network"></a>ネットワークのセットアップ

最初に、ファイアウォールをデプロイするために必要なリソースを含めるリソース グループを作成します。 次に、VNet、サブネット、およびテスト サーバーを作成します。

### <a name="create-a-resource-group"></a>リソース グループを作成する

このリソース グループには、この手順で使用するすべてのリソースが含まれます。

1. Azure Portal [https://portal.azure.com](https://portal.azure.com) にサインインします。
2. Azure portal メニューで **[リソース グループ]** を選択するか、または任意のページから *[リソース グループ]* を検索して選択します。 その後、 **[追加]** を選択します。
4. **[サブスクリプション]** で、ご使用のサブスクリプションを選択します。
1. **[リソース グループ名]** として「*Test-FW-RG*」と入力します。
1. **[リソース グループの場所]** で、場所を選択します。 作成する他のすべてのリソースは同じ場所にある必要があります。
1. **[Review + create]\(レビュー + 作成\)** を選択します。
1. **［作成］** を選択します

### <a name="create-a-vnet"></a>VNet を作成する

この VNet には 3 つのサブネットが含まれます。

> [!NOTE]
> AzureFirewallSubnet サブネットのサイズは /26 です。 サブネットのサイズの詳細については、「[Azure Firewall に関する FAQ](firewall-faq.yml#why-does-azure-firewall-need-a--26-subnet-size)」を参照してください。

1. Azure portal メニュー上または **[ホーム]** ページから **[リソースの作成]** を選択します。
1. **[ネットワーク]**  >  **[仮想ネットワーク]** を選びます。
1. **［作成］** を選択します
1. **[サブスクリプション]** で、ご使用のサブスクリプションを選択します。
1. **[リソース グループ]** で、 **[Test-FW-RG]** を選択します。
1. **[名前]** に「**Test-FW-VN**」と入力します。
1. **[リージョン]** で、以前使用したのと同じ場所を選択します。
1. **Next:次へ: IP アドレス** を選択します。
1. **[IPv4 アドレス空間]** に、「**10.0.0.0/16**」と入力します。
1. **[サブネット]** で、 **[default]\(既定\)** を選択します。
1. **[サブネット名]** に「**AzureFirewallSubnet**」と入力します。 ファイアウォールはこのサブネットに配置されます。サブネット名は AzureFirewallSubnet **でなければなりません**。
1. **[アドレス範囲]** に「**10.0.1.0/26**」と入力します。
1. **[保存]** を選択します。

   次に、ワークロード サーバーのサブネットを作成します。

1. **[サブネットの追加]** を選択します。
4. **[サブネット名]** に「**Workload-SN**」と入力します。
5. **[サブネット アドレス範囲]** に「**10.0.2.0/24**」と入力します。
6. **[追加]** を選択します。
7. **[確認および作成]** を選択します。
8. **［作成］** を選択します

### <a name="create-a-virtual-machine"></a>仮想マシンの作成

ワークロード仮想マシンを作成し、**Workload-SN** サブネットに配置します。

1. Azure portal メニュー上または **[ホーム]** ページから **[リソースの作成]** を選択します。
2. **[Windows Server 2016 Datacenter]** を選択します。
4. 次の仮想マシンの値を入力します。

   |設定  |値  |
   |---------|---------|
   |Resource group     |**Test-FW-RG**|
   |仮想マシン名     |**Srv-Work**|
   |リージョン     |前と同じ|
   |Image|Windows Server 2016 Datacenter|
   |管理者のユーザー名     |ユーザー名を入力します|
   |Password     |パスワードを入力します|

4. **[受信ポートの規則]** の **[パブリック受信ポート]** で、 **[なし]** を選択します。
6. 他の既定値をそのまま使用し、 **[次へ:ディスク]** を選択します。
7. ディスクの既定値をそのまま使用し、 **[次へ:ネットワーク]** を選択します。
8. 仮想ネットワークとして **Test-FW-VN** が選択されていること、およびサブネットが **Workload-SN** であることを確認します。
9. **[パブリック IP]** で、 **[なし]** を選択します。
11. 他の既定値をそのまま使用し、 **[次へ:管理]** を選択します。
12. **[無効]** を選択して、ブート診断を無効にします。 他の既定値をそのまま使用し、 **[確認および作成]** を選択します。
13. 概要ページの設定を確認して、 **[作成]** を選択します。

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="deploy-the-firewall"></a>ファイアウォールをデプロイする

VNet にファイアウォールをデプロイします。

1. Azure portal メニュー上または **[ホーム]** ページから **[リソースの作成]** を選択します。
2. 検索ボックスに「**ファイアウォール**」と入力し、**Enter** キーを押します。
3. **[ファイアウォール]** を選択し、 **[作成]** を選択します。
4. **[ファイアウォールの作成]** ページで、次の表を使用してファイアウォールを構成します。

   |設定  |値  |
   |---------|---------|
   |サブスクリプション     |\<your subscription\>|
   |Resource group     |**Test-FW-RG** |
   |名前     |**Test-FW01**|
   |リージョン     |以前使用したのと同じ場所を選択します|
   |ファイアウォール管理|**ファイアウォール規則 (クラシック) を使用してこのファイアウォールを管理する**|
   |仮想ネットワークの選択     |**[Use Existing]\(既存の使用\)** : **Test-FW-VN**|
   |パブリック IP アドレス     |**[新規追加]**<br>**[名前]** : **fw-pip**|

5. 他の既定値をそのまま使用し、 **[確認および作成]** を選択します。
6. 概要を確認し、 **[作成]** を選択してファイアウォールを作成します。

   デプロイが完了するまでに数分かかります。
7. デプロイが完了したら、**Test-FW-RG** リソース グループに移動し、**Test-FW01** ファイアウォールを選択します。
8. ファイアウォールのプライベートおよびパブリック IP アドレスをメモします。 これらのアドレスは後ほど使用します。

## <a name="create-a-default-route"></a>既定のルートを作成する

ファイアウォール経由で発信および受信接続のルートを作成する場合は、仮想アプライアンスのプライベート IP をネクスト ホップとした 0.0.0.0/0 への既定のルートがあれば十分です。 これにより、ファイアウォールを通過する発信および着信接続が処理されます。 たとえば、ファイアウォールで TCP ハンドシェイクが実行され、受信要求への応答が行われる場合、応答はトラフィックの送信元の IP アドレスに送信されます。 これは仕様です。 

このため、AzureFirewallSubnet の IP 範囲を含めるための追加の UDR を作成する必要はありません。 これにより、接続が切断される可能性があります。 元の既定のルートで十分です。

**Workload-SN** サブネットでは、アウトバウンドの既定ルートがファイアウォールを通過するように構成します。

1. Azure portal メニューで **[すべてのサービス]** を選択するか、または任意のページから *[すべてのサービス]* を検索して選択します。
2. **[ネットワーク]** で、 **[ルート テーブル]** を選択します。
3. **[追加]** を選択します。
5. **[サブスクリプション]** で、ご使用のサブスクリプションを選択します。
6. **[リソース グループ]** で、 **[Test-FW-RG]** を選択します。
7. **[リージョン]** で、以前使用したのと同じ場所を選択します。
4. **[名前]** に「**Firewall-route**」と入力します。
1. **[Review + create]\(レビュー + 作成\)** を選択します。
1. **［作成］** を選択します

デプロイが完了したら、**[リソースに移動]** を選択します。

1. [Firewall-route] ページで、 **[サブネット]** を選択し、 **[関連付け]** を選択します。
1. **[仮想ネットワーク]**  >  **[Test-FW-VN]** の順に選択します。
1. **[サブネット]** で、 **[Workload-SN]** を選択します。 必ずこのルートの **Workload-SN** サブネットのみを選択してください。それ以外の場合、ファイアウォールが正常に動作しません。

13. **[OK]** を選択します。
14. **[ルート]** 、 **[追加]** の順に選択します。
15. **[ルート名]** に「**fw-dg**」と入力します。
16. **[アドレス プレフィックス]** に「**0.0.0.0/0**」と入力します。
17. **[次ホップの種類]** で、 **[仮想アプライアンス]** を選択します。

    Azure Firewall は実際はマネージド サービスですが、この状況では仮想アプライアンスが動作します。
18. **[次ホップ アドレス]** に、前にメモしておいたファイアウォールのプライベート IP アドレスを入力します。
19. **[OK]** を選択します。

## <a name="configure-an-application-rule"></a>アプリケーション ルールを構成する

これは、`www.google.com` へのアウトバウンド アクセスを許可するアプリケーション ルールです。

1. **Test-FW-RG** を開き、 **[Test-FW01]** ファイアウォールを選択します。
2. **Test-FW01** ページの **[設定]** で、 **[規則 (クラシック)]** を選択します。
3. **[アプリケーション ルール コレクション]** タブを選択します。
4. **[アプリケーション ルール コレクションの追加]** を選択します。
5. **[名前]** に「**App-Coll01**」と入力します。
6. **[優先度]** に「**200**」と入力します。
7. **[アクション]** で、 **[許可]** を選択します。
8. **[ルール]** の **[ターゲットの FQDN]** で、 **[名前]** に「**Allow-Google**」と入力します。
9. **[Source type]\(送信元の種類\)** で、 **[IP アドレス]** を選択します。
10. **[送信元]** に「**10.0.2.0/24**」と入力します。
11. **[プロトコル:ポート]** に「**http, https**」と入力します。
12. **[ターゲットの FQDN]** に「 **`www.google.com`** 」と入力します
13. **[追加]** を選択します。

Azure Firewall には、既定で許可されるインフラストラクチャ FQDN 用の組み込みのルール コレクションが含まれています。 これらの FQDN はプラットフォームに固有であり、他の目的には使用できません。 詳細については、[インフラストラクチャ FQDN](infrastructure-fqdns.md) に関する記事を参照してください。

## <a name="configure-a-network-rule"></a>ネットワーク ルールを構成する

これは、ポート 53 (DNS) で 2 つの IP アドレスへのアウトバウンド アクセスを許可するネットワーク ルールです。

1. **[ネットワーク ルール コレクション]** タブを選択します。
2. **[ネットワーク ルール コレクションの追加]** を選択します。
3. **[名前]** に「**Net-Coll01**」と入力します。
4. **[優先度]** に「**200**」と入力します。
5. **[アクション]** で、 **[許可]** を選択します。
6. **[ルール]** の **[IP アドレス]** で、 **[名前]** に「**Allow-DNS**」と入力します。
7. **[プロトコル]** で **[UDP]** を選択します。
9. **[Source type]\(送信元の種類\)** で、 **[IP アドレス]** を選択します。
1. **[送信元]** に「**10.0.2.0/24**」と入力します。
2. **[送信先の種類]** として **[IP アドレス]** を選択します。
3. **[送信先アドレス]** に「**209.244.0.3,209.244.0.4**」と入力します

   これらは、CenturyLink によって運用されるパブリック DNS サーバーです。
1. **[宛先ポート]** に「**53**」と入力します。
2. **[追加]** を選択します。

## <a name="configure-a-dnat-rule"></a>DNAT ルールを構成する

このルールを使用すると、ファイアウォールを介して、リモート デスクトップを Srv-Work 仮想マシンに接続できます。

1. **[NAT ルール コレクション]** タブを選択します。
2. **[NAT ルール コレクションの追加]** を選択します。
3. **[名前]** に「**rdp**」と入力します。
4. **[優先度]** に「**200**」と入力します。
5. **[ルール]** の **[名前]** に「**rdp-nat**」と入力します。
6. **[プロトコル]** で **[TCP]** を選択します。
7. **[Source type]\(送信元の種類\)** で、 **[IP アドレス]** を選択します。
8. **[送信元]** に「 **\*** 」と入力します。
9. **[宛先アドレス]** に、ファイアウォールのパブリック IP アドレスを入力します。
10. **[宛先ポート]** に「**3389**」と入力します。
11. **[変換されたアドレス]** に、**Srv-work** のプライベート IP アドレスを入力します。
12. **[Translated port] (変換されたポート)** に「**3389**」と入力します。
13. **[追加]** を選択します。


### <a name="change-the-primary-and-secondary-dns-address-for-the-srv-work-network-interface"></a>**Srv-Work** ネットワーク インターフェイスのプライマリおよびセカンダリ DNS アドレスを変更する

テストのために、サーバーのプライマリおよびセカンダリ DNS アドレスを構成します。 これは、一般的な Azure Firewall 要件ではありません。

1. Azure portal メニューで **[リソース グループ]** を選択するか、または任意のページから *[リソース グループ]* を検索して選択します。 **[Test-FW-RG]** リソース グループを選択します。
2. **Srv-Work** 仮想マシンのネットワーク インターフェイスを選択します。
3. **[設定]** で、 **[DNS サーバー]** を選択します。
4. **[DNS サーバー]** で、 **[カスタム]** を選択します。
5. **[DNS サーバーの追加]** テキスト ボックスに「**209.244.0.3**」と入力し、次のテキスト ボックスに「**209.244.0.4**」と入力します。
6. **[保存]** を選択します。
7. **Srv-Work** 仮想マシンを再起動します。

## <a name="test-the-firewall"></a>ファイアウォールをテストする

今度は、ファイアウォールをテストして、想定したように機能することを確認します。

1. リモート デスクトップをファイアウォールのパブリック IP アドレスに接続し、**Srv-Work** 仮想マシンにサインインします。 
3. Internet Explorer を開き、 `https://www.google.com` を参照します。
4. Internet Explorer のセキュリティ アラートで、 **[OK]**  >  **[閉じる]** の順に選択します。

   Google のホーム ページが表示されます。

5. `https://www.microsoft.com` を参照します。

   ファイアウォールによってブロックされます。

これで、ファイアウォール ルールが動作していることを確認できました。

* 1 つの許可された FQDN は参照できますが、それ以外は参照できません。
* 構成された外部 DNS サーバーを使用して DNS 名を解決できます。

## <a name="clean-up-resources"></a>リソースをクリーンアップする

引き続きテストを行うために、ファイアウォール リソースを残しておいてもかまいませんが、不要であれば、**Test-FW-RG** リソース グループを削除して、ファイアウォール関連のすべてのリソースを削除してください。

## <a name="next-steps"></a>次のステップ

<<<<<<< HEAD
> [!div class="nextstepaction"]
> [チュートリアル:Azure Firewall のログを監視する](./firewall-diagnostics.md)
=======
[チュートリアル:Azure Firewall のログを監視する](./firewall-diagnostics.md)
>>>>>>> repo_sync_working_branch
