# Spotlight AI — API Reference

Python AI service exposing **9 endpoints** across two categories:
- **Text-to-Text Generation** (APIs 1–4): Autofill, Story, Scene, Visual Asset — all use Azure OpenAI.
- **Media Generation** (APIs 5–6): Single Image (Vertex AI Imagen), Single Video (RunwayML).
- **Upload Script Processing** (APIs 7–9): File upload → extraction/generation pipeline.

All text-generation APIs return the same `usage` envelope including full Azure content filter metadata.

---

## Usage Object — Standard Format (All Text-Generation APIs)

Every text-generation response (APIs 1–4) includes a `usage` object with the following structure:

```json
"usage": {
  "prompt_tokens": 1240,
  "completion_tokens": 4500,
  "total_tokens": 5740,
  "input_tokens": 1240,
  "output_tokens": 4500,
  "content_filters": [
    {
      "blocked": false,
      "source_type": "prompt",
      "content_filter_raw": [],
      "content_filter_results": {
        "hate":      { "filtered": false, "severity": "safe" },
        "sexual":    { "filtered": false, "severity": "safe" },
        "violence":  { "filtered": false, "severity": "safe" },
        "self_harm": { "filtered": false, "severity": "safe" },
        "jailbreak": { "detected": false, "filtered": false }
      },
      "content_filter_offsets": {}
    },
    {
      "blocked": false,
      "source_type": "completion",
      "content_filter_raw": [],
      "content_filter_results": {
        "hate":                    { "filtered": false, "severity": "safe" },
        "sexual":                  { "filtered": false, "severity": "safe" },
        "violence":                { "filtered": false, "severity": "safe" },
        "self_harm":               { "filtered": false, "severity": "safe" },
        "protected_material_code": { "detected": false, "filtered": false },
        "protected_material_text": { "detected": false, "filtered": false }
      },
      "content_filter_offsets": {}
    }
  ],
  "incomplete_details": null
}
```

| Field | Notes |
|---|---|
| `prompt_tokens` / `input_tokens` | Same value — tokens consumed by the prompt |
| `completion_tokens` / `output_tokens` | Same value — tokens in the AI response |
| `total_tokens` | Sum of prompt + completion |
| `content_filters` | Azure filter audit per request — always present, even on safe content |
| `content_filters[].source_type` | `"prompt"` or `"completion"` |
| `content_filters[].blocked` | `true` only when Azure blocked that source |
| `incomplete_details` | `null` on success; `{"reason": "content_filter"}` when blocked |

> `content_filters` is populated on every call — it is Azure's audit trail, not just a failure field.

---

## Content Filter Fallback — Standard Behaviour

When Azure OpenAI blocks content generation (hard block via `finish_reason=content_filter`, or soft refusal via a plain-text apology), **all four text-generation APIs return HTTP 200** with `status: "CONTENT_FILTERED"`. The error message is placed inside the primary data field so the frontend can display it without any code changes.

| API | Field carrying the message |
|---|---|
| Autofill | `data.storyIdea` |
| Story | `data.story` |
| Scene | `data.scenes[0].visual_summary` (single entry) |
| Visual Asset | `data.visualStyle` |

---

