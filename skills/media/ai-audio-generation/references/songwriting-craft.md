# Songwriting Craft & Suno AI Music

## Song Structure

Common skeletons (mix, modify, or invent):
- **ABABCB** — Verse/Chorus/Verse/Chorus/Bridge/Chorus (most pop/rock)
- **AABA** — Verse/Verse/Bridge/Verse (jazz standards, ballads)
- **ABAB** — Verse/Chorus alternating (simple, direct)
- **AAA** — Verse/Verse/Verse (folk, storytelling)

Building blocks: Intro, Verse, Pre-Chorus (optional), Chorus, Bridge, Outro.

## Rhyme & Meter

Rhyme types (tight to loose): perfect, family, assonance, consonance, near/slant. Mix them — all perfect rhymes sound nursery-rhyme-like; all slant sound lazy. Internal rhyme (rhyming within a line) adds richness.

Meter = rhythm of stressed vs unstressed syllables. Match syllable counts between parallel lines. Stressed syllables matter more than total count. Say it out loud — if you stumble, the meter needs work.

## Emotional Arc & Dynamics

Energy mapping (rough): Intro 2-3 → Verse 5-6 → Pre-Chorus 7 → Chorus 8-9 → Bridge varies → Final Chorus 9-10.

Key trick: **Contrast.** Whisper before a scream. Sparse before dense. Slow before fast. The drop works because of the buildup. Silence is an instrument.

## Lyrics That Work

- **Show, don't tell** (usually): "Your hoodie's still on the hook by the door" > "I was sad"
- **The hook**: line people remember — usually title or core phrase, placed where it lands hardest (first/last line of chorus)
- **Prosody**: stable feelings → settled melodies, perfect rhymes, resolved chords. Unstable feelings → wandering melodies, near-rhymes, unresolved chords
- Avoid: clichés on autopilot, Yoda-speak to force rhymes, same energy in every section

## Parody & Adaptation

Map the original's structure first: count syllables per line, mark rhyme scheme, identify stressed syllables, note held/sustained notes. Match stressed syllables to the same beats. Total syllable count can flex by 1-2 unstressed. On held notes, match the vowel sound. Keep some original lines intact for recognizability.

## Suno AI Prompt Engineering

**Style/Genre field formula**: Genre + Mood + Era + Instruments + Vocal Style + Production + Dynamics.

```
BAD:  "sad rock song"
GOOD: "Cinematic orchestral spy thriller, 1960s Cold War era, smoky
       sultry female vocalist, big band jazz, brass section,
       sweeping strings, minor key, vintage analog warmth"
```

Rules: No artist names or trademarks (describe the sound instead). Up to 1,000 chars in Style field (v4.5+). Describe the dynamic journey, not just the genre.

**Metatags** (in `[brackets]` inside lyrics field):
- Structure: `[Intro]` `[Verse]` `[Chorus]` `[Bridge]` `[Instrumental]` `[Outro]`
- Vocals: `[Whispered]` `[Belted]` `[Falsetto]` `[Harmonies]` `[Choir]`
- Dynamics: `[High Energy]` `[Building Energy]` `[Emotional Climax]` `[Quiet arrangement]`
- Atmosphere: `[Melancholic]` `[Euphoric]` `[Nostalgic]` `[Dreamy]`

Keep to 5-8 tags per section max. Don't contradict yourself. Put tags in BOTH style field and lyrics for reinforcement.

**Custom Mode**: Always use Custom Mode for serious work (separate Style + Lyrics fields). Lyrics field limit ~3,000 chars. Always add structural tags.

**Phonetic tricks**: Spell words as they SOUND ("through" → "thru"). ALL CAPS = louder. Vowel extension for sustained notes ("lo-o-o-ove"). Spell out numbers and space acronyms.

## Workflow

1. Write concept/hook first (emotional core)
2. If adapting, map original structure
3. Generate raw material freely before structuring
4. Draft lyrics into structure
5. Read/sing aloud — catch stumbles, fix meter
6. Build Suno style description (dynamic journey)
7. Add metatags for performance direction
8. Generate 3-5 variations minimum — ~3-5 generations per 1 good result
