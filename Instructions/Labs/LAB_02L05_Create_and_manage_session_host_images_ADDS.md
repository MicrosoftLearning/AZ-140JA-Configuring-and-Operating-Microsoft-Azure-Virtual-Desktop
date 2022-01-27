---
lab:
    title: 'ラボ: セッション ホスト イメージを作成および管理する (AD DS)'
    module: 'モジュール 2: WVD インフラストラクチャを実装する'
---

# ラボ - セッション ホスト イメージを作成および管理する (AD DS)
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプション内で所有者または共同作成者のロールを持つ Microsoft アカウントまたは Azure AD アカウント、この Azure サブスクリプションに関連付けられた Azure AD テナント内でグローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD アカウント。
- 実施するラボ - **Azure Virtual Desktop (AD DS) のデプロイを準備する**

## 推定所要時間

60 分

## ラボ シナリオ

Active Directory ドメイン サービス (AD DS) 環境で Azure Virtual Desktop ホスト イメージを作成および管理する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- WVD セッション ホスト イメージを作成および管理する

## ラボ ファイル

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json

## 説明

### 演習 1: セッション ホスト イメージを作成および管理する
  
この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop ホスト イメージの構成を準備する
1. Azure Bastion をデプロイする
1. Azure Virtual Desktop ホスト イメージを構成する
1. Azure Virtual Desktop ホスト イメージを作成する
1. カスタム イメージを使用して Azure Virtual Desktop ホスト プールをプロビジョニングする

#### タスク 1: Azure Virtual Desktop ホスト イメージの構成を準備する

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して 「**Cloud Shell**」 ペインを開きます。
1. **Bash** や **PowerShell** のどちらかを選択するためのプロンプトが表示されたら、**PowerShell** を選択します。 
1. ラボ コンピューターの Azure portal を表示している Web ブラウザーで、Cloud Shell ペインの PowerShell セッションから、以下を実行して、Azure Virtual Desktop ホスト イメージを含むリソース グループを作成します。

   ```powershell
   $vnetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vnetResourceGroupName).Location
   $imageResourceGroupName = 'az140-25-RG'
   New-AzResourceGroup -Location $location -Name $imageResourceGroupName
   ```

1. Azure portal の Cloud Shell ペインのツールバーで、「**ファイルのアップロード/ダウンロード**」 アイコンを選択し、ドロップダウン メニューで 「**アップロード**」 を選択し、ファイル **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json** および **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json**を Cloud Shell ホーム ディレクトリにアップロードします。
1. Cloud Shell ペインの PowerShell セッションから、次のコマンドを実行して、Azure Virtual Desktop クライアントとして機能する Windows 10 を実行している Azure VM を新しく作成されたサブネットにデプロイします。

   ```powershell
   New-AzResourceGroupDeployment `
     -ResourceGroupName $imageResourceGroupName `
     -Name az140lab0205vmDeployment `
     -TemplateFile $HOME/az140-25_azuredeployvm25.json `
     -TemplateParameterFile $HOME/az140-25_azuredeployvm25.parameters.json
   ```

   > **注**: 次の演習を進める前に、デプロイが完了するのを待ちます。デプロイには約 10 分間かかります。

#### タスク 2: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用して、以前のタスクでデプロイしたパブリック エンドポイントなしで、Azure VM に接続できます。一方、オペレーティング システム レベルの資格情報をターゲットとするブルート フォース攻撃に対して保護します。

> **注**: ブラウザーでポップアップ機能が有効になっていることを確認します。

