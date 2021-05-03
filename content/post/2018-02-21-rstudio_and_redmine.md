+++
author = "Harvey"
title = "Using Redmine with RStudio"
date = "2018-02-21"
description = "Integrating Redmine in RStudio"
tags = ["R"]
+++

Redmine is a great tool for tracking project progress and bugs.  With a little configuration it can integrate git repositories and integrate with R program / package development.  The following assumes that Redmine, R and RStudio server are all configured and working.

### Linking a repository to Redmine
Without changing file permissions, Redmine cannot interact directly with git repositories created in RStudio server.  A simple workaround is to clone the repository.  In this example we'll create a clone for a repository under ~/codes/redmine_demo

```
# ~/git_redmine_clones is a directory to hold the clones
cd ~/git_redmine_clones

# clone the repository
git clone --mirror ~/codes/redmine_demo

# set folder content permissions
chmod a+r -R redmine_demo.git
```

This repository can now be added to a Redmine project through Settings->Repository.
The path to the repository needs to be filled out in full.

### Updating clone via cron job
One caveat to this approach is that as changes are made to the repository they are not reflected in Redmine.  This can be overcome by adding a cron job that automatically fetches any new commits and updates the clone.  Changes are visible in Redmine when the clone is updated.

```
# use nano as the cron editor
export EDITOR=nano

# edit crontab file
crontab -e
```

The following line should be aded to the crontab file:  
`*/5 * * * * cd ~/git_redmine_clones/redmine_demo.git && git fetch -q --all –p`  
This states that the active directory should be changed to ~/git_redmine_clones/redmine_demo.git and then a git fetch command is run.  The timing in the crontab file dictates that this will be run every 5 minutes.

