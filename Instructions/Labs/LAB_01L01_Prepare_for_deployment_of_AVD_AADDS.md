---
lab:
    title: 'ラボ: Azure Virtual Desktop (Azure AD DS) のデプロイの準備'
    module: 'モジュール 1: AVD アーキテクチャを計画する'
---

# ラボ - Azure Virtual Desktop (Azure AD DS) のデプロイの準備
# 受講生用ラボ マニュアル

## ラボの依存関係

- Azure サブスクリプション
- Azure サブスクリプションに関連付けられた Azure AD テナント内で グローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD、あるいは Azure サブスクリプション内 で所有者または共同作成はのロールを持つ Microsoft アカウントまたは Azure AD

## 推定所要時間

150 分

>**注**: Azure AD DS のプロビジョニングには約 90 分の待機時間がかかります。

## ラボ シナリオ

Azure Active Directory ドメイン サービス (Azure AD DS) 環境での Azure 仮想デスクトップのデプロイの準備をする必要があります

## 目標
  
このラボを完了すると、次のことができるようになります。

- Azure AD DS ドメインを実装する
- Azure AD DS ドメイン環境を構築する

## ラボ ファイル

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

## 説明

### 演習 0: vCPU クォータの数を増やす

この演習の主なタスクは次のとおりです。

1. 現在の vCPU 使用状況を識別する
1. vCPU クォータの増加を要求する

#### タスク 1: 現在の vCPU 使用状況を識別する

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して **Cloud Shell** ペインを開きます。
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
   
1. 前の手順で実行したコマンドの出力を確認し、ターゲット Azure リージョンの Azure VM の **Standard DSv3 Family** と **StandardBSFamily** の両方に少なくとも **20** 個の使用可能な vCPU があることを確認します。それがすでに当てはまる場合は、次の演習に直接進んでください。それ以外の場合は、この演習の次のタスクに進みます。 

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


### 演習 1: Active Directory Domain Services (AD DS) ドメインを実装する

この演習の主なタスクは次のとおりです。

1. Azure AD DS ドメインを管理するためのAzure AD ユーザー アカウントを作成および構成する
1. Azure portal を使用して Azure AD DS インスタンスをデプロイする
1. Azure AD DS デプロイのネットワークと ID 設定を構成する

#### タスク 1: Azure AD DS ドメインを管理するためのAzure AD ユーザー アカウントを作成および構成する

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者のロールと、Azure サブスクリプションに関連付けられた Azure AD テナントのグローバル管理者のロールを持つユーザー アカウントの資格情報を提供して、サインインします。
1. Azure portal を表示している Web ブラウザーで、**「概要」** ブレードに戻り、左側の垂直メニューバーの **「管理」** セクションで、**「プロパティ」** をクリックします。
1. Azure AD テナントの **「プロパティ」** ブレードで、ブレードの一番下で、**「セキュリティ詳細の管理」** リンクをクリックします。
1. **「セキュリティ既定値の有効化」** ブレードで、必要に応じて、**「いいえ」** を選択し、**「私の組織は条件付きアクセスを使用しています」** チェックボックスを選択して、**「保存」** を選択します。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して **「Cloud Shell」** ペインを開きます。
1. **Bash** や **PowerShell** のどちらかを選択するためのプロンプトが表示されたら、**PowerShell** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」 を選択します。 

1. Cloud Shell ペインから次のコマンドを実行して、Azure AD テナントににサインインします。

   ```powershell
   Connect-AzureAD
   ```

