# GTFS-RT データ仕様書

## GTFS-RTから取得できるすべての情報

GTFS-RTフィードは、`FeedMessage` というトップレベルのメッセージで構成されます。`FeedMessage` にはヘッダー情報 (`FeedHeader`) と、複数のエンティティ (`FeedEntity`) が含まれます。各エンティティは、TripUpdate、VehiclePosition、Alertのいずれかの情報を含みます。

### 1. フィードメッセージ (FeedMessage)

| フィールド名 | 説明 | 必須 | 型 | 実験的 | 詳細説明 |
|------------|------|------|----|-------|---------|
| header | フィードヘッダー | 必須 | FeedHeader | いいえ | このフィードメッセージに関するメタデータ。 |
| entity | フィードエンティティ | 条件付き必須 | repeated FeedEntity | いいえ | フィードの内容。リアルタイム情報が存在する場合に必須。 |

### 2. フィードヘッダー (FeedHeader)

| フィールド名 | 説明 | 必須 | 型 | 実験的 | 詳細説明 |
|------------|------|------|----|-------|---------|
| gtfs_realtime_version | GTFS-RTバージョン | 必須 | string | いいえ | フィード仕様のバージョン (例: "2.0")。 |
| incrementality | 増分性 | 必須 | enum Incrementality | いいえ | フィードが完全なデータセットか差分かを示す (0: FULL_DATASET, 1: DIFFERENTIAL)。現在はDIFFERENTIALは非推奨。 |
| timestamp | タイムスタンプ | 必須 | uint64 | いいえ | フィードが生成された時刻（UNIX時間）。 |
| feed_version | GTFSフィードバージョン | 任意 | string | いいえ | このリアルタイムデータが基づくGTFS静的データのバージョン。 |

### 3. フィードエンティティ (FeedEntity)

各エンティティは、以下のいずれか1つの情報を持ちます。

| フィールド名 | 説明 | 必須 | 型 | 実験的 | 詳細説明 |
|------------|------|------|----|-------|---------|
| id | エンティティID | 必須 | string | いいえ | フィード内で一意のエンティティ識別子。増分更新に使用。 |
| is_deleted | 削除フラグ | 任意 | bool | いいえ | エンティティが削除されるかどうか (増分更新DIFFERENTIALモードでのみ使用)。 |
| trip_update | 運行状況情報 | 条件付き必須 | TripUpdate | いいえ | 運行の遅延、キャンセル、経路変更などの情報。 |
| vehicle | 車両位置情報 | 条件付き必須 | VehiclePosition | いいえ | 車両のリアルタイム位置情報。 |
| alert | 運行障害情報 | 条件付き必須 | Alert | いいえ | 運行に関する障害情報。 |
| shape | 形状情報 | 条件付き必須 | Shape | はい |迂回などのためのリアルタイム形状情報。 |
| stop | 停留所情報 | 条件付き必須 | Stop | はい | 動的に追加された新しい停留所情報。 |
| trip_modifications | 運行変更情報 | 条件付き必須 | TripModifications | はい | 迂回など特定の変更の影響を受ける運行リスト。 |

### 4. 車両位置情報 (VehiclePosition)

