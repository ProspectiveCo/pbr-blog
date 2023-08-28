# Preparing data to be used in Prospective

## Introduction

Prospective (and Perspective) are great tools for visualizing data, but the data needs to be in a format that is supported by the tool.

This article will cover how to prepare nested JSON data spread over multiple files to be used in Prospective.

I've chosen to do the data manipulation in Python, using the [pandas](https://pandas.pydata.org/) library. Pandas is a great tool for manipulating data and is used by many data scientists. It's also a great tool for preparing data to be used in Prospective. Simple Python script will work just fine, but I find that using Jupyter notebooks is a great way to prepare data, as each step can be documented, and the code can be run in sections.

## The Data

For this article, we've chosen to use data from the [Professional Bull Riding](https://pbr.com/) website, as it's formatted well to illustrate a few points.

Rider and bull data is stored in JSON file separated by year, with filenames matching the form `<subject>-<year>.json`, e.g. `bulls-2021.json`. Our file tree looks something like this:

```
.
├── bulls
│   ├── bulls-2013.json
│   ├── bulls-2014.json
│   ├── bulls-2015.json
|   ...etc
├── riders
│   ├── riders-2013.json
│   ├── riders-2014.json
│   ├── riders-2015.json
|   ...etc
```

An example entry of bull data looks like this:

```json
{
  "data": [
    {
      "rank": 1,
      "statistic": {
        "bull_id": 12093,
        "bull_name": "Code Blue",
        ...etc
      },
      "bull": {
        "id": 27876,
        "name": "Code Blue",
        "number": "WWB-644",
        "brand": "644",
        ...etc
        "contractor": {
          "id": 951,
          "key": "WWB",
          "name": "Walton & Wagoner / Berger & Struve",
          "website": null,
          "country": "USA"
        }
      },
      "series": null,
      "season": {
        "id": 14,
        "sort": 25,
        "title": "2010",
        ...etc
      },
      "contractor": {
        "id": 951,
        "key": "WWB",
        "name": "Walton & Wagoner / Berger & Struve",
        "website": null,
        "country": "USA"
      }
    },
  ]
}
```

## Preparation

There are a couple of issues that must be corrected before we can ingest the data into Prospective:

1. The data is spread across multiple files and needs to be combined into one file
2. All data is contained in a top-level `data` field, which needs to be removed
3. The data is nested and needs to be flattened so that it can work as tabular data.

### Importing libraries

First, let's start by importing some libraries we'll need.

```python
import os
import json
import pandas as pd
import pyarrow
```

### 1. Removing the top-level `data` field

We'll use the `os` library to iterate over each file, and the `json` library to read the data, then write it back out without the top-level `data` field. In this particular case, we no longer want to keep the original file data, so we'll overwrite them.

```python
subject = "bulls" # or "riders"
path = "./" + subject
dir_list = os.listdir(path)

# flatten out data stored in a json object matching the form { data: [...] }
for file in dir_list:
    with open(path + "/" + file, 'r') as f:
        data = json.load(f)
        data = data['data'] # remove the top level "data" field
        with open(path + "/" + file, 'w') as fw:
            json.dump(data, fw)
            fw.close()
        f.close()
```

### 3. Normalization

Our data is now one level flatter, but we still need to flatten it further. This process is called `normalization`, and is often done with data that is nested so that it can be used in a relational (tabular) database. Pandas has a method called `json_normalize` that will do this for us. To make things easier for ourselves in the future (wink, wink) we'll also add a column to the data that contains the year the data is from.

```python
for file in dir_list:
    # Only process json files
    if file.endswith('.json'):
        with open(path + "/" + file, 'r') as f:
            data = json.load(f)

            # pull the year from the file name, which looks like this: bulls-2021.json
            year = file.split('-')[1].split('.')[0]

            # parse the year into a number
            year = int(year)

            # Add the year to each record in data
            for record in data:
                record['year'] = year

            # normalize the data
            df = pd.json_normalize(data)

            # remove extension from file name
            file = file.split('.')[0]

            # export the normalized data to a json file
            df.to_json('./' + subject + '/normalized/' + file + '-flat.json', orient='records')

            f.close()
```

Quick note: I've chosen to save the normalized data to a separate folder so that we can easily combine all the data into one file. This is not necessary and can be combined with the following step.

### 2. Writing To a Parquet File

Now that we have our normalized data saved to separate files, we can combine them into one file. We'll use pandas to read each file, and then append it to a dataframe. Once we have all the data in one dataframe, we'll save it to a parquet, arrow, or CSV file.

Why are we using parquet/arrow files? Parquet files are columnar, which means that each column is stored separately. This makes it easy to read only the columns we need and is more efficient than reading the entire file. Arrow files are similar, but are binary files, and are more efficient than parquet files. Both parquet and arrow files are supported by Perspective.

```python
# Create a single dataframe from all the normalized JSON files
path = './' + subject + '/normalized/'
files = os.listdir(path)
files = [path + f for f in files]
df = pd.concat([pd.read_json(f) for f in files], ignore_index = True)

# export the dataframe into a parquet file
df.to_parquet('./' + subject + '/' + subject + '-normalized.parquet')

# OPTIONAL: export the dataframe to arrow and CSV files
# NOTE: "feather" and "arrow" are the same format, so the dataframe.to_feather() method will work just fine.
# df.to_feather('./' + subject + '/' + subject + '-normalized.arrow')
# df.to_csv('./' + subject + '/' + subject + '-normalized.csv')
```

Our normalized data can now be used as a data source in Prospective!

## Bonus: Exploring the data with Jupyter/Perspective

If you are using a Jupyter Notebook, you can use the following code to explore the data in Perspective. This code will create a Perspective table, and load the data from the parquet file we created above.

```python
import pandas as pd
import pyarrow
import perspective

# load the data from the parquet file
df = pd.read_parquet('./bulls/bulls-normalized.parquet')
widget = perspective.PerspectiveWidget(df)
widget
```

