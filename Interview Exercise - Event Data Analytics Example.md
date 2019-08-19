
## Event Data Analytics Exercise

The exercise is to undertake analysis of event data very similar to the data analysed within the Tobii ATEX project. 
The data is received as a stream of events related to an individuals browsing behaviour, with each event having a number
of properties which describe the users browsing.
<br>

We would encourage you to work on three analytical questions and explore the data to try and answer them. When working
on these questions we are particularly interested in your approach and conclusions along with your analysis. Suggestions
of questions to analyse appear at the bottom of this document. Please choose the questions as follows:
    
    1. Answer one of the `Analytical Question Suggestions`
    2. Answer one of the `Modeling Question Suggestions`
    3. Answer one  question of your own design which you find interesting

<br>
As part of your interview with Tobii we will ask you to present and we will discuss this analysis for a period of one
hour.

<br>

### FAQ's

1. Do I have to conduct this in Python or complete the analysis in a jupyter notebook?. <br> 
_No we would encourage you to undertake the analysis in a language, tool or environment you feel comfortable using_
a jupyter notebook in python has only been used here to help illustrate the task<br> 

1. Should I just limit this analysis to 3 questions and how long should I spend?. <br> 
_We would encourage you to only complete 3 tasks as written above, the tasks are broad enough to allow you to explore
the data and the analysis should be short enough to present in half an hour (to allow for half an hour discussion)_

### Event Data Model

The event data can be found within the file `sample_data_120819.csv`. It contains the following fields:
    
- `event_time_stamp` :- ISO formatted timestamp of when the event occured **[timestamp]**
- `beacon_id` :- id for the beacon this event is related too **[str] **
- `content_type` :- content_type of the event **[categorical str]**
- `content`:- content object containing descriptors of the event  **[JSON object]**
    
<br>

In order to understand these events it is important to note that they desrcibe the behaviour of **beacons** which appear
within an individuals browsing sessions. A **beacon** is an area of the website that a user is browsing, which is of
interest to the ATEX project, normally an advertisement. We monitor these advertisements and record the following events
for them (which can be found as the variable `content-type` in the attached dataset):

* `application/vnd.tobii.atex.beacon-detected-event-v1+json`:- initial rendering of the beacon on the webpage
* `application/vnd.tobii.atex.beacon-updated-event-v1+json`:- the beacon is modified, normally html changes
* `application/vnd.tobii.atex.beacon-removed-event-v1+json`:- the beacon is removed from the webpage
* `application/vnd.tobii.atex.beacon-in-view-event-v1+json`:- the beacon becomes visible within the individuals screen


There is also a final content-type `application/vnd.tobii.atex.session-started-event-v1+json` which is not associated
with a beaco. This event is triggered when a new url being is loaded in the browser. This defines a new **session** within the data. These
two concepts **sessions** and **beacons** are related by a hierarchy where every beacon should be assoicated with a 
session.


### Example Initial Analysis

The following cells contain a simple initial analysis of the data to illustrate how you might proceed to analyse the data.

Some useful python libraries for analysis are imported and the data loaded into the notebook


```python
# The commands in this cell should be run if you do not have the required python packages in your environment
#import sys
#!{sys.executable} -m pip install numpy
#!{sys.executable} -m pip install matplotlib
#!{sys.executable} -m pip install pandas
```


```python
import pandas as pd
import numpy as np
```


```python
df = pd.read_csv('sample_data_190819.csv')
```

Before proceeding to analyse the data we view it to check it is loaded correctly and get a feel od the data we have.


