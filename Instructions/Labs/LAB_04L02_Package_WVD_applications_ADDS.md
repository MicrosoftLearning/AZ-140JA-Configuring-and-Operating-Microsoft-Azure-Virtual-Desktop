---
lab:
    title: 'ラボ: Windows Virtual Desktop アプリケーション (AD DS) のパッケージ化'
    module: 'モジュール 4: ユーザーの環境とアプリを管理する'
---

# ラボ - Windows Virtual Desktop アプリケーション (AD DS) のパッケージ化。
# 受講生用ラボ マニュアル

## ラボの依存関係

- Azure サブスクリプション
- Azure サブスクリプションに関連付けられた Azure AD テナント内で グローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD、あるいは Azure サブスクリプション内 で所有者または共同作成はのロールを持つ Microsoft アカウントまたは Azure AD
- 実施するラボ - **Azure Windows Virtual Desktop (AD DS) のデプロイを準備する**または **Azure Windows Virtual Desktop (Azure AD DS) のデプロイを準備する**
- 実施するラボ - **Windows Virtual Desktop プロファイルの管理 (AD DS)** または **Windows Virtual Desktop プロファイルの管理 (Azure AD DS)**

> **注**: このラボを作成している時点で、Windows Virtual Desktop に対する MSIX アプリのアタッチ機能はパブリック プレビュー段階です。試行するには、サブスクリプションで MSIX アプリのタッチを有効にするために、[オンライン フォーム](https://aka.ms/enablemsixappattach)を介して要求を送信する必要があります。要求の承認と処理には、最大 24 営業時間かかることがあります。要求が受入および完了すると、メール確認が送信されます。

## 推定所要時間

90 分

## ラボ シナリオ

Windows Virtual Desktop アプリケーションを Active Directory ドメイン サービス (AD DS) 環境にパッケージ化して展開する必要があります。

## 目標
  
このラボを終了すると、以下ができるようになります。

- MSIX アプリ パッケージを準備および作成する
- AD DS 環境に Windows Virtual Desktop 用 MSIX アプリのアタッチ コンテナーを実装する
- AD DS 環境の Windows Virtual Desktop に MSIX アプリのアタッチを実装する

## ラボ ファイル

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## 説明

### 演習 1: MSIX アプリ パッケージを準備および作成する

このエクササイズの主なタスクは次のとおりです。

1. Windows Virtual Desktop セッション ホストの構成を準備する
1. Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする
1. Windows 10 を実行している Azure VM を MSIX パッケージ用に準備する
1. 署名証明書を生成する
1. パッケージ化するソフトウェアをダウンロードする
1. MSIX パッケージ ツールをインストールする
1. MSIX パッケージの作成

#### タスク 1: Windows Virtual Desktop セッション ホストの構成を準備する

1. ラボのコンピューターから Web ブラウザーを起動し、 [Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザーアカウントの認証情報を提供してサインインします。
1. ラボ コンピューターにの Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで使用する Azure VM をホストする Windows Virtual Desktop セッションを開始します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**注**: コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に開始されるまでに数分かかります。 

   >**注**: Azure VM が起動するのを待たずに、次のタスクに直接進みます。

#### タスク 2: Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする

1. ラボ コンピューターから、Azure portal を表示している Web ブラウザー ウィンドウの Cloud Shell ペインのツールバーで、「**ファイルのアップロード/ダウンロード**」 アイコンを選択し、ドロップダウン メニューで 「**アップロード**」 を選択し、ファイル \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeployvm25.json および \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeployvm25.parameters.jsonを Cloud Shell ホーム ディレクトリにアップロードします。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、MSIX パッケージの作成に使用する Windows 10 を実行している Azure VM をデプロイし、Azure AD DS ドメインに参加させます。

   ```powershell
   $vNetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vNetResourceGroupName).Location
   $resourceGroupName = 'az140-42-RG'
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0402vmDeployment `
     -TemplateFile $HOME/az140-42_azuredeploycl42.json `
     -TemplateParameterFile $HOME/az140-42_azuredeploycl42.parameters.json
   ```

   > **注**: 次のタスクを進める前に、デプロイが完了するのを待ちます。10 分間程度かかる場合があります。 

#### タスク 3: Windows 10 を実行している Azure VM を MSIX パッケージ用に準備する

1. ラボ コンピューターの Azure portal で、**仮想マシン**を検索して選択し、**「仮想マシン」** ブレードの仮想マシンのリストで **az140-cl-vm42** エントリを選択します。これにより、**az140-cl-vm42** ブレードが開きます。
1. **「az140-cl-vm42」** ブレードで、**「接続」**を選択し、ドロップダウン メニューで **「RDP」** を選択し、**「az140-cl-vm42 \| 接続**」 ブレードの **「RDP」** タブで、**「RDPファイルのダウンロード」** を選択します。
1. プロンプトが表示されたら、次の資格情報でサインインします。

   |設定|値|
   |---|---|
   |ユーザー名|**ADATUM\\wvdadmin1**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE**」 コンソールで、以下を実行して、MSIX パッケージ用のオペレーティング システムを準備します。

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   ```

   > **注**: これらのレジストリの最後の変更により、ユーザー アクセス制御が無効になります。これは技術的には必須ではありませんが、このラボで説明するプロセスを簡素化します。

#### タスク 4: 署名証明書を生成する

> **注**: このラボでは、自己署名証明書を使用します。実稼働環境では、使用目的に応じて、公的認証局または内部認証局のいずれかによって発行された証明書を使用する必要があります。

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、共通名属性が **Adatum** に設定された自己署名証明書を生成し、**ローカル マシン**証明書ストアの**個人用**フォルダーに証明書を保存します。

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: **Windows PowerShell ISE」 コンソールで、次のコマンドを実行して、ローカル マシンの証明書ストアを対象とする**証明書**コンソールを起動します。

   ```powershell
   certlm.msc
   ```

1. **「証明書」** コンソール ペインで、**「個人用」** フォルダーを展開し、**「証明書」** サブフォルダーを選択して、**Adatum** 証明書を右クリックし、右クリック メニューで **「すべてのタスク」**、**「エクスポート」** の順に選択します。これにより、**証明書エクスポート ウィザード**が起動します。 
1. **証明書エクスポート ウィザード**の **「証明書エクスポート ウィザードへようこそ」** ページで、**「次へ」**を選択します。
1. **証明書エクスポート ウィザード**の **「秘密鍵のエクスポート」** ページで、**「はい、秘密鍵をエクスポートします」** オプションを選択し、**「次へ」** を選択します。
1. **「証明書エクスポート」 ウィザード**の **「ファイル形式のエクスポート」** ページで、**「すべての拡張プロパティをエクスポートする」** チェックボックスをオンにし、**「証明書のプライバシーを有効にする」** チェックボックスをオフにして、**「次へ」** を選択します。
1. 「**証明書エクスポート」 ウィザード**の **「セキュリティ」** ページで、**「パスワード」** チェックボックスを選択し、下のテキストボックスに「**Pa55w.rd1234**」と入力して、**「次へ」** を選択します。
1. **「証明書エクスポート」ウィザード**の **「エクスポートするファイル」** ページの **「ファイル名」** テキストボックスで **「参照」** を選択し、**「名前を付けて保存」** ダイアログ ボックスで、**C:\\Allfiles\\Labs\\04** フォルダーに移動し (必要に応じてフォルダーを作成します)、**「ファイル名」** テキストボックスに **adatum.pfx** と入力し、**「保存」** を選択します。
1. **「証明書エクスポート」 ウィザード**の **「エクスポートするファイル」** ページに戻り、テキストボックスにエントリ **C:\\Allfiles\\Labs\\04\\adatum.pfx** が含まれていることを確認して、**「次へ」** を選択します。
1. **「証明書エクスポート」 ウィザード**の **「証明書エクスポートウィザードの完了」** ページで、**「完了」** を選択し、**「OK」** を選択してエクスポートが成功したことを確認します。 

   > **注**: 自己署名証明書を使用しているため、ターゲット セッション ホストの **Trusted People** 証明書ストアにインストールする必要があります。

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、新しく生成された証明書をターゲット セッション ホストの **Trusted People** 証明書ストアにインストールします。

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString $cleartextPassword -AsPlainText -Force
   ForEach ($wvdhost in $wvdhosts){
      $localPath = 'C:\Allfiles\Labs\04'
      $remotePath = "\\$wvdhost\C$\Allfiles\Labs\04\"
      Copy-Item -Path "$localPath\adatum.pfx" -Destination $remotePath -Force
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Import-PFXCertificate -CertStoreLocation Cert:\LocalMachine\TrustedPeople -FilePath 'C:\Allfiles\Labs\04\adatum.pfx' -Password $using:securePassword
      } 
   }
   ```

#### タスク 5: パッケージ化するソフトウェアをダウンロードする

1. **az140-dc-vm42** へのリモート デスクトップ セッション内で、Microsoft Edge を起動して、**https://github.com/microsoft/XmlNotepad** にアクセスします。
1. **microsoft/XmlNotepad** **readme.md** ページで、「スタンドアロンのダウンロード可能なインストーラー」(http://www.lovettsoftware.com/downloads/xmlnotepad/xmlnotepadsetup.zip)のダウンロードリンクを選択し、圧縮されたインストールファイルをダウンロードします。
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、ファイル エクスプローラーを起動し、**ダウンロード** フォルダーに移動し、圧縮ファイルを開き、そのコンテンツをコピーして、フォルダー **C:\\AllFiles\\Labs\\04\\** を作成し、コピーしたコンテンツを新しく作成したフォルダーに貼り付けます。 

#### タスク 6: MSIX パッケージ ツールをインストールする

1. **az140-cl-vm42** へのリモート デスクトップセッション内で、**Microsoft Store** アプリを起動します。
1. **Microsoft Store** アプリで、**MSIX パッケージ ツール**を検索して選択し、**「MSIX パッケージ ツール」** ページで **「取得」** を選択します。
1. プロンプトが表示されたら、サインインをスキップし、インストールが完了するのを待ち、**「起動」** を選択し、**「診断データの送信」** ダイアログボックスで **「拒否」** を選択します。 

#### タスク 7: MSIX パッケージの作成

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 ウィンドウに切り替え、「**管理者: Windows PowerShell ISE**」 スクリプト ペインから、以下を実行して、Windows Search サービスを無効にします。

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、以下を実行して、MSIX パッケージをホストするフォルダーを作成します。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、値が 3 の Zone.Identifier 代替データストリームを削除します。これは、インターネットからダウンロードされたことを示します。

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、**MSIX パッケージ ツール インターフェイス**に切り替え、**「タスクの選択」** ページで**「アプリケーション パッケージ」 - 「アプリ パッケージ エントリの作成」** を選択します。これにより、**新しいパッケージの作成**ウィザードが起動します。
1. **「新しいパッケージの作成」** ウィザードの **「環境の選択」** ページで、「このコンピューターにパッケージを作成する」 オプションが選択されていることを確認し、**「次へ」** を選択して、**MSIX パッケージ作成ツールドライバー**のインストールを待ちます。
1. **「新しいパッケージの作成」** ウィザードの **「コンピューターの準備」** ページで、推奨事項を確認します。保留中の再起動がある場合は、オペレーティング システムを再起動し、**ADATUM\wvdadmin1** アカウントを使用して、再度サインインし、**MSIX パッケージ ツール**を再起動してから続行します。 

   >** 注**: MSIX パッケージ ツールは、Windows Update と Windows Search を一時的に無効にします。この場合、Windows Search サービスはすでに無効になっています。 

1. **「新しいパッケージの作成」** ウィザードの **「コンピューターの準備」** ページで、**「次へ」** をクリックします。
1. **「新しいパッケージの作成」** ウィザードの **「インストーラーの選択」** ページで、**「パッケージ化するインストーラーの選択」** テキスト ボックスの横にある **「参照」** を選択し、**「開く」** ダイアログ ボックスで **C:\\AllFiles\\Labs\\04** フォルダーを参照し、**XmlNotepadSetup.msi** を選択して、**「開く」** をクリックします。 
1. **「新しいパッケージの作成」** ウィザードの **「インストーラーの選択」** ページの **「署名設定」** ドロップダウン リストで、**「証明書を使用して署名 (.pfx)」** エントリを選択し、**「証明書の参照」** テキストボックスの横にある **「参照」** を選択して、**「開く」** ダイアログ ボックスで、**C:\\AllFiles\\Labs\\04** フォルダーに移動し、**adatum.pfx** ファイルを選択して、**「開く」** をクリックし、**「パスワード」** テキスト ボックスに **Pa55w.rd1234** と入力して、**「次へ」** を選択します。
1. **「新しいパッケージの作成」** ウィザードの **「パッケージ情報」** ページで、パッケージ情報を確認し、発行元名が **CN=Adatum** に設定されていることを確認して、**「次へ」** を選択します。これにより、ダウンロードしたソフトウェアのインストールがトリガーされます。
1. **「XML Notepad セットアップ」** ウィンドウで、使用許諾契約の条項に同意して **「インストール」** を選択し、インストールが完了したら、**「XML Notepad の起動」** チェックボックスを選択して **「完了」** を選択します。
1. XML Notepad が実行されていることを確認して閉じ、****「MSIX パッケージ ツール」**** ウィンドウで **「新しいパッケージの作成」** ウィザードに戻り、**「次へ」** を選択します。

   > **注**: この場合、インストールを完了するために再起動する必要はありません。

1. **「新しいパッケージの作成」** ウィザードの **「最初の起動タスク」** ページで、提供された情報を確認し、**「次へ」** を選択します。
1. 「**完了しましたか?**」というプロンプトが表示されたら、**「はい」** を選択して次に進みます。
1. **「新しいパッケージの作成」** ウィザードの **「サービス レポート」** ページで、サービスがリストされていないことを確認し、**「次へ」** を選択します。
1. **「新しいパッケージの作成」** ウィザードの **「パッケージの作成」** ページの **「保存場所」** テキストボックスに、**C:\\Allfiles\\Labs\\04\\Xml Notepad** と入力し、**「作成」** をクリックします。
1. **「パッケージが正常に作成されました」** ダイアログ ボックスで、保存されたパッケージの場所をメモし、**「閉じる」** を選択します。
1. ファイル エクスプローラー ウィンドウに切り替え、**C:\\Allfiles\\Labs\\04\\Xml Notepad** フォルダーに移動し、*msix* ファイルと *xml* ファイルが含まれていることを確認します。 


### 演習 2: Azure AD DS 環境に Windows Virtual Desktop 用 MSIX アプリのアタッチ コンテナーを実装する

このエクササイズの主なタスクは次のとおりです。

1. Window 10 Enterprise Edition を実行している Azure VM で Hyper-V を有効にする
1. アプリ アタッチ コンテナーを作成する

#### タスク 1: Window 10 Enterprise Edition を実行している Azure VM で Hyper-V を有効にする

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE**」 コンソールで、以下を実行して、MSIX アプリのアタッチ用のターゲット Windows Virtual Desktop ホストを準備します。 

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
         reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
         reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
         reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
         Set-Service -Name wuauserv -StartupType Disabled
      }
   }
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、Hyper-V とその管理ツール (Hyper-V PowerShell モジュールを含む) を Windows Virtual Desktop ホストにインストールします。

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. 各ホストに Hyper-V コンポーネントをインストールした後、**Y** と入力し、**Enter** キーを押して、ターゲット オペレーティング システムを再起動します。
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、Hyper-V とその管理ツール (Hyper-V PowerShell モジュールを含む) をローカル コンピューターにインストールします。

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Hyper-V コンポーネントのインストールが完了したら、**Y** と入力し、**Enter** キーを押してオペレーティング システムを再起動します。再起動後、パスワード **PA55w.rd1234** で ****ADATUM\wvdadmin1**** アカウントを使用して再度サインインします。

#### タスク 2: アプリ アタッチ コンテナーを作成する

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、**Microsoft Edge** を起動し、**https://aka.ms/msixmgr** を参照し、**msixmgr.zip** ファイルを開くか保存するかを確認するメッセージが表示されたら、**「保存」** をクリックします。これにより、MSIX mgr ツール アーカイブが **Downloads** フォルダーにダウンロードされます。
1. ファイル エクスプローラーで、**ダウンロード** フォルダーに移動し、圧縮ファイルを開いて、**x64** フォルダーを **C:\\AllFiles\\Labs\\04** フォルダーにコピーします。 
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、アプリのアタッチ コンテナーとして機能する VHD ファイルを作成します。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   New-VHD -SizeBytes 128MB -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Dynamic -Confirm:$false
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインから、以下を実行して、新しく作成された VHD ファイルをマウントします。

   ```powershell
   $vhdObject = Mount-VHD -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Passthru
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行してディスクを初期化し、新しいパーティションを作成してフォーマットし、使用可能な最初のドライブ文字を割り当てます。

   ```powershell
   $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
   $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
   Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、MSIX ファイルをホストするフォルダー構造を作成し、前のタスクで作成した MSIX パッケージをその中に解凍します。

   ```powershell
   $appName = 'XmlNotepad'
   $msixPackage = Get-ChildItem -Path "C:\AllFiles\Labs\04\$appName" -Filter *msix -File 
   C:\AllFiles\Labs\04\x64\msixmgr.exe -Unpack -packagePath $msixPackage.FullName -destination "$($partition.DriveLetter):\Apps\$appName" -applyacls
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、ファイル エクスプローラーで、**F:\\Apps\\Xml Noteppad** フォルダーに移動し、その内容を確認します。
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、解凍された MSIX パッケージをホストしているボリュームの GUID を特定します。

   ```powershell
   $uniqueId = (Get-Volume -DriveLetter "$($partition.DriveLetter)").UniqueId
   $regex = [regex]"\{(.*)\}"
   [regex]::match($uniqueId, $regex).Groups[1].value
   ```

   > **注**: 識別した GUID 値を記録します。これは、このラボの次の演習で必要となります。 

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、アプリのアタッチ コンテナーとして機能する VHD ファイルをマウント解除成します。

   ```powershell
   Dismount-VHD -Path "C:\Allfiles\Labs\04\MSIXVhds\$appName.vhd" -Confirm:$false
   ```

### 演習 3: Windows Virtual Desktop セッションホストに MSIX アプリのアタッチを実装する

このエクササイズの主なタスクは次のとおりです。

1. Windows Virtual Desktop ホストを含む Active Directory グループを構成する
1. MSIX アプリのアタッチの Azure Files 共有を設定する
1. MSIX アプリのアタッチ コンテナーを Windows Virtual Desktop セッションホストにマウントして登録する
1. MSIX アプリをアプリケーション グループに公開する
1. MSIX アプリのアタッチ機能を検証する

#### タスク 1: Windows Virtual Desktop ホストを含む Active Directory グループを構成する

1. ラボ コンピューターに切り替え、Azure portal を表示する Web ブラウザーで、**仮想マシン**を検索して選択し、**「仮想マシン」** ブレードから **az140-dc-vm11** を選択します。
1. **「az140-dc-vm11」** ブレードで、**「接続」** を選択し、ドロップダウン メニューで **「RDP」** を選択し、**「az140-vm11 \| 接続**」 ブレードの 「RDP」 タブの **「IP アドレス」** ドロップダウン リストで、**「ロード バランサ―の DNS 名」** エントリ、次に **「RDP ファイルをダウンロード」** を選択します。
1. プロンプトが表示されたら、次の資格情報でサインインします。

   |設定|値|
   |---|---|
   |ユーザー名|**ADATUM\\Student**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、このラボで使用する Azure AD テナントに同期される AD DS グループ オブジェクトを作成します。

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **注**: このグループを使用して、Windows Virtual Desktop ホストに **az140-42-msixvhds** ファイル共有へのアクセス許可を付与します。

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次の手順を実行して、前の手順で作成したグループにメンバーを追加します。

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次を実行して、'az140-hosts-42-p1' グループのメンバーであるサーバーを再起動します。

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | Restart-Computer
   ```

   > **注**: この手順により、グループ メンバーシップの変更が確実に有効になります。 

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールから、以下を実行して、**aadsyncuser** Azure AD ユーザー アカウントのユーザープリンシパル名を識別します。

   ```powershell
   (Get-AzADUser -DisplayName 'aadsyncuser').UserPrincipalName
   ```
   > **注**: この手順で特定したユーザー プリンシパル名を記録します。それはこのタスクの後半で必要になります。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内の **「スタート」** メニューで、**「Azure AD Connect」** フォルダーを展開し、**「Azure AD Connect」** を選択します。
1. **「Microsoft Azure Active Directory Connect」** ウィンドウの **「Azure AD Connect へようこそ」** ページで、**「構成」** を選択します。
1. **「Microsoft Azure Active Directory Connect」** ウィンドウの **「追加のタスク」** ページで、**「同期オプションのカスタマイズ」** を選択し、**「次へ」** を選択します。
1. **「Microsoft Azure Active Directory Connect」** ウィンドウの **「Azure AD に接続」** ページで、このタスクの前半で特定した **aadsyncuser** ユーザー アカウントのユーザー プリンシパル名と **Pa55w.rd1234** パスワードを使用して認証します。
1. **「Microsoft Azure Active Directory Connect」** ウィンドウの **「ディレクトリの接続」** ページで、**「次へ」** を選択します。
1. **「Microsoft Azure Active Directory Connect」** ウィンドウの **「ドメインと OU のフィルタリング」** ページで、「選択したドメインと OU を同期する」 オプションが選択されていることを確認し、**adatum.com** ノードを展開し、**ToSync** OU の横にあるチェックボックスが選択されていることを確認し、**WVDInfra** OU の横にあるチェックボックスを選択して (他の選択したチェックボックスは変更しないでください)、**「次へ」** を選択します。
1. **「Microsoft Azure Active Directory Connect」** ウィンドウの **「オプション機能」** ページで、既定の設定を受け入れて、**「次へ」** を選択します。
1. **「Microsoft Azure Active Directory Connect」** ウィンドウの **「構成の準備完了」** ページで、**「構成の完了時に同期プロセスを開始する」** チェックボックスがオンになっていることを確認し、**「構成」** を選択します。
1. **「構成が完了しました」** ページの情報を確認し、**「終了」** を選択し、**「Microsoft Azure Active Directory Connect」** ウィンドウを閉じます。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Internet Explorer を起動して、[Azure portal](https://portal.azure.com) に移動します。プロンプトが表示されたら、このラボで使用している Azure サブスクリプションに関連付けられている Azure AD テナントのグローバル管理者ロールを持つユーザーアカウントの Azure AD の資格情報を使用してサインインします。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示している Internet Explorer ウィンドウで、**Azure Active Directory** を検索して選択し、このラボで使用している Azure サブスクリプションに関連付けられている Azure AD テナントに移動します。
1. Azure Active Directory ブレードの左側の垂直メニューバーの **「管理」** セクションで、**「グループ」** をクリックします。 
1. 「**グループ | すべてのグループ**」 ブレードで、グループのリストから **az140-hosts-42-p1** エントリを選択します。
1. 「**az140-hosts-42-p1**」 ブレードの左側にある垂直メニュー バーの 「**管理**」 セクションで、「**メンバー**」 をクリックします。
1. 「**az140-hosts-42-p1 | メンバー**」 ブレード、**ダイレクト メンバー**のリストに、このタスクの前半でグループに追加した Windows Virtual Desktop プールの 3 つのホストが含まれていることを確認します。

#### タスク 2: MSIX アプリのアタッチの Azure Files 共有を設定する

1. ラボ コンピューターで、リモート デスクトップ セッションに戻って **az140-cl-vm42** に切り替えます。
1. **az140-cl-vm42** へのリモートデスクトップセッション内で、InPrivate モードで Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザー アカウントの認証情報を提供してサインインします。

   > **注**: 必ず Microsoft Edge InPrivate モードを使用してください。

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、Azure portal を表示する Microsoft Edge ウィンドウで **ストレージ アカウント** を検索して選択し、**「ストレージ アカウント」** ブレードで、ユーザー プロファイルをホストするように構成したストレージ アカウントを選択します。

   > **注**: ラボのこの部分は、ラボ - **Windows Virtual Desktop プロファイルの管理 (AD DS)** または **Windows Virtual Desktop プロファイルの管理 (Azure AD DS)** の完了を条件としています

   > **注**: 実稼働シナリオでは、別のストレージ アカウントの使用を検討する必要があります。これには、Azure AD DS 認証用にそのストレージ アカウントを構成する必要があります。これは、ユーザー プロファイルをホストするストレー ジアカウント用に既に実装されています。同じストレージ アカウントを使用して、個々のラボ間で重複する手順を最小限に抑えています。

1. ストレージ アカウント ブレードの左側の垂直メニューの **「ファイル サービス」** セクションで、**「ファイル共有」** を選択し、**「+ ファイル共有」** を選択します。
1. 「**新しいファイル共有**」 で、次の設定を指定し、「**作成**」 を設定します (他の設定はデフォルト値のままにします)。

   |設定|値|
   |---|---|
   |名前|**az140-42-msixvhds**|

1. Azure portal を表示している Microsoft Edge のファイル共有の一覧で、新しく作成したファイル共有を選択します。 
1. **「az140-42a-msixvhds」** ブレードの左側の垂直メニューで、**「アクセス制御 (IAM)」** を選択します。
1. ストレージ アカウントの 「**az140-42a-msixvhds \| アクセス制御 (IAM)**」 ブレードで、「**+ 追加**」 を選択し、ドロップダウン メニューで 「**ロール割り当ての追加**」 を選択します。 
1. 「**ロール割り当ての追加**」 ブレードで、次のように設定してから 「**保存**」 を選択します。

   |設定|値|
   |---|---|
   |Role|**記憶域ファイル データの SMB 共有の管理者特権共同作成者**|
   |アクセスの割り当て|**ユーザー、グループ、またはサービス プリンシパル**|
   |Select|**az140-wvd-admins**|

   > **注**: **az140-wvd-admins** グループには、共有権限を構成するために使用する **wvdadmin1** ユーザー アカウントが含まれています。 

1. 前の 2 つの手順を繰り返して、次の役割の割り当てを構成します。

   |設定|値|
   |---|---|
   |Role|**記憶域ファイル データの SMB 共有の管理者特権共同作成者**|
   |アクセスの割り当て|**ユーザー、グループ、またはサービス プリンシパル**|
   |Select|**az140-hosts-42-p1**|

   |設定|値|
   |---|---|
   |Role|**記憶域ファイル データの SMB 共有の閲覧者**|
   |アクセスの割り当て|**ユーザー、グループ、またはサービス プリンシパル**|
   |Select|**az140-wvd-users**|

   > **注**: Windows Virtual Desktop のユーザーとホストには、少なくともファイル共有への読み取りアクセス権が必要です。

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、**コマンド プロンプト**を起動し、**コマンド プロンプト** ウィンドウから次のコマンドを実行して、ドライブを az140-42-msixvhds 共有にマップし (`<storage-account-name>` プレース ホルダーをストレージ アカウントの名前に置き換えます)、コマンドが正常に完了することを確認します。

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、**「コマンド プロンプト」** ウィンドウから次のコマンドを実行して、セッション ホストのコンピューター アカウントに必要な NTFS アクセス許可を付与します。

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**注**: **ADATUM\\wvdadmin1** としてサインインしているときに、ファイル エクスプローラーを使用して、これらのアクセス許可を設定することもできます。 

   >**注**: 次に、MSIX アプリのアタッチ機能を検証します

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 ウィンドウで、次の手順を実行して、前の演習で作成した VHD ファイルをこの演習で前に作成した Azure Files 共有にコピーします。

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages' -Force
   ```

#### タスク 3: Windows Virtual Desktop ホストで MSIX アプリのアタッチのステージング スクリプトを実行する

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して PowerShellGet モジュールの最新バージョンをインストールします (確認を求められたら **「はい」** を選択します)。

   ```powershell
   「Net.ServicePointManager」::SecurityProtocol = 「Net.SecurityProtocolType」::Tls12
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

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、このラボで使用している Azure サブスクリプションを認証します。

   ```powershell
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、MSIX パッケージを含むファイル共有をホストしている Azure ストレージ アカウントの名前を特定します。

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName  
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、管理者としてコマンド プロンプトを起動し、コマンド プロンプトから次のコマンドを実行して、WinRM を有効にします (確認を求められたら、**Y** と入力し、**Enter** キーを押します。

   ```cmd
   winrm qc
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、ターゲットの Windows Remote Desktop ホストへの PowerShell リモーティングの CredSSP 認証を有効にします。

   ```powershell
   Enable-WSManCredSSP -Role client -DelegateComputer * -Force
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、ターゲットの Windows Remote Desktop ホストへの PowerShell リモーティングの CredSSP 認証を有効にします。

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
        winrm qc
        Enable-PSRemoting -Force
        Enable-WSManCredSSP -Role server -Force
      } 
   }
   ```

   >** 注**: これは、ステージング スクリプトをリモートで実行できるようにするために行われます。または、各ターゲット ホストでローカルにスクリプトを実行することもできます。

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、MSIX アプリのアタッチのステージングを実行します (`<volume_guid>` プレースホルダーを前の演習で特定したボリューム GUID に置き換えます)。

   ```powershell
   $username = 'ADATUM\wvdadmin1'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString -AsPlainText $cleartextPassword -Force
   $creds = New-Object System.Management.Automation.PSCredential -ArgumentList $username,$securePassword

   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -Authentication Credssp -Credential $creds -ScriptBlock {

         $vhdSrc = "\\$using:storageAccountName.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd"
         Mount-Diskimage -ImagePath $vhdSrc -NoDriveLetter -Access ReadOnly
         $volumeGuid = '<volume_guid>'
         $msixDest = "\\?\Volume{" + $volumeGuid + "}\"

         $parentFolder = '\Apps\XmlNotepad\'
         $msixJunction = 'C:\Allfiles\Labs\04\AppAttach\'
         $packageName = "XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8"

         If (!(Test-Path -Path $msixJunction)) {New-Item -ItemType Directory -Path $msixJunction}
         $msixJunction = $msixJunction + $packageName
         cmd.exe /c mklink /j $msixJunction $msixDest

         [Windows.Management.Deployment.PackageManager,Windows.Management.Deployment,ContentType=WindowsRuntime] | Out-Null
         Add-Type -AssemblyName System.Runtime.WindowsRuntime
         $asTask = ([System.WindowsRuntimeSystemExtensions].GetMethods() | Where { $_.ToString() -eq 'System.Threading.Tasks.Task`1[TResult] AsTask[TResult,TProgress](Windows.Foundation.IAsyncOperationWithProgress`2[TResult,TProgress])'})[0]
         $asTaskAsyncOperation = $asTask.MakeGenericMethod([Windows.Management.Deployment.DeploymentResult], [Windows.Management.Deployment.DeploymentProgress])
         $packageManager = [Windows.Management.Deployment.PackageManager]::new()
         $path = $msixJunction + $parentFolder + $packageName
         $path = ([System.Uri]$path).AbsoluteUri
         $asyncOperation = $packageManager.StagePackageAsync($path, $null, "StageInPlace")
         $task = $asTaskAsyncOperation.Invoke($null, @($asyncOperation))
         $task 
      }
   }
    ```

#### タスク 4: プール ホストの 1 つへのリモート デスクトップ セッションを開始する

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、Microsoft Edge を起動し、[Windows デスクトップ クライアントのダウンロード ページ](https://go.microsoft.com/fwlink/?linkid=2068602)に移動し、プロンプトが表示されたら、**「実行」** を選択してインストールを開始します。**リモート デスクトップ セットアップ** ウィザードの **「インストール範囲」** ページで、**「このマシンのすべてのユーザーにインストールする」** オプションを選択し、**「インストール」** をクリックします。 
1. インストールが完了したら、**「セットアップの終了時にリモート デスクトップを起動する」** チェックボックスがオンになっていることを確認し、**「完了」** をクリックしてリモート デスクトップ クライアントを起動します。
1. **リモート デスクトップ** クライアント ウィンドウで、**「サブスクライブ」** を選択し、プロンプトが表示されたら、プロンプトが表示されたら、ユーザー プリンシパル名として **aduser1**、パスワードとして **Pa55w.rd1234** を使用してサインインします。
1. プロンプトが表示されたら、**「すべてのアプリにサインインしたままにする」** ウィンドウで、**「組織にデバイスの管理を許可する」** チェックボックスをオフにし、**「いいえ、このアプリのみにサインインします」** をクリックします。
1. **リモート デスクトップ** クライアント ウィンドウの **az140-21-ws1** セクションで、SessionDesktop アイコンをダブルクリックして、**az140-21-ws1** ワークスペースの一部であるホスト プールへのリモート デスクトップ セッションを開きます。メッセージが表示されたら、**aduser1** アカウントのパスワードを入力します。

#### タスク 5: Windows Virtual Desktop ホストで MSIX アプリのアタッチの登録スクリプトを実行する

1. **az140-cl-vm42** から **az140-21-ws1** ワークスペースのホスト プールへのリモート デスクトップ セッション内で、**aduser1** としてサインインしているときに、**Windows PowerShell ISE** を起動し、**Windows PowerShell ISE** コンソールから次のコマンドを実行して、MSIX アプリのアタッチの登録を実行します。

   ```powershell
   $packageName = 'XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8'
   $path = 'C:\Program Files\WindowsApps\' + $packageName + '\AppxManifest.xml'
   Add-AppxPackage -Path $path -DisableDevelopmentMode -Register
   ```

1. **az140-cl-vm42** から **az140-21-ws1** ワークスペースのホスト プールへのリモート デスクトップ セッション内で、**ADATUM\aduser1** としてサインインしているときに、**「スタート」** をクリックし、**「スタート」** メニューで **「XML Notepad」** をクリックして、XML Notepad アプリが正常に起動することを確認します。
1. XML Notepad と **WindowsPowerShell ISE** ウィンドウを閉じ、**「スタート」** を右クリックし、右クリックメニューで **「シャットダウン」 または 「サインアウト」** を選択し、カスケード メニューで **「サインアウト」** を選択します。

#### タスク 6: MSIX アプリのアタッチの登録解除スクリプトを実行する

1. **az140-cl-vm42** へのリモート デスクトップ セッションに戻り、「**管理者: Windows PowerShell ISE**」 ウィンドウに切り替え、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次を実行して、**az140-21-hp1** ホスト プール内のすべてのホストで MSIX アプリのアタッチの登録解除を実行します。

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -Authentication Credssp -Credential $creds -ScriptBlock {
         $packageName = "XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8"
         Remove-AppxPackage -AllUsers -Package $packageName
      }
   }
    ```

1. スクリプトが正常に完了したことを確認します。

#### タスク 7: MSIX アプリのアタッチのステージング解除スクリプトを実行する

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、MSIX パッケージを含むファイル共有をホストしている Azure ストレージ アカウントの名前を特定します。

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName  
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次を実行して、**az140-21-hp1** ホスト プール内のすべてのホストで MSIX アプリのアタッチのステージング解除を実行します。

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -Authentication Credssp -Credential $creds -ScriptBlock {
         $vhdSrc = "\\$using:storageAccountName.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd"
         $packageName = "XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8"
         $msixJunction = 'C:\Allfiles\Labs\04\AppAttach'
         Remove-Item "$msixJunction\$packageName" -Recurse -Force -Verbose
         Dismount-DiskImage -ImagePath $vhdSrc -Confirm:$false
      }
   }
    ```

1. スクリプトが正常に完了したことを確認します。


### 演習 4: ラボでプロビジョニングおよび使用された Azure VM を停止および割り当て解除する

このエクササイズの主なタスクは次のとおりです。

1. ラボでプロビジョニングおよび使用された Azure VM を停止および割り当て解除する

>** 注**: この演習では、このラボでプロビジョニングおよび使用した Azure VM を割り当て解除し、対応するコンピューティング料金を最小化します

#### タスク 1: ラボでプロビジョニングおよび使用された Azure VM を割り当て解除する

1. ラボ コンピューターに切り替え、Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成および使用されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成および使用されたすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**: コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
