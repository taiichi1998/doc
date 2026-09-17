# ripgrep (rg) + fzf

## 概要

- `ripgrep (rg)`: コード・設定・ドキュメントを高速全文検索する CLI
- `fzf`: 検索結果やファイル、履歴などを fuzzy search で対話的に絞り込む CLI

組み合わせると、大規模リポジトリを高速に探索できる。

## Claude Code と併用するメリット

Claude Code にとって重要なのは「必要なコンテキストだけを素早く発見すること」。rg/fzf は人間用 CLI としてだけでなく、AI と共同作業するときのコード探索にも便利。

### 1. AI に渡す対象を絞る

```bash
rg "approvalStatus" services/ apps/
```

検索結果から関連ファイルだけ確認し、Claude Code に対象範囲を明示できる。

### 2. 巨大リポジトリで探索を高速化

```bash
rg --files | fzf
```

大量のファイルから目的のファイルを数文字で選択できる。

### 3. AI の変更を人間が確認

Claude Code が多数ファイルを変更した後、関連コードや設定値が他に残っていないか高速確認できる。

```bash
rg "OLD_API_NAME"
rg "TODO|FIXME"
```

### 4. Claude Code の指示を具体化

```text
rg で旧 API 名を検索し、該当箇所を確認してから変更する。
```

のように、探索手順を Claude Code の Skill / 作業指示に組み込める。

## VS Code 検索との違い

VS Code の検索でも多くの用途は満たせる。rg の価値は CLI / Script / AI Agent / CI から同じ検索方法を利用できること。

fzf は必須ではなく、人間がターミナル内で大量候補を選択する操作を高速化する QoL ツール。

## 推奨方針

```text
Claude Code
   +
rg     -> コード探索・確認
   +
fzf    -> 人間の対話的選択
```

## 導入判断

ripgrep: ★★★★☆
fzf: ★★★☆☆

Claude Code が検索を代行できるため必須ではない。ただし rg は軽量で汎用性が高く、AI と人間の両方で同じ探索手段を使えるため導入価値がある。