## 1. Autofill

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/autofill`

Takes partial project data and basic character info — returns all fields fully filled with AI-generated values including complete character depth.

### Request
```json
{
  "projectName": "The rural village cricket game",
  "storyIdea": "A village boy dreams of playing cricket for India",
  "sceneCount": 60,
  "genre": ["COMEDY"],
  "narrativeStyle": "COMMERCIAL_MASS_ENTERTAINER",
  "storyStructure": "THREE_ACT",
  "pace": "BALANCED_NARRATIVE",
  "theme": "FAMILY_AND_BELONGING",
  "contentSensitivity": "CLEAN_FAMILY_SAFE",
  "toneArchetype": "FEEL_GOOD",
  "endingType": "HAPPY_ENDING",
  "region": "Rural India",
  "cultureRegion": "Rural India",
  "targetAudience": ["FAMILY_ALL_AGES"],
  "referenceFilmsWorks": "Lagaan, Chak De India",
  "creativeNotes": "Focus on community and family bonds",
  "characterList": [
    {
      "id": "char-001",
      "name": "Arjun Kumar",
      "role": "MAIN",
      "description": "A determined village boy who lives for cricket."
    },
    {
      "id": "char-002",
      "name": "Vikram Rao",
      "role": "SECOND_LEAD",
      "description": "Arjun's mentor and former school cricketer."
    }
  ]
}
```

| Field | Required | Notes |
|---|---|---|
| All story fields | No | Send whatever is available — AI fills the rest |
| `characterList` | No | Only `name`, `role`, `description` needed; AI fills all depth fields |
| `role` values | — | `MAIN` `SECOND_LEAD` `VILLAIN` `SUPPORTING` |

### Response — Success
```json
{
  "status": "SUCCESS",
  "message": "Story parameters auto-filled successfully",
  "httpStatus": 200,
  "data": {
    "projectName": "The rural village cricket game",
    "storyIdea": "...",
    "sceneCount": 60,
    "genre": ["COMEDY"],
    "narrativeStyle": "COMMERCIAL_MASS_ENTERTAINER",
    "storyStructure": "THREE_ACT",
    "endingType": "HAPPY_ENDING",
    "openingSetupIdea": "...",
    "climaxIdea": "...",
    "pace": "BALANCED_NARRATIVE",
    "toneArchetype": "FEEL_GOOD",
    "theme": "FAMILY_AND_BELONGING",
    "region": "Rural India",
    "targetAudience": ["FAMILY_ALL_AGES"],
    "contentSensitivity": "CLEAN_FAMILY_SAFE",
    "referenceFilmsWorks": "Lagaan, Chak De India",
    "characterList": [
      {
        "id": "char-001",
        "name": "Arjun Kumar",
        "role": "MAIN",
        "description": "...",
        "importanceLevel": "Primary",
        "storySignificance": "...",
        "style": "",
        "imagePrompt": "",
        "imageUrl": "",
        "aspectRatio": "",
        "behaviour": "...",
        "goalInternal": "...",
        "goalExternal": "...",
        "backstory": "...",
        "struggles": "...",
        "enneagramType": "...",
        "traits": "...",
        "coreMotivation": "...",
        "coreValues": "...",
        "coreFear": "...",
        "ghost": "...",
        "lie": "...",
        "want": "...",
        "need": "...",
        "purposeOfLife": "...",
        "superObjective": "...",
        "appearanceInScenes": [],
        "createdBy": "",
        "updatedBy": "",
        "createdOn": "",
        "updatedOn": "",
        "deletedAt": "",
        "deleted": false,
        "isDeleted": false
      }
    ]
  },
  "timestamp": "2026-05-05T14:31:38.123456+00:00",
  "usage": { "...see Usage Object above..." }
}
```

| Field | Notes |
|---|---|
| All story fields | Completed/corrected by AI |
| `characterList` | Full character depth — all fields filled; passthrough DB fields echoed unchanged |
| `isDeleted` | Always `false` |
| `appearanceInScenes` | Always `[]` from Autofill — populated by Visual Asset API |

---

## 2. Generate Story

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
  "creativeNotes": "Focus on community and family bonds",
  "characterList": [
    {
      "id": "char-001",
      "name": "Arjun Kumar",
      "role": "MAIN",
      "behaviour": "...",
      "goalExternal": "...",
      "goalInternal": "...",
      "backstory": "...",
      "struggles": "...",
      "enneagramType": "...",
      "traits": "...",
      "coreFear": "...",
      "coreMotivation": "...",
      "coreValues": "...",
      "lie": "...",
      "ghost": "...",
      "want": "...",
      "need": "...",
      "purposeOfLife": "...",
      "superObjective": "..."
    }
  ]
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
| `creativeNotes` | No | Extra creative direction |
| `characterList` | No | Full character objects from Autofill response — richer characters = richer story |

### Response — Case A: Success
```json
{
  "status": "SUCCESS",
  "message": "Story generated successfully",
  "httpStatus": 200,
  "data": {
    "story": "Full scriptment 3000–4000 words, active present tense, no dialogue...",
    "logline": "One sentence — actual character name, conflict, and stakes.",
    "tone": "Emotional atmosphere and cinematic style description.",
    "beatSheet": {
      "1. Opening Image": "...",
      "2. Catalyst": "...",
      "3. Midpoint": "...",
      "4. All Is Lost": "...",
      "5. Finale": "..."
    },
    "synopsis": "1–3 paragraph producer-facing story summary."
  },
  "timestamp": "2026-05-05T14:31:38.123456",
  "usage": { "...see Usage Object above..." }
}
```

### Response — Case B: Content Filtered (blocked input)

When Azure OpenAI refuses to generate content (e.g. terrorism, explicit violence, harmful synthesis instructions), the API returns HTTP 200 with the following structure — **no new fields; the error message appears in `data.story`**:

```json
{
  "status": "CONTENT_FILTERED",
  "message": "Story generated successfully",
  "httpStatus": 200,
  "data": {
    "story": "We're sorry, this content could not be generated as it may involve sensitive or restricted themes. Please review your inputs and try again with a different topic.",
    "logline": "",
    "tone": "",
    "beatSheet": {},
    "synopsis": ""
  },
  "timestamp": "2026-05-05T16:52:08.686556",
  "usage": {
    "prompt_tokens": 3924,
    "completion_tokens": 10,
    "total_tokens": 3934,
    "input_tokens": 3924,
    "output_tokens": 10,
    "content_filters": [
      {
        "blocked": true,
        "source_type": "completion",
        "content_filter_raw": [],
        "content_filter_results": {},
        "content_filter_offsets": {}
      }
    ],
    "incomplete_details": { "reason": "content_filter" }
  }
}
```

| Field | Notes |
|---|---|
| `status` | `"CONTENT_FILTERED"` — Java/frontend can branch on this |
| `data.story` | Contains the user-facing error message |
| `data.logline` / `tone` / `synopsis` | Empty string `""` |
| `data.beatSheet` | Empty object `{}` |
| `usage.incomplete_details.reason` | Always `"content_filter"` when blocked |
| `usage.content_filters[].blocked` | `true` for the source that was blocked |

> **Note:** The `message` envelope field stays at its standard value — `status` is the signal for routing logic.

| Field | Notes |
|---|---|
| `story` | Full scriptment, no dialogue, active present tense |
| `logline` | One sentence with actual character name |
| `tone` | Cinematic mood/style |
| `beatSheet` | Key plot beats as object with beat titles as keys |
| `synopsis` | Short producer summary |

---

## 3. Scene Treatment

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/scene`

