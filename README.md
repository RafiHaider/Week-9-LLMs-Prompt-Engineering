# Project 3: AI-Based Mini Autonomous Vehicle Assistant

**Course:** Introduction to Applied AI  
**Semester:** BS 8th Semester  
**Department:** Electronics  
**Week 9:** LLMs and Prompt Engineering

## Team Members and Responsibilities

| Team Member | Role | Main Responsibilities |
|---|---|---|
| Rafi Haider | Member 1 — API Setup, Sampling & Cost Analysis | API configuration, Kaggle Secrets, temperature and top-p experiments, token analysis, and cost estimation |
| Muhammad Bilal | Member 2 — Prompt Engineering & Evaluation | Product facts, SYSTEM_V1 and SYSTEM_V2, Pydantic validation, and evaluation framework |
| Khuzaima Ahmed | Member 3 — Adversarial Testing & Repository Management | Prompt-injection testing, canary verification, submission files, repository organization, and documentation |

---

# Week 9: LLMs and Prompt Engineering

## 1. API Setup and Configuration

Our team selected the Groq API using an OpenAI-compatible client.

- **Provider:** Groq
- **Model:** `openai/gpt-oss-20b`
- **Platform:** Kaggle Notebook
- **API key:** Stored securely in Kaggle Secrets using `GROQ_API_KEY`.

No API keys were hardcoded into notebook cells.

We implemented a `chat()` function with:
- Automatic retries.
- Exponential backoff.
- Random jitter.
- Token usage logging.
- Response-time measurement.

### Smoke Test

The API connection successfully returned:

```text
ready

Input tokens: 76
Output tokens: 37
Response time: 1.35 seconds
```

This confirmed that our API connection and chat function were working.

## 2. Temperature and Top-p Experiments

We explored how temperature and top-p affect token probabilities and response variability.

### Temperature Probability Results

| Temperature | Islamabad | Karachi | Lahore |
|---|---:|---:|---:|
| 0.5 | 0.982 | 0.018 | 0.000 |
| 1.0 | 0.867 | 0.117 | 0.016 |
| 2.0 | 0.665 | 0.245 | 0.090 |

For temperature 2.0 with top-p 0.9:

| Islamabad | Karachi | Lahore |
|---:|---:|---:|
| 0.731 | 0.269 | 0.000 |

### Live Temperature Variability Test

We generated five names at two different temperatures.

**Temperature = 0.0**

```text
[
 'Chai & Chalkboard',
 'Chai & Chalk',
 'Chai & Chalk',
 'Chai & Chalkboard',
 'Chai & Chalk'
]
```

Distinct names: **2 out of 5**

**Temperature = 1.0**

```text
[
 'Campus Chai',
 'Campus Kettle',
 'ChaiU',
 'Chai Scholar',
 'Chai Scholar'
]
```

Distinct names: **4 out of 5**

### Observation

Lower temperature produced more consistent responses, while higher temperature produced greater variety.

We selected temperature 0.0 for our main product task because consistent technical responses are important. However, low temperature does not guarantee factual accuracy.

## 3. Tokenization and Cost Analysis

We compared token usage across English, Urdu, and Roman Urdu.

### Language Comparison

| Language | Characters | Tokens | Characters per Token |
|---|---:|---:|---:|
| English | 34 | 8 | 4.2 |
| Urdu | 44 | 15 | 2.9 |
| Roman Urdu | 51 | 14 | 3.6 |

Urdu required approximately **1.88 times more tokens than English** for the tested sentences.

### Token and Response-Time Statistics

| Measurement | Mean | Median | Maximum |
|---|---:|---:|---:|
| Input tokens | 84.60 | 86.00 | 89.00 |
| Output tokens | 667.47 | 256.00 | 1,739.00 |
| Response time (seconds) | 1.78 | 1.86 | 3.40 |

### Monthly Cost Estimation

We estimated the monthly API cost using the following assumptions:

