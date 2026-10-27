_Latest Page Update: 27-10-2026_

***Solution is now available! Download the full solution from here:*** [Solution](../downloads/sol_material-6b.zip){ .md-button .md-button--primary .inline-button }

# Image Segmentation


## Exercise 1 
Display both the T1 and T2 images, their 1 and 2D histograms and scatter plots.
Tips: Use the `plt.imshow()`, `plt.hist()`, `plt.hist2d()` and `plt.scatter()` functions
Add relevant title and label for each axis. One can use `plt.subplots()` to show more subfigures in the same figure. **Remove intensities from background voxels for 1D and 2D histograms.**

![imshow image coordinates](figs/Picture1.png)

The two MRI image modalities contain different types of intensity classes:

1. (Orange): The White Matter (WM) is the tissue type that contain the brain network - like the cables in the internet. The  WM ensure the communication flow between functional brain regions.
2. (Yellow): The Grey Matter (GM) is the tissue type that contain the cell bodies at the end of the brain network and are the functional units in the brain. The functional units are like CPUs in the computer. They are processing our sensorial input and are determining a reacting to these. It could be to start running.
3. (Magenta): Cerebrospinal fluid (CSF) which is the water in the brain 
4. (Green): Background of the image

**Q1**: What is the intensity threshold that can separate the GM and WM classes (roughly) from the 1D histograms? 

**Q2**: Can the GM and WM intensity classes be observed in the 2D histogram and scatter plot?

<!-- START_SOLUTION 1 -->
??? tip "Solution 1"
    ```py


    mask_init = ImgT1 > 10
    mask = opening(mask_init, disk(2))
    mask = closing(mask, disk(3))

    fig, axs = plt.subplots(nrows = 1, ncols = 3, figsize = (10, 10))
    axs[0].imshow(ImgT1, cmap = 'gray')
    axs[1].imshow(mask_init, cmap = 'gray', interpolation = 'none')
    axs[2].imshow(mask, cmap = 'gray', interpolation = 'none')
    plt.show()
    ```

    ```py
    def my_show_1D_histogram(img, mask, img_name, axs):
        axs[0].imshow(img, cmap = 'gray')
        axs[0].set_title(img_name)
        axs[0].set_axis_off()

        intensities = img[mask]
        axs[1].hist(intensities, bins = 100)
        axs[1].set_title(f'1D Histogram {img_name}')
        axs[1].set_xlabel('Intensities')
        axs[1].set_ylabel('Frequency')

        return intensities
    ```

    ```py
    fig, axs = plt.subplots(nrows = 2, ncols = 3, figsize = (15, 8))
    T1_intensities = my_show_1D_histogram(ImgT1, mask, 'T1w image', axs[0, 0:2])
    T2_intensities = my_show_1D_histogram(ImgT2, mask, 'T2w image', axs[1, 0:2])
    axs[0,2].hist2d(T1_intensities, T2_intensities, bins = 100)
    axs[0,2].set_xlabel('T1w intensities')
    axs[0,2].set_ylabel('T2w intensities')
    axs[0,2].set_title('2D histogram')

    axs[1,2].scatter(T1_intensities, T2_intensities)
    axs[1,2].set_xlabel('T1w intensities')
    axs[1,2].set_ylabel('T2w intensities')
    axs[1,2].set_title('Scatter plot')
    plt.tight_layout()
    plt.show()
    ```

    **Q1**: What is the intensity threshold that can separate the GM and WM classes (roughly) from the 1D histograms?
    A1: For T1:about 500 and for T2: about 160


    **Q2**: Can the GM and WM intensity classes be observed in the 2D histogram and scatter plot? 

    A2: Yes there appear clear two clusters with maxium intensitised in the 2D histogram that represent the two classes distributions
<!-- END_SOLUTION 1 -->

## Exercise 2
Place trainings examples i.e. ROI_WM and ROI_GM into variables C1 and C2 representing class 1 and class 2 respectively. Show in a figure the manually expert drawings of the C1 and C2 training examples.

??? TIP
    Use `plt.imshow()`

**Q3**: Does the ROI drawings look like what you expect from an expert? 

<!-- START_SOLUTION 2 -->
??? tip "Solution 2"
    **Q3**: Does the ROI drawings look like what you expect from an expert? 

    A3: No, its just some stribes here and there in the outline the intensities of the two classes

    ```py
    fig, axs = plt.subplots(nrows = 1, ncols = 1, figsize = (5, 5))
    axs.imshow(ImgT1, cmap = 'gray')
    axs.imshow(ROI_GM, cmap = 'Reds', alpha = 0.5, interpolation = 'none')
    axs.imshow(ROI_WM, cmap = 'Greens', alpha = 0.5, interpolation = 'none')
    plt.show()
    ```
