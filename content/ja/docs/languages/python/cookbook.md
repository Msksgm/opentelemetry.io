---
title: クックブック
weight: 100
default_lang_commit: 2f850a610b5f7da5730265b32c25c9226dc09e5f
---

このページは、一般的なシナリオのクックブックです。

## 新しいスパンを作成する {#create-a-new-span}

```python
from opentelemetry import trace

tracer = trace.get_tracer("my.tracer")
with tracer.start_as_current_span("print") as span:
    print("foo")
    span.set_attribute("printed_string", "foo")
```

## スパンを取得して変更する {#getting-and-modifying-a-span}

```python
from opentelemetry import trace

current_span = trace.get_current_span()
current_span.set_attribute("hometown", "Seattle")
```

## ネストしたスパンを作成する {#create-a-nested-span}

```python
from opentelemetry import trace
import time

tracer = trace.get_tracer("my.tracer")

# 何らかの作業を追跡するための新しいスパンを作成
with tracer.start_as_current_span("parent"):
    time.sleep(1)

    # ネストした作業を追跡するためのネストしたスパンを作成
    with tracer.start_as_current_span("child"):
        time.sleep(2)
        # ネストしたスパンはスコープから外れると閉じられます

    # 今度は親スパンが再び現在のスパンになります
    time.sleep(1)

    # このスパンもスコープから外れると閉じられます
```

## 異なるコンテキストでバゲージを取得する {#capturing-baggage-at-different-contexts}

```python
from opentelemetry import trace, baggage

tracer = trace.get_tracer("my.tracer")
with tracer.start_as_current_span(name="root span") as root_span:
    parent_ctx = baggage.set_baggage("context", "parent")
    with tracer.start_as_current_span(
        name="child span", context=parent_ctx
    ) as child_span:
        child_ctx = baggage.set_baggage("context", "child")

print(baggage.get_baggage("context", parent_ctx))
print(baggage.get_baggage("context", child_ctx))
```

## スパンコンテキストを手動で設定する {#manually-setting-span-context}

通常、アプリケーションまたは提供フレームワークがトレースコンテキストのプロパゲーションを処理してくれます。
しかし、場合によっては、トレースコンテキストを（`.inject` で）保存し、他の場所で（`.extract` で）復元する必要があるかもしれません。

```python
from opentelemetry import trace, context
from opentelemetry.trace import NonRecordingSpan, SpanContext, TraceFlags
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, BatchSpanProcessor
from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator

# 何が起こっているかを確認できるように、スパンをコンソールに書き出すシンプルなプロセッサーを設定します。
trace.set_tracer_provider(TracerProvider())
trace.get_tracer_provider().add_span_processor(BatchSpanProcessor(ConsoleSpanExporter()))

tracer = trace.get_tracer("my.tracer")

# TextMapPropagator はデフォルトで任意の辞書ライクオブジェクトをキャリアとして動作します。カスタムゲッターとセッターを実装することもできます。
with tracer.start_as_current_span('first-trace'):
    carrier = {}
    # 現在のコンテキストをキャリアに書き込みます。
    TraceContextTextMapPropagator().inject(carrier)

# 以下は別のスレッド、別のマシンなどにある可能性があります。
# 典型的な例として、これは異なるマイクロサービスにあり、キャリアは
# HTTP ヘッダーを介して転送されているでしょう。

# キャリアからトレースコンテキストを抽出します。
# これは、上記で注入されたような典型的なキャリアの例です。
carrier = {'traceparent': '00-a9c3b99a95cc045e573e163c3ac80a77-d99d251a8caecd06-01'}
# 次に、プロパゲーターを使用してそこからコンテキストを取得します。
ctx = TraceContextTextMapPropagator().extract(carrier=carrier)

# キャリアからトレースコンテキストを抽出する代わりに、すでに SpanContext
# オブジェクトがある場合は、次のようにそこからトレースコンテキストを取得できます。
span_context = SpanContext(
    trace_id=2604504634922341076776623263868986797,
    span_id=5213367945872657620,
    is_remote=True,
    trace_flags=TraceFlags(0x01)
)
ctx = trace.set_span_in_context(NonRecordingSpan(span_context))

# これで、トレースコンテキストを利用するいくつかの方法があります。

# スパンを開始するときにコンテキストオブジェクトを渡すことができます。
with tracer.start_as_current_span('child', context=ctx) as span:
    span.set_attribute('primes', [2, 3, 5, 7])

# または、それを現在のコンテキストにして、次のスパンがそれを選択できるようにできます。
# 返されたトークンを使用して、以前のコンテキストを復元できます。
token = context.attach(ctx)
try:
    with tracer.start_as_current_span('child') as span:
        span.set_attribute('evens', [2, 4, 6, 8])
finally:
    context.detach(token)
```

## 異なるリソースで複数のトレーサープロバイダーを使用する {#using-multiple-tracer-providers-with-different-resource}

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, BatchSpanProcessor

# 一度だけ設定できるグローバルトレーサープロバイダー
trace.set_tracer_provider(
    TracerProvider(resource=Resource.create({"service.name": "service1"}))
)
trace.get_tracer_provider().add_span_processor(BatchSpanProcessor(ConsoleSpanExporter()))

tracer = trace.get_tracer("tracer.one")
with tracer.start_as_current_span("some-name") as span:
    span.set_attribute("key", "value")



another_tracer_provider = TracerProvider(
    resource=Resource.create({"service.name": "service2"})
)
another_tracer_provider.add_span_processor(BatchSpanProcessor(ConsoleSpanExporter()))

another_tracer = trace.get_tracer("tracer.two", tracer_provider=another_tracer_provider)
with another_tracer.start_as_current_span("name-here") as span:
    span.set_attribute("another-key", "another-value")
```
