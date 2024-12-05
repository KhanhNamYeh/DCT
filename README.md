\documentclass{article}
\usepackage{amsmath}
\usepackage{graphicx}

\title{Watermark Embedding and Detection in Images Using DCT}
\author{Group 13}

\begin{document}

\maketitle

\section*{Watermark Embedding, Attacking and Detecting in Images Using DCT}
\section*{I. Libraries used in code}

\begin{itemize}
    \item \textbf{`numpy`} (`np`): Used for handling numerical data and arrays. It is the core of image manipulation since images are essentially arrays of pixel values.
    \item \textbf{`cv2`} (OpenCV): Used for reading, writing, and manipulating images, applying transformations, and working with color channels.
    \item \textbf{`matplotlib.pyplot`} (`plt`): Used for visualizing images and plots, often used in a Jupyter notebook or interactive Python environment.
\end{itemize}

\section*{II. Display function to compare images by eyes}

\section*{III. Normalized Cross-Correlation Function}

\textbf{Formula:}

\[
\text{NCC} = \left| \frac{\frac{1}{N} \sum \left( (I_1 - \mu_{I_1}) \cdot (I_2 - \mu_{I_2}) \right)}{\sigma_{I_1} \cdot \sigma_{I_2}} \right|
\]

Where:

\[
\mu_{I_1} = \frac{1}{N} \sum I_1, \quad \mu_{I_2} = \frac{1}{N} \sum I_2
\]

\[
\sigma_{I_1} = \sqrt{\frac{1}{N} \sum \left( I_1 - \mu_{I_1} \right)^2}, \quad \sigma_{I_2} = \sqrt{\frac{1}{N} \sum \left( I_2 - \mu_{I_2} \right)^2}
\]

Where \( I_1 \) and \( I_2 \) are the matrices of the two images being compared, \( \mu \) represents the mean pixel intensity, and \( \sigma \) represents the standard deviation.

This function calculates the absolute value of the normalized cross-correlation, which measures the similarity between two images while normalizing for differences in brightness and contrast.

\section*{IV. DCT and IDCT Function}

The Discrete Cosine Transform (DCT) converts an image from the spatial domain to the frequency domain. The mathematical definition for a 2D DCT, which is commonly used in image processing, is:

\[
F(u, v) = \frac{1}{\sqrt{N M}} \cdot C(u) \cdot C(v) \cdot \sum_{x=0}^{N-1} \sum_{y=0}^{M-1} f(x, y) \cdot \cos\left(\frac{\pi}{N} u \left(x + \frac{1}{2}\right)\right) \cdot \cos\left(\frac{\pi}{M} v \left(y + \frac{1}{2}\right)\right)
\]

The inverse 2D DCT (IDCT) is:

\[
f(x, y) = \frac{1}{\sqrt{N M}} \cdot \sum_{u=0}^{N-1} \sum_{v=0}^{M-1} C(u) \cdot C(v) \cdot F(u, v) \cdot \cos\left(\frac{\pi}{N} u \left(x + \frac{1}{2}\right)\right) \cdot \cos\left(\frac{\pi}{M} v \left(y + \frac{1}{2}\right)\right)
\]

Where \( C(k) \) is defined as:

\[
C(k) = 
\begin{cases} 
\sqrt{\frac{1}{2}}, & \text{if } k = 0 \\ 
1, & \text{if } k > 0
\end{cases}
\]

\section*{V. Embedding Function}

\textbf{Step-by-step:}
\begin{enumerate}
    \item Change the image from the spatial domain to the frequency domain using DCT.
    \item Embed the watermark by adding its values to the DCT coefficients.
    \item Reconstruct the image using IDCT.
\end{enumerate}

\textbf{Explanation:}

