I've been walking around with a Fitbit Charge 2 for over two years, and figured it would be interesting play with all the data that it collects. This is an exploratory tutorial on how to do that, done as an R markdown file - with a bit of tinkering, you can use it on your data!

### Downloading the data

<details> <summary>Click to expand the section on downloading the data</summary>

Follow the instructions [here](https://help.fitbit.com/articles/en_US/Help_article/1133): set up an account on [fitbit.com](fitbit.com), if you don't already have one, then log in and click on the settings gear (top right). Click `Settings` and `Data Export`, then `Request My Data`. You will need to confirm the request through an email from them. They will assemble the data for export and send you another email when it's ready (happened within an hour or two for me). There will be a link to download the data on that same `Data Export` page.

The data is a zipped archive with a `YourName` folder, and within it, the `user-site-export` folder contains the relevant data files in JSON format. If you want to process your own data using this vignette, unzip and rename `YourName` to `data` and place it in the same folder as the .Rmd file. </details>

Steps
=====

<details open> <summary>How much do I walk, and when?</summary>

The fitbit records how many steps per minute I walk. What does that look like? Figure 1B shows the overall heatmap, where each thin horizontal line represents time along each day, and the colors represent the steps/min. We can already see some clear patterns of activity at the same times of day - get up, go to work, go to lunch, go home. That's for the weekdays (not a ton of walking on the job). When averaged separately for each day of the week (Figure 1A), we can see these walking spikes at particular times again, and they are not present for the weekends. But on some days, there's a lot of walking in general - those are the horizontal red streaks in Figure 1B, also seen in the total steps for each day (Figure 1C, colored by the walking pace of the steps). That's hiking on the weekends. We can also see that there is more walking on Sat and Sun overall, compared to weekdays (total area for each sliver in Figure 1A). But are all the weekends like that? How similar are the weekends and the weekdays to each other? If we apply principal component analysis and stratify the days by their first component (i.e. line them up by their most prominent differences), we can take the top, middle and bottom 10% of the days and average within those groups. That is done separately for weekdays and weekends in Figure 1D. We can see that the weekdays are more similar to each other (although still variable), but the weekends vary the most. There is still couch time on some weekends.

![](Fitbit-analysis/images/steps1-1.png) </details>

Heartrate
=========

<details open> <summary>When is it high and low?</summary>
The heartrate heatmap looks similar to the steps heatmap - obviously, heartrate goes up when I walk and hike (Figure 2B). You can also see the peaks and valleys in the profile for each day of the week (Figure 2A), for the weekdays. On the weekends, the heartrate is generally higher (and that is good?!) An interesting and somewhat unexpected observation is that the heartrate during sleep is much higher for the weekends (and somewhat higher on Mondays). One possibility is that it's because of the beers on the evenings before? Alcohol raises your heartrate, but I did not think it would be that much.

![](Fitbit-analysis/images/heartrate1-1.png)

How does the steps/min and heartrate data correlate with each other? If we plot a heatmap of their combinations (Figure 3), with heartrate on the x axis, steps/min on the y, and the color denoting how many times that combination is encountered, we can see two main regimes that I'm in: a huge blob of low steps/min and (mostly) low heartrate, i.e. just sitting around. And a cloud at 100-120 steps/min and higher heartrate - i.e. walking. Note that there is also a high count of 0 steps/min at higher BPMs - that's likely bicycling, and resting after walking.
![](Fitbit-analysis/images/steps-bpm-1.png) </details>

Sleep
=====

<details open> <summary>Any interesting patterns in the sleep?</summary>
The Fitbit records sleep data in the form of levels - deep, light and REM sleep. The sleep heatmap, color coded by these levels, is fairly uniform (Figure 4A). If you average out the levels over time, separately for weekdays and weekends (Figure 4B), you can see that I go to sleep earlier and get up earlier on the weekdays - no surprise. Within the night, the deep sleep tends to happen earlier in the night, with more REM sleep toward the morning - as expected.

![](Fitbit-analysis/images/sleep-1.png) </details>

### Other data and explanation of data structures

<details> <summary>Expand if you need to know the geeky details</summary>

The data is in a series of .json files, many with dates, and others without. The files with dates at the end are in the following categories:

    ##  [1] "altitude"                  "calories"                 
    ##  [3] "demographic_vo2_max"       "distance"                 
    ##  [5] "heart_rate"                "height"                   
    ##  [7] "lightly_active_minutes"    "moderately_active_minutes"
    ##  [9] "resting_heart_rate"        "sedentary_minutes"        
    ## [11] "sleep"                     "steps"                    
    ## [13] "time_in_heart_rate_zones"  "very_active_minutes"      
    ## [15] "weight"

`height` and `weight` simply contain the values that I input initially, nothing more (perhaps some of these things would get automatically updated with some advanced gatgetry?) `altitude` for me only has a few files with nonsensical values (my phone's GPS is mostly off), but this could be potentially interesting to look into for BPM, etc at higher-altitude hikes.

<details> <summary>`calories`</summary>

The `calories` data generally matches the heartrate data (although it is not a perfect correlation), and is likely calculated based on bpm, steps/min and possibly other stuff.
![](Fitbit-analysis/images/calories-1.png)![](Fitbit-analysis/images/calories-2.png)![](Fitbit-analysis/images/calories-3.png) </details>

<details> <summary>`demographic_vo2_max`</summary> `demographic_vo2_max` is apparently oxygen uptake values, which must be also calculated from the measured data. I saw no obvious correlations with total steps/day.
![](Fitbit-analysis/images/vo2max-1.png) </details>

<details> <summary>`distance`</summary> The `distance` data seems to be very linearly calculated from the steps/min data.
![](Fitbit-analysis/images/distance-1.png) </details>

<details> <summary>`..._minutes`</summary> The `sedentary_minutes`, `lightly_active_minutes`, `moderately_active_minutes` and `very_active_minutes` files contain daily minute tallies of these activity categories, presumably calculated from the heart rate data.
![](Fitbit-analysis/images/minutes-1.png) </details>

<details> <summary>`time_in_heart_rate_zones`</summary> `time_in_heart_rate_zones` is exactly that, how many minutes in each day were spent in four different heart rate zones. How the numeric boundaries of each zone are defined is not clear.
![](Fitbit-analysis/images/time_in_zones-1.png) </details>

The files without dates are:

    ## [1] "badge.json"        "exercise-0.json"   "exercise-100.json"
    ## [4] "exercise-200.json" "exercise-300.json" "exercise-400.json"
    ## [7] "exercise-500.json"

The `badge` file contains all the badges you've earned. The `exercise` files contain the processed info on all the exercise periods you've had. This may be interesting to look into later. </details>

Acknowledgements
================

This exercise has been fun because of all the open source tools and code out there that I happily used, so thanks to the creators of all of it -- ggplot, egg, etc, etc. Thanks to countless webpages that went into details on how to solve particular coding problems - I've relied on that heavily.

<details> <summary>Some of the most useful webpages:</summary> <https://community.fitbit.com/t5/Fitbit-com-Dashboard/Working-with-JSON-data-export-files/td-p/3098623>
<https://cran.r-project.org/web/packages/egg/vignettes/Ecosystem.html>
<https://cran.r-project.org/web/packages/egg/vignettes/Overview.html>
<https://stackoverflow.com/questions/47614314/how-to-put-plots-without-any-space-using-plot-grid>
<https://cran.r-project.org/web/packages/cowplot/vignettes/shared_legends.html>
</details>
