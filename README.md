# Spotlight AI — API Reference

Python AI service covering three workflows: **Autofill**, **Story Generation**, and **Scene Treatment**. All follow the same standard request/response envelope.

---

## 1. Autofill

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/autofill`

Takes partial project data and basic character info — returns all fields fully filled with AI-generated values including complete character depth.

### Request
```json
{
  "projectName": "The rural village cricket game",
  "storyIdea": "A village boy dreams of playing cricket for India",
  "genre": ["COMEDY"],
  "narrativeStyle": "COMMERCIAL_MASS_ENTERTAINER",
  "storyStructure": "THREE_ACT",
  "pace": "BALANCED_NARRATIVE",
  "theme": "FAMILY_AND_BELONGING",
  "contentSensitivity": "CLEAN_FAMILY_SAFE",
  "toneArchetype": "FEEL_GOOD",
  "endingType": "HAPPY_ENDING",
  "region": "Rural India",
  "targetAudience": ["FAMILY_ALL_AGES"],
  "referenceFilmsWorks": "Lagaan, Chak De India",
  "characterList": [
    {
      "name": "Arjun Kumar",
      "role": "MAIN",
      "description": "A determined village boy who lives for cricket."
    },
    {
      "name": "Vikram Rao",
      "role": "SECOND_LEAD",
      "description": "Arjun's mentor and former school cricketer."
    }
  ]
}
```

| Field | Notes |
|---|---|
| All fields | Optional — send whatever is available |
| `characterList` | Only `name`, `role`, `description` needed |
| `role` values | `MAIN` `SECOND_LEAD` `VILLAIN` `SUPPORTING` |

### Response
```json
{
  "status": "SUCCESS",
  "message": "Story parameters auto-filled successfully",
  "httpStatus": 200,
  "data": {
    "projectName": "...",
    "storyIdea": "...",
    "genre": ["COMEDY"],
    "narrativeStyle": "...",
    "storyStructure": "...",
    "endingType": "...",
    "openingSetupIdea": "...",
    "climaxIdea": "...",
    "pace": "...",
    "toneArchetype": "...",
    "theme": "...",
    "region": "...",
    "targetAudience": ["..."],
    "contentSensitivity": "...",
    "referenceFilmsWorks": "...",
    "characterList": [
      {
        "name": "Arjun Kumar",
        "role": "MAIN",
        "description": "...",
        "importanceLevel": "...",
        "storySignificance": "...",
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
        "superObjective": "..."
      }
    ]
  },
  "timestamp": "2026-04-19T14:31:38.123456",
  "usage": {
    "prompt_tokens": 1240,
    "completion_tokens": 4500,
    "total_tokens": 5740,
    "input_tokens": 1240,
    "output_tokens": 4500
  }
}
```

| Field | Notes |
|---|---|
| All story fields | Completed/corrected by AI |
| `characterList` | Full character depth — all 20 fields filled |

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
      "name": "Arjun Kumar",
      "role": "MAIN",
      "description": "A determined village boy who lives for cricket.",
      "goalInternal": "Prove his worth to his family.",
      "goalExternal": "Get selected for the national cricket team.",
      "backstory": "Grew up in poverty, cricket was his only escape.",
      "coreMotivation": "Recognition and belonging.",
      "coreFear": "Being ordinary.",
      "ghost": "His father's disappointment after a crucial match loss.",
      "lie": "He believes he must win alone to matter.",
      "want": "A place in the national squad.",
      "need": "To trust others and accept help."
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
| `creativeNotes` | No | Extra direction |
| `characterList` | No | **Full character object from Autofill** — forwarded to AI for consistent character arcs |

> Pass the `characterList` from the Autofill API response directly here. The AI will use the character profiles (name, role, goals, backstory, fears, etc.) to build a story with consistent, grounded character arcs.

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
  "timestamp": "2026-04-19T14:31:38.123456",
  "usage": {
    "prompt_tokens": 1240,
    "completion_tokens": 4500,
    "total_tokens": 5740,
    "input_tokens": 1240,
    "output_tokens": 4500
  }
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

## 3. Scene Treatment

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/scene`

> Use `story` and `synopsis` from the Generate Story response as inputs here.

