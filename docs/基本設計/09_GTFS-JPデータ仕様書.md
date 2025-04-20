# GTFS-JP データ仕様書

## 概要

GTFS-JPは、公共交通機関の静的な情報（時刻表、経路、運賃など）を記述するための標準フォーマットです。GTFS (General Transit Feed Specification) をベースに、日本のバス事業者向けに拡張された仕様です。

データは複数のテキストファイル（.txt）で構成され、ZIPファイルにまとめて提供されます。

## ファイル構成

### 必須ファイル

これらのファイルは、GTFS-JPフィードを構成するために最低限必要なファイルです。

*   `agency.txt`: 事業者情報
*   `stops.txt`: 停留所情報
*   `routes.txt`: 経路情報
*   `trips.txt`: 運行情報
*   `stop_times.txt`: 時刻表情報
*   `calendar.txt` または `calendar_dates.txt`: 運行日情報（どちらか一方、または両方が必須）

### オプションファイル

これらのファイルは、より詳細な情報を提供するために任意で追加できます。

*   `calendar_dates.txt`: 運行日の例外情報
*   `fare_attributes.txt`: 運賃属性情報
*   `fare_rules.txt`: 運賃規則情報
*   `shapes.txt`: 経路形状情報
*   `frequencies.txt`: 高頻度運行情報
*   `transfers.txt`: 乗換情報
*   `feed_info.txt`: フィード情報
*   `translations.txt`: 翻訳情報
*   `office_jp.txt`: 営業所情報 (GTFS-JP拡張)

## ファイル仕様詳細

### 1. 事業者情報 (agency.txt)

交通事業者に関する情報を提供します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| agency_id | 事業者ID | 条件付き必須 | ID | 複数の事業者を定義する場合に必須。フィード内で一意。 | "A001" |
| agency_name | 事業者名 | 必須 | Text | 事業者の正式名称。 | "サンプル交通" |
| agency_url | 事業者URL | 必須 | URL | 事業者のウェブサイトURL。 | "https://example.com" |
| agency_timezone | タイムゾーン | 必須 | Timezone | 事業者のタイムゾーン (TZ Database Name)。 | "Asia/Tokyo" |
| agency_lang | 言語 | 任意 | Language | 事業者の主要言語 (ISO 639-1)。 | "ja" |
| agency_phone | 電話番号 | 任意 | Phone Number | 事業者の問い合わせ電話番号。 | "03-1234-5678" |
| agency_fare_url | 運賃情報URL | 任意 | URL | 運賃に関する情報を提供するURL。 | "https://example.com/fare" |
| agency_email | メールアドレス | 任意 | Email | 事業者の問い合わせメールアドレス。 | "info@example.com" |

### 2. 停留所情報 (stops.txt)

