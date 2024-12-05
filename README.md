---
title: "Watermark Embedding and Detection in Images Using DCT"
author: "Group 13"
date: "2024-10-12"
---

# Watermark Embedding, Attacking and Detecting in Images Using DCT

This file demonstrates the process of embedding and detecting watermarks in images using Discrete Cosine Transform (DCT). The workflow includes functions for embedding a watermark, adding noise to the watermarked image, and detecting the watermark from both the noisy and original watermarked images.

### **I. Libraries used in code**

1. **`numpy`** (`np`):
   - **Role**: Used for handling numerical data and arrays. It is the core of image manipulation since images are essentially arrays of pixel values.
   
2. **`cv2` (OpenCV)**:
   - **Role**: Used for reading, writing, and manipulating images, applying transformations, and working with color channels.
   
3. **`matplotlib.pyplot`** (`plt`):
   - **Role**: Used for visualizing images and plots, often used in a Jupyter notebook or interactive Python environment.

### **II. Display function to compare images by eyes**

### **III. Normalized Cross-Correlation Function:**
1. Formula
$$
\text{NCC} = \left| \frac{\frac{1}{N} \sum \left( (I_1 - \mu_{I_1}) \cdot (I_2 - \mu_{I_2}) \right)}{\sigma_{I_1} \cdot \sigma_{I_2}} \right|
$$
2. Where:
- $ I_1 $ and $ I_2 $: The matrix of two images being compared.
- $ \mu_{I_1} $ and $ \mu_{I_2} $: The mean pixel intensity of $ I_1 $ and $ I_2 $.
$$
\mu_{I_1} = \frac{1}{N} \sum I_1 \quad \text{and} \quad \mu_{I_2} = \frac{1}{N} \sum I_2
$$
- $ \sigma_{I_1} $ and $ \sigma_{I_2} $: The standard deviations of $ I_1 $ and $ I_2 $, respectively.
$$
\sigma_{I_1} = \sqrt{\frac{1}{N} \sum \left( I_1 - \mu_{I_1} \right)^2} \quad \text{and} \quad \sigma_{I_2} = \sqrt{\frac{1}{N} \sum \left( I_2 - \mu_{I_2} \right)^2}
$$
- $ N $: The total number of pixels in the image.

#### This function calculates the **absolute value of the normalized cross-correlation**, which measures the similarity between two images while normalizing for differences in brightness and contrast.

### **IV. DCT and IDCT Function:**
The **Discrete Cosine Transform (DCT)** converts a image from the spatial domain to the frequency domain. Here's the mathematical definition for a 2D DCT, which is commonly used in image processing:

#### **2D DCT Formula**
For an $ N \times M $ image $ f(x, y) $, the 2D DCT is defined as:

$$
F(u, v) = \frac{1}{\sqrt{N M}} \cdot C(u) \cdot C(v) \cdot \sum_{x=0}^{N-1} \sum_{y=0}^{M-1} f(x, y) \cdot \cos\left[\frac{\pi}{N} u \left(x + \frac{1}{2}\right)\right] \cdot \cos\left[\frac{\pi}{M} v \left(y + \frac{1}{2}\right)\right]
$$
---
#### **Inverse 2D DCT Formula**
The inverse operation (IDCT), to transform back to the spatial domain, is:

$$
f(x, y) = \frac{1}{\sqrt{N M}} \cdot \sum_{u=0}^{N-1} \sum_{v=0}^{M-1} C(u) \cdot C(v) \cdot F(u, v) \cdot \cos\left[\frac{\pi}{N} u \left(x + \frac{1}{2}\right)\right] \cdot \cos\left[\frac{\pi}{M} v \left(y + \frac{1}{2}\right)\right]
$$
---
#### **Normalization Factor**
The scaling factors $ C(u) $ and $C(v) $ are defined as:
$$
C(k) = 
\begin{cases} 
\sqrt{\frac{1}{2}}, & \text{if } k = 0 \\ 
1, & \text{if } k > 0 
\end{cases}
$$
---
#### **Explanation of Variables**
- $ f(x, y) $: Pixel intensity value at position $ (x, y) $ in the spatial domain.
- $ F(u, v) $: DCT coefficient at frequency $ (u, v) $.
- $ N, M $: Dimensions of the image (rows $ N $ and columns $ M $).
- $ x, y $: Spatial coordinates (pixel positions).
- $ u, v $: Frequency domain coordinates.
#### **This code uses the dct and idct functions from the cv2 library:**
**Syntax**
```python
cv2.dct(src, flags=None)
cv2.idct(src, flags=None)
```

**Parameters**
- **`src`**: The source array. It must be of type `np.float32`.
- **`flags`**: Optional flags. Currently, only forward DCT is implemented, so this is generally left as `None`.

**Returns**
- `dct():` A transformed array containing the DCT coefficients, also of type `np.float32`.
- `idct():` A transformed array in the spatial domain, reconstructed from the DCT coefficients, also of type `np.float32`.
### In this code, we want the image that has been embedded to be similar to the original. This means the embedded image should have colors like the original. Therefore, the image is split into its three color channels (Blue, Green, Red).
### **V. Embedding Function:**
#### Step by step: 
- Change a image from the spatial domain to the frequency domain using `dct()`.
- The watermark is embedded by adding its values.
- Reconstruct the image using `idct()`.
#### Explanation:

