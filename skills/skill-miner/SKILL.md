---
name: skill-miner
description: "skill-miner CLIの使い方。Claude Code会話履歴からスキルを自動抽出・デプロイするRustクレート。プラグイン化済み。SessionEndで自動実行。スキルマイナー、skill-miner、スキル抽出、マイニングと言われた時に使用。"
---

# skill-miner

## 場所・実行

- リポジトリ: `~/skill-miner/` (Rust crate)
- 依存: `cli-ai-analyzer`（AI呼び出し基盤）
- 実行: `cd ~/skill-miner && cargo run -- mine --max-days N [--dry-run]`

## サブコマンド

| コマンド | 用途 |
|----------|------|
| `mine --max-days N` | 直近N日の会話からスキル抽出・デプロイ |
| `mine --max-days N --dry-run` | 抽出のみ、ファイル書き込みなし |

## パイプライン

1. **会話収集**: `~/.claude/projects/` のJSONLファイルをスキャン（12h刻みのウィンドウ）
2. **ドメイン分類**: `domains.toml` の12ドメインにAI分類（写真管理、Rust開発、ツール設計等）
3. **パターン抽出**: ドメインごとにAIが知識パターンを抽出（`prompts/extract.txt`）
4. **スキル生成**: パターンをスキルMDファイルに変換
5. **重複チェック**: 既存スキル（`~/.claude/skills/`）と名前・スラグ・部分一致で照合
6. **デプロイ**: `~/.claude/skills/` に書き出し（NEWまたはUPDATE）

## 抽出基準（2026-02-27改修）

- **回数ベースではない**: 「そのトピックについてユーザーとの対話があったか」が基準
- ユーザーがAIを訂正した、判断基準を教えた、複数ターン往復した → 抽出対象
- 1会話でも対話があれば抽出する（MIN_CONVERSATIONS=1）
- 汎用的な開発手順（cargo build, git操作等）は除外

## 主要ファイル

| ファイル | 役割 |
|----------|------|
| `prompts/extract.txt` | パターン抽出プロンプト（抽出基準を定義） |
| `src/extractor.rs` | パターン抽出ロジック |
| `src/generator.rs` | スキルMD生成・既存スキル重複チェック |
| `src/scorer.rs` | パターンスコアリング（40%頻度 + 60%発火率） |
| `domains.toml` | ドメインマスタ（名前・キーワード） |

## 自動マイニング（プラグイン）

- SessionEnd フックにより、毎セッション終了時に `mine --max-days 1 --dry-run` が自動実行される
- ログ: `~/.claude/skill-miner-plugin/last-run.log`
- dry-run なので自動ではデプロイしない（確認用）

## 手動操作

- `cd ~/skill-miner && cargo run -- mine --max-days 3` で直近3日分を抽出＆デプロイ
- `--dry-run` で確認してから本番実行を推奨