> Use `story` and `synopsis` from the Generate Story response as inputs here.

### Request
```json
{
  "projectName": "The rural village cricket game",
  "story": "<full story text from step 2>",
  "synopsis": "<synopsis from step 2>",
  "characterList": [
    {
      "id": "char-001",
      "name": "Arjun Kumar",
      "role": "MAIN",
      "description": "A determined village boy who lives for cricket."
    },
    {
      "id": "char-002",
      "name": "Vikram Rao",
      "role": "SECOND_LEAD",
      "description": "Arjun's mentor, a pragmatic former school cricketer."
    }
  ]
}
```

| Field | Required | Notes |
|---|---|---|
| `story` | Yes | Full story from step 2 |
| `synopsis` | Yes | Synopsis from step 2 |
| `projectName` | No | Project title |
| `characterList` | No | `name`, `role`, `description` only needed |

### Response — Success
```json
{
  "status": "SUCCESS",
  "message": "Scene treatment generated successfully",
  "httpStatus": 200,
  "data": {
    "scenes": [
      {
        "sceneSeq": 1,
        "slugline": "1. EXT. VILLAGE CRICKET FIELD – DAY",
        "int_ext": "EXT.",
        "day_night": "DAY",
        "location": "VILLAGE CRICKET FIELD",
        "script_day": "",
        "pages_eighths_est": "1 2/8",
        "visual_summary": "Arjun stands on the dusty pitch, bat raised, eyes fixed on the horizon. The village crowd watches silently. He swings — the ball arcs against a gold sky. Something shifts in the air: a boy has found his moment.",
        "voice_over_tone": "",
        "slugline": "1. EXT. VILLAGE CRICKET FIELD – DAY",
        "image_prompt": "[Arjun: teenage boy, lean, dusty cricket whites] [VILLAGE CRICKET FIELD: sun-baked earth, thatched huts behind the boundary rope] golden-hour backlighting, wide lens.",
        "main_image_prompt": "[Arjun: teenage boy, lean, dusty cricket whites] [VILLAGE CRICKET FIELD: sun-baked earth, thatched huts] wide establishing shot, warm tones.",
        "main_video_prompt": "Slow push-in on Arjun at the crease, crowd in soft focus behind him, dust rising as he swings, camera follows the ball arc against the sky.",
        "storyBoardList": []
      }
    ]
  },
  "timestamp": "2026-05-05T14:31:38.123456",
  "usage": { "...see Usage Object above..." }
}
```

