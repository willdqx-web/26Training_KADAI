# ミニ課題2 FastAPI連携

**画面からバックエンドAPIを呼び出す**

| このミニ課題のゴール<br>・FastAPIで簡単なAPIを作る。<br>・StreamlitからAPIへ問い合わせ文を送る。<br>・APIから返ってきた結果を画面に表示する。 |
| --- |

# 1. 完成イメージ

Streamlit画面で問い合わせ文を入力してボタンを押すと、FastAPIにリクエストが送られ、カテゴリ・緊急度・回答案が画面に表示される。

```
[Streamlit画面]  --問い合わせ文を送る-->  [FastAPI]
[Streamlit画面]  <--結果を受け取る--      [FastAPI]
```

# 2. APIとは

APIは、画面から別の処理へお願いを出すための窓口です。今回の場合、Streamlit画面が「問い合わせ文を渡します。処理してください」とFastAPIに依頼します。

# 3. 用語

| 用語 | 説明 |
| --- | --- |
| FastAPI | PythonでAPIを作るためのライブラリです。 |
| エンドポイント | APIの入口となるURLです。例: /analyze |
| GET | 情報を取得するときに使うAPIの方法です。 |
| POST | 入力内容を送って処理してもらうときに使うAPIの方法です。 |
| Swagger | APIをブラウザで確認できる画面です。/docsで開けます。 |

# 4. ライブラリを追加する

```powershell
uv pip install fastapi uvicorn requests pydantic
```

# 5. バックエンドを作成する

```powershell
mkdir backend
notepad backend/__init__.py
notepad backend/main.py
```

> **補足：** `backend/__init__.py` は空ファイルです。`backend` フォルダをPythonパッケージとして認識させるために必要です。notepadで保存するだけで内容は不要です。

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class InquiryRequest(BaseModel):
    question: str

@app.get("/")
def root():
    return {"message": "API is running"}

@app.post("/analyze")
def analyze_inquiry(request: InquiryRequest):
    return {
        "category": "その他",
        "priority": "低",
        "answer": f"問い合わせ内容を受け付けました: {request.question}"
    }
```

## サンプルコードの解説

### ライブラリの読み込み

```python
from fastapi import FastAPI
from pydantic import BaseModel
```

| 行 | 意味 |
| --- | --- |
| `from fastapi import FastAPI` | FastAPIライブラリからAPIを作るための部品を取り込む |
| `from pydantic import BaseModel` | 受け取るデータの型を定義するための部品を取り込む |

---

### アプリの作成

```python
app = FastAPI()
```

`FastAPI()` でAPIアプリの本体を作り、`app` という名前で使えるようにしています。後で起動コマンドに `backend.main:app` と書くのは「`backend/main.py` の中の `app` を起動してください」という意味です。

---

### 受け取るデータの形を定義する

```python
class InquiryRequest(BaseModel):
    question: str
```

Streamlitから送られてくるデータの「形」を定義しています。`question: str` は「`question` という名前の文字列データを受け取る」という意味です。送り手と受け手で形を合わせることで、間違ったデータが来たときに自動でエラーを返せます。

---

### GETエンドポイント（動作確認用）

```python
@app.get("/")
def root():
    return {"message": "API is running"}
```

| 部分 | 意味 |
| --- | --- |
| `@app.get("/")` | ブラウザで `http://127.0.0.1:8001/` を開いたとき（GETリクエスト）に呼ばれる、という印 |
| `def root():` | 実際に実行される処理（関数） |
| `return {"message": "API is running"}` | `{"message": "API is running"}` というJSONを返す |

`@app.get(...)` のような `@` から始まる記法を**デコレータ**といいます。関数に「このURLへのリクエストが来たら自分を呼んでください」と登録する役割を持っています。

---

### POSTエンドポイント（メイン処理）

```python
@app.post("/analyze")
def analyze_inquiry(request: InquiryRequest):
    return {
        "category": "その他",
        "priority": "低",
        "answer": f"問い合わせ内容を受け付けました: {request.question}"
    }
```

| 部分 | 意味 |
| --- | --- |
| `@app.post("/analyze")` | `http://127.0.0.1:8001/analyze` へのPOSTリクエストに応答する |
| `request: InquiryRequest` | 上で定義した `InquiryRequest` の形でデータを受け取る |
| `request.question` | 受け取ったデータの中の `question` フィールドの値を取り出す |
| `f"..."` | f文字列。`{}` の中に変数を埋め込んで文字列を作る記法 |

返す値（辞書）がそのままJSONに変換されてStreamlit側に届きます。

---

# 6. バックエンドを起動する

> **重要：** 以下のコマンドは `C:\work\ai-inquiry-app` フォルダ（プロジェクトのルート）で実行してください。別のフォルダで実行すると `backend.main` が見つからないエラーになります。

```powershell
uv run uvicorn backend.main:app --reload --host 127.0.0.1 --port 8001
```

起動に成功すると、ターミナルに以下のように表示されます。

```
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8001 (Press CTRL+C to quit)
```

