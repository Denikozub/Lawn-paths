# Lawn paths recognition system


## Installation

Install from repository:
~~~
pip install -e "git+https://github.com/Denikozub/Lawn_paths.git#egg=lawn_paths"
~~~
Warning: package requires GeoPandas to be installed, which can be problematic on Windows. [This](https://towardsdatascience.com/geopandas-installation-the-easy-way-for-windows-31a666b3610f/) article may help.


## Docker

~~~
docker run -d -it —name final —mount type=bind,source="$(pwd)"/target,target=/app kadmus_map
~~~
target/ - TIF and TFW files directory  
[Docker link](https://disk.yandex.ru/d/32I0DB2AxfSvDw)  


## Description

### Shapefile
[Documentation](https://github.com/Denikozub/kadmus-dev#build-shapefile)  
The program builds a shapefile based on the results of the neural network - NPY image masks and corresponding TFW files of the world.
It is possible to filter the results, smooth the final lines and choose the coordinate system.  

Filtration:
* By width
* According to the distance between trails
* According to the size of the bounding box
* By area  

Additionally, it is possible to efficiently (6 seconds on test data) check intersections with buildings in Moscow to reduce the likelihood of errors. Data downloaded from open source [OpenStreetMap](www.openstreetmap.org), processed and compressed.

### Visualization
[Documentation](https://github.com/Denikozub/kadmus-dev#visualize)  
Implemented the ability to plot found trails on an interactive map OpenStreetMap using the open source service [Leaflet](https://leafletjs.com/). This solution does not require paid API of such services as Google Maps or Yandex Maps and can be used in commercial projects.

### Iterative Makrup
[Documentation](https://github.com/Denikozub/kadmus-dev#preliminary-markup)  
An iterative approach to data labeling using deep learning has been developed and implemented:
1. Mark up some data manually
2. Train the neural network on labeled data
3. Apply a neural network to help label the next piece of data
4. Go to step 1


## Documentation

### Application

~~~
python visual_build/main.py
~~~

### Preliminary Markup

Run using command prompt
~~~
python preliminary_markup/pipeline.py get_mask img.tif
~~~
Returns a mask where the areas found by the neural network are highlighted in red on a white background.
The mask is stored in img_mask.tif  

It remains to erase the extra red marks, then convert the mask to .npy; command to convert:
~~~
python preliminary_markup/pipeline.py get_npy img_mask.tif
~~~
Red is replaced by white, everything else is black.  

Command to view the result of applying a mask and an image:
~~~
python preliminary_markup/pipeline.py blend image.tif mask.tif  
~~~
For each next stage of training the neural network, it is necessary to update the pipeline.


### Build shapefile

~~~python
from lawn_paths.map_builder.shapefile import build_shapefile

build_shapefile(dataset_directory: str,
                file_list: list = None,
                buildings_file: str = "Moscow_Buildings.zip",
                output_filename: str = 'paths.shp',
                crs: str = 'epsg:32637',
                max_path_distance_cm: float = 100.,
                max_path_width_cm: float = 80.,
                min_bbox_size_m: float = 1.,
                max_bbox_size_m: float = 200.,
                max_path_area_m2: float = 100.,
                p_epsilon: float = 0.3,
                c_epsilon: float = 2.)
~~~

__dataset_directory__: directory where .NPY mask files and .TFW world files are contained  
__file_list__: list of filenames to be processed (without extensions)  
__buildings_file__ file with buildings (polygons) in epsg:4326  
__output_filename__: name of the output file (should be .SHP)  
__crs__: initial coordinate reference system  
__max_path_distance_cm__: max distance between paths for them to be connected in cm  
__max_path_width_cm__: max path width  
__min_bbox_size_m__: min size of path's bounding box in meters  
__max_bbox_size_m__: maxsize of path's bounding box in meters  
__max_path_area_m2__ max path area in squared meters  
__p_epsilon__: RDP parameter to smooth path polygons  
__c_epsilon__: RDP parameter to smooth path polygon centerlines  


### Visualize

~~~python
from lawn_paths.map_builder.visualize import visualize

visualize(filename: str, output_file: str)
~~~
matplotlib==3.3.2 required  
__filename__ path to SHP file with paths to visualize  
__output_file__ path to HTML file with interactive map  
