# New York Times Wordle Historical Solutions

This repository maintains a comprehensive, version-controlled historical archive of solutions for the New York Times Wordle game and its variants. The repository is designed with extensibility in mind to support future game variants (e.g., different word lengths, specialized dictionaries) while maintaining backward compatibility and consistent data structures.

## Repository Structure

The repository employs a variant-aware naming convention to support multiple Wordle game types:

- **`played_<N>.json`**: Machine-readable JSON data for N-letter variants
- **`SOLUTIONS_<N>.md`**: Human-readable markdown for N-letter variants

Currently tracking:

- **5-letter variant**: `played_5.json` and `SOLUTIONS_5.md` (standard NYT Wordle)

This architecture allows seamless addition of future variants (e.g., `played_6.json` for 6-letter games) without breaking existing integrations or requiring schema migrations.

## Data Formats

### JSON Format: [`played_5.json`](./played_5.json)

Structured JSON schema optimized for programmatic access and API integration:

**Schema:**

```typescript
{
  info: string;           // Dataset description
  last_update: string;    // ISO 8601 date (YYYY-MM-DD)
  played_count: number;   // Total games recorded
  played: {               // Game number → solution mapping
    [gameNumber: string]: string;
  };
}
```

**Example:**

```json
{
    "info": "Played NYT Wordle solutions",
    "last_update": "2025-11-22",
    "played_count": 1618,
    "played": {
        "1617": "THICK",
        "1616": "VOWEL",
        "1615": "GRAVE"
    }
}
```

**Design Rationale:**

- Game numbers stored as strings to maintain lexicographic ordering in JSON
- Flat key-value structure for O(1) lookup performance
- Metadata fields enable validation and cache invalidation strategies

### Markdown Format: [`SOLUTIONS_5.md`](./SOLUTIONS_5.md)

Human-readable format for documentation, archival, and non-programmatic use cases. Features:

- **Reverse chronological ordering**: Most recent solutions first for quick reference
- **Metadata header**: Last update timestamp and total game count
- **Auto-generated**: Derived from canonical JSON source via CI/CD pipeline
- **Variant-specific**: Filename suffix indicates word length (e.g., `_5` for 5-letter)

The markdown files are automatically synchronized with their JSON counterparts during the automated merge workflow, ensuring data consistency across formats.

## CI/CD Pipeline & Anti-Spoiler Mechanism

The repository implements a time-delayed merge strategy to balance data freshness with spoiler prevention:

### Workflow Architecture

```text
┌─────────────────┐
│  Solution Push  │ Push to branch: game-<N>_<timestamp>
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Auto-PR Job   │ Triggered by: push to game-* branches
│  (.github/      │ Actions:
│   workflows/    │  1. Checkout branch
│   auto-pr.yaml) │  2. Generate SOLUTIONS_<N>.md from played_<N>.json
└────────┬────────┘  3. Commit markdown file
         │           4. Create PR with 'auto-merge-delayed' label
         │
         ▼
┌─────────────────┐
│  13-Hour Delay  │ Cron: */45 * * * * (every 45 minutes)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Auto-Merge Job  │ Triggered by: schedule + workflow_dispatch
│  (.github/      │ Logic:
│   workflows/    │  1. Query open PRs with 'auto-merge-delayed' label
│   auto-merge.   │  2. Filter game-* branches
│   yaml)         │  3. Calculate PR age (created_at → now)
└────────┬────────┘  4. If age ≥ 13 hours: merge PR
         │           5. Create GitHub Release (tag: game number)
         │
         ▼
┌─────────────────┐
│  Master Branch  │ Contains: played_5.json + SOLUTIONS_5.md
│   + Release     │ Tagged: <game_number>
└─────────────────┘
```

### Key Technical Details

**Branch Naming Convention:**

- Pattern: `game-<game_number>_<identifier>`
- Example: `game-1649_20251224`
- Regex: `^game-(\d+)_`

**Delay Mechanism:**

- **Configurable**: `DELAY_HOURS` environment variable (default: 13)
- **Calculation**: `(Date.now() - PR.created_at) / 3600000 >= DELAY_HOURS`
- **Rationale**: 13 hours provides global coverage across time zones while maintaining daily update cadence

**Merge Strategy:**

- Method: `merge` (creates merge commit, preserves branch history)
- Target: `master` branch
- Atomic: Both JSON and markdown files merged simultaneously

**Release Automation:**

- Tag name: Game number (e.g., `1649`)
- Target: Merge commit SHA
- Idempotent: 422 errors (duplicate releases) are logged and ignored

### Spoiler Prevention

The 13-hour delay ensures solutions remain embargoed on the `master` branch until most players worldwide have had opportunity to complete the puzzle. Solutions exist in feature branches immediately but are not indexed or easily discoverable until merge.

## License

This repository is licensed under Creative Commons' **CC0 1.0 Universal License**.