### Request
```json
{
  "projectName": "The rural village cricket game",
  "story": "<story from step 2>",
  "synopsis": "<synopsis from step 2>",
  "characterList": [
    {
      "name": "Arjun Kumar",
      "role": "MAIN",
      "description": "A determined village boy who lives for cricket.",
      "importanceLevel": "Primary",
      "storySignificance": "The protagonist whose journey drives the story.",
      "behaviour": "Determined, passionate, occasionally reckless.",
      "goalInternal": "Prove his worth to his family.",
      "goalExternal": "Get selected for the national cricket team.",
      "backstory": "Grew up in poverty, cricket was his only escape.",
      "struggles": "Self-doubt and financial hardship.",
      "enneagramType": "3 - The Achiever",
      "traits": "Hardworking, loyal, impulsive",
      "coreMotivation": "Recognition and belonging.",
      "coreValues": "Family, integrity, perseverance.",
      "coreFear": "Being ordinary.",
      "ghost": "His father's disappointment after a crucial match loss.",
      "lie": "He believes he must win alone to matter.",
      "want": "A place in the national squad.",
      "need": "To trust others and accept help.",
      "purposeOfLife": "To inspire his village through sport.",
      "superObjective": "Represent India on the world stage."
    }
  ]
}
```

| Field | Required | Notes |
|---|---|---|
| `story` | Yes | Full story from step 2 |
| `synopsis` | Yes | Synopsis from step 2 |
| `projectName` | No | Project title |
| `characterList` | No | Full character object — same structure as Autofill response |

> `characterList` now accepts the **full character object** from the Autofill API response (all 20+ fields). Only `name` and `role` are strictly needed but the more fields provided, the richer the scenes.

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
        "slugline": "SCENE 1. EXT. VILLAGE CRICKET FIELD – DAY",
        "int_ext": "EXT.",
        "day_night": "DAY",
        "location": "VILLAGE CRICKET FIELD",
        "script_day": "Day 1",
        "pages_eighths_est": "1 2/8",
        "visual_summary": "Arjun stands alone on the dusty pitch, gripping a worn cricket ball. The golden afternoon light catches the dust as he winds up for a bowl, his face set with quiet determination.",
        "voice_over_tone": "Hopeful, introspective",
        "main_image_prompt": "[Arjun Kumar: lean teenage boy, sun-darkened skin, worn whites] [VILLAGE CRICKET FIELD: dusty red-soil pitch, eucalyptus trees] golden afternoon backlight, wide establishing shot.",
        "main_video_prompt": "Slow dolly push-in toward Arjun as he releases the ball. Dust rises around his feet. Handheld camera tilts up to reveal the vast empty field behind him."
      }
    ]
  },
  "timestamp": "2026-04-19T14:31:38.123456",
  "usage": {
    "prompt_tokens": 1240,
    "completion_tokens": 4500,
    "total_tokens": 5740,
    "input_tokens": 1240,
    "output_tokens": 4500
  }
}
```

| Field | Notes |
|---|---|
| `sceneSeq` | Sequential scene number |
| `slugline` | Scene header — `SCENE #. INT./EXT. LOCATION – DAY/NIGHT` |
| `int_ext` | `INT.` or `EXT.` |
| `day_night` | `DAY` / `NIGHT` / `DAWN` / `DUSK` |
| `location` | Location name in ALL CAPS |
| `script_day` | Optional script-day estimate |
| `pages_eighths_est` | Page length e.g. `1 2/8` |
| `visual_summary` | 3–6 sentence cinematic visual description |
| `voice_over_tone` | Optional voice-over tone note |
| `main_image_prompt` | Primary image generation prompt |
| `main_video_prompt` | Video/motion generation prompt |

> **Removed fields:** `imagePrompt` and `storyBoardList` are no longer returned in the scene treatment response.

> Scene count is AI-determined by story complexity (40–99 scenes). Response time may be 5–15 minutes.

---

