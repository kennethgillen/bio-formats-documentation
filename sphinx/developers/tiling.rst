Tiled reading and writing in Bio-Formats
========================================


Reading tiled images
--------------------

The reading of tiled images is straightforward and can be done in much the same way as reading a full image. In this case, to read an individual tile, we pass to the reader parameters for the x and y coordinates of the tile to read and the height and width of tile desired.

::

    byte[] tile = reader.openBytes(image, tileX, tileY, tileSizeX, tileSizeY);

For TIFF-based readers, if the image has been written using tiles, then the tile width and height used can be found as below. These values can then be used with the above command to read the correct tiles individually.

::

    IFD tileIFd = reader.getIFDs().get(0);
    int tileHeight = tileIFd.getIFDIntValue(IFD.TILE_LENGTH);
    int tileWidth = tileIFd.getIFDIntValue(IFD.TILE_WIDTH);

Introduction to tiled writing
-----------------------------

Tiled writing is currently supported for TIFF-based formats.
To set up an image writer to use tiling the following 2 API functions are provided:

::

	public int setTileSizeX(int tileSize) throws FormatException
	public int setTileSizeY(int tileSize) throws FormatException

Each function takes in an integer parameter for the desired tile size. As not all tile sizes are supported, the image writer will round the requested value to the nearest supported tile size. The return value will contain the actual tiling size which will be used by the writer.

To find out the tiling size currently being used at any point there are 2 further API functions to get the current tile size for a writer. If tiling is not being used or is not supported then the full image height and width will be returned.

::

	public int getTileSizeX() throws FormatException
	public int getTileSizeY() throws FormatException 

The tiling parameters for writers must be set after the image metadata is set. 
An example of initializing a writer for tiling is shown below.

.. literalinclude:: examples/SimpleTiledWriter.java
   :language: java
   :start-after: initialize-tiling-writer-example-start
   :end-before: initialize-tiling-writer-example-end


Simple tiled writing
--------------------

The simplest way to write a tiled image is to set the tiling parameters on your image writer as 
above and have the writer automatically handle the tiling. Once the tile sizes have been set 
you may simply read and write your image files as normal.

.. literalinclude:: examples/SimpleTiledWriter.java
   :language: java
   :start-after: tiling-writer-example-start
   :end-before: tiling-writer-example-end

Full working example code is provided in    
   :download:`SimpleTiledWriter.java <examples/SimpleTiledWriter.java>` - code from that class is
   referenced here in part. You will need to have :file:`bioformats_package.jar` in your 
   Java CLASSPATH in order to compile :file:`SimpleTiledWriter.java`.

Reading and writing using tiling
--------------------------------

For the most efficient reading and writing of tiles you may instead wish to read in and write out the individual image tiles one at a time. 

To do this you can set up the reader and writer as in the previous example above. In this case, 
when setting the tile height and width used it is important to store the return values which will be the valid tile size used by the writer.

::

      // set the tile height and width and store the actual values used by the writer
      int tileSizeX = writer.setTileSizeX(tileSizeX);
      int tileSizeY = writer.setTileSizeY(tileSizeY);

This time for each image you must determined the number of tiles using the actual tile height and width being used.

.. literalinclude:: examples/TiledReaderWriter.java
   :language: java
   :start-after: tiling-calculations-example-start
   :end-before: tiling-calculations-example-end

Now each tile can be read and written individually.

.. literalinclude:: examples/TiledReaderWriter.java
   :language: java
   :start-after: tiling-example-start
   :end-before: tiling-example-end

Full working example code is provided in
   :download:`TiledReaderWriter.java <examples/TiledReaderWriter.java>` - code from that class is
   referenced here in part. You will need to have :file:`bioformats_package.jar` in your 
   Java CLASSPATH in order to compile :file:`TiledReaderWriter.java`.

Tiles which overlap image size
------------------------------

It may not always be the case that the tile size divides evenly into the image height and width.
In these scenarios in which the last column or row of tiles overlaps with the image boundary, a smaller tile 
is instead read or written for the final row or column. If a full sized tile with padding is written then a 
:javadoc:`loci.formats.FormatException <loci/formats/FormatException.html>` will be thrown for ``Invalid tile size``.

To deal with this we can modify the previous tiled writing example to check if the tile size will overlap
the image boundaries. If it will not overlap then the full tile size is used, if it does then the tile size 
for that tile is modified to reflect the partial tile.

.. literalinclude:: examples/OverlappedTiledWriter.java
   :language: java
   :start-after: overlapped-tiling-example-start
   :end-before: overlapped-tiling-example-end

Full working example code is provided in
   :download:`OverlappedTiledWriter.java <examples/OverlappedTiledWriter.java>` - code from that class is
   referenced here in part. You will need to have :file:`bioformats_package.jar` in your 
   Java CLASSPATH in order to compile :file:`OverlappedTiledWriter.java`.