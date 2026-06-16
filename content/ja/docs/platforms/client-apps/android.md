---
title: Android
description: >-
  Androidプラットフォームで動作するアプリでOpenTelemetryを使用する
weight: 10
vers:
  ot-android: 1.4.0
cSpell:ignore: inactivity
default_lang_commit: 68c29178b
---

OpenTelemetry Androidは、ネイティブAndroidアプリケーションにオブザーバビリティを提供します。
[OpenTelemetry Java](/docs/languages/java/)エコシステムを基盤として構築されており、モバイル環境に特化した自動計装、リアルユーザーモニタリング（RUM）、および手動計装の機能を提供します。

## 機能 {#features}

OpenTelemetry Androidには、以下の主要な機能が含まれます。

- **自動計装**：一般的なAndroidパターン向けの組み込みモジュール。
  - Activityライフサイクル
  - Fragmentライフサイクル
  - ANR（Application Not Responding）検出
  - クラッシュレポート
  - ネットワーク変更検出
  - スロー/フローズンフレームレンダリング検出
  - 起動タイミング
  - 画面の向き
  - ビュークリックイベント
- **セッション管理**：設定可能な非アクティブタイムアウトと最大セッション期間でユーザーセッションを追跡します。
- **オフラインバッファリング**：デバイスがオフラインのときにテレメトリーデータをバッファリングするディスク永続化により、ネットワーク障害中のデータ損失を防ぎます。
- **属性のリダクション**：プライバシーコンプライアンスのために、エクスポート前にスパン属性を編集または変更する機能です。

## はじめに {#getting-started}

### 前提条件 {#prerequisites}

- Android SDK 21（Lollipop）以降
- Kotlinを使用するGradleプロジェクト（Javaも可能な場合があります）

### Gradleの設定 {#gradle-setup}

アプリレベルの`build.gradle.kts`ファイルに、OpenTelemetry Android Agentの依存関係を追加します。
バージョン管理にはBOM（Bill of Materials）を使用します。

```kotlin
dependencies {
    implementation(platform("io.opentelemetry.android:opentelemetry-android-bom:{{% param vers.ot-android %}}"))
    implementation("io.opentelemetry.android:android-agent")
}
```

