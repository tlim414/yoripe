# Yoripe
Yoripe is a full-stack, decoupled recipe vault application designed for high performance and ease of use during cooking. The system uses a modern TypeScript stack to maintain strict end-to-end type safety, separating front-end interaction logic from backend database management.
The following page is the docs for the web app and serves as a developer build log on design choices.

# Frontend
- [Dev log]
React + Vite and deployed on Vercel. (Vercel was chosen based on cost limitations for the project)
## UI
The UI was built using Material UI, chosen for pre-styled components to allow me to focus on the core app features.

# Backend
- [Dev log]
Node.js and Express hosted on Render to keep the core API independent of UI deployments. (Render was chosen based on cost limitations for the project)

## Api Reference
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

## Prisma ORM
Prisma ORM is used for type safety and its powerful ability to define schemas for databases and migrate the schemas to be applied to the database

## API

# Auth
Authentication is handled via Clerk. On landing page users are able to sign in or sign up taking them to Clerk's account portal. Upon sign in users are redirected to /dashboard.
Backend requests handled by TanStack Query and Axios, with Clerk providing the JWT for authentication for requests.
TanStack Query manages the recipe states by caching in memory for UI updates upon receiving data from Axios network calls.

# Dataflow


# Deployment
Frontend and backend are deployed on different hosting services to not exceed free tier limits.
##

# Future Roadmap
- Extracting recipe information using vision LLM
- Explore page to view nearby recipes and save



# Database
Database is a serverless PostgreSQL hosted by Neon. (Neon was chosen based on cost limitations for the project)
- Neon requires a cold start when compute scales to zero resulting in an intial delay of 500ms to 2s but this was actually more of a pro than a con to keep hosting costs to close to $0.
- Neon was also chosen because of the ability to separate live and test database environments should the need arise.
- Although Neon has a disadvantage when scaling to high read/write traffic, considering Yoripe is a personal project, this was a non-issue.