| Field | Notes |
|---|---|
| `sceneSeq` | Sequential scene number (integer) |
| `slugline` | Full scene header in ALL CAPS |
| `int_ext` | `"INT."` or `"EXT."` |
| `day_night` | `"DAY"` / `"NIGHT"` / `"DUSK"` / `"DAWN"` / `"EVENING"` |
| `location` | Location name from slugline |
| `script_day` | Story day number — `""` if not stated |
| `pages_eighths_est` | Page length e.g. `"1 2/8"`, `"3/8"` |
| `visual_summary` | 3–6 sentence cinematic description, no dialogue |
| `voice_over_tone` | VO tone note — `""` if not applicable |
| `image_prompt` | Character + location bracketed image prompt |
| `main_image_prompt` | Primary frame image prompt for Vertex AI |
| `main_video_prompt` | Motion prompt for RunwayML (<1000 chars) |
| `storyBoardList` | Always `[]` — Java populates this |

> Scene count is AI-determined by story complexity (minimum 40, maximum 99 scenes). Response time may be **10–20 minutes** for longer stories.

### Response — Content Filtered
```json
{
  "status": "CONTENT_FILTERED",
  "message": "Scene treatment generated successfully",
  "httpStatus": 200,
  "data": {
    "scenes": [
      {
        "sceneSeq": 1,
        "slugline": "",
        "int_ext": "",
        "day_night": "",
        "location": "",
        "script_day": "",
        "pages_eighths_est": "",
        "visual_summary": "We're sorry, this content could not be generated as it may involve sensitive or restricted themes. Please review your inputs and try again with a different topic.",
        "voice_over_tone": "",
        "image_prompt": "",
        "main_image_prompt": "",
        "main_video_prompt": "",
        "storyBoardList": []
      }
    ]
  },
  "timestamp": "2026-05-05T14:31:38.123456",
  "usage": {
    "prompt_tokens": 0, "completion_tokens": 0, "total_tokens": 0,
    "input_tokens": 0, "output_tokens": 0,
    "content_filters": [
      { "blocked": true, "source_type": "completion", "content_filter_raw": [], "content_filter_results": {}, "content_filter_offsets": {} }
    ],
    "incomplete_details": { "reason": "content_filter" }
  }
}
```

---

