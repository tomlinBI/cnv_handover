======================================
ACTDrugV6 CNV pipeline
======================================

-----------------
Purpose
-----------------

This document provides basic understand to current development process for ACTDrugV6 CNV, and how to execute the pipeline.

----

-----------------
Repo
-----------------

- https://github.com/ACTGenomics/actdrugv6-cnv 
- https://github.com/ACTGenomics/ACT-3rd-Gen-Pipeline 
- https://github.com/etal/cnvkit 

-----------------
Docker Image
-----------------

- `actgenomics/drugv6_plot <https://hub.docker.com/repository/docker/actgenomics/drugv6_plot/general>`_
- `CNVkit bioquay container <https://quay.io/repository/biocontainers/cnvkit?tab=tags&tag=0.9.12--pyhdfd78af_0>`_

-----------------
Other
-----------------

- Reading materials
    - `CNVkit PLos Publication <https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004873>`_
    - `CNVkit readthedocs <https://cnvkit.readthedocs.io/en/stable/pipeline.html>`_

- Division meeting progress:
    - `20241213_DivisionMeeting_BioDev.pptx <https://actgenomics.sharepoint.com/:p:/s/ACTGBioInfo-Bioinformatics/Efx6FVoBvq9FlASccETDdIsB4QOzI9e17-7fSFOe7QtT1w?e=x1KNID>`_
    - `20250213_DivisionMeeting_BioDev.pptx <https://actgenomics.sharepoint.com/:p:/s/ACTGBioInfo-Bioinformatics/EVRw1Z-jgzFItsUwy5bdOdsB0c7fqMtgXpYHBQVcYMNYjA?e=y1inPC>`_
    - `20250327_DivisionMeeting_BioDev.pptx <https://actgenomics.sharepoint.com/:p:/s/ACTGBioInfo-Bioinformatics/ESRav8FM3QBHhDWujjKBgGABaX4SgVPyi706XZw7qL5uhw?e=sF0EXK>`_
    - `20250417_DivisionMeeting_BioDev.pptx <https://actgenomics.sharepoint.com/:p:/s/ACTGBioInfo-Bioinformatics/Ec2XbfDIRJFJoWv5HzpNiRgBtER8pzvBiHyYnYobpzAsdA?e=WP5LcO>`_

----


-----------------
Files
-----------------

The ACTDrugV6 CNV pipeline is a component of the main workflow in ACT-3rd-Gen-Pipeline.

- **Subworkflow**: ``ACT-3rd-Gen-Pipeline/sub-workflows/CNVkit_subwf.nf``
- **Modules**: ``ACT-3rd-Gen-Pipeline/modules/CNV_modules.nf``
- **Configs**: ``ACT-3rd-Gen-Pipeline/config/illumina/PA052/PA052XNA_HS_FFPE_subwf_cnvkit_grch38.config``

Processes
~~~~~~~~~~~~~~

These processes are components of the CNVkit pipeline Subworkflow. 
Process run commands are written based on suggestion from CNVkit github.
For details of the processes and commands, please refer to official `CNVkit readthedocs <https://cnvkit.readthedocs.io/en/stable/pipeline.html>`_

- **CNVkitInputs**: Tuple UUID with files and metadata 
- **CNVkit_DefineRegions**: Define target and antitarget region bin size 
- **CNVkit_CalculateCNN**: Calculate read coverage 
- **CNVkit_GenerateReference**: Calculate PON 
- **CNVkit_FixNormalize**: Coverage data normalization 
- **CNVkit_CalculateSegment**: Calculate segment ratio 
- **CNVkit_CallCNV**: Predict CN gain/loss 
- **CNVkit_CustomPlot**: Visualise coverage data 

----

-----------------
Prepare data
-----------------

The input for CNVkit is processedBAM. 
At the moment, the data is obtained from SNV pipeline output: ``pre_annotation/Processed_Bam/*bam  and  *bam.bai``

Params files needs to be generated manually with the folowing JSON keys:

.. code-block:: JSON

    { 
        "inNormalProcessedBAM": 
        "inFinalVCF": 
        "inTumorPurity": 
        "normal_UUID": 
        "inEvalProcessedBAM": 
        "inEvalFinalVCF": 
        "eval_UUID": 
        "inEvalTumorPurity": 
        "publish_dir": 
    } 

.. note::

    - Because PoN has yet to be established therefore this format is used to build PoN and evaluate the sample in the same run. 
    - In the future, after PoN has been confirmed, there would only be one set of BAM, VCF, UUID, and TP.  
    - Config files would contain pre-defined target / antitarget / PoN files to use in the workflow as data channel.

----

-------------------
Workflow execution
-------------------

**PoN** - For PON building and evaluation at gene level

.. code-block:: console

    nextflow run /mnt/home/tomlin/Github/ACT-3rd-Gen-Pipeline/sub-workflows/CNVkit_subwf.nf \ 
        -c /mnt/home/tomlin/Github_repo/ACT-3rd-Gen-Pipeline/config/illumina/PA052/PA052XNA_HS_FFPE_subwf_cnvkit_grch38.config \ 
        -params-file [PATH to params]
        -entry PoN 

----

**PoN_LGR** - For PON building and evaluation at exon level for BRCA1/2

.. code-block:: console

    nextflow run /mnt/home/tomlin/Github/ACT-3rd-Gen-Pipeline/sub-workflows/CNVkit_subwf.nf \ 
        -c /mnt/home/tomlin/Github_repo/ACT-3rd-Gen-Pipeline/config/illumina/PA052/PA052XNA_HS_FFPE_subwf_cnvkit_grch38.config \ 
        -params-file [PATH to params]
        -entry PoN_LGR 

----

**CNVkit_BatchReference** - A wrapper for default CNVkit pipeline, given predefined reference cnn, target and antitarget regions in the config files 

**CNVkit_steps** - Same process as CNVkit_BatchReference, but each stage is written individually as different processes to allow customisation and debugging. 

**CNVkitPipeline** - Workflow wrapper for CNV pipeline, to be called in the main workflow. 

-----

----------------------
Area for improvements
----------------------

- **PON**
    - Adjust target region size on BED file for small regions
    - Filter probe region with high PON SD spread
    - Filter probe region with extreme read depth

- **Build SNP database**
    - Parse VCF using similar method of existing legacy pipeline SNP database methods
    - Include SNP correction to copy number calculates

- **Plot**
    - Further optimise plot visualisation, spacing colouring...etc
    - SNP plot

- **Summary table**
    - Discuss with BIO/MIS how the new data should be presented