## 4. Generate Complete Visual Asset Package

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/visual-asset`

Generates a complete visual asset package containing scene-by-scene image and video prompts, as well as comprehensive character visual guides.

### Request
```json
{
  "visualStyle": "Dusty Village Evenings, High Contrast Sunsets, Gritty Sports Action",
  "toneArchetype": "INSPIRATIONAL_DRAMA",
  "genre": ["DRAMA", "SPORTS"],
  "logline": "A former star bowler in a remote village must rediscover his courage to lead a ragtag cricket team against a corrupt landlord's dominance.",
  "synopsis": "Vikram, once the village's fastest bowler...",
  "beatSheet": {
      "1. Setup": "...",
      "2. Inciting Incident": "..."
  },
  "story": "Full story text here...",
  "characterList": [
    {
      "name": "Vikram",
      "role": "MAIN",
      "description": "Mid-30s, lean but muscular, with a prominent scar on his bowling wrist.",
      "importanceLevel": "Primary",
      "storySignificance": "Vikram's journey of redemption anchors the entire story.",
      "behaviour": "Pragmatic, disciplined, and strategically sharp.",
      "goalInternal": "To rediscover his self-worth and courage.",
      "goalExternal": "To lead the village cricket team to victory.",
      "backstory": "Once the village's fastest bowler, a wrist injury ended his career.",
      "struggles": "Guilt, self-doubt, fear of failure in public.",
      "enneagramType": "1 - The Reformer",
      "traits": "Disciplined, stubborn, protective",
      "coreMotivation": "Reclaim dignity for himself and his village.",
      "coreValues": "Honour, community, resilience.",
      "coreFear": "Being seen as a failure.",
      "ghost": "The match he walked away from years ago.",
      "lie": "He believes his best days are behind him.",
      "want": "To be left alone.",
      "need": "To reconnect with his purpose through others.",
      "purposeOfLife": "To serve his community through sport.",
      "superObjective": "Restore hope to the village."
    }
  ]
}
```

| Field | Required | Notes |
|---|---|---|
| `visualStyle` | No | Cinematography and visual aesthetic |
| `toneArchetype` | No | Overall tone |
| `genre` | No | List of genres |
| `logline` | No | Story logline |
| `synopsis` | No | Short producer summary |
| `beatSheet` | No | Major story beats map |
| `story` | No | Full generated story text |
| `characterList` | No | **Full character object** — same structure as Autofill response (all 20+ fields) |

> Though technically optional fields, for best results provide the complete story package from the previous generation steps.

### Response
```json
{
  "status": "SUCCESS",
  "message": "Visual asset data generated successfully",
  "httpStatus": 200,
  "data": {
    "visualStyle": "Dusty Village Evenings...",
    "characters": [
      {
        "name": "Vikram",
        "description": "Mid-30s, lean but muscular build...",
        "importanceLevel": "Primary",
        "storySignificance": "Vikram's journey of redemption...",
        "imagePrompt": "Vikram headshot: mid-30s, lean but muscular... golden-hour hard key light.",
        "behaviour": "Pragmatic, disciplined, and strategically sharp",
        "role": "MAIN",
        "goalInternal": "To rediscover his self-worth...",
        "goalExternal": "To lead the cricket team...",
        "appearanceInScenes": [1, 2, 3, 4, 5, 6, 10]
      }
    ],
    "location_visual_guide": [
      {
        "name": "VILLAGE CRICKET PITCH",
        "description": "A sun-baked red-soil pitch surrounded by eucalyptus trees and weathered wooden stands. Dust rises with every footstep.",
        "imagePrompt": "Wide-angle shot of a dusty village cricket pitch at golden hour, red soil, sparse wooden stands, eucalyptus trees lining the boundary, dramatic low sun casting long shadows.",
        "appears_in_scenes": [1, 2, 5, 10, 22, 35]
      }
    ],
    "props_visual_guide": [
      {
        "name": "Vikram's Worn Cricket Ball",
        "description": "A decades-old red leather cricket ball, seams fraying, surface scuffed. A symbol of Vikram's abandoned past.",
        "imagePrompt": "Macro close-up of a worn red cricket ball on dry cracked earth, leather cracked and seams splitting, dramatic side lighting.",
        "appears_in_scenes": [1, 8, 19, 40]
      }
    ],
    "scenes": [
      {
        "sceneSeq": 1,
        "slugline": "SCENE 1. EXT. VILLAGE CRICKET PITCH – EVENING",
        "int_ext": "EXT.",
        "day_night": "EVENING",
        "location": "VILLAGE CRICKET PITCH",
        "pages_eighths_est": "2/8",
        "visual_summary": "Vikram stands alone, staring at the rundown cricket pitch as the last light fades.",
        "image_prompt": "[Vikram: mid-30s, lean, worn cricket whites] [VILLAGE CRICKET PITCH: dusty red field, eucalyptus trees] sunset backlighting, wide shot.",
        "main_video_prompt": "[Vikram] [VILLAGE CRICKET PITCH] dolly shot moving slowly towards Vikram's silhouette as the sun sets behind the boundary trees."
      }
    ]
  },
  "timestamp": "2026-04-26T14:54:43.641547",
  "usage": {
    "prompt_tokens": 1240,
    "completion_tokens": 4500,
    "total_tokens": 5740,
    "input_tokens": 1240,
    "output_tokens": 4500
  }
}
```

| Field | Notes |
|---|---|
| `characters[].imagePrompt` | Detailed headshot prompt for character image generation |
| `characters[].appearanceInScenes` | Scene sequence numbers where character appears |
| `location_visual_guide[].description` | Visual/atmospheric description of the location |
| `location_visual_guide[].imagePrompt` | Image generation prompt for the location |
| `location_visual_guide[].appears_in_scenes` | Scene sequences where this location is used |
| `props_visual_guide[].description` | Description and narrative significance of the prop |
| `props_visual_guide[].imagePrompt` | Image generation prompt for the prop |
| `props_visual_guide[].appears_in_scenes` | Scene sequences where this prop appears |
| `scenes[].image_prompt` | Frame generation prompt combining characters and location |
| `scenes[].main_video_prompt` | Motion/video generation prompt (<1000 chars) |

---

## 5. Single Image Generation

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/generate-single-image`

