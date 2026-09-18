# CLAUDE.md

このリポジトリで作業する AI Agent 向けの実務ガイドです。

## 言語

- ユーザー向けの会話・ドキュメントは日本語

## リポジトリの役割

企画鯖(kikaku-server, NeoForge, Minecraft 1.21.1)の MOD 構成を [packwiz](https://packwiz.infra.link/) で管理し、GitHub Pages でホストするためだけのリポジトリです。`main` への push が即座に以下のURLへ反映されます。

```
https://ryo-server-developer.github.io/kikaku-neoforge-packwiz/pack.toml
```

kikaku-server 本体の Kubernetes マニフェストは別リポジトリ([rouzinkai-dev/rouzinkai-infra](https://github.com/rouzinkai-dev/rouzinkai-infra) の `k8s/prd/manifest/ryo-server/kikaku-server/`)にあり、そちらの `PACKWIZ_URL` 環境変数がこのリポジトリの `pack.toml` を参照する想定です。

## 変更時の基本方針

- **`main` への push は本番の kikaku-server に直接影響する**(サーバー再起動時に新しい `pack.toml` を取得するため)。MOD の追加・更新は対象 MC/ローダーバージョンとの互換性を確認してから行うこと
- `mods/*.pw.toml` を手編集しない。ダウンロードURLとハッシュは packwiz CLI が管理するため、手編集すると起動時のMOD取得整合性が壊れる
- MOD の追加・更新・削除は必ず `packwiz` CLI 経由で行い、`README.md` の手順に従う
- `pack.toml` の `[versions]`(minecraft / neoforge)を変更する場合、既存 MOD 全ての対応バージョンとの整合性を確認する(`packwiz migrate` を利用)

## MOD操作の要点(詳細は README.md 参照)

- 追加: `mise exec -- packwiz modrinth add <slug> -y`
- 特定バージョン指定時: `--project-id` と `--version-id` を両方指定する(スラッグ位置引数と `--version-id` の併用は不可)
- 更新: `mise exec -- packwiz update <slug>` / `--all`
- 削除: `mise exec -- packwiz remove <slug>`
- CurseForge由来のMOD: `packwiz curseforge add`

## 開発環境

- ツール管理は `mise.toml`(Go + packwiz を `go:github.com/packwiz/packwiz` バックエンドで導入)
- `mise install` の後、`mise exec -- packwiz <command>` で実行する

## 注意点

- packwiz は GitHub Releases を配布していないため、`go install` 前提。mise 導入時に Go 本体も併せてインストールされる
- Modrinth API から取得する `version-id` は、`https://api.modrinth.com/v2/project/<slug>/version` のレスポンス内 `id`(または `version_number` 一致)から取得できる
- GitHub Pages は "Deploy from a branch"(`main` / `/`)方式。ビルドステップは不要(静的な toml をそのまま配信)
