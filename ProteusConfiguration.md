# Proteus Configuration

The configuration on proteus uses the registry-2.0.0 distro from EN with the Docker install option.  It
took some finagling to get the docker container to work properly with the local security/SELinux settings,
so unfortunately instead of using http://localhost:8983 as our access port we have to use the wide-open
substitute http://proteus.astro.umd.edu:8983.  This also causes failures in some of the EN docker commands
(as shown in the EN "Operation" manuals for harvest), because the localhost:8983 reference is hard-coded.
There are Solr API work-arounds that can be used.

## Locations and Content

The working directory for testing is */proteus2nb/pds*.  If you're not logged into proteus, this and all
following references should had */n* prepended (i.e., */n/proteus2nb/pds*).  All other directories mentioned
below are under this directory.

The registry installation directory is *registry-2.0.0/*.  Documentation for the registry distro is in the
*registry-2.0.0/doc/* directory.

The test data used to populate the registry is in the *TestData/* subdirectory.

There is a usable version of the harvest tool also in this directory, under the *harvest-2.0.0/* subdirectory.
The executable for harvest is *harvest-2.0.0/bin/harvest*.  The location is not important to the process, 
but if you want to execute the harvest tool from some other directory, you will have to edit the *harvest*
script to change the location of PARENT_DIR accordingly.  Documentation for the harvest tool is in the 
*harvest-2.0.0/doc/* directory.

The *Logs/* directory contains log files for various runs of the harvest tool on the test data.  Typically I
retained the successful run, and any interesting or undiagnosed failures.

The *SolrDocs/* directory contains the output files from the harvest process, which are then posted to the
Solr database to fill the "pds" collection. There are multiple <doc> elements in each file - up to 1000.
The harvest tool produces multiple files when the number of products processed exceeds 1000, which is why
there are subdirectories for a few of these data bundles.

The *HarvestPolicyFiles/* directory contains the per-bundle directions for processing the data from each
*TestData/* bundle.  The changes from bundle to bundle are very minor - user "diff" to highlight them.
Two of these include attributes from the **img:** discipline namespace.

The *HiddenConfig/* files are copies of files Jordan sent me that were used in creating the distros for
__harvest__ and __registry__, but which are not included in a form where users could inspect them. They're
here for reference.

The __clean__ and __gorun__ executables are convenience scripts.

The __Notes.txt__ file is an earlier version of this file, before I did some clean-up.

Anything else was created as part of a test or correction, and shouldn't be considered authoritative or
even informative.