Generates one image using Vertex AI Imagen, uploads it to storage, and returns a 7-day presigned URL.

### Request
```json
{
  "id": "922f5425-2284-410c-897b-6f4c4d215f65",
  "prompt": "Aion headshot: mid-30s, fair-olive complexion, short cropped dark hair, clean-lined neutral tunic.",
  "visualStyle": "CINEMATIC",
  "projectName": "Visual Asset Project",
  "aspectRatio": "16:9",
  "negativePrompt": "blurry, low quality, distorted",
  "sampleCount": 1,
  "addWatermark": false
}
```

| Field | Required | Notes |
|---|---|---|
| `id` | Yes | Asset identifier used in output filename |
| `prompt` | Yes | Base image prompt text |
| `visualStyle` | No | Appended as `in exact <style> style` |
| `projectName` | No | Metadata/context field |
| `aspectRatio` | No | Example: `16:9`, `1:1`, `9:16` |
| `negativePrompt` | No | Undesired elements |
| `sampleCount` | No | Number of samples (default: 1) |
| `addWatermark` | No | Vertex watermark toggle (default: false) |

> **Removed fields:** `personGeneration` and `safetySetting` are no longer accepted in the request body.

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

Generates one video using Runway Gen-4 Turbo from a source image URL, uploads to storage, and returns a 7-day presigned URL.

### Request
```json
{
  "id": "6aa2cac2-ba52-443c-8813-b6ef8c43e515",
  "prompt": "Camera slowly pushes in as wind moves through the field and dust rises.",
  "imageUrl": "https://nm1ecs.yotta.com:9021/previsualization/text-image/image_cea626dd-a9e5-46eb-b0ac-2a4d24927b48.png?...",
  "projectName": "Visual Asset Project"
}
```

| Field | Required | Notes |
|---|---|---|
| `id` | Yes | Asset identifier used in output filename |
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

## Errors

```json
{ "error": "HTTP_ERROR", "message": "Reason here" }
```

| Code | Meaning |
|---|---|
| `422` | Missing required field |
| `500` | AI generation failed |

---

# UPLOAD SCRIPT APIs (File Processing)

These three endpoints accept a PDF or DOCX file via `multipart/form-data` and automatically extract or generate the necessary data for the Spotlight application. They are distinct from the standard generation APIs as they ingest user files directly.

## 7. Process Synopsis (One-Liner)

**POST** `https://dev-api-gateway.storygenartist.com/spotlight-ai-api/process/synopsis`

Accepts a short idea or synopsis. Automatically extracts and generates characters, genre, and key story parameters.
**Fills Screen 1 only (AutoFill).**

