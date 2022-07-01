+++
author = "Harvey"
title = "Running SAS in Quarto"
date = "2022-06-27"
description = "Running SAS in Quarto"
tags = ["Quarto", "python"]
codeMaxLines = 100
draft = true
+++

[Quarto](https://quarto.org/) is a powerful web publishing system based on Pandoc.  Working with jupyter kernels expands the number of languages that Quarto can work with.  In this example we'll add SAS capability to Quarto.


## Setup

First install the `saspy` and the SAS kernel packages.

```bash
pip install saspy
pip install sas_kernel
```

Once installed the sas kernel should be listed when running `jupyter kernelspec list`:

```
sas      ...
python3  ...
```

## Configure saspy

Before the kernel can be used, two configuration files need to be created:
-  sascfg_personal.py contains details on how to connect to a SAS instance
-  _authinfo contains authentication details

Details on the configuration files are available at https://sassoftware.github.io/saspy/configuration.html.  
I'm using SAS on demand and additional information on editing these two files is available at https://support.sas.com/ondemand/saspy.html.

### sascfg_personal.py

Below is an example of a sascfg_personal.py (for a PC configuration).  Here we are specifying the connection to the SAS On Demand for Academics (oda) service, US Home Region 2.  The sasacfg_personal.py can be placed in the .config/saspy/ subfolder under the home folder.

```python
SAS_config_names=['oda']
oda = {'java' : 'C:\\Program Files (x86)\\Common Files\\Oracle\\Java\\javapath\\java.exe',
#US Home Region 1
#'iomhost' : ['odaws01-usw2.oda.sas.com','odaws02-usw2.oda.sas.com','odaws03-usw2.oda.sas.com','odaws04-usw2.oda.sas.com'],
#US Home Region 2
'iomhost' : ['odaws01-usw2-2.oda.sas.com','odaws02-usw2-2.oda.sas.com'],
#European Home Region 1
#'iomhost' : ['odaws01-euw1.oda.sas.com','odaws02-euw1.oda.sas.com'],
#Asia Pacific Home Region 1
#'iomhost' : ['odaws01-apse1.oda.sas.com','odaws02-apse1.oda.sas.com'],
#Asia Pacific Home Region 2
#'iomhost' : ['odaws01-apse1-2.oda.sas.com','odaws02-apse1-2.oda.sas.com'],
'iomport' : 8591,
'authkey' : 'oda',
'encoding' : 'utf-8'
}
```

### _authinfo

The _authinfo file is located in the home folder and, for SAS On Demand for Academics, looks like this:

```
oda user <username> password <password>
```

where _<username>_ and _<password>_ are the credentials to connect to oda.

### testing SAS in Quarto

Now that the SAS kernel has been installed and configured, we should be able to execute SAS code in a Quarto document by specifying the kernel in the yaml:

~~~markdown
---
title: "Quarto Demo with SAS"
format: html
jupyter: sas
---

### Jupyter SAS kernel

```{sas}
proc candisc data=sashelp.iris out=outcan distance anova;
   class Species;
   var SepalLength SepalWidth PetalLength PetalWidth;
run;
```
~~~

{{< figure src="/images/post-images/2022-06-27-quarto_sas/quarto_sas.png" >}}