個々の停留所の位置や名前などの情報を提供します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| stop_id | 停留所ID | 必須 | ID | 停留所を一意に識別するID。 | "S001" |
| stop_code | 停留所コード | 任意 | Text | 停留所の短縮コードまたは番号。 | "T01" |
| stop_name | 停留所名 | 条件付き必須 | Text | 停留所の名称。`location_type` が 0 または 1 の場合は必須。 | "サンプル駅前" |
| tts_stop_name | 停留所名(TTS) | 任意 | Text | 音声合成用の停留所名。 | "サンプルえきまえ" |
| stop_desc | 停留所説明 | 任意 | Text | 停留所の補足説明。 | "北口ロータリー" |
| stop_lat | 緯度 | 条件付き必須 | Float | 停留所の緯度 (WGS84)。`location_type` が 0 または 1 の場合は必須。 | 35.681236 |
| stop_lon | 経度 | 条件付き必須 | Float | 停留所の経度 (WGS84)。`location_type` が 0 または 1 の場合は必須。 | 139.767125 |
| zone_id | ゾーンID | 任意 | ID | 運賃計算に使用するゾーンID。 | "Z01" |
| stop_url | 停留所URL | 任意 | URL | 停留所の詳細情報を提供するURL。 | "https://example.com/stop/S001" |
| location_type | 位置タイプ | 任意 | Enum | 停留所の種類 (0: 停留所, 1: 駅/ターミナル, 2: 入り口/出口, 3: 汎用ノード, 4: 乗降場)。デフォルトは 0。 | 1 |
| parent_station | 親駅ID | 任意 | ID | この停留所が属する駅/ターミナルの`stop_id` (`location_type`=1)。 | "S000" |
| stop_timezone | タイムゾーン | 任意 | Timezone | 停留所のタイムゾーン。指定がない場合は`agency.txt`の値を使用。 | "Asia/Tokyo" |
| wheelchair_boarding | 車椅子乗降 | 任意 | Enum | 車椅子での乗降可否 (0: 情報なし, 1: 可能, 2: 不可)。 | 1 |
| level_id | 階層ID | 任意 | ID | 停留所が存在する階層 (`levels.txt`の`level_id`)。 | "L1" |
| platform_code | プラットフォームコード | 任意 | Text | 停留所のプラットフォーム番号/名称。 | "3番のりば" |

### 3. 経路情報 (routes.txt)

運行経路に関する情報を提供します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| route_id | 経路ID | 必須 | ID | 経路を一意に識別するID。 | "R001" |
| agency_id | 事業者ID | 任意 | ID | この経路を運行する事業者のID (`agency.txt`の`agency_id`)。 | "A001" |
| route_short_name | 経路略称 | 条件付き必須 | Text | 経路の短い名称。`route_long_name` とどちらか一方は必須。 | "7" |
| route_long_name | 経路名称 | 条件付き必須 | Text | 経路の完全な名称。`route_short_name` とどちらか一方は必須。 | "駅前 - 市役所線" |
| route_desc | 経路説明 | 任意 | Text | 経路の補足説明。 | "循環路線" |
| route_type | 経路タイプ | 必須 | Enum | 交通手段の種類 (3: バス)。GTFS-JPでは通常3。 | 3 |
| route_url | 経路URL | 任意 | URL | 経路の詳細情報を提供するURL。 | "https://example.com/route/R001" |
| route_color | 経路色 | 任意 | Color | 経路を表す色 (6桁の16進数)。 | "FFFFFF" |
| route_text_color | 経路文字色 | 任意 | Color | 経路名の文字色 (6桁の16進数)。 | "000000" |
| route_sort_order | 経路表示順 | 任意 | Integer (Non-negative) | 経路リストでの表示順序。 | 1 |
| continuous_pickup | 連続乗車 | 任意 | Enum | 連続的な乗車ポリシー (0: 連続乗車可能, 1: 不可, 2: 電話連絡要, 3: ドライバー連絡要)。 | 0 |
| continuous_drop_off | 連続降車 | 任意 | Enum | 連続的な降車ポリシー (0: 連続降車可能, 1: 不可, 2: 電話連絡要, 3: ドライバー連絡要)。 | 0 |

### 4. 運行情報 (trips.txt)

