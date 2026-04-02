# Music Recommender System — Research Context

## How Major Streaming Platforms Predict What You'll Love

### The Two Core Approaches

#### 1. Collaborative Filtering — "Users Like You Also Liked..."
This approach ignores song attributes entirely. It predicts preference by finding **patterns across users**.

- **How it works:** If User A and User B both liked songs X, Y, Z — and User A also liked song W — the system recommends W to User B.
- **Used by:** Spotify's "Discover Weekly", YouTube's homepage feed
- **Data it needs:** play history, likes/dislikes, skips, playlist adds, replays, listen duration
- **Strength:** Discovers unexpected but genuinely loved songs ("if many jazz fans also love this ambient track, show it to jazz fans")
- **Weakness:** Cold start problem — new users with no history get poor recommendations; new songs with no plays never get surfaced

#### 2. Content-Based Filtering — "Because You Liked Songs With These Attributes..."
This approach ignores other users entirely. It matches **song features to user taste features**.

- **How it works:** If you repeatedly play high-energy pop songs, the system scores other high-energy pop songs higher for you.
- **Used by:** Spotify's audio analysis engine (acquired The Echo Nest in 2014), Pandora's Music Genome Project
- **Data it needs:** tempo, energy, genre, mood, danceability, acousticness, valence
- **Strength:** Works immediately for new users; explainable ("recommended because: high energy, matches your profile")
- **Weakness:** Creates a "filter bubble" — keeps recommending similar things, rarely surprises the user

#### 3. Hybrid Filtering (What Spotify Actually Uses)
Real platforms blend both. Spotify's algorithm has three layers:
1. **Collaborative signals** — what users with similar histories listen to
2. **Content signals** — audio features like tempo, key, loudness, valence
3. **NLP signals** — text from blog posts, reviews, and playlist names scraped from the web

---

### The Main Data Types Involved

| Signal Type | Examples | Used For |
|---|---|---|
| **Explicit feedback** | likes, dislikes, star ratings | Direct preference signal |
| **Implicit feedback** | skips, replays, listen % completed, playlist adds | Stronger signal than explicit in practice |
| **Audio features** | tempo_bpm, energy, danceability, acousticness, valence | Content-based matching |
| **Categorical metadata** | genre, mood, artist | Hard filters and genre affinity |
| **Contextual signals** | time of day, device, location | Situational recommendations (e.g., workout mode) |
| **Social graph** | who your friends follow, shared playlists | Collaborative signal variant |

---

### How This Maps to the Project

The `recommender.py` module implements **content-based filtering**. The `UserProfile` stores taste preferences (`favorite_genre`, `target_energy`, etc.) and `Song` objects carry audio features — this is the content-based model.

A typical content-based score formula:

```python
score = 0
if song.genre == user.favorite_genre:  score += 2.0
if song.mood == user.favorite_mood:    score += 1.5
score += 1.0 - abs(song.energy - user.target_energy)
if user.likes_acoustic and song.acousticness > 0.6:  score += 1.0
```

**What the simulation cannot do (and why it matters):**
- No collaborative filtering — no user behavior data (plays, skips)
- No cold-start strategy — user profile is hardcoded, not learned
- No feedback loop — scores never update based on what the user skips or replays

### Key Takeaway for a Startup Context

Spotify's edge isn't just a better algorithm — it's **scale of implicit feedback data**. A billion skips and replays train collaborative models that no content-based system can match. For a new platform, content-based + explicit taste onboarding (ask users what they like upfront) is the realistic starting point.

---

---

## Feature Analysis of `data/songs.csv`

### The 8 Available Features

| Feature | Type | Range in Dataset | What It Captures |
|---|---|---|---|
| `genre` | categorical | pop, lofi, rock, ambient, jazz, synthwave, indie pop | Broad style family |
| `mood` | categorical | happy, chill, intense, relaxed, focused, moody | Emotional intent |
| `energy` | float 0–1 | 0.28 – 0.93 | Physical intensity / loudness |
| `tempo_bpm` | float | 60 – 152 | Speed / rhythmic drive |
| `valence` | float 0–1 | 0.48 – 0.84 | Positivity / emotional brightness |
| `danceability` | float 0–1 | 0.41 – 0.88 | Rhythmic groove / beat strength |
| `acousticness` | float 0–1 | 0.05 – 0.92 | Organic vs. electronic texture |
| `artist` | categorical | 7 unique artists | Artist affinity (loyalty signal) |

---

## Feature Effectiveness Ranking for Content-Based Filtering

### Tier 1 — High Signal (primary weights)

**`energy` (0–1 float)**
The single most powerful feature. It separates the dataset cleanly:
- Low energy cluster: songs 4, 6, 9 (0.28–0.42) → ambient/lofi/study
- High energy cluster: songs 3, 5 (0.91–0.93) → workout/rock

The distance `abs(song.energy - user.target_energy)` gives a direct, calculable score contribution.

