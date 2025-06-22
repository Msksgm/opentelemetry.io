---
title: 計装ライブラリの使用
linkTitle: ライブラリ
weight: 40
default_lang_commit: 2f850a610b5f7da5730265b32c25c9226dc09e5f
cSpell:ignore: httpx instrumentor uninstrument
---

{{% docs/languages/libraries-intro "python" %}}

## 計装ライブラリの使用 {#use-instrumentation-libraries}

ライブラリがネイティブ OpenTelemetry サポートを提供していない場合、[計装ライブラリ](/docs/specs/otel/glossary/#instrumentation-library)を使用して、ライブラリまたはフレームワークのテレメトリーデータを生成できます。

たとえば、[HTTPX 用の計装ライブラリ](https://pypi.org/project/opentelemetry-instrumentation-httpx/)は、HTTP リクエストに基づいて自動的に [スパン](/docs/concepts/signals/traces/#spans) を作成します。

## セットアップ {#setup}

pip を使用して各計装ライブラリを個別にインストールできます。
例を挙げましょう。

```sh
pip install opentelemetry-instrumentation-{instrumented-library}
```

前の例では、`{instrumented-library}` は計装の名前です。

開発版をインストールするには、`opentelemetry-python-contrib` リポジトリをクローンまたはフォークし、次のコマンドを実行して編集可能なインストールを行います。

```sh
pip install -e ./instrumentation/opentelemetry-instrumentation-{integration}
```

インストール後、計装ライブラリを初期化する必要があります。
各ライブラリは通常、独自の初期化方法を持っています。

## HTTPX 計装の例 {#example-with-httpx-instrumentation}

`httpx` ライブラリを使用して行われる HTTP リクエストを計装する方法を以下に示します。

まず、pip を使用して計装ライブラリをインストールします。

```sh
pip install opentelemetry-instrumentation-httpx
```

次に、instrumentor（計装ライブラリ）を使用してすべてのクライアントからのリクエストを自動的にトレースします。

```python
import httpx
from opentelemetry.instrumentation.httpx import HTTPXClientInstrumentor

url = "https://some.url/get"
HTTPXClientInstrumentor().instrument()

with httpx.Client() as client:
     response = client.get(url)

async with httpx.AsyncClient() as client:
     response = await client.get(url)
```

### 計装を無効にする {#turn-off-instrumentations}

必要に応じて、`uninstrument_client` メソッドを使用して特定のクライアントまたは、すべてのクライアントの計装を無効にできます。
例を挙げましょう。

```python
import httpx
from opentelemetry.instrumentation.httpx import HTTPXClientInstrumentor

HTTPXClientInstrumentor().instrument()
client = httpx.Client()

# 特定のクライアントの計装を無効にする
HTTPXClientInstrumentor.uninstrument_client(client)

# すべてのクライアントの計装を無効にする
HTTPXClientInstrumentor().uninstrument()
```

## 利用可能な計装ライブラリ {#available-instrumentation-libraries}

OpenTelemetry によって作成された計装ライブラリの完全なリストは、[opentelemetry-python-contrib][] リポジトリから入手できます。

[レジストリ](/ecosystem/registry/?language=python&component=instrumentation)でより多くの計装も見つけることができます。

## 次のステップ {#next-steps}

計装ライブラリを設定した後、カスタムテレメトリーデータを収集するために、コードに独自の[計装](/docs/languages/python/instrumentation)を追加することもできます。

また、1 つ以上のテレメトリーバックエンドに[テレメトリデータをエクスポート](/docs/languages/python/exporters) するために、適切なエクスポーターを設定することもできます。

[Python のゼロコード計装](/docs/zero-code/python/) も確認できます。

[opentelemetry-python-contrib]: https://github.com/open-telemetry/opentelemetry-python-contrib/tree/main/instrumentation#readme
