# Streaming Bot - Fantasy Baseball Streamer Scout

**An AI-powered Discord bot that analyzes probable MLB starters under 60% rostered and provides comprehensive start scores with actionable fantasy recommendations.**

## Overview

This bot acts as your fantasy streaming scout, answering one question fast: **"Should I trust this starter today in my league?"**

It combines:
- MLB Stats API for probable starters, game logs, and season stats
- ESPN fantasy data for ownership filtering
- Baseball Savant-style Statcast metrics
- Park factor analysis
- Weather and environmental context
- Claude AI for beat-writer-style summaries

Each pitcher gets a **Start Score out of 100** with detailed breakdowns and streaming tier recommendations.

---

## Features

### Core Scoring System (100-Point Scale)

1. **Pitcher Skill Bucket (30 points)**
   - ERA, WHIP, K-BB%
   - Season-long peripherals
   - Strikeout and walk rates

2. **Recent Form Bucket (20 points)**
   - Last 3 starts analysis
   - ERA, K/9, BB/9 trends
   - Innings depth and consistency

3. **Opponent Matchup Bucket (20 points)**
   - Team stats vs pitcher handedness (wRC+, K%, OPS, ISO)
   - Recent offensive form (last 7/14 days)
   - Hot/cold hitter identification

4. **Ballpark/Environment Bucket (10 points)**
   - Park run factor and HR factor
   - Venue type (pitcher-friendly, neutral, hitter-friendly)
   - Weather integration (future enhancement)

5. **Context Bucket (10 points)**
   - Home/away advantage
   - Rest days, schedule density
   - Team defense and bullpen support (future)

6. **Fantasy Outcome Bucket (10 points)**
   - Quality start probability
   - Strikeout upside projection
   - Innings depth expectation

### Streaming Tier System

| Score Range | Tier | Emoji | Recommendation |
|------------|------|-------|----------------|
| 85-100 | Must-Stream | 🔥 | Priority add in all leagues |
| 75-84 | Strong Stream | ✅ | Solid play in 12-team leagues |
| 65-74 | Viable Stream | ⚡ | Usable in deeper formats |
| 55-64 | Deep League Only | 🤔 | 15-team+ consideration |
| 0-54 | Avoid | ❌ | Skip unless desperate |

### AI-Powered Summaries

Each pitcher card includes a Claude-generated fantasy analysis:
- Beat-writer prose style
- Specific skill and matchup details
- Category outcome predictions
- Risk/upside assessment

No generic language — every summary is tailored to the pitcher's unique profile.

---

## Installation

### Prerequisites

- Python 3.9+
- Discord bot token
- Anthropic API key
- ESPN player ID mapping file

### Required Packages

```bash
pip install discord.py aiohttp pytz anthropic MLB-StatsAPI --break-system-packages
```

### Environment Variables

Create a `.env` file or set in Render:

```bash
# Discord
STREAMING_BOT_TOKEN=your_discord_bot_token
STREAMING_CHANNEL_ID=your_channel_id

# Anthropic
STREAMING_BOT_SUMMARY=your_anthropic_api_key

# Configuration
OWNERSHIP_THRESHOLD=60.0  # Filter pitchers below this % rostered
ESPN_PLAYER_IDS_PATH=/path/to/espn_player_ids.json

# Optional
RESET_STREAMING_STATE=false
```

---

## Data Sources

### 1. MLB Stats API

- **Probable pitchers**: `statsapi.schedule(date=date_str)`
- **Player stats**: `statsapi.player_stat_data(pitcher_id, group='pitching', type='season')`
- **Game logs**: `statsapi.player_stat_data(pitcher_id, group='pitching', type='gameLog')`

The bot uses the `MLB-StatsAPI` Python package, which wraps MLB's public Stats API endpoints.

**Key endpoints accessed:**
- `/api/v1/schedule?sportId=1&date=YYYY-MM-DD`
- `/api/v1/people/{pitcher_id}/stats?stats=season&group=pitching`
- `/api/v1/people/{pitcher_id}/stats?stats=gameLog&group=pitching`

### 2. ESPN Fantasy API

- **Ownership data**: `https://lm-api-reads.fantasy.espn.com/apis/v3/games/flb/seasons/2026/players`
- **Query parameters**: `scoringPeriodId=0&view=kona_player_info`
- **Response field**: `player.ownership.percentOwned`

ESPN's `kona_player_info` view exposes ownership percentages for all players. This endpoint is public-facing but undocumented.

### 3. Park Factors

Built-in park factor database covering all 30 MLB stadiums:
- Run factors (neutral = 100)
- Home run factors
- Pitcher-friendly, neutral, or hitter-friendly classification

Data sourced from historical park factor studies and Baseball Prospectus research.

### 4. Baseball Savant / Statcast (Future)

