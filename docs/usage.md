Usage
=====

How to use DeSide



****

**DeSide have three steps to deconvolution. Here are two ways to use DeSide, one can skip the first two steps and use the provided well-trained model to predict cell fractions directly, if you do not want to train DeSide model by yourself. Another way is run the  program  step by step from the first step of DeSide (DeSide Simulation), during the process, you can reform the single cell dataset, retrain the model by yourself.**



The workflow of deconvolution by DeSide consists of three steps:

1.  DeSide Simulation
2.  DeSide Training
3.  DeSide Prediction



## DeSide Simulation

### a. Using the single cell dataset we provided

In this step, you can generate simulated bulk gene expression profiles (GEPs) based on single cell RNA-seq (scRNA-seq) dataset. We provided a merged dataset contains 6 scRNA-seq datasets so far: `merged_6_sc_datasets.h5ad`. You can use all (or part) of 6 scRNA-seq datasets when you call function `simulated_bulk_gep_generator()`. Generated GEPs can be used as `training set` for training DeSide model or `test set` for testing the model performance after training finished.

### b. Preparing single cell dataset by yourself

If you want to use other scRNA-seq datasets to train DeSide, you can follow our workflow to preprocess single cell datasets and merge them together. Python package `Scanpy` was used heavily in our workflow.

- Preprocessing a single dataset: [Here is an example of how i process my scRNA-seq dataset(s)]().
- Merging multiple datasets together: [Here is an example of how i process my scRNA-seq dataset(s)]().

Also read the section "1. Integrating single cell RNA-seq dataset" in supplementary material for more details.

[Explanation of parameters in this function can be found](). 

#### Example code for generating a training set

```python
from deside.simulation import simulated_bulk_gep_generator

simulated_bulk_gep_generator(n_per_gradient=100, 
                             result_dir="path/result/bulk_simulate/", n_sample_cell_type=5000, 
                             merged_sc_dataset_file_path="path/result/sc_datasets/file.h5ad ",                                  
                             dataset_name="Mixed_N100", sc_dataset_id=["all"], 
                             generated_sc_dataset_dir="path/result/bulk_simulate/sc_data/")
```
#### Example code for generating a test set

```python
from deside.simulation import simulated_bulk_gep_generator

simulated_bulk_gep_generator(n_per_gradient=5, 
                             result_dir="path/result/bulk_simulate/", n_sample_cell_type=5000, 
                             merged_sc_dataset_file_path="path/result/sc_datasets/file.h5ad ", 
                             cell_types=['Cancer Cells', 'CD4 T', 'CD8 T', 'Fibroblasts', 'DC', 'NK'], 
                             gradient_range={'Cancer Cells': (50, 100), 'CD8 T': (1, 30), 
                                             'Fibroblasts': (80, 100)},
                             dataset_name="Test_set", sc_dataset_id=["all"], 
                             generated_sc_dataset_dir="path/result/bulk_simulate/sc_data/")
```

### Input files

