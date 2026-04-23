# Lab 16 Benchmark Report

## Metadata
- Dataset: hotpot_mini.json
- Mode: mock
- Records: 192
- Agents: react, reflexion

## Summary
| Metric | ReAct | Reflexion | Delta |
|---|---:|---:|---:|
| EM | 0.5 | 1.0 | 0.5 |
| Avg attempts | 1 | 1.5 | 0.5 |
| Avg token estimate | 385 | 790 | 405 |
| Avg latency (ms) | 200 | 455 | 255 |

## Failure modes
```json
{
  "react": {
    "none": 48,
    "incomplete_multi_hop": 12,
    "wrong_final_answer": 12,
    "entity_drift": 24
  },
  "reflexion": {
    "none": 96
  }
}
```

## Extensions implemented
- structured_evaluator
- reflection_memory
- benchmark_report_json
- mock_mode_for_autograding

## Discussion
This benchmark highlights a clear performance gap between ReAct and Reflexion agents.

1. Quantitative Comparison:
- ReAct EM: 0.5
- Reflexion EM: 1.0
- EM Improvement: 0.5

2. Concrete Failure Cases (ReAct):

--- Case 1 ---
QID: hp2
Question: What river flows through the city where Ada Lovelace was born?
Predicted: London
Gold: River Thames
Failure Mode: incomplete_multi_hop
Attempts: 1

--- Case 2 ---
QID: hp4
Question: Which ocean borders the country whose capital is Lima?
Predicted: Atlantic Ocean
Gold: Pacific Ocean
Failure Mode: wrong_final_answer
Attempts: 1

--- Case 3 ---
QID: hp6
Question: What sea borders the country where Petra is located?
Predicted: Red Sea
Gold: Dead Sea
Failure Mode: entity_drift
Attempts: 1

--- Case 4 ---
QID: hp8
Question: Which mountain range contains the highest mountain in the country whose capital is Kathmandu?
Predicted: Andes
Gold: Himalayas
Failure Mode: entity_drift
Attempts: 1

--- Case 5 ---
QID: hp2
Question: What river flows through the city where Ada Lovelace was born?
Predicted: London
Gold: River Thames
Failure Mode: incomplete_multi_hop
Attempts: 1

3. Concrete Success Cases (Reflexion correcting errors):

--- Case 1 ---
QID: hp2
Question: What river flows through the city where Ada Lovelace was born?
Final Answer: River Thames
Gold: River Thames
Attempts needed: 2
Reflections used: 1

--- Case 2 ---
QID: hp4
Question: Which ocean borders the country whose capital is Lima?
Final Answer: Pacific Ocean
Gold: Pacific Ocean
Attempts needed: 2
Reflections used: 1

--- Case 3 ---
QID: hp6
Question: What sea borders the country where Petra is located?
Final Answer: Dead Sea
Gold: Dead Sea
Attempts needed: 2
Reflections used: 1

4. Failure Mode Distribution:
{
  "react": {
    "none": 48,
    "incomplete_multi_hop": 12,
    "wrong_final_answer": 12,
    "entity_drift": 24
  },
  "reflexion": {
    "none": 96
  }
}

Interpretation:
- ReAct fails mainly due to 'incomplete_multi_hop_reasoning' (stops after first hop)
- 'entity_drift' occurs when the agent picks a wrong intermediate entity
- Reflexion eliminates most of these via evaluator feedback and reflection memory

5. Efficiency Trade-offs:
- Avg attempts (ReAct → Reflexion): 1 → 1.5
- Δ attempts: +0.5
- Δ tokens: +405
- Δ latency: +255 ms

6. Role of Reflection Memory:
Reflection memory stores previous mistakes and their corrections. For example, in QID hp4 (Peru/Pacific Ocean), the agent first guessed 'Atlantic', then after reflection corrected to 'Pacific' in attempt 2. Without memory, the same mistake would repeat.

7. Key Insight:
Reflexion's strength is not better first-try accuracy, but the ability to recover from mistakes.
This makes it particularly suitable for multi-hop QA where single-pass reasoning is insufficient.

8. Limitations:
- Evaluator quality is critical — if the evaluator is wrong, reflection can reinforce errors
- Mock mode doesn't capture real LLM variability (temperature, sampling, etc.)
- Token cost grows linearly with attempts, which may be prohibitive for some applications

