# 3.6 ユーザーロールと権限管理

---

## 📖 学習目標

Microsoft 365環境における適切な権限設計と管理手法を習得し、最小権限の原則に基づいたセキュアで効率的な権限体系を構築できるスキルを身につけます。

### 🎯 この節で習得すること

- **管理者ロールの種類と特徴**を理解し、適切な権限分離ができる
- **最小権限の原則**を実践し、セキュリティリスクを最小化できる
- **カスタムロール**の設計・作成により、組織固有の要件に対応できる
- **Privileged Identity Management (PIM)**を活用し、特権アクセスを安全に管理できる
- **権限監査と見直し**プロセスを構築し、継続的なセキュリティ向上を実現できる

### 💼 実務での活用場面

- **セキュリティコンプライアンス**: 監査要件を満たす権限設計と管理体制構築
- **組織変更対応**: 人事異動・昇格・退職時の適切な権限変更処理
- **インシデント対応**: セキュリティ侵害時の迅速な権限無効化・復旧
- **運用効率化**: 業務に必要な最小限の権限付与による生産性向上
- **教育機関特有**: 学期制・年度制に対応した動的な権限管理

---

## ⚡ 前提条件

### 必要な権限・環境

- [ ] **Microsoft 365テナント**（本番環境または十分な権限を持つ試用版）
- [ ] **グローバル管理者**権限または**特権ロール管理者**権限
- [ ] **Microsoft Entra ID Premium P2**ライセンス（PIM機能使用時）
- [ ] **PowerShell 5.1以上**（Microsoft Graph PowerShell SDK v2.0+）

### 事前準備

```markdown
📋 権限管理準備チェックリスト
- [ ] 組織の職責・業務フローの詳細把握
- [ ] 現在の権限付与状況の棚卸し
- [ ] セキュリティポリシー・コンプライアンス要件の確認
- [ ] 緊急時対応手順の策定
- [ ] 権限変更承認プロセスの整備
- [ ] 監査ログ保存・分析体制の構築
```

> 💡 **重要**: 権限管理は組織のセキュリティの根幹です。変更前には必ず影響範囲を十分に検討し、テスト環境での検証を行ってください。

---

## 🔐 Microsoft 365 管理者ロールの理解

### 主要な管理者ロールと責任範囲

Microsoft 365では50以上の管理者ロールが提供されており、業務要件に応じた適切な権限分離が可能です。

#### Tier 0（最高権限レベル）

| ロール名 | 主な権限範囲 | 適用対象 | セキュリティリスク |
|----------|-------------|----------|-------------------|
| **グローバル管理者** | テナント全体の設定・管理 | CTO、IT部長レベル | ★★★★★ |
| **特権認証管理者** | 全ユーザーの認証設定 | セキュリティ責任者 | ★★★★★ |
| **特権ロール管理者** | 管理者ロール割り当て | 人事・権限管理者 | ★★★★★ |

```powershell
# Tier 0ロールの現在の割り当て状況確認
$tier0Roles = @(
    "Global Administrator",
    "Privileged Authentication Administrator", 
    "Privileged Role Administrator"
)

foreach ($roleName in $tier0Roles) {
    $role = Get-MgDirectoryRole -Filter "displayName eq '$roleName'"
    if ($role) {
        $members = Get-MgDirectoryRoleMember -DirectoryRoleId $role.Id
        Write-Host "$roleName 割り当て数: $($members.Count)名" -ForegroundColor Red
    }
}
```

#### Tier 1（高権限レベル）

| ロール名 | 主な権限範囲 | 適用対象 | 教育機関での活用 |
|----------|-------------|----------|------------------|
| **Exchange 管理者** | メール・予定表管理 | メール管理者 | 教務・事務メール管理 |
| **SharePoint 管理者** | ファイル共有・サイト管理 | 情報管理者 | 授業資料・研究データ管理 |
| **Teams 管理者** | Teams環境全体管理 | コミュニケーション管理者 | オンライン授業・会議管理 |
| **Intune 管理者** | デバイス・アプリ管理 | デバイス管理者 | 学習用デバイス管理 |

#### Tier 2（専門権限レベル）

| ロール名 | 主な権限範囲 | 適用対象 | 一般企業での活用 |
|----------|-------------|----------|------------------|
| **ユーザー管理者** | ユーザー・グループ管理 | 人事・総務担当者 | 従業員アカウント管理 |
| **ヘルプデスク管理者** | パスワードリセット・基本サポート | IT サポート担当 | 日常的なユーザーサポート |
| **ライセンス管理者** | ライセンス割り当て管理 | IT 管理者 | コスト最適化・使用状況管理 |
| **レポート閲覧者** | 使用状況レポート閲覧 | 管理職・分析担当 | 利用状況分析・意思決定支援 |

### ロール別権限マトリックス

```markdown
📊 業務機能別権限マトリックス

| 業務機能 | グローバル管理者 | ユーザー管理者 | Exchange管理者 | ヘルプデスク管理者 |
|----------|------------------|----------------|----------------|-------------------|
| ユーザー作成・削除 | ✅ | ✅ | ❌ | ❌ |
| パスワードリセット | ✅ | ✅ | ❌ | ✅ |
| ライセンス割り当て | ✅ | ✅ | ❌ | ❌ |
| メールボックス管理 | ✅ | ❌ | ✅ | ❌ |
| セキュリティ設定 | ✅ | ❌ | ❌ | ❌ |
| 監査ログ閲覧 | ✅ | ❌ | ❌ | ❌ |
```

---

## 🎯 最小権限の原則と実装

### 権限設計の基本原則

#### 1. Need-to-Know原則

**🎯 概念**: ユーザーは業務遂行に必要な最小限の情報・機能にのみアクセス可能とする

**実装例**:
```powershell
# 部署別アクセス制御の実装
function Set-DepartmentBasedAccess {
    param($UserId, $Department)
    
    # 部署別グループ割り当て
    $departmentGroups = @{
        "教務部" = @("Teachers-Group", "Curriculum-Access")
        "事務局" = @("Admin-Group", "Student-Records-Access")
        "IT部" = @("IT-Support-Group", "System-Access")
    }
    
    if ($departmentGroups.ContainsKey($Department)) {
        foreach ($groupName in $departmentGroups[$Department]) {
            $group = Get-MgGroup -Filter "displayName eq '$groupName'"
            if ($group) {
                New-MgGroupMember -GroupId $group.Id -DirectoryObjectId $UserId
                Write-Host "✅ $groupName に追加: $UserId" -ForegroundColor Green
            }
        }
    }
}
```

#### 2. Just-in-Time (JIT) アクセス

**🎯 概念**: 高権限は必要な時間のみ有効化し、作業完了後は自動的に無効化

**実装**: Privileged Identity Management (PIM) の活用

```powershell
# PIMによる一時的な権限昇格
function Request-TemporaryAdminAccess {
    param(
        $UserId,
        $RoleName = "User Administrator",
        $DurationHours = 2,
        $Justification = "緊急ユーザーサポート対応"
    )
    
    # PIMロール割り当て要求
    $roleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -Filter "displayName eq '$RoleName'"
    
    $requestBody = @{
        RoleDefinitionId = $roleDefinition.Id
        PrincipalId = $UserId
        RequestType = "SelfActivate"
        ScheduleInfo = @{
            StartDateTime = (Get-Date).ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
            Expiration = @{
                Type = "AfterDuration"
                Duration = "PT$($DurationHours)H"
            }
        }
        Justification = $Justification
    }
    
    try {
        $request = New-MgRoleManagementDirectoryRoleAssignmentScheduleRequest -BodyParameter $requestBody
        Write-Host "✅ PIM権限要求が送信されました: $($request.Id)" -ForegroundColor Green
        return $request
    } catch {
        Write-Error "PIM要求に失敗しました: $($_.Exception.Message)"
    }
}
```

