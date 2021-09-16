# AZ-140: Microsoft Azure Virtual Desktop の構成と運用

- **[最新の学生ハンドブックと AllFiles コンテンツをダウンロードする](../../releases/latest)**
- **MCT の場合:** - [MCT 向け GitHub ユーザー ガイド](https://microsoftlearning.github.io/MCT-User-Guide-JA/)をご覧ください
- **ラボの手順を手動でビルドする必要がありますか。** - 手順は、[MicrosoftLearning/Docker-Build](https://github.com/MicrosoftLearning/Docker-Build) リポジトリで利用可能です

## 実行内容

- このコースをサポートするには、コースで使用される Azure サービスを最新の状態に保つために、コース コンテンツを頻繁に更新する必要があります。  ラボの手順とラボ ファイルは GitHub で公開しています。これにより、コースの作成者と MCT 間でのオープンな作業が可能となり、Azure プラットフォームの変更に合わせてコンテンツを最新の状態に保つことができます。

- ラボでの共同作業をこれまで以上に向上できることを願っています。Azure サービスの内容が変更され、修正が必要な点をラボの実施中に見つけた場合は、ラボのソースをすぐに修正してください。  MCT をお互い支援できるようご協力お願いします。

## リリース済みの MOC ファイルと比べ、これらのファイルはどのように利用できますか?

- 講師用ハンドブックと PowerPoint が、コースの内容を教えるための主要なソースであることに変わりはありません。

- GitHub にあるファイルは、受講者ハンドブックと組み合わせて使えるように設計されています。ただし、MCT とコースの作成者が最新のラボ ファイルのソースを共有できるように、中央リポジトリとして GitHub に置いてあります。

- 講師は、ラボを行うたびに、最新の Azure サービスに合わせて修正された個所がないか GitHub を確認し、最新のラボ用ファイルを取得してください。

## 受講者ハンドブックの変更については?

- 受講者ハンドブックは四半期ごとに見直し、必要があれば通常の MOC リリースの手順を通して更新します。

## 貢献するには?

- MCT は、GitHub repro のコードまたはコンテンツにプル要求を送信できます。Microsoft とコースの作成者は、必要に応じてコンテンツとラボ コードの変更をトリアージして含めます。

- バグ、変更、改善、アイデアを提出できます。  新しい Azure 機能を見つけましたか?  新しいデモを送信してください。

## 注

**ラボは、使用する ID プロバイダーに応じて、2 つの別々のトラックで構成されます。**

- Active Directory ドメイン サービス (AD DS)。このトラックは、次のラボで構成されています。

   - LAB_01L01_Prepare_for_deployment_of_AVD_ADDS.md
   - LAB_02L01_Deploy_host_pools_and_session_hosts_with_the_Azure_portal_ADDS.md
   - LAB_02L02_Implement_and_manage_storage_for_AVD_ADDS.md
   - LAB_02L03_Deploy_host_pools_and_hosts_with_ARM_templates_ADDS.md
   - LAB_02L04_Deploy_and_manage_host_pools_and_hosts_with_PowerShell_ADDS.md
   - LAB_02L05_Create_and_manage_session_host_images_ADDS.md
   - LAB_03L01_Configure_Conditional_Access_policies_for_AVD_ADDS.md
   - LAB_04L01_Implement_and_manage_AVD_profiles_ADDS.md
   - LAB_04L02_Package_AVD_applications_ADDS.md
   - LAB_05L01_Implement_autoscaling_in_host_pools_ADDS.md

- Azure Active Directory Domain Services (Azure AD DS)。このトラックは、次のラボで構成されています。

   - LAB_01L01_Prepare_for_deployment_of_AVD_AADDS.md
   - LAB_02L01_Create_and_configure_host_pools_and_session_hosts_AADDS.md
   - LAB_02L02_Implement_and_manage_storage_for_AVD_AADDS.md
   - LAB_04L01_Implement_and_manage_AVD_profiles_AADDS.md

### コース教材

GitHub の資料は MCT やパートナーがアクセスして入手し、その後受講者へ提供することを強く推奨します。  コースを開催中にラボの手順を示す目的で、受講者に直接 GitHub へアクセスするよう指示した場合、コースの中で別の UI を扱わなければならず、受講者にとっては混乱を招くことになります。ラボのたびに別の教材を提供する理由として、クラウドベースのインターフェイスやプラットフォームは常に進化し続けているという特質を強調して伝えられます。GitHub 上のファイルへのアクセスと GitHub サイトのナビゲーションに関する Microsoft Learning のサポートは、このコースを指導する MCT のみが利用できます。
