# Homework 1: Histogram Equalization and Specification

> 611121202 資工碩一 何昀軒

## Table of Contents

- [Homework 1: Histogram Equalization and Specification](#homework-1-histogram-equalization-and-specification)
  - [Table of Contents](#table-of-contents)
  - [作業要求](#作業要求)
  - [Generate images with different brightness \& contrast](#generate-images-with-different-brightness--contrast)
    - [Original image and histogram](#original-image-and-histogram)
    - [Adjustment method](#adjustment-method)
    - [Result in different brightness and contrast values](#result-in-different-brightness-and-contrast-values)
  - [Contrast Stretching \& Equalization](#contrast-stretching--equalization)
    - [Contrast Stretching](#contrast-stretching)
    - [Result in Min-Max Stretching](#result-in-min-max-stretching)
    - [Histogram Equilization](#histogram-equilization)
    - [Result in Histogram Equilization](#result-in-histogram-equilization)
    - [Comparison \& Discussion](#comparison--discussion)
  - [Equalization and Specification](#equalization-and-specification)
    - [Quarter moon image and histogram](#quarter-moon-image-and-histogram)
    - [Histogram Equilization](#histogram-equilization-1)
    - [Result in Histogram Equilization](#result-in-histogram-equilization-1)
    - [Histogram Specification](#histogram-specification)
    - [Result in Histogram Specification](#result-in-histogram-specification)
    - [Comparison \& Discussion](#comparison--discussion-1)




## 作業要求
1.   Please prepare four images: (a) dark; (b) bright; (*c*) low contrast; (d) high contrast, like the following images.
![](https://i.imgur.com/jKxQG5l.png)
    (a)  Describe how you generate your four images. Show the images together with the corresponding histograms.
    (b)  Find the transfer curves to enhance these images, show the results and histograms as  well. Please  take  into  account  of  the  methods  of  contrast  stretching  and equalization and be sure to give the comparison & discussion.

2. Select one noon picture from the internet, for example,[link](https://www.google.com/search?q=quarter+moon&tbm=isch&ved=2ahUKEwi0uuXx6drvAhUNg5QKHa9_DNQQ2-cCegQIABAA&oq=quarter+moon&gs_lcp=CgNpbWcQAzICCAAyBggAEAcQHjIGCAAQBxAeMgYIABAHEB4yBggAEAcQHjIGCAAQBxAeMgYIABAHEB4yBAgAEB4yBAgAEB4yBAgAEB46BAgAEBM6CAgAEAcHhATULvkAVj17QFgzvEBaABwAHgAgAF_iAGZBpIBAzAuN5gBAKABAaoBC2d3cy13aXotaW1nwAEB&sclient=img&ei=npJkYLTzF42G0gSv_7GgDQ&bih=789&biw=1440&hl=zh-TW.)
Most area of the noon is nearly dark because there is no light there.
Enhance the selected image by using equalization and specification, compare the results and give a discussion.



## Generate images with different brightness & contrast

### Original image and histogram

此為原始圖片以及直方圖
![](https://i.imgur.com/BHmXhaZ.png)


### Adjustment method

我用來調整亮度及對比度的方法是給定兩個變數，亮度值及對比度值，再對每個像素值進行調整，調整的公式為 $contrast * pixel + brightness$

brightness: 對每個像素值加上亮度值，範圍應在-100~100之間，否則圖片在經過調整後會太亮或是太暗
contrast: 對每個像素值乘上對比度值，如果value在0~1.0之間，會產生較低對比度的圖片，而大於1.0的話，則會產生較高對比度的圖片

```python
def transform(brightness, contrast):
    img = np.zeros((height, width), dtype=np.uint8)
    for x in range(width):
        for y in range(height):
            pixel = original[y, x]
            img[y, x] = max(0, min(255, contrast * pixel + brightness))
    return img
```

以下是我調整各個圖片所使用的值:

|            | dark | bright | low contrast | high contrast |
|:----------:|:----:|:------:|:------------:|:-------------:|
| brightness | -60  |   50   |      0       |       0       |
|  contrast  |  1   |   1    |     0.4      |      1.2      |


### Result in different brightness and contrast values

![](https://i.imgur.com/hjDPsP1.png)

## Contrast Stretching & Equalization

### Contrast Stretching

對比度拉伸是透過線性縮放來增強圖片的對比度，使用的方法是Min-Max Stretching，此方法會將輸入圖片的最低值映射到0，最高值映射到255，而其他中間值則根據以下公式重新分配新的intensity values


${I_{new} = 255 \times \dfrac{I_{Input}-I_{Min}}{I_{Max}-I_{Min}}}$


```python
def minmax_stretching(img):
    img_contrast_stretching = np.zeros((height, width), dtype=np.uint8)
    # get the min and max pixel value of the image
    min = np.min(img)
    max = np.max(img)
    # Traverse the pixel of the image and apply Min-Max Stretching formula
    for i in range(height):
        for j in range(width):
            img_contrast_stretching[i,j]  = 255 * (img[i,j] - min) / (max - min)
    return img_contrast_stretching
```

### Result in Min-Max Stretching

![](https://i.imgur.com/74Nqk0N.png)


### Histogram Equilization

直方圖均衡化是透過重新分配像素值來增強圖片對比度，需要計算原始圖片的直方圖、累積分布函數(cumulative distribution function, CDF)以及均衡化後的像素值，再將均衡化後的像素值到原始圖片上

```python
def histogram_equalization(img):
    # calculates the histogram of the image
    hist, bins = np.histogram(img.flatten(), 256, [0, 256])
    # calculates the cumulative histogram(cdf)
    cum_hist = np.cumsum(hist)
    # normalize the cdf
    cum_hist_normalized = cum_hist * float(hist.max()) / cum_hist.max()
    # create a masked array to mask 0 value
    cdf_m = np.ma.masked_equal(cum_hist,0)
    # normalized the masked array
    cdf_m = (cdf_m - cdf_m.min())*255/(cdf_m.max()-cdf_m.min())
    # Fill 0 in the masked position and convert to uint8
    lut = np.ma.filled(cdf_m,0).astype('uint8')
    # transform the input image
    return lut[img]
```

### Result in Histogram Equilization
![](https://i.imgur.com/LgRHoyM.png)


### Comparison & Discussion

#### 對比度拉伸

透過拉伸圖片的直方圖範圍，圖片的Grayscale會分散到整個範圍
從輸出的圖片來看，對比度拉伸用於低對比度的圖片時效果較好

#### 直方圖均衡化
通過重新分配圖片中的Grayscale，使其會均勻分布在整個範圍中，來達到增強圖片的細節及對比度
從輸出的圖片來看，直方圖均衡化綜合表現較佳


## Equalization and Specification

### Quarter moon image and histogram

使用的月亮照片及其直方圖如下

![](https://i.imgur.com/Yyh6r6R.png)


### Histogram Equilization

使用與上方說明的直方圖均衡化同樣的方法進行操作

```python
def histogram_equalization(img):
    # calculates the histogram of the image
    hist, bins = np.histogram(img.flatten(), 256, [0, 256])
    # calculates the cumulative histogram(cdf)
    cum_hist = np.cumsum(hist)
    # normalize the cdf
    cum_hist_normalized = cum_hist * float(hist.max()) / cum_hist.max()
    # create a masked array to mask 0 value
    cdf_m = np.ma.masked_equal(cum_hist,0)
    # normalized the masked array
    cdf_m = (cdf_m - cdf_m.min())*255/(cdf_m.max()-cdf_m.min())
    # Fill 0 in the masked position and convert to uint8
    lut = np.ma.filled(cdf_m,0).astype('uint8')
    # transform the input image
    return lut[img]
```


### Result in Histogram Equilization

![](https://i.imgur.com/Hi7pqbT.png)


### Histogram Specification
又稱為histogram matching，是一種將輸入的圖片映射到另一個指定輸出範圍的方法。

此處我建立一個目標直方圖hist_target，其中直方圖的值是由mean為128，variance為24的高斯函數產生

接著計算原始圖片的直方圖和CDF，並將CDF進行normalized

為了將原始圖片轉換為符合目標直方圖的圖片，需要建立一個lut，將圖片中的每個像素值映射到新的像素值。

把目標直方圖與normalized後的CDF進行插值，得到一個lut，lut[i]表示原始圖片中像素值為i的像素對應的新像素值。

最後，利用OpenCV提供的LUT函數將原始圖片的每個像素值映射為對應的新像素值，得到一張符合目標直方圖的圖片。




```python
def specification(img):
    
    hist_target = np.zeros((256,), dtype=np.float32)
    for i in range(256):
        hist_target[i] = np.exp(-(i - 128)**2 / (2 * 24**2)) / np.sqrt(2 * np.pi * 24**2)
    hist_target = cv2.normalize(hist_target, None, alpha=0, beta=255, norm_type=cv2.NORM_MINMAX)
    hist_target = np.round(hist_target).astype(np.uint8)

    hist, bins = np.histogram(img.flatten(), 256, [0, 256])
    cdf = hist.cumsum()
    cdf_normalized = cdf * hist_target.max() / cdf.max()

    lut = np.interp(hist_target, np.arange(256), cdf_normalized).astype(np.uint8)
    return cv2.LUT(img, lut)
```

### Result in Histogram Specification

![](https://i.imgur.com/QAH3WhD.png)


### Comparison & Discussion

根據原始圖片的直方圖來看，大約有一千萬的像素值為0，一千兩百萬的像素值為1，而圖片的尺寸為5304 X 7952，也就是42177408個像素，代表圖片中約有53%的範圍是沒有足夠的資訊去進行增強的，猜測式這個原因，造成使用兩種增強方式的效果都不是很好


#### 直方圖均衡化

因為圖片過暗，直方圖偏向左側，經過重新分配Grayscale，造成過度曝光，反而看不出圖片的細節

#### 直方圖規定化
效果比直方圖均衡化更佳，但對於較暗的部分仍然看不到過多的細節，也有可能是生成的高斯函數不適合這張圖，如過替換函式或是使用其他圖片直接進行matching也許會有較佳的效果