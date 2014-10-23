leaflet-docset
==============

Documentation for [PostGIS](http://postgis.net/) ("a spatial database extender for PostgreSQL object-relational database") in the Dash [docset](http://kapeli.com/docsets) format, ready for offline reading and searching.

To Generate Docs
----------------
The docs are copied from the PostGIS site and indexed using a Ruby script. To set up and run the Ruby script:

````   
   cd postgis-docset
   bundle install
   rake
   
````