<!-- END_SOLUTION 2 -->

## Exercise 3
For each binary training ROI find the corresponding training examples in ImgT1 and ImgT2. Later these will be extracted for LDA training.

??? TIP
    If you are a MATLAB-like programming lover, you may use the `np.argwhere()` function appropriately to return the index to voxels in the image full filling e.g. intensity values >0 hence belong to a given class. Name the index variables qC1 and qC2, respectively.

**Q4**: What is the difference between the 1D histogram of the training examples and the 1D histogram of the whole image? Is the difference expected?

<!-- START_SOLUTION 3 -->
??? tip "Solution 3"
    **Q4**: What is the difference between the 1D histogram of the training examples and the 1D histogram of the whole image? Is the difference expected?  

    A4: It looks very alike the full image histogram except for better class seperation and there is no background


    ```py
    # This is not necessary with the coding style followed in this solution.
    qC1 = np.argwhere(ROI_WM) # Shape (399, 2) 
    qC2 = np.argwhere(ROI_GM) # (280, 2)
    ```
    ```py
    trainWM_T1 = ImgT1[ROI_WM] # trainWM_T1 = ImgT1[qC1[:,0], qC1[:,1]]
    trainGM_T1 = ImgT1[ROI_GM] # trainGM_T1 = ImgT1[qC2[:,0], qC2[:,1]]

    trainWM_T2 = ImgT2[ROI_WM] # trainWM_T2 = ImgT2[qC1[:,0], qC1[:,1]]
    trainGM_T2 = ImgT2[ROI_GM] # trainGM_T2 = ImgT2[qC2[:,0], qC2[:,1]]

    fig, axs = plt.subplots(1, 2, figsize = (12, 4), sharex = False, sharey = True)
    axs[0].hist(trainWM_T1, bins=20, color = 'navy', label = 'WM', alpha = 0.8)
    axs[0].hist(trainGM_T1, bins=20, color = 'orange', label = 'GM', alpha = 0.8)
    axs[0].set_ylabel('Frequency')
    axs[0].set_xlabel('Intensities')
    axs[0].set_title('T1w')

    axs[1].hist(trainWM_T2, bins=20, color = 'navy', label = 'WM', alpha = 0.8)
    axs[1].hist(trainGM_T2, bins=20, color = 'orange', label = 'GM', alpha = 0.8)
    axs[1].set_xlabel('Intensities')
    axs[1].set_title('T2w')
    plt.legend()
    plt.show()
    ```
<!-- END_SOLUTION 3 -->

## Exercise 4
Make a training data vector (X) and target class vector (T) as input for the `LDA()` function. T and X should have the same length of data points.

**X**: Training data vector should first include all data points for class 1 and then the data points for class 2. Data points are the two input features ImgT1, ImgT2

**T**: Target class identifier for X where '0' are Class 1 and a '1' is Class 2.

??? TIP
    Read the documentation of the provided LDA function to understand the expected input dimensions.

<!-- START_SOLUTION 4 -->
??? tip "Solution 4"
    ```py

    # Feature matrix, X -> Shape: (n_samples, n_features)
    X1 = np.c_[ImgT1[ROI_WM], ImgT2[ROI_WM]] 
    X2 = np.c_[ImgT1[ROI_GM], ImgT2[ROI_GM]]
    X = np.r_[X1, X2]

    # Label vector, T -> Shape: (n_samples)
    n_pixels = X.shape[0]
    n_wm_pixels = X1.shape[0]
    T = np.zeros((n_pixels, 1))
    T[n_wm_pixels:] = 1
    ```
    ```py
    X1.shape
    ```
<!-- END_SOLUTION 4 -->

## Exercise 5 
Make a scatter plot of the training points of the two input features for class 1 and class 2 as green and black circles, respectively. Add relevant title and labels to axis

**Q5**: How does the class separation appear in the 2D scatter plot compared with 1D histogram. Is it better?

<!-- START_SOLUTION 5 -->
??? tip "Solution 5"
    **Q5**: How does the class separation appear in the 2D scatter plot compared with 1D histogram. Is it better?

    A5: The seperation between the two classes appear to be much better in 2D scatter plot

    ```py
    fig, ax = plt.subplots(1,1)
    ax.scatter(X1[:,0], X1[:,1], c = 'g', label = 'WM')
    ax.scatter(X2[:,0], X2[:,1], c = 'k', label = 'GM')
    ax.set_xlabel('T1w intensities')
    ax.set_ylabel('T2w intensities')
    plt.legend()
    plt.show()
    ```
