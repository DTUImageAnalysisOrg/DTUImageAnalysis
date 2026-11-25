***Solution is now available! Download the full solution from here:*** [Solution](../downloads/sol_material-10.zip){ .md-button .md-button--primary .inline-button }

# Real Time Object Detection Using Viola Jones Method


## Part 1: Cascade Classifier used for object detection

In the following exercises, we'll investigate the performance of the Viola-Jones algorithm when trained to detect other classes. To do this, we've trained a simple stop-sign object detector for you, which we'll use in the following exercises. Later on, you'll be able to train such a model yourselves (optional). We'll show, that the framework doesn't necessarily need to be used for faces, but that it is instead a general object detection tool which can be trained to solve many object detection tasks.


We'll start by employing a pretrained stop-sign object detector for image capture. To do this, we'll use the script **VJ_performance_eval.py** in the ***data*** directory. You can find all the scripts and data we'll need [here](https://github.com/DTUImageAnalysisOrg/DTUImageAnalysis/tree/main/exercises/ex10-Viola-Jones/data). Download the folder and `cd` into it.



### Exercise 1: Testing stop-sign model performance
We'll first investigate how the trained model performs on a simple image of a stop sign. 

In the **data** folder, the image *stop_sign.jpg* can be found. 

First, try to visualize the image. 

We've provided a script to evaluate the model performance called *VJ_performance_eval.py*. It is located in the ***data***-folder. 
The script is implemented such that it must be run from the terminal, and it needs to be run with some command line arguments. If you're unsure about how to run it, [look here](../../ex1-setup/running_python_terminal). 

