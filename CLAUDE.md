# CLAUDE.md

このファイルは、AIエージェント（Claude、Copilot、Gemini等）がこのリポジトリで作業する際のガイドラインを提供します。

## プロジェクト概要

このリポジトリは、**Terraform 1.14開発環境用のDockerコンテナ**を提供します。VS Code Dev Container対応で、Fish shellを搭載したDebianベースのコンテナに、Terraform、tflint、trivyがインストールされています。

## ディレクトリ構造

```
.
├── .devcontainer/
│   └── devcontainer.json    # VS Code Dev Container設定
├── .github/
│   └── workflows/
│       └── terraform-1.14.yaml  # GitHub Actions CI/CDワークフロー
├── docker/
│   └── Dockerfile           # コンテナイメージ定義
├── LICENSE
├── Makefile                 # ローカルビルド用コマンド
├── README.md                # プロジェクトドキュメント
└── CLAUDE.md                # このファイル
```

## 重要なファイル

| ファイル | 説明 |
|----------|------|
| `docker/Dockerfile` | Terraform 1.14コンテナのビルド定義。ベースイメージは`ghcr.io/bearfield/debian-fish:bookworm` |
| `.devcontainer/devcontainer.json` | VS Code Dev Container設定。拡張機能、マウント、環境変数を定義 |
| `.github/workflows/terraform-1.14.yaml` | GitHub Actions定義。ARM64/AMD64のマルチプラットフォームビルドを実行 |
| `Makefile` | ローカルでのビルド・テスト用コマンド |

## ビルドコマンド

```bash
# 全アーキテクチャテスト（ARM64 + AMD64）
make test

# 個別アーキテクチャテスト
make test.arm64
make test.amd64

# ビルドのみ（クリーンアップなし）
make test.build.arm64
make test.build.amd64

# 手動クリーンアップ
make test.rmi.arm64
make test.rmi.amd64
```

## 技術スタック

- **ベースイメージ**: `ghcr.io/bearfield/debian-fish:bookworm`
- **OS**: Debian 12 (Bookworm)
- **シェル**: Fish shell
- **主要ツール**（Dockerfile）:
  - Terraform 1.14.*
  - tflint（最新版）
  - trivy（最新版）
- **DevContainer追加機能**:
  - Docker CLI（docker-outside-of-docker feature経由）

## CI/CD

GitHub Actionsを使用した自動ビルド:

- **トリガー**:
  - mainブランチへのプッシュ時（`docker/`または`.github/workflows/`変更時）
  - 毎日19:37 UTC（日本時間04:37）
  - 手動実行（workflow_dispatch）

- **ビルドプロセス**:
  1. `build`ジョブ: ARM64とAMD64を並列ビルドし、各プラットフォーム用イメージをプッシュ
  2. `push-manifest`ジョブ: 両プラットフォームのイメージを統合したマルチプラットフォームマニフェストを作成

- **出力イメージ**:
  - `ghcr.io/bearfield/terraform:1.14` - マルチプラットフォームマニフェスト（push-manifestジョブで作成）
  - `ghcr.io/bearfield/terraform:1.14-arm64` - ARM64専用
  - `ghcr.io/bearfield/terraform:1.14-amd64` - AMD64専用

## 変更時の注意点

### Terraformバージョン更新

`docker/Dockerfile`の`TERRAFORM_VERSION`引数を変更:

```dockerfile
ARG TERRAFORM_VERSION=1.14.*
```

### ワークフロー変更

`.github/workflows/terraform-1.14.yaml`を編集する際:

- `env.IMAGE_TAG`でイメージタグを管理
- matrixで`linux/arm64`と`linux/amd64`の並列ビルドを実行
- `CR_PAT`シークレットでGitHub Container Registryに認証

### ベースイメージ更新

ベースイメージを変更する場合は`docker/Dockerfile`の`FROM`行を編集:

```dockerfile
FROM ghcr.io/bearfield/debian-fish:bookworm
```

## 関連リポジトリ

- [dot-devcontainer-debian-fish-bookworm](https://github.com/bearfield/dot-devcontainer-debian-fish-bookworm) - ベースとなるDebian + Fish環境
