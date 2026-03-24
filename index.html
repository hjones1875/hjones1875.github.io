import urllib.request
import json
import math
import csv
import io

# ── Data sources ──────────────────────────────────────────────────────────────
# OurAirports: free, public-domain, updated daily. No API key needed.
AIRPORTS_CSV = "https://davidmegginson.github.io/ourairports-data/airports.csv"
RUNWAYS_CSV  = "https://davidmegginson.github.io/ourairports-data/runways.csv"

# ── Live METAR fetch ──────────────────────────────────────────────────────────

def fetch_metar(icao):
    url = f"https://aviationweather.gov/api/data/metar?ids={icao}&format=json"
    try:
        with urllib.request.urlopen(url, timeout=8) as resp:
            data = json.loads(resp.read())
        if not data:
            print(f"  No METAR found for {icao}. Check the airport code.")
            return None
        return data[0]
    except Exception as e:
        print(f"  Could not fetch METAR: {e}")
        return None

# ── OurAirports CSV helpers ───────────────────────────────────────────────────

def fetch_csv(url):
    req = urllib.request.Request(url, headers={"User-Agent": "StudentPilotWeatherChecker/1.0"})
    with urllib.request.urlopen(req, timeout=15) as resp:
        raw = resp.read().decode("utf-8")
    reader = csv.DictReader(io.StringIO(raw))
    return list(reader)


def fetch_airport_info(icao):
    icao = icao.upper()

    print(f"  Fetching airport data for {icao} (OurAirports)...")
    try:
        airports = fetch_csv(AIRPORTS_CSV)
    except Exception as e:
        print(f"  Could not fetch airports.csv: {e}")
        return None

    apt = next(
        (a for a in airports
         if a.get("ident", "").upper() == icao
         or a.get("gps_code", "").upper() == icao
         or a.get("icao_code", "").upper() == icao),
        None
    )
    if not apt:
        print(f"  Airport {icao} not found in OurAirports data.")
        return None

    apt_id   = apt["id"]
    apt_type = apt.get("type", "")
    apt_name = apt.get("name", icao)

    if apt_type in ("large_airport", "medium_airport"):
        towered = True
    elif apt_type in ("small_airport", "seaplane_base", "heliport", "balloonport", "closed"):
        towered = False
    else:
        towered = None

    print(f"  Fetching runway data for {icao} (OurAirports)...")
    try:
        all_runways = fetch_csv(RUNWAYS_CSV)
    except Exception as e:
        print(f"  Could not fetch runways.csv: {e}")
        all_runways = []

    runways = []
    for row in all_runways:
        if row.get("airport_ref") != apt_id and row.get("airport_ident", "").upper() != icao:
            continue
        if row.get("closed", "0") == "1":
            continue
        for prefix in ("le_", "he_"):
            ident   = (row.get(f"{prefix}ident") or "").strip()
            hdg_raw = row.get(f"{prefix}heading_degT") or row.get(f"{prefix}heading_degM") or ""
            if not ident:
                continue
            try:
                hdg = float(hdg_raw)
            except (ValueError, TypeError):
                try:
                    hdg = float(ident.rstrip("LRC")) * 10.0
                except ValueError:
                    continue
            runways.append({"name": ident, "heading_deg": hdg % 360})

    return {
        "name":    apt_name,
        "type":    apt_type,
        "towered": towered,
        "runways": runways,
    }

# ── Parse METAR fields ────────────────────────────────────────────────────────

def parse_conditions(metar):
    conditions = {}
    conditions["wind_speed"] = metar.get("wspd") or 0
    gust_raw = metar.get("wgst") or 0
    conditions["gusts"] = max(0, gust_raw - conditions["wind_speed"])
    vis = metar.get("visib")
    try:
        conditions["visibility"] = float(vis) if vis is not None else 10.0
    except:
        conditions["visibility"] = 10.0
    sky = metar.get("sky") or []
    ceiling = 99999
    for layer in sky:
        cover = layer.get("cover", "")
        alt   = layer.get("base", 99999) or 99999
        if cover in ("BKN", "OVC"):
            ceiling = min(ceiling, alt)
    conditions["ceiling"] = ceiling if ceiling < 99999 else 99999
    conditions["wind_dir"] = metar.get("wdir") or 0
    return conditions

