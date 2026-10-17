# Batti-Wala Local Setup Guide

Batti-Wala is a browser-based 2D maze game built with p5.js. It is a
dependency-free JavaScript project and does not currently use npm, Vite,
Webpack, Python packages, or a build step.

You must run it through a local HTTP server because the game uses JavaScript
modules (`import`/`export`) and loads assets from relative paths.

## 1. Requirements

Install or have access to:

- Git
- A modern browser such as Chrome, Edge, or Firefox
- One local HTTP server option:
  - Node.js 18 or newer (recommended)
  - Python 3
  - VS Code with the Live Server extension

Node.js is only needed to run the local server. There is currently no
`package.json`, so do not run `npm install`.

## 2. Open the cloned project

Open PowerShell and change to the repository directory:

```powershell
cd E:\projects\cloned-projects\torchbound
```

The project root should contain:

```text
index.html
main.js
p5.js
assets\
common\
entities\
scenes\
```

The browser entry point is `index.html`.

## 3. Configure Supabase

The current implementation uses Supabase for:

- Player profiles
- Saved level progress
- Personal best scores
- Leaderboards

The configuration is in `common/supabase.js`:

```javascript
const SUPABASE_URL = 'https://uuugmjnljhxruypdzozj.supabase.co';
const SUPABASE_ANON_KEY = '******';
```

The key in the cloned repository is a placeholder. You must replace it with a
valid Supabase public/anonymous key before account and leaderboard features can
work.

### Option A: Use the existing Supabase project

Ask the project owner for the public/anonymous key, then edit
`common/supabase.js`:

```javascript
const SUPABASE_URL = 'https://uuugmjnljhxruypdzozj.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-or-publishable-key';
```

Keep the existing URL unless the project owner provides another URL.

Never put a Supabase `service_role` key in this browser application. Only use a
public/anonymous or publishable key.

### Option B: Create your own Supabase project

If you do not have access to the existing Supabase project:

