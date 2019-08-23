# Harvest Failure Recovery

Harvest does a lot of processing and when it doesn't complete a expected there can be a fair amount of clean-up 
needed.

## What **harvest** Does

**Harvest** crawls the directories listed in the config file and parses every elligible file it encounters.
As it parses the label, it does several things:

* It converts the label into a set of fields identified purely by their xpath, and posts the resulting information
to the *xpath* collection in the registry database.
* It converts the label in its entirety into a binary blob which is also saved into a collection called *.system"
in the registry database.
* It processes relationships and type conversions peculiar to PDS4 and converts them to lines in the output 
```<doc>``` file.
* It produces the output *solr_doc_0.xml* file and its siblings, where needed.

Because **harvest** posts data directory to the registry database, cleaning up from a **harvest** failure can 
involve several steps.

## The xpath and .system Collections

The *xpath* collection has no clear, intended purpose.  (It might actually be the result of a misunderstood 
conversaion about slot-naming.)  It will very likely disappear in the near future, so don't get attached to
it.

The *.system* collection has a vague, hand-wavy use in possibly speeding up the process of re-harvesting, 
maybe, under certain circumstances.  It will hang around for awhile, but it's not clear if it can, in fact,
save significant time if re-harvesting of products turns out to be something that is needed on a regular
basis.

## When **harvest** Fails

If **harvest** fails the first time you run it on a set of products, it probably won't produce any *solr-docs/* files, 
but it probably will make entries in the *xpath* and *.system* collections. 
You might also decide once you have run **harvest** that you want to make some adjustments and
run it again to produce better *solr-doc/* files.  

In either case, the existing *xpath* and *.system* entries will 
prevent new *solr-docs/* files from being created.  In the log file you will see a "WARNING" message about products 
already existing in one of these collections.  Despite the "WARNING" status (as opposed to "ERROR"), **harvest**
will *not* create output ```<doc>``` entries for products that generate WARNINGs.  If your **harvest** goes faster than
expected without producing a Java stack trace, immediately check the success summary at the bottom of the log file. 
You will most likely see a notation that many products were not registered.

In order to re-run **harvest**, you will need to delete the existing contents of the *xpath* and *.system* collections
for those things you plan to re-process.  Deleting should be done with caution, of course.  Deleting can be done with 
either the Solr Admin interface or a properly formatted ```curl``` command to the Solr API, if you can do a better job
of figuring out the Solr query documentation than I can.

The same method can be used to delete entries from the *pds* collection, if you decide you want to start over with a
particular harvest/registry run.

## The Solr Admin Interface

The admin interface is here: http://proteus.astro.umd.edu:8983

The Solr documentation for the version of the interface we're using is here: 
https://lucene.apache.org/solr/guide/7_7/using-the-solr-administration-user-interface.html.  The parts mentioned below
are in the "Collection-Specific Tools" section of that document.

### Choose Your Collection

You'll see the main
dashboard with some info on overall Solr status. Look to the left and you'll see two bubttons that look greyed-out.  The
top one says "Collection Sele...".  Click on that and you should see a drop-down list containing three entries: *xpath*, 
*.system*, and *pds*.  Select the collection you want to delete from, and your screen will update.

### Test Your Selection

Before attempting to delete content from the collection (assuming you don't want to trash the whole collection - and you
don't, do you?), you should make sure you have a query string that will select only the products you want to delete. Look 
to the left again, and you should see a new menu under the button that contains the name of the collection your selected
(you use this button to change collections).  Select the "Query" line and you should then see a form in your main pane.

If you just hit the big, blue "Execute Query" button now, you will see a sample of what is in the collection in JSON format.
The *xpath* and *.system* contents will look starkly different, but if you scroll around you will see that both have a
field called "id", and that field has a value that is based on the LID of the product associated with that entry.  This is
the unique identifier field in these collections, and I've found it to be very useful for clearly identifying data from a
failed **harvest** run that I want to delete.

To the left of the data you'll see a bunch of form fields.  Near the top is one labeled "q" that currently contains 
the string "*:*".  This is your general query box.  The format for this box is *<field name>:<value to match>*. You
can use this to test whether your query selects the right products.

For example, if I want to delete all the **xpath** collection entries created when I was trying to harvest the bundle
with the id "urn:nasa:pds:gbo-kpno", I can search for all products in that bundle by entering ```id:*gbo-kpno*``` in
the *q* box.  If you do this and hit the "Execute Query" button, your results will look something like this:

    {
      "responseHeader":{
        "zkConnected":true,
        "status":0,
        "QTime":133,
        "params":{
          "q":"id:*kpno*",
          "_":"1566577077692"}},
      "response":{"numFound":4684,"start":0,"docs":[
          {
            "Product_Document.Identification_Area.product_class":["Product_Document"],
            "Product_Document.Identification_Area.Modification_History.Modification_Detail.modification_date":["2014-11-21"],
      ...
    }
    
Notice about nine lines down, on the line beginning with "response", there is a field called "numFound" that tells you 
how many products in total matched the query.  If this is the number you expect, then you've probably got the right query.

When you are satisfied you have the right query, copy it to your clipboard.

### Deleting Entries

Now click on the "Documents" line in left collection menu, and you will be presented with another form.

The first field should say "/update", which it does by default.

Below that is a grey menu button.  Select the "Solr Command (raw XML or JSON)" option.

Now in the big box you will enter a delete command.  If I want to delete all my "urn:nasa:pds:gbo-kpno" bundle entries,
for example, I would enter this in the box - pasting my tested query into the middle line:

    <delete>
      <query>id:*gbo-kpno*</query>
    </delete>
    
(Spacing is for readability - you can put this all in one line if you prefer.)  If you're doing this to reset after a 
**harvest** failure, you're going to have to do this
at least twice - so you might want to take a moment to save the contents of the large box to your clipboard before clicking
the "Submit Document" button.

Once you clock "Submit Document", you'll see a little bit of JSON reponse, the first line of which will say either "success" 
or "failed".  If "success" and you try searching for the matching entries again, there should now be no entries returned.
"Failed" is likely the result of a syntax error - fix and repeat.

If you're resetting to **harvest** again, do the above "Documents"-delete 
for both the *xpath* and *.system* collections. If you need to re-post the *solr-docs/* file(s) you'll need to delete
the corresponding existing entries in the *pds* collection. For the *pds*
collection, the unique ID field is actually called "LID", but otherwise the process is identical.

### Retry Harvest

Once the previous content has been deleted from all **xpath** and **.system**, you can re-run **harvest**.  

(If you only want to re-post the *solr-docs/* files, you only need to clear out the **pds** collection.)
