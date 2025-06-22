---
title: mypy の使用
weight: 120
default_lang_commit: 2f850a610b5f7da5730265b32c25c9226dc09e5f
cSpell:ignore: mypy
---

[mypy](https://mypy-lang.org/) を使用している場合、[名前空間パッケージ](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-namespace-packages)を有効にする必要があります。
そうしないと、`mypy` が正しく実行されません。

名前空間パッケージを有効にするには、次のいずれかを行います。

プロジェクト設定ファイルに次を追加します。

```toml
[tool.mypy]
namespace_packages = true
```

または、コマンドラインスイッチを使用します。

```shell
mypy --namespace-packages
```
