# Solar-System-Navigator
I created a model of solar system using HTML and python.
I used HTML and Python for the majority of the code.
The model used a real-time planet position from NASA's Horizon API. The rest of it is just constants (Orbit speed, diameter, distance from the sun, inclination, etc).

# How It Works
It's rather simple. I just use the appropriate parameter that is in Horizon API and tell Python to grab it at **fetch_planets.py**. It then export the data as **data.json**. Lastly, **solar_system.html** will grab the data and calculate the position in real-time.

# The Math
- Circular motion
- Euler-lagrange Solution (Equation of Motion)
- Newton's Law of Universal Gravitation
