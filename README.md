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
    "total_tokens": 5740
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
  "timestamp": "2026-04-19T14:31:38.123456",
  "usage": {
    "prompt_tokens": 1240,
    "completion_tokens": 4500,
    "total_tokens": 5740
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
      "description": "A determined village boy who lives for cricket."
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
| `story` | Yes | Full story from step 2 |
| `synopsis` | Yes | Synopsis from step 2 |
| `projectName` | No | Project title |
| `characterList` | No | `name`, `role`, `description` only |

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
  "timestamp": "2026-04-19T14:31:38.123456",
  "usage": {
    "prompt_tokens": 1240,
    "completion_tokens": 4500,
    "total_tokens": 5740
  }
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
      "description": "Mid-30s, lean but muscular, with a prominent scar on his bowling wrist."
    }
  ]
}
```

| Field | Required | Notes |
|---|---|---|
| `visualStyle` | No | Cinematography and visual esthetic |
| `toneArchetype` | No | Overall tone |
| `genre` | No | List of genres |
| `logline` | No | Story logline |
| `synopsis` | No | Short producer summary |
| `beatSheet` | No | Major story beats map |
| `story` | No | Full generated story text |
| `characterList` | No | List of characters with descriptions |

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
    "scenes": [
      {
        "sceneSeq": 1,
        "slugline": "SCENE 1. EXT. VILLAGE CRICKET PITCH – EVENING",
        "intExt": "EXT.",
        "dayNight": "EVENING",
        "location": "VILLAGE CRICKET PITCH",
        "pagesEighthsEst": "2/8",
        "visualSummary": "Vikram stands alone, staring at the rundown cricket pitch...",
        "image": {
          "prompt": "[Vikram: mid-30s... in worn cricket gear] [VILLAGE CRICKET PITCH: dusty field] sunset backlighting...",
          "video": {
            "prompt": "[Vikram: mid-30s...] [VILLAGE CRICKET PITCH: dusty field] dolly shot moving slowly towards Vikram's face..."
          }
        }
      }
    ]
  },
  "timestamp": "2026-04-26T14:54:43.641547",
  "usage": {
    "prompt_tokens": 1240,
    "completion_tokens": 4500,
    "total_tokens": 5740
  }
}
```

| Field | Notes |
|---|---|
| `characters[].imagePrompt` | Detailed headshot prompt meant for primary character rendering |
| `characters[].appearanceInScenes` | Array of scene sequences where character appears |
| `scenes[].image.prompt` | Frame generation prompt combining characters and location |
| `scenes[].image.video.prompt` | Motion generation video prompt for runway/kling (<1000 chars) |

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
  "personGeneration": "ALLOW_ALL",
  "safetySetting": "BLOCK_MEDIUM_AND_ABOVE",
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
| `sampleCount` | No | Number of samples to request |
| `personGeneration` | No | Person policy for generation |
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