## 4. Generate Complete Visual Asset Package

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/visual-asset`

Generates a complete visual asset package: scene-level image and video prompts, full character visual guides, location guides, and prop guides — all created by AI from the story package.

### Request
```json
{
  "visualStyle": "Warm golden-hour cinematography, dusty earth tones, handheld during matches, locked-off wides for introspection.",
  "toneArchetype": "INSPIRATIONAL_DRAMA",
  "genre": ["DRAMA", "SPORTS"],
  "logline": "A disgraced cricket legend finds redemption coaching a village underdog team to the national championship.",
  "synopsis": "Set in rural Maharashtra...",
  "beatSheet": {
    "1. Opening Image": "...",
    "2. Catalyst": "..."
  },
  "story": "Full story text here...",
  "characterList": [
    {
      "name": "Arjun Kapoor",
      "role": "MAIN",
      "description": "Retired cricket legend, stoic, carries deep guilt.",
      "importanceLevel": "Primary",
      "storySignificance": "Protagonist — redemption arc is the emotional spine.",
      "behaviour": "...",
      "goalInternal": "...",
      "goalExternal": "...",
      "backstory": "...",
      "struggles": "...",
      "coreMotivation": "...",
      "coreFear": "...",
      "traits": "...",
      "enneagramType": "Type 1 — The Reformer"
    }
  ]
}
```

| Field | Required | Notes |
|---|---|---|
| `visualStyle` | No | Cinematography and visual aesthetic |
| `toneArchetype` | No | Overall tone |
| `genre` | No | Array of genres |
| `logline` | No | Story logline |
| `synopsis` | No | Short producer summary |
| `beatSheet` | No | Major story beats map |
| `story` | No | Full generated story text |
| `characterList` | No | Full character objects — richer input = richer visual output |

> For best results provide the complete story package from the previous generation steps.

### Response — Success
```json
{
  "status": "SUCCESS",
  "message": "Visual asset data generated successfully",
  "httpStatus": 200,
  "data": {
    "visualStyle": "Warm golden-hour cinematography, dusty earth tones...",
    "characters": [
      {
        "name": "Arjun Kapoor",
        "role": "MAIN",
        "description": "Retired cricket legend, stoic, carries deep guilt.",
        "importanceLevel": "Primary",
        "storySignificance": "Protagonist — redemption arc is the emotional spine.",
        "imagePrompt": "Arjun Kapoor headshot: mid-50s, weathered face, short greying hair, worn polo shirt, tired but dignified eyes — golden-hour hard key light, shallow depth of field.",
        "behaviour": "...",
        "goalInternal": "...",
        "goalExternal": "...",
        "backstory": "...",
        "struggles": "...",
        "enneagramType": "Type 1 — The Reformer",
        "traits": "...",
        "coreMotivation": "...",
        "coreValues": "...",
        "coreFear": "...",
        "ghost": "...",
        "lie": "...",
        "want": "...",
        "need": "...",
        "purposeOfLife": "...",
        "superObjective": "...",
        "appearanceInScenes": [1, 2, 3, 5, 8, 12, 15]
      }
    ],
    "scenes": [
      {
        "sceneSeq": 1,
        "slugline": "INT. ARJUN'S STUDY – DUSK",
        "int_ext": "INT.",
        "day_night": "DUSK",
        "location": "ARJUN'S STUDY",
        "pages_eighths_est": "2/8",
        "visual_summary": "Arjun sits alone, surrounded by dusty trophies...",
        "image_prompt": "[Arjun Kapoor: mid-50s, greying hair, worn polo] [ARJUN'S STUDY: dim room, dusty cricket trophies, single overhead bulb] melancholic warm light, shallow depth of field.",
        "main_image_prompt": "[Arjun Kapoor: mid-50s, greying hair, worn polo] [ARJUN'S STUDY: dim, dusty trophies] cinematic establishing frame, warm amber tones.",
        "main_video_prompt": "Slow push-in on Arjun seated at his desk, trophies in soft focus behind him, dust motes drifting in the single beam of light, melancholic atmosphere.",
        "storyBoardList": []
      }
    ],
    "location_visual_guide": [
      {
        "name": "ARJUN'S STUDY",
        "description": "Dim, cluttered study with dusty cricket trophies and muted lighting — a stark contrast to past glory.",
        "imagePrompt": "A dim study with dusty cricket trophies on shelves, single overhead bulb, worn desk, warm amber shadows, atmosphere of quiet neglect.",
        "appears_in_scenes": [1, 2]
      }
    ],
    "props_visual_guide": [
      {
        "name": "CRICKET BAT",
        "description": "Riya's father's old bat — cracked and aged, a tangible connection to her past.",
        "imagePrompt": "A cracked, aged cricket bat with worn grip, showing years of use, warm studio light, isolated on neutral background.",
        "appears_in_scenes": [5, 18]
      }
    ]
  },
  "timestamp": "2026-05-05T14:31:38.123456",
  "usage": { "...see Usage Object above..." }
}
```

| Field | Notes |
|---|---|
| `characters[].imagePrompt` | Detailed headshot prompt for primary character rendering (Vertex AI) |
| `characters[].appearanceInScenes` | Scene sequence numbers where this character appears |
| `scenes[].image_prompt` | Bracketed character + location frame prompt |
| `scenes[].main_image_prompt` | Primary image generation prompt |
| `scenes[].main_video_prompt` | RunwayML motion prompt — always under 1000 characters |
| `location_visual_guide[].appears_in_scenes` | Scene numbers where this location is used |
| `props_visual_guide[].appears_in_scenes` | Scene numbers where this prop appears |

### Response — Content Filtered
```json
{
  "status": "CONTENT_FILTERED",
  "message": "Visual asset data generated successfully",
  "httpStatus": 200,
  "data": {
    "visualStyle": "We're sorry, this content could not be generated as it may involve sensitive or restricted themes. Please review your inputs and try again with a different topic.",
    "characters": [],
    "scenes": [],
    "location_visual_guide": [],
    "props_visual_guide": []
  },
  "timestamp": "2026-05-05T14:31:38.123456",
  "usage": {
    "prompt_tokens": 0, "completion_tokens": 0, "total_tokens": 0,
    "input_tokens": 0, "output_tokens": 0,
    "content_filters": [
      { "blocked": true, "source_type": "completion", "content_filter_raw": [], "content_filter_results": {}, "content_filter_offsets": {} }
    ],
    "incomplete_details": { "reason": "content_filter" }
  }
}
```

---

## 5. Single Image Generation

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/generate-single-image`

