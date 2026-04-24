#  Identikit

**Synthetic identity attacks on music recommender systems**

---

## Overview

Music recommender systems infer a user’s musical *identity* from historical listening behavior and use this inferred identity to personalize recommendations. **Identikit** is a (for now, casual) attempt how robust that inferred identity really is.

> **Can a legitimate user, using only valid listening behavior, deliberately manipulate how a recommender system perceives their taste?**

Rather than attacking system infrastructure, this project targets the **preference inference mechanism itself**, treating the user as a potential adversary.
My recommender is much simpler than Spotify’s, by design. I wanted to isolate the core mechanism of identity inference and understand its behavior without confounding factors. The goal wasn’t realism, it was interpretability.

This ongoing project is still in the extremely newborn stage. Two simple attacks on a simple recommender are implemented. 

---

## Core Idea

Recommender systems implicitly assume:

> *Observed behavior ≈ genuine preference*

Identikit challenges this assumption by demonstrating that **algorithmic identity can be constructed, shifted, and manipulated** through carefully chosen listening patterns, without violating any rules. The result is a phenomenon we refer to as **synthetic taste**: an identity that looks authentic to the system, regardless of genuine preference.

---

## Dataset

This project uses the **HetRec 2011 Last.fm dataset**, which contains:

- ~1.9k users  
- ~17k artists  
- User–artist listening counts (implicit feedback)  
- Artist metadata and tags  

Each user’s listening history is treated as **implicit preference data**, closely mirroring real-world music streaming platforms.

I used this particular dataset because it summarizes real users with repeated behavior, both strong and weak identity profiles. It is reproducible and ToS-safe and widely used in recommender systems research. (Also lfm-1b is not available to download right now due to licensing so this is some good old cope)

---

## Methodology

### 1. User Identity Representation

A user’s musical identity is modeled as a **normalized artist preference vector**, derived from listening counts: identity[user][artist] = playcount / total_playcount

This vector represents the system’s belief about the user’s taste.

---

### 2. Baseline Recommender

We implement a **user–user collaborative filtering recommender**:

- Users are represented by identity vectors  
- Cosine similarity is used to find similar users  
- Recommendations are aggregated from similar users’ preferences  

The recommender is intentionally simple and fixed to focus on **robustness**, not recommendation quality.

---

### 3. Adversarial User Behavior

We simulate an adversarial user who:

- Is a legitimate user  
- Performs valid listening actions  
- Intentionally amplifies listening to a target artist  

#### Attack Strategy: Playcount Injection

Listening events are injected for a target artist to study how inferred identity shifts.

Two variants are evaluated:

- **Absolute injection** (e.g. +20 listens)
- **Relative injection** (e.g. +20% of total listening history)

This allows us to measure how **effort scales with identity strength**.

---

#### Attack Strategy: Boundary Artist Injection

We targets artists that are:
- frequently listened to by users *similar* to the victim
- completely absent from the victim’s listening history

Rather than amplifying a single axis of preference, this attack injects a small number of listens across multiple **semantically adjacent but previously unobserved dimensions**.

Despite injecting only **250 total listens** into a user with an extremely large listening history, we observe:

- **Near-maximal identity drift (≈ 1.0)**
- **Severe recommendation turnover (overlap = 2 / 10)**

Notably, the injected listening volume is **negligible relative to the user’s total playcount**, yet it causes a near-complete reorientation of the inferred identity.

This indicates that collaborative recommender systems are **far more sensitive to directional changes in preference space than to absolute listening volume**. Introducing new, semantically aligned dimensions can rotate a user’s identity vector enough to alter similarity relationships and downstream recommendations.

In effect, this attack demonstrates that **identity can be redefined not by how much a user listens, but by *where* they listen within the preference space**.


### 4. Evaluation Metrics

We quantify impact using two metrics:

#### Identity Drift
Cosine distance between identity vectors before and after the attack:

drift = 1 − cosine_similarity(identity_before, identity_after)


Higher drift indicates greater identity change.

#### Recommendation Overlap
The number of shared items between the top-K recommendations before and after the attack.

Lower overlap indicates a stronger behavioral impact on recommendations.

---

## Key Findings

### • Small absolute manipulations are ineffective
Injecting a small number of listens into users with strong identities produces **negligible identity drift** and **no recommendation changes**.

### • Relative manipulation causes significant identity shift
When injection is scaled relative to a user’s historical listening volume, we observe:

- Substantial identity drift (≈ 0.3 for strong users)
- Partial but meaningful recommendation changes

This demonstrates that **inferred identity is manipulable when adversarial effort is proportional to existing signal strength**.


### • Identity changes faster than recommendations
In multiple experiments, identity vectors shifted significantly while recommendations remained largely stable.

This suggests:
- Preference inference updates continuously
- Recommendation outputs exhibit inertia

This behavior mirrors real-world recommender systems.

---

## Main Insight

- Weak or sparse user histories are highly vulnerable
- Strong identities require more effort but remain manipulable
- Algorithmic identity is continuous, not categorical


---

## Limitations

The recommender is a surrogate model, not a production system. My idea is to expand this to the real recommender system that music streaming uses (unfortunately, Spotify has paused integrations, so my original idea was scrapped for now). Listening counts approximate real-time behavior, and social and contextual signals are not modeled.


---

## Future Work
I'm going to work on tag-based and hybrid identity representations, and use the currently defunct Spotify API services to extend this project. Until that happens, I'll export my own listening data from Spotify and do a case study on turning into a metalhead under the recommender system in about two days. Temporal persistence and decay of synthetic identity , and defensive mechanisms (smoothing, regularization, anomaly detection) are other stuff I'd like to focus on.


---



Identity is performative.
