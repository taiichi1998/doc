# Gitleaks

## 用途

Gitリポジトリや変更差分からSecret、API Key、Token、Password等の誤コミットを検出する。

## メリット

- Secret漏洩を専用ルールで早期検出できる
- ローカルとCIの両方で実行可能
- Git履歴も検査できる
- 軽量でPipelineへ組み込みやすい

## このプロジェクトでの使い方

```text
Developer
  ↓
Pre-commit（任意）
  ↓
PR Pipeline
  ↓
Gitleaks
  ↓
検出時はPRをBlock
```

Entra Client Secret、Azure接続情報、API Keyなどを対象にする。原則としてSecretはManaged Identity / Key Vault等へ寄せ、Gitleaksは漏洩防止の最後のガードとして扱う。

## Trivyとの棲み分け

TrivyにもSecret検出機能はあるため、最初に検出範囲と実行時間を比較する。Gitleaks専用導入で重複が大きい場合は無理に増やさない。

## 導入判断

優先度: 高。PRのCritical/Highブロック方針と相性が良い。
