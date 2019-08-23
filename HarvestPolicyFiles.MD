# Harvest Policy Files - Overview

Here's a quick review of what's in the "policy files" used by *harvest* for the prototyp runs, and how to
make a new one should you be so inclined.

All directory references are relative to */n/proteus2nb/pds*.

## Purpose

The policy file acts as an input script for the **harvest** tool. It tells **harvest** how to select files
for processing, provides a mapping from physical space to URL, specifies which attributes are to be 
selected from each type of PDS4 label, and maps those attributes to fields in the registry (Solr) database.

> **A Word About Data**
>
> It is not necessary to copy the data from the archive to someplace else in order to process it. The
> __harvest__ and __registry__ tools do not write anything into the source directories.  However, the
> processing is fairly intense, so it should not be run on the public copies of our data just to avoid
> contention with users trying to access the data.  If you want to pull down a local copy or work 
> directly on a copy of the server data, use the backup copies located on sbnumd.astro.umd.edu.

## Creating a Harvest Policy File
The first step is to create a policy file for the data, to be used by **harvest** to define what gets
processed and what is pulled out.  The policy files I created are in the *HarvestPolicyFiles/* directory.

You can start with the *AAexample_Harvest.xml* file, if you like a lot of irrelevant garbage in your
processing files, or you can start with one the other data set files in that directory.  Working from 
the top of the file down:

1. In the ```<collections>``` area, remove the ```<file>``` elements you don't want and add one for
each file you do want to process. Typically, we'll be processing one collection at a time, but the
examples here process all the bundles in a collection. The bundle label itself is not listed, but is
picked up because of the ```<directories>``` section, following.

    > The files in the ```<collections>``` list don't actually have to be collection labels.  Note the
    > format of the *one-off.xml* policy file that processes a single label by listing the label under
    > ```<collections>``` and removing the ```<directories>``` section entirely.

2. In the ```<directories>``` area:
   * Include one ```<path>``` for each root directory to be scanned for
     label files.  There will likely be only one of these - the root of the bundle or collection you want to
     register. **Harvest** will descend through the directory hierarchy and look in every nook and cranny for 
     label files, so make sure that is what you want for each path listed here.
   * Indicate what the PDS4 label file names look like in the ```<fileFilter>``` area.  For the moment, this should
     always contain "*.xml", but this will become more interesting in future.  Only files matching this criterion 
     will be processed. There is also an ```<exclude>``` option, not used in any of the testing.  If you use this.
     successfull or otherwise, let the world know.
   * If there are directories in your path that you do not want to be processed, you can use the ```directoryFilters>```
     area to exclude them.  You can exclude multiple patterns.  I have not tested this, although the exclusion you
     would expect to work for our archive directories is in there.
     
3. Set the URL-disk path relationship in the ```<accessURLs>``` section.  This information is used to create the 
   URL links to the label and data files that are stored in the registry.  I think the point is to have one
   ```<accessURL>``` defined for each ```<path>``` in the preceeding section - anything else would require way
   more cleverness in the code than I would tend to ascribe to the original authors. **Harvest** appears to do 
   a direct substitution of ```<offset>``` value for ```<baseURL>```, so the URLs will work even if **harvest**
   was run on a local copy unless you mess with the internal structure of the bundles/collections you copy.
   
   > __N.B.:__ The location of the file on disk is also written to the registry database, so while copying data is
   > not an issue for testing, we will need to re-run harvest on the actual archive data when we go to production.
   
4. In the ```<candidates>``` area, you can generally leave things alone until you have a good idea what you're doing.
   **Harvest** will ignore irrelevant product types if you leave them in, but it will not gather data from product
   types you do not list in this area - so make sure you've got all the product types covered one way or another.
   The standard SBN list includes:
   * Product_Bundle
   * Product_Collection
   * Product_Observational
   * Product_Document
   There might be others in large datasets that include browse products, SPICE kernels, and other strange beasts.
   
## XPaths and Slot Names

In the ```<candidates>``` area, you'll see a long list of lines that look like this:

    <xPath slotName="observation_start_date_time">
      //Observation_Area/Time_Coordinates/start_date_time
    </xPath>
    
This line is mapping a field (a.k.a. a "slot") in the registry database to an attribute in the PDS4 label.  The
attribute is identified by the xpath expression between the tags.  Most slots have a simple xpath expression, like
the one above.  A few are more complicated and involve selection logic, like this:

    <xPath slotName="instrument_host_name">
      //Observing_System/Observing_System_Component[type='Spacecraft']/name
    </xPath>

__*Do not change ```slotName``` values or ```xPath``` expressions*__.  These are intended to be standardized across PDS so that users and APIs get
predictable results.  

_Do not add new slot names_ without talking to me first, because they require additional configuration 
elsewhere and we need to standardize names for discipline dictionaries across PDS. Eveything else at least needs to be
standardized across the SBN holdings.
