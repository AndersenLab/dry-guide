# Andersen Lab Image Analysis Pipeline
## Implemented using CellProfiler

### CellProfiler Directory Structure (With Example Files)

```
CellProfiler
  ├── batch_files
      ├── 20191119_example_batch_20201018.h5
  ├── metadata
      ├── 20191119_example_metadata_20201018.csv
  ├── pipelines
      ├── 20191119_example.cpproj
      ├── sample_pipelines
  ├── projects
      ├── 20191119_example
        ├── raw_images
        ├── output_data
            ├── 20191119_example_data_1603047856
                ├── CellProfiler-Analysis_20191119_example_data_1603047856run1
                ├── Logs
                ├── ProcessedImages
                ├── OverlappingWorms_Data
                ├── NonOverlappingWorms_Data
  ├── scripts
      ├── generate_metadata.R
      ├── run_cellprofiler.sh
      ├── cellprofiler_parallel.sh
      ├── check_run_cellprofiler.sh
      ├── aggregate_cellprofiler_results.R
  ├── well_masks
      ├── wellmask_98.png
  ├── worm_models
      ├── Adult_N2_HB101_100w.xml
      ├── L1_N2_HB101_100w.xml
      ├── L2L3_N2_HB101_100w.xml
      ├── L4_N2_HB101_100w.xml
      ├── WM_FBZ_control.xml
      ├── WM_FBZ_dose.xml
      ├── high_dose_worm_model.xml
```

### To recreate CellProfiler (CP) on QUEST:
1) Navigate to the directory where you wish to clone CP:
```
ssh -X user@quest.it.northwestern.edu
cd [desired directory]
```
2) Clone CP repository
```
git clone https://github.com/AndersenLab/CellProfiler.git
```
3) Download CP Docker images and convert to Singularity images within CP directory:
```
cd CellProfiler
module load singularity
singularity pull docker://cellprofiler/cellprofiler:3.1.9
singularity pull docker://cellprofiler/cellprofiler:4.0.3
```

### To execute the CP pipeline on QUEST:
#### Full instructions [here](https://docs.google.com/document/d/1IfnxFeNoG0JehJquMV_DorH5KpUsRJAg2OYdpBFppcM/edit):
1) Navigate to the CP Directory:
```
cd CellProfiler
```
2) Generate metadata CP file:
```
Rscript scripts/generate_metadata.R 20191119_example
```
3) Download metadata file and create CellProfiler pipeline *locally*. 
4) Upload properly named batch file and CellProfiler pipeline to appropriate QUEST directories. (see file structure above)
5) Collect measurements using CellProfiler:
```
bash scripts/run_cellprofiler.sh projects/20191119_example batch_files/20191119_example_batch_[date].h5
bash scripts/check_run_cellprofiler.sh projects/20191119_example batch_files/20191119_example_batch_[date].h5 [timestamp]
```
6) Aggregate measurement data:
```
Rscript scripts/aggregate_cellprofiler_results.R 20191119_example_data_[timestamp] 20191119_example_metadata_[date].csv [output_info]
```
At this point, a summarized .RData file will be available for download, containing measurement outputs corresponding to each worm model used in the pipeline:
```
CellProfiler
  ├── projects
      ├── 20191119_example
        ├── raw_images
        ├── output_data
            ├── 20191119_example_summary_data
                ├── CellProfiler-Analysis_20191119_example_data_[output_info]_[timestamp].RData
                ├── Logs
                ├── ProcessedImages
                ├── OverlappingWorms_Data
                ├── NonOverlappingWorms_Data
```
These data can be analyzed using the [R/easyXpress](https://github.com/AndersenLab/easyXpress) package 
