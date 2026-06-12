"""
Solar System Navigator — Live Planet Data Fetcher
==================================================
Fetches real-time planet positions from NASA's JPL Horizons API
and saves them to data.json for the solar_system.html viewer.
"""

import requests
import json
import re
from datetime import datetime, timezone, timedelta


# Planet definitions -- Manual insert
PLANETS = [
    { "name":"Mercury", "horizons_id":"199", "color":"#b5b5b5", "radius":0.38,
      "type":"Rocky / Terrestrial", "diameter":"4,879 km", "moons":"0",
      "fact":"Mercury's surface swings from -180°C at night to 430°C during the day — the most extreme temperature range in the solar system." },
    { "name":"Venus",   "horizons_id":"299", "color":"#e8cda0", "radius":0.95,
      "type":"Rocky / Terrestrial", "diameter":"12,104 km", "moons":"0",
      "fact":"Venus rotates backwards. A day on Venus is longer than its year, and the Sun rises in the west." },
    { "name":"Earth",   "horizons_id":"399", "color":"#4a90d9", "radius":1.0,
      "type":"Rocky / Terrestrial", "diameter":"12,742 km", "moons":"1",
      "fact":"Earth is the only known planet confirmed to harbor life, and the only one with liquid water on its surface." },
    { "name":"Mars",    "horizons_id":"499", "color":"#c1440e", "radius":0.53,
      "type":"Rocky / Terrestrial", "diameter":"6,779 km", "moons":"2",
      "fact":"Mars hosts Olympus Mons, the tallest volcano in the solar system — nearly 3x the height of Mount Everest." },
    { "name":"Jupiter", "horizons_id":"599", "color":"#c88b3a", "radius":2.6,
      "type":"Gas Giant", "diameter":"139,820 km", "moons":"95",
      "fact":"Jupiter's Great Red Spot is a storm raging for over 350 years. Two Earths could fit inside it." },
    { "name":"Saturn",  "horizons_id":"699", "color":"#e4d191", "radius":2.2,
      "type":"Gas Giant", "diameter":"116,460 km", "moons":"146",
      "fact":"Saturn is so low in density it would float in water. Its rings are made of billions of ice and rock fragments." },
    { "name":"Uranus",  "horizons_id":"799", "color":"#7de8e8", "radius":1.7,
      "type":"Ice Giant", "diameter":"50,724 km", "moons":"28",
      "fact":"Uranus is tilted 98 degrees on its side — likely from an ancient collision. Its seasons last over 20 years each." },
    { "name":"Neptune", "horizons_id":"899", "color":"#4b70dd", "radius":1.65,
      "type":"Ice Giant", "diameter":"49,244 km", "moons":"16",
      "fact":"Neptune has the fastest winds in the solar system — up to 2,100 km/h. Its moon Triton orbits in reverse." },
]

# 1 Astronomical Unit in km — used to convert Horizons km output → AU
KM_PER_AU = 149_597_870.7

HORIZONS_URL = "https://ssd.jpl.nasa.gov/api/horizons.api"


def fetch_planet_position(planet: dict, date_str: str) -> dict:
    
    # Calls NASA Horizons and returns the planet's distance from the Sun in AU.
    # Horizons VECTORS output gives X, Y, Z in km — we divide by KM_PER_AU.
    
    stop = (datetime.strptime(date_str, "%Y-%m-%d") + timedelta(days=1)).strftime("%Y-%m-%d")

    query = (
        f"format=json"
        f"&COMMAND='{planet['horizons_id']}'"
        f"&OBJ_DATA='NO'"
        f"&MAKE_EPHEM='YES'"
        f"&EPHEM_TYPE='VECTORS'"
        f"&CENTER='500@10'"
        f"&START_TIME='{date_str}'"
        f"&STOP_TIME='{stop}'"
        f"&STEP_SIZE='1d'"
        f"&VEC_TABLE='2'"
        f"&REF_PLANE='ECLIPTIC'"
    )

    print(f"  Fetching {planet['name']}...", end=" ", flush=True)

    try:
        response = requests.get(f"{HORIZONS_URL}?{query}", timeout=15)
        response.raise_for_status()
        data = response.json()

        if "error" in data:
            raise ValueError(f"API error: {data['error']}")

        # parse_xyz returns km (Horizons default unit for VECTORS)
        x_km, y_km, z_km = parse_xyz(data.get("result", ""))

        # Convert km → AU
        x_au = x_km / KM_PER_AU
        y_au = y_km / KM_PER_AU
        z_au = z_km / KM_PER_AU

        distance_au = round((x_au**2 + y_au**2 + z_au**2) ** 0.5, 6)

        print(f"OK  {distance_au:.4f} AU from Sun")

        return {
            "x_au": round(x_au, 6),
            "y_au": round(y_au, 6),
            "z_au": round(z_au, 6),
            "distance_au": distance_au,
            "fetch_success": True
        }

    except Exception as e:
        print(f"ERROR: {e}")
        return { "x_au":None, "y_au":None, "z_au":None,
                 "distance_au":None, "fetch_success":False, "error":str(e) }


def parse_xyz(text: str):

    # Parses X, Y, Z from Horizons plain-text response (values are in km).
    # Data is between $$SOE and $$EOE markers.
    # Example line:  X = 1.234567E+08 Y =-3.456789E+07 Z = 1.234567E+05

    soe = text.find("$$SOE")
    eoe = text.find("$$EOE")
    if soe == -1 or eoe == -1:
        snippet = text[:600] if len(text) > 600 else text
        raise ValueError(f"Missing $$SOE/$$EOE markers. Response:\n{snippet}")

    block = text[soe:eoe]
    pattern = r"X\s*=\s*([-+]?\d+\.\d+E[+-]\d+)\s+Y\s*=\s*([-+]?\d+\.\d+E[+-]\d+)\s+Z\s*=\s*([-+]?\d+\.\d+E[+-]\d+)"
    match = re.search(pattern, block, re.IGNORECASE)
    if not match:
        raise ValueError("Could not parse X/Y/Z from response block.")

    return float(match.group(1)), float(match.group(2)), float(match.group(3))


def main():
    today = datetime.now(timezone.utc).strftime("%Y-%m-%d")
    print(f"\n  Solar System Navigator — Fetching positions for {today}")
    print("=" * 60)

    results = []
    for planet in PLANETS:
        pos = fetch_planet_position(planet, today)
        results.append({ **planet, **pos })

    output = {
        "fetched_at": datetime.now(timezone.utc).isoformat(),
        "date": today,
        "planets": results
    }

    with open("data.json", "w") as f:
        json.dump(output, f, indent=2)

    success = sum(1 for p in results if p.get("fetch_success"))
    print("=" * 60)
    print(f"  Done! {success}/{len(PLANETS)} planets fetched.")
    print(f"  Saved to data.json — open solar_system.html to view.\n")


if __name__ == "__main__":
    main()