# ── Wind component calculations ───────────────────────────────────────────────

def crosswind_component(wind_speed, wind_dir, runway_heading):
    angle = abs(wind_dir - runway_heading)
    if angle > 180:
        angle = 360 - angle
    return round(wind_speed * math.sin(math.radians(angle)), 1)

def headwind_component(wind_speed, wind_dir, runway_heading):
    angle = wind_dir - runway_heading
    return round(wind_speed * math.cos(math.radians(angle)), 1)

# ── Best runway selector ──────────────────────────────────────────────────────

def rank_runways(runways, wind_speed, wind_dir):
    scored = []
    for rwy in runways:
        cx = abs(crosswind_component(wind_speed, wind_dir, rwy["heading_deg"]))
        hw = headwind_component(wind_speed, wind_dir, rwy["heading_deg"])
        scored.append((rwy["name"], cx, hw, rwy["heading_deg"]))
    scored.sort(key=lambda x: (x[1], -x[2]))
    return scored

# ── Condition rating ──────────────────────────────────────────────────────────

def rate(key, value, limits):
    lo = limits[key]
    if key in ("visibility", "ceiling"):
        if value <= lo["no_go"]:   return "NO-GO"
        if value <= lo["caution"]: return "CAUTION"
        return "GO"
    else:
        if value >= lo["no_go"]:   return "NO-GO"
        if value >= lo["caution"]: return "CAUTION"
        return "GO"

# ── Pretty printer ────────────────────────────────────────────────────────────

def display(airport, apt_name, conditions, limits, runway_hdg, runway_name, profile_label, role="DEPARTURE"):
    cx = crosswind_component(conditions["wind_speed"], conditions["wind_dir"], runway_hdg)
    conditions = dict(conditions)   # don't mutate the original
    conditions["crosswind"] = cx

    check_keys = ["wind_speed", "visibility", "ceiling", "crosswind", "gusts"]
    labels = {
        "wind_speed":  ("Wind Speed",  "kt"),
        "visibility":  ("Visibility",  "sm"),
        "ceiling":     ("Ceiling",     "ft"),
        "crosswind":   ("Crosswind",   "kt"),
        "gusts":       ("Gusts",       "kt"),
    }

    results = {k: rate(k, conditions[k], limits) for k in check_keys}
    overall = (
        "NO-GO"   if "NO-GO"   in results.values() else
        "CAUTION" if "CAUTION" in results.values() else
        "GO"
    )

    sep = "=" * 56
    print(f"\n{sep}")
    print(f"  {role} WEATHER — {airport} ({apt_name})")
    print(f"  Runway {runway_name}   |   Profile: {profile_label}")
    print(sep)
    for k in check_keys:
        lbl, unit = labels[k]
        val    = conditions[k]
        status = results[k]
        print(f"  {lbl:<14} {val:>7} {unit:<4}  →  {status}")

    print(f"\n{sep}")
    print(f"  OVERALL:  {overall}")
    if overall == "GO":
        print(f"  Conditions are within your minimums. Good to go.")
    elif overall == "CAUTION":
        print(f"  Marginal conditions — consult your instructor before proceeding.")
    else:
        print(f"  Conditions exceed your minimums. Do not proceed.")
    print(sep)

    # Collect only the conditions that are CAUTION or NO-GO for the summary
    warnings = []
    for k in check_keys:
        if results[k] != "GO":
            lbl, unit = labels[k]
            warnings.append(f"{lbl} {conditions[k]} {unit}  ->  {results[k]}")

    return {
        "overall":  overall,
        "runway":   runway_name,
        "warnings": warnings,
    }

# ── Flight profiles ───────────────────────────────────────────────
#
# caution == no_go on visibility means no warning band:
# above the limit is GO, at/below is NO-GO immediately.

