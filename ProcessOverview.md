# Harvest/Register Process Overview

_The following assumes you're working with the docker install of the registry software that is on proteus, and you have
the harvest executable in your path._

The steps involved in ingesting data from PDS4 labels into the registry database, ready for searching, go like this:

## 1. Validate and Inspect the Data

Trying to harvest metadata from invalid labels will only lead to corrupt search data.  Archived data should all be 
validated, so unless you're running a test harvest, this should already be done.

Inspect the data to determine the full range of product type included, and it wouldn't be a bad idea to note how many of each
product type (or in each collection) you expect to register.  Helps in tracking down harvesting glitches.

The look at the typical data labels and note any fields from discipline dictionaries that would be useful to index for searching. 
If we aren't already harvesting these fields, we should add them to the configuration.  Let me (Anne) know, and we'll do that.

## 2. Create the Harvest Policy File

See [HarvestPolicyFiles.MD](HarvestPolicyFiles.MD) for more detailed notes. For now, at least, your best bet is to start
with a policy file for a similar data source, and modify it.

## 3. Run Harvest

The typical invocation of **harvest** in our test area looks like this:

    % harvest -c policy_file -l log_file -o solr_docs_directory
    
where:
* _policy_file_ is the file created in step 2 (include path information if it's not local). One file only, and this is 
required.
* _log_file_ directs the flood of INFO messages (and any errors) to a file for you to review in a sensible way should
something go wrong.  It's optional and may be omitted.  Check the last 30 lines of the file to see the processing summary.
* _solr_docs_directory is where **harvest** should create the *solr_docs/* subdirectory that holds its output files.  
This is also optional.  If you don't specify a destination directory, **harvest** will create a *solr_docs/* directory 
in your current working directory.  It will do this even if you have a *solr_docs/* directory already present in the
location you give it/default to.  It will not delete the current version, though, but instead adds a rather long time
tag to the existing *solr-docs/* directory before creating a new one.

**Harvest** takes a long time to run - about 1.08 second per label to be processed working on proteus with the data in
the *TestData/* directory.  You should do the math and
figure out how long the **harvest** run ought to take.  If the run completes early, something has gone wrong...

Once **harvest** has completed, check the bottom of the log file.  You should see a message telling you that all files
succeeded. There will also be a breakdown by collection, if you were processing any collections. 

If you see something non-zero next to "Failure", look in the log file for the strings "ERROR" and "WARNING".  Either 
can result in a failure.  "ERROR" lines typically mean none of the **harvest** processing succeeded, while "WARNING" 
lines usually mean part of the process succeeded, but there will be no ```<doc>``` entry in the output files for the
label(s) that failed.  If you have a failure, see [HarvestFailureRecovery.md](HarvestFailureRecovery.md) for further 
instructions.

### **Harvest** Output

**Harvest** produces the *solr_docs/* files, each of which contains up to 1000 ```<doc>``` elements, each corresponding
to one label processed.  A ```<doc>``` file looka like this:

    <doc>
    <field name="lid"><![CDATA[urn:nasa:pds:gbo-kpno:document:echspec_manual]]></field>
    <field name="objectType"><![CDATA[Product_Document]]></field>
    <field name="product_class"><![CDATA[Product_Document]]></field>
    <field name="citation_author_list"><![CDATA[Willmarth, D.]]></field>
    <field name="file_ref_size">1750</field>
    <field name="file_ref_size">342905</field>
    <field name="modification_date">2014-11-21T00:00:00.000Z</field>
    <field name="file_ref_name"><![CDATA[4mechspec1995.xml]]></field>
    <field name="file_ref_name"><![CDATA[4mechspec1995.pdf]]></field>
    ...
    </doc>
    
The ```name``` values indicate the field (a.k.a., "slot") in the registry database that will hold the associated values, 
which were extracted from the label. The fields are in no particular order - Solr doesn't care.
(The ```CDATA``` tag are superfluous and will, one hopes, disappear in future implementations.)

These files could be posted directly through the Solr API, but some additional processing is done by **registry**
(specifically, a stylesheet translation is applied) to modify some field values for better searching. 
The **registry** software is used to post data to the dockerized Solr instance.

## 4. Move *solr_docs/* Files

The dockerized Solr instance has extremely limited ability to reference disk space outside the docker container,
and it has to be configured as part of the container creation process (at least until we get a local Docker
expert here to tell us otherwise).  So in order to post the *solr-docs/* file(s)
created by the **harvest** run, they need to be copied to a directory that is already connected to the docker
container.  This directory is *registry-2.0.0/solr-docs*.

Copy the files you want to post into *registry-2.0.0/solr-docs/*.  You can copy files from multiple runs of 
**harvest**; you can copy an entire directory tree in the *registry-2.0.0/solr-docs/* directory and **registry**
will crawl it. What you cannot do is delete, rename, or replace the *registry-2.0.0/solr-docs/* directory with a
link - that will break the connection to the docker container.  If that happens, your posting operation will appear
to succeeed, but hang before completing the last step.  Re-establishing the proper directory and 
restarting the docker container will clear the problem.

## 5. Run **registry post** 

Once the files are in *register-2.0.0/solr-docs/*, this command should post the data to the "pds" collection
in the registry database:

    % docker exec -it -c pds -host proteus.astro.umd.edu -port 8983 -params "tr=add-hierarchy.xsl" /data/solr-docs
    
Note the addition of the ```-host``` and ```-port``` options not included in the **registry** documentation.  The
```-port``` option is probably not needed, since we are using the default port, but I haven't tested that myself.

This should run very quickly.  My last post took on the order of 10 seconds to post 7000+ ```<doc>``` elements in 
half a dozen different files.  Once this completes, the new data will be visible in the "pds" collection via the
Solr admin interface (http://proteus.astro.umd.edu:8983).


