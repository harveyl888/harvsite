+++
author = "Harvey"
title = "cron, python and slack - a beautiful combination"
date = "2018-08-05"
description = "Tracking Fantasy Football injuries using cron, python and slack"
tags = ["python"]
+++

This year I'm trying out slack as a platform for out fantasy football league.  It provides channels for banter, keeping track of transfers and, with the use of rss feeds, up to date information.  In addition I've set up a channel dedicated to injuries tied to an incoming webhook.  The injury channel works by running a small python code on a server via a weekly cron job.  The python code scrapes a site that lists current Premiership injuries and suspensions, reformats the output and then pushes it out to the slack channel.  This results in a simple solution that updates a channel weekly.

```python
# -*- coding: utf-8 -*-
import pandas as pd
from bs4 import BeautifulSoup
import requests
import json

url = "https://www.fantasyfootballscout.co.uk/fantasy-football-injuries/"
webhook = ## provided webhook url


response = requests.get(url)

soup = BeautifulSoup(response.text, "html5lib")

table = soup.find('table', attrs={'class':'ffs-ib-full-content'})
table_body = table.find('tbody')

data = []

rows = table_body.find_all('tr')
for row in rows:
    cols = row.find_all('td')
    cols = [ele.text.strip() for ele in cols]
    data.append([ele for ele in cols if ele]) # Get rid of empty values

status = {"Injured": ":inj-injured:", 
          "Available": ":inj-available:",
          "Unavailable": ":inj-unavailable:",
          "Knock": ":inj-knock:",
          "On Loan": ":inj-on-loan:",
          "Suspended": ":inj-disciplinary:",
          "Doubt 25%": ":inj-doubt-25:", 
          "Doubt 50%": ":inj-doubt-50:", 
          "Doubt 75%": ":inj-doubt-75:"}

p1 = pd.DataFrame(data, columns = ['Name', 'Club', 'Status', 'Return Date', 'Latest News', 'Last Updated'])
p1['status_icon'] = p1['Status'].replace(status)
p1['out'] = p1['status_icon'] + ' ' + p1['Name'] + ' ' + p1['Club']

# build the payload
payload = {}
payload['username'] = 'injury-bot'
payload['channel'] = '#injuries'

attachments = []
attachments.append({"fallback":"Link to injury website: <https://www.fantasyfootballscout.co.uk/fantasy-football-injuries/|full injury table>"})
attachments.append({"pretext":"Link to injury website: <https://www.fantasyfootballscout.co.uk/fantasy-football-injuries/|full injury table>"})

field_inj = {}
field_inj['title'] = 'Injury Table'
field_inj['value'] = '\n'.join(p1['out'].tolist())
field_inj['short'] = False

attachments.append({"fields": [field_inj]})
payload['attachments'] = attachments

requests.post(webhook, data = json.dumps(payload))
```