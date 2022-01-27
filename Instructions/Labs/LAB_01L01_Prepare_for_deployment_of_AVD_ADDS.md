---
lab:
    title: 'ラボ: Azure Virtual Desktop (AD DS) のデプロイの準備'
    module: 'モジュール 1: AVD アーキテクチャを計画する'
---

# ラボ - Azure Virtual Desktop (AD DS) のデプロイの準備
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプション内で所有者または共同作成者のロールを持つ Microsoft アカウントまたは Azure AD アカウント、この Azure サブスクリプションに関連付けられた Azure AD テナント内でグローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD アカウント。

## 推定所要時間

60 分

>**注**: Azure AD DS のプロビジョニングには約 90 分の待機時間がかかります。

## ラボ シナリオ

Active Directory ドメイン サービス (AD DS) 環境でのデプロイの準備をする必要があります

## 目標
  
このラボを完了すると、次のことができるようになります。

- Azure VM を使用して Active Directory Domain Services (AD DS) シングルドメイン フォレストをデプロイする
- AD DS フォレストを Azure Active Directory (Azure AD) テナントに統合する

## ラボ ファイル

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

## 説明

### 演習 0: vCPU クォータの数を増やす

この演習の主なタスクは次のとおりです。

1. 現在の vCPU 使用状況を識別する
1. vCPU クォータの増加を要求する

#### タスク 1: 現在の vCPU 使用状況を識別する

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して 「**Cloud Shell**」 ペインを開きます。
1. **Bash** や **PowerShell** のどちらかを選択するためのプロンプトが表示されたら、**PowerShell** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」 を選択します。 

1. **Microsoft.Compute** リソース プロバイダーが登録されていない場合は、Azure portal の **Cloud Shell** の PowerShell で、次のコマンドを実行して登録します。

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. Azure portal の **Cloud Shell** の PowerShell で、次のコマンドを実行して、**Microsoft.Compute** リソース プロバイダーの登録の状態を確認します。

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**注**: 「状態」 が 「**登録済み**」 と表示されていることを確認します。そうでない場合は、数分待ってから、この手順を繰り返します。

1. Azure portal の **Cloud Shell** の PowerShell セッションで、次のコマンドを実行して、vCPU の現在の使用状況と、**StandardDSv3Family** および **StandardBSFamily** Azure VMの対応する制限を特定します (`<Azure_region>` プレースホルダーを Azure の名前に置き換えます。たとえば、`eastus`)。

   ```powershell
   $location = '<Azure_region>'
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardBSFamily'}
   ```

   > **注**: Azure リージョンの名前を特定するには、**Cloud Shell** の PowerShell プロンプトで `(Get-AzLocation).Location` を実行します。
   
1. 前の手順で実行したコマンドの出力を確認し、ターゲット Azure リージョンの Azure VM の **Standard DSv3 Family** と **StandardBSFamily** の両方に少なくとも **40** 個の使用可能な vCPU があることを確認します。それがすでに当てはまる場合は、次の演習に直接進んでください。それ以外の場合は、この演習の次のタスクに進みます。 

#### タスク 2: vCPU クォータの増加を要求する

1. Azure portal で、**「サブスクリプション」** を検索して選択し、**「サブスクリプション」** ブレードから、このラボで使用する予定の Azure サブスクリプションを表すエントリを選択します。
1. Azure portal のサブスクリプションブレードの左側の垂直メニューの **「設定」** セクションで、**「使用量 + クォータ」** を選択します。 
1. サブスクリプションの **「使用状況 + クォータ」** ブレードで、「**増量のリクエスト**」 を選択します。
1. **「新しいサポート要求」** ブレードの **「基本」** タブで、次のように指定し、**「次へ: ソリューション >」** を選択します。

   |設定|値|
   |---|---|
   |問題の種類|**サービスとサブスクリプションの制限 (クォータ)**|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |クォータの種類|**Compute-VM (cores-vCPU) サブスクリプションの制限の増加**|
   |サポート プラン|ターゲット サブスクリプションに関連付けられているサポート プランの名前|

