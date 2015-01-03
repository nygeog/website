---
layout: post
title:  "Converting File Geodatabase (FGDB) to Shapefile or CSV via OSGEO4W with OGR2OGR"
date:   2014-08-28 11:31:10
categories: 2014, File Geodatabase, Shapefile, OSGEO4W
---

##The Problem:
I've recently run into a problem. I've been running some [Near Analysis (arcpy)](http://resources.arcgis.com/en/help/main/10.1/index.html#//00080000001q000000) on a project that I've been working on (more on that at a later blogpost). The problem is as follows; I'll have 1,460 (365 * 4 tables) geodatabase tables (in 12 geodatabases - days and months) that each comprise of 400,000+ records. 

Normally, when I'm trying to get my GIS data out of Esri format (such as FGDB) I used a script like this:

#####File Geodatabase (feature class or table) to CSV (python)

	import arcpy
	import csv

	wd = #<working directory>
	table   = wd+"/treedn.gdb/trees"
	outfile = wd+"/treedn/trees.csv"      

	fields = arcpy.ListFields(table)
	field_names = [field.name for field in fields]

	with open(outfile,'wb') as f:
    	w = csv.writer(f)
    	w.writerow(field_names)
    	for row in arcpy.SearchCursor(table):
        	field_vals = [row.getValue(field.name) for field in fields]
        	w.writerow(field_vals)
    	del row
    	
    	
What this script does is basically read each table, row by row and then write each row to a CSV. However, I've found it to be painfully slow on larger datasets. 

<em>Quick caveat: I'm assuming that this FGDB export using OGR2OGR will be faster. I haven't tested a comparison yet, but early trials make me think it will.</em>

##The Pain:
My first inclination was to try this on Mac. However, I can't seem to get the [Esri File Geodatabase API 1.3 ](http://www.esri.com/apps/products/download/#File_Geodatabase_API_1.3) installed. I'm not savvy enough to figure out what I'm doing wrong. I tried the code from README file a few times to no avail and am not really willing to bother with Esri support (I also don't know if they offer support for this API?). 

##The Solution:
I posted some of my pains towards Twitter and [@jasonscheirer](https://twitter.com/jasonscheirer) responded with a few welcome words of advice. Using PostgreSQL was suggested as well as OSGEO4W. For some reason on Monday night OSGEO4W's website was down but on Tuesday evening (during the advanced D3 presentation on projections at [ @MaptimeNYC](http://www.meetup.com/Maptime-NYC/)) I was able to download and install [OSGEO4W](http://trac.osgeo.org/osgeo4w/). I'm not going to bore you with how to get the File Geodatabase Driver installed as there is a totally awesome [StackExchange on FileGeodatbase GDB Support in QGIS](http://gis.stackexchange.com/questions/26285/file-geodatabase-gdb-support-in-qgis).

So with OSGEO4W up and running and then the Advanced Install (or whatever that step was from StackExchange) I was able to start using the OSGEO4W command prompt to try and convert Geodatabases. 

With help from MaptimeNYC's [@ebrelsford](https://twitter.com/ebrelsford), we were able to conjure up some GDAL/OGR code to make the conversion happen. 

First solution was to just see if it could convert that test file (all the 2010 Census tracts in the US - sort of a silly dataset to use for testing since its so large) to Shapefile. Here's the code:

#####File Geodatabase to Shapefile (OSGEO4W command line)

	ogr2ogr -overwrite -f "ESRI Shapefile" "W:\GIS\Data\Census\census_2010\tracts\test.shp" "W:\GIS\Data\Census\census_2010\tracts\census.gdb" "tracts_2010"

The next step was to test CSV:

#####File Geodatabase to CSV (OSGEO4W command line)

	ogr2ogr -overwrite -f CSV "W:\GIS\Data\Census\census_2010\tracts\testCSV.csv" "W:\GIS\Data\Census\census_2010\tracts\census.gdb" "tracts_2010"
	
Success.

I still need to learn the OGR syntax but I'm very pleased this worked.

##What's Next:
My next step is to write a python file that will run this OGR2OGR conversion and loop through the tables. My Near Analysis are still running (plus Labor Day) so more on that later...
	
	