# 3.4 School Data Sync でのアカウント管理

---

## 📖 学習目標

Microsoft School Data Sync（SDS）を活用した教育機関向けの自動化されたアカウント管理手法を習得し、学生情報システム（SIS）と Microsoft 365 の効率的な連携を実現できるスキルを身につけます。

### 🎯 この節で習得すること

- **School Data Sync の概要**と教育機関での活用意義を理解し、適切な導入判断ができる
- **CSVファイル連携**と**OneRoster API連携**の特徴を把握し、組織に適した方法を選択できる
- **自動化されたユーザー・クラス管理**の設定方法をマスターし、運用負荷を大幅に削減できる
- **Microsoft Teams との連携**設定を理解し、教育現場でのデジタル授業環境を構築できる
- **トラブルシューティング**能力を身につけ、同期エラーや運用課題に対処できる

### 💼 実務での活用場面

- **新年度対応**: 校務支援システムからの自動的な新入生・新任教員アカウント作成
- **クラス編成**: 時間割・履修情報に基づく Microsoft Teams クラスの自動生成
- **保護者連携**: 保護者情報の同期による Teams での保護者コミュニケーション環境構築
- **デジタル授業**: OneNote Class Notebook、課題機能等の教育ツールとの自動連携

---

## ⚡ 前提条件と重要な変更点

### 📅 School Data Sync の新しいエクスペリエンス

> ⚠️ **重要な変更**: School Data Sync (クラシック) エクスペリエンスは、2024 年 12 月 31 日に終了します。2025年以降は新しいSchool Data Syncエクスペリエンスが標準となります。

| 項目 | 従来版（クラシック） | 新版（2025年〜） |
|------|-------------------|-----------------|
| **提供期間** | 〜2024年12月31日 | 2025年1月〜 |
| **新規作成** | 2024年7月1日以降停止 | 利用可能 |
| **対応形式** | SDS v1 CSV、OneRoster 1.0 | SDS v2.1 CSV、OneRoster 1.1 API |
| **主要機能** | 基本的な同期機能 | 強化された分析・監視機能 |

### 必要な権限・環境

```markdown
📋 School Data Sync 導入の前提条件
- [ ] **Microsoft 365 Education**ライセンス（A1、A3、A5のいずれか）
- [ ] **グローバル管理者**権限
- [ ] **学生情報システム（SIS）**からのデータエクスポート機能
- [ ] **ドメイン管理権限**（カスタムドメイン使用時）
- [ ] **テスト環境**での事前検証環境
```

> 💡 **日本での活用**: 文部科学省が掲げる GIGA スクール構想に対応するために、マイクロソフトが提供する教育ソリューションとして、SDSは日本の教育機関でも積極的に活用されています。

---

## 🔍 School Data Sync とは

### 基本概念と特徴

School Data Sync (SDS) は、学生情報または管理システムのユーザーと名簿データを Microsoft 365 と同期するプロセスを自動化するのに役立つ、教育機関向けの無料サービスです。

#### 主要な機能

**🔄 自動同期機能**
- **ユーザーアカウント**: 学生・教職員のアカウント自動作成・更新
- **組織階層**: 学校・学部・学科等の組織構造の同期
- **クラス編成**: 履修情報に基づくクラスグループの自動生成
- **権限管理**: 役職・職種に応じた適切な権限割り当て

**📚 Microsoft 365 統合**
- **Microsoft Teams**: クラスチームの自動作成とメンバー追加
- **OneNote Class Notebook**: 授業用ノートブックの自動プロビジョニング
- **SharePoint Online**: クラス専用サイトの作成
- **Exchange Online**: クラス用メール配布リストの生成

### 教育機関での具体的なメリット

#### 運用負荷の大幅削減

**従来の手動運用**
```markdown
❌ 従来の課題
- 新年度時の数百〜数千アカウントの手動作成（数週間の作業）
- クラス変更時の手動メンバー調整（各学期ごと）
- 卒業・転校時の個別アカウント処理
- Teams クラスの手動作成・メンバー追加
```