#### 3. 権限の分離と監視

**🎯 概念**: 重要な操作は複数人による承認を要求し、全ての権限変更を監視

```powershell
# 権限変更の監視システム
function Monitor-PrivilegedRoleChanges {
    param([int]$DaysBack = 7)
    
    $startDate = (Get-Date).AddDays(-$DaysBack)
    
    # 管理者ロール変更の監査
    $roleChanges = Get-MgAuditLogDirectoryAudit -Filter "activityDateTime ge $($startDate.ToString('yyyy-MM-dd')) and category eq 'RoleManagement'"
    
    $privilegedChanges = $roleChanges | Where-Object {
        $_.ActivityDisplayName -match "Add member to role|Remove member from role"
    } | Select-Object ActivityDateTime, ActivityDisplayName, InitiatedBy, TargetResources
    
    if ($privilegedChanges) {
        Write-Host "⚠️ 過去$DaysBack日間の権限変更:" -ForegroundColor Yellow
        $privilegedChanges | Format-Table -AutoSize
        
        # 高権限ロールの変更を特に強調
        $criticalChanges = $privilegedChanges | Where-Object {
            $_.TargetResources.displayName -match "Global Administrator|Privileged"
        }
        
        if ($criticalChanges) {
            Write-Host "🚨 重要: 高権限ロールの変更が検出されました:" -ForegroundColor Red
            $criticalChanges | Format-Table -AutoSize
        }
    } else {
        Write-Host "✅ 権限変更はありませんでした" -ForegroundColor Green
    }
    
    return $privilegedChanges
}
```

---

## 🛠️ カスタムロールの設計と作成

### 組織固有のロール要件分析

教育機関や企業では、標準ロールでは対応できない固有の権限要件が存在します。

#### 教育機関向けカスタムロール例

```powershell
# 教育機関向け「学年主任」カスタムロール
function New-GradeLevelManagerRole {
    $roleName = "学年主任管理者"
    $roleDescription = "特定学年の学生アカウントとTeamsクラス管理権限"
    
    # 必要な権限の定義
    $permissions = @(
        "microsoft.directory/users/basic/update",           # 学生基本情報更新
        "microsoft.directory/groups/members/update",        # クラスグループ管理
        "microsoft.office365.skypeForBusiness/allEntities/allTasks", # Teams管理
        "microsoft.office365.serviceHealth/allEntities/allTasks"     # サービス状況確認
    )
    
    # カスタムロール作成
    $customRole = @{
        DisplayName = $roleName
        Description = $roleDescription
        IsEnabled = $true
        ResourceScopes = @("/")
        RolePermissions = @(
            @{
                AllowedResourceActions = $permissions
            }
        )
    }
    
    try {
        $newRole = New-MgRoleManagementDirectoryRoleDefinition -BodyParameter $customRole
        Write-Host "✅ カスタムロール作成完了: $roleName" -ForegroundColor Green
        return $newRole
    } catch {
        Write-Error "カスタムロール作成に失敗: $($_.Exception.Message)"
    }
}
```

#### 企業向けカスタムロール例

```powershell
# 企業向け「プロジェクトリーダー」カスタムロール
function New-ProjectLeaderRole {
    $roleName = "プロジェクトリーダー"
    $roleDescription = "プロジェクトメンバーの管理とリソースアクセス制御"
    
    $permissions = @(
        "microsoft.directory/groups/basic/update",          # プロジェクトグループ管理
        "microsoft.directory/groups/members/update",        # メンバー追加・削除
        "microsoft.office365.sharepoint/allEntities/allTasks", # SharePointサイト管理
        "microsoft.office365.exchange/allEntities/basic/read"  # メール配布リスト閲覧
    )
    
    $customRole = @{
        DisplayName = $roleName
        Description = $roleDescription
        IsEnabled = $true
        ResourceScopes = @("/")
        RolePermissions = @(
            @{
                AllowedResourceActions = $permissions
            }
        )
    }
    
    $newRole = New-MgRoleManagementDirectoryRoleDefinition -BodyParameter $customRole
    Write-Host "✅ カスタムロール作成完了: $roleName" -ForegroundColor Green
    return $newRole
}
```

### スコープ制限付きロール

**🎯 概念**: 管理権限を特定のグループやユーザーに限定し、影響範囲を制御

```powershell
# 部署限定管理者ロールの作成
function New-DepartmentScopedAdminRole {
    param(
        $DepartmentName,
        $AdminUserId
    )
    
    # 管理対象となる部署グループを取得
    $departmentGroup = Get-MgGroup -Filter "displayName eq '$DepartmentName'"
    
    if ($departmentGroup) {
        # 管理単位（Administrative Unit）の作成
        $adminUnit = @{
            DisplayName = "$DepartmentName 管理単位"
            Description = "$DepartmentName の限定管理範囲"
        }
        
        $newAdminUnit = New-MgDirectoryAdministrativeUnit -BodyParameter $adminUnit
        
        # 部署メンバーを管理単位に追加
        $departmentMembers = Get-MgGroupMember -GroupId $departmentGroup.Id
        foreach ($member in $departmentMembers) {
            New-MgDirectoryAdministrativeUnitMember -AdministrativeUnitId $newAdminUnit.Id -DirectoryObjectId $member.Id
        }
        
        # スコープ付きロール割り当て
        $userAdminRole = Get-MgDirectoryRole -Filter "displayName eq 'User Administrator'"
        $scopedRoleAssignment = @{
            RoleDefinitionId = $userAdminRole.Id
            PrincipalId = $AdminUserId
            DirectoryScopeId = "/administrativeUnits/$($newAdminUnit.Id)"
        }
        
        New-MgRoleManagementDirectoryRoleAssignment -BodyParameter $scopedRoleAssignment
        
        Write-Host "✅ 部署限定管理者を設定しました: $DepartmentName" -ForegroundColor Green
        return $newAdminUnit
    }
}
```

---

## 🔄 動的グループと権限の自動化

### 属性ベースの動的権限割り当て

ユーザーの属性（部署、役職、場所など）に基づいて、権限を自動的に付与・剥奪するシステムを構築します。

#### 動的グループによる権限管理

