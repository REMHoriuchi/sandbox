# Claude Code GitHub Actions セットアップガイド

このガイドでは、Claude CodeをGitHub Actionsで使用するための設定方法を説明します。

## 📋 目次

1. [前提条件](#前提条件)
2. [初期設定（リポジトリ管理者向け）](#初期設定リポジトリ管理者向け)
3. [概要](#概要)
4. [トークンの発行方法](#トークンの発行方法)
5. [GitHubアカウント名の確認方法](#githubアカウント名の確認方法)
6. [GitHubリポジトリのSecretsの設定方法](#githubリポジトリのsecretsの設定方法)
7. [動作確認](#動作確認)

---

## 前提条件

このガイドを進める前に、以下が必要です：

- **Claude Code**: ローカルにインストールされていること
  - インストール方法は[公式ドキュメント](https://docs.claude.com/en/docs/claude-code/)を参照
- **GitHubアカウント**: 有効なGitHubアカウントを持っていること
- **リポジトリへのアクセス権限**:
  - 初期設定: リポジトリの管理者権限が必要
  - 個人のトークン設定: リポジトリへの書き込み権限があれば可能

---

## 初期設定（リポジトリ管理者向け）

> 📌 **このセクションは、リポジトリに初めてClaude Code GitHub Actionsを設定する管理者向けです。**
> 既に設定済みの場合は、[トークンの発行方法](#トークンの発行方法)に進んでください。

### 0. 初期設定が完了しているリポジトリかの確認方法
- [ ] Claude GitHub Appがリポジトリにインストールされている
- [ ] `.github/workflows/claude.yml`ファイルが存在する
- [ ] ワークフローファイルで`${{ secrets[format('{0}_CLAUDE_CODE_OAUTH_TOKEN', github.actor)] }}`を使用している
- [ ] 自動生成された`CLAUDE_CODE_OAUTH_TOKEN` Secretが削除されている
- [ ] 変更がmainブランチにプッシュされている

**GitHub Appの確認：**
- リポジトリの「Settings」→「Integrations」→「GitHub Apps」に「Claude」が表示されていれば初期設定済み

### 1. `/install-github-app` コマンドを実行

ターミナルで対象のリポジトリのディレクトリに移動し、Claude Codeを起動します：

```bash
cd /path/to/your/repository
claude
```

Claude Codeの対話モード内で以下のコマンドを実行します：

```
/install-github-app
```

表示される指示に従って進んでください。

このコマンドは以下の処理を自動的に行います：
- Claude GitHub Appのインストール
- 必要な権限の設定（Contents、Issues、Pull Requests）
- GitHub Actionsワークフローファイルの生成
- リポジトリSecretsへの`CLAUDE_CODE_OAUTH_TOKEN`の追加

### 2. GitHub Appのインストール確認

1. GitHubで対象のリポジトリにアクセス
2. 「**Settings**」→「**Integrations**」→「**GitHub Apps**」を確認
3. 「**Claude**」アプリがインストールされていることを確認

### 3. ワークフローファイルの修正

`/install-github-app`コマンドで自動生成されたワークフローファイルを、複数ユーザー対応に修正します。

#### ファイルの場所

```
.github/workflows/claude.yml
```

#### 修正箇所

ファイル内の`claude_code_oauth_token`の行を以下のように修正します：

**変更前：**
```yaml
- name: Run Claude Code
  uses: anthropics/claude-code-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

**変更後：**
```yaml
- name: Run Claude Code
  uses: anthropics/claude-code-action@v1
  with:
    claude_code_oauth_token: ${{ secrets[format('{0}_CLAUDE_CODE_OAUTH_TOKEN', github.actor)] }}
```

この変更により、ワークフローは各GitHubユーザー名に応じたSecretを参照するようになります。

### 4. 自動生成されたSecretの削除

`/install-github-app`コマンドで自動作成された`CLAUDE_CODE_OAUTH_TOKEN`を削除します。

#### 削除手順

1. GitHubで対象のリポジトリにアクセス
2. 「**Settings**」→「**Security**」→「**Secrets and variables**」→「**Actions**」を開く
3. Secrets一覧から「**CLAUDE_CODE_OAUTH_TOKEN**」を探す
4. 右側の「**Remove**」または「**Delete**」ボタンをクリック
5. 確認ダイアログで削除を確定

> 💡 **なぜ削除？**: 複数ユーザー対応のため、各ユーザーが個別に`{ユーザー名}_CLAUDE_CODE_OAUTH_TOKEN`形式でトークンを設定する必要があります。

### 5. 初期設定完了


---

## 概要

各ユーザーごとに個別のClaude Code OAuth Tokenを使用します。
ワークフローは以下の命名規則で各ユーザーのトークンを参照します：

```
{GitHubユーザー名}_CLAUDE_CODE_OAUTH_TOKEN
```

**例：**
- ユーザー名が `HOGE` の場合: `HOGE_CLAUDE_CODE_OAUTH_TOKEN`
- ユーザー名が `alice` の場合: `alice_CLAUDE_CODE_OAUTH_TOKEN`

---

## トークンの発行方法

### 1. Claude Codeを起動

ターミナルで以下のコマンドを実行してClaude Codeを起動します：

```bash
claude setup-token
```

### 2. 認証フローに従う

コマンドを実行すると、認証フローが開始されます。
画面の指示に従って、ブラウザで認証を完了してください。

### 3. トークンを取得

認証が完了すると、OAuth Tokenが表示されます。
このトークンをコピーして、安全な場所に一時的に保存してください。

> ⚠️ **重要**: このトークンは一度しか表示されません。必ずコピーして保存してください。

---

## GitHubアカウント名の確認方法

自分のGitHubアカウント名（ユーザー名）を確認する方法です。

### 方法1: GitHubプロフィールページから確認

1. [GitHub](https://github.com) にログインします
2. 右上のプロフィールアイコンをクリック
3. ドロップダウンメニューの「Your profile」をクリック
4. URLの `https://github.com/ユーザー名` の部分があなたのユーザー名です

**例：**
- URL が `https://github.com/Hoge` の場合、ユーザー名は `Hoge`

### 方法2: GitHubの設定から確認

1. [GitHub](https://github.com) にログインします
2. 右上のプロフィールアイコンをクリック
3. 「Settings」をクリック
4. 左側のメニューで「Public profile」を選択
5. ページ上部の「Username」の欄に表示されているのがあなたのユーザー名です

---

## GitHubリポジトリのSecretsの設定方法

### 1. リポジトリのSettings画面にアクセス

1. GitHubで対象のリポジトリにアクセスします
2. リポジトリページ上部の「**Settings**」タブをクリック

> ⚠️ **注意**: Settings タブが表示されない場合、あなたはリポジトリの管理者権限を持っていません。リポジトリのオーナーに権限付与を依頼してください。

### 2. Secrets and variables の設定画面に移動

1. 左側のサイドバーで「**Security**」セクションを探します
2. 「**Secrets and variables**」をクリックして展開
3. 「**Actions**」をクリック

### 3. 新しいSecretを追加

1. 「**New repository secret**」ボタンをクリック
2. 以下の情報を入力します：

   **Name（名前）:**
   ```
   {あなたのGitHubユーザー名}_CLAUDE_CODE_OAUTH_TOKEN
   ```

   **例：**
   - ユーザー名が `Hoge` の場合: `Hoge_CLAUDE_CODE_OAUTH_TOKEN`
   - ユーザー名が `alice` の場合: `alice_CLAUDE_CODE_OAUTH_TOKEN`

   **Secret（値）:**
   ```
   先ほどコピーしたOAuth Token
   ```

3. 「**Add secret**」ボタンをクリックして保存

### 4. 設定完了の確認

Secrets一覧画面に、あなたが設定したSecretの名前が表示されていれば設定完了です。

> 📝 **メモ**: Secretの値は設定後に確認することはできません。間違えて入力した場合は、Secretを削除して再度作成してください。

---

## 動作確認

### 1. IssueまたはPull Requestでテスト

1. 任意のIssueまたはPull Requestを開きます
2. コメント欄に `@claude` とメンションを含むコメントを投稿します

   **例：**
   ```
   @claude こんにちは！
   ```

3. GitHub Actionsが起動し、Claudeがコメントに返信すれば設定成功です

### 2. トラブルシューティング

#### Claudeが反応しない場合

- **Secretの名前を確認**: `{ユーザー名}_CLAUDE_CODE_OAUTH_TOKEN` の形式になっているか
- **ユーザー名の大文字小文字**: GitHubのユーザー名と完全に一致しているか（大文字小文字を区別）
- **トークンの有効性**: トークンが正しくコピーされているか、期限切れではないか
- **権限の確認**: Claude GitHub Appがリポジトリにインストールされているか

#### GitHub Actionsのログを確認

1. リポジトリの「**Actions**」タブをクリック
2. 失敗したワークフローをクリック
3. ログを確認してエラーメッセージを確認

---

## 補足情報

### トークンの更新

トークンが期限切れになった場合や、新しいトークンが必要な場合は、同じ手順で再度 `claude setup-token` コマンドを実行してください。

### セキュリティ上の注意

- **トークンは絶対にコードにコミットしないでください**
- GitHub Secretsを使用して安全に管理してください
- トークンが漏洩した場合は、すぐに新しいトークンを発行し、古いトークンを無効化してください

---

## 参考リンク

- [Claude Code 公式ドキュメント](https://docs.claude.com/en/docs/claude-code/)
- [GitHub Actions ドキュメント](https://docs.github.com/en/actions)
- [GitHub Secrets の管理](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)
