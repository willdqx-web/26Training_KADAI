# ミニ課題3 JSON保存

**問い合わせ履歴をファイルに保存する**

| このミニ課題のゴール<br>・問い合わせ内容をJSONファイルに保存する。<br>・保存済みの問い合わせを読み込んで一覧表示する。<br>・アプリを再起動しても履歴が残ることを確認する。 |
| --- |

# 1. 完成イメージ

問い合わせを登録すると `data/inquiries.json` にデータが追記される。アプリを再起動しても過去の問い合わせが一覧に残っている状態を目指す。

```json
[
  {
    "id": 1,
    "created_at": "2025-05-01 10:00:00",
    "question": "有給休暇の申請方法を教えてください。",
    "category": "休暇",
    "priority": "中",
    "answer": "..."
  }
]
```

# 2. JSONとは

JSONは、データを「項目名」と「値」の組み合わせで保存するテキスト形式です。"JavaScript Object Notation" の略ですが、Pythonをはじめ多くの言語で標準的に使われています。

## JSONの基本構造

JSONには3つの書き方があります。

### ① オブジェクト（辞書）: `{ }` で囲む

「項目名（キー）」と「値」をコロンで区切って書きます。複数ある場合はカンマで区切ります。

```json
{
  "id": 1,
  "question": "有給休暇の申請方法を教えてください。",
  "priority": "中"
}
```

### ② 配列（リスト）: `[ ]` で囲む

複数のデータをまとめて並べます。

```json
[
  "休暇",
  "備品",
  "その他"
]
```

### ③ 入れ子（ネスト）: オブジェクトの中に配列やオブジェクトを入れる

実際のアプリではこの形が多く使われます。

```json
[
  {
    "id": 1,
    "question": "有給休暇の申請方法を教えてください。",
    "category": "休暇",
    "priority": "中",
    "answer": "人事ポータルから申請できます。"
  },
  {
    "id": 2,
    "question": "社員証を紛失しました。",
    "category": "設備",
    "priority": "高",
    "answer": "総務窓口まで連絡してください。"
  }
]
```

## JSONで使えるデータの種類

| 種類 | 書き方の例 | Pythonでの対応 |
| --- | --- | --- |
| 文字列 | `"有給休暇"` | `str` |
| 数値 | `1`, `3.14` | `int`, `float` |
| 真偽値 | `true`, `false` | `True`, `False` |
| null | `null` | `None` |
| 配列 | `["休暇", "備品"]` | `list` |
| オブジェクト | `{"key": "value"}` | `dict` |

> **補足：** JSONの真偽値は小文字（`true`/`false`）です。Pythonの `True`/`False`（大文字）とは書き方が異なりますが、`json.dump` / `json.load` が自動で変換してくれます。

## JSONの便利なところ

**① テキストなのに構造がある**

Excelファイル（.xlsx）やPDFと違い、メモ帳で開いてそのまま読めます。バグ調査のときにファイルを直接確認できるのは大きな利点です。

**② どの言語・ツールとも連携できる**

Python、JavaScript、Java、SQL…ほぼすべての言語にJSONを読み書きする機能が標準で備わっています。FastAPIがリクエスト・レスポンスにJSONを使っているのもこのためです。

**③ Pythonの辞書・リストと1対1で対応している**

```python
# Pythonの辞書をそのままJSONに変換できる
import json

data = {"question": "有給申請の方法は？", "priority": "中"}
json_text = json.dumps(data, ensure_ascii=False)
# → '{"question": "有給申請の方法は？", "priority": "中"}'

# 逆もできる
data2 = json.loads(json_text)
print(data2["question"])  # → 有給申請の方法は？
```

## このアプリでのJSONの使いどころ

このミニ課題では `data/inquiries.json` に問い合わせ履歴を蓄積します。将来的には以下のような活用が考えられます。

| 活用シーン | 具体的なイメージ |
| --- | --- |
| 履歴の再利用 | 過去の問い合わせを読み込んでAIに渡し、「似た問い合わせ」を検索する |
| 集計・分析 | カテゴリ別・優先度別に件数を集計し、どの部門から問い合わせが多いか把握する |
| 外部ツール連携 | JSONをそのままSlackやメールシステムへ送り、担当者に通知する |
| DBへの移行 | データ量が増えたときにJSONをSQLiteやPostgreSQLにインポートする移行元として使う |

# 3. 用語

| 用語 | 説明 |
| --- | --- |
| JSON | テキスト形式のデータ保存形式です。`{"key": "value"}` の形で書きます。 |
| Path | Pythonでファイルのパスを扱うためのクラスです。`pathlib.Path` を使います。 |
| encoding | ファイルを読み書きするときの文字コードです。日本語を扱うには `utf-8` を指定します。 |
| ensure_ascii | `json.dump` のオプションです。`False` にすると日本語をそのまま保存できます。 |

# 4. 保存用ファイルを作る

```powershell
notepad backend/storage.py
```

> **補足：** `data` フォルダは `storage.py` のコードが自動で作成します（`DATA_PATH.parent.mkdir(exist_ok=True)` の部分）。手動で作成しなくても問題ありません。

# 5. JSON保存コード

