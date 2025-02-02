# Primer on How to Use CityEngine Generally

## 1. Street Network Generation

After creating a new empty scene in CityEngine, go to

    Graph > Grow Streets

By _Growing Streets_, one randomly creates a network of roads and accompanying ground plots based on random distributions. Individual streets can be added in specific locations, shapes, and sizes, but is ultimately too costly for the generation of the requisite amount of data. It is only used after random network generation to fill in odd gaps (like in the below image) to build a cohesive city.

<img src="./readmeFigures/network_gaps.png" alt="Network Gaps"
	title="Network Gaps" width="250"  />

[comment]: #

From here, the **Basic Settings** of the street layout can be set. Here they are with their default values.

- Number of streets: 500
- Pattern of major streets: ORGANIC
- Pattern of minor streets: RASTER
- Long length: 150.0
- Long length deviation: 50.0
- Short length: 80.0
- Short length deviation: 20.0

Each of the two pattern settings can independently take on ORGANIC, RASTER, or RADIAL as pattern settings.

There are also accompanying presets for network generation below. These presets will automatically set the necessary parameters to resemble the layouts described.

- Berlin-like
- Radial
- Barcelona-like
- Spiral
- Siena-like
- Hexagon
- San Francisco-like

## 2. Target City Creation

After creating the street network in CityEngine (the wireframe per se), the actual buildings and grounds be applied to the scene. This is done in a few separate stages.

### A) Texture Acquisition

A series of building roof and facade textures were obtained from Google Maps (Earth). These were obtained by going to the target city in question and taking screenshots from Google Maps' different view modes.

- Satellite view for building roof textures
- Street view for building facade textures

The textures were sampled in an attempt to approximate the variety of building types unique to this "target" location. For example, in the accompanying cities, Austin had more industrial roofing while Vienna had more red-tiled roofing.

These screenshots were then consolidated into a library that could be sampled from during the city rendering.

Here are some examples of textures pulled for the city of Austin (from top: facade, roof, roof):

<img src="./readmeFigures/gf_1.png" alt="Facade Example"
	title="Facade Example" width="250"  />

<img src="./readmeFigures/flatRoof_47.jpg" alt="Flat Roof Example"
  title="Solar Panels" width="250"  />

<img src="./readmeFigures/flatRoof_49.jpg" alt="Flat Roof Example2"
  title="Industrial" width="250"  />

### B) .cga Rule Files

These rule files primarily fill in the wireframe that is the network of street layouts. It can broadly be broken down into a series of categories that can be changed independently and have feature selections within them. The changes we are most interested in are the City-Level and Building-Level Adjustments.

1. City-Level Adjustments
2. Building-Level Adjustments
3. Vegetation
4. Groundcover


#### i) City-Level Adjustments

These features in the code allow us to control distributions of different "parts" of the city, namely

- Highrise Area
- Commercial Area
- Apartment Area
- Residential Area

Thus, we can control the size of downtown and more residential areas of the city relative to the overall scene, including how much they overlap with each other. The type of zone an area is designated as will control the types of buildings generated in it, along with a lot about their size.

#### ii) Building-Level Adjustments

At building level granularity, the specific textures, the sizes, and the shapes of buildings are all controlled.

Attributes such as building shape (e.g. L-shaped, U-shaped, square) can be adjusted in the rule file. This includes pulling the size of the "L" or the "U" from a random distribution to create variety in the _precise_ shape.

Factors like building height can be uniformly increased by scaling factors or tweaked by adjusting average building height and narrowing or widening the standard deviation interval for the heights it will sample.

The textures it uses for building roofs and facades are selected from a directory on a uniform random distribution for each building.

#### iii) Vegetation

The types of vegetation (e.g. tropical, temperate) can be adjusted to fit the specific climatic zone the particular city is in.

Beyond the types of vegetation, the distribution of the amount can also be varied to have denser coverage over the city.

#### iv) Groundcover

The rule files can also control the groundcover between the different building lots in the city. These options primarily include grassy areas and parking spaces. It also controls the density of buildings and how much of the groundcover is actually visible.

## 3. Target City Image Generation

Once the city is actually rendered, the images are taken. This is done by running the script

    dynamic_shoot_syn_1_colorful_city.py

within CityEngine.

The first step of this is to use the CityEngine GUI and find the four farthest vertices of the city to plug in as the starting and ending coordinates for the camera script. Within the shooting script, parameters can be chosen to mimic real life imagery that might be obtained. These parameters include

- Light intensity
- Light angle
- Camera angle
- Camera height
- Camera lens type (Field of View)
- Image Size

Additionally, the distribution that the first three are sampled from can be set and the script will select from this for _each_ image taken. This allows us to collect a set of imagery for the same scene that is diverse in lighting and viewpoint. This has impacts on the the angle the images are taken at (obviously), but also things such as shadow length and opacity.

The image resolution is intimately dependent on the camera height, lens type, and image size together. Thus, to match a desired m/pixel resolution, a function has been written to automatically calculate camera height based on the lens used and size of image wanted. This automatic camera height calculator has passed testing for fully perpendicular to the ground camera angles. When you choose to tilt the camera, resolution is not consistent throughout an image due to the angle.

## 4. GT City Generation

Once we generate the set of RGB images, we can move on to the the city that will become the set of GTs. This uses a separate rule file. The primary job of these .cga rule files is to make all the building surface textures black while removing vegetation and other extraneous object renderings. This is done so when we take images of this version of the city, we can isolate anything that is all black as a building easily for the labels. To assist in this endeavor, the specialized rule files also remove vegetation and groundcover. This leaves only buildings and the lots that they sit on. One then hides the street network from view in CityEngine to hide the roads. This generates the barebones version of the city that makes the labelling of the synthetic data quick and easy.

## 5. GT City Image Generation

After generating the black-building version of the city, we move on to the image generation of it. The same script

    dynamic_shoot_syn_1_colorful_city.py

as for the RGB tiles is used with a few changes to the shooting aspect so that the labels turn out as cleanly as possible.

- Light intensity = 1.0
- Light angle = 90

The light settings are set to these values and the variation is also set to 0. This removes shadows that might look dark enough to be classified as buildings later and makes the overall GT image cleaner. Camera angle should remain the same and the script should be setting the same random seed as before in order to make sure that each tile has the same specific camera angle as for the RGB version of the tile.

## 6. Binarizing Labels

The preprocessing script

    pre_process_syn.py

loads in pairs of images for the same spot in the city, one RGB and one GT. First, it makes sure that at least 90% of the RGB tile is city, i.e. it removes images that were taken at the edge of the scene and thus include pure white. For images that meet this threshold, it then turns the GT images taken in CityEngine (where buildings are black) and turns them into image labels (where buildings are pure white (255,255,255) and everything else is black (0,0,0)). It saves the new pairs of images and these are what can then be used for training.
