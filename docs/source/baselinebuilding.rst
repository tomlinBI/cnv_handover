======================================
Legacy CNV Baseline Building handover
======================================

-----------------
Purpose
-----------------
This page provide a tutorial of how to build CNV baseline and its preliminary evaluation.
This applies to all legacy pipeline projects.

-----------------
Repos
-----------------
- `Bitbucket <https://bitbucket.org/actgenomics/actcnv_baseline_automation/src/master/>`_
- `Github <https://github.com/ACTGenomics/cnv_baselinebuild_legacy>`_

-----------------
Docker Images
-----------------
- `actgenomics/cnv_bsl_build <https://hub.docker.com/repository/docker/actgenomics/cnv_bsl_build>`_

----------------------------
Past memeber tutorial videos
----------------------------
These files are found in TP-FS01 storage

- /2024SF/BIA&BI/Bioinformatics/Team/TsungHsun/Handover/Baseline automation 
- /2024SF/BIA&BI/Bioinformatics/Team/August/[PanelRD]_BaselineBuilding 

------
Steps
------

Prepare raw data
=================
- Prepare a delimited list of UUIDs
- Normal sample with Lv1 and Lv2 data from production pipeline

.. image:: _img/bslbuild_rawdata.png
    :width: 600px
    :align: center
    :alt: Example of UUID list

----

- Generate a physical copy in local directory and group the files into one folder

.. code-block:: console
    
    # find files
    python3 /mnt/home/tomlin/dev_script/find_files.py \ 
        -f1 [Lv1 DIR] \ 
        -f2 [Lv2 DIR] \ 
        -i samplelist \ 
        -o [output DIR]

    # create directory for build input
    cd [output DIR]; mkdir coverage_arm coverage_cnv rawbackup
    
    # here the example output directory is '/mnt/home/tomlin/handover/test'
    cp /mnt/home/tomlin/handover/test/Lv1/*/CoverageAnalysis/*/*EXON*.xls coverage_cnv 
    cp /mnt/home/tomlin/handover/test/Lv1/*/CoverageAnalysis/*/*SNP*.xls coverage_arm 
    cp /mnt/home/tomlin/handover/test/Lv2/SNV/*/*/annotation_result/*rawbackup.xlsx rawbackup 

    # create directory for build output
    mkdir -p output/{selection_cnv,selection_arm,build_cnv,build_arm,SNPdb_build} 


- Coverage directory is needed for different \*.amplicon.cov.xls for ONCOCNV baseline
- Rawbackup directory contains \*.rawbackup.xls annotation table for SNP database


Prepare building files
=======================
Building files are usually the same every time, therefore stored in repository.

- `PA037 <https://github.com/ACTGenomics/cnv_baselinebuild_legacy/tree/develop/Panels/PA037/building_files>`_
- `PA031 <https://bitbucket.org/actgenomics/actcnv_baseline_automation/src/master/Panels/PA031/building_files>`_
- `Onco2M7 <https://bitbucket.org/actgenomics/actcnv_baseline_automation/src/master/Panels/Onco2M7pv5/building_files>`_

Explanation of different build files:

- cutoff_files: contain parameters to test for different high/low amplification efficienty and amplicon CVs.

.. note:: 

    Final cutoff files should only contain one set of parameters.

.. image:: _img/bslbuild_cutoff.png
    :width: 600px
    :align: center
    :alt: Example of cutoff files

-----

- bed: the original BED files of amplicon inserts (all the designed amplicon for the panel)

.. image:: _img/bslbuild_bed.png
    :width: 600px
    :align: center
    :alt: Example of BED file

-----

- pseudo: the pseudo gene list of the panel

.. image:: _img/bslbuild_pseudo.png
    :width: 600px
    :align: center
    :alt: Example of pseudo gene list

-----

- lgr_rawbed: BED files with exon information in amplicon name

.. image:: _img/bslbuild_lgrbed.png
    :width: 600px
    :align: center
    :alt: Example of LGR BED

-----

- genelist: CNV gene list (same as GeneInfo file)

.. image:: _img/bslbuild_genelist.png
    :width: 600px
    :align: center
    :alt: Example of CNV gene list

-----

- lgr_genelist: Gene list but with exon level info for BRCA1/2

.. image:: _img/bslbuild_lgrgenelist.png
    :width: 600px
    :align: center
    :alt: Example of lgr gene list

-----

- cyto: cytoband file label p and q arms of chromosome for ArmCNV

.. image:: _img/bslbuild_cytoband.png
    :width: 600px
    :align: center
    :alt: Example of cytoband file

-----

Prepare config json
========================

Prepare deployment YAML
========================

Deploy container
========================


Build Baseline
========================


Evaluate Baseline
========================

