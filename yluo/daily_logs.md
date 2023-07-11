# Daily logs

Link to my [meeting notes](https://docs.google.com/document/d/1LRnpN_eE1WZ5-LrI0CYndENyy3PiCGERJvU9nurvOXs/edit?usp=sharing)


## Week 07/10 -- 07/16

### 07/11 Tue

- Attended the AI Testbed SombaNova training. The amount of work to alter the existing model and code will
be a significant hindrance to use SombaNove. Tomorrow's training session might dive deeper into code alteration and compilation
- Evaluated the RGB+Greyscale `compare_rgb16_rgb8_bs128_ep150` model with full sample. Result is shown in the first two figure.
- Evaluated the random crop + random horizontal/vertical flip model `aug_vit_rgb16_ir8_bs128_ep150`. The result is shown as the 
second set of figures. The clusterings are not as clear as full image model, such as `vit_rgb16_ir8_bs128_ep150`. This means the model 
really relied somehow on the pointing. However, on the other hand, the separation still is very obvious, which means model can
learn different information from different images.

<img src="./plots/gs_tsne_all.png" alt="isolated" width="400"/>
<img src="./plots/gs_tsne_four.png" alt="isolated" width="400"/>

<img src="./plots/aug_tsne_all.png" alt="isolated" width="400"/>
<img src="./plots/aug_tsne_four.png" alt="isolated" width="400"/>

### 07/10 Mon

- ALCF is down, so I have to wait.
- Worked on planning for next steps
- Worked on some astronomy stuff


## Week 07/03 -- 07/09

### 07/07 Fri

- Met with Dario and made a plan for next steps. The plan is on overleaf.
- Downloading more images, and optimized the downloading workflow
- Model trained with smaller images can be used to evaluate larger images.
- Build a workflow for contrastive learning. (Model design, training, testing and validating)

### 07/06 Thu

- Based on what have been achieved, we need to narrow down the scope and focus on some science cases
- Had another discussion with Dario and Bhupendra, here are the discussion details:
- Science case: cloud
  - altitude, texture of the clouds
  - color usually doesn't matter, so will use Greyscale+IR (1-ch + 1-ch) image pairs
  - narrow down to look at sky images only
  - Look at attention map and check what clouds the models are paying attention to
  - Bhupendra has a framework for motion data for cloud, might look into that.
- **Issue**: after 07/04, all ANL nodes should not be used for analysis because of some hardware alternations.
- Started the random crop + flip model training. The model is `aug_vit_rgb16_ir8_bs128_ep150`


### 07/05 Wed

- Attended the talk on `pyCOMPSS` framework. The framework can simplify the parallelization and deployment.
Another library `dislib` on distributed computing was introduced as well.
- Discussed with Dario and had following suggestions:
  - KNN classification to compare RGB+IR and RGB+Greyscale to understand how the knowledge learned by these two frameworks
  - Relative position of objects in an image should not matter as the model was trained to learn objects rather than positions.
  - Next step: use random crop and flip to augment the images preventing models from cheating on using positions to do clustering
  - The crop and flip must be synchronous among RGB and IR or Greyscale images to ensure they contain relatively similar information.
    (field of view, objects in the image)
- Color might not be important so we might use Greyscale+IR for a new model
- New project on all-sky camera: SAGE nodes have fisheye cameras that we can use.


### 07/04 Tue

Independence day, no work

### 07/03 Mon

- Checked the model training was finished.
- Start the evaluation.


## Week 06/26 -- 07/02

### 06/30 Fri

- Had another discussion on evaluating the model.
  - It's possible that the model might be learning the pointing rather than the content of the image (i.e., poles, sky)
  - Have to understand what mapping between RGB and IR the model learned.
  - To compare what is learned, start another model to compare RGB+Greyscale images.
  - The greyscale image is composed from RGB only.
- Started the RGB + Greyscale model training and well perform evaluation later. The mode is `compare_rgb16_rgb8_bs128_ep150`

### 06/29 Thu

Get-together, no work

### 06/28 Wed

- Attended the LANS seminar on low-precision mathematic-hardware. The idea is very interesting -- by introducing
stochastic rounding (i.e., probabilistically round up or down or zero or nearest). Improve the stagnation point and dynamic range by a lot!
- Analyzed the clustering and got attention maps based on our discussion yesterday. The result still looks pretty similar as expected. Because
we are minimizing the loss from two branches. 
- Discussed with my research advisor at Wyoming about my research here at ANL and communicated with Dario about potential application of 
self-supervised learning in astronomy. An idea is to combine photometry and spectroscopy information together through contrastive learning.

### 06/27 Tue

- Had another discussion with Dario and here are things to do:
  - Use Silhouettes score to check clustering method score
  - build clusters and check visually what the clustering is doing
  - model evaluation and inference
  - add pairs to tSNE plots and then evaluate a small batch of the files by eye
  - add intermediate checkpoint for model training to understand the evolution of position of points in the clustering
  - With Bhupendra: have another experiment with two PCA and cluster on RGB and IR individually. Then, comparing the results 
  with another instance that has single PCA and clustering fitting on RGB+IR together. This comparison can be used to understand
  the "closeness" of two models on inference
- Calculated embeddings with `vitsmall_rgb16_ir8_bs128_ep150` model.
- Clustering with `vit_rgb16_ir4_bs128_ep150`, result is shown below. The result is incredible as the separation between different 
categories of images is very clear and the relationships between image pairs have been learned as well.

<img src="./plots/vit_image.png" alt="isolated" width="400"/>
<img src="./plots/vit_all_clusters.png" alt="isolated" width="400"/>

### 06/26 Mon

- Had a deep discussion with Dario and Bhupendra about the scope of the project and potential deliverables
  - Current goal: **understand what a RGB model learned from the IR model counterpart**
  - Two aspect to consider:
    1. the common features RGB and IR models both learned.
    2. the mapping from RGB image to IR image learned by models. (This is important because IR images are less available than RGB images)
  - To evaluate models based on the above two criterions, need to have following steps:
      1. narrow down the scope: train and test images with certain features (i.e., night time images, sky images, cloud images)
      2. cluster the images
         1. if labeled data available, apply the same clustering on labeled data to understand some physical meaning of the cluster
         2. if no labeled data, try some meta-analysis with the information from node (location, node number, sky position, local time -> day, night)
  - some deliverables:
    - Object segementation and detection
    - cloud characteristics
      - thick and thin clouds?
      - object differentiation, sky vs. ground
    - other unexpected results from clustering
  - Additional idea to understand RGB vs. IR model: use an identical framework + model to train RGB + RGB pair. Where the second 
  RGB image, converted to grayscale image, replaces the paired IR image but remains the same shape and resolution as IR image
- Issue to resolve: data cannot be fitted in memory for analysis task due to too many (~100,000) embedding vectors

## Week 06/19 -- 06/25

### 06/23 Fri

- Automated the attention map generation step.
- Visually inspected some attention maps generated and found the `checkpoint_vit_rgb16_ir8` performed fairly well!
- Started training with smaller patches (`vit_rgb16_ir4_bs128_ep150`) on 6 nodes and `vit_small` architecture (`vitsmall_rgb16_ir8_bs128_ep150`)
on 2 nodes.
- Will need to check whether the training will run into not enough memory issue.
- Discussed with Dario about DINO training, will finalize it on Monday.
  - Train with 3-channel RGB + artificial 3-channel IR images. The 3-ch IR is made by separating temperate into three bins.
  - Need to put cloud into one bins. Will talk to Bhupendra about possible cloud temperature.
  - Training strategy
    - Training alternatively with RGB and IR images, and alternating the image type for both branches at the same time.
    - Train two models: RGB teacher + IR student, and IR teacher + RGB student

### 06/22 Thu

- Presented at the week meeting [Link](https://docs.google.com/presentation/d/1Oqt4WQ0bol99NMFtKYLR8YZGVwwUif35_zIo5Mjw0L4/edit?usp=sharing)
- Work on visualization and clusterization
- Got TSNE plots with PCA reduction
- Automated the embedding generation process
- Found there was a jump on learning rate for `checkpoint_vit_rgb16_ir8`, which could be caused by changing training epoch from 100 to 150
this changed the optimizer schedule. DO NOT change training epoch after the training started!

### 06/21 Wed

- The distributed training worked! Here are the changes made to add MPI:

1. For training code, before import `init_dist_mode` function or any MPI/NCCL initialization,
add this code

```py
try:
    os.environ['CUDA_VISIBLE_DEVICES']=os.environ['OMPI_COMM_WORLD_LOCAL_RANK']
except:
    print('Not OMPI_COMM_WORLD_LOCAL_RANK in this run!')
```

2. Then, use `utils.py` included in `self_supervised_bird_analysis` 
[Github link](https://github.com/dariodematties/self_supervised_bird_analysis/blob/main/utils.py)
3. Make sure MPI library is installed inside the container and is compatible with host system
MPI library. MPI ranks must be one rank per hardware thread (nproc x num_nodes).
4. Test it with multi-nodes!

- With a working MPI, can now train bigger model. Started to train `vit_rgb16_ir8_bs128_ep150`
- Document the workflow

### 06/20 Tue

- Continued to refine the workflow with some details on the template file, adding more macro options
- Started to work on the distributed memory training. Initially would like to use NVidia NCCL library
because it was built-in for CUDA but Dario has a working version of CUDA+MPI, so I will start to test
the working MPI solution first.

### 06/19 Mon

- Refine ML training workflow
  - Wrote a template with Macros to submit jobs
  - Wrote a python script to substitute macros in template with input and create target training directory
  - Reorganize data repository to standardize file access
- Now, the ML training process has a standardized, reproducible workflow with isolated directory to hold
model in the training as well as evaluation data.
- The environment is reproducible with the help of singularity container
- Data can be trackable with the pair index files.
- Working on parallelizing the training into multiple nodes


## Week 06/12 -- 06/18

## 06/16 Fri

- Use DINO's `visualize_attention.py` to generate attention map and there are three heads for `vit_tiny` architecture
- The resolution for IR is too low, so reduce the patch size for IR images while keep patch size for RGB the same
- There will be lots of training, so need to refine the whole workflow and keep track of everything.
- Started to train `vit_tiny` with RGB patch size 16, IR 8, and batch size 128. `checkpoint_vit_rgb16_ir8`

## 06/15 Thursday

- Presented at the [weekly meeting](https://docs.google.com/presentation/d/1zUZoETjtCwAetGjYBYzTIRs_4WDyj9cckZk5EvsfaQI/edit?usp=sharing)
- Made some visualization
  - Corner plots to show internal relationship between parameters
  - Attention maps from Vision transformers
- the resolution for IR image is low so the patch size must be smaller to create high
resolution attention map. But this created memory problem because smaller patches will
yield more blocks to train on and thus single node does not
have enough to fit both models and smaller patches. Need to use multi-node with distributed memory.


### 06/14 Wed

- Removed one model from the training so we just need to calculate loss and backpropagate for one model. The training will be more efficient
- Start to calculate embeddings for the test sample
- Got attention maps from vision transformer, but it didn't present any information?

### 06/13 Tue

- Found two image issues:
  1. Images before ~03/14 didn't have positions attached to their file names. Solution: Leave the files here.
  2. Some images (~2000) have discrepancy between position reported on RGB files (`.jpg`) and thermal files (`.csv`). There are two
plugins trying to take images causing a race condition. The image pair is technically correct, but one of the position is wrong because
one plugin tries to move the camera continuously and the other one is taking images disrupting the position recording. Solution:
move those files to `bad_pairs` directory, so they don't interfere with the training/testing
- Found another potential issue with the current implementation of the framework:
  - backpropagation might be incorrect because the two branches had two different models. Checked this one, and from the original paper
this shouldn't be a issue as the derivatives for `x` and `y` branches in their implementation are calculated back.
  - However, the current implementation has a duplicate model as the two models (IR and RGB) learn the same information. I will remove one
model before next training. I evaluated the two models and found them identical by comparing embeddings
- Now, data is ready and models are ready too for evaluation.

### 06/12 Mon

- ALCF is offline, so I have to wait
- Worked on a paper on reproducibility to be published
- Read DINOv2 paper

## Week 06/05 -- 06/11

### 06/09 Fri

- Continued the training for VICReg + ViT to ensure both models (ResNet and ViT) have 150 training epochs
- Evaluate the trained models using the embedding vector for new images.
- Discussed with Dario about next steps
  - for DINO, separate thermal IR images into three channels by setting temperature threshold
  - for all-sky camera, refer to [05/31](#0531-wed), and [this section](#cloud_pred).
- Downloaded new images, and optimized the processing workflow to build index cache. This should make the processing 
and pair-creating much faster.
- TODO: show some clusterization results next week

### 06/08 Thu

- Presented at the group meeting
- Chose to use DINO's vision transformer, will document this thing
- Tested ViT in VICReg with the current modifications
- Start to train VICReg + ViT, and continue training VICReg + ResNet
- Assembling a testing and validation dataset to evaluate these two models and do clusterization analysis

### 06/07 Wed

- Came to two realizations for Vision Transformers (ViT)
  - Augmentation is the human prior for network. The feature destroyed in
augmentation process should be things we don't care
  - Training dataset construction is very critical, which dictates the attention of the network
- Trained VICReg + ResNet-50 on a full-node. Training loss and learning rate are shown below.
- Start to swap in ViT to the model part and test using a single thread training.
  - Somehow the hugging-face implementation "official" for ViT does **not** work
  - DINO's vision transformer, claimed to be taken from above, does work!

<img src="./plots/ResNet_dist_loss.png" alt="isolated" width="800"/>
<img src="./plots/ResNet_dist_learninig_rate.png" alt="isolated" width="400"/>

### 06/06 Tue

- Modified `DataLoader` to fit the containerized training environment,
as there are volume binding that changed the original file path
- Tested the containerized training environment. This should conclude
the development for VICReg preprocessing and training, now move to change
the model
- Started to train VICReg with ResNet-50 on `full-node` queue
- Challenged by the batch system on ThetaGPU system, `cobalt` is an imposter
batch system using the "same" syntax as the more popular `PBS` batch job
management system.

### 06/05 Mon

- Truncated the rgb image to similar FoV as the thermal camera and downsampled
the image to reduce its size.
- Tested the training on both single-gpu and single-node with 8 gpus
- Maximum batch-size for 8 gpu on  the `full-node` queue is 128
- Wrote `.def` to create singularity image for training
- Can use conda to setup the environment as well, need to decide later.

## Week 05/29 -- 06/04

### 06/02 Fri

- Attended DSL seminar for a review and discussion of recent update on Scientific Machine Learning.
Unsupervised LG-Net might be interesting to consider for solving well-posed PDEs
- Attempted to train the modified VICReg using a single GPU but have to modify the batch size. This 
likely would yield a meaningless results as the batch size is too small (4) to fit into single GPU.
- Discussed with Dario and found out the training was too slow and not useful as aforementioned.
- After the discussion, there are three things to do:
  1. Parallelize the current single thread VICReg
  2. Increase the batch size. This can be done after parallelization
  3. Download more image data. Now only have 7000 images but would need more.
  images are being downloaded right now to ALCF.

<img src="./plots/training_loss_two_panels.png" alt="isolated" width="800"/>
<img src="./plots/learning_rate.png" alt="isolated" width="400"/>


### 06/01 Thu

- Make the Quad chart!
- Changed the VICReg model to fit two different inputs
  - Two instances of the same architecture (i.e., ResNet) -- done
  - Two different architectures -- TODO
- Need to modify the loss function in order to back-propagate to both branches
- Finished the modification today and successfully ran the code on a single GPU

### 05/31 Wed

- Finished implementing the dataloader for sage images data, now the loader works
- Tested the loading function with a single-gpu and it works without distribution package, i.e., do it sequentially.
- <a id="cloud_pred"></a>Discussed with Dario about the cloud prediction project using all-sky camera, and Dario suggested two frameworks:
  - Two component: Joint embedding architecture (JEA) + single transformer fed with embedding vector from the JEA. JEA for image characterization, while the second transformer is for prediction. See this two papers: [DETR](https://arxiv.org/abs/2005.12872), [I-JEPA](https://arxiv.org/abs/2301.08243)
  - Single component: JEA but with two branches feeding different time of the sky image to let one NN model (current image + time input) match the embedding vector of that of the other NN model with future image.
- TODO: switch out ResNet and make sure the training works with a single gpu before putting the large scale training.
- Will continue to download more SAGE data for training.

### 05/30 Tue

- Transferred local data to ALCF
- Attended the software sustainability seminar
- Data loader and reader look fine but still has issue with multiprocessing, will look into it tomorrow.
- 03/10 - 03/24 data has been downloaded.
- The data format is
  - original unmodified JPG RGB images are in `rgb` directory
  - processed JPG images with only left half (optical images) are in `processed` folder under `rgb` directory
  - thermal images in `.csv` file with celsius degree per pixel are in `thermal` folder
  - metadata files created from querying using `sage_data_client` are in `sage_meta` folder
  - to check data consistency, I created a txt file for each JPG & thermal image pair and put them in `pairs` folder, and put in directories with SAGE node name.

### 05/29 Mon

Memorial day, no work.

## Week 05/22 -- 05/28

### 05/26 Fri

- Finally got access to ALCF!
- Migrated the workflow to ALCF
- Continued to change the input to fit into VICReg framework
- Acquired username and password from Raj to access the private node images

### 05/25 Thu

- Downloaded imaging data based on links in metadata files
- Send Bhupendra a list of URLs to files I cannot download due to a credential issue.
- Tweak the framework to change the input from single input with augmentation to two inputs (optical RGB + thermal IR).
- Still don’t have access to ALCF yet, so will try to use google colab for testing
- The W056 node physical location changed from ANL to San Deigo around 05/20/2023, so there might be some changes in the image quality
- Daytime and night time calculations for each node would need the physical location information to get the timezone. This hasn’t been done yet!

### 05/24 Wed

- Checked the VICReg framework and discussed the detail with Dario
- Automated the data downloading process
- Reformated the downloaded images for training
- Splitted the JPG images to left RGB + right thermal images. (only keep the RGB images)

### 05/23 Tue

- Continued to refine the scope of the project
- Downloaded sage node data but there was an issue with credential to download data
- Inspected the sage node data and developed method to extract specific image
- Getting access to ALCF theta

### 05/22 Mon

- Attended orientation
- Met with Dario and Bhupendra to discuss the project detail
- Discussed the SAGE node data format
  - 4 pointing per column (1, 5, 9,13, …)
  - A total of 32 pointing positions and 8 are pointing sky
  - Client code: https://github.com/RBhupi/sage-data-client
  - Data web: https://portal.sagecontinuum.org/ (use ANL to login)
  - Data model is a CSV file with “celsius” per pixel at each position (read the head to understand that)
  - A sample code to query: ​​https://github.com/RBhupi/Konza_Mobo_Analysis/blob/main/mobo_data_download_nc.py
  - Figure out how many images are available at each position and what’s the production rate for each position
  - Want “thermal.celsius.csv” in file name.
