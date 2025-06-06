# 🍅 RED TEAM: Run baseline MIA methods

Workflow for running and evaluating with baseline MIA methods:
1. [**Prerequisites**](#prerequisites):
2. [**Configuration of baseline methods**](#baseline-mia-methods-and-configuration):
3. [**Running baseline methods**](#running-the-baseline-mia-methods):
4. [**Evaluation & baseline performances**](#evaluation):

## Prerequisites

### Activate the environment

```bash
micromamba activate <environment>
```

### Generate synthetic datasets to launch membership inference
In order to re-produce the MIA methods, you first need some synthetic datasets. You can refer to [Blue team home page](/experiments/track_i/blue_team/) for detailed instructions. 


## Baseline MIA methods and configuration
Assuming you generated synthetic datasets using the blue team's pipeline, now let's move on how you can run the baseline methods. We provide some of the baselines available in [DOMIAS package](https://github.com/holarissun/DOMIAS),

- MC (`MC`)
- LOGAN (`LOGAN-D1`)
- GAN-Leaks (`gan_leaks`)
- GAN-Leaks (calibrated) (`gan_leaks_cal`)
- DOMIAS (KDE) (`domias_kde`)

- In order to run these methods,  ``attack_model`` parameter  inside the ``config.yaml`` is **by default** set as ``domias_baselines``. This configuration will run the above attacks against the generator defined in the ``generator_config``. 

- Please be reminded that you need to put `config.yaml` in the same directory you are running your experiment. 

- DOMIAS, GAN-leaks calibrated and LOGAN require a reference dataset, a dataset that reflect the true data distribition and not utilized during generative process. **We provide a reference dataset for TCGA-COMBINED dataset only.**

- :chart_with_upwards_trend:  You are free to use relevant public datasets as a reference set in case your method depends on it. 

## Running baseline MIA methods

### Example 1: Running an attack on a synthetic data generated by Multivariate

In the below example, ``config.yaml`` runs ``domias_baselines`` attack models on the synthetic dataset generated by ``multivariate`` method for the given configuration  ``noise_0.5`` on ``TCGA-BRCA`` dataset. 

This assumes the existence of the synthetic data saved in the path ``/path/to/project/data_splits/TCGA-BRCA/synthetic/multivariate/noise_0.5``. You can modify the config according to your needs. 


```bash
.
.
.
dataset_config:
  name: "TCGA-BRCA"
  membership_label_col: "membership_label"
  
attack_model: "domias_baselines"

generator_config: 
  model_name: "multivariate"
  experiment_name: "noise_0.5"
  
domias_baselines_config:
  taxonomy: "bb"
```

In the provided baseline generator methods, there are total 5 datasets created by the stratified 5-fold setting. Let's assume we lauch the attack on the first split: 

```bash

# path to synthetic data if you generate it using the workflow provided for Blue Team
#synthetic_data_pth = /path/to/project/data_splits/TCGA-BRCA/synthetic/multivariate/noise_0.5

python {src_dir}/mia/red_team.py run-mia 
            /path/to/synthetic/data/  + "synthetic_data_split_1.csv"
            /path/to/dataset/ + "TCGA-BRCA_primary_tumor_star_deseq_VST_lmgenes.tsv"
            "experiment_synthetic_data_1"
            --mmb_labels_file /path/to/example/label/csv/ + "synthetic_data_1_gt.csv"
            #--reference_file {reference_path} 
```
- You can download [running_baseline_example.zip](/experiments/track_i/red_team/1_mia/running_baseline_example.zip), that contains `synthetic_data_split_1.csv` and `synthetic_data_1_gt.csv` files for your convenience. 

- The script will generate an `evaluation_results.csv` under `/{home_dir}/results/mia/{dataset_name}/{attacker_name}/{generator_model}/{experiment_name}/{mia_experiment_name}` directory, 
     - i.e. `/{home_dir}/results/mia/TCGA-BRCA/domias_baselines/multivariate/noise_0.5/synthetic_data_1`


- Since TCGA-BRCA dataset doesn't have a separate reference file we omitted the optional argument  ``--reference_file``. 

- In case ``--mmb_labels_file`` optional argument is not available, the code only saves the predicted membership scores. 


## Evaluation

Classification metrics, accuracy, AUC, and AUPR, is utilized to evaluate attack performances. We report here the MIA performances against some of the baseline generators, using the default parameter values provided in the [config.yaml](/experiments/track_i/blue_team/2_generation/config.yaml). 

### TCGA-BRCA
#### Multivariate

| Method      | Accuracy   | AUCROC     | Average Precision |
|-------------|------------|------------|-------------------|
| MC          | 0.515152   | 0.521906   | 0.806972          |
| gan_leaks   | 0.511846   | 0.524526   | 0.812951          |

#### CVAE

| Method      | Accuracy | AUCROC   | Average Precision |
|-------------|----------|----------|--------------------|
| MC          | 0.614325 | 0.656251 | 0.843067          |
| gan_leaks   | 0.614325 | 0.723706 | 0.921273          |


### TCGA-COMBINED

#### Multivariate

| Method        | Accuracy | AUCROC  | Average Precision |
|---------------|----------|---------|-------------------|
| LOGAN_D1      | 0.500208 | 0.500172| 0.800439          |
| MC            | 0.510941 | 0.515112| 0.803992          |
| domias_kde    | 0.499931 | 0.499167| 0.799945          |
| gan_leaks     | 0.446264 | 0.514199| 0.807170          |
| gan_leaks_cal | 0.527319 | 0.562164| 0.836836          |

#### CTGAN

| Method         | Accuracy  | AUCROC    | Average Precision |
|----------------|-----------|-----------|--------------------|
| LOGAN_D1       | 0.498358  | 0.500121  | 0.800026           |
| MC             | 0.501874  | 0.502864  | 0.800275           |
| domias_kde     | 0.500578  | 0.499929  | 0.800366           |
| gan_leaks      | 0.463752  | 0.501731  | 0.800338           |
| gan_leaks_cal  | 0.501874  | 0.502757  | 0.801166           |