# ── Solo student profiles (unchanged) ─────────────────────────────
SOLO_PROFILES = {
    "local": {
        "label":      "Solo — Local",
        "wind_speed": {"caution": 10,   "no_go": 12  },
        "crosswind":  {"caution":  6,   "no_go":  7  },
        "gusts":      {"caution": 10,   "no_go": 12  },
        "visibility": {"caution":  9.9, "no_go":  9.9},
        "ceiling":    {"caution": 4200, "no_go": 3500},
    },
    "pattern": {
        "label":      "Solo — Pattern",
        "wind_speed": {"caution": 10,   "no_go": 12  },
        "crosswind":  {"caution":  6,   "no_go":  7  },
        "gusts":      {"caution": 10,   "no_go": 12  },
        "visibility": {"caution":  6,   "no_go":  5  },
        "ceiling":    {"caution": 1800, "no_go": 1500},
    },
    "crosscountry": {
        "label":      "Solo — Cross Country",
        "wind_speed": {"caution": 10,   "no_go": 12  },
        "crosswind":  {"caution":  6,   "no_go":  7  },
        "gusts":      {"caution": 10,   "no_go": 12  },
        "visibility": {"caution":  9.9, "no_go":  9.9},
        "ceiling":    {"caution": 7200, "no_go": 6000},
    },
}

# ── Private Pilot profiles ─────────────────────────────────────
# Conservative personal margins above FAR 91.155 VFR minimums.
PPL_PROFILES = {
    "local": {
        "label":      "Private Pilot — Local",
        "wind_speed": {"caution": 18,   "no_go": 25  },
        "crosswind":  {"caution": 12,   "no_go": 15  },
        "gusts":      {"caution": 12,   "no_go": 18  },
        "visibility": {"caution":  4,   "no_go":  3  },
        "ceiling":    {"caution": 1500, "no_go": 1000},
    },
    "crosscountry": {
        "label":      "Private Pilot — Cross Country",
        "wind_speed": {"caution": 18,   "no_go": 25  },
        "crosswind":  {"caution": 12,   "no_go": 15  },
        "gusts":      {"caution": 12,   "no_go": 18  },
        "visibility": {"caution":  5,   "no_go":  3  },
        "ceiling":    {"caution": 2000, "no_go": 1500},
    },
}

# ── Instrument Rating profiles ─────────────────────────────────
# Personal minimums layered on published approach mins. Not a substitute
# for checking approach plates. 300 ft / 3/4 SM conservative IFR no-go.
IFR_PROFILES = {
    "ifr_flight": {
        "label":      "IFR Flight",
        "wind_speed": {"caution": 22,   "no_go": 30   },
        "crosswind":  {"caution": 15,   "no_go": 20   },
        "gusts":      {"caution": 15,   "no_go": 20   },
        "visibility": {"caution":  1,   "no_go":  0.75},
        "ceiling":    {"caution":  500, "no_go":  300 },
    },
    "vfr_on_top": {
        "label":      "IFR — VFR on Top / VMC Cruise",
        "wind_speed": {"caution": 22,   "no_go": 30  },
        "crosswind":  {"caution": 15,   "no_go": 20  },
        "gusts":      {"caution": 15,   "no_go": 20  },
        "visibility": {"caution":  4,   "no_go":  3  },
        "ceiling":    {"caution": 1500, "no_go": 1000},
    },
}

# ── Commercial Pilot profiles ─────────────────────────────────
# Part 91 personal flight. Higher wind tolerance reflects greater proficiency.
CPL_PROFILES = {
    "local": {
        "label":      "Commercial — Local / Proficiency",
        "wind_speed": {"caution": 20,   "no_go": 28  },
        "crosswind":  {"caution": 15,   "no_go": 20  },
        "gusts":      {"caution": 15,   "no_go": 20  },
        "visibility": {"caution":  4,   "no_go":  3  },
        "ceiling":    {"caution": 1500, "no_go": 1000},
    },
    "crosscountry": {
        "label":      "Commercial — Cross Country",
        "wind_speed": {"caution": 20,   "no_go": 28  },
        "crosswind":  {"caution": 15,   "no_go": 20  },
        "gusts":      {"caution": 15,   "no_go": 20  },
        "visibility": {"caution":  5,   "no_go":  3  },
        "ceiling":    {"caution": 2000, "no_go": 1500},
    },
}