\begin{enumerate}
    \item \textbf{Read the Images and Get Dimensions}: The cover image is read in color, while the watermark is read in grayscale.
    
    \item \textbf{Calculate the Watermark's Position}: The position of where the watermark will be placed in the cover image is calculated using the secret key.

    \item \textbf{Split the Cover Image into Channels}: The cover image is split into its three color channels (Blue, Green, Red).

    \item \textbf{Apply DCT}: The DCT is applied to each of the three color channels.

    \item \textbf{Embed the Watermark}: The watermark is embedded by adding its values (scaled by the $\alpha$ parameter) to the DCT coefficients of the image.

    \item \textbf{Apply Inverse DCT (IDCT)}: After embedding the watermark in the frequency domain, apply the IDCT to each channel to bring the image back to the spatial domain.

    \item \textbf{Reconstruct the Image}: The image channels are clipped to ensure they remain within valid pixel intensity values, and the final image is reconstructed.
\end{enumerate}

\section*{VI. Attacking Function (Noise)}

\begin{enumerate}
    \item \textbf{Generate Random Noise}: The function first generates random noise using the following code:
    \[
    \text{noise} = \text{np.random.randint}(0, \text{noise\_level}, \text{image.shape}, \text{dtype='uint8'})
    \]
    This creates an array of random integers with values between $0$ and \text{noise\_level}, having the same shape as the input image. The noise is generated with `uint8` (unsigned 8-bit integer) type to match typical image data types (0 to 255 for pixel values).
    
    \item \textbf{Add Noise to the Image}: The random noise is added to the original image using:
    \[
    \text{noisy\_image} = \text{np.add}(\text{image}, \text{noise})
    \]
    This operation adds the pixel values of the noise array to the corresponding pixels in the image.
    
    \item \textbf{Clip the Values}: The result is clipped to ensure all pixel values are within the valid range of $[0, 255]$ using:
    \[
    \text{np.clip}(\text{noisy\_image}, 0, 255)
    \]
    This prevents any pixel values from going outside the valid range for an image (since pixel values in an image must be between 0 and 255).
    
    \item \textbf{Return the Noisy Image}: Finally, the function returns the noisy image with the added random noise.
\end{enumerate}

\section*{VII. Detecting Function}

\textbf{Step-by-step:}

\begin{enumerate}
    \item Change the embedded image from the spatial domain to the frequency domain using \texttt{dct()}.
    \item Compute the difference between the DCT coefficients of the watermarked and the original cover image.
    \item Divide the difference by the $\alpha$ value to recover the watermark’s original values.
\end{enumerate}

\textbf{Explanation:}

\begin{enumerate}
    \item \textbf{Load the Images}: Load the original (cover) image and the watermarked image using OpenCV.

    \item \textbf{Get Image Dimensions}: Get the height and width of the original image and calculate the watermark's position using the \texttt{secret\_key}.
    
    \item \textbf{Split into Color Channels}: Split both images into Blue, Green, and Red color channels.

    \item \textbf{Apply DCT (Discrete Cosine Transform)}: Apply DCT to each color channel of both images to convert them into the frequency domain.

    \item \textbf{Extract Watermark}: Initialize arrays to store the extracted watermark for each color channel. For each pixel in the watermark region, calculate the difference in DCT coefficients between the watermarked and original images, and divide by $\alpha$ to recover the watermark.

    \item \textbf{Rebuild the Watermark}: Average the color channels to get a grayscale watermark. Clip pixel values to ensure they are within the range $[0, 255]$ and convert to 8-bit format.

    \item \textbf{Save the Extracted Watermark}: Save the extracted watermark as an image.

    \item \textbf{Return the Watermark}: Return the extracted watermark as a NumPy array.
\end{enumerate}

\textbf{Key Parameters}:

\begin{itemize}
    \item \textbf{`original\_data`}: Path to the original image.
    \item \textbf{`test\_data`}: Path to the watermarked image.
    \item \textbf{`secret\_key`}: Determines the watermark's position and security.
\end{itemize}

\end{document}