Generates one image using Vertex AI Imagen 3, uploads to Yotta/S3, and returns a 7-day presigned URL.

### Request
```json
{
  "id": "922f5425-2284-410c-897b-6f4c4d215f65",
  "prompt": "Arjun headshot: mid-50s, weathered face, short greying hair, worn polo shirt, tired but dignified eyes.",
  "visualStyle": "CINEMATIC",
  "projectName": "The Last Innings",
  "aspectRatio": "16:9",
  "negativePrompt": "blurry, low quality, distorted",
  "sampleCount": 1,
  "personGeneration": "ALLOW_ALL",
  "safetySetting": "BLOCK_MEDIUM_AND_ABOVE",
  "addWatermark": false
}
```

| Field | Required | Notes |
|---|---|---|
| `id` | Yes | Asset identifier — used in output filename |
| `prompt` | Yes | Base image prompt text |
| `visualStyle` | No | Appended to prompt as `in exact <style> style` |
| `projectName` | No | Metadata/context field |
| `aspectRatio` | No | `16:9` `1:1` `9:16` `4:3` `3:4` |
| `negativePrompt` | No | Undesired elements |
| `sampleCount` | No | Number of samples (1–4) |
| `personGeneration` | No | `ALLOW_ALL` `ALLOW_ADULT` `DONT_ALLOW` |
| `safetySetting` | No | Vertex safety threshold |
| `addWatermark` | No | Vertex watermark toggle |

