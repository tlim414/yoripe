# Yoripe
Yoripe is a full-stack, decoupled recipe vault application designed for high performance and ease of use during cooking. The system uses a modern TypeScript stack to maintain strict end-to-end type safety, separating front-end interaction logic from backend database management.
The following page is the docs for the web app and serves as a developer build log on design choices.

# Table of Contents
- [Architecture & Tech Stack](#architecture)
- [Deployment](#deployment)
- [API Reference](#api-reference)
- [Build Process & Development Workflow](#build-&-dev-workflow)
- [Dev Logs](#dev-logs)
- [Future Roadmap](#future-roadmap)

# Architecture
## Frontend
React + Vite
**Programming Language:** Typescript
**UI:** Material UI + Tailwind CSS

## Backend
Node.js + Express
**Programming Language:** Typescript

## Database
PostgreSQL + Prisma ORM (data modelling)

### Data Models
**Recipes:**
**Ingredients:**

## Authentication
Integrated via clerk.
***Authentication Flow:**
On landing page users are able to sign in or sign up taking them to Clerk's account portal. Upon sign in users are redirected to /dashboard.
Backend requests handled by TanStack Query and Axios, with Clerk providing the JWT for authentication for requests.
TanStack Query manages the recipe states by caching in memory for UI updates upon receiving data from Axios network calls.

# Deployment
**Vercel (Frontend):**
**Render (Backend):** 
**Neon (Serverless PostgreSQL Database):**
- Neon requires a cold start when compute scales to zero resulting in an intial delay of 500ms to 2s but this was actually more of a pro than a con to keep hosting costs to close to $0.
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
| `description ` | String | No | A short description of the recipe |
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

# Build & Dev Workflow

# Dev Logs
- [Frontend Dev Log](frontend-dev-log)
- [Backend Dev Log](backend-dev-log)


# Future Roadmap
- Extracting recipe information using vision LLM
- Explore page to view nearby recipes and save

