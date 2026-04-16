# Spotlight API — Test Case Report

**Project:** Spotlight Upload-to-Output API  
**Test Date:** 15–16 April 2026  
**Tester:** Harsh  
**Environment:** Local · Azure OpenAI (gpt-4o-mini + ada-002)

---

## API Coverage Summary

End-to-end validation of the three Spotlight upload APIs. Each API accepts a different type of narrative input and automatically fills a specific set of screens in the Spotlight application.

| API | Upload Type | Screens Filled | What It Does |
|-----|-------------|----------------|--------------|
| **API 1** — Bound Script / Screenplay | Full structured script | Screen 1 + Screen 2 + Screen 3 | Fills AutoFill (story params + characters), Generate Story (prose, beat sheet, synopsis), and Scene Treatment (detailed scene breakdown). Complete end-to-end flow. |
| **API 2** — Story Upload | Prose narrative / treatment | Screen 1 + Screen 2 | Fills AutoFill and Generate Story. Scene Treatment is not generated. Original story prose is preserved verbatim in the output. |
| **API 3** — Synopsis / One-Liner | Short idea or synopsis | Screen 1 only | Fills AutoFill only — extracts or generates characters, genre, and key story parameters from a brief input. |

---

## Test Summary Table

| TC ID | Story | Upload Type | Endpoint | Screens | Status | Screen Scores |
|-------|-------|-------------|----------|---------|--------|---------------|
| TC01 | The Last Signal | Synopsis / One-Liner | `POST /process/synopsis` | S1 only | ✅ PASS | S1: 100 |
| TC02 | Borrowed Time | Synopsis / One-Liner | `POST /process/synopsis` | S1 only | ✅ PASS | S1: 100 |
| TC03 | The Bridge Between Us | Story / Narrative | `POST /process/story` | S1 + S2 | ✅ PASS | S1: 100, S2: 100 |
| TC04 | The Confession Hour | Story / Narrative | `POST /process/story` | S1 + S2 | ⚠️ PASS w/ Issues | S1: 100, S2: 100 |
| TC05 | PK (Bound Script) | Screenplay | `POST /process/screenplay` | S1 + S2 + S3 | ⚠️ PASS w/ Issues | S1: 100, S2: 100, S3: 100 |

> ✅ PASS · ⚠️ PASS w/ Issues · ❌ FAIL

---

## TC01 — AutoFill: Synopsis (The Last Signal)

**Endpoint:** `POST /process/synopsis`  
**Input File:** `autofill_test_1_synopsis.pdf`

### Input

```
[Paste input here]
```

### Output

```
[Paste output here]
```

### Output Analysis

| Field | Result |
|-------|--------|
| `status` = SUCCESS | ✅ |
| `httpStatus` = 200 | ✅ |
| `uploadType` = SYNOPSIS | ✅ |
| `processingMode` = GENERATION | ✅ |
| `suggestedStopScreen` = 1 | ✅ |
| `primaryLanding` = AUTOFILL_CORE_DETAILS | ✅ |
| `screenScores.screen1` = 100 | ✅ |
| `data.generateStory` absent | ✅ |
| `data.generateScenes` absent | ✅ |
| `autoFill.projectName` = The Last Signal | ✅ |
| `autoFill.genre` = [SCI_FI, THRILLER] | ✅ |
| `autoFill.toneArchetype` = SUSPENSEFUL | ✅ |
| `autoFill.targetAudience` = ADULTS | ✅ |
| `autoFill.pace` = SLOW_BURN | ✅ |
| `autoFill.region` = Rural Montana | ✅ |
| `autoFill.theme` matches input | ✅ |
| `characterList` has 2 characters | ✅ |
| `characterList[0].role` = PROTAGONIST | ✅ |
| `characterList[1].role` = ANTAGONIST | ✅ |
| `characterList[0].importanceLevel` = LEAD | ✅ |
| `characterList[1].importanceLevel` = LEAD/SUPPORTING | ⚠️ Got: `"ANTAGONIST"` |
| `nextSteps` present (3 items) | ✅ |

### Defects Found

| # | Severity | Field | Issue | Impact |
|---|----------|-------|-------|--------|
| D01 | 🟢 Low | `characterList[1].importanceLevel` | Value is `"ANTAGONIST"` — must be `LEAD \| SUPPORTING \| MINOR` | Incorrect importance level on frontend |

