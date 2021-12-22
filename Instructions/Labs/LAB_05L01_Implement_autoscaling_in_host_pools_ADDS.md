---
lab:
    title: 'ラボ: ホスト プールに自動スケールを実装する (AD DS)'
    module: 'モジュール: WVD インフラストラクチャを監視および保守する'
---

# ラボ - ホスト プールに自動スケールを実装する (AD DS)
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプション内で所有者または共同作成者のロールを持つ Microsoft アカウントまたは Azure AD アカウント、この Azure サブスクリプションに関連付けられた Azure AD テナント内でグローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD アカウント。
- 実施するラボ - **Azure Virtual Desktop (AD DS) のデプロイを準備する**
- 実施するラボ - **Azure portal (AD DS) を使用してホスト プールとセッション ホストをデプロイする**

## 推定所要時間

60 分

## ラボ シナリオ

Active Directory ドメイン サービス (AD DS) 環境で Azure Virtual Desktop セッション ホストの自動スケーリングを構成する必要があります。

## 目標
  
このラボを終了すると、下記ができるようになります。

- Azure Virtual Desktop セッションホストの自動スケールを構成する
- Azure Virtual Desktop セッションホストの自動スケールを確認する

## ラボ ファイル

- なし

## 手順

### 演習 1: Azure Virtual Desktop セッションホストの自動スケールを構成する

この演習の主なタスクは次のとおりです:

1. Azure Virtual Desktop セッション ホストの自動スケールを準備する
1. Azure Automation アカウントを作成および構成する
1. Azure Logic App を作成する

#### タスク 1: Azure Virtual Desktop セッション ホストの自動スケールを準備する

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. ラボ コンピューターにの Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで使用する Azure VM をホストする Azure Virtual Desktop セッションを開始します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**注**: コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に開始されるまでに数分かかります。 

#### タスク 2: Azure Automation アカウントを作成および構成する

1. ラボ コンピューターから、Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portalで、「**仮想マシン**」を検索して選択し、**「Virtual Machines」** ブレードで、**az140-vm11** を選択します。
1. **az140-dc-vm11** ブレードで、「**接続**」を選択し、ドロップダウン メニューで、「**Bastion**」を選択し、「**az140-dc-vm11 \| 接続**」ブレードの Bastion タブで、「**Bastion の使用**」を選択します。
1. プロンプトが表示されたら、次の資格情報を入力して、「**接続**」を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**受講生**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. 「**管理者: Windows PowerShell ISE**」 コンソールで、次を実行して Azure サブスクリプションにサインインします。

   ```powershell
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、自動スケール リューションの一部である Azure Automation アカウントの作成に使用する PowerShell スクリプトをダウンロードします。

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   New-Item -ItemType Directory -Path $labFilesfolder -Force
   Set-Location -Path $labFilesfolder
   $uri = 'https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzAutoAccount.ps1'
   Invoke-WebRequest -Uri $Uri -OutFile '.\CreateOrUpdateAzAutoAccount.ps1'
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、スクリプト パラメーターに割り当てる変数の値を設定します。

   ```powershell
   $aadTenantId = (Get-AzContext).Tenant.Id
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   $suffix = Get-Random
   $automationAccountName = "az140-automation-51$suffix"
   $workspaceName = "az140-workspace-51$suffix"
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、このラボで使用するリソース グループを作成します。

   ```powershell
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、このラボで使用する Azure Log Analytics ワークスペースを作成します。

   ```powershell
   New-AzOperationalInsightsWorkspace -Location $location -Name $workspaceName -ResourceGroupName $resourceGroupName
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 で、**C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzAutoAccount.ps1** スクリプトを開き、**82** 行目と **86** 行目の間のコードを次のように複数行のコメントで囲みます。

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
	throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで新しいタブを開き、次のスクリプトを貼り付けて実行し、自動スケール ソリューションの一部である Azure Automation アカウントを作成します。

   ```powershell
   $Params = @{
     "AADTenantId" = $aadTenantId
     "SubscriptionId" = $subscriptionId 
     "UseARMAPI" = $true
     "ResourceGroupName" = $resourceGroupName
     "AutomationAccountName" = $automationAccountName
     "Location" = $location
     "WorkspaceName" = $workspaceName
   }

   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   .\CreateOrUpdateAzAutoAccount.ps1 @Params
   ```

   >**注**: スクリプトが完了するのを待ちます。これにはおよそ 10 分かかる場合があります。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、スクリプトの出力を確認します。 

   >**注**: 出力には、Webhook URI、Log Analytics ワークスペース ID、および自動スケール ソリューションの一部である Azure Logic App をプロビジョニングするときに提供する必要のある対応する主キー値が含まれます。

