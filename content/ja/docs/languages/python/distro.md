---
title: OpenTelemetry ディストロ
linkTitle: ディストロ
weight: 110
default_lang_commit: 2f850a610b5f7da5730265b32c25c9226dc09e5f
cSpell:ignore: distro
---

柔軟性を犠牲にすることなく、OpenTelemetry と自動計装の使用を可能な限り迅速にするために、OpenTelemetry ディストロは、より一般的なオプションの一部をユーザー向けに自動的に設定するメカニズムを提供します。
その力を活用することで、OpenTelemetry のユーザーは必要に応じてコンポーネントを設定できます。
`opentelemetry-distro` パッケージは、開始を検討しているユーザーにいくつかのデフォルトを提供し、次のものを設定します。

- SDK TracerProvider
- BatchSpanProcessor
- OpenTelemetry Collector にデータを送信する OTLP `SpanExporter`

このパッケージは、代替ディストロの作成に興味のある人にとっての出発点も提供します。
このパッケージによって実装されるインターフェースは、`opentelemetry_distro` および`opentelemetry_configurator` エントリーポイントを介して自動計装によってロードされ、他のコードが実行される前にアプリケーションを設定します。

OpenTelemetry から OpenTelemetry Collector にデータを自動的にエクスポートするために、パッケージをインストールすると、必要なすべてのエントリーポイントが設定されます。

```sh
pip install opentelemetry-distro[otlp] opentelemetry-instrumentation
```

データがエクスポートされることを確認するために、Collector をローカルで開始します。
次のファイルを作成します。

```yaml
# /tmp/otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
exporters:
  # 注意: v0.86.0 より前では `debug` の代わりに `logging` を使用してください。
  debug:
    verbosity: detailed
processors:
  batch:
service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [debug]
      processors: [batch]
```

次に Docker コンテナを開始します。

```sh
docker run -p 4317:4317 \
    -v /tmp/otel-collector-config.yaml:/etc/otel-collector-config.yaml \
    otel/opentelemetry-collector:latest \
    --config=/etc/otel-collector-config.yaml
```

次のコードは、設定なしでスパンを作成します。

```python
# no_configuration.py
from opentelemetry import trace

with trace.get_tracer("my.tracer").start_as_current_span("foo"):
    with trace.get_tracer("my.tracer").start_as_current_span("bar"):
        print("baz")
```

最後に、自動計装で `no_configuration.py` を実行します。

```sh
opentelemetry-instrument python no_configuration.py
```

結果のスパンは Collector からの出力に表示され、次のようになります。

```nocode
Resource labels:
     -> telemetry.sdk.language: STRING(python)
     -> telemetry.sdk.name: STRING(opentelemetry)
     -> telemetry.sdk.version: STRING(1.1.0)
     -> service.name: STRING(unknown_service)
InstrumentationLibrarySpans #0
InstrumentationLibrary __main__
Span #0
    Trace ID       : db3c99e5bfc50ef8be1773c3765e8845
    Parent ID      : 0677126a4d110cb8
    ID             : 3163b3022808ed1b
    Name           : bar
    Kind           : SPAN_KIND_INTERNAL
    Start time     : 2021-05-06 22:54:51.23063 +0000 UTC
    End time       : 2021-05-06 22:54:51.230684 +0000 UTC
    Status code    : STATUS_CODE_UNSET
    Status message :
Span #1
    Trace ID       : db3c99e5bfc50ef8be1773c3765e8845
    Parent ID      :
    ID             : 0677126a4d110cb8
    Name           : foo
    Kind           : SPAN_KIND_INTERNAL
    Start time     : 2021-05-06 22:54:51.230549 +0000 UTC
    End time       : 2021-05-06 22:54:51.230706 +0000 UTC
    Status code    : STATUS_CODE_UNSET
    Status message :
```