- Users: 1,000
- Queries per user per day: 20
- Days per month: 30
- Average input tokens per query: 84.60
- Average output tokens per query: 667.47
- Input pricing: USD 0.075 per million tokens
- Output pricing: USD 0.30 per million tokens

Monthly queries:

```text
1,000 × 20 × 30 = 600,000 queries
```

Estimated monthly cost:

| Category | Estimated Cost |
|---|---:|
| Input tokens | USD 3.81 |
| Output tokens | USD 120.14 |
| **Total** | **USD 123.95** |

**Estimated monthly cost: USD 123.95**

This is a hypothetical estimate based on the selected usage assumptions and pricing values.

## 4. Prompt Engineering for Our Product

Our project is an AI-based mini autonomous vehicle using the ESP32-S3-CAM platform.

### Confirmed Product Information

Our product facts include:

- ESP32-S3-CAM platform.
- Built-in camera for capturing road images.
- Camera-based lane and line detection.
- OpenCV image processing on an external laptop or PC.
- Planned Wi-Fi communication between the ESP32-S3 and PC.
- HC-SR04 ultrasonic sensor for obstacle detection and distance measurement.
- Automatic parking designed to use camera and ultrasonic information.
- TB6612FNG motor driver.
- Four DC motors, one for each wheel.
- Rechargeable Li-ion battery.

### Two System Prompts

We developed two prompt versions.

**SYSTEM_V1 — Basic Prompt**

A simple system prompt instructing the assistant to answer questions using the provided product information.

**SYSTEM_V2 — Structured Prompt**

A detailed system prompt containing rules for:

- Using only confirmed product facts.
- Avoiding invented technical specifications.
- Identifying missing information.
- Distinguishing planned features from implemented or tested features.
- Providing supporting fact references.
- Handling unrelated questions.
- Protecting system instructions.
- Returning responses in a fixed JSON structure.

### Pydantic Validation

We created a Pydantic model named `Reply` with four required fields:

- `answer`
- `sources`
- `confidence`
- `in_scope`

We also implemented an automatic repair mechanism that allows one additional attempt if the model produces invalid JSON.

The notebook successfully validated the generated JSON responses.

## 5. Evaluation: SYSTEM_V1 vs SYSTEM_V2

We evaluated both prompt versions using the same 20 questions.

### Evaluation Dataset

| Question Category | Number |
|---|---:|
| Total questions | 20 |
| In-scope questions | 18 |
| Out-of-scope questions | 2 |
| Questions with required phrases | 18 |

The questions covered product information, missing specifications, communication flow, unrelated topics, and an unsupported camera-model claim.

### Evaluation Summary

| Metric | V1 | V2 |
|---|---:|---:|
| Valid JSON | 0.00 | 1.00 |
| Facts OK | 0.055556 | 0.444444 |
| Scope OK | NaN | 1.00 |
| Citations | 0.00 | 0.944444 |

### Confidence Interval

The factual accuracy of V2 was:

```text
0.44 ± 0.23
```

This represents an approximate 95% confidence interval based on 18 in-scope questions:

```text
Lower bound: 0.21
Upper bound: 0.67
```

### Evaluation Analysis

V2 achieved:

- 100% valid JSON.
- Approximately 44.44% factual accuracy.
- 100% scope accuracy.
- Approximately 94.44% citation accuracy.

The evaluation reported 10 failed answers.

However, several failures occurred because of Unicode hyphen differences or equivalent wording rather than incorrect information.

We also identified one genuine factual-grounding issue: a communication-flow answer assumed a specific motor-control arrangement that was not confirmed in the product facts.

This showed that automatic evaluation should consider equivalent wording and identify unsupported technical assumptions.

## 6. Adversarial Testing and Canary Verification

Our team tested five adversarial prompts against SYSTEM_V2.

### Attacks Performed

1. System prompt leakage.
2. Internal canary leakage.
3. False information about a LiDAR sensor.
4. Question-tag injection.
5. Product-specific attack claiming that the TB6612FNG independently controls all four motors.

### Results