<!-- END_SOLUTION 5 -->

## Exercise 6
Train the linear discriminant classifier using the Fisher discriminant function and estimate the weight-vector coefficient W (i.e. $w_0$ and $w$) for classification given X and T by using the `W=LDA()` function. The LDA function outputs W=[[w01, w1]; [w02, w2]] for class 1 and 2 respectively.

??? TIP
    Read the Bishop note on Chapter 4.

```python
W = LDA(X,T)
```

<!-- START_SOLUTION 6 -->
??? tip "Solution 6"
    ```py

    W = LDA(X, T)
    ```
<!-- END_SOLUTION 6 -->

## Exercise 7
Apply the linear discriminant classifier i.e. perform multi-modal classification using the trained weight-vector $W$ for each class: It calculates the linear score $Y$ for **all** image data points within the brain slice i.e. $y(x)= w+w_0$. Actually, $y(x)$ is the $\log(P(Ci|x))$.

```python
Xall= np.c_[ImgT1.ravel(), ImgT2.ravel()]
Y = np.c_[np.ones((len(Xall), 1)), Xall] @ W.T
```

<!-- START_SOLUTION 7 -->
??? tip "Solution 7"
    ```py

    Xall= np.c_[ImgT1[mask].flatten(), ImgT2[mask].flatten()]
    Y = np.c_[np.ones((len(Xall), 1)), Xall] @ W.T
    ```
<!-- END_SOLUTION 7 -->

## Exercise 8
Perform multi-modal classification: Calculate the posterior probability i.e. $P(C_1|X)$ of a data point belonging to class 1

??? NOTE
    Using Bayes [Eq 1]: Since* $y(x)$ *is the log of the posterior probability [Eq2] we take* $\exp(y(x))$ *to get* $P(C_1|X)=P(X|\mu,\sigma)P(C_1)$ *and divide with the marginal probability* $P(X)$ *as normalisation factor.*

    ```python
    PosteriorProb = np.clip(np.exp(Y) / np.sum(np.exp(Y),1)[:, np.newaxis], 0, 1)
    ```

<!-- START_SOLUTION 8 -->
??? tip "Solution 8"
    ```py

    PosteriorProb = np.clip(np.exp(Y) / np.sum(np.exp(Y),1)[:,np.newaxis], 0, 1)
    ```
<!-- END_SOLUTION 8 -->

## Exercise 9
Apply segmentation: Find all voxles in the T1w and T2w image with $P(C_1|X)>0.5$ as belonging to Class 1. You may use the `np.where()` function. Similarly, find all voxels belonging to class 2.

<!-- START_SOLUTION 9 -->
??? tip "Solution 9"
    ```py

    posteriorC1 = np.zeros(ImgT1.shape)
    posteriorC2 = np.zeros(ImgT1.shape)

    posteriorC1[mask] = PosteriorProb[:,0]
    posteriorC2[mask] = PosteriorProb[:,1]
    mask_WM = posteriorC1 >= 0.5
    mask_GM = posteriorC2 > 0.5
    ```
    ```py
    fig, axs = plt.subplots(nrows = 1, ncols = 3, figsize = (10, 5))
    rgb = np.stack((posteriorC1, posteriorC2, np.zeros_like(posteriorC1)), axis = -1).astype(np.double)
    axs[0].imshow(rgb)
    axs[0].set_title('Posteriors (R-WM, G-GM)')
    axs[1].imshow(mask_WM, cmap = 'gray', interpolation = 'none')
    axs[1].set_title('WM Mask')
    axs[2].imshow(mask_GM, cmap='gray', interpolation = 'none')
    axs[2].set_title('GM Mask')
    plt.show()
    ```
<!-- END_SOLUTION 9 -->

## Exercise 10
Show scatter plot of segmentation results as in 5.

**Q6** Can you identify where the hyperplane is placed i.e. y(x)=0?

**Q7** Is the linear hyper plane positioned as you expected or would a non-linear hyper plane perform better?

**Q8** Would segmentation be as good as using a single image modality using thresholding?

**Q9** From the scatter plot does the segmentation results make sense? Are the two tissue types segmented correctly.

