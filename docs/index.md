# Yoripe
Yoripe is a full-stack, decoupled recipe vault application designed for high performance and ease of use during cooking. The system uses a modern TypeScript stack to maintain strict end-to-end type safety, separating front-end interaction logic from backend database management.
The following page is the docs for the web app and serves as a developer build log on design choices.

# Table of Contents
- [Architecture & Tech Stack](#architecture)
- [Deployment](#deployment)
- [API Reference](#api-reference)
- [Linter & Formatting](#linter--formatting)
- [Build Process & Development Workflow](#build--dev-workflow)
- [Dev Logs](#dev-logs)
- [Future Roadmap](#future-roadmap)

# Architecture
## Frontend
React + Vite
- Vite allows for faster dev build and peformance, starting the server faster. Vite uses native browser imports to serve code-on-demand resulting in faster server start times.
- Also scales better when app grows larger, through Hot Module Replacement, updating browser with changes almost instantly. 

**Programming Language:** Typescript
**UI:** Material UI + Tailwind CSS

## Backend
Node.js + Express

**Programming Language:** Typescript

## Database
PostgreSQL + Prisma ORM (data modelling)
- Prisma ORM is used for type safety and its powerful ability to define schemas for databases and migrate the schemas to be applied to the database

### Database Schema
#### Entity Relationship Overview
* **User (`users`):** Scoped by Clerk User ID (`userId`). One-to-Many relationship with `Recipe`.
* **Recipe (`recipes`):** Belongs to one User. One-to-Many relationship with `Ingredient`.
* **Ingredient (`ingredients`):** Belongs to one Recipe.

#### `Recipe` Table
Stores the core metadata and instructions steps for a saved recipe
```prisma
model Recipe {
  id           String       @id @default(uuid())
  userId       String
  title        String
  description  String?
  instructions String[]
  ingredients  Ingredient[]
  createdAt    DateTime     @default(now())
  updatedAt    DateTime     @updatedAt
}
```
| Field | Type | Attributes | Nullable| Description |
| :--- | :--- | :--- | :--- | :--- |
| id | String | @id.uuid() | No | Primary Key |
| userId | String | indexed | No | Foreign Key mapping to the Clerk authentication user |
| title | String | - | No | Recipe Title |
| description | String | - | Yes | Optional description of the recipe |
| instructions | String[] | - | No | Array of instruction steps where array index maps to step order |
| ingredients | Ingredient[] | relation | No | Related list of ingredient objects |
| createdAt | DateTime | @default(now()) | No | Creation timestamp |
| updatedAt | DateTime | @updatedAt | No | Automatically updated on record changes |




#### `Ingredient` Table
Stores the discrete ingredient components tied to a specific recipe.
```prisma
model Ingredient {
  id       String @id @default(uuid())
  recipeId String
  recipe   Recipe @relation(fields: [recipeId], references: [id], onDelete: Cascade)
  name     String
  amount   Float
  unit     String

}
```
| Field | Type | Attributes | Nullable| Description |
| :--- | :--- | :--- | :--- | :--- |
| id | String | @id.uuid() | No | Primary Key|
| name | String | - | No | Name of the ingredient |
| amount | Float | - | No | The amount of the ingredient |
| unit | String | - | No | The unit for the amount of the ingredient |

## Authentication
Integrated via clerk.
***Authentication Flow:**
On landing page users are able to sign in or sign up taking them to Clerk's account portal. Upon sign in users are redirected to /dashboard.
Backend requests handled by TanStack Query and Axios, with Clerk providing the JWT for authentication for requests.
TanStack Query manages the recipe states by caching in memory for UI updates upon receiving data from Axios network calls.

# Deployment
**Vercel (Frontend):**
- Decoupled frontend to make UI tweaks isolated from deploying backend, keeping pipelines faster.
**Render (Backend):**
**Neon (Serverless PostgreSQL Database):**
- Neon requires a cold start when compute scales to zero resulting in an initial delay of 500ms to 2s but this was actually more of a pro than a con to keep hosting costs to close to $0.
- Neon was also chosen because of the ability to separate live and test database environments should the need arise.
- Although Neon has a disadvantage when scaling to high read/write traffic, considering Yoripe is a personal project, this was a non-issue.
# API Reference
All requests require a valid Clerk authentication JWT in the `Authorization` header (`Bearer <token>`). Endpoints automatically scope read and write operations to the authenticated user's ID.

### Recipes (`/recipes`)

#### `GET /recipes`
Retrieves all recipes saved by the user returning only title and description of the recipe. Supports optional search filtering and field sorting.

**Query Parameters:**

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `q` | String | None | Search query string to match against recipe fields. |
| `by` | String | `all` | Target field to search/sort by (`all`, `title`, or `ingredient`). |

**Response:** `200/OK`

#### `GET /recipes/:id`
Retrieves detailed info including title, description, ingredients and instructions of a specific recipe with `id`. 

**Path Parameters:**

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | String | Yes | The unique id of the recipe|

**Response:** `200/OK`

#### `DELETE /recipes/:id`
Deletes the specified recipe

**Path Parameters:**

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | String | Yes | The unique id of the recipe |

**Response:** `200/OK`

#### `PATCH /recipes/:id`
Partially updates the specified recipe.

**Request Body (application/json):**

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `title` | String | No | The name of the recipe |
| `description` | String | No | A short description of the recipe |
| `instructions` | String[] | No | Steps of the recipe, where each step corresponds to the index of the list |
| `ingredients` |  `IngredientArray` | No | The list of ingredients

**`IngredientArray`:**

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `name` | String | Yes | The name of the ingredient |
| `amount ` | String | Yes | The amount of the ingredient |
| `unit` | String | Yes | The unit for the amount of the ingredient |

**Response:** `200/OK`

#### `POST /recipes`
Creates a new recipe under the authenticated user's account

**Request Body (application/json):**

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `title` | String | Yes | The name of the recipe |
| `description` | String | No | A short description of the recipe |
| `instructions` | String[] | Yes | Steps of the recipe, where each step corresponds to the index of the list |
| `ingredients` |  `IngredientArray` | Yes | The list of ingredients

**`IngredientArray`:**

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `name` | String | Yes | The name of the ingredient |
| `amount` | String | Yes | The amount of the ingredient |
| `unit` | String | Yes | The unit for the amount of the ingredient |

**Response:** `201/CREATED`

# Linter & Formatting
Linting and formatting aligns with **ESLint** standards and formatted using **Prettier** through **pnpm** scripts

# Build & Dev Workflow
```text
[ Local Dev Environment ]
       │
       ├─► Frontend: Vite Dev Server (pnpm dev)
       ├─► Backend: Express API + Prisma Client
       └─► Database: Local / Hosted PostgreSQL
       │
[ Production Pipeline ]
       ├─► GitHub (main branch push)
       ├─► Vercel Pipeline (Static Build + CDN Edge Deployment)
       └─► Render Pipeline (Express API Node Environment + Prisma Migration Step)
```
## Branching & Deployment Strategy

Yoripe follows a feature-branch workflow to maintain stable production deployments and isolated development environments.

| Branch | Target Environment | Automation / Triggers |
| :--- | :--- | :--- |
| `main` | Production | Auto-deploys frontend to Vercel and backend to Render on push. |
| `feature/*` | Local / Preview | Isolated feature development. Triggers Vercel preview deployments on PRs |
| `fix/*` | Local / Preview | Isolated bug fixes |
| `chore/*` | Local / Preview | Project maintenance, dependency upgrades, build configuration, tooling updates that do not alter source code behaviour |

#### Workflow Rules:
1. All active development occurs on dedicated `feature/` or `fix/` branches.
2. Direct commits to `main` are restricted to documentation and patch updates.
3. Merges into `main` trigger automated builds and database migration checks via Prisma.

# Dev Logs
- [Frontend Dev Log](frontend-dev-log)
- [Backend Dev Log](backend-dev-log)


# Future Roadmap
- Extracting recipe information using vision LLM
- Explore page to view nearby recipes and save

