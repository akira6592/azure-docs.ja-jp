---
title: Azure Key Vault とは? | Microsoft Docs
description: Azure Key Vault で、クラウド アプリケーションやサービスで使われる暗号化キーとシークレットをどのように保護するかについて学習します。
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 01/18/2019
ms.author: mbaldwin
ms.openlocfilehash: a8f0088077d5b7d0ec9fbc4336ee68a3459845b1
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/04/2021
ms.locfileid: "111411811"
---
# <a name="azure-key-vault-basic-concepts"></a>Azure Key Vault の基本的な概念

Azure Key Vault は、シークレットを安全に保管し、それにアクセスするためのクラウド サービスです。 シークレットは、API キー、パスワード、証明書、暗号化キーなど、アクセスを厳密に制御する必要がある任意のものです。 Key Vault サービスでは、ボールトおよびマネージド ハードウェア セキュリティ モジュール (HSM) プールという 2 種類のコンテナーがサポートされています。 ボールトでは、ソフトウェアと HSM でバックアップされるキー、シークレット、証明書を保存できます。 Managed HSM プールは、HSM でバックアップされるキーにのみ対応しています。 詳細については、[Azure Key Vault REST API の概要](about-keys-secrets-certificates.md)に関するページを参照してください。

その他の重要な用語を次に示します。

- **Tenant**: テナントは、Microsoft クラウド サービスの特定のインスタンスを所有および管理する組織です。 ほとんどの場合、この用語は、組織の Azure と Microsoft 365 の一連のサービスを指すために使用されます。

- **コンテナーの所有者**:コンテナー所有者は、キー コンテナーを作成し、それにフル アクセスして制御することができます。 コンテナー所有者は、だれがシークレットとキーにアクセスしたかをログに記録するように監査を設定することもできます。 管理者は、キーのライフサイクルを制御できます。 新しいバージョンのキーへの切り替え、キーのバックアップ、および関連タスクを行うことができます。

- **コンテナー コンシューマー**:コンテナー コンシューマーは、コンテナー所有者によってアクセスを許可されると、キー コンテナー内のアセットに対してアクションを実行できます。 使用可能なアクションは、付与されるアクセス許可によって異なります。

- **Managed HSM 管理者**:管理者ロールが割り当てられているユーザーは、Managed HSM プールを完全に制御できます。 ロール割り当てをさらに作成し、制御されたアクセスを他のユーザーに委任できます。

- **Managed HSM Crypto Officer とユーザー**:Managed HSM でキーを使用して暗号操作を実行するユーザーまたはサービス プリンシパルに通常割り当てられる組み込みロール。 Crypto User は新しいキーを作成できますが、キーを削除することはできません。

- **Managed HSM 暗号化サービスの暗号化ユーザー**: 顧客が管理するキーで保存データを暗号化するためのサービス アカウント管理サービス ID (ストレージ アカウントなど) に通常割り当てられる組み込みロール。

- **リソース**:リソースは、Azure を通じて使用できる、管理可能な要素です。 一般的な例として、仮想マシン、ストレージ アカウント、Web アプリ、データベース、仮想ネットワークなどがあります。 他にもさまざまなリソースが存在します。

- **[リソース グループ]** :リソース グループは、Azure ソリューションの関連するリソースを保持するコンテナーです。 リソース グループには、ソリューションのすべてのリソースか、グループとして管理したいリソースのみを含めることができます。 組織にとって最も有用になるように、リソースをリソース グループに割り当てる方法を決定します。

- **セキュリティ プリンシパル**:Azure セキュリティ プリンシパルは、ユーザーが作成したアプリ、サービス、オートメーション ツールで特定の Azure リソースにアクセスするために使用されるセキュリティ ID です。 アクセス許可が厳しく管理された、特定のロールが与えられた "ユーザー ID" (ユーザー名とパスワードまたは証明書) と考えてください。 一般的なユーザー ID とは異なり、セキュリティ プリンシパルで行うタスクは制限する必要があります。 管理タスクを実行するために必要な最小限のアクセス許可レベルのみを付与すれば、セキュリティは向上します。 アプリケーションまたはサービスと共に使用されるセキュリティ プリンシパルは、特に **サービス プリンシパル** と呼ばれています。