1. Azure portal を表示しているブラウザー ウィンドウで、別のタブを開き、ブラウザー タブで、Azure portal に移動します。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して **「Cloud Shell」** ペインを開きます。
1. Cloud Shell ペインの PowerShell セッションで、以下を実行して、**AzureBastionSubnet** という名前のサブネットをこの演習の前半で作成した **az140-25-vnet** という名前の仮想ネットワークに追加します。

   ```powershell
   $resourceGroupName = 'az140-25-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-25-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.25.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cloud Shell ペインを閉じます。
1. Azure portal で **「Bastions」** を選択し、**「Bastions」** ブレードで **「+ 作成」** を選択します。
1. **「Bastion の作成」** ブレードの **「基本」**  タブで、次の設定を指定して、**「確認および作成」** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|**az140-25-RG**|
   |名前|**az140-25-bastion**|
   |リージョン|この演習の前のタスクでリソースにデプロイした Azure リージョンと同じです|
   |階層|**Basic**|
   |仮想ネットワーク|**az140-25-vnet**|
   |サブネット|**AzureBastionSubnet (10.25.254.0/24)**|
   |パブリック IP アドレス|**新規作成**|
   |Public IP name|**az140-25-vnet-ip**|

1. **「Bastion の作成」** ブレードの **「確認および作成」** タブで、**「作成」** を選択します。

   > **注**: 次の演習を進める前に、デプロイが完了するのを待ちます。デプロイには約 5 分間かかります。

#### タスク 3: Azure Virtual Desktop ホスト イメージを構成する

1. Azure portalで、「**仮想マシン**」を検索して選択し、**「仮想マシン」** ブレードで、**az140-25-vm0** を選択します。
1. **az140-25-vm0** ブレードで、「**接続**」を選択し、ドロップダウン メニューで、「**Bastion**」を選択し、「**az140-25-vm0 \| 接続**」ブレードの Bastion タブで、「**Bastion の使用**」を選択します。
1. プロンプトが表示されたら、次の資格情報を入力して、「**接続**」を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**Student**|
   |パスワード|**Pa55w.rd1234**|

   > **注**: まず、FSLogix バイナリをインストールします。

1. **az140-25-vm0** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. **az140-25-vm0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、イメージを構成するための一時的な場所として使用するフォルダーを作成します。

   ```powershell
   New-Item -Type Directory -Path 'C:\Allfiles\Labs\02' -Force
   ```

1. **az140-25-vm0** へのリモート デスクトップ セッション内で、Microsoft Edge を起動し [FSLogix ダウンロード ページ](https://aka.ms/fslogix_download)を参照し、FSLogix 圧縮インストール バイナリを **C:\\Allfiles\\Labs\\02** フォルダーにダウンロードし、ファイル エクスプローラーから **x64** サブフォルダーを同じフォルダーに抽出します。
1. **az140-25-vm0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 ウィンドウに切り替え、「**管理者: Windows PowerShell ISE**」 コンソールで、以下を実行して、OneDrive のマシンごとのインストールを実行します。

   ```powershell
   Start-Process -FilePath 'C:\Allfiles\Labs\02\x64\Release\FSLogixAppsSetup.exe' -ArgumentList '/quiet' -Wait
   ```

   > **注**: インストールが完了するまで待ちます。1 分間程度かかる場合があります。インストールによって再起動がトリガーされた場合は、**az140-25-vm0** に再接続します。

   > **注**: 次に、Microsoft Teams のインストールと構成を順を追って説明します (このラボで使用するイメージには Teams が既に存在するため、学習目的で)。

1. **az140-25-vm0** へのリモート デスクトップ セッション内で、**「スタート」** を右クリックし、右クリック メニューで **「ファイル名を指定して実行」** を選択し、**「ファイル名を指定して実行」** ダイアログ ボックスの **「開く」** テキストボックスに **cmd** と入力し、**Enter** キーを押して**コマンド プロンプト**を開始します。
1. 「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウで、コマンド プロンプトから次のコマンドを実行して、Microsoft Teams のマシンごとのインストールの準備をします。

   ```cmd
   reg add "HKLM\Software\Microsoft\Teams" /v IsWVDEnvironment /t REG_DWORD /d 1 /f
   ```

1. **az140-25-vm0** へのリモート デスクトップセッション内で、Microsoft Edge で、[Microsoft Visual C ++ 再頒布可能パッケージのダウンロード ページ](https://aka.ms/vs/16/release/vc_redist.x64.exe)を参照し、**VC_redist.x64** を **C:\\Allfiles\\Labs\\02** フォルダーに保存します。
1. **az140-25-vm0** へのリモート デスクトップ セッション内で、「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウで、コマンド プロンプトから次のコマンドを実行して、Microsoft Visual C ++ 再頒布可能パッケージのインストールを実行します。

   ```cmd
   C:\Allfiles\Labs\02\vc_redist.x64.exe /install /passive /norestart /log C:\Allfiles\Labs\02\vc_redist.log
   ```

1. **az140-25-vm0** へのリモート デスクトップ セッション内で、Microsoft Edge で、以下のタイトルのドキュメント ページを参照します。
[に展開し Teams デスクトップアプリを VM](https://docs.microsoft.com/ja-jp/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm)、**64 ビット バージョン**のリンクをクリックして、プロンプトが表示されたら、**Teams_windows_x64.msi** ファイルを **C:\\Allfiles\\Labs\\02** フォルダーに保存します。
1. **az140-25-vm0** へのリモート デスクトップ セッション内で、「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウで、コマンド プロンプトから次のコマンドを実行して、Microsoft Teams のマシンごとのインストールを実行します。

   ```cmd
   msiexec /i C:\Allfiles\Labs\02\Teams_windows_x64.msi /l*v C:\Allfiles\Labs\02\Teams.log ALLUSER=1
   ```

   > **注**: インストーラーは、ALLUSER = 1 および ALLUSERS = 1 パラメーターをサポートします。ALLUSER = 1 パラメーターは、VDI 環境でのマシンごとのインストールを目的としています。ALLUSERS = 1 パラメーターは、VDI 以外の環境と VDI 環境で使用できます。 

1. **az140-25-vm0** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE**」コンソールで、次のコマンドを実行して Microsoft Edge Chromium をインストールします (このラボで使用するイメージには Edge が既に存在するため、学習目的で)。

   ```powershell
   Start-BitsTransfer -Source "https://aka.ms/edge-msi" -Destination 'C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi'
   Start-Process -Wait -Filepath msiexec.exe -Argumentlist "/i C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi /q"
   ```

   > **注**: インストールが完了するまで待ちます。これにはおよそ 2 分かかる場合があります。

   > **注**: 多言語環境で操作する場合は、言語パックのインストールが必要になる場合があります。この手順の詳細については、Microsoft Docs の記事「[Windows 10 マルチセッションイメージへの言語パックの追加](https://docs.microsoft.com/ja-jp/azure/virtual-desktop/language-packs)」を参照してください。

   > **注**: 次に、Windows の自動更新を無効にし、ストレージ センサーを無効にし、タイムゾーンのリダイレクトを構成し、テレメトリの収集を構成します。一般に、最初に現在のすべての更新を最初に適用する必要があります。このラボでは、ラボの期間を最小限に抑えるために、この手順をスキップします。

1. **az140-25-vm0** へのリモート デスクトップ セッション内で、「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウに切り替え、コマンド プロンプトから次のコマンドを実行して、自動更新を無効にします。

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
   ```

