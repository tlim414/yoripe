---
layout: default
title: "Frontend Dev Log"
permalink: /frontend-dev-log
---

# Frontend Dev Log
This page serves as a logbook for the frontend changes

## WIP
### September 4
- Update SignIn redirectURL to `/my-recipes` and not `/dashboard`

## Standardize units from image extraction response data
### August 30
Implemented unit normalization
- Created a mapping dictionary (UNIT_ALIASES) to consolidate alternate unit representations into supported frontend options
- Added a utility function to standardize ingredient units returned by the extraction endpoint before they reach the UI, mitigating edge cases where extracted recipes use non-standard or unsupported unit variants.
```text
export const UNIT_ALIASES: Record<string, UnitType> = {
  // Cups
  cup: UNITS.CUPS,
  cups: UNITS.CUPS,
  ...
```

## Add Image Uploading to Backend
### August 28
- Integrated file uploader: Implemented `react-dropzone` to allow image selection via file browser or drag-and-drop.
- Image preview & explicit trigger: Displayed an immediate preview of the selected image alongside its file name. Opted for an explicit "Extract" button rather than auto-submitting on upload, avoiding unnecessary API usage and traffic if a user selects the wrong file.
- Flexible fraction parsing for amounts: Updated the amount input field to accept string representations instead of strict numeric types:
- Updated to regex validation on input change to support integers, decimals, mixed fractions, and slash-separated fractions (e.g., 3, 2.75, 1 1/3, 3/5).
- Updated the "Save" button's disabled state validation to block submission when amount formats are invalid.

## MVP
### July 21 - August 14
Routing Infrastructure:
-	Configured client-side routing using `react-router-dom` with strict Clerk auth protection wrappers:
-	`/`: Public landing page showcasing app branding and Clerk authentication options.
-	`/my-recipes`: Protected route serving the authenticated user's saved recipe vault.
-	`/explore`: Protected route placeholder for discovering community-shared recipes.
-	`/settings`: Protected route for managing user application preferences (e.g., appearance theme).
-	Added fallback routing (*) to direct authenticated users to `/my-recipes` and unauthenticated visitors to `/`.

Authentication Integration:
-	Integrated `@clerk/react` using `ClerkProvider` for app-wide user session management.
-	Implemented `SignInButton` and `SignUpButton` components operating in modal mode, configured with post-auth redirect fallbacks to `/dashboard`.

Landing Page:
-	Created a high-level hero landing page displaying dynamic adaptive assets based on active color scheme modes.
-	Rendered core product messaging alongside secondary product context.

Recipe Card Component:
-	Designed a modular `RecipeCard` component using MUI Card and Typography.
-	Accepts a `RecipeSummary` prop to render title, description, and formatted creation date (en-ZA locale format).
-	Exposes an `onClick` callback prop to trigger detail modal views upon card selection.

Recipe Detail Modal :
-	Created `RecipeDetails` to display full recipe payloads, mapping structured ingredient lists (bullet points, name, amount, unit) alongside numbered instruction steps.
-	Wrapped inside `RecipeDetailsManager` using custom `AppDialog` modal primitives:
-	Handles dynamic data loading states (`useRecipe` query hook) with spinner and error boundaries.
-	Implemented an anchor action menu offering quick access to edit and delete actions (`useDeleteRecipe` mutation).

Recipe Form & Creation/Edit Workflows:
-	Built a versatile multi-purpose form handling both CREATE and EDIT modes based on a formMode prop.
-	Implemented multi-step dynamic state handlers for adding/removing ingredients and cooking steps dynamically.

Navigation & Sidebar Drawer:
-	Implemented an main application layout wrapping protected routes with an app bar (NavBar) and a slide-out navigation Drawer.
-	Built navigation list items dynamically mapping routes (`/my-recipes`, `/explore`, `/settings`) with active state highlighted based on `location.pathname`.

Theme & Appearance Engine:
-	Built custom theme configurations via MUI `createTheme` using the `colorSchemes` feature for seamless dark/light switching.
-	Configured setting page controls allowing users to switch dynamically between system, light, and dark modes using `useColorScheme`.
-	Augmented MUI TypeScript module declarations (TypeBackground) to support custom search bar background states:
  ```text
  declare module '@mui/material/styles' {
    interface TypeBackground {
      search?: string;
      searchHover?: string;
    }
  }
  ...
  background {
    ...
    search: '#f5f5f5',
    searchHover: '#ebebeb',
  }
  ```

