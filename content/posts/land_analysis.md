---
title: "Land Analysis (In Progress)"
date: 2022-06-27
summary: Extracting publicly available county parcel data to gain insights into real estate sales.
showtoc: true
draft: false
---

Currently completed code can be found [here]("https://github.com/trey-capps/land-analysis").

## Overview

This project will consist of two parts:
1. Extracting county data across the Triangle area and creating a Data Warehouse on AWS
2. Use machine learning to predict parcels that will be likely to sell or are currently undervalued

## Extract
Data can be extracted directly from the county's website using the ```wget``` command. A python script can be build around this to improve flexability and organize the extraction process. For the initial data extraction a bash script was created to scrape for 3 counties.