1. 「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウで、コマンド プロンプトから次のコマンドを実行して、ストレージ センサーを無効にします。

   ```cmd
   reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
   ```

1. 「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウで、コマンド プロンプトから次のコマンドを実行して、タイム ゾーン リダイレクトを構成します。

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
   ```

1. 「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウで、コマンド プロンプトから次のコマンドを実行して、テレメトリ データのフィードバック ハブ収集を無効にします。

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
   ```

1. 「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウで、コマンド プロンプトから次のコマンドを実行して、このタスクの前半で作成した一時フォルダーを削除します。

   ```cmd
   rmdir C:\Allfiles /s /q
   ```

1. 「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウで、コマンド プロンプトから、ディスク クリーンアップ ユーティリティを実行し、完了したら **「OK」** をクリックします。

   ```cmd
   cleanmgr /d C: /verylowdisk
   ```

#### タスク 4: Azure Virtual Desktop ホスト イメージを作成する

1. **az140-25-vm0** へのリモート デスクトップ セッション内の 「**管理者: C:\windows\system32\cmd.exe**」 ウィンドウで、コマンド プロンプトから、イメージを生成するためのオペレーティング システムを準備し、自動的にシャットダウンするために、sysprep ユーティリティを実行します。

   ```cmd
   C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown
   ```

   > **注**: sysprep プロセスが完了するまで待ちます。これにはおよそ 2 分かかる場合があります。これにより、オペレーティング システムが自動的にシャットダウンされます。 

