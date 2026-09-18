# kikaku-neoforge-packwiz

企画鯖(NeoForge, Minecraft 1.21.1)の MOD 構成を [packwiz](https://packwiz.infra.link/) で管理するリポジトリです。

`main` ブランチへの push をトリガーに GitHub Pages が自動更新され、以下の URL で `pack.toml` が常に最新の状態で公開されます。

```
https://ryo-server-developer.github.io/kikaku-neoforge-packwiz/pack.toml
```

このリポジトリは kikaku-server 本体の Kubernetes マニフェスト([rouzinkai-infra](https://github.com/rouzinkai-dev/rouzinkai-infra) の `k8s/prd/manifest/ryo-server/kikaku-server/`)とは別リポジトリです。サーバー起動時に `PACKWIZ_URL` 環境変数からこの `pack.toml` を参照し、MOD 一式を自動ダウンロードします。

## 構成

GitHub Pages の公開範囲を packwiz 用ファイルだけに絞るため、公開対象は `docs/` 配下のみです(Pages の設定で source path を `/docs` にしている)。`README.md` / `CLAUDE.md` / `USAGE.md` / `mise.toml` はリポジトリ直下にあり、**クライアント(PrismLauncher等)からは参照できません**。

- `docs/pack.toml` / `docs/index.toml`: packwiz 本体の定義ファイル(Pages公開対象)
- `docs/mods/*.pw.toml`: MOD ごとのダウンロード元URL・ハッシュ・更新元情報(jar実体は含まない、Pages公開対象)
- Minecraft: `1.21.1` / NeoForge: `21.1.251`(初期化時点)

以降の `packwiz` コマンドは全て `docs/` ディレクトリ内で実行してください。

## セットアップ(初回のみ)

このリポジトリの `mise.toml` に Go と packwiz(`go install` ベース)が定義されているので、`mise install` だけで導入できます。

```bash
mise install
mise exec -- packwiz --version
```

packwiz は GitHub Releases を配布していないため、mise では `go:github.com/packwiz/packwiz` バックエンド(内部で `go install` を実行)を使っています。

## MOD の追加・更新・削除(手動)

MOD 管理は必ず `packwiz` CLI 経由で行ってください。`mods/*.pw.toml` を手編集すると、ダウンロードURLとハッシュの不整合でサーバー起動時のMOD取得に失敗します。

### 追加

**特定バージョンを指定したい場合(推奨)**: Modrinthの「バージョンページのURL」をそのまま渡します。プロジェクトと対象バージョン(ローダー/MCバージョン込み)を一発で解決できるため、スラッグだけの指定で誤ったローダー版(例: NeoForge対応のつもりがFabric版を取得してしまう)を引く事故を防げます。

```bash
cd docs
mise exec -- packwiz modrinth add "https://modrinth.com/mod/<slug>/version/<version-id-or-number>" -y
```

**最新版でよい場合**: スラッグだけを渡します(対応ローダー/MCバージョンに合う最新版が自動選択されます)。

```bash
cd docs
mise exec -- packwiz modrinth add <slug> -y
```

CurseForge限定配布のMODは `packwiz curseforge add <slug>` を使用します。

### 更新

```bash
# 個別MOD
mise exec -- packwiz update <slug>

# 全MOD一括
mise exec -- packwiz update --all
```

### 削除

```bash
mise exec -- packwiz remove <slug>
```

### 反映

コマンド実行後、`docs/pack.toml` / `docs/index.toml` / `docs/mods/*.pw.toml` の差分を確認し、コミット・`main` へ push してください。push すると GitHub Pages が自動的に再ビルドされ、kikaku-server の次回起動時から新しいMOD構成が取得されます。

```bash
git add docs/pack.toml docs/index.toml docs/mods/
git commit -m "MODを更新"
git push
```

**注意**: このリポジトリへの push は本番稼働中の kikaku-server が参照する `pack.toml` に直接反映されます。追加・更新するMODが対象バージョン(MC 1.21.1 / NeoForge)に対応しているか事前に確認してください。
