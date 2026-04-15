<p style="text-align:center">
    <a href="https://skills.network/?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01" target="_blank">
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/assets/logos/SN_web_lightmode.png" width="200" alt="Skills Network Logo"  />
    </a>
</p>


# **Launch Sites Locations Analysis with Folium**


Estimated time needed: **40** minutes


The launch success rate may depend on many factors such as payload mass, orbit type, and so on. It may also depend on the location and proximities of a launch site, i.e., the initial position of rocket trajectories. Finding an optimal location for building a launch site certainly involves many factors and hopefully we could discover some of the factors by analyzing the existing launch site locations.


In the previous exploratory data analysis labs, you have visualized the SpaceX launch dataset using `matplotlib` and `seaborn` and discovered some preliminary correlations between the launch site and success rates. In this lab, you will be performing more interactive visual analytics using `Folium`.


## Objectives


This lab contains the following tasks:

*   **TASK 1:** Mark all launch sites on a map
*   **TASK 2:** Mark the success/failed launches for each site on the map
*   **TASK 3:** Calculate the distances between a launch site to its proximities

After completed the above tasks, you should be able to find some geographical patterns about launch sites.


Let's first import required Python packages for this lab:



```python
!pip3 install folium
!pip3 install wget
```

    Requirement already satisfied: folium in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (0.11.0)
    Requirement already satisfied: numpy in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from folium) (1.21.6)
    Requirement already satisfied: jinja2>=2.9 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from folium) (3.1.2)
    Requirement already satisfied: requests in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from folium) (2.28.1)
    Requirement already satisfied: branca>=0.3.0 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from folium) (0.6.0)
    Requirement already satisfied: MarkupSafe>=2.0 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from jinja2>=2.9->folium) (2.1.1)
    Requirement already satisfied: charset-normalizer<3,>=2 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from requests->folium) (2.1.1)
    Requirement already satisfied: certifi>=2017.4.17 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from requests->folium) (2022.9.24)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from requests->folium) (1.26.13)
    Requirement already satisfied: idna<4,>=2.5 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from requests->folium) (3.4)
    Requirement already satisfied: wget in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (3.2)



```python
import folium
import wget
import pandas as pd
import numpy as np
```


```python
# Import folium MarkerCluster plugin
from folium.plugins import MarkerCluster
# Import folium MousePosition plugin
from folium.plugins import MousePosition
# Import folium DivIcon plugin
from folium.features import DivIcon
```

If you need to refresh your memory about folium, you may download and refer to this previous folium lab:


[Generating Maps with Python](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module\_3/DV0101EN-3-5-1-Generating-Maps-in-Python-py-v2.0.ipynb)


## Task 1: Mark all launch sites on a map


First, let's try to add each site's location on a map using site's latitude and longitude coordinates


The following dataset with the name `spacex_launch_geo.csv` is an augmented dataset with latitude and longitude added for each site.



```python
# Download and read the `spacex_launch_geo.csv`
spacex_csv_file = wget.download('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/datasets/spacex_launch_geo.csv')
spacex_df=pd.read_csv(spacex_csv_file)
```

Now, you can take a look at what are the coordinates for each site.



```python
# Select relevant sub-columns: `Launch Site`, `Lat(Latitude)`, `Long(Longitude)`, `class`
spacex_df = spacex_df[['Launch Site', 'Lat', 'Long', 'class']]
launch_sites_df = spacex_df.groupby(['Launch Site'], as_index=False).first()
launch_sites_df = launch_sites_df[['Launch Site', 'Lat', 'Long']]
launch_sites_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Launch Site</th>
      <th>Lat</th>
      <th>Long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CCAFS LC-40</td>
      <td>28.562302</td>
      <td>-80.577356</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
    </tr>
    <tr>
      <th>2</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
    </tr>
    <tr>
      <th>3</th>
      <td>VAFB SLC-4E</td>
      <td>34.632834</td>
      <td>-120.610745</td>
    </tr>
  </tbody>
</table>
</div>



Above coordinates are just plain numbers that can not give you any intuitive insights about where are those launch sites. If you are very good at geography, you can interpret those numbers directly in your mind. If not, that's fine too. Let's visualize those locations by pinning them on a map.


We first need to create a folium `Map` object, with an initial center location to be NASA Johnson Space Center at Houston, Texas.



```python
# Start location is NASA Johnson Space Center
nasa_coordinate = [29.559684888503615, -95.0830971930759]
site_map = folium.Map(location=nasa_coordinate, zoom_start=10)
```

We could use `folium.Circle` to add a highlighted circle area with a text label on a specific coordinate. For example,



```python
# Create a blue circle at NASA Johnson Space Center's coordinate with a popup label showing its name
circle = folium.Circle(nasa_coordinate, radius=1000, color='#d35400', fill=True).add_child(folium.Popup('NASA Johnson Space Center'))
# Create a blue circle at NASA Johnson Space Center's coordinate with a icon showing its name
marker = folium.map.Marker(
    nasa_coordinate,
    # Create an icon as a text label
    icon=DivIcon(
        icon_size=(20,20),
        icon_anchor=(0,0),
        html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % 'NASA JSC',
        )
    )
site_map.add_child(circle)
site_map.add_child(marker)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/python-visualization/folium/master/folium/templates/leaflet.awesome.rotate.css&quot;/&gt;
    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_a612c59bc0214b0dd1013da8f0a9bdb4 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_a612c59bc0214b0dd1013da8f0a9bdb4&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_a612c59bc0214b0dd1013da8f0a9bdb4 = L.map(
                &quot;map_a612c59bc0214b0dd1013da8f0a9bdb4&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_058b11bcf74374fcfdf68f52690842d1 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_a612c59bc0214b0dd1013da8f0a9bdb4);


            var circle_a7f2674993b3787e886be61ed55aba45 = L.circle(
                [29.559684888503615, -95.0830971930759],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_a612c59bc0214b0dd1013da8f0a9bdb4);


        var popup_8c1281471abc71417ae3dff92c0577e0 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_e59bbfa5374fa3102ab7049e52087e7a = $(`&lt;div id=&quot;html_e59bbfa5374fa3102ab7049e52087e7a&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;NASA Johnson Space Center&lt;/div&gt;`)[0];
            popup_8c1281471abc71417ae3dff92c0577e0.setContent(html_e59bbfa5374fa3102ab7049e52087e7a);


        circle_a7f2674993b3787e886be61ed55aba45.bindPopup(popup_8c1281471abc71417ae3dff92c0577e0)
        ;




            var marker_4f7d503852dcb967a58e577be06edfd4 = L.marker(
                [29.559684888503615, -95.0830971930759],
                {}
            ).addTo(map_a612c59bc0214b0dd1013da8f0a9bdb4);


            var div_icon_a0821fa0eb36b89370cbfc4f775706a2 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eNASA JSC\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_4f7d503852dcb967a58e577be06edfd4.setIcon(div_icon_a0821fa0eb36b89370cbfc4f775706a2);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



and you should find a small yellow circle near the city of Houston and you can zoom-in to see a larger circle.


Now, let's add a circle for each launch site in data frame `launch_sites`


*TODO:*  Create and add `folium.Circle` and `folium.Marker` for each launch site on the site map


An example of folium.Circle:


`folium.Circle(coordinate, radius=1000, color='#000000', fill=True).add_child(folium.Popup(...))`


An example of folium.Marker:


`folium.map.Marker(coordinate, icon=DivIcon(icon_size=(20,20),icon_anchor=(0,0), html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % 'label', ))`



```python
# Initial the map
site_map = folium.Map(location=nasa_coordinate, zoom_start=5)
# For each launch site, add a Circle object based on its coordinate (Lat, Long) values. In addition, add Launch site name as a popup label

```