特定の経路における一連の運行（トリップ）を定義します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| route_id | 経路ID | 必須 | ID | この運行が属する経路のID (`routes.txt`の`route_id`)。 | "R001" |
| service_id | サービスID | 必須 | ID | この運行の運行日パターンを示すID (`calendar.txt`または`calendar_dates.txt`の`service_id`)。 | "S001" |
| trip_id | 運行ID | 必須 | ID | 運行を一意に識別するID。 | "T001" |
| trip_headsign | 行先表示 | 任意 | Text | 乗客向けの行先表示。 | "市役所方面" |
| trip_short_name | 運行略称 | 任意 | Text | 運行の内部的な短い名称。 | "T1" |
| direction_id | 方向ID | 任意 | Enum | 経路内での運行方向 (0: 往路, 1: 復路)。 | 0 |
| block_id | ブロックID | 任意 | ID | 同一車両で連続して運行される場合のブロックID。 | "B001" |
| shape_id | 形状ID | 任意 | ID | この運行の経路形状を示すID (`shapes.txt`の`shape_id`)。 | "Sh001" |
| wheelchair_accessible | 車椅子利用 | 任意 | Enum | この運行での車椅子利用可否 (0: 情報なし, 1: 可能, 2: 不可)。 | 1 |
| bikes_allowed | 自転車持込 | 任意 | Enum | この運行での自転車持込可否 (0: 情報なし, 1: 可能, 2: 不可)。 | 0 |
| trip_desc | 運行摘要 | 任意 | Text | 運行の摘要。 | "平日のみ運行" |
| trip_desc_symbol | 運行摘要記号 | 任意 | Text | 運行の摘要を表す記号。 | "平" |

### 5. 時刻表情報 (stop_times.txt)

各運行が各停留所に到着・出発する時刻を定義します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| trip_id | 運行ID | 必須 | ID | この時刻情報が属する運行のID (`trips.txt`の`trip_id`)。 | "T001" |
| arrival_time | 到着時刻 | 条件付き必須 | Time | 停留所への到着時刻 (HH:MM:SS)。`timepoint`=1の場合は必須。 | "08:00:00" |
| departure_time | 出発時刻 | 条件付き必須 | Time | 停留所からの出発時刻 (HH:MM:SS)。`timepoint`=1の場合は必須。 | "08:01:00" |
| stop_id | 停留所ID | 必須 | ID | この時刻情報が属する停留所のID (`stops.txt`の`stop_id`)。 | "S001" |
| stop_sequence | 停留所順序 | 必須 | Integer (Non-negative) | 運行内での停留所の通過順序。昇順である必要あり。 | 1 |
| stop_headsign | 停留所行先表示 | 任意 | Text | この停留所での行先表示。 | "市役所" |
| pickup_type | 乗車タイプ | 任意 | Enum | 乗車ポリシー (0: 通常可能, 1: 不可, 2: 電話連絡要, 3: ドライバー連絡要)。 | 0 |
| drop_off_type | 降車タイプ | 任意 | Enum | 降車ポリシー (0: 通常可能, 1: 不可, 2: 電話連絡要, 3: ドライバー連絡要)。 | 0 |
| continuous_pickup | 連続乗車 | 任意 | Enum | `routes.txt`の値を上書きする場合に指定。 | 1 |
| continuous_drop_off | 連続降車 | 任意 | Enum | `routes.txt`の値を上書きする場合に指定。 | 1 |
| shape_dist_traveled | 形状移動距離 | 任意 | Float (Non-negative) | 経路形状に沿った始点からの移動距離。 | 150.5 |
| timepoint | 主要停留所 | 任意 | Enum | 時刻の正確性を示すフラグ (0: 不正確, 1: 正確)。デフォルトは 1。 | 1 |

### 6. 運行日情報 (calendar.txt)

週ごとの定期的な運行パターンを定義します。
`calendar.txt` または `calendar_dates.txt` のどちらか一方、または両方が必要です。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| service_id | サービスID | 必須 | ID | この運行日パターンを一意に識別するID。 | "S001" |
| monday | 月曜日 | 必須 | Enum (0 or 1) | 月曜日に運行するかどうか (1: 運行, 0: 運休)。 | 1 |
| tuesday | 火曜日 | 必須 | Enum (0 or 1) | 火曜日に運行するかどうか。 | 1 |
| wednesday | 水曜日 | 必須 | Enum (0 or 1) | 水曜日に運行するかどうか。 | 1 |
| thursday | 木曜日 | 必須 | Enum (0 or 1) | 木曜日に運行するかどうか。 | 1 |
| friday | 金曜日 | 必須 | Enum (0 or 1) | 金曜日に運行するかどうか。 | 1 |
| saturday | 土曜日 | 必須 | Enum (0 or 1) | 土曜日に運行するかどうか。 | 0 |
| sunday | 日曜日 | 必須 | Enum (0 or 1) | 日曜日に運行するかどうか。 | 0 |
| start_date | 開始日 | 必須 | Date | このサービスが有効になる開始日 (YYYYMMDD)。 | "20230401" |
| end_date | 終了日 | 必須 | Date | このサービスが有効になる終了日 (YYYYMMDD)。 | "20240331" |

