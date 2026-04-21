[gpsTrackerInPythonV2-checkpoint.ipynb](https://github.com/user-attachments/files/26920259/gpsTrackerInPythonV2-checkpoint.ipynb)
{
 "cells": [
  {
   "cell_type": "code",
   "execution_count": 35,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      " * Serving Flask app '__main__'\n",
      " * Debug mode: off\n"
     ]
    },
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.\n",
      " * Running on http://127.0.0.1:5000\n",
      "Press CTRL+C to quit\n",
      "127.0.0.1 - - [21/Apr/2026 13:05:42] \"GET / HTTP/1.1\" 200 -\n"
     ]
    },
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Waiting for GPS...\n"
     ]
    },
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "127.0.0.1 - - [21/Apr/2026 13:05:45] \"GET /save?lat=1.3757305688987094&lon=103.75275917109461 HTTP/1.1\" 200 -\n"
     ]
    },
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Got location!\n",
      "\n",
      " Clean Location:\n",
      "Block 2, 1 Choa Chu Kang Grove\n"
     ]
    }
   ],
   "source": [
    "from flask import Flask, request\n",
    "import folium\n",
    "import webbrowser\n",
    "import os\n",
    "import threading\n",
    "import time\n",
    "import requests\n",
    "\n",
    "app = Flask(__name__)\n",
    "\n",
    "# This will store GPS from browser\n",
    "coords = {\"lat\": None, \"lon\": None}\n",
    "\n",
    "\n",
    "@app.route(\"/\")\n",
    "def home():\n",
    "    return \"\"\"\n",
    "    <h2>Getting your location...</h2>\n",
    "\n",
    "    <script>\n",
    "        navigator.geolocation.getCurrentPosition(function(position) {\n",
    "            fetch(`/save?lat=${position.coords.latitude}&lon=${position.coords.longitude}`)\n",
    "            .then(() => {\n",
    "                document.body.innerHTML = \"Location received! You can close this tab.\";\n",
    "            });\n",
    "        });\n",
    "    </script>\n",
    "    \"\"\"\n",
    "\n",
    "\n",
    "@app.route(\"/save\")\n",
    "def save():\n",
    "    global coords\n",
    "    coords[\"lat\"] = float(request.args.get(\"lat\"))\n",
    "    coords[\"lon\"] = float(request.args.get(\"lon\"))\n",
    "    return \"OK\"\n",
    "\n",
    "\n",
    "def get_clean_address(lat, lon):\n",
    "    url = \"https://nominatim.openstreetmap.org/reverse\"\n",
    "    params = {\n",
    "        \"lat\": lat,\n",
    "        \"lon\": lon,\n",
    "        \"format\": \"json\",\n",
    "        \"addressdetails\": 1\n",
    "    }\n",
    "    headers = {\n",
    "        \"User-Agent\": \"gps-app\"\n",
    "    }\n",
    "\n",
    "    response = requests.get(url, params=params, headers=headers)\n",
    "    data = response.json()\n",
    "\n",
    "    address = data.get(\"address\", {})\n",
    "\n",
    "    building = address.get(\"building\")\n",
    "    house = address.get(\"house_number\")\n",
    "    road = address.get(\"road\")\n",
    "\n",
    "    parts = []\n",
    "\n",
    "    if building:\n",
    "        parts.append(building)\n",
    "\n",
    "    if house and road:\n",
    "        parts.append(f\"{house} {road}\")\n",
    "    elif road:\n",
    "        parts.append(road)\n",
    "\n",
    "    return \", \".join(parts) if parts else data.get(\"display_name\", \"Unknown\")\n",
    "\n",
    "\n",
    "def create_map(lat, lon):\n",
    "    address = get_clean_address(lat, lon)\n",
    "\n",
    "    m = folium.Map(location=[lat, lon], zoom_start=15)\n",
    "    folium.Marker([lat, lon], popup=address).add_to(m)\n",
    "\n",
    "    file_path = os.path.abspath(\"real_location_map.html\")\n",
    "    m.save(file_path)\n",
    "\n",
    "    webbrowser.open(\"file:///\" + file_path)\n",
    "\n",
    "\n",
    "def run_server():\n",
    "    app.run(port=5000)\n",
    "\n",
    "\n",
    "if __name__ == \"__main__\":\n",
    "    # Run Flask server in background\n",
    "    threading.Thread(target=run_server).start()\n",
    "\n",
    "    # Open browser GPS page\n",
    "    webbrowser.open(\"http://127.0.0.1:5000/\")\n",
    "\n",
    "    # Wait for user location\n",
    "    print(\"Waiting for GPS...\")\n",
    "\n",
    "    while coords[\"lat\"] is None:\n",
    "        time.sleep(1)\n",
    "\n",
    "    print(\"Got location!\")\n",
    "\n",
    "    # Get clean address\n",
    "    address = get_clean_address(coords[\"lat\"], coords[\"lon\"])\n",
    "\n",
    "    print(\"\\n Clean Location:\")\n",
    "    print(address)\n",
    "\n",
    "    create_map(coords[\"lat\"], coords[\"lon\"])"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3 (ipykernel)",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.11.7"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 4
}
