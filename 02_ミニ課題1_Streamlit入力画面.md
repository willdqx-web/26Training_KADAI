# ミニ課題1 Streamlit入力画面

**Pythonだけで簡単な画面を作る**

| このミニ課題のゴール<br>・社員から総務への問い合わせ文を入力できる画面を作る。<br>・ボタンを押したら、入力内容を画面に表示する。<br>・未入力の場合はエラーメッセージを表示する。 |
| --- |

# 1. 完成イメージ

| 画面タイトル: 総務問い合わせ入力<br>入力欄: 問い合わせ内容<br>ボタン: 確認する<br>表示: 入力された問い合わせ内容 |
| --- |

# 2. 用語

| 用語 | 説明 |
| --- | --- |
| Streamlit | Pythonだけで簡単なWeb画面を作るためのライブラリです。 |
| 入力欄 | ユーザーが文字を入力する部品です。 |
| ボタン | 押したときに処理を実行する部品です。 |
| バリデーション | 入力内容が正しいかを確認することです。今回は空欄チェックを行います。 |
| ウィジェット | Streamlitで画面に配置する部品（入力欄・ボタン・セレクトボックスなど）の総称です。 |

# 3. ライブラリを追加する

```powershell
uv pip install streamlit
```

# 4. ファイルを作成する

```powershell
mkdir frontend
notepad frontend/app.py
```

# 5. サンプルコード

```python
import streamlit as st

st.title("総務問い合わせ入力")
st.write("社員から総務への問い合わせを入力してください。")

question = st.text_area("問い合わせ内容", height=160)

if st.button("確認する"):
    if question.strip() == "":
        st.error("問い合わせ内容を入力してください。")
    else:
        st.success("入力内容を受け付けました。")
        st.subheader("入力された内容")
        st.write(question)
```

# 6. Streamlitのよく使う機能一覧

総務問い合わせアプリで役立つStreamlitの主な機能を紹介します。実際に書いて動かすことで理解を深めてください。

## 6-1. 入力ウィジェット

```python
# 1行テキスト
name = st.text_input("氏名")

# 複数行テキスト（問い合わせ内容など）
question = st.text_area("問い合わせ内容", height=160)

# ドロップダウン
category = st.selectbox("カテゴリ", ["休暇", "給与", "福利厚生", "その他"])

# ラジオボタン
priority = st.radio("緊急度", ["高", "中", "低"])

# チェックボックス
agree = st.checkbox("内容を確認しました")
```

## 6-2. メッセージ表示

```python
st.success("登録が完了しました。")              # 緑
st.error("入力に誤りがあります。")               # 赤
st.warning("この操作は取り消せません。")          # 黄
st.info("担当者が回答するまでお待ちください。")    # 青
```

## 6-3. テキスト・見出し

```python
st.title("ページタイトル")
st.header("大見出し")
st.subheader("小見出し")
st.write("本文テキスト")
st.markdown("**太字**や *斜体* も書けます")
```

## 6-4. データ表示

```python
import pandas as pd

inquiries = [
    {"id": 1, "question": "有給の申請方法は？", "category": "休暇"},
    {"id": 2, "question": "健康保険証を紛失した", "category": "保険"},
]
df = pd.DataFrame(inquiries)

st.dataframe(df)   # 操作可能なテーブル（列幅調整・ソートができる）
st.table(df)       # 静的なテーブル
```

## 6-5. レイアウト

```python
# 列分割（入力フォームと結果を横並びにするときなど）
col1, col2 = st.columns(2)
with col1:
    st.write("左側：入力フォーム")
with col2:
    st.write("右側：回答結果")

# 折りたたみ（詳細情報を隠しておくときなど）
with st.expander("回答の詳細を見る"):
    st.write("ここに詳細内容が入ります。")

# サイドバー（ページ切り替えメニューなど）
st.sidebar.title("メニュー")
page = st.sidebar.radio("ページ", ["問い合わせ入力", "履歴一覧"])
```

## 6-6. 処理中の表示（spinner）

API呼び出しなど時間のかかる処理に使います。

```python
with st.spinner("回答を生成中..."):
    result = analyze_with_gemini(question)  # 時間のかかる処理
st.success("完了しました。")
```

## 6-7. フォーム（一括送信）

入力欄が複数あるとき、すべて入力してからまとめて送信できます。

```python
with st.form("inquiry_form"):
    name = st.text_input("氏名")
    question = st.text_area("問い合わせ内容", height=160)
    category = st.selectbox("カテゴリ", ["休暇", "給与", "福利厚生", "その他"])
    submitted = st.form_submit_button("送信する")

if submitted:
    if question.strip() == "":
        st.error("問い合わせ内容を入力してください。")
    else:
        st.success(f"{name}さんの問い合わせを受け付けました。")
```

> **補足：** `st.form` を使うと、フォーム内の入力を変更しても送信ボタンを押すまでStreamlitが再実行されません。入力ウィジェットが多い画面では `st.form` の使用を検討してください。

# 7. 起動方法

```powershell
uv run streamlit run frontend/app.py
```

> **注意：** `uv run` を使って実行します。`streamlit run` を直接実行すると、仮想環境外のPythonが使われることがあります。
>
> ブラウザに http://localhost:8501 が自動で開きます。開かない場合はターミナルに表示されたURLをコピーして手動で開いてください。
>
> 停止するときはターミナルで `Ctrl+C` を押します。

# 8. 動作確認

- [ ] http://localhost:8501 が開く。

- [ ] 問い合わせ文を入力できる。

- [ ] ボタンを押すと入力内容が表示される。

- [ ] 空欄のままボタンを押すとエラーが表示される。

- [ ] selectbox・radioなど別のウィジェットを試した。

- [ ] st.spinner を使って処理中の表示ができる。

- [ ] st.form を使ってまとめて送信できる。

# 9. よくあるエラー

| 現象 | 確認すること |
| --- | --- |
| streamlitコマンドが見つからない | uv pip install streamlitを実行したか確認する。起動はuv runを使う。 |
| 画面が開かない | ターミナルに表示されたURLを確認する。既に別ポートで起動していないか確認する。 |
| 日本語が文字化けする | ファイルをUTF-8で保存する。 |
| ウィジェットの値が消える | ボタンを押すとStreamlitは上から再実行される仕様のため、st.session_stateを使って値を保持する。 |

# 10. 本番設計につながる問い

- [ ] 問い合わせ入力画面には、問い合わせ内容以外に何を入力させるべきか。

- [ ] 未入力以外に、どのような入力エラーを確認すべきか。

- [ ] 社員が迷わない画面にするために、どんな説明文が必要か。

- [ ] 問い合わせ一覧画面と入力画面は、1つの画面に収めるかページを分けるか。

# 11. 振り返り

このミニ課題で学んだこと：

- Pythonだけで、HTMLやCSSを書かずにWeb画面を作れることを確認した。
- `st.text_area`、`st.button`、`st.error`、`st.success` などの部品を組み合わせて画面を構成する方法を学んだ。
- 入力内容のバリデーション（空欄チェック）を実装した。
- `st.selectbox`、`st.radio` などの選択ウィジェットや、`st.form` によるまとめ送信の使い方を確認した。
- `st.spinner` を使って、時間のかかる処理中にユーザーへフィードバックする方法を学んだ。
- `uv run` を使って仮想環境内でStreamlitを起動する方法を確認した。

# 参考リンク

Streamlit input widgets: https://docs.streamlit.io/develop/api-reference/widgets

Streamlit layouts: https://docs.streamlit.io/develop/api-reference/layout

Streamlit status elements: https://docs.streamlit.io/develop/api-reference/status