### Response
```json
{
  "data": {
    "outputs": [
      {
        "imageUrl": "https://nm1ecs.yotta.com:9021/previsualization/text-image/image_922f5425-2284-410c-897b-6f4c4d215f65.png?...",
        "seed": 1234,
        "finishReason": "SUCCESS"
      }
    ],
    "usage": {
      "image_count": 1
    }
  }
}
```

---

## 6. Single Video Generation

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/generate-single-video`

Generates one video using RunwayML Gen-4 Turbo (image-to-video), uploads to Yotta/S3, and returns a 7-day presigned URL.

### Request
```json
{
  "id": "6aa2cac2-ba52-443c-8813-b6ef8c43e515",
  "prompt": "Camera slowly pushes in as wind moves through the field and dust rises.",
  "imageUrl": "https://nm1ecs.yotta.com:9021/previsualization/text-image/image_922f5425.png?...",
  "projectName": "The Last Innings"
}
```

| Field | Required | Notes |
|---|---|---|
| `id` | Yes | Asset identifier — used in output filename |
| `prompt` | Yes | Motion/video prompt |
| `imageUrl` | Yes | Source image URL (presigned or public) |
| `projectName` | No | Metadata/context field |

### Response
```json
{
  "data": {
    "outputs": [
      {
        "videoUrl": "https://nm1ecs.yotta.com:9021/previsualization/image-video/video_6aa2cac2-ba52-443c-8813-b6ef8c43e515.mp4?...",
        "duration": 5,
        "finishReason": "SUCCESS"
      }
    ],
    "usage": {
      "video_count": 1
    }
  }
}
```

---

## Error Reference

```json
{ "error": "HTTP_ERROR", "message": "Reason here" }
```

| Code | Meaning |
|---|---|
| `422` | Missing required field or content blocked by Azure (pitch-deck endpoint only) |
| `500` | AI generation failed — unexpected error |

> Text-generation APIs (1–4) never return 500 for content filtering — they return HTTP 200 with `status: "CONTENT_FILTERED"` instead.

---

# UPLOAD SCRIPT APIs (File Processing)

These three endpoints accept a PDF or DOCX file via `multipart/form-data` and automatically extract or generate the necessary data for the Spotlight application.

## 7. Process Synopsis (One-Liner)

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/process/synopsis`

Accepts a short idea or synopsis. Automatically extracts and generates characters, genre, and key story parameters.
**Fills Screen 1 only (AutoFill).**

### Request (multipart/form-data)
| Field | Type | Notes |
|---|---|---|
| `file` | File | Synopsis document (PDF/DOCX/TXT) |

### Response
```json
{
  "status": "SUCCESS",
  "message": "Synopsis processed successfully",
  "httpStatus": 200,
  "sessionId": "df6328f0-2f3a-4530-b85f-99b296b70436",
  "uploadType": "SYNOPSIS",
  "processingMode": "GENERATION",
  "suggestedStopScreen": 1,
  "primaryLanding": "AUTOFILL_CORE_DETAILS",
  "screenScores": { "screen1": 100 },
  "data": {
    "autoFill": {
      "projectName": "The Last Signal",
      "storyIdea": "...",
      "sceneCount": 50,
      "genre": ["SCI_FI", "THRILLER"],
      "narrativeStyle": "LINEAR",
      "storyStructure": "THREE_ACT",
      "endingType": "AMBIGUOUS",
      "characterList": [
        {
          "name": "Dr. Eleanor Voss",
          "importanceLevel": "LEAD",
          "role": "PROTAGONIST",
          "description": "...",
          "traits": ["INTELLIGENT", "DETERMINED"]
        }
      ]
    }
  },
  "nextSteps": [
    "User reviews and edits auto-filled Core Details",
    "User proceeds to Story Generation (Screen 2)",
    "User proceeds to Scene Treatment (Screen 3)"
  ]
}
```

