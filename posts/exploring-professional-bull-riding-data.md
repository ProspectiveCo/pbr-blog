# Exploring Professional Bull Riding data

## Introduction

Prospective is a tool purpose-built for quickly exploring data, regardless of its source. To illustrate this point, let's use Prospective to explore data from a unique source, Professional Bull Riding.

In most sports, the rules and opportunities are designed to be the same for all participants. In field sports, each team is given the same amount of playing field, or in Baseball's case, given an equal chance to be on offense and defense. Professional Bull Riding is different. In Professional Bull Riding (PBR) the balance between riders and bulls is asymmetrical. Riders will always be on the defense, and the Bulls will always be on the offense. Both riders and bulls have careers, with proper statistics to track their performance.

The [PBR site](https://pbr.com/) includes a statistics page, containing data for each rider and bull across many years. For our purposes, I've chosen to use rider data between the years 2013, and 2023. The data is in the form of JSON, with nested values. Nested values are not supported by [Perspective](https://perspective.finos.org/), so it is necessary to flatten the data.
## Exploring the data

### Total dollars over buck-off (Does it pay to get back on the bull?)

Let's examine if it "pays to get back on the bull". In other words, does the amount of money a rider makes correlate to the number of times they get bucked off? As a first step, let's make a scatter plot of the total dollars earned over the number of times a rider was bucked off. We will group by the rider name, place "total dollars" on the Y axis, "buck offs" on the X axis, "rider rank" as the color, and "events attended" as the size of the dot. For good measure, let's also add a tooltip of the rider's name.

```
EXAMPLE: total-dollars-per-year.json
```

On this graph, we can see there is a trend in both the number of buck-offs and the total dollars earned, but also in rider rank and events attended. This all makes sense, as the more events a rider attends, the more likely they are to get bucked off, and the more likely they are to earn money. To get a better picture of the data, let's normalize the data a bit by dividing the total dollars by the number of events attended. In [Perspective](https://perspective.finos.org/) this is fairly easy to do with [Expression Columns](https://perspective.finos.org/docs/expressions/). Adding a custom column is easy; click the "New Column" button at the top of the columns section in the chart sidebar. Custom columns use a language called [ExprTK](https://github.com/ArashPartow/exprtk) to create expressions. You can do a lot with ExprTK, but for now, we will just use it to create a new column showing the total dollars divided by the number of events attended. Click on the "New Column" button. An additional side panel will open up with a text box. In the text box, enter the following expression:

```
// dollars per event
"statistic.total_dollars" / "statistic.events_attended"
```

When you are done, click `save`, then drag your new custom column to the "Size" section. You should see a nice variety of dot sizes now.

Let's make another custom column to display the inverse of “rank” so that the highest-ranked riders are at the top of the graph. Repeat the steps above, but this time use the following expression:

```
// inverse rank
"statistic.rider_rank" * -1
```

We'll move our "inverse rank" column to the Y axis, and set it to "avg". Now we get a better picture of how buck-offs relate to rank and dollars.

```
EXAMPLE: buck-offs-relative-to-rank.json
```

## Viewing rider's stats over time

One of the great features of Perspective is Global Filters. With Global Filters, we can select a row in one panel and change the filtering of a related chart in a different panel. To kick the tires on this feature, we'll create a means of viewing how much each rider has earned each year.

First, let's create a data grid with our PBR dataset, grouped by rider. We can add/remove any additional columns we want to see while viewing the chart, as the data grid's columns will still be viewable. We will also add the rank column as a reference.

**PRO TIP**: You can filter which columns are shown in the data grid by holding Shift and left-clicking on the column selector (the button to the right of the column block). This will add only the selected column to the grid and remove all others. This is a great way to quickly add or remove columns from the data grid.

Now comes the magic: right-click the data grid and select `Create Global Filter`. You should see the data grid shift to the left side, and a large space open up to the right of it. This is where we will put our chart. Right-click again, and select `New Table -> path/to/our/dataset`. Once selected, a new data grid should appear in the empty cavity.

In the new panel, create a Y Bar chart, with data grouped by year, and the Y axis set to "total dollars". The "Where" field should already be populated with a rider's name. Now when you click on a row in the left panel, the right panel will update with the relevant information!

```json
EXAMPLE: stats-over-time.json
```

## Final thoughts

Perspective is a great tool for exploring data. It is easy to use and has many great features, such as custom columns, and global filters. If you have a dataset you would like to see explored or have any questions about this article, Please feel free to reach out!
