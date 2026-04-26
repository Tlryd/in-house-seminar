作成しました。
Canvasに **「Google Docs Meeting Memo Corrector Requirements」** としてMarkdown形式の要件定義書を置いています。

内容はCopilot CLIに読ませやすいように、以下を含めています。

* MVP範囲
* 機能要件
* 画面要件
* データ設計案
* API設計案
* 処理フロー
* エラーハンドリング
* テスト観点
* Copilot CLI向け実装指示

このまま `requirements.md` などで保存して、Copilot CLIに

```bash
copilot "requirements.md を読んで、このMVPを実装してください。まず設計方針と作業計画を提示してください。"
```

のように渡せます。
