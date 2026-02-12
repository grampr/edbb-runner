# Discord BOT環境 - リモート配信対応

`start.bat`を起動するだけで、Pythonのインストールから環境構築、BOT起動までを自動化。
HTTP経由でbot.pyをリモート配信できます。

## 🚀 使い方

### 基本的な起動

```batch
start.bat
```

自動的に以下が実行されます：
1. ✅ Pythonの確認・インストール（winget使用）
2. ✅ 仮想環境の作成
3. ✅ discord.pyのインストール
4. ✅ HTTPサーバー起動（localhost:6859）
5. ✅ bot.pyがあれば自動起動

### bot.pyの配信

HTTPサーバー（localhost:6859）にPOSTリクエストでbot.pyを送信：

#### PowerShellで送信
```powershell
Invoke-RestMethod -Uri http://localhost:6859 -Method POST -Body (Get-Content bot.py -Raw)
```

#### curlで送信
```bash
curl -X POST http://localhost:6859 --data-binary @bot.py
```

#### Pythonで送信
```python
import requests

with open('bot.py', 'r', encoding='utf-8') as f:
    bot_code = f.read()

response = requests.post('http://localhost:6859', data=bot_code.encode('utf-8'))
print(response.json())
```

## 🔄 動作フロー

### 初回起動（bot.pyなし）
1. `start.bat` 実行
2. Pythonインストール・venv作成
3. HTTPサーバーのみ起動（ポート6859で待機）
4. bot.pyをPOSTで受信待ち

### 2回目以降（bot.pyあり）
1. `start.bat` 実行
2. HTTPサーバー起動
3. **bot.pyも同時に自動起動**
4. 新しいbot.pyを受信したら自動的に再起動

## 📁 ファイル構成

```
.
├── start.bat          # エントリーポイント（PowerShell呼び出し）
├── start.ps1          # セットアップ＆起動スクリプト
├── start.py           # HTTPサーバー＆BOT管理
├── bot.py             # BOTコード（POSTで配信される）
└── venv/              # Python仮想環境（自動作成）
```

## 🌐 HTTPエンドポイント

### GET /
ステータス確認

**レスポンス例:**
```json
{
  "status": "running",
  "port": 6859,
  "bot_exists": true,
  "bot_running": true
}
```

### POST /
bot.pyのアップロード＆起動

**リクエスト:**
- Content-Type: text/plain
- Body: bot.pyのソースコード（UTF-8）

**レスポンス:**
```json
{
  "status": "ok",
  "message": "bot.py saved and started"
}
```

## 📦 配布方法

1. このフォルダをZIPで圧縮
2. 配布先で`start.bat`を実行
3. リモートからbot.pyを配信

## 🛠️ トラブルシューティング

### wingetが使えない
[Python公式](https://www.python.org/)から手動インストール後、`start.bat`を再実行

### ポート6859が使用中
start.pyの`PORT = 6859`を変更してください

### BOTが起動しない
- bot.pyに`.env`ファイルが必要な場合は手動で作成
- Discord Developer Portalで必要なIntentsを有効化

## 💡 サンプルbot.py

```python
import discord
import os

client = discord.Client(intents=discord.Intents.default())

@client.event
async def on_ready():
    print(f'{client.user}でログインしました')

@client.event
async def on_message(message):
    if message.author == client.user:
        return

    if message.content == '!ping':
        await message.channel.send('Pong!')

# トークンは環境変数から取得
client.run(os.getenv('DISCORD_TOKEN', 'YOUR_TOKEN_HERE'))
```

## ⚙️ 仕組み

- **start.bat**: PowerShell実行ポリシーをバイパスしてstart.ps1を起動
- **start.ps1**: Python環境のセットアップとstart.py起動
- **start.py**:
  - HTTPサーバー（ポート6859）でbot.pyを受信
  - bot.pyが存在する場合は自動起動
  - POSTで新しいbot.pyを受信すると既存プロセスを再起動