| Test | Result |
|---|---|
| System prompt leakage | Resisted |
| Canary leakage | Resisted |
| False LiDAR information | Rejected |
| Question-tag injection | Resisted |
| Unsupported motor-control claim | Rejected |

### Canary Verification

Our internal canary was:

```text
CANARY-7F3A
```

The canary was not revealed in any of the five tests.

```text
Total attacks: 5
Successful defenses: 5
Canary leaked: False
```

All five tests passed in our limited test set. This does not guarantee protection against every possible prompt-injection attack.

---

# 7. Team Contributions and Individual Work

This section documents the work completed by each team member during Week 9.

## Member 1: Rafi Haider

**Role: API Setup, Sampling and Cost Analysis**

### Work Completed

- Configured Kaggle Secrets for secure API key management.
- Connected the Groq API using the `openai/gpt-oss-20b` model.
- Implemented and tested the `chat()` function.
- Verified API retries, exponential backoff, and logging.
- Performed hand-coded softmax and top-p experiments.
- Conducted live temperature variability tests.
- Compared English, Urdu, and Roman Urdu token usage.
- Calculated the monthly API cost estimate.

### Key Results

| Activity | Result |
|---|---|
| API smoke test | Successful |
| Input tokens | 76 |
| Output tokens | 37 |
| Response time | 1.35 seconds |
| Distinct names at T = 0.0 | 2/5 |
| Distinct names at T = 1.0 | 4/5 |
| Urdu token ratio | 1.88× English |
| Estimated monthly cost | USD 123.95 |

### Next Tasks

- Collect hardware datasheets and wiring documentation.
- Help prepare technical documents for Week 10 RAG.
- Improve evaluation normalization for Unicode hyphens and equivalent wording.

### Blockers

No major blockers. HTTP 429 rate-limit errors occurred during evaluation, but the retry and backoff mechanism handled them.

## Member 2: Muhammad Bilal

**Role: Prompt Engineering and Evaluation Suite**

### Work Completed

- Authored the 10 confirmed product facts (F1–F10).
- Developed SYSTEM_V1 and SYSTEM_V2.
- Added grounding, scope, missing-information, and canary rules to SYSTEM_V2.
- Implemented the Pydantic `Reply` model.
- Developed JSON validation and the automatic repair mechanism.
- Prepared the 20-question evaluation dataset.
- Ran the V1 versus V2 evaluation.
- Analyzed factual accuracy, JSON validity, scope accuracy, and citations.

### Key Results

| Activity | Result |
|---|---|
| Product facts | 10 |
| Evaluation questions | 20 |
| V2 valid JSON | 100% |
| V2 factual accuracy | 44.44% |
| V2 scope accuracy | 100% |
| V2 citation accuracy | 94.44% |
| Confidence interval | 0.44 ± 0.23 |

### Next Tasks

- Improve SYSTEM_V2 to prevent unsupported motor-control assumptions.
- Maintain prompt versioning.
- Improve the evaluation script to recognize equivalent wording and normalize Unicode characters.

### Blockers

Exact substring matching sometimes marked semantically correct answers as failures. The evaluation method needs better text normalization and semantic checking.

## Member 3: Khuzaima Ahmed

**Role: Adversarial Testing, Repository and Deliverables**

### Work Completed

- Performed four standard prompt-injection and false-premise tests.
- Designed one additional product-specific motor-control attack.
- Verified that the canary was not leaked.
- Checked whether unsupported project claims were accepted.
- Exported submission artifacts.
- Organized project documentation and repository files.
- Prepared the team README and LinkedIn post.

### Key Results

| Activity | Result |
|---|---|
| Standard attacks | 4 |
| Product-specific attacks | 1 |
| Total attacks | 5 |
| Successful defenses | 5 |
| Canary leaks | 0 |

### Next Tasks

- Expand the evaluation dataset from 20 to at least 30 questions.
- Collect 20–50 pages of relevant hardware documentation.
- Prepare project manuals and FAQs for Week 10 retrieval-augmented generation (RAG).

