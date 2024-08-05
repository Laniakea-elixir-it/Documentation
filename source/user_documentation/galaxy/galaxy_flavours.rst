Galaxy Flavours
================

Each Galaxy instance is customizable, through the web front-end, with different sets of pre installed tools (e.g. SAMtools, BamTools, Bowtie, MACS, RSEM, etc...), exploiting CONDA as default dependency resolver. New tools are automatically installed using the official GalaxyProject python library `Ephemeris <https://ephemeris.readthedocs.io/en/latest/index.html>`_.

Currently the following Galaxy flavours are available on Laniakea

------------------
``Galaxy minimal``
------------------

:Description:
	Galaxy production-grade server (Galaxy, PostgreSQL, NGINX, proFTPd, uWSGI).

:Reference data repository:
	**usegalaxy.org Galaxy reference data CVMFS repository**

-----------------
``Galaxy CoVaCS``
-----------------

:Description:
	Workflow for genotyping and variant annotation of whole genome/exome and target-gene sequencing data.

        For more information on CoVaCs Flavour visit this page: :doc:`galaxy_covacs`.

:Reference data repository:
        **ELIXIR-IT Galaxy CoVaCS reference data CVMFS repository**

:Reference:
	https://www.ncbi.nlm.nih.gov/pubmed/29402227

------------------------------
``Galaxy GDC Somatic Variant``
------------------------------

:Description:
	Port of the Genomic Data Commons (GDC) pipeline for the identification of somatic variants on whole exome/genome sequencing data.

	For more information on GDC Somatic Variant visit this page: :doc:`galaxy_gdc`.

:Reference data repository:
        **usegalaxy.org Galaxy reference data CVMFS repository**

:Reference:
	https://gdc.cancer.gov/node/246

------------------------
``Galaxy RNA workbench``
------------------------

:Description:
	More than 50 tools for RNA centric analysis.

:Reference data repository:
        **usegalaxy.org Galaxy reference data CVMFS repository**

:Reference:
	https://www.ncbi.nlm.nih.gov/pubmed/28582575

-----------------
``Galaxy Epigen``
-----------------

:Description:
	Based on Epigen project.

:Reference data repository:
        **usegalaxy.org Galaxy reference data CVMFS repository**

:Reference:
	`Galaxy Epigen server <http://159.149.160.87/galaxy>`_

Create new Galaxy flavours
--------------------------

New flavors can be created through yaml recipes with the list of tools. A tool list example can be found `here <https://raw.githubusercontent.com/indigo-dc/Galaxy-flavors-recipes/master/galaxy-testing/galaxy-testing-tool-list.yml>`_.

For more information on how to create a flavour visit this page: :doc:`galaxy_flavours_creation`.

References
----------

`Galaxy flavors <https://github.com/bgruening/docker-galaxy-stable#Extending-the-Docker-Image>`_

`Ephemeris <https://ephemeris.readthedocs.io/en/latest/>`_

`Ephemeris documentation <https://github.com/galaxyproject/ephemeris>`_

`Conda for Galaxy tools dependencies <https://docs.galaxyproject.org/en/master/admin/conda_faq.html>`_
