---
lab:
    title: 'ラボ: Azure Virtual Desktop プロファイル (Azure AD DS) を実装および管理する'
    module: 'モジュール 4: ユーザーの環境とアプリを管理する'
---

# ラボ - Azure Virtual Desktop プロファイル (Azure AD DS) を実装および管理する
# 受講生用ラボ マニュアル

## ラボの依存関係

- Azure サブスクリプション
- Azure サブスクリプションに関連付けられた Azure AD テナント内で グローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD、あるいは Azure サブスクリプション内 で所有者または共同作成はのロールを持つ Microsoft アカウントまたは Azure AD
- ラボ「**Azure Virtual Desktop の概要 (Azure AD DS)**」でプロビジョニングされた Azure Virtual Desktop 環境

## 推定所要時間

30 分

## ラボ シナリオ

Azure Active Directory ドメイン サービス (Azure AD DS) 環境に Azure Virtual Desktop プロファイル管理を実装する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- Azure AD DS 環境で Azure Virtual Desktop 用プロファイル コンテナーを格納するために Azure Files を構成する
- Azure AD DS 環境で Azure Virtual Desktop の FSLogix ベース プロファイルを実装および管理する

## ラボ ファイル

- なし

## 説明

### 演習 1: Azure Virtual Desktop の FSLogix ベースのプロファイルを実装する

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop セッション ホスト VM でローカル管理者グループを構成する
1. Azure Virtual Desktop セッション ホスト VM で FSLogix ベースのプロファイルを構成する
1. Azure Virtual Desktop を使用して FSLogix ベースのプロファイルをテストする
1. Azure ラボ リソースを削除する

#### タスク 1: Azure Virtual Desktop セッション ホスト VM でローカル管理者グループを構成する

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. ラボ コンピューターの Azure portal で、**仮想マシン**を検索して選択し、**「仮想マシン」** ブレードから **az140-cl-vm11a** エントリを選択します。これにより、**az140-cl-vm11a** ブレードが開きます。
1. **az140-cl-vm11a** ブレードで、「**接続**」を選択し、ドロップダウン メニューで、「**Bastion**」を選択し、「**az140-cl-vm11a \| 接続**」ブレードの Bastion タブで、「**Bastion の使用**」を選択します。
1. プロンプトが表示されたら、次の資格情報を入力して、「**接続**」を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**Student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-cl-vm11a** へのリモートデスクトップセッション内で、「スタート」 メニューの **「Windows 管理ツール」** フォルダーに移動して展開し、**「Active Directory ユーザーとコンピューター」** を選択します。
1. **Active Directory ユーザーとコンピューター** コンソールで、ドメイン ノードを右クリックし、**「新規」**、**「組織単位」** の順に選択し、**「新しいオブジェクト-組織単位」** ダイアログ ボックスの **「名前」** テキストボックスに「**ADDC ユーザー**」と入力して、**「OK」** を選択します。
1. **Active Directory ユーザーとコンピューター** コンソールで、AD DC ユーザーを右クリックし、**「新規」**、**「グループ」** の順に選択し、**「新しいオブジェクト-組織単位」** ダイアログ ボックスで、次の設定を指定して **「OK」** を選択します。

   |設定|値|
   |---|---|
   |グループ名|**ローカル管理者**|
   |グループ名 (Windows 2000 より前)|**ローカル管理者**|
   |グループのスコープ|**グローバル**|
   |グループ タイプ|**セキュリティ**|

1. **Active Directoryユーザーとコンピューター** コンソールで、**「ローカル管理者」** グループのプロパティを表示し、**「メンバー」** タブに切り替え、**「追加」** を選択し、**「ユーザー、連絡先、コンピューター、サービス アカウント、またはグループの選択」** ダイアログ ボックスで、**「選択するオブジェクト名を入力」** で、**aadadmin1** と入力して、**「OK」** を選択します。
1. **az140-cl-vm11a** へのリモートデスクトップセッション内で、「スタート」 メニューの **「Windows 管理ツール」** フォルダーに移動して展開し、**「グループ ポリシー管理」** を選択します。
1. **グループポリシー管理**コンソールで、**AADDC Computers** OU に移動し、**AADDC Computers GPO** アイコンを右クリックして、**「編集」** を選択します。
1. **グループ ポリシー管理エディター**コンソールで、**「コンピューターの構成」**、**「ポリシー」**、**「Windows の設定」**、**「セキュリティの設定」** の順に展開し、**「制限されたグループ」** を右クリックして、**「グループの追加」** を選択します。
1. **「グループの追加」** ダイアログ ボックスの **「グループ」** テキスト ボックスで、**「参照」**、**「グループの選択」** ダイアログ ボックス、**「選択するオブジェクト名の入力」** で、**「ローカル管理者」** と入力し、**「OK」** を選択します。
1. **「グループの追加」** ダイアログ ボックスに戻り、**「OK」** を選択します。
1. **「ADATUM\ Local Admins のプロパティ」** ダイアログ ボックスの **「このグループはメンバーです」** というラベルの付いたセクションで、**「追加」** を選択し、**「グループ メンバー シップ」** ダイアログ ボックスで「**Administrators**」と入力して **「OK」** を選択し、もう一度 **「OK」** を選択して変更を確定します。

   >**注**: **このグループは以下のメンバーです**というラベルの付いたセクションを必ず使用してください

