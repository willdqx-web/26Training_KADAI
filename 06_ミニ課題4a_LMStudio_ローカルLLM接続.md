# ミニ課題4α LM StudioローカルLLM接続

**ローカルPC上のLLMをAPIとして呼び出す**

| このミニ課題のゴール<br>・LM StudioでローカルLLMを起動する。<br>・PythonからLM StudioのOpenAI互換APIへ接続する。<br>・Gemini APIとの違いを説明できるようにする。 |
| --- |

# 1. 位置づけ

この課題は本番アプリの必須機能ではなく、比較学習用です。PC性能やモデルによって動作速度が大きく変わるため、まずは短い問い合わせ文で接続確認を行います。

```
Gemini API: クラウド上のLLMを呼び出す
LM Studio:  自分のPC上で動くLLMを呼び出す
```

# 2. 完成イメージ

LM Studioでローカルサーバーを起動した後、Pythonスクリプトから問い合わせ文を送ると、ローカルLLMから回答テキストが返ってくる。同じ問い合わせ文でGeminiとの回答を比較できる状態を目指す。

# 3. 用語

| 用語 | 説明 |
| --- | --- |
| LM Studio | ローカルPCでLLMを動かすためのアプリです。 |
| ローカルLLM | 自分のPC上で動く言語モデルです。外部API料金は基本的にかかりません。 |
| OpenAI互換API | OpenAI用のクライアントから似た形式で呼び出せるAPIです。 |
| base_url | APIの接続先URLです。LM Studioでは http://127.0.0.1:1234/v1 を使います。 |

# 4. LM Studio側の準備

## 手順1: LM Studioをインストールする

本課題では **LM Studio 0.4.12 (Build 1)** を使用します。

LM Studioがまだインストールされていない場合は、公式サイトからダウンロードします。

```
https://lmstudio.ai/
```

「Download」ボタンからWindows版のインストーラーを取得し、指示に従ってインストールします。

---

## 手順2: LM Studioを起動してモデルを検索する

1. LM Studioを起動する
2. 左側メニューの **虫眼鏡アイコン（Discover）** をクリックする
3. 検索欄に **`qwen2.5-3b`** と入力する
4. 検索結果に **`Qwen2.5-3B-Instruct-GGUF`** が表示されることを確認する

> **補足：** 初回起動時にはモデルのキャッシュ場所の設定を求められる場合があります。空き容量が十分なドライブを選んでください（モデル1つあたり1〜5GB程度）。

---

## 手順3: モデルをダウンロードする

本課題では **`Qwen2.5-3B-Instruct-GGUF`** を使用します。

| モデル名 | パラメータ数 | ファイルサイズ目安 | 推奨環境 |
| --- | --- | --- | --- |
| `Qwen2.5-3B-Instruct-GGUF` | 3B | 約2GB | RAM 8GB以上 |

1. 検索結果から **`Qwen2.5-3B-Instruct-GGUF`** をクリックして詳細を開く
2. **「Download」** ボタンをクリックする（量子化バリアントが複数表示される場合は `Q4_K_M` など中程度のものを選ぶ）
3. ダウンロードが完了するまで待つ（進捗バーで確認できます）

---

## 手順4: モデルを読み込む

1. 左側メニューの **チャットアイコン（Chat）** をクリックする
2. 画面上部のモデル選択欄（`Select a model to load` と表示されている部分）をクリックする
3. **`Qwen2.5-3B-Instruct-GGUF`** を選択する
4. モデルの読み込みが完了するまで待つ（下部にプログレスバーが表示されます）
5. 読み込み完了後、チャット欄にメッセージを送って動作確認する

---

## 手順5: ローカルサーバーを起動する

LM Studio 0.4.12 では、Developer セクションの左サイドバーから操作します。

1. 左側メニューの **`<->`アイコン（Developer）** をクリックする
2. サイドバーの **「Local Server」** をクリックする
3. 画面上部の **「Status」トグルスイッチ** をオンにする（緑色になれば起動済み）
4. 右上に **`Reachable at: http://127.0.0.1:1234`** と表示されれば起動成功
5. まだモデルを読み込んでいない場合は右上の **「+ Load Model」** ボタンをクリックし、`Qwen2.5-3B-Instruct-GGUF` を選択する
6. 「Loaded Models」欄に **`READY`** バッジとともにモデル名が表示されれば準備完了

---

## 手順6: 接続先URLを確認する

Local Server 画面の右上に表示されている「Reachable at」のURLを確認します。

```
http://127.0.0.1:1234
```

Pythonコードでは末尾に `/v1` を付けた以下のURLを `base_url` に指定します。

```
http://127.0.0.1:1234/v1
```

デフォルトのポート（1234）から変更している場合は、Pythonコードの `base_url` も合わせて修正してください。

> **補足：** サーバーを起動したままにしておかないとPythonからの接続が失敗します。Pythonスクリプトを実行する間はLM Studioのウィンドウを閉じないでください。

# 5. ライブラリ追加

```powershell
uv pip install openai pydantic
```

# 6. Python接続コード

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://127.0.0.1:1234/v1",
    api_key="lm-studio"
)

