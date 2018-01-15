Title: Loading External GeoJSON: A(nother) Way to Do It with jQuery
Date: 2017-02-08 00:00
Modified: 2018-01-14 22:15
Category: How-To
Tags:  geojson, jQuery, JavaScript, LeafletJS
Slug: loading-external-geojson
Authors: Ryan Cooper
Summary: Load GeoJSON from a URL for use in Leaflet map.

# Loading External GeoJSON: A(nother) Way to Do It with jQuery

It’s a common mistake for those of us getting into making maps with Leaflet: After running through a few examples, the excited mapper is ready to make a visualization from their own data. We get everything set-up and call `L.geoJSON()` on some data onto a web server. *CTRL+S…F5…*

![Code and console showing what happens if you try to load GeoJSON from a URL using the L.geoJSON function in Leaflet](https://cdn-images-1.medium.com/max/1000/1*eC8n9vbKDgXS0_7dsFDclw.png)

Let me just get this admission out of the way: I don’t really understand asynchronous loading or callbacks. As such, I spend a lot of time that could be dedicated to learning about asynchronous loading and callbacks instead trying to find ways to load data in the more linear, step-wise fashion. Here’s what I want to do:

1. Get some external data
2. Make a map with that data

I like using GeoJSON as my data format for map data, but I don’t like hard-coding it into my map script. I want to load it from somewhere else and assign it a variable within my map script because that’s how [the official tutorial on using GeoJSON in Leaflet](http://leafletjs.com/examples/geojson/) more or less suggests I could do it. I also don’t necessarily want to pull my data from the same directory as my project. I want to start each project with the assumption that my data will be coming from an external service because that’s where most of my data is located.

The problem is that my basic workflow for loading GeoJSON into a map is at odds with my technical ability to use callbacks. What to do? In this post I will demonstrate how I use `jQuery.ajax` and jQuery’s `.when()`/`.done()` functionality to load external GeoJSON into a webmap. We won’t be avoiding asynchronicity or callbacks entirely, but will be taming them in a way that allows us to create a Leaflet map with external data that doesn’t involve a descent into [“callback hell”](http://callbackhell.com/).

### Set up a Map

We’ll begin with some boilerplate code to get our map started. We’ll generate afull window map focused on Kentucky using [Leaflet
v1](http://leafletjs.com/download.html).

[gist:id=749ca67f1d96f896b1d4c58d7a1e7739]

![Our basic starter map.](https://cdn-images-1.medium.com/max/800/1*tmqaZDxVKdh685JorqwhwQ.png)]
(*[View full
map](http://bl.ocks.org/maptastik/raw/749ca67f1d96f896b1d4c58d7a1e7739/)*)

### Add jQuery

**[jQuery](http://jquery.com/)** has some nice tools that will help us control loading our data and adding it to our map in the order that we want to. I usually load it from [Google’s hosted libraries](https://developers.google.com/speed/libraries/), but you could download it from jQuery and host it yourself if you’d like!

We’ll add jQuery before our map script. I like to include it as the first script in my map placing it just ahead of Leaflet:

[gist:id=c2169793e9cc4f90bbab20cf20a56fc4]

![Our map with jQuery included. Nothing changed. That’s OK!](https://cdn-images-1.medium.com/max/800/1*tmqaZDxVKdh685JorqwhwQ.png)
(*[View full map](http://bl.ocks.org/maptastik/c2169793e9cc4f90bbab20cf20a56fc4))*)

Our map should look the same as it did earlier. That’s fine. All we did was add jQuery. We haven’t actually asked jQuery to help us do anything though. Let’s change that.

### Load external data with AJAX

[**AJAX**](https://developer.mozilla.org/en-US/docs/AJAX/Getting_Started) stands for Asynchronous JavaScript and XML. AJAX allows us to send and receive data as well as update a web page without reloading it. For our case, it allows us to fetch external GeoJSON so that we can add it to our map. While you can use AJAX with vanilla JavaScript, [jQuery has a nice set of AJAX tools](http://api.jquery.com/jquery.ajax/) that make implementing it, arguably, a little easier.

```JavaScript
$.ajax({
    url:<URL to data>,
    dataType: "json",
    success: console.log("Data successfully loaded!"),
    error: function (xhr) {
        alert(xhr.statusText)
    }
});
```

For our map, we are going to request some GeoJSON that is located at this [here](https://gist.githubusercontent.com/maptastik/df8e483d5ac1c6cae3dc4a7c02ea9039/raw/9cd46849bddcfa90aab240772a12275408d6d8d0/kyCounties.geojson). We will specify a **data type**, JSON, and provide instructions for what should happen if the request succeeds (`success`)or fails (`error`). The result of this request will be stored in a variable, `counties`. We’ll add this code at the very top of our map script.

[gist:id=fbdbbef6c5ac72fbad775812fdcd9d33]

![](https://cdn-images-1.medium.com/max/800/1*tmqaZDxVKdh685JorqwhwQ.png)
(*[View full map](http://bl.ocks.org/maptastik/fbdbbef6c5ac72fbad775812fdcd9d33)*)

What’s happening here? When jQuery makes the AJAX request, it tries to get whatever is at the URL we specified. If there is an error, we get an alert with some information about what went wrong. If the request is successful, we get a little message in our console to tell us so. Ultimately, the variable `counties` will be the result of that request and we will be able to continue with our map.

Our map still won’t look any different, but behind the scenes we’ve done what we need to request the external GeoJSON data we want to add to our map. You can check this by opening your developer tools printing `counties` to the console.

```JavaScript
console.log(counties);
```

![](https://cdn-images-1.medium.com/max/1000/1*VWLZbytmQiYygWOvrfVEmw.png)

You might have expected that `counties` would be the raw GeoJSON from the specified URL. It isn’t. Instead, `counties` is an **[XMLHttpResponse object](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)** with a whole lot of information regarding our data request. However, notice the `responseJSON` value. We got that property because we set:

    ```JavaScript
    responseType: "json",
    ```
in our AJAX request. The resulting value looks a lot like GeoJSON. That’s because it is! We’ll use it shortly.

### `.when()` & `.done()`

One of the advantages of AJAX is that your page can load even if the the AJAX request is slow or fails. In our case, we’re trying to wrangle that functionality a bit in order to control the order in which elements of our map load. Specifically, we don’t want our map to load until the data request has finished. The reason for this is that it is possible that the code that draw our map could execute before our data request is finished. If that were to happen, then we would potentially be passing a variable, `counties`, that has not yet been fully defined. To ensure that everything loads in the proper order and at the proper time, we can chain two jQuery methods together: [`.when()`](http://api.jquery.com/jQuery.when/) and [`.done()`](http://api.jquery.com/deferred.done/).

```
    <script>
    // Load data with .ajax()
    $.when(data).done(function(){
      //Do something
    });
    </script>
```

In the snippet above we’ve got some data we requested via `.ajax()`. We then commanded that *when* the *data* we requested is *done* loading, do something. For our map we’ll request the data like we did earlier, but wrap the rest of the map script in our `when`/`done` function.

[gist:id=e7ac58edd5f34fb9487faa711ffc4fb8]

![This map looks the same as the others, but this time the request for our GeoJSON data was completed before the rest of the map was drawn.](https://cdn-images-1.medium.com/max/800/1*tmqaZDxVKdh685JorqwhwQ.png)
(*[View full map](http://bl.ocks.org/maptastik/e7ac58edd5f34fb9487faa711ffc4fb8)*)

Again, our map won’t look any different than it did before, but behind the scenes there is a different story. If we look at our previous map, our data request and rendering of our map happened at the same time. In fact, the map rendering finished before the data request was completed!

![](https://cdn-images-1.medium.com/max/1000/1*1bjLFjwOnOP57NvamyHmoQ.png)

By using `.when()` and `.done()`, we’ve made it so the map does not render until the data request is completed.

![](https://cdn-images-1.medium.com/max/1000/1*2iOttenDMYT3mltt8T364Q.png)

### Adding External GeoJSON to the Map

Now that we’ve set up our app to request the external data first and the rest of the map after that request has has completed, we can finally get to mashing our data onto our map.

With Leaflet, we can use [`L.geoJSON()`](http://leafletjs.com/reference-1.0.3.html#geojson) to add GeoJSON data to a map. Earlier, when we made our AJAX request for data, we stored the result in a variable, `counties`. But if we call…

```JavaScript
L.geoJSON(counties).addTo(map);
```

…we will get an error. Why? Because the value of `counties` is not the raw GeoJSON located at the URL we passed to our request. It is an `XMLHttpRequest` object that contains information about the request. One of the properties of that object is `responseJSON`. We looked at earlier and thought it looked a lot like GeoJSON. What if we try accessing that property with `L.geoJSON()`?

```JavaScript
    L.geoJSON(counties.responseJSON).addTo(map);
```

Assuming our data request is successful, we should get a map of Kentucky counties with Leaflet’s default blue styling.

![Map of Kentucky counties with data loaded from external GeoJSON](https://cdn-images-1.medium.com/max/800/1*zjuy-oWYCck5DcdkVyhIIQ.png)

(*([View full map](http://bl.ocks.org/maptastik/00312131250f9c8b534d97d1be258f1a)*)

[gist:id=00312131250f9c8b534d97d1be258f1a]

With our external data now in our map, we can carry on styling the counties and adding more interaction.

### Concluding Thoughts

I recognize that this method is not ideal for all circumstances. I suspect that this may not be a method that scales well. For instance, by waiting for the external data request to be completed before rendering the rest of the map, the load time of the whole map is increased. Under normal circumstances, an approach that allows for the data and rest of the map to load at the same time should decrease the overall load time. Nonetheless, I feel like this approach offers a nice stepping stone towards faster, yet more complicated methods of mapping external GeoJSON data.

Thinking about Leaflet, the workflow I’ve presented looks similar to the workflow demonstrated in the tutorial [“Using GeoJSON with Leaflet”](http://leafletjs.com/examples/geojson/). In that tutorial, GeoJSON is a variable, hard-coded into the map script. That variable is then passed to the `L.geoJSON()` method. But I would venture to guess that most people do not want to have huge blocks of GeoJSON text in their script. It is messy looking if you have more than a couple point featuers and is not really sustainable if you have a dataset that changes regularly. Furthermore. the workflow demonstrated in that tutorial does not work if you want to grab external data. You have to figure out a new workflow, one which typically requires callbacks. What I like about the method I’ve presented here is that it is similar to the more linear approach used in beginner tutorials. It may have it’s pitfalls, but it allows someone like me to still be able to make a map while trying to better understand more efficient methods of doing so. I hope this post is useful to you as well.

### Other Resources

There are multiple ways to add external data to a map in Leaflet. Here are a few examples and tool for doing just that:

- [“Adding External GeoJSON with
Mapbox.js”](http://lyzidiamond.com/posts/external-geojson-mapbox) — Lyzi Diamond
- [leaflet-ajax](https://github.com/calvinmetcalf/leaflet-ajax) plugin — Calvin Metcalf
- [“Creating a Leaflet.js mapping app from the ground up”](http://zevross.com/blog/2014/10/28/tips-for-creating-leafleft-js-maps/) — Zev Ross
- [“Leaflet: Make a web map!”](https://maptimeboston.github.io/leaflet-intro/) — Ryan Mullins & Andy Woodruff (maptimeBoston)

If you want to avoid using jQuery, I’d recommend checking out [“YOU MIGHT NOT NEED JQUERY”](http://youmightnotneedjquery.com/).