| フィールド名 | 説明 | 必須 | 型 | 実験的 | 詳細説明 |
|------------|------|------|----|-------|---------|
| trip | 運行記述子 | 任意 | TripDescriptor | いいえ | この車両が運行中の運行情報。特定できない場合は空。 |
| vehicle | 車両記述子 | 任意 | VehicleDescriptor | いいえ | この車両に関する追加情報。 |
| position | 位置情報 | 任意 | Position | いいえ | 車両の現在位置。 |
| current_stop_sequence | 現在の停留所順序 | 任意 | uint32 | いいえ | 現在の停留所のインデックス (stop_times.txt の stop_sequence)。 |
| stop_id | 停留所ID | 任意 | string | いいえ | 現在の停留所のID (stops.txt の stop_id)。 |
| current_status | 現在の状態 | 任意 | enum VehicleStopStatus | いいえ | 現在の停留所に対する車両の状態 (1: INCOMING_AT, 2: STOPPED_AT, 0: IN_TRANSIT_TO)。 |
| timestamp | タイムスタンプ | 任意 | uint64 | いいえ | 車両位置の計測時刻（UNIX時間）。 |
| congestion_level | 混雑レベル | 任意 | enum CongestionLevel | いいえ | 車両に影響を与えている混雑のレベル (0: UNKNOWN_CONGESTION_LEVEL, 1: RUNNING_SMOOTHLY, 2: STOP_AND_GO, 3: CONGESTION, 4: SEVERE_CONGESTION)。 |
| occupancy_status | 乗車率ステータス | 任意 | enum OccupancyStatus | はい | 車両または車両の乗車率の状態 (0: EMPTY, 1: MANY_SEATS_AVAILABLE, 2: FEW_SEATS_AVAILABLE, 3: STANDING_ROOM_ONLY, 4: CRUSHED_STANDING_ROOM_ONLY, 5: FULL, 6: NOT_ACCEPTING_PASSENGERS, 7: NO_DATA_AVAILABLE, 8: NOT_BOARDABLE)。 |
| occupancy_percentage | 乗車率(%) | 任意 | uint32 | はい | 車両の乗車率 (%)。 |
| multi_carriage_details | 複数車両詳細 | 任意 | repeated CarriageDetails | はい | 複数車両の詳細情報。 |

### 5. 運行状況情報 (TripUpdate)

| フィールド名 | 説明 | 必須 | 型 | 実験的 | 詳細説明 |
|------------|------|------|----|-------|---------|
| trip | 運行記述子 | 必須 | TripDescriptor | いいえ | このメッセージが適用される運行。 |
| vehicle | 車両記述子 | 任意 | VehicleDescriptor | いいえ | この運行を担当する車両に関する追加情報。 |
| stop_time_update | 停留所時刻更新 | 条件付き必須 | repeated StopTimeUpdate | いいえ | 運行の停留所時刻の更新情報 (予測または実績)。trip.schedule_relationshipがCANCELED, DELETED, DUPLICATEDでない限り、最低1つ必要。 |
| timestamp | タイムスタンプ | 任意 | uint64 | いいえ | StopTime予測のために車両の進捗が最後に測定された時刻（UNIX時間）。 |
| delay | 遅延 | 任意 | int32 | はい | 運行の現在のスケジュールからの遅延（秒）。StopTimeUpdateの遅延が優先される。 |
| trip_properties | 運行プロパティ | 任意 | TripProperties | はい | 運行の更新されたプロパティ。 |

### 6. 停留所時刻更新情報 (StopTimeUpdate)

| フィールド名 | 説明 | 必須 | 型 | 実験的 | 詳細説明 |
|------------|------|------|----|-------|---------|
| stop_sequence | 停留所順序 | 条件付き必須 | uint32 | いいえ | GTFS stop_times.txt の stop_sequence。stop_id とどちらか一方は必須。 |
| stop_id | 停留所ID | 条件付き必須 | string | いいえ | GTFS stops.txt の stop_id。stop_sequence とどちらか一方は必須。 |
| arrival | 到着イベント | 条件付き必須 | StopTimeEvent | いいえ | 到着予測/実績。schedule_relationship が SCHEDULED または指定なしの場合、arrival か departure のどちらか必須。 |
| departure | 出発イベント | 条件付き必須 | StopTimeEvent | いいえ | 出発予測/実績。schedule_relationship が SCHEDULED または指定なしの場合、arrival か departure のどちらか必須。 |
| departure_occupancy_status | 出発時乗車率 | 任意 | enum OccupancyStatus | はい | 指定された停留所出発直後の車両の乗車率予測。 |
| schedule_relationship | スケジュール関係 | 任意 | enum ScheduleRelationship | いいえ | この StopTime と静的スケジュールの関係 (0: SCHEDULED (デフォルト), 1: SKIPPED, 2: NO_DATA, 3: UNSCHEDULED)。 |
| stop_time_properties | 停留所時刻プロパティ | 任意 | StopTimeProperties | はい | GTFS stop_times.txt で定義された特定のプロパティのリアルタイム更新。 |