```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>event_time_stamp</th>
      <th>beacon_id</th>
      <th>content_type</th>
      <th>content</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019-08-19 10:12:30.854 Europe/Stockholm</td>
      <td>NaN</td>
      <td>application/vnd.tobii.atex.session-started-eve...</td>
      <td>{"geo":{"country":{"name":"Sweden","iso_3166_1...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-08-19 10:12:43.505 Europe/Stockholm</td>
      <td>NaN</td>
      <td>application/vnd.tobii.atex.session-started-eve...</td>
      <td>{"geo":{"country":{"name":"Sweden","iso_3166_1...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-08-19 10:12:50.689 Europe/Stockholm</td>
      <td>NaN</td>
      <td>application/vnd.tobii.atex.session-started-eve...</td>
      <td>{"geo":{"country":{"name":"Sweden","iso_3166_1...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019-08-19 10:13:01.101 Europe/Stockholm</td>
      <td>NaN</td>
      <td>application/vnd.tobii.atex.session-started-eve...</td>
      <td>{"geo":{"country":{"name":"Sweden","iso_3166_1...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-08-19 10:13:08.773 Europe/Stockholm</td>
      <td>NaN</td>
      <td>application/vnd.tobii.atex.session-started-eve...</td>
      <td>{"geo":{"country":{"name":"Sweden","iso_3166_1...</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>event_time_stamp</th>
      <th>beacon_id</th>
      <th>content_type</th>
      <th>content</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2910</td>
      <td>2324</td>
      <td>2910</td>
      <td>2910</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>1495</td>
      <td>1183</td>
      <td>5</td>
      <td>2910</td>
    </tr>
    <tr>
      <th>top</th>
      <td>2019-08-13 21:33:21.336 Europe/Stockholm</td>
      <td>0be56815-1065-4d17-8c5d-aef1e0bd3b10</td>
      <td>application/vnd.tobii.atex.beacon-detected-eve...</td>
      <td>{"request":{"headers":{"x-forward-cloudfront-f...</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>17</td>
      <td>9</td>
      <td>1182</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



This simple example analysis will look at the time distribution of events we have data for. 

The `event_time_stamp` column is currently formatted as an object and should be changed to a datetime? in order to
allows us to manipulate and analyse it.


```python
from datetime import datetime

### convert the string timestamps into datetime objects for analysis
def convert_to_datetime_hour(string, date_format='%Y-%m-%d %H:%M:%S.%f'):
    tz_naive_string = string[:23]
    datetime_string = datetime.strptime(tz_naive_string, date_format)
    return datetime_string.replace(minute=0, second=0, microsecond=0)
    
df['datetime_hour'] = df['event_time_stamp'].apply(lambda x: convert_to_datetime_hour(x))
```


```python
keep_columns = ['datetime_hour', 'content_type']
# group the data set by 'datetime' object and count the number of events
df_date_time = df.loc[:, keep_columns].groupby('datetime_hour').count().reset_index()
df_date_time.columns = ['Hour', 'Number_Events']
print(df_date_time)
```

                      Hour  Number_Events
    0  2019-08-13 21:00:00            126
    1  2019-08-13 22:00:00              8
    2  2019-08-14 09:00:00            256
    3  2019-08-14 10:00:00             47
    4  2019-08-14 11:00:00              5
    5  2019-08-15 15:00:00              2
    6  2019-08-15 16:00:00             31
    7  2019-08-15 17:00:00            198
    8  2019-08-15 18:00:00             21
    9  2019-08-15 20:00:00            219
    10 2019-08-16 07:00:00             97
    11 2019-08-16 10:00:00             24
    12 2019-08-16 11:00:00             36
    13 2019-08-16 14:00:00            170
    14 2019-08-16 15:00:00            144
    15 2019-08-16 16:00:00            156
    16 2019-08-16 17:00:00             12
    17 2019-08-17 22:00:00             20
    18 2019-08-17 23:00:00             35
    19 2019-08-18 10:00:00             25
    20 2019-08-18 11:00:00             31
    21 2019-08-18 12:00:00            219
    22 2019-08-18 13:00:00            187
    23 2019-08-18 15:00:00            122
    24 2019-08-18 16:00:00            241
    25 2019-08-18 18:00:00             14
    26 2019-08-19 10:00:00             26
    27 2019-08-19 11:00:00             32
    28 2019-08-19 12:00:00             12
    29 2019-08-19 13:00:00            108
    30 2019-08-19 14:00:00              7
    31 2019-08-19 16:00:00             37
    32 2019-08-19 17:00:00             78
    33 2019-08-20 09:00:00              5
    34 2019-08-20 10:00:00              4
    35 2019-08-20 11:00:00              3
    36 2019-08-20 13:00:00             23
    37 2019-08-20 14:00:00             26
    38 2019-08-20 15:00:00            102
    39 2019-08-20 16:00:00              1



```python
import matplotlib
```


```python
import matplotlib.pyplot as plt
# Will allow us to embed images in the notebook
%matplotlib inline
import seaborn as sns