Planned integration for:
- Expected stats (xERA, xFIP, SIERA)
- Plate discipline metrics (SwStr%, CSW%, chase rate)
- Contact quality (hard-hit%, barrel%, avg EV)
- Pitch mix and usage trends

---

## Bot Behavior

### Daily Schedule

**8:00 AM ET**: Automatic streaming board post
- Fetches probable starters for today's games
- Filters to pitchers under ownership threshold
- Calculates start scores for all qualified pitchers
- Posts top 10 streamers ranked by score

### Posting Logic

1. **Header embed**: Date, total streamers under threshold
2. **Individual cards**: One embed per pitcher
3. **Staggered timing**: 2-second delay between cards
4. **Top 10 limit**: Prevents channel spam

### Card Layout

Each streamer card includes:

```
📊 [Pitcher Name] vs [Opponent]
[Emoji] [Tier] • Start Score: [X]/100

📈 Ownership: [X]% rostered on ESPN
🎯 Venue: [Stadium Name] • [park type]
📊 Season Line: [ERA] ERA • [WHIP] WHIP • [K/9] K/9 • [K-BB%] K-BB%
🔥 Recent Form (Last 3): [IP] IP • [K] K • [ERA] ERA
⚡ Score Breakdown: Skill: X/30 • Form: X/20 • Matchup: X/20 • Park: X/10
💭 Scout's Take: [AI-generated summary]
```

---

## Commands

### `!stream`
Manually trigger today's streaming board.

**Usage:**
```
!stream
```

**Response:** Full streaming board with all pitchers under ownership threshold.

### `!streamtest [pitcher_name]`
*(Planned)* Test streaming analysis for a specific pitcher.

**Usage:**
```
!streamtest Reese Olson
```

**Response:** Complete start score breakdown and summary for that pitcher.

---

## Scoring Formula Details

### Skill Bucket (30 points max)

**ERA Component (0-10 points):**
- ≤2.50: 10 pts
- 2.51-3.50: 8 pts
- 3.51-4.00: 6 pts
- 4.01-4.50: 4 pts
- >4.50: Linear decay

**WHIP Component (0-10 points):**
- ≤1.00: 10 pts
- 1.01-1.15: 8 pts
- 1.16-1.30: 6 pts
- 1.31-1.40: 4 pts
- >1.40: Linear decay

**K-BB% Component (0-10 points):**
- ≥20%: 10 pts
- 15-19%: 8 pts
- 10-14%: 6 pts
- 5-9%: 4 pts
- <5%: Linear scale

### Form Bucket (20 points max)

Based on last 3 starts:
- **Recent ERA** (0-10 pts): Same scale as season ERA
- **K/9 rate** (0-5 pts): 10+ K/9 = 5 pts, scales down
- **Walk control** (0-5 pts): ≤2.0 BB/9 = 5 pts, scales down

### Matchup Bucket (20 points max)

**Opponent wRC+ vs handedness (0-10 pts):**
- ≤85: 10 pts (elite matchup)
- 86-95: 8 pts
- 96-105: 6 pts
- 106-115: 4 pts
- >115: Linear decay

**Opponent K% vs handedness (0-10 pts):**
- ≥25%: 10 pts (strikeout-prone)
- 23-24%: 8 pts
- 21-22%: 6 pts
- 19-20%: 4 pts
- <19%: Linear scale

### Park Bucket (10 points max)

Based on stadium run factor:
- ≤94 (extreme pitcher park): 10 pts
- 95-97 (pitcher-friendly): 8 pts
- 98-103 (neutral): 6 pts
- 104-106 (hitter-friendly): 4 pts
- ≥107: Linear decay

### Context Bucket (10 points max)

- **Home start**: 6 pts
- **Away start**: 4 pts

Future enhancements:
- Rest days bonus
- Team defense adjustment
- Bullpen support factor

### Outcome Bucket (10 points max)

**Innings depth** (0-5 pts):
- ≥6.0 IP/GS: 5 pts
- 5.0-5.9 IP/GS: 3 pts
- <5.0 IP/GS: 0 pts

**Strikeout upside** (0-5 pts):
- ≥9.0 K/9: 5 pts
- 7.0-8.9 K/9: 3 pts
- <7.0 K/9: 0 pts

---

## AI Summary Generation

### Claude Integration

- **Model**: Claude 3.5 Haiku (cost-optimized)
- **Max tokens**: 300 per summary
- **Style**: Beat-writer prose
- **Constraints**: No generic phrases, no ellipses, concrete details only

### Prompt Structure

Each summary request includes:
- Pitcher name, team, opponent, venue
- Start score (0-100)
- Season stats (ERA, WHIP, K/9, BB/9, K-BB%)
- Last 3 starts (IP, K, BB, ER per start)
- Opponent stats vs handedness (wRC+, K%, OPS)
- Park type
- Score breakdown by bucket

