---
title: 'チュートリアル: Azure Active Directory と Voyance の統合 | Microsoft Docs'
description: Azure Active Directory と Voyance の間でシングル サインオンを構成する方法について説明します。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/07/2019
ms.author: jeedes
ms.openlocfilehash: 662a16c66cdd2f552e84290d508f4671a03772a6
ms.sourcegitcommit: 0770a7d91278043a83ccc597af25934854605e8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/13/2021
ms.locfileid: "124772336"
---
# <a name="tutorial-azure-active-directory-integration-with-voyance"></a>チュートリアル: Azure Active Directory と Voyance の統合

このチュートリアルでは、Voyance と Azure Active Directory (Azure AD) を統合する方法について説明します。
Voyance と Azure AD の統合には、次の利点があります。

* Voyance にアクセスする Azure AD ユーザーを制御できます。
* ユーザーが自分の Azure AD アカウントを使用して自動的に Voyance にサインイン (シングル サインオン) するよう指定できます。
* 1 つの中央サイト (Azure Portal) でアカウントを管理できます。

SaaS アプリと Azure AD の統合の詳細については、「 [Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](../manage-apps/what-is-single-sign-on.md)」を参照してください。
Azure サブスクリプションをお持ちでない場合は、開始する前に[無料アカウントを作成](https://azure.microsoft.com/free/)してください。

## <a name="prerequisites"></a>前提条件

Azure AD と Voyance の統合を構成するには、次のものが必要です。

* Azure AD サブスクリプション。 Azure AD の環境がない場合は、[無料アカウント](https://azure.microsoft.com/free/)を取得できます
* Voyance でのシングル サインオンが有効なサブスクリプション

## <a name="scenario-description"></a>シナリオの説明

このチュートリアルでは、テスト環境で Azure AD のシングル サインオンを構成してテストします。

* Voyance では、**SP** と **IDP** によって開始される SSO がサポートされます

* Voyance では、**Just-In-Time** ユーザー プロビジョニングがサポートされます

## <a name="adding-voyance-from-the-gallery"></a>ギャラリーからの Voyance の追加

Azure AD への Voyance の統合を構成するには、ギャラリーから管理対象 SaaS アプリの一覧に Voyance を追加する必要があります。

**ギャラリーから Voyance を追加するには、次の手順に従います。**

1. **[Azure Portal](https://portal.azure.com)** の左側のナビゲーション ウィンドウで、 **[Azure Active Directory]** アイコンをクリックします。

    ![Azure Active Directory のボタン](common/select-azuread.png)

2. **[エンタープライズ アプリケーション]** に移動し、 **[すべてのアプリケーション]** オプションを選択します。

    ![[エンタープライズ アプリケーション] ブレード](common/enterprise-applications.png)

3. 新しいアプリケーションを追加するには、ダイアログの上部にある **[新しいアプリケーション]** をクリックします。

    ![[新しいアプリケーション] ボタン](common/add-new-app.png)

4. 検索ボックスに「**Voyance**」と入力し、結果ウィンドウで **[Voyance]** を選び、 **[追加]** をクリックして、アプリケーションを追加します。

     ![結果一覧の Voyance](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Azure AD シングル サインオンの構成とテスト

このセクションでは、**Britta Simon** というテスト ユーザーに基づいて、Voyance で Azure AD のシングル サインオンを構成し、テストします。
シングル サインオンを機能させるには、Azure AD ユーザーと Voyance 内の関連ユーザー間にリンク関係が確立されている必要があります。

Voyance で Azure AD のシングル サインオンを構成してテストするには、次の構成要素を完了する必要があります。

1. **[Azure AD シングル サインオンの構成](#configure-azure-ad-single-sign-on)** - ユーザーがこの機能を使用できるようにします。
2. **[Voyance シングル サインオンの構成](#configure-voyance-single-sign-on)** - アプリケーション側でシングル サインオン設定を構成します。
3. **[Azure AD のテスト ユーザーの作成](#create-an-azure-ad-test-user)** - Britta Simon で Azure AD のシングル サインオンをテストします。
4. **[Azure AD テスト ユーザーの割り当て](#assign-the-azure-ad-test-user)** - Britta Simon が Azure AD シングル サインオンを使用できるようにします。
5. **[Voyance のテスト ユーザーの作成](#create-voyance-test-user)** - Voyance で Britta Simon に対応するユーザーを作成し、Azure AD の Britta Simon にリンクさせます。
6. **[シングル サインオンのテスト](#test-single-sign-on)** - 構成が機能するかどうかを確認します。

### <a name="configure-azure-ad-single-sign-on"></a>Azure AD シングル サインオンの構成

このセクションでは、Azure portal 上で Azure AD のシングル サインオンを有効にします。

Voyance で Azure AD シングル サインオンを構成するには、次の手順に従います。

1. [Microsoft Azure portal](https://portal.azure.com/) の **Voyance** アプリケーション統合ページで、**[シングル サインオン]** を選択します。

    ![シングル サインオン構成のリンク](common/select-sso.png)

2. **[シングル サインオン方式の選択]** ダイアログで、 **[SAML/WS-Fed]** モードを選択して、シングル サインオンを有効にします。

    ![シングル サインオン選択モード](common/select-saml-option.png)

3. **[SAML でシングル サインオンをセットアップします]** ページで、 **[編集]** アイコンをクリックして **[基本的な SAML 構成]** ダイアログを開きます。

    ![基本的な SAML 構成を編集する](common/edit-urls.png)

4. **[基本的な SAML 構成]** セクションで、アプリケーションを **IDP** 開始モードで構成する場合は、次の手順を実行します。

    ![このスクリーンショットは、[基本的な SAML 構成] を示しています。ここで、識別子と応答 U R L を入力し、[保存] を選択できます。](common/idp-intiated.png)

    a. **[識別子]** ボックスに、`https://<companyname>.nyansa.com` の形式で URL を入力します。

    b. **[応答 URL]** ボックスに、`https://<companyname>.nyansa.com/saml/create/` のパターンを使用して URL を入力します

5. アプリケーションを **SP** 開始モードで構成する場合は、 **[追加の URL を設定します]** をクリックして次の手順を実行します。

    ![このスクリーンショットは、[追加の U R L を設定します] を示しています。ここで、サインオン U R L を入力できます。](common/metadata-upload-additional-signon.png)

    **[サインオン URL]** ボックスに、`https://<companyname>.nyansa.com/` という形式で URL を入力します。

    > [!NOTE]
    > これらは実際の値ではありません。 実際の識別子、応答 URL、サインオン URL でこれらの値を更新します。 これらの値を取得するには、[Voyance クライアント サポート チーム](mailto:support@nyansa.com)に問い合わせてください。 Azure portal の **[基本的な SAML 構成]** セクションに示されているパターンを参照することもできます。

6. **[SAML でシングル サインオンをセットアップします]** ページの **[SAML 署名証明書]** セクションで、 **[ダウンロード]** をクリックして要件のとおりに指定したオプションからの **証明書 (Base64)** をダウンロードして、お使いのコンピューターに保存します。

    ![証明書のダウンロードのリンク](common/certificatebase64.png)

7. **[Set up Voyance]**(Voyance のセットアップ) セクションで、要件のとおりに適切な URL をコピーします。

    ![構成 URL のコピー](common/copy-configuration-urls.png)

    a. ログイン URL

    b. Azure AD 識別子

    c. ログアウト URL

### <a name="configure-voyance-single-sign-on"></a>Voyance シングル サインオンの構成

1. 別の Web ブラウザーのウィンドウで、管理者として Voyance テナントにサインオンします。

2. ナビゲーション バーの右上隅に移動し、**[プロファイル]** をクリックします。
    
    ![アプリ側でのシングル サインオンの構成: Acme University](./media/voyance-tutorial/tutorial_voyance_001.png) 

3. **[管理者設定]** をクリックします。

    ![アプリ側でのシングル サインオンの構成: 管理者設定](./media/voyance-tutorial/tutorial_voyance_002.png)

4. **[ユーザー アクセス]** タブをクリックします。

    ![アプリ側でのシングル サインオンの構成: ユーザー アクセス](./media/voyance-tutorial/tutorial_voyance_003.png)

5. **[SSO is disabled]** (SSO は無効) ボタンをクリックして、SAML 2.0 を使用する IdP として Azure AD を構成します。

    ![アプリ側でのシングル サインオンの構成: SSO は無効](./media/voyance-tutorial/tutorial_voyance_004.png)

6. **SAML V2** セクションに移動して、次の手順を実行します。

    ![アプリ側でのシングル サインオンの構成: SAML V2](./media/voyance-tutorial/tutorial-voyance-005.png)
    
    a. **[Enabled]** を選択します。
    
    b. **[IdP Login UR]**(IdP ログイン URL) テキストボックスに、Microsoft Azure portal からコピーした **ログイン URL** を貼り付けます。

    c. ダウンロード済みの Base64 でエンコードされた証明書をメモ帳で開き、その内容をクリップボードにコピーして、**[IdP 証明書]** ボックスに貼り付けます。
    
    d. **[保存]** をクリックします。

### <a name="create-an-azure-ad-test-user"></a>Azure AD のテスト ユーザーの作成 

このセクションの目的は、Azure Portal で Britta Simon というテスト ユーザーを作成することです。

1. Azure portal の左側のウィンドウで、 **[Azure Active Directory]** 、 **[ユーザー]** 、 **[すべてのユーザー]** の順に選択します。

    ![[ユーザーとグループ] と [すべてのユーザー] リンク](common/users.png)

2. 画面の上部にある **[新しいユーザー]** を選択します。

    ![[新しいユーザー] ボタン](common/new-user.png)

3. [ユーザーのプロパティ] で、次の手順を実行します。

    ![[ユーザー] ダイアログ ボックス](common/user-properties.png)

    a. **[名前]** フィールドに「**BrittaSimon**」と入力します。
  
    b. **[ユーザー名]** フィールドに「brittasimon@yourcompanydomain.extension」と入力します。 たとえば、BrittaSimon@contoso.com のように指定します。

    c. **[パスワードを表示]** チェック ボックスをオンにし、[パスワード] ボックスに表示された値を書き留めます。

    d. **Create** をクリックしてください。

### <a name="assign-the-azure-ad-test-user"></a>Azure AD テスト ユーザーの割り当て

このセクションでは、Britta Simon に Voyance へのアクセスを許可することで、このユーザーが Azure シングル サインオンを使用できるようにします。

1. Microsoft Azure portal 上で **[エンタープライズ アプリケーション]** を選択し、**[すべてのアプリケーション]** を選択してから、**[Voyance]** を選択します。

    ![[エンタープライズ アプリケーション] ブレード](common/enterprise-applications.png)

2. アプリケーションの一覧で **[Voyance]** を選択します。

    ![アプリケーションの一覧の Voyance リンク](common/all-applications.png)

3. 左側のメニューで **[ユーザーとグループ]** を選びます。

    ![[ユーザーとグループ] リンク](common/users-groups-blade.png)

4. **[ユーザーの追加]** をクリックし、 **[割り当ての追加]** ダイアログで **[ユーザーとグループ]** を選択します。

    ![[割り当ての追加] ウィンドウ](common/add-assign-user.png)

5. **[ユーザーとグループ]** ダイアログの [ユーザー] の一覧で **[Britta Simon]** を選択し、画面の下部にある **[選択]** ボタンをクリックします。

6. SAML アサーション内に任意のロール値が必要な場合、 **[ロールの選択]** ダイアログでユーザーに適したロールを一覧から選択し、画面の下部にある **[選択]** をクリッします。

7. **[割り当ての追加]** ダイアログで、 **[割り当て]** ボタンをクリックします。

### <a name="create-voyance-test-user"></a>Voyance のテスト ユーザーの作成

このセクションでは、Britta Simon というユーザーを Voyance に作成します。 Voyance では、Just-In-Time ユーザー プロビジョニングがサポートされており、既定で有効になっています。 このセクションでは、ユーザー側で必要な操作はありません。 Voyance にユーザーがまだ存在していない場合は、認証後に新しく作成されます。

>[!NOTE]
>ユーザーを手動で作成する必要がある場合は、[Voyance サポート チーム](maiLto:support@nyansa.com)に問い合わせてください。

### <a name="test-single-sign-on"></a>シングル サインオンのテスト 

このセクションでは、アクセス パネルを使用して Azure AD のシングル サインオン構成をテストします。

アクセス パネル上で [Voyance] タイルをクリックすると、SSO を設定した Voyance に自動的にサインインします。 アクセス パネルの詳細については、[アクセス パネルの概要](https://support.microsoft.com/account-billing/sign-in-and-start-apps-from-the-my-apps-portal-2f3b1bae-0e5a-4a86-a33e-876fbd2a4510)に関する記事を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [SaaS アプリと Azure Active Directory を統合する方法に関するチュートリアルの一覧](./tutorial-list.md)

- [Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](../manage-apps/what-is-single-sign-on.md)

- [Azure Active Directory の条件付きアクセスとは](../conditional-access/overview.md)