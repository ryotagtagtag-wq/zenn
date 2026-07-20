---
title: "opencodeはClaudeコードのオープンソース版だった｜導入から実践まで"
emoji: "🚀"
type: "tech"
topics: ["opencode", "AI", "CLI", "開発環境"]
published: true
---

こんにちは、game_ryoです。

つい先日、**opencode-ai** というツールを試してみたんですが、これが思いの外いい。Claude Codeのオープンソース版というわけではなく、むしろ「複数のAIモデルに対応した、ターミナルネイティブなコード生成・実行環境」という方が正確です。

本記事では、導入から実際の使用までを、スクリーンショットを交えて解説します。

---

## opencodeとは何か

まず一言で言うと：**ターミナルから直接、AIにコード生成を指示し、その場で実行・反復できるツール**です。

### 特徴

- **複数のAIモデル対応**: デフォルトのFreeモデルのほか、Claude、DeepSeek、Gemini等と連携可能
- **CLI-first**: GUIに頼らず、ターミナルで完結
- **迅速なフィードバックループ**: 生成 → 実行 → 修正が同一環境で完結
- **設定ファイルサポート**: プロジェクト単位での設定をサポート

エンジニアにとっては、「ChatGPTのWebUIを開く手間」がゼロになるというだけで、体験が大きく変わります。

---

## インストール

### 必須環境

- **Node.js**: v18以上推奨（v20以上あるとなお良し）
- **npm**: v9以上

```bash
node --version  # 確認用
npm --version   # 確認用
```

### インストール手順

全体で30秒程度です。

```bash
npm install -g opencode-ai
```

インストール完了後、バージョン確認：

```bash
opencode --version
```

出力例：`opencode-ai/1.x.x`

---

## 初期設定

### 1. 最初の起動

```bash
opencode
```

実行するとこのような画面が現れます：

![opencodeの初期起動画面](/images/Screenshot_2026-07-20_07.56.03.png)

### 2. モデルの選択

初回起動時、モデル選択画面が表示されます。

**おすすめは、デフォルトで用意されている `Free` モデル** です。レート制限はありますが、個人開発や学習用途なら十分実用的です。

#### Freeモデルの仕様

| 項目 | 内容 |
|------|------|
| **レート制限** | 1時間あたり10回程度 |
| **コンテキスト窓** | 約8,000トークン |
| **応答時間** | 平均3～5秒 |
| **コスト** | 無料 |

### 3. 他のAIモデルと連携する場合

```bash
opencode auth login
```

このコマンドで、以下のモデル群と連携できます：

- **Claude** (Anthropic) → APIキー必須
- **DeepSeek** → APIキー必須
- **Gemini** (Google) → APIキー必須
- **Grok** (xAI) → APIキー必須

各プロバイダーのダッシュボードでAPIキーを取得し、プロンプトに従って入力すればOKです。

---

## 実際に使ってみる

### 基本的な使用フロー

#### ケース1: シンプルなスクリプト生成

```bash
opencode "Pythonで、今年の日付をYYYY-MM-DD形式で出力するスクリプトを書いて"
```

実行すると、opencodeは以下を自動で行います：

1. AIに指示を送信
2. コードを生成
3. ファイルを作成
4. 自動実行
5. 結果を表示

出力例：

```
✓ Created: date_script.py
✓ Executed successfully
2026-07-20
```

#### ケース2: 対話的なコード改善

生成後、「もっと高速に」「エラーハンドリングを追加」といった指示で、同じセッション内で反復できます。

```bash
opencode "上のスクリプトにタイムゾーン対応を加えて。JST表記で"
```

---

## より実践的な設定

### プロジェクト単位での設定（`.opencoderc`）

プロジェクトルートに `.opencoderc` ファイルを作成することで、プロジェクト固有の設定が可能です：

```json
{
  "model": "claude",
  "language": "typescript",
  "autoExecute": true,
  "timeout": 30,
  "outputDir": "./generated"
}
```

| 設定項目 | 説明 |
|---------|------|
| `model` | 使用するAIモデル（free / claude / deepseek等） |
| `language` | デフォルト言語（python / javascript / typescript / rust等） |
| `autoExecute` | コード生成後、自動実行するか（true/false） |
| `timeout` | 実行タイムアウト（秒） |
| `outputDir` | 生成ファイルの出力ディレクトリ |