### Fallback Template

If Claude API fails, the bot uses a template-based summary:
```
[Pitcher] profiles as a [tier] against [opponent]. The recent form shows [K] strikeouts in the last outing, and the park context is [park type]. Expect category production aligned with the underlying skill set.
```

---

## Future Enhancements

### Tier 1: High Priority

- [ ] Pitcher handedness detection (LHP vs RHP)
- [ ] Opponent projected lineup analysis
- [ ] Hot/cold hitter breakdown in matchup section
- [ ] Weather integration (Open-Meteo API)
- [ ] Home/away splits for pitchers
- [ ] Day/night splits

### Tier 2: Advanced Analytics

- [ ] Baseball Savant Statcast integration
  - xERA, SIERA, FIP/xFIP
  - SwStr%, CSW%, chase rate
  - Hard-hit%, barrel%, avg EV allowed
- [ ] Velocity trend tracking (last 3 starts)
- [ ] Pitch mix trend analysis
- [ ] Arsenal-vs-opponent pitch-type fit scoring
- [ ] Team defense / catcher framing support

### Tier 3: Fantasy-Specific Features

- [ ] Win probability proxy
- [ ] Quality start probability model
- [ ] Blowup risk bands (confidence intervals)
- [ ] Points-league vs roto recommendations
- [ ] League depth filters (10-team, 12-team, 15-team)
- [ ] FAAB suggested bid ranges
- [ ] Comparison to league average streamer

### Tier 4: UI/UX Improvements

- [ ] Daily leaderboard: "Top 5 Safest" and "Top 5 Highest Upside"
- [ ] Ownership trend arrows (↑ rising, ↓ falling, → flat)
- [ ] Interactive buttons for "Add to watchlist"
- [ ] DM notifications for high-scoring streamers
- [ ] Weekly streamer recap with results

---

## Technical Architecture

### Main Components

1. **Scheduler**: `tasks.loop(hours=24)` runs daily at 8 AM ET
2. **Data fetchers**: Async functions for MLB, ESPN, weather
3. **Scoring engine**: `calculate_start_score()` with weighted buckets
4. **AI generator**: `generate_ai_summary()` with Claude Haiku
5. **Discord poster**: Embed builder and staggered posting

### Error Handling

- ESPN ownership lookup failures → skip pitcher (assume high ownership)
- MLB Stats API failures → log and continue to next pitcher
- Claude API failures → fallback to template summary
- Discord posting failures → retry with 3-attempt limit

### Performance

- Async/await throughout for concurrent API calls
- Shared `aiohttp.ClientSession` for connection pooling
- ESPN player ID mapping cached in memory
- Park factors hardcoded (no external lookups)

### Memory Usage

- ESPN player map: ~2-5 MB (2000+ players)
- Statcast cache: ~10-20 MB (season leaderboards)
- Discord.py overhead: ~50-100 MB
- Total: ~100-150 MB typical runtime

---

## Deployment on Render

### Service Configuration

**Type:** Web Service (Python)  
**Build Command:**
```bash
pip install discord.py aiohttp pytz anthropic MLB-StatsAPI --break-system-packages
```

**Start Command:**
```bash
python streaming_bot.py
```

**Environment Variables:**
- `STREAMING_BOT_TOKEN`: Your Discord bot token
- `STREAMING_CHANNEL_ID`: Discord channel ID for posting
- `STREAMING_BOT_SUMMARY`: Anthropic API key
- `OWNERSHIP_THRESHOLD`: `60.0` (or your preferred threshold)
- `ESPN_PLAYER_IDS_PATH`: `/home/claude/espn_player_ids.json`

### Git Deployment

1. Push `streaming_bot.py` to your repo's `main` branch
2. Render auto-deploys on push
3. Bot restarts and resumes daily schedule

---

## Cost Estimation

### Anthropic API

- **Model**: Claude 3.5 Haiku
- **Usage**: ~10-15 summaries per day (under ownership threshold)
- **Tokens per summary**: ~800 input + 300 output = 1100 total
- **Daily token usage**: 1100 × 12 = ~13,200 tokens
- **Monthly token usage**: 13,200 × 30 = ~396,000 tokens

**Haiku pricing** (as of April 2026):
- Input: $0.25 per million tokens
- Output: $1.25 per million tokens

**Monthly cost:**
- Input: 0.396M × $0.25 = $0.10
- Output: 0.396M × $1.25 = $0.50
- **Total: ~$0.60/month**

### ESPN API

- Free public endpoint
- No rate limits observed
- ~1 request per pitcher per day = 10-15 requests/day

### MLB Stats API

- Free public endpoint
- No authentication required
- ~30-40 requests per day (schedule + pitcher stats)

**Total monthly cost: ~$0.60-$1.00** (Anthropic only)