<!-- START_SOLUTION 10 -->
??? tip "Solution 10"
    **Q6**: Can you identify where the hyper plan is placed i.e. y(x)=0? 

    A6: Yes, all the segmented points show a clear line at their
           interface=Hyperplane

    **Q7**: Is the linear hyper plane positioned as you expected or would a non-linear hyperplane perform better?

    A7: Yes and no - I would expect all black and green training data
           points to be on each side of the hyperplane. Moreover the hyperplane seems
           to be too linear and there seem to be an extras class at the low image
           resolutions. 

    **Q8**: Would segmentation be as good as using a single image modality using
        thresholding?

    A8: If that should be the case the hyperplane would be orthogonal to one
           of the features. It is not. So no.

    **Q9**: From the scatter plot does the segmentation results make sense? Are the two tissue types segmented correctly .

    A9: Looks good, but hard to say from a scatter plot


    ```py
    Xall_WM = Xall[PosteriorProb[:,0] > 0.5, :]
    Xall_GM = Xall[PosteriorProb[:,1] > 0.5, :]
    ```

    ```py
    fig, ax = plt.subplots(1,1, figsize = (10,5))
    ax.scatter(Xall_GM[:,0], Xall_GM[:,1], c = 'gray', label = 'GM')
    ax.scatter(Xall_WM[:,0], Xall_WM[:,1], c = 'greenyellow', label = 'WM')
    ax.scatter(X1[:,0], X1[:,1], c = 'g', label = 'WM Training')
    ax.scatter(X2[:,0], X2[:,1], c = 'k', label = 'GM Training')
    ax.set_xlabel('T1w intensities')
    ax.set_ylabel('T2w intensities')
    ax.set_title('Hyperplane?: C1 and C2 training samples onto all segmented voxels')

    plt.axis([0, 750, 0, 400])
    plt.legend()
    plt.show()
    ```
    ```py
    # Projection of the training data onto the hyperplane

    fig, ax = plt.subplots(1,1, figsize = (10,5))
    ax.scatter(X1[:,0], X1[:,1], c = 'g', label = 'WM Training')
    ax.scatter(X2[:,0], X2[:,1], c = 'k', label = 'GM Training')
    W_ = W[0,1:] # The weights indicate the normal to the hyperplane, for the projection we need the perpendicular
    W_ = (W_/np.linalg.norm(W_))[::-1]
    W_[0] = -W_[0]
    for point in X1:
      proj = np.dot(point,W_)/np.dot(W_,W_) * W_
      ax.scatter(proj[0],proj[1],color='g')

    for point in X2:
      proj = np.dot(point,W_)/np.dot(W_,W_) * W_
      ax.scatter(proj[0],proj[1],color='k')

    plt.legend()
    plt.show()

    # W_ = W[1,1:] # The weights indicate the normal to the hyperplane, for the projection we need the perpendicular
    # W_ = (W_/np.linalg.norm(W_))[::-1]
    # W_[0] = -W_[0]
    # for point in X1:
    #   proj = np.dot(point,W_)/np.dot(W_,W_) * W_
    #   ax.scatter(proj[0],proj[1],color='r')

    # for point in X2:
    #   proj = np.dot(point,W_)/np.dot(W_,W_) * W_
    #   ax.scatter(proj[0],proj[1],color='b')
    ```
<!-- END_SOLUTION 10 -->

## Exercise 11

**Q10** Are the training examples representative for the segmentation results? Are you surprised that so few training examples perform so well? Do you need to be an anatomical expert to draw these?

**Q11** Compare the segmentation results with the original image. Are the segmentation results satisfactory? Why not?

**Q12** Is one class completely wrong segmented? What is the problem?

