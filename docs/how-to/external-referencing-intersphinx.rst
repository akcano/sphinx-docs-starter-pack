.. _how-to-external-referencing-intersphinx:

Link to external references with Intersphinx
============================================

This guide provides instructions on how to set up external references using the Intersphinx extension. 
Linking to external Sphinx-based documentation sets via hyperlinks is difficult to maintain. 
Intersphinx provides a more robust alternative, linking to sections of other Sphinx projects based on 
labels mapped out in an inventory list ``objects.inv``. To effectively use Intersphinx, sections in 
the target documentation set must have labels.

Setup
-----

In the ``conf.py`` file in your docs directory, add or enable ``sphinx.ext.intersphinx`` under ``extensions``:

.. code-block:: python
   :caption: conf.py

   extensions = [
      ...
       "sphinx.ext.intersphinx",
       ]

In the ``conf.py`` file, there is a section to map the Sphinx documentation sets referenced in the current 
documentation. Any new documentation set referenced in the text must be added here and contain a unique key. 
Mapped links have the format ``(target, inventory)``, where target is the URL of the Sphinx project and 
inventory indicates where the inventory file is located. To specify a non-standard location of the inventory, 
specify the path as the ``inventory`` parameter.

.. code-block:: python
   :caption: conf.py

   # Add configuration for intersphinx mapping
   # Map only the Sphinx documentation sets that you need to link to from your docs set.
   intersphinx_mapping = {
       'project-key': ('https://example.com', None)
   }

Check the external target labels
--------------------------------

To check external target page labels, either search the project source code 
manually or inspect the ``objects.inv`` file. In this file, each section has a 
unique label. To access the file, run::

    python -m sphinx.ext.intersphinx https://example.com/objects.inv

The appropriate labels are under ``std:label``.

Add references to the text
--------------------------

To add in-line references, follow this structure:

``:external+project_key:ref:`an_external_target```