1. Create a project at [supabase.com](https://supabase.com).
2. Open **Project Settings > API**.
3. Copy the project URL and the anon/public or publishable key.
4. Put both values in `common/supabase.js`.

For example:

```javascript
const SUPABASE_URL = 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-or-publishable-key';
```

Run the following SQL in the Supabase **SQL Editor**:

```sql
create table public.profiles (
    player_id text primary key,
    username text not null,
    unlocked_level integer not null default 1,
    high_scores jsonb not null default
        '{"1": 0, "2": 0, "3": 0, "4": 0, "5": 0}'::jsonb,
    tutorial_done boolean not null default false,
    created_at timestamptz not null default now()
);

create table public.scores (
    player_id text not null references public.profiles(player_id)
        on delete cascade,
    display_name text not null,
    level integer not null check (level between 1 and 5),
    score integer not null check (score >= 0),
    created_at timestamptz not null default now(),
    primary key (player_id, level)
);
```

Enable Row Level Security:

```sql
alter table public.profiles enable row level security;
alter table public.scores enable row level security;
```

For local development, create policies that allow the browser client to read
and write the required records:

```sql
create policy "Allow public profile reads"
on public.profiles
for select
to anon
using (true);

create policy "Allow public profile creation"
on public.profiles
for insert
to anon
with check (true);

create policy "Allow public profile updates"
on public.profiles
for update
to anon
using (true)
with check (true);

create policy "Allow public score reads"
on public.scores
for select
to anon
using (true);

create policy "Allow public score creation"
on public.scores
for insert
to anon
with check (true);

create policy "Allow public score updates"
on public.scores
for update
to anon
using (true)
with check (true);
```

These policies are convenient for local testing, but they are not strong
production authorization because the game does not currently use authenticated
user accounts. For production, protect profile and score writes with Supabase
Auth or a trusted backend.

## 4. Start the local server

### Recommended: Node.js

From the project root, run:

```powershell
cd E:\projects\cloned-projects\torchbound
npx --yes http-server . -p 8080
```

Open this address in your browser:

```text
http://localhost:8080
```

The server may also display:

```text
http://127.0.0.1:8080
```

Stop the server with `Ctrl+C`.

### Alternative: Python

If Python 3 is installed:

```powershell
cd E:\projects\cloned-projects\torchbound
python -m http.server 8080
```

If `python` is not available on Windows, try:

```powershell
py -m http.server 8080
```

Then open:

```text
http://localhost:8080
```

### Alternative: VS Code Live Server

1. Install the **Live Server** extension in VS Code.
2. Open `E:\projects\cloned-projects\torchbound` as the workspace folder.
3. Right-click `index.html`.
4. Select **Open with Live Server**.
5. Open the URL provided by VS Code, usually:
   `http://127.0.0.1:5500/index.html`.

## 5. Do not open the HTML file directly

Do not double-click `index.html` to open it as a `file:///` URL:

```text
file:///E:/projects/cloned-projects/torchbound/index.html
```

Direct file loading can cause JavaScript module, CORS, and asset-loading
errors. Always use a local HTTP URL such as:

```text
http://localhost:8080
```

## 6. Verify the game

After opening the game:

1. Confirm that the Batti-Wala main menu is visible.
2. Click the page once to allow browser audio.
3. Open the tutorial or start a level.
4. Move with `W`, `A`, `S`, `D`, or the arrow keys.
5. Open the account screen and create a player profile.
6. Complete a level and check the leaderboard.

If you configured Supabase, verify that:

- A profile appears in the `profiles` table.
- Progress is saved after changing levels.
- A score appears in the `scores` table after completing a level.

## 7. Optional: use the bundled local p5.js file

The repository contains a local p5.js copy at `p5.js`.

By default, `index.html` loads p5.js from the CDN:

```html
<script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/p5.min.js"></script>
```

To use the bundled copy instead, edit `index.html`:

```html
<!-- <script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/p5.min.js"></script> -->
<script src="p5.js"></script>
```

Supabase is still loaded from its CDN unless you separately provide a local
Supabase browser bundle.

## 8. CDN and internet requirements

The default `index.html` loads:

- p5.js from jsDelivr
- Supabase JavaScript from jsDelivr

Therefore, the browser needs internet access for the default configuration.
The game assets themselves are included in the repository under `assets\`.

If the browser is offline:

- Use the bundled `p5.js` file as described above.
- Provide a local copy of the Supabase browser bundle, or remove/replace
  Supabase-dependent functionality.

## 9. Troubleshooting

### `Failed to load module script`

You probably opened `index.html` directly. Start a local HTTP server and open:

```text
http://localhost:8080
```

### `window.supabase is undefined`

The Supabase CDN script did not load. Check:

- Internet connectivity
- The browser developer console
- That the Supabase script appears before `main.js` in `index.html`

### `Invalid API key`, `401`, or Supabase errors

Check `common/supabase.js`:

- The key must not still be `******`.
- The key must belong to the configured Supabase project.
- Use the anon/public or publishable key, not the `service_role` key.

### `relation "profiles" does not exist`

The Supabase database tables have not been created. Run the schema from the
Supabase section of this guide.

### Account creation does not save

Open the browser developer console and look for:

```text
[profile] createProfile error:
```

Common causes are:

- Invalid Supabase URL or key
- Missing `profiles` table
- Missing Row Level Security policies
- Column names that do not match the schema

### The leaderboard is empty

Check:

- The `scores` table exists.
- The `select`, `insert`, and `update` policies exist.
- The Supabase URL and key are correct.
- The browser console for `[leaderboard]` errors.

### No audio plays

Browsers block autoplay until the user interacts with the page. Click the
game once, then start the tutorial or a level.

### Assets are missing

Make sure the server was started from the repository root:

```powershell
cd E:\projects\cloned-projects\torchbound
```

Starting the server from another directory can make paths such as
`assets/maps/map-light.png` resolve incorrectly.

## 10. Quick-start checklist

For a normal local run:

```powershell
cd E:\projects\cloned-projects\torchbound
npx --yes http-server . -p 8080
```

Then open:

```text
http://localhost:8080
```

If accounts or leaderboards are needed, first replace the placeholder key in
`common/supabase.js` and make sure the Supabase tables and policies have been
created.

## 11. Project structure

```text
torchbound\
├── assets\       Images, audio, fonts, and game artwork
├── common\       Shared constants, levels, profiles, audio, and leaderboard code
├── entities\     Player, map, camera, traps, power-ups, and background code
├── scenes\       Menu, tutorial, and in-game rendering code
├── index.html    Browser entry point
├── main.js       p5.js game orchestrator
├── p5.js         Bundled p5.js library
├── README.md     Project overview
└── starrup.md    Local setup guide
```

## 12. Gameplay controls

- `W`, `A`, `S`, `D`: Move
- Arrow keys: Move
- Mouse: Use menu buttons
- Browser interaction/click: Enable audio playback
