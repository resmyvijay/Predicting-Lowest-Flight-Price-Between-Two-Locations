import kivy
from kivy.app import App
from kivy.uix.label import Label
from kivy.uix.gridlayout import GridLayout
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button


#Pandas for structuring our data
import pandas as pd
import time
import datetime
import json
import requests
from lxml import html
from lxml.html import fromstring
from collections import OrderedDict
import argparse
import certifi
import urllib3
http = urllib3.PoolManager(cert_reqs='CERT_REQUIRED',ca_certs=certifi.where())
urllib3.disable_warnings()

from kivy.app import App
from kivy.uix.gridlayout import GridLayout
from kivy.properties import ObjectProperty

########   Flight Price Scraper  ########
#
# A Selenium based flight price scraping tool
#
# 2019/05/17
# v0.2, updated URL generator and parsing scheme to match the new Google
#     Flights website
#
# Denedencies:
#   Firefox: Web browser, https://www.mozilla.org/en-US/firefox/new/
#   Geckodriver: Firefox webdriver, https://github.com/mozilla/geckodriver/releases
#       - For Linux, extract to /usr/bin
#       - For OSX, extract to /usr/local/bin/
#   Selenium: Browser automation tool, http://www.seleniumhq.org/
#   Beautiful Soup 4: Library for scraping HTML and XML, https://www.crummy.com/software/BeautifulSoup/
#   lxml: Library for parsing HTML and XML, http://lxml.de/
#
#########################################

# import selenium to load page in Firefox
from selenium import webdriver
from selenium.webdriver.firefox.firefox_binary import FirefoxBinary
# import time to add a delay for the page to fully load
import time
# import BeautifulSoup to parse what we download
from bs4 import BeautifulSoup
# import re to process source code with regular expressions
import re
# import smtplib to allow us to email
import smtplib
import csv
import datetime

# flight price currency (USD,EUR,GBP,etc.)
currency = "USD"

# min and max departure dates as datetime objects
depart_date = datetime.datetime.strptime("2020-07-11","%Y-%m-%d")
return_date = datetime.datetime.strptime("2020-07-14","%Y-%m-%d")
print(depart_date)
print(return_date)
print(depart_date.strftime("%Y-%m-%d"))
# airline codes from Google Flights search URL, none=ALL
airlines_searched = ""

def main(airports_from, airports_to, depart_d, return_d):
    # tell selenium to use Firefox
    browser = webdriver.Firefox()

    msg = scrape_google_flights(browser, airports_from, airports_to, depart_d, return_d)
    return(msg)

def scrape_google_flights(browser, airports_from, airports_to, depart_d, return_d):
    # populate URL with departure and return dates
    url = "https://www.google.com/flights/#flt=" + airports_from + \
          "." + airports_to + "." + depart_d + \
          "*" + airports_to + "." + airports_from + "." \
          + return_d + ";c:" + currency + ";e:1" \
          + ";a:" + airlines_searched + "*" + airlines_searched + ";sd:1;t:f"

    for i in range(1):
        try:
            # open browser and navigate to URL
            browser.get(url)

            # delete all cookies before each search
            browser.delete_all_cookies()

            # refresh the page
            browser.refresh()

            # wait 10 seconds for page to load before grabbing source code
            time.sleep(10)
            html = browser.page_source
            soup = BeautifulSoup(html, "lxml")
            soup.prettify()

            # find all elements in price class inside HTML soup
            reduced_soup = soup.findAll("div", {"class": "gws-flights-results__price"})

            # record Gogole's "Best" flight price and strip special characters
            price = int(re.sub("[^0-9]", "", reduced_soup[0].getText()))

            # record airports
            reduced_soup = soup.findAll("div", {"class": "gws-flights-results__airports"})
            airport_from = reduced_soup[0].findAll("span")[0].getText()
            airport_to = reduced_soup[0].findAll("span")[1].getText()

            # exit while loop if price scrape is successful
            break
        # throw error and try again if the page can't be parsed
        # (e.g. network error, server error)
        except Exception:
            pass

        print("ERROR: Can't get price data. Refreshing page...")

        # wait to avoid constant refreshing if there's an error
        time.sleep(5)

    # open FlightPrices.csv to append data
    #with open("GFlights_.csv") as csvfile:
        # CSV data is tab delimited, quotechar "|" around data with tabs
        #writer = csv.writer(csvfile, delimiter="\t", quotechar="|",
                            #quoting=csv.QUOTE_MINIMAL)
        #writer.writerow((price))

    print("------------------------")
    print(depart_d + " to " +
          return_d)
    print("{0:.2f}".format(price) + " " + currency + ", " +
          airport_from + "-" + airport_to)

    a = "Minimum Flight Price is $" + str(price)

    return(a)

class DemoGridLayout(GridLayout):
    entry = ObjectProperty(None)

    def update(self, source, destination,depart_date,return_date):
        if source:
            output = main(source, destination, depart_date, return_date)
            print("Output",output)
            a = "          " + output
            self.ids.resultsLabel.text = a

    def clear(self):
        self.source.text = ""
        self.destination.text=""
        self.ids.resultsLabel.text="Results will display here"
        self.return_date.text = ""
        self.depart_date.text = ""

class DemoApp(App):
    def build(self):
        return DemoGridLayout()

if __name__ == "__main__":
    demoApp = DemoApp()
    demoApp.run()