ブラウザで http://127.0.0.1:8001/docs を開くと、API確認画面（Swagger UI）が表示されます。

> **補足：** `--reload` オプションを付けると、コードを変更するたびにFastAPIが自動で再起動します。開発中は必ず付けておきましょう。

## Swagger UIでAPIをテストする

FastAPIには、APIをブラウザ上でテストできる画面（Swagger UI）が自動で用意されています。Streamlitを作る前にここで動作確認しておくと、問題の切り分けが楽になります。

### 手順1: Swagger UIを開く

ブラウザで以下のURLにアクセスします。

```
http://127.0.0.1:8001/docs
```

画面が開くと、`GET /` と `POST /analyze` の2つのエンドポイントが一覧表示されます。

---

### 手順2: GETエンドポイントをテストする

1. **`GET /`** の行をクリックして展開する
2. 右上の **「Try it out」** ボタンをクリックする
3. **「Execute」** ボタンをクリックする
4. 画面下の **Response body** に以下が表示されれば成功です

```json
{
  "message": "API is running"
}
```

---

### 手順3: POSTエンドポイントをテストする

1. **`POST /analyze`** の行をクリックして展開する
2. 右上の **「Try it out」** ボタンをクリックする
3. **Request body** の入力欄に以下を入力する（`string` の部分を書き換える）

```json
{
  "question": "社員証を紛失しました"
}
```

4. **「Execute」** ボタンをクリックする
5. 画面下の **Response body** に以下が表示されれば成功です

```json
{
  "category": "その他",
  "priority": "低",
  "answer": "問い合わせ内容を受け付けました: 社員証を紛失しました"
}
```

---

### 確認ポイント

| 確認項目 | 期待する結果 |
| --- | --- |
| `GET /` のResponse bodyに `"API is running"` が含まれる | ✅ APIが起動できている |
| `POST /analyze` のResponse codeが `200` になる | ✅ 正常にリクエストを受け取れている |
| Response bodyに `category`, `priority`, `answer` が含まれる | ✅ 正しいJSONを返せている |
| `question` を空文字にして送信すると `422` エラーになる | ✅ 入力チェックが機能している |

> **補足：** Response codeの `422 Unprocessable Entity` は「送ったデータの形が間違っている」という意味です。`question` フィールドが抜けていたり、文字列以外を送ったときに自動で返されます。

---

# 7. StreamlitからAPIを呼び出す

> **重要：** FastAPIのターミナルは起動したまま、**別のターミナルを新しく開いて**以下の作業をしてください。FastAPIを止めてしまうとStreamlitからAPIを呼び出せません。

`frontend/app.py` を以下のように書き換えます（ミニ課題1で作成したファイルを修正します）。

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
            "http://127.0.0.1:8001/analyze",
            json={"question": question},
            timeout=30
        )
        result = response.json()
        st.write("カテゴリ:", result["category"])
        st.write("緊急度:", result["priority"])
        st.write("回答案:", result["answer"])
```

Streamlitを（別ターミナルで）起動します。

```powershell
uv run streamlit run frontend/app.py
```

# 8. 動作確認

- [ ] FastAPIを起動できる。

- [ ] http://127.0.0.1:8001/docs が開く。

- [ ] Streamlitから問い合わせを送信できる。

- [ ] カテゴリ、緊急度、回答案が画面に表示される。

# 9. よくあるエラー

| 現象 | 確認すること |
| --- | --- |
| `[WinError 10013]` ソケットへのアクセスが拒否された | ポート8000がWindowsのファイアウォールや他のプロセスに占有されている。本資料ではポート8001を使用しているため、コマンドに `--port 8001` が指定されているか確認する。 |
| Connection refused | FastAPIが起動しているか確認する。FastAPIとStreamlitは別のターミナルで同時に起動する必要がある。 |
| ModuleNotFoundError: No module named 'backend' | uvicornをプロジェクトルート（ai-inquiry-appフォルダ）から実行しているか確認する。 |
| ModuleNotFoundError: No module named 'fastapi' | uv pip install fastapi uvicorn requests pydantic を実行したか確認する。 |
| 404 Not Found | URLやエンドポイント名 /analyze が間違っていないか確認する。 |

# 10. 本番設計につながる問い

- [ ] 本番アプリでは、どのAPIが必要か。

- [ ] 問い合わせ登録API、一覧取得API、詳細取得APIは分けるべきか。

- [ ] APIが失敗したとき、画面には何を表示すべきか。

# 11. 振り返り

このミニ課題で学んだこと：

- 画面（Streamlit）と処理（FastAPI）を別々のプログラムに分ける考え方を理解した。
- POSTリクエストで問い合わせ文を送り、JSONレスポンスとして結果を受け取る流れを確認した。
- Swagger（/docs）を使ってAPIを画面から確認・テストする方法を学んだ。
- FastAPIとStreamlitを同時に起動して連携させる手順を確認した。

# 参考リンク

FastAPI First Steps: https://fastapi.tiangolo.com/tutorial/first-steps/
