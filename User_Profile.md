# User Profiles

Here are the taste profiles I'm using to test the recommendation system. The idea is each one should pull a clearly different set of songs to the top — if two profiles are returning the same results then either the weights are off or the profiles aren't distinct enough.

---

## Profile 1 — The Late Night Studier

```python
user_studier = {
    "favorite_genre": "lofi",
    "favorite_mood": "focused",
    "target_energy": 0.38,
    "likes_acoustic": True,
    "target_valence": 0.58,
    "target_acousticness": 0.80
}
```

This one should surface Library Rain, Focus Flow, and Midnight Coding at the top. If Storm Runner (rock, intense, energy=0.91) is showing up anywhere near the top for this profile then something is wrong with the scoring weights.

---

## Profile 2 — The Workout User

```python
user_workout = {
    "favorite_genre": "pop",
    "favorite_mood": "intense",
    "target_energy": 0.90,
    "likes_acoustic": False,
    "target_valence": 0.75,
    "target_acousticness": 0.05
}
```

This one should pull Gym Hero and Storm Runner to the top. The `likes_acoustic: False` and low `target_acousticness` is what really separates this profile from the studier — both are pointing in opposite directions on the acoustic preference, which should matter a lot to the scoring.

---

## Profile 3 — The Sunday Morning Listener

```python
user_sunday = {
    "favorite_genre": "jazz",
    "favorite_mood": "relaxed",
    "target_energy": 0.40,
    "likes_acoustic": True,
    "target_valence": 0.68,
    "target_acousticness": 0.85
}
```

This is the hard one and the real differentiation test. Coffee Shop Stories (jazz, relaxed, energy=0.37) and Library Rain (lofi, chill, energy=0.35) are really close numerically so the system has to use `favorite_genre` and `favorite_mood` to push Coffee Shop Stories ahead. If the weights are right that's what should happen — if not, those two songs will basically tie and the system isn't actually differentiating intent.
