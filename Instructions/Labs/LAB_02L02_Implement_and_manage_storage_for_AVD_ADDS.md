---
lab:
    title: 'ラボ: AVD (AD DS) 用のストレージを実装および管理する'
    module: 'モジュール 2: AVD インフラストラクチャを実装する'
---

# ラボ - AVD (AD DS) 用のストレージを実装および管理する
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプション内で所有者または共同作成者のロールを持つ Microsoft アカウントまたは Azure AD アカウント、この Azure サブスクリプションに関連付けられた Azure AD テナント内でグローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD アカウント。
- 実施するラボ - **Azure Virtual Desktop (AD DS) のデプロイを準備する**

## 推定所要時間

30 分

## ラボ シナリオ

Azure Active Directory ドメイン サービス (Azure AD DS) 環境で Azure Virtual Desktop をデプロイするためのストレージを実装および管理する必要があります。

## 目標
  
このラボを終了すると、下記ができるようになります。

- Azure Virtual Desktop 用プロファイル コンテナーを格納するために Azure Files を構成する

## ラボ ファイル

- なし

## 手順

### 演習 1: Azure Virtual Desktop 用プロファイル コンテナーを格納するために Azure Files を構成する

この演習の主なタスクは次のとおりです:

1. Azure Storage アカウントを作成する
1. Azure Files 共有を作成する
1. Azure Storage アカウントに対する AD DS 認証を有効にする 
1. Azure Files RBAC ベースのアクセス許可を構成する
1. Azure Files ファイル システムののアクセス許可を構成する