1. **「新しいサポート要求」** ブレードの **「詳細」** タブで、**「詳細の提供」** リンクを選択します。
1. **「新しいサポート要求」** ブレードの **「クォータの詳細」** タブで、次のように指定し、**「保存して続行」** を選択します。

   |設定|値|
   |---|---|
   |デプロイ モデル|**リソース マネージャー**|
   |場所|このラボで使用する予定の Azure リージョンの名前|
   |種類|**Standard**|
   |Standard|**BS シリーズ**|
   |新しい vCPU 制限|新しい制限|
   |Standard|**DSv3 シリーズ**|
   |新しい vCPU 制限|新しい制限|

   >**注**: この場合、**BS シリーズ** Azure VM の使用は、ラボ環境の実行コストを最小限に抑えることを目的としています。これは、Azure Virtual Desktop シナリオでの **BS シリーズ** Azure VM の使用目的を表すことを意図したものではありません。

1. **「新しいサポート要求」** ブレードの **「詳細」** タブに戻り、次のように指定し、**「次へ: 確認および作成 >**」 を選択します。

   |設定|値|
   |---|---|
   |重要度|**C - 最小限の影響**|
   |ご希望の連絡方法|ご希望のオプションを選択し、連絡先の詳細を入力してください|
    
1. **「新しいサポート要求」** ブレードの **「確認および作成」** タブで、**「作成」** を選択します。

   > **注**: この範囲の vCPU 内のクォータ増加要求は、通常、数時間以内に完了します。ただし、待機することなく、このラボを完了できます。

### 演習 1: Active Directory Domain Services (AD DS) ドメインをデプロイする

この演習の主なタスクは次のとおりです。

1. Azure VM のデプロイの準備
1. Azure Resource Manager QuickStart テンプレートを使用して、AD DS ドメイン管理者を実行する Azure VM をデプロイする
1. Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする
1. Azure Bastion をデプロイする

#### タスク 1: Azure VM のデプロイの準備

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal を表示している Web ブラウザーで、「**概要**」ブレードに戻り、左側の垂直メニューバーの「**管理**」セクションで、「**プロパティ**」をクリックします。
1. Azure AD テナントの「**プロパティ**」ブレードで、ブレードの一番下で、「**セキュリティ詳細の管理**」リンクをクリックします。
1. 「**セキュリティ既定値の有効化**」ブレードで、必要に応じて、「**いいえ**」を選択し、「**私の組織は条件付きアクセスを使用しています**」チェックボックスを選択して、「**保存**」を選択します。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して **「Cloud Shell」** ペインを開きます。
1. **Bash** や **PowerShell** のどちらかを選択するためのプロンプトが表示されたら、**PowerShell** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」 を選択します。 


#### タスク 2: Azure Resource Manager QuickStart テンプレートを使用して、AD DS ドメイン管理者を実行する Azure VM をデプロイする

1. ラボ コンピューターの Azure portal を表示している Web ブラウザーの Cloud Shell ペインの PowerShell セッションで、以下を実行して、リソース グループを作成します (`<Azure_region>` プレースホルダーをたとえば、`eastus` などこのラボで使用する予定の Azure リージョンの名前に置き換えます)。

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Azure portal で、**Cloud Shell** ウィンドウを閉じます。
1. ラボ コンピューターの同じブラウザー ウィンドウで、別な Web ブラウザー ウィンドウを開き、[新しい Windows VM を作成し、新しい AD Forest、Domain、DC を作成する](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)という名前の QuickStart テンプレートのカスタマイズされたバージョンに移動します。 
1. 「**新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する**」 ページで、「**Azure にデプロイする**」 を選択します。これにより、ブラウザーが Azure portal の「**新しい AD フォレストで Azure VM を作成する**」ブレードに自動的にリダイレクトされます。
1. 「**新しい AD フォレストで Azure VM を作成する**」 ブレードで、「**パラメーターの編集**」 を選択します。
1. 「**パラメーターの編集**」 ブレードで、「**ロード ファイル**」 を選択し、「**開く**」 ダイアログ ボックスで、「**\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json**」 を選択し、「**開く**」 を選択してから、「**保存**」 を選択します。 
1. 「**新しい AD フォレストを使用して Azure VM を作成する**」ブレードで、次の設定を指定します (他のユーザーには既存の値を残します)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|**az140-11-RG**|
   |ドメイン名|**adatum.com**|

1. 「**新しい AD フォレストを使用して Azure VM を作成する**」 ブレードで、「**確認および作成**」 を選択し、「**作成**」 を選択します。

   > **注**: 次の演習を進める前に、デプロイが完了するのを待ちます。15 分間程度かかる場合があります。 

#### タスク 3: Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする

