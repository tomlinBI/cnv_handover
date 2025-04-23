======================================
Legacy CNV pipeline handover
======================================

-----------------
Purpose
-----------------

Provide a tutorial of how to deploy and troubleshoot during maintenance and testing of Illumina and Torrent legacy CNV pipeline


-----------------
Files
-----------------

- YAML

.. code-block:: console 
    
    /mnt/home/tomlin/dockercompose/

- Testing data

.. code-block:: console

    /mnt/BI3/Team_workdir/tom_workdir/Validation_input/


-----------------
Bitbucket Repos
-----------------

- `Illumina CNV Integration <https://bitbucket.org/actgenomics/illumina_cnv_integration/src>`_

- `Torrent ACTOnco2M7 <https://bitbucket.org/actgenomics/actcnv_onco2m7_ldt/src/master/>`_

- `Torrent ACTDrugV4 <https://bitbucket.org/actgenomics/actcnv_drug_ldt/src/master/>`_


-----------------
Docker Images
-----------------
- `actgenomics/ilmn_cnv <https://hub.docker.com/repository/docker/actgenomics/ilmn_cnv/general>`_

- `actgenomics/ldt_torrent_actcnv_onco <https://hub.docker.com/repository/docker/actgenomics/ldt_torrent_actcnv_onco/general>`_

- `actgenomics/ldt_torrent_actcnv_drug <https://hub.docker.com/repository/docker/actgenomics/ldt_torrent_actcnv_drug/general>`_


-----------------
Deployment
-----------------
Use the following docker compose template to initiate container


**Illumina** - `v1.11.3 (54022ed) <https://bitbucket.org/actgenomics/illumina_cnv_integration/src/v1.11.3/>`_ 

.. code-block:: console

    docker-compose -f /mnt/home/tomlin/dockercompose/ilmn_testing.yml up -d

.. image:: _img/deployment_ilmn.png
    :width: 600px
    :align: center
    :alt: Successful deployment of Illumina CNV container

----

**Torrent Onco** - `v3.2.9 (1a5b490) <https://bitbucket.org/actgenomics/actcnv_onco2m7_ldt/src/v3.2.9/>`_

.. code-block:: console

    docker-compose -f /mnt/home/tomlin/dockercompose/ACTOnco_testing.yml -d

.. image:: _img/deployment_onco.png
    :width: 600px
    :align: center
    :alt: Successful deployment of ACTOnco CNV container

----

**Torrent DrugV4** - `v3.0.9 (5ef3649) <https://bitbucket.org/actgenomics/actcnv_drug_ldt/src/v3.0.9/>`_

.. code-block:: console

    docker-compose -f /mnt/home/tomlin/dockercompose/ACTDrugV4_testing.yml

.. image:: _img/deployment_drugv4.png
    :width: 600px
    :align: center
    :alt: Successful deployment of ACTDrugV4 CNV container

----

-------------------
Pipeline execution
-------------------


When container has been deployed, use the following command triggers a job execution.

- Illumina: All four biomarker (ArmCNV, CNV, LGR, LOH) should trigger. Data will output in the respective Lv2 directory

.. code-block:: console

    python3 /tools/cnv_app/cnv_pipeline/cnv_pipeline.py -i [RunBarcode] --panel [panelID]

.. image:: _img/run_ilmn.png
    :width: 600px
    :align: center
    :alt: Execute Illumina cnv_pipeline

----

- Torrent: For Torrent pipelines, if panel ID is not provided, it will be inferred via the all_seq_list

.. code-block:: console
    
    python /home/CNV/script/ACTOnco_CNV_Onco2M7.py -i [RunBarcode] -b FFPE -p Onco2M7pv6

.. image:: _img/run_onco.png
    :width: 600px
    :align: center
    :alt: Execute ACTOnco cnv_pipeline

----

.. code-block:: console
    
    python /home/CNV/script/ACTDrugV4_CNV_PA027M1.py -i [RunBarcode] -b FFPE -p PA027M1

.. image:: _img/run_drugv4.png
    :width: 600px
    :align: center
    :alt: Execute ACTDrugV4 cnv_pipeline

----

-----------------
Troubleshoot
-----------------


Server DNS Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

Server needs to contain necessary DNS for API in /etc/hosts

.. image:: _img/dns.png
    :width: 600px
    :align: center
    :alt: output of /etc/hosts


Otherwise add the following to docker-compose

.. code-block:: YAML

    extra_hosts:
    
      - "actg-sso-back.actgenomics.com=192.168.6.8"
    
      - "actg-sso.actgenomics.com=192.168.6.8"
    
      - "lm-back.actgenomics.com=192.168.6.8"


Mount volumes exist
~~~~~~~~~~~~~~~~~~~~~

The mock directory of Lv1 and Lv2 contained test data for pipeline execution

.. image:: _img/mount_vol.png
    :width: 600px
    :align: center
    :alt: Volumes for Lv1 and Lv2 highlighted in YAML

----

Image building
~~~~~~~~~~~~~~~~~~~~

When building Illumina CNV container, it needs to contain an entry point as it works as a component in the entire pipeline.


The dockerfile to use when building production container: 

.. code-block:: console
    
    illumina_cnv_integration/cnv_df/docker_swarm/Dockerfile

.. image:: _img/build_swarm.png
    :width: 600px
    :align: center
    :alt: Correct startup message for Illumina container

----

For development and local testing, without crontab entrypoint:

.. code-block:: console

    illumina_cnv_integration/cnv_df/Dockerfile


Test new data / Debug sample run
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


When new data is required for testing, a mock directory can be created providing the sample data is already in production volumes
Generate text file with 1-column containing sample UUID.  Example UUID : AANB01_502_IDX703503_AA-25-10005

.. image:: _img/new_data1.png
    :width: 600px
    :align: center
    :alt: Example of samplelist file

----

Use custom script to generate mock folder. This directory will contain a physical copy of Lv1 Lv2 files with samples, which can be mounted to testing container

.. code-block:: console

    python3 /mnt/home/tomlin/dev_script/file_files.py \
        -f1 [Lv1 DIR] \
        -f2 [Lv2 DIR] \
        -i [File containing list of UUIDs] \
        -o [Output DIR]

.. image:: _img/new_data2.png
    :width: 600px
    :align: center
    :alt: Example of generating mock Lv1/Lv2 directory