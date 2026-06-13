# Solar-System-Navigator
# 1. Problem Understanding
When building a digital solar system, developers typically encounter two conflicting goals: **Visual Clarity and Scientific Accuracy**.
- **The Scale Paradox**: True astronomical scales are impossible to view interactively. If the Sun is sized accurately, the planets are invisible sub-pixels; if orbital distances are realistic, the inner planets clump into an indistinguishable center dot.
- **The Flat Plane Myth**: Most standard diagrams depict the solar system as perfectly flat. In reality, every planet orbits on a slightly different geometric plane, tilted relative to Earth's orbital path (the ecliptic plane).
- **The Stale Data Problem**: Hardcoded orbital math merely guesses where planets are based on arbitrary loops. True situational awareness requires real-time ephemeris coordinates from space agencies.
## The Solution Strategy
The uploaded ecosystem elegant settles these conflicts through a decoupled architecture:
1. Scale Distortions: It scales planet radii and orbital spacing logarithmically (`ORBIT_LAYOUT`) to ensure a smooth user interface, while preserving exact real-world ratios for inclination angles and relative speeds.
2. Live State Fallback: It utilizes a python-based background synchronization engine to cache real planet coordinates, falling back smoothly to calculated orbits if an internet connection or native server context is absent.

# 2. Technical Decomposition & Architecture
## Layer A: The Python Ephemeris Extractor (fetch_planets.py)
This script acts as the automated data pipeline. It leverages `requests` and `re` (Regular Expressions) to automate interaction with the NASA JPL Horizons API using explicit configuration parameters:
- `CENTER='500@10'`: Sets the coordinate origin precisely at the Center of the Sun.
- `REF_PLANE='ECLIPTIC'`: Ensures all rectangular vector assignments (_X_,_Y_,_Z_) align directly with Earth's orbital plane.
- `VEC_TABLE='2'`: Requests raw state vectors (position components measured natively in kilometers).
The script isolates data sandwiched between NASA's traditional plain-text formatting blocks (`$$SOE` / `$$EOE`), converts raw kilometers to Astronomical Units (1 AU ≈ 149,597,870.7 km), and calculates the real-time distance vector via the Pythagorean theorem in 3D space:

$distance au = \sqrt{x^2 + y^2 + z^2}$
## Layer B: The Structured Data Cache (data.json)
Acts as a lightweight data contract between backend extraction and frontend presentation. It stores static structural metadata (planet colors, diameters, educational fun facts, and moon counts) alongside the dynamic positioning values (`x_au`, `y_au`, `z_au`, `distance_au`) freshly calculated by the Python script.
## Layer C: The Interactive 3D Canvas (solar_system.html)
The frontend rendering engine uses the Three.js WebGL library to generate a fluid, hardware-accelerated viewport.
- **Starfields & Lighting**: Utilizes an asynchronous `BufferGeometry` array holding 9,000 spatial points to draw stars, a high-intensity central `PointLight` acting as the Sun, and a soft `AmbientLight` to avoid pitch-black unlit planet faces.
- **Camera Controls**: Implements a custom mathematical orbital camera system via pointer event listeners (`mousedown`, `mousemove`, `wheel`, `touch`). It maps raw pixel shifts to spherical coordinate angles (_θ_ and _ϕ_), providing fluid mouse drags and scrolling zooms without requiring heavy external add-ons.