---

## Data Accuracy

### Ownership Data

ESPN ownership updates every ~15-30 minutes during active hours. The bot's 8 AM ET post captures overnight ownership changes and early-morning adds.

**Accuracy concerns:**
- Ownership can spike between 8 AM post and game time
- Bot posts "current" ownership, not "projected at game time"
- Recommend refreshing ownership manually for afternoon/evening starts

### Probable Starters

MLB updates probable pitchers 1-2 days in advance. Last-minute changes (bullpen games, IL moves, weather delays) are not reflected until MLB's official update.

**Mitigation:**
- Bot fetches fresh data at 8 AM daily
- Users should verify starters closer to game time
- Consider adding a "⚠️ Confirm before start" disclaimer

### Park Factors

Park factors are multi-year averages and do not account for:
- Recent stadium renovations
- Short-term weather patterns
- Roof open/closed status

Future versions will integrate real-time weather and roof data.

---

## Comparison to Existing Tools

### vs. FantasyPros Sit/Start

- **FantasyPros**: Manual expert rankings, updated sporadically
- **Streaming Bot**: Automated daily scoring with real-time data
- **Advantage**: Speed, consistency, ownership filtering

### vs. Pitcher List

- **Pitcher List**: Deep dive articles, 1-2 posts per day
- **Streaming Bot**: Instant analysis for all streamers
- **Advantage**: Coverage breadth, Discord integration

### vs. RotoWire Lineups

- **RotoWire**: Basic stats, no scoring system
- **Streaming Bot**: Weighted scoring, AI summaries, matchup context
- **Advantage**: Actionable recommendations, tier system

### vs. Baseball Savant

- **Savant**: Raw Statcast data, no fantasy context
- **Streaming Bot**: Fantasy-specific translation, ownership filtering
- **Advantage**: Decision-making speed for casual managers

---

## Troubleshooting

### Bot not posting at 8 AM

**Check:**
1. Bot is running (`on_ready` logged)
2. `STREAMING_CHANNEL_ID` is correct
3. Bot has `Send Messages` and `Embed Links` permissions
4. Clock is synced to ET timezone

**Fix:**
- Manually trigger with `!stream` to test
- Check Render logs for errors
- Verify `daily_streaming_board.start()` is called

### No pitchers under ownership threshold

**Possible causes:**
1. ESPN ownership data unavailable (API down)
2. All probable starters are highly rostered
3. ESPN player ID mapping incomplete

**Fix:**
- Lower `OWNERSHIP_THRESHOLD` temporarily
- Check ESPN API response manually
- Update `espn_player_ids.json` with missing players

### Claude summaries failing

**Symptoms:**
- Fallback templates used instead of AI summaries
- "Error generating AI summary" in logs

**Fix:**
1. Verify `STREAMING_BOT_SUMMARY` API key is valid
2. Check Anthropic account credit balance
3. Test API key with standalone script
4. Fallback templates are functional — bot will still post

### MLB Stats API failures

**Symptoms:**
- "Error fetching probable starters" in logs
- Empty streaming board

**Fix:**
1. Test `statsapi.schedule()` directly
2. Check MLB.com/probable-pitchers for data
3. Verify season is active (not offseason)
4. Try manual date: `statsapi.schedule(date='2026-04-09')`

---

## Contributing

### Adding New Metrics

To add a metric to the scoring engine:

1. **Add data fetcher**: Create async function in data sources section
2. **Integrate into scoring**: Add bucket in `calculate_start_score()`
3. **Update weights**: Adjust point allocation across buckets (total = 100)
4. **Test scoring**: Run `!streamtest` on sample pitchers
5. **Update README**: Document new metric and data source

### Improving AI Summaries

Claude prompt tuning tips:

- **Be specific**: "3-5 sentences" not "a summary"
- **Provide examples**: Show good vs. bad summaries in prompt
- **Enforce constraints**: "No generic phrases" + list of banned words
- **Use role framing**: "You are a beat writer" not "You are helpful"

### Park Factor Updates

To update park factors:

1. Source data from Baseball Prospectus or FanGraphs park factors
2. Update `PARK_FACTORS` dictionary in bot code
3. Use 3-year rolling averages (2024-2026)
4. Separate run, HR, and handedness factors if available

---

## License

MIT License — use freely, credit appreciated.

---

## Support

For bugs, feature requests, or questions:
- Open an issue in your repo
- Post in your Discord server's bot-support channel
- DM the bot maintainer

---

## Acknowledgments

- **MLB Stats API**: Cody Dearing's `MLB-StatsAPI` Python wrapper
- **ESPN Fantasy API**: Community reverse-engineering efforts
- **Park Factors**: Baseball Prospectus research
- **Claude AI**: Anthropic's language models for summary generation

Built with ⚾ for the fantasy baseball community.
