# Section 3: Optimization Prompts

## Phase 1: Baseline Prompt (Zero-shot)

### English Summary
**Role**: You are an AI that extracts Protected Health Information (PHI) from medical text.  
**Instruction**: Scrutinize the input medical text and detect all information corresponding to the target categories below, outputting in JSON format.  
**Target Categories**: Patient Name, Date of Birth, Age, Gender, Patient ID, Phone Number, Address, Hospital Name, Doctor Name, Department, Examination Date/Time, Admission/Discharge Date.  

### Japanese Prompt
```japanese
あなたは医療テキストから個人識別情報（PHI）を抽出するAIです。

# 指示
入力された医療テキストを精査し、米国のHIPAA（医療保険の相互運用性と説明責任に関する法律）に準拠して、以下の「対象カテゴリ」に該当する情報をすべて検出し,JSON形式で出力してください。

# 抽出対象カテゴリ
- 患者名
- 生年月日
- 年齢
- 性別
- 患者ID
- 電話番号
- 住所
- 病院名
- 医師名
- 診療科
- 検査日時
- 入退院日

# 出力形式
{
  "患者名": ["値1", "値2"],
  "医師名": ["値A", "ValueB"],
  ...
}
※ 該当するPHIがないカテゴリは、空の配列 [] にしてください。
```

## Phase 2: Capacity-Matched Prompts

### High-Performance Group (Score >75/90)
**Models**: gemma3:12b, gemma3:27b, mistral-small3.2:24b, phi4:14b, gpt-oss:20b, EZO2.5-gemma-3

```japanese
# 命令 (Instruction)
あなたは医療テキストからPHIを抽出するAIです。あなたのタスクは、後続の**マスキング処理**で利用するため、テキストに含まれるPHIの**正確な文字列**と**カテゴリ**を特定し、JSONリストとして出力することです。

# 抽出ルール (Rules)
1. **原文の完全一致:** 抽出する文字列（`value`）は、**テキストに書かれている通り、一字一句変更せずに**抜き出してください。
2. **完全な網羅:** 下記「対象カテゴリ」のPHIを**一つも見逃さないこと**を最優先してください。
3. **文脈による分類:** 肩書きや前後の文脈をヒントに、最も適切なカテゴリ（`type`）を判断してください。

# 対象カテゴリ (Categories)
- `NAME` (患者名, 連絡先氏名)
- `DOCTOR` (医師名)
- `DATE` (日付)
- `AGE` (年齢)
- `GENDER` (性別)
- `ID` (患者ID)
- `PHONE` (電話番号)
- `ADDRESS` (住所)
- `FACILITY` (病院名)
- `DEPARTMENT` (診療科、および医師の肩書き)

# 出力形式 (Output Format)
{"phi_list": [
  {"type": "NAME", "value": "田中太郎"},
  {"type": "DOCTOR", "value": "佐藤"},
  ...
]}
```

### Lightweight/Potential Group (Score <75/90)
**Models**: llama3.2:3b, llama3.1:8b, llama4:16x17b(!), llama-swallow-8b, command-r:35b, qwen3:32b, elyza-llama3-8b, Swallow-8B-v0.5

```japanese
# 命令 (Instruction)
あなたは医療テキストからPHIを抽出するAIです。テキスト中にある全てのPHIを、指定されたカテゴリ（`type`）と文字列（`value`）のペアとして、JSONリスト形式で出力してください。

# 抽出ルール (Rules)
1. **原文のまま抽出:** `value`には、テキストに書かれている文字列を**一字一句変更せず**にそのまま入れてください。
2. **全て見つける:** 下の「対象カテゴリ」に含まれる情報を、**一つも見逃さず**に全て検出してください。
3. **文脈で分類:** 「担当医：山田」なら「山田」を`DOCTOR`に分類してください。

# 出力形式 (Output Format)
{"phi_list": [
  {"type": "NAME", "value": "田中太郎"},
  {"type": "DOCTOR", "value": "山田"}
]}
```

