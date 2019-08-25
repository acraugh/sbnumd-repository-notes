# Creating a New Search Field in the Registry Database

This is at minimum a three-part process at the moment, even for a simple (i.e., not to be defined as "faceted") search field.
There are three types of files involved in defining a new search field:

1. The harvest search site configuration files that define what will be collected from each product type;
2. The harvest policy file that is created for the specific collection or bundle; and
3. The schema file that defines the fields of the Solr/Lucene datbase (i.e., The Registry database).

Most, if not all, core (i.e., in the *pds:* namespace) attributes that are reasonable candidates for search fields are already in the 
default site configuration files provided in the *harvest* distro. The discipline namespace fields that should also be included will
be systematically added before we go into production, but you may want/need to add some yourself for testing.  Other local dictionary
fields will also be needed down the road.

__*Note:*__
