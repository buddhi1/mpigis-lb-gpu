MPI-GIS software uses Message Passing Interface and MPI-IO to read and distribute the geospatial intersection computation among MPI processes running on multiple compute nodes in a cluster for geospatial join and overlay on a cluster of compute nodes.

This code is based on the following three papers:

1) Satish Puri, Anmol Paudel, Sushil K. Prasad:
MPI-Vector-IO: Parallel I/O and Partitioning for Geospatial Vector Data. ICPP 2018: 13:1-13:11

2) Satish Puri, Sushil K. Prasad:
A Parallel Algorithm for Clipping Polygons with Improved Bounds and a Distributed Overlay Processing System Using MPI. CCGRID 2015: 576-585

3) Puri, S. (2019). SpatialMPI: Message Passing Interface for GIS Applications. The Geographic Information Science & Technology Body of Knowledge (2nd Quarter 2019 Edition), John P. Wilson (Ed.). DOI: 10.22224/gistbok/2019.2.6

Example to show how to use: test_mpi-vector-io.cpp
```
Parser Interface (Parser.h)
	virtual list<Geometry*>* parse(const FileSplits &split) = 0;
```
FileSplits is defined as a list of "string". It represents a chunk of file, that an MPI process gets after file partioning using MPI-IO.

Example: WKTParser class implements Parser Interface to return a list of Geometry instances given a list of string representation of co-ordinates

```
int main(int argc, char **argv) 
{	
    Config args(argc, argv); 
    
    args.initMPI(argc, argv);
    
    FilePartitioner *partitioner = new MPI_File_Partitioner();
	
    partitioner->initialize(args);
    
    pair<FileSplits*, FileSplits*> splitPair = partitioner->partition();
    
    Parser *parser = new WKTParser();
    
    list<Geometry*> *layer1Geoms = parser->parse(*splitPair.first);
    cout<<layer1Geoms->size()<<endl;
    
    list<Geometry*> *layer2Geoms = parser->parse(*splitPair.second);
    
    // Filter step: Spatial partitioning into uniform/adaptive grid cells.
    Grid grid (numPartitions, universe);
    grid.populateGridCells(L1Geomsid, L2Geomsid);

    //Generate a mapping from MPI process id to list of cells. Default is block-cyclic.
    
    // Perform MPI all-to-all communication to exchange geometries. Each MPI process gets a list of (cellId, geoms) pairs. 
    // Geometries are grouped by cell.
    
    // Refine step: Spatial join computation in each cell.
    int refine(int cell, list<Geometry> L1cell, list<Geometry> L2cell);
    
    MPI_Finalize();
}
```
```
Envelope recv;
Envelope env(1,2,3,4);    
MPI_Allreduce(env, &recv, 1, MPI_RECT, MPI_UNION, MPI_COMM_WORLD);

Rect in_rect(1,2,3,4);  
MPI_Bcast(&in_rect, 1, MPI_RECT, 0, MPI_COMM_WORLD);
```

```
Build Instructions: make
Run: mpirun -np #processes ./mpiio #filePartitions datasets/sports datasets/cemetery 
Download datasets from http://spatialhadoop.cs.umn.edu/datasets.html and https://star.cs.ucr.edu/
Library dependency on GEOS to perform computational geometry algorithms (https://trac.osgeo.org/geos/)
```
# mpigis-lb-gpu