**1. Read the Images and Get Dimensions**
- The first step in the `embedded` function is to read both the cover image and the watermark. The cover image is read in color, while the watermark is read in grayscale.
  
  - **`cv2.imread(cover_data, cv2.IMREAD_COLOR)`**: Loads the cover image in color.
  - **`cv2.imread(watermark, cv2.IMREAD_GRAYSCALE)`**: Loads the watermark in grayscale.
  
  The function also extracts the height and width of both the watermark and the cover image for later use.

---

**2. Calculate the Watermark's Position**
- The position of where the watermark will be placed in the cover image is calculated using the `secret_key`.
  - **`start_x` and `start_y`**: These are calculated to determine where to place the watermark in the cover image based on the dimensions of both.
  - **Ensuring Bounds**: The position is adjusted to ensure that the watermark doesn’t extend beyond the edges of the cover image. This is done using `min` and `max` functions.

---
**3. Split the Cover Image into Channels**
- The cover image is split into its three color channels (Blue, Green, Red). Each channel will be processed independently.
  
  - **`cv2.split(cover_data)`**: This splits the image into its individual color components (BGR).

---

**4. Apply DCT (Discrete Cosine Transform)**
- The DCT is applied to each of the three color channels (Blue, Green, and Red).
  - **`cv2.dct(np.float32(cover_b))`**: Converts the Blue channel into the frequency domain, making it easier to embed the watermark in the frequency coefficients.
  - The same process is applied to the Green and Red channels.

---

**5. Embed the Watermark**
- The watermark is embedded by adding its values (scaled by the `alpha` parameter) to the DCT coefficients of the image. 
  - This is done for each pixel of the watermark in all three color channels.
  - **`alpha * watermark[j, i]`**: This scales the watermark’s pixel value, where `alpha` controls how noticeable the watermark is.

---

**6. Apply Inverse DCT (IDCT)**
- After the watermark has been embedded in the frequency domain, the Inverse DCT is applied to each channel (Blue, Green, Red) to bring the image back to the spatial domain.
  
  - **`cv2.idct(dct_b)`**: Converts the Blue channel back to the spatial domain (image space).
  - The same process is applied to the Green and Red channels.

---

**7. Reconstruct the Image**
- After IDCT, the values of the channels are clipped to make sure they are within the valid range (0-255), preventing any invalid pixel values.
  
  - **`np.clip(watermarked_b, 0, 255)`**: Clamps the pixel values to the valid range.
  
- Finally, the three channels are merged back into a single image.

  - **`cv2.merge([watermarked_b, watermarked_g, watermarked_r])`**: Merges the processed Blue, Green, and Red channels back together into a full image.

---

**8. Save the Watermarked Image**
- The final watermarked image is saved to the disk as a file called `watermarked_color.jpg`.

  - **`cv2.imwrite(output_path, np.uint8(watermarked_image))`**: Saves the final image after watermarking.

---
#### **Key Parameters**:
- **`cover_data`**: Path to the image which is prepare for embedded.
- **`watermark`**: Path to the watermark for embedded image.
- **`secret_key`**: Determines the watermark's position and security.
---

### **VI. Attacking Function (Noise)**:

1. **Generate Random Noise**:
   - The function first generates random noise using `np.random.randint(0, noise_level, image.shape, dtype='uint8')`. This creates an array of random integers with values between `0` and `noise_level`, having the same shape as the input image. The noise is generated with `uint8` (unsigned 8-bit integer) type to match typical image data types (0 to 255 for pixel values).
---
2. **Add Noise to the Image**:
   - The random noise is added to the original image using `np.add(image, noise)`. This operation adds the pixel values of the noise array to the corresponding pixels in the image.
---
3. **Clip the Values**:
   - The result is then clipped to ensure all pixel values are within the valid range of `[0, 255]` using `np.clip(noisy_image, 0, 255)`. This prevents any pixel values from going outside the valid range for an image (since pixel values in an image must be between 0 and 255).
---
4. **Return the Noisy Image**:
   - Finally, the function returns the noisy image with the added random noise.
---
### **VII. Detecting Function**:
#### Step by step: 
- Change embedded image from the spatial domain to the frequency domain using `dct()`.
- Compute the difference between the DCT coefficients of the watermarked and the original cover image.
- Divide the difference by the `alpha` value to recover the watermark’s original values.

#### Explanation
1. **Load the Images**:
   - Load the original (cover) image and the watermarked image using OpenCV.
---
2. **Get Image Dimensions**:
   - Get the height and width of the original image and calculate the watermark's position using the `secret_key`.
---
3. **Split into Color Channels**:
   - Split both images into Blue, Green, and Red color channels.
---
4. **Apply DCT (Discrete Cosine Transform)**:
   - Apply DCT to each color channel of both images to convert them into the frequency domain.
---
5. **Extract Watermark**:
   - Initialize arrays to store the extracted watermark for each color channel.
   - For each pixel in the watermark region, calculate the difference in DCT coefficients between the watermarked and original images, and divide by `alpha` to recover the watermark.
---
6. **Rebuild the Watermark**:
   - Average the color channels to get a grayscale watermark.
   - Clip pixel values to ensure they are within the range [0, 255] and convert to 8-bit format.
---
7. **Save the Extracted Watermark**:
   - Save the extracted watermark as an image.
---
8. **Return the Watermark**:
   - Return the extracted watermark as a NumPy array.
---
#### **Key Parameters**:
- **`original_data`**: Path to the original image.
- **`test_data`**: Path to the watermarked image.
- **`secret_key`**: Determines the watermark's position and security.
---
