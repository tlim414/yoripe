---
layout: default
title: "Backend Dev Log"
permalink: /backend-dev-log
---

# Frontend Dev Log
This page serves as a logbook for the backend changes

## Update API for recipe info extraction using Vision LLM
### August 30, 2026
Added extraction endpoint: Created `POST:/extract-recipe-info` to process uploaded images using Google Gemini 1.5 Flash.
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
-	Returns `400 Bad Request` if no image file is present in the payload.
-	Returns `500 Internal Server Error` with details if the LLM request fails.

Update `.env`
- Added `GEMINI_API_KEY` environment variable

## Configure EsLint and Prettier
### August 11
- Added EsLint and Prettier to backend codebase
- Linting and formatting is run by `pnpm` scripts in `package.json`:
  - `"lint": "eslint . --report-unused-disable-directives --max-warnings 0"`
  - `"lint:fix": "eslint . --fix"`
  - `"format": "prettier --write \"src/**/*.{ts,js,json,md}\""`
  - `"check": "pnpm lint && pnpm build"`

## Update dev environment
### July 22
Updated `.env`
- Created `.env.example` to keep environment consistent with clones
- Outlined the environment variables needed to run backend server including:

| Variable | Local? |
| :------- | :----- |
| PORT | NO |
| FRONTEND_URL | Yes |
| DATABASE_URL | NO |
| DIRECT_URL | NO |
| CLERK_PUBLISHABLE_KEY | Yes |
| CLERK_SECRET_KEY | Yes |

## Add Core Endpoints
### July 22
Setup local environment variables
- Read `DATABASE_URL` and `PORT` number from `.env`
  
Add reading, creating, updating and deleting recipe endpoints
- All endpoints perform an auth check at the start to make sure user is signed in. Returns `401 Unauthorized` on authentication failure.

Create Recipe Endpoint: Created `POST:/recipes` to create and save a recipe to the database.
- Using Prisma's `create` function on `Recipe` table, creates a new recipe entry with the data provided from request body
- Returns `201 Created` on successful recipe creation
- Returns `500 Internal Server Error` with details if the creation failed

Get All Recipes for User Endpoint: Created `GET:/recipes` endpoint for retrieving a list of all recipes for the user
- Configured to return a list of recipe objects with the following data:
  ```text
  {
    id: stirng,
    title: string,
    description: string,
    createdAt: DateTime
  }
  ```
- Accepts query params `q` and `by` representing query string and the query filter respectively
  - Accepts `title` or `ingredient` for filters
  - Defaults to matching in all fields of recipe data 
- Returns `200 Ok` on successful recipe retrieval with recipe list JSON data
- Returns `500 Internal Server Error` with details if fetching recipes for user failed

Get Detailed Info of a Recipe by ID: Created `GET:/recipes/:id/` endpoint for fetching info on recipe with id `id`
- Configured to return all data related to a specific recipe'
- Returns the data including all fields (title, description, instructiion list and ingredient list)
- Returns `200 Ok` on successful recipe retrieval with recipe list JSON data
- Returns `500 Internal Server Error` with details if fetching recipes for user failed
  
Update a Recipe by ID: Created `PATCH:/recipes/:id` endpoint to update fields of recipe with id `id`
- Configured to update the database entry using the provided fields in request body
- Returns `200 Ok` on successful recipe retrieval with recipe list JSON data
- Returns `500 Internal Server Error` with details if fetching recipes for user failed

Delete a Recipe: Created `DELETE:/recipes/:id/` endpoint to delete a recipe with id `id`
- Given the id of a recipe, deletes the database entry with primary key `id` from `Recipe` table
- Also deleted all ingredients from `Ingredient` table with foreign key of `recipeID` matching `id`
- Returns `200 Ok` on successful recipe retrieval with recipe list JSON data
- Returns `500 Internal Server Error` with details if fetching recipes for user failed
  
Setup schema for Prisma
- Defined Recipe table schema
- Defined Ingredient table schema
- Decided not to create a schema for users since auth is being handled by Clerk and `userId` corresponds to the id provided by Clerk
