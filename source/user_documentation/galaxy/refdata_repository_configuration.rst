ELIXIR-Italy CVMFS documentation
================================

ELIXIR-Italy maintain two CVMFS repository, exploited by Laniakea.

.. csv-table::
   :header: "CVMFS", "Flavours supported", "folder tree" 
   :widths: auto
   :file: table_cvmfs.csv

A complete list of the reference data, with download link, is available `here <https://docs.google.com/spreadsheets/d/1l0dbaVuT4qiXMGevrYtkRDvsNa052WT8WbBg_dFpqFM/edit?usp=sharing>`_.

Default folders structure
-------------------------

The basic structure of the CVMFS repositories is the same. The repository directories are referred to the model organism genome different assemblies:

.. hlist::
   :columns: 2
   
   - at10
   - at9
   - dm2
   - dm3
   - dm6
   - hg18
   - hg19
   - hg38
   - mm10
   - mm8
   - mm9
   - sacCer1
   - sacCer2
   - sacCer3

Inside each assembly directory there is the ``genome.fa`` and the refseq ``gtf`` and ``gff`` downloaded from UCSC and the tools indeces:

-------
``bwa`` 
-------

It has been created using the default command 

.. code:: bash

   $ bwa index -a bwtsw genome.fa

-----------
``bowtie2``
-----------

It has been created using the default command 
  

.. code:: bash

   $ bowtie2-build

----------
``bowtie``
----------

Created using the default command
 
.. code:: bash

   $ bowtie-build

--------
``rsem``
--------

Created using the default command
 
.. code:: bash

   $ rsem-prepare-reference --gtf (.gtf) --transcript-to-gene-map (table.txt) --bowtie (.fa) <assembly-name> 


Additional folders
------------------

The two repositories hosts also spceific directories:

-------------------------------
``elixir-italy.covacs.refdata``
-------------------------------

**************
``annovar_db``
**************

Hosts the databases needed to perform CoVaCS pipeline downloaded from annovar repository using the annotate_variation.pl perl script.

*******************
``bed_file_covacs``
*******************

Hosts the bed files needed to perform CoVacs pipeline, the same bed files were present in the CINECA implementation of the CoVaCS pipeline.

************
``location``
************

Hosts the .loc file and the tool_data_table.xml file that will be used by galaxy CoVaCS flavours.

-------------------------------
``elixir-italy.galaxy.refdata``
-------------------------------

****************
``rRNAdatabase``
****************

Location of ribosomial RNA for sortmeRNA tool in galaxy RNA workbench flavour.

*********************
``index_GATK_bundle``
*********************

Location of genome indices for GATK toools for hg38 and hg19 assembly downloaded from GATK ftp bundle (https://software.broadinstitute.org/gatk/download/bundle).

************
``location``
************

Hosts the .loc file and the tool_data_table.xml file that will be used by galaxy RNA workbench, galaxy EPIGEN and galaxy GDC Somatic Variant flavours

CVMFS server details
--------------------

Since, cvmfs relies on OverlayFS or AUFS as default storage driver and Ubuntu 16.04 natively supports OverlayFS, it is used as default choice to create and populate the cvmfs server.

A resign script is located in ``/usr/local/bin/Cvmfs-stratum0-resign`` and the corresponding weekly cron job is set to ``/etc/cron.d/cvmfs_server_resign``.

Log file is located in ``/var/log/Cvmfs-stratum0-resign.log``.