**SDS による自動化**
```markdown
✅ SDS による改善
- 校務支援システムからの自動同期（数時間で完了）
- 時間割変更の自動反映（リアルタイム同期）
- 卒業・転校処理の自動化
- Teams クラスの自動生成・管理
```

#### 教育効果の向上

Microsoft Teams に組み込まれたReading Coachと Reading Progressで、基礎的なリテラシースキルを伸ばしますなど、SDSによる環境整備により、デジタル教育ツールの活用が促進されます。

---

## 📊 データ連携方式の比較と選択

### 方式1: CSV ファイル連携

#### SDS v2.1 CSV 形式（推奨）

SDS V2.1 CSV ファイル形式を使用してデータを取り込むと、プロビジョニング用のコア SDS 機能を利用でき、Microsoft 365 の製品と機能のエクスペリエンスを強化するのにも役立ちます。

**📄 必須ファイル構成**
```
orgs.csv        # 組織情報（学校・学部・学科等）
users.csv       # 全ユーザー情報（学生・教職員・保護者）
roles.csv       # ユーザーと組織の関係・役割
classes.csv     # クラス・科目情報
```

**📄 オプションファイル**
```
enrollments.csv     # 履修・受講情報
academicSessions.csv # 学期・学年情報
relationships.csv   # 保護者と学生の関係
courses.csv        # コース情報
```

#### CSV形式のメリット・デメリット

| メリット | デメリット |
|---------|------------|
| **簡単な導入**: 既存システムからのエクスポートが容易 | **手動更新**: データ変更時は手動アップロードが必要 |
| **幅広い対応**: ほとんどのSISがCSVエクスポートに対応 | **同期頻度**: 1日最大6回の制限 |
| **柔軟性**: データ内容の事前確認・編集が可能 | **作業負荷**: 定期的なアップロード作業が必要 |
| **コスト**: 追加ライセンス不要 | **リアルタイム性**: 即座の反映は困難 |

### 方式2: OneRoster API 連携

#### API連携の特徴

OneRoster API を使用してこの同期方法により、SIS/SMS プロバイダーが開発した REST ベースの OneRoster 1.1 API を使用して、SIS/SMS に直接接続できます。

**🔗 接続に必要な情報**
```
Web Access URL    # OneRoster APIのエンドポイントURL
Client ID         # API接続用のクライアントID
Client Secret     # API接続用のクライアントシークレット
Access Token URL  # OAuth2認証用のトークンURL（オプション）
```

#### API連携のメリット・デメリット

| メリット | デメリット |
|---------|------------|
| **リアルタイム同期**: データ変更の即座反映 | **対応システム限定**: OneRoster対応SISが必要 |
| **自動化**: 手動アップロード作業が不要 | **技術的複雑性**: API設定の専門知識が必要 |
| **継続同期**: 常時接続によるデータ整合性確保 | **依存性**: SIS側のAPI安定性に依存 |
| **差分同期**: 変更分のみの効率的な同期 | **コスト**: SIS側でのAPI機能が有償の場合あり |

### 教育機関規模別推奨方式

| 規模 | 学生数 | 推奨方式 | 理由 |
|------|---------|----------|------|
| **小規模** | 〜500名 | CSV形式 | 導入の簡易性、運用コストの低さ |
| **中規模** | 500〜2000名 | CSV形式 または API | 更新頻度とコストのバランスで判断 |
| **大規模** | 2000名〜 | OneRoster API | リアルタイム性と運用効率の重要性 |

---

## 🛠️ CSV ファイル連携の設定手順

### Step 1: CSVファイルの準備

#### 基本的なファイル構造例

**orgs.csv（組織情報）**
```csv
sourcedId,name,type,identifier,parentSourcedId
sch001,サンプル大学,university,SAMPLE_UNIV,
fac001,情報学部,college,INFO_FACULTY,sch001
dept001,情報工学科,department,INFO_DEPT,fac001
dept002,数学科,department,MATH_DEPT,fac001
```

