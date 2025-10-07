# video2manual_documents
動画からマニュアルを生成
# Video Manual Generator

**動画から自動でマニュアルを生成するシステム**

[![CI](https://github.com/user/video-manual-generator/workflows/CI/badge.svg)](https://github.com/user/video-manual-generator/actions)

## 概要

Video Manual Generator は、操作説明動画から自動的にステップバイステップのマニュアルを生成するシステムです。

### 主要機能

- ✅ **音声認識 (STT)**: GPT-4o Transcribe で高精度な文字起こし、タイムスタンプ付きで記録
- ✅ **AI要約**: GPT-5 で文字起こしテキストを自動要約
- ✅ **シーン検出**: 画面が切り替わるタイミングで自動キャプチャを取得
- ✅ **キャプチャ選択**: 取得したキャプチャをユーザーが選択・編集
- ✅ **テンプレート適用**: Markdown/PDF 形式でマニュアルを出力
- ✅ **拡張性**: Strategy パターンによる STT・シーン検出エンジンの差し替え対応

### アーキテクチャ

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Frontend   │─────▶│   Backend    │─────▶│   Storage    │
│ React + Vite │      │   FastAPI    │      │    Local     │
└──────────────┘      └──────────────┘      └──────────────┘
                             │
                    ┌────────┴────────┐
                    │                 │
              ┌─────▼─────┐   ┌──────▼──────┐
              │  GPT-4o   │   │   OpenCV    │
              │Transcribe │   │Scene Detect │
              └─────┬─────┘   └─────────────┘
                    │
              ┌─────▼─────┐
              │  GPT-5    │
              │Summarizer │
              └───────────┘
```

---

## セットアップ

### 必要要件

- **Docker**: 20.10 以上
- **Docker Compose**: 2.0 以上

### Dockerのインストール

#### Windows

1. **Docker Desktop for Windows をダウンロード**
   - https://www.docker.com/products/docker-desktop/

2. **インストーラーを実行**
   - WSL 2 バックエンドを使用する設定を有効にする

3. **インストール確認**
   ```powershell
   docker --version
   docker-compose --version
   ```

#### macOS

1. **Docker Desktop for Mac をダウンロード**
   - https://www.docker.com/products/docker-desktop/

2. **インストーラーを実行**
   - アプリケーションフォルダにドラッグ&ドロップ

3. **インストール確認**
   ```bash
   docker --version
   docker-compose --version
   ```

#### Linux (Ubuntu/Debian)

```bash
# Dockerのインストール
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Dockerをsudoなしで実行できるようにする
sudo usermod -aG docker $USER
newgrp docker

# インストール確認
docker --version
docker compose version
```

### インストール手順

#### 1. リポジトリのクローン

```bash
git clone <repository-url>
cd create_manual
```

#### 2. 環境変数の設定

```bash
# backend/.env ファイルを編集してOpenAI APIキーを設定
# OPENAI_API_KEY の値を実際のAPIキーに置き換える
```

`backend/.env` の内容：
```env
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_MODEL=gpt-5
```

#### 3. Docker Composeで起動

```bash
# すべてのサービスをビルド・起動
docker-compose up -d

# ログを確認
docker-compose logs -f
```

**起動確認**:
- Backend: http://localhost:8000
- Frontend: http://localhost:5173

---

## 起動方法

### 初回起動

```bash
docker-compose up -d --build
```

### 通常起動

```bash
docker-compose up -d
```

### 停止

```bash
docker-compose down
```

### コンテナの再起動

```bash
docker-compose restart
```

### ログ確認

```bash
# 全サービスのログ
docker-compose logs -f

# Backendのみ
docker-compose logs -f backend

# Frontendのみ
docker-compose logs -f frontend
```

---

## 使い方

### GUI での操作 (推奨)

1. **動画アップロード**: http://localhost:5173 にアクセスし、動画をアップロード
2. **自動解析**: GPT-4o Transcribe による音声認識とシーン検出が自動実行される (数分かかる場合あり)
3. **要約生成**: 文字起こし後、GPT-5 が自動で要約を生成
4. **キャプチャ選択**: 検出されたキャプチャを確認し、採用/除外を選択
5. **エクスポート**: Markdown または PDF 形式でダウンロード

### API での操作

#### 1. 動画アップロード

```bash
curl -F "file=@./sample.mp4" http://localhost:8000/videos/upload
```

レスポンス例:
```json
{
  "video_id": "123e4567-e89b-12d3-a456-426614174000",
  "filename": "sample.mp4",
  "size_bytes": 10485760,
  "duration_sec": 120.5
}
```

#### 2. 音声認識 + 要約

```bash
curl -X POST http://localhost:8000/process/transcribe/{video_id}
```

#### 3. シーン検出

```bash
curl -X POST http://localhost:8000/process/scene-detect/{video_id}
```

#### 4. マニュアル計画作成

```bash
curl -X POST http://localhost:8000/manual/plan \
  -H "Content-Type: application/json" \
  -d '{"video_id": "{video_id}", "title": "操作マニュアル"}'
```

#### 5. Markdown エクスポート

```bash
curl -X POST http://localhost:8000/export/markdown \
  -H "Content-Type: application/json" \
  -d '{"video_id": "{video_id}", "format": "markdown"}'
```

#### 6. PDF エクスポート

```bash
curl -X POST http://localhost:8000/export/pdf \
  -H "Content-Type: application/json" \
  -d '{"video_id": "{video_id}", "format": "pdf"}'
```

---

## ディレクトリ構成

```
.
├── docker-compose.yml         # Docker Compose設定
├── backend/                   # FastAPI バックエンド
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── .env                   # 環境変数（OpenAI APIキー等）
│   ├── app/
│   │   ├── main.py           # エントリポイント
│   │   ├── routes/           # API ルート
│   │   ├── services/         # ビジネスロジック
│   │   │   ├── stt/          # 音声認識 (Whisper)
│   │   │   ├── summarizer/   # GPT-5 要約
│   │   │   ├── scenes/       # シーン検出 (OpenCV)
│   │   │   ├── capture/      # マニュアル計画
│   │   │   ├── template/     # テンプレート描画
│   │   │   └── export/       # PDF生成
│   │   ├── models/           # データモデル
│   │   ├── core/             # 設定・ロガー
│   │   └── utils/            # FFmpeg ラッパー
│   └── tests/                # pytest テスト
│
├── frontend/                  # React フロントエンド
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── src/
│   │   ├── pages/            # 画面
│   │   ├── components/       # 共通コンポーネント
│   │   └── lib/api.ts        # API クライアント
│   └── package.json
│
├── templates/                 # Jinja2 テンプレート
│   └── manual_default.md.j2
│
├── data/                      # データ保存先 (Docker volume)
│   ├── uploads/              # アップロード動画
│   ├── captures/             # キャプチャ画像
│   ├── intermediate/         # 中間生成物 (JSON)
│   └── exports/              # エクスポート結果
│
└── README.md                  # このファイル
```

---

## 設定

### 環境変数 (backend/.env)

| 変数名                     | デフォルト値      | 説明                                        |
|--------------------------|----------------|--------------------------------------------|
| `HOST`                   | 0.0.0.0        | サーバーホスト                                 |
| `PORT`                   | 8000           | サーバーポート                                 |
| `DEBUG`                  | False          | デバッグモード                                 |
| `STT_ENGINE`             | gpt4o          | STTエンジン (`gpt4o` / `whisper` / `dummy`) |
| `WHISPER_MODEL`          | base           | Whisperモデル (`tiny`/`base`/`small`/etc.)   |
| `WHISPER_DEVICE`         | cpu            | 実行デバイス (`cpu` / `cuda`)                |
| `GPT4O_TRANSCRIBE_MODEL` | gpt-4o-transcribe | GPT-4o文字起こしモデル                      |
| `SCENE_THRESHOLD`        | 30.0           | シーン変化検出閾値 (0-100、大きいほど厳しい)        |
| `SCENE_DETECTION_METHOD` | histogram      | 検出方法 (`histogram` / `ssim`)             |
| `PDF_ENGINE`             | playwright     | PDF生成エンジン (`playwright` / `weasyprint`) |
| `OPENAI_API_KEY`         | (必須)          | OpenAI APIキー（GPT-4o STT・GPT-5要約に必要） |
| `OPENAI_MODEL`           | gpt-5          | 要約用GPTモデル                              |
| `OPENAI_MAX_COMPLETION_TOKENS` | 2000    | GPT要約の最大トークン数                        |

### チューニングポイント

**シーン検出**
- **シーン検出が多すぎる場合**: `SCENE_THRESHOLD` を大きくする (例: 50.0)
- **シーン検出が少なすぎる場合**: `SCENE_THRESHOLD` を小さくする (例: 20.0)

**音声認識（STT）**
- **高精度・日本語に強い**: `STT_ENGINE=gpt4o`（現在のデフォルト、APIキー必要）
- **コスト削減・オフライン実行**: `STT_ENGINE=whisper`（ローカル実行、無料）
  - 高速化: `WHISPER_MODEL=tiny` (精度は下がる)
  - 高精度: `WHISPER_MODEL=small` または `medium` (処理時間増)
- **開発・テスト用**: `STT_ENGINE=dummy`（固定のサンプルテキスト生成）

---

## トラブルシューティング

### Q1. Dockerコンテナが起動しない

**A**: ログを確認してください。

```bash
docker-compose logs -f
```

### Q2. OpenAI APIキーのエラーが出る

**A**: `backend/.env` ファイルの `OPENAI_API_KEY` が正しく設定されているか確認してください。

### Q3. ポート番号が競合する

**A**: `docker-compose.yml` のポート番号を変更してください。

```yaml
ports:
  - "8001:8000"  # 8000 → 8001 に変更
```

### Q4. コンテナをクリーンに再構築したい

**A**: 以下のコマンドで完全に削除して再構築します。

```bash
docker-compose down -v
docker-compose up -d --build
```

---

## 開発ガイド

### ローカル開発でコードの変更を反映

Docker Composeはボリュームマウントを使用しているため、コードの変更は自動的にコンテナ内に反映されます。

- Backend: `backend/app/` 内のファイルを編集すると自動リロード
- Frontend: `frontend/src/` 内のファイルを編集すると自動リロード

### 新しい STT エンジンを追加

1. `backend/app/services/stt/` に新しいクラスを作成
2. `STTStrategy` を継承して `transcribe()` メソッドを実装
3. `backend/app/services/stt/__init__.py` の `get_stt_engine()` に追加

### 新しいテンプレートを追加

1. `templates/` に `.md.j2` ファイルを作成
2. Jinja2 構文で Markdown を記述
3. API 呼び出し時に `template` パラメータで指定

---

## ライセンス

MIT License

---

## 貢献

Issue や Pull Request を歓迎します!

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 参考資料

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [OpenAI Whisper](https://github.com/openai/whisper)
- [OpenAI GPT API](https://platform.openai.com/docs/)
- [OpenCV Scene Detection](https://docs.opencv.org/)
- [Docker Documentation](https://docs.docker.com/)

---

**Video Manual Generator v1.0.0**