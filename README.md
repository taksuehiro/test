# AWS App Runner 最小デプロイ

このリポジトリは AWS App Runner で最短デプロイできる Flask アプリケーションです。

## ローカル動作確認

### 1. 仮想環境の作成とアクティベート
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### 2. 依存関係のインストール
```bash
pip install -r requirements.txt
```

### 3. アプリケーションの起動
```bash
# 開発用（Flask直接起動）
python app.py

# 本番用（Gunicorn起動）
gunicorn -w 2 -k gthread -b 0.0.0.0:8080 app:app
```

### 4. 動作確認
ブラウザで `http://localhost:8080` にアクセスして "OK: App Runner minimal" が表示されることを確認

## AWS App Runner デプロイ設定

### コンソール設定値

**基本設定:**
- Runtime: Python 3.11
- Source directory: /
- Port: 8080

**Build commands:**
```
python -m pip install --upgrade pip setuptools wheel
pip install -r requirements.txt
```

**Start command:**
```
gunicorn -w 2 -k gthread -b 0.0.0.0:8080 app:app
```

**環境変数（オプション）:**
- `PYTHONUNBUFFERED`: `1`

## Basic認証の追加（オプション）

セキュリティを強化したい場合は、以下のミドルウェアを `app.py` に追加できます：

```python
from functools import wraps
from flask import request, Response
import os

def check_auth(username, password):
    """認証チェック"""
    return username == os.environ.get('BASIC_USER') and \
           password == os.environ.get('BASIC_PASS')

def authenticate():
    """認証要求"""
    return Response(
        'Could not verify your access level for that URL.\n'
        'You have to login with proper credentials', 401,
        {'WWW-Authenticate': 'Basic realm="Login Required"'})

def requires_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        auth = request.authorization
        if not auth or not check_auth(auth.username, auth.password):
            return authenticate()
        return f(*args, **kwargs)
    return decorated

# 各ルートに @requires_auth デコレータを追加
@app.get("/")
@requires_auth
def hello():
    return "OK: App Runner minimal"
```

**App Runner環境変数設定:**
- `BASIC_USER`: ユーザー名（例: admin）
- `BASIC_PASS`: パスワード（例: securepassword123）

## デプロイ後の確認

1. **URL確認**: App Runner コンソールで表示される URL にアクセス
2. **ログ確認**: コンソールの「ログ」タブでエラーがないか確認

## 課金停止方法

不要になった場合は以下の手順で課金を停止：

1. AWS App Runner コンソールにアクセス
2. 該当サービスを選択
3. 「アクション」→「サービス削除」をクリック
4. 確認ダイアログで「削除」をクリック

**注意**: 削除後は復元できません。重要なデータがある場合は事前にバックアップしてください。
