---
title: Stereo Camera
tags:
  - Sensors
date: 2026-08-22
---

Stereo camera utilise the [[Monocular Camera]] model for 2 cameras with known relative position generally rigidly attached to each other.

![[Pasted image 20260823181015.png|453]]

- **Left Camera Matrix:** $P_L = K_L [I \vert{} \mathbf{0}]$
- **Right Camera Matrix:** $P_R = K_R [R_s \vert{} \mathbf{t}_s]$
(For simplicity its assumed that the left camera center is the origin of the world coordinate system.)
Here $R_s, t_s$ are the relative position of the right camera center with respect to the left camera center. In a perfectly rectified and aligned stereo camera rig $R_s = I$ and $t_s = [-b, 0, 0]^T$ where $b$ is the physical baseline ie. the distance between the 2 cameras.

### Essential Matrix ($E$)
The essential matrix links the points from the left camera to the right camera irrespective of the lens intrinsics, that is

$$E = [\mathbf{t}_s]_{\times} R_s$$
here $[\mathbf{t}_s]_{\times}$ is the skew-symmetric matrix of the translation vector,

#### Epipolar Constraints
If $\mathbf{x}_L$ and $\mathbf{x}_R$ are the normalized coordinates of the same 3D point in the left and right cameras respectively, then 

$$\mathbf{x}_R^T E \mathbf{x}_L = 0$$
This basically says that the 3 vectors $x_R, x_L$ and $E$ lie on the same plane, known as the epipolar plane. In the diagram above its the plane created by O1, O2 and P.

### Fundamental Matrix ($F$)
The fundamental matrix is the parameters that are used in real life for calculating the estimated position of a feature from one camera to the other. Its a $3 \times 3$ matrix that represents the translation, rotation and the optical properties of both of the cameras in the stereo setup.

$$F = K_R^{-T} E K_L^{-1}$$
#### Where is the Fundamental Matrix used ?
Suppose the location of a feature in the left camera feed to be $p_L = [u_l, v_L, 1]^T$. The fundamental matrix is a $3 \times 3$ skew-symmetric matrix. 

Given the epipolar constraints we can say that there will be a line $l_R$ in the right camera feed that will be containing the feature. We can calculate

$$l_R = Fp_L$$
here $l_R$ will be a $3 \times 1$ matrix say $[a, b, c]^T$ that represents the line $au_R + bv_R + c = 0$ in the right camera feed. We can then plug in $u_R$ values and check for the respective feature. Therefore your feature matching can now just compare features along the 1D line instead of the entire 2D image.

##### Note 
The line $l_R$ is generally horizontal in a calibrated system, given that the cameras are vertically aligned and are not tilted relative to each other. But in scenarios where the cameras are relatively tilted or vertically misaligned the line $l_R$ can be slanted. 
This is corrected using a homeographic transformation that reprojects the images in a mathematically perfect virtual space.


### Depth Reconstruction
After calibration of the stereo setup the disparity map can be calculated. Disparity map ($d$) is the horizontal pixel coordinate distance between matching features.

$$d = u_L - u_R$$
To convert the disparity back into point clouds we compute the Reprojection Matrix ($Q$) 

$$\begin{bmatrix} X \\ Y \\ Z \\ W \end{bmatrix} = \begin{bmatrix} 1 & 0 & 0 & -c_x \\ 0 & 1 & 0 & -c_y \\ 0 & 0 & 0 & f \\ 0 & 0 & -1/b & (c_x - c_x')/b \end{bmatrix} \begin{bmatrix} u \\ v \\ d \\ 1 \end{bmatrix}$$

where,
- $b$: Baseline distance.
- $f$: Focal length.
- $c_x$: Principal point x-coordinate (left camera).
- $c_x'$: Principal point x-coordinate (right camera, usually equal to $c_x$ after rectification).
- To get the final metric 3D point, divide by the homogeneous coordinate $W$: $(X/W, Y/W, Z/W)$. Because $W = -d/b + (c_x - c_x')/b$, you can see that depth ($Z/W$) is inversely proportional to disparity ($d$).
