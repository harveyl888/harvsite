+++
author = "Harvey"
title = "Stopping a crashed RStudio Server Instance"
date = "2019-01-08"
description = "Finding and Stopping a Crashed Linux RStudio Server Instance"
tags = ["R"]
+++

RStudio Server is a fantastic IDE.  Amongst its numerous features is the STOP button which allows execution to be halted.  This works well in most instances but there are times when its use can cause a session to hang, in particular when the interrupted code is running a C function (this is a behavior of R rather than RStudio Server).  The process can still be stopped through the linux command line, however after running the following command to identify rsessions running under a specific user id (my user id is nm61135n):

`ps aux |grep "rsession.*nm61135n"`

{{< figure src="/images/post-images/2019-01-08-crashed_session/ps-grab.png" >}}

The output returns the process id along with the rsession identifier allowing the culprit to be shut down using the `kill` command.