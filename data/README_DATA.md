# 📊 Data Privacy and Access Policy

Due to ethical considerations, patient privacy protections, and the Data Use Agreements (DUA) governing clinical genomic data, the original dataset from the **International Warfarin Pharmacogenetics Consortium (IWPC)** cannot be publicly shared in this repository.

## 📥 How to Obtain the Real Dataset
Researchers and developers who wish to reproduce the study or train the models from scratch can request official access to the IWPC dataset directly through PharmGKB:
👉 **[PharmGKB - IWPC Pharmacogenetics Data](https://www.pharmgkb.org/downloads)**

## 🧪 Using the Sample Data (`sample_data.csv`)
For testing the machine learning pipelines and ensuring the code runs without errors, we have provided a `sample_data.csv` file. 
* This file contains **100% synthetic, randomized dummy data**.
* It exactly matches the schema (column names and data types) of the original dataset.
* You can safely run the provided Jupyter notebooks (`01_Data_Preprocessing.ipynb`, etc.) using this file.