```powershell
# 属性ベースの動的セキュリティグループ作成
function New-DynamicSecurityGroup {
    param(
        $GroupName,
        $DynamicRule,
        $Description
    )
    
    $groupParams = @{
        DisplayName = $GroupName
        Description = $Description
        GroupTypes = @("DynamicMembership")
        SecurityEnabled = $true
        MailEnabled = $false
        MembershipRule = $DynamicRule
        MembershipRuleProcessingState = "On"
    }
    
    try {
        $newGroup = New-MgGroup -BodyParameter $groupParams
        Write-Host "✅ 動的グループ作成完了: $GroupName" -ForegroundColor Green
        
        # グループメンバーシップの確認（作成直後は空の場合があります）
        Start-Sleep -Seconds 5
        $memberCount = (Get-MgGroupMember -GroupId $newGroup.Id -All).Count
        Write-Host "📊 現在のメンバー数: $memberCount 名" -ForegroundColor Cyan
        
        return $newGroup
    } catch {
        Write-Error "動的グループ作成に失敗: $($_.Exception.Message)"
    }
}

# 教育機関向けの動的グループ例
function Create-EducationDynamicGroups {
    # 1. 教職員グループ（役職が教授、准教授、講師、事務職員の場合）
    $facultyRule = '(user.jobTitle -eq "教授") -or (user.jobTitle -eq "准教授") -or (user.jobTitle -eq "講師") -or (user.jobTitle -eq "事務職員")'
    New-DynamicSecurityGroup -GroupName "教職員" -DynamicRule $facultyRule -Description "全教職員を含む動的グループ"
    
    # 2. 学部別グループ（部署属性ベース）
    $departments = @("情報学部", "工学部", "経済学部", "文学部")
    foreach ($dept in $departments) {
        $deptRule = "user.department -eq `"$dept`""
        New-DynamicSecurityGroup -GroupName "$dept-全員" -DynamicRule $deptRule -Description "$dept の全メンバー"
    }
    
    # 3. 新入生グループ（作成日ベース）
    $currentYear = (Get-Date).Year
    $newStudentRule = "(user.jobTitle -eq `"学生`") -and (user.createdDateTime -ge `"$currentYear-04-01T00:00:00Z`")"
    New-DynamicSecurityGroup -GroupName "$currentYear年度新入生" -DynamicRule $newStudentRule -Description "$currentYear年度の新入生"
}

# 企業向けの動的グループ例
function Create-CorporateDynamicGroups {
    # 1. 管理職グループ
    $managerRule = '(user.jobTitle -contains "部長") -or (user.jobTitle -contains "課長") -or (user.jobTitle -contains "係長") -or (user.jobTitle -contains "主任")'
    New-DynamicSecurityGroup -GroupName "管理職" -DynamicRule $managerRule -Description "全管理職を含む動的グループ"
    
    # 2. リモートワーカーグループ（勤務地ベース）
    $remoteRule = '(user.physicalDeliveryOfficeName -eq "在宅") -or (user.physicalDeliveryOfficeName -eq "サテライト")'
    New-DynamicSecurityGroup -GroupName "リモートワーカー" -DynamicRule $remoteRule -Description "リモート勤務者向けグループ"
    
    # 3. 契約社員・派遣社員グループ
    $contractRule = '(user.employeeType -eq "契約社員") -or (user.employeeType -eq "派遣社員")'
    New-DynamicSecurityGroup -GroupName "非正規雇用者" -DynamicRule $contractRule -Description "契約・派遣社員グループ"
}
```

### 条件付きアクセスポリシーとの連携

動的グループと条件付きアクセスを組み合わせ、セキュリティと利便性を両立します。

```powershell
# 条件付きアクセスポリシーの作成（動的グループベース）
function New-ConditionalAccessPolicy {
    param(
        $PolicyName,
        $TargetGroupId,
        $Requirements
    )
    
    $policy = @{
        DisplayName = $PolicyName
        State = "enabled"
        Conditions = @{
            Users = @{
                IncludeGroups = @($TargetGroupId)
                ExcludeUsers = @()
            }
            Applications = @{
                IncludeApplications = @("All")
            }
            Locations = @{
                IncludeLocations = @("All")
                ExcludeLocations = @("AllTrusted")
            }
        }
        GrantControls = @{
            Operator = "OR"
            BuiltInControls = $Requirements
        }
    }
    
    try {
        $newPolicy = New-MgIdentityConditionalAccessPolicy -BodyParameter $policy
        Write-Host "✅ 条件付きアクセスポリシー作成完了: $PolicyName" -ForegroundColor Green
        return $newPolicy
    } catch {
        Write-Error "ポリシー作成に失敗: $($_.Exception.Message)"
    }
}