**`mood` (categorical)**
Mood captures *intent* more than any numeric feature. "Chill" and "intense" are opposites in user context — a user studying wants chill, not moody or intense, even if energy numbers overlap. Worth a high weight as a hard filter.

**`genre` (categorical)**
Genre is the first thing users self-identify with ("I'm a jazz person"). In a 10-song catalog, genre separates songs into 7 distinct buckets, making it a reliable anchor. Worth the highest categorical weight.

---

### Tier 2 — Strong Supporting Signal (secondary weights)

**`acousticness` (0–1 float)**
Has the widest spread in the data (0.05 to 0.92) — meaning it discriminates well. Maps to a real listener preference: "I like raw, organic sound" vs. "I prefer electronic/produced sound."

- Songs 6 and 7 cluster together at 0.89–0.92 (organic)
- Songs 3 and 5 cluster at 0.05–0.10 (electronic)

**`valence` (0–1 float)**
Valence (emotional positivity) is distinct from energy:
- Storm Runner: high energy (0.91) but low valence (0.48) → intense but dark
- Gym Hero: high energy (0.93) AND high valence (0.77) → intense and triumphant

In this 10-song set the range is narrower (0.48–0.84), so it adds nuance rather than strong discrimination.

---

### Tier 3 — Weaker in This Dataset (use carefully or skip)

**`tempo_bpm` (float)**
Correlated with energy in this data (fast songs are high energy) — adds noise more than signal. BPM 118 vs. 124 between two upbeat pop songs is not a meaningful difference to a listener. Matters at extremes (60 BPM vs. 152 BPM), but mostly duplicates the energy signal in a small catalog.

**`danceability` (0–1 float)**
In this dataset, danceability and energy are highly correlated:

```
Song 5  energy=0.93  danceability=0.88  ✓ correlated
Song 6  energy=0.28  danceability=0.41  ✓ correlated
```

Adding both to a score without rebalancing causes **double-counting** of the same underlying preference.

**`artist` (categorical)**
Artist loyalty is a real signal, but in a 10-song catalog with 7 artists it is too sparse to matter. Worth adding once the catalog grows.

---

## Do These Features Capture Musical "Vibe"?

### What They Get Right

The combination of `energy + mood + acousticness` maps well to how people describe listening context:

- *"I need something to study to"* → energy < 0.5, mood = chill/focused, acousticness high
- *"I need a workout track"* → energy > 0.85, mood = intense
- *"Sunday morning coffee"* → genre = jazz, mood = relaxed, acousticness > 0.8

Library Rain (lofi, chill, energy=0.35, acousticness=0.86) and Coffee Shop Stories (jazz, relaxed, energy=0.37, acousticness=0.89) correctly cluster together as "quiet morning" songs — a real vibe captured accurately.

### What These Features Miss

- **Lyrics and language** — a slow sad song and a slow love song have the same audio features but completely different emotional meanings
- **Cultural context** — genre labels are Western-centric; "jazz" means very different things across regions
- **Listener state** — the same person wants different vibes at 8am vs. 11pm; none of these features capture time/context
- **Novelty vs. familiarity** — users sometimes want something they know; sometimes they want discovery. No feature here encodes that
- **Harmonic feel** — minor vs. major key creates a dark vs. bright feeling that valence approximates but doesn't fully capture. A song can be high valence but in a minor key and still feel melancholic

### The Honest Gap

`valence` is the most contested feature. Spotify defines it as "musical positiveness" derived from audio signal processing — but human emotion doesn't work that cleanly. Night Drive Loop (synthwave, moody, valence=0.49) might feel nostalgic and comforting to one listener and lonely to another. The number 0.49 collapses that entirely.

---

## Recommended Feature Weighting for `recommender.py`

```python
# Suggested weights for the recommend() method
if song.genre == user.favorite_genre:                                           score += 2.0   # Tier 1
if song.mood == user.favorite_mood:                                             score += 1.5   # Tier 1
score += (1.0 - abs(song.energy - user.target_energy)) * 1.5                               # Tier 1
score += (1.0 - abs(song.acousticness - (1.0 if user.likes_acoustic else 0.0))) * 1.0      # Tier 2
score += (1.0 - abs(song.valence - 0.7)) * 0.5                                             # Tier 2, lower weight
# Skip tempo_bpm  — redundant with energy in this catalog
# Skip danceability — correlated with energy, causes double-counting
```

This gives genre and mood the heaviest influence (categorical match = clear user intent), energy as the strongest continuous signal, and acousticness + valence as refinement layers.

# Algorithm Recipe draft

Okay Here is my Algorithm Recipe so far, I want to use a hybrid filtering similar to Spotify's. I want collaborative signals, NLP signals, and I want Content Signals (more of an emphasis on this signal right now since this is a start up but later on I want a more balanced stack). I would like to be able to have collaborative filtering, a cold start strategy and a feedback loop if possible, but understand this might be a later implementation thing. I will for now be realistic and go for a content-based + explicit taste onboarding (ask users what they like upfront) for now, but keeping in mind a "scale of implicit feedback data" for later. The primary weights will be the normal Energy, mood and genere. 
