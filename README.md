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