1. Cloud Shell ペインから、以下を実行して、Azure サブスクリプションに関連付けられている Azure AD テナントのプライマリ DNS ドメイン名を取得します。

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Cloud Shell ペインから、以下を実行して、昇格された特権を付与する Azure AD ユーザーを作成します (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**: 必ず使用したパスワードは覚えておいてください。このラボとその後のラボで必要になります。

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. Cloud Shell ペインから、次の操作を実行して、新しく作成された 最初の Azure AD ユーザーにグローバル管理者ロールを割り当てます。

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **注**: Azure AD PowerShell モジュールは、グローバル管理者ロールを会社の管理者と呼びます。

1. Cloud Shell ウィンドウから、次の操作を実行して、新しく作成された Azure AD ユーザーのユーザー プリンシパル名を識別します。

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **注**: ユーザー プリンシパル名を記録します。これついては、演習の後半で取り上げます。 

1. 「Cloud Shell」 ペインを閉じます。
1. Azure portal 内で、**「サブスクリプション」** を検索して選択し、**「サブスクリプション」** ブレードから、このラボで使用している Azure サブスクリプションを選択します。 
1. Azure サブスクリプションのプロパティを表示するブレードで、「**アクセス制御 (IAM)**」 を選択し、「**+ 追加**」 を選択して、ドロップダウン リストで、「**ロールの割り当ての追加**」 を選択します。 
1. 「**ロール割り当ての追加**」 ブレードで、次のように設定してから、「**保存**」 を選択します。

   |設定|値|
   |---|---|
   |ロール|**所有者**|
   |アクセスの割り当て|**ユーザー、グループ、またはサービス プリンシパル**|
   |選択|**aadadmin1**|

   > **注**: **aadadmin1** アカウントを使用して、Azure サブスクリプションと、ラボの後半で Windows 10 Azure VM に参加した Azure AD DS からの対応する Azure AD テナントを管理します。 


#### タスク 2: Azure portal を使用して Azure AD DS インスタンスをデプロイする

1. ラボ コンピューターの Azure portal で、**Azure AD ドメイン サービス**を検索して選択し、**「Azure AD ドメイン サービス」** ブレードから **「+ 追加」** を選択します。これにより、**「Azure AD ドメインサービスの作成」** ブレードが開きます。
1. **「Azure AD ドメイン サービスの作成」** ブレードの **「基本」** タブで、次の設定を指定し、**「次へ」** を選択します (他のユーザーは既存の値のままにします)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|新しいリソース グループの名前 **az140-11a-RG**|
   |DNS ドメイン名|**adatum.com**|
   |リージョン|AVD デプロイをホストするリージョンの名前|
   |SKU|**Standard**|
   |フォレストの種類|**User**|

   > **注**: これは技術的には必須ではありませんが、一般に、既存の Azure またはオンプレミスの DNS 名前空間とは異なる Azure AD DS ドメイン名を割り当てる必要があります。

1. **「Azure AD ドメイン サービスの作成」** ブレードの **「ネットワーク」** タブで、**「仮想ネットワーク」** ドロップダウン リストの横にある **「新規作成」** を選択します。
1. 「**仮想ネットワークの作成**」ブレードで、次のように指定してから、「**OK**」を選択します。

   |設定|値|
   |---|---|
   |名前|**az140-aadds-vnet11a**|
   |アドレス範囲|**10.10.0.0/16**|
   |サブネット名|**aadds-Subnet**|
   |サブネット名|**10.10.0.0/24**|

1. **「仮想ネットワークの作成」** ブレードの **「ネットワーク」** タブに戻り、**「次へ」** を選択します (他のユーザーは既存の値のままにします)。
1. **「Azure AD ドメイン サービスの作成」** ブレードの **「管理」** タブで、既定の設定を受け入れて **「次へ」** を選択します。
1. **「Azure AD ドメイン サービスの作成」** ブレードの **「同期」** タブで、**「すべて」** が選択されていることを確認してから、**「次へ」** を選択します。
1. **「Azure AD ドメイン サービスの作成」** ブレードの **「セキュリティ設定」** タブで、既定の設定を受け入れて **「次へ」** を選択します。
1. **「Azure AD ドメイン サービスの作成」** ブレードの **「Review + create」** タブで、**「作成」** を選択します。 
1. Azure AD DS ドメインの作成後に変更できない設定に関する通知を確認し、**「OK」** を選択します。

   >**注**: Azure AD DS ドメインのプロビジョニング後に変更できない設定には、DNS 名、Azure サブスクリプション、リソース グループ、ドメイン コントローラーをホストする仮想ネットワークとサブネット、およびフォレストの種類が含まれます。

   > **注**: 次の演習を進める前に、デプロイが完了するのを待ちます。90 分間程度かかる場合があります。 

#### タスク 3: Azure AD DS デプロイのネットワークと ID 設定を構成する

1. ラボ コンピューターの Azure portal で、**Azure AD ドメイン サービス**を検索して選択し、**「Azure AD ドメイン サービス」** ブレードから **adatum.com** エントリを選択して、新しくプロビジョニングされたAzure AD DS インスタンスに移動します。 
1. Azure AD DS インスタンスの **adatum.com** ブレードで、**管理するドメインの構成の問題が検出された**ことを示す警告をクリックします。 
1. **adatum.com** で **| 「構成診断 (プレビュー)」** ブレードで、**「実行」** をクリックします。
1. **「検証」** セクションで、**「DNS レコード」** ペインを展開し、**「修正」** をクリックします。
1. **「DNS レコード」** ブレードで、もう一度 **「修正」** をクリックします。
1. Azure AD DS インスタンスの **adatum.com** ブレードに戻り、**「必要な構成手順」** セクションで、Azure AD DS のパスワードハッシュ同期に関する情報を確認します。 

   > **注**: Azure AD DS ドメイン コンピューターとそのリソースにアクセスできる必要がある既存のクラウドのみのユーザーは、パスワードを変更するか、パスワードをリセットする必要があります。これは、このラボの前半で作成した **aadadmin1** アカウントに適用されます。

1. ラボ コンピューターの Azure portal で、**Cloud Shell** ペインで **PowerShell** セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、Azure AD **aadadmin1** ユーザー アカウントの objectID 属性を識別します。

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、前の手順で特定した objectId である **aadadmin1** ユーザー アカウントのパスワードをリセットします (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**: 必ず使用したパスワードは覚えておいてください。このラボとその後のラボで必要になります。

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **注**: 実際のシナリオでは、通常、**-ForceChangePasswordNextLogin**の値を $true に設定します。この場合、ラボの手順を簡素化するために **$false** を選択しました。 

1. 前述の 2 つの手順を繰り返して、**wvdaadmin1** ユーザー アカウントのパスワードをリセットします。


### 演習 2: Azure AD DS ドメイン環境を構築する
  
この演習の主なタスクは次のとおりです。

1. Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする
1. Azure Bastion をデプロイする
1. Azure AD DS ドメインの既定の構成を確認する
1. Azure AD DS と同期する AD DS ユーザーとグループを作成する

#### タスク 1: Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする

1. ラボ コンピューターから、Azure portal で、Cloud Shell ペインの PowerShell セッションから、以下を実行して、前のタスクで作成した **az140-aadds-vnet11a** という名前の仮想ネットワークに **cl-Subnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Azure portal の Cloud Shell ペインのツールバーで、「**ファイルのアップロード/ダウンロード**」 アイコンを選択し、ドロップダウン メニューで 「**アップロード**」 を選択し、ファイル **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** および **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** を Cloud Shell ホーム ディレクトリにアップロードします。
1. Cloud Shell ペインの PowerShell セッションから、次のコマンドを実行して、Azure Virtual Desktop クライアントとして機能する Windows 10 を実行している Azure VM を展開し、Azure AD DS ドメインに参加させます。

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11a.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11a.parameters.json
   ```

   > **注**: デプロイには約 10 分間かかります。次のタスクを進める前に、デプロイが完了するのを待ちます。 


#### タスク 2: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用して、以前のタスクでデプロイしたパブリック エンドポイントなしで、Azure VM に接続できます。一方、オペレーティング システム レベルの資格情報をターゲットとするブルート フォース攻撃に対して保護します。

> **注**: ブラウザーでポップアップ機能が有効になっていることを確認します。

1. Azure portal を表示しているブラウザー ウィンドウで、別のタブを開き、ブラウザー タブで、Azure portal に移動します。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して「**Cloud Shell**」ペインを開きます。
1. Cloud Shell ペインの PowerShell セッションで、以下を実行して、**AzureBastionSubnet** という名前のサブネットをこの演習の前半で作成した **az140-adds-vnet11** という名前の仮想ネットワークに追加します。

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.10.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cloud Shell ペインを閉じます。
1. Azure portal で、「**Bastions**」を選択し、**Bastions** ブレードで、「**+ 作成**」を選択します。
1. **Bastion の作成**ブレードの**基本** タブで、次の設定を指定して、「**確認および作成**」を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|**az140-11a-RG**|
   |名前|**az140-11a-bastion**|
   |リージョン|この演習の前のタスクでリソースにデプロイした Azure リージョンと同じです|
   |階層|**Basic**|
   |仮想ネットワーク|**az140-aadds-vnet11a**|
   |サブネット|**AzureBastionSubnet (10.10.254.0/24)**|
   |パブリック IP アドレス|**新規作成**|
   |Public IP name|**az140-aadds-vnet11a-ip**|

1. **Bastion の作成**ブレードの**確認および作成**タブで、「**作成**」を選択します。

   > **注**: この演習の次のタスクに進む前に、デプロイが完了するのを待ちます。デプロイには約 5 分間かかります。


#### タスク 3: Azure AD DS ドメインの既定の構成を確認する

> **注**: 新しく AzureAD DS に参加したコンピューターにサインインする前に、サインインする予定のユーザーアカウントを **AAD DC 管理者** Azure AD グループに追加する必要があります。この Azure AD グループは、Azure AD DS インスタンスをプロビジョニングした Azure サブスクリプションに関連付けられた Azure AD テナントに自動的に作成されます。

> **注**: Azure AD DS インスタンスをプロビジョニングするときに、このグループに既存の Azure AD ユーザー アカウントを追加するオプションがあります。

1. ラボ コンピューターの Azure portalの Cloud Shell ペインで、次のコマンドを実行して、**aadadmin1** Azure AD ユーザー アカウントを **AAD DC 管理者** Azure AD グループに追加します。

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. 「Cloud Shell」 ペインを閉じます。
1. ラボ コンピューターの Azure portal で、**仮想マシン**を検索して選択し、**「仮想マシン」** ブレードから **az140-cl-vm11a** エントリを選択します。これにより、**az140-cl-vm11a** ブレードが開きます。
1. **az140-cl-vm11a** ブレードで、「**接続**」を選択し、ドロップダウン メニューで、「**Bastion**」を選択し、「**az140-cl-vm11a \| 接続**」ブレードの **Bastion** タブで、「**Bastion の使用**」を選択します。
1. 指示されたら **aadadmin1** ユーザーとして、このラボで先ほど識別したプリンシパル名と、ラボで先ほど作成した際にこのユーザー アカウントで設定したパスワードを使用してサインインします。
1. リモート デスクトップから **az140-cl-vm11a** Azure VMで、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE** スクリプト」 ペインで、次を実行して Active Directory と DNS 関連のリモート サーバー管理ツールをインストールします。

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **注**: インストールが完了するのを待ってから、次の手順に進みます。これにはおよそ 2 分かかる場合があります。

1. リモート デスクトップ内の **az140-cl-vm11a** Azure VMで、**「スタート」** メニューの **「Windows 管理ツール」** フォルダーに移動して展開し、ツールのリストから **「Active Directory ユーザーとコンピューター」** を起動します。 
1. **Active Directory ユーザーとコンピューター** コンソールで、**AADDC コンピューター**と **AADDC ユーザー**の組織単位を含むデフォルトの階層を確認します。前者には **az140-cl-vm11a** コンピューター アカウントが含まれ、後者には Azure ADDS インスタンスの展開をホストする Azure サブスクリプションに関連付けられた Azure AD テナントから同期されたユーザー アカウントが含まれることに注意してください。**AADDC ユーザー**組織単位には、同じ Azure AD テナントから同期された **AADDC 管理者**グループとそのグループ メンバーシップも含まれます。このメンバーシップは、Azure AD DS ドメイン内で直接変更することはできませんが、代わりに、Azure AD DS テナント内で管理する必要があります。変更はすべて、Azure AD DS ドメインでホストされているグループのレプリカと自動的に同期されます。 

   > **注**: 現在、グループには **aadadmin1** ユーザー アカウントのみが含まれています。

1. **Active Directory ユーザーとコンピューター** コンソールの **AADDC ユーザー** OU で、**aadadmin1** ユーザー アカウントを選択し、その **「プロパティ」** ダイアログ ボックスを表示して、**「アカウント」** タブに切り替えます。ユーザー プリンシパル名のサフィックスは、プライマリ Azure AD DNS ドメイン名と一致し、変更できないことに注意してください。 
1. **Active Directory ユーザーとコンピューター** コンソールで、**ドメイン コントローラー**組織単位の内容を確認し、ランダムに生成された名前を持つ 2 つのドメイン コントローラーのコンピューター アカウントが含まれていることに注意してください。 

#### タスク 4: Azure AD DS と同期する AD DS ユーザーとグループを作成する

1. リモート デスクトップ内で **az140-cl-vm11a** Azure VM に移動し、Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動します。このラボで先ほど設定したパスワードを使用して **aadadmin1** ユーザー アカウントのユーザー プリンシパル名を指定してサインインします。
1. Azure portal で **Cloud Shell** を開きます。
1. **Bash** や **PowerShell** のどちらかを選択するプロンプトが表示されたら、**PowerShell** を選択します。 

   >**注**: **aadadmin1** ユーザー アカウントを使用して **Cloud Shell** を起動するのはこれが初めてなので、Cloud Shell のホームディレクトリを構成する必要があります。**「ストレージがマウントされていません」** というメッセージが表示されたら、このラボで使用しているサブスクリプションを選択し、**「ストレージの作成」** を選択します。 

1. Cloud Shell ペインの PowerShell セッションから、次のコマンドを実行してサインインし、Azure AD テナントに認証します。

   ```powershell
   Connect-AzureAD
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、Azure サブスクリプションに関連付けられている Azure AD テナントのプライマリ DNS ドメイン名を取得します。

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、今後のラボで使用する Azure AD ユーザー アカウントを作成します (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**: 必ず使用したパスワードは覚えておいてください。このラボとその後のラボで必要になります。

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   $aadUserNamePrefix = 'aaduser'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AzureADUser -AccountEnabled $true -DisplayName "$aadUserNamePrefix$counter" -PasswordProfile $passwordProfile -MailNickName "$aadUserNamePrefix$counter" -UserPrincipalName "$aadUserNamePrefix$counter@$aadDomainName"
   } 
   ```

1. Cloud Shell ペインの PowerShell セッションから、次のコマンドを実行して **「az140-wvd-aadmins」** という名前の Azure AD グループを作成し、それに **aadadmin1** および **wvdaadmin1** ユーザー アカウントを追加します。

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. Cloud Shell ペインから、前の手順を繰り返して、今後のラボで使用するユーザー用の Azure AD グループを作成し、以前に作成した Azure AD ユーザー アカウントを追加します。

   ```powershell
   $az140wvdausers = New-AzureADGroup -Description 'az140-wvd-ausers' -DisplayName 'az140-wvd-ausers' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-ausers'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId

   $az140wvdaremoteapp = New-AzureADGroup -Description "az140-wvd-aremote-app" -DisplayName "az140-wvd-aremote-app" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-aremote-app"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId

   $az140wvdapooled = New-AzureADGroup -Description "az140-wvd-apooled" -DisplayName "az140-wvd-apooled" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apooled"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId

   $az140wvdapersonal = New-AzureADGroup -Description "az140-wvd-apersonal" -DisplayName "az140-wvd-apersonal" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apersonal"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   ```

1. Cloud Shell ペインを閉じます。
1. リモート デスクトップから **az140-cl-vm11a** Azure VM まで、Azure portal を表示している Microsoft Edge ウィンドウで、左側の垂直メニュー バーにある Azure AD テナント ブレードで **Azure Active Directory** ブレードを検索して選択します。**「管理」** セクションで **「ユーザー」** を選択し、**「ユーザー \| すべてのユーザー」** ブレードで、新しいユーザー アカウントが作成されていることを確認します
1. Azure AD テナント ブレードに戻り、左側の垂直メニュー バーの **「管理」** セクションで **「グループ」** を選択し、**「グループ \| すべてのグループ」** ブレードで、新しいグループ アカウントが作成されたことを確認します。
1. リモート デスクトップ内で **az140-cl-vm11a** Azure VM に移動し、**Activ eDirectory ユーザーとコンピューター** コンソールに切り替えます。**Active Directory ユーザーとコンピューター** コンソールで、**AADDC ユーザー** OU に移動し、同じユーザー アカウントとグループ アカウントが含まれていることを確認します。

   >**注**: コンソールのビューを更新する必要がある場合があります。
