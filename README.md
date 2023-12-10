# Panorama Stitching
 
In this repository, we attempt to solve the problem of automatically stitching several images that are possibly in a panorama.


## Approach

There are 4 major things that in this task:
- Matching features
    - This has been done using the inbuilt ORB feature descriptor finder.
    - To match features, we use the inbuilt BF matcher that brute forces matches between ORB features between two images using a distance function.
- Finding the homography matrix
    - We use the RANSAC algorithm with a total of 10000 trials, taking 4 matches at a time to estimate the Homography.
- Warping images
    - Given a homography, 2 methods of warping have been implemented
        - Forward projection: Here, for each pixel in the source image, we project the pixel onto a warped image. However, this causes holes in the projected image.
        - Inverse sampling: Here, we compute the Inverse homography matrix, and sample pixels for each destination pixel from the source image. This solves the problem of having holes.

    - In order to speed up the process, the transformation has been converted from a O(n^2) loop to a matrix multiplication using the indices.
    - Since we have several images, we compute the homography of further images as follows:
        - Don't warp the first image.
        - 2nd image is warped using computed homography matrix.
        - Homography between unwarped 2nd and 3rd images is found. The homography matrix with 2 wrt 1 is pre multiplied in order to maintain the changing perspective, as well as the offset.
        - Same is done for 4 wrt 3. H43_net = H43 * H32 * H21 and so on.
        - This process is repeated for images in the left as well.

- Blending images
    - In order to blend the images, we use the method of Cross-Dissolve or Linear Blending:
        - Cross-dissolve is a basic blending technique that involves linearly interpolating pixel values between adjacent images.
        - This method gradually transitions from one image to another by smoothly blending the overlapping regions.
        - In this approach, each pixel in the overlapping region is assigned a weighted average of the corresponding pixels in the two images.
        - The weights vary linearly from 1 to 0 across the transition area, ensuring a smooth and gradual blend.

## Implementation Specifics

- It is assumed that the order of the images in the dataset is sorted from left to right, and is alphabetically arranged with respect to the same

- Since the ORB feature descriptor is not very robust, we notice that the homography matrix computed is often incorrect. Moreover, we notice that even the inbuilt homography estimator using the ORB descriptor is not very good. We hence attribute this to improper feature descriptors and bad matching.

We can clearly see that the matches are not good.

- Due to the stitching method, we see that the results are ocasionally such that the stitching happens by not utilizing all of the data. This is because of the vertical/diagonal stitching method.

- Ocasionally, we notice that the warping fails since the warped image doesnt fit within the temporary image buffer. This can be fixed by changing the image size.

## Results

All the results can be viewed in the `outputs/*/*` directory.
