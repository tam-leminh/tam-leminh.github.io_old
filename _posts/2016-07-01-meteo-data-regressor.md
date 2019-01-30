---
bg: "meteo.jpg"
layout: post
title:  "Meteo-DR"
crawlertitle: "Meteo-DR - Tâm Le Minh"
summary: "How to interpolate climate data?"
date:   2016-06-29 20:09:47 +0700
categories: posts
author: Tâm Le Minh
type: project
github: https://github.com/tam-leminh/meteo-dr
---

The purpose of Meteo-DR is to make the analysis of meteorological data easy. It implements a 
full pipeline from acquiring weather data for locations across Europe, applying interpolation 
models and visualize them. Meteo-DR provides a framework and all the blocks of a typical machine 
learning pipeline. It is easy to automate it by combining the different functions offered by this 
framework. It is also designed to work well with scikit-learn, so it can be extended either with 
elements from scikit-learn or custom functions.

The pipeline consists of the following sequence of steps:
- weather data acquisition,
- definition of interpolation models,
- prediction applying models on the data,
- optimization of model hyperparameters,
- benchmark report generation,
- visualization of the data.

<p/>
* ToC
{:toc}

## Data acquisition

Open Weather Map provides online weather maps and forecasts. The data is available through 
an API, answering in JSON, XML or HTML. Without subscription, one can still retrieve real-
time weather with a rate of 60 API calls per minute. In order to get the data for one city, 
the easiest way to perform these calls is to pass the city's ID, which is specific to OWM.

PyOWM is a wrapper library for this API, allowing to use the API with Python functions. PyOWM 
also has a database of cities, containing their coordinates and their OWM IDs. It uses 
**location** objects to represent them. 

It is not reasonable to process all the data for European cities. Indeed, this would be a lot 
of data to analyze, while it's not necessarily useful to be that precise. More importantly, 
retrieving the data for all of them would take a lot of time, since the free OWM API key only 
allows for 60 calls per minute. Instead, I selected a handful of cities spread across the 
continent (~1000). These cities are listed in the *cities.csv* file.

<div style="text-align: center">
<img src="/assets/images/cities.png" alt="cities" width="66%">
<em>Cities included in cities.csv</em>
</div>

The next step is to build up the list of IDs associated with these cities. For that, the script 
*Cities2Locations.py* converts the entries from *cities.csv* to PyOWM **location** objects. The 
**location** objects are stored in *locations.csv*. Then, *DownloadData.py* performs the queries 
to the OWM API through PyOWM and outputs the results in a csv file, with the date and the current 
time in the filename (e.g. *Temp-2019_01_03-14_19.csv*).

In the end, only one execution of *DownloadData.py* is sufficient to perform a data acquisition, 
provided the *locations.csv* file exists. *Cities2Locations.py* doesn't need to be executed, 
unless the list of cities *cities.csv* has been modified.

[![meteodr-data_acquisition]({{ site.images | relative_url }}/meteodr-data_acquisition.png)]({{ site.images | relative_url }}/meteodr-data_acquisition.png)
<div style="text-align: center">
<em>Data acquisition process</em>
</div>

## Meteo-DR model framework

The initial goal of Meteo-DR was to build a framework for a full pipeline, including scikit-learn
and custom models. The custom models were motivated by the fact that meteorological data has
geospatial features, namely the lat-lon coordinates of the data points. Some machine learning
techniques are based on the distance between data points, for instance, nearest neighbors
algorithms or kernel methods. These distance are usually calculated with the $$L_2$$-norm. That
means, for two data points $$x_1$$ and $$x_2$$ whose features consist of lat-lon geographic
coordinates, the $$L_2$$ distance between them is:

$$d(x_1, x_2) = \sqrt{(lat_2-lat_1)^2 + (lon_2-lon_1)^2}$$

which is not really representative of the actual geographical distance. This is simply because
latitude and longitude are not cartesian coordinates and we cannot assume that the surface of Earth
is flat at a continental scale. Yet, we want to find a way to introduce the hypothesis that points
are geographically correlated and closer points are more correlated than remote points.

One technique would be to use the WGS 84 database which defines a model of the shape of the Earth
(reference ellipsoid) and geographic coordinate system based on this. Several Python libraries, such
as LatLon, can be used to calculate a precise distance between two points on Earth. However, this
calculation is computationally expensive and would significantly slow the interpolation algorithms
down.

Meteo-DR makes a compromise, by assuming the Earth is a perfect sphere. Therefore, the distance
between two points on its surface (Great Circle distance) can be computed with the Haversine formula:

{% highlight python %}

def hv_distance(lat1, lon1, lat2, lon2):
    radius = 6371
    dlat = np.radians(lat2-lat1)
    dlon = np.radians(lon2-lon1)
    a = np.sin(dlat/2) * np.sin(dlat/2) + np.cos(np.radians(lat1)) \
        * np.cos(np.radians(lat2)) * np.sin(dlon/2) * np.sin(dlon/2)
    c = 2 * np.arctan2(np.sqrt(a), np.sqrt(1-a))
    d = radius * c
    return d

{% endhighlight %}
<div style="text-align: center">
<em>Haversine formula implementation</em>
</div>

Despite being more complex than the $$L_2$$-norm, the calculation time is still acceptable. Especially,
after vectorization, the calculation of pairwise distances between the elements of two vectors is very
efficient. In the end, it also provides a way better estimate of the distance between two points
referenced by their lat-lon coordinates.

[![greatcircle]({{ site.images | relative_url }}/greatcircle.png)]({{ site.images | relative_url }}/greatcircle.png)
<div style="text-align: center">
<em>Great circle distance</em>
</div>

However, not every regression model makes use of the distances (e.g. regression trees, regression splines,
etc.). Therefore it is useless to reimplement these models, when we could use scikit-learn. Meteo-DR
custom models are built with some common functions from scikit-learn:
- *fit*,
- *predict*,
- *set_params* and *get_params*.

This allow Meteo-DR to define a common wrapper, called **LearningFramework**, which acts as an upper functional
layer. Its main functions are:
- *score*, computes a score based on a error or loss function (such as mean squared error or can be defined),
- *train*, fits the model on the data, can compute a (training) score,
- *predict*, applies a trained model on a set of data, can compute a (test) score,
- *optimize*, performs a search of optimal parameters, which values have to be specified (ranges or list of values).

## Benchmarking

## Visualization


[![meteodr-pipeline]({{ site.images | relative_url }}/meteodr-pipeline.png)]({{ site.images | relative_url }}/meteodr-pipeline.png)
<div style="text-align: center">
<em>MeteoPipeline steps</em>
</div>