response = client.chat.completions.create(
    model="qwen2.5-3b-instruct",
    messages=[
        {
            "role": "system",
            "content": (
                "あなたは総務部門の問い合わせ一次回答担当です。\n"
"社員からの問い合わせを読み、category・priority・answerを判定してください。\n"
"- category: 問い合わせのカテゴリを「勤怠」「休暇」「給与」「交通費」「出張」「その他」のいずれかで返す \n"
"- priority: 緊急度を「高」「中」「低」のいずれかで返す \n"
"- answer: 社員への一次回答案（日本語、100文字程度）"
            )
        },
        {
            "role": "user",
            "content": (
                "本日の出勤打刻を忘れてしまいました。勤務は通常通り9時から開始しています。勤怠システム上でどのように修正すればよいでしょうか。"
            )
        }
    ],
    temperature=0.3
)

print(response.choices[0].message.content)
```

# 7. 構造化出力（category・priority・answer を個別に取得する）

セクション6のコードでは回答を1つの文字列として受け取っています。LM Studio の OpenAI 互換 API は `response_format` で JSON スキーマを指定でき、05のGemini版と同様に構造化データとして取得できます。

## コード

```python
import json
from openai import OpenAI
from pydantic import BaseModel

client = OpenAI(
    base_url="http://127.0.0.1:1234/v1",
    api_key="lm-studio"
)

MODEL = "qwen2.5-3b-instruct"


class InquiryResult(BaseModel):
    category: str
    priority: str
    answer: str


response = client.chat.completions.create(
    model=MODEL,
    messages=[
        {
            "role": "system",
            "content": (
                "あなたは総務部門の問い合わせ一次回答担当です。\n"
                "社員からの問い合わせを読み、カテゴリ・緊急度・回答案を判定してください。\n\n"
                "- category: 問い合わせのカテゴリ（例: 休暇、備品、給与、保険、その他）\n"
                "- priority: 緊急度を「高」「中」「低」のいずれかで返す\n"
                "- answer: 社員への一次回答案（日本語、2〜3文程度）"
            )
        },
        {
            "role": "user",
            "content": (
                "健康保険証を紛失してしまいました。\n"
                "病院を受診する予定があるため、再発行の手続きと急ぎの対応方法を教えてください。"
            )
        }
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "InquiryResult",
            "schema": InquiryResult.model_json_schema()
        }
    },
    temperature=0.3
)

result = InquiryResult.model_validate_json(response.choices[0].message.content)
print(result.category)   # → 保険
print(result.priority)   # → 高
print(result.answer)     # → 総務窓口へご連絡ください。…

# FastAPIやJSON保存に渡すときはそのまま辞書にできる
print(result.model_dump())
# → {"category": "保険", "priority": "高", "answer": "..."}
```

## Gemini版との違い

| 観点 | Gemini版（セクション8） | LM Studio版（このセクション） |
| --- | --- | --- |
| スキーマ指定 | `response_schema=InquiryResult` | `response_format={"type": "json_schema", ...}` |
| パース方法 | `response.parsed` | `InquiryResult.model_validate_json(...)` |
| 対応モデル | gemini-2.5-flash-lite など | LM Studio で読み込み済みのモデル |

> **補足：** `response_format` の `json_schema` に対応していないモデルでは空文字やエラーが返ることがあります。その場合は `"type": "json_object"` に変えてプロンプトに「JSON形式で返してください」と明記し、`json.loads()` でパースしてください。

# 8. Geminiとの比較観点

| 観点 | Gemini | LM Studio |
| --- | --- | --- |
| 実行場所 | クラウド | ローカルPC |
| 費用 | API利用料が発生する可能性あり | ローカル実行分は基本無料 |
| 速度 | ネットワークとAPI側に依存 | PC性能とモデルサイズに依存 |
| APIキー | 必要 | 基本不要。クライアント都合でダミー文字列を入れる場合あり |
| 研修での扱い | 本番実装の主軸 | 比較学習・発展課題 |

# 9. 動作確認

- [ ] LM Studioでモデルを読み込めた。

- [ ] ローカルサーバーを起動できた。

- [ ] Pythonから回答を取得できた。

- [ ] 構造化出力でカテゴリ・緊急度・回答案を個別に取得できた。

- [ ] Geminiと同じ問い合わせで回答を比較できた。

- [ ] クラウドLLMとローカルLLMの違いを説明できた。

# 10. よくあるエラー

| 現象 | 確認すること |
| --- | --- |
| Connection refused | LM Studioのローカルサーバーが起動しているか確認する。 |
| モデルが見つからない | LM Studioで読み込んだモデル名を確認する。 |
| 回答が遅い | 軽量モデルを使う。短い問い合わせ文で確認する。 |
| openaiライブラリがない | uv pip install openaiを実行する。 |

# 11. 本番設計につながる問い

- [ ] 本番アプリではGemini固定にするか、LLMを切り替えられるようにするか。

- [ ] ローカルLLMが起動していない場合、画面にどう表示すべきか。

- [ ] 個人情報を含む問い合わせは、クラウドLLMとローカルLLMのどちらに送るべきか。

# 12. 振り返り

このミニ課題で学んだこと：

- LM StudioでローカルLLMを起動し、Pythonから接続する手順を確認した。
- OpenAI互換APIの形式を使うと、クライアントコードをほぼ変えずにLLMを切り替えられることを理解した。
- Gemini APIとローカルLLMの費用・速度・セキュリティの違いを比較した。
- 本番アプリではGeminiを主軸とし、ローカルLLMは比較学習・発展課題として扱う理由を説明できるようになった。

# 参考リンク

LM Studio OpenAI Compatibility: https://lmstudio.ai/docs/developer/openai-compat

LM Studio Developer Docs: https://lmstudio.ai/docs/developer
