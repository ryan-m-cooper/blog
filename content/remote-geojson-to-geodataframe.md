Title: Remote GeoJSON to GeoDataFrame
Date: 2017-11-01 13:00
Modified: 2018-01-14 16:56
Category: How-To
Tags: geopandas, geojson, Python Pandas, Python
Slug: remote-geojson-to-geodataframe
Authors: Ryan Cooper
Summary: How to get geojson from a web source and load it into a geopandas GeoDataFrame

# Remote GeoJSON to GeoDataFrame

I’m inspired by Stephen Smith’s [quick post](https://medium.com/@TheMapSmith/messin-around-with-maps-ce3a3ffd86c) about rapid prototyping during his lunch break. Lately I’ve been using my lunch break to watch videos that are both related to the work I do for the City of Raleigh Park, Recreation & Cultural Resources Department and of interest to me. But sometimes I’d rather just mess around while I eat my turkey sandwich. I did just that today.

We’re an Esri shop here at City of Raleigh, so most of my day-to-day is spent in that ecosystem. That said, I’ve been really interested in the Python library **[geopandas](http://geopandas.org/index.html)** for a bit now and wanted to spend my lunch with it.

One thing that’s great about geopandas is that you can read in all sorts of different data using `gpd.read_file()`. That’s great if you have a Shapefile or some GeoJSON on your machine. But what if you want to pull in some data from a URL? `gpd.read_file()` isn’t going to work.

So during lunch I messed around (e.g. Wrote some Python using [Spyder](https://github.com/spyder-ide), Googled things, ducked into [Stack Overflow](https://stackoverflow.com/questions/37728540/create-a-geodataframe-from-a-geojson-object)) and cobbled together a little function that takes a URL pointing to some GeoJSON as an input and returns a GeoDataFrame:

[gist:id=6030df87e8bd62fa583a83ecea9fceab]

What’s going on here? We use **[requests](http://docs.python-requests.org/en/master/)** to get our GeoJSON from the URL we pass. We then create a GeoDataFrame, `gdf`, by using `gpd.GeoDataFrame.from_features()` iterate through the features in our GeoJSON. Finally, we return `gdf` so that we can access our newly created GeoDataFrame for plotting or doing further analysis. There’s also an option to plot the result if you want. Just set `display` to `True` when you call the function.

For example, let’s say we want to look at the City of Raleigh council districts:

[gist:id=803cbff714ca3353b62baa2c612905ee]

In line 1 we pass the URL of council district GeoJSON on [Raleigh’s open data site](http://data-ral.opendata.arcgis.com/) and assign the returned GeoDataFrame to a variable, `districts`. We can then use geopandas (but really matplotlib) to plot the GeoDataFrame.

![Plot of Raleigh city council districts generated from remote GeoJSON]({filename}/images/remote-geojson-to-geodataframe_1.png)

This is a pretty narrow use case for this code and I’m sure there are lots of ways to bork it. Still, I hope you find it useful!
