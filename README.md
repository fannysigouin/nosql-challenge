# NoSQL Challenge: Eat, Safe, Love

This repository contains my work for the 12th challenge of the UofT SCS edX Data Bootcamp.

## Background

The UK Food Standards Agency evaluates various establishments across the United Kingdom, and gives them a food hygiene rating. You've been contracted by the editors of a food magazine, Eat Safe, Love, to evaluate some of the ratings data in order to help their journalists and food critics decide where to focus future articles.

## Summary

This challenge was completed in three parts: setting up, updating, and analyzing the database.

### Part 1: Database Set Up

This first part of the challenge was completed in `NoSQL_setup_starter.ipynb`. The data found in Resources > establishments.json was imported from the Terminal (command can be found at the top of the Jupyter Notebook). In the Jupyter Notebook, the necessary libraries were imported, an instance of Mongo Client was created, and the database and collection were confirmed to be created and loaded properly. The establishments collection was then assigned to a variable in preparation for the next parts.

### Part 2: Updating the Database

This second part was also completed in `NoSQL_setup_starter.ipynb` to fulfill the magazine editors' requests to modify the database. A new restaurant, Penang Flavours, was added to the database (using `db.collection.insert_one()`) and its business ID was updated (using `db.collection.update_one()`).

The magazine is not interested in any establishments located in Dover. The number of documentations containing Dover as the Local Authority was found and the documents were then removed from the database (using `db.collection.delete_many()`). Lastly, number values that were stored as strings (`geocode.latitude`, `geocode.longitude`, and `RatingValue`) were updated to be stored as decimals (for latitude and longitude, using `$toDouble`) and as integers (for RatingValue, using `$toInt`).

### Part 3: Exploratory Analysis

This last part was completed in `NoSQL_analysis_starter.ipynb` to answer specific questions from the magazine in order to help them find locations to visit or avoid. After building the query to get the resutls for each question, the results were also converted to a Pandas dataframe.

#### Considerations
RatingValue refers to the overall rating decided by the Food Authority and ranges from 1-5. The higher the value, the better the rating.
The scores for Hygiene, Structural, and ConfidenceInManagement work in reverse. This means, the higher the value, the worse the establishment is in these areas.

#### Questions
1. Which establishments have a hygiene score equal to 20?
This question was answered by querying the database using this query: `query = {'scores.Hygiene': 20}` which identified 41 such establishments.

2. Which establishments in London have a RatingValue greater than or equal to 4?
This question was answered by querying the database using this query: `query = {'LocalAuthorityName': {'$regex': 'London'}, 'RatingValue': {'$gte': 4}}` which identified 33 establishments.

3. What are the top 5 establishments with a RatingValue of 5, sorted by lowest hygiene score, nearest to the new restaurant added, "Penang Flavours"?
To answer this question, the database was first queried to find the longitude and latitude of Penang Flavours. They were then stored in variables to form another query in order to identify the nearest establishments, in addition to filtering RatingValue greater than or equal to 4. The results were also sorted by ascending hygiene score (as the lower the score, the better the establishment) and limited to only 5 results. 

```
query = {'RatingValue': 5, 'geocode.latitude': {'$gte': (latitude - degree_search), '$lte': (latitude + degree_search)}, 'geocode.longitude': {'$gte': (longitude - degree_search), '$lte': (longitude + degree_search)}}
sort = [('scores.Hygiene', 1)]
limit = 5
results = list(establishments.find(query).sort(sort).limit(limit))
```

The top 5 establishments are: Howe and Co Fish and Chips, Lumbini Grocery Ltd T/A Al-Iman, Atlantic Fish Bar, Iceland, and Volunteer.

4. How many establishments in each Local Authority area have a hygiene score of 0? 
An aggregation pipeline was created with:
- A match query to filter only establishments with a hygiene score of 0.

    `match_query = {'$match': {'scores.Hygiene': 0}}`

- A group query to group the results by Local Authority and keep track of their count of establishments matching the match query.

    `group_query = {'$group': {'_id': '$LocalAuthorityName', 'count': {'$sum': 1}}}`

- Sorted values from highest to lowest by count.

    `sort_values = {'$sort': {'count': -1}}`

The first 5 results were:
- Thanet: 1130 establishments
- Greenwich: 882 establishments
- Maidstone: 713 establishments
- Newham: 711 establishments
- Swale: 686 establishments

## References
[UK Food Standards Agency](https://www.food.gov.uk/) (2022). [UK food hygiene rating data API](https://ratings.food.gov.uk/open-data/en-GBLinks). Contains public sector information licensed under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/). Accessed Sept 9, 2022 and Sept 12, 2022 with the establishment settings as follows: longitude=51.5072, latitude=-0.1276, maxdistancelimit=4567, pagesize=10000, sortoptionkey=distance, pagenumber=(1,2,3,4,5,6,7,8).