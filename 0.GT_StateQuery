# -*- coding: utf-8 -*-
# Felipe Comments:
#   The code provided in this github page is for replicability purposes only. The API key and other access parameters have 
#   been removed in order to protect the restricted data.
# Instructions
# Get the state / dma / sountries file in the folder and cd to it
# pip install any libraries missing.
# change the directory to the same where the states.csv list file is (google state tickers)
# Run
"""
Created on Thu Mar 12 12:18:45 2020
@author: thakk

Edited on Thu Mar 19 
@edits:  flozanor
"""
"""Sample code showing how to access the Google Flu Trends API."""

import csv
import datetime
import sys
import time
import pandas as pd

from apiclient.discovery import build

# ------ Insert your API key in the string below. -------
API_KEY = 

SERVER = 
API_VERSION = 
DISCOVERY_URL_SUFFIX = 
DISCOVERY_URL = 

MAX_QUERIES = 30

# ------    -------
def DateToISOString(datestring):
  """Convert date from (eg) 'Jul 04 2004' to '2004-07-11'.
  Args:
    datestring: A date in the format 'Jul 11 2004', 'Jul 2004', or '2004'
  Returns:
    The same date in the format '2004-11-04'
  Raises:
    ValueError: when date doesn't match one of the three expected formats.
  """

  try:
    new_date = datetime.datetime.strptime(datestring, '%b %d %Y')
  except ValueError:
    try:
      new_date = datetime.datetime.strptime(datestring, '%b %Y')
    except ValueError:
      try:
        new_date = datetime.datetime.strptime(datestring, '%Y')
      except:
        raise ValueError("Date doesn't match any of '%b %d %Y', '%b %Y', '%Y'.")

  return new_date.strftime('%Y-%m-%d')


def GetQueryVolumes(queries, start_date, end_date,
                    geo='US', geo_level='country', frequency='week'):
  """Extract query volumes from Flu Trends API.
  Args:
    queries: A list of all queries to use.
    start_date: Start date for timelines, in form YYYY-MM-DD.
    end_date: End date for timelines, in form YYYY-MM-DD.
    geo: The code for the geography of interest which can be either country
        (eg "US"), region (eg "US-NY") or DMA (eg "501").
    geo_level: The granularity for the geo limitation. Can be "country",
              "region", or "dma"
    frequency: The time resolution at which to pull queries. One of "day",
              "week", "month", "year".

  Returns:
    A list of lists (one row per date) that can be output by csv.writer.

  Raises:
    ValueError: when geo_level is not one of "country", "region" or "dma".
  """

  if not API_KEY:
    raise ValueError('API_KEY not set.')

  service = build('trends', API_VERSION,
                  developerKey=API_KEY,
                  discoveryServiceUrl=DISCOVERY_URL)

  dat = {}

  # Note that the API only allows querying 30 queries in one request. In
  # the event that we want to use more queries than that, we need to break
  # our request up into batches of 30.
  batch_intervals = range(0, len(queries), MAX_QUERIES)

  for batch_start in batch_intervals:
    batch_end = min(batch_start + MAX_QUERIES, len(queries))
    query_batch = queries[batch_start:batch_end]
    # Make API query
    if geo_level == 'country':
      # Country format is ISO-3166-2 (2-letters), e.g. 'US'
      req = service.getTimelinesForHealth(terms=query_batch,
                                          time_startDate=start_date,
                                          time_endDate=end_date,
                                          timelineResolution=frequency,
                                          geoRestriction_country=geo)
    elif geo_level == 'dma':
      # See https://support.google.com/richmedia/answer/2745487
      req = service.getTimelinesForHealth(terms=query_batch,
                                          time_startDate=start_date,
                                          time_endDate=end_date,
                                          timelineResolution=frequency,
                                          geoRestriction_dma=geo)
    elif geo_level == 'region':
      # Region format is ISO-3166-2 (4-letters), e.g. 'US-NY' (see more examples
      # here: en.wikipedia.org/wiki/ISO_3166-2:US)
      req = service.getTimelinesForHealth(terms=query_batch,
                                          time_startDate=start_date,
                                          time_endDate=end_date,
                                          timelineResolution=frequency,
                                          geoRestriction_region=geo)
    else:
      raise ValueError("geo_type must be one of 'country', 'region' or 'dma'")

    res = req.execute()

    # Sleep for 1 second so as to avoid hittting rate limiting.
    time.sleep(1)

    # Convert the data from the API into a dictionary of the form
    # {(query, date): count, ...}
    res_dict = {(line[u'term'], DateToISOString(point[u'date'])):
                point[u'value']
                for line in res[u'lines']
                for point in line[u'points']}

    # Update the global results dictionary with this batch's results.
    dat.update(res_dict)

  # Make the list of lists that will be the output of the function
  res = [['date'] + queries]
  for date in sorted(list(set([x[1] for x in dat]))):
    vals = [dat.get((term, date), 0) for term in queries]
    res.append([date] + vals)

  return res


