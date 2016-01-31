telco-poc

This is a cartodb deep-insights dashboard illustrating a scenario where a telco provider wants to see their current assets (retail locations and tower sites) overlaid on top of demographic information (census tracts with ACS data)

The following "hacks" are in place that are not built-in deep insights functionality as of 1/31/2016:

Instead of filtering a layer on the map when a user interacts with a histogram widget, the selected ranges are extracted for use in building a SQL query that makes use of a custom PostgreSQL function to calculate "fitness" of a polygon's value within the selected ranges.

The query template is as follows (located in cartodb.js/src/windshaft/windshaft-map.js:

"SELECT a.*, b.median_age, b.median_household_income,range_score(median_age::numeric,{{median_age.min}},{{median_age.max}},0.5) * range_score(median_household_income::numeric, {{median_household_income.min}}, {{median_household_income.max}}) as score FROM nyct2010 a LEFT JOIN megaacs b ON a.geoid = b.geoid"


2) The new SQL is applied to the map layer.  The CartoCSS is already set to visualize a range from 0 to 1, so it does not need to be modified.