### Request (multipart/form-data)
| Field | Type | Notes |
|---|---|---|
| `file` | File | The synopsis document (PDF/DOCX) |

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
  "screenScores": {
    "screen1": 100
  },
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
        },
        {
          "name": "Director Hayes",
          "importanceLevel": "SUPPORTING",
          "role": "ANTAGONIST",
          "description": "...",
          "traits": ["RUTHLESS", "AUTHORITATIVE"]
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

Accepts a prose narrative or treatment. Extracts core details and builds a beat sheet, while preserving the original story verbatim.
**Fills Screen 1 + Screen 2 (AutoFill & Generate Story).** Scene treatment is NOT generated automatically.

### Request (multipart/form-data)
| Field | Type | Notes |
|---|---|---|
| `file` | File | The story document (PDF/DOCX) |

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
  "screenScores": {
    "screen1": 100,
    "screen2": 100
  },
  "data": {
    "autoFill": {
      "projectName": "THE BRIDGE BETWEEN US",
      "genre": ["DRAMA"],
      "characterList": [
        {
          "name": "Arjun Mehta",
          "importanceLevel": "LEAD",
          "role": "PROTAGONIST"
        },
        {
          "name": "Ramesh Mehta",
          "importanceLevel": "SUPPORTING",
          "role": "MENTOR"
        }
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

Accepts a full structured script/screenplay. Performs a complete end-to-end extraction and generation process.
**Fills Screen 1 + Screen 2 + Screen 3 (AutoFill, Generate Story, and Scene Treatment).**

### Request (multipart/form-data)
| Field | Type | Notes |
|---|---|---|
| `file` | File | The bound script document (PDF/DOCX) |

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
  "screenScores": {
    "screen1": 100,
    "screen2": 100,
    "screen3": 100
  },
  "data": {
    "autoFill": {
      "projectName": "PK",
      "genre": ["COMEDY", "DRAMA", "SCI_FI"],
      "characterList": [
        {
          "name": "PK",
          "importanceLevel": "LEAD",
          "role": "PROTAGONIST"
        },
        {
          "name": "Jaggu",
          "importanceLevel": "SUPPORTING",
          "role": "SIDEKICK"
        }
      ]
    },
    "generateStory": {
      "story": "Generated scriptment based on screenplay...",
      "logline": "...",
      "beatSheet": { 
        "setup": "...",
        "incitingIncident": "..."
      },
      "synopsis": "..."
    },
    "generateScenes": {
      "scenes": [
        {
          "sceneSeq": 1,
          "intExt": "EXT",
          "dayNight": "NIGHT",
          "location": "SPACE",
          "pagesEighthsEst": "1 1/8",
          "visualSummary": "In the vastness of space, millions of twinkling stars...",
          "slugline": "EXT. SPACE - NIGHT (1 1/8)"
        },
        {
          "sceneSeq": 2,
          "intExt": "EXT",
          "dayNight": "NIGHT",
          "location": "SAMBHAR LAKE",
          "pagesEighthsEst": "1 2/8",
          "visualSummary": "A wide barren landscape unfolds, dominated by a small temple...",
          "slugline": "EXT. SAMBHAR LAKE - NIGHT (1 2/8)"
        }
      ]
    }
  }
}
```

---

## 10. Content Blocked — AI Safety Filter Response

When input content triggers Azure OpenAI's content safety filters (e.g. terrorism, extreme violence, adult content), the API does **not** return a `500` error. Instead it returns a structured `200` response with `status: BLOCKED`, surfacing the blocked message inside the existing data fields so the client can display a clear message to the user.

This applies to: **`/autofill`** and **`/story`** endpoints.

### Example — Blocked Story Generation

**Request**
```json
{
  "storyIdea": "Generate a detailed story glorifying extreme violence and illegal terrorist activities.",
  "genre": ["Action"],
  "storyStructure": "Three-Act",
  "narrativeStyle": "Realistic",
  "creativeNotes": "",
  "endingType": "Happy",
  "openingSetupIdea": "",
  "climaxIdea": "",
  "pace": "Fast",
  "toneArchetype": "Dark",
  "theme": "Conflict",
  "region": "Global",
  "targetAudience": ["Adults"],
  "contentSensitivity": "High",
  "referenceFilmsWorks": ""
}
```

**Response**
```json
{
  "status": "BLOCKED",
  "message": "Content blocked by AI safety filter",
  "httpStatus": 200,
  "data": {
    "story": "I'm sorry, this story could not be generated due to content sensitivity restrictions imposed by the AI safety system.",
    "logline": "I'm sorry, this story could not be generated due to content sensitivity restrictions imposed by the AI safety system.",
    "tone": "",
    "beatSheet": {},
    "synopsis": "I'm sorry, this story could not be generated due to content sensitivity restrictions imposed by the AI safety system."
  },
  "timestamp": "2026-05-03T05:45:12.063505",
  "usage": {
    "prompt_tokens": 3892,
    "completion_tokens": 5911,
    "total_tokens": 9803,
    "input_tokens": 3892,
    "output_tokens": 5911
  }
}
```

| Field | Value | Notes |
|---|---|---|
| `status` | `BLOCKED` | Distinguishes from `SUCCESS` |
| `httpStatus` | `200` | Not a server error — client must check `status` field |
| `message` | `Content blocked by AI safety filter` | Human-readable reason |
| `data.story` / `data.storyIdea` | Blocked message string | Reuses existing field — no new attributes added |
| `usage` | Populated | Token counts still returned even for blocked calls |

> **How to handle on the client side:** Check the `status` field. If `"BLOCKED"`, display the message from `data.story` (or `data.storyIdea` for autofill) to the user as a content restriction notice. Do **not** treat this as an error — the HTTP status is `200`.