The provided single cell dataset, `merged_6_sc_dataset.h5ad`,  is a `h5ad` file (see more details about this file format from [anndata](https://anndata.readthedocs.io/en/latest/index.html)) which contains 6 scRNA-seq datasets and 11 cell types. You can download this file from [our datasets file]() (location: "datasets/single_cell/merged_6_sc_datasets.h5ad")

- merged_6_sc_dataset.h5ad

  - `obs` contains the information of single cell samples such as sample id, cell type and dataset id.

  - `var` contains gene names of samples. 

  - `X` is a matrix of gene expression profiles (GEPs, log space) with the shape of n_sample  by n_gene.

  - 6 scRNA-seq dataset ids: "hnscc_cillo_01", "pdac_pengj_02", "hnscc_puram_03", "pdac_steele_04", "luad_kim_05", "nsclc_guo_06"

  - 11 cell types: B Cells, CD4 T, CD8 T, Cancer Cells, DC, Endothelial cells, Fibroblasts, Macrophages, Mast Cells, NK, Neutrophils.

    

    This file can be accessed by:

    ```python
    import anndata as an
    
    merged_sc_dataaset = an.read_h5ad('path/to/merged_6_sc_dataset.h5ad')
    print(merged_sc_dataset.X.shape)  # (67870, 11785)
    ```

    

### Output files

File `simu_bulk_exp_xx_log2cpm1p.h5ad` can be used as input data of function `dnn_training()` for training model or function `dnn_predict()` for testing model performance.

`xx` in the following file name is same as the parameter `dataset_name`.

- generated_11_cell_type_n5000_all.h5ad: this is an intermediate file before simulating bulk GEPs. This file contains the generated single cell dataset that contains 11 cell types from all 6 scRNA-seq datasets and each cell type has 5000 samples if the parameters was set by `sc_dataset_id = ['all'], n_sample_cell_type = 5000`;
- generated_frac_xx_.csv: Cell fraction of each cell type per simulated bulk sample;
- simu_bulk_exp_xx_selected_cell_id.csv: Cell id of cells in merged_6_sc_dataset which were used in this simulated bulk data;
- simu_bulk_exp_xx_CPM.txt [optional]: GEPs of simulated bulk data were given in counts per million (CPM) if `save_tpm=True`.
- simu_bulk_exp_xx_log2cpm1p.csv: GEPs of simulated bulk data were given in log2(CPM+1). 
- simu_bulk_exp_xx_log2cpm1p.h5ad: Contains the information of both gene expression ( given in  log2(CPM+1) ) and cell fraction of each cell type.

Example files: You can downloda this files from [our datasets file]() (location: datasets/simulated_bulk_cell_dataset).



## DeSide Training

Once the training set was set-up, you can start training a DeSide model with function `dnn_training()`.  [See details on this function]().  



### Example code

```python
from deside.decon_cf import dnn_training

dnn_training("DeSide", h5ad_file_path="path/result/bulk_simulate/simu_bulk_exp_Mixed_N100_log2cpm1p.h5ad",
             result_dir="path/result/DeSide_training/")
```
### Input files

In this step, you can use the simulated bulk GEPs (training set) were saved in `.h5ad` file. This file usually generated from  `DeSide Simulation` step. The training set we used can be downloaded from [our datasets file]() (location: datasets/simulated_bulk_cell_dataset/simu_bulk_exp_Mixed_N100_log2cpm1p.h5ad)  

`xx` in the following file name is same as the parameter `dataset_name` in function `simulated_bulk_gep_generator`.

- simu_bulk_exp_xx_log2cpm1p.h5ad:	Contains information both of GEPs (count in log2(CPM+1) ) and fraction of cell types.

  

### Output files

You will get 5 files in this step and details are show below. The whole result directory (same as parameter `result_dir`) can be used in the `DeSide prediction` step. `xx` in the following file name is same as the parameter `model_name` in function  `dnn_training()`.

- celltypes.txt:	Cell types which were used in the training process.
- genes.txt:	Genes which were used in the training process.
- history_reg.csv: ...
- loss.png: ...
- model_xx.h5: Well-trained model.

Example files: You can downloda this file from [our datasets file]() (location: datasets/well_trained_model).     #目录指定到最好的模型目录？



## DeSide prediction

After training by simulated bulk GEPs, You can predict the cell fractions of each cell type by well-trained model with function `dnn_prediction()`. [See details on this function]().  



you can use the following command to perform the deconvolution:

### Example code

```python
from deside.decon_cf import dnn_prediction

dnn_prediction(model_name="DeSide", model_dir="path/result/DeSide_training/",
               cancer_type='ACC',
               bulk_exp_fp="path/ACC_TPM.csv",
               result_dir="path/DeSide_prediction/", exp_type="TPM")
```

### Input files

In this step, you can use the whole result directory of `result_dir` in `DeSide train` step as the input of  parameter `model_dir`, **[or you can download our best well_trained model file from datasets file]() (location: datasets/well_trained_model/DeSide) and use the real directory in your computer.** Besides, bulk GEPs need to be delivered which were given in transcripts per million (TPM) or log2(TPM + 1). It should be separated by ','  and saved in a `.csv` file. The shape of this file should be m×n, where m is the number of features (genes) and n is the number of samples. You can download an example file from [our datasets file]() (location: datasets\TCGA\tpm). If you have not this format of bulk GEPs file, DeSide can only provide functionality to create a file of correct format from TCGA read count data(S) (**?**). Have a look at the [Data Processing]() section for instructions on how to use this function. 

- `model_dir`: ...
- xx.csv: Bulk GEPs.



### Output files

You will get a `.csv` file which contains cell fraction on each cell type per sample. `xx` in the following file name is same as the parameter `model_name` in function  `dnn_prediction()`. **?**

- cancer_purity_merged_xx_predicted_result.csv: Predicted cell fractions, sample by cell type.

Example files: You can download this file from [our results file]() (location: results/predicted_cell_fraction).     



## Data Processing

We provide function `read_counts2tpm()`  to create a file of correct format from TCGA read count data(s), [See details on this function]().

### Example code

```python
from deside.bulk_cell import read_counts2tpm

read_counts2tpm(read_counts_file_path='path/ACC_htseq.counts.csv', file_name_prefix='ACC',
                annotation_file_path='path/gencode.gene.info.v22.tsv', result_dir='path/result/bulk_GEPs/')
```



### Input files

In this step, TCGA read counts data (htseq.counts) in a .csv file (separated by ",")  or .txt file (separated by "\t") should be prepared. This file has the shape of m×n, where m is the number of features (genes) and n is the number of samples. You can download an example file from [our datasets file]() (location: datasets\TCGA\merged_data). Besides, file `gencode.gene.info.v22.tsv` for .. is also need, you can download this file from [our datasets file]() (location: datasets\TCGA\gencode.gene.info.v22.tsv)

- xx.csv: TCGA read counts data. 
- gencode.gene.info.v22.tsv:	?

### Output files

You will get 3 files transfer from TCGA read counts data, details  show below. `xx` in the following file name is same as the parameter `file_name_prefix` in function  `read_counts2tpm()`.

- xx_htseq.counts.csv:	TCGA read counts data(?).
- xx_TPM.csv: GEPs given in TPM.
- xx_log2tpm1p.csv: GEPs given in log2(TPM + 1).

Example files: You can download this file from [our results file]() (location: datasets\TCGA\tpm).     



  

##  Directory tree structure of `datasets`

We provided the datasets file with following directory tree structure, [you can download this file here](). Our datasets file contains all input datas and some intermediate output datas in our process by used DeSide. 



datasets/
├── cancer_purity/
│   └── cancer_purity.csv
├── simulated_bulk_cell_dataset/
│   ├── generated_frac\_\*\*.csv
│   ├── simu_bulk_exp\_\*\*\_log2cpm1p.csv
│   ├── simu_bulk_exp\_\*\*\_log2cpm1p.h5ad
│   ├── simu_bulk_exp\_\*\*\_selected_cell_id.csv
│   ├── sc_dataset/
│   │   ├── generated_10_cell_type_n5000_pdac_pengj_02_pdac_steele_04.h5ad
│   │   ├── generated_11_cell_type_n5000_all.h5ad
│   │   ├── generated_8_cell_type_n5000_hnscc_cillo_01_hnscc_puram_03.h5ad
│   │   └── generated_9_cell_type_n5000_luad_kim_05.h5ad
│   └── test_set/
│       ├── generated_frac\_\*\*.csv
│       ├── simu_bulk_exp\_\*\*\_CPM.txt
│       ├── simu_bulk_exp\_\*\*\_log2cpm1p.csv
│       ├── simu_bulk_exp\_\*\*\_log2cpm1p.h5ad
│       └── simu_bulk_exp\_\*\*\_selected_cell_id.csv
├── single_cell/
│   ├── count_by_cell_type_and_dataset2.csv
│   ├── merged_6_sc_datasets.h5ad
│   ├── merged_6_sc_datasets.rar
│   └── merged_single_cell_dataset_sample_info.csv
├── TCGA/
│   ├── gdc_sample_sheet_10_tumors.tsv
│   ├── gencode.gene.info.v22.tsv
│   ├── merged_data/
│   │   └── \*\*/
│   │       └── merged\_\*\*\_htseq.counts.csv
│   └── tpm/
│       └── \*\*/
│            ├── \*\*\_htseq.counts.csv
│            ├── \*\*\_log2tpm1p.csv
│            └── \*\*\_TPM.csv
└── well_trained_model/
    └── \*\*/
        └── \*\*/
            ├── celltypes.txt
            ├── genes.txt
            ├── history_reg.csv
            ├── loss.png
            └── model\_\*\*.h5



## Directory tree structure of `results`

We provided the results file with following directory tree structure, [you can download this file here](). Our results file contains final results in our process by used DeSide. 



results/
├── predicted_cell_fraction/
│   └── \*\*/
│       ├── cell_fraction_by_DeSide\_\*\*/
│       │   ├── cancer_purity_merged_DeSide\_\*\*\_predicted_result.csv
│       │   ├── CD8A_vs_predicted_CD8 T_proportion.png
│       │   ├── CPE_vs_predicted_1-others_proportion.png
│       │   ├── CPE_vs_predicted_Cancer Cells_proportion.png
│       │   ├── pred_cell_frac_before_decon.png
│       │   └── y_predicted_result.csv
│       └── cell_fraction_by_Scaden\_\*\*/
│           ├── cancer_purity_merged_Scaden\_\*\*\_predicted_result.csv
│           ├── CD8A_vs_predicted_CD8 T_proportion.png
│           ├── CPE_vs_predicted_1-others_proportion.png
│           ├── CPE_vs_predicted_Cancer Cells_proportion.png
│           ├── pred_by_model_m1024.csv
│           ├── pred_by_model_m256.csv
│           ├── pred_by_model_m512.csv
│           ├── pred_cell_frac_before_decon.png
│           └── y_predicted_result.csv
└── test_set_pred/
    └── Test_set\*\*/
        ├── DeSide\_\*\*/
        │   ├── Cancer Cells_true_vs_predicted_1-others_proportion.png
        │   ├── Cancer Cells_true_vs_predicted_Cancer Cells_pred_proportion.png
        │   ├── CD8 T_true_vs_predicted_CD8 T_pred_proportion.png
        │   ├── CD8A_vs_predicted_CD8 T_true_proportion.png
        │   ├── model_performance_evaluation.csv
        │   ├── y_predicted_result.csv
        │   ├── y_true_vs_absolute_error_deside.png
        │   ├── y_true_vs_absolute_error_deside_1-others.png
        │   ├── y_true_vs_y_pred_deside.png
        │   └── y_true_vs_y_pred_deside_1-others.png
        └── Scaden\_\*\*/
            ├── Cancer Cells_true_vs_predicted_Cancer Cells_pred_proportion.png
            ├── CD8 T_true_vs_predicted_CD8 T_pred_proportion.png
            ├── CD8A_vs_predicted_CD8 T_true_proportion.png
            ├── model_performance_evaluation.csv
            ├── pred_by_model_m1024.csv
            ├── pred_by_model_m256.csv
            ├── pred_by_model_m512.csv
            ├── y_predicted_result.csv
            ├── y_true_vs_absolute_error_average.png
            ├── y_true_vs_absolute_error_m1024.png
            ├── y_true_vs_absolute_error_m256.png
            ├── y_true_vs_absolute_error_m512.png
            ├── y_true_vs_y_pred_average.png
            ├── y_true_vs_y_pred_m1024.png
            ├── y_true_vs_y_pred_m256.png
            └── y_true_vs_y_pred_m512.png
