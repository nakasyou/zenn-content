---
title: "大阪万博 API 解説" # 記事のタイトル
emoji: "🫀" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["expo", "osakaexpo"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

大阪万博が昨日閉幕したので、大阪万博の予約サイトの API について自分が調べたことを公開しようと思います。
数日前にタイムスリップした気持ちでお読みください。

また、脆弱だと思ったポイントもあるので、閉幕したのでそれも解説しちゃいます。

## 想定読者

* 万博のアプリの裏側をちょっと覗いてみたい人
* 大阪万博の脆弱っぽい部分をちょっと知りたい人
* 閉幕してしまったけれど自分がどこに行ったかを思い出したい人
* 次日本で万博が開催されたときにアプリを開発する人、および発注する人

## tl;dr

成果物: https://github.com/pnsk-lab/myakumyakujs

## 認証の仕様

### 二段階認証

大阪万博では、二段階認証が有効化されています。しかし、16 歳未満のアカウントは二段階認証を有効化する必要がないようで、そのおかげで楽に API をたたくことができました。

### 認証方法

認証に必要なページは待合室で保護されているので、その URL を得るのに待合室を通過する必要があります。

まず、ログインページの URL は動的に変化するので、その URL を取得してあげる必要があります。
`https://ticket.expo2025.or.jp/api/d/expo_login` というエンドポイントにアクセスすると、ログインページの URL にリダイレクトされます。

```ts
const redirectPage = await fetch('https://ticket.expo2025.or.jp/api/d/expo_login', {
  redirect: 'manual'
})
const loginPageUrl = redirectPage.headers.get('Location')
```

次に、ログインページの HTML を取得して、そこから `<form>` タグの `action` 属性を取得します。

```ts
const loginPage = await fetch(loginPageUrl, {
  redirect: 'manual'
})
const targetUrl = (await loginPage.text()).match(/(?<=action=")[^"]+(?=")/)?.[0].replaceAll('amp;', '')
```

最後に、`targetUrl` にメールアドレスとパスワードを POST します。すればいいのですが、ここで気を付けなければならないポイントがあります。それは Cookie です。先ほどのログインページとリダイレクトページの Cookie を引き継いであげる必要があります。

また、この画面では待合室が有効化されているので、リダイレクトを手動で処理してあげる必要があります。

https://github.com/pnsk-lab/myakumyakujs/blob/a0c4fc8a4257455b3d21a355700dc8f3dec68729/src/auth.ts#L29-L59

こんな感じで while ループを回して、リダイレクトが終了するまで待ちます。リクエストは `application/x-www-form-urlencoded` で送信します。username, password のほかに credentialId という謎のパラメーターがありますが、それは空文字列でいけました。

ここで取得しておくべき Cookie は、`session_id` のみです。なぞに AUTH_SESSION_ID みたいな Cookie や、 SESSION みたいな Cookie も発行されますが、これらは無視して大丈夫だった記憶です。

## チケットの仕様

### 取得方法

万博のチケットの仕様を解説します。

`https://ticket.expo2025.or.jp/api/d/my/tickets/?count=1` に GET を Cookie つきで送るだけです。簡単ですね。
返ってくるのは JSON で、チケットの配列です。

### チケットの例

```json
{
    "id": 12345678, // チケットの数値 ID
    "ticket_id": "AAAAAAAAAAA", // チケット ID
    "agent_code": "0010", // 謎
    "item_group_code": "0012", // 謎
    "item_code": "00452", // 謎
    "item_group_order": 19, // 謎
    "item_order": 2, // 謎
    "simple_ticket_id": "AAAAAAAAAAA", // 何が違うん
    "disp_status": 3, // 謎
    "item_group_name": "Summer Pass", // 夏パスだったのでこれ
    "item_name": "Summer Pass", // 同上
    "item_abb_name": "12-17", // 12-17 歳用のチケットだったのでこれ
    "item_summary": "Youth (aged from 12 to 17)", // 同上
    "image_large_path": "/tickethub_file/images/0433/00433/0010/0/item_large_image/9b3cc94808d9c04aee090190ab0123c398b8.gif",　// 動く夏パスの画像
    "order_number": null, // 謎
    "receive_type": 2, // 謎
    "received_at": "2025-07-15 16:59:16 +0900", // たぶんチケットを受け取った日時
    "schedules": [
        {
            "user_visiting_reservation_id": 1234567, // 入場予約 ID じゃない？
            "use_state": 1, // 謎
            "entrance_date": "20250812", // 入場日
            "gate_type": 2, // たぶん入場ゲートの種類。東か西か
            "schedule_name": "10:00-", // 何時から入場できるか
            "start_time": "0900", // 上と何が違うんだ
            "proxy_reserve": true, // 代理予約かどうか?
            // 以下は不明
            "group_ticket_qr_divi": 0,
            "admission_time": "104227",
            "admission_buf_during": false,
            "empty_frame": false,
            "on_the_day": true,
            "month_lottery": false,
            "day_lottery": false,
            "lotteries": { "month": [], "day": [] }
        },
        // ... 他の日の入場予約も同様の形式で入る
    ],
    "event_schedules": [
      {
        "id": 1234567, // 予約 ID かも
        "schedule_code": "2025081706HOH0",
        "schedule_no": "06",
        "schedule_name": "11:40-12:00",
        "program_code": "HOH0",
        "entrance_date": "20250817",
        "start_time": "1140",
        "end_time": "1200",
        "suspend_divi": 0,
        "updated_at": "2025-10-14 02:10:56 +0900",
        "created_at": "2024-09-19 21:15:51 +0900",
        "event_code": "HOH0",
        "event_name": "BLUE OCEAN DOME",
        "event_summary": "The theme of Blue Ocean Dome is “Reviving the Ocean”.With the “consciousness transforming devices” that will cause an attitude change toward the earth and oceans,The Blue Ocean Dome will offer an unprecedented experience to visitors through various events related to the sustainable use of the oceans in the exhibition space .",
        "virtual_url": "https://contents.ssv.virtualexpo.expo2025.or.jp/deeplink/cushion_page.html?SpaceId=SS-108932",
        "virtual_url_desc": "Click here for the Virtual Expo",
        "portal_url": "https://www.zeri.jp/expo2025/",
        "portal_url_desc": "Click here for details",
        "ticket_id": "ADFGHJJKL",
        "registered_channel": 5,
        "proxy_reserve": false,
        "use_state": 1,
        "group_ticket_qr_divi": 0,
        "program_ticket_state": 1,
        "admission_time": "114452",
        "admission_buf_during": false
      }
      // ... 他のイベントも同様の形式で入る
    ],
    "ticket_type_id": "22",
    "change_reservation": 0,
    "fast_lottery": false,
    "lotteries": { "fast": [] },
    "adult_type": true,
    "disp_bid_status": 2
}
```

ちなみにこの `image_large_path` はこれです、かわいいですね。
![夏パス](https://ticket.expo2025.or.jp/tickethub_file/images/0433/00433/0010/0/item_large_image/9b3cc94808d9c04aee090190ab0123c398b8.gif)

あと、この API は閉幕後の今でも動いています。止まる前に思い出を取っておきたい人は今のうちに JSON を保存しておきましょう。

## イベント情報の仕様

認証方法なんてどうでもよくて、本題はここからです。

まず、"イベント" についてです。この "イベント" というのは、各パビリオンで行われる入場方法を意味します。
例えば、
```
電力館
  |-- 通常入場
  |-- 車いす入場
null²
  |-- インスタレーションモード
  |-- ダイアログモード
  |-- ウォークスルーモード
```
のように各パビリオンは異なる形式を提供しますが、それぞれ別の "イベント" として扱われます。

### イベントの取得 API

イベントの取得 API は `https://ticket.expo2025.or.jp/api/d/events` です。これに認証済みの Cookie をつけて GET します。

URL パラメーターは以下の通りです。

* `ticket_ids[]`: カンマ区切りでチケット ID を指定します。
  * 例: `T1CKET,T2CKET`
  * このパラメーターは個人的にうわってなったランキング上位です。
* `entrance_date`: 入場日を指定します。`YYYYMMDD` 形式の文字列です。
  * 例: `20250413`
* `count`: これはなにか忘れました。 `1` を指定しておけばよいと思います。
* `limit`: 取得する最大件数だったと思います。例: `20`
* `event_type`: これはわからなかったか忘れました。`0` を指定しておけばよいと思います。
* `channel`: チャンネル。後で解説します。
* `event_name`: これは必須ではありません。これ event_name っていう名前しておきながら検索クエリなんですよ？ query とかにすべきだったと思います。

例えばすべてのイベントを `ReadableStream` で返す関数は以下のようにできます:

https://github.com/pnsk-lab/myakumyakujs/blob/74594d793df638f7cd168d9cbb4c860b9ba8046b/src/events.ts#L96-L196

### `channel` について

channel は予約の方法を表します。以下の enum を見てもらえばいいと思います

- 0: 来場日時予約（指定した日時の予約）
- 1: 超早割特別抽選
- 2: 2か月前抽選
- 3: 7日前抽選
- 4: 空き枠先着予約 (いわゆる 3日前予約)
- 5: 当日登録

それぞれの値は予約方法の違いを表しています。

また、後ほど書くのですが、なんと 3 日前予約の情報を 3 日前以前から取得できてしまうのですw

### イベントが持つ状態

チャンネルについて説明します。

ここでは null² のナオライモードのイベントを例にします。

```json
{
  "id": 49420,
  "event_code": "IC0C",
  "event_name": "Signature Pavilion OCHIAI Yoichi 'null²' Naorai Mode *10 min before the reservation start time",
  "program_code": "IC0C",
  "event_summary": "［AverageDuration:10min］［10 min before the reservation start time］Naorai Mode – Farewell to the Forest of NullThis 10-minute ceremony is the final Naorai shared by the pavilion and its visitors. Soon, null² will return to Nul, and we celebrate its departure through a feast. In a space that shifts from the solemn resonance of a Buddhist indō (ritual of farewell) to the lively rhythms of festival applause and cheers, all participants become one community, sharing “goodbye” and “thank you.” It is not an end, but a prayer and a hospitable rite for a new beginning.",
  "virtual_url": "https://contents.ssv.virtualexpo.expo2025.or.jp/deeplink/cushion_page.html?SpaceId=SS-664929",
  "virtual_url_desc": "Click here for Virtual Expo ",
  "portal_url": "https://expo2025.digitalnatureandarts.or.jp/specialsite.html",
  "portal_url_desc": "Click here for details",
  "date_status": 2
}
```

それぞれ解説していきますね。

* `id`: イベントの数値 ID です。使われているところを見たことがないです。
* `event_code`: イベントコードです。API でイベントを指定するときに使います。
* `event_name`: イベントのタイトルです。
* `program_code`: これは謎です。イベントコードと異なる値が入っていることを見たことがないです。
* `event_summary`: イベントの説明です。
* `virtual_url`: VR の大阪万博の URL です。
* `virtual_url_desc`: VR の大阪万博のリンクテキストです。これわざわざ API で返す必要ある？
* `portal_url`: 大阪万博のポータルサイトの URL です。
* `portal_url_desc`: ポータルサイトのリンクテキストです。これもわざわざ API で返す必要ある？
* `date_status`: これはイベントの空き状態です。
  * `0`: すいてる
  * `1`: もうすぐ埋まる
  * `2`: 予約不可

### イベントの詳細な空き情報

以上の例では一日の中で空きがあるかどうかしかわかりませんが、実際には時間帯ごとの空き情報も取得できます。

エンドポイントは `https://ticket.expo2025.or.jp/api/d/events/:event_code` です。`:event_code` の部分にはイベントコードを入れます。
また、パラメータは以下の通りです。
* `ticket_ids[]`: カンマ区切りでチケット ID を指定
* `entrance_date`: 入場日を `YYYYMMDD` 形式で指定
* `channel`: 予約方法を指定

例えば、閉幕日である 10/13 の null² のナオライモードの空き情報の結果は以下のような JSON です:
```json
{
  "event_code": "IC0C",
  "event_name": "Signature Pavilion OCHIAI Yoichi 'null²' Naorai Mode *10 min before the reservation start time",
  "event_summary": "［AverageDuration:10min］［10 min before the reservation start time］Naorai Mode – Farewell to the Forest of NullThis 10-minute ceremony is the final Naorai shared by the pavilion and its visitors. Soon, null² will return to Nul, and we celebrate its departure through a feast. In a space that shifts from the solemn resonance of a Buddhist indō (ritual of farewell) to the lively rhythms of festival applause and cheers, all participants become one community, sharing “goodbye” and “thank you.” It is not an end, but a prayer and a hospitable rite for a new beginning.",
  "image_large_path": null, // 謎
  "image_small_path": null,
  "thumbnail_image_path": "/tickethub_file/images/0433/00433/0010/0/item_thumbnail_image/5507accb056d6042cb08f7204976029dd089.png",
  "virtual_url": "https://contents.ssv.virtualexpo.expo2025.or.jp/deeplink/cushion_page.html?SpaceId=SS-664929",
  "virtual_url_desc": "Click here for Virtual Expo ",
  "portal_url": "https://expo2025.digitalnatureandarts.or.jp/specialsite.html",
  "portal_url_desc": "Click here for details",
  "event_schedules": {
    // ...
    "1820": {
      "schedule_code": "2025101301IC0C",
      "schedule_no": "01", // 謎
      "schedule_name": "18:20-18:30", // 予約時間帯
      "program_code": "IC0C", // コード、ここに入れる必要ある？
      "entrance_date": "20251013", // これも必要ある？
      "start_time": "1820", // 予約時間帯の開始時間
      "end_time": "1830", // 予約時間帯の終了時間
      "time_status": 2, // さっきと同じ。0: すいてる, 1: もうすぐ埋まる, 2: 予約不可
      "unavailable_reason": 1 // 解析してない
    },
    "1830": {
      "schedule_code": "2025101302IC0C",
      "schedule_no": "02",
      "schedule_name": "18:30-18:40",
      "program_code": "IC0C",
      "entrance_date": "20251013",
      "start_time": "1830",
      "end_time": "1840",
      "time_status": 2,
      "unavailable_reason": 1
    }
  },
  "reserve_now": 1 // 謎
}
``` 

### イベントの予約 API

さて、もしみなさんが万博開催中にタイムスリップしたとして一番欲しいのは予約 API ですよね (意味深)

エンドポイントは、`https://ticket.expo2025.or.jp/api/d/user_event_reservations` です。
POST で `application/json` を送信します。例は以下です:
```json
{
  "ticket_ids": ["T1CKET", "T2CKET"], // チケット ID の配列。これがカンマ区切りかどうか夜悩んだのはいい思い出
  "entrance_date": "20251013", // 入場日
  "start_time": "1820", // 予約したい時間帯の開始時間
  "event_code": "IC0C", // 予約したいイベントコード
  "registered_channel": "4" // これは channel と同じ意味
}
```

このレスポンスはどこにもメモしていないのでもうわかりませんが、成功すると 2xx 系が返ってきた記憶はあります。

## 脆弱だなーリスト

私が大阪にいるとき脆弱だなーってなりました

* 3 日前予約の情報を 3 日前以前から取得できる
  * UI 上だと 3 日前の 0 時から予約が解放されてどのくらいとれるか見られるけど、API ではそれ以前から見られる
  * これだめじゃね？？？？
  * でも狙いを定めることができたので個人的にラッキー
* 二段階認証が 16 歳未満のアカウントで無効化できる
* 待合室が API 経由でバイパスされる
  * UI にしか待合室が適用されないので、API だと待たずに操作し放題

* こういうのもあるらしい: https://x.com/j416dy/status/1977690789863157811

## 応用例

### TypeScript クライアントライブラリ

私はこの API を使って、TypeScript クライアントライブラリをつくりました (publish してないです):
https://github.com/pnsk-lab/myakumyakujs

こんな感じで使えます

```ts
import { fetchEvents, login, ReservationChannel } from 'jsr:@pnsk-lab/myakumyakujs'

const sess = await login(userId, password)

const events = await Array.fromAsync(fetchEvents(sess, {
    ticket: tickets[0],
    channel: ReservationChannel.SAME_DAY_REGISTRATION,
    entranceDate: '20250812',
}))

console.log(events)
```

### MCP

https://x.com/nakasyou0/status/1957264897500467599

GitHub Copilot に万博の空きを聞くことができます。

### cli

CLI を作成しました。
もちろん未来の空き情報を見れるので、大活躍です。

![deno -A ./cli/cli.ts get-schedule IC0C --date=20251013
IC0C, Signature Pavilion OCHIAI Yoichi 'null²' Naorai Mode *10 min before the reservation start time
Channel: 5, Date: 20251013
Status  Time
🚫      18:20- 18:30
🚫      18:30- 18:40
🚫      18:40- 18:50
🚫      18:50- 19:00
🚫      19:00- 19:10
🚫      19:10- 19:20
🚫      19:20- 19:30
🚫      19:30- 19:40
🚫      19:40- 19:50
🚫      19:50- 20:00
🚫      20:00- 20:10
🚫      20:10- 20:20
🚫      20:20- 20:30
🚫      20:30- 20:40
🚫      20:40- 20:50](https://storage.googleapis.com/zenn-user-upload/802f86ba3411-20251014.png)


![deno -A ./cli/cli.ts list-events --date=20250501
Listing events for 20250501 for channel 5, for tickets: ZHHADFXDKJ
Status  ID      Code    Name
✅      30236   C060    Ireland: Guided Tour with Live Music/30min/Arrive 10min Early/No Late Entry/All ages require booking
✅      30237   C063    Ireland: Expo Players Concert/60min/Arrive at Assigned Time/No Late Entry/All ages require booking
✅      30238   C066    Ireland: Exhibition Free Exploration/No Live Music/All ages require booking
⏳      2716    C2N0    Italy Pavilion, also hosting the Holy See　～15:00
⏳      2717    C2N3    Italy Pavilion, also hosting the Holy See 15:00～
⏳      312     C730    Australia Pavilion - Chasing the Sun
⏳      1569    C7R0    Netherlands Pavilion Common Ground
⏳      1235    C930    Canada Pavilion
⏳      2718    C9J0    KOREA PAVILION
⏳      2719    CCB0    State of Kuwait Pavilion
⏳      3       CFR0    International Red Cross and Red Crescent Movement Pavilion
⏳      1236    CFV0    UN Pavilion
⏳      1237    CO70    Thailand Pavilion (General Admission)
⏳      1238    CO73    Thailand Pavilion (For Disabled Person)
✅      45405   CQ70    Czech Cultural Program
⏳      315     D630    POLAND PAVILION
⏳      316     D633    POLAND PAVILION: CHOPIN CONCERT (entrance to the Concert Hall only)※Please arrive 10 min early
✅      16945   EDF0    Jordan Pavilion
⏳      1239    H1H9    Japan Pavilion (3-area tour)
🚫      1240    H1HC    Japan Pavilion Biogas Plant Tour (includes Japan Pavilion access)
🚫      1241    H1HF    Japan Pavilion Biogas Plant Tour for Wheelchair Users (includes Japan Pavilion access)
⏳      1242    H3H0    Women's Pavilion in collaboration with Cartier
⏳      318     H5H0    Osaka Healthcare Pavilion（Main Building）Reborn Experience
⏳      319     H5H3    Osaka Healthcare Pavilion（Main Building）Reborn Experience+The Game of Life REBORN in 2050
⏳      320     H5H9    Osaka Healthcare Pavilion(XD Hall)Monster Hunter Bridge*Wheelchair users: see session below.
⏳      321     H5HC    [For wheelchair users]Osaka Healthcare Pavilion(XD Hall)Monster Hunter Bridge](https://storage.googleapis.com/zenn-user-upload/2c30dd57dd57-20251014.png)

## 感想

早めに万博行けばよかった。来年ないの寂しい
