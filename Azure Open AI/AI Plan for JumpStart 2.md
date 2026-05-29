# AI Recommended Talents Pipeline — Skills Matching with Embeddings

## Summary

When a TalentRequest is created, an in-app BackgroundService extracts skill names from the job description using Azure OpenAI, resolves each extracted skill to a canonical skill in the database using embedding-based similarity matching, then scores every eligible talent by comparing those resolved skills against their TalentSkill records. The top N candidates by match percent are inserted into the AI Recommended stage (OrderIndex 0) via TalentPipeline.

Embeddings are used only for skill name resolution — not for scoring talents. Talent scoring is pure skill overlap logic in C#, which keeps it fast, cheap, and fully explainable. Every match percent is directly traceable to specific skills.

---

## Azure Setup

| Resource                         | What to do                                                    | Cost        |
| -------------------------------- | ------------------------------------------------------------- | ----------- |
| Azure OpenAI (existing, US East) | Deploy `text-embedding-3-small` for skill name embeddings     | ~$1–5/month |
| Azure OpenAI (existing, US East) | Deploy a chat model (e.g. `gpt-4o-mini`) for skill extraction | ~$1–3/month |
| Key Vault (existing or new)      | Store endpoints, API keys                                     | ~$0         |

**Total estimated cost: ~$2–8/month.**

Everything else — background job, scoring logic, embedding storage — runs inside your existing app and database.

### When to add more infrastructure

|Trigger|What to add|
|---|---|
|App restarts are losing recommendation jobs mid-run|Azure Service Bus (~$10–11/month)|
|Talent pool exceeds ~10,000 profiles and scoring slows down|Azure AI Search Basic (~$75/month)|

Do not add these now.

---

## App Configuration

In App Service → Configuration, add as Key Vault references:

```
AZURE_OPENAI_ENDPOINT                   → your OpenAI resource endpoint
AZURE_OPENAI_API_KEY                    → Key Vault reference
AZURE_OPENAI_EMBEDDING_DEPLOYMENT       → e.g. "text-embedding-3-small"
AZURE_OPENAI_CHAT_DEPLOYMENT            → e.g. "gpt-4o-mini"
RECOMMENDATION_MIN_SCORE                → e.g. 0.60 (minimum match percent to include)
RECOMMENDATION_SKILL_MATCH_THRESHOLD    → e.g. 0.85 (cosine similarity threshold for skill resolution)
RECOMMENDATION_STALE_DAYS              → e.g. 14 (days before a re-run is triggered)
```

---

## Database Changes

### Changes to Skill table

Add embedding fields to the existing `Skill` table:

```
Skill (existing)
+ EmbeddingVector    (nvarchar(max), nullable) — JSON float array of the skill name embedding
+ EmbeddedAt         (datetime, nullable)      — when the embedding was last generated
+ EmbeddingModel     (nvarchar(50), nullable)  — model version used e.g. "text-embedding-3-small"
```

Embeddings are generated once per skill and reused. Only regenerated when the skill name changes or the model version is updated.

### Changes to TalentPipeline table

Add recommendation metadata fields:

```
TalentPipeline (existing)
+ RecommendationScore      (float, nullable)         — match percent as 0–1 decimal
+ RecommendationPercent    (int, nullable)            — RecommendationScore * 100, shown in UI
+ RecommendationSource     (nvarchar(50), nullable)   — always "SkillsMatch" for this approach
+ RecommendationVersion    (nvarchar(50), nullable)   — embedding model version used
+ RecommendedAt            (datetime, nullable)       — when this recommendation was inserted
+ MatchedSkills            (nvarchar(max), nullable)  — JSON array of matched skill names
+ MissingSkills            (nvarchar(max), nullable)  — JSON array of required skills the talent lacks
```

`MissingSkills` is worth adding — it gives recruiters immediate visibility into skill gaps, not just the match score.

Add migration for all changes in `JumpStartVAContext.cs`.

---

## How Scoring Works

### Step A — Skill extraction

Send the job description text to Azure OpenAI chat and ask it to return a raw list of skill names. No constraints — let it extract freely.

```csharp
var prompt = $"""
    Extract all technical and professional skills mentioned in the job description below.
    Return only a JSON array of skill name strings.
    No explanation, no markdown, no extra text — just the JSON array.

    Job description:
    {jobDescriptionText}
    """;

// Example response: ["NodeJS", "React.js", "Mongo", "REST APIs", "TypeScript"]
```

Parse the returned string into a `List<string>`.

### Step B — Skill name resolution via embeddings

For each extracted skill name, generate an embedding and compare it against the pre-embedded skills in your `Skill` table. Pick the closest match above the configured threshold.

