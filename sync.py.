import json
import urllib.request

LEAGUE_ID = 152084
START_YEAR = 2014
END_YEAR = 2026
BASE_URL = "https://fleaflicker.com"

def get_json(endpoint):
    url = f"{BASE_URL}/{endpoint}"
    req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
    try:
        with urllib.request.urlopen(req, timeout=15) as resp:
            return json.loads(resp.read().decode("utf-8"))
    except Exception as e:
        print(f"Skipping {endpoint}: {e}")
        return None

seasons = []
games = []
drafts = []
league_name = f"League #{LEAGUE_ID}"

for yr in range(START_YEAR, END_YEAR + 1):
    print(f"Fetching {yr}...")
    st = get_json(f"FetchLeagueStandings?sport=NFL&league_id={LEAGUE_ID}&season={yr}")
    if not st or "divisions" not in st:
        continue

    if st.get("league", {}).get("name"):
        league_name = st["league"]["name"]

    teams = []
    for div in st.get("divisions", []):
        for t in div.get("teams", []):
            owner = t["owners"][0]["displayName"] if t.get("owners") else t.get("name", "Unknown")
            teams.append({
                "id": t.get("id"),
                "name": t.get("name", "Unknown"),
                "owner": owner,
                "rank": t.get("rankOverall", 99),
                "pf": t.get("pointsFor", {}).get("value", 0)
            })
    seasons.append({"year": yr, "teams": teams})

    for wk in range(1, 19):
        sb = get_json(f"FetchLeagueScoreboard?sport=NFL&league_id={LEAGUE_ID}&season={yr}&scoring_period={wk}")
        if not sb or not sb.get("games"):
            break
        for g in sb["games"]:
            h_pts = g.get("homeScore", {}).get("score", {}).get("value", 0)
            a_pts = g.get("awayScore", {}).get("score", {}).get("value", 0)
            if g.get("home") and g.get("away") and (h_pts > 0 or a_pts > 0):
                h_owner = g["home"]["owners"][0]["displayName"] if g["home"].get("owners") else g["home"].get("name", "Unknown")
                a_owner = g["away"]["owners"][0]["displayName"] if g["away"].get("owners") else g["away"].get("name", "Unknown")
                games.append({
                    "year": yr,
                    "week": wk,
                    "home": h_owner,
                    "away": a_owner,
                    "homePts": h_pts,
                    "awayPts": a_pts
                })

    dr = get_json(f"FetchLeagueDraftBoard?sport=NFL&league_id={LEAGUE_ID}&season={yr}")
    if dr and dr.get("rows"):
        first_round = []
        for idx, cell in enumerate(dr["rows"][0].get("cells", [])):
            p = cell.get("player", {}).get("proPlayer", {})
            tm = cell.get("team", {})
            mgr = tm["owners"][0]["displayName"] if tm.get("owners") else tm.get("name", "Unknown")
            first_round.append({
                "pick": f"1.{idx + 1}",
                "manager": mgr,
                "player": p.get("nameFull", "Unknown"),
                "pos": p.get("position", "")
            })
        drafts.append({"year": yr, "picks": first_round})

output = {
    "leagueName": league_name,
    "leagueId": LEAGUE_ID,
    "seasons": seasons,
    "games": games,
    "drafts": drafts
}

with open("data.json", "w", encoding="utf-8") as f:
    json.dump(output, f, indent=2)

print(f"Saved {len(seasons)} seasons and {len(games)} games!")