**users.csv（ユーザー情報）**
```csv
sourcedId,username,givenName,familyName,email,role,orgSourcedIds
usr001,s2025001,太郎,田中,s2025001@sample-univ.ac.jp,student,dept001
usr002,t001,花子,佐藤,t001@sample-univ.ac.jp,teacher,dept001
usr003,a001,一郎,山田,a001@sample-univ.ac.jp,administrator,sch001
```

**roles.csv（役割情報）**
```csv
sourcedId,userSourcedId,roleType,orgSourcedId,isPrimary
role001,usr001,student,dept001,true
role002,usr002,teacher,dept001,true
role003,usr003,administrator,sch001,true
```

**classes.csv（クラス情報）**
```csv
sourcedId,title,courseSourcedId,orgSourcedId,termSourcedIds
cls001,プログラミング基礎A,crs001,dept001,term001
cls002,データ構造とアルゴリズム,crs002,dept001,term001
cls003,基礎数学I,crs003,dept002,term001
```

#### データ形式の注意点

**📝 必須要件**
- **文字エンコード**: UTF-8（BOM無し）
- **日付形式**: ISO 8601形式（YYYY-MM-DD）
- **電話番号**: E.164形式（+81901234567）
- **必須フィールド**: 各ファイルの「Required」列が「Yes」の項目

**⚠️ よくある問題**
```markdown
❌ 避けるべき形式
- Shift-JIS、CP932等の日本語エンコード
- 和暦表記（平成31年4月1日）
- 国内形式の電話番号（090-1234-5678）
- Excel独自の日付シリアル値

✅ 正しい形式
- UTF-8エンコード
- 西暦表記（2025-04-01）
- 国際形式の電話番号（+81901234567）
- ISO形式の日付
```

### Step 2: Microsoft 365 管理センターでの設定

#### 初期設定の手順

1. **School Data Sync の有効化**
   ```markdown
   1. Microsoft 365 管理センター（https://admin.microsoft.com）にアクセス
   2. 左側メニューから「設定」→「組織設定」を選択
   3. 「School Data Sync」を検索して選択
   4. 「School Data Sync を有効にする」をクリック
   5. 15分程度でサービスがプロビジョニング完了
   ```

2. **データ接続の作成**
   ```markdown
   1. SDS管理ポータル（https://sds.edu.cloud.microsoft/）にアクセス
   2. 「Connect your data」をクリック
   3. 「Create new inbound flow」を選択
   4. データソース：「Connect to my data」を選択
   5. フォーマット：「CSV」を選択
   6. フォーマットバージョン：「SDS v2.1 CSV」を選択
   ```

3. **CSVファイルのアップロード**

ファイル名が検証された後、ファイルがアップロードされ、選択されたファイルの構造を検証し、SDS CSV v2.1 仕様に基づいてヘッダー名で正しくフォーマットされているかを確認します。

#### ユーザーIDマッチング規則の設定

**🔍 マッチング方式の選択**
```markdown
スタッフ（教職員）のマッチング規則:
- UPN（User Principal Name）
- メールアドレス
- 社員番号（SIS ID）

学生のマッチング規則:
- UPN（User Principal Name）
- メールアドレス  
- 学籍番号（SIS ID）
```

> 💡 **運用ヒント**: 異なる規則を適用する場合は「Split option」を選択し、スタッフと学生で個別に設定することが可能です。

### Step 3: 同期の実行と確認

#### 同期処理の監視

**📊 同期状況の確認**
```markdown
1. SDS管理ポータルの「Health and Monitoring」タブ
2. 同期実行のステータス確認
   - Connected: データソースとの接続確認
   - Imported: データの取り込み完了
   - Validated: データ形式の検証完了
   - Processed: Microsoft 365への同期完了
```

