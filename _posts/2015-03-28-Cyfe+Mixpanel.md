---
layout: post
title: Cyfe + MixPanel
date:   2015-03-28 03:00:08
categories: Python
comments: false
---
# Intro

I discovered [Cyfe](http://www.cyfe.com/) about one year ago and played a bit with the PushAPI widget which was very handy and easy to use.
I recently wanted to know if I could use Cyfe to display our Mixpanel metrics. And happily Cyfe has now a "widget" for Mixpanel. But sadly the widget is currently very basic and does not allow segmentation or complex requests. So I decided to implement my own "connector" with help of the existing "Private Url" widget.

# Why

Why did I spent time implementing my own Mixpanel connector for Cyfe ? Here is short list of reasons:

- One big limitation of the current Mixpanel widget is that you cannot display multiple curves in the same widget even if you specify a property to segment on.
- Actually you can specify a property to segment on, but it will be effective only if you select 'Top unique value'. But then you will have to pick one possible value and you will see a plot of the occurrences of this value over time (one single curve).
- There is no mean to "cleanup" raw data to avoid unwanted values like empty strings or 'undefined' values.
- No pie chart

# How

To build this connector I used the following technologies:

- python + flask to build a small rest api
- pythonanywhere to host the rest api
- Mixpanel REST API to get data
  1. You will find the flask rest api code on bitbucket at https://bitbucket.org/jams/pymixpanel
  2. Create an account on PythonAnywhere and create a project using the flask wizard.
  3. Upload the files in your flask application folder and reload.
  4. Your personal rest api should be up and ready for test.

The endpoint url should look like:
http://username.pythonanywhere.com:80/mixpanel/api/v1.0/event_name/apisecretkey/apikey?by=propertyname&clean=True&charttype=pie

Where:

- **username** is your Python Anywhere user name.
- **event_name** is the mixpanel event name, just replace space chararcters with underscores
- **apisecretkey** is your mixpanel secret key
- **apikey** is your mixpanel project api key
- (optional) **propertyname** is one of the event's property (space are replaced with underscores) to segment on
- (optional) **clean** is just a way to remove data with empty or undefined value for the segmented property
- **charttype** allows to select good formatting depending on which kind of plot you select (currently I just implemented lines and pie). If not specified lines is the default.

# Result

![cyfe-privateurl](/images/cyfe-privateurl.png)
> How to configure your "Private Url" widget

![cyfe-privateurl](/images/cyfe-mixpanel.png)
> 4 widgets using "my" mixpanel connector (5 widgets allowed with free cyfe plan)

#Conclusion

With a very little programming anyone can leverage the power of Cyfe and Mixpanel. Anyway it seems that Cyfe is quite active and probably already working on improvements for the Mixpanel widget. So my connector may be useless soon, but in the mean time I will use it.

I'm now working on worldmap charts to display Mixpanel data, but that will be the purpose of another article.

# Code


> flask_app.py

```python
#!flask/bin/python
from datetime import date, timedelta
from flask import Flask
from flask import request

import mixpanel
import cyfe

app = Flask(__name__)

def get_period(start, end):
    today = date.today()
    startdate = today - timedelta(start)
    enddate = today - timedelta(end)

    return (str(startdate), str(enddate))

@app.route('/mixpanel/api/v1.0/<event>/<apisecret>/<apikey>', methods=['GET'])
def get_event(event, apisecret, apikey):
    api = mixpanel.API(api_key = apikey, api_secret = apisecret)
    period = get_period(30, 1)
    event = event.replace('_', ' ')
    by = request.args.get('by', 'None')
    charttype = request.args.get('charttype', 'None')

    if by != None and by != '' and by != 'none':
        by = by.replace('_', ' ')
        data = cyfe.get_segmentation(api, event, by, period[0], period[1])
    else:
        data = cyfe.get_event(api, event, period[0], period[1])

    clean = request.args.get('clean')

    return cyfe.mixpaneldata_to_piecyfe(data, clean) if charttype=='pie' else cyfe.mixpaneldata_to_cyfe(data, clean)

# Uncomment this line to run in local
#app.run(debug=True)
```

> cyfe.py

```python
def get_event(api, eventName, from_date, to_date):
    return api.request(['events'], {
    'event' : [eventName,],
    'type' : 'general',
    'unit' : 'day',
    'interval' : 10,
    'from_date': from_date,
    'to_date': to_date,
    })

def get_segmentation(api, eventName, property, from_date, to_date):
    return api.request(['segmentation'], {
    'event' : eventName,
    'type' : 'general',
    'unit' : 'day',
    'on': 'properties["%s"]' % property,
    'from_date': from_date,
    'to_date': to_date,
    })

def mixpaneldate_to_cyfe(date):
    return date.replace('-', '')

def mixpaneldata_to_cyfe(data, clean=True):
    series = data['data']['series']
    values = data['data']['values']

    # remove unwanted values
    if clean:
        values.pop('', None)
        values.pop('undefined', None)

    fieldnames = ['Date'] + values.keys()
    csvText = ','.join(fieldnames) + '\n'
    for i in range(0, len(series)):
        date = series[i]
        cyfeDate = mixpaneldate_to_cyfe(date)
        csvText = csvText + cyfeDate + ','
        for serie in values.items():
            csvText = csvText + str(serie[1][date]) + ','
        csvText = csvText + '\n'

    return csvText

def mixpaneldata_to_piecyfe(data, clean=True):
    values = data['data']['values']

    # remove unwanted values
    if clean:
        values.pop('', None)
        values.pop('undefined', None)

    csvText = ','.join(values.keys()) + '\n'
    for item in values.items():
        csvText = csvText + str(sum(item[1].values())) + ','
    csvText = csvText + '\n'

    return csvText
```

> mixpanel.py

```python
#! /usr/bin/env python
#
# Mixpanel, Inc. -- http://mixpanel.com/
#
# Python API client library to consume mixpanel.com analytics data.
#
# Copyright 2010-2013 Mixpanel, Inc
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import hashlib
import urllib
import urllib2
import time
try:
    import json
except ImportError:
    import simplejson as json

class API(object):

    ENDPOINT = 'https://mixpanel.com/api'
    VERSION = '2.0'

    def __init__(self, api_key, api_secret):
        self.api_key = api_key
        self.api_secret = api_secret

    def request(self, methods, params, format='json'):
        """
            methods - List of methods to be joined, e.g. ['events', 'properties', 'values']
                      will give us http://mixpanel.com/api/2.0/events/properties/values/
            params - Extra parameters associated with method
        """
        params['api_key'] = self.api_key
        params['expire'] = int(time.time()) + 600   # Grant this request 10 minutes.
        params['format'] = format
        if 'sig' in params: del params['sig']
        params['sig'] = self.hash_args(params)

        request_url = '/'.join([self.ENDPOINT, str(self.VERSION)] + methods) + '?' + self.unicode_urlencode(params)
        request = urllib2.urlopen(request_url, timeout=120)
        data = request.read()

        return json.loads(data)

    def unicode_urlencode(self, params):
        """
            Convert lists to JSON encoded strings, and correctly handle any
            unicode URL parameters.
        """
        if isinstance(params, dict):
            params = params.items()
        for i, param in enumerate(params):
            if isinstance(param[1], list):
                params[i] = (param[0], json.dumps(param[1]),)

        return urllib.urlencode(
            [(k, isinstance(v, unicode) and v.encode('utf-8') or v) for k, v in params]
        )

    def hash_args(self, args, secret=None):
        """
            Hashes arguments by joining key=value pairs, appending a secret, and
            then taking the MD5 hex digest.
        """
        for a in args:
            if isinstance(args[a], list): args[a] = json.dumps(args[a])

        args_joined = ''
        for a in sorted(args.keys()):
            if isinstance(a, unicode):
                args_joined += a.encode('utf-8')
            else:
                args_joined += str(a)

            args_joined += '='

            if isinstance(args[a], unicode):
                args_joined += args[a].encode('utf-8')
            else:
                args_joined += str(args[a])

        hash = hashlib.md5(args_joined)

        if secret:
            hash.update(secret)
        elif self.api_secret:
            hash.update(self.api_secret)
        return hash.hexdigest()
```
