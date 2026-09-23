_Latest Page Update: 23-09-2026_

## Computing camera parameters

### Exercise 1

Explain how to calculate the angle $\theta$ when $a$ and $b$ is given
in the figure below. Calculate $\theta$ (in degrees) when
$a = 10$ and $b=3$ using the function `math.atan2()`. Remember to import `math` and find out what `atan2` does.

![Angle in triangel](figures/ArcTangens.png)

<!-- START_SOLUTION 1 -->
??? tip "Solution 1"
    ```py


    import math

    a, b = 10, 3
    theta_rad = math.atan2(b, a)
    theta_deg = theta_rad*180/math.pi

    print(f"Angle (deg): {theta_deg}")
    ```
<!-- END_SOLUTION 1 -->

### Exercise 2

Create a Python function called `camera_b_distance`.

The function should accept two arguments, a focal length f and an
object distance g. It should return the distance from the lens to
where the rays are focused (b) (where the CCD should be placed)

The function should start like this:

```python
def camera_b_distance(f, g):
    """
    camera_b_distance returns the distance (b) where the CCD should be placed
    when the object distance (g) and the focal length (f) are given
    :param f: Focal length
    :param g: Object distance
    :return: b, the distance where the CCD should be placed
    """
```

It should be based on Gauss' lens equation:

$$\frac{1}{g} + \frac{1}{b} = \frac{1}{f}$$

You should decide if your function should calculate distances in mm or
in meters, but remember to be consistent!

Use your function to find out where the CCD should be placed when the
focal length is 15 mm and the object distance is 0.1, 1, 5, and 15
meters.

What happens to the place of the CCD when the object distance is increased?

<!-- START_SOLUTION 2 -->
??? tip "Solution 2"
    ```py


    def camera_b_distance(f, g):
        """
        camera_b_distance returns the distance (b) where the CCD should be placed
        when the object distance (g) and the focal length (f) are given
        :param f: Focal length
        :param g: Object distance
        :return: b, the distance where the CCD should be placed
        """
        b = 1 / (f**(-1) - g**-(1))
        return b
    ```
    ```py
    focal_distance = 15e-3 # meters
    object_distances = [0.1, 1, 5, 15] # meters

    for d in object_distances:
        val = camera_b_distance(focal_distance, d)
        print(f"Focal distance (m): {focal_distance} \t Object distance (m): {d} \t CCD place (m): {val}")
    ```
<!-- END_SOLUTION 2 -->

## Camera exercise

In the following exercise, you should remember to explain when
something is in mm and when it is in meters. To convert between
radians and degrees you can use:

```
angle_degrees = 180.0 / math.pi * angle_radians
```

### Exercise 3

Thomas is 1.8 meters tall and standing 5 meters from a camera. The
cameras focal length is 5 mm. The CCD in the camera can be seen in
the figure below. It is a 1/2" (inches) CCD chip and the
image formed by the CCD is 640x480 pixels in a (x,y) coordinate system.

![CCD chip](figures/CCDChip.png)

It is easiest to start by drawing the scene. The scene should
contain the optical axis, the optical center, the lens, the focal
point, the CCD chip, and Thomas. Do it on paper or even better in a
drawing program.

#### Exercise 3.1: A focused image of Thomas is formed inside the camera. At which distance from the lens?
<!-- START_SOLUTION 3 -->
??? tip "Solution 3"
    ```py


    f = 5e-3 # m
    g = 5 # m
    b = camera_b_distance(f, g)

    print(f"The image is formed at {b} m from the lens")
    ```
<!-- END_SOLUTION 3 -->

#### Exercise 3.2: How tall (in mm) will Thomas be on the CCD-chip?
<!-- START_SOLUTION 4 -->
??? tip "Solution 4"
    ```py


    G = 1.8 # m

    B = b*G/g
    print(f"Thomas is {B*1e3} mm tall on the CCD chip")
    ```
<!-- END_SOLUTION 4 -->

#### Exercise 3.3: What is the size of a single pixel on the CCD chip? (in mm)?
<!-- START_SOLUTION 5 -->
??? tip "Solution 5"
    ```py


    # If you understand this as the real world size of an object in a pixel...
    B = 4.8e-3
    G_aux = g*B/b # Real world distance corresponding to 480 pixels

    px_size = G_aux/480
    print(f"The real world distance corresponding to a single pixel is {px_size*1e3} mm")
    ```

    ```py
    # If you understand this as the "physical" size of a pixel on the CCD chip...

    size_y = 4.8 / 480 # mm
    size_x = 6.4 / 640 # mm

    print("The pixel size is: ", size_x, "x", size_y)
    ```
    **Note**: Some people may understand this question as what is the size of a "physical" pixel on the CCD chip. The answer to that is 6.4/640 mm x 4.8/480 mm = 0.01 x 0.01 mm.
