# app-config

iOS 3 アプリ（[shuchu-capsule](https://github.com/hiroooo/shuchu-capsule) / [sublog](https://github.com/hiroooo/sublog) / [kireilens](https://github.com/hiroooo/kireilens)）の **バージョンアップゲート設定** を配信する public リポ。

各アプリは起動時と scenePhase `.active` 復帰時に `https://raw.githubusercontent.com/hiroooo/app-config/main/<bundleId>.json` を fetch し、
現在バージョンと比較して強制/任意アップデートダイアログを出す。詳細は
[AppVersionGate の設計ドキュメント](https://github.com/hiroooo/hiroaki/blob/main/docs/app-version-gate.md)（private）参照。

## スキーマ

```json
{
  "storeUrl": "https://apps.apple.com/jp/app/id...",
  "releaseNotes": { "ja": "...", "en": "..." },
  "updatedAt": "YYYY-MM-DDThh:mm:ss+09:00",
  "majors": {
    "1": {
      "latestVersion": "1.0.0",
      "minSupportedVersion": "1.0.0"
    }
  }
}
```

## 判定ロジック（買い切り対応）

現在バージョンから major を抽出し、**同じメジャー内でのみ** 判定する。
メジャー跨ぎでは何も起きない（買い切り課金者の v2 再課金を強制しない）。

| 条件 | 判定 | UI |
|---|---|---|
| `current < minSupportedVersion` | forced | fullScreenCover（閉じ不可） |
| `min ≤ current < latestVersion` | recommended | sheet（「後で」24h 抑制） |
| `current ≥ latestVersion` | none | 何もしない |
| `majors["<major>"]` が無い | none | 何もしない |

## 運用

Claude Code の `/bump-version` slash コマンドで JSON を書き換え、push で即反映。

```
/bump-version sublog 1.1.0             # 任意アップデート
/bump-version sublog 1.2.0 --force     # 強制
/bump-version sublog --min 1.5.0       # 既存 recommended を強制化
/bump-version sublog --new-major 2.0.0 # 新メジャー追加
```

直接 JSON 編集も可。反映は raw CDN キャッシュで ~5 分。
