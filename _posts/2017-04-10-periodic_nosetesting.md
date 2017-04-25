---
layout: post
title:  “Periodic web service testing with nose and cron”
date:   2017-04-10
categories: nosetest, cron, api testing
---

Having a diverse set of projects competing for my attention, it can sometimes be weeks between when personally make use of the website I developed for the MINE. On the other hand, this app is the primary way most scientists access and use my work and so it’s important to keep make sure that I’m the first know if the service becomes nonfunctional. There are several powerful commercial tools available to test and monitor API such as [Runscope](https://www.runscope.com/) and [AlertSite](https://smartbear.com/product/alertsite/api-monitoring/) but these seemed a little overkill for my project and would have significant starting costs.

On the other hand, I had already written unit tests for the API during the development process so all I really needed was a way to execute these tests automatically and notify me if anything failed. Cron is the perfect package for the job because it allows for the execution of commands at specified intervals and is ubiquitous on unix machines. The last bit of the puzzle is notifying me of the test results if any tests fail which can be done with the `sendmail` command. The resulting bash script looks like this:
```
#!/bin/sh
cd /var/MINE-Server/
results=$(nosetests -v ./scripts/MINE_Client_tests.py 2>&1)
if echo $results | grep -q "FAIL"; then
        echo -e "Subject: Client Tests failed at" $(date) "\n\n" $results | sendmail myemail@gmail.com
fi
```
All that remains is to use `crontab -e` to specify the execution schedule. In my case I felt hourly testing offered a pretty reasonable tradeoff email/response time for my service so I added the following line to my crontab. `49 * * * * bash /var/MINE-server/server_tests.sh` Should my service ever begin to fail the unit tests I designed, I’ll get an email on the 49th minute of each hour until I fix the service (or comment out the cronjob). Not perfect but I’ll take that over rewriting a couple hundred lines of unit tests any day.
