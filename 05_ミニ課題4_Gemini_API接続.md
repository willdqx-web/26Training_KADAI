# ミニ課題4 Gemini API接続

**クラウドLLMに問い合わせ文を送って回答を得る**

| このミニ課題のゴール<br>・Gemini APIキーを.envで管理する。<br>・PythonからGemini APIを呼び出す。<br>・問い合わせ文からカテゴリ、緊急度、回答案を生成する。 |
| --- |

# 1. 完成イメージ

問い合わせ文をGeminiに送ると、カテゴリ・緊急度・回答案を含む回答テキストが返ってくる。

```
入力: 「健康保険証を紛失しました。再発行の手続きを教えてください。」
　↓
Gemini API
　↓
出力: カテゴリ、緊急度、回答案を含むテキスト
```

# 2. Gemini APIとは

Gemini APIは、GoogleのLLMをプログラムから呼び出すための仕組みです。今回のアプリでは、社員の問い合わせ文をGeminiに送り、分類と回答案の作成を行います。

# 3. 用語

| 用語 | 説明 |
| --- | --- |
| Gemini API | GoogleのLLM（大規模言語モデル）をプログラムから呼び出すためのAPIです。 |
| APIキー | APIを使うための認証情報です。パスワードと同様に扱い、外部に漏らしてはいけません。 |
| .env | 環境変数をファイルに書いておく仕組みです。APIキーなどの秘密情報を管理します。 |
| 環境変数 | プログラムの外で設定する変数です。ソースコードに値を直接書かずに済みます。 |
| dotenv | `.env` ファイルを読み込むためのPythonライブラリです。 |
| プロンプト | LLMに送る指示文・質問文のことです。 |

# 4. ライブラリを追加する

```powershell
uv pip install google-genai python-dotenv
```

# 5. APIキーを取得する

Gemini APIを使うには、Googleアカウントでキーを発行する必要があります。以下の手順で取得してください。

## 手順1: Google AI Studioにアクセスする

ブラウザで以下のURLを開きます。

```
https://aistudio.google.com/
```

Googleアカウントでサインインしていない場合はサインインを求められます。

---

## 手順2: APIキーを発行する

1. 左側メニューの **「Get API key」** をクリックする
2. **「Create API key」** ボタンをクリックする
3. プロジェクトの選択を求められた場合は **「Create API key in new project」** を選ぶ
4. 発行されたAPIキーが画面に表示される（例: `AIzaSy...` から始まる長い文字列）
5. **「Copy」** ボタンでクリップボードにコピーする

> **重要：** APIキーはこの画面を閉じると再表示できません（再発行は可能）。必ずコピーしてからページを離れてください。

---

## 手順3: 無料枠を確認する

Gemini APIは無料枠（Free tier）が用意されており、開発・学習目的であれば費用がかからない範囲で使えます。

| モデル | 無料枠のリクエスト制限（目安） |
| --- | --- |
| gemini-2.5-flash-lite | 1分あたり30リクエスト / 1日あたり1,500リクエスト |