We may evaluate the model performance qualitatively using the *VJ_performance_eval.py* script. You run it by opening a CLI in your exercise-root (the directory which contains ***data/*** as a sub-directory) and issuing the following command:

```
python data/VJ_performance_eval.py --model data/stop_sign_detector.xml --input data/stop_sign.jpg --type image --w 24 --h 24
```

**Question 1**: *Does the model perform as expected?*

**Question 2**: *Inspect the script contents and check how the model configures its bounding box shape. Now try to print the values. Specify the top left and bottom right points of the bounding box frame.*


<!-- START_SOLUTION 1 -->
??? tip "Solution 1"
    ```py

    #inside the loop one can find the corner coordinates
    x1, y1 = x0 + w, y0 + h
    print(f"Upper left corner: {x0,y0}, Lower right corner: {x1,y1}")
    ```
<!-- END_SOLUTION 1 -->


### Exercise 2: Visual inspection of Haar features
In order to understand the underlying decision logic that the model makes during execution, we'll take a closer look at the computed Haar features that the model accepts during evaluation. As a result, we have visualized the Haar features for a single $24\times 24$ crop of *stop_sign.jpg*. Refer to the directory **haar_features** to get insights into what the model has learned to put attention towards at different stages when given the sample image as input.

**Question 3**: *Looking at some of the computed haar features, can you deduce which features the model attends to at the different classification stages?*

!!! NOTE "Interpretation hint"
    Each feature in the figures is a spatial filter. The earlier features are "coarse", acting as an initial filter to discard "definitely negative" samples, but accepting many false positives. The filter location in the image is where in the window the feature has been fitted for evaluation. The window is then slided across the complete image frame; as a result, some of the images may have a filter location which is not a good match with the specific image. Therefore, the sizes are more easily interpreted than the specific location. Whites indicate "sum these intensities", whereas blacks indicate "subtract these pixel intensities". 

??? Tip "Hint:"
    It helps thinking about which edge orientations or patterns the features will produce high or low values for. As an example, stage 0 features seem closely related to a specific part within the stop-sign. Which? And how about the features coming right after?

<!-- START_SOLUTION 2 -->
??? tip "Solution 2"
    All stage 0 features are especially well-suited for detecting vertical text and gaps between text (vertical edges), and are placed somewhat in the middle of the window.

    Stage 1, feature 0 is perfectly suited for detecting angled (bright) edges, and seems to be somewhat local, possibly detecting angled boundaries of letters S, O or P. 

    Stage 1, feature 1 is a global feature. It would become high-valued if there is a "darker" sign within a brighter background, and low-valued if there is a bright region in the middle and darker regions at the boundary of the window.

    Similarily, stage 1, feature 2 would be well-suited for detecting bright text in the middle of a dark sign, producing low / negative values.
<!-- END_SOLUTION 2 -->


### Exercise 3: Video stream object detection
Now we'd like to look at the performance of the stop sign detector when tasked with a moving image, to see whether it is robust to real-time image changes, such as object scaling and pose adjustments. Again run the file *VJ_performance_eval.py*, now using the videos **van_video.mp4** and **scale_diff.mp4** as image capture input. Remember to change the type of input to video format when issuing the command in the terminal, e.g.:


```
python data/VJ_performance_eval.py --model data/stop_sign_detector.xml --input data/van_video.mp4 --type video --w 24 --h 24
```

**Question 4**: *How does the model perform? When does it correctly detect or not detect the stop sign?*

During detection, the model computes features based on a series of scaled image sizes, forming a pyramid of images. OpenCV allows us to modify this pyramid through their detection framework.

**Question 5**: *Examine the script file and try to modify the scale_factor parameter of the detection function. Explain the effects of increasing and decreasing the scale factor (accuracy vs. frames per second). Try and see if you can find a scale-factor for which the stop sign is detected at all distances while it is completely contained in the frame.*

<!-- START_SOLUTION 3 -->
??? tip "Solution 3"
    Increasing the scale-factor achieves larger FPS. This is due to the lower number of windows in the window pyramid. But increaseing the scale-factor too much makes it impossible to detect small instances (i.e. signs at large distances). Thus, too large scale factor increases the False Negative Rate. Decreasing it to values closer to 1 dramatically reduces the FPS-performance, while decreasing the False Negative Rate. But in some cases, this may increase the False Positive Rate as well, as there is a larger chance for a false positive evaluation at some level in the window pyramid. 

    The value 1.072 works well for **van_video.mp4**, whereas the value 1.015 somewhat works well for **scale_diff.mp4**.

    In the case for **scale_diff.mp4**, there will always be a couple of false negative frames. 
    The persistent False Negatives (e.g. frames 8-10) in scale_diff.mp4 are likely due to spatial quantization and a limited training dataset. Even with a small scale factor of 1.015, the detector creates a discrete 'ladder' of search windows. As the sign moves smoothly in 3D space, its projection on the 2D sensor passes through scales that do not perfectly align with the model's rigid Haar-feature geometry.

    Furthermore, since the model was trained on a limited dataset, it lacks 'generalization', as it likely requires very specific and localized contrast matches to pass all cascade stages. When the sign is scaled to a size that forces pixel interpolation (blurring the edges), the mathematical sum of the Haar rectangles falls below the stage threshold, causing the cascade to reject the window, resulting in fewer detections. This can in some cases be alleviated by reducing the minimum number of neighbors necessary for a detection.
<!-- END_SOLUTION 3 -->



## Part 2: Haar feature diagnostics 
---
Viola-Jones is based on Haar features. Each stage of the decision tree evaluates one or multiple Haar features, and applies a threshold for "continuing" the cascade. 
We'd now like to take a deeper look at what a Viola-Jones style model actually bases decisions on in relation to an image window. In the **haar_features**-directory in data you are provided with 5 classifier stages, each showcasing 3 or 2 Haar features used in the specific stage. 


### Exercise 4: Loading and cropping images (Optional)
We will have a look at the last Haar feature provided in **stage_3.png**. The background image is **annotation_img.jpg**, and each square in the feature has sidelength 2. 

!!! NOTE 
    Visualization of the cascade classifiers/Haar regions was made using OpenCV2's CLI tool *opencv_visualisation*. This tool has some limitations, namely, the input image has to be of shape $(w_{window}, h_{window})$, yet the output image is upscaled x10. Upscaling adds some artifacts to the image (e.g. a decrease in contrast). Our attention will therefore be on the reference image only.

Your task is: 
1. Load the two images, both as grayscale ubyte images
2. Crop the feature image such that we only see the last feature of stage 3
3. Identify the pixel indices where the Haar feature is located in the feature image 
4. we now want to slice our *reference* image such, that we only include the pixels that are contained within the chosen haar feature and the 1-pixel border. Find the pixel coordinates and slice the reference image accordingly. In other words, we want to know the coordinate correspondents in the $24\times24$ image and visualize the crop of that exact region, including its one pixel boundary.

??? TIP "Hint:"
    Hint: Histograms could be a valuable asset for identifying pixel coordinates. 

??? TIP "Code template if you are stuck: "
    ```py 
    feat_dir = 'data/haar_features'
    img_path = os.path.join(feat_dir, 'stage_3.png')
    stage_img = io.imread(img_path)
    ref_path = 'data/annotation_img.jpg'
    ref_img = io.imread(ref_path)
    ref_img = img_as_ubyte(rgb2gray(ref_img)) # Want the image to be grayscaled
    w_enlargened, h_enlargened = (stage_img.shape[0],stage_img.shape[0])
    ROI_feat = 2
    ROI_img = stage_img[:, ?*w_enlargened:(?+1)*w_enlargened]

    ys, xs = np.where((ROI_img == ...) | (ROI_img == ...))
    coords = np.vstack((..., ...)) 

    # Get (y0,x0) and (y1,x1), slice and show image
    ymin, xmin, ymax, xmax = np.concatenate((coords.min(axis=1), coords.max(axis=1)))

    #now convert back to reference image by taking scale into account
    y_min_ref, y_max_ref = ymin//?,ymax//?
    x_min_ref, x_max_ref = xmin//?,xmax//?
    print(y_min_ref, y_max_ref, x_min_ref, x_max_ref)
    sliced_img = ref_img[y_min_ref-1:y_max_ref+2,x_min_ref-1:x_max_ref+2]
    ```

<!-- START_SOLUTION 4 -->
??? tip "Solution 4"
    In the following, we'll take a deeper look at what the model sees when computing a Haar feature. We'll both get more comfortable with what an integral image is and also how it's employed to produce fast computation of large image slices.


    ```py

    import os 

    import matplotlib.pyplot as plt
    import numpy as np
    from skimage import io
    from skimage.color import rgb2gray
    from skimage.util import img_as_ubyte
    ```
    ### Loading and displaying the Haar images 

    We'll start by loading both the reference and a computed Haar stage image.

    ```py
    # Load both the stage image and the reference image
    feat_dir = 'data/haar_features'
    img_path = os.path.join(feat_dir, 'stage_3.png')
    stage_img = io.imread(img_path)
    ref_path = 'data/annotation_img.jpg'
    ref_img = io.imread(ref_path)
    ref_img = img_as_ubyte(rgb2gray(ref_img)) # Want the image to be grayscaled

    # Look at the stage image shape
    print(stage_img.shape)
    ```

    We find, that the images have been upscaled by x10 (240x1200) // 10 -> (24x24x5). Slicing according to our chosen feature and resizing, we get:

    ```py
    # Rescale image down to w_window x h_window
    w_enlargened, h_enlargened = (stage_img.shape[0],stage_img.shape[0])
    ROI_feat = 2
    ROI_img = stage_img[:, ROI_feat*w_enlargened:(ROI_feat+1)*w_enlargened]

    _, axes = plt.subplots(ncols=2, figsize=(8,8))

    axes[0].imshow(ROI_img, cmap='gray')
    axes[0].set_title('Haar image frame')
    axes[1].imshow(ref_img, cmap='gray')
    axes[1].set_title('Reference image')
    plt.show()
    ```
    We now want to zoom in to the region of interest. Looking at the image histogram for the feature image, this gives us some hints.



    ```py
    plt.hist(ROI_img.ravel(), bins=256)
    plt.show()
    ```
    Two very clear spikes form at 0 and 255, which incidentally are also the pixels we'd like to mask. Now, we find the coordinates of this area and scale them back to the actual 

    ```py
    ys, xs = np.where((ROI_img == 0) | (ROI_img == 255))
    coords = np.vstack((ys, xs)) # ys,xs due to standard convention of packages

    # Get (y0,x0) and (y1,x1), slice and show image
    ymin, xmin, ymax, xmax = np.concatenate((coords.min(axis=1), coords.max(axis=1)))

    #now convert back to reference image by taking scale into account
    y_min_ref, y_max_ref = ymin//10,ymax//10
    x_min_ref, x_max_ref = xmin//10,xmax//10
    print(y_min_ref, y_max_ref, x_min_ref, x_max_ref)
    ```
    Notice, we choose to append an extra border layer, in order to effectively compute the Haar feature, as it depends on values outside of the bounding box. This is only relevant if we choose to use *np.cumsum* - if you use *cv2.integral* the border is zero padded.
<!-- END_SOLUTION 4 -->



### Exercise 5: Compute the integral image of image slice
Having found the ROI, compute the integral image over the reference image slice. Visualize the integral image along with the original reference image slice.


??? TIP "If you skipped the last exercise"
    You can find the crop we need using 
    ```py
    sliced_img = ref_img[14-1:17+2,4-1:7+2]
    ```

!!! INFO
    *It is important that the boundary of your integral image goes beyond the ROI by at least 1 pixel in all directions. cv2.integral fixes this for you by zero-padding the upper and left boundary, but keep this in mind when using numpy.cumsum - that is the reason we included the 1-pixel boundary*

??? TIP "Hint:" 
    The functions numpy.cumsum or cv2.integral may be of use, but you could also implement it exactly as done in the VJ paper.

<!-- START_SOLUTION 5 -->
??? tip "Solution 5"
    Now we compute the integral image using *np.cumsum*. We then visualize 
    ```py

    # Retrieve sliced image
    sliced_img = ref_img[y_min_ref-1:y_max_ref+2,x_min_ref-1:x_max_ref+2]

    # Integral image options
    # integral_img = cv2.integral(sliced_img)[1:,1:] # Pads boundary - hence we may remove the first row of y,x respectfully
    integral_img = np.cumsum(np.cumsum(sliced_img, axis=0), axis=1)

    # Visualize the sliced image along with its integral counterpart
    fig, axes = plt.subplots(ncols=2, figsize=(8,8))
    axes[0].imshow(sliced_img, cmap='gray')
    axes[0].set_title('Real image slice')
    axes[0].set_aspect('equal')
    img = axes[1].imshow(integral_img, cmap='viridis')
    axes[1].set_title('Cumulative sum image of image slice')
    axes[1].set_aspect('equal')

    # Write numbers on each pixel of cumsum image to gain extra insights
    for (i, j), val in np.ndenumerate(integral_img):
        axes[1].text(j, i, f"{val}", ha='center', va='center', color='white', fontsize=10)

    plt.tight_layout()
    fig.colorbar(img, label='Cumulative pixel value', ax=axes, shrink=0.38)
    plt.show()
    ```
<!-- END_SOLUTION 5 -->

### Exercise 6: Compute the Haar feature
Now, compute your Haar feature. To do so, extract the haar corners of each Haar region and compute their individual Haar sum. Finally, compute:

$$H = [\text{Sum of regions with white pixels}] - [\text{Sum of regions with black pixels}]$$

To retrieve the Haar feature. The following figure might become of use:

![Summed area table](figures/Summed_area_table.png)

Figure copied from [Wikipedia](https://en.wikipedia.org/wiki/Summed-area_table).

??? TIP "Hint:"
    You can make a function for evaluating a rectangle in an integral image based on its corner coordinates, a function for computing a diagonal type feature and a function for indexing start and stop values

<!-- START_SOLUTION 6 -->
??? tip "Solution 6"
    We're now ready to compute our Haar feature. We first need to separate Haar regions accordingly. In our case, we're working with a four rectangle diagonal Haar feature, where each rectangle is a square of side-length 2. We can find the corresponding starting positions (top left corner) of each rectangle as below:

    ```py
    # Retrieve w, h of the Haar regions
    w, h = integral_img.shape[1], integral_img.shape[0]
    w_rec, h_rec = w//2, h//2

    # As regions are standard uniform, we may retrieve start positions of each region like:
    start_positions = np.array([
        [(y, x) for x in np.linspace(start=1, stop=w_rec, num=2)]
        for y in np.linspace(start=1, stop=h_rec, num=2)]
    ).reshape(-1, 2)

    print(start_positions)
    ```
    Now, we'll add some functions, that will help locate the corners of each Haar region, compute their individual summed area and finally compute the 4 region Haar feature.

    ```py
    def compute_integral_image(integral_img, y0, x0, w, h):
        """
            Computes the integral image within the bounding box represented by (y0,x0,w,h).

            :param numpy.ndarray integral_img:      The integral image input. Dimensions need to supercede those of the bounding box.
            :param int y0:                          Initial starting y position of bounding box.
            :param int x0:                          Initial starting x position of bounding box.
            :param int w:                           Width of bounding box.
            :param int h:                           Height of bounding box.
        """
    
        # Retrieve corners of bounding box
        pos1, pos2, pos3, pos4 = (y0-1, x0-1), (y0-1, x0-1+w-1),(y0-1+h-1, x0-1), (y0-1+h-1, x0-1+w-1)

        # Compute integral image
        A = integral_img[pos1[0], pos1[1]]
        B = integral_img[pos2[0], pos2[1]]
        C = integral_img[pos3[0], pos3[1]]
        D = integral_img[pos4[0], pos4[1]]
        return D + A - B - C

    def compute_diagonal_feat(integral_img, start_positions, w_reg, h_reg):
        """
            Computes the diagonal Haar feature of a 
        """

        # Compute integral image for each square region
        regions = []
        for y,x in start_positions:
            reg_integral = compute_integral_image(integral_img, int(y), int(x), w_reg, h_reg)
            regions.append(reg_integral)
   
        # Compute Haar feature
        white_pixels = regions[1] + regions[2]
        black_pixels = regions[0] + regions[3]
        haar_feat =  white_pixels - black_pixels

        return haar_feat
    ```
    Using the function, we can compute the value of the Haar feature.
    ```py
    haar_feat = compute_diagonal_feat(integral_img, start_positions, w_rec, h_rec)
    print(f"The computed Haar feature of image slice = {haar_feat}")
    ```
<!-- END_SOLUTION 6 -->

### Exercise 7: Trying to relate a Haar feature to a single scale class instance 
Haar feature responses are used for training many weak classifiers in Viola-Jones style classification. Each stage compares the feature response to a threshold value. Therefore, larger feature responses (measured per absolute value) will in general have a larger tendency to play a role in the threshold decision. Therefore, it is informative to understand which sorts of image features produce a high and low values of the filter in order to gain an understanding of which image features may play a large role in the decision cascade. We have evaluated the same diagonal stage 3 Haar filter across the complete image **annotation_img.jpg** by sliding it spatially. While this is not what happens in the Haar cascade, it allows us to understand which image features are positive/negative at this exact scale. 

The output is shown below: 

![Spatially evaluated Haar feature response](figures/haar_feature_response.png)

Looking at the image, which image features translate to high vs. low values in the filter response? In which regions is the filter zero? How is the filter behaviour different from a standard edge-detector?

??? EXAMPLE "Code for producing output"
    ```py 
    import matplotlib.pyplot as plt
    ref_path = 'annotation_img.jpg'
    ref_img = io.imread(ref_path)
    ref_img = img_as_ubyte(rgb2gray(ref_img)) # Want the image to be grayscaled
    integral_img = cv2.integral(ref_img)[1:,1:]


    # Parameters for diagonal Haar (Total size 4x4)
    w_reg, h_reg = 2, 2 
    img_h, img_w = integral_img.shape

    # The heatmap size: 24x24 image - 4x4 feature + 1 = 22x22
    heatmap = np.zeros((img_h - 3, img_w - 3)) 

    for y in range(1, img_h - 3): # Start at 1 because compute_diagonal_feats expects y0-1
        for x in range(1, img_w - 3):
            
            # Define the start positions for the 4 internal squares 
            # based on the current top-left (y, x)
            # [Top-Left, Top-Right, Bottom-Left, Bottom-Right]
            current_starts = np.array([
                (y, x),             # Region 0 (Black)
                (y, x + w_reg),     # Region 1 (White)
                (y + h_reg, x),     # Region 2 (White)
                (y + h_reg, x + w_reg) # Region 3 (Black)
            ])
            print(current_starts)
            response = compute_diagonal_feat(integral_img, current_starts, w_reg, h_reg) #Haar response at location
            # Store in heatmap (adjusting indices for 0-based array)
            heatmap[y, x] = response

    plt.figure(figsize=(10, 5))
    plt.subplot(1, 2, 1)
    plt.title("Original Stop Sign (24x24)")
    plt.imshow(ref_img, cmap='gray')

    plt.subplot(1, 2, 2)
    plt.title("Haar Diagonal Feature Spatial Response")
    plt.imshow(heatmap, cmap='RdBu') # Red-Blue shows positive/negative peaks
    plt.colorbar()
    plt.tight_layout()
    plt.savefig("haar_feature_response.png")
    plt.show()
    ```

!!! NOTE "Interpretability in practice..."
    If you actually open the XML file for the classifier, you would find the following for this exact stage 3 feature:
    ```
    <_>
    <internalNodes>
        0 -1 8 6.9459797814488411e-03
    </internalNodes>
    <leafValues>
        -8.5354012250900269e-01 6.2478762865066528e-01
    </leafValues>
    </_>
    ``` 
    which means that the actual threshold for the decision rule is $x\geq 6.9459797814488411e-03\approx 0$. The leafValues indicate the value of $h_i(x)$ assigned based on the test (left or right), i.e. $h_i(x)=-0.854$ if $F_i<6.94e-03$. The stage-threshold for stage 3 is negative: -1.18, so in this exact case, a positive weak classifier output (right_val = +0.6248) pushes the stage score upward, increasing the chance that the window passes stage 3, and thus is not rejected.


## Part 3: Training your own object detector (**Optional**)
---
The stop-sign model was trained using data from Caltech-101, consisting of 64 training images with bounding box annotations. We'll now try to train our own object detector from the same dataset! The dataset contains 101 annotated object categories each with separate subset sizes, ranging from 40-800 images. You can download the dataset from [here](https://data.caltech.edu/records/mzrjq-6wc02). In order for everything to work as expected, you need to place the root of the Caltech-101 dataset into the **data** folder. 

In order to perform this exercise, we highly recommend using Anaconda. Training an object detector using tools provided by OpenCV requires an older version of OpenCV, which can be handled much easier using Anaconda.

### Exercise 8: Setup new virtual environment (**Optional**)
We first need to setup a new virtual environment, in order to run the tools we need, which have been disabled for newer versions of OpenCV (>3.4). To start with, if you're currently in a virtual environment, deactivate by running:

```bash
conda deactivate
```

Now we create a virtual environment with the necessary installation of OpenCV and activate it. This can be done as follows:

```bash
conda create -n obj_train python 'opencv>=3,<4' scipy scikit-image
conda activate obj_train
```

### Exercise 9: Preprocess data (**Optional**)
In order for our data to fit to the semantic notation needed for the OpenCV training tools, we first need to perform some pre-processing on our annotations. To do so, you need to choose **1)** a *positive* class, denoting the class you want your model to identify, which allows the model to learn relevant features; and **2)** a *negative* class, which is intended to teach the model relevant rejection features. In the case of the stop-sign model, the chosen positive class was *stop_sign* and the negative class was *car_side*.

Run the following command when at the root of the **process_dataset.py** script:

```bash
python process_dataset.py --pos <pos_class> --neg <neg_class>
```

With *pos_class* and *neg_class* being the explicit names of the categories in the Caltech-101 dataset (they must be present in the Annotations directory)

!!! TIP On which classes to choose 
    As the dataset is fairly small, please choose a positive class and negative class which are fairly different. 

Having run this script, the data files *pos_class_pos.dat* and *pos_class_neg.dat* should be contained in the Caltech-101 folder along with their gray-scaled datasets in the folders **pos_class_gray** and **neg_class_gray**.

### Exercise 10: Train object detector (**Optional**)
Now we'll commence the training portion of the exercise! To start with, *cd* into the Caltech-101 folder, as the following OpenCV tools only work from the folder where the data resides. We first need to create positive samples, which define what the model should look for when trying to find the object of interest. This will be handled by the OpenCV create_samples CLI tool, which creates a *positive vector file* holding the positive samples. This can be done as follows:

```bash
opencv_createsamples -info <pos_class>_pos.dat -vec positive.vec -w <width> -h <height>
```

!!! NOTE
    You may need to set an optional flag *-num N*, where N is the number of samples to generate. By default 1000 samples are created, however, if the number of bounding boxes in your dataset exeeds this amount, then you'll be prompted by an error.


Where the *-w* and *-h* parameters specify the width and height respectfully for the window size. You can just set them as 24 for now. From this, the file *positive.vec* should exist in your current working directory.

We may now start training our model. To do so, first make a directory to append the output files to. Run the following commands:
```bash
mkdir my_detector
opencv_traincascade -data my_detector -vec positive.vec -bg  <pos_class>_neg.dat -numPos <N> -numNeg <M> -numStages <S> -w <width> -h <height>
```

Here, the width and height arguments are the same that you used in the last command execution. M should be set as the number of negative samples available. 

!!! NOTE 
    If you are training the stop-sign detector, we recommend to use the following setup for training
    ```opencv_traincascade -data my_stop_sign_detector -vec positive.vec -bg  stop_sign_neg.dat -numPos 58  -numNeg 64 -numStages 18 -w 24 -h 24```


!!! NOTE 
    The -numPos parameter should be set to roughly 80-90% of the total samples in your .vec file. This creates a 'buffer' of extra images. During training, each stage must satisfy a minHitRate (default 0.995). If a specific positive sample is too 'hard' to be correctly classified by the current features while maintaining that rate, the trainer discards it and pulls a 'fresh' sample from the remaining buffer in the .vec file. If -numPos is set to the absolute maximum, the trainer has no flexibility; it must classify every single image perfectly, which often leads to the training 'crashing' or terminating prematurely when it hits a difficult sample.

For added context and potential extensions of the above, please refer to the documentation of the OpenCV tools *opencv_createsamples* and *opencv_traincascade* [here](https://gregorkovalcik.github.io/opencv_contrib/tutorial_traincascade.html).

After running the above, you should have a working Viola Jones object detection model! The model is named *cascade.xml* and is located in the folder **my_detector**.

Now, you're able to evaluate the performance of your own detector! Try to retrieve some images and videos from e.g. [Google.com](https://www.google.com/) or [Pexels.com](https://www.pexels.com/) and run the model evaluation script to see if it performs up to standard, or whether extra data is necessary to make a better model. Note that the image should be RGB. 

1. change your path to the ***data*** folder
2. Run: `python data/VJ_performance_eval.py --model <path/to/my_detector/cascade.xml> --input <path/to/your_image.jpg> --type image --w <width> --h <height>`


!!! INFO
    *If you want to visualize cascade classifiers on your own image annotation, look at the documentation of the beforementioned function [here](https://docs.opencv.org/3.4/dc/d88/tutorial_traincascade.html).*


## Part 4: Applied Viola-Jones on webcam stream 

Now that we've familiarized ourselves with the underlying workings of the Viola-Jones method, the remaining exercises intend to showcase how the algorithm can be applied in practice. To do so, we will employ some pretrained cascade classifiers, which can be found [here](https://github.com/opencv/opencv/tree/master/data/haarcascades). For the exercises, we'll need the classifiers:

- _haarcascade_frontalface_default.xml_ 
- _haarcascade_eye.xml_
- _haarcascade_eye_tree_eyeglasses.xml_

Save them somewhere where you can easily define the path for loading, e.g. in a folder called **pretrained_classifiers**.

In this first part, you'll familiarize yourself with the tools you'll use in [Part 5](#part-5-snapchat-like-object-filters), where you'll create your own _Snapchat_ like filter!

??? NOTE "Automatic download for Mac and linux users"
    In the exercise [data/](../../downloads/material-10.zip) folder, there is a script you can use to download all the classifiers called download_classifiers.sh. 
    To do so, do the following two steps in a terminal with your root as the exercise root: 
    ```
    cd your_exercise_10_root 
    chmod +x data/download_classifiers.sh
    ./data/download_classifiers.sh
    ```

    If you experience any issues, simply download them manually. 

!!! WARNING "Warning: Lack of model detections"
    For the exercises in [Part 4](#part-4-warming-up-and-gaining-familiarity) and [Part 5](#part-5-snapchat-like-object-filters), you are encouraged to use your webcam. As the classifier models are highly susceptible to poor lighting conditions, e.g. produced by directional lighting, it might be difficult for it to detect eyes and faces.

    If this is the case, in order for you to still be able to go through with the exercises, you have two options:

    1. Relocate to somewhere with better lighting (brighter and/or diffused lighting),
    2. Instead of using your webcam, you may use a video instead. For this, we've provided a video _speech_snippet.mp4_ at the root of the [exercise data](../../downloads/material-10.zip). Set the videopath as value of the `VIDEO_FILE` variable.

### Exercise 11: using a pretrained classifier on web-cam feed 
Start out by opening the file *part_1_viola_jones.py*. Change the variable `pretrained_dir` such that it points to your pretrained classifiers and gain a quick overview of the script.

??? EXAMPLE "Code Template for exercises 11-13"
    The contents of the file part_one_viola_jones.py are roughly: *

    ```python
    from __future__ import print_function
    from typing import Tuple
    import time
    import cv2

    """
    An adapted version of https://docs.opencv.org/3.4/db/d28/tutorial_cascade_classifier.html
    """

    def connect_camera(use_droid_cam: bool = False) -> cv2.VideoCapture:
        Attempts to connect to the webcam. 
        """
        If this fails, prints an error message and exits the complete script. 

        Args:
            use_droid_cam (bool, optional): whether to use the android camera, as done in video change detection exercises, ex2b. Defaults to False.

        Returns:
            cv2.VideoCapture: the opencv camera object which we can pull frames from
        """
        print("Opening connection to camera")
        url = 0
        use_droid_cam = False
        if use_droid_cam:
            url = "http://192.168.1.120:4747/video"
        cap = cv2.VideoCapture(url)
        if not cap.isOpened():
            print("Cannot open camera")
            exit()

        ret, _ = cap.read()
        if not ret:
            print("---(!)Error reading frame from webcam")
            exit()
        print("Successfully connected to camera capturing device")
        return cap 

    def load_cascades(cascade_directory: str) -> Tuple[cv2.CascadeClassifier, cv2.CascadeClassifier]:
        """
        A function to load a the pretrained opencv cascade detectors (Viola-Jones-type)
        Remember to download the classifiers beforehand (https://raw.githubusercontent.com/opencv/opencv/3.4/data/haarcascades/)

        Args:
            cascade_directory (str): the directory the classifier .xml files are in

        Returns:
            tuple[cv2.CascadeClassifier]: a tuple of the two classifiers
        
        """
        face_cascade = cv2.CascadeClassifier() #initialize a classifier instance
        eyes_cascade = cv2.CascadeClassifier()
        
        face_cascade_name = cascade_directory + "/haarcascade_frontalface_default.xml"
        eyes_cascade_name = cascade_directory + "/haarcascade_eye.xml" #specify which classifiers to load
        
        
        if not face_cascade.load(cv2.samples.findFile(face_cascade_name)):
            print('--(!)Error loading face cascade')
            exit(0)
        if not eyes_cascade.load(cv2.samples.findFile(eyes_cascade_name)):
            print('--(!)Error loading eyes cascade')
            exit(0)

        print("Loaded cascade classifiers")
        return face_cascade, eyes_cascade


    if __name__=="__main__":
        ###step 1: connect to webcam
        USE_DROID_CAM = False 
        cap = connect_camera(USE_DROID_CAM)
        
        ###step 2: load classifiers, remember to download them beforehand 
        pretrained_dir = "data/pretrained_classifiers"
        face_cascade, eyes_cascade = load_cascades(pretrained_dir)
        
        # To keep track of frames per second
        start_time = time.time()
        font = cv2.FONT_HERSHEY_COMPLEX
        n_frames = 0

        #make the loop
        stop = False

        while not stop:
            ret, frame = cap.read() #read rgb frame
            if frame is None:
                print('--(!) No captured frame -- Break!')
                break
        
            frame_gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY) #convert to grayscale
            frame_gray = cv2.equalizeHist(frame_gray) #see pp 42-44 in MIA book. Necessary because the pre-trained classifiers were trained on this
            

            faces = face_cascade.detectMultiScale(frame_gray) #face detection
            for (x,y,w,h) in faces:
                center = (x + w//2, y + h//2)
                frame = cv2.ellipse(frame, center, (w//2, h//2), 0, 0, 360, (255, 0, 255), 4)
                faceROI = frame_gray[y:y+h,x:x+w]
                eyes = eyes_cascade.detectMultiScale(faceROI) #constrain eye-detection to be within the face region
                for (x2,y2,w2,h2) in eyes:
                    eye_center = (x + x2 + w2//2, y + y2 + h2//2)
                    radius = int(round((w2 + h2)*0.25))
                    frame = cv2.circle(frame, eye_center, radius, (255, 0, 0 ), 4)
            
            # Keep track of frames-per-second (FPS)
            n_frames = n_frames + 1
            elapsed_time = time.time() - start_time
            fps = int(n_frames / elapsed_time)

            """
            exercise 2) count the number of detected faces and eyes
            """
            N_faces = -1
            N_eyes = -1

            # Put the information on the image frame: FPS, number of faces, number of eyes 
            str_out = f"fps: {fps}, N_f: {N_faces}, N_e: {N_eyes}"
            cv2.putText(frame, str_out, (100, 100), font, 1, 255, 1)
            cv2.imshow('Capture - Face detection', frame)

            if cv2.waitKey(1) == ord('q'):
                stop = True
    ```

**Question 6**: *Which functions does what? Where is the detection actually happening?*

Now run the script and try moving a bit around in front of the webcam, e.g. by rotating your head, looking to the sides, and changing your distance to the screen.

**Question 7**: *When does the detection work well, and can you tell why the model has deficiencies?* 

### Exercise 12: Counting the number of detections
Implement functionality to count the number of faces and eyes detected. This should be based on the output of the `detectMultiScale` function calls, i.e. the variables faces and eyes. Print the number of eyes and faces detected into the text which already measures the frames per second. Ensure that it works correctly when multiple faces with eyes are detected.    

??? TIP "Hint:" 
    In order to figure out what the function returns, try to write print(faces) when a face is detected vs when you put a finger in front of the webcam. Is the return type consistent in the two cases?
 
??? TIP "Hint 2:"
    ***len()*** works for numpy arrays, python lists and tuples.

<!-- START_SOLUTION 12 -->
??? tip "Solution 12"

    ```py
    #-- Detect faces
    faces = face_cascade.detectMultiScale(frame_gray) #is a np.array (N,4)

    N_faces = len(faces)
    N_eyes = 0
    for (x,y,w,h) in faces:
        center = (x + w//2, y + h//2)
        frame = cv2.ellipse(frame, center, (w//2, h//2), 0, 0, 360, (255, 0, 255), 4)
        faceROI = frame_gray[y:y+h,x:x+w]
        #-- In each face, detect eyes
        eyes = eyes_cascade.detectMultiScale(faceROI,minSize = (w//7,h//7), maxSize = (w//4,h//4)) #optimal parameters depend on distance to camera, lighting etc. 
        N_eyes += len(eyes)
        for (x2,y2,w2,h2) in eyes:
            eye_center = (x + x2 + w2//2, y + y2 + h2//2)
            radius = int(round((w2 + h2)*0.25))
            frame = cv2.circle(frame, eye_center, radius, (255, 0, 0 ), 4)
    ```
<!-- END_SOLUTION 12 -->

### Exercise 13: Decreasing false positive and false negative detections
You may notice a number of false positive detections, especially for the eye detection. Likewise, if you wear glasses, you may experience some false negatives. 
Let's try to improve upon that. There are two quick fixes we will try out:

1. Check out the `detectMultiScale()` documentation [here](https://docs.opencv.org/master/d1/de5/classcv_1_1CascadeClassifier.html). We can restrict detections to be larger than a certain size. Try to tune the size such that nostrils are no longer detected. You can also set a `maxSize`, if many detections are larger than what you expect eyes would be. Both should ideally be set adaptively based on the height and width of the face detected. 

2. Using another trained detection model. We could e.g. try the *haarcascade_eye_tree_eyeglasses.xml* model instead, which is trained to be more robust. Change the function `load_cascades` so that you use the eyeglasses-model instead and see if this improves robustness. Try comparing the quality when the `minSize` and `maxSize` arguments are set vs. not set. 

You could also implement these changes into the face detection, the quality of which also is based on lighting conditions, etc.



<!-- START_SOLUTION 11-13 -->
??? tip "Solution 11-13"
    Run from jupyter by executing solution file: 

    ```py
    !python part_4_viola_jones_solution.py
    ```

    in a code block.
    The solution file contains:

    ```py
    from __future__ import print_function
    from typing import Tuple
    import time
    import cv2

    """
    An adapted version of https://docs.opencv.org/3.4/db/d28/tutorial_cascade_classifier.html
    """

    def connect_camera(video_file: str = "", use_droid_cam: bool = False) -> cv2.VideoCapture:
        """
        Attempts to connect to the webcam. 
        If this fails, prints an error message and exits the complete script. 

        Args:
            use_droid_cam (bool, optional): whether to use the android camera, as done in video change detection exercises, ex2b. Defaults to False.

        Returns:
            cv2.VideoCapture: the opencv camera object which we can pull frames from
        """
        print("Opening connection to camera")
        if video_file != "":
            cap = cv2.VideoCapture(video_file)

        else:
            url = 0
            use_droid_cam = False
            if use_droid_cam:
                url = "http://192.168.1.120:4747/video"
            cap = cv2.VideoCapture(url)

        if not cap.isOpened():
            print("Cannot open camera")
            exit()

        ret, _ = cap.read()
        if not ret:
            print("---(!)Error reading frame from webcam")
            exit()
        print("Successfully connected to camera capturing device")
        return cap 

    def load_cascades(cascade_directory: str) -> Tuple[cv2.CascadeClassifier, cv2.CascadeClassifier]:
        """
        A function to load a the pretrained opencv cascade detectors (Viola-Jones-type)
        Remember to download the classifiers beforehand (https://raw.githubusercontent.com/opencv/opencv/3.4/data/haarcascades/)

        Args:
            cascade_directory (str): the directory the classifier .xml files are in

        Returns:
            tuple[cv2.CascadeClassifier]: a tuple of the two classifiers
    
        """
        face_cascade = cv2.CascadeClassifier() #initialize a classifier instance
        eyes_cascade = cv2.CascadeClassifier()
    
        face_cascade_name = cascade_directory + "/haarcascade_frontalface_default.xml"
        #eyes_cascade_name = pretrained_dir + "/haarcascade_eye.xml" #specify which classifiers to load
        eyes_cascade_name = cascade_directory + "/haarcascade_eye_tree_eyeglasses.xml" #more robust to lighting and glasses
    
    
        if not face_cascade.load(cv2.samples.findFile(face_cascade_name)):
            print('--(!)Error loading face cascade')
            exit(0)
        if not eyes_cascade.load(cv2.samples.findFile(eyes_cascade_name)):
            print('--(!)Error loading eyes cascade')
            exit(0)

        print("Loaded cascade classifiers")
        return face_cascade, eyes_cascade

    if __name__=="__main__":
        ###step 1: connect to webcam
        USE_DROID_CAM = False 
        VIDEO_FILE = ""
        cap = connect_camera(video_file=VIDEO_FILE, use_droid_cam=USE_DROID_CAM)
    
        ###step 2: load classifiers, remember to download them beforehand 
        pretrained_dir = "data/pretrained_classifiers"
        face_cascade, eyes_cascade = load_cascades(pretrained_dir)
    
        # To keep track of frames per second
        start_time = time.time()
        font = cv2.FONT_HERSHEY_COMPLEX
        n_frames = 0

        #make the loop
        stop = False

        while not stop:
            ret, frame = cap.read() #read rgb frame
            if frame is None:
                print('--(!) No captured frame -- Break!')
                break
    
            frame_gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY) #convert to grayscale
            frame_gray = cv2.equalizeHist(frame_gray) #see pp 42-44 in MIA book. Necessary because the pre-trained classifiers were trained on this
        
            #< START_SOLUTION 12 >
            #-- Detect faces
            faces = face_cascade.detectMultiScale(frame_gray) #is a np.array (N,4)
        
            N_faces = len(faces)
            N_eyes = 0
            for (x,y,w,h) in faces:
                center = (x + w//2, y + h//2)
                frame = cv2.ellipse(frame, center, (w//2, h//2), 0, 0, 360, (255, 0, 255), 4)
                faceROI = frame_gray[y:y+h,x:x+w]
                #-- In each face, detect eyes
                eyes = eyes_cascade.detectMultiScale(faceROI,minSize = (w//7,h//7), maxSize = (w//4,h//4)) #optimal parameters depend on distance to camera, lighting etc. 
                N_eyes += len(eyes)
                for (x2,y2,w2,h2) in eyes:
                    eye_center = (x + x2 + w2//2, y + y2 + h2//2)
                    radius = int(round((w2 + h2)*0.25))
                    frame = cv2.circle(frame, eye_center, radius, (255, 0, 0 ), 4)
            #< END_SOLUTION 12 >
        
            # Keep track of frames-per-second (FPS)
            n_frames = n_frames + 1
            elapsed_time = time.time() - start_time
            fps = int(n_frames / elapsed_time)

            # Put the information on the image frame: FPS, number of faces, number of eyes 
            str_out = f"fps: {fps}, N_f: {N_faces}, N_e: {N_eyes}"
            cv2.putText(frame, str_out, (100, 100), font, 1, 255, 1)
            cv2.imshow('Capture - Face detection', frame)

            if cv2.waitKey(1) == ord('q'):
                stop = True
    ```
<!-- END_SOLUTION 11-13 -->

## Part 5: Snapchat-like object filters 

We've already seen that we can detect eyes and faces somewhat robustly. Now, we will use this knowledge to make a Snapchat-like filter, where an object is placed on an image (the webcam/video feed) on an anchor-point (e.g. the top of the head). The goal is, that the object should track the anchor-point on the head through successive frames. 

In order to achieve this, we will need to be able to load an image with either a transparent background (RGBA-image), or alternatively load an image which we can make a mask for which could achieve the same end.

### Exercise 14: loading an RGBA image
Open the file *part_2_viola_jones_snapchat.py* in the ***data/*** directory. Finish the function `load_png_object` and use it to load the hat object which has the path *image_props/hat_rescaled.png*. Pass the flag `cv2.IMREAD_UNCHANGED`, which forces _OpenCV_ to load the image using an extra alpha-channel. Return the image and the image shape. Test that the shape is as you expect.

??? FAILURE "No hat is displayed"
    In order to render the hat initially, the eye model needs to detect two eyes. After this, the hat will persistently be displayed at a dummmy position.

**Question 8**: *How can you interpret the alpha-channel?*

<!-- START_SOLUTION 14 -->
??? tip "Solution 14"

    ```py
    def load_png_object(file_path: str) -> Tuple[np.ndarray, Tuple[int, int, int]]:
        """
        Loads a png object as RGBA. Note, the .png should have a transparent background and be loaded with an alpha channel

        Args:
            file_path (str): relative path to the .png file 

        Returns:
            Tuple[np.ndarray, Tuple[int, int, int]]: the image array of shape (H, W, C) and a tuple (H, W, C) where C=4 for RGBA image 
        """

        im = cv2.imread(file_path,cv2.IMREAD_UNCHANGED)
        im_dims = im.shape 

        print(f"loaded object: {file_path} with shape: {im_dims}")
        return im, im_dims
    ```
<!-- END_SOLUTION 14 -->


### Exercise 15: Eye-centre landmark registration 
For the approach we will use for visualizing the hat on top of the head, we will use a simplified landmark-based approach. In order to place the hat correctly, we will need to infer three landmarks:


1. The centre coordinate of the left eye detection;

2. The centre coordinate of the right eye detection; and

3. The centre coordinate of the top of the detected face.

For each detected face and pair of eyes, we will need to save these coordinates.

All landmarks we need to infer are defined in the following loop:

```python 
for i, (x,y,w,h) in enumerate(faces):
            center = (x + w//2, y + h//2)
            face_center = np.array([center[0],center[1]])
            frame = cv2.ellipse(frame, center, (w//2, h//2), 0, 0, 360, (255, 0, 255), 4)
            faceROI = frame_gray[y:y+h,x:x+w]
            #detect eyes
            eyes = eyes_cascade.detectMultiScale(faceROI)
            N_eyes = len(eyes)
            eye_center_holder = []
            if N_eyes==2: #only go further if we have detected two eyes.. 
                eye_coord_arr = np.zeros((2,2))
                for i, (x2,y2,w2,h2) in enumerate(eyes):
                    eye_center = (x + x2 + w2//2, y + y2 + h2//2)
                    radius = int(round((w2 + h2)*0.25))
                    frame = cv2.circle(frame, eye_center, radius, (255, 0, 0 ), 4)
                    eye_coord_arr[i,:] = np.array([eye_center[0],eye_center[1]]) #x, y in frame coordinates
```
Which can be found in the reference file.

First, we will start by detecting the left and right eye centre coordinates.
If we work from a simplified assumption that the right eye will be located to the right of the left eye, we can identify the centre coordinates based on the contents of `eye_coord_arr`. 

Your task is to finish the function `assign_eyes` which should output the left and right eye centre coordinates as numpy arrays.
<!-- START_SOLUTION 15 -->
??? tip "Solution 15"

    ```py
    def assign_eyes(coords: np.ndarray) -> tuple[np.ndarray,np.ndarray]:
        """
        Assign left and right eye based on location in frame

        Args:
            eye_coords (np.ndarray): eye coordinate array where each row corresponds to x,y centre coordinates, shape (N, 2) (in current implementation of main loop, N=2)

        Returns:
            tuple[np.ndarray,np.ndarray]: left eye centre coordinates, right eye centre coordinates
        """
        #Example solution: 
        right_eye_idx = np.argmax(coords[:,0]) #actually we should use the pair which has the maximal horizontal distance
        right_eye = coords[right_eye_idx,:]
        left_eye = coords[1-right_eye_idx,:]

        if left_eye[0] > right_eye[0]:
            left_eye, right_eye = right_eye, left_eye

        return left_eye, right_eye
    ```
<!-- END_SOLUTION 15 -->

### Exercise 16: Constructing a normal vector for finding the top of the face 
Having found the left and right eye centre coordinates, we now want to identify the top of the head. This can examplewise be done by finding a vector which always points upwards in the face coordinate system. This can be achieved by first finding the vector which goes from the left eye to the right eye $\vec{P}$, and followingly finding its normalized normal vector, $\vec{n}_{\vec{P}}$. 

Your task is to: 

1. Calculate the vector going from left eye to right eye, $\vec{P}$

2. Find the corresponding normal vector, $\vec{n}_{\vec{P}}$

3. Calculate the normalizing factor such that it is a unit-norm normal vector
   
The normalizing factor can be found by using `np.linalg.norm(n_vec)`.

!!! NOTE
    In some cases, the CascadeClassifier registers multiple eye instances in the same eye. In this case, the norm will be zero, and normalization will yield an error. As a result, we will in this case not proceed further with calculating the normalized normal vector.
<!-- START_SOLUTION 16 -->
??? tip "Solution 16"

    ```py
    vec = right_eye - left_eye
    ###find the normal vector - to find the translation we need  
    n_vec = np.array([-vec[1],vec[0]])
    norm = np.linalg.norm(n_vec)
    ```
<!-- END_SOLUTION 16 -->

### Exercise 17: Identifying the coordinate at the top-middle of the face
If we from the centre coordinate of the face go along the normal vector $\vec{n}_{\vec{P}}$ along the length $\frac{h}{2}$, where $h$ is the height of the face, we should reach the top coordinate of the face. This will yield our third landmark.

Calculate the end-point on the face. Ensure that the output is an integer, and if not, round it to an integer. 
You can debug your solution with: 
```python
cv2.circle(frame,(end_point_hat_x,end_point_hat_y),10, color=(0,255,0),thickness=3, lineType=8, shift=0) #blue circle - top of face 
``` 

which inserts a circle at the coordinate you've found.
<!-- START_SOLUTION 17 -->
??? tip "Solution 17"

    ```py
    end_point_hat_x = int(face_center[0] - n_vec[0]*w//2)
    end_point_hat_y = int(face_center[1] - n_vec[1]*h//2) #subtract because image coordinate system
    ```
<!-- END_SOLUTION 17 -->

### Exercise 18: Transformations!
A Snapchat filter resizes the objects such that they somewhat fit with the width of the head, so we will do the same - currently, it is way too small! Finish the function `rescale_object` such that the hat somewhat fits on the face. Use `skimage.transforms.rescale` for this. 
<!-- START_SOLUTION 18 -->
??? tip "Solution 18"

    ```py
    def rescale_object(object_im: np.ndarray, face_width: int) -> np.ndarray: 
        """
        Rescales the object image to a specific width so it can match face width. 

        Args:
            object_im (np.ndarray): object image, shape (H,W,C)
            face_width (int): the detected face width

        Returns:
            np.ndarray: the rescaled object image

        """
        """
        Solution exercise 8: change the width_scale_factor such that the function adaptively fits the object onto the head
        """
        width_scale_factor = face_width/object_im.shape[1]
        out_im = rescale(object_im, width_scale_factor,channel_axis=-1, order=1)
        return out_im
    ```
<!-- END_SOLUTION 18 -->

### Exercise 19: Place the hat on the coordinate
In order to place the hat, we need to identify which coordinate in the hat coordinate frame we wish to align with the found `end_point_hat_x` and `end_point_hat_y`. In other words, we need to find the necessary translation between the hat coordinate frame (local) and the webcam/video image coordinate frame (global). We save these coordinates in format $y,x$ in the `anchor_point` variable. 
Try defining a suitable alignment point and visualizing the results by running the script.

!!! NOTE
    The transformation is carried out in the `insert_hat_on_frame` function, where the variables `slice_start_y` and `slice_start_x`:
    ```python
    def insert_hat_on_frame(frame_rgb, obj, coords, anchor):
        # coords = where anchor should go in the rgb frame, i.e. face coordinate position
        target_y, target_x = coords
        anchor_y, anchor_x = anchor  # anchor inside the hat image

        # top-left corner on frame where hat-image should start
        slice_start_y = int(target_y - anchor_y) 
        slice_start_x = int(target_x - anchor_x)
    ```

define at which global coordinate the hat image $(0,0)$ should be located. 

??? TIP "Hint"
    A good coordinate could be the bottom-centre of the hat image.

<!-- START_SOLUTION 19 -->
??? tip "Solution 19"

    ```py
    #anchor point calculation: bottom centre of hat 
    anchor_point = np.array([object_im_trf.shape[0]-1,object_im_trf.shape[1]//2]) #y,x always
    ```
<!-- END_SOLUTION 19 -->

### Exercise 20: Rotating for realism
Based on our normal-vector $\vec{n}_{\vec{P}}$, we can even calculate the corresponding angle the hat should be rotated to follow the face rotation. The angle of rotation is measured in counter-clockwise direction compared to the horizontal axis, and as such can be calculated using the normal-vector. 

Set the boolean `ROTATE` flag as `True` and implement the calculation of the rotation angle. Store the result in `theta`, and ensure it is in degrees. 
The `rotate_object` function rotates the object using the `anchor_point` as rotation centre with the `skimage.transform.rotate` function we've worked with before. Use the center bottom point of the object image as the centre coordinate, i.e.: 
```python
anchor_point = np.array([object_im_trf.shape[0]-1,object_im_trf.shape[1]//2])
```

The function outputs the position of the rotation-centre post transformation aswell. 

!!! NOTE "Rotation limitations"
    For larger rotation angles the object will become cropped. If you set the flag allow_resize=True, the rotated object image is resized such that no information is lost. This adds a layer of complexity, as we in this case need to correct for the resizing of the object as well. This is handled by the variables dy and dx. Only run it with resize=False. 

<!-- START_SOLUTION 20 -->
??? tip "Solution 20"

    ```py
    theta = np.arctan2(-vec[1],vec[0]) #is in radians
    theta *= 180/np.pi
    ```
<!-- END_SOLUTION 20 -->

<!-- START_SOLUTION 14-20 -->
??? tip "Solution 14-20"
    Run from jupyter by executing solution file: 

    ```py
    !python part_5_viola_jones_snapchat_solutions.py
    ```

    in a code block.
    The solution file contains:

    ```py
    from __future__ import print_function
    from typing import Tuple
    import cv2
    import time
    import numpy as np 
    from skimage.transform import rotate, rescale

    """
    An adapted version of https://docs.opencv.org/3.4/db/d28/tutorial_cascade_classifier.html

    Experimental: 
        - No support for resize=True. 
        - Also no optimizations introduced yet. 
    """

    def connect_camera(video_file: str = "", use_droid_cam: bool = False) -> cv2.VideoCapture:
        """
        Attempts to connect to the webcam. 
        If this fails, prints an error message and exits the complete script. 

        Args:
            use_droid_cam (bool, optional): whether to use the android camera, as done in video change detection exercises, ex2b. Defaults to False.

        Returns:
            cv2.VideoCapture: the opencv camera object which we can pull frames from
        """
        print("Opening connection to camera")
        if video_file != "":
            cap = cv2.VideoCapture(video_file)

        else:
            url = 0
            use_droid_cam = False
            if use_droid_cam:
                url = "http://192.168.1.120:4747/video"
            cap = cv2.VideoCapture(url)

        if not cap.isOpened():
            print("Cannot open camera")
            exit()

        ret, _ = cap.read()
        if not ret:
            print("---(!)Error reading frame from webcam")
            exit()
        print("Successfully connected to camera capturing device")
        return cap 

    def load_cascades(cascade_directory: str) -> Tuple[cv2.CascadeClassifier, cv2.CascadeClassifier]:
        """
        A function to load a the pretrained opencv cascade detectors (Viola-Jones-type)
        Remember to download the classifiers beforehand (https://raw.githubusercontent.com/opencv/opencv/3.4/data/haarcascades/)

        Args:
            cascade_directory (str): the directory the classifier .xml files are in

        Returns:
            tuple[cv2.CascadeClassifier]: a tuple of the two classifiers
    
        """
        face_cascade = cv2.CascadeClassifier() #initialize a classifier instance
        eyes_cascade = cv2.CascadeClassifier()
    
        face_cascade_name = cascade_directory + "/haarcascade_frontalface_default.xml"
        eyes_cascade_name = cascade_directory + "/haarcascade_eye_tree_eyeglasses.xml" #seems to be more robust to lighting and has less false detections
    
    
        if not face_cascade.load(cv2.samples.findFile(face_cascade_name)):
            print('--(!)Error loading face cascade')
            exit(0)
        if not eyes_cascade.load(cv2.samples.findFile(eyes_cascade_name)):
            print('--(!)Error loading eyes cascade')
            exit(0)

        print("Loaded cascade classifiers")
        return face_cascade, eyes_cascade

    #< START_SOLUTION 14 >
    def load_png_object(file_path: str) -> Tuple[np.ndarray, Tuple[int, int, int]]:
        """
        Loads a png object as RGBA. Note, the .png should have a transparent background and be loaded with an alpha channel

        Args:
            file_path (str): relative path to the .png file 

        Returns:
            Tuple[np.ndarray, Tuple[int, int, int]]: the image array of shape (H, W, C) and a tuple (H, W, C) where C=4 for RGBA image 
        """

        im = cv2.imread(file_path,cv2.IMREAD_UNCHANGED)
        im_dims = im.shape 
    
        print(f"loaded object: {file_path} with shape: {im_dims}")
        return im, im_dims
    #< END_SOLUTION 14 >

    def insert_hat_on_frame(frame_rgb, obj, coords, anchor):
        frame = frame_rgb.copy()
    
        Hf, Wf = frame.shape[:2]
        Hh, Wh = obj.shape[:2]

        hat_mask = obj[:, :, 3] > 0 #alpha "not transparent" mask
        hat_rgb  = obj[:, :, :3]

        # coords = where anchor should go in the rgb frame, i.e. face coordinate position
        target_y, target_x = coords
        anchor_y, anchor_x = anchor  # anchor inside the hat image

        # top-left corner on frame where hat-image should start
        slice_start_y = int(target_y - anchor_y)
        slice_start_x = int(target_x - anchor_x)

        # ---- STEP 1: handle negative object starts (crop top/left) ----
        crop_x_left = max(0, -slice_start_x)
        crop_y_top  = max(0, -slice_start_y)

        x1 = max(0, slice_start_x)
        y1 = max(0, slice_start_y)

        # ---- STEP 2: compute end positions before cropping bottom/right ----
        x2 = x1 + (Wh - crop_x_left)
        y2 = y1 + (Hh - crop_y_top)

        # ---- STEP 3: crop bottom/right if object is going outside frame in bottom/right ----
        crop_x_right = max(0, x2 - Wf)
        crop_y_bottom = max(0, y2 - Hf)

        # Compute final slice ranges after all cropping
        x2 = x2 - crop_x_right
        y2 = y2 - crop_y_bottom

        # Corresponding crop on hat image
        hat_crop = hat_rgb[crop_y_top:Hh - crop_y_bottom,
                           crop_x_left:Wh - crop_x_right]

        mask_crop = hat_mask[crop_y_top:Hh - crop_y_bottom,
                             crop_x_left:Wh - crop_x_right]

        # ---- STEP 4: insert ----
        im_mask = np.zeros((Hf, Wf), dtype=bool)
        full_hat = np.zeros_like(frame)

        im_mask[y1:y2, x1:x2] = mask_crop
        full_hat[y1:y2, x1:x2] = hat_crop

        frame[im_mask] = full_hat[im_mask]

        return frame

    def time_smoothen_detections(arr_new: np.ndarray | float, arr_ref: np.ndarray | float, alpha: float = 0.95) -> np.ndarray | float:
        """
        Time smoothen detections. Hint: equation 4 in video change detection notes.

        Args:
            arr_new (np.ndarray or float): the input array or value to be smoothened using reference. If array shape (N,features)
            arr_ref (np.ndarray or float): the reference array or value to use for smoothing. If array shape (N,features)
            alpha (float, optional): smoothing-factor. Defaults to 0.95.

        Returns:
            np.ndarray or float: the smoothened array or value 
        """
        return (alpha*arr_ref + (1-alpha)*arr_new)
    

    def rotate_object(object_im: np.ndarray, angle: float, anchor_point: tuple, allow_resize: bool = False) -> Tuple[np.ndarray,np.ndarray,int,int] | Tuple[np.ndarray,np.ndarray]:
        """
        Rotates an image using skimage.transforms.rotate around the anchor_point. 
        Finds the corresponding location of the anchor point in the rotated image frame. 
        Returns the difference in image size before and after rotation, dx and dy.  
    
        Args:
            object_im (np.ndarray): the object image to be rotated, (H, W, C)
            angle (float): rotation angle in counter-clockwise direction in degrees. 
            anchor_point (tuple): the centre in object image used rotation. For this implementation, use the bottom centre. 

        Returns:
            Tuple[np.ndarray,np.ndarray,int,int] 
        """
        h, w = object_im.shape[0], object_im.shape[1]
        object_im = rotate(object_im, angle, center=(anchor_point[1],anchor_point[0]),order=1, resize=allow_resize)
    
        #we need to know the anchor point post-translation and resizing. Easiest way is to rotate a mask containing the anchor point with the same transform used for the image 
        anchor_mask = np.zeros((h, w), dtype=np.uint8)
        anchor_mask[anchor_point] = 1
        anchor_rot = rotate(anchor_mask, angle, center=(anchor_point[1],anchor_point[0]), resize=allow_resize, order=0, preserve_range=True)
        #find the new anchor coordinate in the rotated image. This is the coordinate which needs to be mapped to the top centre of head.
        new_anchor = np.argwhere(anchor_rot == 1)[0]   # row, col in rotated space -> y, x 
    
        #calculate the difference in sizes between original image and rotated image. Used for correcting position on top of head if resize is true.
        delta_width = w-object_im.shape[1]
        delta_height = h-object_im.shape[0]
    
        return object_im, new_anchor, delta_width, delta_height 
    
    
    #< START_SOLUTION 18 >
    def rescale_object(object_im: np.ndarray, face_width: int) -> np.ndarray: 
        """
        Rescales the object image to a specific width so it can match face width. 

        Args:
            object_im (np.ndarray): object image, shape (H,W,C)
            face_width (int): the detected face width

        Returns:
            np.ndarray: the rescaled object image

        """
        """
        Solution exercise 8: change the width_scale_factor such that the function adaptively fits the object onto the head
        """
        width_scale_factor = face_width/object_im.shape[1]
        out_im = rescale(object_im, width_scale_factor,channel_axis=-1, order=1)
        return out_im
    #< END_SOLUTION 18 >
    
    #< START_SOLUTION 15 >
    def assign_eyes(coords: np.ndarray) -> tuple[np.ndarray,np.ndarray]:
        """
        Assign left and right eye based on location in frame

        Args:
            eye_coords (np.ndarray): eye coordinate array where each row corresponds to x,y centre coordinates, shape (N, 2) (in current implementation of main loop, N=2)

        Returns:
            tuple[np.ndarray,np.ndarray]: left eye centre coordinates, right eye centre coordinates
        """
        #Example solution: 
        right_eye_idx = np.argmax(coords[:,0]) #actually we should use the pair which has the maximal horizontal distance
        right_eye = coords[right_eye_idx,:]
        left_eye = coords[1-right_eye_idx,:]

        if left_eye[0] > right_eye[0]:
            left_eye, right_eye = right_eye, left_eye
    
        return left_eye, right_eye
    #< END_SOLUTION 15 >


    if __name__=="__main__":
        ###step 1: connect to webcam
        USE_DROID_CAM = False 
        VIDEO_FILE = ""
        cap = connect_camera(video_file=VIDEO_FILE, use_droid_cam=USE_DROID_CAM)
    
        ###step 2: load classifiers 
        pretrained_dir = "data/pretrained_classifiers"
        face_cascade, eyes_cascade = load_cascades(pretrained_dir)
    
        # To keep track of frames per second
        start_time = time.time()
        font = cv2.FONT_HERSHEY_COMPLEX

        object_im, object_dims = load_png_object("data/image_props/hat_rescaled.png")
        object_im_trf = object_im.copy()
       
        #set loop parameters
        n_frames = 0
        ROTATE = True #set to true if we want to rotate the object of interest with an angle 
        SMOOTHEN = True 
        stop = False
        N_eyes = 0
        N_skipped = 0 #for counting how many frames we do not have any detections
    
        #initialize for fallback when no detections are present
        object_im_trf = None
        end_point_hat_y = None
        end_point_hat_x = None 
        anchor_point = None 

        while not stop:
            ret, frame = cap.read()
            if frame is None:
                print('--(!) No captured frame -- Break!')
                break
            
            frame_gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY) #convert to grayscale
            frame_gray = cv2.equalizeHist(frame_gray) 
            faces = face_cascade.detectMultiScale(frame_gray) #is a np.array (N,4) or tuple
            N_faces = len(faces)
            for i, (x,y,w,h) in enumerate(faces):
                center = (x + w//2, y + h//2)
                face_center = np.array([center[0],center[1]])
                frame = cv2.ellipse(frame, center, (w//2, h//2), 0, 0, 360, (255, 0, 255), 4)
                faceROI = frame_gray[y:y+h,x:x+w]
                #detect eyes
                eyes = eyes_cascade.detectMultiScale(faceROI)
                N_eyes = len(eyes)
                eye_center_holder = []
                if N_eyes==2: #only go further if we have detected two eyes.. 
                    eye_coord_arr = np.zeros((2,2))
                    for i, (x2,y2,w2,h2) in enumerate(eyes):
                        eye_center = (x + x2 + w2//2, y + y2 + h2//2)
                        radius = int(round((w2 + h2)*0.25))
                        frame = cv2.circle(frame, eye_center, radius, (255, 0, 0 ), 4)
                        eye_coord_arr[i,:] = np.array([eye_center[0],eye_center[1]]) #x, y in global coords
                
                    left_eye, right_eye = assign_eyes(eye_coord_arr)
                    """
                    Solution exercise 16: find the normalized normal vector
                    """
                    #< START_SOLUTION 16 >
                    vec = right_eye - left_eye
                    ###find the normal vector - to find the translation we need  
                    n_vec = np.array([-vec[1],vec[0]])
                    norm = np.linalg.norm(n_vec)
                    #< END_SOLUTION 16 >

                    if norm == 0:  #becomes zero if the same eye is detected twice - we can't place a hat then
                        print("Warning: zero-length eye vector, skipping hat placement")
                        N_skipped += 1
                    else: #normal vector has a length - continue with hat placement
                        n_vec = n_vec/norm #normalize normal-vector 
                        """
                        Solution exercise 17: calculate the end-points of the hat, i.e. where it should be placed in the image
                        """
                        #< START_SOLUTION 17 >
                        end_point_hat_x = int(face_center[0] - n_vec[0]*w//2)
                        end_point_hat_y = int(face_center[1] - n_vec[1]*h//2) #subtract because image coordinate system 
                        #< END_SOLUTION 17 >
                    
                    
                        cv2.circle(frame,(end_point_hat_x,end_point_hat_y),10, color=(0,255,0),thickness=3, lineType=8, shift=0) #blue circle - top of face 
                        cv2.line(frame,tuple(face_center),(end_point_hat_x,end_point_hat_y),color=(255, 0, 0 )) #green line
                    
                        """
                        Exercise 18: rescale size such that it fits the head-width 
                        """
                        object_im_trf = rescale_object(object_im, w+100) #finish the implementation of this function

                        """
                        Solution exercise 19: define the coordinates for alignment in the local hat coordinate frame
                        """
                        #< START_SOLUTION 19 >
                        #anchor point calculation: bottom centre of hat 
                        anchor_point = np.array([object_im_trf.shape[0]-1,object_im_trf.shape[1]//2]) #y,x always     
                        #< END_SOLUTION 19 >
                    
                        if ROTATE:      
                            """
                            Solution exercise 20: calculate the rotation angle which the hat should be rotated.
                            """ 
                            #< START_SOLUTION 20 >
                            theta = np.arctan2(-vec[1],vec[0]) #is in radians
                            theta *= 180/np.pi 
                            #< END_SOLUTION 20 >
                            object_im_trf, anchor_point_new, dx, dy = rotate_object(object_im_trf, theta, anchor_point, allow_resize=False) #get the corresponding anchor point after applying rotation
                            #TODO: anchor_point_new in principle needs to be part of the transform when allow_resize=True - else we get wrong transform. Use allow_resize = False for now.  
                            anchor_point[0] = anchor_point[0] - dy 
                            anchor_point[1] = anchor_point[1] - dx 
                        frame = insert_hat_on_frame(frame, object_im_trf, (end_point_hat_y,end_point_hat_x), anchor_point) 
                        N_skipped = 0
                else: #case: no valid detection, fall-back to references 
                    N_skipped += 1
            if N_skipped >= 1 and all(x is not None for x in (object_im_trf, end_point_hat_x, end_point_hat_y, anchor_point)):
                frame = insert_hat_on_frame(frame, object_im_trf, (end_point_hat_y,end_point_hat_x), anchor_point) 
            # Keep track of frames-per-second (FPS) and also show skipped frames information
            n_frames = n_frames + 1
            elapsed_time = time.time() - start_time
            fps = int(n_frames / elapsed_time)
            # Put the information on the image frame: FPS, number of faces, number of eyes 
            str_out = f"fps: {fps}, N_skip: {N_skipped}"
            cv2.putText(frame, str_out, (0, 30), font, 1, 255, 1)
            cv2.imshow('Capture - Face detection', frame)
        
            if cv2.waitKey(1) == ord('q'):
                stop = True
    ```
<!-- END_SOLUTION 14-20 -->

### Exercise 21: Speeding things up (**Optional**)
You might have noticed that the video feed is quite sluggish. In its current state, there are quite a few inefficiencies in the code. The largest overhead is in the image transformations, so reducing the number of times they are called or making them more efficient would improve real-time performance. 
Try to see if you can speed things up. Can you find some obvious inefficiencies? 

??? TIP "Optimization suggestions" 
    Here are a few of the more obvious suggestions for increasing performance: 
    
    ??? TIP "Suggestion 1:"
        The interpolation order for rotations and rescalings could in some cases be set to 0 with no large effect on the output. This is cheaper to compute 
    
    ??? TIP "Suggestion 2:"
        Rotating an object can be fairly expensive, especially if it is a large image. Similarly resizing is not always necessary. We could spare the transformations if we chose to only resize when the face width is significantly different to last iteration. Similarly for the rotation angle. Else we could simply use the transformed image from the last iteration. 
    
    ??? TIP "Suggestion 4:"
        While good, skimage.transform.rotate and skimage.transform.rescale are significantly slower than opencv alternatives. Try to implement some of the functionality using opencv instead.
    
    ??? TIP "Suggestion 5:"
        As we learned in the week with geometric transformations, transformations can be combined. As a result, we could combine the translation, scaling and rotation into a single rigid transformation. Of course, you would need to keep track of where the alignment centre ends up in the transformed object image
    
    ??? TIP "Suggestion 6:"
        The rotation of the anchor-point mask is currently done numerically, thus making one unecessary rotation per iteration. Could you find the coordinate analytically instead?
    
    ??? TIP "Suggestion 7:"
    	Optimizing the cascade classifier: increasing the scaleFactor will reduce the number of layers in the image pyramid, and thus yield faster evaluation speeds. Similarly, increasing the number minNeighbors will allow faster rejection of candidates. Lastly, for the face-detection, the minSize argument could preferentially be tuned to only allow larger detections, which significantly will speed up the search. 
    ??? TIP "Suggestion 8:"
    	Currently the tracker is detecting faces on the complete image for each frame, but in practice a person in front of the camera would probably be located within a neighborhood of the last detection. If we reduce the search region of interest to only be within the crop $x\pm\Delta$ and $y\pm\Delta$, we would be able to speed up the complete pipeline significantly. We could then evaluate it on the complete frame only if the face is lost.  
    
    ??? TIP "Suggestion 9:" 
    	Image downscaling: instead of evaluating the cascade classifier on the input image size, we could downscale it beforehand, e.g. by a factor 2. All detected coordinates would then simply have to be multiplied by a factor two in the shown modified image.  

    You can try any of these optimizations (there exist more than these), and see if you can measure the impact on the fps displayed. 

<!-- START_SOLUTION 21 -->
??? tip "Solution 21"
    Run from jupyter by executing solution file: 

    ```py
    !python part_5_viola_jones_snapchat_solutions_optim.py
    ```

    in a code block.
    The solution file contains:

    ```py
    from __future__ import print_function
    from typing import Tuple
    import cv2
    import time
    import numpy as np 
    from skimage.transform import rotate, rescale

    """
    An adapted version of https://docs.opencv.org/3.4/db/d28/tutorial_cascade_classifier.html

    Experimental: 
        - No support for resize=True. 
        - Also no optimizations introduced yet. 
    """

    def connect_camera(video_file: str = "", use_droid_cam: bool = False) -> cv2.VideoCapture:
        """
        Attempts to connect to the webcam. 
        If this fails, prints an error message and exits the complete script. 

        Args:
            use_droid_cam (bool, optional): whether to use the android camera, as done in video change detection exercises, ex2b. Defaults to False.

        Returns:
            cv2.VideoCapture: the opencv camera object which we can pull frames from
        """
        print("Opening connection to camera")
        if video_file != "":
            cap = cv2.VideoCapture(video_file)

        else:
            url = 0
            use_droid_cam = False
            if use_droid_cam:
                url = "http://192.168.1.120:4747/video"
            cap = cv2.VideoCapture(url)

        if not cap.isOpened():
            print("Cannot open camera")
            exit()

        ret, _ = cap.read()
        if not ret:
            print("---(!)Error reading frame from webcam")
            exit()
        print("Successfully connected to camera capturing device")
        return cap 

    def load_cascades(cascade_directory: str) -> Tuple[cv2.CascadeClassifier, cv2.CascadeClassifier]:
        """
        A function to load a the pretrained opencv cascade detectors (Viola-Jones-type)
        Remember to download the classifiers beforehand (https://raw.githubusercontent.com/opencv/opencv/3.4/data/haarcascades/)

        Args:
            cascade_directory (str): the directory the classifier .xml files are in

        Returns:
            tuple[cv2.CascadeClassifier]: a tuple of the two classifiers
    
        """
        face_cascade = cv2.CascadeClassifier() #initialize a classifier instance
        eyes_cascade = cv2.CascadeClassifier()
    
        face_cascade_name = cascade_directory + "/haarcascade_frontalface_default.xml"
        eyes_cascade_name = cascade_directory + "/haarcascade_eye_tree_eyeglasses.xml" #seems to be more robust to lighting and has less false detections
    
    
        if not face_cascade.load(cv2.samples.findFile(face_cascade_name)):
            print('--(!)Error loading face cascade')
            exit(0)
        if not eyes_cascade.load(cv2.samples.findFile(eyes_cascade_name)):
            print('--(!)Error loading eyes cascade')
            exit(0)

        print("Loaded cascade classifiers")
        return face_cascade, eyes_cascade

    def load_png_object(file_path: str) -> Tuple[np.ndarray, Tuple[int, int, int]]:
        """
        Loads a png object as RGBA. Note, the .png should have a transparent background and be loaded with an alpha channel

        Args:
            file_path (str): relative path to the .png file 

        Returns:
            Tuple[np.ndarray, Tuple[int, int, int]]: the image array of shape (H, W, C) and a tuple (H, W, C) where C=4 for RGBA image 
        """

        im = cv2.imread(file_path,cv2.IMREAD_UNCHANGED)
        im_dims = im.shape 
    
        print(f"loaded object: {file_path} with shape: {im_dims}")
        return im, im_dims

    def insert_hat_on_frame(frame_rgb, obj, coords, anchor):
        frame = frame_rgb.copy()
    
        Hf, Wf = frame.shape[:2]
        Hh, Wh = obj.shape[:2]

        hat_mask = obj[:, :, 3] > 0 #alpha "not transparent" mask
        hat_rgb  = obj[:, :, :3]

        # coords = where anchor should go in the rgb frame, i.e. face coordinate position
        target_y, target_x = coords
        anchor_y, anchor_x = anchor  # anchor inside the hat image

        # top-left corner on frame where hat-image should start
        slice_start_y = int(target_y - anchor_y)
        slice_start_x = int(target_x - anchor_x)

        # ---- STEP 1: handle negative object starts (crop top/left) ----
        crop_x_left = max(0, -slice_start_x)
        crop_y_top  = max(0, -slice_start_y)

        x1 = max(0, slice_start_x)
        y1 = max(0, slice_start_y)

        # ---- STEP 2: compute end positions before cropping bottom/right ----
        x2 = x1 + (Wh - crop_x_left)
        y2 = y1 + (Hh - crop_y_top)

        # ---- STEP 3: crop bottom/right if object is going outside frame in bottom/right ----
        crop_x_right = max(0, x2 - Wf)
        crop_y_bottom = max(0, y2 - Hf)

        # Compute final slice ranges after all cropping
        x2 = x2 - crop_x_right
        y2 = y2 - crop_y_bottom

        # Corresponding crop on hat image
        hat_crop = hat_rgb[crop_y_top:Hh - crop_y_bottom,
                           crop_x_left:Wh - crop_x_right]

        mask_crop = hat_mask[crop_y_top:Hh - crop_y_bottom,
                             crop_x_left:Wh - crop_x_right]

        # ---- STEP 4: insert ----
        im_mask = np.zeros((Hf, Wf), dtype=bool)
        full_hat = np.zeros_like(frame)

        im_mask[y1:y2, x1:x2] = mask_crop
        full_hat[y1:y2, x1:x2] = hat_crop

        frame[im_mask] = full_hat[im_mask]

        return frame

    def time_smoothen_detections(arr_new: np.ndarray | float, arr_ref: np.ndarray | float, alpha: float = 0.95) -> np.ndarray | float:
        """
        Time smoothen detections. Hint: equation 4 in video change detection notes.

        Args:
            arr_new (np.ndarray or float): the input array or value to be smoothened using reference. If array shape (N,features)
            arr_ref (np.ndarray or float): the reference array or value to use for smoothing. If array shape (N,features)
            alpha (float, optional): smoothing-factor. Defaults to 0.95.

        Returns:
            np.ndarray or float: the smoothened array or value 
        """
        return (alpha*arr_ref + (1-alpha)*arr_new)
    

    def rotate_object(object_im: np.ndarray, angle: float, anchor_point: tuple, allow_resize: bool = False) -> Tuple[np.ndarray,np.ndarray,int,int] | Tuple[np.ndarray,np.ndarray]:
        """
        Rotates an image using skimage.transforms.rotate around the anchor_point. 
        Finds the corresponding location of the anchor point in the rotated image frame. 
        Returns the difference in image size before and after rotation, dx and dy.  
    
        Args:
            object_im (np.ndarray): the object image to be rotated, (H, W, C)
            angle (float): rotation angle in counter-clockwise direction in degrees. 
            anchor_point (tuple): the centre in object image used rotation. For this implementation, use the bottom centre. 

        Returns:
            Tuple[np.ndarray,np.ndarray,int,int] 
        """
        h, w = object_im.shape[0], object_im.shape[1]
        object_im = rotate(object_im, angle, center=(anchor_point[1],anchor_point[0]),order=1, resize=allow_resize)
    
        #we need to know the anchor point post-translation and resizing. Easiest way is to rotate a mask containing the anchor point with the same transform used for the image 
        anchor_mask = np.zeros((h, w), dtype=np.uint8)
        anchor_mask[anchor_point] = 1
        anchor_rot = rotate(anchor_mask, angle, center=(anchor_point[1],anchor_point[0]), resize=allow_resize, order=0, preserve_range=True)
        #find the new anchor coordinate in the rotated image. This is the coordinate which needs to be mapped to the top centre of head.
        new_anchor = np.argwhere(anchor_rot == 1)[0]   # row, col in rotated space -> y, x 
    
        #calculate the difference in sizes between original image and rotated image. Used for correcting position on top of head if resize is true.
        delta_width = w-object_im.shape[1]
        delta_height = h-object_im.shape[0]
    
        return object_im, new_anchor, delta_width, delta_height 
    
    def rotate_object_cv(object_im: np.ndarray, angle: float, anchor_point: tuple, allow_resize: bool = False) -> Tuple[np.ndarray,np.ndarray,int,int] | Tuple[np.ndarray,np.ndarray]:
        """
        Rotates an image using skimage.transforms.rotate around the anchor_point. 
        Finds the corresponding location of the anchor point in the rotated image frame. 
        Returns the difference in image size before and after rotation, dx and dy.  
    
        Args:
            object_im (np.ndarray): the object image to be rotated, (H, W, C)
            angle (float): rotation angle in counter-clockwise direction in degrees. 
            anchor_point (tuple): the centre in object image used rotation. For this implementation, use the bottom centre. 

        Returns:
            Tuple[np.ndarray,np.ndarray,int,int] 
        """
        h, w = object_im.shape[0], object_im.shape[1]
        rot_mat = cv2.getRotationMatrix2D(center=(float(anchor_point[1]),float(anchor_point[0])),angle=angle,scale=1.0)
        object_im = cv2.warpAffine(src=object_im, M=rot_mat, dsize=(h, w))
    
        #we need to know the anchor point post-translation and resizing. Easiest way is to rotate a mask containing the anchor point with the same transform used for the image 
        anchor_mask = np.zeros((h, w), dtype=np.uint8)
        anchor_mask[anchor_point] = 1
        anchor_rot = cv2.warpAffine(src=anchor_mask, M=rot_mat, dsize=(h, w))
        #find the new anchor coordinate in the rotated image. This is the coordinate which needs to be mapped to the top centre of head.
        new_anchor = np.argwhere(anchor_rot == 1)[0]   # row, col in rotated space -> y, x 
    
        #calculate the difference in sizes between original image and rotated image. Used for correcting position on top of head if resize is true.
        delta_width = w-object_im.shape[1]
        delta_height = h-object_im.shape[0]
    
        return object_im, new_anchor, delta_width, delta_height 
    
    def rescale_object(object_im: np.ndarray, face_width: int) -> np.ndarray: 
        """
        Rescales the object image to a specific width so it can match face width. 

        Args:
            object_im (np.ndarray): object image, shape (H,W,C)
            face_width (int): the detected face width

        Returns:
            np.ndarray: the rescaled object image

        """
        """
        Solution exercise 8: change the width_scale_factor such that the function adaptively fits the object onto the head
        """
        width_scale_factor = face_width/object_im.shape[1]
        out_im = rescale(object_im, width_scale_factor,channel_axis=-1, order=1)
        return out_im

    def rescale_object_cv(object_im: np.ndarray, face_width: int) -> np.ndarray: 
        """
        Rescales the object image to a specific width so it can match face width. 

        Args:
            object_im (np.ndarray): object image, shape (H,W,C)
            face_width (int): the detected face width

        Returns:
            np.ndarray: the rescaled object image

        """
        """
        Solution exercise 8: change the width_scale_factor such that the function adaptively fits the object onto the head
        """
        width_scale_factor = face_width/object_im.shape[1]
        h, w = int(width_scale_factor*object_im.shape[0]), int(width_scale_factor*object_im.shape[1])
        #out_im = rescale(object_im, width_scale_factor,channel_axis=-1, order=1)
        out_im = cv2.resize(object_im, dsize=(w,h),interpolation=cv2.INTER_LINEAR)
        return out_im
    
    def assign_eyes(coords: np.ndarray) -> tuple[np.ndarray,np.ndarray]:
        """
        Assign left and right eye based on location in frame

        Args:
            eye_coords (np.ndarray): eye coordinate array where each row corresponds to x,y centre coordinates, shape (N, 2) (in current implementation of main loop, N=2)

        Returns:
            tuple[np.ndarray,np.ndarray]: left eye centre coordinates, right eye centre coordinates
        """
        #Example solution: 
        right_eye_idx = np.argmax(coords[:,0]) #actually we should use the pair which has the maximal horizontal distance
        right_eye = coords[right_eye_idx,:]
        left_eye = coords[1-right_eye_idx,:]

        if left_eye[0] > right_eye[0]:
            left_eye, right_eye = right_eye, left_eye
    
        return left_eye, right_eye


    if __name__=="__main__":
        ###step 1: connect to webcam
        USE_DROID_CAM = False 
        VIDEO_FILE = "data/speech_snippet.mp4"
        cap = connect_camera(video_file=VIDEO_FILE, use_droid_cam=USE_DROID_CAM)
    
        ###step 2: load classifiers 
        pretrained_dir = "data/pretrained_classifiers"
        face_cascade, eyes_cascade = load_cascades(pretrained_dir)
    
        # To keep track of frames per second
        start_time = time.time()
        font = cv2.FONT_HERSHEY_COMPLEX

        object_path = "data/image_props/hat_rescaled.png"
        object_im, object_dims = load_png_object(object_path)
        object_im_trf = object_im.copy()
       
        #set loop parameters
        n_frames = 0
        ROTATE = False #set to true if we want to rotate the object of interest with an angle 
        SMOOTHEN = True 
        stop = False
        N_eyes = 0
        N_skipped = 0 #for counting how many frames we do not have any detections
    
        #initialize for fallback when no detections are present
        object_im_trf = None
        end_point_hat_y = None
        end_point_hat_x = None 
        anchor_point = None 

        while not stop:
            ret, frame = cap.read()
            if frame is None:
                print('--(!) No captured frame -- Break!')
                break
            
            frame_gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY) #convert to grayscale
            h_, w_ = frame_gray.shape
            # frame_gray = cv2.resize(frame_gray,(w_//2,h_//2))
            frame_gray = cv2.equalizeHist(frame_gray) 
            # faces = face_cascade.detectMultiScale(frame_gray,scaleFactor=1.8, minNeighbors=3,minSize=(60,60)) #is a np.array (N,4) or tuple
            faces = face_cascade.detectMultiScale(frame_gray) #is a np.array (N,4) or tuple
            N_faces = len(faces)
            for i, (x,y,w,h) in enumerate(faces):
                center = (x + w//2, y + h//2)
                face_center = np.array([2*center[0],2*center[1]])
                frame = cv2.ellipse(frame, face_center, (w, h), 0, 0, 360, (255, 0, 255), 4)
                faceROI = frame_gray[y:y+h,x:x+w]
                #detect eyes
                # eyes = eyes_cascade.detectMultiScale(faceROI,scaleFactor=1.4,minNeighbors=2, minSize=(5,5))
                eyes = eyes_cascade.detectMultiScale(faceROI)
                N_eyes = len(eyes)
                eye_center_holder = []
                if N_eyes==2: #only go further if we have detected two eyes.. 
                    eye_coord_arr = np.zeros((2,2))
                    for i, (x2,y2,w2,h2) in enumerate(eyes):
                        eye_center = (2*(x + x2 + w2//2), 2*(y + y2 + h2//2))
                        radius = int(round((w2 + h2)*0.25))
                        frame = cv2.circle(frame, eye_center, radius, (255, 0, 0 ), 4)
                        eye_coord_arr[i,:] = np.array([eye_center[0],eye_center[1]]) #x, y in global coords
                
                    left_eye, right_eye = assign_eyes(eye_coord_arr)
                    """
                    Solution exercise 6: find the normalized normal vector
                    """
                    face_center = face_center 
                    vec = right_eye - left_eye
                    ###find the normal vector - to find the translation we need  
                    n_vec = np.array([-vec[1],vec[0]])
                    norm = np.linalg.norm(n_vec)

                    if norm == 0:  #becomes zero if the same eye is detected twice - we can't place a hat then
                        print("Warning: zero-length eye vector, skipping hat placement")
                        N_skipped += 1
                    else: #normal vector has a length - continue with hat placement

                        if not any(x is None for x in (vec, n_vec, norm)): # True if (vec,n_vec,norm) implemented
                            n_vec = n_vec/norm #normalize normal-vector 
                        else: # Pass if not yet implemented
                            pass

                        """
                        Solution exercise 7: calculate the end-points of the hat, i.e. where it should be placed in the image
                        """
                        end_point_hat_x = int(face_center[0] - n_vec[0]*w)
                        end_point_hat_y = int(face_center[1] - n_vec[1]*h) #subtract because image coordinate system 
                    
                        cv2.circle(frame,(end_point_hat_x,end_point_hat_y),10, color=(0,255,0),thickness=3, lineType=8, shift=0) #blue circle - top of face 
                        cv2.line(frame,tuple(face_center),(end_point_hat_x,end_point_hat_y),color=(255, 0, 0 )) #green line
                    
                        """
                        Exercise 8: rescale size such that it fits the head-width 
                        """
                        object_im_trf = rescale_object_cv(object_im, w+100) #finish the implementation of this function

                        """
                        Solution exercise 9: define the coordinates for alignment in the local hat coordinate frame
                        """
                        #anchor point calculation: bottom centre of hat 
                        anchor_point = np.array([object_im_trf.shape[0]-1,object_im_trf.shape[1]//2]) #y,x always     
                    
                        if ROTATE:      
                            """
                            Solution exercise 10: calculate the rotation angle which the hat should be rotated.
                            """ 
                            theta = np.arctan2(-vec[1],vec[0]) #is in radians
                            theta *= 180/np.pi 
                            object_im_trf, anchor_point_new, dx, dy = rotate_object_cv(object_im_trf, theta, anchor_point, allow_resize=False) #get the corresponding anchor point after applying rotation
                            #TODO: anchor_point_new in principle needs to be part of the transform when allow_resize=True - else we get wrong transform. Use allow_resize = False for now.  
                            anchor_point[0] = anchor_point[0] - dy 
                            anchor_point[1] = anchor_point[1] - dx 
                        frame = insert_hat_on_frame(frame, object_im_trf, (end_point_hat_y,end_point_hat_x), anchor_point) 
                        N_skipped = 0
                else: #case: no valid detection, fall-back to references 
                    N_skipped += 1
                
            if N_skipped>=1 and all(x is not None for x in (object_im_trf, end_point_hat_x, end_point_hat_y, anchor_point)):
                frame = insert_hat_on_frame(frame, object_im_trf, (end_point_hat_y,end_point_hat_x), anchor_point) 
            # Keep track of frames-per-second (FPS) and also show skipped frames information
            n_frames = n_frames + 1
            elapsed_time = time.time() - start_time
            fps = int(n_frames / elapsed_time)
            # Put the information on the image frame: FPS, number of faces, number of eyes 
            str_out = f"fps: {fps}, N_skip: {N_skipped}"
            cv2.putText(frame, str_out, (0, 30), font, 1, 255, 1)
            cv2.imshow('Capture - Face detection', frame)
        
            if cv2.waitKey(1) == ord('q'):
                stop = True
    ```
<!-- END_SOLUTION 21 -->

### Exercise 22: Increasing robustness (**Optional**)
In some lighting conditions, the hat placement is still very unrobust and eye-detection may still be faulty. Try to tune the max and min-size of face and eye detections. In addition, you could build in "memory" of earlier rotation angles `theta` and the `end_point_hat_y` and `end_point_hat_x` positions.

??? TIP "Hint"
    Consider equation 4 from the video-change-detection note. Finish the `time_smoothen_detections` function and add smoothing support. Does your result improve? 
    Consider the reason why we calculate the smoothing only on the end-points and angle, and not on the intermediary quantities.


## Part 6: Exam preparation 
Below are two example exam exercises. Work with them, and if you have issues, please ask the TAs, as you will not be able to get help after the last exercise round.

### Exercise from 02502 Image Analysis Exam Spring 2025

*Question 1: One of the following statements is not correct. Which one?*

- [ ] Do not know
- [ ] Haar features can be computed fast using an integral image.
- [ ] Procrustes alignment can be used to do groupwise registration of several point clouds
- [ ] The Hough transform can find the shortest curved path trough an image
- [ ] The Prewitt filter can be used to detect high gradients in an image
- [ ] A pixel with RGB values of [255, 255, 255] is white

<!-- START_SOLUTION 22 -->
??? tip "Solution 22"
    - [ ] Do not know
    - [ ] Haar features can be computed fast using an integral image.
    - [ ] Procrustes alignment can be used to do groupwise registration of several point clouds
    - [x] The Hough transform can find the shortest curved path trough an image
    - [ ] The Prewitt filter can be used to detect high gradients in an image
    - [ ] A pixel with RGB values of [255, 255, 255] is white
<!-- END_SOLUTION 22 -->


### Exercise from 02502 Image Analysis Exam Fall 2022: Fast face detection
You want to create a new app that can detect faces and put funny hats on them. You are
basing your method on the well known Viola Jones face detector.

![Exam_fig_1](figures/exam_fig_1.png)

*Question 2: You need to compute many image features fast and have chosen to use the Haar features. In the image, you can see the three rectangle Haar feature and how it is placed in an image. What is the resulting feature value?*

- [ ] 116
- [ ] Do not know
- [ ] -317 
- [ ] -495
- [ ] 324
- [ ] -9

<!-- START_SOLUTION 23 -->
??? tip "Solution 23"
    - [ ] 116
    - [ ] Do not know
    - [ ] -317 
    - [x] -495
    - [ ] 324
    - [ ] -9
<!-- END_SOLUTION 23 -->


![Exam_fig_2](figures/exam_fig_2.png)

*Question 3: To be able to compute image features very fast, you pre-compute an integral image. The input image can be seen above. What is the value of the integral image at the marked pixel?*

- [ ] 109
- [ ] 516
- [ ] 936 
- [ ] 789
- [ ] Do not know
- [ ] 884

<!-- START_SOLUTION 24 -->
??? tip "Solution 24"
    - [ ] 109
    - [ ] 516
    - [x] 936 
    - [ ] 789
    - [ ] Do not know
    - [ ] 884
<!-- END_SOLUTION 24 -->


# References
---
[YOLO object detection](https://arxiv.org/abs/1506.02640)

[Comparison of Viola-Jones performance vs. YOLO v3](https://www.researchgate.net/publication/364311838_Comparative_of_Viola-Jones_and_YOLO_v3_for_Face_Detection_in_Real_time)

[Study on the strength and limitations of the Viola-Jones algorithm](https://www.researchgate.net/publication/367584143_evaluation_study_of_face_detection_by_Viola-Jones_algorithm)

[Speech at the International Union of Electrical Radio and Machine Workers](https://archive.org/details/201291_International_Union_of_Electrical_Radio_and_Machine_Workers)






    
