# FlowDoChat SwiftUI実装ガイド

> **目的:** 各HTMLファイルにセクションを書く際のルールと、SwiftUI実装全体のルールを定義する。

---

## 📁 ディレクトリ構造

```
FlowDoChat/
├── FlowDoChatApp.swift    # エントリーポイント
├── Core/
│   ├── Models/            # Struct, Enum定義
│   ├── Services/          # API呼び出し, 永続化
│   └── Utilities/         # 共通ユーティリティ
├── Features/
│   └── Main/
│       └── Home/          # 各Feature単位でフォルダ分け
│           ├── HomeView.swift
│           ├── ViewModels/
│           └── Views/     # サブコンポーネント
├── Shared/
│   └── Styles/            # Color+Extension, Modifiers
└── Assets.xcassets/
```

---

## 🗄️ データ永続化ルール

| データ | 保存先 | 理由 |
|---|---|---|
| タスクリスト | `@AppStorage` + Codable JSON | 軽量、CoreData不要 |
| ユーザープロフィール | `@AppStorage` + Codable | オンボーディングで設定 |
| ヒートマップ履歴 | `@AppStorage` + Codable | 簡易的な永続化 |

**ルール:**
- ❌ CoreData / SwiftData は使用しない
- ✅ `@AppStorage` + Codable で十分

---

## 🔌 API呼び出しルール

| 項目 | ルール |
|---|---|
| **Service** | `OpenAIService.swift` に集約 |
| **Model** | `gpt-4o-mini`（コスト最適化） |
| **呼び出し** | async/await + Task {} |
| **エラー** | do-catch で握りつぶさない |

**参照:** `API叩き方まとめ.html`

---

## 🧭 画面遷移ルール

| 遷移タイプ | 使うAPI | 使用シーン |
|---|---|---|
| **モーダル** | `.sheet` | 設定画面、詳細表示 |
| **フルスクリーンモーダル** | `.fullScreenCover` | オンボーディング |
| **プッシュ** | `NavigationStack` + `.navigationDestination` | ヒートマップ詳細 |
| **タブ** | `TabView` | ホーム/ヒートマップ切り替え |

---

## 📦 State管理ルール

| Property Wrapper | 使用シーン |
|---|---|
| `@State` | View内で閉じる一時的な状態 |
| `@Binding` | 親Viewから渡される状態 |
| `@StateObject` | ViewModelの所有（1回だけ生成） |
| `@ObservedObject` | ViewModelの参照（親から渡される） |
| `@AppStorage` | 永続化が必要なユーザー設定 |
| `@Environment` | 共有状態（例: カラースキーム） |

---

## 🎨 アニメーションルール

> **コンセプト:** 「ゆったり、優しく、生きている」

### 統一ルール（全画面共通）

| 用途 | Animation | 時間 | プリセット |
|------|-----------|------|------------|
| 画面遷移 | `.easeInOut` | 0.35秒 | `.standard` |
| タップ | `.easeInOut` | 0.15秒 | `.fast` |
| 出現 | `.easeOut` | 0.3秒 | `.appear` |
| 消失 | `.easeIn` | 0.25秒 | `.disappear` |
| 完了演出 | `.spring(dampingFraction: 0.7)` | 0.4秒 | `.completionSpring` |
| Aura呼吸 | `.easeInOut` 繰り返し | 4秒 | `.auraBreath` |

### View拡張

| Modifier | 用途 | 効果 |
|----------|------|------|
| `.gentleTapAnimation(isPressed:)` | ボタン押下 | scaleEffect(0.97) |
| `.gentleSelectionAnimation(isSelected:)` | カード選択 | scaleEffect(1.01) + 白シャドウ |
| `.gentleFadeIn(isVisible:)` | 優しい出現 | opacity |
| `.slideUpFadeIn(isVisible:)` | 下から出現 | opacity + offset |
| `.auraBreathing()` | Aura呼吸 | scale 0.85↔1.0 |

### Transition

| Transition | 用途 |
|------------|------|
| `.gentleSlideForward` | 進む（右から＋フェード） |
| `.gentleSlideBack` | 戻る（左から＋フェード） |
| `.slideUpFade` | シート展開 |
| `.gentleScaleFade` | モーダル |

### 禁止事項

| ❌ 禁止 | 理由 |
|--------|------|
| 1秒以上のブロッキングアニメ | ユーザーストレス |
| スキップ不可の演出 | 離脱リスク |
| 派手なバウンス | ゲームっぽい |
| 急激なスナップ | 威圧感 |
| 点滅（3Hz以上） | アクセシビリティ |

### 実装ファイル

- `AnimationStyles.swift` : 全プリセット定義

---

## 📝 HTMLセクション構成

各画面設計HTMLファイルには、ページ上部に**視覚的な統合ヘッダーセクション**を配置する。

### セクション構成（2カラムレイアウト）

```
┌─────────────────────────────────────────────────────────────┐
│  [タイトル] + [サブタイトル] + [フェーズバッジ]               │
├───────────────────────────────┬─────────────────────────────┤
│  📱 DESIGN SPEC              │  🛠️ SWIFTUI GUIDE           │
│  ・デザイン意図               │  ・STATE: @StateObject等     │
│  ・ユーザー体験の説明         │  ・NAV: 遷移方法             │
│  ・操作フロー                 │  ・NOTES: 実装注意点         │
│                              │  ・API: 呼び出しタイミング    │
└───────────────────────────────┴─────────────────────────────┘
```

### なぜこの構成？

| 問題 | 解決 |
|---|---|
| コメント形式はブラウザで見えない | **視覚的なHTML要素**で表示 |
| デザインと実装の情報が分離 | **2カラム**で並べて比較可能 |
| 下部に散らばったセクション | **ページ上部に集約** |
| AI以外の開発者が迷う | カラーコーディングで**瞬時に判別** |

### カラーコーディング

| 色 | 用途 |
|---|---|
| 🔵 **Blue** | DESIGN SPEC（デザイン・UX情報） |
| 🟢 **Emerald** | SWIFTUI GUIDE（技術情報） |
| 🟣 **Purple** | API呼び出し情報 |
| 🟡 **Yellow** | 実装上の注意点 |
| 🔴 **Red** | 重要な警告・難しい実装 |

### 適用済みファイル（ホーム画面）

| ファイル | 状態 |
|---|---|
| ホーム_3段階ミニバー.html | ✅ 完了 |
| ChatPatern.html | ✅ 完了 |
| ホーム_Day2朝.html | ✅ 完了 |
| ホーム_タスクゼロ.html | ✅ 完了 |
| ホーム_タスク提案カード.html | ✅ 完了 |
| ホーム_入力フォーカス.html | ✅ 完了 |
| タスク詳細.html | ⏳ 既存セクションあり |

---

## 🔗 関連文書

- `コーディング規約.md` : コーディングスタイル
- `実装引継ぎ書.md` : 設計意図の引継ぎ
- `API叩き方まとめ.html` : データ構造・APIスキーマ