# 教育機関向けポリシー例
function Create-EducationConditionalAccessPolicies {
    # 学生グループの取得
    $studentsGroup = Get-MgGroup -Filter "displayName eq '学生'"
    
    if ($studentsGroup) {
        # 学生向け: 多要素認証必須
        New-ConditionalAccessPolicy -PolicyName "学生-MFA必須" -TargetGroupId $studentsGroup.Id -Requirements @("mfa")
        
        # 教職員向け: MFA + 準拠デバイス必須
        $facultyGroup = Get-MgGroup -Filter "displayName eq '教職員'"
        if ($facultyGroup) {
            New-ConditionalAccessPolicy -PolicyName "教職員-高セキュリティ" -TargetGroupId $facultyGroup.Id -Requirements @("mfa", "compliantDevice")
        }
    }
}
```

---

## 🔍 権限監査と継続的な改善

### 定期的な権限棚卸し

**🎯 目的**: 不要な権限の特定・削除とコンプライアンス要件の維持

```powershell
# 包括的な権限監査レポート生成
function Generate-ComprehensiveAccessAudit {
    param(
        [string]$OutputPath = "C:\Reports\AccessAudit_$(Get-Date -Format 'yyyyMMdd').xlsx"
    )
    
    Write-Host "🔍 権限監査レポート生成を開始します..." -ForegroundColor Cyan
    
    # 1. 管理者ロール割り当て状況
    $adminRoleReport = @()
    $adminRoles = Get-MgDirectoryRole | Where-Object {$_.DisplayName -match "Administrator|管理者"}
    
    foreach ($role in $adminRoles) {
        $members = Get-MgDirectoryRoleMember -DirectoryRoleId $role.Id
        foreach ($member in $members) {
            $user = Get-MgUser -UserId $member.Id -ErrorAction SilentlyContinue
            if ($user) {
                $lastSignIn = $user.SignInActivity.LastSignInDateTime
                $daysSinceLastSignIn = if ($lastSignIn) { 
                    (Get-Date) - [datetime]$lastSignIn | Select-Object -ExpandProperty TotalDays 
                } else { 
                    999 
                }
                
                $adminRoleReport += [PSCustomObject]@{
                    RoleName = $role.DisplayName
                    UserName = $user.DisplayName
                    UserPrincipalName = $user.UserPrincipalName
                    Department = $user.Department
                    JobTitle = $user.JobTitle
                    AccountEnabled = $user.AccountEnabled
                    LastSignIn = $lastSignIn
                    DaysSinceLastSignIn = [math]::Round($daysSinceLastSignIn)
                    RiskLevel = if ($daysSinceLastSignIn -gt 90) { "高" } elseif ($daysSinceLastSignIn -gt 30) { "中" } else { "低" }
                }
            }
        }
    }
    
    # 2. 高権限ユーザーの分析
    $highPrivilegedUsers = $adminRoleReport | Group-Object UserPrincipalName | Where-Object {$_.Count -gt 1}
    
    Write-Host "📊 監査結果サマリー:" -ForegroundColor Yellow
    Write-Host "  管理者ロール保有者: $($adminRoleReport.Count)名" -ForegroundColor White
    Write-Host "  複数ロール保有者: $($highPrivilegedUsers.Count)名" -ForegroundColor White
    
    $inactiveAdmins = $adminRoleReport | Where-Object {$_.DaysSinceLastSignIn -gt 90}
    if ($inactiveAdmins) {
        Write-Host "  ⚠️ 90日以上未サインイン管理者: $($inactiveAdmins.Count)名" -ForegroundColor Red
        $inactiveAdmins | ForEach-Object {
            Write-Host "    - $($_.UserPrincipalName) ($($_.RoleName))" -ForegroundColor Red
        }
    }
    
    # 3. ゲストユーザーの権限状況
    $guestUsers = Get-MgUser -Filter "userType eq 'Guest'" -All
    $guestWithRoles = @()
    
    foreach ($guest in $guestUsers) {
        $guestRoles = Get-MgUserMemberOf -UserId $guest.Id | Where-Object {$_.AdditionalProperties."@odata.type" -eq "#microsoft.graph.directoryRole"}
        if ($guestRoles) {
            $guestWithRoles += [PSCustomObject]@{
                GuestEmail = $guest.UserPrincipalName
                DisplayName = $guest.DisplayName
                Roles = ($guestRoles.AdditionalProperties.displayName -join "; ")
                CreatedDate = $guest.CreatedDateTime
                LastSignIn = $guest.SignInActivity.LastSignInDateTime
            }
        }
    }
    
    # 4. PIM対象ロールの状況（Premium P2ライセンス必要）
    try {
        $pimEligibleAssignments = Get-MgRoleManagementDirectoryRoleEligibilityScheduleInstance -All
        $pimActiveAssignments = Get-MgRoleManagementDirectoryRoleAssignmentScheduleInstance -All
        
        Write-Host "  PIM対象割り当て: $($pimEligibleAssignments.Count)件" -ForegroundColor Cyan
        Write-Host "  PIM現在アクティブ: $($pimActiveAssignments.Count)件" -ForegroundColor Cyan
    } catch {
        Write-Host "  PIM情報取得不可（Premium P2ライセンス必要）" -ForegroundColor Yellow
    }
    
    # レポート出力
    $adminRoleReport | Export-Csv "$OutputPath.csv" -Encoding UTF8 -NoTypeInformation
    Write-Host "✅ 権限監査レポートを出力しました: $OutputPath.csv" -ForegroundColor Green
    
    return @{
        AdminRoles = $adminRoleReport
        HighPrivilegedUsers = $highPrivilegedUsers
        InactiveAdmins = $inactiveAdmins
        GuestWithRoles = $guestWithRoles
    }
}
```

### 自動化された権限レビューシステム

```powershell
# 自動権限レビューシステム
function Start-AutomatedAccessReview {
    param(
        [int]$ReviewCycleDays = 90,  # レビューサイクル（日）
        [string]$ApproverEmail = "security-team@organization.com"
    )
    
    Write-Host "🔄 自動権限レビューシステムを開始します..." -ForegroundColor Cyan
    
    # 1. レビュー対象の特定
    $reviewTargets = @()
    
    # 高権限ロール保有者
    $criticalRoles = @("Global Administrator", "Privileged Role Administrator", "Security Administrator")
    foreach ($roleName in $criticalRoles) {
        $role = Get-MgDirectoryRole -Filter "displayName eq '$roleName'"
        if ($role) {
            $members = Get-MgDirectoryRoleMember -DirectoryRoleId $role.Id
            foreach ($member in $members) {
                $user = Get-MgUser -UserId $member.Id -ErrorAction SilentlyContinue
                if ($user) {
                    $reviewTargets += [PSCustomObject]@{
                        UserPrincipalName = $user.UserPrincipalName
                        DisplayName = $user.DisplayName
                        RoleName = $roleName
                        ReviewReason = "高権限ロール"
                        LastReviewed = $null  # 実装時に過去のレビュー日を参照
                        NextReviewDue = (Get-Date).AddDays($ReviewCycleDays)
                    }
                }
            }
        }
    }
    
    # 2. 長期間未使用のアカウント
    $allUsers = Get-MgUser -All | Where-Object {$_.AccountEnabled -eq $true}
    foreach ($user in $allUsers) {
        $lastSignIn = $user.SignInActivity.LastSignInDateTime
        if ($lastSignIn) {
            $daysSinceSignIn = (Get-Date) - [datetime]$lastSignIn
            if ($daysSinceSignIn.TotalDays -gt 60) {  # 60日以上未サインイン
                $userRoles = Get-MgUserMemberOf -UserId $user.Id | Where-Object {$_.AdditionalProperties."@odata.type" -eq "#microsoft.graph.directoryRole"}
                if ($userRoles) {
                    $reviewTargets += [PSCustomObject]@{
                        UserPrincipalName = $user.UserPrincipalName
                        DisplayName = $user.DisplayName
                        RoleName = ($userRoles.AdditionalProperties.displayName -join "; ")
                        ReviewReason = "長期間未サインイン ($([math]::Round($daysSinceSignIn.TotalDays))日)"
                        LastReviewed = $null
                        NextReviewDue = (Get-Date)  # 即座にレビュー必要
                    }
                }
            }
        }
    }
    
    # 3. レビュー通知の送信
    if ($reviewTargets) {
        $reviewReport = $reviewTargets | ConvertTo-Html -Title "権限レビュー対象ユーザー" -PreContent "<h2>定期権限レビューが必要なユーザー</h2>"
        
        # メール通知機能（実装時はSendGridやExchange Onlineを使用）
        Write-Host "📧 レビュー通知対象: $($reviewTargets.Count)件" -ForegroundColor Yellow
        Write-Host "   通知先: $ApproverEmail" -ForegroundColor Yellow
        
        # CSVレポート出力
        $reviewTargets | Export-Csv "C:\Reports\AccessReview_$(Get-Date -Format 'yyyyMMdd').csv" -Encoding UTF8 -NoTypeInformation
        Write-Host "✅ レビューレポートを出力しました" -ForegroundColor Green
    }
    
    return $reviewTargets
}
```

---

## 🛡️ Privileged Identity Management (PIM) の実装

### PIM基本設定と運用

**🎯 目的**: 高権限アクセスの Just-in-Time 提供とアクセス履歴の完全追跡

```powershell
# PIMロール設定の自動化
function Configure-PIMRole {
    param(
        $RoleName,
        $MaxActivationDuration = "PT8H",  # 最大8時間
        $RequireApproval = $true,
        $ApproverIds = @(),
        $RequireJustification = $true,
        $RequireMFA = $true
    )
    
    try {
        # ロール定義の取得
        $roleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -Filter "displayName eq '$RoleName'"
        
        if (-not $roleDefinition) {
            Write-Error "ロールが見つかりません: $RoleName"
            return
        }
        
        # PIMポリシー設定
        $policyRules = @(
            @{
                "@odata.type" = "#microsoft.graph.unifiedRoleManagementPolicyActivationRule"
                Id = "Activation_Admin_Approval"
                EnabledRule = $RequireApproval
                Setting = @{
                    ApprovalStages = @(
                        @{
                            ApprovalStageTimeOutInDays = 1
                            IsApproverJustificationRequired = $true
                            EscalationTimeInMinutes = 0
                            PrimaryApprovers = $ApproverIds | ForEach-Object {
                                @{
                                    "@odata.type" = "#microsoft.graph.singleUser"
                                    UserId = $_
                                }
                            }
                        }
                    )
                }
            },
            @{
                "@odata.type" = "#microsoft.graph.unifiedRoleManagementPolicyActivationRule"
                Id = "Activation_Max_Duration"
                EnabledRule = $true
                Setting = @{
                    MaximumDuration = $MaxActivationDuration
                }
            },
            @{
                "@odata.type" = "#microsoft.graph.unifiedRoleManagementPolicyActivationRule"
                Id = "Activation_MFA"
                EnabledRule = $RequireMFA
            }
        )
        
        Write-Host "✅ PIMロール設定完了: $RoleName" -ForegroundColor Green
        Write-Host "   最大有効時間: $MaxActivationDuration" -ForegroundColor Cyan
        Write-Host "   承認必須: $RequireApproval" -ForegroundColor Cyan
        Write-Host "   MFA必須: $RequireMFA" -ForegroundColor Cyan
        
    } catch {
        Write-Error "PIM設定に失敗しました: $($_.Exception.Message)"
    }
}

