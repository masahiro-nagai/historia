# 設計書

## 概要

「物語るヒストリア」は、テキストベースの小説形式で歴史を学習できるモバイルアプリです。ユーザーは坂本龍馬の視点から幕末の出来事を追体験し、物語の節目で選択肢を選ぶことで異なる展開を楽しむことができます。

## アーキテクチャ

### 技術スタック

**フロントエンド:**
- Flutter（iOS/Android対応のクロスプラットフォーム開発）
- Dart（型安全性とパフォーマンスを両立）
- Flutter Navigator 2.0（画面遷移管理）
- SharedPreferences / Hive（ローカルデータ保存）

**バックエンド:**
- 初期版では不要（全てローカルデータで完結）
- 将来的にはFirebase（ユーザー進捗の同期、分析）

**状態管理:**
- Provider / Riverpod（Flutterに最適化された状態管理）

**広告・課金:**
- Google Mobile Ads SDK for Flutter（広告表示）
- in_app_purchase（アプリ内課金）

### システム構成

```
┌─────────────────┐
│   モバイルアプリ   │
├─────────────────┤
│  プレゼンテーション層 │
│  - 画面コンポーネント │
│  - ナビゲーション    │
├─────────────────┤
│    ビジネス層      │
│  - 物語進行ロジック  │
│  - 選択肢処理      │
│  - 進捗管理       │
├─────────────────┤
│    データ層       │
│  - ローカルストレージ │
│  - 物語データ      │
│  - ユーザー設定    │
└─────────────────┘
```

## コンポーネントとインターフェース

### 主要コンポーネント

#### 1. 画面コンポーネント

**TitleScreen（タイトル画面）**
- アプリロゴ表示
- スタートボタン
- 設定ボタン

**StorySelectionScreen（物語選択画面）**
- 利用可能な物語一覧
- 進捗表示
- 課金状況表示

**StoryProgressScreen（物語進行画面）**
- テキスト表示エリア
- 選択肢ボタン群
- サポート機能ボタン
- 進捗インジケーター

**SupportModal（サポート機能モーダル）**
- タブ切り替え（人物/用語/地図）
- 解説コンテンツ表示
- 閉じるボタン

**PurchaseScreen（課金案内画面）**
- 広告非表示オプションの説明
- 購入ボタン
- 価格表示

#### 2. ビジネスロジックコンポーネント

**StoryEngine**
- 物語の進行管理
- 選択肢の処理
- 分岐ロジックの実行

**ProgressManager**
- ユーザーの進捗保存
- セーブデータ管理
- 選択履歴の記録

**ContentManager**
- 物語データの読み込み
- サポート情報の提供
- コンテンツの検索

**AdManager**
- 広告の表示制御
- 課金状況の確認
- 広告非表示の管理

### データモデル定義