### 7. 運行日例外情報 (calendar_dates.txt)

特定の日付における運行の追加・削除を定義します。
`calendar.txt` または `calendar_dates.txt` のどちらか一方、または両方が必要です。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| service_id | サービスID | 必須 | ID | 例外を適用するサービスのID (`calendar.txt`の`service_id`)。 | "S001" |
| date | 日付 | 必須 | Date | 例外を適用する日付 (YYYYMMDD)。 | "20230505" |
| exception_type | 例外タイプ | 必須 | Enum | 例外の種類 (1: 運行追加, 2: 運休)。 | 2 |

### 8. 運賃属性情報 (fare_attributes.txt) - オプション

運賃に関する基本情報を提供します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| fare_id | 運賃ID | 必須 | ID | 運賃を一意に識別するID。 | "F001" |
| price | 価格 | 必須 | Float (Non-negative) | 運賃の金額。 | 220.0 |
| currency_type | 通貨 | 必須 | Currency Code | 通貨コード (ISO 4217)。 | "JPY" |
| payment_method | 支払方法 | 必須 | Enum | 支払方法 (0: 乗車時, 1: 事前)。 | 0 |
| transfers | 乗換 | 必須 | Enum (0, 1, 2, ...) | 乗換可能回数 (0: 不可, 1: 1回, 2: 2回, ...)。空欄は無制限。 | 0 |
| agency_id | 事業者ID | 任意 | ID | 運賃を定義する事業者のID (`agency.txt`の`agency_id`)。 | "A001" |
| transfer_duration | 乗換有効時間 | 任意 | Integer (Non-negative) | 乗換が有効な時間（秒）。 | 3600 |

### 9. 運賃規則情報 (fare_rules.txt) - オプション

特定の経路や区間にどの運賃が適用されるかを定義します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| fare_id | 運賃ID | 必須 | ID | 適用する運賃のID (`fare_attributes.txt`の`fare_id`)。 | "F001" |
| route_id | 経路ID | 任意 | ID | 運賃が適用される経路のID (`routes.txt`の`route_id`)。 | "R001" |
| origin_id | 出発ゾーンID | 任意 | ID | 運賃が適用される出発ゾーンID (`stops.txt`の`zone_id`)。 | "Z01" |
| destination_id | 到着ゾーンID | 任意 | ID | 運賃が適用される到着ゾーンID (`stops.txt`の`zone_id`)。 | "Z02" |
| contains_id | 経由ゾーンID | 任意 | ID | 運賃が適用される経由ゾーンID (`stops.txt`の`zone_id`)。 | "Z03" |

### 10. 経路形状情報 (shapes.txt) - オプション

運行経路の地理的な形状を定義します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| shape_id | 形状ID | 必須 | ID | 経路形状を一意に識別するID。 | "Sh001" |
| shape_pt_lat | 形状点緯度 | 必須 | Float | 形状点の緯度 (WGS84)。 | 35.681236 |
| shape_pt_lon | 形状点経度 | 必須 | Float | 形状点の経度 (WGS84)。 | 139.767125 |
| shape_pt_sequence | 形状点順序 | 必須 | Integer (Non-negative) | 形状内での点の順序。昇順である必要あり。 | 1 |
| shape_dist_traveled | 形状移動距離 | 任意 | Float (Non-negative) | 形状に沿った始点からの移動距離。 | 150.5 |

### 11. 高頻度運行情報 (frequencies.txt) - オプション