# 教育機関向けPIM設定例
function Setup-EducationPIMRoles {
    # IT管理者の承認者設定（複数承認者）
    $itManagerIds = @(
        "it-director@university.ac.jp",
        "security-manager@university.ac.jp"
    ) | ForEach-Object {
        (Get-MgUser -Filter "userPrincipalName eq '$_'").Id
    }
    
    # 重要ロールの設定
    Configure-PIMRole -RoleName "Global Administrator" -MaxActivationDuration "PT4H" -RequireApproval $true -ApproverIds $itManagerIds
    Configure-PIMRole -RoleName "Exchange Administrator" -MaxActivationDuration "PT8H" -RequireApproval $true -ApproverIds $itManagerIds
    Configure-PIMRole -RoleName "SharePoint Administrator" -MaxActivationDuration "PT8H" -RequireApproval $false  # 承認不要
    Configure-PIMRole -RoleName "Teams Administrator" -MaxActivationDuration "PT8H" -RequireApproval $false
}
```

### PIMアクセス要求の自動化

```powershell
# PIMロール昇格要求の簡易化
function Request-PIMElevation {
    param(
        $RoleName,
        $DurationHours = 2,
        $Justification,
        $TicketNumber = $null
    )
    
    # 現在のユーザーIDを取得
    $currentUser = Get-MgContext
    if (-not $currentUser.Account) {
        Write-Error "Microsoft Graphに接続してください"
        return
    }
    
    $userId = (Get-MgUser -Filter "userPrincipalName eq '$($currentUser.Account)'").Id
    
    # ロール定義の取得
    $roleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -Filter "displayName eq '$RoleName'"
    
    if (-not $roleDefinition) {
        Write-Error "ロールが見つかりません: $RoleName"
        return
    }
    
    # 完全な正当化理由の構築
    $fullJustification = $Justification
    if ($TicketNumber) {
        $fullJustification += " (チケット番号: $TicketNumber)"
    }
    $fullJustification += " - 要求日時: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')"
    
    # PIM要求の作成
    $requestBody = @{
        Action = "selfActivate"
        PrincipalId = $userId
        RoleDefinitionId = $roleDefinition.Id
        DirectoryScopeId = "/"
        Justification = $fullJustification
        ScheduleInfo = @{
            StartDateTime = (Get-Date).ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
            Expiration = @{
                Type = "AfterDuration"
                Duration = "PT$($DurationHours)H"
            }
        }
    }
    
    try {
        $request = New-MgRoleManagementDirectoryRoleAssignmentScheduleRequest -BodyParameter $requestBody
        
        Write-Host "✅ PIM昇格要求を送信しました" -ForegroundColor Green
        Write-Host "   要求ID: $($request.Id)" -ForegroundColor Cyan
        Write-Host "   ロール: $RoleName" -ForegroundColor Cyan
        Write-Host "   期間: $DurationHours 時間" -ForegroundColor Cyan
        Write-Host "   正当化理由: $fullJustification" -ForegroundColor Cyan
        
        # 承認待ちの場合の確認方法
        Write-Host "`n⏳ 要求状態の確認方法:" -ForegroundColor Yellow
        Write-Host "   Get-MgRoleManagementDirectoryRoleAssignmentScheduleRequest -UnifiedRoleAssignmentScheduleRequestId '$($request.Id)'" -ForegroundColor Gray
        
        return $request
        
    } catch {
        Write-Error "PIM要求に失敗しました: $($_.Exception.Message)"
    }
}

# 使用例関数
function Show-PIMUsageExamples {
    Write-Host "📋 PIM使用例:" -ForegroundColor Cyan
    Write-Host ""
    
    Write-Host "1. 緊急対応（2時間の一時的昇格）:" -ForegroundColor Yellow
    Write-Host "   Request-PIMElevation -RoleName 'User Administrator' -DurationHours 2 -Justification '緊急ユーザーサポート対応' -TicketNumber 'INC-2025-001'" -ForegroundColor Gray
    
    Write-Host "`n2. 定期メンテナンス（8時間の昇格）:" -ForegroundColor Yellow
    Write-Host "   Request-PIMElevation -RoleName 'Exchange Administrator' -DurationHours 8 -Justification '月次メンテナンス作業'"  -ForegroundColor Gray
    
    Write-Host "`n3. 権限確認（短時間での確認作業）:" -ForegroundColor Yellow
    Write-Host "   Request-PIMElevation -RoleName 'Security Reader' -DurationHours 1 -Justification 'セキュリティ監査確認作業'" -ForegroundColor Gray
}
```

---

## 🧪 実習演習: 権限管理シナリオ

### 演習1: 教育機関の新年度権限設定（30分）

**🎯 シナリオ**: 大学の新年度開始に伴う権限体系の再構築

```markdown
🎓 サンプル大学の新年度権限要件
- 新任教授 3名: 各学部の管理権限が必要
- 学年主任 6名: 担当学年の学生管理権限
- 新入職員 5名: 基本的な事務権限
- 卒業・退職者: 権限無効化が必要
```

#### 実習手順

```powershell
# 1. 新年度用カスタムロールの作成
function Setup-NewAcademicYearRoles {
    Write-Host "🎓 新年度権限設定を開始します..." -ForegroundColor Cyan
    
    # 学部管理者ロール
    $facultyAdminPermissions = @(
        "microsoft.directory/users/basic/update",
        "microsoft.directory/groups/members/update",
        "microsoft.office365.skypeForBusiness/allEntities/allTasks"
    )
    
    $facultyAdminRole = @{
        DisplayName = "学部管理者"
        Description = "学部内の学生・教職員管理権限"
        IsEnabled = $true
        ResourceScopes = @("/")
        RolePermissions = @(@{
            AllowedResourceActions = $facultyAdminPermissions
        })
    }
    
    New-MgRoleManagementDirectoryRoleDefinition -BodyParameter $facultyAdminRole
    Write-Host "✅ 学部管理者ロール作成完了" -ForegroundColor Green
    
    # 学年主任ロール
    $gradeLevelPermissions = @(
        "microsoft.directory/groups/members/update",
        "microsoft.office365.exchange/allEntities/basic/read"
    )
    
    $gradeLevelRole = @{
        DisplayName = "学年主任"
        Description = "担当学年の学生管理権限"
        IsEnabled = $true
        ResourceScopes = @("/")
        RolePermissions = @(@{
            AllowedResourceActions = $gradeLevelPermissions
        })
    }
    
    New-MgRoleManagementDirectoryRoleDefinition -BodyParameter $gradeLevelRole
    Write-Host "✅ 学年主任ロール作成完了" -ForegroundColor Green
}

