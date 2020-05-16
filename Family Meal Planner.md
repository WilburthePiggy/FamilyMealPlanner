# Family Meal Planner

Goal: Automate the weekly meal planning process.
Language: HTML and friends
Form: Webpage. maybe host it on a github

```
Daily Notes
-- 10/5/20 Sunday --
Table is up. Replacer is up.
The meal plan is an array with 7 objects containing: day, vege, meat, soup, extra.
Next steps:
automated array builder
In parallel with Building Up a List of Foods - meat, vege, soup

-- 11/5/20 Monday --
Starting the database. I'm having difficulty thinking up an efficient way of choosing the meals. What are our rules? See the section on Schema and Algo.

-- 14/5/20 Thursday --
nextDay() done. Will be doing the one-day-planning with planDay(). Need to reference the algo rules below. 
Update: Day planning works, but i can't get the proposed dishes to fit properly into plannedMeal. 
Update 2 (2.05pm): works now.
Update 3 - find out how to deep copy
		- copy into the table before carrying on
		JSON.parse(JSON.stringify(food))
--15/5/20 Friday --
Janky parse-stringify solved the problem.
Next steps:
1. Live google-sheet json sauce https://www.freecodecamp.org/news/cjn-google-sheets-as-json-endpoint/
2. freezing values
3. CSS.

--16/5/20 Saturday--
https://spreadsheets.google.com/feeds/list/19hNEkTIdGchbbCHyzZCBpZidnI1WnCY9-rgO93lNVp4/od6/public/values?alt=json

https://docs.google.com/spreadsheets/d/e/2PACX-1vRl2LZIbrXFEHJNUalAm_ndTJd52E1t-KIMbf0Duu9XnFPYfqnCr9D1Q4ImmNQEMQVCkER6Ajc9I6rE/pubhtml

https://docs.google.com/spreadsheets/d/19hNEkTIdGchbbCHyzZCBpZidnI1WnCY9-rgO93lNVp4/edit#gid=0

sprsht id: 19hNEkTIdGchbbCHyzZCBpZidnI1WnCY9-rgO93lNVp4

https://spreadsheets.google.com/feeds/worksheets/19hNEkTIdGchbbCHyzZCBpZidnI1WnCY9-rgO93lNVp4/public/basic?alt=json

https://spreadsheets.google.com/feeds/list/19hNEkTIdGchbbCHyzZCBpZidnI1WnCY9-rgO93lNVp4/od6/public/values?alt=json

https://docs.google.com/spreadsheets/d/19hNEkTIdGchbbCHyzZCBpZidnI1WnCY9-rgO93lNVp4/gviz/tq?tqx=out:csv
```

[TOC]



## Requirements

Critical

1. Generate a table and automatically fill it.
   1. Recommendations fulfil the Meat-Vege minimum and no soup-soup rules.
2. Have a database of different foods, with different associated traits
   1. Traits include: Meat/Vege/Soup, Minimum size requirements(?)

Additional

1. No repeats within the same week.
2. Preferred pairs? Like weightages?
3. Minumum size? So you take in the no. of people eating as a factor
4. 'force fill' - like someone already wants something?

Not a requirement

1. Recommend new dishes

## Research

### Database:

Probably a json object with the keys: name, type, effort-value (?) 

Have to populate with all the meals that my family comes up with. Maybe think of:

1. Type of vege/meat/soup
2. Think of the many ways we cook em

### Tables:

How do tables work in HTML and JS?
How do we automate the construction of tables?
Does this process require indexing of the JSON object?

https://www.w3schools.com/jsref/met_table_insertrow.asp

// Find a <table> element with id="myTable":
var table = document.getElementById("myTable");

// Create an empty <tr> element and add it to the 1st position of the table:
var row = table.insertRow(0);

// Insert new cells (<td> elements) at the 1st and 2nd position of the "new" <tr> element:
var cell1 = row.insertCell(0);
var cell2 = row.insertCell(1);

// Add some text to the new cells:
 cell1.innerHTML = "NEW CELL1";
cell2.innerHTML = "NEW CELL2"; 

loop wrt to a json object?

### Google Sheet at JSON source. 

https://coderwall.com/p/duapqq/use-a-google-spreadsheet-as-your-json-backend 

### Sample?

| Day     | Vege | Meat | Soup | Additional |
| ------- | ---- | ---- | ---- | ---------- |
| Monday  | Blah | Balh | Blah | Blah       |
| Tuesday |      |      |      |            |


So:

tHead - always 

<th>Day</th> <th>Vege</th><th>Meat</th><th>Soup</th><th>Additional</th>

tBody - always:

<tr>

<td> Day</td><td>vege from where? </td><

## Database and Algorithm

### Rules

What are the rules when deciding the menu? 

1. There must be vege and meat in the menu.
2. There must be enough food for the family
3. Soups cannot be consecutive
4. Alternate between white and green vege/cannot be consecutive
5. Mushrooms and beans are a no when papa is around -> screw this
6. You cannot have a pot-on-pot. Eg: Black sauce egg and taoki soup. They must be mutually exclusive.

Proposed schema:
Every food item has a vege, K (vitamin), meat and soup weight value.

#### For a day:

1. Check the existing values on the day

   | Day  | Vege | K    | Meat | Soup | Extra |
   | ---- | ---- | ---- | ---- | ---- | ----- |
   | 0    | 0    | 0    | 0    | 0    | 0     |

   

2. Randomly select an item. Check if it takes any of the values above their thresholds. 1.5 for vege, meat, 1 for K, soup and extra.

   1. If true, repeat.
   2. Else, add to menu the slot matching the class.

3. When Vege, Meat and Soup are all at least 1, then move to the next day.

#### Shifting to the next day:

Carry over the previous day's K and Soup. Reduce K by 0.5 

```
Reducing K by 0.5, while all K values are at 1 means 
Day K = 0: can add one K
Day K = 0.5: no K. On cooldown.
Day K = 1: already have K. No more.
```

Reduce Soup by 0.5

```
Reducing Soup by 0.5 also works like K. However, pseudo-soups like black sauce carry 0.5 weight so:
Day Soup = 0.5 -> leftover soup available. 
Pseudo-soup will enter, but a full soup cannot.
```

When Day = Saturday (index = 6), end loop.

--

#### Item Schema

| Name      | Class                   | Vege        | K                | Meat      | Soup                          | Extra                              |
| --------- | ----------------------- | ----------- | ---------------- | --------- | ----------------------------- | ---------------------------------- |
| food name | which slot will it take | is it vege? | is it vitamin k? | yeah boi. | pseudosoups 0.5, true soups 1 | items with egg, beans and the like |

#### Day Schema

| Day  | Vege | Meat | Soup | Extra | Vegevalue | Kvalue | Meatvalue | SoupValue | ExtraValue |
| ---- | ---- | ---- | ---- | ----- | --------- | ------ | --------- | --------- | ---------- |
|      |      |      |      |       |           |        |           |           |            |

The day doesn't care what the individual items' values are. just the sum.