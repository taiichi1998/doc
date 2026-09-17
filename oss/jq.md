# jq

## 用途

CLIでJSONを検索・抽出・変換するツール。

## メリット

- Azure CLIやREST APIのJSONを素早く加工できる
- Pipelineでレスポンス値を次工程へ渡しやすい
- 巨大なJSONから必要な値だけ確認できる
- AIが生成したCLI処理にも組み込みやすい

## 使用例

```bash
az group list | jq '.[].name'
```

```bash
cat response.json | jq '.items[] | {id, status}'
```

## このプロジェクトでの使い方

Azure CLI、デプロイ確認、APIデバッグ、CI/CDの補助処理に利用する。PowerShellのJSON処理で十分な処理を無理に置き換えず、短いCLI処理で効果がある箇所に限定する。

## 導入判断

優先度: 高。軽量で用途が広く、開発端末とSelf-hosted Agentの両方に導入候補。
