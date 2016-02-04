#labs-telco-insights

This is a cartodb deep-insights dashboard proof of concept illustrating a scenario where a telco provider wants to see their current assets (retail locations and tower sites) overlaid on top of demographic information (census tracts with ACS data)

The PoC makes use of a hacked version of cartodb.js (deepinsights.js includes cartodb.js), which is available on the [telco branch of cartodb.js](https://github.com/CartoDB/cartodb.js/tree/telco)

Most of the logic is in the [createInstance] method of [windshaft-map.js](https://github.com/CartoDB/cartodb.js/blob/telco/src/windshaft/windshaft-map.js#L38)

The idea is to evaluate which ranges have been selected from the histogram widgets, and then gauge the "fitness" of each polygon to the selected criteria.  A polygon's fitness for a given attribute is calculated using the following algorithm, resulting in a numerical value between 0 and 1.  (1 meaning it is within the selected range.  As the value gets farther from the selected range, the output will get smaller and closer to 0)

```
create or replace function range_score(x Numeric, xmin Numeric, xmax Numeric, sigma Numeric DEFAULT  0.5) returns numeric as $$

begin 

	if x < xmin then 
		return exp(-1.0*power((x-xmin)/(sigma*xmin),2));
	ELSIF x>xmin and x < xmax then
		return 1;
	else
		return exp(-1.0*power( (x-xmax)/(sigma*xmax),2));
	end IF;
end;


$$
LANGUAGE plpgsql
--by Stuart Lynn!!
```

Since we have many histogram widgets, we can multiply several `range_score` values together and get an aggregate score that is also somewhere between 0 and 1.

Then the filters are updated, deepinsights.js passes a `filters` URL parameter, and windshaft makes a new instance of the map.  This hack does not send the `filters`object, rather it manually builds a new SQL query based on the `range_score` calculations above, and sends that to Windshaft.  Windshaft updates the layergroupid and the map refreshes.  The CartoCSS remains static, rendering a choropleth on the aggregte score with lighter values when the score is closer to 1, and darker when it is closer to zero

