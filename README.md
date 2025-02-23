## newer libs
```
pcl opencv cmake
```

## CalibAnything

This is a project for LiDAR to camera extrinsic calibration using Segment Anything. It needs an image, a frame of point cloud, the camera intrinsic and an initial value of the extrinsic. The related paper is [Calib-Anything: Zero-training LiDAR-Camera Extrinsic Calibration Method Using Segment Anything](https://arxiv.org/abs/2306.02656). For more calibration codes, please refer to the link <a href="https://github.com/PJLab-ADG/SensorsCalibration" title="SensorsCalibration">SensorsCalibration</a>.

## Environment
```shell
# pull docker image
docker pull xiaokyan/opencalib:v1
```

## Compile
```shell
git clone https://github.com/OpenCalib/CalibAnything.git
cd CalibAnything
# mkdir build
mkdir -p build && cd build
# build
cmake .. && make
```

## Test Example
We provide examples of two dataset:
```
cd CalibAnything
./bin/run_lidar2camera ./data/st/1
./bin/run_lidar2camera ./data/ours/1
```
We have given a reference value of the extrinsic. You can set an initial error to the reference extrinsic in file `data/initial_error.txt` and test the accuracy of the algorithm. The format is "roll pitch yaw(degree) x y z(m)"

## Test your own data
```
./bin/run_lidar2camera <data_folder>
```
The data folder should contain:
- image file
- lidar file: the reflectivity is needed
- calib file: 
  - camera intrinsic: 3x3 matrix, flattened in row-major order
  - distortion coefficient: `[k1, k2, p1, p2, p3, ...]`, use the same order as [here](https://amroamroamro.github.io/mexopencv/matlab/cv.initUndistortRectifyMap.html)
  - a rough LiDAR-to-camera extrinsic: the first three rows of the transformation matrix, flattened in row-major order

Example of `calib.txt`:
```
K: 2152.8 0 971.3 0 2155.5 605.9 0 0 1
D: -0.1192 0.162 0.00073985 0.0014
T: 0.0188623 -0.999822 -9.36529e-05 -0.0323222 0.0288601 0.000638227 -0.999583 -0.396685 0.999405 0.0188516 0.028867 -0.0869361 
```
- `masks`: masks generated by Segment Anything

If you don't want to add error to your initial extrinsic, change the inital error to all zeros in file `data/initial_error.txt`. 

### * Generate Masks
Follow the instructions in [Segment Anything](https://github.com/facebookresearch/segment-anything) and generate masks of your image.

First download a model checkpoint. You can choose [vit-l](https://dl.fbaipublicfiles.com/segment_anything/sam_vit_l_0b3195.pth).
```
# environment: python>=3.8, pytorch>=1.7, torchvision>=0.8

# install
git clone git@github.com:facebookresearch/segment-anything.git
cd segment-anything; pip install -e .
pip install opencv-python pycocotools matplotlib onnxruntime onnx

# run
python scripts/amg.py --checkpoint <path/to/checkpoint> --model-type <model_type> --input <image_or_folder> --output <path/to/output>
## example(recommended parameter)
python scripts/amg.py --checkpoint sam_vit_l_0b3195.pth --model-type vit_l --input 1.png --output ./output/ --stability-score-thresh 0.9 --box-nms-thresh 0.4 --stability-score-offset 0.9
```

## Output
- initial projection: `init_proj.png`, `init_proj_seg.png`
- projection after adding error: `error_proj.png`, `error_proj_seg.png`
- refined projection: `refined_proj.png`, `refined_proj_seg.png`
- refined extrinsic: `extrinsic.txt`

## Citation
If you find this project useful in your research, please consider cite:
```
@misc{luo2023calibanything,
      title={Calib-Anything: Zero-training LiDAR-Camera Extrinsic Calibration Method Using Segment Anything}, 
      author={Zhaotong Luo and Guohang Yan and Yikang Li},
      year={2023},
      eprint={2306.02656},
      archivePrefix={arXiv},
      primaryClass={cs.CV}
}
```