```python
import json
from pathlib import Path
from datetime import datetime

DATA_PATH = Path("data/inquiries.json")  # プロジェクトルートからの相対パス


def load_inquiries():
    if not DATA_PATH.exists():
        return []
    with DATA_PATH.open("r", encoding="utf-8") as f:
        return json.load(f)


def save_inquiry(question, category, priority, answer):
    inquiries = load_inquiries()
    new_id = len(inquiries) + 1
    item = {
        "id": new_id,
        "created_at": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "question": question,
        "category": category,
        "priority": priority,
        "answer": answer,
    }
    inquiries.append(item)
    DATA_PATH.parent.mkdir(exist_ok=True)
    with DATA_PATH.open("w", encoding="utf-8") as f:
        json.dump(inquiries, f, ensure_ascii=False, indent=2)
    return item
```

# 6. FastAPIに保存処理を組み込む

`backend/main.py` を以下のように修正します。ミニ課題2で作成した `main.py` に、import文の追加と既存の `/analyze` エンドポイントの書き換え、新しい `/inquiries` エンドポイントの追加を行います。

```python
from fastapi import FastAPI
from pydantic import BaseModel
from backend.storage import save_inquiry, load_inquiries  # 追加

app = FastAPI()


class InquiryRequest(BaseModel):
    question: str


@app.get("/")
def root():
    return {"message": "API is running"}


@app.post("/analyze")
def analyze_inquiry(request: InquiryRequest):
    item = save_inquiry(          # ← ミニ課題2の return {...} からsave_inquiryを呼ぶように書き換える
        question=request.question,
        category="その他",
        priority="低",
        answer="問い合わせ内容を受け付けました。"
    )
    return item


@app.get("/inquiries")            # 追加
def get_inquiries():
    return load_inquiries()
```

> **注意：** `@app.post("/analyze")` はミニ課題2で既に書いたものを**書き換え**ます。同じエンドポイントを2つ書くと片方が無視されます。

# 7. StreamlitにAPI連携で一覧表示を追加する

`frontend/app.py` に、問い合わせ一覧を取得して表示するコードを追加します。

```python
import streamlit as st
import requests

st.title("総務問い合わせ入力")
question = st.text_area("問い合わせ内容", height=160)

if st.button("APIに送信する"):
    if question.strip() == "":
        st.error("問い合わせ内容を入力してください。")
    else:
        response = requests.post(
            "http://127.0.0.1:8000/analyze",
            json={"question": question},
            timeout=30
        )
        result = response.json()
        st.write("カテゴリ:", result["category"])
        st.write("緊急度:", result["priority"])
        st.write("回答案:", result["answer"])

# 問い合わせ一覧を取得して表示する（追加部分）
st.subheader("問い合わせ一覧")
resp = requests.get("http://127.0.0.1:8000/inquiries", timeout=10)
if resp.status_code == 200:
    inquiries = resp.json()
    if inquiries:
        for item in inquiries:
            st.write(f"[{item['id']}] {item['created_at']} | {item['question'][:40]}")
    else:
        st.write("まだ問い合わせはありません。")
```

> **補足：** Streamlitはボタンを押したときや画面を更新したときに上から再実行されます。そのため、一覧取得コードを末尾に書いておくだけで、送信後に一覧が更新されます。

# 9. 動作確認

- [ ] 問い合わせを登録すると data/inquiries.json が作成される。

- [ ] JSONファイルに日本語が読める形で保存される。

- [ ] 複数回登録すると履歴が増える。

- [ ] アプリを再起動しても履歴が残っている。

- [ ] Streamlitの画面に問い合わせ一覧が表示される。

- [ ] http://127.0.0.1:8000/inquiries をブラウザで開くとJSON形式で一覧が返ってくる。

# 8. よくあるエラー

| 現象 | 確認すること |
| --- | --- |
| JSONDecodeError | JSONファイルを手作業で壊していないか確認する。 |
| ファイルが見つからない / data/inquiries.json が作成されない | uvicornをプロジェクトルートから起動しているか確認する。`Path("data/inquiries.json")` はプロジェクトルートからの相対パスになる。 |
| 日本語が\u形式になる | json.dumpでensure_ascii=Falseを指定しているか確認する。 |
| ModuleNotFoundError: No module named 'backend.storage' | `backend/storage.py` が存在するか確認する。uvicornをプロジェクトルートから起動しているか確認する。 |

# 10. 本番設計につながる問い

- [ ] 問い合わせ履歴には、どの項目を保存すべきか。

- [ ] ステータス（未対応・対応中・完了）管理は必要か。必要な場合、どのタイミングで実装するか。

- [ ] 個人情報を含む問い合わせを保存するとき、どのような注意が必要か。

# 11. 振り返り

このミニ課題で学んだこと：

- Pythonの `json` モジュールと `pathlib.Path` を使ってファイルの読み書きができることを確認した。
- アプリを再起動してもデータが残る「永続化」の仕組みを体験した。
- FastAPIのエンドポイントにストレージ処理を組み込む方法を学んだ。
- `ensure_ascii=False` と `encoding="utf-8"` で日本語を正しく扱う方法を確認した。
- 相対パスはuvicornを起動するフォルダ（プロジェクトルート）を基準にすることを確認した。

# 参考リンク

Python json モジュール: https://docs.python.org/ja/3/library/json.html

Python pathlib: https://docs.python.org/ja/3/library/pathlib.html