sns.set(rc={'figure.figsize':(11, 4)})
df_date_time.plot(x='Hour', y='Number_Events', linewidth=0.5);
plt.title("Number of events per hour")
plt.ylabel("Number of Events")
plt.xlabel("Hour")
plt.show()
```


![png](Interview%20Exercise%20-%20Event%20Data%20Analytics%20Example_files/Interview%20Exercise%20-%20Event%20Data%20Analytics%20Example_17_0.png)


### Example JSON analysis

This second example analysis shows how we might interogate the JSON `content` field in order to extract properties for
analysis.


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2910 entries, 0 to 2909
    Data columns (total 5 columns):
    event_time_stamp    2910 non-null object
    beacon_id           2324 non-null object
    content_type        2910 non-null object
    content             2910 non-null object
    datetime_hour       2910 non-null datetime64[ns]
    dtypes: datetime64[ns](1), object(4)
    memory usage: 113.8+ KB



```python
import json

df['content_json'] = df['content'].apply(json.loads) #Creating a column with the JSON content
df_content = pd.io.json.json_normalize(df['content_json']) #Helper function to create flat table from nested JSON

```


```python
# remove NA's from the browser hostname data
df_hostname = df_content.loc[:,['browser.page.url.hostname', 'time.stamp']].dropna()
# create an hourly column
date_format='%Y-%m-%dT%H:%M:%S.%f'
df_hostname['datetime_hour'] = df_hostname['time.stamp'].apply(lambda x: convert_to_datetime_hour(x, date_format))
# renaming columns
df_hostname.columns = ['Hostname', 'Datetime', 'Hour'] # grouping and counting events by hostname by hour

```


```python
sorted_df = df_hostname.groupby(['Hostname']).count().reset_index().sort_values(by=['Datetime'], ascending=False)
sorted_df = sorted_df[0:5]
```


```python
# plotting bar chart over time
df_hostname_popular = df_hostname[df_hostname.Hostname.isin(sorted_df['Hostname'].unique())]
df_hostname_popular.groupby(['Hour','Hostname']).size().unstack().plot(kind='bar', stacked='True')

