---
lab:
    title: 'ラボ: AVD (Azure AD DS) 用のストレージを実装および管理する'
    module: 'モジュール 2: AVD インフラストラクチャを実装する'
---

# ラボ - AVD (Azure AD DS) 用のストレージを実装および管理する
# 受講生用ラボ マニュアル

## ラボの依存関係

- Azure サブスクリプション
- Azure サブスクリプションに関連付けられた Azure AD テナント内で グローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD、あるいは Azure サブスクリプション内 で所有者または共同作成はのロールを持つ Microsoft アカウントまたは Azure AD
- 実施するラボ - **Azure Virtual Desktop (Azure AD DS) のデプロイを準備する**

## 推定所要時間

30 分

## ラボ シナリオ

Azure Active Directory ドメイン サービス (Azure AD DS) 環境で Azure Virtual Desktop をデプロイするためのストレージを実装および管理する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- Azure AD DS 環境で Azure Virtual Desktop 用プロファイル コンテナーを格納するために Azure Files を構成する

## ラボ ファイル

- なし

## 説明

### 演習 1: Azure Virtual Desktop 用プロファイル コンテナーを格納するために Azure Files を構成する

この演習の主なタスクは次のとおりです。

1. Azure Storage アカウントを作成する
1. Azure Files 共有を作成する
1. Azure Storage アカウントに対する Azure AD DS 認証を有効にする 
1. Azure Files 共有のアクセス許可を構成する
1. Azure Files のディレクトリおよびファイル レベルのアクセス許可を構成する

#### タスク 1: Azure Storage アカウントを作成する

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. ラボ コンピューターの Azure portal で、**仮想マシン**を検索して選択し、**「仮想マシン」** ブレードから **az140-cl-vm11a** エントリを選択します。これにより、**az140-cl-vm11a** ブレードが開きます。
1. **az140-cl-vm11a** ブレードで、「**接続**」を選択し、ドロップダウン メニューで、「**Bastion**」を選択し、「**az140-cl-vm11a \| 接続**」ブレードの **Bastion** タブで、「**Bastion の使用**」を選択します。
1. プロンプトが表示されたら、次の資格情報を入力して、「**接続**」を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**aadadmin1@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

1. リモート デスクトップ内で **az140-cl-vm11a** Azure VM に移動し、Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動します。このアカウントの作成時に設定したパスワードを使用して **aadadmin1** ユーザー アカウントのユーザー プリンシパル名を指定してサインインします。

   >**注**: **aadadmin1** アカウントのユーザープリンシパル名 (UPN) 属性を特定するには、Active Directory ユーザーとコンピューター コンソールからプロパティ ダイアログ ボックスを確認するか、ラボ コンピューターに戻って、Azure portal の Azure AD テナント ブレードからプロパティを確認します。

1. **az140-cl-vm11a** へのリモート デスクトップ セッション内で、Azure portal を表示する Microsoft Edge で「**ストレージ アカウント**」を検索して選択し、**「ストレージ アカウント」** ブレードで **「+ 作成」** を選択します。
1. 「**ストレージ アカウントの作成**」ブレードの「**基本**」タブで、次のように設定を行います (他の設定は既定値のままにします)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|新しいリソース グループの名前 **az140-22a-RG**|
   |ストレージ アカウント名|小文字と数字で構成され、文字で始まる 3 〜 15 の長さのグローバルに一意の名前|
   |場所|Azure Virtual Desktop ラボ環境をホストする Azure リージョンの名前|
   |パフォーマンス|**Standard**|
   |レプリケーション|**ローカル冗長ストレージ (LRS)**|

   >**注**: ストレージアカウント名の長さが 15 文字を超えないようにしてください。この名前は、ストレージ アカウントを含む Azure サブスクリプションに関連付けられた Azure AD テナントと統合された Active Directory ドメイン サービス (AD DS) ドメインにコンピューター アカウントを作成するために使用されます。これにより、このストレージ アカウントでホストされているファイル共有にアクセスするときに、AD DS ベースの認証が可能になります。

1. 「**ストレージ アカウントの作成**」 ブレードの 「**基本**」 タブで、「**確認および作成**」 を選択し、検証プロセスが完了するのを待ってから、「**作成**」 を選択します。

   >**注**: ストレージ アカウントが作成されるのを待ちます。この手順には約 2 分かかります。

#### タスク 2: Azure Files 共有を作成する

1. **az140-cl-vm11a** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、**「ストレージ アカウント」** ブレードに戻り、新しく作成されたストレージ アカウントを表すエントリを選択します。
1. ストレージ アカウント ブレードの左側の垂直メニューの **「データ ストレージ」** セクションで、**「ファイル共有」** を選択し、**「+ ファイル共有」** を選択します。
1. 「**新しいファイル共有**」 で、次の設定を指定し、「**作成**」 を選択します (他の設定はデフォルト値のままにします)。

   |設定|値|
   |---|---|
   |名前|**az140-22a-profiles**|

#### タスク 3: Azure Storage アカウントに対する Azure AD DS 認証を有効にする

