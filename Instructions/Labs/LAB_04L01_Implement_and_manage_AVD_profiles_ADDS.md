---
lab:
    title: 'ラボ: Azure Virtual Desktop プロファイル (AD DS) を実装および管理する'
    module: 'モジュール 4: ユーザーの環境とアプリを管理する'
---

# ラボ - Azure Virtual Desktop プロファイル (AD DS) を実装および管理する
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプション内で所有者または共同作成者のロールを持つ Microsoft アカウントまたは Azure AD アカウント、この Azure サブスクリプションに関連付けられた Azure AD テナント内でグローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD アカウント。
- 実施するラボ - **Azure Virtual Desktop (AD DS) のデプロイを準備する**
- 実施するラボ - **WVD (AD DS) 用のストレージを実装および管理する**

## 推定所要時間

30 分

## ラボ シナリオ

Active Directory ドメイン サービス (AD DS) 環境に Azure Virtual Desktop プロファイル管理を実装する必要があります。

## 目標
  
このラボを終了すると、下記ができるようになります。

- Azure Virtual Desktop の FSLogix ベースのプロファイルを実装する

## ラボ ファイル

- なし

## 手順

### 演習 1: Azure Virtual Desktop の FSLogix ベースのプロファイルを実装する

この演習の主なタスクは次のとおりです:

1. Azure Virtual Desktop セッション ホスト VM で FSLogix ベースのプロファイルを構成する
1. Azure Virtual Desktop を使用して FSLogix ベースのプロファイルをテストする
1. ラボにデプロイした Azure リソースを削除する

