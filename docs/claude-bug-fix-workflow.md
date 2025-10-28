# Claude Bug Fix ワークフロー

Issueに`bug`ラベルが付けられた際、Claudeが自動的にバグを修正してPull Requestを作成するGitHub Actionsワークフローです。

## 📋 目次

1. [概要](#概要)
2. [機能](#機能)
3. [使い方](#使い方)
4. [PR先ブランチの変更方法](#pr先ブランチの変更方法)
5. [必要な設定](#必要な設定)
6. [トラブルシューティング](#トラブルシューティング)

---

## 概要

このワークフローは、GitHubのIssueに`bug`ラベルが付けられた際に自動的にトリガーされ、Claudeがバグの内容を理解して修正を行い、Pull Requestを作成します。

**ワークフローファイル:** `.github/workflows/claude-bug-fix.yml`

---

## 機能

### ✨ 主な機能

1. **自動バグ修正**
   - Issueの内容を読み取り、バグの原因を特定
   - 適切な修正を自動的に実施
   - 修正内容を新しいブランチにコミット

2. **Pull Request自動作成**
   - `fix/issue-{番号}` という名前でブランチを作成
   - 修正内容を含むPRを自動作成
   - PR説明文にIssueへのリンクを含める（自動クローズ対応）

3. **Issue通知**
   - 修正完了時にIssueにコメント追加
   - エラー発生時もIssueに通知

4. **柔軟なブランチ設定**
   - PR先のブランチを後から変更可能
   - デフォルトは`main`ブランチ

---

## 使い方

### 🤖 自動実行（推奨）

1. **Issueを作成**
   - 通常通りIssueを作成します
   - バグの内容を詳しく記載してください

2. **`bug`ラベルを付ける**
   - Issueに`bug`ラベルを追加
   - または、Issue作成時に`bug`ラベルを選択

3. **自動実行**
   - ワークフローが自動的に起動します
   - Claudeがバグを修正してPRを作成します

4. **PRを確認**
   - 作成されたPull Requestを確認
   - 必要に応じてレビュー・マージ

### 🔧 手動実行

既存のIssueに対して手動でワークフローを実行することもできます。

1. **Actionsタブに移動**
   - リポジトリの「Actions」タブをクリック

2. **ワークフローを選択**
   - 左側のワークフロー一覧から「Claude Bug Fix」を選択

3. **Run workflowをクリック**
   - 右上の「Run workflow」ボタンをクリック

4. **パラメータを入力**
   - **Issue number**: 修正したいIssueの番号を入力
   - **Base branch**: PR先のブランチ（省略時は`main`）

5. **実行**
   - 「Run workflow」ボタンをクリックして実行

---

## PR先ブランチの変更方法

### デフォルトブランチの変更

ワークフローファイルの`default`値を変更します。

**ファイル:** `.github/workflows/claude-bug-fix.yml`

```yaml
workflow_dispatch:
  inputs:
    base_branch:
      description: 'Base branch for PR (default: main)'
      required: false
      type: string
      default: 'main'  # ← ここを変更（例: 'develop'）
```

### 手動実行時にブランチを指定

手動実行時、Run workflowの画面で`Base branch`欄に任意のブランチ名を入力できます。

**例:**
- `develop` - developブランチにPRを作成
- `staging` - stagingブランチにPRを作成
- `feature/xxx` - 特定のfeatureブランチにPRを作成

---

## 必要な設定

### 1. Claude Code OAuth Token

このワークフローを使用するには、各ユーザーのOAuth Tokenが必要です。

詳細な設定方法は[Claude GitHub Actions セットアップガイド](./claude-github-actions-setup.md)を参照してください。

**必要なSecret名:**
```
{GitHubユーザー名}_CLAUDE_CODE_OAUTH_TOKEN
```

### 2. `bug`ラベルの作成

リポジトリに`bug`ラベルが存在することを確認してください。

#### ラベルの確認・作成方法

1. リポジトリの「Issues」タブをクリック
2. 「Labels」をクリック
3. `bug`ラベルが存在しない場合:
   - 「New label」をクリック
   - Label name: `bug`
   - 色とdescriptionは任意
   - 「Create label」をクリック

### 3. ワークフローの権限

ワークフローには以下の権限が自動的に付与されます：
- `contents: write` - コード修正とコミット
- `pull-requests: write` - PR作成
- `issues: read` - Issue内容の読み取り

> 📝 **注意**: リポジトリの設定で「Actions」の権限が適切に設定されている必要があります。

---

## トラブルシューティング

### ワークフローが実行されない

#### 原因1: `bug`ラベルが付いていない
- **確認**: Issueに`bug`ラベルが付いているか確認
- **対処**: `bug`ラベルを追加する

#### 原因2: ワークフローファイルがmainブランチにない
- **確認**: `.github/workflows/claude-bug-fix.yml`がmainブランチに存在するか
- **対処**: ファイルをmainブランチにプッシュする

### Claudeが修正に失敗する

#### 原因: Issue内容が不明確
- **対処**: Issueに以下の情報を追加
  - バグの再現手順
  - 期待される動作
  - 実際の動作
  - エラーメッセージ（あれば）
  - 環境情報

### PRが作成されない

#### 原因1: 権限不足
- **確認**: リポジトリの「Settings」→「Actions」→「General」
- **対処**: 「Workflow permissions」で「Read and write permissions」を選択

#### 原因2: OAuth Tokenが設定されていない
- **確認**: リポジトリSecretsに`{ユーザー名}_CLAUDE_CODE_OAUTH_TOKEN`が存在するか
- **対処**: [セットアップガイド](./claude-github-actions-setup.md)を参照してトークンを設定

### ブランチ作成エラー

#### 原因: 同名のブランチが既に存在
- **対処**:
  1. 既存の`fix/issue-{番号}`ブランチを削除
  2. または、手動実行で異なるbase branchを指定

---

## 実行例

### 自動実行の流れ

1. **Issue作成**
   ```
   Title: ログイン時にエラーが発生する
   Labels: bug

   Body:
   ログインボタンをクリックすると、以下のエラーが表示されます：
   "Cannot read property 'token' of undefined"

   再現手順：
   1. ログイン画面にアクセス
   2. メールアドレスとパスワードを入力
   3. ログインボタンをクリック
   ```

2. **ワークフロー自動実行**
   - `bug`ラベルが付いたことをトリガーに起動
   - ブランチ `fix/issue-123` を作成
   - Claudeがバグを修正
   - PRを自動作成

3. **Issue通知**
   ```
   ✅ Claudeによるバグ修正が完了しました！

   ブランチ: fix/issue-123
   ベースブランチ: main

   作成されたPull Requestを確認してください。
   ```

4. **PR確認・マージ**
   - 作成されたPRをレビュー
   - テストを確認
   - マージしてバグ修正完了

---

## ワークフローファイルの完全なコード

`.github/workflows/claude-bug-fix.yml`の完全なコードサンプルです。

```yaml
name: Claude Bug Fix

on:
  issues:
    types: [opened, edited, labeled]
  workflow_dispatch:
    inputs:
      issue_number:
        description: 'Issue number to fix'
        required: true
        type: number
      base_branch:
        description: 'Base branch for PR (default: master)'
        required: false
        type: string
        default: 'master'

jobs:
  auto-fix-bug:
    # bugラベルが付いているissueの場合のみ実行
    if: contains(github.event.issue.labels.*.name, 'bug') || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    permissions:
      contents: write        # コードの修正とコミットに必要
      pull-requests: write   # PR作成に必要
      issues: write          # issueの読み取りとコメント追加に必要
      id-token: write       # OAuth認証に必要

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # PR作成のため全履歴を取得

      - name: Get issue information
        id: get_issue
        uses: actions/github-script@v7
        with:
          script: |
            const issue_number = ${{ github.event.issue.number || inputs.issue_number }};

            // Issueの詳細を取得
            let issue;
            try {
              const { data } = await github.rest.issues.get({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issue_number
              });
              issue = data;
            } catch (error) {
              core.setFailed(`❌ Issue #${issue_number} の情報を取得できませんでした: ${error.message}`);
              core.exportVariable('ERROR_MESSAGE', `Issue #${issue_number} が見つかりません`);
              return;
            }

            // Issue本文のバリデーション
            const issueBody = issue.body?.trim();

            if (!issueBody || issueBody.length === 0) {
              core.setFailed(`❌ Issue #${issue.number} の本文が空です。バグの詳細を記載してください。`);
              core.exportVariable('ERROR_MESSAGE', 'Issue本文が空です。バグの詳細（再現手順、エラーメッセージなど）を記載してください。');
              core.exportVariable('ISSUE_NUMBER', issue.number);
              return;
            }

            if (issueBody.length < 20) {
              core.setFailed(`❌ Issue #${issue.number} の本文が短すぎます（${issueBody.length}文字）。バグの詳細を記載してください。`);
              core.exportVariable('ERROR_MESSAGE', `Issue本文が短すぎます（${issueBody.length}文字）。バグ修正には以下の情報が必要です：\n- バグの再現手順\n- 期待される動作\n- 実際の動作\n- エラーメッセージ（あれば）`);
              core.exportVariable('ISSUE_NUMBER', issue.number);
              return;
            }

            // Issue情報をMarkdownファイルとして保存
            const fs = require('fs');
            const issueContent = '# Issue #' + issue.number + ': ' + issue.title + '\n\n' +
              '**作成者:** @' + issue.user.login + '\n' +
              '**状態:** ' + issue.state + '\n' +
              '**ラベル:** ' + issue.labels.map(l => l.name).join(', ') + '\n\n' +
              '## 説明\n\n' +
              issueBody + '\n';

            fs.writeFileSync('ISSUE_CONTEXT.md', issueContent);

            // 環境変数に保存
            core.exportVariable('ISSUE_NUMBER', issue.number);
            core.exportVariable('ISSUE_TITLE', issue.title);
            core.exportVariable('ISSUE_VALID', 'true');

      - name: Create fix branch
        if: env.ISSUE_VALID == 'true'
        run: |
          BASE_BRANCH="${{ inputs.base_branch || 'master' }}"
          ISSUE_NUMBER="${{ env.ISSUE_NUMBER }}"
          FIX_BRANCH="fix/issue-${ISSUE_NUMBER}"

          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

          # ベースブランチから新しいブランチを作成
          git fetch origin "${BASE_BRANCH}"
          git checkout -b "${FIX_BRANCH}" "origin/${BASE_BRANCH}"
          git push -u origin "${FIX_BRANCH}"

          # 環境変数に保存
          echo "FIX_BRANCH=${FIX_BRANCH}" >> $GITHUB_ENV
          echo "BASE_BRANCH=${BASE_BRANCH}" >> $GITHUB_ENV

      - name: Run Claude Code to fix bug
        if: env.ISSUE_VALID == 'true'
        id: claude
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets[format('{0}_CLAUDE_CODE_OAUTH_TOKEN', github.actor)] }}

          prompt: |
            プロジェクトルートに配置されている **ISSUE_CONTEXT.md** ファイルを読んで、
            報告されたバグの内容を理解してから、修正作業を開始してください。

            **タスク:**
            1. ISSUE_CONTEXT.md を読んで、Issue の内容を確認する
            2. バグの原因を特定する（既存のソースコードを分析）
            3. 適切な修正を行う
            4. 修正内容を現在のブランチ (${{ env.FIX_BRANCH }}) にコミットする
            5. ${{ env.BASE_BRANCH }} ブランチに対するPull Requestを作成する

            **PR作成時の要件:**
            - 全て日本語で記載すること
            - PR タイトル: "Fix: [Issue #${{ env.ISSUE_NUMBER }}] ${{ env.ISSUE_TITLE }}"
            - PR 説明文に以下を含める:
              - バグの原因と修正内容の説明
              - 修正したファイルの概要
              - テスト方法（もしあれば）
              - `Close #${{ env.ISSUE_NUMBER }}` を記載（issueを自動クローズするため）

            **重要な制約:**
            - ISSUE_CONTEXT.md ファイルは削除すること（コミットしない）
            - コミットメッセージは適切に付けること

            修正が完了したら、作成したPRのURLを教えてください。

          claude_args: |
            --max-turns 30
            --permission-mode bypassPermissions
            --model claude-sonnet-4-5-20250929

      - name: Cleanup
        if: always()
        run: |
          # ISSUE_CONTEXT.mdを削除（念のため）
          rm -f ISSUE_CONTEXT.md

      - name: Comment on issue
        if: always() && env.ISSUE_NUMBER
        uses: actions/github-script@v7
        with:
          script: |
            const issue_number = ${{ env.ISSUE_NUMBER }};
            const error_message = process.env.ERROR_MESSAGE;
            const get_issue_status = '${{ steps.get_issue.outcome }}';
            const claude_status = '${{ steps.claude.outcome }}';

            let comment_body;

            // Issue情報取得で失敗した場合
            if (error_message) {
              comment_body = '⚠️ **自動バグ修正を開始できませんでした**\n\n' +
                '**理由:**\n' + error_message + '\n\n' +
                'Issue内容を追記してから、再度`bug`ラベルを付け直すか、手動でワークフローを実行してください。\n\n' +
                '[Actionsログを確認](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }})';
            }
            // Claude実行が成功した場合
            else if (claude_status === 'success') {
              comment_body = '✅ **Claudeによるバグ修正が完了しました！**\n\n' +
                '**ブランチ:** `${{ env.FIX_BRANCH }}`\n' +
                '**ベースブランチ:** `${{ env.BASE_BRANCH }}`\n\n' +
                '作成されたPull Requestを確認してください。';
            }
            // Claude実行が失敗した場合
            else if (claude_status === 'failure') {
              comment_body = '❌ **バグ修正中にエラーが発生しました**\n\n' +
                'Claudeによる修正処理中に問題が発生しました。\n' +
                '[Actionsログを確認](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }})して、エラーの詳細を確認してください。';
            }
            // その他のエラー
            else {
              comment_body = '⚠️ **ワークフロー実行中に問題が発生しました**\n\n' +
                '[Actionsログを確認](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }})してください。';
            }

            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: issue_number,
              body: comment_body
            });
```

---

## 関連ドキュメント

- [Claude GitHub Actions セットアップガイド](./claude-github-actions-setup.md)
- [Claude Code 公式ドキュメント](https://docs.claude.com/en/docs/claude-code/)
- [GitHub Actions ドキュメント](https://docs.github.com/en/actions)
