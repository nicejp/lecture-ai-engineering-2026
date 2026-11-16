# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
  「AIE2026の講座の宿題にある「3レベル」とは何ですか？」で比べた。app.askは講座のことを知らないので「可能性が非常に高いです」と前置きして、レベル1〜3の一般論を長々と書いた。app.agentは用語集を引いて、基礎・実装・発展の3つで、どれか1つで完了、と用語集の定義どおりに短く答え、最後に「第1回」を付けた。
- `--trace`で見えたツール呼び出し（ツール名と引数）：
  上の質問では`search_glossary({'query': '3レベル'})`が1回。「LoRAとQLoRAの違いは？」では`search_glossary({'query': 'LoRA'})`と`search_glossary({'query': 'QLoRA'})`の2回で、用語を1つずつ調べていた。
- 1つの質問でのLLM呼び出し回数（`requests`）：
  どちらの質問も`requests=2`。LoRA/QLoRAは`tool_calls=2`だったがrequestsは2のままだった。
- 追加したツールと、docstringに書いた説明：
  `make_quiz(lecture: int)`。指定した回の用語からrandom.choiceで1つ選び、`{"question": "「用語」の意味は？", "answer": 定義}`を返す。用語が無い回は、用語がある回の一覧を返す。docstringは「指定した回の用語から1つを選び、確認問題と答えを返す。「第2回の確認問題を出して」のように、復習用の問題を求められたときに使う。」。`--list-tools`に出ること、「第2回の内容から確認問題を1問出して」で`make_quiz({'lecture': 2})`が呼ばれること、`pytest -q`が11 passedになることを確認した。

## 宿題

- 取り組んだレベル：基礎
- やったこと：
  1か所だけ変えて、同じ質問を`--trace`付きで2回ずつ実行した（gemini-3.5-flash-lite）。試したのは次の2つで、終わるたびに元に戻した。
  - B：INSTRUCTIONSから「必ずsearch_glossaryで用語集を確認する」の行を消す
  - C：search_glossaryのdocstringの「用語を1つだけ入れる（例: "LoRA"）」を「まとめて入れる（例: "LoRA QLoRA"）」にする
  質問は「LoRAとQLoRAの違いは？」。Bは「lecNN-startタグとは？」でも試した。
- 結果：

  | 条件 | ツール呼び出し | requests | tool_calls | input_tokens |
  |---|---|---|---|---|
  | 元のまま | `search_glossary('LoRA')`、`search_glossary('QLoRA')` | 2 | 2 | 1196 |
  | B指示の行を削除 | 元のままと同じ | 2 | 2 | 1148 |
  | C docstringを「まとめて」 | `search_glossary('LoRA QLoRA')` | 2 | 1 | 891 |

  Bは何も変わらなかった。lecNN-startタグの質問でも検索してから答えた。
  Cは検索が1回になり、入力トークンも減った。`search_glossary("LoRA QLoRA")`を直接呼んでみると、LoRAとQLoRAの両方が返っていた。
- 気づき・疑問：
  - Bで検索が止まらなかったのは、docstringにも「答える前に使うこと」と書いてあるからだと思う。両方から消せば検索しなくなるのかもしれないが、そこまでは試していない。
  - `search_glossary("LoRA")`だけでQLoRAも部分一致で返ってくるので、元の2回目の検索は実はいらなかった。Cがうまくいったのも、ツール側が部分一致で複数拾う作りだったから。
  - 検索が2回でも1回でもrequestsは2だった。2回の検索は1回の応答でまとめて頼まれていたようだ。requestsが3以上になるのはどんな質問のときなのかが分からなかった。