<!-- END_SOLUTION 5 -->

#### Exercise 3.4: How tall (in pixels) will Thomas be on the CCD-chip?
<!-- START_SOLUTION 6 -->
??? tip "Solution 6"
    ```py


    h = 1.8/px_size
    print(f"Thomas will be {h} pixels tall")
    ```
    ```py
    # You can also do it like this (i.e., like a proportion)
    B = 1.8018018018018018e-3 # Exercise 3.2
    h = 480*B/4.8e-3
    print(f"Thomas will be {h} pixels tall")
    ```
<!-- END_SOLUTION 6 -->

#### Exercise 3.5: What is the horizontal field-of-view (in degrees)?
<!-- START_SOLUTION 7 -->
??? tip "Solution 7"
    ```py


    FOV_x = 2*math.atan2(3.2e-3, b)*180/math.pi
    print(f"Horizontal FOV is {FOV_x} deg")
    ```
<!-- END_SOLUTION 7 -->

#### Exercise 3.6: What is the vertical field-of-view (in degrees)?
<!-- START_SOLUTION 8 -->
??? tip "Solution 8"
    ```py


    FOV_y = 2*math.atan2(2.4e-3, b)*180/math.pi
    print(f"Vertical FOV is {FOV_y} deg")
    ```
<!-- END_SOLUTION 8 -->




## Exam preparation 
Below are three example exam exercises related to this weeks material. Work with them, and if you have issues or questions, please ask the TAs, as you will not be able to get help after the last exercise round.

*Exam question 1: A company is making an automated system for fish inspection. They are using a camera with a CCD chip that measures 5.4 x 4.2 mm and that has a focal length of 10 mm. It can be assumed that f = b. The camera takes photos that have dimensions 6480 x 5040 pixels and the camera is placed 110 cm from the fish, where a sharp image can be acquired of the fish. How many pixels wide is a fish that has a length of 40 cm?*

- [ ] 4364
- [ ] 4135
- [ ] 3213
- [ ] 5612
- [ ] 1872
- [ ] Do not know


<!-- START_SOLUTION 9 -->
??? tip "Solution 9"
    - [x] 4364
    - [ ] 4135
    - [ ] 3213
    - [ ] 5612
    - [ ] 1872
    - [ ] Do not know
<!-- END_SOLUTION 9 -->



*Exam question 2: You have a camera with a focal length of 52 mm and a CCD chip of 8 mm x 6 mm. The image dimensions are 3200 x 2400 pixels. It can be assumed that b = f. From a distance of 10 cm you have taken a sharp picture of an eye with a completely round pupil. The image is thresholded such that only the pupil is visible. You find the area of the pupil to be 416248 pixels. What is the real diameter of the pupil given in millimeters?*

- [ ] 4.2 millimeter
- [ ] 3.5 millimeter
- [ ] 2.9 millimeter
- [ ] 3.8 millimeter
- [ ] 4.4 millimeter
- [ ] Do not know


<!-- START_SOLUTION 10 -->
??? tip "Solution 10"
    - [ ] 4.2 millimeter
    - [x] 3.5 millimeter
    - [ ] 2.9 millimeter
    - [ ] 3.8 millimeter
    - [ ] 4.4 millimeter
    - [ ] Do not know
<!-- END_SOLUTION 10 -->


*Exam question 3: You have a camera with a field-of-view of 35◦ both horizontally and vertically. The cameras focal length is 20 mm and it can be assumed that f = b. What should the height of the CCD chip be, in order to take an image of the whole field-of-view?*

- [ ] 8.2 mm
- [ ] 9.8 mm
- [ ] 13.7 mm
- [ ] 10.1 mm
- [ ] 12.6 mm
- [ ] Do not know

<!-- START_SOLUTION 11 -->
??? tip "Solution 11"
    - [ ] 8.2 mm
    - [ ] 9.8 mm
    - [ ] 13.7 mm
    - [ ] 10.1 mm
    - [x] 12.6 mm
    - [ ] Do not know
<!-- END_SOLUTION 11 -->