```csharp
public async Task<Skill?> ResolveSkill(
    string extractedName,
    List<Skill> embeddedSkills,
    float threshold)
{
    var extractedEmbedding = await GenerateEmbedding(extractedName);

    return embeddedSkills
        .Where(s => s.EmbeddingVector != null)
        .Select(s => new {
            Skill = s,
            Score = CosineSimilarity(
                extractedEmbedding,
                JsonSerializer.Deserialize<float[]>(s.EmbeddingVector)
            )
        })
        .Where(x => x.Score >= threshold)
        .OrderByDescending(x => x.Score)
        .FirstOrDefault()?.Skill;
}

// "NodeJS"    → resolves to → Skill: "Node.js"
// "React.js"  → resolves to → Skill: "React"
// "Mongo"     → resolves to → Skill: "MongoDB"
// "SomethingObscure" → returns null → log for skills library review
```

If no match is found above the threshold, log the unresolved skill name for review — over time this builds a queue of skills missing from your library.

After resolution you have a clean list of `Skill` IDs that are required for this job request. Store these on the `TalentRequest` or pass them through the job.

### Step C — Talent scoring

For each eligible candidate, compare the resolved required skill IDs against their `TalentSkill` records.

```csharp
public RecommendationResult ScoreTalent(
    List<int> requiredSkillIds,
    List<TalentSkill> talentSkills)
{
    var talentSkillIds = talentSkills.Select(s => s.SkillId).ToHashSet();

    var matched = requiredSkillIds.Where(id => talentSkillIds.Contains(id)).ToList();
    var missing = requiredSkillIds.Where(id => !talentSkillIds.Contains(id)).ToList();

    float score = (float)matched.Count / requiredSkillIds.Count;

    return new RecommendationResult
    {
        RecommendationScore   = score,
        RecommendationPercent = (int)(score * 100),
        MatchedSkillIds       = matched,
        MissingSkillIds       = missing
    };
}
```

### Optional — weight by proficiency and years

If a required skill is present but only at beginner level, discount the match:

```csharp
float weightedScore = 0;

foreach (var skillId in requiredSkillIds)
{
    var talentSkill = talentSkills.FirstOrDefault(s => s.SkillId == skillId);
    if (talentSkill == null) continue;

    float proficiencyWeight = talentSkill.Proficiency switch
    {
        "Expert"       => 1.0f,
        "Intermediate" => 0.7f,
        "Beginner"     => 0.4f,
        _              => 0.5f
    };

    float yearsWeight = Math.Min(talentSkill.Years / 5f, 1.0f); // caps at 5 years = 1.0

    weightedScore += (proficiencyWeight * 0.6f) + (yearsWeight * 0.4f);
}

float finalScore = weightedScore / requiredSkillIds.Count;
```

Start without weighting and add it once the basic flow is working.

### Cosine similarity implementation

```csharp
public static float CosineSimilarity(float[] a, float[] b)
{
    float dot = 0, magA = 0, magB = 0;
    for (int i = 0; i < a.Length; i++)
    {
        dot  += a[i] * b[i];
        magA += a[i] * a[i];
        magB += b[i] * b[i];
    }
    return dot / (MathF.Sqrt(magA) * MathF.Sqrt(magB));
}
// 1.0 = identical meaning, 0.0 = completely unrelated
// Use 0.85 as the threshold for skill name matching
```

---

## Steps

### Step 1 — Deploy models in Azure OpenAI

In ai.azure.com → your US East resource → Deployments:

- Deploy `text-embedding-3-small` — for generating skill name embeddings
- Deploy `gpt-4o-mini` (or equivalent) — for extracting skills from job descriptions

Note both deployment names and add them to Key Vault.

### Step 2 — Add database migrations

Add the embedding fields to the `Skill` table and the recommendation metadata fields to `TalentPipeline`. Run the migration via `JumpStartVAContext.cs`.

### Step 3 — Build the skill embedding service

Create a `SkillEmbeddingService` that generates and stores embeddings for skills in your `Skill` table.

Trigger on:

- New skill added to the `Skill` table — embed immediately
- Skill name edited — regenerate since the name changed
- Nightly sweep — embed any skills with a null `EmbeddingVector`
- Model version update — re-embed all skills with the old model version

```csharp
public async Task EmbedSkillAsync(Skill skill)
{
    var embedding = await GenerateEmbedding(skill.Name);

    skill.EmbeddingVector = JsonSerializer.Serialize(embedding);
    skill.EmbeddedAt      = DateTime.UtcNow;
    skill.EmbeddingModel  = _embeddingDeployment;

    await _db.SaveChangesAsync();
}
```

Run the nightly sweep once on deploy to back-fill any existing skills that have no embedding yet.

### Step 4 — Build the BackgroundService job

Register an in-app `BackgroundService` that listens to an in-memory `Channel<int>` of `TalentRequestId` values.

On `TalentRequest` creation in `TalentRequestLogic.cs`, write the new request ID to the channel. The BackgroundService picks it up and does the following:

1. Load the TalentRequest — get job description text, required skills if any, and derive N
2. Call Azure OpenAI chat to extract raw skill names from the job description
3. Load all pre-embedded skills from the `Skill` table
4. For each extracted skill name, call `ResolveSkill` to find the closest match in the DB
5. Log any unresolved skill names to a review queue
6. Collect the resolved `Skill` IDs — these are the required skills for this request
7. Apply hard filters to the candidate pool — Approved OrgProfile, same org scope, active status, not already in any stage on this request
8. For each candidate, load their `TalentSkill` records and call `ScoreTalent`
9. Resolve matched and missing skill names from IDs for storage
10. Drop any candidate whose score is below `RECOMMENDATION_MIN_SCORE`
11. Sort descending by score and take the top N
12. Insert `TalentPipeline` rows with all metadata fields populated
13. On any failure, roll back all inserts for this run — never leave the stage partially populated

### Step 5 — Update pipeline retrieval APIs

In `TalentRequestPipelineLogic.cs`, update view models for the AI Recommended stage to surface:

- `RecommendationPercent` — displayed as match percentage
- `MatchedSkills` — the skills the talent has that the job requires
- `MissingSkills` — the skills the talent is lacking
- `RecommendationSource` — always `"SkillsMatch"` for this approach

### Step 6 — Handle stale recommendations

A scheduled sweep inside the same BackgroundService checks open TalentRequests where `RecommendedAt` is older than `RECOMMENDATION_STALE_DAYS`. For each, clear existing stage 0 rows and re-enqueue the job. This keeps the pool fresh as new talent profiles and skills are added.

---

## Unresolved Skills Review Queue

When an extracted skill name fails to match anything in the `Skill` table above the threshold, log it:

```
UnresolvedSkill (new table or just a log)
- ExtractedName    — what OpenAI returned e.g. "Prompt Engineering"
- TalentRequestId  — which request triggered it
- LoggedAt         — datetime
```

Review this periodically and add genuinely missing skills to the `Skill` table. The nightly embedding sweep will pick them up automatically. Over time this keeps your skills library current with what is actually appearing in real job descriptions.

---

## Safeguards

Enforced inside the job — not optional checks:

- Never insert a duplicate into stage 0 for the same request
- Never recommend a talent already in any later stage on the same request
- Only Approved OrgProfiles within the correct org scope enter the candidate pool
- Hard filters applied before scoring — skill matching never overrides access rules
- Job failures roll back cleanly — no partial inserts
- Minimum score threshold prevents weak matches from padding the pool
- Skills with no embedding are skipped during resolution — never cause a crash

---

## Fallback — Deterministic Skill Overlap

If Azure OpenAI is unavailable, fall back to using the skills already on the TalentRequest (if any are stored) and doing a direct ID match against TalentSkill records — no extraction or embedding needed.

Writes `RecommendationSource = "Deterministic"` so results are clearly distinguishable. Output shape is identical so no downstream code changes.

---

## Verification

1. Add a skill to the `Skill` table and confirm `EmbeddingVector` is populated
2. Edit a skill name and confirm the embedding is regenerated
3. Run skill extraction against a sample job description and confirm the raw list looks correct
4. Run skill resolution and confirm `"NodeJS"` → `"Node.js"`, `"Mongo"` → `"MongoDB"` etc.
5. Confirm unresolved skill names are logged
6. Create a TalentRequest and confirm the BackgroundService inserts exactly N rows into AI Recommended (OrderIndex 0) with score, percent, matched skills, missing skills, source, version, and timestamp all populated
7. Confirm no duplicates and no talents already in later stages appear in stage 0
8. Confirm only Approved OrgProfiles in the correct org scope appear
9. Set a candidate's score below the threshold and confirm they are excluded
10. Trigger a deliberate failure mid-run and confirm no partial rows remain
11. Confirm pipeline retrieval endpoints return percent, matched skills, and missing skills
12. Wait past the stale threshold and confirm the sweep clears old stage 0 rows and repopulates

---

## Relevant Files

|File|What changes|
|---|---|
|`TalentRequestLogic.cs`|Enqueue recommendation job on request creation|
|`TalentPipeline.cs`|Add score, matched skills, missing skills, and metadata fields|
|`PipelineLogic.cs`|Reference for insert patterns|
|`TalentRequestPipelineLogic.cs`|Expose percent, matched skills, and missing skills in view models|
|`TalentRequest.cs`|Source of job description text and N|
|`OrgProfile.cs`|Approval status and org scope filter|
|`TalentSkill.cs`|Talent skill records used for scoring|
|`Skill.cs`|Add EmbeddingVector, EmbeddedAt, EmbeddingModel fields|
|`Stage.cs`|AI Recommended stage (OrderIndex 0)|
|`JumpStartVAContext.cs`|Migration for Skill embedding fields and TalentPipeline metadata fields|

---

## Related Notes

- [[AI Recommended Talents Pipeline]]
- [[Azure AI Search]]
- [[Azure AI Document Intelligence]]
- [[Building Apps with OpenAI]] EOF 