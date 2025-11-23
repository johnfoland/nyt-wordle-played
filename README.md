# New York Times Wordle Historical Solutions

This repository maintains a historical archive of solutions for the New York Times Wordle game.

## Data

The primary data file is [`played_5.json`](./played_5.json). It contains a JSON object with the following structure:

- `info`: Description of the file.
- `last_update`: Date of the last update (YYYY-MM-DD).
- `played_count`: Total number of played games recorded.
- `played`: An object mapping game numbers (as strings) to their 5-letter solution words.

### Example

```json
{
    "info": "Played NYT Wordle solutions",
    "last_update": "2025-11-22",
    "played_count": 1618,
    "played": {
        "1617": "THICK",
        "1616": "VOWEL",
        ...
    }
}
```

## Automation & Spoilers

This repository uses an automated workflow to publish solutions while attempting to minimize spoilers:

1. **Update Trigger**: A new branch containing the latest solution is pushed to the repository (e.g., `game-1234_...`).
2. **Auto PR**: A GitHub Action automatically creates a Pull Request for this branch.
3. **Delayed Merge**: Another Action monitors these PRs and waits for approximately **13 hours** after creation before merging them.
4. **Release**: Once merged, a GitHub Release is automatically created and tagged with the game number.

This delay ensures that the solution is not immediately merged into the `master` branch, giving players time to solve the puzzle before it appears in the official history.

## License

This repository is licensed under Creative Commons' **CC0 1.0 Universal License**.