def print_profile_table(profile):
    print(f"\n  Profile loaded: {profile['label']}")
    print(f"  {'Condition':<18} {'Caution':>10}  {'No-Go':>10}")
    print(f"  {'-'*18}  {'-'*9}  {'-'*9}")
    rows = [
        ("Wind Speed",  profile["wind_speed"], "kt"),
        ("Crosswind",   profile["crosswind"],  "kt"),
        ("Gusts",       profile["gusts"],      "kt"),
        ("Visibility",  profile["visibility"], "sm"),
        ("Ceiling",     profile["ceiling"],    "ft"),
    ]
    for name, vals, unit in rows:
        print(f"  {name:<18} {vals['caution']:>8} {unit}  {vals['no_go']:>8} {unit}")


def build_custom_limits(label="Custom"):
    print("\n  Enter your minimums (press Enter to accept the shown default):")
    defaults = {
        "wind_speed":  {"caution": 15,   "no_go": 20  },
        "visibility":  {"caution":  6,   "no_go":  3  },
        "ceiling":     {"caution": 2500, "no_go": 1500},
        "crosswind":   {"caution":  8,   "no_go": 12  },
        "gusts":       {"caution": 10,   "no_go": 15  },
    }
    prompts = {
        "wind_speed": ("Max total wind — caution (kt)",   "Max total wind — no-go (kt)"),
        "visibility": ("Min visibility — caution (sm)",   "Min visibility — no-go (sm)"),
        "ceiling":    ("Min ceiling — caution (ft AGL)",  "Min ceiling — no-go (ft AGL)"),
        "crosswind":  ("Max crosswind — caution (kt)",    "Max crosswind — no-go (kt)"),
        "gusts":      ("Max gust add-on — caution (kt)",  "Max gust add-on — no-go (kt)"),
    }
    limits = {}
    for key, (q_c, q_n) in prompts.items():
        d = defaults[key]
        raw_c = input(f"    {q_c} [{d['caution']}]: ").strip()
        raw_n = input(f"    {q_n} [{d['no_go']}]: ").strip()
        limits[key] = {
            "caution": float(raw_c) if raw_c else d["caution"],
            "no_go":   float(raw_n) if raw_n else d["no_go"],
        }
    return limits, label


def ask_destination(prompt="  Are you landing at a different airport? [y/N]: "):
    """Ask if landing elsewhere; return ICAO string or None."""
    diff = input(prompt).strip().lower()
    if diff in ("y", "yes"):
        icao = input("  Enter destination airport ICAO code (e.g. KSAV): ").strip().upper()
        if not icao:
            print("  No destination entered — skipping destination check.")
            return None
        return icao
    return None


