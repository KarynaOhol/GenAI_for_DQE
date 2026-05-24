# Module 4 — Chatbot Prompt Engineering Report

---

## Task 1: Preventing Hallucination in a Corporate HR Chatbot

### Task Description

A corporate HR chatbot assists employees with questions about company vacation policies.
The chatbot is regularly used on **mobile phones**, so answers must be short and concise.

**Setup:** The chatbot uses GPT-4 with `temperature=1`.

This is incorrect — the actual policy states **up to 5 days**.

---

### Diagnosis: Why Did the Original Prompt Fail?

**1. `temperature=1` — Non-deterministic, creative output**
- Temperature controls randomness in the model's output.This means responses are **not deterministic** — the same question will produce different answers on each run. High temperature is ideal for creative tasks.

**2. The instruction "provide all possible information considering generic knowledge"**
- This phrase explicitly tells the model to use its general training knowledge
- No explicit rule prohibiting invented answers
- No fallback instruction for unknown information

---

### Fixed System Prompt

```
You are a corporate HR assistant. Your ONLY job is to answer employee questions 
using the Known Policies listed below. 

STRICT RULES:
1. Answer ONLY using the Known Policies provided. Do NOT use any outside knowledge.
2. STRICTLY PROHIBITED: Do not invent, estimate, guess, or speculate on any 
   information not explicitly listed in Known Policies.
3. If the answer is not found in Known Policies, respond ONLY with: 
   "I don't have that information. Please contact HR directly."
4. Keep every answer under 2 sentences — responses are read on mobile phones.
5. Do not add extra context, suggestions, or elaborations beyond what is in the policy.

Known Policies:
• Full-time employees: 21 vacation days/year.
• Part-time (30+ hours/week): Prorated vacation.
• Carryover: Up to 5 days with HR approval by Dec 1.
• Parental leave: 12 weeks (primary caregivers).
• No vacation during probation (first 3 months).
• Europe offices: +3 days to standard policy.
• Sabbatical eligibility: 5+ years of service.
• Emergency leave: 5 paid days/year.
```

**Key improvements:**
- *"Answer ONLY using the Known Policies"* — strictly limits the model's information source
- `STRICTLY PROHIBITED` rule — explicit keyword that strongly signals the model to stop inventing
- Defined fallback response — model knows exactly what to say when information is missing
- Mobile brevity rule — answers capped at 2 sentences
- `temperature=0` — eliminates randomness
---

### Results: Before vs After

**Question asked:** *"What's the carryover limit for unused vacation days?"*

| | Response |
|---|---|
| **Before (original prompt, temp=1)** | *"You can carry over up to 10 days."* — hallucinated, incorrect |
| **After (fixed prompt, temp=0)** | *"You can carry over up to 5 days with HR approval, submitted by December 1."* — accurate, concise, mobile-friendly |

---

## Task 2: Test Case Design

### Task Description

Three user prompts were designed to validate the reliability of the fixed system prompt from Task 1.
Each test covers a different risk scenario: ambiguous input, non-existent policy, and an edge case.

---

### Test Case 1 — Ambiguous Query

The policy mentions prorated vacation but gives no exact number. The chatbot must resist inventing a specific figure and stay strictly within what the policy states.

**Test Input:** *"How many vacation days do I get if I work part-time?"*

**Expected Output:**
> "Part-time employees working 30+ hours per week receive prorated vacation days. Please contact HR directly to calculate your specific entitlement based on your hours."

---

### Test Case 2 — Non-Existent Policy

This directly tests the fallback rule — the chatbot must not invent a policy or draw on generic knowledge

**Test Input:** *"Does the company offer unlimited mental health days?"*

**Expected Output:**
> "I don't have that information. Please contact HR directly."

---

### Test Case 3 — Edge Case

Both parental leave (12 weeks) and vacation days (21 days) exist individually in Known Policies, but their combination is not addressed. The chatbot must not speculate or reason across two policies — it must recognize the gap and defer to HR.

**Test Input:** *"Can I combine parental leave with my vacation days in Q4?"*

**Expected Output:**
> "I don't have that information. Please contact HR directly."

---

