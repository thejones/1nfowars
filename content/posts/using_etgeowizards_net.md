---
title: "Using ET Geowizards in .Net"
date: 2018-06-09T14:09:43-08:00
draft: false 
---
The only part that I really want to touch on and mostly for my own reference, is that most of these tools take a feature class as input. The Esri Geoprocessing tools also need a feature class but are more flexible on *how* it is passed in. 

Works with Esri: 
```
var tempFileGeodatabase = "C:\\temp\\db.gdb\\points";
var interpolatedPointsLyr = tempFileGeodatabase + "\\InterpolatedPoints_Lyr";
AddGeometryAttributes(interpolatedPointsLyr); 
```

ET Geo Wizards: 
```
ETGWCore wizard = new ETGWCore();
IWorkspace workspace = FileGdbWorkspaceFromPath(tempFileGeodatabase);
IFeatureClass interpolatedPointFc = featureWorkspace.OpenFeatureClass("InterpolatedPoints_Lyr");
PointsToPolylines(interpolatedPointFc, interpolatedLinesLyr, wizard);

private IFeatureClass PointsToPolylines(IFeatureClass interpolatedPointsFc, dynamic interpolatedLinesLyr, ETGWCore wizard)
{
  return wizard.PointsToPolylines(interpolatedPointsFc, interpolatedLinesLyr, "GEO_ID", "ET_ORDER", "POINT_Z", "ET_M");
}

public IWorkspace FileGdbWorkspaceFromPath(string path)
{
  IWorkspaceFactory workspaceFactory = new FileGDBWorkspaceFactoryClass();
   return workspaceFactory.OpenFromFile(path, 0);
}
```
I pulled this out of another file so it could be written a bit better as a standalone example. The point is, in Esri they let you basically just pass a connection string and do it for you. With ET Geo Wizards you need to make sure you have the correct work-space and pass the IFeatureClass into the tool.

