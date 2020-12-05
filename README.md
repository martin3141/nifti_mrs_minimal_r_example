Minimal NIfTI MRS reader example for R
================

The purpose of this document is to demonstrate how [NIfTI
MRS](https://docs.google.com/document/d/1tC4ugzGUPLoqHRGrWvOcGCuCh_Dogx_uu0cxKub0EsM/edit?usp=sharing)
may be imported into the [`R`](https://www.r-project.org/) programming
language, and perform basic processing steps with just few lines of
code. Whilst a dedicated MRS analysis packages, such as
[`spant`](https://martin3141.github.io/spant/), are recommended for
typical MRS analysis tasks in [`R`](https://www.r-project.org/), this
example code is provided to aid software developers interested in
developing their own tools to process NIfTI MRS files.

Firstly we download an example file, available from the [NIfTI MRS
github repository](https://github.com/wexeee/mrs_nifti_standard), and
load the excellent [`RNifti`](https://github.com/jonclayden/RNifti)
library to read the data:

``` r
fname <- "csi.nii.gz"
if (!file.exists(fname)) {
  download.file("https://dl.dropboxusercontent.com/s/g5jwikq7btikdtt/csi.nii.gz?dl=0", mode = "wb", fname)
}
library(RNifti)
nifti_mrs <- readNifti(fname)
```

We might be interested to know the FID sampling frequency,

``` r
1 / pixdim(nifti_mrs)[4]
```

    ## [1] 4000

and spatial transform to map our MRS data onto other MR images:

``` r
xform(nifti_mrs)
```

    ##         [,1]    [,2] [,3]      [,4]
    ## [1,] -14.375   0.000  0.0  124.0140
    ## [2,]   0.000 -14.375  0.0  138.4644
    ## [3,]   0.000   0.000 12.5 -101.4674
    ## [4,]   0.000   0.000  0.0    1.0000
    ## attr(,"code")
    ## [1] 2

MRS specific information is stored in the NIfTI header extension (code
44) and formatted as JSON. Here we load the
[`jsonlite`](https://arxiv.org/abs/1403.2805) library and output the
first few items:

``` r
ext_str  <- extension(nifti_mrs, 44, "character")
library(jsonlite)
ext_json <- fromJSON(ext_str)
head(ext_json)
```

    ## $TransmitterFrequency
    ## [1] 49.86013
    ## 
    ## $ResonantNucleus
    ## [1] "31P"
    ## 
    ## $EchoTime
    ## [1] 0.0023
    ## 
    ## $RepetitionTime
    ## [1] 1
    ## 
    ## $InversionTime
    ## [1] 0
    ## 
    ## $MixingTime
    ## NULL

To plot a voxel we simply treat `nifti_mrs` as a multidimensional array,
with the first three dimensions being the x, y and z voxel coordinates
(8, 8, 12) and the fourth dimension as the FID. Once the FID has been
extracted we perform a Fourier transform and shift the data with
[`pracma::fftshift`](https://cran.r-project.org/web/packages/pracma/index.html)
to show 0 Hz in the center of the spectrum to conform with convention:

``` r
library(pracma)
fid  <- nifti_mrs[8, 8, 12, ]
spec <- fftshift(fft(fid))
plot(Re(spec), type = "l", xlab = "frequency [a.u.]", ylab = "intensity [a.u.]")
```

![](README_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

Finally, basic phasing and line-broading steps may be performed to aid
visual interpretation:

``` r
fid_proc  <- fid * exp(-2i - 5e-3 * seq(0, dim(nifti_mrs)[4] - 1))
spec_proc <- fftshift(fft(fid_proc))
plot(Re(spec_proc), type = "l", xlab = "frequency [a.u.]", ylab = "intensity [a.u.]")
```

![](README_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->
