---
title: "3D Polylines from a table of points"
date: 2018-06-22T14:24:18-08:00
draft: false
---

This project came up at work as we needed to take lines, split at vertex, interpolate them using 3D Analyst, and then take all of those points with a new elevation and assemble them back into polylines. First and foremost this process had already been fleshed out by one of our GIS Analysts so we knew it would work. Dealing with 11,000 lines it ended up being 500,000 points after the were split. Using ArcGIS Desktop the process took 4 hours from start to finish. 

Baseline: 
*  11,000 lines (input)
*  500,000 points (vertices)
*  4 hour run-time. 

For anybody familiar with ArcGIS and somewhat large datasets or heavy processing tasks 4 hours is not horrible. I used to generate data that would routinely take an entire weekend and fails 36 hours in. Rinse and repeat. 

The desktop process used ET GeoWizard software to crack and pack the lines/points. I was able to integrate that and write a script using Arc Objects that replicated the process. Same inputs and same outputs. The issue was it took 22 hours to run. 

Iteration 1: 
*  ET GeoWizard + .Net
*  Success!
*  Not practical - 22 hours is almost 18 hours more than 4 hours. 

With that I reached out to ET GeoWizards to make sure I was not missing something obvious. I was hoping for , "Oh, set options = RUN_FAST " but instead got, "Not sure". To be fair this stuff is hard to figure out and I provided very little information. That being said I am also not on the most current version of anything (except Node.js) so I did not expect much. Thus started option 2, a hand rolled, bespoke, artisan, .Net solution. 

The process can be broken down into a series of steps and I am only pasting a few key pieces. The biggest part that I needed to come up with was taking my points that had X,Y,Z,M values and correctly create poly-lines from them.  

So, in order to do this I created a method called `Create3dLinesFromPoints` with that the hard part was over. 
`private void Create3DLinesFromPoints(IFeatureClass interpolatedPointFc, dynamic lineDestinationSchema)`
This method takes my points and an SDE schema. The second part could be another IFeatureClass for lines and save you some trouble of getting a workspace. 

After you get the lineFC workspace and FeatureClass you will be ready to go. 

The way I went about this, which I am not claiming to be the most efficient, is that I start with getting a cursor for the unique data. For my example it is a field called "geo_id". 

```
var lineFC InsertCursor = lineFCSdeFeatureClass.Insert(true);
var lineFC FeatureBuffer = lineFCSdeFeatureClass.CreateFeatureBuffer();
int skipCount = 0;
ICursor uniquePointsCursor = (ICursor) interpolatedPointFc.Search(null, false);

IDataStatistics dataStatistics = new DataStatisticsClass();
dataStatistics.Field = "geo_id";
dataStatistics.Cursor = uniquePointsCursor;

System.Collections.IEnumerator enumerator = dataStatistics.UniqueValues;
enumerator.Reset();
//First loop gives us our unique points....
while (enumerator.MoveNext())
   {
   }
```

That block also opens up the insert cursor on out lineFC in order to write records to it later. 

With that we need to fill in what will happen in that while block. The ful method (some names changed from above): 

