Title: Remote GeoJSON to GeoDataFrame
Date: 2017-11-01 13:00
Modified: 2018-01-14 16:56
Category: Python
Tags: geopandas, geojson, Python Pandas, Python
Slug: remote-geojson-to-geodataframe
Authors: Ryan Cooper
Summary: How to get geojson from a web source and load it into a geopandas GeoDataFrame

# Remote GeoJSON to GeoDataFrame

I’m inspired by Stephen Smith’s quick post about rapid prototyping during his lunch break. Lately I’ve been using my lunch break to watch videos that are both related to the work I do for the City of Raleigh Park, Recreation & Cultural Resources Department and of interest to me. But sometimes I’d rather just mess around while I eat my turkey sandwich. I did just that today.

We’re an Esri shop here at City of Raleigh, so most of my day-to-day is spent in that ecosystem. That said, I’ve been really interested in the Python library geopandas for a bit now and wanted to spend my lunch with it.

One thing that’s great about geopandas is that you can read in all sorts of different data using gpd.read_file(). That’s great if you have a Shapefile or some GeoJSON on your machine. But what if you want to pull in some data from a URL? gpd.read_file() isn’t going to work.

So during lunch I messed around (e.g. Wrote some Python using Spyder, Googled things, ducked into Stack Overflow) and cobbled together a little function that takes a URL pointing to some GeoJSON as an input and returns a GeoDataFrame:

[gist:id=6030df87e8bd62fa583a83ecea9fceab]