```dart
// 物語データの構造
class StoryData {
  final String id;
  final String title;
  final List<Chapter> chapters;
  final List<Character> characters;
  final List<GlossaryItem> glossary;

  StoryData({
    required this.id,
    required this.title,
    required this.chapters,
    required this.characters,
    required this.glossary,
  });

  factory StoryData.fromJson(Map<String, dynamic> json) => StoryData(
    id: json['id'],
    title: json['title'],
    chapters: (json['chapters'] as List).map((e) => Chapter.fromJson(e)).toList(),
    characters: (json['characters'] as List).map((e) => Character.fromJson(e)).toList(),
    glossary: (json['glossary'] as List).map((e) => GlossaryItem.fromJson(e)).toList(),
  );
}

class Chapter {
  final String id;
  final String title;
  final List<Scene> scenes;

  Chapter({required this.id, required this.title, required this.scenes});

  factory Chapter.fromJson(Map<String, dynamic> json) => Chapter(
    id: json['id'],
    title: json['title'],
    scenes: (json['scenes'] as List).map((e) => Scene.fromJson(e)).toList(),
  );
}

class Scene {
  final String id;
  final String text;
  final List<Choice>? choices;
  final String? nextSceneId;

  Scene({required this.id, required this.text, this.choices, this.nextSceneId});

  factory Scene.fromJson(Map<String, dynamic> json) => Scene(
    id: json['id'],
    text: json['text'],
    choices: json['choices'] != null 
      ? (json['choices'] as List).map((e) => Choice.fromJson(e)).toList()
      : null,
    nextSceneId: json['nextSceneId'],
  );
}

class Choice {
  final String id;
  final String text;
  final String nextSceneId;
  final List<String>? consequences;

  Choice({
    required this.id,
    required this.text,
    required this.nextSceneId,
    this.consequences,
  });

  factory Choice.fromJson(Map<String, dynamic> json) => Choice(
    id: json['id'],
    text: json['text'],
    nextSceneId: json['nextSceneId'],
    consequences: json['consequences']?.cast<String>(),
  );
}

// ユーザー進捗データ
class UserProgress {
  final String storyId;
  final String currentSceneId;
  final List<String> choiceHistory;
  final List<String> completedChapters;
  final List<String> purchasedStories;

  UserProgress({
    required this.storyId,
    required this.currentSceneId,
    required this.choiceHistory,
    required this.completedChapters,
    required this.purchasedStories,
  });

  factory UserProgress.fromJson(Map<String, dynamic> json) => UserProgress(
    storyId: json['storyId'],
    currentSceneId: json['currentSceneId'],
    choiceHistory: json['choiceHistory'].cast<String>(),
    completedChapters: json['completedChapters'].cast<String>(),
    purchasedStories: json['purchasedStories'].cast<String>(),
  );

  Map<String, dynamic> toJson() => {
    'storyId': storyId,
    'currentSceneId': currentSceneId,
    'choiceHistory': choiceHistory,
    'completedChapters': completedChapters,
    'purchasedStories': purchasedStories,
  };
}

// サポート情報
class Character {
  final String id;
  final String name;
  final String description;
  final String historicalContext;

  Character({
    required this.id,
    required this.name,
    required this.description,
    required this.historicalContext,
  });

  factory Character.fromJson(Map<String, dynamic> json) => Character(
    id: json['id'],
    name: json['name'],
    description: json['description'],
    historicalContext: json['historicalContext'],
  );
}

enum GlossaryCategory { person, term, place, event }

class GlossaryItem {
  final String id;
  final String term;
  final String definition;
  final GlossaryCategory category;

  GlossaryItem({
    required this.id,
    required this.term,
    required this.definition,
    required this.category,
  });

  factory GlossaryItem.fromJson(Map<String, dynamic> json) => GlossaryItem(
    id: json['id'],
    term: json['term'],
    definition: json['definition'],
    category: GlossaryCategory.values.firstWhere(
      (e) => e.toString().split('.').last == json['category'],
    ),
  );
}
```

## データモデル

### ローカルストレージ構造

```
SharedPreferences / Hive:
├── user_progress_{storyId}     // ユーザーの進捗データ
├── user_settings              // アプリ設定
├── purchased_content          // 購入済みコンテンツ
└── ad_preferences            // 広告設定
```

### 物語データ構造

```
assets/stories/
├── bakumatsu_ryoma/
│   ├── story.json           // メインストーリーデータ
│   ├── characters.json      // 登場人物情報
│   ├── glossary.json       // 用語集
│   └── metadata.json       // メタデータ
```

## エラーハンドリング

### エラーカテゴリ

1. **データ読み込みエラー**
   - 物語データの破損
   - ローカルストレージアクセス失敗
   - 対処：エラーメッセージ表示、再試行オプション

2. **課金処理エラー**
   - 購入処理の失敗
   - レシート検証エラー
   - 対処：ユーザーへの明確な説明、サポート連絡先提示

3. **広告表示エラー**
   - 広告読み込み失敗
   - ネットワークエラー
   - 対処：広告なしで継続、後で再試行