def get_limits():
    """Ask pilot rating then flight type. Returns (limits, label, dest_icao)."""
    sep = "-" * 56
    print(f"\n{sep}")
    print("  PILOT RATING & FLIGHT PROFILE")
    print(sep)
    print("\n  What is your pilot rating?")
    print("    1  Solo student")
    print("    2  Private Pilot (PPL)")
    print("    3  Instrument Rating (IFR)")
    print("    4  Commercial Pilot (CPL)")
    print("    5  Flying with instructor (dual)")
    print("    6  Custom — enter my own minimums")
    rating = input("\n  Choice [1-6]: ").strip()

    # ── 1. SOLO STUDENT ───────────────────────────────────────────────
    if rating == "1":
        print("\n  What kind of solo flight?")
        print("    1  Local")
        print("    2  Pattern")
        print("    3  Cross country")
        print("    4  Custom — enter my own minimums")
        solo_choice = input("\n  Choice [1/2/3/4]: ").strip()
        key_map = {"1": "local", "2": "pattern", "3": "crosscountry"}
        if solo_choice in key_map:
            profile = SOLO_PROFILES[key_map[solo_choice]]
            print_profile_table(profile)
            limits = {k: v for k, v in profile.items() if k != "label"}
            dest_icao = None
            if solo_choice == "3":
                dest_icao = input("\n  Enter destination ICAO (e.g. KSAV): ").strip().upper() or None
                if not dest_icao:
                    print("  No destination entered — skipping.")
            elif solo_choice in ("1", "2"):
                dest_icao = ask_destination()
            return limits, profile["label"], dest_icao
        else:
            limits, _ = build_custom_limits()
            return limits, "Solo — Custom", None

    # ── 2. PRIVATE PILOT ──────────────────────────────────────────
    elif rating == "2":
        print("\n  What kind of flight?")
        print("    1  Local / Pattern")
        print("    2  Cross country")
        print("    3  Custom — enter my own minimums")
        ppl_choice = input("\n  Choice [1/2/3]: ").strip()
        ppl_map = {"1": "local", "2": "crosscountry"}
        if ppl_choice in ppl_map:
            profile = PPL_PROFILES[ppl_map[ppl_choice]]
            print_profile_table(profile)
            limits = {k: v for k, v in profile.items() if k != "label"}
            dest_icao = None
            if ppl_choice == "2":
                dest_icao = input("\n  Enter destination ICAO (e.g. KSAV): ").strip().upper() or None
                if not dest_icao:
                    print("  No destination entered — skipping.")
            else:
                dest_icao = ask_destination()
            return limits, profile["label"], dest_icao
        else:
            limits, _ = build_custom_limits()
            return limits, "Private Pilot — Custom", None

    # ── 3. INSTRUMENT RATING ────────────────────────────────────
    elif rating == "3":
        print("\n  What kind of flight?")
        print("    1  IFR flight (in IMC / flying an instrument approach)")
        print("    2  VFR on top / VMC cruise")
        print("    3  Custom — enter my own minimums")
        ifr_choice = input("\n  Choice [1/2/3]: ").strip()
        ifr_map = {"1": "ifr_flight", "2": "vfr_on_top"}
        if ifr_choice in ifr_map:
            profile = IFR_PROFILES[ifr_map[ifr_choice]]
            print_profile_table(profile)
            limits = {k: v for k, v in profile.items() if k != "label"}
            print("\n  NOTE: Always check published approach plate minimums —")
            print("  these personal minimums do not replace LOC/ILS/RNAV DAs.")
            dest_icao = input("\n  Enter destination ICAO (or Enter to skip): ").strip().upper() or None
            if not dest_icao:
                dest_icao = ask_destination()
            return limits, profile["label"], dest_icao
        else:
            limits, _ = build_custom_limits()
            return limits, "IFR — Custom", None

    # ── 4. COMMERCIAL PILOT ─────────────────────────────────────
    elif rating == "4":
        print("\n  What kind of flight?")
        print("    1  Local / Proficiency")
        print("    2  Cross country")
        print("    3  Custom — enter my own minimums")
        cpl_choice = input("\n  Choice [1/2/3]: ").strip()
        cpl_map = {"1": "local", "2": "crosscountry"}
        if cpl_choice in cpl_map:
            profile = CPL_PROFILES[cpl_map[cpl_choice]]
            print_profile_table(profile)
            limits = {k: v for k, v in profile.items() if k != "label"}
            dest_icao = None
            if cpl_choice == "2":
                dest_icao = input("\n  Enter destination ICAO (e.g. KSAV): ").strip().upper() or None
                if not dest_icao:
                    print("  No destination entered — skipping.")
            else:
                dest_icao = ask_destination()
            return limits, profile["label"], dest_icao
        else:
            limits, _ = build_custom_limits()
            return limits, "Commercial — Custom", None

    # ── 5. DUAL / WITH INSTRUCTOR ───────────────────────────────
    elif rating == "5":
        print("\n  Dual flight — using relaxed instructor defaults.")
        print("  Press Enter to accept each value, or type your own.\n")
        dual_defaults = {
            "wind_speed":  {"caution": 18,   "no_go": 25  },
            "visibility":  {"caution":  4,   "no_go":  3  },
            "ceiling":     {"caution": 2000, "no_go": 1000},
            "crosswind":   {"caution": 10,   "no_go": 15  },
            "gusts":       {"caution": 12,   "no_go": 18  },
        }
        prompts = {
            "wind_speed": ("Max total wind — caution (kt)",   "Max total wind — no-go (kt)"),
            "visibility": ("Min visibility — caution (sm)",   "Min visibility — no-go (sm)"),
            "ceiling":    ("Min ceiling — caution (ft AGL)",  "Min ceiling — no-go (ft AGL)"),
            "crosswind":  ("Max crosswind — caution (kt)",    "Max crosswind — no-go (kt)"),
            "gusts":      ("Max gust add-on — caution (kt)",  "Max gust add-on — no-go (kt)"),
        }
        limits = {}
        for key, (q_c, q_n) in prompts.items():
            d = dual_defaults[key]
            raw_c = input(f"    {q_c} [{d['caution']}]: ").strip()
            raw_n = input(f"    {q_n} [{d['no_go']}]: ").strip()
            limits[key] = {
                "caution": float(raw_c) if raw_c else d["caution"],
                "no_go":   float(raw_n) if raw_n else d["no_go"],
            }
        dest_icao = ask_destination()
        return limits, "Dual / With Instructor", dest_icao

    # ── 6. FULLY CUSTOM ──────────────────────────────────────────────
    else:
        limits, label = build_custom_limits()
        dest_icao = ask_destination()
        return limits, label, dest_icao


