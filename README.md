# Today's Tasks & Progress

---

## 1. ORBITS — STT Optimization (Translation Pipeline)

Applied model and config changes to reduce transcription time. Results below:

| Stage | Before | After | Improvement |
|---|---|---|---|
| Transcription | 498,771 ms (8m 18s) | 88,007 ms (1m 28s) | **5.6x faster** |
| Translation | 15,054 ms | 18,870 ms | similar |
| Speaking | 17,191 ms | 21,732 ms | similar |
| **Total** | **531,016 ms (8m 51s)** | **~128,609 ms (2m 8s)** | **4x faster** |

**What was changed:**
- Switched Whisper model from `large-v3` → `large-v3-turbo`
- Set `cpu_threads=8` to use all i5-1250P cores

---

## 2. File Summary — Image URL Fix

**Problem:** Image files were failing — the URL was sent directly to Azure Vision API, which couldn't reach the server, causing `is_healthcare_related: false` even for valid blood reports.

**Fix:** Now always encodes the image as **Base64** before sending to Azure Vision API. Since the file bytes are already downloaded locally, no URL is ever passed to Azure — works for all images regardless of server visibility.
