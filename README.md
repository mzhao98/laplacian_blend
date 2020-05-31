# Laplacian Blending

Consider a scenario in which you wanted to blend two images. Suppose you start with these two images: one of an apple and one of an orange.
![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap1.png)

If you wanted to blend two, into a half-apple, half-orange image, you could take the left half of the apple and combine it side-by-side with the right half of the orange.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap2.png)

But this image looks terrible. It’s incredibly unnatural-looking, and the line split in the middle of the fruit sticks out like a sore thumb. What we want is an image more like this.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap3.png)

Note the smooth transition that blends the apple and orange. Although this still doesn’t look realistic, this is of course what we’d prefer over the unblended image. Laplacian pyramid image blending will allow us to do this:

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap4.png)

Laplacian pyramid image blending isn’t limited to blending down the middle of an image. It can help blend contours, and merge images in all sorts of ways.

So how does it work?
# The objective in Laplacian Pyramid Blending:
## Given 2 input images and an image mask, blend the images in a seamless way.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap6.png)

The Laplacian Pyramid structure is as follows. It looks confusing, but is actually very straightforward.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap7.png)

The Laplacian pyramid builds from the Gaussian pyramid. The Gaussian pyramid serves to represent information, in this case an image, at different scales, at each of which the information from the original is preserved. In short, the Gaussian pyramid is a sequence of images, starting with the original, the original shrunk by ½, the original shrunk by ¼, and so on. At every transition of the pyramid, we want to downscale the image by a factor of ½.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap8.png)

To reduce the scale of the image by ½, Gaussian pyramids combine smoothing with down-sampling. First, the image is smoothed using a Gaussian filter, and then is down sampled by 1/2. To down sample by ½, just take every other pixel in each row and column.
You might be asking yourself, why not just down sample by ½, and skip the smoothing step? It still would result in a ½ scaled image. However, the problem lies in aliasing.

When you down-sample by skipping over pixels, you are making it possible to lose important information from the image. Specifically, depending on the granularity of your down sampling, you might lose all areas of contrast. Take the below image for example. In checkerboard A and B, the downsampling occurs at enough of a frequency so that the pattern is preserved. However, in checkerboard C, the downsampling becomes at too low of a frequency and the sampled image is all black. In board D, the downsampled image isn’t representative of the original checkerboard pattern at all.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap9.png)

Smoothing reduces the maximum frequency of image features, and reduces sharp contrasts and fast changes that subsampling-only would miss. Gaussian smoothing the image by convolving it with a Gaussian filter essentially performs a low-pass filter over the image. The purpose of a low pass filter over an image is to retain the low-frequency information (think low contrast locations) within an image, while reducing the high frequency information (think edges). Smoothing removes the edges!

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap10.png)

To perform the convolution, we apply this convolution operation at every pixel, using mirror or zero padding as appropriate.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap11.png)

Using the Gaussian pyramid, at each scale, the image size decreases by a factor of 2, and the scale of the aggregate gaussian smoothing filter applied increases by a factor of 2. Simply, the image gets smaller and more blurry at every level of the pyramid.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap12.png)

Next, we construct the Laplacian pyramid. To construct the Laplacian at a given level i, we first upsample the downscaled image of the next smallest level from the Gaussian pyramid. We subtract the upsampled image from the image in the Gaussian pyramid at the current level.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap13.png)

To upsample the smaller image, we upsample with interpolation, which is upsampling followed by filtering. Upsampling by a factor of 2 inserts 1 zero between every pixel of the original image. Then we perform lowpass filtering on this image with a Gaussian filter, to remove imaging artifacts and abnormalities inserted by the upsampling. We then subtract the upsampled image (blue box) from the image at the current scale in the Gaussian pyramid. These images will be the same size after upsampling, and the result is the Laplacian at the current scale.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap14.png)

The Laplacian is essentially a high pass filter. It captures only the details and edges of the images. Intuitively, we can image that subtracting a smoothed image with no details from an image with details, will leave only the details.
We continue with the rest of the Laplacian pyramid.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap15.png)


For both of the input images: the Apple and the Orange, we must build a Laplacian pyramid. Then, we construct a Gaussian pyramid for the mask.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap16.png)


![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap17.png)

Applying the Gaussian filter to the subsampled mask makes the image blend smooth. The mask serves to help us combine the Laplacian pyramids for the two inputs. Using an alpha+(1-alpha) combination, at each scale, we multiply the mask by Image A’s Laplacian, and then multiply Image B’s Laplacian by (1-the mask) and sum the two.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap18.png)

Finally, we reconstruct the original image at each scale by adding the combined Laplacian to the original Gaussian-resized images multiplied by their respective masks. This is akin to adding the details lost while resizing (combined Laplacian) back into the Gaussian-smoothed image, combined according to the desired form (the mask). We perform this operation repeatedly, upsampling the result and adding the result to the combined Laplacian, until we have our fully blended image at the original scale.
Here we have the complete algorithm for this image blend:
![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap19.png)

To visualize what this algorithm is doing, the final Laplacian pyramid looks like this:

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap20.png)

Laplacian Pyramids are a pretty awesome way to blend images. With Laplacian image blending, we can do all sorts of things, like blend celebrity faces! We can also use different masks that allow more complex combinations.

![alt text](https://github.com/mzhao98/laplacian_blend/blob/master/images/lap5.png)

I can't figure out how to center these images, sorry! :)