1. Azure Automation アカウントの構成を確認するには、**az140-dc-vm11** へのリモート デスクトップ セッション内で、Microsoft Edge を起動して、[Azure portal](https://portal.azure.com) に移動します。プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、**Automationアカウント**を検索して選択し、**Automationアカウント** ブレードで、新しくプロビジョニングされたAzure Automation アカウント (名前は **az140-automation-51** プレフィックスで始まる) を表すエントリを選択します。
1. 「Automation Account」 ブレードの左側の垂直メニューの **「Process Automation」** セクションで、**「Runbooks」** を選択し、Runbook のリストで **WVDAutoScaleRunbookARMBased** Runbook の存在を確認します。
1. 「Automation Account」 ブレードの左側の垂直メニューの **「アカウント設定」** セクションで、**「アカウントとして実行」** を選択し、右側のアカウントのリストで、**「Azure 実行アカウント」** の横にある **「+ 作成」** をクリックします。
1. **「Azure 実行アカウントの追加」** ブレードで、**「作成」** をクリックし、新しいアカウントが正常に作成されたことを確認します。

#### タスク 3: Azure Logic App を作成する

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 ウィンドウに切り替え、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、次のコマンドを実行して、自動スケール ソリューションの一部である Azure Logic アプリの作成に使用する PowerShell スクリプトをダウンロードします。

   ```powershell
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   Set-Location -Path $labFilesfolder
   $uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzLogicApp.ps1"
   Invoke-WebRequest -Uri $uri -OutFile ".\CreateOrUpdateAzLogicApp.ps1"
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 で、**C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzLogicApp.ps1** スクリプトを開き、**134** 行目と **138** 行目の間のコードを次のように複数行のコメントで囲みます。

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
	throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインで、以下を実行して、スクリプト パラメーターに割り当てる変数の値を設定します。

   ```powershell
   $AADTenantId = (Get-AzContext).Tenant.Id
   $AzSubscription = (Get-AzContext).Subscription.Id
   $ResourceGroup = Get-AzResourceGroup -Name 'az140-51-RG'
   $WVDHostPool = Get-AzResource -ResourceType "Microsoft.DesktopVirtualization/hostpools" -Name 'az140-21-hp1'
   $LogAnalyticsWorkspace = (Get-AzOperationalInsightsWorkspace -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $LogAnalyticsWorkspaceId = $LogAnalyticsWorkspace.CustomerId
   $LogAnalyticsWorkspaceKeys = (Get-AzOperationalInsightsWorkspaceSharedKey -ResourceGroupName $ResourceGroup.ResourceGroupName -Name $LogAnalyticsWorkspace.Name)
   $LogAnalyticsPrimaryKey = $LogAnalyticsWorkspaceKeys.PrimarySharedKey
   $RecurrenceInterval = 2
   $BeginPeakTime = '1:00'
   $EndPeakTime = '1:01'
   $TimeDifference = '0:00'
   $SessionThresholdPerCPU = 1
   $MinimumNumberOfRDSH = 1
   $MaintenanceTagName = 'CustomMaintenance'
   $LimitSecondsToForceLogOffUser = 5
   $LogOffMessageTitle = 'Autoscaling'
   $LogOffMessageBody = 'Forcing logoff due to autoscaling'

   $AutoAccount = (Get-AzAutomationAccount -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $AutoAccountConnection = Get-AzAutomationConnection -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName

   $WebhookURIAutoVar = Get-AzAutomationVariable -Name 'WebhookURIARMBased' -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName    $AutoAccount.AutomationAccountName
   ```

   >**注**: パラメーターの値は、自動スケール動作を加速することを目的としています。実稼働環境では、特定の要件に一致するように調整する必要があります。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」 スクリプト ペインを開き、次のスクリプトを貼り付けて実行し、自動スケール ソリューションの一部である Azure Logic アプリを作成します。

   ```powershell
   $Params = @{
     "AADTenantId"                   = $AADTenantId                             # Optional. If not specified, it will use the current Azure context
     "SubscriptionID"                = $AzSubscription.Id                       # Optional. If not specified, it will use the current Azure context
     "ResourceGroupName"             = $ResourceGroup.ResourceGroupName         # Optional. Default: "WVDAutoScaleResourceGroup"
     "Location"                      = $ResourceGroup.Location                  # Optional. Default: "West US2"
     "UseARMAPI"                     = $true
     "HostPoolName"                  = $WVDHostPool.Name
     "HostPoolResourceGroupName"     = $WVDHostPool.ResourceGroupName           # Optional. Default: same as ResourceGroupName param value
     "LogAnalyticsWorkspaceId"       = $LogAnalyticsWorkspaceId                 # Optional. If not specified, script will not log to the Log Analytics
     "LogAnalyticsPrimaryKey"        = $LogAnalyticsPrimaryKey                  # Optional. If not specified, script will not log to the Log Analytics
     "ConnectionAssetName"           = $AutoAccountConnection.Name              # Optional. Default: "AzureRunAsConnection"
     "RecurrenceInterval"            = $RecurrenceInterval                      # Optional. Default: 15
     "BeginPeakTime"                 = $BeginPeakTime                           # Optional. Default: "09:00"
     "EndPeakTime"                   = $EndPeakTime                             # Optional. Default: "17:00"
     "TimeDifference"                = $TimeDifference                          # Optional. Default: "-7:00"
     "SessionThresholdPerCPU"        = $SessionThresholdPerCPU                  # Optional. Default: 1
     "MinimumNumberOfRDSH"           = $MinimumNumberOfRDSH                     # Optional. Default: 1
     "MaintenanceTagName"            = $MaintenanceTagName                      # Optional.
     "LimitSecondsToForceLogOffUser" = $LimitSecondsToForceLogOffUser           # Optional. Default: 1
     "LogOffMessageTitle"            = $LogOffMessageTitle                      # Optional. Default: "Machine is about to shut down."
     "LogOffMessageBody"             = $LogOffMessageBody                       # Optional. Default: "Your session will be logged off. Please save and close everything."
     "WebhookURI"                    = $WebhookURIAutoVar.Value
   }

   .\CreateOrUpdateAzLogicApp.ps1 @Params
   ```

   >**注**: スクリプトが完了するのを待ちます。これにはおよそ 2 分かかる場合があります。

1. Azure Logic アプリの構成を確認するには、**az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示する Microsoft Edge ウィンドウに切り替え、**Logic Apps** を検索して選択し、**「Logic Apps」** ブレードで、**az140-21-hp1_Autoscale_Scheduler** という名前の新しくプロビジョニングされた Azure Logic アプリを表すエントリを選択します。
1. 「**az140-21-hp1_Autoscale_Scheduler**」 ブレードの左側にある垂直メニューの 「**開発ツール**」 セクションで、「**ロジック アプリ デザイナー**」 をクリックします。 
1. 「デザイナー」 ペインで、「**繰り返し**」というラベルの付いた長方形をクリックし、自動スケールの必要性が評価される頻度を制御するために使用できることに注意してください。 

### 演習 2: Azure Virtual Desktop セッションホストの自動スケールを確認およびレビューする

この演習の主なタスクは次のとおりです:

1. Azure Virtual Desktop セッションホストの自動スケールを確認する
1. Azure Log Analytics を使用して Azure Virtual Desktop イベントを追跡する

#### タスク 1: Azure Virtual Desktop セッションホストの自動スケールを確認する

1. Azure Virtual Desktop セッション ホストの自動スケールを確認するには、**az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示する Microsoft Edge ウィンドウで、**仮想マシン**を検索して選択し、**「仮想マシン」** ブレードで、**az140-21-RG** リソース グループ内の 3 つの Azure VM のステータスを確認します。
1. 3 つの Azure VM のうち 2 つが割り当て解除の過程にあるか、すでに **停止 (割り当て解除)** されていることを確認します。

   >**注**: 自動スケールが機能していることを確認したらすぐに、Azure Logic アプリを無効にして、対応する料金を最小限に抑える必要があります。

1. Azure Logic アプリを無効にするには、**az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示する Microsoft Edge ウィンドウにで、**Logic Apps** を検索して選択し、**「Logic Apps」** ブレードで、**az140-21-hp1_Autoscale_Scheduler** という名前の新しくプロビジョニングされた Azure Logic アプリを表すエントリを選択します。
1. **「az140-21-hp1_Autoscale_Scheduler」** ブレードのツールバーで、**「無効化」** をクリックします。 
1. **「az140-21-hp1_Autoscale_Scheduler」** ブレードの **「Essentials」** セクションで、過去 24 時間の成功した実行の数や、再発の頻度を示す **「Summary」** セクションなどの情報を確認します。 
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、**Automationアカウント**を検索して選択し、**Automationアカウント** ブレードで、新しくプロビジョニングされた Azure Automation アカウント (名前は **az140-automation-51** プレフィックスで始まる) を表すエントリを選択します。
1. **「Automation Account」** ブレードの左側の垂直メニューの **「Process Automation」** セクションで、**「ジョブ」** を選択し、**WVDAutoScaleRunbookARMBased** Runbook の個々の呼び出しに対応するジョブのリストを確認します。
1. 最新のジョブを選択し、そのブレードで **「すべてのログ」** タブのヘッダーをクリックします。これにより、ジョブ実行ステップの詳細なリストが表示されます。

#### タスク 2: Azure Log Analytics を使用して Azure Virtual Desktop イベントを追跡する

>**注**: 自動スケールやその他の Azure Virtual Desktop イベントを分析するには、Log Analytics を使用できます。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、**Log Analytics ワークスペース**を検索して選択し、**「Log Analytics ワークスペース」** ブレードで、このラボで使用されている Azure Log Analytics ワークスペースを表すエントリを選択します (名前は **az140-workspace-51** プレフィックスで始まります)。
1. 「Log Analytics ワークスペース」 ブレードの左側の垂直メニューの **「全般」** セクションで、**「ログ」** をクリックし、必要に応じて、**「Log Analytics へようこそ」** ペインで **「クエリ」** をクリックします。
1. **「クエリ」** ペインの左側にある **「すべてのクエリ」** 垂直メニューで、**「Azure Virtual Desktop」** を選択し、事前定義されたクエリを確認します。
1. **「クエリ」** ペインを閉じます。これにより、**「新しいクエリ 1」** タブが自動的に表示されます。
1. 「クエリ」 ウィンドウで、次のクエリを貼り付け、**「実行」** をクリックして、このラボで使用されているホスト プールのすべてのイベントを表示します。

   ```kql
   WVDTenantScale_CL
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

   >**注**: 結果が表示されない場合は、数分待ってからもう一度確認してください。

1. 「クエリ」 ウィンドウで、次のクエリを貼り付け、**「実行」** をクリックして、ターゲット ホスト プールで現在実行中のセッション ホストとアクティブなユーザー セッションの総数を表示します。

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Number of running session hosts:"
     or logmessage_s contains "Number of user sessions:"
     or logmessage_s contains "Number of user sessions per Core:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. 「クエリ」 ウィンドウで、次のクエリを貼り付け、**「実行」** をクリックして、ホスト プールですべてのセッション ホスト VM のステータスを表示を表示します。

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Session host:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. 「クエリ」 ウィンドウで、次のクエリを貼り付け、**「実行」** をクリックして、スケーリング関連のエラーと警告を表示します。

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "ERROR:" or logmessage_s contains "WARN:"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

>**注**: `TenantId` に関するエラーメッセージを無視します

### 演習 3: ラボでプロビジョニングされた Azure VM を停止および割り当て解除する

この演習の主なタスクは次のとおりです:

1. ラボでプロビジョニングされた Azure VM を停止および割り当て解除する

>**注**: この演習では、このラボでプロビジョニングした Azure VM を割り当て解除し、対応するコンピューティング料金を最小化します

#### タスク 1: ラボでプロビジョニングされた Azure VM を割り当て解除する

1. ラボ コンピューターに切り替え、Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成されたすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**: コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