### 7. 停留所時刻イベント (StopTimeEvent)

| フィールド名 | 説明 | 必須 | 型 | 実験的 | 詳細説明 |
|------------|------|------|----|-------|---------|
| delay | 遅延時間 | 条件付き必須 | int32 | いいえ | スケジュールからの遅延（秒）。time とどちらか一方は必須。 |
| time | 時刻 | 条件付き必須 | int64 | いいえ | イベントの絶対時刻（UNIX時間）。delay とどちらか一方は必須。 |
| uncertainty | 不確かさ | 任意 | int32 | いいえ | 予測の不確かさ（秒）。0は完全な予測。 |

### 8. 運行障害情報 (Alert)

| フィールド名 | 説明 | 必須 | 型 | 実験的 | 詳細説明 |
|------------|------|------|----|-------|---------|
| active_period | 有効期間 | 任意 | repeated TimeRange | いいえ | 障害情報が表示されるべき期間。複数指定可。 |
| informed_entity | 影響範囲 | 必須 | repeated EntitySelector | いいえ | この障害情報の影響を受けるエンティティ（運行、路線、停留所など）。最低1つ必要。 |
| cause | 原因 | 条件付き必須 | enum Cause | いいえ | 障害の原因 (1: OTHER_CAUSE, 2: TECHNICAL_PROBLEM, 3: STRIKE, 4: DEMONSTRATION, 5: ACCIDENT, 6: HOLIDAY, 7: WEATHER, 8: MAINTENANCE, 9: CONSTRUCTION, 10: POLICE_ACTIVITY, 11: MEDICAL_EMERGENCY, 0: UNKNOWN_CAUSE (デフォルト))。cause_detail が含まれる場合は必須。 |
| cause_detail | 原因詳細 | 任意 | TranslatedString | はい | 事業者固有の言語での原因説明。 |
| effect | 影響 | 条件付き必須 | enum Effect | いいえ | 障害の影響 (1: NO_SERVICE, 2: REDUCED_SERVICE, 3: SIGNIFICANT_DELAYS, 4: DETOUR, 5: ADDITIONAL_SERVICE, 6: MODIFIED_SERVICE, 7: OTHER_EFFECT, 8: UNKNOWN_EFFECT (デフォルト), 9: STOP_MOVED, 10: NO_EFFECT, 11: ACCESSIBILITY_ISSUE)。effect_detail が含まれる場合は必須。 |
| effect_detail | 影響詳細 | 任意 | TranslatedString | はい | 事業者固有の言語での影響説明。 |
| url | 詳細URL | 任意 | TranslatedString | いいえ | 障害に関する追加情報を提供するURL。 |
| header_text | 見出し | 必須 | TranslatedString | いいえ | 障害情報の見出し（プレーンテキスト）。 |
| description_text | 説明 | 必須 | TranslatedString | いいえ | 障害情報の本文（プレーンテキスト）。 |
| tts_header_text | 見出し(TTS) | 任意 | TranslatedString | いいえ | 音声合成用の見出しテキスト。 |
| tts_description_text | 説明(TTS) | 任意 | TranslatedString | いいえ | 音声合成用の説明テキスト。 |
| severity_level | 重大度 | 任意 | enum SeverityLevel | はい | 障害の重大度 (0: UNKNOWN_SEVERITY (デフォルト), 1: INFO, 2: WARNING, 3: SEVERE)。 |
| image | 画像 | 任意 | TranslatedImage | はい | 障害情報と共に表示される画像。 |
| image_alternative_text | 代替テキスト | 任意 | TranslatedString | はい | 画像の代替テキスト（アクセシビリティ用）。 |