```python
#Rewrite site map variable to avoid overlapping from the previous 
site_map2 = folium.Map(location=nasa_coordinate,zoom_start=5)
# instantiate a feature group for the launchsites in the dataframe
launchsite = folium.map.FeatureGroup()

# Assign for each Latitude and Longitude for the launchsites 

LAT = launch_sites_df['Lat']
LONG = launch_sites_df['Long']
LABELS = launch_sites_df['Launch Site']

# loop through the launchsites plus add a circle and label
for lat, lng, label in zip(LAT, LONG, LABELS):
    launchsite.add_child(
        folium.Circle(
            [lat, lng],
            radius=1000,  # define how big you want the circle markers to be
            color='#d35400',
            fill=True
        ).add_child(folium.Popup('NASA Johnson Space Center'))
    )
    launchsite.add_child(folium.map.Marker(
        [lat, lng], icon=DivIcon(icon_size=(20, 20),
                            icon_anchor = (0, 0),
                                html='%s'
                                % label,)))

site_map2.add_child(launchsite)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/python-visualization/folium/master/folium/templates/leaflet.awesome.rotate.css&quot;/&gt;
    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_fc3113efcd51f7b9567c90e090fc4c92 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_fc3113efcd51f7b9567c90e090fc4c92&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_fc3113efcd51f7b9567c90e090fc4c92 = L.map(
                &quot;map_fc3113efcd51f7b9567c90e090fc4c92&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_176762ff9a42af54bc3876f62eb82055 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_fc3113efcd51f7b9567c90e090fc4c92);


            var feature_group_970df7129cef5f50a5c36d7db31084e0 = L.featureGroup(
                {}
            ).addTo(map_fc3113efcd51f7b9567c90e090fc4c92);


            var circle_6f1925fcb31e48c3a78d193a14abedb9 = L.circle(
                [28.56230197, -80.57735648],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(feature_group_970df7129cef5f50a5c36d7db31084e0);


        var popup_a3bf20e78a3e898fbf8851b4f3c02fae = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_cf257ab6f1596c19f3fc18caae771ec9 = $(`&lt;div id=&quot;html_cf257ab6f1596c19f3fc18caae771ec9&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;NASA Johnson Space Center&lt;/div&gt;`)[0];
            popup_a3bf20e78a3e898fbf8851b4f3c02fae.setContent(html_cf257ab6f1596c19f3fc18caae771ec9);


        circle_6f1925fcb31e48c3a78d193a14abedb9.bindPopup(popup_a3bf20e78a3e898fbf8851b4f3c02fae)
        ;




            var marker_61c957e181c1b84d4d375d4b0e5c85c8 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(feature_group_970df7129cef5f50a5c36d7db31084e0);


            var div_icon_ae06edcc791b58bd1f07fb2d071237fe = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;CCAFS LC-40&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_61c957e181c1b84d4d375d4b0e5c85c8.setIcon(div_icon_ae06edcc791b58bd1f07fb2d071237fe);


            var circle_dc6b25016fecaa7527a3ae57ac00622d = L.circle(
                [28.56319718, -80.57682003],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(feature_group_970df7129cef5f50a5c36d7db31084e0);


        var popup_701fc191a0d1e126cdf662a7892a082b = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_51be8d9f28091ea03389ca10d0fccba5 = $(`&lt;div id=&quot;html_51be8d9f28091ea03389ca10d0fccba5&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;NASA Johnson Space Center&lt;/div&gt;`)[0];
            popup_701fc191a0d1e126cdf662a7892a082b.setContent(html_51be8d9f28091ea03389ca10d0fccba5);


        circle_dc6b25016fecaa7527a3ae57ac00622d.bindPopup(popup_701fc191a0d1e126cdf662a7892a082b)
        ;




            var marker_c69dc87e7dd0e6eff605924d8d3ffc6c = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(feature_group_970df7129cef5f50a5c36d7db31084e0);


            var div_icon_d1a435a9662abcf476e48735f61dc164 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;CCAFS SLC-40&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_c69dc87e7dd0e6eff605924d8d3ffc6c.setIcon(div_icon_d1a435a9662abcf476e48735f61dc164);


            var circle_a1b6f6d789e3e9a06a8862b15a55e3e5 = L.circle(
                [28.57325457, -80.64689529],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(feature_group_970df7129cef5f50a5c36d7db31084e0);


        var popup_be64cf9668d9614d6055074c6a2f9f9b = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_4f69004ffe397da881939b8d74a06d33 = $(`&lt;div id=&quot;html_4f69004ffe397da881939b8d74a06d33&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;NASA Johnson Space Center&lt;/div&gt;`)[0];
            popup_be64cf9668d9614d6055074c6a2f9f9b.setContent(html_4f69004ffe397da881939b8d74a06d33);


        circle_a1b6f6d789e3e9a06a8862b15a55e3e5.bindPopup(popup_be64cf9668d9614d6055074c6a2f9f9b)
        ;




            var marker_bf6770271948e7b48f3c69923f384586 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(feature_group_970df7129cef5f50a5c36d7db31084e0);


            var div_icon_cd76b45bb3f45133cda3eeb63680d57d = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;KSC LC-39A&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_bf6770271948e7b48f3c69923f384586.setIcon(div_icon_cd76b45bb3f45133cda3eeb63680d57d);


            var circle_62a224cba67a28b22dbb17ba2380f3c9 = L.circle(
                [34.63283416, -120.6107455],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(feature_group_970df7129cef5f50a5c36d7db31084e0);


        var popup_be3a77216c3edf63c056f66b6e743153 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_c8fd3033f5429b20919c7d84bd4dc8d8 = $(`&lt;div id=&quot;html_c8fd3033f5429b20919c7d84bd4dc8d8&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;NASA Johnson Space Center&lt;/div&gt;`)[0];
            popup_be3a77216c3edf63c056f66b6e743153.setContent(html_c8fd3033f5429b20919c7d84bd4dc8d8);


        circle_62a224cba67a28b22dbb17ba2380f3c9.bindPopup(popup_be3a77216c3edf63c056f66b6e743153)
        ;




            var marker_f3069999d75dbfc1a87116f899903972 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(feature_group_970df7129cef5f50a5c36d7db31084e0);


            var div_icon_f5d6e00ff07f812c995b2f3e34504beb = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;VAFB SLC-4E&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_f3069999d75dbfc1a87116f899903972.setIcon(div_icon_f5d6e00ff07f812c995b2f3e34504beb);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



The generated map with marked launch sites should look similar to the following:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/launch_site_markers.png" />
</center>


Now, you can explore the map by zoom-in/out the marked areas
, and try to answer the following questions:

*   Are all launch sites in proximity to the Equator line?
*   Are all launch sites in very close proximity to the coast?

Also please try to explain your findings.


### Q1) No, As it appears that the one of the launch sites sits closer CCAFS LC-40.
### Q2) Yes, it seems that all these launch sites are very close to the coast

# Task 2: Mark the success/failed launches for each site on the map


Next, let's try to enhance the map by adding the launch outcomes for each site, and see which sites have high success rates.
Recall that data frame spacex_df has detailed launch records, and the `class` column indicates if this launch was successful or not



```python
spacex_df.tail(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Launch Site</th>
      <th>Lat</th>
      <th>Long</th>
      <th>class</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>46</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
    </tr>
    <tr>
      <th>47</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
    </tr>
    <tr>
      <th>48</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
    </tr>
    <tr>
      <th>49</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
    </tr>
    <tr>
      <th>50</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
    </tr>
    <tr>
      <th>51</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
    </tr>
    <tr>
      <th>52</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
    </tr>
    <tr>
      <th>53</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
    </tr>
    <tr>
      <th>54</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
    </tr>
    <tr>
      <th>55</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Next, let's create markers for all launch records.
If a launch was successful `(class=1)`, then we use a green marker and if a launch was failed, we use a red marker `(class=0)`


Note that a launch only happens in one of the four launch sites, which means many launch records will have the exact same coordinate. Marker clusters can be a good way to simplify a map containing many markers having the same coordinate.


Let's first create a `MarkerCluster` object



```python
marker_cluster = MarkerCluster()
marker_cluster
```




    <folium.plugins.marker_cluster.MarkerCluster at 0x7f84509f2c50>



*TODO:* Create a new column in `launch_sites` dataframe called `marker_color` to store the marker colors based on the `class` value



```python

# Apply a function to check the value of `class` column
# If class=1, marker_color value will be green
# If class=0, marker_color value will be red
# df['marker_color'] = np.where(spacex_df['Class'] = '1', 'green', 'red')
"""
def colorClass(row):
    if row['class'] == 1:
        val = 'green'
    elif row['class'] == 0:
        val = 'red'
    else:
        val = X
    return val

spacex_df['marker_color'] = spacex_df.apply(colorClass, axis=1)

spacex_df['marker_color']
"""
spacex_df['marker_color'] = np.where(spacex_df['class'] == 1, 'green', np.where(spacex_df['class'] == 0, 'red', -1))
```


```python
# Function to assign color to launch outcome
def assign_marker_color(launch_outcome):
    if launch_outcome == 1:
        return 'green'
    else:
        return 'red'
    
spacex_df['marker_color'] = spacex_df['class'].apply(assign_marker_color)
spacex_df.tail(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Launch Site</th>
      <th>Lat</th>
      <th>Long</th>
      <th>class</th>
      <th>marker_color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>46</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>47</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>48</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>49</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>50</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>51</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
      <td>red</td>
    </tr>
    <tr>
      <th>52</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
      <td>red</td>
    </tr>
    <tr>
      <th>53</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
      <td>red</td>
    </tr>
    <tr>
      <th>54</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>55</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
      <td>red</td>
    </tr>
  </tbody>
</table>
</div>



*TODO:* For each launch result in `spacex_df` data frame, add a `folium.Marker` to `marker_cluster`



```python
# Add marker_cluster to current site_map
site_map.add_child(marker_cluster)

# for each row in spacex_df data frame
# create a Marker object with its coordinate
# and customize the Marker's icon property to indicate if this launch was successed or failed, 
# e.g., icon=folium.Icon(color='white', icon_color=row['marker_color']
for index, record in spacex_df.iterrows():
    # TODO: Create and add a Marker cluster to the site map
    launch_nasa_coordinate = record[['Lat','Long']].values
    icon_settings=folium.Icon(color='white', icon_color=record['marker_color'])
    # marker = folium.Marker(...)
    marker = folium.Marker(launch_nasa_coordinate, icon=icon_settings)
    marker_cluster.add_child(marker)

site_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/python-visualization/folium/master/folium/templates/leaflet.awesome.rotate.css&quot;/&gt;
    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_53ba2e67ab4671683422076420748d4c {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_53ba2e67ab4671683422076420748d4c&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_53ba2e67ab4671683422076420748d4c = L.map(
                &quot;map_53ba2e67ab4671683422076420748d4c&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_f59ae0101471d408b57a22b16b5736d0 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_cluster_941ec86d95673aae547cfec5fb548fda = L.markerClusterGroup(
                {}
            );
            map_53ba2e67ab4671683422076420748d4c.addLayer(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var marker_0a23aa93bcef11785869419872ccfa5d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4a8d02985f5e937b3152954c68e7bdb6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0a23aa93bcef11785869419872ccfa5d.setIcon(icon_4a8d02985f5e937b3152954c68e7bdb6);


            var marker_7875dde0bfbdebbeed55a08fac2cdc7f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6e26c65a13aa8211a2f414f575e19866 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7875dde0bfbdebbeed55a08fac2cdc7f.setIcon(icon_6e26c65a13aa8211a2f414f575e19866);


            var marker_ce95b05cb9db3540b5325eb44c2ac02b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ddbe42d5dd0263ee528890917c294ce = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ce95b05cb9db3540b5325eb44c2ac02b.setIcon(icon_9ddbe42d5dd0263ee528890917c294ce);


            var marker_55e8409872774d0c01bfe5787bcead78 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52e73e14e134d2a719095f3d70983f9d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_55e8409872774d0c01bfe5787bcead78.setIcon(icon_52e73e14e134d2a719095f3d70983f9d);


            var marker_dfea4fe8af7d7d67d79336d83d010f8e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_60f2958bb5bcaffcc68bea86fc3cb292 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfea4fe8af7d7d67d79336d83d010f8e.setIcon(icon_60f2958bb5bcaffcc68bea86fc3cb292);


            var marker_92d8aeb54828ef020d7cff63d5795732 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ad4fab9878aba9c0265c72bd72de510 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_92d8aeb54828ef020d7cff63d5795732.setIcon(icon_9ad4fab9878aba9c0265c72bd72de510);


            var marker_dc0e69183b9b31603d65fc5cd9deee43 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_fcfe1289cb844b5d69165e7769d8198f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dc0e69183b9b31603d65fc5cd9deee43.setIcon(icon_fcfe1289cb844b5d69165e7769d8198f);


            var marker_6769999948cb97ada4bab81bc5d2a14d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f9332b6f0ed640fbeeca149509e3c50f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6769999948cb97ada4bab81bc5d2a14d.setIcon(icon_f9332b6f0ed640fbeeca149509e3c50f);


            var marker_07d14173bf823cf17d3c2a4c8cfbbab7 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ea1a8439b795e87217ee4a1f9bb403b0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_07d14173bf823cf17d3c2a4c8cfbbab7.setIcon(icon_ea1a8439b795e87217ee4a1f9bb403b0);


            var marker_6fd06eaed04ac7ebe1aa20349ff53162 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bd127eb14a87d7e91f1cc84c7387c91b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fd06eaed04ac7ebe1aa20349ff53162.setIcon(icon_bd127eb14a87d7e91f1cc84c7387c91b);


            var marker_ec497dd6e102ee53fc1b5dec40a65f64 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_040aa65ec787091dce526203907f221a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ec497dd6e102ee53fc1b5dec40a65f64.setIcon(icon_040aa65ec787091dce526203907f221a);


            var marker_8159f289cbabc5fd5e5dc7026bc7d21b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d6e83c92d9d850067b6d7914dd40adc5 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8159f289cbabc5fd5e5dc7026bc7d21b.setIcon(icon_d6e83c92d9d850067b6d7914dd40adc5);


            var marker_40b11555ab6746af7bcacf050416caea = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_0aec047f0282befd188a74504272b391 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_40b11555ab6746af7bcacf050416caea.setIcon(icon_0aec047f0282befd188a74504272b391);


            var marker_512efd561a7ed33835e3cd9ca974b37c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3ed5417e90a7dab69fd03be6a555fe04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_512efd561a7ed33835e3cd9ca974b37c.setIcon(icon_3ed5417e90a7dab69fd03be6a555fe04);


            var marker_9df2741233008061c42e8981f6dd1879 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ad271e1fe93789b09b3b54f157374d6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9df2741233008061c42e8981f6dd1879.setIcon(icon_2ad271e1fe93789b09b3b54f157374d6);


            var marker_824928a2cea4ad038c847d1c483bfec9 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6f10aa1d799d6ffe076921d51c2ab74a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_824928a2cea4ad038c847d1c483bfec9.setIcon(icon_6f10aa1d799d6ffe076921d51c2ab74a);


            var marker_dcaf1d48a2366e4c755f41831ae92295 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5d7f18508e1f3f79b5959c5b3c34df9a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dcaf1d48a2366e4c755f41831ae92295.setIcon(icon_5d7f18508e1f3f79b5959c5b3c34df9a);


            var marker_90257c431160ea0d44d15a7b5e46e8d5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_14b5d92ac6eef21e0d48aa78b8892be9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_90257c431160ea0d44d15a7b5e46e8d5.setIcon(icon_14b5d92ac6eef21e0d48aa78b8892be9);


            var marker_941279f09478f8be5d9dfb7a586cacef = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e0534156851dc54df739c8d763e9252d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_941279f09478f8be5d9dfb7a586cacef.setIcon(icon_e0534156851dc54df739c8d763e9252d);


            var marker_20ccb58935670a91d7f8956bfe1927e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e161e7dadd4f412094a14165368039db = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20ccb58935670a91d7f8956bfe1927e5.setIcon(icon_e161e7dadd4f412094a14165368039db);


            var marker_c209d201ddb46f35a845816ab2f4d2c8 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5f27bb2c0d22f2107f8559b1379b1299 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c209d201ddb46f35a845816ab2f4d2c8.setIcon(icon_5f27bb2c0d22f2107f8559b1379b1299);


            var marker_169ab0e4cb67cbe85d44ab0c8251f0eb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4e03b968160eaf82ea68574c9624e8b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_169ab0e4cb67cbe85d44ab0c8251f0eb.setIcon(icon_c4e03b968160eaf82ea68574c9624e8b);


            var marker_60139c9ec27ef0250373bc4b8f984e84 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f4ce3a0946c467928197c676668e5dc3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_60139c9ec27ef0250373bc4b8f984e84.setIcon(icon_f4ce3a0946c467928197c676668e5dc3);


            var marker_e9be2ed2cfe754c94568b719bea80004 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d70e7e0896a7c489ca8dedf477267e05 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e9be2ed2cfe754c94568b719bea80004.setIcon(icon_d70e7e0896a7c489ca8dedf477267e05);


            var marker_3a183638a44a6b9cd65127f0bf6307fa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1ce558f8258123fe14864124b15e2084 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3a183638a44a6b9cd65127f0bf6307fa.setIcon(icon_1ce558f8258123fe14864124b15e2084);


            var marker_3947fdf0a867451fb19d913d4093c81b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_04d2d7fa4a6072318bf5a7b353c97631 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3947fdf0a867451fb19d913d4093c81b.setIcon(icon_04d2d7fa4a6072318bf5a7b353c97631);


            var marker_6fffd16f5ae730ceaa422d419d320576 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_28740f5f393435fb2003638408b71e58 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fffd16f5ae730ceaa422d419d320576.setIcon(icon_28740f5f393435fb2003638408b71e58);


            var marker_c1bf67569aa521702f3386fd9f50234a = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52cb509e069d15e22cf323de6506ad25 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c1bf67569aa521702f3386fd9f50234a.setIcon(icon_52cb509e069d15e22cf323de6506ad25);


            var marker_29d491e6de899c170425087fbb311383 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f72f377dc3330acc4b54350fa91c5716 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_29d491e6de899c170425087fbb311383.setIcon(icon_f72f377dc3330acc4b54350fa91c5716);


            var marker_72f39afb39c9f5f23aa60f16afdef230 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a0740b5c422d2f6578cc10694de88388 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_72f39afb39c9f5f23aa60f16afdef230.setIcon(icon_a0740b5c422d2f6578cc10694de88388);


            var marker_ac78816dda8465a19c4a995b73f33238 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_8fc291b6cc723535c0f964e257a972ad = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ac78816dda8465a19c4a995b73f33238.setIcon(icon_8fc291b6cc723535c0f964e257a972ad);


            var marker_279d3cee1c792691b5ffa75d4687f554 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4b2ee1983091d9cb26cbc028206ac092 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_279d3cee1c792691b5ffa75d4687f554.setIcon(icon_4b2ee1983091d9cb26cbc028206ac092);


            var marker_d9cb37142f9f6726330307870bc1308c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ab1134476094a2011e6974547527004c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9cb37142f9f6726330307870bc1308c.setIcon(icon_ab1134476094a2011e6974547527004c);


            var marker_57f0bae8fd13cdec70cf3336b64a911b = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bef1652752ad79d1f0322aed315e5777 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_57f0bae8fd13cdec70cf3336b64a911b.setIcon(icon_bef1652752ad79d1f0322aed315e5777);


            var marker_88ec1fbe61fb99bb535692d2b802fd5c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ac49c0fa53302d8846e7674f30b7479 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_88ec1fbe61fb99bb535692d2b802fd5c.setIcon(icon_2ac49c0fa53302d8846e7674f30b7479);


            var marker_556cd21ced97417d7a963d4392ac6a17 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_192e01d38cf9d0e572f15bb8997a41a6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_556cd21ced97417d7a963d4392ac6a17.setIcon(icon_192e01d38cf9d0e572f15bb8997a41a6);


            var marker_5e8ff3a48458f66287101ea5c224d39e = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_7cf89242f2929f573d286bc6fe320133 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5e8ff3a48458f66287101ea5c224d39e.setIcon(icon_7cf89242f2929f573d286bc6fe320133);


            var marker_984908df28561a3b343589c335b33cf8 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f58da7b1b64bf22a316e1013963a5f60 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_984908df28561a3b343589c335b33cf8.setIcon(icon_f58da7b1b64bf22a316e1013963a5f60);


            var marker_a4db0ad3b213a5023128d584ccf38fe7 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a195dca5aeb1c16776ac344b5beb9c5e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a4db0ad3b213a5023128d584ccf38fe7.setIcon(icon_a195dca5aeb1c16776ac344b5beb9c5e);


            var marker_c0908593b3cb8a7ae49d3e9a06da1368 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d1dd57d1b24ae5034cf9198133fa8e51 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0908593b3cb8a7ae49d3e9a06da1368.setIcon(icon_d1dd57d1b24ae5034cf9198133fa8e51);


            var marker_34daabbdbdd75fa5e55ea1e53114efa6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4c2b743c70f9a5fa5882d9266517bd7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34daabbdbdd75fa5e55ea1e53114efa6.setIcon(icon_c4c2b743c70f9a5fa5882d9266517bd7);


            var marker_bef2e91a44bed729403772a3038ca9f3 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_717805283a95cff674edd47a94df2eec = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bef2e91a44bed729403772a3038ca9f3.setIcon(icon_717805283a95cff674edd47a94df2eec);


            var marker_6e36f26380dc73f855999301ee49cd16 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_72011afd61c6be9503a788ba150dbd4e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e36f26380dc73f855999301ee49cd16.setIcon(icon_72011afd61c6be9503a788ba150dbd4e);


            var marker_bded94e0806fa77f6b0c4a10affa8cae = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_655849f5b2768f6b7ab2e9d18503aefd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bded94e0806fa77f6b0c4a10affa8cae.setIcon(icon_655849f5b2768f6b7ab2e9d18503aefd);


            var marker_b1db39bf40f86fd0c8bc7f388365ca5c = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_dd6666c810ae59df2450bb495b3752fd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1db39bf40f86fd0c8bc7f388365ca5c.setIcon(icon_dd6666c810ae59df2450bb495b3752fd);


            var marker_44617e15bbef4ffd7cbfeb4bcc39c0b9 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1387879bbdcd524f4841c5b2b306e375 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_44617e15bbef4ffd7cbfeb4bcc39c0b9.setIcon(icon_1387879bbdcd524f4841c5b2b306e375);


            var marker_dbdfec5daa277d8df66495575aeecb89 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c77b6079543bc6d2f2530125adb14d0d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbdfec5daa277d8df66495575aeecb89.setIcon(icon_c77b6079543bc6d2f2530125adb14d0d);


            var marker_7c497e8a937df15b4a478dfee391fa38 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_57aaf5bd80d908a0b9b0e1e20189631c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7c497e8a937df15b4a478dfee391fa38.setIcon(icon_57aaf5bd80d908a0b9b0e1e20189631c);


            var marker_bf8d11c3a84bc01496efb5c26e0cd01b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c5d800da3c3cdcfa47de2c3474b3e7b3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bf8d11c3a84bc01496efb5c26e0cd01b.setIcon(icon_c5d800da3c3cdcfa47de2c3474b3e7b3);


            var marker_b1945b1bff32820e5774bb8307c3c9e2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_34c8f2c9d7cda03c16e1d5b9665abfe3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1945b1bff32820e5774bb8307c3c9e2.setIcon(icon_34c8f2c9d7cda03c16e1d5b9665abfe3);


            var marker_9c8d922308159b4b75f21d85df4d194e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e35d347a145b8bba035e00ee60c71454 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9c8d922308159b4b75f21d85df4d194e.setIcon(icon_e35d347a145b8bba035e00ee60c71454);


            var marker_544d04728ae5afbba83f6c0c90a7b104 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_314c8754a83ea323e7a1247e01edb917 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_544d04728ae5afbba83f6c0c90a7b104.setIcon(icon_314c8754a83ea323e7a1247e01edb917);


            var marker_59acf65b4ed29a3c4dbbb16161042f63 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_280f880ecf1c412d286eecc9debeefd1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_59acf65b4ed29a3c4dbbb16161042f63.setIcon(icon_280f880ecf1c412d286eecc9debeefd1);


            var marker_a85af35f3acafb816c34c4c9f2e998b9 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3656b5e5aaa32301bf5cb15e9066719f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a85af35f3acafb816c34c4c9f2e998b9.setIcon(icon_3656b5e5aaa32301bf5cb15e9066719f);


            var marker_c0f1e91c861dbe2f0069a83272b805d2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5bdcd6baff1025a96628cf40fefd8bfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0f1e91c861dbe2f0069a83272b805d2.setIcon(icon_5bdcd6baff1025a96628cf40fefd8bfd);


            var marker_f9c3660b4b31e3539f52a2f7db5af5b5 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_78cb4af7bffca3a49010a28ac71e37c2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f9c3660b4b31e3539f52a2f7db5af5b5.setIcon(icon_78cb4af7bffca3a49010a28ac71e37c2);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Your updated map may look like the following screenshots:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/launch_site_marker_cluster.png" />
</center>


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/launch_site_marker_cluster_zoomed.png" />
</center>


From the color-labeled markers in marker clusters, you should be able to easily identify which launch sites have relatively high success rates.


# TASK 3: Calculate the distances between a launch site to its proximities


Next, we need to explore and analyze the proximities of launch sites.


Let's first add a `MousePosition` on the map to get coordinate for a mouse over a point on the map. As such, while you are exploring the map, you can easily find the coordinates of any points of interests (such as railway)



```python
# Add Mouse Position to get the coordinate (Lat, Long) for a mouse over on the map
formatter = "function(num) {return L.Util.formatNum(num, 5);};"
mouse_position = MousePosition(
    position='topright',
    separator=' Long: ',
    empty_string='NaN',
    lng_first=False,
    num_digits=20,
    prefix='Lat:',
    lat_formatter=formatter,
    lng_formatter=formatter,
)

site_map.add_child(mouse_position)
site_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/python-visualization/folium/master/folium/templates/leaflet.awesome.rotate.css&quot;/&gt;
    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_53ba2e67ab4671683422076420748d4c {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_53ba2e67ab4671683422076420748d4c&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_53ba2e67ab4671683422076420748d4c = L.map(
                &quot;map_53ba2e67ab4671683422076420748d4c&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_f59ae0101471d408b57a22b16b5736d0 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_cluster_941ec86d95673aae547cfec5fb548fda = L.markerClusterGroup(
                {}
            );
            map_53ba2e67ab4671683422076420748d4c.addLayer(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var marker_0a23aa93bcef11785869419872ccfa5d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4a8d02985f5e937b3152954c68e7bdb6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0a23aa93bcef11785869419872ccfa5d.setIcon(icon_4a8d02985f5e937b3152954c68e7bdb6);


            var marker_7875dde0bfbdebbeed55a08fac2cdc7f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6e26c65a13aa8211a2f414f575e19866 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7875dde0bfbdebbeed55a08fac2cdc7f.setIcon(icon_6e26c65a13aa8211a2f414f575e19866);


            var marker_ce95b05cb9db3540b5325eb44c2ac02b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ddbe42d5dd0263ee528890917c294ce = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ce95b05cb9db3540b5325eb44c2ac02b.setIcon(icon_9ddbe42d5dd0263ee528890917c294ce);


            var marker_55e8409872774d0c01bfe5787bcead78 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52e73e14e134d2a719095f3d70983f9d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_55e8409872774d0c01bfe5787bcead78.setIcon(icon_52e73e14e134d2a719095f3d70983f9d);


            var marker_dfea4fe8af7d7d67d79336d83d010f8e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_60f2958bb5bcaffcc68bea86fc3cb292 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfea4fe8af7d7d67d79336d83d010f8e.setIcon(icon_60f2958bb5bcaffcc68bea86fc3cb292);


            var marker_92d8aeb54828ef020d7cff63d5795732 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ad4fab9878aba9c0265c72bd72de510 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_92d8aeb54828ef020d7cff63d5795732.setIcon(icon_9ad4fab9878aba9c0265c72bd72de510);


            var marker_dc0e69183b9b31603d65fc5cd9deee43 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_fcfe1289cb844b5d69165e7769d8198f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dc0e69183b9b31603d65fc5cd9deee43.setIcon(icon_fcfe1289cb844b5d69165e7769d8198f);


            var marker_6769999948cb97ada4bab81bc5d2a14d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f9332b6f0ed640fbeeca149509e3c50f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6769999948cb97ada4bab81bc5d2a14d.setIcon(icon_f9332b6f0ed640fbeeca149509e3c50f);


            var marker_07d14173bf823cf17d3c2a4c8cfbbab7 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ea1a8439b795e87217ee4a1f9bb403b0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_07d14173bf823cf17d3c2a4c8cfbbab7.setIcon(icon_ea1a8439b795e87217ee4a1f9bb403b0);


            var marker_6fd06eaed04ac7ebe1aa20349ff53162 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bd127eb14a87d7e91f1cc84c7387c91b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fd06eaed04ac7ebe1aa20349ff53162.setIcon(icon_bd127eb14a87d7e91f1cc84c7387c91b);


            var marker_ec497dd6e102ee53fc1b5dec40a65f64 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_040aa65ec787091dce526203907f221a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ec497dd6e102ee53fc1b5dec40a65f64.setIcon(icon_040aa65ec787091dce526203907f221a);


            var marker_8159f289cbabc5fd5e5dc7026bc7d21b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d6e83c92d9d850067b6d7914dd40adc5 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8159f289cbabc5fd5e5dc7026bc7d21b.setIcon(icon_d6e83c92d9d850067b6d7914dd40adc5);


            var marker_40b11555ab6746af7bcacf050416caea = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_0aec047f0282befd188a74504272b391 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_40b11555ab6746af7bcacf050416caea.setIcon(icon_0aec047f0282befd188a74504272b391);


            var marker_512efd561a7ed33835e3cd9ca974b37c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3ed5417e90a7dab69fd03be6a555fe04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_512efd561a7ed33835e3cd9ca974b37c.setIcon(icon_3ed5417e90a7dab69fd03be6a555fe04);


            var marker_9df2741233008061c42e8981f6dd1879 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ad271e1fe93789b09b3b54f157374d6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9df2741233008061c42e8981f6dd1879.setIcon(icon_2ad271e1fe93789b09b3b54f157374d6);


            var marker_824928a2cea4ad038c847d1c483bfec9 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6f10aa1d799d6ffe076921d51c2ab74a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_824928a2cea4ad038c847d1c483bfec9.setIcon(icon_6f10aa1d799d6ffe076921d51c2ab74a);


            var marker_dcaf1d48a2366e4c755f41831ae92295 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5d7f18508e1f3f79b5959c5b3c34df9a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dcaf1d48a2366e4c755f41831ae92295.setIcon(icon_5d7f18508e1f3f79b5959c5b3c34df9a);


            var marker_90257c431160ea0d44d15a7b5e46e8d5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_14b5d92ac6eef21e0d48aa78b8892be9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_90257c431160ea0d44d15a7b5e46e8d5.setIcon(icon_14b5d92ac6eef21e0d48aa78b8892be9);


            var marker_941279f09478f8be5d9dfb7a586cacef = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e0534156851dc54df739c8d763e9252d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_941279f09478f8be5d9dfb7a586cacef.setIcon(icon_e0534156851dc54df739c8d763e9252d);


            var marker_20ccb58935670a91d7f8956bfe1927e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e161e7dadd4f412094a14165368039db = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20ccb58935670a91d7f8956bfe1927e5.setIcon(icon_e161e7dadd4f412094a14165368039db);


            var marker_c209d201ddb46f35a845816ab2f4d2c8 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5f27bb2c0d22f2107f8559b1379b1299 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c209d201ddb46f35a845816ab2f4d2c8.setIcon(icon_5f27bb2c0d22f2107f8559b1379b1299);


            var marker_169ab0e4cb67cbe85d44ab0c8251f0eb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4e03b968160eaf82ea68574c9624e8b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_169ab0e4cb67cbe85d44ab0c8251f0eb.setIcon(icon_c4e03b968160eaf82ea68574c9624e8b);


            var marker_60139c9ec27ef0250373bc4b8f984e84 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f4ce3a0946c467928197c676668e5dc3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_60139c9ec27ef0250373bc4b8f984e84.setIcon(icon_f4ce3a0946c467928197c676668e5dc3);


            var marker_e9be2ed2cfe754c94568b719bea80004 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d70e7e0896a7c489ca8dedf477267e05 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e9be2ed2cfe754c94568b719bea80004.setIcon(icon_d70e7e0896a7c489ca8dedf477267e05);


            var marker_3a183638a44a6b9cd65127f0bf6307fa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1ce558f8258123fe14864124b15e2084 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3a183638a44a6b9cd65127f0bf6307fa.setIcon(icon_1ce558f8258123fe14864124b15e2084);


            var marker_3947fdf0a867451fb19d913d4093c81b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_04d2d7fa4a6072318bf5a7b353c97631 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3947fdf0a867451fb19d913d4093c81b.setIcon(icon_04d2d7fa4a6072318bf5a7b353c97631);


            var marker_6fffd16f5ae730ceaa422d419d320576 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_28740f5f393435fb2003638408b71e58 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fffd16f5ae730ceaa422d419d320576.setIcon(icon_28740f5f393435fb2003638408b71e58);


            var marker_c1bf67569aa521702f3386fd9f50234a = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52cb509e069d15e22cf323de6506ad25 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c1bf67569aa521702f3386fd9f50234a.setIcon(icon_52cb509e069d15e22cf323de6506ad25);


            var marker_29d491e6de899c170425087fbb311383 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f72f377dc3330acc4b54350fa91c5716 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_29d491e6de899c170425087fbb311383.setIcon(icon_f72f377dc3330acc4b54350fa91c5716);


            var marker_72f39afb39c9f5f23aa60f16afdef230 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a0740b5c422d2f6578cc10694de88388 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_72f39afb39c9f5f23aa60f16afdef230.setIcon(icon_a0740b5c422d2f6578cc10694de88388);


            var marker_ac78816dda8465a19c4a995b73f33238 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_8fc291b6cc723535c0f964e257a972ad = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ac78816dda8465a19c4a995b73f33238.setIcon(icon_8fc291b6cc723535c0f964e257a972ad);


            var marker_279d3cee1c792691b5ffa75d4687f554 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4b2ee1983091d9cb26cbc028206ac092 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_279d3cee1c792691b5ffa75d4687f554.setIcon(icon_4b2ee1983091d9cb26cbc028206ac092);


            var marker_d9cb37142f9f6726330307870bc1308c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ab1134476094a2011e6974547527004c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9cb37142f9f6726330307870bc1308c.setIcon(icon_ab1134476094a2011e6974547527004c);


            var marker_57f0bae8fd13cdec70cf3336b64a911b = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bef1652752ad79d1f0322aed315e5777 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_57f0bae8fd13cdec70cf3336b64a911b.setIcon(icon_bef1652752ad79d1f0322aed315e5777);


            var marker_88ec1fbe61fb99bb535692d2b802fd5c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ac49c0fa53302d8846e7674f30b7479 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_88ec1fbe61fb99bb535692d2b802fd5c.setIcon(icon_2ac49c0fa53302d8846e7674f30b7479);


            var marker_556cd21ced97417d7a963d4392ac6a17 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_192e01d38cf9d0e572f15bb8997a41a6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_556cd21ced97417d7a963d4392ac6a17.setIcon(icon_192e01d38cf9d0e572f15bb8997a41a6);


            var marker_5e8ff3a48458f66287101ea5c224d39e = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_7cf89242f2929f573d286bc6fe320133 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5e8ff3a48458f66287101ea5c224d39e.setIcon(icon_7cf89242f2929f573d286bc6fe320133);


            var marker_984908df28561a3b343589c335b33cf8 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f58da7b1b64bf22a316e1013963a5f60 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_984908df28561a3b343589c335b33cf8.setIcon(icon_f58da7b1b64bf22a316e1013963a5f60);


            var marker_a4db0ad3b213a5023128d584ccf38fe7 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a195dca5aeb1c16776ac344b5beb9c5e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a4db0ad3b213a5023128d584ccf38fe7.setIcon(icon_a195dca5aeb1c16776ac344b5beb9c5e);


            var marker_c0908593b3cb8a7ae49d3e9a06da1368 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d1dd57d1b24ae5034cf9198133fa8e51 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0908593b3cb8a7ae49d3e9a06da1368.setIcon(icon_d1dd57d1b24ae5034cf9198133fa8e51);


            var marker_34daabbdbdd75fa5e55ea1e53114efa6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4c2b743c70f9a5fa5882d9266517bd7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34daabbdbdd75fa5e55ea1e53114efa6.setIcon(icon_c4c2b743c70f9a5fa5882d9266517bd7);


            var marker_bef2e91a44bed729403772a3038ca9f3 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_717805283a95cff674edd47a94df2eec = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bef2e91a44bed729403772a3038ca9f3.setIcon(icon_717805283a95cff674edd47a94df2eec);


            var marker_6e36f26380dc73f855999301ee49cd16 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_72011afd61c6be9503a788ba150dbd4e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e36f26380dc73f855999301ee49cd16.setIcon(icon_72011afd61c6be9503a788ba150dbd4e);


            var marker_bded94e0806fa77f6b0c4a10affa8cae = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_655849f5b2768f6b7ab2e9d18503aefd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bded94e0806fa77f6b0c4a10affa8cae.setIcon(icon_655849f5b2768f6b7ab2e9d18503aefd);


            var marker_b1db39bf40f86fd0c8bc7f388365ca5c = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_dd6666c810ae59df2450bb495b3752fd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1db39bf40f86fd0c8bc7f388365ca5c.setIcon(icon_dd6666c810ae59df2450bb495b3752fd);


            var marker_44617e15bbef4ffd7cbfeb4bcc39c0b9 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1387879bbdcd524f4841c5b2b306e375 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_44617e15bbef4ffd7cbfeb4bcc39c0b9.setIcon(icon_1387879bbdcd524f4841c5b2b306e375);


            var marker_dbdfec5daa277d8df66495575aeecb89 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c77b6079543bc6d2f2530125adb14d0d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbdfec5daa277d8df66495575aeecb89.setIcon(icon_c77b6079543bc6d2f2530125adb14d0d);


            var marker_7c497e8a937df15b4a478dfee391fa38 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_57aaf5bd80d908a0b9b0e1e20189631c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7c497e8a937df15b4a478dfee391fa38.setIcon(icon_57aaf5bd80d908a0b9b0e1e20189631c);


            var marker_bf8d11c3a84bc01496efb5c26e0cd01b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c5d800da3c3cdcfa47de2c3474b3e7b3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bf8d11c3a84bc01496efb5c26e0cd01b.setIcon(icon_c5d800da3c3cdcfa47de2c3474b3e7b3);


            var marker_b1945b1bff32820e5774bb8307c3c9e2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_34c8f2c9d7cda03c16e1d5b9665abfe3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1945b1bff32820e5774bb8307c3c9e2.setIcon(icon_34c8f2c9d7cda03c16e1d5b9665abfe3);


            var marker_9c8d922308159b4b75f21d85df4d194e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e35d347a145b8bba035e00ee60c71454 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9c8d922308159b4b75f21d85df4d194e.setIcon(icon_e35d347a145b8bba035e00ee60c71454);


            var marker_544d04728ae5afbba83f6c0c90a7b104 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_314c8754a83ea323e7a1247e01edb917 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_544d04728ae5afbba83f6c0c90a7b104.setIcon(icon_314c8754a83ea323e7a1247e01edb917);


            var marker_59acf65b4ed29a3c4dbbb16161042f63 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_280f880ecf1c412d286eecc9debeefd1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_59acf65b4ed29a3c4dbbb16161042f63.setIcon(icon_280f880ecf1c412d286eecc9debeefd1);


            var marker_a85af35f3acafb816c34c4c9f2e998b9 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3656b5e5aaa32301bf5cb15e9066719f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a85af35f3acafb816c34c4c9f2e998b9.setIcon(icon_3656b5e5aaa32301bf5cb15e9066719f);


            var marker_c0f1e91c861dbe2f0069a83272b805d2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5bdcd6baff1025a96628cf40fefd8bfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0f1e91c861dbe2f0069a83272b805d2.setIcon(icon_5bdcd6baff1025a96628cf40fefd8bfd);


            var marker_f9c3660b4b31e3539f52a2f7db5af5b5 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_78cb4af7bffca3a49010a28ac71e37c2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f9c3660b4b31e3539f52a2f7db5af5b5.setIcon(icon_78cb4af7bffca3a49010a28ac71e37c2);


            var mouse_position_4cb24fc5aae35d52b8e375b8113bd864 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_53ba2e67ab4671683422076420748d4c.addControl(mouse_position_4cb24fc5aae35d52b8e375b8113bd864);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Now zoom in to a launch site and explore its proximity to see if you can easily find any railway, highway, coastline, etc. Move your mouse to these points and mark down their coordinates (shown on the top-left) in order to the distance to the launch site.


You can calculate the distance between two points on the map based on their `Lat` and `Long` values using the following method:



```python
from math import sin, cos, sqrt, atan2, radians

def calculate_distance(lat1, lon1, lat2, lon2):
    # approximate radius of earth in km
    R = 6373.0

    lat1 = radians(lat1)
    lon1 = radians(lon1)
    lat2 = radians(lat2)
    lon2 = radians(lon2)

    dlon = lon2 - lon1
    dlat = lat2 - lat1

    a = sin(dlat / 2)**2 + cos(lat1) * cos(lat2) * sin(dlon / 2)**2
    c = 2 * atan2(sqrt(a), sqrt(1 - a))

    distance = R * c
    return distance
```

*TODO:* Mark down a point on the closest coastline using MousePosition and calculate the distance between the coastline point and the launch site.



```python
# find coordinate of the closet coastline
# e.g.,: Lat: 28.56367  Lon: -80.57163
# distance_coastline = calculate_distance(launch_site_lat, launch_site_lon, coastline_lat, coastline_lon)
```

*TODO:* After obtained its coordinate, create a `folium.Marker` to show the distance



```python
# Create and add a folium.Marker on your selected closest coastline point on the map
# Display the distance between coastline point and launch site using the icon property 
# for example
#distance_marker = folium.Marker(
#    coordinate,
#    icon=DivIcon(
#        icon_size=(20, 20),
#        icon_anchor=(0, 0),
#        html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % "{:10.2f} KM".format(distance),
#        )
#    )

launch_site_lat = 28.573255
launch_site_lon = -80.646895

coastline_lat = 28.57307
coastline_lon = -80.71707

distance_coastline = calculate_distance(launch_site_lat, launch_site_lon, coastline_lat, coastline_lon)
# distance_coastline = calculate_distance(launch_site_lat, launch_site_lon, lat_formatter, lng_formatter)
distance_coastline
```




    6.85492332928881



*TODO:* Draw a `PolyLine` between a launch site to the selected coastline point



```python
# Create a `folium.PolyLine` object using the coastline coordinates and launch site coordinate
# lines=folium.PolyLine(locations=coordinates, weight=1)
#site_map.add_child(lines)
origin_locations_coords = [28.573255, -80.646895]
dest_locations_coords = [28.57307, -80.71707]

lines = folium.PolyLine(locations=[origin_locations_coords, dest_locations_coords], weight=1)
site_map.add_child(lines)

coordinates = [[launch_site_lat,launch_site_lon],[coastline_lat,coastline_lon]]
lines=folium.PolyLine(locations=coordinates, weight=1)
site_map.add_child(lines)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/python-visualization/folium/master/folium/templates/leaflet.awesome.rotate.css&quot;/&gt;
    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_53ba2e67ab4671683422076420748d4c {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_53ba2e67ab4671683422076420748d4c&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_53ba2e67ab4671683422076420748d4c = L.map(
                &quot;map_53ba2e67ab4671683422076420748d4c&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_f59ae0101471d408b57a22b16b5736d0 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_cluster_941ec86d95673aae547cfec5fb548fda = L.markerClusterGroup(
                {}
            );
            map_53ba2e67ab4671683422076420748d4c.addLayer(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var marker_0a23aa93bcef11785869419872ccfa5d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4a8d02985f5e937b3152954c68e7bdb6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0a23aa93bcef11785869419872ccfa5d.setIcon(icon_4a8d02985f5e937b3152954c68e7bdb6);


            var marker_7875dde0bfbdebbeed55a08fac2cdc7f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6e26c65a13aa8211a2f414f575e19866 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7875dde0bfbdebbeed55a08fac2cdc7f.setIcon(icon_6e26c65a13aa8211a2f414f575e19866);


            var marker_ce95b05cb9db3540b5325eb44c2ac02b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ddbe42d5dd0263ee528890917c294ce = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ce95b05cb9db3540b5325eb44c2ac02b.setIcon(icon_9ddbe42d5dd0263ee528890917c294ce);


            var marker_55e8409872774d0c01bfe5787bcead78 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52e73e14e134d2a719095f3d70983f9d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_55e8409872774d0c01bfe5787bcead78.setIcon(icon_52e73e14e134d2a719095f3d70983f9d);


            var marker_dfea4fe8af7d7d67d79336d83d010f8e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_60f2958bb5bcaffcc68bea86fc3cb292 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfea4fe8af7d7d67d79336d83d010f8e.setIcon(icon_60f2958bb5bcaffcc68bea86fc3cb292);


            var marker_92d8aeb54828ef020d7cff63d5795732 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ad4fab9878aba9c0265c72bd72de510 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_92d8aeb54828ef020d7cff63d5795732.setIcon(icon_9ad4fab9878aba9c0265c72bd72de510);


            var marker_dc0e69183b9b31603d65fc5cd9deee43 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_fcfe1289cb844b5d69165e7769d8198f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dc0e69183b9b31603d65fc5cd9deee43.setIcon(icon_fcfe1289cb844b5d69165e7769d8198f);


            var marker_6769999948cb97ada4bab81bc5d2a14d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f9332b6f0ed640fbeeca149509e3c50f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6769999948cb97ada4bab81bc5d2a14d.setIcon(icon_f9332b6f0ed640fbeeca149509e3c50f);


            var marker_07d14173bf823cf17d3c2a4c8cfbbab7 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ea1a8439b795e87217ee4a1f9bb403b0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_07d14173bf823cf17d3c2a4c8cfbbab7.setIcon(icon_ea1a8439b795e87217ee4a1f9bb403b0);


            var marker_6fd06eaed04ac7ebe1aa20349ff53162 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bd127eb14a87d7e91f1cc84c7387c91b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fd06eaed04ac7ebe1aa20349ff53162.setIcon(icon_bd127eb14a87d7e91f1cc84c7387c91b);


            var marker_ec497dd6e102ee53fc1b5dec40a65f64 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_040aa65ec787091dce526203907f221a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ec497dd6e102ee53fc1b5dec40a65f64.setIcon(icon_040aa65ec787091dce526203907f221a);


            var marker_8159f289cbabc5fd5e5dc7026bc7d21b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d6e83c92d9d850067b6d7914dd40adc5 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8159f289cbabc5fd5e5dc7026bc7d21b.setIcon(icon_d6e83c92d9d850067b6d7914dd40adc5);


            var marker_40b11555ab6746af7bcacf050416caea = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_0aec047f0282befd188a74504272b391 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_40b11555ab6746af7bcacf050416caea.setIcon(icon_0aec047f0282befd188a74504272b391);


            var marker_512efd561a7ed33835e3cd9ca974b37c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3ed5417e90a7dab69fd03be6a555fe04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_512efd561a7ed33835e3cd9ca974b37c.setIcon(icon_3ed5417e90a7dab69fd03be6a555fe04);


            var marker_9df2741233008061c42e8981f6dd1879 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ad271e1fe93789b09b3b54f157374d6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9df2741233008061c42e8981f6dd1879.setIcon(icon_2ad271e1fe93789b09b3b54f157374d6);


            var marker_824928a2cea4ad038c847d1c483bfec9 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6f10aa1d799d6ffe076921d51c2ab74a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_824928a2cea4ad038c847d1c483bfec9.setIcon(icon_6f10aa1d799d6ffe076921d51c2ab74a);


            var marker_dcaf1d48a2366e4c755f41831ae92295 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5d7f18508e1f3f79b5959c5b3c34df9a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dcaf1d48a2366e4c755f41831ae92295.setIcon(icon_5d7f18508e1f3f79b5959c5b3c34df9a);


            var marker_90257c431160ea0d44d15a7b5e46e8d5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_14b5d92ac6eef21e0d48aa78b8892be9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_90257c431160ea0d44d15a7b5e46e8d5.setIcon(icon_14b5d92ac6eef21e0d48aa78b8892be9);


            var marker_941279f09478f8be5d9dfb7a586cacef = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e0534156851dc54df739c8d763e9252d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_941279f09478f8be5d9dfb7a586cacef.setIcon(icon_e0534156851dc54df739c8d763e9252d);


            var marker_20ccb58935670a91d7f8956bfe1927e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e161e7dadd4f412094a14165368039db = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20ccb58935670a91d7f8956bfe1927e5.setIcon(icon_e161e7dadd4f412094a14165368039db);


            var marker_c209d201ddb46f35a845816ab2f4d2c8 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5f27bb2c0d22f2107f8559b1379b1299 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c209d201ddb46f35a845816ab2f4d2c8.setIcon(icon_5f27bb2c0d22f2107f8559b1379b1299);


            var marker_169ab0e4cb67cbe85d44ab0c8251f0eb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4e03b968160eaf82ea68574c9624e8b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_169ab0e4cb67cbe85d44ab0c8251f0eb.setIcon(icon_c4e03b968160eaf82ea68574c9624e8b);


            var marker_60139c9ec27ef0250373bc4b8f984e84 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f4ce3a0946c467928197c676668e5dc3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_60139c9ec27ef0250373bc4b8f984e84.setIcon(icon_f4ce3a0946c467928197c676668e5dc3);


            var marker_e9be2ed2cfe754c94568b719bea80004 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d70e7e0896a7c489ca8dedf477267e05 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e9be2ed2cfe754c94568b719bea80004.setIcon(icon_d70e7e0896a7c489ca8dedf477267e05);


            var marker_3a183638a44a6b9cd65127f0bf6307fa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1ce558f8258123fe14864124b15e2084 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3a183638a44a6b9cd65127f0bf6307fa.setIcon(icon_1ce558f8258123fe14864124b15e2084);


            var marker_3947fdf0a867451fb19d913d4093c81b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_04d2d7fa4a6072318bf5a7b353c97631 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3947fdf0a867451fb19d913d4093c81b.setIcon(icon_04d2d7fa4a6072318bf5a7b353c97631);


            var marker_6fffd16f5ae730ceaa422d419d320576 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_28740f5f393435fb2003638408b71e58 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fffd16f5ae730ceaa422d419d320576.setIcon(icon_28740f5f393435fb2003638408b71e58);


            var marker_c1bf67569aa521702f3386fd9f50234a = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52cb509e069d15e22cf323de6506ad25 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c1bf67569aa521702f3386fd9f50234a.setIcon(icon_52cb509e069d15e22cf323de6506ad25);


            var marker_29d491e6de899c170425087fbb311383 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f72f377dc3330acc4b54350fa91c5716 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_29d491e6de899c170425087fbb311383.setIcon(icon_f72f377dc3330acc4b54350fa91c5716);


            var marker_72f39afb39c9f5f23aa60f16afdef230 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a0740b5c422d2f6578cc10694de88388 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_72f39afb39c9f5f23aa60f16afdef230.setIcon(icon_a0740b5c422d2f6578cc10694de88388);


            var marker_ac78816dda8465a19c4a995b73f33238 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_8fc291b6cc723535c0f964e257a972ad = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ac78816dda8465a19c4a995b73f33238.setIcon(icon_8fc291b6cc723535c0f964e257a972ad);


            var marker_279d3cee1c792691b5ffa75d4687f554 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4b2ee1983091d9cb26cbc028206ac092 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_279d3cee1c792691b5ffa75d4687f554.setIcon(icon_4b2ee1983091d9cb26cbc028206ac092);


            var marker_d9cb37142f9f6726330307870bc1308c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ab1134476094a2011e6974547527004c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9cb37142f9f6726330307870bc1308c.setIcon(icon_ab1134476094a2011e6974547527004c);


            var marker_57f0bae8fd13cdec70cf3336b64a911b = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bef1652752ad79d1f0322aed315e5777 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_57f0bae8fd13cdec70cf3336b64a911b.setIcon(icon_bef1652752ad79d1f0322aed315e5777);


            var marker_88ec1fbe61fb99bb535692d2b802fd5c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ac49c0fa53302d8846e7674f30b7479 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_88ec1fbe61fb99bb535692d2b802fd5c.setIcon(icon_2ac49c0fa53302d8846e7674f30b7479);


            var marker_556cd21ced97417d7a963d4392ac6a17 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_192e01d38cf9d0e572f15bb8997a41a6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_556cd21ced97417d7a963d4392ac6a17.setIcon(icon_192e01d38cf9d0e572f15bb8997a41a6);


            var marker_5e8ff3a48458f66287101ea5c224d39e = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_7cf89242f2929f573d286bc6fe320133 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5e8ff3a48458f66287101ea5c224d39e.setIcon(icon_7cf89242f2929f573d286bc6fe320133);


            var marker_984908df28561a3b343589c335b33cf8 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f58da7b1b64bf22a316e1013963a5f60 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_984908df28561a3b343589c335b33cf8.setIcon(icon_f58da7b1b64bf22a316e1013963a5f60);


            var marker_a4db0ad3b213a5023128d584ccf38fe7 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a195dca5aeb1c16776ac344b5beb9c5e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a4db0ad3b213a5023128d584ccf38fe7.setIcon(icon_a195dca5aeb1c16776ac344b5beb9c5e);


            var marker_c0908593b3cb8a7ae49d3e9a06da1368 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d1dd57d1b24ae5034cf9198133fa8e51 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0908593b3cb8a7ae49d3e9a06da1368.setIcon(icon_d1dd57d1b24ae5034cf9198133fa8e51);


            var marker_34daabbdbdd75fa5e55ea1e53114efa6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4c2b743c70f9a5fa5882d9266517bd7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34daabbdbdd75fa5e55ea1e53114efa6.setIcon(icon_c4c2b743c70f9a5fa5882d9266517bd7);


            var marker_bef2e91a44bed729403772a3038ca9f3 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_717805283a95cff674edd47a94df2eec = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bef2e91a44bed729403772a3038ca9f3.setIcon(icon_717805283a95cff674edd47a94df2eec);


            var marker_6e36f26380dc73f855999301ee49cd16 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_72011afd61c6be9503a788ba150dbd4e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e36f26380dc73f855999301ee49cd16.setIcon(icon_72011afd61c6be9503a788ba150dbd4e);


            var marker_bded94e0806fa77f6b0c4a10affa8cae = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_655849f5b2768f6b7ab2e9d18503aefd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bded94e0806fa77f6b0c4a10affa8cae.setIcon(icon_655849f5b2768f6b7ab2e9d18503aefd);


            var marker_b1db39bf40f86fd0c8bc7f388365ca5c = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_dd6666c810ae59df2450bb495b3752fd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1db39bf40f86fd0c8bc7f388365ca5c.setIcon(icon_dd6666c810ae59df2450bb495b3752fd);


            var marker_44617e15bbef4ffd7cbfeb4bcc39c0b9 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1387879bbdcd524f4841c5b2b306e375 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_44617e15bbef4ffd7cbfeb4bcc39c0b9.setIcon(icon_1387879bbdcd524f4841c5b2b306e375);


            var marker_dbdfec5daa277d8df66495575aeecb89 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c77b6079543bc6d2f2530125adb14d0d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbdfec5daa277d8df66495575aeecb89.setIcon(icon_c77b6079543bc6d2f2530125adb14d0d);


            var marker_7c497e8a937df15b4a478dfee391fa38 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_57aaf5bd80d908a0b9b0e1e20189631c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7c497e8a937df15b4a478dfee391fa38.setIcon(icon_57aaf5bd80d908a0b9b0e1e20189631c);


            var marker_bf8d11c3a84bc01496efb5c26e0cd01b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c5d800da3c3cdcfa47de2c3474b3e7b3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bf8d11c3a84bc01496efb5c26e0cd01b.setIcon(icon_c5d800da3c3cdcfa47de2c3474b3e7b3);


            var marker_b1945b1bff32820e5774bb8307c3c9e2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_34c8f2c9d7cda03c16e1d5b9665abfe3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1945b1bff32820e5774bb8307c3c9e2.setIcon(icon_34c8f2c9d7cda03c16e1d5b9665abfe3);


            var marker_9c8d922308159b4b75f21d85df4d194e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e35d347a145b8bba035e00ee60c71454 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9c8d922308159b4b75f21d85df4d194e.setIcon(icon_e35d347a145b8bba035e00ee60c71454);


            var marker_544d04728ae5afbba83f6c0c90a7b104 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_314c8754a83ea323e7a1247e01edb917 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_544d04728ae5afbba83f6c0c90a7b104.setIcon(icon_314c8754a83ea323e7a1247e01edb917);


            var marker_59acf65b4ed29a3c4dbbb16161042f63 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_280f880ecf1c412d286eecc9debeefd1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_59acf65b4ed29a3c4dbbb16161042f63.setIcon(icon_280f880ecf1c412d286eecc9debeefd1);


            var marker_a85af35f3acafb816c34c4c9f2e998b9 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3656b5e5aaa32301bf5cb15e9066719f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a85af35f3acafb816c34c4c9f2e998b9.setIcon(icon_3656b5e5aaa32301bf5cb15e9066719f);


            var marker_c0f1e91c861dbe2f0069a83272b805d2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5bdcd6baff1025a96628cf40fefd8bfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0f1e91c861dbe2f0069a83272b805d2.setIcon(icon_5bdcd6baff1025a96628cf40fefd8bfd);


            var marker_f9c3660b4b31e3539f52a2f7db5af5b5 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_78cb4af7bffca3a49010a28ac71e37c2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f9c3660b4b31e3539f52a2f7db5af5b5.setIcon(icon_78cb4af7bffca3a49010a28ac71e37c2);


            var mouse_position_4cb24fc5aae35d52b8e375b8113bd864 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_53ba2e67ab4671683422076420748d4c.addControl(mouse_position_4cb24fc5aae35d52b8e375b8113bd864);


            var poly_line_80917a1a9db93f1c25941a7038492545 = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var poly_line_2a776a1de738320496019188fd362d5c = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var poly_line_cc2fe6324449a98d5299df2801ee1eea = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Your updated map with distance line should look like the following screenshot:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/launch_site_marker_distance.png" />
</center>


*TODO:* Similarly, you can draw a line betwee a launch site to its closest city, railway, highway, etc. You need to use `MousePosition` to find the their coordinates on the map first


A railway map symbol may look like this:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/railway.png" />
</center>


A highway map symbol may look like this:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/highway.png" />
</center>


A city map symbol may look like this:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/city.png" />
</center>



```python
# Create a marker with distance to a closest city, railway, highway, etc.
# Draw a line between the marker to the launch site
closest_highway = 28.57115, -80.58285
closest_railroad = 28.57206, -80.57125
closest_city = 28.3200, -80.6076
distance_highway = calculate_distance(launch_site_lat, launch_site_lon, closest_highway[0], closest_highway[1])
print('distance_highway =',distance_highway, ' km')
distance_railroad = calculate_distance(launch_site_lat, launch_site_lon, closest_railroad[0], closest_railroad[1])
print('distance_railroad =',distance_railroad, ' km')
distance_city = calculate_distance(launch_site_lat, launch_site_lon, closest_city[0], closest_city[1])
print('distance_city =',distance_city, ' km')

# closest highway marker
distance_marker = folium.Marker(
closest_highway,
icon=DivIcon(
    icon_size=(25,25),
    icon_anchor=(0,0),
    html='%s' % "{:10.2f} KM".format(distance_highway),
 )
   )
site_map.add_child(distance_marker)
```

    distance_highway = 6.260533640741881  km
    distance_railroad = 7.390448362335072  km
    distance_city = 28.430448337899534  km





<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/python-visualization/folium/master/folium/templates/leaflet.awesome.rotate.css&quot;/&gt;
    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_53ba2e67ab4671683422076420748d4c {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_53ba2e67ab4671683422076420748d4c&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_53ba2e67ab4671683422076420748d4c = L.map(
                &quot;map_53ba2e67ab4671683422076420748d4c&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_f59ae0101471d408b57a22b16b5736d0 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_cluster_941ec86d95673aae547cfec5fb548fda = L.markerClusterGroup(
                {}
            );
            map_53ba2e67ab4671683422076420748d4c.addLayer(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var marker_0a23aa93bcef11785869419872ccfa5d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4a8d02985f5e937b3152954c68e7bdb6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0a23aa93bcef11785869419872ccfa5d.setIcon(icon_4a8d02985f5e937b3152954c68e7bdb6);


            var marker_7875dde0bfbdebbeed55a08fac2cdc7f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6e26c65a13aa8211a2f414f575e19866 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7875dde0bfbdebbeed55a08fac2cdc7f.setIcon(icon_6e26c65a13aa8211a2f414f575e19866);


            var marker_ce95b05cb9db3540b5325eb44c2ac02b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ddbe42d5dd0263ee528890917c294ce = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ce95b05cb9db3540b5325eb44c2ac02b.setIcon(icon_9ddbe42d5dd0263ee528890917c294ce);


            var marker_55e8409872774d0c01bfe5787bcead78 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52e73e14e134d2a719095f3d70983f9d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_55e8409872774d0c01bfe5787bcead78.setIcon(icon_52e73e14e134d2a719095f3d70983f9d);


            var marker_dfea4fe8af7d7d67d79336d83d010f8e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_60f2958bb5bcaffcc68bea86fc3cb292 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfea4fe8af7d7d67d79336d83d010f8e.setIcon(icon_60f2958bb5bcaffcc68bea86fc3cb292);


            var marker_92d8aeb54828ef020d7cff63d5795732 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ad4fab9878aba9c0265c72bd72de510 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_92d8aeb54828ef020d7cff63d5795732.setIcon(icon_9ad4fab9878aba9c0265c72bd72de510);


            var marker_dc0e69183b9b31603d65fc5cd9deee43 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_fcfe1289cb844b5d69165e7769d8198f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dc0e69183b9b31603d65fc5cd9deee43.setIcon(icon_fcfe1289cb844b5d69165e7769d8198f);


            var marker_6769999948cb97ada4bab81bc5d2a14d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f9332b6f0ed640fbeeca149509e3c50f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6769999948cb97ada4bab81bc5d2a14d.setIcon(icon_f9332b6f0ed640fbeeca149509e3c50f);


            var marker_07d14173bf823cf17d3c2a4c8cfbbab7 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ea1a8439b795e87217ee4a1f9bb403b0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_07d14173bf823cf17d3c2a4c8cfbbab7.setIcon(icon_ea1a8439b795e87217ee4a1f9bb403b0);


            var marker_6fd06eaed04ac7ebe1aa20349ff53162 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bd127eb14a87d7e91f1cc84c7387c91b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fd06eaed04ac7ebe1aa20349ff53162.setIcon(icon_bd127eb14a87d7e91f1cc84c7387c91b);


            var marker_ec497dd6e102ee53fc1b5dec40a65f64 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_040aa65ec787091dce526203907f221a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ec497dd6e102ee53fc1b5dec40a65f64.setIcon(icon_040aa65ec787091dce526203907f221a);


            var marker_8159f289cbabc5fd5e5dc7026bc7d21b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d6e83c92d9d850067b6d7914dd40adc5 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8159f289cbabc5fd5e5dc7026bc7d21b.setIcon(icon_d6e83c92d9d850067b6d7914dd40adc5);


            var marker_40b11555ab6746af7bcacf050416caea = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_0aec047f0282befd188a74504272b391 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_40b11555ab6746af7bcacf050416caea.setIcon(icon_0aec047f0282befd188a74504272b391);


            var marker_512efd561a7ed33835e3cd9ca974b37c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3ed5417e90a7dab69fd03be6a555fe04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_512efd561a7ed33835e3cd9ca974b37c.setIcon(icon_3ed5417e90a7dab69fd03be6a555fe04);


            var marker_9df2741233008061c42e8981f6dd1879 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ad271e1fe93789b09b3b54f157374d6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9df2741233008061c42e8981f6dd1879.setIcon(icon_2ad271e1fe93789b09b3b54f157374d6);


            var marker_824928a2cea4ad038c847d1c483bfec9 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6f10aa1d799d6ffe076921d51c2ab74a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_824928a2cea4ad038c847d1c483bfec9.setIcon(icon_6f10aa1d799d6ffe076921d51c2ab74a);


            var marker_dcaf1d48a2366e4c755f41831ae92295 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5d7f18508e1f3f79b5959c5b3c34df9a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dcaf1d48a2366e4c755f41831ae92295.setIcon(icon_5d7f18508e1f3f79b5959c5b3c34df9a);


            var marker_90257c431160ea0d44d15a7b5e46e8d5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_14b5d92ac6eef21e0d48aa78b8892be9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_90257c431160ea0d44d15a7b5e46e8d5.setIcon(icon_14b5d92ac6eef21e0d48aa78b8892be9);


            var marker_941279f09478f8be5d9dfb7a586cacef = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e0534156851dc54df739c8d763e9252d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_941279f09478f8be5d9dfb7a586cacef.setIcon(icon_e0534156851dc54df739c8d763e9252d);


            var marker_20ccb58935670a91d7f8956bfe1927e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e161e7dadd4f412094a14165368039db = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20ccb58935670a91d7f8956bfe1927e5.setIcon(icon_e161e7dadd4f412094a14165368039db);


            var marker_c209d201ddb46f35a845816ab2f4d2c8 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5f27bb2c0d22f2107f8559b1379b1299 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c209d201ddb46f35a845816ab2f4d2c8.setIcon(icon_5f27bb2c0d22f2107f8559b1379b1299);


            var marker_169ab0e4cb67cbe85d44ab0c8251f0eb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4e03b968160eaf82ea68574c9624e8b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_169ab0e4cb67cbe85d44ab0c8251f0eb.setIcon(icon_c4e03b968160eaf82ea68574c9624e8b);


            var marker_60139c9ec27ef0250373bc4b8f984e84 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f4ce3a0946c467928197c676668e5dc3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_60139c9ec27ef0250373bc4b8f984e84.setIcon(icon_f4ce3a0946c467928197c676668e5dc3);


            var marker_e9be2ed2cfe754c94568b719bea80004 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d70e7e0896a7c489ca8dedf477267e05 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e9be2ed2cfe754c94568b719bea80004.setIcon(icon_d70e7e0896a7c489ca8dedf477267e05);


            var marker_3a183638a44a6b9cd65127f0bf6307fa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1ce558f8258123fe14864124b15e2084 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3a183638a44a6b9cd65127f0bf6307fa.setIcon(icon_1ce558f8258123fe14864124b15e2084);


            var marker_3947fdf0a867451fb19d913d4093c81b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_04d2d7fa4a6072318bf5a7b353c97631 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3947fdf0a867451fb19d913d4093c81b.setIcon(icon_04d2d7fa4a6072318bf5a7b353c97631);


            var marker_6fffd16f5ae730ceaa422d419d320576 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_28740f5f393435fb2003638408b71e58 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fffd16f5ae730ceaa422d419d320576.setIcon(icon_28740f5f393435fb2003638408b71e58);


            var marker_c1bf67569aa521702f3386fd9f50234a = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52cb509e069d15e22cf323de6506ad25 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c1bf67569aa521702f3386fd9f50234a.setIcon(icon_52cb509e069d15e22cf323de6506ad25);


            var marker_29d491e6de899c170425087fbb311383 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f72f377dc3330acc4b54350fa91c5716 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_29d491e6de899c170425087fbb311383.setIcon(icon_f72f377dc3330acc4b54350fa91c5716);


            var marker_72f39afb39c9f5f23aa60f16afdef230 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a0740b5c422d2f6578cc10694de88388 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_72f39afb39c9f5f23aa60f16afdef230.setIcon(icon_a0740b5c422d2f6578cc10694de88388);


            var marker_ac78816dda8465a19c4a995b73f33238 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_8fc291b6cc723535c0f964e257a972ad = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ac78816dda8465a19c4a995b73f33238.setIcon(icon_8fc291b6cc723535c0f964e257a972ad);


            var marker_279d3cee1c792691b5ffa75d4687f554 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4b2ee1983091d9cb26cbc028206ac092 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_279d3cee1c792691b5ffa75d4687f554.setIcon(icon_4b2ee1983091d9cb26cbc028206ac092);


            var marker_d9cb37142f9f6726330307870bc1308c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ab1134476094a2011e6974547527004c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9cb37142f9f6726330307870bc1308c.setIcon(icon_ab1134476094a2011e6974547527004c);


            var marker_57f0bae8fd13cdec70cf3336b64a911b = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bef1652752ad79d1f0322aed315e5777 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_57f0bae8fd13cdec70cf3336b64a911b.setIcon(icon_bef1652752ad79d1f0322aed315e5777);


            var marker_88ec1fbe61fb99bb535692d2b802fd5c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ac49c0fa53302d8846e7674f30b7479 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_88ec1fbe61fb99bb535692d2b802fd5c.setIcon(icon_2ac49c0fa53302d8846e7674f30b7479);


            var marker_556cd21ced97417d7a963d4392ac6a17 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_192e01d38cf9d0e572f15bb8997a41a6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_556cd21ced97417d7a963d4392ac6a17.setIcon(icon_192e01d38cf9d0e572f15bb8997a41a6);


            var marker_5e8ff3a48458f66287101ea5c224d39e = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_7cf89242f2929f573d286bc6fe320133 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5e8ff3a48458f66287101ea5c224d39e.setIcon(icon_7cf89242f2929f573d286bc6fe320133);


            var marker_984908df28561a3b343589c335b33cf8 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f58da7b1b64bf22a316e1013963a5f60 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_984908df28561a3b343589c335b33cf8.setIcon(icon_f58da7b1b64bf22a316e1013963a5f60);


            var marker_a4db0ad3b213a5023128d584ccf38fe7 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a195dca5aeb1c16776ac344b5beb9c5e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a4db0ad3b213a5023128d584ccf38fe7.setIcon(icon_a195dca5aeb1c16776ac344b5beb9c5e);


            var marker_c0908593b3cb8a7ae49d3e9a06da1368 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d1dd57d1b24ae5034cf9198133fa8e51 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0908593b3cb8a7ae49d3e9a06da1368.setIcon(icon_d1dd57d1b24ae5034cf9198133fa8e51);


            var marker_34daabbdbdd75fa5e55ea1e53114efa6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4c2b743c70f9a5fa5882d9266517bd7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34daabbdbdd75fa5e55ea1e53114efa6.setIcon(icon_c4c2b743c70f9a5fa5882d9266517bd7);


            var marker_bef2e91a44bed729403772a3038ca9f3 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_717805283a95cff674edd47a94df2eec = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bef2e91a44bed729403772a3038ca9f3.setIcon(icon_717805283a95cff674edd47a94df2eec);


            var marker_6e36f26380dc73f855999301ee49cd16 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_72011afd61c6be9503a788ba150dbd4e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e36f26380dc73f855999301ee49cd16.setIcon(icon_72011afd61c6be9503a788ba150dbd4e);


            var marker_bded94e0806fa77f6b0c4a10affa8cae = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_655849f5b2768f6b7ab2e9d18503aefd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bded94e0806fa77f6b0c4a10affa8cae.setIcon(icon_655849f5b2768f6b7ab2e9d18503aefd);


            var marker_b1db39bf40f86fd0c8bc7f388365ca5c = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_dd6666c810ae59df2450bb495b3752fd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1db39bf40f86fd0c8bc7f388365ca5c.setIcon(icon_dd6666c810ae59df2450bb495b3752fd);


            var marker_44617e15bbef4ffd7cbfeb4bcc39c0b9 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1387879bbdcd524f4841c5b2b306e375 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_44617e15bbef4ffd7cbfeb4bcc39c0b9.setIcon(icon_1387879bbdcd524f4841c5b2b306e375);


            var marker_dbdfec5daa277d8df66495575aeecb89 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c77b6079543bc6d2f2530125adb14d0d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbdfec5daa277d8df66495575aeecb89.setIcon(icon_c77b6079543bc6d2f2530125adb14d0d);


            var marker_7c497e8a937df15b4a478dfee391fa38 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_57aaf5bd80d908a0b9b0e1e20189631c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7c497e8a937df15b4a478dfee391fa38.setIcon(icon_57aaf5bd80d908a0b9b0e1e20189631c);


            var marker_bf8d11c3a84bc01496efb5c26e0cd01b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c5d800da3c3cdcfa47de2c3474b3e7b3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bf8d11c3a84bc01496efb5c26e0cd01b.setIcon(icon_c5d800da3c3cdcfa47de2c3474b3e7b3);


            var marker_b1945b1bff32820e5774bb8307c3c9e2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_34c8f2c9d7cda03c16e1d5b9665abfe3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1945b1bff32820e5774bb8307c3c9e2.setIcon(icon_34c8f2c9d7cda03c16e1d5b9665abfe3);


            var marker_9c8d922308159b4b75f21d85df4d194e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e35d347a145b8bba035e00ee60c71454 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9c8d922308159b4b75f21d85df4d194e.setIcon(icon_e35d347a145b8bba035e00ee60c71454);


            var marker_544d04728ae5afbba83f6c0c90a7b104 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_314c8754a83ea323e7a1247e01edb917 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_544d04728ae5afbba83f6c0c90a7b104.setIcon(icon_314c8754a83ea323e7a1247e01edb917);


            var marker_59acf65b4ed29a3c4dbbb16161042f63 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_280f880ecf1c412d286eecc9debeefd1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_59acf65b4ed29a3c4dbbb16161042f63.setIcon(icon_280f880ecf1c412d286eecc9debeefd1);


            var marker_a85af35f3acafb816c34c4c9f2e998b9 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3656b5e5aaa32301bf5cb15e9066719f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a85af35f3acafb816c34c4c9f2e998b9.setIcon(icon_3656b5e5aaa32301bf5cb15e9066719f);


            var marker_c0f1e91c861dbe2f0069a83272b805d2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5bdcd6baff1025a96628cf40fefd8bfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0f1e91c861dbe2f0069a83272b805d2.setIcon(icon_5bdcd6baff1025a96628cf40fefd8bfd);


            var marker_f9c3660b4b31e3539f52a2f7db5af5b5 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_78cb4af7bffca3a49010a28ac71e37c2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f9c3660b4b31e3539f52a2f7db5af5b5.setIcon(icon_78cb4af7bffca3a49010a28ac71e37c2);


            var mouse_position_4cb24fc5aae35d52b8e375b8113bd864 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_53ba2e67ab4671683422076420748d4c.addControl(mouse_position_4cb24fc5aae35d52b8e375b8113bd864);


            var poly_line_80917a1a9db93f1c25941a7038492545 = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var poly_line_2a776a1de738320496019188fd362d5c = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var poly_line_cc2fe6324449a98d5299df2801ee1eea = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_5aaf94821f34176089a4edb65388986d = L.marker(
                [28.57115, -80.58285],
                {}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var div_icon_4758b29a2242be4724ba8a45cb0051fb = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;      6.26 KM&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [25, 25]});
            marker_5aaf94821f34176089a4edb65388986d.setIcon(div_icon_4758b29a2242be4724ba8a45cb0051fb);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
# closest highway line
coordinates = [[launch_site_lat,launch_site_lon],closest_highway]
lines=folium.PolyLine(locations=coordinates, weight=1)
site_map.add_child(lines)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/python-visualization/folium/master/folium/templates/leaflet.awesome.rotate.css&quot;/&gt;
    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_53ba2e67ab4671683422076420748d4c {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_53ba2e67ab4671683422076420748d4c&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_53ba2e67ab4671683422076420748d4c = L.map(
                &quot;map_53ba2e67ab4671683422076420748d4c&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_f59ae0101471d408b57a22b16b5736d0 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_cluster_941ec86d95673aae547cfec5fb548fda = L.markerClusterGroup(
                {}
            );
            map_53ba2e67ab4671683422076420748d4c.addLayer(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var marker_0a23aa93bcef11785869419872ccfa5d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4a8d02985f5e937b3152954c68e7bdb6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0a23aa93bcef11785869419872ccfa5d.setIcon(icon_4a8d02985f5e937b3152954c68e7bdb6);


            var marker_7875dde0bfbdebbeed55a08fac2cdc7f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6e26c65a13aa8211a2f414f575e19866 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7875dde0bfbdebbeed55a08fac2cdc7f.setIcon(icon_6e26c65a13aa8211a2f414f575e19866);


            var marker_ce95b05cb9db3540b5325eb44c2ac02b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ddbe42d5dd0263ee528890917c294ce = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ce95b05cb9db3540b5325eb44c2ac02b.setIcon(icon_9ddbe42d5dd0263ee528890917c294ce);


            var marker_55e8409872774d0c01bfe5787bcead78 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52e73e14e134d2a719095f3d70983f9d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_55e8409872774d0c01bfe5787bcead78.setIcon(icon_52e73e14e134d2a719095f3d70983f9d);


            var marker_dfea4fe8af7d7d67d79336d83d010f8e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_60f2958bb5bcaffcc68bea86fc3cb292 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfea4fe8af7d7d67d79336d83d010f8e.setIcon(icon_60f2958bb5bcaffcc68bea86fc3cb292);


            var marker_92d8aeb54828ef020d7cff63d5795732 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ad4fab9878aba9c0265c72bd72de510 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_92d8aeb54828ef020d7cff63d5795732.setIcon(icon_9ad4fab9878aba9c0265c72bd72de510);


            var marker_dc0e69183b9b31603d65fc5cd9deee43 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_fcfe1289cb844b5d69165e7769d8198f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dc0e69183b9b31603d65fc5cd9deee43.setIcon(icon_fcfe1289cb844b5d69165e7769d8198f);


            var marker_6769999948cb97ada4bab81bc5d2a14d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f9332b6f0ed640fbeeca149509e3c50f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6769999948cb97ada4bab81bc5d2a14d.setIcon(icon_f9332b6f0ed640fbeeca149509e3c50f);


            var marker_07d14173bf823cf17d3c2a4c8cfbbab7 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ea1a8439b795e87217ee4a1f9bb403b0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_07d14173bf823cf17d3c2a4c8cfbbab7.setIcon(icon_ea1a8439b795e87217ee4a1f9bb403b0);


            var marker_6fd06eaed04ac7ebe1aa20349ff53162 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bd127eb14a87d7e91f1cc84c7387c91b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fd06eaed04ac7ebe1aa20349ff53162.setIcon(icon_bd127eb14a87d7e91f1cc84c7387c91b);


            var marker_ec497dd6e102ee53fc1b5dec40a65f64 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_040aa65ec787091dce526203907f221a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ec497dd6e102ee53fc1b5dec40a65f64.setIcon(icon_040aa65ec787091dce526203907f221a);


            var marker_8159f289cbabc5fd5e5dc7026bc7d21b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d6e83c92d9d850067b6d7914dd40adc5 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8159f289cbabc5fd5e5dc7026bc7d21b.setIcon(icon_d6e83c92d9d850067b6d7914dd40adc5);


            var marker_40b11555ab6746af7bcacf050416caea = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_0aec047f0282befd188a74504272b391 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_40b11555ab6746af7bcacf050416caea.setIcon(icon_0aec047f0282befd188a74504272b391);


            var marker_512efd561a7ed33835e3cd9ca974b37c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3ed5417e90a7dab69fd03be6a555fe04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_512efd561a7ed33835e3cd9ca974b37c.setIcon(icon_3ed5417e90a7dab69fd03be6a555fe04);


            var marker_9df2741233008061c42e8981f6dd1879 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ad271e1fe93789b09b3b54f157374d6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9df2741233008061c42e8981f6dd1879.setIcon(icon_2ad271e1fe93789b09b3b54f157374d6);


            var marker_824928a2cea4ad038c847d1c483bfec9 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6f10aa1d799d6ffe076921d51c2ab74a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_824928a2cea4ad038c847d1c483bfec9.setIcon(icon_6f10aa1d799d6ffe076921d51c2ab74a);


            var marker_dcaf1d48a2366e4c755f41831ae92295 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5d7f18508e1f3f79b5959c5b3c34df9a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dcaf1d48a2366e4c755f41831ae92295.setIcon(icon_5d7f18508e1f3f79b5959c5b3c34df9a);


            var marker_90257c431160ea0d44d15a7b5e46e8d5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_14b5d92ac6eef21e0d48aa78b8892be9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_90257c431160ea0d44d15a7b5e46e8d5.setIcon(icon_14b5d92ac6eef21e0d48aa78b8892be9);


            var marker_941279f09478f8be5d9dfb7a586cacef = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e0534156851dc54df739c8d763e9252d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_941279f09478f8be5d9dfb7a586cacef.setIcon(icon_e0534156851dc54df739c8d763e9252d);


            var marker_20ccb58935670a91d7f8956bfe1927e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e161e7dadd4f412094a14165368039db = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20ccb58935670a91d7f8956bfe1927e5.setIcon(icon_e161e7dadd4f412094a14165368039db);


            var marker_c209d201ddb46f35a845816ab2f4d2c8 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5f27bb2c0d22f2107f8559b1379b1299 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c209d201ddb46f35a845816ab2f4d2c8.setIcon(icon_5f27bb2c0d22f2107f8559b1379b1299);


            var marker_169ab0e4cb67cbe85d44ab0c8251f0eb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4e03b968160eaf82ea68574c9624e8b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_169ab0e4cb67cbe85d44ab0c8251f0eb.setIcon(icon_c4e03b968160eaf82ea68574c9624e8b);


            var marker_60139c9ec27ef0250373bc4b8f984e84 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f4ce3a0946c467928197c676668e5dc3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_60139c9ec27ef0250373bc4b8f984e84.setIcon(icon_f4ce3a0946c467928197c676668e5dc3);


            var marker_e9be2ed2cfe754c94568b719bea80004 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d70e7e0896a7c489ca8dedf477267e05 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e9be2ed2cfe754c94568b719bea80004.setIcon(icon_d70e7e0896a7c489ca8dedf477267e05);


            var marker_3a183638a44a6b9cd65127f0bf6307fa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1ce558f8258123fe14864124b15e2084 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3a183638a44a6b9cd65127f0bf6307fa.setIcon(icon_1ce558f8258123fe14864124b15e2084);


            var marker_3947fdf0a867451fb19d913d4093c81b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_04d2d7fa4a6072318bf5a7b353c97631 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3947fdf0a867451fb19d913d4093c81b.setIcon(icon_04d2d7fa4a6072318bf5a7b353c97631);


            var marker_6fffd16f5ae730ceaa422d419d320576 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_28740f5f393435fb2003638408b71e58 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fffd16f5ae730ceaa422d419d320576.setIcon(icon_28740f5f393435fb2003638408b71e58);


            var marker_c1bf67569aa521702f3386fd9f50234a = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52cb509e069d15e22cf323de6506ad25 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c1bf67569aa521702f3386fd9f50234a.setIcon(icon_52cb509e069d15e22cf323de6506ad25);


            var marker_29d491e6de899c170425087fbb311383 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f72f377dc3330acc4b54350fa91c5716 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_29d491e6de899c170425087fbb311383.setIcon(icon_f72f377dc3330acc4b54350fa91c5716);


            var marker_72f39afb39c9f5f23aa60f16afdef230 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a0740b5c422d2f6578cc10694de88388 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_72f39afb39c9f5f23aa60f16afdef230.setIcon(icon_a0740b5c422d2f6578cc10694de88388);


            var marker_ac78816dda8465a19c4a995b73f33238 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_8fc291b6cc723535c0f964e257a972ad = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ac78816dda8465a19c4a995b73f33238.setIcon(icon_8fc291b6cc723535c0f964e257a972ad);


            var marker_279d3cee1c792691b5ffa75d4687f554 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4b2ee1983091d9cb26cbc028206ac092 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_279d3cee1c792691b5ffa75d4687f554.setIcon(icon_4b2ee1983091d9cb26cbc028206ac092);


            var marker_d9cb37142f9f6726330307870bc1308c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ab1134476094a2011e6974547527004c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9cb37142f9f6726330307870bc1308c.setIcon(icon_ab1134476094a2011e6974547527004c);


            var marker_57f0bae8fd13cdec70cf3336b64a911b = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bef1652752ad79d1f0322aed315e5777 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_57f0bae8fd13cdec70cf3336b64a911b.setIcon(icon_bef1652752ad79d1f0322aed315e5777);


            var marker_88ec1fbe61fb99bb535692d2b802fd5c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ac49c0fa53302d8846e7674f30b7479 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_88ec1fbe61fb99bb535692d2b802fd5c.setIcon(icon_2ac49c0fa53302d8846e7674f30b7479);


            var marker_556cd21ced97417d7a963d4392ac6a17 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_192e01d38cf9d0e572f15bb8997a41a6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_556cd21ced97417d7a963d4392ac6a17.setIcon(icon_192e01d38cf9d0e572f15bb8997a41a6);


            var marker_5e8ff3a48458f66287101ea5c224d39e = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_7cf89242f2929f573d286bc6fe320133 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5e8ff3a48458f66287101ea5c224d39e.setIcon(icon_7cf89242f2929f573d286bc6fe320133);


            var marker_984908df28561a3b343589c335b33cf8 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f58da7b1b64bf22a316e1013963a5f60 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_984908df28561a3b343589c335b33cf8.setIcon(icon_f58da7b1b64bf22a316e1013963a5f60);


            var marker_a4db0ad3b213a5023128d584ccf38fe7 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a195dca5aeb1c16776ac344b5beb9c5e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a4db0ad3b213a5023128d584ccf38fe7.setIcon(icon_a195dca5aeb1c16776ac344b5beb9c5e);


            var marker_c0908593b3cb8a7ae49d3e9a06da1368 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d1dd57d1b24ae5034cf9198133fa8e51 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0908593b3cb8a7ae49d3e9a06da1368.setIcon(icon_d1dd57d1b24ae5034cf9198133fa8e51);


            var marker_34daabbdbdd75fa5e55ea1e53114efa6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4c2b743c70f9a5fa5882d9266517bd7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34daabbdbdd75fa5e55ea1e53114efa6.setIcon(icon_c4c2b743c70f9a5fa5882d9266517bd7);


            var marker_bef2e91a44bed729403772a3038ca9f3 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_717805283a95cff674edd47a94df2eec = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bef2e91a44bed729403772a3038ca9f3.setIcon(icon_717805283a95cff674edd47a94df2eec);


            var marker_6e36f26380dc73f855999301ee49cd16 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_72011afd61c6be9503a788ba150dbd4e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e36f26380dc73f855999301ee49cd16.setIcon(icon_72011afd61c6be9503a788ba150dbd4e);


            var marker_bded94e0806fa77f6b0c4a10affa8cae = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_655849f5b2768f6b7ab2e9d18503aefd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bded94e0806fa77f6b0c4a10affa8cae.setIcon(icon_655849f5b2768f6b7ab2e9d18503aefd);


            var marker_b1db39bf40f86fd0c8bc7f388365ca5c = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_dd6666c810ae59df2450bb495b3752fd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1db39bf40f86fd0c8bc7f388365ca5c.setIcon(icon_dd6666c810ae59df2450bb495b3752fd);


            var marker_44617e15bbef4ffd7cbfeb4bcc39c0b9 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1387879bbdcd524f4841c5b2b306e375 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_44617e15bbef4ffd7cbfeb4bcc39c0b9.setIcon(icon_1387879bbdcd524f4841c5b2b306e375);


            var marker_dbdfec5daa277d8df66495575aeecb89 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c77b6079543bc6d2f2530125adb14d0d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbdfec5daa277d8df66495575aeecb89.setIcon(icon_c77b6079543bc6d2f2530125adb14d0d);


            var marker_7c497e8a937df15b4a478dfee391fa38 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_57aaf5bd80d908a0b9b0e1e20189631c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7c497e8a937df15b4a478dfee391fa38.setIcon(icon_57aaf5bd80d908a0b9b0e1e20189631c);


            var marker_bf8d11c3a84bc01496efb5c26e0cd01b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c5d800da3c3cdcfa47de2c3474b3e7b3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bf8d11c3a84bc01496efb5c26e0cd01b.setIcon(icon_c5d800da3c3cdcfa47de2c3474b3e7b3);


            var marker_b1945b1bff32820e5774bb8307c3c9e2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_34c8f2c9d7cda03c16e1d5b9665abfe3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1945b1bff32820e5774bb8307c3c9e2.setIcon(icon_34c8f2c9d7cda03c16e1d5b9665abfe3);


            var marker_9c8d922308159b4b75f21d85df4d194e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e35d347a145b8bba035e00ee60c71454 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9c8d922308159b4b75f21d85df4d194e.setIcon(icon_e35d347a145b8bba035e00ee60c71454);


            var marker_544d04728ae5afbba83f6c0c90a7b104 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_314c8754a83ea323e7a1247e01edb917 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_544d04728ae5afbba83f6c0c90a7b104.setIcon(icon_314c8754a83ea323e7a1247e01edb917);


            var marker_59acf65b4ed29a3c4dbbb16161042f63 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_280f880ecf1c412d286eecc9debeefd1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_59acf65b4ed29a3c4dbbb16161042f63.setIcon(icon_280f880ecf1c412d286eecc9debeefd1);


            var marker_a85af35f3acafb816c34c4c9f2e998b9 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3656b5e5aaa32301bf5cb15e9066719f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a85af35f3acafb816c34c4c9f2e998b9.setIcon(icon_3656b5e5aaa32301bf5cb15e9066719f);


            var marker_c0f1e91c861dbe2f0069a83272b805d2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5bdcd6baff1025a96628cf40fefd8bfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0f1e91c861dbe2f0069a83272b805d2.setIcon(icon_5bdcd6baff1025a96628cf40fefd8bfd);


            var marker_f9c3660b4b31e3539f52a2f7db5af5b5 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_78cb4af7bffca3a49010a28ac71e37c2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f9c3660b4b31e3539f52a2f7db5af5b5.setIcon(icon_78cb4af7bffca3a49010a28ac71e37c2);


            var mouse_position_4cb24fc5aae35d52b8e375b8113bd864 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_53ba2e67ab4671683422076420748d4c.addControl(mouse_position_4cb24fc5aae35d52b8e375b8113bd864);


            var poly_line_80917a1a9db93f1c25941a7038492545 = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var poly_line_2a776a1de738320496019188fd362d5c = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var poly_line_cc2fe6324449a98d5299df2801ee1eea = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_5aaf94821f34176089a4edb65388986d = L.marker(
                [28.57115, -80.58285],
                {}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var div_icon_4758b29a2242be4724ba8a45cb0051fb = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;      6.26 KM&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [25, 25]});
            marker_5aaf94821f34176089a4edb65388986d.setIcon(div_icon_4758b29a2242be4724ba8a45cb0051fb);


            var poly_line_c72d5f95761b32e292ef260450464cee = L.polyline(
                [[28.573255, -80.646895], [28.57115, -80.58285]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
# closest railroad marker
distance_marker = folium.Marker(
 closest_railroad,
   icon=DivIcon(
       icon_size=(25,25),
       icon_anchor=(0,0),
       html='%s' % "{:10.2f} KM".format(distance_railroad),
       )
   )
site_map.add_child(distance_marker)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/python-visualization/folium/master/folium/templates/leaflet.awesome.rotate.css&quot;/&gt;
    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_53ba2e67ab4671683422076420748d4c {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
    &lt;script src=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://rawcdn.githack.com/ardhi/Leaflet.MousePosition/c32f1c84/src/L.Control.MousePosition.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_53ba2e67ab4671683422076420748d4c&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_53ba2e67ab4671683422076420748d4c = L.map(
                &quot;map_53ba2e67ab4671683422076420748d4c&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_f59ae0101471d408b57a22b16b5736d0 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_cluster_941ec86d95673aae547cfec5fb548fda = L.markerClusterGroup(
                {}
            );
            map_53ba2e67ab4671683422076420748d4c.addLayer(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var marker_0a23aa93bcef11785869419872ccfa5d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4a8d02985f5e937b3152954c68e7bdb6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0a23aa93bcef11785869419872ccfa5d.setIcon(icon_4a8d02985f5e937b3152954c68e7bdb6);


            var marker_7875dde0bfbdebbeed55a08fac2cdc7f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6e26c65a13aa8211a2f414f575e19866 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7875dde0bfbdebbeed55a08fac2cdc7f.setIcon(icon_6e26c65a13aa8211a2f414f575e19866);


            var marker_ce95b05cb9db3540b5325eb44c2ac02b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ddbe42d5dd0263ee528890917c294ce = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ce95b05cb9db3540b5325eb44c2ac02b.setIcon(icon_9ddbe42d5dd0263ee528890917c294ce);


            var marker_55e8409872774d0c01bfe5787bcead78 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52e73e14e134d2a719095f3d70983f9d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_55e8409872774d0c01bfe5787bcead78.setIcon(icon_52e73e14e134d2a719095f3d70983f9d);


            var marker_dfea4fe8af7d7d67d79336d83d010f8e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_60f2958bb5bcaffcc68bea86fc3cb292 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfea4fe8af7d7d67d79336d83d010f8e.setIcon(icon_60f2958bb5bcaffcc68bea86fc3cb292);


            var marker_92d8aeb54828ef020d7cff63d5795732 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_9ad4fab9878aba9c0265c72bd72de510 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_92d8aeb54828ef020d7cff63d5795732.setIcon(icon_9ad4fab9878aba9c0265c72bd72de510);


            var marker_dc0e69183b9b31603d65fc5cd9deee43 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_fcfe1289cb844b5d69165e7769d8198f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dc0e69183b9b31603d65fc5cd9deee43.setIcon(icon_fcfe1289cb844b5d69165e7769d8198f);


            var marker_6769999948cb97ada4bab81bc5d2a14d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f9332b6f0ed640fbeeca149509e3c50f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6769999948cb97ada4bab81bc5d2a14d.setIcon(icon_f9332b6f0ed640fbeeca149509e3c50f);


            var marker_07d14173bf823cf17d3c2a4c8cfbbab7 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ea1a8439b795e87217ee4a1f9bb403b0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_07d14173bf823cf17d3c2a4c8cfbbab7.setIcon(icon_ea1a8439b795e87217ee4a1f9bb403b0);


            var marker_6fd06eaed04ac7ebe1aa20349ff53162 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bd127eb14a87d7e91f1cc84c7387c91b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fd06eaed04ac7ebe1aa20349ff53162.setIcon(icon_bd127eb14a87d7e91f1cc84c7387c91b);


            var marker_ec497dd6e102ee53fc1b5dec40a65f64 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_040aa65ec787091dce526203907f221a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ec497dd6e102ee53fc1b5dec40a65f64.setIcon(icon_040aa65ec787091dce526203907f221a);


            var marker_8159f289cbabc5fd5e5dc7026bc7d21b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d6e83c92d9d850067b6d7914dd40adc5 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8159f289cbabc5fd5e5dc7026bc7d21b.setIcon(icon_d6e83c92d9d850067b6d7914dd40adc5);


            var marker_40b11555ab6746af7bcacf050416caea = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_0aec047f0282befd188a74504272b391 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_40b11555ab6746af7bcacf050416caea.setIcon(icon_0aec047f0282befd188a74504272b391);


            var marker_512efd561a7ed33835e3cd9ca974b37c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3ed5417e90a7dab69fd03be6a555fe04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_512efd561a7ed33835e3cd9ca974b37c.setIcon(icon_3ed5417e90a7dab69fd03be6a555fe04);


            var marker_9df2741233008061c42e8981f6dd1879 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ad271e1fe93789b09b3b54f157374d6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9df2741233008061c42e8981f6dd1879.setIcon(icon_2ad271e1fe93789b09b3b54f157374d6);


            var marker_824928a2cea4ad038c847d1c483bfec9 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_6f10aa1d799d6ffe076921d51c2ab74a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_824928a2cea4ad038c847d1c483bfec9.setIcon(icon_6f10aa1d799d6ffe076921d51c2ab74a);


            var marker_dcaf1d48a2366e4c755f41831ae92295 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5d7f18508e1f3f79b5959c5b3c34df9a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dcaf1d48a2366e4c755f41831ae92295.setIcon(icon_5d7f18508e1f3f79b5959c5b3c34df9a);


            var marker_90257c431160ea0d44d15a7b5e46e8d5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_14b5d92ac6eef21e0d48aa78b8892be9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_90257c431160ea0d44d15a7b5e46e8d5.setIcon(icon_14b5d92ac6eef21e0d48aa78b8892be9);


            var marker_941279f09478f8be5d9dfb7a586cacef = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e0534156851dc54df739c8d763e9252d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_941279f09478f8be5d9dfb7a586cacef.setIcon(icon_e0534156851dc54df739c8d763e9252d);


            var marker_20ccb58935670a91d7f8956bfe1927e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e161e7dadd4f412094a14165368039db = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20ccb58935670a91d7f8956bfe1927e5.setIcon(icon_e161e7dadd4f412094a14165368039db);


            var marker_c209d201ddb46f35a845816ab2f4d2c8 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5f27bb2c0d22f2107f8559b1379b1299 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c209d201ddb46f35a845816ab2f4d2c8.setIcon(icon_5f27bb2c0d22f2107f8559b1379b1299);


            var marker_169ab0e4cb67cbe85d44ab0c8251f0eb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4e03b968160eaf82ea68574c9624e8b = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_169ab0e4cb67cbe85d44ab0c8251f0eb.setIcon(icon_c4e03b968160eaf82ea68574c9624e8b);


            var marker_60139c9ec27ef0250373bc4b8f984e84 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f4ce3a0946c467928197c676668e5dc3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_60139c9ec27ef0250373bc4b8f984e84.setIcon(icon_f4ce3a0946c467928197c676668e5dc3);


            var marker_e9be2ed2cfe754c94568b719bea80004 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d70e7e0896a7c489ca8dedf477267e05 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e9be2ed2cfe754c94568b719bea80004.setIcon(icon_d70e7e0896a7c489ca8dedf477267e05);


            var marker_3a183638a44a6b9cd65127f0bf6307fa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1ce558f8258123fe14864124b15e2084 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3a183638a44a6b9cd65127f0bf6307fa.setIcon(icon_1ce558f8258123fe14864124b15e2084);


            var marker_3947fdf0a867451fb19d913d4093c81b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_04d2d7fa4a6072318bf5a7b353c97631 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3947fdf0a867451fb19d913d4093c81b.setIcon(icon_04d2d7fa4a6072318bf5a7b353c97631);


            var marker_6fffd16f5ae730ceaa422d419d320576 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_28740f5f393435fb2003638408b71e58 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6fffd16f5ae730ceaa422d419d320576.setIcon(icon_28740f5f393435fb2003638408b71e58);


            var marker_c1bf67569aa521702f3386fd9f50234a = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_52cb509e069d15e22cf323de6506ad25 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c1bf67569aa521702f3386fd9f50234a.setIcon(icon_52cb509e069d15e22cf323de6506ad25);


            var marker_29d491e6de899c170425087fbb311383 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f72f377dc3330acc4b54350fa91c5716 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_29d491e6de899c170425087fbb311383.setIcon(icon_f72f377dc3330acc4b54350fa91c5716);


            var marker_72f39afb39c9f5f23aa60f16afdef230 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a0740b5c422d2f6578cc10694de88388 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_72f39afb39c9f5f23aa60f16afdef230.setIcon(icon_a0740b5c422d2f6578cc10694de88388);


            var marker_ac78816dda8465a19c4a995b73f33238 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_8fc291b6cc723535c0f964e257a972ad = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ac78816dda8465a19c4a995b73f33238.setIcon(icon_8fc291b6cc723535c0f964e257a972ad);


            var marker_279d3cee1c792691b5ffa75d4687f554 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_4b2ee1983091d9cb26cbc028206ac092 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_279d3cee1c792691b5ffa75d4687f554.setIcon(icon_4b2ee1983091d9cb26cbc028206ac092);


            var marker_d9cb37142f9f6726330307870bc1308c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_ab1134476094a2011e6974547527004c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9cb37142f9f6726330307870bc1308c.setIcon(icon_ab1134476094a2011e6974547527004c);


            var marker_57f0bae8fd13cdec70cf3336b64a911b = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_bef1652752ad79d1f0322aed315e5777 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_57f0bae8fd13cdec70cf3336b64a911b.setIcon(icon_bef1652752ad79d1f0322aed315e5777);


            var marker_88ec1fbe61fb99bb535692d2b802fd5c = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_2ac49c0fa53302d8846e7674f30b7479 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_88ec1fbe61fb99bb535692d2b802fd5c.setIcon(icon_2ac49c0fa53302d8846e7674f30b7479);


            var marker_556cd21ced97417d7a963d4392ac6a17 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_192e01d38cf9d0e572f15bb8997a41a6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_556cd21ced97417d7a963d4392ac6a17.setIcon(icon_192e01d38cf9d0e572f15bb8997a41a6);


            var marker_5e8ff3a48458f66287101ea5c224d39e = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_7cf89242f2929f573d286bc6fe320133 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5e8ff3a48458f66287101ea5c224d39e.setIcon(icon_7cf89242f2929f573d286bc6fe320133);


            var marker_984908df28561a3b343589c335b33cf8 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_f58da7b1b64bf22a316e1013963a5f60 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_984908df28561a3b343589c335b33cf8.setIcon(icon_f58da7b1b64bf22a316e1013963a5f60);


            var marker_a4db0ad3b213a5023128d584ccf38fe7 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_a195dca5aeb1c16776ac344b5beb9c5e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a4db0ad3b213a5023128d584ccf38fe7.setIcon(icon_a195dca5aeb1c16776ac344b5beb9c5e);


            var marker_c0908593b3cb8a7ae49d3e9a06da1368 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_d1dd57d1b24ae5034cf9198133fa8e51 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0908593b3cb8a7ae49d3e9a06da1368.setIcon(icon_d1dd57d1b24ae5034cf9198133fa8e51);


            var marker_34daabbdbdd75fa5e55ea1e53114efa6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c4c2b743c70f9a5fa5882d9266517bd7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34daabbdbdd75fa5e55ea1e53114efa6.setIcon(icon_c4c2b743c70f9a5fa5882d9266517bd7);


            var marker_bef2e91a44bed729403772a3038ca9f3 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_717805283a95cff674edd47a94df2eec = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bef2e91a44bed729403772a3038ca9f3.setIcon(icon_717805283a95cff674edd47a94df2eec);


            var marker_6e36f26380dc73f855999301ee49cd16 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_72011afd61c6be9503a788ba150dbd4e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e36f26380dc73f855999301ee49cd16.setIcon(icon_72011afd61c6be9503a788ba150dbd4e);


            var marker_bded94e0806fa77f6b0c4a10affa8cae = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_655849f5b2768f6b7ab2e9d18503aefd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bded94e0806fa77f6b0c4a10affa8cae.setIcon(icon_655849f5b2768f6b7ab2e9d18503aefd);


            var marker_b1db39bf40f86fd0c8bc7f388365ca5c = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_dd6666c810ae59df2450bb495b3752fd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1db39bf40f86fd0c8bc7f388365ca5c.setIcon(icon_dd6666c810ae59df2450bb495b3752fd);


            var marker_44617e15bbef4ffd7cbfeb4bcc39c0b9 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_1387879bbdcd524f4841c5b2b306e375 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_44617e15bbef4ffd7cbfeb4bcc39c0b9.setIcon(icon_1387879bbdcd524f4841c5b2b306e375);


            var marker_dbdfec5daa277d8df66495575aeecb89 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c77b6079543bc6d2f2530125adb14d0d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbdfec5daa277d8df66495575aeecb89.setIcon(icon_c77b6079543bc6d2f2530125adb14d0d);


            var marker_7c497e8a937df15b4a478dfee391fa38 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_57aaf5bd80d908a0b9b0e1e20189631c = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7c497e8a937df15b4a478dfee391fa38.setIcon(icon_57aaf5bd80d908a0b9b0e1e20189631c);


            var marker_bf8d11c3a84bc01496efb5c26e0cd01b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_c5d800da3c3cdcfa47de2c3474b3e7b3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bf8d11c3a84bc01496efb5c26e0cd01b.setIcon(icon_c5d800da3c3cdcfa47de2c3474b3e7b3);


            var marker_b1945b1bff32820e5774bb8307c3c9e2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_34c8f2c9d7cda03c16e1d5b9665abfe3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b1945b1bff32820e5774bb8307c3c9e2.setIcon(icon_34c8f2c9d7cda03c16e1d5b9665abfe3);


            var marker_9c8d922308159b4b75f21d85df4d194e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_e35d347a145b8bba035e00ee60c71454 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9c8d922308159b4b75f21d85df4d194e.setIcon(icon_e35d347a145b8bba035e00ee60c71454);


            var marker_544d04728ae5afbba83f6c0c90a7b104 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_314c8754a83ea323e7a1247e01edb917 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_544d04728ae5afbba83f6c0c90a7b104.setIcon(icon_314c8754a83ea323e7a1247e01edb917);


            var marker_59acf65b4ed29a3c4dbbb16161042f63 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_280f880ecf1c412d286eecc9debeefd1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_59acf65b4ed29a3c4dbbb16161042f63.setIcon(icon_280f880ecf1c412d286eecc9debeefd1);


            var marker_a85af35f3acafb816c34c4c9f2e998b9 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_3656b5e5aaa32301bf5cb15e9066719f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a85af35f3acafb816c34c4c9f2e998b9.setIcon(icon_3656b5e5aaa32301bf5cb15e9066719f);


            var marker_c0f1e91c861dbe2f0069a83272b805d2 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_5bdcd6baff1025a96628cf40fefd8bfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0f1e91c861dbe2f0069a83272b805d2.setIcon(icon_5bdcd6baff1025a96628cf40fefd8bfd);


            var marker_f9c3660b4b31e3539f52a2f7db5af5b5 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_941ec86d95673aae547cfec5fb548fda);


            var icon_78cb4af7bffca3a49010a28ac71e37c2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f9c3660b4b31e3539f52a2f7db5af5b5.setIcon(icon_78cb4af7bffca3a49010a28ac71e37c2);


            var mouse_position_4cb24fc5aae35d52b8e375b8113bd864 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_4cb24fc5aae35d52b8e375b8113bd864.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_53ba2e67ab4671683422076420748d4c.addControl(mouse_position_4cb24fc5aae35d52b8e375b8113bd864);


            var poly_line_80917a1a9db93f1c25941a7038492545 = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var poly_line_2a776a1de738320496019188fd362d5c = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var poly_line_cc2fe6324449a98d5299df2801ee1eea = L.polyline(
                [[28.573255, -80.646895], [28.57307, -80.71707]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_5aaf94821f34176089a4edb65388986d = L.marker(
                [28.57115, -80.58285],
                {}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var div_icon_4758b29a2242be4724ba8a45cb0051fb = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;      6.26 KM&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [25, 25]});
            marker_5aaf94821f34176089a4edb65388986d.setIcon(div_icon_4758b29a2242be4724ba8a45cb0051fb);


            var poly_line_c72d5f95761b32e292ef260450464cee = L.polyline(
                [[28.573255, -80.646895], [28.57115, -80.58285]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var marker_87934e959f3ab734a90f1ba76da980b8 = L.marker(
                [28.57206, -80.57125],
                {}
            ).addTo(map_53ba2e67ab4671683422076420748d4c);


            var div_icon_cb3c22eafca14d2eb8ad89caa41b0582 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;      7.39 KM&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [25, 25]});
            marker_87934e959f3ab734a90f1ba76da980b8.setIcon(div_icon_cb3c22eafca14d2eb8ad89caa41b0582);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



After you plot distance lines to the proximities, you can answer the following questions easily:

*   Are launch sites in close proximity to railways?
*   Are launch sites in close proximity to highways?
*   Are launch sites in close proximity to coastline?
*   Do launch sites keep certain distance away from cities?

Also please try to explain your findings.


# Next Steps:

Now you have discovered many interesting insights related to the launch sites' location using folium, in a very interactive way. Next, you will need to build a dashboard using Ploty Dash on detailed launch records.


## Authors


[Yan Luo](https://www.linkedin.com/in/yan-luo-96288783/?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01)

[Ekene Emmanuel Ajemba](https://www.linkedin.com/in/ekene-e-ajemba-b3566329/?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDeveloperSkillsNetworkDA0101ENSkillsNetwork20235326-2021-01-01)


### Other Contributors


Joseph Santarcangelo


## Change Log


| Date (YYYY-MM-DD) | Version | Changed By | Change Description          |
| ----------------- | ------- | ---------- | --------------------------- |
| 2021-05-26        | 1.0     | Yan        | Created the initial version |


Copyright © 2021 IBM Corporation. All rights reserved.

