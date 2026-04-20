# Spotlight AI — API Reference

Python AI service covering two workflows: **Story Generation** and **Scene Treatment**. Both follow the same standard request/response envelope.

---

## 1. Generate Story

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/story`

### Request
```json
{
  "storyIdea": "A village boy dreams of playing cricket for India",
  "storyStructure": "THREE_ACT",
  "narrativeStyle": "COMMERCIAL_MASS_ENTERTAINER",
  "pace": "BALANCED_NARRATIVE",
  "theme": "FAMILY_AND_BELONGING",
  "contentSensitivity": "CLEAN_FAMILY_SAFE",
  "genre": ["COMEDY"],
  "toneArchetype": "FEEL_GOOD",
  "endingType": "HAPPY_ENDING",
  "region": "Rural India",
  "targetAudience": ["FAMILY_ALL_AGES"],
  "referenceFilmsWorks": "Lagaan, Chak De India",
  "creativeNotes": "Focus on community and family bonds"
}
```

| Field | Required | Notes |
|---|---|---|
| `storyIdea` | Yes | Core story concept |
| `storyStructure` | Yes | Story format |
| `narrativeStyle` | Yes | Storytelling style |
| `pace` | Yes | Story speed/rhythm |
| `theme` | Yes | Central theme |
| `contentSensitivity` | Yes | Content rating |
| `genre` | No | Array of genres |
| `toneArchetype` | No | Emotional tone |
| `endingType` | No | How story ends |
| `region` | No | Cultural setting |
| `targetAudience` | No | Intended viewers |
| `referenceFilmsWorks` | No | Style references |
| `creativeNotes` | No | Extra direction |

### Response
```json
{
  "status": "SUCCESS",
  "message": "Story generated successfully",
  "httpStatus": 200,
  "data": {
    "story": "Full scriptment 3000-4000 words...",
    "logline": "One line — character, conflict, stakes.",
    "tone": "Emotional atmosphere description.",
    "beatSheet": {
      "1. Setup": "...",
      "2. Inciting Incident": "...",
      "3. First Turning Point": "...",
      "4. Midpoint": "...",
      "5. All Is Lost": "...",
      "6. Climax": "...",
      "7. Resolution": "..."
    },
    "synopsis": "1-3 paragraph producer summary."
  },
  "timestamp": "2026-04-19T14:31:38.123456"
}
```

| Field | Notes |
|---|---|
| `story` | Full scriptment, no dialogue |
| `logline` | One sentence summary |
| `tone` | Cinematic mood/style |
| `beatSheet` | Key plot beats as object |
| `synopsis` | Short producer summary |

---

## 2. Generate Scene Treatment

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/scene`

> Use `story` and `synopsis` from the Generate Story response above as inputs here.

### Request
```json
{
  "projectName": "The rural village cricket game",
  "story": "<paste story from step 1>",
  "synopsis": "<paste synopsis from step 1>",
  "characterList": [
    {
      "name": "Arjun Kumar",
      "role": "MAIN",
      "description": "A determined 15-year-old village boy who lives for cricket."
    },
    {
      "name": "Vikram Rao",
      "role": "SECOND_LEAD",
      "description": "Arjun's mentor, a pragmatic former school cricketer."
    }
  ]
}
```

| Field | Required | Notes |
|---|---|---|
| `story` | Yes | Full story from step 1 |
| `synopsis` | Yes | Synopsis from step 1 |
| `projectName` | No | Project title |
| `characterList` | No | name, role, description only |

**Character roles:** `MAIN` `SECOND_LEAD` `VILLAIN` `SUPPORTING`

### Response
```json
{
  "status": "SUCCESS",
  "message": "Scene treatment generated successfully",
  "httpStatus": 200,
  "data": {
    "scenes": [
      {
        "sceneSeq": 1,
        "slugline": "SCENE 1. EXT. VILLAGE CRICKET FIELD - DAY",
        "intExt": "EXT.",
        "dayNight": "DAY",
        "location": "VILLAGE CRICKET FIELD",
        "pagesEighthsEst": "1 2/8",
        "visualSummary": "Arjun stands on the dusty pitch...",
        "image": {
          "prompt": "Cinematic image generation prompt..."
        },
        "storyBoardList": []
      }
    ]
  },
  "timestamp": "2026-04-19T14:31:38.123456"
}
```

| Field | Notes |
|---|---|
| `sceneSeq` | Scene number |
| `slugline` | Scene header line |
| `intExt` | Interior or exterior |
| `dayNight` | Time of day |
| `location` | Scene location |
| `pagesEighthsEst` | Page length estimate |
| `visualSummary` | What happens on screen |
| `image.prompt` | Image generation text |
| `storyBoardList` | Always empty `[]` |

> Scene count is AI-determined based on story complexity (40–99 scenes). Response time may be 5–15 minutes.

---

## Errors

```json
{ "error": "HTTP_ERROR", "message": "Reason here" }
```

| Code | Meaning |
|---|---|
| `422` | Missing required field |
| `500` | AI generation failed |