- [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md):Azure AD は、テナント向けの Active Directory サービスです。 各ディレクトリには、1 つまたは複数のドメインが存在します。 ディレクトリには複数のサブスクリプションを関連付けることができますが、テナントは 1 つだけです。

- **Azure テナント ID**:テナント ID は、Azure サブスクリプション内で Azure AD インスタンスを識別するための独自の方法です。

- **マネージド ID**:Azure Key Vault は、資格情報およびその他のキーやシークレットを安全に保管する方法を提供しますが、コードは Key Vault に認証してそれらを取得する必要があります。 マネージド ID を使用すると、Azure AD で自動的に管理される ID が Azure サービスに提供されるため、この問題の解決が簡単になります。 この ID を使用すると、コードに資格情報が含まれていなくても、Key Vault または Azure AD 認証をサポートする任意のサービスの認証を受けることができます。 詳細については、下の画像と [Azure リソースのマネージド ID の概要](../../active-directory/managed-identities-azure-resources/overview.md)に関するページを参照してください。

## <a name="authentication"></a>認証
Key Vault で操作を行うには、まず、それを認証する必要があります。 次の 3 つの方法で Key Vault を認証します。

- [Azure リソースのマネージド ID](../../active-directory/managed-identities-azure-resources/overview.md):Azure の仮想マシンにアプリをデプロイするときに、Key Vault にアクセスできる仮想マシンに ID を割り当てることができます。 [他の Azure リソース](../../active-directory/managed-identities-azure-resources/overview.md)にも ID を割り当てることができます。 この手法の利点は、最初のシークレットのローテーションがアプリやサービスで管理されないことにあります。 Azure では、ID が自動的にローテーションされます。 ベスト プラクティスとして、この手法をお勧めします。 
- **サービス プリンシパルと証明書**:サービス プリンシパルと、Key Vault にアクセスできる関連証明書を使用することができます。 アプリケーションの所有者または開発者が証明書をローテーションする必要があるため、この手法はお勧めできません。
- **サービス プリンシパルとシークレット:** サービス プリンシパルとシークレットを使用して Key Vault に対する認証を行うことはできますが、これはお勧めできません。 Key Vault に対する認証で使用されるブートストラップ シークレットを自動的にローテーションするのは困難です。

## <a name="encryption-of-data-in-transit"></a>転送中のデータの暗号化

