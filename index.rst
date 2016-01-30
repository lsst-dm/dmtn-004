..
  Content of technical report.

  See http://docs.lsst.codes/en/latest/development/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label
     :target: http://target.link/url

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-report-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

This document contains information about setting up and enabling debug tools correctly for Qserv developers inside Docker containers, although it applies generally to anyone who would like to add these tools. The intention is to have a similar debugging setup as one would in a bare metal machine with standard tools, and decouple idiosyncrasies associated with performing any work inside a container. 

This is a work-in-progress document and can/should be expanded by others when they have created a solution towards issues in this specific topic.

Wishlist of Debug Tools
=======================

A list of minimum tools that would be ideal to have inside a base Qserv image/container:

- ``gdb``
- ``lsof``
- ``gcore``
- ``pstack`` 

As of this writing, ``gdb``, ``lsof`` and ``gcore`` are included.

Enabling ``gdb``
================

In general, Docker containers will take on the host machine's settings in ``/proc/sys``. The Qserv Docker image and VM's at NCSA currently being used by the DM team are based on Ubuntu. Since Ubuntu 10.10, the default ``ptrace_scope`` setting has been changed for security reasons. It needs to be re-enabled in order for ``gdb`` to attach to any processes in the container. Details of the issue can be found `here <http://askubuntu.com/questions/41629/after-upgrade-gdb-wont-attach-to-process>`_ . On the host machine execute the following command:

.. prompt:: bash

   echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope

Generating Core Dump Files
==========================

Another issue experienced while testing Qserv with Docker containers was the absense of core files when a service in the stack fails, making it impossible to debug.

The core dump file location settings are controlled in ``/proc/sys/kernel/core_pattern``. In the Ubuntu VM host filesystem, this usually includes a path to Ubuntu's crash handling system **Apport**. However, this path does not exist inside the container, so the core dump files are lost. This is also an effect of Docker containers using the host's settings. To re-enable core dump files inside the container, set a valid path inside the file by executing the following command on the host machine:

.. prompt:: bash

   echo 'core.%e.%p' | sudo tee /proc/sys/kernel/core_pattern

This particular command produces a core dump file in the current working directory, but in principle one can choose any path that is valid in both the host and container filesystem. Another point to note is that the core dump file size limit should be non-zero:

.. prompt:: bash

   ulimit -c unlimited 
