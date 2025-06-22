---
title: プロパゲーション
description: Python SDK のコンテキストプロパゲーション
weight: 65
default_lang_commit: 2f850a610b5f7da5730265b32c25c9226dc09e5f
---

プロパゲーションは、サービスとプロセス間でデータを移動するメカニズムです。トレースに限定されるものではありませんが、プロセスとネットワークの境界を超えて任意に分散されたサービス間で、システムに関する因果関係の情報をトレースが構築できるようにするものです。

OpenTelemetry は、[W3C Trace Context](https://www.w3.org/TR/trace-context/) HTTP ヘッダーを使用してリモートサービスにコンテキストを伝搬するテキストベースのアプローチを提供します。

## 自動コンテキストプロパゲーション {#automatic-context-propagation}

Jinja2、Flask、Django、Celery などの人気のある Python フレームワークやライブラリ用の計装ライブラリが、サービス間でコンテキストを伝搬します。

{{% alert title="注意" %}}

コンテキストのプロパゲーションには計装ライブラリを使用してください。
手動でコンテキストを伝搬することも可能ですが、Python の自動計装と計装ライブラリは十分にテストされており、使いやすいです。

{{% /alert %}}

## 手動コンテキストプロパゲーション {#manual-context-propagation}

以下の汎用的な例では、トレースコンテキストを手動で伝搬する方法を示しています。

まず、送信側のサービスで、現在の `context` を注入します。

```python
from flask import Flask
import requests
from opentelemetry import trace, baggage
from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator
from opentelemetry.baggage.propagation import W3CBaggagePropagator
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, BatchSpanProcessor

app = Flask(__name__)

trace.set_tracer_provider(TracerProvider())
trace.get_tracer_provider().add_span_processor(BatchSpanProcessor(ConsoleSpanExporter()))

tracer = trace.get_tracer(__name__)

@app.route('/')
def hello():
    with tracer.start_as_current_span("api1_span") as span:
        ctx = baggage.set_baggage("hello", "world")

        headers = {}
        W3CBaggagePropagator().inject(headers, ctx)
        TraceContextTextMapPropagator().inject(headers, ctx)
        print(headers)

        response = requests.get('http://127.0.0.1:5001/', headers=headers)
        return f"Hello from API 1! Response from API 2: {response.text}"

if __name__ == '__main__':
    app.run(port=5002)
```

受信側のサービスでは、たとえばパースされた HTTP ヘッダーから `context` を抽出し、それを現在のトレースコンテキストとして設定します。

```python
from flask import Flask, request
from opentelemetry import trace, baggage
from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, BatchSpanProcessor
from opentelemetry.baggage.propagation import W3CBaggagePropagator

app = Flask(__name__)

trace.set_tracer_provider(TracerProvider())
trace.get_tracer_provider().add_span_processor(BatchSpanProcessor(ConsoleSpanExporter()))

tracer = trace.get_tracer(__name__)

@app.route('/')
def hello():
    # 例: API 2 でリクエストで受信したヘッダーをログ出力
    headers = dict(request.headers)
    print(f"Received headers: {headers}")
    carrier ={'traceparent': headers['Traceparent']}
    ctx = TraceContextTextMapPropagator().extract(carrier=carrier)
    print(f"Received context: {ctx}")

    b2 ={'baggage': headers['Baggage']}
    ctx2 = W3CBaggagePropagator().extract(b2, context=ctx)
    print(f"Received context2: {ctx2}")

    # 新しいスパンを開始
    with tracer.start_span("api2_span", context=ctx2):
       # 伝搬されたコンテキストを使用
        print(baggage.get_baggage('hello', ctx2))
        return "Hello from API 2!"

if __name__ == '__main__':
    app.run(port=5001)
```

そこから、デシリアライズされたアクティブなコンテキストがあれば、他のサービスからの同じトレースの一部であるスパンを作成できます。

## 次のステップ {#next-steps}

プロパゲーションについて詳しく学ぶには、[プロパゲーター API](/docs/specs/otel/context/api-propagators/) を参照してください。