# ── Airport assessment (reusable for both departure and destination) ───────────

def assess_airport(icao, limits, profile_label, role="DEPARTURE"):
    """
    Fetch METAR + airport info for `icao`, pick the best/assigned runway,
    run the weather assessment, and return the overall result string.
    """
    sep = "-" * 56
    print(f"\n{sep}")
    print(f"  {role} AIRPORT: {icao}")
    print(sep)

    # METAR
    print(f"\n  Fetching live METAR for {icao}...")
    metar = fetch_metar(icao)
    if not metar:
        print(f"  Cannot assess {icao} — no METAR available.")
        return None

    raw_text = metar.get("rawOb", "")
    print(f"  Raw METAR: {raw_text}")

    conditions = parse_conditions(metar)
    wind_speed = conditions["wind_speed"]
    wind_dir   = conditions["wind_dir"]

    # Airport / runway info
    apt_info = fetch_airport_info(icao)

    if apt_info:
        towered  = apt_info["towered"]
        apt_name = apt_info["name"]
        apt_type = apt_info["type"]
        runways  = apt_info["runways"]
        print(f"\n  Airport: {apt_name}  ({apt_type})")
    else:
        towered  = None
        apt_name = icao
        runways  = []

    if towered is True:
        print(f"  ✈  TOWERED — ATC will assign your runway.")
    elif towered is False:
        print(f"  📻  UNTOWERED — you choose your runway.")
    else:
        print(f"  ℹ  Tower status unknown.")

    # Runway selection
    if runways:
        ranked = rank_runways(runways, wind_speed, wind_dir)

        lbl = "Runway wind breakdown" if towered else "Runway recommendations"
        print(f"\n  {lbl} (wind {wind_dir:03d}° @ {wind_speed} kt):")
        print(f"  {'Runway':<8} {'Hdg':>5}   {'Crosswind':>10}  {'Headwind':>10}")
        print(f"  {'-'*8}  {'-'*5}   {'-'*9}  {'-'*9}")
        for name, cx, hw, hdg in ranked:
            tailwind = " ⚠ tailwind!" if hw < 0 else ""
            marker   = "  ← BEST" if (not towered and name == ranked[0][0]) else ""
            print(f"  {name:<8} {hdg:>5.0f}°  {cx:>8.1f} kt  {hw:>+8.1f} kt{tailwind}{marker}")

        if not towered:
            # Auto-recommend best runway
            best_name, _, _, best_hdg = ranked[0]
            use_best = input(f"\n  Use recommended runway {best_name}? [Y/n]: ").strip().lower()
            if use_best in ("", "y", "yes"):
                runway_hdg, runway_name = best_hdg, best_name
            else:
                names = [r[0] for r in ranked]
                chosen = input(f"  Enter runway {names}: ").strip().upper()
                match = next((r for r in ranked if r[0] == chosen), None)
                if match:
                    runway_hdg, runway_name = match[3], match[0]
                else:
                    runway_hdg  = float(input("  Runway magnetic heading: ").strip() or "360")
                    runway_name = chosen or "manual"
        else:
            # Towered — ATC assigns; user enters expected runway
            rwy_input = input("\n  Enter assigned/expected runway (name or heading): ").strip().upper()
            match = next((r for r in ranked if r[0] == rwy_input), None)
            if match:
                runway_hdg, runway_name = match[3], match[0]
            else:
                try:
                    runway_hdg  = float(rwy_input)
                    runway_name = rwy_input
                except ValueError:
                    runway_hdg, runway_name = 360, "unknown"
    else:
        print("\n  No runway data found — entering heading manually.")
        runway_input = input("  Planned runway heading (magnetic, e.g. 270): ").strip()
        runway_hdg   = float(runway_input) if runway_input else 360
        runway_name  = runway_input or "360"

    # Assessment
    result = display(icao, apt_name, conditions, limits, runway_hdg, runway_name, profile_label, role)
    return result

