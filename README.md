# Running the code

STEP-1: 
Please note that the files in the foler "Data" must be in the same directory as the files in the folder "Code" for the proper functioning of the code files.
If you wish to execute the ipynb files, make sure that the data files (.npy) are in the SAME DIRECTORY as the ipynb files.

STEP-2:
The files in the folder Code are as follows

review_dataset_feature_extraction.ipynb - this file extracts the feature vectors from the attacked and original images
data.preprocessing.ipynb - this file adds labels to the feature vectors 
final_svm.ipynb - for training and testing the svm on the data

Because the attacks and the feature extraction takes a lot of time. We have already kept the vectors (*.npy files) in the folder "Data"

STEP-3:
Once all the files have been taken out from the folder "Data" ad have been put in the same folder as the ipynb files, you can go to the file final_svm.ipynb and run each cell as shown in the demo which is in the end of our video.

# Video
video link- https://drive.google.com/drive/folders/16TPy5--0cJ-EKjfLnXUNyzv5Rf7RVKFj?usp=sharing


# Report

## Analytical Global Median Filtering Forensics  

**Authors:**  
- Vedansh Sharma (12242000)
- Shivangi Gaur (12241720)  
- Shubham Mahajan (12241730)  

*Date: May 6, 2025*  

---

## 1 Introduction  

In recent years, the field of digital image forensics has drawn attention due to its role in verifying the authenticity of images.  

This paper presents a new approach to detecting median filtering. It introduces a unique set of features based on the skewness and kurtosis of small image regions. These features help highlight subtle changes introduced by the filtering process.  

- **Passive image forensics** focuses on identifying signs of tampering without needing access to the original image. These methods typically rely on detecting statistical inconsistencies that hint at common types of forgery.  

- Lately, researchers have been working on detecting specific image editing operations such as median filtering, contrast adjustment, and resampling. Among these, **median filtering is particularly tricky**—it’s a popular nonlinear filter that smooths images while preserving edges, which makes it a go-to method for hiding traces of manipulation.  

- Compared to existing techniques, the proposed method achieves better accuracy, especially in challenging conditions like compressed JPEG images or low-resolution formats.  

- The paper introduces a novel feature set derived from **skewness and kurtosis histograms of image blocks** for detecting median filtering.  

### Skewness  
- Represents the asymmetry of the distribution.  
- Zero skewness indicates symmetry, while positive and negative skewness denote right-tail and left-tail distributions, respectively.  

### Kurtosis  
- Describes the distribution’s peakedness or flatness.  
- Absolute kurtosis can be positive or undefined, indicating the distribution’s tails’ weight relative to the center.  

---

## 2 Feature Set Construction  

The proposed 19-dimensional feature vector is built by examining statistical patterns that emerge when an image undergoes median filtering. These patterns manifest in terms of changes in skewness and kurtosis computed over small sliding blocks within the image.  

The process involves dividing both the original and median-filtered images into overlapping ξ × ξ blocks (commonly 3×3), then calculating local and global moment-based statistics.  

The feature vector aggregates four types of information:  

1. **Skewness Histogram Bin Heights (SBH):** 9 values indicating frequency of specific skewness levels across blocks.  
2. **Kurtosis Histogram Bin Heights (KBH):** 4 values capturing distribution of kurtosis values.  
3. **Statistical Moments of Skewness and Kurtosis (fMOM):** 4-element vector (variance and mean of skewness/kurtosis).  
4. **Same Intensity Block (SIB) Count (fSIB):** A scalar counting identical pixel-value blocks.  

Thus,  

SK = [ h<sup>SBH</sup> , h<sup>KBH</sup> , f<sup>MOM</sup> , f<sup>SIB</sup> ]

Total = **19 features**, 9 from SBH, 4 from KBH, 4 from MOM, and 1 from SIB.

---

## 3 Our Implementation  

Steps in Python:  

1. **Preprocessing**  
   - Convert to grayscale, apply median filter (3×3, 5×5).  
2. **Sliding Window**  
   - 3×3 stride-1 patches.  
3. **Per Block Features**  
   - Flatten → histogram (256 bins).  
   - Normalize → probability distribution.  
   - Sample at {0, 64, 128, 192}.  
   - Compute skewness & kurtosis.  
4. **Aggregation**  
   - Mean skewness, kurtosis.  
   - Histogram stats, variance, entropy, contrast.  
5. **Final Vector**  
   - 19-D per image.  
   - Concatenate original + filtered → **38-D vector**.  

---

## 4 Experiment Setup  

### 4.1 Dataset and Attacks  

UCID dataset used. Training/testing pairs include:  

- Median filtering (3×3, 5×5).  
- Average filtering.  
- Gaussian filtering (σ = 0.5).  
- Bicubic rescaling (×1.5).  
- JPEG compression (Q=90).  

Both **512×384** and **256×256** images tested.  

### 4.2 Experimental Procedure  

- Two-class classification (median vs others).  
- Same Intensity Blocks excluded.  
- Train: 75%, Test: 25%.  

### 4.3 Support Vector Machine (SVM)  

- Polynomial kernel (degree=3).  
- C = 1.0, probability estimates enabled.  
- Metrics:  
  - **PoE (1 - Accuracy)**  
  - **AUC**  

---

## 5 Experimental Results  

### 5.1 Median Filtering vs Other Manipulations  

**Table 1: Performance comparison for uncompressed images**  