1. ラボ コンピューターの Azure portal を表示するWebブラウザーで、Cloud Shell ペインの PowerShell セッションを開き、以下を実行して、前のタスクで作成した **az140-adds-vnet11** という名前の仮想ネットワークに **cl-Subnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Azure portal の Cloud Shell ペインのツールバーで、「**ファイルのアップロード/ダウンロード**」 アイコンを選択し、ドロップダウン メニューで 「**アップロード**」 を選択し、ファイル **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** および **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json** を Cloud Shell ホーム ディレクトリにアップロードします。
1. Cloud Shell ペインの PowerShell セッションから、次のコマンドを実行して、クライアントとして機能する Windows 10 を実行している Azure VM を新しく作成されたサブネットにデプロイします。

   ```powershell
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11.parameters.json
   ```

   > **注**: デプロイが完了するのを待たずに、代わりに次のタスクに進みます。デプロイには約 10 分間かかります。

#### タスク 4: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用して、以前のタスクでデプロイしたパブリック エンドポイントなしで、Azure VM に接続できます。一方、オペレーティング システム レベルの資格情報をターゲットとするブルート フォース攻撃に対して保護します。

> **注**: ブラウザーでポップアップ機能が有効になっていることを確認します。

1. Azure portal を表示しているブラウザー ウィンドウで、別のタブを開き、ブラウザー タブで、Azure portal に移動します。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して「**Cloud Shell**」ペインを開きます。
1. Cloud Shell ペインの PowerShell セッションで、以下を実行して、**AzureBastionSubnet** という名前のサブネットをこの演習の前半で作成した **az140-adds-vnet11** という名前の仮想ネットワークに追加します。

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cloud Shell ペインを閉じます。
1. Azure portal で、**「Bastions」** を選択し、**Bastions** ブレードで、**「+ 作成」** を選択します。
1. **Bastion の作成**ブレードの**基本** タブで、次の設定を指定して、**「確認および作成」** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|**az140-11-RG**|
   |名前|**az140-11-bastion**|
   |リージョン|この演習の前のタスクでリソースにデプロイした Azure リージョンと同じです|
   |階層|**Basic**|
   |仮想ネットワーク|**az140-adds-vnet11**|
   |サブネット|**AzureBastionSubnet (10.0.254.0/24)**|
   |パブリック IP アドレス|**新規作成**|
   |Public IP name|**az140-adds-vnet11-ip**|

1. **Bastion の作成**ブレードの**確認および作成**タブで、**「作成」** を選択します。

   > **注**: 次の演習を進める前に、デプロイが完了するのを待ちます。デプロイには約 5 分間かかります。

### 演習 2: AD DS フォレストを Azure AD テナントに統合する
  
この演習の主なタスクは次のとおりです。

1. Azure AD と同期する AD DS ユーザーとグループを作成する
1. AD DS UPN サフィックスを構成する
1. Azure AD との同期を構成するために使用する Azure AD ユーザーを作成する
1. Azure AD Connect をインストールする
1. ハイブリッド Azure AD 参加を構成する

#### タスク 1: Azure AD と同期する AD DS ユーザーとグループを作成する