### Verdict

**✅ PASS** — All structural requirements met. All correct screens returned. 1 minor schema field-value defect (D01).

---

## TC02 — AutoFill: Synopsis (Borrowed Time)

**Endpoint:** `POST /process/synopsis`  
**Input File:** `autofill_test_2_synopsis.pdf`

### Input

```
[Paste input here]
```

### Output

```
[Paste output here]
```

### Output Analysis

| Field | Result |
|-------|--------|
| `status` = SUCCESS | ✅ |
| `uploadType` = SYNOPSIS | ✅ |
| `processingMode` = GENERATION | ✅ |
| `suggestedStopScreen` = 1 | ✅ |
| `primaryLanding` = AUTOFILL_CORE_DETAILS | ✅ |
| `screenScores.screen1` = 100 | ✅ |
| `data.generateStory` absent | ✅ |
| `data.generateScenes` absent | ✅ |
| `autoFill.projectName` = Borrowed Time | ✅ |
| `autoFill.genre` = [DRAMA, FANTASY] | ✅ |
| `autoFill.endingType` = BITTERSWEET | ✅ |
| `autoFill.targetAudience` = FAMILY | ✅ |
| `autoFill.region` = Mumbai, India | ✅ |
| `autoFill.toneArchetype` = MELANCHOLIC | ✅ |
| `autoFill.theme` matches input | ✅ |
| `characterList` has 3 characters | ✅ |
| Riya Sharma — LEAD, PROTAGONIST | ✅ |
| Meera Sharma — SUPPORTING, MENTOR | ✅ |
| Uncle Ajit — SUPPORTING, FOIL | ✅ |
| `traits` in UPPERCASE (consistent) | ⚠️ Mixed case (e.g., "Intelligent") |

### Defects Found

| # | Severity | Field | Issue | Impact |
|---|----------|-------|-------|--------|
| D02 | 🟢 Low | `characterList[*].traits` | Trait values in mixed case instead of consistent UPPERCASE as in TC01 | Minor display inconsistency |

### Verdict

**✅ PASS** — All 3 input characters correctly identified with full psychological depth. Correct screen gating. 1 minor consistency defect (D02).

---

## TC03 — Story Upload (The Bridge Between Us)

**Endpoint:** `POST /process/story`  
**Input File:** `story_test_1_full_narrative.pdf`

### Input

```
[Paste input here]
```

### Output

```
[Paste output here]
```

### Output Analysis

| Field | Result |
|-------|--------|
| `status` = SUCCESS | ✅ |
| `uploadType` = STORY | ✅ |
| `processingMode` = EXTRACTION | ✅ |
| `suggestedStopScreen` = 2 | ✅ |
| `primaryLanding` = GENERATE_STORY | ✅ |
| `screenScores.screen1` = 100 | ✅ |
| `screenScores.screen2` = 100 | ✅ |
| `data.generateScenes` absent | ✅ |
| `autoFill.projectName` = The Bridge Between Us | ✅ |
| `autoFill.genre` = [DRAMA] | ✅ |
| `autoFill.endingType` = BITTERSWEET | ✅ |
| `autoFill.toneArchetype` = MELANCHOLIC | ✅ |
| `characterList` has 3 characters | ✅ |
| Arjun Mehta — LEAD, PROTAGONIST | ✅ |
| Ramesh Mehta — LEAD, role correct | ⚠️ Got: `PROTAGONIST` (should be MENTOR) |
| Priya — SUPPORTING, LOVE_INTEREST | ✅ |
| `generateStory.story` = original prose preserved | ✅ (with PDF spacing artifact) |
| `generateStory.logline` present | ✅ |
| `generateStory.beatSheet` — 8 beats filled | ✅ |
| `generateStory.synopsis` present | ✅ |
| `generateStory.storyType` = FEATURE_FILM | ✅ |
| `generateStory.visualStyle` = CINEMATIC_REALISM | ✅ |

### Defects Found

| # | Severity | Field | Issue | Impact |
|---|----------|-------|-------|--------|
| D03 | 🟢 Low | `characterList[1].role` (Ramesh) | Ramesh given `PROTAGONIST` — only one protagonist allowed; should be `MENTOR` | Character role display issue |
| D04 | 🟢 Low | `generateStory.story` | Story preserved but with PDF double-space artifacts (e.g., `"Arjun  Mehta"`). Content is correct — artifact is from PDF-to-text parser. | Visual/cosmetic only |