# 2. 新任教授への権限付与
function Assign-NewFacultyRoles {
    $newProfessors = @(
        @{Name="新任教授A"; Email="prof-a@university.ac.jp"; Department="情報学部"},
        @{Name="新任教授B"; Email="prof-b@university.ac.jp"; Department="工学部"},
        @{Name="新任教授C"; Email="prof-c@university.ac.jp"; Department="経済学部"}
    )
    
    $facultyAdminRole = Get-MgDirectoryRole -Filter "displayName eq '学部管理者'"
    
    foreach ($professor in $newProfessors) {
        $user = Get-MgUser -Filter "userPrincipalName eq '$($professor.Email)'"
        if ($user) {
            # ロール割り当て
            New-MgDirectoryRoleMember -DirectoryRoleId $facultyAdminRole.Id -DirectoryObjectId $user.Id
            
            # 部署別グループ作成・追加
            $deptGroupName = "$($professor.Department)-管理者"
            $deptGroup = Get-MgGroup -Filter "displayName eq '$deptGroupName'"
            if (-not $deptGroup) {
                $newGroup = @{
                    DisplayName = $deptGroupName
                    Description = "$($professor.Department)の管理者グループ"
                    SecurityEnabled = $true
                    MailEnabled = $false
                }
                $deptGroup = New-MgGroup -BodyParameter $newGroup
            }
            
            New-MgGroupMember -GroupId $deptGroup.Id -DirectoryObjectId $user.Id
            Write-Host "✅ $($professor.Name) に学部管理権限を付与しました" -ForegroundColor Green
        }
    }
}

# 実習実行
Setup-NewAcademicYearRoles
Assign-NewFacultyRoles
```

### 演習2: セキュリティインシデント対応（20分）

**🎯 シナリオ**: 不審なアクティビティ検出時の緊急権限無効化

```powershell
# セキュリティインシデント対応自動化
function Handle-SecurityIncident {
    param(
        $SuspiciousUserEmail,
        $IncidentId,
        $Severity = "Medium"  # Low, Medium, High, Critical
    )
    
    Write-Host "🚨 セキュリティインシデント対応開始: $IncidentId" -ForegroundColor Red
    
    $user = Get-MgUser -Filter "userPrincipalName eq '$SuspiciousUserEmail'"
    if (-not $user) {
        Write-Error "ユーザーが見つかりません: $SuspiciousUserEmail"
        return
    }
    
    # 重要度に応じた対応レベル
    switch ($Severity) {
        "Critical" {
            # 即座にアカウント無効化
            Update-MgUser -UserId $user.Id -AccountEnabled $false
            Write-Host "🔒 アカウントを無効化しました: $SuspiciousUserEmail" -ForegroundColor Red
            
            # 全てのセッションを強制終了
            Revoke-MgUserSignInSession -UserId $user.Id
            Write-Host "🔄 全セッションを終了しました" -ForegroundColor Yellow
            
            # 管理者権限の一時停止
            $userRoles = Get-MgUserMemberOf -UserId $user.Id | Where-Object {$_.AdditionalProperties."@odata.type" -eq "#microsoft.graph.directoryRole"}
            foreach ($role in $userRoles) {
                Remove-MgDirectoryRoleMember -DirectoryRoleId $role.Id -DirectoryObjectId $user.Id
                Write-Host "⚠️ 管理者権限を削除: $($role.AdditionalProperties.displayName)" -ForegroundColor Yellow
            }
        }
        
        "High" {
            # MFA強制リセット
            Reset-MgUserAuthenticationMethod -UserId $user.Id
            Write-Host "🔐 MFA設定をリセットしました" -ForegroundColor Orange
            
            # 一時的な権限制限
            $restrictedGroup = Get-MgGroup -Filter "displayName eq '制限ユーザー'"
            if ($restrictedGroup) {
                New-MgGroupMember -GroupId $restrictedGroup.Id -DirectoryObjectId $user.Id
                Write-Host "⚠️ 制限グループに追加しました" -ForegroundColor Orange
            }
        }
        
        "Medium" {
            # パスワード強制変更
            Update-MgUser -UserId $user.Id -PasswordProfile @{
                ForceChangePasswordNextSignIn = $true
                Password = New-RandomPassword
            }
            Write-Host "🔑 パスワードリセットを設定しました" -ForegroundColor Yellow
        }
    }
    
    # インシデントログ記録
    $incidentLog = @{
        Timestamp = Get-Date
        IncidentId = $IncidentId
        UserEmail = $SuspiciousUserEmail
        Severity = $Severity
        ActionsPerformed = "AccountDisabled, SessionRevoked, RolesRemoved"
        Operator = (Get-MgContext).Account
    }
    
    $incidentLog | ConvertTo-Json | Add-Content "C:\SecurityLogs\incident_$IncidentId.json"
    Write-Host "📝 インシデントログを記録しました" -ForegroundColor Cyan
}

# パスワード生成ヘルパー関数
function New-RandomPassword {
    $chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*"
    return (-join ((1..12) | ForEach-Object { $chars[(Get-Random -Maximum $chars.Length)] }))
}

# 使用例
# Handle-SecurityIncident -SuspiciousUserEmail "suspicious.user@company.com" -IncidentId "INC-2025-001" -Severity "Critical"
```

### 演習3: 権限最適化プロジェクト（40分）

**🎯 シナリオ**: 組織全体の権限見直しとコスト最適化

```powershell
# 権限最適化分析システム
function Analyze-PermissionOptimization {
    Write-Host "🔍 権限最適化分析を開始します..." -ForegroundColor Cyan
    
    # 1. 過剰権限の検出
    $overPrivilegedUsers = @()
    $allUsers = Get-MgUser -All
    
    foreach ($user in $allUsers) {
        $userRoles = Get-MgUserMemberOf -UserId $user.Id | Where-Object {$_.AdditionalProperties."@odata.type" -eq "#microsoft.graph.directoryRole"}
        
        if ($userRoles.Count -gt 2) {  # 3個以上のロール = 要確認
            $lastSignIn = $user.SignInActivity.LastSignInDateTime
            $overPrivilegedUsers += [PSCustomObject]@{
                UserPrincipalName = $user.UserPrincipalName
                DisplayName = $user.DisplayName
                Department = $user.Department
                JobTitle = $user.JobTitle
                RoleCount = $userRoles.Count
                Roles = ($userRoles.AdditionalProperties.displayName -join "; ")
                LastSignIn = $lastSignIn
                OptimizationPriority = if (-not $lastSignIn -or ((Get-Date) - [datetime]$lastSignIn).TotalDays -gt 90) { "高" } else { "中" }
            }
        }
    }
    
    # 2. 未使用ライセンスの検出
    $licenses = Get-MgSubscribedSku
    $unusedLicenses = @()
    
    foreach ($license in $licenses) {
        $availableUnits = $license.PrepaidUnits.Enabled - $license.ConsumedUnits
        if ($availableUnits -gt 0) {
            $costPerMonth = Get-LicenseCost -SkuId $license.SkuId  # 架空の関数
            $unusedLicenses += [PSCustomObject]@{
                SkuPartNumber = $license.SkuPartNumber
                UnusedUnits = $availableUnits
                EstimatedMonthlySavings = $availableUnits * $costPerMonth
            }
        }
    }
    
    # 3. 推奨アクション
    Write-Host "`n📊 最適化分析結果:" -ForegroundColor Green
    Write-Host "   過剰権限ユーザー: $($overPrivilegedUsers.Count)名" -ForegroundColor Yellow
    
    if ($overPrivilegedUsers) {
        Write-Host "`n⚠️ 権限見直し推奨ユーザー:" -ForegroundColor Yellow
        $overPrivilegedUsers | Where-Object {$_.OptimizationPriority -eq "高"} | ForEach-Object {
            Write-Host "   - $($_.UserPrincipalName): $($_.RoleCount)個のロール" -ForegroundColor Red
        }
    }
    
    # 4. 最適化計画の生成
    $optimizationPlan = @{
        AnalysisDate = Get-Date
        OverPrivilegedUsers = $overPrivilegedUsers
        UnusedLicenses = $unusedLicenses
        RecommendedActions = @(
            "高優先度ユーザーの権限レビュー実施",
            "90日以上未サインインユーザーの権限無効化",
            "未使用ライセンスの削減検討",
            "動的グループによる権限自動化"
        )
        EstimatedSavings = ($unusedLicenses | Measure-Object EstimatedMonthlySavings -Sum).Sum
    }
    
    # レポート出力
    $optimizationPlan | ConvertTo-Json -Depth 3 | Out-File "C:\Reports\PermissionOptimization_$(Get-Date -Format 'yyyyMMdd').json"
    $overPrivilegedUsers | Export-Csv "C:\Reports\OverPrivilegedUsers_$(Get-Date -Format 'yyyyMMdd').csv" -Encoding UTF8 -NoTypeInformation
    
    Write-Host "✅ 最適化分析完了 - レポートを出力しました" -ForegroundColor Green
    return $optimizationPlan
}