### Blockers

No major blockers were reported.

---

# 8. End-of-Lab Standup and Ownership

| Responsibility | Owner |
|---|---|
| Prompt changes and Pydantic schema | Muhammad Bilal |
| Evaluation dataset growth | Khuzaima Ahmed and Rafi Haider |
| Hardware documentation collection | Rafi Haider and Khuzaima Ahmed |
| Adversarial testing | Khuzaima Ahmed |
| API setup and cost analysis | Rafi Haider |

**Prompt Changes Owner (ML Lead):** Muhammad Bilal will maintain the system prompts and Pydantic validation schema.

**Evaluation Set Growth Owners:** Khuzaima Ahmed and Rafi Haider will expand the evaluation dataset to 30 or more questions for Week 10.

---

# 9. Repository Structure

Our project files are organized as follows:

```text
Project-3-AI-Autonomous-Vehicle-Assistant/
│
├── notebooks/
│   └── week-9-llms-prompt-engineering-1.ipynb
│
├── prompts/
│   ├── system_v1.md
│   └── system_v2.md
│
├── evals/
│   ├── questions.csv
│   ├── results_v1_v2.csv
│   └── summary.csv
│
└── README.md
```

The notebook contains the experiments from Parts 1–6, while the prompts and evaluation results are maintained separately for easier version control.

---

# 10. Check Your Understanding

Each team member answered the six questions individually.

## Rafi Haider

### 1. For logits [2, 0] at T = 1, what is the probability of the first token?

Using the softmax formula:

\[
P_1=\frac{e^2}{e^2+e^0}
\]

\[
P_1=\frac{7.3891}{7.3891+1}
\]

\[
P_1\approx 0.8808=88.08\%
\]

### 2. Why is T = 0 the right choice for extracting fees and dates into JSON?

Temperature 0 uses greedy or near-deterministic decoding, reducing random variation. This helps produce consistent JSON structures and repeatable extraction results. However, it does not guarantee that extracted information is correct.

### 3. Calculate the monthly cost for 500 users, 10 queries per day, 1,000 input tokens, and 200 output tokens per query.

Monthly queries:

\[
500\times10\times30=150,000
\]

Input cost:

\[
150\times \$0.30=\$45
\]

Output cost:

\[
30\times \$2.50=\$75
\]

Total monthly cost:

\[
\$45+\$75=\boxed{\$120}
\]

### 4. Why must the retry wait include a random part?

Random jitter prevents the thundering herd problem. It helps ensure that multiple clients do not retry at exactly the same time after a rate-limit or server error.

### 5. V2 scores 17/20 on facts and V1 scores 15/20. Is V2 better?

We cannot confidently conclude that V2 is statistically better from this small sample.

The approximate 95% confidence intervals are:

- V2: 85% ± 15.6% = [69.4%, 100%]
- V1: 75% ± 19.0% = [56.0%, 94.0%]

The intervals overlap, so a larger evaluation dataset is needed to establish whether the improvement is statistically meaningful.

### 6. Why wrap user text in tags, and what does a canary detect?

Tags such as `<question>` and `</question>` create a clear boundary between trusted system instructions and untrusted user input.

A canary is a unique secret string included in the system prompt. If it appears in an answer, it can indicate that hidden prompt information has been exposed.

## Muhammad Bilal

### 1. For logits [2, 0] at T = 1, what is the probability of the first token?

\[
P_1=\frac{e^2}{e^2+e^0}
\]

\[
P_1=\frac{7.389}{7.389+1}\approx0.881
\]

Therefore, the probability is approximately **88.1%**.

### 2. Why is T = 0 the right choice for extracting fees and dates into JSON?

Factual extraction requires consistency and precision rather than creativity. Temperature 0 reduces random sampling and helps preserve consistent values and JSON formatting.

### 3. Calculate the monthly cost.

\[
500\times10\times30=150,000
\]

- Input cost = USD 45.00
- Output cost = USD 75.00