### Verdict

**✅ PASS** — Both screens returned correctly. Original story preserved. Beat sheet and logline generated correctly. Screen 3 correctly absent.

---

## TC04 — Story Upload (The Confession Hour)

**Endpoint:** `POST /process/story`  
**Input File:** `story_test_2_full_narrative.pdf`

### Input

```
[Paste input here]
```

### Output

```
[Paste output here]
```

### Output Analysis

| Field | Result |
|-------|--------|
| `status` = SUCCESS | ✅ |
| `uploadType` = STORY | ✅ |
| `processingMode` = EXTRACTION | ✅ |
| `suggestedStopScreen` = 2 | ✅ |
| `primaryLanding` = GENERATE_STORY | ✅ |
| `screenScores.screen1` = 100 | ✅ |
| `screenScores.screen2` = 100 | ✅ |
| `data.generateScenes` absent | ✅ |
| `autoFill.projectName` = The Confession Hour | ❌ Got: `"The Quiet City"` |
| `generateStory.projectName` = The Confession Hour | ❌ Got: `"The Quiet City"` |
| `autoFill.genre` = [DRAMA] | ✅ |
| `autoFill.toneArchetype` = MELANCHOLIC | ✅ |
| `autoFill.region` = Kozhikode | ✅ |
| `characterList` has 3 characters | ✅ |
| Khalid — LEAD, PROTAGONIST | ✅ |
| Nadia — SUPPORTING, FOIL | ✅ |
| Sara — SUPPORTING, SIDEKICK (daughter, age 9) | ❌ Got: `LOVE_INTEREST` |
| `generateStory.story` = original prose preserved | ✅ (with PDF spacing artifact) |
| `generateStory.logline` present | ✅ |
| `generateStory.beatSheet` — 8 beats filled | ✅ |
| `generateStory.synopsis` present | ✅ |

### Defects Found

| # | Severity | Field | Issue | Impact |
|---|----------|-------|-------|--------|
| D05 | 🟡 Medium | `projectName` (both screens) | Title renamed from **"The Confession Hour"** → **"The Quiet City"** by AI inference | Wrong project name shown across all screens |
| D06 | 🟡 Medium | `characterList[1].role` (Sara) | Sara is Khalid's 9-year-old daughter; classified as `LOVE_INTEREST` | Incorrect character role on breakdown screen |
| D07 | 🟢 Low | `generateStory.story` | PDF spacing artifact — same as D04 | Visual/cosmetic only |

### Verdict

**⚠️ PASS with Issues** — Both screens returned, beat sheet complete, original story preserved. 2 medium-severity AI inference errors: title renamed, child character misclassified.

---

## TC05 — Screenplay Upload / Bound Script (PK)

**Endpoint:** `POST /process/screenplay`  
**Input File:** `Pk-movie-script-boundscript.pdf`

### Input

```
[Paste input here]
```

### Output

```
[Paste output here]
```

### Output Analysis

| Field | Result |
|-------|--------|
| `status` = SUCCESS | ✅ |
| `uploadType` = SCREENPLAY | ✅ |
| `processingMode` = EXTRACTION | ✅ |
| `suggestedStopScreen` = 3 | ✅ |
| `primaryLanding` = SCENE_TREATMENT | ✅ |
| `screenScores.screen1` = 100 | ✅ |
| `screenScores.screen2` = 100 | ✅ |
| `screenScores.screen3` = 100 | ✅ |
| `autoFill.projectName` = PK | ✅ |
| `autoFill.genre` = [COMEDY, DRAMA, SCI_FI] | ✅ |
| `autoFill.toneArchetype` = COMEDIC | ✅ |
| `autoFill.targetAudience` = ALL_AGES | ✅ |
| PK — LEAD, PROTAGONIST | ✅ |
| Jaggu — SUPPORTING, SIDEKICK | ✅ |
| `generateStory.logline` present | ✅ |
| `generateStory.beatSheet` — 8 beats filled | ✅ |
| `generateScenes.scenes` count = 50 | ✅ |
| All scenes sequential (`sceneSeq` 1–50) | ✅ |
| All scenes have `slugline` with page estimate | ✅ |
| All scenes have `pagesEighthsEst` | ✅ |
| `intExt` = only INT / EXT / INT/EXT | ❌ 3 scenes use `"I/E"` (scenes 12, 13, 16) |
| `pagesEighthsEst` notation valid (max 7/8) | ⚠️ Scene 8: `"1 8/8"` (should be `"2 0/8"`) |