---

## 8. Process Story (Narrative)

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/process/story`

Accepts a prose narrative or treatment. Extracts core details and builds a beat sheet while preserving the original story verbatim.
**Fills Screen 1 + Screen 2 (AutoFill & Generate Story).** Scene treatment is NOT generated automatically.

### Request (multipart/form-data)
| Field | Type | Notes |
|---|---|---|
| `file` | File | Story document (PDF/DOCX/TXT) |

### Response
```json
{
  "status": "SUCCESS",
  "message": "Story processed successfully",
  "httpStatus": 200,
  "sessionId": "97f51e7b-d9fe-46d0-b204-08cb0f66937c",
  "uploadType": "STORY",
  "processingMode": "EXTRACTION",
  "suggestedStopScreen": 2,
  "primaryLanding": "GENERATE_STORY",
  "screenScores": { "screen1": 100, "screen2": 100 },
  "data": {
    "autoFill": {
      "projectName": "THE BRIDGE BETWEEN US",
      "genre": ["DRAMA"],
      "characterList": [
        { "name": "Arjun Mehta", "importanceLevel": "LEAD", "role": "PROTAGONIST" },
        { "name": "Ramesh Mehta", "importanceLevel": "SUPPORTING", "role": "MENTOR" }
      ]
    },
    "generateStory": {
      "story": "Original story text preserved verbatim...",
      "logline": "...",
      "beatSheet": {
        "setup": "...",
        "incitingIncident": "...",
        "firstTurningPoint": "..."
      },
      "synopsis": "..."
    }
  }
}
```

---

## 9. Process Screenplay (Bound Script)

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/process/screenplay`

Accepts a full structured script/screenplay. Performs a complete end-to-end extraction and generation.
**Fills Screen 1 + Screen 2 + Screen 3 (AutoFill, Generate Story, Scene Treatment).**

### Request (multipart/form-data)
| Field | Type | Notes |
|---|---|---|
| `file` | File | Bound script document (PDF/DOCX/FDX/FOUNTAIN) |

### Response
```json
{
  "status": "SUCCESS",
  "message": "Screenplay processed successfully",
  "httpStatus": 200,
  "sessionId": "60874537-e7ed-44eb-8162-49a04ea8c360",
  "uploadType": "SCREENPLAY",
  "processingMode": "EXTRACTION",
  "suggestedStopScreen": 3,
  "primaryLanding": "SCENE_TREATMENT",
  "screenScores": { "screen1": 100, "screen2": 100, "screen3": 100 },
  "data": {
    "autoFill": {
      "projectName": "PK",
      "genre": ["COMEDY", "DRAMA", "SCI_FI"],
      "characterList": [
        { "name": "PK", "importanceLevel": "LEAD", "role": "PROTAGONIST" },
        { "name": "Jaggu", "importanceLevel": "SUPPORTING", "role": "SIDEKICK" }
      ]
    },
    "generateStory": {
      "story": "Generated scriptment based on screenplay...",
      "logline": "...",
      "beatSheet": { "setup": "...", "incitingIncident": "..." },
      "synopsis": "..."
    },
    "generateScenes": {
      "scenes": [
        {
          "sceneSeq": 1,
          "slugline": "EXT. SPACE – NIGHT",
          "int_ext": "EXT.",
          "day_night": "NIGHT",
          "location": "SPACE",
          "script_day": "",
          "pages_eighths_est": "1 1/8",
          "visual_summary": "In the vastness of space, millions of twinkling stars fill the frame...",
          "voice_over_tone": "",
          "image_prompt": "...",
          "main_image_prompt": "...",
          "main_video_prompt": "...",
          "storyBoardList": []
        }
      ]
    }
  }
}
```