## Phase 3: Model-Specific Prompts

### Mistral-Small-3.2 Specific Prompt
**Main Optimizations**:
- Enhanced time extraction rules for concatenated date-time patterns
- Improved kana name recognition through contextual examples
- Added 10 examples of temporal expression variations

### EZO-2.5-Gemma-3 Specific Prompt
**Main Optimizations**:
- Explicit "One item per JSON entry" rule
- Anti-hallucination constraints
- Information separation improvements to prevent category mixing

## Phase 4: Self-Refine QA Prompts

### Mistral-Small-3.2 QA Prompt
```japanese
【最重要原則】
**あなたの初期出力は非常に優秀です。99%は完璧であり、修正は不要です。**
間違いを探すのではなく、「もし万が一、ごく稀な間違いがあった場合に備える」という観点で、以下のチェックリストを**確認のみ**してください。
**少しでも迷ったら、絶対に初期出力を変更しないでください。**

# QAチェックリスト（確認項目）
1. **【過剰検出の最終チェック】**
2. **【カテゴリ分類の最終チェック】**
3. **【情報混入の最終チェック】**

# 最終指示
* 上記3つのチェック項目に**明白な誤り**があった場合のみ、最小限の修正を行った最終版JSONを出力してください。
* **修正点が一つもなかった場合は、[AIの初期出力]をそのまま最終版として出力してください。**
```

### EZO-2.5-Gemma-3 QA Prompt
```japanese
【最重要原則】
**あなたの仕事は「見逃し（False Negative）を発見して追加する」こと、そして「情報混入を分割する」ことです。**
それ以外の修正（特に、既に正しく抽出できている項目の`value`の変更）は、原則として禁止します。

# QAチェックリスト（修正・追加項目）
1. **【網羅性の強化：見逃し項目の追加】**
   * **かな氏名:** 漢字氏名の直後にある括弧書きのかな (`NAME`)。**最優先で確認してください。**
   * **イニシャル:** `T.K.氏`のような形式 (`NAME`)。

2. **【情報分離の徹底：混入項目の分割】**

3. **【過剰検出の抑制：推測情報の削除】**

# 最終指示
* 上記のQAチェックリストに基づき、修正・追加・削除を行った**最終版のJSONオブジェクトのみ**を出力してください。
```

## Phase 5: Final Validation

The final validation phase used the same prompts from Phase 3 (model-specific) combined with the optimized QA prompts developed during Phase 4. Testing was performed on 60 held-out cases to ensure reproducibility and generalization.

## Key Findings

### Performance-Based Grouping (Phase 2)
- High-Performance Group: +4.2 points improvement (79.0→83.2)
- Lightweight Group: +8.1 points improvement (68.4→76.5)
- The larger improvement in the Lightweight group validated our targeted approach

### Self-Refine Threshold (Phase 4-5)
- Models with baseline <87-88 points: +6.9 points average improvement
- Models with baseline >87-88 points: No significant improvement (-0.0 points)
- Critical finding: Self-Refine effectiveness has a clear performance threshold

### Model-Specific Observations
- **Llama4:16x17b (MoE)**: Despite 272B parameters, achieved only 77/90 points
- **Mistral-Small-3.2**: Best response to optimization (+6.9 points with Self-Refine)
- **EZO-2.5-Gemma-3**: Already optimized, no benefit from Self-Refine
- **Gemma-3/Phi-4**: Architectural limitations prevented improvement

## Reproducibility Notes

All prompts are provided in their original Japanese with English summaries. The complete prompt evolution, including failed attempts and refinements, demonstrates the iterative nature of prompt engineering for medical NLP tasks. Researchers can reproduce our results using:

1. **Ollama v0.11.5** for model management
2. **Temperature = 0** for deterministic outputs
3. **Top-p = 1.0** for full probability distribution
4. **Fixed random seeds** for consistency

The effectiveness of each prompt depends heavily on model architecture and baseline capabilities, emphasizing the importance of model-specific optimization rather than one-size-fits-all approaches.