1. ラボ コンピューターの Azure portal を表示する Web ブラウザーで、**仮想マシン**を検索して選択し、**「仮想マシン」** ブレードから **az140-dc-vm11** を選択します。
1. **az140-dc-vm11** ブレードで、**「接続」** を選択し、ドロップダウン メニューで、**「Bastion」** を選択し、**「az140-dc-vm11 \| 接続」** ブレードの **Bastion** タブで、**「Bastion の使用」** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力して、**「接続」** を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**Student**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、管理者向け Internet Explorer のセキュリティ強化を無効にします。

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次を実行して、このラボで使用される Azure AD テナントへの同期のスコープに含まれるオブジェクトを含む AD DS 組織単位を作成します。

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、Windows 10 ドメインに参加しているクライアント コンピューターのコンピューター オブジェクトを含む AD DS 組織単位を作成します。

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、このラボで使用する Azure AD テナントに同期される AD DS ユーザー アカウントを作成します (`<password>` プレースホルダーはランダムで複雑なパスワードに置き換えます)。

   > **注**: 必ず使用したパスワードは覚えておいてください。このラボとその後のラボで必要になります。

   ```powershell
   $ouName = 'ToSync'
   $ouPath = "OU=$ouName,DC=adatum,DC=com"
   $adUserNamePrefix = 'aduser'
   $adUPNSuffix = 'adatum.com'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AdUser -Name $adUserNamePrefix$counter -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix$counter@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString <password> -AsPlainText -Force) -passThru
   } 

   $adUserNamePrefix = 'wvdadmin1'
   $adUPNSuffix = 'adatum.com'
   New-AdUser -Name $adUserNamePrefix -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString <password> -AsPlainText -Force) -passThru

   Get-ADGroup -Identity 'Domain Admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

   > **注**: このスクリプトは、**aduser1** - **aduser9** という名前の 9 つの非特権ユーザー アカウントと、**wvdadmin1** という名前の **ADATUM\\Domain Admins** グループのメンバーである 1 つの特権アカウントを作成します。

1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、このラボで使用する Azure AD テナントに同期される AD DS グループ オブジェクトを作成します。

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次の手順を実行して、前の手順で作成したグループにメンバーを追加します。

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### タスク 2: AD DS UPN サフィックスを構成する

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して PowerShellGet モジュールの最新バージョンをインストールします (確認を求められたら **「はい」** を選択します)。

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、最新バージョンの Az PowerShell モジュールをインストールします (確認を求められたら、**「すべてはい」** を選択します)。

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次を実行して Azure サブスクリプションにサインインします。

   ```powershell
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を入力します。
1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、Azure サブスクリプションに関連付けられている Azure AD テナントの Id プロパティを取得します。

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、最新バージョンの Azure ADPowerShell モジュールをインストールします。

   ```powershell
   Install-Module -Name AzureAD -Force
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、Azure AD テナントに対して認証します。

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. プロンプトが表示されたら、このタスクで前に使用したのと同じ資格情報でサインインします。 
1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次のコマンドを実行して、Azure サブスクリプションに関連付けられている Azure AD テナントのプライマリ DNS ドメイン名を取得します。

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次の手順を実行して、Azure サブスクリプションに関連付けられている Azure AD テナントのプライマリ DNS ドメイン名を、AD DS フォレストの UPN サフィックスのリストに追加します。

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、Azure サブスクリプションに関連付けられている Azure AD テナントのプライマリ DNS ドメイン名を、AD DS ドメイン内のすべてのユーザーの UPN サフィックスとして割り当てます。

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、以下を実行して、**adatum.com** UPN サフィックスを **Student** ドメイン ユーザーに割り当てます。

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### タスク 3: ディレクトリ同期の構成に使用される Azure AD ユーザーを作成する

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、新しい Azure AD ユーザーを作成します (`<password>` プレースホルダーはランダムで複雑なパスワードに置き換えます)。

   > **注**: 必ず使用したパスワードは覚えておいてください。このラボとその後のラボで必要になります。:

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインから、次の操作を実行して、新しく作成された Azure AD ユーザーにグローバル管理者ロールを割り当てます。 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **注**: Azure AD PowerShell モジュールは、グローバル管理者ロールを会社の管理者と呼びます。

1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインから、次の操作を実行して、新しく作成された Azure AD ユーザーのユーザー プリンシパル名を識別します。

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **注**: ユーザー プリンシパル名を記録します。これついては、演習の後半で取り上げます。 


#### タスク 4: Azure AD Connect をインストールする

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」スクリプト ペインから、以下を実行して、TLS 1.2 を有効にします。

   ```powershell
   New-Item 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   Write-Host 'TLS 1.2 has been enabled.'
   ```
   
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Internet Explorer を起動して、[Microsoft Edge for Business のダウンロード ページ](https://www.microsoft.com/ja-jp/edge/business/download)に移動します。
1. az140-dc-vm11 へのリモート デスクトップ セッション内で、Internet Explorer を起動し、[Microsoft Edge for Business のダウンロードページ](https://www.microsoft.com/ja-jp/edge/business/download)に移動します。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Microsoft Edge を使用し、[Azure portal](https://portal.azure.com) に移動します。プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. Azure portal で、Azure portal ページの上部にある **「リソース、サービス、およびドキュメントの検索」** テキスト ボックスを使用して、**Azure Active Directory** ブレードを検索して移動し、Azure AD テナント ブレードのハブ メニューの **「管理」** セクションで、**「Azure AD Connect」** を選択します。
1. **「Azure AD Connect」** ブレードで、**「Azure AD Connect をダウンロードする」** リンクを選択します。これにより、**Microsoft Azure Active Directory Connect** のダウンロード ページを表示する新しいブラウザー タブが自動的に開きます。
1. 「**Microsoft Azure Active Directory Connect** のダウンロード」 ページで、**「ダウンロード」** を選択します。
1. **AzureADConnect.msi** インストーラーを実行するか保存するかを確認するメッセージが表示されたら、**「実行」** を選択して **Microsoft Azure Active Directory Connect** ウィザードを開始します。
1. **「Microsoft Azure Active Directory Connect」** ウィザードの **「Azure AD Connect へようこそ」** ページで、チェックボックス **「ライセンス条項とプライバシーに関する通知に同意します」** を選択し、**「続行」** を選択します。
1. **「Microsoft Azure Active Directory Connect」** ウィザードの **「簡易設定」** のページで、**カスタマイズ** オプションを選択します。
1. **「必要なコンポーネントをインストールする」** ページで、オプションの構成オプションをすべて選択解除したままにして、**「インストール」** を選択します。
1. **「ユーザーのサインイン」** ページで、**「パスワード ハッシュの同期」** のみ有効にして、**「次へ」** を選択します。
1. **「Azure AD への接続」** ページで、前の演習で作成した **aadsyncuser** ユーザー アカウントの資格情報を使用して認証し、**「次へ」** を選択します。 

   > **注**: この演習の前半で記録した **aadsyncuser** アカウントの userPrincipalName 属性を指定し、このラボで先ほど設定したパスワードをそのパスワードとして指定します。

1. **「ディレクトリを接続する」** ページで、**adatum.com** フォレスト エントリの右側にある **「ディレクトリの追加」** ボタンを選択します。
1. **「AD フォレスト アカウント」** ウィンドウで、**新しい AD アカウントを作成する**オプションが選択されていることを確認し、次の資格情報を指定して、**「OK」** を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**ADATUM\Student**|
   |パスワード|**Pa55w.rd1234**|

1. **ディレクトリを接続する**ページに戻り、**adatum.com** エントリが構成済みディレクトリとして表示されていることを確認し、**「次へ」** を選択します
1. **「Azure AD サインイン情報の構成」** ページで、**UPN サフィックスが確認済みのドメイン名と一致しない場合、ユーザーはオンプレミスの資格情報で Azure AD にサインインできません**という警告に注意して、チェックボックス **「すべての UPN サフィックスを確認済みドメインに一致させずに続行」** を有効にし、**「次へ」** を選択します。

   > **注**: Azure AD テナントには、**adatum.com** AD DS の UPN サフィックスの 1 つと一致する検証済みのカスタム DNS ドメインがないため、こは予想されます。

1. **「ドメインと OU フィルタリング」** ページで、オプション **「選択したドメインと OU を同期する」** を選択し、すべてのチェックボックスをクリアにし、**ToSync** OU の横にあるチェックボックスのみを選択して、**「次へ」** を選択します。
1. **「ユーザーを一意に識別する」** ページで、既定値の設定を承諾し、**「次へ」** を選択します。
1. **ユーザーとデバイスをフィルタリングする**ページで、既定値の設定を承諾し、**「次へ」** を選択します。
1. **「オプション機能」** ページで、既定の設定値を承諾してから、**「次へ」** を選択します。
1. **「構成する準備ができました」** ページで、**「構成が完了したら同期プロセスを開始します」** チェックボックスが選択されていることを確認し、**「インストール」** を選択します。

   > **注**: インストールにはおよそ 2 分かかります。

1. 「**構成が完了しました**」 ページの情報を確認し、「**終了**」 を選択し、**Microsoft Azure Active Directory Connect** ウィンドウを閉じます。
1. リモート デスクトップ セッション内から **az140-vm11** へ、Azure portal を表示している Microsoft Edge ウィンドウで、Adatum Lab Azure AD テナントの **「ユーザー - すべてのユーザー」** ブレードへ移動します。
1. 「**ユーザー \| すべてのユーザー**」 ブレードでは、ユーザー オブジェクトのリストには、このラボで以前に作成した AD DS ユーザー アカウントのリストが含まれており、**「ディレクトリの同期」** 列に **「はい」** エントリが表示されていることに注意してください。

   > **注**: AD DS ユーザー アカウントが表示されるまで、数分待ってからブラウザー ページを更新する必要がある場合があります。
