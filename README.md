# Afghan-Streamlined
## Content

- [Why Afghanistan?](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#why-afghanistan)
- [Before starting](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#before-starting)
- [Querying the Data](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#querying-the-data)
- [Overview](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#overview)
- [Flattening the json](https://github.com/gabriel1200/Afghan-Streamlined/blob/master/README.md#flattening-the-json)
- [Translating the Tweets](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#translating-the-tweets)
- [Cleaning & Selecting the Data](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#cleaning--selecting-the-data)
- [Calculating Sentiment](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#calculating-sentiment)
- [Merging Column & Sentiment Data](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#merging-column-data-and-sentiment-data)
- [Visualizing the Data](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#visualizing-the-data)
- [What we learned](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#what-we-learned)
- [Appendix](https://github.com/gabriel1200/Afghanistan/blob/master/README.md#what-we-learned)

## Why Afghanistan?
- Large sample, able to retrieve over 400k tweets within (x) kilometers of the country. 
- Continuous change with clear inflection points. The country was taken over throughout the month(allowing us to compare sentiment by day) and had multiple incidents that generated high traffic (allowing us to compare sentiment surrounding key events)
- Allowed us to showcase Watson’s translation functionality & identify sentiment in non English languages.

**Software Used**

All scripting was done in **Python**, using **Jupyter Notebook** for live development and **Pandas** for data manipulation. The project was hosted on the **CP4D** platform, with all dashboards created via **Cognos** on that platform.

**What you'll need **
 - A jupyter notebook setup on your local computer.
 -  Access to a CP4D cluster
 -  _Twitter API Developer Key to pull the query_* (For the purposes of the cookbook, we'll link the csv file with our query results instead)
## Querying the Data

**Skip this step** when creating your cookbook. It's included only for completion and replication purposes.
Used Twitter API to pull 374k tweets between July 30th and August 30th, all of which were
-  within 25 kilometers of Kabul
-  had their location set to Afghanistan
-  had profiles that were derived to be from Afghanistan


```python
with open('creds.json') as f:
    creds = json.load(f)
payload = {}

#loading in key values from the json needed to make the query
headers = {
      'Authorization': f"Bearer {creds['bearer_token']}",
      'Cookie': f"personalization_id={creds['personalization_id']}; guest_id={creds['guest_id']}"
    }

base_url = f"https://api.twitter.com/1.1/tweets/search/30day/dev.json?fromDate=202109040000&toDate=202110040000&query=point_radius:[-82.3666 23.1136 25mi] -is:retweet -is:reply OR profile_country:AF -is:reply -is:retweet profile_locality:Afghanistan
'''the query for the data, collecting all tweets 
for the month of August, within 25 miles of Kabul 
with a derived location of Afghanistan'''

count = 1
check=True
while check==True:
    if count == 1:
        url = base_url
#using the base url for the first request
    
    else:
        url = base_url + "&next={0}".format(next_page)
#combining the base url and the url extension for a new auery   
    
    response = requests.request("GET", url, headers=headers, data = payload)
#retrieving the query data
    
    jsonfile_name = f'data/data{count}.json'
    
    d = json.loads(response.text)
#loading in the json data
    
    if d['results']:
        with open(jsonfile_name, 'w') as f:
            json.dump(d, f)

#loading the flattened json data into the file
        next_page = d['next']

#saving the url extension for the next page, used to create the query in the next iteration
    elif d['error'] or len(d['next']) == 0:
        print(d)
        check==False
        break
        
    count += 1
    time.sleep(5)
```

## Flattening the json
-Reads in the Json data gathered in the previous step

```python
import json
import numpy as np
import pandas as pd
import glob
data_dir = 'data/json_data'
def flatten_json(y):
    out = {}

    def flatten(x, name=''):
        if type(x) is dict:
            for a in x:
                flatten(x[a], name + a + '_')
        elif type(x) is list:
            i = 0
            for a in x:
                flatten(a, name + str(i) + '_')
                i += 1
        else:
            out[name[:-1]] = x

    flatten(y)
    return out
df_list = []
for filepath in glob.iglob(f'{data_dir}/*.json'):
    with open(filepath) as json_data:
        d = json.loads(json_data.read())
        json_data.close()
    for i in d['results']:
        i = flatten_json(i)
        df_list.append(i)
    
df = pd.DataFrame(df_list)
df.shape
df.to_csv('data/afghan_csv_full.csv',index=False)

#
```
