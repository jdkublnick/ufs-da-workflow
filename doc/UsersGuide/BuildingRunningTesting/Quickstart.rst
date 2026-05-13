.. _QuickStart:

*****************
Quick Start Guide
*****************

This chapter provides quick-start instructions for building and running the
Unified Forecast System (UFS) DA Workflow. It summarizes the application
configurations and the common end-to-end run paths you will encounter in ufs-da-workflow. 

.. attention::

   These steps are designed for Level 1 systems: Ursa, Orion, Hercules,
   Gaea-C6, and Derecho. When using ecFlow on Ursa, run the ecFlow steps on
   the ``uecflow01`` node.

.. _app-configurations:

Application Configurations
==========================

Select the application with the ``APP`` setting in ``config.yaml`` and with the
``-a=[APP]`` option to ``app_build.sh`` when building workflow components.
See :ref:`component-configuration` for the full component tables.

.. list-table:: Supported forecast model applications
   :header-rows: 1
   :widths: 15 35 50

   * - APP
     - Coupling
     - Description
   * - ``S2SWA``
     - ATM/OCN/ICE/WAV/CHM
     - Atmosphere, ocean, sea ice, wave, and chemistry configuration used for
       GFS and GEFS-style applications. GOCART is excluded for GFS cases.
   * - ``S2SWAL``
     - ATM/OCN/ICE/WAV/CHM/LND
     - Fully coupled configuration with the Noah-MP land surface model.
   * - ``NG-GODAS``
     - DATM/OCN/ICE
     - Next Generation Global Ocean Data Assimilation System configuration.
   * - ``ATML``
     - ATM/LND
     - Atmosphere and land configuration used for Noah-MP land DA workflows.
   * - ``ATM``
     - ATM
     - Stand-alone atmospheric model configuration using FV3 and CCPP.

.. _quickstart-analysis-forecast:

Analysis and Forecast Workflow
==============================

Use this path when the workflow includes both analysis tasks, such as JEDI, and
forecast tasks, such as the UFS Weather Model.

#. Retrieve the ``develop`` branch of the UFS DA Workflow:

   .. code-block:: console

      git clone --recursive https://github.com/ufs-community/ufs-da-workflow
      cd ufs-da-workflow/sorc

   The path to the ``ufs-da-workflow`` directory is referred to as
   ``${HOMEufsda}`` in the commands below.

#. Build the executables. Select one of the following options:

   .. code-block:: console

      # Workflow components without JEDI-bundle or GDAS App
      ./app_build.sh -a=[APP]

      # Workflow components with JEDI-bundle
      ./app_build.sh -a=[APP] --jedi=bundle

      # JEDI-bundle only, without workflow components
      ./app_build.sh --jedi=bundle-only

      # Workflow components with GDAS App
      ./app_build.sh -a=[APP] --jedi=gdas

      # GDAS App only, without workflow components
      ./app_build.sh --jedi=gdas-only

      # Link FIX directories only, without building
      ./app_build.sh --fix-only

   Replace ``[APP]`` with ``S2SWA``, ``S2SWAL``, ``NG-GODAS``, ``ATML``, or
   ``ATM``.

   .. note::

      GDAS App is not available on Derecho.

#. Load the modules and Python environment for the workflow:

   .. code-block:: console

      cd ${HOMEufsda}
      module use modulefiles
      module load wflow_[workflow_manager]_[machine]

   Replace ``[workflow_manager]`` with ``ecflow`` or ``rocoto``. Replace
   ``[machine]`` with ``gaeac6``, ``hercules``, ``orion``, ``ursa``, or
   ``derecho``.

#. Copy a sample configuration and edit it for the experiment:

   .. code-block:: console

      cd ${HOMEufsda}/parm
      cp config_samples/config.[APP].[case_name].yaml config.yaml
      vim config.yaml

   Update the parameter values as needed, especially ``ACCOUNT``. If JEDI is
   installed in a separate location, set ``JEDI_BIN_PATH`` and the corresponding
   IODA converter path in ``config.yaml``; otherwise, the workflow scripts use
   the default paths.

#. Set up the case-specific experiment directory:

   .. code-block:: console

      ./setup_wflow_env.py

   To specify the platform explicitly:

   .. code-block:: console

      ./setup_wflow_env.py -p [machine]