# ── Main ──────────────────────────────────────────────────────────────────────

def main():
    sep = "=" * 56
    print(sep)
    print("   STUDENT PILOT WEATHER CHECKER  (live METAR)")
    print(sep)

    departure_icao = input("\nEnter departure airport ICAO code (e.g. KPDK): ").strip().upper()

    # ── Flight profile + optional destination ──
    limits, profile_label, dest_icao = get_limits()

    # ── Departure airport assessment ──
    dep_result = assess_airport(departure_icao, limits, profile_label, role="DEPARTURE")

    # ── Destination airport assessment (cross country only) ──
    dest_result = None
    if dest_icao:
        dest_result = assess_airport(dest_icao, limits, profile_label, role="DESTINATION")

    # ── Final summary ──
    sep2 = "=" * 56
    print(f"\n{sep2}")
    is_xc = dest_icao and dest_result
    print("  CROSS COUNTRY FLIGHT SUMMARY" if is_xc else "  FLIGHT SUMMARY")
    print(sep2)

    airports_to_summarise = []
    if dep_result:
        airports_to_summarise.append((departure_icao, "Departure",   dep_result))
    if dest_result:
        airports_to_summarise.append((dest_icao,      "Destination", dest_result))

    for icao_code, role_label, res in airports_to_summarise:
        overall  = res["overall"]
        runway   = res["runway"]
        warnings = res["warnings"]
        icon = "✅" if overall == "GO" else ("❌" if overall == "NO-GO" else "⚠️ ")
        print(f"\n  {icon}  {role_label} — {icao_code}")
        print(f"     Runway: {runway}")
        print(f"     Overall: {overall}")
        if warnings:
            print("     Conditions outside minimums:")
            for w in warnings:
                print(f"       • {w}")
        else:
            print("     All conditions within minimums.")

    print()
    all_overall = [r["overall"] for _, _, r in airports_to_summarise if r]
    if all(o == "GO" for o in all_overall):
        print("  All conditions GO. Have a great flight — fly safe!")
    elif "NO-GO" in all_overall:
        print("  ❌  One or more NO-GO conditions. Do not depart.")
    else:
        print("  ⚠️   Marginal conditions present. Consult your instructor.")
    print(sep2)

if __name__ == "__main__":
    main()