**🔔 同期エラーの対処**
```markdown
よくあるエラーと対処法:
- "File format error": CSVヘッダーの確認、UTF-8エンコードの確認
- "User matching failed": UPN重複の確認、マッチング規則の見直し
- "License assignment failed": ライセンス在庫の確認
- "Permission denied": 管理者権限の確認
```

---

## 🌐 OneRoster API 連携の設定手順

### Step 1: SIS側のAPI設定確認

#### 対応プロバイダーの確認

SDS では、OneRoster Provider が一覧にない場合、OneRoster Provider Overview で、パイロット テストに参加する手順や、OneRoster プロバイダーとして追加するためにプロバイダーに送信する情報を参照してください。

**🏢 主要な対応SISプロバイダー**
- PowerSchool
- Clever
- ClassLink
- Infinite Campus
- その他多数のOneRoster 1.1対応システム

#### API接続情報の取得

**📋 必要な接続情報**
```markdown
基本接続情報:
- Web Access URL: https://your-sis.example.com/ims/oneroster/v1p1
- Client ID: api_client_id_from_sis
- Client Secret: api_client_secret_from_sis

OAuth2使用時の追加情報:
- Access Token URL: https://your-sis.example.com/oauth/token
- Grant Type: client_credentials（推奨）
```

### Step 2: SDS での API 接続設定

#### 接続ウィザードの実行

1. **プロバイダーの選択**
   ```markdown
   1. SDS管理ポータルで「Connect your data」を選択
   2. データソース形式：「API」を選択
   3. 対応プロバイダー一覧から該当するSISを選択
   4. 「Next」をクリックして接続設定へ進む
   ```

2. **認証情報の設定**
   ```markdown
   1. Web Access URL: SISのOneRoster APIエンドポイントを入力
   2. Client ID: SIS管理者から提供された認証ID
   3. Client Secret: SIS管理者から提供された認証シークレット
   4. Access Token URL: OAuth2使用時のトークンエンドポイント
   ```

3. **接続テストの実行**

SDS は、最後の画面で入力された情報に基づいて、SIS/SMS への接続をテストします。

#### オプションデータの設定

**📊 取り込み可能なオプションデータ**
```markdown
利用可能な場合のオプション設定:
☑️ Demographics（人口統計情報）
☑️ Student Contact Relationships（保護者情報）  
☑️ Student User Flags（学生フラグ情報）
☑️ Custom Properties（カスタムプロパティ）
```

> 💡 **プロバイダー依存**: プロバイダーからサポートされているオプション データ機能に基づいて、他のデータを含めるためのトグルが表示されます。利用可能な機能はSISプロバイダーによって異なります。

---

## 🚀 Microsoft Teams クラス連携の設定

### Teams for Education との自動連携

#### クラスチーム作成の自動化

チームの作成: オンにすると、クラス グループが作成されるときに、クラス グループに基づいてクラス チームを作成するように Teams に要求が送信されます。

**⚙️ 自動作成の設定オプション**
```markdown
Teams クラス作成設定:
☑️ 自動チーム作成: SDSでクラスグループ作成時に自動でTeamsクラスを生成
☑️ 教師による事前確認: 学生アクセス前に教師が内容を確認・準備
☑️ OneNote統合: クラスノートブックの自動作成
☑️ 課題機能: Assignments機能の自動有効化
```

#### 教師の準備期間とアクティベーション

**👨‍🏫 教師の事前準備**

教育者、または SDS から作成された Class Teams を使用しているグループ所有者の場合、学生やその他のグループ メンバーの前に早期アクセスできます。教師の準備ができたら、「アクティブ化」を選択して、学生やその他のグループ メンバーへのアクセスを許可できます。

```markdown
🔄 アクティベーションプロセス
1. SDSによるクラスチーム自動作成（教師のみアクセス可能）
2. 教師による授業準備
   - チャンネル構成の調整
   - 授業資料のアップロード
   - OneNoteクラスノートブックの設定
   - 課題テンプレートの準備
3. 準備完了後に「アクティブ化」実行
4. 学生のアクセス許可とTeams利用開始
```

