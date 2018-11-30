---
layout: post
title: Periodically check a web page and get notified when it changes
date: 19:34 29-11-2018
headline: 
taxonomy:
    category: blog
    tag: [drupal]
---

Here is a quick script that can be very useful if you want to monitor a web page for changes. We are going to use [Selenium](https://www.seleniumhq.org/) (with Python 3) which is a framework allowing you to emulate a browser and control it entirely (meaning you can also interact, click, scroll, etc. with anything). It is especially useful for automating tests, but it's not its purpose today!

In my case, I wanted to check if my local clinic (which is always busy) had any appointment available. Luckily they provide a website where you can specify the reason for the appointment (so one click) and it tells you if there are any slots available.

We are going to use Firefox, but Selenium is compatible with any browser. You will need to download [geckodriver](https://github.com/mozilla/geckodriver/releases) and put the executable in `/usr/local/bin/`. For Windows, put it anywhere as long as it's readable and do not forget to change the `executable_path` configuration variable in the script below.

Here is the full code, explained bit by bit:

```
# Imports, we are going to use Firefox but you
# can use any browser
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
browser.get('https://www.websiteclinic.com/')

# Get the button I need to click for the appointment
button = browser.find_element_by_xpath("//div[@data-v='cvl']")
  
try:
  button.click();
  
  # Wait a bit because there is
  # some javascript executing on the page
  time.sleep(2)

  # Get the message
  page = browser.find_element_by_xpath("//div[@class='contenu_page_deux']")
  
  # If there is no appointment, log the attempt
  if "Désolé, il n'y a actuellement plus aucun rendez-vous" in page.text:
    logger.info('Aucun RDV : ' + today.strftime("%Y-%m-%d %H:%M"))
  else:
    # Else, send a notification via IFTTT webhooks
    logger.info('RDV trouvé ! : ' + today.strftime("%Y-%m-%d %H:%M"))
    requests.post("https://maker.ifttt.com/trigger/[action]/with/key/[your_key]")

except Exception as e:
  logger.info('Error on ' + today.strftime("%Y-%m-%d %H:%M") + ': ' + e.text)

# Close the session
browser.close()
```

And voilà! You get a notification on your phone as soon as a slot is free so you can be the first to book it! (thank you [IFTTT](http://ifttt.com/) for your wonderful webhooks!

Now you can add this to your crontab with by adding the line:
```
0-59 * * * 1-5 /usr/local/bin/python3 /path_to_script/script.py
```
This will check every minute every opening day but you can tweak it to your liking!
