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

The sequence of steps in the pipeline is:
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

The next step is to build up the list of IDs associated with these cities. For that, the script 
*Cities2Locations.py* converts the entries from *cities.csv* to PyOWM **location** objects. The 
**location** objects are stored in *locations.csv*. Then, *DownloadData.py* performs the queries 
to the OWM API through PyOWM and outputs the results in a csv file, with the date and the current 
time in the filename (e.g. *Temp-2019_01_03-14_19.csv*).

In the end, only one execution of *DownloadData.py* is sufficient to perform a data acquisition, 
provided the *locations.csv* file exists. *Cities2Locations.py* doesn't need to be executed, 
unless the list of cities *cities.csv* has been modified.