1. ラボ コンピューターの Azure portal を表示する Web ブラウザーで、**仮想マシン**を検索して選択し、**「仮想マシン」** ブレードから **az140-25-vm0** を選択します。
1. **az140-25-vm0** ブレードの **「Essentials」** セクションの上のツールバーで **「更新」** をクリックし、Azure VM の**ステータス**が **「停止」** に変更されたことを確認して **「停止」** をクリックします。確認を求められたら、**「OK」** をクリックして、Azure VM を**停止 (割り当て解除)** 状態に移行します。
1. **az140-25-vm0** ブレードで、Azure VM の**ステータス**が**停止 (割り当て解除)** 状態に変更されたことを確認し、ツールバーで **「キャプチャ」** をクリックします。これにより、**「イメージの作成」** ブレードが自動的に表示されます。
1. **「イメージの作成」** ブレードの **「基本」** タブで、次の設定を指定します。

   |設定|値|
   |---|---|
   |Azure コンピューティング ギャラリーにイメージを共有します|**はい、イメージ バージョンとしてギャラリーに共有します**|
   |イメージの作成後、この仮想マシンを自動的に削除します|チェックボックスがオフになっている|
   |ターゲット Azure コンピューティング ギャラリー|新しいギャラリーの名前 **az14025imagegallery**|
   |オペレーティング·システムの状態|**一般化**|

1. **「イメージの作成」** ブレードの **「基本」** タブで、**「ターゲット イメージ定義」** テキストボックスの下にある **「新規作成」** をクリックします。
1. **「イメージ定義の作成」** で、次の設定を指定して **「OK」** をクリックします。

   |設定|値|
   |---|---|
   |イメージ定義名|**az140-25-host-image**|
   |Publisher|**MicrosoftWindowsDesktop**|
   |プラン|**office-365**|
   |SKU|**20h1-evd-o365pp**|

1. **「イメージの作成」** ブレードの **「基本」** タブで、次の設定を指定し、**「確認および作成」** を選択します。

   |設定|値|
   |---|---|
   |バージョン番号|**1.0.0**|
   |最新から除外|チェックボックスがオフになっている|
   |有効期限の終了日|現在の日付から 1 年先|
   |既定のレプリカ数|**1**|
   |ターゲット リージョンのレプリカ数|**1**|
   |ストレージ アカウント タイプ|**Premium SSD**|

1. **「イメージの作成」** ブレードの **「確認および作成」** タブで、**「作成」** をクリックします。

   > **注**: デプロイが完了するのを待ちます。これにはおよそ 20 分かかる場合があります。

1. ラボ コンピューターの Azure portal を表示している Web ブラウザーで、**Azure コンピューティング ギャラリー**を検索して選択し、**Azure コンピューティング ギャラリー** ブレードで、**az14025imagegallery** エントリを選択し、****az14025imagegallery**** ブレードで、新しく作成されたイメージを表す **az140-25-host-image** エントリが存在していることを確認します。

#### タスク 5: カスタム イメージを使用して Azure Virtual Desktop ホスト プールをプロビジョニングする

1. ラボ コンピューターからで、Azure portal で、**「リソース、サービス、およびドキュメントの検索」** テキストボックスを使用して、**仮想ネットワーク**を検索して移動し、**「仮想ネットワーク」** ブレードで **az140-adds-vnet11** を選択します。 
1. **「az140-adds-vnet11」** ブレードで **「サブネット」** を選択し、**「サブネット」** ブレードで **「+ サブネット」** を選択し、**「サブネットの追加」** ブレードで次の設定を指定し (他のすべての設定はデフォルト値のままにします)、**「保存」** をクリックします。

   |設定|値|
   |---|---|
   |名前|**hp4-Subnet**|
   |サブネット アドレス範囲|**10.0.4.0/24**|

