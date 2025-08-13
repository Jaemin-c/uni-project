# XChange — Skill-Exchange & Mentoring Matchmaking Web App

## Overview
**XChange** connects people based on what they can **offer** and what they want to **learn**.  
Users register their Offered and Wanted skills, explore recommendations, browse by category, search, view profiles & reviews, and start a **1:1 chat** to coordinate lessons or mentoring.

---

## Features
- **Auth & Onboarding**
  - Sign up with profile, Offered/Wanted skills
  - Per-skill description & category (e.g., Technology, Languages)
- **Discover (Recommendations)**
  - Prioritizes users whose **Offered** skills intersect with **your Wanted** skills
  - Category & minimum rating filters
- **Explore & Search**
  - Browse users by category
  - Keyword search across categories and bios
- **Profiles & Reviews**
  - Rich user profiles with skill descriptions
  - Star ratings & written reviews; average rating displayed
- **Messaging**
  - 1:1 chat screen and inbox with recent conversations

---

## User Flow
1. **Sign Up** → add profile, Offered/Wanted skills → add per-skill description/category  
2. **Discover** → recommendations + filters; **Explore/Search** for more options  
3. **Connect** → open profile → start **chat** to plan details  
4. **Review** → leave rating/comment after the exchange (feeds into future trust & filters)

---

## Matching Logic
- **Priority:** users whose **Offered** skills intersect with **your Wanted** skills are listed first.
- **Filters:** selected category match (string-contains) and **min rating** (integer threshold).

---

## Data Model
- **User**
  - `email`, `name`, `password (BCrypt hash)`, `bio`, `profile_picture`, `preferred_categories`
  - relationships: `reviews_received`, `reviews_given`, `user_skills`
  - helpers: `offered_skill_names`, `wanted_skill_names`, `get_average_rating()`, `get_review_count()`
- **Skill**
  - Catalog entity shared by all users: `name`, `category`
- **UserSkill**
  - Resolves many-to-many: `user_id`, `skill_id`, `description`, `type ("offered" | "wanted")`
  - Unique constraint: `(user_id, skill_id, type)` to prevent duplicates
- **Review**
  - `rating`, `comment`, `skill_offered`, `reviewer_id`, `reviewed_user_id`
- **Message**
  - `sender_id`, `receiver_id`, `content`, `timestamp (UTC)`

---

## Tech Stack & Serving
- **Backend:** Flask (Blueprints), Flask-Login, Flask-Bcrypt, SQLAlchemy ORM
- **Templates & Static:** Jinja templates; static assets served via Flask routes:
  - `/assets/<path>`, `/styles/<path>`, `/scripts/<path>`
- **Serving =** returning content to the browser
  - **Template serving:** `render_template(...)` → server-rendered HTML
  - **Static serving:** CSS/JS/images returned as files

---

## Routes 
> Prefixes set in `server.py`. Function names from each module (selected list).

### Auth (`/auth`)
- `GET /auth/login` → login page (`auth.login_page`)
- `POST /auth/login` → login submit (`auth.login`)
- `GET /auth/signup_page` → sign-up form (`auth.signup_page`)
- `POST /auth/signup_skills` → store sign-up data in session (`auth.signup_skills`)
- `GET /auth/signup_descriptions_page` → per-skill details (`auth.signup_descriptions_page`)
- `POST /auth/signup_finalize` → create `User` + `UserSkill` (`auth.signup_finalize`)
- `POST /auth/logout` → logout (`auth.logout`)

### Profile (`/user`)
- `GET /user/profile` → my profile (`profile.my_profile`)
- `GET /user/profile/<id>` → view others (`profile.view_profile`)
- `PUT /user/profile` → update profile & skills (`profile.update_profile`)
- `DELETE /user/profile` → delete account (`profile.delete_profile`)
- `POST /user/profile/picture` → upload profile picture (`profile.update_profile_picture`)

### Discover / Explore / Search
- `GET /discover` → recommendations (`discover.discover`)
- `GET /explore` → category index (`explore.explore_skills`)
- `GET /explore/<category>` → users in category (`explore.show_category`)
- `GET /search` → keyword search (`search.search`)

### Messaging (no prefix)
- `POST /messages` → send message (`messages.send_message`)
- `GET /messages?user_id=<id>` → fetch 1:1 thread (`messages.get_messages`)
- `GET /chat/<id>` → chat UI (`messages.chat`)
- `GET /inbox` → recent conversations (`messages.inbox`)

### Reviews (`/reviews`)
- `GET /reviews/api/reviews/<user_id>` → list + avg rating (`reviews.get_user_reviews`)
- `POST /reviews/api/reviews` → create review (`reviews.create_review`)
- `DELETE /reviews/api/reviews/<review_id>` → delete own review (`reviews.delete_review`)
- `GET /reviews/user/<user_id>/reviews` → HTML page (`reviews.view_reviews`)

---

## Local Development (Quick Start)
General Python Set-Up
1. Install Python3.X

2. Run the following pip install -r requirements.txt

Login
1. Run the following 'python3 backend/database.py'

2. Run the following 'python3 backend/app.py'

3. A port should open up. Click to view the program in a local browser.

4. Click 'Create Account' and enter the necessary information.

5. Login with credentials used in signup.