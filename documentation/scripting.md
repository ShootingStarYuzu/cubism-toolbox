# Python Scripting

> This documentation is pretty bad and I know it too. >_>
> Help make it better by giving feedback via **Issues** or directly contribute by creating a **Pull Request**. ^^

The Python extension adds a new menu to Cubism, which enables executing Python scripts from inside Cubism that interact with the program.
The scripts are read from the `scripts` directory under the Cubism data folder.
You can open this folder by selecting the menu option `Open Scripts folder`.
The menu option `Reload Scripts` has to be used when new scripts have been added or the name of the script has been changed to re-build the menu.
It is not required to be used when changing the script's code as it only re-indexes the scripts's metadata and file names, but not the content.

## Implementation Status

The following things have been implemented but are barely tested:

- Cubism Model files
  - Art Meshes
  - Parts (Folders)
  - Glue
  - Warp Deformers
  - Rotation Deformers

The following things have not been implemented yet:

- Cubism Animation files

## How to access data?

### The Project

All currently opened model and animation files (called documents) are summarized in a project.
The current project can be accessed via the **project** variable and all documents are summarized in the **documents** member variable.
To only list model or animation documents the list can be filtered via the **onlyDocuments()** or **onlyAnimations()** method.

```python
#!/bin/env python2.7

# Enumerating documents (shorthand for project.documents)
for document in project:
    print(document)

# Accessing the document list directly
for document in project.documents:
    if isinstance(document, ModelDocument):
        print('Model: %s' % document)
    elif isinstance(document, AnimationDocument):
        print('Animation: %s' % document)

# Using filters to only get a specific kind of documents
for document in project.documents.everything:
    print(document)
for document in project.documents.onlyModels:
    print(document)
for document in project.documents.onlyAnimations:
    print(document)

# The document that is currently shown in Cubism
print(project.documents.current)

# Creating documents (completely untested)
myModel = project.createModel('My Model')
myAnimation = project.createAnimation('My Animation')

project += myModel
project += myAnimation
project.documents += myModel
project.documents += myAnimation
```

class `Project`:

- property `name: str`: The name of the project
- property `documents: DocumentList`: All documents in the project
- method `createModel(name): ModelDocument`: Creates a new model
- method `createAnimation(name): AnimationDocument`: Creates a new animation

class `DocumentList`:

- property `everything: DocumentList`: Removes all filters from list
- property `onlyModels: DocumentList`: Filters the list by models
- property `onlyAnimations: DocumentList`: Filters the list by animations
- operator `[]` (getitem) with `int`: Returns the n-th document
- operator `[]` (getitem) with `guid`: Returns the document with GUID
- operator `+=` (iadd): Adds the document to the project
- operator `-=` (isub): Removes the document from the project
- operator `in` (contains): Returns `True` if the document is contained in the project
---

### The Model Document

Model documents are used for rigging and contain the following elements:

- Linked PSD Sources (See Project tab)
- Model Images (See Project tab)
- Parameters (See Parameter tab)
- Physics (See Modeling -> Open Physics)
- Model Elements (See Part tab)

class `ModelDocument`:

- property `name: str`: The name of the model
- property `filename: str`: The file path of the model
- property `size: (int, int)`: The model dimensions in as with x height tuple
- property `imageSources: ImageSourceList`: All linked PSD image sources
- property `modelImageGroups: ModelImageGroupList`: All model image groups
- property `parameters: ParameterList`: All parameters
- property `physics: PhysicsList`: All physics
- property `elements: ElementList`: All model elements
- method `createImageSource(name: str, path: str)`: Creates new PSD source file from path
- method `createModelImageGroup(name: str)`: Creates new model image group
- method `createModelImage(name: str, layer: Layer)`: Creates new model image for the provided PSD layer
- method `createParameter(name: str, id: str)`: Creates a new parameter
- method `createParameterGroup(name: str, id: str)`: Creates a new parameter group
- method `createPhysics(name: str, id: str)`: Creates a new physics setting
- method `createPart(name: str, id: str)`: Creates a new part (folder) element
- method `createArtMesh(name: str, id: str, image: ModelImage)`: Creates a new art mesh element
- method `createWarpDeformer(name: str, id: str, x: float, y: float, width: float, height: float, columns: int, rows: int)`: Creates a new warp deformer at the specified location with the specified dimensions and number of subdivisions
- method `createRotationDeformer(name: str, id: str, x: float, y: float, angle: float, scale: float)`: Creates a new rotation deformer at the specified location with the provided angle and scale