1. **az140-cl-vm11a** へのリモート デスク トップセッション内、Microsoft Edge ウィンドウ、Azure portal、前のタスクで作成したストレージ アカウントのプロパティを表示するブレード、左側の垂直メニュー、**「データ ストレージ」** セクションで、**「ファイル共有」** を選択します。 
1. 「**ファイル共有設定**」セクションの「**Active Directory**」ラベルの横にある「**未構成**」リンクを選択します。
1. 「**Active Directory ソースを有効にする**」セクションの「**Azure Active Directory ドメイン サービス**」というラベルの付いた長方形で、「**セットアップ**」を選択します。
1. 「**ID ベースのアクセス**」ブレードで、「**有効**」オプションを選択し、「**保存**」を選択します。

#### タスク 4: Azure Files RBAC ベースのアクセス許可を構成する

1. **az140-cl-vm11a** へのリモート デスクトップ セッション内、Azure portal を表示する Microsoft Edge ウィンドウ、この演習の前半で作成したストレージ アカウントのプロパティを表示するブレード、左側の垂直メニュー、**「データ ストレージ」** セクションで **「ファイル共有」** を選択し、共有のリストで **az140-22a-profiles** エントリを選択します。
1. **「az140-22a-profiles」** ブレードの左側の垂直メニューで、**「アクセス制御 (IAM)」** を選択します。
1. 「**az140-22a-profiles \| アクセス制御 (IAM)**」 ブレードで、「**+ 追加**」 を選択し、ドロップダウン メニューで 「**ロール割り当ての追加**」 を選択します。
1. 「**ロールの割り当ての追加**」ブレードで、「**ストレージ ファイル データの SMB 共有共同作成者**」を選択し、「**次へ**」を選択します。
1. 「**メンバー**」ブレードで、「**アクセスの割り当て**」を選択し、「**+ メンバーの選択**」をクリックします。
1. 「**メンバーの選択**」ブレードの「**選択**」テキスト ボックスに「**az140-wvd-users**」と入力してから、「**選択**」をクリックします。
1. 「**メンバー**」ブレードで、「**レビュー + 割り当て**」を 2 回選択します。
1. 上記の手順 3 - 8 を繰り返し、次の設定を指定します。

   |設定|値|
   |---|---|
   |ロール|**記憶域ファイル データの SMB 共有の管理者特権共同作成者**|
   |選択|**az140-wvd-aadmins**|

   > **注**: **az140-wvd-aadmins** グループのメンバーである **aadadmin1** ユーザー アカウントを使用して、ファイル共有のアクセス許可を構成します。 

#### タスク 5: Azure Files のディレクトリおよびファイル レベルのアクセス許可を構成する

1. **az140-cl-vm11a** へのリモート デスクトップ セッション内で、**コマンド プロンプト**を起動し、**コマンド プロンプト** ウィンドウから次のコマンドを実行して、ドライブをターゲット共有にマップします (`<storage-account-name>` プレースホルダーをストレージ アカウントの名前に置き換えます)。

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-22a-profiles
   ```

1. **az140-cl-vm11a** へのリモート デスクトップ セッション内で、ファイル エクスプローラーを開き、新しくマップされた Z: ドライブに移動し、**「プロパティ」** ダイアログ ボックスを表示し、**「セキュリティ」** タブを選択し、**「編集」**、**「追加」** の順に選択します。**「ユーザー、コンピューター、サービス アカウント、およびグループの選択」** ダイアログ ボックスで、**「この場所から」** テキストボックスに **adatum.com** エントリが含まれていることを確認し、**「選択するオブジェクト名を入力してください」** テキストボックスに **az140-wvd-ausers** と入力し、**「OK」** をクリックします。
1. マップされたドライブのアクセス許可を表示するダイアログ ボックスの **「セキュリティ」** タブに戻り、**az140-wvd-ausers** エントリが選択されていることを確認し、**「許可」** 列の **「変更」** チェックボックスを選択して **「OK」** をクリックし、**「Windows セキュリティ」** テキストボックスに表示されるメッセージを確認して、**「はい」** をクリックします。 
1. マップされたドライブのアクセス許可を表示しているダイアログ ボックスの **「セキュリティ」** タブに戻り、**「編集」**、**「追加」** の順に選択します。**「ユーザー、コンピューター、サービス アカウント、およびグループの選択」** ダイアログ ボックスで、**「この場所から」** テキストボックスに **adatum.com** エントリが含まれていることを確認し、**「選択するオブジェクト名を入力してください」** テキストボックスに **「az140-wvd-aadmins」 **と入力し、**「OK」** をクリックします。
1. マップされたドライブのアクセス許可を表示するダイアログ ボックスの **「セキュリティ」** タブに戻り、**「az140-wvd-aadmins」** エントリが選択されていることを確認し、**「許可」** 列の **「フル コントロール」** チェックボックスを選択して、**「OK」** をクリックします。 
1. マップされたドライブのアクセス許可を表示するダイアログ ボックスの **「セキュリティ」** タブで、グループとユーザー名のリストで **「編集」** を選択し、**「認証されたユーザー」** エントリを選択して、**「削除」** を選択します。
1. マップされたドライブのアクセス許可を表示するダイアログ ボックスの **「セキュリティ」** タブで、**「編集」** を選択し、グループとユーザー名のリストで **「ユーザー」** エントリを選択し、**「削除」** を選択して、**「OK」** をクリックし、**「OK」** を 2 回クリックしてプロセスを完了します。 

   >**注**: または、**icacls** コマンドライン ユーティリティを使用して権限を設定することもできます。 