<!-- START_SOLUTION 11 -->
??? tip "Solution 11"
    **Q10**: Are the training examples representative for the segmentation results? Are you surprised that so few training examples perform so well? Do you need to be an anatomical expert to draw these?

    A10: Yes they are representative. Yes and no; one do not need to be an anatomical expert, but you need to understand what the method needs as input to perform well.

    **Q11**: Compare the segmentation results with the original image. Is the segmentation results satisfactory? Why not?

    A11: Yes, the WM and GM classes looks correct segmented when comparing to the anatomical structures in ImgT1 and ImgT2.

    **Q12**: Is one class completely wrong segmented? What is the problem?

    A12: Yes - If you have not masked out the backgroud voxels, all background and skull voxels also are classified as GM. Solution is to identify the background voxels and exclude these from the segmentation or to add an extra class for background/skull voxels. Also the CSF class is segmented as both GM and WM. The segmentation problem is that there are more classes in the image than the classifier has been trained to segment.


    ```py
    tmp = np.zeros(mask_WM.shape)
    red_WM = np.stack((mask_WM, tmp, tmp), axis = -1)
    green_GM = np.stack((tmp, mask_GM, tmp), axis = -1)
    ```

    ```py
    def overlay_segmentation(I, M):
        I = I/I.max()
        I_aux, I_red = I.copy(), I.copy()
        I_aux[M] = 0
        I_red[M] = 1
        tmp = np.stack((I_red, I_aux, I_aux), axis = -1)
        return tmp
    ```
    ```py
    fig, axs = plt.subplots(2, 3, figsize = (15, 10))
    axs[0,0].imshow(ImgT1, cmap = 'gray')
    axs[0,0].set_title('T1w')
    axs[0,1].imshow(overlay_segmentation(ImgT1, mask_WM))
    axs[0,1].set_title('WM')
    axs[0,2].imshow(overlay_segmentation(ImgT1, mask_GM))
    axs[0,2].set_title('GM')

    axs[1,0].imshow(ImgT2, cmap = 'gray')
    axs[1,0].set_title('T2w')
    axs[1,1].imshow(overlay_segmentation(ImgT2, mask_WM))
    axs[1,1].set_title('WM')
    axs[1,2].imshow(overlay_segmentation(ImgT2, mask_GM))
    axs[1,2].set_title('GM')

    plt.show()
    ```
<!-- END_SOLUTION 11 -->




## Exam preparation 
Below are some example exam exercises related to this weeks material. Work with them, and if you have issues or questions, please ask the TAs, as you will not be able to get help after the last exercise round.


*Exam question 1: You are asked to design a system that can track skiers on a ski slope. Due to the weather, you can expect that the lightning conditions slowly changes during the slope opening times. What elements do you need in your image analysis system?*


- [ ] Estimation of a slowly changing background image, computing difference between current video frame and background, detection of significant changes in the image, analysis of moving objects.
- [ ] Conversion of image from RGB to grey scale, parametric classification of pixel values, image registration to a template figure
- [ ] Dynamic programming to find slope, image filtering to sort birds from the image, fast facial recognition to find skiers
- [ ] Do not know
- [ ] Image registration of slopes to a template slope, LDA classification of registration results, morphological closing of outlines
- [ ] Morphological opening of binary image from camera, median filtering of binary images, image rotation to get slope in the right camera view?


<!-- START_SOLUTION 12 -->
??? tip "Solution 12"
    - [x] Estimation of a slowly changing background image, computing difference between current video frame and background, detection of significant changes in the image, analysis of moving objects.
    - [ ] Conversion of image from RGB to grey scale, parametric classification of pixel values, image registration to a template figure
    - [ ] Dynamic programming to find slope, image filtering to sort birds from the image, fast facial recognition to find skiers
    - [ ] Do not know
    - [ ] Image registration of slopes to a template slope, LDA classification of registration results, morphological closing of outlines
    - [ ] Morphological opening of binary image from camera, median filtering of binary images, image rotation to get slope in the right camera view?
<!-- END_SOLUTION 12 -->

--- 

We have used data points from two classes in a two-dimensional space and trained a classifier model using two Gaussian distributions. The mean values are [24, 3] and [30, 7] for classes 1 and 2 respectively. As a classifier, we use the Linear Discriminant Classifier that assumes equal prior-probabilities and isotropic covariances. The covariance matrix is:

$$\bm{\Sigma} = \begin{bmatrix} 2 & 0\\ 0 & 2\end{bmatrix}$$

A general formulation for a hyperplane and points belonging to class 2 is:

$$\bm{y}_{C\in 2}(\bm{x}) = \bm{x}^T\bm{w}+\bm{cw}$$

where $w$ is the weight vector and $bm{cw}$ is the threshold defining the hyperplane. So, if $\bm{x}^T\bm{w}>\bm{cw}$ a sample point belongs to class 2.

*Exam question 2: What is $\bm{y}_{C\in 2}(\bm{x})$ for $\bm{x}=(23,5)$ and what class does the sample belong to?*

- [ ] -12 and class 1
- [ ] 170 and class 2
- [ ] -30 and class 1
- [ ] 12 and class 1
- [ ] Do not know
- [ ] -1.7 and class 2


<!-- START_SOLUTION 13 -->
??? tip "Solution 13"
    - [x] -12 and class 1
    - [ ] 170 and class 2
    - [ ] -30 and class 1
    - [ ] 12 and class 1
    - [ ] Do not know
    - [ ] -1.7 and class 2
<!-- END_SOLUTION 13 -->
