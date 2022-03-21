##############################
Containers and Virtual Machine
##############################

-- crt-green

.. uml::
    :align: center

    ' Set theme
    !theme cerulean

    ' Elements
    actor "Application User" as User
    [Mail server] as Mail <<Mail>>
    
    package "Sample Application" {
        [Controller] <<Spring REST controllers>>
        [Service] <<Spring service>>
    }
    
    ' Connections
    Controller --> Service
    Service <-- GNU C Libraries
    GNU C Libraries <-- Kernel