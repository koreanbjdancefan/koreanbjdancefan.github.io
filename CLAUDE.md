# kbjdfan-site プロジェクト

## 概要
kbjdfan.com のランディングページ。GitHub Pages でホスティング。
SNSプロフィール（TikTok以外）からのリンク先として使用。

## リポジトリ・ホスティング
- ローカル: `D:\Claude\kbjdfan-site\`
- GitHub: `koreanbjdancefan/koreanbjdancefan.github.io`（GitHub Pages）
- ドメイン: kbjdfan.com（Cloudflare DNS → GitHub Pages）
- GitHub PAT: koreanbjdancefan アカウントの Fine-grained PAT（有効期限: 2026-03-31）
  - リモートURLにトークン埋め込み済み

## ファイル構成
| ファイル | 説明 |
|---------|------|
| `index.html` | メインランディングページ |
| `vip.html` | VIPプラン・チャンネル案内ページ |
| `stats.json` | 統計データ（NASから毎日自動生成・GitHub REST API経由でpush） |
| `KBJFAN.jpg` | ロゴ画像 |
| `CNAME` | カスタムドメイン設定（kbjdfan.com） |
| `welcome.html` | Bot認証後のウェルカムページ |
| `tiktok.html` | TikTok用リンクページ（Telegramリンクなし） |
| `tg/` | Telegram関連 |

## index.html の構成（上から順）
1. GA4トラッキング（G-JZBCL6B1NQ）+ クリックイベント追跡
2. Google Translate（en/ko/ja/zh-CN/zh-TW）
3. ロゴ + ブランド名 + 多言語タグライン
4. Stats Counter Strip（subscribers / videos / BJs / Daily Updates）
5. Telegram CTA + Joinボタン（4言語説明文）
6. SNSリンクボタン（YouTube / TikTok / Instagram）
7. TikTok/Instagram ランダム埋め込み（stats.json の sns_posts からランダム1本表示）
8. YouTube アップロードプレイリスト iframe 埋め込み + 最新3本サムネイル
9. Free vs VIP 比較セクション
10. VIP Membership ボタン
11. フッター

## stats.json の構造
```json
{
  "updated_at": "ISO8601",
  "youtube": { "subscribers", "total_views", "video_count", "latest_videos": [...] },
  "telegram": { "members" },
  "totals": { "videos", "bjs" },
  "channels": { "ChannelName": { "videos", "bjs", "tier", "latest" } },
  "sns_posts": {
    "tiktok": ["VIDEO_ID_1", "VIDEO_ID_2", ...],
    "instagram": ["SHORTCODE_1", ...]
  }
}
```
- NASの `generate_stats.py` が毎日生成 → GitHub REST API（PUT /repos/.../contents/stats.json）でpush
- `channels` は現在空（`videos.channel_source` がDB未設定のため）
- `sns_posts.instagram` は現在空（手動で追加する必要あり）

## stats.json 生成スクリプト
- ローカル: `D:\Claude\videodatabase\local-dev\stats-generator\generate_stats.py`
- NAS: `M:\kbjdf-web\stats-generator\generate_stats.py`（同期先）
- 実行: NASでcron（DSM Task Scheduler）または手動
- データソース:
  - VideoDatabase API: `/api/stats/public`（動画数・BJ数・チャンネル別・Telegramメンバー数）
  - YouTube Data API v3: チャンネル統計 + 最新動画3本
  - TikTokプロフィールスクレイピング: 動画IDリスト自動更新
- 環境変数: `VIDEODB_API_URL`, `YOUTUBE_API_KEY`, `YOUTUBE_CHANNEL_ID`, `GITHUB_PAT`, `GITHUB_REPO`
- NAS .env: `/volume1/docker/kbjdf-web/.env`
- 実行スクリプト: `/volume1/docker/kbjdf-web/stats-generator/run_stats.sh`

## vip.html の動的データ
- stats.json の `channels` からチャンネル別の動画数・BJ数を表示
- 各チャンネルカードに `data-channel="ChannelName"` 属性
- プランカードに合計stats（Entry / VIP / Premier）
- PLAN_CHANNELS マッピング:
  - Entry: Dance, ChineseDance
  - VIP: New and old, NudeDance, CrewDance, Short Dance + Entry全部
  - Premier: almighty, Dance clippings, CrewDance Full, strage + VIP全部

## SNS埋め込み仕様
- TikTok: `https://www.tiktok.com/embed/v2/{VIDEO_ID}` をiframeで表示
- Instagram: `https://www.instagram.com/p/{SHORTCODE}/embed/` をiframeで表示
- ページ読み込み時にランダム1本ずつ選択
- TikTokのみ表示時は1カラムレイアウト（single-col）

## Git運用
- D:ドライブが正（GitHub Pagesリポジトリ）
- NASからはGitHub REST APIでstats.jsonのみ直接push
- HTML変更はD:でcommit → `git push origin main`
- push競合時（NASのstats.json push後）は `--force` で上書きOK

## デプロイ手順
1. D:でファイル編集
2. `cd D:/Claude/kbjdfan-site && git add ... && git commit && git push origin main`
3. GitHub Pages が自動デプロイ（数分で反映）

## 未完了タスク
- [ ] DSM Task Scheduler でcron設定（毎日6:00 AM、`run_stats.sh`実行）
- [ ] `videos.channel_source` をDBに設定（チャンネル別データ表示のため）
- [ ] Instagram投稿ショートコードの手動追加（stats.json の sns_posts.instagram）
- [ ] YouTube APIキー制限（YouTube Data API v3のみに制限推奨）
