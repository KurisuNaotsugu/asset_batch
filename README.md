asset-batch

このプロジェクトは、
asset-lib（データ処理ライブラリ）を使ってローカルでDBを構築・更新するための実行用プロジェクトです。

外部APIからデータ取得

XML解析・加工

SQLite DBの構築・更新s

をローカル環境で実行することを目的としています。

役割

DBを作る

DBを更新する

定期実行・手動実行の起点になる

👉 設計・ロジックはすべて asset-lib 側にあります。
👉 このプロジェクトは「実行」と「設定」だけを担当します。

ディレクトリ構成
asset-batch/
├─ batch/
│   ├─ __init__.py
│   └─ build_db.py        # DB構築・更新の実行スクリプト
│
├─ data/
│   └─ asset.db           # 生成されるSQLite DB（git管理しない）
│
├─ .env                   # 環境変数（git管理しない）
├─ pyproject.toml
└─ README.md

前提

Python 3.10+

Poetry

asset-lib にアクセスできること（git）

セットアップ
1. リポジトリを取得
git clone <asset-batch-repo>
cd asset-batch

2. Poetry環境構築
poetry install

3. asset-lib を追加
poetry add git+ssh://github.com/your-name/asset-lib.git

4. .env を作成
ASSET_DB_PATH=data/asset.db
API_KEY=your_api_key


※ .env は git 管理しません

使い方
初回構築（フルビルド）
poetry run python batch/build_db.py --init


想定処理：

DB作成

全件取得

正規化

二次テーブル作成

更新（差分更新）
poetry run python batch/build_db.py --update


想定処理：

差分取得

既存DB更新

必要な再集計

実行スクリプトの役割

batch/build_db.py は以下だけを行います：

環境変数の読み込み

設定の生成

asset-lib の services 呼び出し

実行モードの振り分け

ロジックは持たせません。

build_db.py 例
import argparse
import os
from asset_lib.services.collect_service import collect_all, update_all
from asset_lib.config.settings import Settings
from dotenv import load_dotenv

load_dotenv()

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--init", action="store_true")
    parser.add_argument("--update", action="store_true")
    args = parser.parse_args()

    settings = Settings(
        db_path=os.getenv("ASSET_DB_PATH"),
        api_key=os.getenv("API_KEY")
    )

    if args.init:
        collect_all(settings)
    elif args.update:
        update_all(settings)
    else:
        print("Please specify --init or --update")

if __name__ == "__main__":
    main()

DBの扱い

DBは data/ 配下に生成

git管理しない

消せば再構築できる前提

.gitignore 例：

data/*.db
.env
__pycache__/

想定運用
ローカル更新
poetry run python batch/build_db.py --update

ライブラリ更新後
poetry update asset-lib
poetry run python batch/build_db.py --init

設計方針

DB構築ロジックは asset-lib に集約

このプロジェクトは「実行と設定」のみ

DBは成果物として扱う

いつでも再構築できる状態を保つ