```
          var workspaceEdit = (IWorkspaceEdit2)groundElevationSdeWorkspace;
          Log.Debug("performing workspaceEdit.PerformNonVersionedEdit");
          workspaceEdit.PerformNonVersionedEdit(() => {
            GeoprocessingHelper.DeleteRows(groundElevationSdeConnection.GetFullPathToFeatureClass(groundElevationSdeFeatureClass));

            var groundElevationInsertCursor = groundElevationSdeFeatureClass.Insert(true);
            var groundElevationFeatureBuffer = groundElevationSdeFeatureClass.CreateFeatureBuffer();
            int skipCount = 0;
            ICursor uniqueRoutesCursor = (ICursor) interpolatedPointFc.Search(null, false);

            IDataStatistics dataStatistics = new DataStatisticsClass();
            dataStatistics.Field = "geo_id";
            dataStatistics.Cursor = uniqueRoutesCursor;

            System.Collections.IEnumerator enumerator = dataStatistics.UniqueValues;
            enumerator.Reset();
            //First loop gives us our unique routeids....
            while (enumerator.MoveNext())
            {
                object geoIdObj = enumerator.Current;
                //InnerCursor
                int geoIdField = interpolatedPointFc.Fields.FindField("geo_id");
                IQueryFilter queryFilter = new QueryFilterClass();
                queryFilter.WhereClause = "\"geo_id\" = " + geoIdObj.ToString();
                //int count = interpolatedPointFc.FeatureCount(queryFilter);
                ITableSort tableSort = new TableSortClass();
                ITable table = (ITable) interpolatedPointFc;
                tableSort.Table = table;
                tableSort.Fields = "POINT_M";
                tableSort.QueryFilter = queryFilter;
                tableSort.set_Ascending("POINT_M", true);
                tableSort.Sort(null);
                ICursor cursor = tableSort.Rows;

                int geoIdNameIndex = cursor.Fields.FindField("geo_id");
                int measureNameIndex = cursor.Fields.FindField("POINT_M");
                int zFieldNameIndex = cursor.Fields.FindField("POINT_Z");

                //Create a pointCollection for a line
                IPointCollection pointCollection = new PolylineClass();
                var zaware = (IZAware) pointCollection;
                zaware.ZAware = true;
                var maware = (IMAware) pointCollection;
                maware.MAware = true;

                //Loop to find the points to add to point collection
                IRow row = null;
                while ((row = cursor.NextRow()) != null)
                {
                    //Each point for the Route - Sorted by measure (ASC)
                    Log.Debug("GEOID" + row.get_Value(geoIdNameIndex) + " POINT_M : " + row.get_Value(measureNameIndex));
                    //var Missing = Type.Missing;
                    var currentRow = (IFeature) row;
                    var point = (IPoint)currentRow.Shape;
                    point.Z = (double)row.get_Value(zFieldNameIndex);
                    point.M = (double)row.get_Value(measureNameIndex);
                    pointCollection.AddPoint(point);
                }


                var shapePolyline = pointCollection as IPolyline;
                //Check vertice count
                // Set the buffer geometry to that of the shape to add.
                if (shapePolyline.Length != 0)
                {
                    //Build the Polyline feature
                    int groundElevationRouteField = groundElevationSdeFeatureClass.Fields.FindField("ROUTE");
                    //Add Attributes
                    groundElevationFeatureBuffer.Value[groundElevationRouteField] = geoIdObj.ToString();
                    //Add shape
                    groundElevationFeatureBuffer.Shape = shapePolyline;
                }
                else
                {
                    skipCount++;
                }

                // And insert the feature into the buffer.
                groundElevationInsertCursor.InsertFeature(groundElevationFeatureBuffer);
                Marshal.FinalReleaseComObject(cursor);
            }

            // We're done--flush the cursor and free appropriate workspaces
            groundElevationInsertCursor.Flush();
            Log.Debug("Tried real hard but skipped " + skipCount + " features due to a Zero length geometry.");
            Marshal.FinalReleaseComObject(uniqueRoutesCursor);
            Marshal.FinalReleaseComObject(groundElevationInsertCursor);
        }); //End workspace edit

    }

```

Cool. I ran this on a subset and thought all was well. Before I ran it on the whole I saved the unique ids out to a list to make life a bit simpler. Kicked it off at 5:00 PM and it was running when I got in a just after 10:00 the next day. At this point I said, "Shit. This sucks more than #45". Alright back to the drawing board. I started down these two paths as it was the proven path and it worked in desktop. Esri Geoprocessing tools always work well in .Net for me so I transitioned over to using some core built in tools. 

Iteration 2: 
*  .Net
*  Success!
*  Not practical - Still a long ass time. Quit before it finished as we were already well over 4 hours. 

So, as these things go we stay patient and remember that we can usually crush it by keeping it simple. In Desktop I was able to run 2 tools, pointsToLines and calibrateRoutes. The first I need to make sure I enable M and Z values in the geoprocessing environment or I will only have Z available. The second will take my points and re-calculate the M values from the attributes in the point feature class. I tested this and again it seemed promising. 
```
PointsToLine(interpolatedPointsLyr, pointsToLinesLyr, "geo_id", "POINT_M");

CalibrateRoutes(pointsToLinesLyr, "geo_id", interpolatedPointsLyr, "geo_id", "POINT_M", calibratedLinesLyr, "MEASURES");
```

First of all you will notice that this is so effing simple to read that I am willing to forgo some performance. No Multi Cursors, no looping twice, no tracking variables, etc. That is nice. Plus Esri usually has spent time optimizing. I need to optimize stuff to but I have limited time and budget. Lets see how this goes. Kicked the process off at 12:48 PM.  
AND... Done. 
Maybe not that fast but the first run was 1:13 minutes and the next was 58 minutes. 

Iteration 3: 
* .Net pure Esri tools
* Total execution time: 00:58:37.3705807
* Awesome. 

I would say there are no real lessons learned besides that sometimes you just need to try a few things and see what sticks. In this case we were able to bring down the running time from 4 hours to 1. Not bad...