```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f86b550>




![png](Interview%20Exercise%20-%20Event%20Data%20Analytics%20Example_files/Interview%20Exercise%20-%20Event%20Data%20Analytics%20Example_23_1.png)


The following code prints out the content fields for a single `beacon-in-view-event` to illustrate the fields it contains. It should be noted that `beacon.meta.html`contains a large amount of html data to process.


```python
#from pprint import pprint
#pprint(json.loads(df[df['content_type'].isin(["application/vnd.tobii.atex.beacon-in-view-event-v1+json"])].iloc[0]['content']))
```

    {'beacon': {'height': '94',
                'id': 'cff73bae-7abc-44e3-ab35-3cd9eb6e64b6',
                'in_view_ratio_percentile': '100',
                'meta': {'ad': {'video': {'subtype': 'Overlay', 'type': 'Instream'},
                                'youtubeformat': 'Overlay'},
                         'html': '<div class="ytp-ad-image-overlay" '
                                 'style="max-width: 728px;"><div '
                                 'class="ytp-ad-overlay-ad-info-button-container"><span '
                                 'class="ytp-ad-hover-text-button '
                                 'ytp-ad-info-hover-text-button" '
                                 'id="ad-info-hover-text-button:1q" '
                                 'style=""><button class="ytp-ad-button '
                                 'ytp-ad-button-link ytp-ad-clickable" '
                                 'id="button:1r" style=""><span '
                                 'class="ytp-ad-button-icon"><svg fill="#fff" '
                                 'height="100%" version="1.1" viewBox="0 0 48 48" '
                                 'width="100%"><path d="M0 0h48v48H0z" '
                                 'fill="none"></path><path d="M22 '
                                 '34h4V22h-4v12zm2-30C12.95 4 4 12.95 4 24s8.95 20 '
                                 '20 20 20-8.95 20-20S35.05 4 24 4zm0 36c-8.82 '
                                 '0-16-7.18-16-16S15.18 8 24 8s16 7.18 16 16-7.18 '
                                 '16-16 '
                                 '16zm-2-22h4v-4h-4v4z"></path></svg></span></button><div '
                                 'class="ytp-ad-hover-text-container '
                                 'ytp-ad-info-hover-text-short">Why this '
                                 'ad?</div></span></div><div '
                                 'class="ytp-ad-overlay-close-container"><button '
                                 'class="ytp-ad-overlay-close-button"><svg '
                                 'height="100%" viewBox="0 0 24 24" '
                                 'width="100%"><path d="M19 6.41L17.59 5 12 10.59 '
                                 '6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 '
                                 '17.59 19 19 17.59 13.41 12z" '
                                 'fill="#fff"></path></svg></button></div><div '
                                 'class="ytp-ad-overlay-image"><img '
                                 'src="https://pagead2.googlesyndication.com/pagead/imgad?id=CICAgKDb4u_G5AEQ2AUYWjIIFp0QLMSTlBk" '
                                 'width="728" height="90"></div></div>',
                         'htmlbaseurl': 'https://www.youtube.com/',
                         'text': '',
                         'view_id': '6183ccae-e6e2-4e45-bbec-7484cc9af637'},
                'parent_ids': [],
                'tags': [],
                'type': 'ad/youtube.com-watch-player-overlay-v1',
                'width': '728',
                'x': '143',
                'y': '417'},
     'browser': {'adblock_detected': False,
                 'client': {'height': '684',
                            'scroll': {'x': '0', 'y': '0'},
                            'width': '1440',
                            'x': '0',
                            'y': '79'},
                 'page': {'height': '2250',
                          'url': {'host': 'www.youtube.com',
                                  'hostname': 'www.youtube.com',
                                  'href': 'https://www.youtube.com',
                                  'origin': 'https://www.youtube.com',
                                  'protocol': 'https:'},
                          'width': '1440',
                          'x': '0',
                          'y': '79',
                          'zoom_factor_percentile': '100'},
                 'screen': {'available': {'height': '812',
                                          'left': '0',
                                          'top': '23',
                                          'width': '1440'},
                            'height': '900',
                            'orientation': {'angle': '0',
                                            'type': 'landscape-primary'},
                            'os_zoom_factor_percentile': '100',
                            'pixel_ratio_percentile': '200',
                            'width': '1440'},
                 'window': {'height': '812',
                            'viewport': {'x': '0', 'y': '79'},
                            'width': '1440',
                            'x': '0',
                            'y': '23'}},
     'client': {'id': 'chrome-extension://jlinlkoiaifccncicmegnchnjiemnokg',
                'installation_id': 'c74c5cfe-647b-42f0-8bd2-1f9222c68698',
                'name': 'Tobii Pro Attention Panel (local)',
                'version': '0.1.151.7928099'},
     'duration': 'PT1S',
     'experimental': {'http_block_detected': False,
                      'http_block_urls': '["http://freegeoip.net"]',
                      'permissions_block_detected': False},
     'eyetracker': {'configuration_valid': False,
                    'connected': False,
                    'detected': False},
     'geo': {'city': {'name': 'Stockholm'},
             'country': {'iso_3166_1_alpha_2': 'SE', 'name': 'Sweden'},
             'region': {'name': 'Stockholm'}},
     'id': 'c86365b1-ef70-4ab7-bf72-613ebc2ecee7',
     'intersection': {'ratio_percentile': '100', 'threshold_percentile': '50'},
     'ntp': {'in_sync': True, 'offset': '0'},
     'platform': {'description': 'Chrome 76.0.3809.100 on OS X 10.14.0 64-bit',
                  'layout': 'Blink',
                  'name': 'Chrome',
                  'os': {'architecture': '64',
                         'family': 'OS X',
                         'version': '10.14.0'},
                  'ua': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) '
                        'AppleWebKit/537.36 (KHTML, like Gecko) '
                        'Chrome/76.0.3809.100 Safari/537.36',
                  'version': '76.0.3809.100'},
     'request': {'headers': {'accept': '*/*',
                             'accept-encoding': 'gzip, deflate, br',
                             'accept-language': 'en-GB,en-US;q=0.9,en;q=0.8',
                             'cloudfront-forwarded-proto': 'https',
                             'cloudfront-is-desktop-viewer': 'true',
                             'cloudfront-is-mobile-viewer': 'false',
                             'cloudfront-is-smarttv-viewer': 'false',
                             'cloudfront-is-tablet-viewer': 'false',
                             'cloudfront-viewer-country': 'SE',
                             'content-length': '3457',
                             'content-type': 'application/vnd.tobii.atex.beacon-in-view-event-v1+json',
                             'host': 'i0r8i4mqd4.execute-api.eu-west-1.amazonaws.com',
                             'origin': 'chrome-extension://jlinlkoiaifccncicmegnchnjiemnokg',
                             'sec-fetch-mode': 'cors',
                             'sec-fetch-site': 'cross-site',
                             'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X '
                                           '10_14_0) AppleWebKit/537.36 (KHTML, '
                                           'like Gecko) Chrome/76.0.3809.100 '
                                           'Safari/537.36',
                             'via': '2.0 '
                                    '05c02ade53b3395a9e9f2e8f66c7e4d1.cloudfront.net '
                                    '(CloudFront), 1.1 '
                                    'bb45ea5b3a4c19db9fecccf1bc9e803d.cloudfront.net '
                                    '(CloudFront)',
                             'x-amz-cf-id': 'Gdfp1g6GAmcEOyHivF9QoVOtHiyAq6S21NxJ2cv8W099x_hCFvlIUg==',
                             'x-amzn-apigateway-api-id': 'bkfx4gphe3',
                             'x-amzn-trace-id': 'Self=1-5d556fa2-40569cac1ea0890090005acc;Root=1-5d556fa2-97fbe064b79b6e48c3744918',
                             'x-forward-cloudfront-forwarded-proto': 'https',
                             'x-forward-cloudfront-is-desktop-viewer': 'true',
                             'x-forward-cloudfront-is-mobile-viewer': 'false',
                             'x-forward-cloudfront-is-smarttv-viewer': 'false',
                             'x-forward-cloudfront-is-tablet-viewer': 'false',
                             'x-forward-cloudfront-viewer-country': 'SE',
                             'x-forwarded-for': '84.217.178.135, 205.251.218.152, '
                                                '108.128.162.40, 52.46.34.101',
                             'x-forwarded-port': '443',
                             'x-forwarded-proto': 'https'},
                 'method': 'POST'},
     'session': {'id': 'b1fa013f-8063-4420-bd8f-255d602c9b95'},
     'time': {'stamp': '2019-08-15T14:43:45.703Z',
              'utc_offset': '120',
              'zone': 'Europe/Stockholm'}}


### Analytical Question Suggestions



1. Within the content JSON object there are a large number of interesting fields to extract and visualise such as the
`beacon.height`, the `beacon.width` or the `browser.page.url.hostname`. Explore some of these variables graphically to
understand the ads.

1. There is a specific field within content `beacon.meta.html`, which contains the html code found within the beacon.
You could explore using regular expressions or text mining to try and identify url's that are contained here.

1. Given the different events it is possible to try and calculate some metrics which might be interesting. Each
beacon_id should have a `beacon-detected-event-v1+json`, but only a fraction of these beacon_ids have a
`atex.beacon-in-view-event-v1+json`. It is possible to calculate this fraction and possibly intersting to see how it 
varies for different websites.


### Modelling Question Suggestions

Although each event is quite large, there are not a huge number of distinct events to undertake prediction. However it
is interesting to model which variables may be correlated and could act as predictors of others. Suggested models
include.

1. `Regression model`:- Counting the number of distinct beacons per session and predicting this number from a combination
    of the website and time of day.
    
1. `Classification Model`:- Using the beacon_width or beacon_height, and the time of day to predict which website a beacon
    appears on.
    
1. `Time Series Model`:- Calculating the number of beacons or number of sessions per hour and trying to see any periodicity
    in this or predict this value for the next period.
