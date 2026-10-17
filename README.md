# LiteLLM Proxy for Gemini

このプロジェクトは、[LiteLLM](https://github.com/BerriAI/litellm) を使用して、Google Gemini の各種モデルへのリクエストを中継するプロキシサーバーのDocker環境です。

特定のモデル名（例えばClaude系のモデル名など）に対するリクエストを、対応するGeminiモデルへ自動的にルーティングするように構成されています。これにより、Claude CodeやCodex CLIから、別のモデル名としてGeminiを利用することが可能になります。

## 特徴

- **Gemini モデルへのルーティング**: 以下の最新Geminiモデルへのプロキシを提供します。
  - Gemini 4 Argon (`gemini-4-argon`) -> `gemini/gemini-4-argon`
  - Gemini 3.8 Flash (`gemini-3.8-flash`) -> `gemini/gemini-3.8-flash`
  - Gemini 3.5 Flash Lite (`gemini-3.5-flash-lite`) -> `gemini/gemini-3.5-flash-lite`

- **Claude エイリアス機能**: ツール側でClaudeのモデル名として指定されたリクエストを、対応するクラスのGeminiモデルへ転送します。
  - `claude-opus-*` → `gemini-4-argon`
  - `claude-sonnet-*` → `gemini-3.8-flash`
  - `claude-haiku-*` → `gemini-3.5-flash-lite`

## 必要要件

- Docker および Docker Compose
- Gemini API Key ([Google AI Studio](https://aistudio.google.com/) から取得できます)

## セットアップ手順

1. リポジトリをクローンまたはダウンロードし、ディレクトリに移動します。

2. `.env.example` をコピーして `.env` ファイルを作成します。

   ```bash
   cp .env.example .env
   ```

   作成した `.env` ファイルを開き、ご自身の Gemini APIキーを設定してください。

   ```env
   GEMINI_API_KEY=your_gemini_api_key_here
   ```

3. Docker Compose を使用してプロキシサーバーを起動します。

   ```bash
   docker compose up -d
   ```

4. LiteLLMプロキシサーバーが `http://localhost:4000` で起動します。

## 使い方

本プロジェクトには、CLIツールである **Claude Code** と **Codex CLI** 向けの設定ファイルがあらかじめ用意されています。
これらの設定（`.claude/settings.json`、 `.codex/config.toml`、および `.codex/model-catalog.local.json`）は、グローバル設定ディレクトリ（`~/.claude/` や `~/.codex/`）にそのまま配置（置き換え）して使用することを想定しています。もしすでに各ツールの既存の設定ファイルが存在する場合は、本プロジェクトの設定内容を既存のものにマージ（追記）してご利用ください。

### Claude Code での利用
`.claude/settings.json` に設定が用意されています。Claude Code を実行すると自動的にこの設定が読み込まれ、環境変数 `ANTHROPIC_BASE_URL` がローカルの `http://localhost:4000` に向きます。
これにより、Claude Code からのリクエストがLiteLLMプロキシを経由し、Geminiモデルで処理されます。
（APIキーにはダミーの値 `sk-ant-dummy` が設定されています。）

### Codex CLI での利用
`.codex/config.toml` に設定が用意されています。
モデルプロバイダーとして `litellm` を定義し、ローカルの `http://localhost:4000` を参照する設定が含まれています。
また、カスタムモデル（Geminiの各モデル名など）をCodex CLIに正しく認識させるため、ローカルのモデルカタログファイルを参照する設定（`model_catalog_json = "~/.codex/model-catalog.local.json"`）が追加されています。
デフォルトのモデルとして `gemini-3.8-flash` が指定されています。必要に応じてモデル名を変更してご利用ください。


## 設定のカスタマイズ

モデルの追加やエイリアスの変更を行いたい場合は、`litellm_config.yaml` を編集してください。
編集後は、設定を反映させるためにコンテナを再起動してください。

```bash
docker compose restart
```
