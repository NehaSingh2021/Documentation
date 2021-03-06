# Overview
# Table of Contents
1. [Overview](#overview)
2. [Table of Contents](#table-of-contents)
3. [Input](#input)<br/>
    - [Radiometric Orthomosaic](#radiometric-orthomosaic)
    - [Inferno Orthomosaic](#inferno-orthomosaic)
    - [RGB Orthomosaic](#rgb-orthomosaic)
    - [Panel orientation](#panel-orientation)
    - [No of columns in a table](#no-of-columns-in-a-table)
    - [No of rows in a table](#no-of-rows-in-a-table)
4. [Preprocessing](#preprocessing)
    - [Normalization of radiometric orthomosaic](#normalization-of-radiometric-orthomosaic)
    - [Rotation](#rotation)
    - [Slicing](#slicing)
    - [Annotation (required for training and QA)](#annotation-required-for-training-and-qa)<br/>
       i [Defect Annotation](#defect-annotation)<br/>
       ii [Table Annotation](#table-annotation)
    - [Image enhancement](#image-enhancement)
    - [Augmentation](#augmentation)
5. [Model Training](#model-training)
    - [Training the model for detection of defects](#training-the-model-for-detection-of-defects)
    - [Validating the model for defects detection](#validating-the-model-for-defects-detection)
    - [Training the model for detection of tables](#training-the-model-for-detection-of-tables)
    - [Validating the model for table detection](#validating-the-model-for-table-detection)
    - [Masking and Contouring](#masking-and-contouring)
    - [Mapping](#mapping)
6. [Deployment of the model](#deployment-of-the-model)
    - [Creating SO files](#creating-so-files)
    - [Directory structure](#directory-structure)
    - [Input Path](#input-path)
    - [API’s](#apis)<br/>
          i [Defect Detection](#defect-detection)<br/>
          ii [Get Result](#get-result)<br/>
          iii [Defect Mapping](#defect-mapping)<br/>
          iv [Table Detection](#table-detection)
7. [Database](#database)

# Input
## Radiometric Orthomosaic
Radiometric orthomosaic image is made by stitching raw thermal images captured by flying drone over solar plant. It contains temperature data. As shown below in Fig 1.1 it is a sliced normalized grey scale radiometric image. The name of this input image should be in the instructed format inside the directory that is ``plantname_mission_radiometric.tif``.

<img src="https://user-images.githubusercontent.com/75617171/127639002-306c5ce7-73e4-4520-85f6-d2f2fa9e969b.png" width="300" height="300"><br/>
> Fig 1.1 Normalized sliced grey scale Radiometric Image

## Inferno Orthomosaic
Inferno orthomosaic image is created by radiometric image by pouring Inferno color scheme in it as shown in Fig 1.2. The name of this input image should be in the instructed format inside the directory that is ``plantname_mission_inferno.tif``.

<img src="https://user-images.githubusercontent.com/75617171/127693062-76afd971-7d54-4d47-950c-ccc6efed7446.png" width="300" height="300"><br/>
> Fig 1.2 Inferno orthomosaic image

## RGB Orthomosaic
RGB orthomosaic image is a normal image which contains three channels of color red, blue and green made by stitching raw images captured by flying drone over solar plant as shown in Fig 1.3. The name of this input image should be in the instructed format inside the directory that is ``plantname_mission_rgb.tif``.

<img src="https://user-images.githubusercontent.com/75617171/127713046-cc48e1b3-e7a9-4ff2-98a8-b662ced9ecb6.png" width="300" height="300"><br/>
> Fig 1.3 RGB orthomosaic image

## Panel orientation
The orientation of the panel with respect to tables is either portrait or landscape. The input contains two values either portrait or landscape. If the height of table is less than width then it is landscape and if height of the table is greater than it’s width then it is portrait. The input is to be given as ``panel_orientation = PORTRAIT or  LANDSCAPE``. 
NOTE: The input name of portrait of landscape needs to be in all caps format.
As shown below in Fig 1.4 (a) Is a Portrait Panel with rotation and without rotation respectively. Fig 1.4 (b) Is a Landscape Panel with rotation and without rotation respectively.

<img src="https://user-images.githubusercontent.com/75617171/127713133-1e8f28a8-97a1-4f0c-a37b-7ac51685b640.png" width="600" height="200"><br/>
> Fig 1.4 (a) Portrait Panel with rotation and without rotation

<img src="https://user-images.githubusercontent.com/75617171/127713549-62c917a0-5993-4f2e-a116-7dd533b5bf18.jpg" width="600" height="200"><br/>
> Fig 1.4 (b) Landscape Panel with rotation and without rotation

## No of columns in a table
There are many tables in solar plant. So number of columns of a single table is to be provided. As shown in Fig 1.5 No of columns in this image is 10. The input is to be given as ``no_of_columns_per_table = 10``.

## No of rows in a table
Number of rows of a single table is to be provided. As shown in Fig 1.5 No of rows in this image is 4. The input is to be given as ``no_of_rows_per_table = 4``.

<img src="https://user-images.githubusercontent.com/75617171/127714053-2cec3b3c-3ea7-4cb7-92a4-ba348a400a46.png" width="300" height="400"><br/>
> Fig 1.5 No of columns and rows per table

# Preprocessing

## Normalization of radiometric orthomosaic 
Scaling the radiometric image in grey color with the minimum and maximum values as the temperature difference in the solar plant. If in a plant if a temperature is of 30-50 degree then the entire image will be scaled for 20 degree of temperature difference. 30 degree will be considered as 0 pixel values and 50 degree will be considered as 255 pixel values.
Normalization allows better correspondence between temperature values irrespective of location. (Higher temp locations may have temperatures from 35-45 degrees, lower temp locations may vary from 20-30 degrees) so by this process the data is preprocessed to be independent and identically distributed. The data contains many outliers, which tend to diminish the quality of the grayscale, so the outliers are clipped. For normalization of radiometric orthomosaic the input required is the Radiometric geotiff image path.
Assumptions: Minimum temperature of any plant is considered to be above -50 degrees Celsius.<br/>
The GitHub link to source code: <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/AB%23242-code-integration-and-traverse-through-directory/solarai/pre_processor/Radiometric2Gray.py" target="_blank">GitHub Radiometric2Gray.py file </a>

<img src="https://user-images.githubusercontent.com/75617171/127715587-176c22d2-b67a-4074-9bfa-2b560b7ca631.png" width="300" height="300"><br/>
> Fig 1.6 Normalized sliced greyscale Radiometric Image

## Rotation 
The orthomosaics (Inferno and Radiometric) images are rotated so that the panels are either Portrait or Landscape as shown in Fig 1.7. This is done by finding the angle of rotation. This is a mandatory requirement as the annotation boxes are always horizontal rectangles of the entire panel/table.

<img src="https://user-images.githubusercontent.com/75617171/127716262-8042368c-73cc-4342-9398-b5967f42d154.png" width="400" height="400"><br/>
> Fig 1.7 Above is a original inferno image, below is a rotated inferno image

It searches an image file with extensions (jpg, png, jpeg, PNG, bmp, BMP, tif, Tif) so, all the images should be in the given format. <br/>
The GitHub link to source code: <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/main/solarai/pre_processor/rotate_orthomosaic.py" target="_blank">GitHub Rotate_orthomosaic.py file </a>

## Slicing
The orthomosaic (Inferno and normalized Radiometric) images are very big in dimensions, such size cannot be directly consumed by any object detection algorithm. So, images are sliced into small size (as shown in Fig 1.8) so that image can be consumed by the algorithm. Each slice typically covers 3-4 tables.

<img src="https://user-images.githubusercontent.com/75617171/127717325-05eee3d2-c167-4508-9e26-06c398f307d7.png" width="600" height="200"><br/>
> Fig 1.8 Both are orthomosaic images sliced with each image covering 3-4 tables

The slicing requires input_dir, output_dir, panel_orientation, tables, tables_per_slice, no_of_panels_in_table_from_plant as inputs and returns sliced images in a created directory.<br/>
The GitHub link to the source code: <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/main/solarai/pre_processor/slice_orthomosaic.py" target="_blank">GitHub Slice_orthomosaic.py file </a>

## Annotation (required for training and QA)
All the defects are annotated through an annotation tool <a href="https://github.com/openvinotoolkit/cvat" target="_blank">CVAT</a>. It is an OpenCV project to provide easy labeling for computer vision datasets. CVAT allows to utilize an easy to use interface to make annotations efficiently. This tool generates an xml file for each image. As shown in Fig 1.9 each type of defect is annotated with table annotation.<br/>
The GitHub link to the source code: <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/main/solarai/pre_processor/create_bbox_rotate_xml.py" target="_blank"> Annotation GitHub file </a>

<img src="https://user-images.githubusercontent.com/75617171/127717872-e9125712-1ee2-4e8d-9318-b7d565335c7d.png" width="800" height="400"><br/>
> Fig 1.9 Each defect is annotated with table annotation

## Defect Annotation 
Defect annotation is making rectangular boundary around the defected panel. Fig 1.10 is a sliced inferno image which shows different types of defect annotated. Annotation creates an XML file with the Xmin, Ymin, Xmax, Ymax value of the solar panel and the defect type.

<img src="https://user-images.githubusercontent.com/75617171/127718003-73d03f62-1e4f-4238-a9b0-a5870710268c.png" width="800" height="400"><br/>
> Fig 1.10 Annotation on Defect using CVAT

## Table Annotation 
The tables in every sliced image is annotated. In annotation rectangular shape boundary is made around the tables. Fig 1.11 shows the Annotation of table.

<img src="https://user-images.githubusercontent.com/75617171/127718005-c1743f06-9387-4d3c-a384-fd0bfdb63d8c.png" width="800" height="400"><br/>
> Fig 1.11 Annotation on Defect using CVAT

There are 7 type of defects in solar panels. These defects are annotated and used to train the model about defects. 
The types of defects are:

#### BYPASS DIODE ACTIVE: 
Part of the module surface is homogeneously heated up and heat dissipation by the bypass diode, which is operating, is visible. Temperature difference of the glass on top of the junction box containing the operating bypass diode differs with construction (or failure of a bypass diode). Loss of contact at a cell connection might lead to a serial arc visible.

<img src="https://user-images.githubusercontent.com/75617171/127718492-63627240-e47f-4593-baf9-4e53c01b8a4a.png" width="200" height="70"><br/>
> Fig 1.12 Bypass Diode Active defect image

#### MODULE SHORT CIRCUIT: 
At one or more substrings, easily mistaken for cell breakage or cell defects, Potential induced degradation (PID) or mismatch (or failure of a bypass diode).

<img src="https://user-images.githubusercontent.com/75617171/127718476-2281271c-f569-4907-98d4-96790fdddfea.png" width="200" height="70"><br/>
> Fig 1.13 Module Short Circuit defect image

#### MODULE HOT: 
The module surface is homogeneously heated than other panels.

<img src="https://user-images.githubusercontent.com/75617171/127718473-10bb899d-d7d6-455b-8cdd-ad31844c5a94.png" width="200" height="70"><br/>
> Fig 1.14 Module Hot defect image

#### STRING HOT: 
The module surface is homogeneously heated. Temperature difference of the junction box is similar to operational state.

<img src="https://user-images.githubusercontent.com/75617171/127718471-4efc5f74-30e5-4d66-80fb-03b5dd04b27f.png" width="200" height="90"><br/>
> Fig 1.15 String Hot defect image

#### HOTSPOT (Cell Failure, Cell Chipping and Delamination): 
Difference in temperature increases with load, cell efficiency and number of cells in a substring.

<img src="https://user-images.githubusercontent.com/75617171/127718468-a84bd6a6-10af-4b9b-adcd-3d23875e3b1e.png" width="200" height="70"><br/>
> Fig 1.16 Hotspot defect image

#### DIRT/SHADOW: 
Assessable by thermal pattern, visual Image and classified typically as an extended area abnormality. Compared with RGB Tiff.

<img src="https://user-images.githubusercontent.com/75617171/127718467-d5b9dabc-536d-46c5-863a-84bd7d6178e9.png" width="200" height="50"><br/>
> Fig 1.17 Dirt/Shadow defect image

#### VEGETATION : 
The nearby area plants may grow tall and they may lay on top of panels.  It covers the solar panels which usually occurs at the edge of the panel, and is sometimes seen as a tiny bright dot or shadowed.

<img src="https://user-images.githubusercontent.com/75617171/127718466-6a3c43c9-3011-4ef3-b4d8-c0ec6802f560.png" width="200" height="70"><br/>
>Fig 1.18 Vegetation defect image

## Image enhancement 
Various image processing techniques are used to enhance the image quality.
These techniques vary from defect to defect. Examples of these techniques are CLAHE, Unsharp Masking etc., as shown in Fig 1.19.
Some techniques used are:

#### Image Normalization 
The images are normalized so that there is minimum variation in the image quality of various orthomosaics as it helps in improving the accuracy of model.

#### Contrast Limited Adaptive Histogram Equalization (CLAHE)
CLAHE is a variant of Adaptive histogram equalization (AHE) which takes care of over-amplification of the contrast. CLAHE operates on small regions in the image, called tiles, rather than the entire image. The neighboring tiles are then combined using bilinear interpolation to remove the artificial boundaries. This algorithm can be applied to improve the contrast of images.

#### Unsharp Masking 
In Unsharp masking technique an image is sharpened by subtracting a blurred (Unsharp) version of the image from itself. An Unsharp filter is an operator used to sharpen an image.
Sharpening can help you emphasize texture and detail, and is critical when post-processing most digital images. It detects edges and create a mask by subtracting the blurred image copy from it’s original image and then uses the mask to increase contrast at the edges by adding higher contrast copy, Unsharp mask and original image which results into sharpened final image.

<img src="https://user-images.githubusercontent.com/75617171/127718673-cd92ba14-999e-434c-a70c-986f058c0fba.png" width="800" height="400"><br/>
> Fig 1.19 Image processing done using CLAHE, Unsharp Masking technique.

## Augmentation
Image augmentation is a technique of altering the existing data to create some more data for the model training process. Some of the image augmentation technique used are horizontal flip, vertical flip and diagonal flip.

<img src="https://user-images.githubusercontent.com/75617171/127719096-e009908f-918d-44a6-9534-386938b173aa.png" width="800" height="400"><br/>
> Fig 1.20 Image augmentation.

# Model Training 
Model is trained for tables and each defect separately. After annotations all the rotated, sliced images and xml files with their respective name as image name is stored in a folder which is then used to train the model.

<img src="https://user-images.githubusercontent.com/75617171/127719170-0bafd95f-c0c7-41c6-925d-64a40d93813d.jpeg" width="800" height="800"><br/>
>Fig 1.21 Model Training Flow Chart

## Training the model for detection of defects
For each defect the model was trained separately. The dataset annotated in previous steps is split into train and test data such that 80% data is used for training and with 20% data testing is done where, an entire orthomosaic file is used for training and entire another orthomosaic file is used for testing. If 5 orthomosaic files are shared then 3 of them are used for training and 2 for testing. <a href="https://github.com/AlexeyAB/darknet" target="_blank">Darknet</a> is used for training and testing which is a framework for object detection classifier. Then anchor boxes for each of the defect is calculated. Refer the <a href="https://github.com/AlexeyAB/darknet#how-to-use-on-the-command-line" target="_blank">GitHub link</a> to calculate anchor box which is instructed in readme file to calculate anchors:<br/>
```darknet.exe detector calc_anchors data/obj.data -num_of_clusters 12 -width 640 -height 640 ```<br/>
After calculating anchor boxes image processing techniques mention in the preprocessing (Augmentation) are applied. The model is then trained to obtain mish YOLOv4 P4 (640 X 640).

## Validating the model for defects detection
For each training the train weights at every 1000 iterations is saved. Once all the weights are received mAP is calculated of each weight on test data. <a href="https://github.com/AlexeyAB/darknet" target="_blank">GitHub Link</a> for reference of mAP calculation Then graph of mAP is drawn to find the perfect fit and optimum weight.  Roc curve is calculated to find the optimum threshold.  

## Training the model for detection of tables
Same steps (5(a)) are followed for model training of detection of tables as for model training of detection of defects.

## Validating the model for table detection
Same steps (5(b)) are followed for validation of tables detection as for validation of defects detection.

## Masking and Contouring
The tables are present in many different size in a plant which is mapped using contouring. Image is converted into a black and white image which gives the total length of all table in a line which is mapped using the width of a single panel x number of columns which gives length of one table. All the tables are then traced and boundary of rectangular boxes is drawn around each table which is then sorted and mapped with the defects.<br/>

## Mapping
Mapping of every defect with the corresponding tables. This is done by calculating minimum distance between the centroid (latitude and longitude) of table and defect.<br/>
The GitHub link to the source code: <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/AB%23201-Post-processing-to-correct-the-table-bounding-boxes/solarai/pre_processor/mapping_defect_table.py" target="_blank">Mapping_defect_table.py</a>

# Deployment of the model 
For first time activity the deployment process given in Fig 1.22 is followed for each plant.

<img src="https://user-images.githubusercontent.com/75617171/127719173-042a9d79-4bd6-48a2-824a-332ad4bd195f.png" width="800" height="200"><br/>
> Fig 1.22 First time activity for each plant

Deployment process for defect detection for each plant is given in Fig 1.23.

<img src="https://user-images.githubusercontent.com/75617171/127719177-50af8798-5747-402d-9d47-895b48762aec.png" width="800" height="300"><br/>
>Fig 1.23 Defect identification for each plant

Deployment process for defect detection for each plant is given in Fig 1.23.

## Creating SO files 
To create SO files follow the instructions given in <a href="https://github.com/AlexeyAB/darknet#how-to-use-yolo-as-dll-and-so-libraries" target="_blank">How to use Yolo as DLL and SO Libraries</a>.
```
-On Linux
-using build.sh or
-build darknet using cmake or
-set LIBSO=1 in the Makefile and do make
```
```
-On Windows
-using build.ps1 or
-build darknet using cmake or
-compile build\darknet\yolo_cpp_dll.sln solution or  build\darknet\yolo_cpp_dll_no_gpu.sln  solution.
```

## Directory structure
<img src="https://user-images.githubusercontent.com/75617171/130359861-38326d01-e163-442c-8387-4e75f4f8add5.PNG" width="634" height="580"><br/>
>Fig 1.23 Directory structure model 

## Input Path
The input files or images can be saved in a directory created here or there and can be fixed for later use. The path of directory is sent as input for image path or file location when calling the api for defect detection, table detection and defect correction respectively. API is a software intermediary that allows two applications to talk to each other. The “api/” in the URL directs the api to api services app via main app solarai <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/main/backend/solarai/solarai/urls.py" target="_blank">urls.py</a> which further instructs the api for all calls made with “api/” to be directed to the <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/main/backend/solarai/api_services/urls.py" target="_blank">urls.py </a>of api services app. The api services app has path set for all the API’s to it’s function stored in <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/main/backend/solarai/api_services/views.py" target="_blank">views.py</a> file.

## API’s
An API (Application Programming Interface) is a set of functions that allows applications to access data and interact with external software components, operating systems, or microservices. An API delivers a user response to a system and sends the system's response back to a user. Postman is an API development tool which helps to build, test and modify APIs. It has the ability to make various types of HTTP requests(GET, POST, PUT, PATCH), saving environments for later use, converting the API to code for various languages(like JavaScript, Python). The four primary tabs are the stepwise process for developing a complete API : 
- *Define*: The first tab creates the schema of the API. In addition to that, the user can also import schema from somewhere else to work on it by themselves.
- *Develop*: Through this tab, a user can add the endpoint to the mock server, create and publish documentation for the API. Additionally, the user can also play with the environment to switch between different setups without changing the request.
-  *Test*: The test tab provides a variety of testing options for the tester to develop a spotless and great API. It includes creating a new test suite to test the API behavior, performing integration testing on the API to check if the API works well with the third-party applications or not. Additionally, contract tests to make sure that the API works according to its specifications.
-  *Observe*: Monitor the performance of the API under various conditions and intervals with the help of Postman Monitors.
The response body is requested in JSON format.<br/>
The four API’s developed using postman are : 

#### Defect Detection 
This API is used to detect the defects from Orthomosaic images.
After the main call is directed by the solarai app to api services app, the URL “v1/defect_detection“ calls the function views.detection , which requests for parameters.
where parameters required are:<br/>
- In a directory three image files should be there with the given naming convention (Ref. 3(a), 3(b) and 3(c)) and path of the directory should be passed in image_path. Example  image_path=/home/ubuntu/solaraidata/
- panel_orientaion = PORTRAIT / LANDSCAPE (Ref. 3(d))
- no_of_columns = 10 (Ref. Fig 1.5)
- no_of_rows = 4 (Ref. Fig 1.5)

All the inputs are send to a function imported from <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/main/backend/solarai/api_services/tasks.py" target="_blank">tasks.py</a> named “detection” which performs operations of rotation and slice on the orthomosaic where the rotation and slice functions are imported from rotate orthomosaic.py and slice orthomosaic.py respectively, then loads the appropriate weights of model on the type of image given, then writes the output in the output directory (where can one find the output directory) and returns “Task completed” on the completion of task to the detection function of views.py which then writes the status of the task and updates it in the <a href="https://github.com/baggageai/dml-prescinto-solar-ai/blob/main/backend/solarai/solarai/celery.py" target="_blank">celery</a> (imported as app) with the respected task id. Celery is an asynchronous task queue which stores the tasks id and the status of task. Then detection function of views.py return a JSON response.<br/>
The JSON response received after sending the POST request contains: <br/>
   1. An “id” will be provided which will be used for further API calls.<br/>
   2. ”get_detection” shows the status of the POST call made. The response can be : <br/>
      - Success: Conveys that data has been received by get_detection API .<br/>
      - Pending: Conveys that the process is in progress.<br/>
      - Failure: Conveys that the request got failed due to some error.<br/>
   3. ”db_upload_status” response can be either True or False.<br/>
      - True: The upload in database was successful.<br/>
      - False: The upload in database was not successful.

#### Get Result
This API gives the status of the image detection.<br/>
After the main call is directed by the solarai app to api services app, the URL “v1/get_result“ calls the function views.get_result , which requests for parameters. 
where “id” received when image was sent through POST/api/v1/defect_detection is required as a parameter for this API call. 
The get result function requests celery for state of a task by passing task id as parameter to AsyncResult method and compares the state with the given three states: success, pending, failure and sends a JSON response respectively.<br/>
The JSON response will be of 3 types: <br/>
                                1. Success: If state of task is successful it returns a JSON response of job status as “SUCCESS“ and the output location of outputs.<br/>
                                2. Pending: When the state of task is unknown or in progress it sends a JSON response of job status as “PENDING”.<br/>
                                3. Failure: If the status of task is failed or none of the above it sends a JSON response of job status as “FAILURE”.<br/>

#### Defect Mapping
This API will give the output with Orthomosaic ID or Orthomosaic Name, Table ID, Defects Details and Lat-Long Coordinates.<br/>
After the main call is directed by the solarai app to api services app, the URL “v1/defect_mapping“ calls the function views.defect_mapping, which requests for parameters. <br/>
It requires file location of corrected XML files in file_location and it reads the data and loads it in json format and calls tasks.defect_mapping which further sorts the table and calls mapping_defect_table from the preprocessing.py file which then maps the defects found in plant with tables. It send a response in which it syncs the task state with success pending or failure. The output path is set in which the file response will be received of the mapped defect if task was successful.


#### Table Detection 
This API is used for Table Detection.<br/>
After the main call is directed by the solarai app to api services app, the URL “v1/table_detection“ calls the function views.table_detection, which requests for parameters.
where parameters required are:<br/>
- In a directory three image files should be there with the given naming convention (Ref. 3(a), 3(b) and 3(c)) and path of the directory should be passed in image_path. Example : image_path=/home/ubuntu/solaraidata/<br/>
- panel_orientaion = PORTRAIT / LANDSCAPE (Ref. 3(d))<br/>
- no_of_columns = 10 (Ref. Fig 1.5)<br/>
- no_of_rows = 4 (Ref. Fig 1.5)

All the inputs are send to a function imported from tasks.py named “detection” which performs operations of rotation and slice on the orthomosaic where the rotation and slice functions are imported from rotate orthomosaic.py and slice orthomosaic.py respectively, then loads the appropriate weights of model on the type of image given, then writes the output in the output directory (where can one find the output directory) and returns “Task completed” on the completion of task to the detection function of views.py which then writes the status of the task and updates it in the celery (imported as app) with the respected task id. Celery is an asynchronous task queue which stores the tasks id and the status of task. Then detection function of views.py return a JSON response.<br/>
The JSON response received after sending the POST request contains:
   1. An “id” will be provided which will be used for further API calls.<br/>
   2. ”table_detection” shows the status of the POST call made. The response can be : <br/>
      - Success: Conveys that data has been received by get_detection API .<br/>
      - Pending: Conveys that the process is in progress.<br/>
      - Failure: Conveys that the request got failed due to some error.<br/>
   3. ”db_upload_status” response can be either True or False.<br/>
      - True: The upload in database was successful.<br/>
      - False: The upload in database was not successful.

## Database
Databases store very large numbers of records efficiently. All the data and records are stored in SQL server hosted on azure. SQL is a language used to fetch, add or manipulate data in databases. There are three database tables used: plant_master, table_master and defects each used to store their respective generated data.

- Table 1: plant_master <br/>
  - plant_id varchar(30)<br/>
  - ortho_name varchar(30)<br/>
  - inspection_id varchar(30) DEFAULT NULL<br/>
  - ortho_size float DEFAULT NULL<br/>
  - rotation_info nvarchar(max) DEFAULT NULL<br/>
  - pannel_oriantation varchar(20)<br/>
  - no_of_rows int<br/>
  - no_of_columns int<br/>
  - date_time datetime DEFAULT(getdate())<br/>
                          
- Table 2: table_master<br/>
  - plant_id varchar(30)<br/>
  - ortho_id varchar(30)<br/>
  - inspection_id varchar(30) DEFAULT NULL<br/>
  - plant_name varchar(30)<br/>
  - table_id varchar(30)<br/>
  - rotated_table_cor nvarchar(max)<br/>
  - table_cor nvarchar(max)<br/>
  - table_lat_gps float<br/>
  - table_long_gps float<br/>
  - rotated_panel_cor nvarchar(max)<br/>
  - panel_cor nvarchar(max)<br/>
  - panel_centroid_cor nvarchar(max)<br/>
  - panel_id varchar(20)<br/>
  - panel_lat_gps float<br/>
  - panel_long_gps float<br/>
  - string_id varchar(30) DEFAULT NULL<br/>
  - string_cor nvarchar(max) DEFAULT NULL<br/>
  - date_time datetime DEFAULT(getdate())<br/>    
                  
- Table 3: defects<br/>
  - ortho_id varchar(30) <br/>
  - defects_name varchar(30) <br/>
  - inspection_id varchar(30) DEFAULT NULL <br/>
  - rotated_ortho_cor nvarchar(max) <br/>
  - ortho_cor nvarchar(max) <br/>
  - lat float <br/>
  - long float <br/>
  - panel_id varchar(30) <br/>
  - plant_name varchar(30) <br/>
  - table_id varchar(30) <br/>
  - min_temp float <br/>
  - max_temp float <br/>
  - median_temp float <br/>
  - date_time datetime DEFAULT(getdate()) <br/>              