### 保護者との連携設定

#### School Connection アプリの活用

スクール コネクションを有効にすると、同期したすべての保護者の連絡先がMicrosoft Teamsモバイル アプリにサインインし、学生をプロフィールに追加できます。

**👨‍👩‍👧‍👦 保護者連携の設定要件**
```markdown
保護者情報の設定方法:
1. SDS CSV: relationships.csv ファイルでの保護者情報設定
2. API同期: OneRoster APIでの保護者情報取得
3. 手動設定: 個別の保護者アカウント作成

必要なCSVファイル（v2.1形式）:
- orgs.csv
- users.csv（保護者情報含む）
- roles.csv
- relationships.csv（保護者と学生の関係）
```

**📱 保護者向けアプリ機能**
```markdown
School Connection アプリでの利用可能機能:
- 学生の課題進捗確認
- 授業出席状況の確認
- 教師とのメッセージ交換
- 学校からのお知らせ受信
- 成績・評価の確認
```

---

## 🔧 運用管理とトラブルシューティング

### 定期的な運用業務

#### データ更新の手順

**📅 CSV形式での定期更新**

最初の実行後にソース データからの変更を更新するために手動で変更をアップロードするには、「ソース データを SDS CSV で更新する」を参照してください。

```markdown
🔄 更新サイクルの推奨例

日次更新（必要に応じて）:
- 新入生・転入生の追加
- 退学・転出の処理
- 緊急的なクラス変更

週次更新:
- 履修登録の変更
- 教職員の異動
- 保護者情報の更新

学期更新:
- 新学期のクラス編成
- 時間割の大幅変更
- 組織構造の変更
```

**⚠️ 更新時の制限**
- 1日あたり最大6回のアップロード（対応する実行付き）
- ファイルセットの一貫性維持（初回と同じファイル構成を維持）

#### 同期ステータスの監視

**📊 Health and Monitoring ダッシュボード**
```markdown
監視すべき主要指標:
✅ 接続状態: SISとの接続が正常
✅ 同期成功率: エラー率5%以下を維持
✅ 処理時間: 通常の同期時間内での完了
✅ ユーザー数: 予想されるユーザー数と一致
✅ クラス数: 予想されるクラス数と一致
```

### よくある問題と解決方法

#### 問題1: ユーザー同期エラー

**症状**: 特定のユーザーアカウントが同期されない

**🔍 診断手順**
```markdown
1. ユーザーIDの重複確認
   - CSV内での重複チェック
   - Microsoft 365 内の既存ユーザー確認

2. データ形式の確認
   - メールアドレス形式の妥当性
   - 必須フィールドの入力漏れ

3. ライセンス割り当ての確認
   - 利用可能ライセンス数
   - ユーザー種別とライセンス種別の整合性
```

**💡 解決方法**
```powershell
# Microsoft Graph PowerShell でのトラブルシューティング

# 重複ユーザーの確認
Get-MgUser -Filter "startsWith(userPrincipalName,'dup_user')" | 
    Select-Object DisplayName, UserPrincipalName, Id

# ライセンス状況の確認
Get-MgSubscribedSku | 
    Select-Object SkuPartNumber, ConsumedUnits, 
    @{Name="AvailableUnits";Expression={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}}

# SDS特有の属性確認
Get-MgUser -UserId "user@domain.com" | 
    Select-Object -ExpandProperty AdditionalProperties | 
    Where-Object {$_.Key -like "*education*"}
```

#### 問題2: Teams クラス作成失敗

**症状**: クラスグループは作成されるが、Teams チームが生成されない

**🔍 原因と対処**
```markdown
原因1: Teams ライセンスの不足
対処: A1/A3/A5 Education ライセンスの割り当て確認

原因2: Teams サービスプランの無効化
対処: ライセンス管理でTeams機能の有効化

原因3: 組織のTeams設定による制限
対処: Teams管理センターでのポリシー確認

原因4: クラス命名規則の問題
対処: 特殊文字の除去、文字数制限の確認
```