Azure Key Vault では、Azure Key Vault とクライアント間をデータが移動するときにデータを保護するために、[トランスポート層セキュリティ](https://en.wikipedia.org/wiki/Transport_Layer_Security) (TLS) プロトコルが強制されます。 クライアントは、Azure Key Vault との TLS 接続をネゴシエートします。 TLS には、強力な認証、メッセージの機密性、整合性 (メッセージの改ざん、傍受、偽造の検出が有効)、相互運用性、アルゴリズムの柔軟性、デプロイと使用のしやすさといったメリットがあります。

[Perfect Forward Secrecy](https://en.wikipedia.org/wiki/Forward_secrecy) (PFS) は、顧客のクライアント システムと Microsoft クラウド サービスとの間の接続を一意のキーで保護します。 接続には RSA ベースの 2,048 ビット長の暗号化キーも使用します。 この組み合わせにより、転送中のデータの傍受やデータへのアクセスを困難にしています。

## <a name="key-vault-roles"></a>Key Vault の役割

次の表を使用して、Key Vault が開発者やセキュリティ管理者のニーズを満たすのに役立つ方法を十分に理解します。

| Role | 問題の説明 | Azure Key Vault による解決 |
| --- | --- | --- |
| Azure アプリケーションの開発者 |"キーを使って署名と暗号化を行う Azure 用のアプリケーションを作成したい。 しかし、ソリューションが地理的に分散したアプリケーションに合うように、これらのキーをアプリケーションの外部に設定したい。 <br/><br/>これらのキーとシークレットは、自分でコードを記述せずに保護したい。 さらに、アプリケーションから最適なパフォーマンスで簡単に使用できるようにしたい。" |√ キーは、資格情報コンテナーに格納され、必要に応じて、URI によって呼び出されます。<br/><br/> √ キーは、業界標準のアルゴリズム、キーの長さ、ハードウェア セキュリティ モジュールを使用して、Azure によってセキュリティで保護されています。<br/><br/> √ キーは、アプリケーションと同じ Azure データセンターに存在する HSM で処理されます。 この方法では、オンプレミスなどの別の場所に存在するキーより、信頼性が向上し、待機時間が削減されます。 |
| サービスとしてのソフトウェア (SaaS) の開発者 |"お客様のテナント キーやシークレットに対して義務や潜在的責任を負いたくない。 <br/><br/>お客様がキーを自分で所有して管理すれば、自分はコア ソフトウェア機能を提供することに集中してベストを尽くすことができる。" |√ 顧客は Azure に自分のキーをインポートして管理できます。 SaaS アプリケーションで顧客のキーを使用して暗号化操作を実行する必要がある場合は、アプリケーションに代わって、Key Vault によりこれらの操作が行われます。 アプリケーションからは顧客のキーが見えません。 |
| 最高セキュリティ責任者 (CSO) |"アプリケーションが、セキュリティで保護されたキー管理のために FIPS 140-2 レベル 2 または FIPS 140-2 レベル 3 HSM に準拠していることを確認したい。 <br/><br/>組織が、キーのライフサイクルを管理し、キーの使用状況を確実に監視できるようにしたい。 <br/><br/>複数の Azure サービスとリソースを使用しているが、Azure の 1 つの場所からキーを管理したい。" |√ FIPS 140-2 レベル 2 への準拠が検証済みの HSM に **ボールト** を選択します。<br/>√ FIPS 140-2 レベル 3 への準拠が検証済みの HSM に **Managed HSM プール** を選択します。<br/><br/>√ Key Vault は、マイクロソフトがキーを確認または抽出しないように作られています。<br/>√ キーの使用状況は、ほぼリアルタイムで記録されます。<br/><br/>√ コンテナーは、Azure にあるコンテナーの数、サポートするリージョン、使用するアプリケーションに関係なく、1 つのインターフェイスを提供します。 |

Azure サブスクリプションを持つユーザーはだれでも、Key Vault を作成して使用できます。 Key Vault は開発者とセキュリティ管理者にとってメリットがありますが、他の Azure サービスを管理する組織の管理者がこれを実装して管理することができます。 たとえば、この管理者は Azure サブスクリプションを使用してサインインし、キーの格納先に組織用のコンテナーを作成し、次のような運用タスクを担当できます。

- キーやシークレットの作成やインポート
- キーやシークレットの取り消しや削除
- ユーザーまたはアプリケーションに Key Vault へのアクセスを許可します。結果、ユーザーまたはアプリケーションはそのキーおよびシークレットを管理または使用できるようになります。
- キーの使用状況の構成 (署名や暗号化など)
- キーの使用状況の監視

その後、この管理者は開発者に、アプリケーションから呼び出す URI を提供します。 また、この管理者は、セキュリティ管理者にキーの使用状況のログ情報を提供します。 

![Azure Key Vault の動作の概要][1]

開発者は、API を使用してキーを直接管理することもできます。 詳細については、「 [Azure Key Vault 開発者ガイド](developers-guide.md)」を参照してください。

## <a name="next-steps"></a>次のステップ

- [Azure Key Vault セキュリティ機能](security-features.md)について学習する
- [Managed HSM プールをセキュリティで保護する](../managed-hsm/access-control.md)方法を学習する

<!--Image references-->
[1]: ../media/key-vault-whatis/AzureKeyVault_overview.png
Azure Key Vault は、ほとんどのリージョンで使用できます。 詳細については、 [Key Vault の価格のページ](https://azure.microsoft.com/pricing/details/key-vault/)を参照してください。
