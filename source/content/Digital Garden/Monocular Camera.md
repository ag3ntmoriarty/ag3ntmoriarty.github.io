Monocular or single camera model is important in terms of understanding the properties that are later applied to stereo models. 

## Single Camera Projection Model
Properties of the single camera model transform the coordinates from the world coordinates ie.  $\mathbf{P}_w = [X_w, Y_w, Z_w, 1]^T$ to the 2D image coordinate system $[u, v]^T$.

### Extrinsic Matrix (World -> Camera coordinate system)
Defines where the camera is in the world coordinate frame and its orientation relative to the world origin. Transforms the world coordinate frame to the camera's local coordinate system.$$\mathbf{P}_c = \begin{bmatrix} R & \mathbf{t} \\ \mathbf{0} & 1 \end{bmatrix} \mathbf{P}_w$$
The R and t define the rotational and translational properties of the camera with respect to the world coordinate, where R and t are of size $3 \times 3$ and $3 \times 1$ respectively.

The $\mathbf P_c$ defines the position of the coordinate with respect to the camera center. Therefore, $\mathbf P_c = [X_c, Y_c, Z_c, 1]^T$ .

### Intrinsic Matrix (Camera -> Image coordinate system)
Projects the 3D coordinate system relative to the camera center to the 2D image coordinate system based on the physical properties of the camera like the lens, sensor, etc. $$s \begin{bmatrix} u \\ v \\ 1 \end{bmatrix} = \begin{bmatrix} f_x & \gamma & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} X_c \\ Y_c \\ Z_c \end{bmatrix}$$ where $f_x, f_y$ are the focal length in terms of the pixel dimensions, 
$c_x, c_y$ are the optical centers of the image coordinate plane.  
$\gamma$ is the skew coefficient (usually 0 in modern camera systems)
$s$ is the scaling factor that is equal to depth $Z_c$

#### Note
	The intrinsic matrix of the camera is figured out using camera calibration methods that basically use 2 or more points with known position in the world coordinate frame and correlate them to their position in the image plane. Generally a pattern like the checkboard pattern is used.

It can also be noted that the $Z_c$ term basically vanishes when converting the camera coordinate system to the image coordinate system. This is also observed in real life since the one cant figure out the depth and position of the object from just an image. It can be described as a ray shooting from the camera center to the image coordinate in question where the object may lie anywhere on the ray.

One can although roughly figure out the position of an object using its size and position with respect to other objects in the image, but that requires an understanding of the environment. This understanding has been developed by humans over the course of their lifetime, but deep learning based depth estimation like VGGT, DepthAnything take advantage of this principle to estimate the depth map of an image.

### Projection Matrix
Therefore, the projection matrix of the camera can be defined as
$$s \mathbf{p} = K [R \vert{} \mathbf{t}] \mathbf{P}_w$$

## Related
- [[Stereo Camera]]
- [[RGBD Camera]]

## Tags
#CameraModels 
#Sensors