4. **アプリクラッシュ**
   - 予期しないエラー
   - メモリ不足
   - 対処：エラーログ記録、安全な状態への復旧

### エラー処理戦略

```dart
// グローバルエラーハンドラー
class ErrorHandler {
  static void handleError(dynamic error, StackTrace stackTrace) {
    // エラーログの記録
    _logError(error, stackTrace);
    
    // ユーザーフレンドリーなメッセージ表示
    _showUserFriendlyMessage(error);
  }

  static void _logError(dynamic error, StackTrace stackTrace) {
    // Firebase Crashlytics等にログ送信
    print('Error: $error');
    print('StackTrace: $stackTrace');
  }

  static void _showUserFriendlyMessage(dynamic error) {
    // ユーザーに分かりやすいエラーメッセージを表示
  }
}

// 非同期処理のエラーハンドリング
Future<T?> handleAsyncError<T>(Future<T> Function() operation) async {
  try {
    return await operation();
  } catch (error, stackTrace) {
    ErrorHandler.handleError(error, stackTrace);
    return null;
  }
}

// Flutterアプリ全体のエラーハンドリング設定
void main() {
  FlutterError.onError = (FlutterErrorDetails details) {
    ErrorHandler.handleError(details.exception, details.stack ?? StackTrace.empty);
  };
  
  PlatformDispatcher.instance.onError = (error, stack) {
    ErrorHandler.handleError(error, stack);
    return true;
  };
  
  runApp(MyApp());
}
```

## テスト戦略

### テストレベル

1. **ユニットテスト**
   - ビジネスロジックの単体テスト
   - データ変換処理のテスト
   - Flutter Test Framework

2. **ウィジェットテスト**
   - UIコンポーネントのテスト
   - ユーザーインタラクションのテスト
   - Flutter Widget Testing

3. **統合テスト**
   - 画面遷移のテスト
   - データフローのテスト
   - Flutter Integration Test

4. **ユーザビリティテスト**
   - 実際のユーザーによる操作テスト
   - 物語の読みやすさ確認
   - 選択肢の分かりやすさ検証

### テスト対象

**重要度：高**
- 物語進行ロジック
- 選択肢処理
- 進捗保存・復元
- 課金処理

**重要度：中**
- 画面遷移
- サポート機能
- 広告表示

**重要度：低**
- UI アニメーション
- 設定画面

### テストデータ

```dart
// テスト用の物語データ
final mockStoryData = StoryData(
  id: 'test_story',
  title: 'テスト物語',
  chapters: [
    Chapter(
      id: 'chapter_1',
      title: '第一章',
      scenes: [
        Scene(
          id: 'scene_1',
          text: 'テストシーンです。',
          choices: [
            Choice(
              id: 'choice_1',
              text: '選択肢1',
              nextSceneId: 'scene_2',
            ),
          ],
        ),
      ],
    ),
  ],
  characters: [],
  glossary: [],
);

// テスト用のユーザー進捗データ
final mockUserProgress = UserProgress(
  storyId: 'test_story',
  currentSceneId: 'scene_1',
  choiceHistory: [],
  completedChapters: [],
  purchasedStories: [],
);
```

## パフォーマンス考慮事項

### 最適化戦略

1. **メモリ管理**
   - 大きなテキストデータの遅延読み込み
   - 不要なコンポーネントのアンマウント
   - 画像キャッシュの適切な管理

2. **レンダリング最適化**
   - const コンストラクタの活用
   - RepaintBoundary の適切な使用
   - ListView.builder の実装（長いリスト用）

3. **データアクセス最適化**
   - SharedPreferences / Hive の非同期処理最適化
   - 必要なデータのみの読み込み
   - キャッシュ戦略の実装

### パフォーマンス目標

- アプリ起動時間：3秒以内
- 画面遷移時間：1秒以内
- テキスト表示応答時間：0.5秒以内
- メモリ使用量：100MB以下（通常使用時）