正確な時刻表ではなく、運行間隔で定義される高頻度運行の情報を提供します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| trip_id | 運行ID | 必須 | ID | この運行間隔が適用される運行のID (`trips.txt`の`trip_id`)。 | "T001" |
| start_time | 開始時刻 | 必須 | Time | この運行間隔が適用される開始時刻 (HH:MM:SS)。 | "06:00:00" |
| end_time | 終了時刻 | 必須 | Time | この運行間隔が適用される終了時刻 (HH:MM:SS)。 | "22:00:00" |
| headway_secs | 運行間隔 | 必須 | Integer (Positive) | この時間帯の運行間隔（秒）。 | 600 |
| exact_times | 正確な時刻 | 任意 | Enum (0 or 1) | 運行間隔が厳密かどうか (0: 不正確, 1: 正確)。 | 0 |

### 12. 乗換情報 (transfers.txt) - オプション

停留所間の乗換ルールを定義します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| from_stop_id | 乗換元停留所ID | 必須 | ID | 乗換元の停留所ID (`stops.txt`の`stop_id`)。 | "S001" |
| to_stop_id | 乗換先停留所ID | 必須 | ID | 乗換先の停留所ID (`stops.txt`の`stop_id`)。 | "S101" |
| transfer_type | 乗換タイプ | 必須 | Enum | 乗換のタイプ (0: 推奨, 1: 時間指定, 2: 最低限時間, 3: 不可)。 | 2 |
| min_transfer_time | 最低乗換時間 | 任意 | Integer (Non-negative) | `transfer_type`=2の場合の最低乗換時間（秒）。 | 300 |
| from_route_id | 乗換元経路ID | 任意 | ID | 乗換元の経路ID。 | "R001" |
| to_route_id | 乗換先経路ID | 任意 | ID | 乗換先の経路ID。 | "R101" |
| from_trip_id | 乗換元運行ID | 任意 | ID | 乗換元の運行ID。 | "T001" |
| to_trip_id | 乗換先運行ID | 任意 | ID | 乗換先の運行ID。 | "T101" |

### 13. フィード情報 (feed_info.txt) - オプション

GTFSデータフィード自体のメタ情報を提供します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| feed_publisher_name | フィード発行者名 | 必須 | Text | フィードを発行する組織の名称。 | "サンプル交通" |
| feed_publisher_url | フィード発行者URL | 必須 | URL | フィードを発行する組織のウェブサイトURL。 | "https://example.com" |
| feed_lang | フィード言語 | 必須 | Language | フィードで使用される主要言語 (ISO 639-1)。 | "ja" |
| default_lang | デフォルト言語 | 任意 | Language | `translations.txt` が提供される場合のデフォルト言語。 | "ja" |
| feed_start_date | フィード有効開始日 | 任意 | Date | このフィードが有効な期間の開始日 (YYYYMMDD)。 | "20230401" |
| feed_end_date | フィード有効終了日 | 任意 | Date | このフィードが有効な期間の終了日 (YYYYMMDD)。 | "20240331" |
| feed_version | フィードバージョン | 任意 | Text | フィードのバージョンを示す文字列。 | "1.0.0" |
| feed_contact_email | フィード連絡先Email | 任意 | Email | フィードに関する連絡先メールアドレス。 | "gtfs@example.com" |
| feed_contact_url | フィード連絡先URL | 任意 | URL | フィードに関する連絡先ウェブサイトURL。 | "https://example.com/contact" |

### 14. 翻訳情報 (translations.txt) - オプション

