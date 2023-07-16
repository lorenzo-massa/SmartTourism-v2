<a name="readme-top"></a>

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/lorenzo-massa/SmartTourism">
    <img src="images/logo_1.png" alt="Logo" width="400" height="80">
  </a>

  <p align="center">
    Image Recognition for Android Devices
    <br />
    <a href="https://github.com/lorenzo-massa/SmartTourism"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/lorenzo-massa/SmartTourism/issues">Report Bug</a>
    ·
    <a href="https://github.com/lorenzo-massa/SmartTourism/issues">Request Feature</a>
  </p>
</div>

## Getting Started
<div align="center">
  <img src="images/screen1.png" alt="Logo" width="200" height="380"> &nbsp; &nbsp; &nbsp; <img src="images/screen3.png" alt="Logo" width="200" height="380">
</div>

### APK
You will find the APK in `app/build/outputs/apk/support/debug` . 

If you want to generate another APK file, please refer to the following guide: [How to Generate APK and Signed APK Files in Android Studio](https://code.tutsplus.com/tutorials/how-to-generate-apk-and-signed-apk-files-in-android-studio--cms-37927)

### Guide
The repository consists of two parts:
* Python
* Android

The python part is used to generate sqlite files from an image dataset. You should use and edit this part only if you want to use another neural network or image dataset.
The android application is ready to use and you should change it just to add files to the monuments guides.

You will find all the instuction you need just below.

### Prerequisites

Python library required:
* imutils
* tensorflow
* opencv
* pandas
* gensim
* progressbar
* faiss-cpu (Anaconda required)
* scikit-learn

## Monument guides creation 
Go to the `models\src\main\assets\guides` folder. Inside it there is the folder `Template Monument` which is to be used as a template, so without altering its structure. It is only possible to change the name of the folder with the name of the monument which, however, must be the same uilized in the dataset folder.</br>
Text, images, audios and videos can be shown using Mardkown.

### Markdown
Each guide has to be given as a markdown file. The first 4 rows of each `.md` have to be completed as the following:
 ```html
<!-- Use the following commented lines to include monument coordinates, categories and attributes (leave empty lines if the monument has no additional info)
Coordinate of the monument spaced by a blank space
Categories of the monument spaced by commas
Attributes of the monument spaced by commas
 -->
```

IMPORTANT: Files must have the same name as the files in the template folder.

NOTE: For the time being, Italian and English languages are supported.

### Categories
Go to the `models\src\main\assets\categories` folder and move inside it one image for each category present in at least one monument guide.

IMPORTANT: Images must have the same name as the categories.

## Database creation
Complete the previous step before creating the database.

The repository contains the file `Python/build_sqlite.py` which must be executed by adding the argument `-i` or `--images` indicating the path to the dataset folder as in the following example:

```sh
python build_sqlite.py -i datasetFolder
```

IMPORTANT: The indicated folder must contains one folder per monument and each of which contains the images, as in the following example:

```

datasetFolder
├───Battistero SanGiovanni
│       img1.jpg
│       img2.jpg
│       img3.jpg
│
├───Campanile Giotto
│       img1.jpg
│       img2.jpg
│       img3.jpg
│
├───Cattedrale Duomo
│       img1.jpg
│       img2.jpg
│       img3.jpg
│
└───Palazzo Vecchio
       img1.jpg
       img2.jpg
       img3.jpg

```

The file `build_sqlite.py` will create three `.sqlite` and `.pck` files. Each pair of files is created with a different neural network. To change the neural network see dedicated paagraph.

IMORTANT: Do not change the names of the files created.


<!--

### ONLY FOR EXPERT USERS

## Creating the database with different neural networks
Place the neural network model `.tflite` in `Models/src/main/assets`.
Open the file `build_sqlite.py`: add the name of the neural network and the model path to the "types" list. Example:

```python
types = [ #neural networks
    ('MobileNetV3_Large_100', '../models/src/main/assets/lite-model_imagenet_mobilenet_v3_large_100_224_classification_5_default_1.tflite'), 
    ('MobileNetV3_Large_075', '../models/src/main/assets/lite-model_imagenet_mobilenet_v3_large_075_224_classification_5_default_1.tflite'),
    ('MobileNetV3_Small_100', '../models/src/main/assets/lite-model_imagenet_mobilenet_v3_small_100_224_classification_5_default_1.tflite'),
    ('newNeuralNetworkModel.tflite', 'pathNewNeauralNetworkModel.tflite')
]
```

Run 'build_sqlite.py'.

## Using a database created with a different neural network
Go to `lib_support\src\main\java\org\tensorflow\lite\examples\classification\tflite`.
1) Create a class that extends the `Classifier` class with a name that indicates the new neural network.
TIP: There is a template class named `ClassifierNewNeuralNetworkClass`. Rename the class and change the `getModelPath()` method with the filename of the new neural network.
2) Modify the file `Classifier.java` by adding a name indicating the new model as in the example:

```java
/** The model type used for classification. */
  public enum Model {
    MOBILENET_V3_LARGE_100,
    MOBILENET_V3_LARGE_075,
    MOBILENET_V3_SMALL_100,
    NEWMODEL_NAME
  }
```
3) Modify the `create(Activity, Model, Device, int)` method by adding an else if with the previously created class and model:

```java
if (model == Model.MOBILENET_V3_LARGE_100) {
      return new ClassifierMobileNetLarge100(activity, device, numThreads);
    } else if (model == Model.MOBILENET_V3_LARGE_075) {
      return new ClassifierMobileNetLarge075(activity, device, numThreads);
    } else if (model == Model.MOBILENET_V3_SMALL_100) {
      return new ClassifierMobileNetSmall100(activity, device, numThreads);
    }else if (model == Model.NEWMODEL_NAME) {
      return new ClassifierNewClassName(activity, device, numThreads);
    } else {
      throw new UnsupportedOperationException();
    }
 ```
 
4) Modify `Retrievor.java` by adding an if in the `Retrievor(Context, Cassifier.model)` constructor method as in the example:

```java
if (model == Classifier.Model.MOBILENET_V3_LARGE_100) {
            dbName = "MobileNetV3_Large_100_db.sqlite"
        } else if (model == Classifier.Model.MOBILENET_V3_LARGE_075) {
            dbName = "MobileNetV3_Large_075_db.sqlite"
        } else if (model == Classifier.Model.MOBILENET_V3_SMALL_100) {
            dbName = "MobileNetV3_Small_100_db.sqlite"
        }else if (model == Classifier.Model.NEWMODEL_NAME) {
            dbName = "newDatabaseFile_db.sqlite"
        } else {
            throw new UnsupportedOperationException();
        }
```

`newDatabaseFile.sqlite` is the new sqlite file that is the created in the previous paragraph.

5) Finally edit the file `app/src/main/res/values/strings.xml` by inserting the name of the new model in `string-array name="tfe_ic_models"` as in the example:

```xml
    <string-array name="tfe_ic_models" translatable="false">
        <item>MobileNet_V3_Large_100</item>
        <item>MobileNet_V3_Large_075</item>
        <item>MobileNet_V3_Small_100</item>
        <item>NewModel_Name</item>
    </string-array>
```

IMORTANT: The model name must be the same as entered in step 2) of this paragraph, capitalization not required.
Once you have completed these steps you can use the new model by selecting it directly in the application menu.

-->
<!-- CONTACT -->
## Contact

Lorenzo Massa - lorenzo.massa@stud.unifi.it

Project Link: [https://github.com/lorenzo-massa/SmartTourism](https://github.com/lorenzo-massa/SmartTourism)

<p align="right">(<a href="#readme-top">back to top</a>)</p>