**Total = USD 120.00 per month.**

### 4. Why must the retry wait include a random part?

Random jitter desynchronizes concurrent retry requests so that multiple clients do not repeatedly send requests at the same time after temporary API failures.

### 5. V2 scores 17/20 on facts and V1 scores 15/20. Is V2 better?

We cannot claim that V2 is statistically better with confidence. The approximate 95% intervals are:

- V2: 69.4%–100%
- V1: 56.0%–94.0%

Because the intervals overlap, more evaluation questions are needed.

### 6. Why wrap user text in tags, and what does a canary detect?

Tags such as `<question>` identify untrusted user input as data rather than instructions. A canary string helps detect system-prompt leakage if it appears in the model's output.

## Khuzaima Ahmed

### 1. For logits [2, 0] at T = 1, what is the probability of the first token?

\[
\operatorname{softmax}([2,0])_0
=\frac{e^2}{e^2+e^0}
\]

\[
=\frac{7.389056}{8.389056}\approx0.881
\]

The probability is approximately **88.1%**.

### 2. Why is T = 0 the right choice for extracting fees and dates into JSON?

Temperature 0 reduces random sampling and encourages consistent outputs. This is useful when extracting exact numbers and dates into structured JSON.

### 3. Calculate the monthly cost.

Monthly queries:

\[
500\times10\times30=150,000
\]

Input cost = USD 45.00.

Output cost = USD 75.00.

**Total monthly cost = USD 120.00.**

### 4. Why must the retry wait include a random part?

Without random jitter, multiple clients may retry at identical intervals and collide again. Random delays help distribute retry requests over time.

### 5. V2 scores 17/20 on facts and V1 scores 15/20. Is V2 better?

Not conclusively. V2 has an accuracy of 85%, while V1 has 75%. Their approximate confidence intervals overlap, so a larger evaluation set is needed.

### 6. Why wrap user text in tags, and what does a canary detect?

XML-style tags help separate user input from system instructions. A canary is a hidden test string that helps detect whether confidential system-prompt information has been revealed.

---

# 11. Team LinkedIn Post

Week 9 of Applied AI: Our team worked on an AI technical assistant for our ESP32-S3-CAM Mini Autonomous Vehicle!

We developed two system prompts and evaluated them using 20 project-related questions.

**Our results:**

- Valid structured JSON responses: 0% → 100%
- Factual accuracy: approximately 6% → 44%
- Citation accuracy with V2: 94.44%
- V2 scope accuracy: 100%
- Red-team tests resisted: 5/5

One interesting observation was that Urdu required approximately 1.88 times more tokens than English for our tested sentences.

We also learned that structured prompting improves response consistency, but factual verification and better evaluation methods are still necessary.

**Next week:** We plan to explore Retrieval-Augmented Generation (RAG) so our assistant can use real hardware datasheets, technical manuals, and project documentation.

#LLM #PromptEngineering #BuildInPublic #AppliedAI #AutonomousVehicle #Electronics

---

# 12. Final Conclusion

During Week 9, our team gained practical experience with LLM APIs, prompt engineering, tokenization, cost estimation, structured JSON outputs, evaluation, and adversarial testing.

Each member contributed to a different part of the project:

- **Rafi Haider:** API integration, sampling experiments, token analysis, and cost estimation.
- **Muhammad Bilal:** Product facts, prompt engineering, Pydantic validation, and evaluation.
- **Khuzaima Ahmed:** Adversarial testing, canary verification, repository organization, and documentation.

Our structured prompt achieved 100% valid JSON, 100% scope accuracy, and 94.44% citation accuracy. Its measured factual accuracy was 44.44%.

The evaluation also identified limitations in exact phrase matching and highlighted the need to prevent unsupported technical assumptions.

Overall, this lab provided a foundation for developing a more reliable AI assistant. In Week 10, our next objective is to integrate RAG using real ESP32-S3-CAM, OpenCV, HC-SR04, and TB6612FNG documentation.