# 架空のライセンスコスト計算関数
function Get-LicenseCost {
    param($SkuId)
    # 実際の実装では Azure Cost Management API または 価格表を参照
    $costMap = @{
        "STANDARDPACK" = 500      # Office 365 E1: 500円/月
        "ENTERPRISEPACK" = 1200   # Office 365 E3: 1200円/月
        "SPE_E5" = 3000          # Microsoft 365 E5: 3000円/月
    }
    return $costMap[$SkuId] ?? 0
}

# 実習実行
$optimizationResults = Analyze-PermissionOptimization
```

---

## 🛠️ トラブルシューティング

### よくある権限問題と解決方法

#### 問題1: 権限割り当てエラー

**症状**: "Insufficient privileges to complete the operation" エラー

**原因と解決**:

```powershell
# 権限不足の診断と解決
function Diagnose-PermissionIssue {
    param($UserEmail, $TargetRole)
    
    Write-Host "🔍 権限問題の診断中..." -ForegroundColor Cyan
    
    # 1. 現在のユーザー権限確認
    $currentUser = Get-MgUser -Filter "userPrincipalName eq '$UserEmail'"
    if (-not $currentUser) {
        Write-Error "ユーザーが見つかりません: $UserEmail"
        return
    }
    
    $currentRoles = Get-MgUserMemberOf -UserId $currentUser.Id | Where-Object {$_.AdditionalProperties."@odata.type" -eq "#microsoft.graph.directoryRole"}
    
    Write-Host "📋 現在の権限:" -ForegroundColor Yellow
    $currentRoles.AdditionalProperties.displayName | ForEach-Object {
        Write-Host "   - $_" -ForegroundColor Gray
    }
    
    # 2. 必要権限の確認
    $roleRequirements = @{
        "User Administrator" = @("Privileged Role Administrator", "Global Administrator")
        "Exchange Administrator" = @("Global Administrator")
        "Security Administrator" = @("Privileged Role Administrator", "Global Administrator")
    }
    
    if ($roleRequirements.ContainsKey($TargetRole)) {
        $requiredRoles = $roleRequirements[$TargetRole]
        $hasRequiredRole = $false
        
        foreach ($requiredRole in $requiredRoles) {
            if ($currentRoles.AdditionalProperties.displayName -contains $requiredRole) {
                $hasRequiredRole = $true
                break
            }
        }
        
        if (-not $hasRequiredRole) {
            Write-Host "❌ 権限不足: $TargetRole の割り当てには以下の権限が必要です:" -ForegroundColor Red
            $requiredRoles | ForEach-Object {
                Write-Host "   - $_" -ForegroundColor Red
            }
        } else {
            Write-Host "✅ 権限は充分です" -ForegroundColor Green
        }
    }
    
    # 3. PIMによる一時昇格の確認
    try {
        $pimEligible = Get-MgRoleManagementDirectoryRoleEligibilityScheduleInstance -Filter "principalId eq '$($currentUser.Id)'"
        if ($pimEligible) {
            Write-Host "⏳ PIMで利用可能な権限:" -ForegroundColor Cyan
            $pimEligible | ForEach-Object {
                $roleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -UnifiedRoleDefinitionId $_.RoleDefinitionId
                Write-Host "   - $($roleDefinition.DisplayName)" -ForegroundColor Cyan
            }
        }
    } catch {
        Write-Host "ℹ️ PIM情報の取得に失敗（Premium P2ライセンス必要）" -ForegroundColor Yellow
    }
}

# 使用例
# Diagnose-PermissionIssue -UserEmail "admin@company.com" -TargetRole "User Administrator"
```

#### 問題2: 条件付きアクセスによるブロック

**症状**: "Access blocked by conditional access policy" エラー

**解決手順**:

```powershell
# 条件付きアクセスのトラブルシューティング
function Troubleshoot-ConditionalAccess {
    param($UserEmail, $ApplicationName = "Microsoft 365")
    
    $user = Get-MgUser -Filter "userPrincipalName eq '$UserEmail'"
    
    # 1. 適用されているポリシーの確認
    $caPolicy = Get-MgIdentityConditionalAccessPolicy -All
    $applicablePolicies = @()
    
    foreach ($policy in $caPolicy) {
        # ユーザーが対象に含まれているかチェック
        $userGroups = Get-MgUserMemberOf -UserId $user.Id
        $userGroupIds = $userGroups.Id
        
        $isTargeted = $false
        if ($policy.Conditions.Users.IncludeUsers -contains $user.Id -or 
            $policy.Conditions.Users.IncludeUsers -contains "All") {
            $isTargeted = $true
        }
        
        if ($policy.Conditions.Users.IncludeGroups) {
            foreach ($groupId in $policy.Conditions.Users.IncludeGroups) {
                if ($userGroupIds -contains $groupId) {
                    $isTargeted = $true
                    break
                }
            }
        }
        
        if ($isTargeted) {
            $applicablePolicies += [PSCustomObject]@{
                PolicyName = $policy.DisplayName
                State = $policy.State
                GrantControls = ($policy.GrantControls.BuiltInControls -join ", ")
                SessionControls = if ($policy.SessionControls) { "設定あり" } else { "なし" }
            }
        }
    }
    
    Write-Host "📋 適用される条件付きアクセスポリシー:" -ForegroundColor Yellow
    $applicablePolicies | Format-Table -AutoSize
    
    # 2. 解決案の提示
    Write-Host "`n💡 解決案:" -ForegroundColor Green
    foreach ($policy in $applicablePolicies | Where-Object {$_.State -eq "enabled"}) {
        Write-Host "   ポリシー: $($policy.PolicyName)" -ForegroundColor Cyan
        switch -Regex ($policy.GrantControls) {
            "mfa" { 
                Write-Host "     → MFA設定を完了してください" -ForegroundColor Gray 
            }
            "compliantDevice" { 
                Write-Host "     → デバイスをIntune準拠状態にしてください" -ForegroundColor Gray 
            }
            "domainJoinedDevice" { 
                Write-Host "     → ドメイン参加デバイスを使用してください" -ForegroundColor Gray 
            }
            "approvedApplication" { 
                Write-Host "     → 承認されたアプリケーションを使用してください" -ForegroundColor Gray 
            }
            default { 
                Write-Host "     → 管理者に確認してください: $($policy.GrantControls)" -ForegroundColor Gray 
            }
        }
    }
}
```

#### 問題3: PIM昇格が承認されない

**症状**: PIM要求が長時間ペンディング状態

**解決手順**:

```powershell
# PIM承認状況の確認と催促
function Check-PIMApprovalStatus {
    param($RequestId)
    
    try {
        # 要求の詳細確認
        $request = Get-MgRoleManagementDirectoryRoleAssignmentScheduleRequest -UnifiedRoleAssignmentScheduleRequestId $RequestId
        
        Write-Host "📋 PIM要求状況:" -ForegroundColor Cyan
        Write-Host "   要求ID: $($request.Id)" -ForegroundColor Gray
        Write-Host "   状態: $($request.Status)" -ForegroundColor Gray
        Write-Host "   作成日時: $($request.CreatedDateTime)" -ForegroundColor Gray
        Write-Host "   正当化理由: $($request.Justification)" -ForegroundColor Gray
        
        # ロール詳細の取得
        $roleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -UnifiedRoleDefinitionId $request.RoleDefinitionId
        Write-Host "   対象ロール: $($roleDefinition.DisplayName)" -ForegroundColor Gray
        
        # 承認者の確認
        $rolePolicy = Get-MgRoleManagementDirectoryRoleAssignmentPolicy -UnifiedRoleAssignmentPolicyId $request.RoleDefinitionId
        if ($rolePolicy.Rules) {
            $approvalRule = $rolePolicy.Rules | Where-Object { $_."@odata.type" -match "approval" }
            if ($approvalRule) {
                Write-Host "`n👥 承認者情報:" -ForegroundColor Yellow
                # 承認者の詳細表示（実装依存）
            }
        }
        
        # 状態別のアドバイス
        switch ($request.Status) {
            "PendingApproval" {
                Write-Host "`n⏳ 承認待ち状態です" -ForegroundColor Yellow
                Write-Host "   承認者に直接連絡することを検討してください" -ForegroundColor Gray
            }
            "Approved" {
                Write-Host "`n✅ 承認済みです - ロールが有効化されているか確認してください" -ForegroundColor Green
            }
            "Denied" {
                Write-Host "`n❌ 拒否されました - 正当化理由を見直して再申請してください" -ForegroundColor Red
            }
            "TimedOut" {
                Write-Host "`n⏰ タイムアウトしました - 新しい要求を作成してください" -ForegroundColor Red
            }
        }
        
    } catch {
        Write-Error "PIM要求の確認に失敗しました: $($_.Exception.Message)"
    }
}