1. リモート デスクトップから az140-cl-vm11aAzure VMで、管理者として PowerShell ISE を起動し、以下を実行して 2 つの Azure Virtual Desktop ホストを再起動し、グループ ポリシー処理をトリガーします。

   ```powershell
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Restart-Computer -ComputerName $servers -Force -Wait
   ```

1. スクリプトが完了するのを待ちます。通常は 3 分ほどかかります。

#### タスク 2: Azure Virtual Desktop セッション ホスト VM で FSLogix ベースのプロファイルを構成する

1. **az140-cl-vm11a** へのリモート デスクトップ セッション内で、**az140-21-p1-0** へのリモート デスクトップ セッションを開始します。プロンプトが表示されたら、 **ADATUM\wvdaadmin1** ユーザー名と、このユーザー アカウントの作成時に設定したパスワードを使ってサインインします。 
1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、Microsoft Edge を起動し、[FSLogix ダウンロード ページ](https://aka.ms/fslogix_download)を参照し、FSLogix 圧縮インストール バイナリをダウンロードして、**C:\\Source** フォルダーに抽出し、**x64\\Release** サブフォルダーに移動し、**FSLogixAppsSetup.exe** を使用して、Microsoft FSLogixApps をデフォルト設定でインストールします。

   > **注**: イメージがすでに含まれているかどうがにより、FXLogic のインストールは不要になる場合があります。FX Logix のインストールには、再起動が必要です。

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して PowerShellGet モジュールの最新バージョンをインストールします (確認を求められたら **「はい」** を選択します)。

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、最新バージョンの Az PowerShell モジュールをインストールします (確認を求められたら、**「すべてはい」** を選択します)。

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、以下を実行して、実行ポリシーを変更します。

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次を実行して Azure サブスクリプションにサインインします。

   ```powershell
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、このラボで以前に構成した AzureStorage アカウントの名前を取得します。

   ```powershell
   $resourceGroupName = 'az140-22a-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインから、以下を実行して、プロファイル レジストリ設定を構成します。

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. **az140-21-p1** へのリモート デスクトップ セッション内で、**「スタート」** を右クリックし、右クリック メニューで **「ファイル名を指定して実行」** を選択し、**「ファイル名を指定して実行」** ダイアログ ボックスの **「開く」** テキストボックスに次のように入力し、**「OK」** を選択して **「ローカル ユーザーとグループ」** ウィンドウを起動します。

   ```cmd
   lusrmgr.msc
   ```

1. **ローカル ユーザーとグループ** コンソールで、名前が **FSLogix** 文字列で始まる 4 つのグループに注意してください。

   - FSLogix ODFC 除外リスト
   - FSLogix ODFC 包含リスト
   - FSLogix プロファイル除外リスト
   - FSLogix プロファイル包含リスト

1. **ローカル ユーザーとグループ** コンソールで、**FSLogix プロファイル包含リスト** グループ エンティティをダブルクリックし、**\\Everyone** グループが含まれていることに注意して、 
**「OK」** を選択してグループの **「プロパティ」** ウィンドウを閉じます。 
1. **ローカル ユーザーとグループ** コンソールで、**FSLogix プロファイル除外リスト** グループ エンティティをダブルクリックし、既定ではグループ メンバーが含まれていないことに注意し、**「OK」** を選択してグループの **「プロパティ」** ウィンドウを閉じます。 

   > **注**: 一貫したユーザー エクスペリエンスを提供するには、すべての Azure Virtual Desktop セッション ホストに FSLogix コンポーネントをインストールして構成する必要があります。このタスクは、ラボ環境の他のセッション ホストで無人で実行します。 

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次を実行して FSLogix コンポーネントを **az140-21-p1-1** セッション ホストにインストールします。

   ```powershell
   $server = 'az140-21-p1-1' 
   $localPath = 'C:\Source\x64'
   $remotePath = "\\$server\C$\Source\x64\Release"
   Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
   Invoke-Command -ComputerName $server -ScriptBlock {
      Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
   } 
   ```

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次を実行して、**az140-21-p1-1** セッション ホストでプロファイル レジストリ設定を構成します。

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   Invoke-Command -ComputerName $server -ScriptBlock {
      New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
   }
   ```

   > **注**: FSLogix ベースのプロファイル機能をテストする前に、テストに使用する ADATUM\wvdaadmin1 アカウントのローカルにキャッシュされたプロファイルを、前のラボで使用した Azure Virtual Desktop セッション ホストから削除する必要があります。

1. リモート デスクトップ セッションを **az140-cl-vm11a** に切り替え、リモート デスクトップ セッションを **az140-cl-vm11a** に切り替え、「**管理者: Windows PowerShell ISE**」 ウィンドウに切り替え、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次を実行して、ADATUM\aaduser1 アカウントのローカルにキャッシュされたプロファイルを削除します。

   ```powershell
   $userName = 'aaduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### タスク 3: Azure Virtual Desktop を使用して FSLogix ベースのプロファイルをテストする

1. **az140-cl-vm11a** へのリモート デスクトップ セッション内で、リモート デスクトップ クライアントに切り替えます。
1. **az140-cl-vm11a** へのリモート デスクトップ セッション内の 「**リモート デスクトップ** クライアント」 ウィンドウのアプリケーションのリストで、**「コマンド プロンプト」** をダブルクリックし、プロンプトが表示されたら、パスワードを入力し、**「コマンド プロンプト」** ウィンドウが起動することを確認します。 

   > **注**: 最初は、アプリケーションの起動に数分かかる場合がありますが、その後、アプリケーションの起動ははるかに高速になります。

1. **「コマンド プロンプト」** ウィンドウの左上隅にある **「コマンド プロンプト」** アイコンを右クリックし、ドロップダウン メニューで **「プロパティ」** を選択します。
1. **「コマンド プロンプトのプロパティ」** ダイアログ ボックスで、**「フォント」** タブを選択し、サイズとフォント設定を変更して、**「OK」** を選択します。
1. **「コマンド プロンプト」** ウィンドウで、**logoff** と入力し、**Enter** キーを押して、リモート デスクトップ セッションからサインアウトします。
1. **az140-cl-vm11a** へのリモート デスクトップ セッション内の 「**リモート デスクトップ** クライアント」 ウィンドウのアプリケーションのリストで、**「SessionDesktop」** をダブルクリックし、リモート デスクトップ セッションが起動することを確認します。 
1. **SessionDesktop** セッション内で、**「スタート」** を右クリックし、右クリック メニューで **「ファイル名を指定して実行」** を選択し、**「ファイル名を指定して実行」** ダイアログ ボックスの **「開く」** テキストボックスに **cmd** と入力し、**「OK」** を選択して**コマンド プロンプト** ウィンドウを起動します。
1. **コマンド プロンプト** ウィンドウのプロパティが、このタスクの前半で設定したものと一致することを確認します。
1. **SessionDesktop** セッション内で、すべてのウィンドウを最小化し、デスクトップを右クリックし、右クリック メニューで **「新規」** を選択し、カスケード メニューで **「ショートカット」** を選択します。 
1. **どの項目にショートカットを作成しますか?** **ショートカットの作成**ウィザードのページで、**「項目の場所を入力してください」** テキスト ボックスに「**メモ帳**」と入力し、**「次へ」** を選択します。
1. **「ショートカットの作成」** ウィザードの **「ショートカットの名前を付ける」** ページで、**「このショートカットの名前を入力してください」** テキスト ボックスに「**メモ帳**」と入力し、**「完了」** を選択します。
1. **SessionDesktop** セッション内で、**「スタート」** を右クリックし、右クリック メニューで **「シャットダウン」 または 「サインアウト」** を選択してから、カスケード メニューで **「サインアウト」** を選択します。
1. **az140-cl-vm11a** へのリモート デスクトップ セッションに戻り、「**リモート デスクトップ** クライアント」 ウィンドウのアプリケーションのリストで、**「SessionDesktop」** をダブルクリックし、新しいリモート デスクトップ セッションを開始します。 
1. **SessionDesktop** セッション内で、**メモ帳**のショートカットがデスクトップに表示されることを確認します。
1. **SessionDesktop** セッション内で、**「スタート」** を右クリックし、右クリック メニューで **「シャットダウン」 または 「サインアウト」** を選択してから、カスケード メニューで **「サインアウト」** を選択します。
1. **az140-cl-vm11a** へのリモート デスクトップ セッションに切り替え、Azure portal を表示する Microsoft Edge ウィンドウに切り替えます。
1. Azure portal を表示している Microsoft Edge ウィンドウで、**「ストレージ アカウント」** ブレードに戻り、前の演習で作成したストレージ アカウントを表すエントリを選択します。
1. ストレージ アカウント ブレードの **「ファイル サービス」** セクションで **「ファイル共有」** を選択し、ファイル共有のリストで **「az140-22a-profiles」** を選択します。 
1. **az140-22a-profiles** ブレードで、そのコンテンツに、**ADATUM\\aaduser1** アカウントのセキュリティ識別子 (SID) とそれに続く **_aaduser1** サフィックスの組み合わせで構成される名前のフォルダーが含まれていることを確認します。
1. 前の手順で特定したフォルダーを選択し、**Profile_aaduser1.vhd** という名前の 1 つのファイルが含まれていることに注意してください。

#### タスク 4: Azure ラボ リソースを削除する

1. [Azure portal を使用した Azure Active Directory ドメイン サービスの管理対象ドメインの削除](https://docs.microsoft.com/ja-jp/azure/active-directory-domain-services/delete-aadds)で説明されている手順に従って、Azure AD DS のデプロイを削除します。
1. [Azure Resource Manager リソース グループとリソースの削除](https://docs.microsoft.com/ja-jp/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-portal) で説明されている手順に従って、このコースの Azure AD DS ラボでプロビジョニングしたすべての Azure リソース グループを削除します。
