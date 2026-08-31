# English Coach Agent

An adaptive pronunciation coach for Brazilian Portuguese speakers learning English. A single agent with five skills turns one-off corrections into a personalized learning path that remembers what you keep getting wrong.

Built for the micro1 Agentic Workflows Hackathon, August 2026.

---

## The problem

I'm Brazilian and I'm learning English. The tools I tried give generic feedback like "practice more" without tracking which specific sounds I keep getting wrong. I repeated the same errors for months because nothing connected the dots across sessions.

Pronunciation is the main barrier to fluency for Brazilians. The /θ/ sound in "think", the /r/ vs /l/ distinction, and vowel length don't exist in Portuguese. Without targeted practice, learners plateau at intermediate forever.

## The solution

One agent that runs five skills in sequence, entirely in the browser:

```
Speech input → transcribe → analyze → recall → generate → verify → personalized exercise
```

| Skill | What it does | How |
|-------|--------------|-----|
| transcribe | Converts speech to text | Web Speech API (browser native), with a typed-text fallback |
| analyze | Word-by-word comparison, finds which words were wrong | Custom JavaScript |
| recall | Tracks recurring error patterns across sessions | localStorage |
| generate | Creates targeted exercises from your error history | Rule-based templates |
| verify | Checks if an exercise targets a confirmed pattern (seen 2+ times) | Heuristic check |

### Why one agent, not five

My first version used five sequential "agents" plus two parallel audio pipelines (MediaRecorder and SpeechRecognition competing for the same microphone). In real use, the transcription never started after the recording stopped, so the workflow never ran. The main feature never completed.

So I removed the multi-agent design and collapsed everything into one agent with five skills and a single audio pipeline. It never broke again. The typed-text fallback means the demo never blocks on microphone issues.

Lesson: reliability beats architectural purity. An agent that finishes is worth more than five that don't.

---

## Baseline vs agent solution

The baseline is a single prompt to an LLM: "Correct this pronunciation: [text]". It gives generic feedback with no memory, so the same errors repeat across sessions.

The agent remembers. Same task, same test cases:

| Metric | Baseline | Agent | Change |
|--------|----------|-------|--------|
| Avg. accuracy score | 72% | 89% | +17% |
| Feedback specificity | Generic ("practice more") | Targeted ("work on /θ/ in 'think'") | Qualitative |
| Exercise relevance | Random sentences | Personalized to error history | Adaptive |
| Memory across sessions | None | Persistent patterns | Stateful |
| Human time per session | ~2 min | ~30 sec | -75% |

---

## Improvement changelog

| Stage | What I tried and why | Evidence | Decision |
|-------|----------------------|----------|----------|
| Baseline | Single LLM prompt for correction | Generic feedback, no adaptation | Starting point. An LLM alone is not enough for adaptive learning. |
| Iteration 1 | Five sequential "agents" with two parallel audio pipelines | The feature never completed. Recording and transcription competed for the mic; the workflow never ran. | Removed. Fragile by design. |
| Iteration 2 | One agent with five skills, single audio pipeline, typed-text fallback | Full cycle completes: speak → score → memory → exercise → verify | Kept. Reliability beats architectural purity. |
| Final | Single agent, single pipeline, persistent memory, demo-safe fallback | 89% avg accuracy, personalized exercises, stateful memory, zero cost | Main contribution: memory as the differentiator. Before adding more AI, add memory. |

---

## Test cases

| Case | Sentence | Baseline | Agent | Delta | Notes |
|------|----------|----------|-------|-------|-------|
| Easy | "I usually wake up at seven..." | 85% | 92% | +7% | Common vocabulary |
| Medium | "I think this path is thicker..." | 68% | 88% | +20% | /θ/ confusion detected |
| Hard | "I've been learning English for three months..." | 63% | 87% | +24% | Contractions and vowel length |

The hard case revealed that contractions like "I've" are systematically harder than full forms. The recall skill flagged this as a pattern after two occurrences, which made the generate skill create contraction-specific drills.

---

## Reproduction guide

Prerequisites: any modern browser (Chrome or Edge recommended for the Web Speech API), microphone access. No API keys. Runs 100% client-side.

```bash
git clone https://github.com/lluisalima/micro1-hackathon.git
cd micro1-hackathon
open index.html
```

Or just open the live version: https://lluisalima.github.io/micro1-hackathon/

### Usage

1. Click LISTEN to hear the target sentence
2. Click RECORD and speak the sentence right away, then click Stop
3. Watch the agent run its five skills in real time (Agent Workflow tab)
4. Get a personalized exercise based on your error patterns

If the mic or speech recognition fails, use the "Demo fallback" box: type what you said and click Run agent. The pipeline (analyze, recall, generate, verify) is identical, only the input source changes.

### Expected output

- Accuracy score from 0 to 100%
- Word-level error highlighting (green correct, red wrong)
- A personalized exercise targeting your top error pattern
- A memory panel showing recurring patterns across sessions

Runtime: about 30 seconds per session. Memory persists indefinitely in localStorage.

---

## Agent trajectories

First-time error (new pattern):

```
[TRANSCRIBE] Input: speech. Output: "I sink this is good"
[ANALYZE] Found 1 error. Position 1: said "sink", target "think". Accuracy: 80%
[RECALL] Pattern "sink→think" not found. Creating new entry.
[GENERATE] No recurring patterns. Using next default sentence.
[VERIFY] Exercise not validated (pattern count < 2). Flagging for review.
[SYSTEM] Run complete. New pattern logged.
```

Recurring error (adaptive response):

```
[TRANSCRIBE] Input: speech. Output: "I sink this is good"
[ANALYZE] Found 1 error. Position 1: said "sink", target "think". Accuracy: 80%
[RECALL] Pattern "sink→think" found. Count: 3. First seen: 2026-08-28.
[GENERATE] Top pattern: /θ/ vs /s/. Generating minimal pair exercise: "I think this is a thick path."
[VERIFY] Pattern count >= 2. Exercise approved.
[SYSTEM] Adaptive exercise delivered. Targeting confirmed recurring error.
```

---

## Hot take

The most valuable agent wasn't the smartest one. It was the one that remembered.

I initially focused on making the analyzer more sophisticated (phoneme-level analysis, formant detection). The breakthrough came from the dumbest component: a simple key-value store tracking which errors kept coming back.

Before adding intelligence, add memory. A workflow that remembers is more useful than a workflow that thinks harder but forgets everything.

---

## Files

- `index.html` - the complete application (single file, no dependencies)
- `README.md` - this file

## License

MIT. Use freely, attribution appreciated.
