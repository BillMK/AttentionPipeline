# Fast and accurate object detection in high resolution 4K and 8K video using GPUs [arXiv:[1810.10551](https://arxiv.org/abs/1810.10551)] [media: [1](https://www.ece.cmu.edu/news-and-events/story/2018/11/franchetti-object-detection.html), [2](https://techxplore.com/news/2018-11-4k-8k-video-gpus.html)]
internship project at CMU, 2017-2018

![Illustration image](https://github.com/previtus/intership_project/blob/master/project_illustration.jpg)

Working with high resolution videos to locate certain objects. First pass serves as attention selection on low res version / fast check, second pass then checks more thoroughly on selected areas.

# Instructions

## Installation

Sample workstation:
- Ubuntu 16.04.3
- CUDA 8.0 (cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb)
- CUDNN 5.1 (cudnn-8.0-linux-x64-v5.1.tgz).

Used Anaconda3 and these modules (likely also works with newer versions)
- matplotlib 2.0.0
- tensorflow-gpu 1.0.1
- Theano 0.9.0
- Keras 2.0.3
- hdf5 1.8.17
- pillow 4.1.0

_ps: useful to install on server with restricted access: `python setup.py install --user`_

**Install**
- python 3.6.1, tensorflow with gpu and cuda support, keras (see list above)
- version 1: YAD2K, python YOLO v2 implementation: https://github.com/allanzelener/YAD2K (commit hash a42c760ef868bc115e596b56863dc25624d2e756)
  * put files from "__to-be-put-with-YAD2K" to YAD2K folder
  * make sure that there is correct path to the YAD2K folder in "yolo_handler.py" on line `yolo_paths = ["/home/<whatever>/YAD2K/","<more possible paths>"]`

- version 2: darkflow, another tensorflow YOLO v2 implementation, worked better with server deployment: https://github.com/thtrieu/darkflow
  * See [this gist](https://gist.github.com/previtus/bbecf03ae2ab1e952eb6cde26dd85638) to test out darkflow on it's own
- prepare data *(see the ffmpeg commands bellow)* so it follows this hierarchy:
  * VideoName (whatever name, for example `PL_Pizza sample`)
    * input
      * frames (whatever name again, for example separate different fps)
        * 0001.jpg
        * 0002.jpg
        * ...

## Data preparation

Works with high resolution videos, respectively with the individual frames saved into input folder.
We can convert the resulting annotated output frames back into video.

**[Video to frames]** 30 images every second (30 fps, can be changed), named frames/0001.jpg, frames/0002.jpg, ...
- `ffmpeg -i VIDEO.mp4 -qscale:v 5 -vf fps=30 frames/%04d.jpg`

**[Frames to video]** Keep the same framerate
- `ffmpeg -r 30/1 -pattern_type glob -i 'frames/*.jpg' -c:v libx264 -vf fps=30 -pix_fmt yuv420p out_30fps.mp4`

## Running v1

- Go through proper installation of everything required.
- `cd /<path to project>/video_parser_v1`
- `python run_fast_sketch.py -horizontal_splits 2 -attention_horizontal_splits 1 -input "/<custom path>/PL_Pizza sample/input/frames/" -name "_ExampleRunNameHere"`
- See the results in `/<custom path>/PL_Pizza sample/output_ExampleRunNameHere`

## Running v2

- Go through proper installation of everything required.
- `cd /<path to project>/video_parser_v2`
- `python run_serverside.py -horizontal_splits 2 -atthorizontal_splits 1 -input "/<custom path>/PL_Pizza sample/input/frames/" -name "_ExampleRunNameHere"`
- See the results in `/<custom path>/__Renders/<_ExampleRunNameHere>/`

To add the support of additional server workers:

![Client - server illustration](https://github.com/previtus/AttentionPipeline/blob/master/client-servers.jpg)

- (optionally) prepare server workers with either this setup: 
  * python Server.py	(useful `CUDA_VISIBLE_DEVICES=0 python Server.py`) on several server nodes. Each of these binds port :8123
  * connect via ssh tunnel from the client, use `python ssh_server_connect.py` as a guidance on which calls you need to run. It will be something like `ssh -N -f -L 9000:"+server_name+":8123 "+user+"@"+server_name+".pvt.bridges.psc.edu`
  * (the main client code will try to connect to ports 9000 .. 9099 locally - with this tunnel it will connect to the server workers on their individual servers and port :8123)

- (optionally) alternatively run on one server with multiple GPU's with this setup: 
  * `CUDA_VISIBLE_DEVICES=0 python Server_gpuN.py 1`
  * `CUDA_VISIBLE_DEVICES=1 python Server_gpuN.py 2`
  * etc. `CUDA_VISIBLE_DEVICES=<id-1> python Server_gpuN.py <id>`
  * Each one will again listen on local ports, this time it's :8000 + id*(11) (where id was 1,2, ...) 
  * Finally run the client on yet another GPU and it will try to locally connect to all ports between 8000 ... 8099

## (optional) Annotation
When the python code is run with `-annotategt 'True'`, then the model will look for which frames have ground truth annotations accompanying them (in VOC style .xml file next to the .jpg). For these frames it then saves results into the output folder (into files `annotbboxes.txt` and `annotnames.txt`).

Visualization tool can be then run (with paths to the input and output folder set correctly). For example set file "visualize_gt_prediction.py" with these paths:

`gt_path_folder = "/<path>/intership_project/_side_projects/annotation_conversion/annotated examples/input/auto_annot/"
output_model_predictions_folder = "/<path>/intership_project/_side_projects/annotation_conversion/annotated examples/output_annotation_results/"`

As result we should see something like image in  [*_side_projects/annotation_conversion/annotated examples/example_visualization_of_ap_measurement.jpg*](https://github.com/previtus/intership_project/blob/master/_side_projects/annotation_conversion/annotated%20examples/example_visualization_of_ap_measurement.jpg)

- hand annotation possible with labelImg: https://github.com/tzutalin/labelImg (install with `pip install labelImg`)
- automatic annotation of the "PNNL ParkingLot Pizza" dataset with _side_projects/annotation_conversion/convert_parking_lot_to_voc.py

## (optional) Time profiling

I used kernprof from https://github.com/rkern/line_profiler#kernprof. Follow installation mentioned there (`pip install line_profiler`).
- Put `@profile` before each function for profiling
- Run `kernprof -l -v run_fast_sketch.py`