> **補足：** 無料枠は変更される場合があります。最新の情報は [Google AI Studioの料金ページ](https://ai.google.dev/gemini-api/docs/rate-limits) で確認してください。

---

## 手順4: APIキーを安全に管理する

取得したAPIキーは次のルールで管理します。

| ルール | 理由 |
| --- | --- |
| コードに直接書かない | GitHubに上げると全世界に公開されてしまう |
| `.env` ファイルに書いて `.gitignore` に追加する | ローカルだけに留め、Gitの管理対象から外す |
| 他人に共有しない | 不正利用されると自分のアカウントに課金される |
| 漏洩した場合はすぐ削除・再発行する | Google AI Studioの「API keys」画面からいつでも操作できる |

---

# 6. .envを作成する

`.env` はAPIキーなどの秘密情報をソースコードと切り離して管理するための設定ファイルです。プロジェクトのルートフォルダに置き、Pythonが起動時に読み込みます。

## 手順1: .envファイルを作成する

プロジェクトルート（`ai-inquiry-app` フォルダ）で以下を実行します。

```powershell
notepad .env
```

メモ帳が開いたら以下の内容を入力して保存します。`取得したAPIキー` の部分は、セクション5で発行した実際のキー文字列に置き換えてください。

```
GEMINI_API_KEY=取得したAPIキー
GEMINI_MODEL=gemini-2.5-flash-lite
```

**記入例（実際のキーはこのような形式になります）**

```
GEMINI_API_KEY=AIzaSyABC123XYZexampleKeyString456789
GEMINI_MODEL=gemini-2.5-flash-lite
```

> **注意：** `=` の前後にスペースを入れないでください。`GEMINI_API_KEY = xxx` のようにスペースがあると読み込みに失敗します。

---

## 手順2: .gitignoreに追加してGit管理から除外する

`.gitignore` ファイルがプロジェクトルートに存在する場合は開いて `.env` の行を追加します。存在しない場合は新規作成します。

```powershell
notepad .gitignore
```

以下の内容を追記して保存します。

```
.env
```

> **重要：** この設定をしないと `git push` のタイミングでAPIキーがGitHubに公開されてしまいます。必ず設定してください。

---

## 手順3: ファイルの配置を確認する

作成後のフォルダ構成は以下のようになります。

```
ai-inquiry-app/          ← プロジェクトルート
├── .env                 ← 今作成したファイル（Gitに上げない）
├── .gitignore           ← .envを除外する設定を追記した
├── backend/
│   ├── __init__.py
│   ├── main.py
│   └── storage.py
└── frontend/
    └── app.py
```

---

## 手順4: 読み込めているか確認する

以下のコマンドでAPIキーが正しく読み込まれているか確認できます。

```powershell
uv run python -c "from dotenv import load_dotenv; import os; load_dotenv(); print(os.getenv('GEMINI_API_KEY'))"
```

APIキーの文字列が出力されれば成功です。`None` と表示された場合は `.env` の場所または書き方を見直してください。

| 症状 | 確認ポイント |
| --- | --- |
| `None` と出力される | `.env` がプロジェクトルートにあるか確認する |
| `None` と出力される | キー名のスペルミスがないか確認する（`GEMINI_API_KEY` は大文字） |
| `None` と出力される | `=` の前後にスペースがないか確認する |

> **重要**
> - APIキーはパスワードのようなものです。コードに直接書かないでください。
> - `.env` は `.gitignore` に入れて、Gitに上げないでください。

# 7. Gemini接続コード

```python
import os
from dotenv import load_dotenv
from google import genai

load_dotenv()

client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))
MODEL = os.getenv("GEMINI_MODEL", "gemini-2.5-flash-lite")


def analyze_with_gemini(question: str) -> str:
    prompt = f"""
あなたは総務部門の問い合わせ一次回答担当です。
社員からの問い合わせを読み、以下の3点を日本語で返してください。

1. カテゴリ
2. 緊急度（高・中・低）
3. 回答案

問い合わせ:
{question}
"""
    response = client.models.generate_content(
        model=MODEL,
        contents=prompt
    )
    return response.text
```

> **補足：本番実装に向けて**
> このミニ課題ではGeminiの回答を1つの文字列として受け取っています。
> 本番アプリではカテゴリ・緊急度・回答案をそれぞれ別のフィールドとして保存する必要があります。
> プロンプトでJSON形式の出力を指示する方法（例: 「JSON形式で返してください」）を要件定義・設計フェーズで検討しましょう。

# 8. 試す問い合わせ文

```
健康保険証を紛失してしまいました。
病院を受診する予定があるため、再発行の手続きと急ぎの対応方法を教えてください。
```

# 9. 動作確認

- [ ] .envからAPIキーを読み込める。

- [ ] Geminiから回答が返ってくる。

- [ ] カテゴリ、緊急度、回答案が含まれている。

- [ ] 回答が社内ルールを断定しすぎていない。

- [ ] API失敗時に画面へ分かりやすいエラーを表示できる。

# 10. よくあるエラー

| 現象 | 確認すること |
| --- | --- |
| APIキーエラー | .envにGEMINI_API_KEYが設定されているか確認する。 |
| ModuleNotFoundError: google | uv pip install google-genai python-dotenvを実行したか確認する。 |
| 回答が期待通りでない | プロンプトに出力形式や役割を明確に書く。 |

# 11. 本番設計につながる問い

- [ ] Geminiの回答をそのまま保存してよいか。

- [ ] カテゴリを自由入力にするか、決められたカテゴリから選ばせるか。

- [ ] LLMが失敗したとき、問い合わせ登録自体はできるようにするか。

- [ ] Geminiの回答文字列をカテゴリ・緊急度・回答案に分離するにはどうすればよいか。

# 12. 振り返り

このミニ課題で学んだこと：

- `.env` と `python-dotenv` を使ってAPIキーをソースコードから分離する方法を学んだ。
- PythonからGemini APIを呼び出し、テキスト回答を受け取る基本的な流れを確認した。
- プロンプトの書き方によって回答内容が変わることを体験した。
- LLMの回答は必ずしも正しいとは限らないため、回答品質を評価することの重要性を理解した。

# 参考リンク

Gemini API Quickstart: https://ai.google.dev/gemini-api/docs/quickstart

Gemini API Models: https://ai.google.dev/gemini-api/docs/models