### 9. その他 (ネストされたメッセージ型など)

上記テーブル内で参照されている複合型（メッセージ型）や列挙型（enum）の詳細については、GTFS Realtime の公式リファレンスを参照してください。主要なものを以下に示します。

*   **TripDescriptor:** 運行インスタンスを識別 (trip_id, route_id, direction_id, start_time, start_date, schedule_relationship などを含む)。
*   **VehicleDescriptor:** 車両を識別 (id, label, license_plate, wheelchair_accessible などを含む)。
*   **Position:** 地理的位置 (latitude, longitude, bearing, odometer, speed を含む)。
*   **TimeRange:** 時間範囲 (start, end を含む)。
*   **EntitySelector:** GTFSエンティティを選択 (agency_id, route_id, route_type, trip, stop_id を含む)。
*   **TranslatedString:** 多言語対応テキスト (複数の Translation を含む)。
*   **Translation:** 特定言語のテキスト (text, language を含む)。
*   **OccupancyStatus (enum):** 乗車率の状態。
*   **CongestionLevel (enum):** 混雑レベル。
*   **VehicleStopStatus (enum):** 車両の停留所での状態。
*   **ScheduleRelationship (enum):** スケジュールとの関係。
*   **Cause (enum):** Alert の原因。
*   **Effect (enum):** Alert の影響。

## 補足

- GTFS-RTはリアルタイムの運行情報を提供するプロトコルです。
- データはProtocol Buffers形式で提供されます。
- 更新頻度は事業者によって異なりますが、通常は数秒〜数十秒間隔で更新されます。
- 位置情報の精度はGPSなどの測位システムの精度に依存します。
- 遅延情報や到着予測時刻は予測値であり、実際の運行状況と異なる場合があります。
- 本仕様書はGTFS Realtime v2.0 を基準としていますが、常に最新の情報は公式リファレンス ([https://gtfs.org/realtime/reference/](https://gtfs.org/realtime/reference/)) を確認してください。
- 「実験的」カラムが「はい」のフィールドは、将来変更される可能性があります。最新の公式リファレンスでも、`shape`, `stop`, `trip_modifications`, `occupancy_status`, `occupancy_percentage`, `multi_carriage_details`, `delay` (TripUpdate), `trip_properties`, `departure_occupancy_status`, `stop_time_properties`, `cause_detail`, `effect_detail`, `severity_level`, `image`, `image_alternative_text`, `direction_id` (TripDescriptor), `modified_trip`, `wheelchair_accessible` (VehicleDescriptor), `cause_detail`, `effect_detail`, `severity_level`, `image`, `image_alternative_text` (Alert) など、多くの要素が依然として実験的 (Experimental) とされています。
- **公式リファレンスとの主な差分 (2024年5月時点、主に実験的な要素):**
    - 公式リファレンスでは、各フィールドの必須条件 (Required, Conditionally required, Optional) がより詳細に定義されています。
    - `VehiclePosition` に `WheelchairAccessible` (enum) が実験的に追加されています (本仕様書にも追記)。
    - `StopTimeUpdate` の `ScheduleRelationship` (enum) に `UNSCHEDULED` が実験的に追加されています (本仕様書では記載済み)。
    - 停留所の動的割り当て (`StopTimeProperties` 内 `assigned_stop_id`) や運行プロパティの更新 (`TripProperties`) に関する実験的なフィールドが追加・更新されています (本仕様書にも追記)。
    - 迂回などを扱う Trip Modifications 関連のメッセージ (`TripModifications`, `Modification` 等) が実験的に追加・詳細化されています (本仕様書にも追記)。
    - `Stop` メッセージ (`FeedEntity`内) が実験的に追加され、`wheelchair_boarding` (enum) フィールドが含まれています (本仕様書にも追記)。
    - `Alert` に画像関連のフィールド (`TranslatedImage`, `LocalizedImage`, `image_alternative_text` 等) が実験的に追加されています (本仕様書にも追記)。 