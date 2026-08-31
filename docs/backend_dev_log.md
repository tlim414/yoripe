---
layout: default
title: "Backend Dev Log"
permalink: /backend-dev-log
---

# Frontend Dev Log
This page serves as a logbook for the frontend changes

## Updae API for recipe info extraction using Vision LLM
### August 30, 2026
Added extraction endpoint: Created /extract-recipe-info to process uploaded images using Google Gemini 1.5 Flash.
-	In-memory image handling: Configured multer memory storage instead of cloud storage, as storing the original photo in the UI is unnecessary. This keeps data lifecycle management simple and reduces infrastructure overhead.
- Structured output schema: Enforced a standardized JSON response schema for the vision model:

```text
properties: {
  title: { type: Type.STRING },
  description: { type: Type.STRING },
  instructions: {
    type: Type.ARRAY,
    items: { type: Type.STRING },
  },
  ingredients: {
    type: Type.ARRAY,
    items: {
      type: Type.OBJECT,
      properties: {
        name: { type: Type.STRING },
        amount: { type: Type.STRING },
        unit: { type: Type.STRING },
      },
      required: ['name', 'amount', 'unit'],
    },
  },
},
required: ['title', 'description', 'instructions', 'ingredients'],
```

Error handling:
-	Returns 400 Bad Request if no image file is present in the payload.
-	Returns 500 Internal Server Error with details if the LLM request fails.
