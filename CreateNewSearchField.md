# Creating a New Search Field in the Registry Database

This is at minimum a three-part process at the moment, even for a simple (i.e., not to be defined as "faceted") search field.
There are three types of files involved in defining a new search field:

1. The harvest search site-configuration files that define what will be collected from each product type;
2. The harvest policy file that is created for the specific collection or bundle; and
3. The schema file that defines the fields of the Solr/Lucene datbase (i.e., The Registry database).

Most, if not all, core (i.e., in the *pds:* namespace) attributes that are reasonable candidates for search fields are already in the 
default site configuration files provided in the *harvest* distro. The discipline namespace fields that should also be included will
be systematically added before we go into production, but you may want/need to add some yourself for testing.  Other local dictionary
fields will also be needed down the road.

> __*Note:*__ The harvest *search-conf* files have an unspecified hierarchy for determining which one will be applied.  Be careful 
> about what files you are editing and what files you are applying when running **harvest**.  It is probably better to used the
> "-C" option and be sure of what site configuration files are being read than relying on "default" directory locations.

> We will have a common *search-conf* directory for our routine production processing, but you may also have a private directory
> you use for your test-harvesting or development purposes.

## Example: Adding a Spectral Search Term

Let's say we're processing our first spectral collection, and we want to add the spectral dimension type (*wavelength*, 
*frequency*, or *wavenumber*) to the search parameters in our registry.  The xpath to this attribute from the top of the
Spectral Dictionary wrapper class is:

    sp:Spectral_Characteristics/sp:spectral_bin_type
    
Note that the namespace identifier, ```sp:``` must be included at all levels, not just the top class name.    

### Updating Harvest Search Site Configuration Files

The ```sp:Spectral_Characteristics``` class is typically found in observational files and in the collection product for spectral 
collections.  It is unlikely to appear in a bundle label (unless we had a bundle that contained only spectral collections), so 
for now we will only look for the spectral type in observational and collection product labels.  That means we need to add it 
to two files in the *search-conf* (or equivalent) directory we'll be referencing when running **harvest**.
