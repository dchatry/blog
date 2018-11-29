---
layout: post
title: Periodically check a web page and get notified when it changes
date: 19:34 29-11-2018
headline: 
taxonomy:
    category: blog
    tag: [drupal]
---

Here is a quick script if you want to monitor a web page for changes. We are going to use [Selenium](https://www.seleniumhq.org/) (with Python 3) which is a framework allowing you to emulate a browser and make 

```
from selenium.webdriver import Firefox
from selenium.webdriver.firefox.options import Options
import time
import datetime
import requests
import logging

# We need to define the browser
# to run in headless mode (no window)
opts = Options()
opts.set_headless()
assert opts.headless  

# Log everything to make sure it 
# is actually working
logger = logging.getLogger('monitoring')
logger.setLevel(logging.DEBUG)
today = datetime.datetime.now()
fh = logging.FileHandler('/Users/YOU/monitoring.log')
fh.setLevel(logging.DEBUG)
logger.addHandler(fh)

# Create your Firefox instance
browser = Firefox(options=opts, executable_path="/usr/local/bin/geckodriver")

# Specify the page you want to monitor
browser.get('http://rdv.polyclinique-limoges.fr/')

button = browser.find_element_by_xpath("//div[@data-v='cvl']")

  
  
try:
  button.click();

  time.sleep(2)


  # Check message
  page = browser.find_element_by_xpath("//div[@class='contenu_page_deux']")
  if "Désolé, il n'y a actuellement plus aucun" in page.text:
    logger.info('Aucun RDV : ' + today.strftime("%Y-%m-%d %H:%M"))
  else:
    logger.info('RDV trouvé ! : ' + today.strftime("%Y-%m-%d %H:%M"))
    requests.post("https://maker.ifttt.com/trigger/ophtal/with/key/qBJ6y2INbvgZdmlUVxPsd")

except Exception as e:
  logger.info('Error on ' + today.strftime("%Y-%m-%d %H:%M") + ': ' + e.text)

# Close the session
browser.close()
```