---

The linked PSD sources from the Project tab can be accessed via the model document's **imageSources** member variable.
Each image source represents a single PSD file which can hold nested folders (image layer groups) and image layers which can be accessed via the **layers** member variable.

![Project Tab](./images/project-tab.png)

For the ease of accessing elements in the model file-like paths can be used.
The use of them will be demonstrated further below.
The following paths exists to access image sources and their layer information:

- ImageSourceName: A linked PSD image file
- LayerName: A layer (no groups) in a linked PSD image file
- LayerChildName: A layer or group in a linked PSD image file
- LayerGroupname: A layer group in a linked PSD image file

```python
#!/bin/env python2.7

myModel = project.documents.onlyModels[0]

def printLayers(layers, prefix=''):
    for layer in layers:
        if isinstance(layer, ImageLayer):
            print('%sImage Layer: %s' % (prefix, layer.name))
        elif isinstance(layer, ImageLayerGroup):
            print('%sImage Layer Group: %s' % (prefix, layer.name))
            printLayers(layer.layers, prefix + '  ')

# Prints a list all PSD image sources and the contained layers
for imageSource in myModel.imageSources:
    print('Image Source: %s at "%s"' \
        % (imageSource.name, imageSource.filename))
    printLayers(imageSource.layers)

# Paths can be used to faster access specific layers of the PSD file
myPsdFile  = myModel[ImageSourceName / 'MyDrawing.psd']
leftEye  = myPsdFile[LayerChildName  / 'Head' / 'Eyes' / 'EyeL']
leftIris = myPsdFile[LayerName       / 'Head' / 'Eyes' / 'EyeL' / 'Iris']
rightEye = myPsdFile[LayerGroupName  / 'Head' / 'Eyes' / 'EyeR']
```

class `ImageSourceList`:

- TODO

class `ImageSource`:

- TODO

class `LayerList`:

- TODO

class `Layer`:

- TODO

class `LayerGroup`:

- TODO

---

The PSD image sources are referenced by model images.
Model images are used by art meshes and define the image / layer that is displayed.
A single model image can have many linked PSD layers.
However, only one can be active at the same time.
Model images are summarized in model groups that can be accessed via the **modelImageGroups** member variable of the document.

The following paths exist for accessing model images:

- ModelImageGroupName: A model image group
- ModelImageName: A model image

```python
#!/bin/env python2.7

myModel = project.documents.onlyModels[0]

# Print all model images
for imageGroup in document.modelImageGroups:
    print('Model Image Group: %s' % imageGroup.name)
    for modelImage in imageGroup.modelImages:
        print('  Model Image: %s' % modelImage.name)
        activeLayer = modelImage.activeLinkedLayer
        for imageLayer in modelImage.linkedLayers:
            active = ' (active)' if imageLayer == activeLayer else ''
            print('    Image Layer: %s at "%s"%s' \
                % (imageLayer.name, imageLayer.filename, active))

# Paths can be used to faster access specific model images
myGroup     = document[ModelImageGroupName / 'My Group']
headOutline = document[ModelImageName      / 'My Group' / 'HeadOutline']
eyeL        = document[ModelImageName      / 'My Group' / 'EyeL']
```

class `ModelImageGroupList`:

- TODO

class `ModelImageGroup`:

- TODO

class `ModelImageList`:

- TODO

class `ModelImage`:

- TODO

class `LinkedLayerList`:

- TODO

---

Parameters are show in Cubism in Parameter tab and can be accessed via the **parameters** member variable.

![Parameter Tab](./images/parameter-tab.png)

```python
#!/bin/env python2.7

myModel = project.documents.onlyModels[0]

def printParameter(param, prefix=''):
    if isinstance(param, Parameter):
        print('%sParameter: %s' % (prefix, param))
    elif isinstance(param, ParameterGroup):
        print('%sParameter Group: %s' % (prefix, param))
        for parameter in param:
            printParameter(parameter, prefix + '  ')

# Prints all parameters contained in this model
for parameter in myModel.parameters:
    printParameter(parameter)
```

The following paths exist for accessing model images:

- ModelImageGroupName: A model image group
- ModelImageName: A model image

class `ParameterList`:

- TODO

class `Parameter`:

- TODO

class `ParameterGroup`:

- TODO

---

Physics: TODO

---

Model Elements: TODO