フィード内の特定の値を多言語に翻訳する情報を提供します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| table_name | テーブル名 | 必須 | Enum | 翻訳対象のフィールドが含まれるテーブル名。 | "stops" |
| field_name | フィールド名 | 必須 | Text | 翻訳対象のフィールド名。 | "stop_name" |
| language | 言語コード | 必須 | Language | 翻訳後の言語 (ISO 639-1 または BCP 47)。 | "en" |
| translation | 翻訳 | 必須 | Text | 翻訳されたテキスト。 | "Sample Sta." |
| record_id | レコードID | 条件付き必須 | ID | 翻訳対象のレコードを一意に示すID (`*_id`)。`record_sub_id` と合わせて使用する場合がある。 | "S001" |
| record_sub_id | レコードサブID | 任意 | ID | 翻訳対象レコードをさらに絞り込むためのID (例: `stop_times`の`stop_sequence`)。 | 1 |
| field_value | フィールド値 | 条件付き必須 | Text | 翻訳対象の元のフィールド値。`record_id` が指定できない場合に使用。 | "サンプル駅前" |

### 15. 営業所情報 (office_jp.txt) - オプション (GTFS-JP拡張)

事業者の営業所に関する情報を提供します。

| フィールド名 | 説明 | 必須 | 型 | 詳細説明 | 値の例 |
|------------|------|------|----|---------|-------|
| office_id | 営業所ID | 必須 | ID | 営業所を一意に識別するID。 | "O001" |
| office_name | 営業所名 | 必須 | Text | 営業所の名称。 | "サンプル営業所" |
| office_url | 営業所URL | 任意 | URL | 営業所のウェブサイトURL。 | "https://example.com/office/O001" |
| office_phone | 営業所電話番号 | 任意 | Phone Number | 営業所の電話番号。 | "03-9876-5432" |

## 補足

- 本仕様書は「標準的なバス情報フォーマット 第3版」(2021年7月30日 国土交通省) を主に参考にしています。
- 最新かつ完全な情報は、国土交通省のウェブサイトや関連ドキュメント、およびGTFS Schedule公式リファレンスを参照してください。
- 「型」カラムはGTFS仕様で定義されているデータ型を示します (ID, Text, URL, Enum, Float, Integer, Date, Time, Color, Timezone, Language, Phone Number, Email, Currency Code)。
- 必須フラグはGTFS仕様に基づきます。「条件付き必須」はそのフィールドが必須となる条件があることを示します。
- GTFS-JP独自の拡張ファイルとして `office_jp.txt` があります。これらGTFS-JP独自の拡張は、公式のGTFS Schedule仕様には含まれません。
- **GTFS Schedule仕様の更新について:** 本仕様書のベースとなっているGTFS Schedule仕様は、[https://gtfs.org/](https://gtfs.org/) で継続的に更新されています。特に、本仕様書作成後に以下の大きな更新がありました（ただし、これらは「標準的なバス情報フォーマット」にはまだ反映されていない可能性があります）。
    - **Fares v2:** より柔軟な運賃体系を表現するための新しいファイル群 (`fare_media.txt`, `fare_products.txt`, `fare_leg_rules.txt`, `fare_transfer_rules.txt`, `timeframes.txt`, `rider_categories.txt` 等) が導入されました。本仕様書は従来の Fares v1 (`fare_attributes.txt`, `fare_rules.txt`) のみを記載しており、これらは **Legacy Fares (v1)** として扱われています。
    - **Flex v2 (On-Demand):** デマンド型交通サービス（予約制サービスなど）を表現するための新しいファイル群 (`locations.geojson`, `booking_rules.txt`) や関連フィールドが導入されました。
    - **その他の追加・更新:**
        - ネットワーク情報 (`networks.txt`, `route_networks.txt`)
        - 提供元情報 (`attributions.txt`)
        - エリア情報 (`areas.txt`, `stop_areas.txt`)
        - 場所グループ (`location_groups.txt`, `location_group_stops.txt`)
        - その他、既存ファイルのフィールド追加・更新などが行われています。
- 本仕様書は上記の最新のGTFS Schedule仕様の変更点を網羅していません。常に最新のGTFS Schedule仕様については、公式リファレンス ([https://gtfs.org/schedule/reference/](https://gtfs.org/schedule/reference/)) を確認してください。 