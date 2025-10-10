# SLAM-Tools
The repository contains instructions for Localization and Map Analysis.

# Localization Analysis
## [EVO](https://github.com/MichaelGrupp/evo) Download and Install:
```
git clone https://github.com/MichaelGrupp/evo.git
cd ~/evo
pip2 install --editable . --upgrade --no-binary evo
```
Download the `bagmerge.py` file from this repository and put that inside `~/evo/test/data` directory. Merge 2 rosbag files:
```
cd ~/evo/test/data
python bagmerge.py gt_odometry.bag odometry.bag -o odometry_analysis.bag
```
## Trajectory Analysis and Plot:
```
cd ~/evo/test/data
evo_traj bag odometry_analysis.bag /odom --ref /gt_odom -va --plot --plot_mode=xyz
```
## Absolute Position Error (APE) Analysis, Plot and Save:
```
cd ~/evo/test/data
mkdir results
evo_ape bag odometry_analysis.bag /gt_odom /odom -va --plot --plot_mode=xyz --save_results results/odometry_analysis.zip
```
Timestamps are used to establish point correspondence (registration) for the alignment. They don't need to be the same but in the same interval and ideally synchronized. You can play around with the --t_offset and --t_max_diff parameters if you know your timesync, see evo_ape tum --help:
```
  --t_max_diff T_MAX_DIFF   maximum timestamp difference for data association
                        
  --t_offset T_OFFSET   constant timestamp offset for data association
```
If you want to align the trajectory just based on the geography shape, you have to use a format without timestamps (e.g. the kitti format) and both files have to have exactly the same number of corresponding poses.
## Relative Position Error (RPE) Analysis, Plot and Save:
```
cd ~/evo/test/data
mkdir results
evo_rpe bag odometry_analysis.bag /gt_odom /odom -va --plot --plot_mode=xyz --save_results results/odometry_analysis.zip
```
If you want to align origin during RPE calculation, add `--align_origin` flag in the command line.
## Post Result Analysis and Plot:
```
cd ~/evo/test/data
mkdir results
evo_res results/odometry_analysis.zip -p --save_table results/table.csv
```
## Rosbag to Text File Conversion (Optional):
```
rostopic echo -b odometry_analysis.bag -p /odom > odom.txt
```
# Map Analysis
## CloudCompare Download and Install
Run the following command inside terminal:
```
sudo snap install cloudcompare
```
## Convert PCD files to PLY
You need to download pcl-tools to convert .pcd files to .ply files.
```
sudo apt install pcl-tools
```
Now, go to the directory where the map and reference ground truth map is located. Then run in the terminal:
```
pcl_pcd2ply map.pcd map.ply
pcl_pcd2ply gt_map.pcd gt_map.ply
```
## Compare with CloudCompare
Run Cloud compare by typing the following command inside terminal:
```
cloudcompare.CloudCompare
```
Open the two ply files. Just drag and drop them inside Cloud Compare one by one. Select both of them one by one (hold ctrl key) to activate distance compare button on top. 

## Compute Error
After installing pcl-tools, you can check the functions [here](https://github.com/PointCloudLibrary/pcl/tree/master/tools). Use this command line with pcl-tool:
```
pcl_compute_cloud_error filename.pcd error_pc_out.pcd -correspondence nn > error.txt
```
Here are the descriptions:
```
filename.pcd - the input
error_pc_out.pcd - output point cloud (colored with error)
-correspondence nn indicates nearest neighbor correspondence for errors (there are other options)
error.txt - saving the errors to a text file
```
To downsample/filter to reduce the data size (i.e. applying a grid filter)):
```
pcl_voxel_grid output.pcd input.pcd -leaf 0.0025
```
## Follow Instructions from Cloud Compare
Follow [these instructions](https://www.cloudcompare.org/doc/wiki/index.php?title=Cloud-to-Cloud_Distance) to compute point to point distance. Remember that the compared is 'map.ply and the reference is 'gt_map.ply'. After you have imported 2 pointcloud, click on compute to compute the distance.

## Visualize Error
You can see the Mean Distance and Standard Deviation on the bottom console. Select `map.ply` from 'DB Tree' (upper left pane) and activate the colour bar from 'Properties' (lower left pane), change the background (Display--> Display Settings--> Colors and Materials--> Colors--> Background) and hide the `gt_map.ply` from 'DB Tree'. Hide the centre '+' sign from Display--> Toggle Viewer Based Perspective.