#### タスク 1: Azure Storage アカウントを作成する

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portalで、「**仮想マシン**」を検索して選択し、**「Virtual Machines」** ブレードで、**az140-vm11** を選択します。
1. 「**az140-dc-vm11**」ブレードで、「**接続**」を選択し、ドロップダウン メニューで、「**Bastion**」を選択し、「**az140-dc-vm11 \| 接続**」ブレードの「**Bastion**」タブで、「**Bastion の使用**」を選択します。
1. プロンプトが表示されたら、次の資格情報を入力して、「**接続**」を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**Student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Microsoft Edge を起動して、[Azure portal](https://portal.azure.com) に移動します。プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示する Microsoft Edge で「**ストレージ アカウント**」を検索して選択し、**「ストレージ アカウント」** ブレードで **「+ 追加」** を選択します。
1. 「**ストレージ アカウントの作成**」ブレードの「**基本**」タブで、次のように設定を行います (他の設定は既定値のままにします)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|新しいリソース グループの名前 **az140-22-RG**|
   |ストレージ アカウント名|小文字と数字で構成され、文字で始まる 3 〜 15 の長さのグローバルに一意の名前|
   |Region|Azure Virtual Desktop ラボ環境をホストする Azure リージョンの名前|
   |パフォーマンス|**Standard**|
   |冗長性|**geo 冗長ストレージ (GRS)**|
   |リージョンが利用できない場合に、データへの読み取りアクセスを利用可能にする|有効|

   >**注**: ストレージアカウント名の長さが 15 文字を超えないようにしてください。この名前は、ストレージ アカウントを含む Azure サブスクリプションに関連付けられた Azure AD テナントと統合された Active Directory ドメイン サービス (AD DS) ドメインにコンピューター アカウントを作成するために使用されます。これにより、このストレージ アカウントでホストされているファイル共有にアクセスするときに、AD DS ベースの認証が可能になります。

1. 「**ストレージ アカウントの作成**」 ブレードの 「**基本**」 タブで、「**Review + Create**」 を選択し、検証プロセスが完了するのを待ってから、「**作成**」 を選択します。

   >**注**: ストレージ アカウントが作成されるのを待ちます。これにはおよそ 2 分かかる場合があります。

#### タスク 2: Azure Files 共有を作成する

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、**「ストレージ アカウント」** ブレードに戻り、新しく作成されたストレージ アカウントを表すエントリを選択します。
1. ストレージ アカウント ブレードの 「**データ ストレージ**」 セクションで、「**ファイル共有**」 を選択し、「**+ ファイル共有**」 を選択します。
1. 「**新しいファイル共有**」 で、次の設定を指定し、「**作成**」 を選択します (他の設定はデフォルト値のままにします)。

   |設定|値|
   |---|---|
   |名前|**az140-22-profiles**|
   |階層|**トランザクション最適化**|

#### タスク 3: Azure Storage アカウントに対する AD DS 認証を有効にする 

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Microsoft Edge ウィンドウで別のタブを開き、[Azure Files サンプルの GitHub リポジトリ](https://github.com/Azure-Samples/azure-files-samples/releases)に移動し、圧縮された **AzFilesHybrid.zip** PowerShell モジュールの最新バージョンをダウンロードし、そのコンテンツを **C:\\Allfiles\\Labs\\02** フォルダーに抽出します (必要に応じてフォルダーを作成します)。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、値が **3** の **Zone.Identifier** 代替データストリームを削除します。これは、インターネットからダウンロードされたことを示します。

   ```powershell
   Get-ChildItem -Path C:\Allfiles\Labs\02 -File -Recurse | Unblock-File
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次を実行して Azure サブスクリプションにサインインします。

   ```powershell
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、後続のスクリプトを実行するために必要な変数を設定します。

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、このタスクの前半で作成した Azure Storage アカウントを表し、AD DS 認証の実装に使用されるAD DS コンピューター オブジェクトを作成します。

   ```powershell
   Set-Location -Path 'C:\Allfiles\Labs\02'
   .\CopyToPSPath.ps1 
   Import-Module -Name AzFilesHybrid
   Join-AzStorageAccountForAuth `
      -ResourceGroupName $ResourceGroupName `
      -StorageAccountName $StorageAccountName `
      -DomainAccountType 'ComputerAccount' `
      -OrganizationalUnitDistinguishedName 'OU=WVDInfra,DC=adatum,DC=com'
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、Azure Storage アカウントで AD DS 認証が有効になっていることを確認します。

   ```powershell
   $storageaccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
   $storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
   $storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions
   ```

1. コマンド `$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties` の出力が `AD` を返すことを確認します。これは、ストレージ アカウントのディレクト リサービスを表し、`$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions` コマンドの出力を表します。また、ディレクトリ ドメイン情報を表す、次の形式に似ています (`DomainGuid`、`DomainSid`、および `AzureStorageSid` の値は異なります)。

   ```
   DomainName        : adatum.com
   NetBiosDomainName : adatum.com
   ForestName        : adatum.com
   DomainGuid        : 47c93969-9b12-4e01-ab81-1508cae3ddc8
   DomainSid         : S-1-5-21-1102940778-2483248400-1820931179
   AzureStorageSid   : S-1-5-21-1102940778-2483248400-1820931179-2109
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示する Microsoft Edge ウィンドウに切り替え、ストレージ アカウントのプロパティを表示するブレードで、「**ファイル共有**」を選択し、**Active Directory**　設定が**構成**されていることを確認します。

   >**注**: 変更を Azure portal に反映させるには、ブラウザー ページを更新する必要がある場合があります。

#### タスク 4: Azure Files RBAC ベースのアクセス許可を構成する

1. **az140-dc-vm11** へのリモート デスクトップ セッション内、Azure portal を表示する Microsoft Edge ウィンドウ、この演習の前半で作成したストレージ アカウントのプロパティを表示するブレード、左側の垂直メニュー、**「データ ストレージ」** セクションで、**「ファイル共有」** を選択します。
1. **「ファイル共有」** ブレードの共有リストで、**az140-22-profiles** エントリを選択します。
1. **「az140-22-profiles」** ブレードの左側の垂直メニューで、**「アクセス制御 (IAM)」** を選択します。
1. ストレージ アカウントの **「アクセス制御 (IAM)**」 ブレードで、「**+ 追加**」 を選択し、ドロップダウン メニューで 「**ロール割り当ての追加**」 を選択します。 
1. 「**ロール割り当ての追加**」 ブレードで、次のように設定してから、「**保存**」 を選択します。

   |設定|値|
   |---|---|
   |Role|**記憶域ファイル データの SMB 共有の共同作成者**|
   |アクセスの割り当て|**ユーザー、グループ、またはサービス プリンシパル**|
   |Select|**az140-wvd-users**|

1. ストレージ アカウントの **「アクセス制御 (IAM)**」 ブレードで、「**+ 追加**」 を選択し、ドロップダウン メニューで 「**ロール割り当ての追加**」 を選択します。 
1. 「**ロール割り当ての追加**」 ブレードで、次のように設定してから、「**保存**」 を選択します。

   |設定|値|
   |---|---|
   |Role|**記憶域ファイル データの SMB 共有の管理者特権共同作成者**|
   |アクセスの割り当て|**ユーザー、グループ、またはサービス プリンシパル**|
   |Select|**az140-wvd-admins**|

#### タスク 5: Azure Files ファイル システムののアクセス許可を構成する

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 ウィンドウに切り替え、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、この演習の前半で作成したストレージ アカウントの名前とキーを参照する変数を作成します。

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0]
   $storageAccountName = $storageAccount.StorageAccountName
   $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
   ```

1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、この演習の前半で作成したファイル共有へのドライブ マッピングを作成します。

   ```powershell
   $fileShareName = 'az140-22-profiles'
   net use Z: "\\$storageAccountName.file.core.windows.net\$fileShareName" /u:AZURE\$storageAccountName $storageAccountKey
   ```

1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次を実行して、現在のファイル システムのアクセス許可を表示します。

   ```powershell
   icacls Z:
   ```

   >**注**: 既定では、**NT Authority\\Authenticated Users** と **BUILTIN\\Users**の両方に、ユーザーが他のユーザーのプロファイル コンテナーを読み取ることを許可する権限があります。それらを削除し、代わりに最低限必要な権限を追加します。

1. 「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、最小特権の原則に準拠するようにファイル システムのアクセス許可を調整します。

   ```powershell
   $permissions = 'ADATUM\az140-wvd-admins'+':(F)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'ADATUM\az140-wvd-users'+':(M)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'Creator Owner'+':(OI)(CI)(IO)(M)'
   cmd /c icacls Z: /grant $permissions
   icacls Z: /remove 'Authenticated Users'
   icacls Z: /remove 'Builtin\Users'
   ```

   >**注**: または、ファイル エクスプローラーを使用してアクセス許可を設定することもできます。
