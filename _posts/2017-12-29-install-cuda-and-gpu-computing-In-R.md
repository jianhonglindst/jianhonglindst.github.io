---
layout: post
title:  "(Ubuntu 16.04 LTS) Install CUDA And An Easy Sample For GPU Computing With R"
date:   2017-12-29 23:30:47 +0800
categories: jekyll update
---

此文記錄我如何安裝 `CUDA` 驅動程式的過程，最後會有一個簡單的 `GPU` 在 `R` 實現的範例。

**請注意!! 我的環境為 ubuntu 16.04 LTS**

本文內容
--------

-   CUDA 安裝
-   CUDA 路徑設定
-   GPU 簡單運算範例 with R

CUDA 安裝
---------

首先先到 [CUDA toolkit](https://developer.nvidia.com/cuda-toolkit) 下載驅動安裝檔，步驟如下

![](https://jianhonglindst.github.io/assets/2017-12-29-install-cuda-and-gpu-computing-In-R/0000_CUDA%20toolkit.png)

可以用終端機安裝，我這邊是直接對執行檔點兩下安裝

**接著安裝 `cuda`**

    sudo apt-get update
    sudo apt-get install cuda

CUDA 路徑設定
-------------

在 `Terminal` 中執行

    gedit ~/.bashrc

在 `.bashrc` 的最下面輸入下面四行 (這邊因為我是cuda-9.1版，若使用別的版本或有更新，請自行更改版本號)

    export CUDA_HOME=/usr/local/cuda-9.1
    export LD_LIBRARY_PATH=${CUDA_HOME}/lib64

    PATH=${CUDA_HOME}/bin:${PATH}
    export PATH

套用設定 (或直接重開機讓它生效)

    source ~/.bashrc

即完成 `CUDA` 路徑設定

GPU 簡單運算範例 with R
-----------------------

首先先安裝 `packages::gpuR` 開啟 `R` 輸入以下指令

    install.packages("gpuR")

接著試跑一段程式碼看看 `GPU` 的效果，以下程式碼完全來自網路上的範例: [gpuR example](https://rpubs.com/christoph_euler/gpuR_examples)

其內容為計算 *N* × *N* 的**矩陣相乘**所需的計算時間，其中 *N* = 2, 4, 8, 16, ..., 2048, 4096

``` r
library(gpuR)
```

    Number of platforms: 1
    - platform: NVIDIA Corporation: OpenCL 1.2 CUDA 9.1.84
      - context device index: 0
        - GeForce GTX 1080
    checked all devices
    completed initialization

``` r
result <- data.frame()

for (exponent in seq(2, 24, 2)){
  A = matrix(rnorm(2^exponent), nrow = sqrt(2^exponent))
  B = matrix(rnorm(2^exponent), nrow = sqrt(2^exponent))
  
  now <- Sys.time()
  gpuA = gpuMatrix(A, type = "double")
  gpuB = gpuMatrix(B, type = "double")
  gpuC = gpuA %*% gpuB
  gpu <- Sys.time() - now
  
  now <- Sys.time()
  C = A %*% B
  classic <- Sys.time() - now
  
  now <- Sys.time()
  vclA = vclMatrix(A, type = "double")
  vclB = vclMatrix(B, type = "double")
  vclC = vclA %*% vclB
  vcl <- Sys.time() - now
  
 result <- rbind(result, c(nrow(A), classic, gpu, vcl)) 
}
colnames(result) <- c("nrow", "time_classic", "time_gpu", "time_vcl")
```

計算結果如下

       nrow time_classic time_gpu time_vcl
    1     2      0.00002  0.18472  0.02049
    2     4      0.00001  0.02328  0.02282
    3     8      0.00001  0.00964  0.00860
    4    16      0.00001  0.00881  0.00947
    5    32      0.00002  0.01088  0.00885
    6    64      0.00010  0.00957  0.00884
    7   128      0.00082  0.00988  0.00880
    8   256      0.00556  0.01053  0.00995
    9   512      0.06189  0.01697  0.01144
    10 1024      0.38631  0.04674  0.02119
    11 2048      3.62639  0.25846  0.10911
    12 4096     31.34916  1.23133  0.42269

圖表顯示如下

``` r
library(ggplot2)

ggplot(data = result, mapping = aes(x = nrow)) +
  geom_line(mapping = aes(y = time_classic, group = 1, color = "CPU")) +
  geom_line(mapping = aes(y = time_gpu, group = 1, color = "GPU with RAM")) +
  geom_line(mapping = aes(y = time_vcl, group = 1, color = "GPU with internal RAM")) +
  theme_light() +
  theme(panel.border = element_blank(),
        legend.position = c(0.3, 0.85),
        legend.text = element_text(size = 14),
        plot.title = element_text(hjust = 0.5, size = 14, color = "black"),
        axis.text.x = element_text(angle = 00, hjust = 1, size = 14, color = "black"),
        axis.text.y = element_text(angle = 00, hjust = 1, size = 14, color = "black"),
        axis.title.x = element_text(angle = 00, hjust = 0.5, size = 14, color = "black"),
        axis.title.y = element_text(angle = 90, hjust = 0.5, size = 14, color = "black")) +
  labs(x = "Dimesion of Matrix", y = "Computing Time (sec)") +
  scale_color_manual(name = "Type", values = c("CPU" = "black", "GPU with RAM" = "red", "GPU with internal RAM" = "blue"))
```

![](https://jianhonglindst.github.io/assets/2017-12-29-install-cuda-and-gpu-computing-In-R/unnamed-chunk-8-1.png)

馬上就感受到 GPU 的威力惹!!!

參考資訊
--------

1.  [NVIDIACUDA Toolkit Documentation](http://docs.nvidia.com/cuda/cuda-getting-started-guide-for-linux/index.html#post-installation-actions)
2.  [CUDA toolkit 下載](https://developer.nvidia.com/cuda-toolkit)
3.  [gpuR example](https://rpubs.com/christoph_euler/gpuR_examples)