| Implementation   | Resolution | Metric | MF3 vs AVG | MF3 vs GAU | MF3 vs JPEG | MF3 vs RES | MF5 vs AVG | MF5 vs GAU | MF5 vs JPEG | MF5 vs RES | MF35 vs ALL |
|------------------|------------|--------|------------|------------|-------------|------------|------------|------------|-------------|------------|-------------|
| Research paper   | 512x384    | Pe     | 0.009      | 0.0041     | 0.0078      | 0.0011     | 0.0067     | 0.0019     | 0.0034      | 0.0015     | 0.0075      |
|                  |            | AUC    | 0.9951     | 0.9986     | 0.9958      | 1.0        | 0.9981     | 0.9999     | 0.9971      | 1.0        | 0.9973      |
| Grp-15           | 512x384    | Pe     | 0.0075     | 0.0075     | 0.009       | 0.009      | 0.006      | 0.006      | 0.0075      | 0.0075     | 0.0075      |
|                  |            | AUC    | 0.9952     | 0.9992     | 0.995       | 0.9958     | 0.9949     | 0.9977     | 0.9941      | 0.9953     | 0.9942      |
| Research paper   | 256x256    | Pe     | 0.006      | 0.006      | 0.0075      | 0.0037     | 0.0022     | 0.003      | 0.0056      | 0.0011     | 0.0101      |
|                  |            | AUC    | 0.999      | 0.9966     | 0.9957      | 0.9976     | 0.9987     | 0.9991     | 0.9984      | 0.9986     | 0.9954      |
| Grp-15           | 256x256    | Pe     | 0.4343     | 0.4388     | 0.4239      | 0.4493     | 0.4358     | 0.4403     | 0.4254      | 0.4507     | 0.4222      |
|                  |            | AUC    | 0.7434     | 0.7149     | 0.7481      | 0.6095     | 0.8041     | 0.7812     | 0.8179      | 0.6834     | 0.7836      |

---

### 5.2 Post-JPEG Compressed Results  

**Table 2: Post-JPEG compressed median filtered vs original images**  

| Implementation   | Resolution | Pe     | AUC   | Pe     | AUC   |
|------------------|------------|--------|-------|--------|-------|
|                  |            | ORI vs MF3+Q |       | ORI vs MF5+Q |       |
| Research paper   | 512x384    | 0.0135 | 0.9932| 0.006  | 0.9956|
| Grp-15           | 512x384    | 0.2791 | 0.9858| 0.0657 | 0.9886|
| Research paper   | 256x256    | 0.0594 | 0.9637| 0.0299 | 0.9825|
| Grp-15           | 256x256    | 0.5418 | 0.4496| 0.5269 | 0.42  |

---

**Table 3: Median filtered images post-JPEG vs other manipulations**  

| Implementation   | Resolution | Metric | MF3+Q vs AVG | MF3+Q vs GAU | MF3+Q vs RES | MF5+Q vs AVG | MF5+Q vs GAU | MF5+Q vs RES | MF35+Q vs ALL |
|------------------|------------|--------|--------------|--------------|--------------|--------------|--------------|--------------|---------------|
| Research paper   | 512x384    | Pe     | 0.0105       | 0.0138       | 0.0123       | 0.0071       | 0.0093       | 0.0056       | 0.0179        |
|                  |            | AUC    | 0.994        | 0.9901       | 0.9909       | 0.9958       | 0.9934       | 0.9961       | 0.9882        |
| Grp-15           | 512x384    | Pe     | 0.2776       | 0.2776       | 0.2791       | 0.0642       | 0.0642       | 0.0657       | 0.0479        |
|                  |            | AUC    | 0.9579       | 0.9908       | 0.9883       | 0.9849       | 0.993        | 0.9898       | 0.9891        |
| Research paper   | 256x256    | Pe     | 0.0213       | 0.0389       | 0.0426       | 0.0265       | 0.0176       | 0.0217       | 0.0407        |
|                  |            | AUC    | 0.9884       | 0.9791       | 0.975        | 0.98         | 0.9875       | 0.9852       | 0.9754        |
| Grp-15           | 256x256    | Pe     | 0.5015       | 0.506        | 0.5164       | 0.4866       | 0.491        | 0.5015       | 0.482         |
|                  |            | AUC    | 0.6482       | 0.6157       | 0.492        | 0.6078       | 0.5775       | 0.4598       | 0.5922        |

---

**Table 4: Post-JPEG compressed median filtered vs post-JPEG original images**  

| Implementation   | Resolution | Pe     | AUC   | Pe     | AUC   |
|------------------|------------|--------|-------|--------|-------|
|                  |            | ORI+Q vs MF3+Q |       | ORI+Q vs MF5+Q |       |
| Research paper   | 512x384    | 0.0086 | 0.9962| 0.003  | 0.9994|
| Grp-15           | 512x384    | 0.2791 | 0.9647| 0.0657 | 0.9855|
| Research paper   | 256x256    | 0.0116 | 0.994 | 0.0037 | 0.9981|
| Grp-15           | 256x256    | 0.491  | 0.6504| 0.4761 | 0.6082|

---

## Key Observations  

1. High-res uncompressed images → strong detection.  
2. Low-res / JPEG → weaker detection.  
3. Python vs MATLAB → small technical differences.  
4. Future improvements: larger datasets, compression-aware features.  

---

## Contents  

- Introduction  
- Feature Set Construction  
- Our Implementation  
- Experiment Setup  
  - Dataset and Attacks  
  - Experimental Procedure  
  - Support Vector Machine (SVM) Classification  
- Experimental Results  
  - Median Filtering vs Other Manipulations  
  - Post-JPEG Compressed Results  



