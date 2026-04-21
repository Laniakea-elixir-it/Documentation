File structure
==============

The YAML parameter file has two sections: ``tabs`` and ``ìnputs``.

--------
``tabs``
--------

:Description:
	This section is optional, if set creates the listed tabs instead of the ``Input values`` one. It is possible to display each input in the desired tab, using the option ``tab`` in the input section. If not specified, all inputs will be displayed by default in the ``Ìnput values`` tab, as default behaviour.

:Example:
	::

	  # Set here the list of the tabs to be displayed
	  tabs: [ "tab_1", "tab_2"]
	  ...

----------
``inputs``
----------

:Description:
	The list of the inputs is mandatory. Each input must have the same name of the corresponding TOSCA template input value, to be correctly associated.

:Example:
	::
	
	  # Set here the list of the tabs to be displayed
	  tabs: [ "tab_1", "tab_2"]
	  
	  # Set here a new set of inputs to be displayed
	  inputs:
	  
	    first_input:
	      display_name: "<name to be displayed>"
	      tag_type: "<specific tag type for this input>"
	      description: "<description to desplayed>"
	      tab: "tab_1"
	
	    another_input:
	      display_name: "<name to be displayed>"
	      tag_type: "<specific tag type for this input>"
	      description: "<description to desplayed>"
	      tab: "tab_2"
	
	    ...

