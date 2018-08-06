---
layout: post
title: Visualizing Strava activites with Folium and Jupyter Notebook
---

Few days ago I participated in my first race ever! It was short triathlon called [Slapský triatlon](https://slapsky-triatlon.webnode.cz/). My friend was talking me into it for almost two years. The swimming is 500m, cycling is 34km and running is 4.5km. I don't swim or run that much but I was pretty sure I can handle this. I was pretty confident with cycling which turned out to be a mistake. I was happy with the result overall. I wasn't last. I'm pretty satisfied with swimming and running, but I could push a bit harder on the bike. The result is that I have some interesting data I can play around.

<iframe src="/public/slapsky-triatlon.html" style="width: 100%; height: 400px;border:none;"></iframe>
<p style="text-align:center;color: #555;font-size:0.8rem;">Speed visualization of the cycling part. Ranging from deep blue for slowest to deep red for fastest.</p>

When exploring the data I always use Jupyter Notebook and Folium. Folium lets you generate `leaflet.js` maps directly from Python.

Getting data from Strava is pretty simple. First, you need your Access Token. You can get them in [My API Application](https://www.strava.com/settings/api) section in Settings.

```python
from stravalib.client import Client
client = Client(access_token = <STRAVA_ACCESS_TOKEN>)

types = ['time', 'distance', 'latlng', 'altitude', 'velocity_smooth', 'moving', 'grade_smooth']
streams = client.get_activity_streams(<ACTIVITY_ID>, types=types)
```

Then to get stream data use `streams['altitude'].data`. To make resulting map look a bit more smooth we need to group similar velocity data points. This does pretty good job.

```python
locations = streams['latlng'].data
velocities = streams['velocity_smooth'].data

data = zip(locations, velocities)

groups = []
last_velocity = None
for location, velocity in data:
    if not last_velocity or abs(abs(last_velocity) - abs(velocity)) > 0.4:
        groups.append({'velocity': velocity, 'velocities': [], 'locations': []})
    groups[-1:][0]['locations'].append(location)
    groups[-1:][0]['velocities'].append(velocity)
    last_velocity = velocity

import statistics

for group in groups:
    group['velocity'] = statistics.median_high(group['velocities'])
```

To correctly colorize segments we need to normalize and map the values to colors. Luckily `matplotlib` does this for us with the exception of converting the color to hex. First, we will use `Normalize` to, well, normalize the value and then [colormap](https://matplotlib.org/tutorials/colors/colormaps.html) value to color. The last step is converting rgb 0.0 - 1.0 to hex.

```python
def convert_to_hex(rgba_color):
    red = str(hex(int(rgba_color[0]*255)))[2:].capitalize()
    green = str(hex(int(rgba_color[1]*255)))[2:].capitalize()
    blue = str(hex(int(rgba_color[2]*255)))[2:].capitalize()

    if blue=='0':
        blue = '00'
    if red=='0':
        red = '00'
    if green=='0':
        green='00'

    return '#'+ red + green + blue

import matplotlib.cm as cm
from matplotlib.colors import Normalize

cmap = cm.bwr
norm = Normalize(vmin=min(velocities), vmax=max(velocities))

def colorize(grade):
    return convert_to_hex(cmap(norm(grade)))
```

With grouped data and correct color the only thing we need to do is generate `PolyLine`s for each group and display map.

```python
import folium

m = folium.Map(location=[49.7947117,14.3916288], tiles='Stamen Toner', zoom_start=12.2)

for group in groups:
    folium.PolyLine(
        group['locations'],
        weight=10,
        color=colorize(group['velocity'])
    ).add_to(m)

m
```

You can see the entire notebook on [nbviewer.jupyter.org](https://nbviewer.jupyter.org/github/stlk/strava-playground/blob/master/Strava%20route%20visualizer.ipynb). Here are my [ride](https://www.strava.com/activities/1732801109) and [run](https://www.strava.com/activities/1732871895) if you want to explore my activities on Strava.