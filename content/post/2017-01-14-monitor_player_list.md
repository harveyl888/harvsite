+++
author = "Harvey"
title = "Monitoring Fantasy Football Transfers"
date = "2017-01-14"
description = "Let me know when there's an update to the Fantasy Premierleague Player List"
tags = ["R"]
+++

There are many fantasy sports out there.  Since I'm from England I like playing fantasy football (that's football, also known as soccer where I am now).  Rules are pretty straight forward - pick a team of players from the Premier League and score points depending on how they perform.  Most British leagues work on the principle that any team can contain any player and that two teams within a single league can each have the same player.  This is contrary to many American fantasy NFL leagues in which the number of managers is limited and player repeats are not allowed.  Several years ago, we changed our Premiership League so that each player could only exist in a single team.  

This means that there is now an advantage to being first to transfer in an on-form player or a potential star once they hit the Premiership.  FIFA regulates that leagues should have two transfer windows - one prior to the start of a season and one in the middle.  For the Premiership, the in-season transfer window is the month of January.  Now, since our league works on the princple that only one team can have any player and that the player has to be posted on the Premierleague site before he can be transferred into a team, there's a distinct advantage to knowing as soon as a player is available.  This can be easily accomplished using the script below.  

The code works by monitoring the Premierleague site, in particular the web page that lists all the players.  It grabs the player list, compares it to the previously grabbed list and, if there have been any changes, sends out a pushbullet notification.  Pushbullet's a great service allowing communication to multiple devices simultaneously.  I run this on a Digital Ocean droplet through a cron job.  The code is accompanied by a .rpushbullet.json file which contains the pushbullet key and device parameters.  

```r
##
## transferWatch
##
## Monitor fantasy premierleague player list for updates during transfer window.
## Send a pushbullet notification when the player list has changed.
## Run as a cron job to send updates at regular intervals
##
## crontab:
## 0 * * * * Rscript $HOME/codes/transferWatch/transferWatch.R
## run every hour
##

library(rvest)
library(xml2)
library(dplyr)
library(RPushbullet)
library(methods)

dataFolder <- Sys.getenv('HOME')
l.devices <- RPushbullet::pbGetDevices()
devices <- unlist(lapply(l.devices[['devices']], function(x) x[['nickname']]))

# Scrape site and pull back tables of players
url <- 'https://fantasy.premierleague.com/player-list/'
pos <- c('GLK', 'DEF', 'MID', 'FWD')
tabs <- url %>% xml2::read_html(url) %>% html_nodes('.ism-table') %>% html_table()

for (i in seq_along(tabs)) {
  tabs[[i]]$Pos <- pos[(i+1) %/% 2]
}
new.tables <- dplyr::bind_rows(tabs)

filename <- paste0(dataFolder, '/players.csv')
if (file.exists(filename)) {
  # Read in old data
  old.tables <- read.csv(filename, stringsAsFactors = F, encoding = 'UTF-8')
  # Compare tables
  df.new <- dplyr::setdiff(new.tables[, c(1,2,5)], old.tables[, c(1,2,5)])
  if (nrow(df.new) > 0) {
    newPlayers <- paste0(apply(df.new, 1, function(x) paste0(x, collapse = ', ')), collapse='\n')
    RPushbullet::pbPost('note', 'Transfer News', body = newPlayers, recipients = devices)
  }
} 

# Write new data to file
write.csv(new.tables, filename, row.names=F)
```