### Scene-Level Defects

| Scene # | Field | Got |
|---------|-------|-----|
| Scene 12 | `intExt` | `"I/E"` ❌ |
| Scene 13 | `intExt` | `"I/E"` ❌ |
| Scene 16 | `intExt` | `"I/E"` ❌ |
| Scene 8  | `pagesEighthsEst` | `"1 8/8"` ⚠️ |

### Defects Found

| # | Severity | Field | Issue | Impact |
|---|----------|-------|-------|--------|
| D08 | 🟡 Medium | `scenes[12,13,16].intExt` | `"I/E"` used instead of required `"INT/EXT"` — schema says `exactly INT \| EXT \| INT/EXT` | Frontend `intExt` mapping fails for these 3 scenes |
| D09 | 🟢 Low | `scenes[8].pagesEighthsEst` | `"1 8/8"` — eighths run 0/8–7/8 then reset; should be `"2 0/8"` | Minor notation issue |

### Verdict

**⚠️ PASS with Issues** — All 3 screens returned, 100% scores, 50 scenes with sluglines and page estimates. 3 scenes have `intExt` shorthand defect — prompt enforcement gap.

---

## Overall Defect Summary

| Defect ID | TC | Severity | Field | Description |
|-----------|-----|----------|-------|-------------|
| D01 | TC01 | 🟢 Low | `importanceLevel` | Value `"ANTAGONIST"` — must be `LEAD/SUPPORTING/MINOR` |
| D02 | TC02 | 🟢 Low | `traits` | Mixed case instead of UPPERCASE — inconsistent across runs |
| D03 | TC03 | 🟢 Low | `role` (Ramesh) | Two characters both assigned `PROTAGONIST` |
| D04 | TC03 | 🟢 Low | `story` | PDF double-space extraction artifact |
| D05 | TC04 | 🟡 Medium | `projectName` | Title renamed "The Confession Hour" → "The Quiet City" |
| D06 | TC04 | 🟡 Medium | `role` (Sara) | Daughter (age 9) classified as `LOVE_INTEREST` |
| D07 | TC04 | 🟢 Low | `story` | PDF double-space extraction artifact (same as D04) |
| D08 | TC05 | 🟡 Medium | `intExt` | `"I/E"` used instead of `"INT/EXT"` in 3 scenes |
| D09 | TC05 | 🟢 Low | `pagesEighthsEst` | `"1 8/8"` instead of `"2 0/8"` |

| Severity | Count |
|----------|-------|
| 🔴 High | 0 |
| 🟡 Medium | 3 |
| 🟢 Low | 6 |

---

## Key Findings

### ✅ What Worked
- All 5 TCs returned `status: SUCCESS` with no API errors
- Screen gating correct: Synopsis → S1, Story → S1+S2, Screenplay → S1+S2+S3
- `suggestedStopScreen` and `primaryLanding` correct in every test
- `screenScores` = 100 on all filled screens across all 5 cases
- Original story prose preserved in TC03, TC04 (content verbatim — PDF spacing is parser-level)
- All 50 PK scenes generated with sequential numbering, sluglines, and page estimates
- Beat sheet (8 milestones), logline, synopsis correct in TC03, TC04, TC05

### ⚠️ Next Sprint Fixes (Prompt Level)
1. **D08** — Add to `fill_screen3`: _"never write 'I/E' — always write 'INT/EXT' in full"_
2. **D05** — LLM to extract title from document heading, not infer thematically
3. **D06** — Add guidance: child characters of protagonist default to `SIDEKICK`, not `LOVE_INTEREST`

### 📝 Known Limitation
- **D04 / D07 — PDF double-spacing:** `pypdf` extractor can produce double spaces when reading PDF files generated from Word/Docs. Story content is 100% intact. Files uploaded as `.txt` will not have this artifact.

---

*Generated: 16 April 2026 · Output JSONs in `test_files/`*