#### 問題3: 保護者アクセスエラー

**症状**: 保護者が School Connection アプリにアクセスできない

**🔧 確認ポイント**
```markdown
1. 保護者データの同期確認
   - relationships.csv の形式確認
   - 保護者ユーザーの作成状況確認

2. アプリ設定の確認
   - School Connection の有効化状態
   - 管理単位の設定確認

3. 保護者アカウントの確認
   - 個人メールアドレスの使用確認
   - 電話番号認証の実行状況
```

#### 問題4: 大規模更新時のパフォーマンス問題

**症状**: 数千名規模の更新で処理時間が長時間になる

**⚡ パフォーマンス最適化**
```markdown
最適化手法:
1. 段階的更新: 部署・学年別の分割更新
2. 差分更新: 変更分のみの更新（API連携で自動化）
3. 夜間実行: 利用者の少ない時間帯での更新
4. 事前検証: テスト環境での動作確認

推奨更新サイズ:
- 新規ユーザー: 500名/回以下
- 既存ユーザー更新: 1000名/回以下
- クラス変更: 100クラス/回以下
```

---

## 📈 高度な活用方法

### Education Insights との連携

#### 学習分析データの活用

School Data Sync によってクラス情報を追加したうえで Education Insights Premium (無償) を使うことで、管理者はチームを横断した活動状況の分析が可能になります。

**📊 分析可能な指標**
```markdown
個別学生レベル:
- Teams への参加頻度
- 課題の提出状況と成績
- OneDrive でのファイル活動
- OneNote での学習記録

クラス・科目レベル:
- クラス全体の参加度
- 課題の平均得点
- 協働学習の状況
- 教材へのアクセス頻度

学校・組織レベル:
- ICT活用度の全体傾向
- 各科目での利用状況比較
- 学年進行での活用変化
- 教師の指導方法分析
```

### Power Platform との統合

#### カスタムアプリケーションの構築

**🔧 統合シナリオ例**
```markdown
Power Apps:
- 保護者向け成績確認アプリ
- 教職員向け出席管理アプリ
- 施設予約・管理システム

Power Automate:
- 欠席連絡の自動処理フロー
- 成績通知の自動送信
- 緊急連絡の一斉配信

Power BI:
- 学習データの可視化ダッシュボード
- 出席率・成績分析レポート
- ICT活用状況の経年変化分析
```

### サードパーティ アプリとの連携

#### Education API の活用

Microsoft Graph を通じて、プロビジョニングによりサードパーティ アプリケーションで名簿情報を利用できるようになります。

**🌐 連携可能なアプリケーション例**
```markdown
学習管理システム（LMS）:
- Moodle
- Canvas
- Blackboard

評価・採点システム:
- Gradebook アプリ
- オンライン テスト システム
- ポートフォリオ システム

コミュニケーション ツール:
- 校内放送システム
- 安全確認システム
- 図書館管理システム
```

---

## ✅ 学習成果の確認

### 習得スキルチェック

以下の項目について、実際に操作・設定できることを確認してください：

- [ ] **SDS の有効化**: Microsoft 365 でのSchool Data Sync サービス有効化
- [ ] **CSV設計**: 教育機関の要件に合わせたCSVファイル設計・作成
- [ ] **接続設定**: CSVファイルまたはAPI連携での同期設定
- [ ] **Teams連携**: クラスチーム自動作成と教師アクティベーション
- [ ] **保護者連携**: School Connection アプリでの保護者機能設定
- [ ] **監視運用**: 同期状況の監視とエラー対応
- [ ] **トラブルシューティング**: 一般的な問題の診断と解決

### 実践課題: 教育機関向け導入プロジェクト

**🎯 総合演習: 中学校でのSDS導入**