1. ラボ コンピューターから、Azure portal の Azure portal を表示する Web ブラウザーの画面で、**Azure Virtual Desktop** を検索して選択し、**「Azure Virtual Desktop」** ブレードで **「ホスト プール」** を選択し、**「Azure Virtual Desktop \| ホスト プール」** ブレードで **「+ 追加」** を選択します。 
1. **「ホスト プールの作成」** ブレードの **「基本」** タブで、次の設定を指定し、**「次へ: 仮想マシン >」** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|**az140-25-RG**|
   |ホストプール名|**az140-25-hp4**|
   |場所|このラボの最初の演習でリソースをデプロイした Azure リージョンの名前|
   |検証環境|**いいえ**|
   |ホスト プールの種類|**プールされた**|
   |最大セッション数|**50**|
   |負荷分散アルゴリズム|**幅優先**|

1. **「ホスト プールの作成」** ブレードの **「仮想マシン」** タブで、次の設定を指定します。

   |設定|値|
   |---|---|
   |仮想マシンの追加|**はい**|
   |リソース グループ|**既定はホストプールと同じ**|
   |名前のプレフィックス|**az140-25-p4**|
   |仮想マシンの場所|このラボの最初の演習でリソースをデプロイした Azure リージョンの名前|
   |可用性オプション|インフラストラクチャの冗長性は必要ありません|
   |イメージの種類|**ギャラリー**|

1. **「ホスト プールの作成」** ブレードの **「仮想マシン」** タブで、**「イメージ」** ドロップダウン リストのすぐ下にある **「すべてのイメージを表示」** リンクをクリックします。
1. **「イメージの選択」** ブレードで **「マイ アイテム」** タブをクリックし、**「共有イメージ」** をクリックして、共有イメージのリストで **「az140-25-host-image」** を選択します。 
1. **「ホスト プールの作成」** ブレードの **「仮想マシン」** タブに戻り、次の設定を指定し、**「次へ: ワークスペース >」** を選択します。

   |設定|値|
   |---|---|
   |仮想マシンのサイズ|**Standard D2s v3**|
   |VM の数|**1**|
   |OS ディスクの種類|**Standard SSD LRS**|
   |仮想ネットワーク|**az140-adds-vnet11**|
   |サブネット|**hp4-Subnet (10.0.4.0/24)**|
   |ネットワーク セキュリティ グループ|**Basic**|
   |パブリック受信ポート|**はい**|
   |許可する受信ポート|**RDP**|
   |AD ドメインの UPN への参加|**student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|
   |特定のドメインまたはユニット|**はい**|
   |参加するドメイン|**adatum.com**|
   |組織単位パス|**OU=WVDInfra,DC=adatum,DC=com**|
   |ユーザー名|Student|
   |パスワード|Pa55w.rd1234|
   |パスワードの確認|Pa55w.rd1234|

1. **「ホスト プールの作成」** ブレードの **「ワークスペース」** タブで、次の設定を指定し、**「確認および作成」** を選択します。

   |設定|値|
   |---|---|
   |デスクトップ アプリ グループを登録する|**いいえ**|

1. **「ホストプールの作成」** ブレードの **「確認および作成」** タブで、**「作成」** を選択します。

   > **注**: デプロイが完了するのを待ちます。これにはおよそ 10 分かかる場合があります。

   > **注**: カスタム イメージに基づいてホストを展開した後、[GitHub リポジトリ](https://github.com/The-Virtual-Desktop-Team/)から入手できる仮想デスクトップ最適化ツールの実行を検討する必要があります。


### 演習 2: ラボでプロビジョニングされた Azure VM を停止および割り当て解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングされた Azure VM を停止および割り当て解除する

> **注**: この演習では、このラボでプロビジョニングした Azure VM を割り当て解除し、対応するコンピューティング料金を最小化します

#### タスク 1: ラボでプロビジョニングされた Azure VM を割り当て解除する

1. ラボ コンピューターに切り替え、Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG'
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成されたすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG' | Stop-AzVM -NoWait -Force
   ```

   > **注**: コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
