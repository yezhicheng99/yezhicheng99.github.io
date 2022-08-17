---
title: OpenCV相机标定
date: 2022-08-17 15:28:06
categories:
tags:
top_img:
---

本文介绍如何用OpenCV对相机进行标定，代码来源于 [OpenCV](https://docs.opencv.org/3.4/dc/dbb/tutorial_py_calibration.html) 官方教程，并对代码基于自己的理解做了详细的注释。

## 相机内参概念回顾

### 内参矩阵
$$
camera intrinsic matrix =  \begin{bmatrix}f_{x}&0&c_{x}\\0&f_{y}&c_{y}\\0&0&1\end{bmatrix} 
$$

$f_{x} = \alpha f$ 和 $f_{y} = \beta f$ 分别是相机在 $x$ 和 $y$ 轴方向上的缩变系数，$f$ 是相机焦距，$c_{x}$ 和 $c_{y}$ 分别是成像中心到图像左上角原点的位移，单位是像素。

### 径向畸变
径向畸变表现为三维空间中的直线在图像上变成曲线的现象。

$$
$$
\begin{matrix}
x_{distorted}=x(1+K_{1}r^{2}+K_{2}r^{4}+K_{3}r^{6}) \\
y_{distorted}=y(1+K_{1}r^{2}+K_{2}r^{4}+K_{3}r^{6})
\end{matrix}
$$
$$

### 切向畸变
切向畸变表现为三维空间中平行的两条直线在图像中变得不平行，原因是生产中镜片和成像平面不平行导致的。

$$
\begin{matrix}
x_{distored}=x+2p_{1}xy+p_{2}(r^{2}+2x^{2}) \\
y_{distored}=y+p_{1}(r^{2}+2y^{2})+2 p_{2}xy
\end{matrix}
$$

### 畸变系数向量
因此，通常关注的畸变可以用下面的五个参数表示：

$$
Distortion coefficients =  \begin{pmatrix}k_{1}&k_{2}&p_{1}&p_{2}&p_{3}\end{pmatrix} 
$$

## 标定代码（python版本）

```python
#!/usr/bin/env python

'''
camera calibration for distorted images with chess board samples
reads distorted images, calculates the calibration and write undistorted images

usage:
    calibrate.py [--debug <output path>] [--square_size] [<image mask>]

default values:
    --debug:    ./output/
    --square_size: 1.0
    <image mask> defaults to ../data/left*.jpg
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv

# local modules
from common import splitfn

# built-in modules
import os

def main():
    import sys
    import getopt
    from glob import glob

    args, img_mask = getopt.getopt(sys.argv[1:], '', ['debug=', 'square_size=', 'threads='])
    args = dict(args)
    args.setdefault('--debug', './output/')
    args.setdefault('--square_size', 1.0)
    args.setdefault('--threads', 4)
    if not img_mask:
        img_mask = '../data/left??.jpg'  # default
    else:
        img_mask = img_mask[0]

    img_names = glob(img_mask)
    debug_dir = args.get('--debug')
    if debug_dir and not os.path.isdir(debug_dir):
        os.mkdir(debug_dir)
    square_size = float(args.get('--square_size'))

    pattern_size = (9, 6)
    pattern_points = np.zeros((np.prod(pattern_size), 3), np.float32)
    pattern_points[:, :2] = np.indices(pattern_size).T.reshape(-1, 2)
    pattern_points *= square_size

    obj_points = []
    img_points = []
    h, w = cv.imread(img_names[0], cv.IMREAD_GRAYSCALE).shape[:2]  # TODO: use imquery call to retrieve results

    def processImage(fn):
        print('processing %s... ' % fn)
        img = cv.imread(fn, 0)
        if img is None:
            print("Failed to load", fn)
            return None

        assert w == img.shape[1] and h == img.shape[0], ("size: %d x %d ... " % (img.shape[1], img.shape[0]))
        found, corners = cv.findChessboardCorners(img, pattern_size)
        if found:
            term = (cv.TERM_CRITERIA_EPS + cv.TERM_CRITERIA_COUNT, 30, 0.1)
            cv.cornerSubPix(img, corners, (5, 5), (-1, -1), term)

        if debug_dir:
            vis = cv.cvtColor(img, cv.COLOR_GRAY2BGR)
            cv.drawChessboardCorners(vis, pattern_size, corners, found)
            _path, name, _ext = splitfn(fn)
            outfile = os.path.join(debug_dir, name + '_chess.png')
            cv.imwrite(outfile, vis)

        if not found:
            print('chessboard not found')
            return None

        print('           %s... OK' % fn)
        return (corners.reshape(-1, 2), pattern_points)

    threads_num = int(args.get('--threads'))
    if threads_num <= 1:
        chessboards = [processImage(fn) for fn in img_names]
    else:
        print("Run with %d threads..." % threads_num)
        from multiprocessing.dummy import Pool as ThreadPool
        pool = ThreadPool(threads_num)
        chessboards = pool.map(processImage, img_names)

    chessboards = [x for x in chessboards if x is not None]
    for (corners, pattern_points) in chessboards:
        img_points.append(corners)
        obj_points.append(pattern_points)

    # calculate camera distortion
    rms, camera_matrix, dist_coefs, _rvecs, _tvecs = cv.calibrateCamera(obj_points, img_points, (w, h), None, None)

    print("\nRMS:", rms)
    print("camera matrix:\n", camera_matrix)
    print("distortion coefficients: ", dist_coefs.ravel())

    # undistort the image with the calibration
    print('')
    for fn in img_names if debug_dir else []:
        _path, name, _ext = splitfn(fn)
        img_found = os.path.join(debug_dir, name + '_chess.png')
        outfile = os.path.join(debug_dir, name + '_undistorted.png')

        img = cv.imread(img_found)
        if img is None:
            continue

        h, w = img.shape[:2]
        newcameramtx, roi = cv.getOptimalNewCameraMatrix(camera_matrix, dist_coefs, (w, h), 1, (w, h))

        dst = cv.undistort(img, camera_matrix, dist_coefs, None, newcameramtx)

        # crop and save the image
        x, y, w, h = roi
        dst = dst[y:y+h, x:x+w]

        print('Undistorted image written to: %s' % outfile)
        cv.imwrite(outfile, dst)

    print('Done')


if __name__ == '__main__':
    print(__doc__)
    main()
    cv.destroyAllWindows()



```