#. Launch the workflow tasks.

   For ``WORKFLOW_MANAGER: ecflow``:

   .. code-block:: console

      cd ../../exp_case/ecf_server
      ./start_server.sh
      cd ../../exp_case/[EXP_CASE_NAME]/ecf
      ./begin_suite.sh
      ecflow_ui &

   ``[EXP_CASE_NAME]`` is set in ``config.yaml``. If the ecFlow server is
   already running, do not start it again. After all suites are complete, stop
   the server:

   .. code-block:: console

      cd ../../exp_case/ecf_server
      ./stop_server.sh

   For ``WORKFLOW_MANAGER: rocoto``:

   .. code-block:: console

      cd ../../exp_case/[EXP_CASE_NAME]
      ./automate_launch_script.py

   The automation script runs ``launch_rocoto_wflow.sh`` every 30 seconds by
   default. To use a 60-second interval:

   .. code-block:: console

      ./automate_launch_script.py -i 60

   To launch Rocoto manually instead:

   .. code-block:: console

      ./launch_rocoto_wflow.sh
      vim log.rocoto_launch

   Submit the manual launch script sequentially until all tasks are complete.

#. Check the generated result and log links:

   * ``com_dir``: symlink to the directory containing result files
   * ``log_dir``: symlink to the directory containing log files
   * ``tmp_dir``: symlink to the directory containing working directories

.. _quickstart-analysis-only:

Analysis-Only Workflow
======================

Use this path for a JEDI analysis workflow without the UFS Weather Model.

#. Retrieve the code and enter ``sorc``:

   .. code-block:: console

      git clone --recursive https://github.com/ufs-community/ufs-da-workflow
      cd ufs-da-workflow/sorc

#. Build DA utilities with either JEDI-bundle or GDAS App:

   .. code-block:: console

      ./app_build.sh --da-utils-only --jedi=bundle
      ./app_build.sh --da-utils-only --jedi=gdas

   .. note::

      GDAS App is not available on Derecho.

#. Load the workflow environment:

   .. code-block:: console

      cd ${HOMEufsda}
      module use modulefiles
      module load wflow_[workflow_manager]_[machine]

#. Copy and edit an analysis-only configuration:

   .. code-block:: console

      cd ${HOMEufsda}/parm
      cp config_samples/config.anal-only.[case_name].yaml config.yaml
      vim config.yaml

#. Set up and launch the experiment:

   .. code-block:: console

      ./setup_wflow_env.py

   The PDF reference describes analysis-only launch with ecFlow:

   .. code-block:: console

      cd ../../exp_case/ecf_server
      ./start_server.sh
      cd ../../exp_case/[EXP_CASE_NAME]/ecf
      ./begin_suite.sh
      ecflow_ui &

   Stop the ecFlow server after all relevant suites are complete:

   .. code-block:: console

      cd ../../exp_case/ecf_server
      ./stop_server.sh

.. _quickstart-forecast-only:

Forecast-Only Workflow
======================

Use this path for a stand-alone UFS Weather Model forecast without JEDI-bundle
or GDAS App.

#. Retrieve the code and enter ``sorc``:

   .. code-block:: console

      git clone --recursive https://github.com/ufs-community/ufs-da-workflow
      cd ufs-da-workflow/sorc

#. Build the workflow components:

   .. code-block:: console

      ./app_build.sh -a=[APP]

#. Load the workflow environment:

   .. code-block:: console

      cd ${HOMEufsda}
      module use modulefiles
      module load wflow_[workflow_manager]_[machine]

#. Copy and edit a forecast-only configuration:

   .. code-block:: console

      cd ${HOMEufsda}/parm
      cp config_samples/config.[APP].free-fcst.cycle.ecflow.yaml config.yaml
      vim config.yaml

#. Set up and launch the experiment with ecFlow:

   .. code-block:: console

      ./setup_wflow_env.py
      cd ../../exp_case/ecf_server
      ./start_server.sh
      cd ../../exp_case/[EXP_CASE_NAME]/ecf
      ./begin_suite.sh
      ecflow_ui &

   Stop the ecFlow server after all relevant suites are complete:

   .. code-block:: console

      cd ../../exp_case/ecf_server
      ./stop_server.sh
