---
layout: post
title: Face detection and recognition in shotwell
---

After dabbling a bit with [OpenFace](https://cmusatyalab.github.io/openface), I wanted to add similar face detection and
recognition abilities to a typical Linux desktop photo app. So I discovered
[Shotwell](https://wiki.gnome.org/Apps/Shotwell), which is a photo manager for Gnome. Shotwell had a
[partial implementation](https://wiki.gnome.org/Apps/Shotwell/FacesTool) of face detection (no recognition) which was
under a build define and not enabled in the releases. With that code as the starting point, I started integrating the
ideas from OpenFace into Shotwell.

The WIP source code is [here](https://gitlab.gnome.org/nma83/shotwell/tree/wip/faces). Dependencies are:

* [OpenCV](https://opencv.org) >= 2.3.0
* [DNN models](https://cmusatyalab.github.io/openface/models-and-accuracies/) from OpenFace 

## Enable face-detect

Shotwell uses [meson](https://mesonbuild.com) for its build system. The face detection code is disabled by default in
the build configuration. Do the following in the meson build directory to enable it:

    meson configure -Dface-detection=true

## User interface

> The UI for face detection and recognition is still evolving and needs some refinement before it could be
> enabled in the build by default.

After building shotwell with the face-detection flag enabled, a new button, Faces, shows up in the photo UI.

![Button]({{ site.github.url }}/assets/shotwell-facedetect-button.png)

Faces can either be manually entered by drawing a rectangle on the photo, or auto-detected using the 'Detect Faces' button.
After running face detection on the photo, rectangles are drawn around the faces detected.
To tag a face, the name of the face is entered in place of the 'Unknown face' placeholder using the 'Edit' button. If
not changed, the face is not saved into the shotwell database. Once tagged, a new list called 'Faces' appears in the
sidebar. This list contains all the faces tagged in current photo database.

![Tag]({{ site.github.url }}/assets/shotwell-facedetect-face.png)

In order to use a face in a photo as a reference for automatic recognition in other photos, click on the face name in
the 'Faces' sidebar. Then select the photo containing the face and click the menu item in Faces -> Train Face ... From
Photo.

![Page]({{ site.github.url }}/assets/shotwell-facedetect-page.png)

There are more enhancements required in the interface for providing list of suggestions during recognition and batch
multiple photos for recognition.

## Architecture

The face detection and recognition has to happen in a separate process since OpenCV and shotwell can require different
versions of GTK in some installations. The earlier implementation used to spawn the face detection process (a.k.a
facedetect) as a child each time the 'Detect Faces' button on the GUI was clicked. This is not very efficient and does
not have a flexible API. So the first change was to move to [DBus](https://dbus.freedesktop.org) for talking to the
external process. The [DBus daemon](https://dbus.freedesktop.org/doc/dbus-daemon.1.html) is used to start the facedetect
process if it is not already running, using a DBus service file.

### facedetect API

1. `boolean LoadNet(string dir)`: 
   * Loads the OpenFace DNN models, `res10_300x300_ssd_iter_140000_fp16.caffemodel` for detection and
     `openface.nn4.small2.v1.t7` for recognition 
   * Returns true if load was successful
1. `FaceRect[] DetectFaces(string image, string cascade, float scale, boolean infer)`:
   * Loads `image` and runs it through DNN based face detection if DNN loaded, else runs it through
     [HAAR cascade](https://docs.opencv.org/3.4.1/d7/d8b/tutorial_py_face_detection.html) based detection using the
     `casacade` file name passed in
   * If `infer` is true, each face is converted to 128 element
     [embedding](http://www.cv-foundation.org/openaccess/content_cvpr_2015/app/1A_089.pdf)
   * Returns a list of `FaceRect`s that contain the bounding box and embedding vector per face

### Using the API

The face detection flow in shotwell calls `DetectFaces` for a photo with `infer` set to true. The facedetect process
returns a list of face bounding boxes and embedding vectors. The face table maintained in the shotwell database has an
additional column to store the 128 element embedding for each face (it had just the bounding box previously).

Face recognition is done by computing the [dot-product](https://en.wikipedia.org/wiki/Dot_product) of the embedding
vector of faces. Once a reference face is set using the menu item in the Faces page, shotwell uses the embedding vector
from that photo and computes the dot-product with faces detected in other photos (via the 'Detect Faces' button).

A threshold of 70% is used to determine if a face matches the reference face. Any such matches are automatically tagged
by shotwell before presenting the result dialog.

## Next steps

* Improve UI for face detection and recognition
* Sanity check faces for aspect ratio of bounding box and duplicate matches in a photo
* Batch processing of photos to label faces 
* [Align faces](https://docs.opencv.org/2.4/modules/contrib/doc/facerec/facerec_tutorial.html?highlight=eigenface#aligning-face-images)
  before generating embedding vector
* Implement other [clustering techniques](https://en.wikipedia.org/wiki/K-means_clustering) for face recognition
* Use the [OpenFace API](https://openface-api.readthedocs.io/en/latest/index.html), but it depends on [dlib](dlib.net)
  which is not available as a package on many distros

## Credits

* OpenCV - [this sample code](https://github.com/opencv/opencv/blob/master/samples/dnn/js_face_recognition.html) in particular
* [OpenFace](https://cmusatyalab.github.io/openface) - for the DNN models