# PIM要求の一覧表示
function Show-MyPIMRequests {
    $currentUserId = (Get-MgUser -Filter "userPrincipalName eq '$((Get-MgContext).Account)'").Id
    
    $myRequests = Get-MgRoleManagementDirectoryRoleAssignmentScheduleRequest -Filter "principalId eq '$currentUserId'" | 
                  Sort-Object CreatedDateTime -Descending |
                  Select-Object -First 10
    
    Write-Host "📋 最近のPIM要求（上位10件）:" -ForegroundColor Cyan
    $myRequests | ForEach-Object {
        $roleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -UnifiedRoleDefinitionId $_.RoleDefinitionId
        Write-Host "   $($_.CreatedDateTime): $($roleDefinition.DisplayName) - $($_.Status)" -ForegroundColor Gray
    }
}
```

---

## ✅ 学習成果の確認

### 習得スキルチェック

この節を完了したら、以下の項目について実際に操作できることを確認してください：

- [ ] **ロール体系の理解**: 組織要件に適した管理者ロールの選択・設計
- [ ] **最小権限の実装**: 業務に必要な最小限の権限付与
- [ ] **カスタムロール作成**: 標準ロールでは対応できない要件への対応
- [ ] **PIM導入・運用**: Just-in-Time アクセスの実装と管理
- [ ] **権限監査**: 定期的な棚卸しと最適化の実施
- [ ] **インシデント対応**: セキュリティ問題発生時の迅速な権限制御
- [ ] **トラブルシューティング**: 権限関連問題の診断・解決

### 実践課題: 包括的権限管理システム構築

実際の組織を想定した総合的な課題に取り組んでください：

```markdown
🎯 総合課題: 多拠点教育機関の権限管理統合

## 背景
3つのキャンパスを持つ大学が Microsoft 365 を統合導入。
各キャンパスの独自運用から統一的な権限管理体制への移行が必要。

## 要件
- 本部: 全体統制、ポリシー策定
- 各キャンパス: 独自の教務・学務管理
- 学部横断: 研究プロジェクト管理
- セキュリティ: 統一基準、監査対応

## 実施内容
1. 3層権限体系の設計（本部・キャンパス・学部）
2. 動的グループによる自動権限管理
3. PIM導入による高権限管理
4. 包括的監査システムの構築
5. 運用手順書・緊急時対応計画の策定

## 成果物
- 権限設計書（PowerShellスクリプト含む）
- 運用手順書（日常・緊急時）
- 監査レポートテンプレート
- ユーザー向けガイダンス資料
```

---

## 🚀 次のステップ

この節を完了したら、以下の学習に進むことを推奨します：

### 推奨学習パス

<!-- 1. **[第5章: セキュリティとコンプライアンス](../05-security-compliance/)** -->
- 高度なセキュリティ機能との連携
- コンプライアンス要件への対応

<!-- 2. **[第6章: デバイス管理](../06-device-management/)** -->
- Intune による包括的なデバイス制御
- 条件付きアクセスとの統合

<!-- 3. **[第8章: 運用監視とトラブルシューティング](../08-monitoring-troubleshooting/)** -->
- 継続的な権限監視システム
- 自動化された問題検出・対応

### 発展学習

- **Microsoft Entra ID Governance**: より高度なアイデンティティガバナンス
- **Azure Sentinel**: セキュリティ情報イベント管理（SIEM）
- **Microsoft Purview**: データガバナンスとコンプライアンス
- **Zero Trust Architecture**: ゼロトラストセキュリティの実装

---

## 📚 参考リソース

### Microsoft公式ドキュメント

- [Microsoft Entra ID の組み込みロール](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/permissions-reference)
- [Privileged Identity Management](https://docs.microsoft.com/ja-jp/azure/active-directory/privileged-identity-management/)
- [条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/conditional-access/)
- [Microsoft Graph PowerShell SDK](https://docs.microsoft.com/en-us/powershell/microsoftgraph/)

### セキュリティベストプラクティス

- [Microsoft セキュリティベストプラクティス](https://docs.microsoft.com/ja-jp/security/)
- [Azure AD セキュリティ運用ガイド](https://docs.microsoft.com/ja-jp/azure/active-directory/fundamentals/security-operations-introduction)
- [特権アクセス セキュリティロードマップ](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/security-planning)

### 実用ツール・テンプレート

<!-- - [権限管理チェックリスト](../../appendices/checklists/role-management-checklist.md) -->
<!-- - [PowerShell権限管理スクリプト集](../../appendices/powershell-commands.md#権限管理) -->
<!-- - [権限設計テンプレート](../../appendices/templates/role-design-template.xlsx) -->

---

*📅 最終更新: 2025年6月2日*  
*✍️ 更新者: Microsoft 365 ハンドブック編集チーム*  
*📖 対象バージョン: Microsoft 365 (2025年6月版) 