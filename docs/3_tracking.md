# Network Tracking
Open the terminal and type `jupyter notebook` to start Jupyter Notebook.  
Find the root directory of MitoTNT and open `mitotnt_tracking_pipeline.ipynb` for mitochondrial network tracking.

## 1. Generate inputs for tracking
**In this section we will process the raw data into a format that is used for the subsequent tracking.**

First specify the directories we will use:

- `work_dir`: the directory where data will be processed and stored. For test data, you can use the directory of `test_data` on your machine

- `data_dir`: the directory where MitoGraph segmented mitochondria is stored. For test data, this is `test_data/mitograph`.

- `input_dir`: the directory where the processed inputs used for tracking will be stored. For test data, you can use `test_data/tracking_inputs`. This is an empty folder that will be created.

After specifying the folders, we need to set a few parameters:

- `start_frame`, `end_frame`: the range of frames to process.

- `node_gap_size`: the number of nodes to skip when creating full-resolution graphs from mitograph `.gnet` files. Default to 0 (use all nodes).

All processed inputs will be saved as a single compressed `.npz` file in `input_dir`.

To start making inputs:
```
generate_tracking_inputs.generate()
```

## 2. Frame-to-frame tracking
**In this section we will perform node assignments for each consecutive frames.**

In addition to the directories declared above, we will create `output_dir` to store the tracking outputs. For test data, you can use `test_data/tracking_outputs`.

Additional parameters needed for frame-to-frame tracking:
- `tracking_interval`: the frame interval between the two frames to be tracked. Default to 1 (every consecutive frame).  

Distance threshold is computed as the minimum of two values:  
1) the distance to the N-th closest neighbor where N is given by `cutoff_num_neighbor`, default to 10;  
2) the frame interval times the maximum allowed speed given by `cutoff_speed`, default to `None` and estimated from the distance matrix.

- `graph_matching_depth`: the maximum level used for graph comparison. Default to 2 (usually sufficient).
- `dist_exponent`, `top_exponent`: the final cost term is given by D<sup>dist_exponent</sup> x T<sup>top_exponent</sup>, where D, T are the distance and topology costs respectively. Default both to 1 (equal weighting).


To run tracking:
```
network_tracking.frametoframe_tracking()
```

## 3. Gap closing
**In this section we attempt to merge tracks that are mistakenly terminated during frame-to-frame tracking.**

Additional parameters need to be set:

- `min_track_size`: the minimum number of frames for the tracks to be kept. Default to 5.

- `max_gap_size`: the maximum number of frames for which gap closing is allowed. Default to 3. Value of 1 indicates no gap closing.

- `block_size_factor`: values less than 1 allows the sliding block implementation of gap closing to prevent memory overflow due to large cost matrix. The size of the block is given by `block_size_factor` * total number of tracks to be closed. Default to 1 (close all tracks at once).

The final node trajectories are saved in `final_node_tracks.csv` file.
Each row is one node at one time point. 
Each column is an attribute of the given node, described below.

To use gap closing:
```
network_tracking.gap_closing()
```

## 4. Evaluate output

The final node trajectories are saved in final_node_tracks.csv file.  
Each row is one node at one time point.  
Each column is an attribute of the given node, described below.  

### Columns

- `frame_id`: frame number of the node.

- `frame_node_id`: node id at the given frame. Each frame has its own indexing.

- `unique_node_id`: node id shared by all the nodes in the same track at different frames. Each track is uniquely indexed throughout the whole trajectory. This is essetially the tracking information.

- `frame_seg_id`: segment id for all the nodes in the same segment. Each frame has its own indexing. The branching points are not assigned.

- `frame_frag_id`: fragment id for all the nodes in the same connected component. Each frame has its own indexing.

- `connected_unique_node_id`: unique_node_id for neigboring nodes in the graph. This has all the topology information.

- `x`, `y`, `z`: coordinates for the node.

- `intensity`, `width`: pixel intensity and tubular width for the node given by MitoGraph.