def main():
    
    # Read the states file
    keywords1 = ["coronavirus", "corona virus", "covid", "covid19", "covid-19", "covid 19",
                "coronavirus symptoms", "corona virus symptoms", "covid symptoms", "covid19 symptoms", "covid-19 symptoms", "covid 19 symptoms"]
    keywords2 = ["coronavirus treatment", "testing", "coronavirus testing", "testing near me", "hospital", "hospital beds",
                "hospitals near me", "hand sanitizer", "face masks", "masks", "isolation", "quarantine", "social distancing",
                "school closures", "online work", "toiletpaper", "toilet paper", "coronavirus conspiracy", "coronavirus hoax", "coronavirus overblown"]
    keywords3 = ["coronavirus plot" , "ibuprofen", "Paper towels", "flu vaccine", "flu shot",  "chinese virus",
                "dayquil", "NyQuil", "Robitussin", "Tylenol", "test cost", "cvs near me", "remote working", "home remedies",
                "cough syrup", "Advil", "China hoax", "alcohol delivery", "grocery delivery"]
    keywords4 = ["coronavirus  home remedies","coronavirus testing near me","coronavirus test cost","coronavirus remote working","coronavirus online work",
                "coronavirus school closures","coronavirus isolation","coronavirus quarantine","coronavirus grocery delivery","coronavirus alcohol delivery",
                "coronavirus hand sanitizer","coronavirus face masks","coronavirus Paper towels","coronavirus toilet paper","coronavirus conspiracy","coronavirus hoax",
                "coronavirus overblown","coronavirus China hoax"]                                    
   
    state = pd.read_csv("states.csv")
    regions = state['region'].values

    col = keywords1+["state"]
    result_df = pd.DataFrame(columns = col)
  
    for reg in regions:         
        ma_region_daily = pd.DataFrame(GetQueryVolumes(keywords1,
                                        start_date='2020-01-01',
                                        end_date='2020-03-20',
                                        geo=reg,
                                        geo_level='region',
                                        frequency='day'),columns=["date"]+keywords1)
        ma_region_daily['state'] = reg
            
        ma_region_daily = ma_region_daily.iloc[1:]
            
        result_df = pd.concat([result_df,ma_region_daily],ignore_index = True,axis = 0)
            
    print(ma_region_daily)
    print(result_df)
    result_df.to_csv("result_states_0320_1.csv")


    col = keywords2+["state"]
    result_df = pd.DataFrame(columns = col)
  
    for reg in regions:         
        ma_region_daily = pd.DataFrame(GetQueryVolumes(keywords2,
                                        start_date='2020-01-01',
                                        end_date='2020-03-20',
                                        geo=reg,
                                        geo_level='region',
                                        frequency='day'),columns=["date"]+keywords2)
        ma_region_daily['state'] = reg
            
        ma_region_daily = ma_region_daily.iloc[1:]
            
        result_df = pd.concat([result_df,ma_region_daily],ignore_index = True,axis = 0)
            
    print(ma_region_daily)
    print(result_df)
    result_df.to_csv("result_states_0320_2.csv")

    col = keywords3+["state"]
    result_df = pd.DataFrame(columns = col)
  
    for reg in regions:         
        ma_region_daily = pd.DataFrame(GetQueryVolumes(keywords3,
                                        start_date='2020-01-01',
                                        end_date='2020-03-20',
                                        geo=reg,
                                        geo_level='region',
                                        frequency='day'),columns=["date"]+keywords3)
        ma_region_daily['state'] = reg
            
        ma_region_daily = ma_region_daily.iloc[1:]
            
        result_df = pd.concat([result_df,ma_region_daily],ignore_index = True,axis = 0)
            
    print(ma_region_daily)
    print(result_df)
    result_df.to_csv("result_states_0320_3.csv")

    col = keywords4+["state"]
    result_df = pd.DataFrame(columns = col)
  
    for reg in regions:         
        ma_region_daily = pd.DataFrame(GetQueryVolumes(keywords4,
                                        start_date='2020-01-01',
                                        end_date='2020-03-20',
                                        geo=reg,
                                        geo_level='region',
                                        frequency='day'),columns=["date"]+keywords4)
        ma_region_daily['state'] = reg
            
        ma_region_daily = ma_region_daily.iloc[1:]
            
        result_df = pd.concat([result_df,ma_region_daily],ignore_index = True,axis = 0)
            
    print(ma_region_daily)
    print(result_df)
    result_df.to_csv("result_states_0320_4.csv")
   

if __name__ == '__main__':
  main()