```markdown
## 背景
地域の公立中学校（生徒600名、教職員50名）でMicrosoft 365 Educationを導入。
校務支援システムからの自動連携でアカウント管理を効率化したい。

## 要件
- 生徒アカウント: 各学年200名 × 3学年
- 教職員アカウント: 教員40名、事務職員10名
- 保護者連携: 保護者約800名との連絡体制構築
- クラス編成: 1学年6クラス × 3学年 = 18クラス

## 実施内容
1. SDS導入方式の決定（CSV vs API）
2. 校務支援システムからのデータエクスポート設計
3. CSV形式の設計と検証用データ作成
4. テスト環境での動作確認
5. 本番環境での段階的導入
6. Teams クラス環境の構築
7. 保護者向けガイダンス資料作成
8. 運用手順書とトラブルシューティングガイド作成

## 成果物
- SDS設定仕様書
- CSVファイルテンプレート
- 導入手順書（管理者向け）
- 利用ガイド（教師・保護者向け）
- 運用監視・保守計画書
```

---

## 🚀 次のステップ

この節を完了したら、以下の学習・発展を推奨します：

### 推奨学習パス

1. **[Microsoft Teams for Education 管理]**
   - 教育機関特有のTeams設定
   - クラスルーム機能の活用

2. **[Intune for Education 導入]**
   - SDSと連携したデバイス管理
   - 教育機関向けアプリ配布

3. **[Microsoft 365 Education Analytics]**
   - 学習データの分析活用
   - Education Insights の高度活用

### 発展学習

- **Microsoft Graph Education API**: カスタムアプリケーション開発
- **Power Platform Education**: ノーコード/ローコード開発
- **Azure Active Directory B2B**: 他機関との連携
- **Compliance Center**: 教育機関でのデータ保護対応

---

## 📚 参考リソース

### Microsoft 公式ドキュメント

- [School Data Sync ウェルカム ページ](https://learn.microsoft.com/ja-jp/schooldatasync/)
- [SIS の同期 - M365 Education](https://learn.microsoft.com/ja-jp/microsoft-365/education/deploy/school-data-sync)
- [SDS V2.1 CSV File Format](https://learn.microsoft.com/en-us/SchoolDataSync/sds-v2.1-csv-file-format)

### 実用ツール・テンプレート

- **SDS GitHub Repository**: サンプルCSVファイルとスクリプト
- **OneRoster API プロバイダー一覧**: 対応SISの確認
- **Education Insights ガイド**: 学習分析の活用方法

### 日本国内のサポート

- [Microsoft GIGA スクールパッケージ](https://www.microsoft.com/ja-jp/biz/education/gigaschool-sds.aspx)
- **日本マイクロソフト教育機関サポート**: 導入支援・技術相談
- **パートナー企業**: SDS導入支援サービス

### コミュニティ・情報交換

- **Microsoft Education Community**: 教育関係者向けコミュニティ
- **Tech Community - Education**: 技術的な質問・情報交換
- **教育機関向けユーザー会**: 事例共有・ベストプラクティス

---

## 💡 導入成功のポイント

### 計画フェーズ

```markdown
🎯 成功要因
- 校務支援システムとの連携可能性の事前確認
- 教職員・保護者への十分な説明と研修
- 段階的導入による影響範囲の限定
- テスト環境での十分な検証

⚠️ 失敗リスク
- データ形式の事前確認不足
- 関係者への周知・研修不足
- 一括導入による影響の拡大
- バックアップ・復旧計画の不備
```

### 運用フェーズ

```markdown
📈 継続的改善
- 定期的な同期状況の監視
- ユーザーフィードバックの収集・反映
- 新機能・アップデートへの対応
- 運用手順の定期的な見直し

🔧 技術的考慮事項
- SIS側のバージョンアップ対応
- Microsoft 365新機能との連携
- セキュリティ設定の定期見直し
- パフォーマンス監視と最適化
```

---

*📅 最終更新: 2025年6月2日*  
*✍️ 更新者: Microsoft 365 ハンドブック編集チーム*  
*📖 対象バージョン: School Data Sync (新エクスペリエンス 2025年版)*