> [!NOTE]
>
> 最新バージョンは
> [OpenTelemetry Androidリリース](https://github.com/open-telemetry/opentelemetry-android/releases)
> でご確認ください。

### エージェントの初期化 {#initialize-the-agent}

`Application`クラスの`onCreate()`メソッドでOpenTelemetryを初期化します：

```kotlin
class MyApplication : Application() {
    lateinit var openTelemetryRum: OpenTelemetryRum

    override fun onCreate() {
        super.onCreate()
        openTelemetryRum = initializeOpenTelemetry(this)
    }
}

private fun initializeOpenTelemetry(context: Context): OpenTelemetryRum =
    OpenTelemetryRumInitializer.initialize(
        context = context,
        configuration = {
            httpExport {
                baseUrl = "https://your-collector-endpoint:4318"
                baseHeaders = mapOf("Authorization" to "Bearer <token>")
            }
            instrumentations {
                // すべての計装はデフォルトで有効です。
                // 必要に応じて特定のものを無効にします：
                slowRendering { enabled(false) }
            }
            session {
                backgroundInactivityTimeout = 15.minutes
                maxLifetime = 4.days
            }
        }
    )
```

## 設定 {#configuration}

OpenTelemetry Androidは、上記の初期化例に示すように、KotlinのDSLを使用して設定します。
以下の表は、利用可能な設定オプションを説明しています：

### 設定オプション {#configuration-options}

| ブロック                                     | 説明                                              |
| ----------------------------------------- | ------------------------------------------------- |
| `httpExport { baseUrl }`                  | テレメトリーをエクスポートするOTLPエンドポイントURL |
| `httpExport { baseHeaders }`              | エクスポートリクエストに含めるカスタムヘッダー      |
| `globalAttributes`                        | すべてのテレメトリーに追加される属性               |
| `session { backgroundInactivityTimeout }` | 新しいセッションを開始するまでの非アクティブタイムアウト |
| `session { maxLifetime }`                 | セッションの最大有効期間                          |
| `instrumentations`                        | 個々の自動計装モジュールを設定する                 |

## 自動計装 {#automatic-instrumentation}

OpenTelemetry Androidは、有効または無効にできる自動計装モジュールを提供します。
各計装の詳細情報（出力されるテレメトリーや設定オプションを含む）については、リンク先のドキュメントを参照してください。

### Activityライフサイクル {#activity-lifecycle}

Activityライフサイクルイベント（`onCreate`、`onStart`、`onResume`、`onPause`、`onStop`、`onDestroy`）のスパンを自動的にキャプチャします。
[Activity計装](https://github.com/open-telemetry/opentelemetry-android/blob/main/instrumentation/activity/README.md)を参照してください。

### Fragmentライフサイクル {#fragment-lifecycle}

Fragmentライフサイクルイベントのスパンをキャプチャします。シングルアクティビティアーキテクチャ内のナビゲーション追跡に便利です。
[Fragment計装](https://github.com/open-telemetry/opentelemetry-android/blob/main/instrumentation/fragment/README.md)を参照してください。

### ANR検出 {#anr-detection}

Application Not Responding（ANR）の状態を検出してスパンとして報告し、UIスレッドのブロッキング問題の特定を支援します。
[ANR計装](https://github.com/open-telemetry/opentelemetry-android/blob/main/instrumentation/anr/README.md)を参照してください。

### クラッシュレポート {#crash-reporting}

未処理の例外をスタックトレースとともにキャプチャし、クラッシュをユーザーセッションやトレースと関連付けられるようにします。
[Crash計装](https://github.com/open-telemetry/opentelemetry-android/blob/main/instrumentation/crash/README.md)を参照してください。

### ネットワーク監視 {#network-monitoring}

ネットワーク状態の変化を検出してテレメトリーに接続情報を追加し、エラー発生時のネットワーク状況を把握できるようにします。
[Network計装](https://github.com/open-telemetry/opentelemetry-android/blob/main/instrumentation/network/README.md)を参照してください。

### スローフレームとフローズンフレーム {#slow-and-frozen-frames}

フレームレンダリングのパフォーマンスを監視し、スローレンダリング（16ms超）やフローズンフレーム（700ms超）を報告することで、UIパフォーマンスのボトルネック特定を支援します。
[スローレンダリング計装](https://github.com/open-telemetry/opentelemetry-android/blob/main/instrumentation/slowrendering/README.md)を参照してください。

## 手動計装 {#manual-instrumentation}

手動計装のためにOpenTelemetry APIにアクセスします：

```kotlin
val openTelemetry = openTelemetryRum.openTelemetry
val tracer = openTelemetry.getTracer("com.example.myapp")

val span = tracer.spanBuilder("my-operation")
    .startSpan()

try {
    span.makeCurrent().use {
        // ここにコードを記述
    }
} finally {
    span.end()
}
```

## HTTPクライアント計装 {#http-client-instrumentation}

OkHttpクライアントを計装してネットワークリクエストをトレースします：

```kotlin
val okHttpClient = OkHttpTelemetry.builder(openTelemetryRum.openTelemetry)
    .build()
    .newCallFactory(OkHttpClient.Builder().build())
```

## ベストプラクティス {#best-practices}

### リソース制約 {#resource-constraints}

モバイルデバイスのリソースは限られています。以下のベストプラクティスを考慮してください：

- **バッチエクスポート**：ネットワーク呼び出しとバッテリー消費を削減するために、バッチ処理がデフォルトで有効になっています。
- **サンプリング**：代表的なテレメトリーを維持しながらデータ量を削減するために、サンプリング戦略を実装してください。
- **オフラインバッファリング**：断続的な接続に対応するために、ディスク永続化がデフォルトで有効になっています。

### プライバシーへの配慮 {#privacy-considerations}

- エクスポート前に機密データを削除するために属性リダクションを使用してください。
- テレメトリーの収集に関するユーザーの同意要件を考慮してください。
- スパン名や属性に個人を特定できる情報（PII）を含めないようにしてください。

### テスト {#testing}

エミュレーターでテストする場合は、ローカルマシンのコレクターにアクセスするためのホストアドレスとして`10.0.2.2`を使用します：

```kotlin
httpExport {
    baseUrl = "http://10.0.2.2:4318"
}
```

## リソース {#resources}

- [OpenTelemetry Android GitHub](https://github.com/open-telemetry/opentelemetry-android)
- [OpenTelemetry Javaドキュメント](/docs/languages/java/)
- [Androidセマンティック規則](/docs/specs/semconv/registry/attributes/android/)
- [サンプルアプリケーション](https://github.com/open-telemetry/opentelemetry-android/tree/main/demo-app)

## ヘルプとフィードバック {#help-and-feedback}

ご質問がある場合は、
[GitHub Issues](https://github.com/open-telemetry/opentelemetry-android/issues)または[CNCF Slack](https://slack.cncf.io/)の[#otel-android](https://cloud-native.slack.com/archives/C05J0T9K27Q)チャンネルからお問い合わせください。