### 環境変数の活用

APIキーは、環境変数でも管理できます（セキュリティベストプラクティス）：

```bash
export OPENCODE_CLAUDE_KEY="sk-ant-xxxxx"
export OPENCODE_DEEPSEEK_KEY="sk-xxxxx"
```

その後、`opencode auth login` をスキップしても自動認識されます。

---

## トラブルシューティング

### Q1: インストール後、`opencode` コマンドが見つからない

**原因**: npm globalのパスが環境変数に含まれていない

**解決法**:

```bash
# npm globalディレクトリを確認
npm config get prefix

# 出力例: /home/user/.npm-global
# ~/.bashrcまたは~/.zshrcに以下を追記
export PATH="$PATH:/home/user/.npm-global/bin"

# 反映させる
source ~/.bashrc  # or ~/.zshrc
```

### Q2: `auth login` が失敗する

**原因**: APIキーが正しくない、またはプロバイダーが一時的に利用不可

**対処法**:

1. APIキーが**コピーペーストミス**していないか確認
2. 対象プロバイダーのダッシュボードでキーが**有効化**されているか確認
3. ネットワーク接続を確認（VPN/プロキシがある場合は設定確認）

### Q3: Freeモデルのレート制限に引っかかった

**症状**: `Error: Rate limit exceeded`

**解決法**:

- 1時間待つか、有料モデルに切り替える
- 開発中は `.opencoderc` で `model: "claude"` 等に変更

### Q4: 生成されたコードが実行されない

**確認項目**:

```bash
# 1. ファイル権限を確認
ls -la generated_file.py

# 2. 実行権限を付与
chmod +x generated_file.py

# 3. 依存関係があれば、手動インストール
pip install -r requirements.txt  # Python の場合
```

---

## 実用的なユースケース

### 1. **ワンショットのユーティリティスクリプト**

```bash
opencode "CSVファイルをJSONに変換するスクリプト（Pythonで）。stdin対応で"
```

### 2. **テンプレート生成**

```bash
opencode "React 18 + Tailwindで、シンプルな Todo アプリのスターターを作成"
```

### 3. **データ処理パイプライン**

```bash
opencode "
JSONログファイルを読み込んで、
- 1時間単位でグループ化
- 各グループで応答時間の平均・中央値・95パーセンタイルを計算
- 結果をCSVで出力
するNode.jsスクリプトを書いて
"
```

---

## ベストプラクティス

### 指示の書き方

opencodeを最大限活用するには、**プロンプトの質**がすべてです。

#### ❌ 曖昧な指示

```
"Webアプリを作成"
```

#### ✅ 明確な指示

```
"React 18 + Vite + TypeScript で、
入力フィールド + 送信ボタン + 結果表示のシンプルなサーチアプリを作成。
- Tailwind CSS で基本的なスタイリング
- フォーム送信時に、入力値を大文字に変換して表示
- エラーハンドリング付き
"
```

### セッション管理

長時間の開発では、セッションを分割すると良いです：

```bash
# セッション1: コア機能
opencode "ユーザー認証機能を実装"

# セッション2: UI
opencode "上で作ったAuthコンポーネントをReactでラップ"

# セッション3: テスト
opencode "認証ロジックのユニットテストを Jest で作成"
```

---

## 次のステップ

opencodeの基本は、今この瞬間でマスターできました。ここからの活用は、あなたの想像力次第です。

次回のテーマ案：
- **opencodeでの Web API ラッピング**
- **複数言語混在プロジェクトの構築**
- **CI/CDパイプラインとの統合**

---

## まとめ

| ポイント | 内容 |
|---------|------|
| **導入** | `npm install -g opencode-ai` で瞬時 |
| **セットアップ** | `opencode` で初回設定、Freeモデルがおすすめ |
| **使い方** | 明確なプロンプト → AI生成 → 自動実行 |
| **拡張性** | `.opencoderc` で細かい制御が可能 |
| **実用性** | ワンショットスクリプトから、複雑なプロジェクト生成まで対応 |

ターミナルでの開発体験を一段階上げたいなら、opencodeは**相当な投資対効果**を持ったツールです。

では、皆さんが楽しく、そして生産的にコードを書けることを願っています。

game_ryo