#### タスク 1: Azure Virtual Desktop セッション ホスト VM で FSLogix ベースのプロファイルを構成する

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal で、「**仮想マシン**」を検索して選択し、「**仮想マシン**」ブレードで、「**az140-21-p1**」を選択します。
1. 「**az140-21-p1-0**」ブレードで、「**開始**」を選択して、仮想マシンのステータスが「**実行中**」に変化するまで待機します。
1. 「**az140-21-p1-0**」ブレードで、「**接続**」を選択し、ドロップダウン メニューで、「**Bastion**」を選択し、「**az140-21-p1-0 \| 接続**」ブレードの「**Bastion**」タブで、「**Bastion の使用**」を選択します。
1. プロンプトが表示されたら、次の認証情報を入力します。

   |設定|値|
   |---|---|
   |ユーザー名|**ADATUM\Student**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、Microsoft Edge を起動して、[Azure portal](https://portal.azure.com) に移動します。プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、Cloud Shell ペイン内の PowerShell セッションを開始します。 
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで使用する Azure VM をホストする Azure Virtual Desktop セッションを開始します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**注**: Azure VM が実行中になるのを待ってから、次の手順に進みます。

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、Microsoft Edge を起動し、[FSLogix ダウンロード ページ](https://aka.ms/fslogix_download)を参照し、FSLogix 圧縮インストール バイナリをダウンロードして、**C:\\Allfiles\\Labs\\04** フォルダー (必要に応じて、フォルダ－を作成する) に抽出し、**x64\\Release** サブフォルダーに移動し、**FSLogixAppsSetup.exe** ファイルをダブルクリックして、**Microsoft FSLogix Apps セットアップ** ウィザードを実行し、デフォルト設定で Microsoft FSLogix Appsをインストールします。

   > **注**: イメージがすでに含まれている場合、FXLogic のインストールは不要です。

3. **az140-21-p1-0** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して PowerShellGet モジュールの最新バージョンをインストールします (確認を求められたら **「はい」** を選択します)。

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
1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、このラボで以前に構成した AzureStorage アカウントの名前を取得します。

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインから、以下を実行して、プロファイル レジストリ設定を構成します。

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. **az140-21-p1** へのリモート デスクトップ セッション内で、**「スタート」** を右クリックし、右クリック メニューで **「ファイル名を指定して実行」** を選択し、**「ファイル名を指定して実行」** ダイアログ ボックスの **「開く」** テキストボックスに次のように入力し、**「OK」** を選択して **「ローカル ユーザーとグループ」** コンソールを起動します。

   ```cmd
   lusrmgr.msc
   ```

1. **ローカル ユーザーとグループ** コンソールで、名前が **FSLogix** 文字列で始まる 4 つのグループに注意してください。

   - FSLogix ODFC 除外リスト
   - FSLogix ODFC 包含リスト
   - FSLogix プロファイル除外リスト
   - FSLogix プロファイル包含リスト

1. **「ローカル ユーザーとグループ」** コンソールのグループのリストで、**「FSLogix プロファイ包含リスト」** グループをダブルクリックし、**\\Everyone** グループが含まれていることを確認し、**「OK」** を選択して グループの **「プロパティ」** ウィンドウを閉じます。 
1. **ローカル ユーザーとグループ** コンソールのグループのリストで、**FSLogix プロファイル除外リスト** グループをダブルクリックし、既定ではグループ メンバーが含まれていないことに注意し、**「OK」** を選択してグループの **「プロパティ」** ウィンドウを閉じます。 

   > **注**: 一貫したユーザー エクスペリエンスを提供するには、すべての Azure Virtual Desktop セッション ホストに FSLogix コンポーネントをインストールして構成する必要があります。このタスクは、ラボ環境の他のセッション ホストで無人で実行します。 

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次を実行して FSLogix コンポーネントを **az140-21-p1-1** および **'az140-21-p1-1'** セッション ホストにインストールします。

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **注**: スクリプトの実行が完了するのを待ちます。これにはおよそ 2 分かかる場合があります。

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、**az140-21-p1-1** および **'az140-21-p1-1'** セッション ホストで、プロファイル・レジストリー設定を構成します。

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **注**: FSLogix ベースのプロファイル機能をテストする前に、テストに使用する **ADATUM\aduser1** アカウントのローカルにキャッシュされたプロファイルを、前のラボで使用した Azure Virtual Desktop セッション ホストから削除する必要があります。

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次を実行して、セッションホストとして機能するすべての Azure VM でローカルにキャッシュされた **ADATUM\\aduser1** アカウントのプロファイルを削除します。

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### タスク 2: Azure Virtual Desktop を使用して FSLogix ベースのプロファイルをテストする

1. ラボ コンピューターから、Azure portal を表示している Web ブラウザーの画面を表示しているラボ コンピューターに切り替え、**仮想ネットワーク**を検索して選択し、**「仮想ネットワーク」** ブレードから **az140-cl-vm11** エントリを選択します。
1. **az140-cl-vm11** ブレードで、「**接続**」を選択し、ドロップダウン メニューで、「**Bastion**」を選択し、「**az140-cl-vm11 \| 接続**」ブレードの Bastion タブで、「**Bastion の使用**」を選択します。
1. プロンプトが表示されたら、次の資格情報を入力して、「**接続**」を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**Student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-cl-vm11** へのリモート デスクトップ セッション内で、**「スタート」** をクリックし、**「スタート」** メニューで、**「リモート デスクトップ」** をクリックして、リモート デスクトップ クライアントを起動します。
1. **az140-cl-vm11** へのリモート デスクトップ セッション内の「**リモート デスクトップ** クライアント」ウィンドウで、**「購読」** を選択し、プロンプトが表示されたら、**aduser1** の資格情報を使用してサインインします。
1. アプリケーションのリストで、**「コマンド プロンプト」** をダブルクリックし、プロンプトが表示されたら、**aduser1** アカウントのパスワードを入力し、**「コマンド　プロンプト」** ウィンドウが正常に開くことを確認します。
1. **「コマンド プロンプト」** ウィンドウの左上隅にある **「コマンド プロンプト」** アイコンを右クリックし、ドロップダウン メニューで **「プロパティ」** を選択します。
1. **「コマンド プロンプトのプロパティ」** ダイアログ ボックスで、**「フォント」** タブを選択し、サイズとフォント設定を変更して、**「OK」** を選択します。
1. **「コマンド プロンプト」** ウィンドウで、**logoff** と入力し、**Enter** キーを押して、リモート デスクトップ セッションからサインアウトします。
1. **az140-cl-vm11** へのリモート デスクトップ セッション内の 「**リモート デスクトップ** クライアント」 ウィンドウのアプリケーションのリストで、az-140-21-s1 の下の **「SessionDesktop」** をダブルクリックし、リモート デスクトップ セッションが起動することを確認します。 
1. **SessionDesktop** セッション内で、**「スタート」** を右クリックし、右クリック メニューで **「ファイル名を指定して実行」** を選択し、**「ファイル名を指定して実行」** ダイアログ ボックスの **「開く」** テキストボックスに **cmd** と入力し、**「OK」** を選択して**コマンド プロンプト** ウィンドウを起動します。
1. **コマンド プロンプト** ウィンドウの設定が、このタスクの前半で構成した設定と一致していることを確認します。
1. **SessionDesktop** セッション内で、すべてのウィンドウを最小化し、デスクトップを右クリックし、右クリック メニューで **「新規」** を選択し、カスケード メニューで **「ショートカット」** を選択します。 
1. **どの項目にショートカットを作成しますか?** **ショートカットの作成**ウィザードのページで、**「項目の場所を入力してください」** テキスト ボックスに「**メモ帳**」と入力し、**「次へ」** を選択します。
1. **「ショートカットの作成」** ウィザードの **「ショートカットの名前を付ける」** ページで、**「このショートカットの名前を入力してください」** テキスト ボックスに「**メモ帳**」と入力し、**「完了」** を選択します。
1. **SessionDesktop** セッション内で、**「スタート」** を右クリックし、右クリック メニューで **「シャットダウン」 または 「サインアウト」** を選択してから、カスケード メニューで **「サインアウト」** を選択します。
1. **az140-cl-vm11** へのリモート デスクトップ セッションに戻り、「**リモート デスクトップ** クライアント」 ウィンドウのアプリケーションのリストで、**「SessionDesktop」** をダブルクリックし、新しいリモート デスクトップ セッションを開始します。 
1. **SessionDesktop** セッション内で、**メモ帳**のショートカットがデスクトップに表示されることを確認します。
1. **SessionDesktop** セッション内で、**「スタート」** を右クリックし、右クリック メニューで **「シャットダウン」 または 「サインアウト」** を選択してから、カスケード メニューで **「サインアウト」** を選択します。
1. ラボ コンピューターに切り替え、Azure portal を表示している Microsoft Edge ウィンドウで、**「ストレージ アカウント」** ブレードに移動し、前の演習で作成したストレージ アカウントを表すエントリを選択します。
1. ストレージ アカウント ブレードの **「ファイル サービス」** セクションで **「ファイル共有」** を選択し、ファイル共有のリストで **「az140-22-profiles」** を選択します。 
1. **az140-22-profiles** ブレードで、そのコンテンツに、**ADATUM\\aduser1** アカウントのセキュリティ識別子 (SID) とそれに続く **_aduser1** サフィックスの組み合わせで構成される名前のフォルダーが含まれていることを確認します。
1. 前の手順で特定したフォルダーを選択し、**Profile_aduser1.vhd** という名前の 1 つのファイルが含まれていることに注意してください。

### 演習 2: ラボでプロビジョニングおよび使用された Azure VM を停止および割り当て解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングおよび使用された Azure VM を停止および割り当て解除する

>**注**: この演習では、このラボでプロビジョニングおよび使用した Azure VM を割り当て解除し、対応するコンピューティング料金を最小化します

#### タスク 1: ラボでプロビジョニングおよび使用された Azure VM を割り当て解除する

1. ラボ コンピューターに切り替え、Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成および使用されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成および使用されたすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**: コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
